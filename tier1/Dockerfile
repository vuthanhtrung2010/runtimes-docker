FROM alpine:latest

RUN echo "http://dl-cdn.alpinelinux.org/alpine/edge/testing" >> /etc/apk/repositories && \
    apk update && \
    apk add --no-cache \
        curl file gcc g++ musl-dev python3 py3-pip python3-dev py3-setuptools py3-wheel cython libseccomp-dev bzip2 gzip \
        fpc openjdk11-jdk openjdk11-jre util-linux \
        tini ca-certificates binutils shadow && \
    mkdir -p /etc/perl && \
    adduser -D -s /bin/sh judge

ENTRYPOINT ["/sbin/tini", "--", "sh", "/code/run"]
