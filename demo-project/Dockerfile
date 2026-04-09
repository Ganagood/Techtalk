FROM ubuntu:24.04

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y --no-install-recommends \
    bash \
    build-essential \
    ca-certificates \
    curl \
    git \
    openjdk-21-jre-headless \
    python3 \
    unzip \
   vim-tiny \
    zip \
 && ln -sf /usr/bin/vim.tiny /usr/local/bin/v \
 && ln -sf /usr/bin/vim.tiny /usr/local/bin/vim \
 && rm -rf /var/lib/apt/lists/*

RUN arch="$(dpkg --print-architecture)" \
 && case "$arch" in \
      amd64) bazelisk_arch=amd64 ;; \
      arm64) bazelisk_arch=arm64 ;; \
      *) echo "Unsupported architecture: $arch" >&2; exit 1 ;; \
    esac \
 && curl -fsSL "https://github.com/bazelbuild/bazelisk/releases/download/v1.21.0/bazelisk-linux-${bazelisk_arch}" -o /usr/local/bin/bazel \
 && chmod +x /usr/local/bin/bazel

WORKDIR /workspace

CMD ["/bin/bash"]
