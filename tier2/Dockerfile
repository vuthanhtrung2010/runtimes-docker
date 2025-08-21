FROM debian:bookworm-slim AS base

ENV DEBIAN_FRONTEND=noninteractive

# Core dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl file gcc g++ make \
    python3-full python3-pip python3-dev python3-setuptools python3-wheel \
    cython3 libseccomp-dev bzip2 gzip tini ca-certificates-java \
    fp-compiler libxtst6 \
  && rm -rf /var/lib/apt/lists/*

# --- Install OpenJDK 8 (Temurin 8u452) ---
ARG TARGETARCH
RUN case "$TARGETARCH" in \
      amd64) ARCH=x64; FNAME="OpenJDK8U-jdk_x64_linux_hotspot_8u452b09.tar.gz"; ;; \
      arm64) ARCH=aarch64; FNAME="OpenJDK8U-jdk_aarch64_linux_hotspot_8u452b09.tar.gz"; ;; \
      *) echo "Unsupported arch: $TARGETARCH" && exit 1 ;; \
    esac && \
    curl -fsSL "https://github.com/adoptium/temurin8-binaries/releases/download/jdk8u452-b09/${FNAME}" \
      | tar -xz -C /opt && \
    JDK_DIR=$(find /opt -maxdepth 1 -type d -name "jdk8u*") && \
    ln -s "${JDK_DIR}/bin/"* /usr/local/bin/

# --- Install PyPy 3.11 (7.3.20) ---
RUN case "$TARGETARCH" in \
      amd64) PYPY_ARCH=linux64 ;; \
      arm64) PYPY_ARCH=aarch64 ;; \
      *) echo "Unsupported arch: $TARGETARCH" && exit 1 ;; \
    esac && \
    curl -fsSL "https://downloads.python.org/pypy/pypy3.11-v7.3.20-${PYPY_ARCH}.tar.bz2" \
      | tar -xj -C /opt && \
    PYPY_DIR=$(find /opt -maxdepth 1 -type d -name "pypy3.11-v7.3.20-*") && \
    ln -s "${PYPY_DIR}/bin/pypy3" /usr/local/bin/pypy3 && \
    ln -s "${PYPY_DIR}/bin/pip3" /usr/local/bin/pip3 && \
    pypy3 -m ensurepip && \
    pip3 install --upgrade pip setuptools wheel

# Non-root judge user
RUN useradd -m judge
USER judge
WORKDIR /home/judge

ENTRYPOINT ["/usr/bin/tini", "--", "/code/run"]
