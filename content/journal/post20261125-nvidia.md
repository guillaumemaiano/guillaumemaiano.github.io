---
title: "REX: Running a GTX 1080 on Fedora 45 (Kernel 7.3) with NVIDIA 580xx"
date: 2026-11-25T14:00:00+01:00
draft: false
tags: ["linux", "fedora", "nvidia", "kernel", "rex", "debugging"]
categories: ["Devlog", "REX"]
summary: "Diagnosing and fixing the NVIDIA 580xx akmods open-kernel module mismatch on a Pascal GTX 1080 running Fedora 45 (kernel 7.3.4)."
---

**Environment:** Workstation *Terra* — Fedora 45 (`7.3.4-300.fc45.x86_64`), EVGA GeForce GTX 1080 (GP104, `10de:1b80`) alongside an integrated Intel GPU (`i915`, Skylake `1912`).

Keeping a 2016 Pascal GPU running smoothly on a bleeding-edge Linux kernel (`7.3.4`) occasionally requires digging into `akmods` packaging internals. Here is a full post-mortem on separating overlapping hardware/USB issues from a subtle RPM Fusion spec behavior—and getting the proprietary NVIDIA driver building cleanly again.

## 1. Initial Symptoms: GPU Traps and USB Drops

The workstation was experiencing UI freezes, `localsearch-3` coredumps, and PipeWire audio dropouts. Checking `dmesg` and `journalctl` revealed three distinct issues compounding each other:

1. **`nouveau` execution traps:** Without a working proprietary module, `nvidia-fallback.service` loaded `nouveau`, which hit `gr: TRAP (GPC0–3/PROP trap: [RT_HEIGHT_OVERRUN])` errors under load.
2. **Cross-GPU scanout copies:** The monitors were split between the integrated Intel GPU (`i915`) and the GTX 1080, forcing unnecessary cross-GPU frame copies.
3. **Monitor USB hub instability:** A Røde Podcaster microphone plugged into the Iiyama monitor's USB-C passthrough was triggering repeated `usb 3-1` errors (`-110` / `-32`), causing cascading PipeWire device drops.

**Hardware cleanup first:** Wiring both monitors (`DP-4` and `HDMI-A-2`) directly into the GTX 1080 and moving the Røde Podcaster directly onto a motherboard USB port immediately eliminated the USB disconnects and `i915` scanout overhead.

## 2. The Root Cause: `kernel-open` on a Pascal GPU

Even after blacklisting `nouveau` and `nova_core` on the kernel command line (`rd.driver.blacklist=nouveau,nova_core modprobe.blacklist=nouveau,nova_core`), the RPM Fusion `akmod-nvidia-580xx` package (`3:580.196.07-1.fc45`) failed to initialize the card—`nvidia` rejected probing the GPU with error `-1`, leaving the desktop in fallback mode.

Checking the compiled module's license exposed why:

```bash
modinfo -F license /lib/modules/7.3.4-300.fc45.x86_64/extra/nvidia-580xx/nvidia.ko*
# Output: Dual MIT/GPL
```

The `akmods` build had compiled the **`kernel-open`** tree rather than the closed proprietary kernel module. NVIDIA's open kernel modules rely on the **GSP** (*GPU System Processor*), a hardware microcontroller introduced with Turing (RTX 2000 series). Because the GTX 1080 is **Pascal (GP104)** and has no GSP, `NVRM` refuses to probe the device when loaded with the open module.

## 3. Why `akmods --rebuild` Kept Looping

Simply running `akmods --force` or `akmods --rebuild` repeated the exact same failure. Inspecting the `%prep` stage inside `/usr/src/akmods/nvidia-580xx-kmod.latest` showed why:

```spec
mv kernel kernel-closed
mv kernel-open kernel
```

The spec file unconditionally moved the proprietary `kernel` directory aside to `kernel-closed` and replaced it with `kernel-open` before compiling. To get the GTX 1080 working, I needed to bypass that directory swap and test whether the original proprietary `kernel` tree still compiled cleanly against kernel `7.3.4`.

## 4. The Fix: Patching the Spec and Building Locally

Rather than mutating system files in `/usr/src`, the clean approach is to unpack the `akmods` source RPM in a temporary staging directory, comment out the directory swap in the `.spec` file, and build a local binary `kmod` RPM.

### Stage and patch the `.spec`

```bash
rm -rf ~/tmp/nvidia-580xx-src ~/tmp/rpmbuild/SOURCES ~/tmp/kmod-check && \
mkdir -p ~/tmp/nvidia-580xx-src ~/tmp/rpmbuild/SOURCES ~/tmp/kmod-check && \
cd ~/tmp/nvidia-580xx-src && \
rpm2cpio /usr/src/akmods/nvidia-580xx-kmod.latest | cpio -idmv && \
sed -i 's/^\s*mv kernel kernel-closed/# &/; s/^\s*mv kernel-open kernel/# &/' *.spec && \
find . -maxdepth 1 -type f ! -name '*.spec' -exec cp -f {} ~/tmp/rpmbuild/SOURCES/ \;
```

### Build the binary RPM and verify the module license

```bash
rpmbuild -bb --define "_topdir $HOME/tmp/rpmbuild" \
  --define "kernels 7.3.4-300.fc45.x86_64" \
  --target x86_64 *.spec && \
cd ~/tmp/kmod-check && \
rpm2cpio ~/tmp/rpmbuild/RPMS/x86_64/kmod-nvidia-580xx-7.3.4-300.fc45.x86_64-*.rpm | \
  cpio -idmv './lib/modules/*/extra/nvidia-580xx/nvidia.ko*' && \
modinfo -F license ./lib/modules/7.3.4-300.fc45.x86_64/extra/nvidia-580xx/nvidia.ko*
```

This time, `modinfo` returned `NVIDIA`, confirming that the proprietary kernel tree compiles cleanly against `7.3.4-300.fc45.x86_64` without extra kernel compatibility patches.

### Install, rebuild initramfs, and clean up

```bash
sudo dnf install --assumeyes --setopt=localpkg_gpgcheck=0 \
  ~/tmp/rpmbuild/RPMS/x86_64/kmod-nvidia-580xx-7.3.4-300.fc45.x86_64-580.196.07-1.fc45.x86_64.rpm && \
sudo dracut --force --kver 7.3.4-300.fc45.x86_64 && \
rm -rf ~/tmp/nvidia-580xx-src ~/tmp/rpmbuild ~/tmp/kmod-check
```

*(Passing the full kernel release string `7.3.4-300.fc45.x86_64` to `dracut --kver` is required so `dracut` locates the matching `/lib/modules/` tree.)*

## 5. Verification & Maintenance Notes

After rebooting, `lspci -nnk -s 01:00.0`, `nvidia-smi`, and `/sys/module/nvidia_drm/parameters/modeset` (`Y`) confirmed that the GTX 1080 was properly bound to `nvidia` `580.196.07` with `nvidia_drm`, `nvidia_modeset`, and `nvidia_uvm` active.

Two things to keep in mind until upstream RPM Fusion updates the `580xx` spec behavior for pre-Turing cards:

* **Check the license after driver or kernel updates:** If a future `akmod-nvidia-580xx` update reverts to `Dual MIT/GPL`, it has staged `kernel-open` again and will fail to probe on Pascal.
* **Keep a known-good kernel in GRUB:** Retain the `7.3.4-300.fc45.x86_64` boot entry as a fallback in case a newer kernel introduces API changes that break `580.196.07` `kernel-closed` compilation.