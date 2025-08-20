FROM debian:bookworm-slim AS base

# Common runtime deps
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl file gcc g++ make unzip \
        python3 python3-pip python3-dev python3-setuptools python3-wheel cython3 \
        libseccomp-dev bzip2 gzip \
        fp-compiler tini ca-certificates-java \
        $([ "$(arch)" = aarch64 ] && echo binutils-arm-linux-gnueabihf) && \
    rm -rf /var/lib/apt/lists/*

# --- Install OpenJDK 17 manually from Launchpad PPA builds ---
RUN if [ "$(arch)" = x86_64 ]; then \
        DEB_ARCH=amd64 && OPENJDK_BUILD=24567508; \
    else \
        DEB_ARCH=arm64 && OPENJDK_BUILD=24567509; \
    fi && \
    OPENJDK_FILES_URL="https://launchpad.net/~openjdk-security/+archive/ubuntu/ppa/+build/$OPENJDK_BUILD/+files" && \
    curl -LO $OPENJDK_FILES_URL/openjdk-17-jdk-headless_17.0.5+8-2ubuntu1~22.04_$DEB_ARCH.deb && \
    curl -LO $OPENJDK_FILES_URL/openjdk-17-jre-headless_17.0.5+8-2ubuntu1~22.04_$DEB_ARCH.deb && \
    dpkg -i *.deb && rm -f *.deb

# --- Install GCC 11 manually from Toolchain PPA builds ---
RUN if [ "$(arch)" = x86_64 ]; then \
        DEB_ARCH=amd64 && GCC_BUILD=23596444; \
    else \
        DEB_ARCH=arm64 && GCC_BUILD=23596445; \
    fi && \
    GCC_FILES_URL="https://launchpad.net/~ubuntu-toolchain-r/+archive/ubuntu/ppa/+build/$GCC_BUILD/+files" && \
    curl -LO $GCC_FILES_URL/cpp-11_11.3.0-1ubuntu1~22.04_$DEB_ARCH.deb && \
    curl -LO $GCC_FILES_URL/g++-11_11.3.0-1ubuntu1~22.04_$DEB_ARCH.deb && \
    curl -LO $GCC_FILES_URL/gcc-11_11.3.0-1ubuntu1~22.04_$DEB_ARCH.deb && \
    curl -LO $GCC_FILES_URL/gcc-11-base_11.3.0-1ubuntu1~22.04_$DEB_ARCH.deb && \
    curl -LO $GCC_FILES_URL/libasan6_11.3.0-1ubuntu1~22.04_$DEB_ARCH.deb && \
    curl -LO $GCC_FILES_URL/libgcc-11-dev_11.3.0-1ubuntu1~22.04_$DEB_ARCH.deb && \
    curl -LO $GCC_FILES_URL/libstdc++-11-dev_11.3.0-1ubuntu1~22.04_$DEB_ARCH.deb && \
    curl -LO $GCC_FILES_URL/libtsan0_11.3.0-1ubuntu1~22.04_$DEB_ARCH.deb && \
    dpkg -i *.deb && rm -f *.deb

# --- Install Kotlin (compiler only) ---
RUN curl -L -o kotlin.zip https://github.com/JetBrains/kotlin/releases/download/v1.7.21/kotlin-compiler-1.7.21.zip && \
    unzip kotlin.zip && mv kotlinc /opt/kotlin && rm kotlin.zip
ENV PATH="/opt/kotlin/bin:${PATH}"

# --- Install PyPy (from official binaries, supports amd64 & arm64) ---
RUN if [ "$(arch)" = x86_64 ]; then \
        PYPY_ARCH=linux64; \
    else \
        PYPY_ARCH=aarch64; \
    fi && \
    curl -L -o pypy.tar.bz2 https://downloads.python.org/pypy/pypy3.10-v7.3.11-$PYPY_ARCH.tar.bz2 && \
    tar -xjf pypy.tar.bz2 -C /opt/ && rm pypy.tar.bz2 && \
    ln -s /opt/pypy3.10-v7.3.11-$PYPY_ARCH/bin/pypy3 /usr/bin/pypy3

# --- Cleanup & user ---
RUN apt-get clean && rm -rf /var/lib/apt/lists/* && \
    useradd -m judge

ENTRYPOINT ["/usr/bin/tini", "--", "/code/run"]
