FROM mcr.microsoft.com/vscode/devcontainers/base:0-buster
ENV DEBIAN_FRONTEND=noninteractive

# Requirements: Hugo, Zsh, Git (wget serves to pull Oh-My-Zsh)
RUN apt-get update \
  && apt-get install -y --no-install-recommends hugo zsh wget git \
  && apt-get clean -y && rm -rf /var/lib/apt/lists/* \
  && chsh -s $(which zsh)

# While the shell is now ZSH in the codespace, I also want to use ZSH for my RUN commands
SHELL ["/usr/bin/zsh", "-c"]

# Make dialog available for further use
ENV DEBIAN_FRONTEND=dialog
