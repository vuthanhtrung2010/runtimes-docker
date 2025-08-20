FROM debian:bookworm-slim

# Always use noninteractive for CI builds
ENV DEBIAN_FRONTEND=noninteractive

# Install essentials
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl file gcc g++ make \
    python3-full python3-pip python3-dev python3-setuptools python3-wheel cython3 \
    libseccomp-dev bzip2 gzip tini ca-certificates-java \
    fp-compiler libxtst6 \
    && rm -rf /var/lib/apt/lists/*

# ------------------------
# Install OpenJDK 8 (Temurin)
# ------------------------
ARG TARGETARCH
RUN case "$TARGETARCH" in \
      amd64) ARCH=x64 ;; \
      arm64) ARCH=aarch64 ;; \
      *) echo "Unsupported arch: $TARGETARCH" && exit 1 ;; \
    esac && \
    curl -fsSL https://github.com/adoptium/temurin8-binaries/releases/latest/download/OpenJDK8U-jdk_${ARCH}_linux_hotspot.tar.gz \
    | tar -xz -C /opt && \
    ln -s /opt/jdk8*/bin/* /usr/local/bin/

# ------------------------
# Install PyPy (latest stable from pypy.org)
# ------------------------
RUN case "$TARGETARCH" in \
      amd64) ARCH=linux64 ;; \
      arm64) ARCH=aarch64 ;; \
      *) echo "Unsupported arch: $TARGETARCH" && exit 1 ;; \
    esac && \
    curl -fsSL https://downloads.python.org/pypy/pypy3.10-v7.3.16-${ARCH}.tar.bz2 \
    | tar -xj -C /opt && \
    ln -s /opt/pypy3.10-v7.3.16-${ARCH}/bin/pypy3 /usr/local/bin/pypy3 && \
    /usr/local/bin/pypy3 -m ensurepip

# ------------------------
# Add judge user
# ------------------------
RUN useradd -m judge

USER judge
WORKDIR /home/judge
