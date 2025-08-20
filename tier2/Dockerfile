FROM tier1 AS tier2
ARG TARGETARCH

# -------------------------
# Install GCC 11 (from Ubuntu PPA builds)
# -------------------------
RUN set -eux; \
    case "$TARGETARCH" in \
      amd64) DEB_ARCH=amd64; GCC_BUILD=23596444 ;; \
      arm64) DEB_ARCH=arm64; GCC_BUILD=23596445 ;; \
      *) echo "Unsupported arch: $TARGETARCH" && exit 1 ;; \
    esac && \
    GCC_URL="https://launchpad.net/~ubuntu-toolchain-r/+archive/ubuntu/ppa/+build/${GCC_BUILD}/+files"; \
    for pkg in cpp-11 g++-11 gcc-11 gcc-11-base libasan6 libgcc-11-dev libstdc++-11-dev libtsan0; do \
      curl -fsSL -O ${GCC_URL}/${pkg}_11.3.0-1ubuntu1~22.04_${DEB_ARCH}.deb; \
    done && \
    dpkg -i *.deb && rm -f *.deb

# -------------------------
# Install OpenJDK 8 (Temurin, pinned release)
# -------------------------
RUN set -eux; \
    case "$TARGETARCH" in \
      amd64) FILE="OpenJDK8U-jdk_x64_linux_hotspot_8u452b09.tar.gz" ;; \
      arm64) FILE="OpenJDK8U-jdk_aarch64_linux_hotspot_8u452b09.tar.gz" ;; \
      *) echo "Unsupported arch: $TARGETARCH" && exit 1 ;; \
    esac && \
    curl -fsSL "https://github.com/adoptium/temurin8-binaries/releases/download/jdk8u452-b09/${FILE}" \
      | tar -xz -C /opt && \
    ln -s /opt/jdk8*/bin/* /usr/local/bin/

# Install PyPy (3.11 v7.3.20)
RUN case "$TARGETARCH" in \
      amd64) PYPY_ARCH=linux64 ;; \
      arm64) PYPY_ARCH=aarch64 ;; \
      *) echo "Unsupported arch: $TARGETARCH" && exit 1 ;; \
    esac && \
    echo "Installing PyPy for $PYPY_ARCH" && \
    curl -fsSL "https://downloads.python.org/pypy/pypy3.11-v7.3.20-${PYPY_ARCH}.tar.bz2" \
      | tar -xj -C /opt && \
    PYPY_DIR=$(find /opt -maxdepth 1 -type d -name "pypy3.11-v7.3.20-*") && \
    ln -s "${PYPY_DIR}/bin/pypy3" /usr/local/bin/pypy3 && \
    ln -s "${PYPY_DIR}/bin/pip3" /usr/local/bin/pip3 && \
    pypy3 -m ensurepip && \
    pip3 install --upgrade pip setuptools wheel

# -------------------------
# Final user + entrypoint
# -------------------------
RUN useradd -m judge
ENTRYPOINT ["/usr/bin/tini", "--", "/code/run"]
