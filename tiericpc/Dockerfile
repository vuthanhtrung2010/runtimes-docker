FROM debian:sid-slim AS base

# Add bullseye + experimental repos for compatibility
RUN echo deb http://deb.debian.org/debian/ bullseye main > /etc/apt/sources.list.d/bullseye.list && \
    echo deb http://security.debian.org/debian-security/ bullseye-security main >> /etc/apt/sources.list.d/bullseye.list && \
    echo deb http://deb.debian.org/debian/ experimental main > /etc/apt/sources.list.d/experimental.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
        curl file gcc g++ python3-full python3-pip python3-dev python3-setuptools python3-wheel cython3 \
        libseccomp-dev bzip2 gzip unzip \
        fp-compiler tini ca-certificates-java \
        openjdk-17-jdk-headless openjdk-17-jre-headless \
        $([ "$(arch)" = aarch64 ] && echo binutils-arm-linux-gnueabihf) && \
    apt-get clean && rm -rf /var/lib/apt/lists/* && \
    useradd -m judge

# Install Kotlin (latest release from GitHub)
RUN set -eux; \
    KOTLIN_LATEST_URL=$(curl -s https://api.github.com/repos/JetBrains/kotlin/releases/latest \
        | grep -oP '"browser_download_url":\s*"\K[^"]*kotlin-compiler-[0-9.]+\.zip') && \
    curl -L -o kotlin.zip "$KOTLIN_LATEST_URL" && \
    unzip kotlin.zip && mv kotlinc /opt/kotlin && rm kotlin.zip
ENV PATH="/opt/kotlin/bin:${PATH}"

# Install PyPy3 (latest prebuilt binary, like tier2)
RUN if [ "$(arch)" = x86_64 ]; then PYPY_ARCH=linux64; else PYPY_ARCH="$(arch)"; fi && \
    mkdir /opt/pypy3 && curl -L "$(curl https://pypy.org/download.html \
        | grep "/pypy3.*$PYPY_ARCH" | head -n1 | cut -d'"' -f4)" \
        | tar xj -C /opt/pypy3 --strip-components=1 && \
    /opt/pypy3/bin/pypy -m compileall && \
    rm -f /opt/pypy3/bin/python*
ENV PATH="/opt/pypy3/bin:${PATH}"

ENTRYPOINT ["/usr/bin/tini", "/code/run"]
