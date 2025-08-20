FROM alpine:latest

RUN echo "http://dl-cdn.alpinelinux.org/alpine/edge/testing" >> /etc/apk/repositories && \
    apk update && \
    apk add --no-cache \
        curl file gcc g++ musl-dev python3 py3-pip python3-dev py3-setuptools py3-wheel python2 cython libseccomp-dev bzip2 gzip \
        fpc openjdk8-jdk-headless openjdk8-jre-headless \
        tini ca-certificates binutils shadow && \
    mkdir -p /etc/perl && \
    adduser -D -s /bin/sh judge

ENTRYPOINT ["/usr/bin/tini", "/code/run"]
