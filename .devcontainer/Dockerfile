FROM mcr.microsoft.com/vscode/devcontainers/base:0-buster
ENV DEBIAN_FRONTEND=noninteractive

# Requirements: Hugo, Zsh, Git (wget serves to pull Oh-My-Zsh)
run apt-get update \
  && apt-get install -y --no-install-recommends hugo zsh wget git \
  && apt-get clean -y && rm -rf /var/lib/apt/lists/* \
  && chsh -s $(which zsh)
