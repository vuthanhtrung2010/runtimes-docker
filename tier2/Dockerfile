# Tier 2 builds ON TOP of your tier1 image to reuse layers
FROM vuthanhtrung2010/runtimes-tier1

USER root
ENV DEBIAN_FRONTEND=noninteractive

# Core extras for tier2 (and tools needed by later installers)
RUN apt-get update && apt-get install -y --no-install-recommends \
      jq apt-transport-https dirmngr gnupg ca-certificates xz-utils \
      $( [ "$(dpkg --print-architecture)" = "amd64" ] && echo libc6-dev-i386 ) \
      clang llvm ghc golang racket ruby scala nasm chicken-bin \
      make m4 patch unzip libgmp-dev pkg-config git \
  && rm -rf /var/lib/apt/lists/*

# JDK 22 (keep a "new" JDK in addition to JDK8 from tier1) – use Adoptium to avoid debian:sid
RUN case "$(dpkg --print-architecture)" in amd64) ARCH=x64;; arm64) ARCH=aarch64;; *) echo unsupported; exit 1;; esac && \
    curl -fsSL "https://github.com/adoptium/temurin22-binaries/releases/download/jdk-22.0.2+9/OpenJDK22U-jdk_${ARCH}_linux_hotspot_22.0.2_9.tar.gz" \
    | tar -xz -C /opt && \
    JDK22_DIR="$(find /opt -maxdepth 1 -type d -name 'jdk-22*' | head -n1)" && \
    ln -sf "${JDK22_DIR}/bin/"* /usr/local/bin/

# OPAM / OCaml (matches your original flow)
RUN export OPAMYES=1 OPAMJOBS="$(($(nproc)+2))" && \
    curl -fsSL https://raw.githubusercontent.com/ocaml/opam/master/shell/install.sh | sh -s -- --no-backup && \
    runuser -u judge -- opam init --shell-setup --disable-sandboxing --bare && \
    runuser -u judge -- opam switch create dmoj --packages=ocaml.5.0.0,ocaml-option-flambda && \
    runuser -u judge -- opam install base core stdio zarith && \
    runuser -u judge -- opam clean && rm -rf /home/judge/.opam/repo

# Keep your legacy layout: /opt/pypy2 and /opt/pypy3 in addition to tier1’s PyPy3
RUN set -eux; arch="$(dpkg --print-architecture)"; \
    case "$arch" in amd64) PARCH=linux64;; arm64) PARCH=aarch64;; *) echo unsupported; exit 1;; esac; \
    mkdir -p /opt/pypy2 /opt/pypy3; \
    curl -fsSL "https://downloads.python.org/pypy/pypy2.7-v7.3.11-${PARCH}.tar.bz2" | tar -xj -C /opt/pypy2 --strip-components=1; \
    /opt/pypy2/bin/pypy -m compileall || true; \
    chmod a+rx /opt/pypy2/lib /opt/pypy2/lib/*.so* || true; \
    rm -f /opt/pypy2/bin/python*; \
    curl -fsSL "https://downloads.python.org/pypy/pypy3.10-v7.3.11-${PARCH}.tar.bz2" | tar -xj -C /opt/pypy3 --strip-components=1; \
    /opt/pypy3/bin/pypy -m compileall || true; \
    rm -f /opt/pypy3/bin/python*

# Rust toolchain and warm Cargo registry (same prefetch trick you used)
RUN runuser -u judge -- bash -lc 'curl https://sh.rustup.rs -sSf | sh -s -- -y' && \
    mkdir -p /tmp/rust && cd /tmp/rust && \
    curl -sL https://raw.githubusercontent.com/VNOI-Admin/judge-server/master/dmoj/executors/RUST.py \
      | sed -n '/^CARGO_TOML/,/^"""/p' | sed '1d;$d' > Cargo.toml && \
    mkdir src && \
    curl -sL https://raw.githubusercontent.com/VNOI-Admin/judge-server/master/dmoj/executors/RUST.py \
      | sed -n '/^TEST_PROGRAM/,/^"""/p' | sed '1d;$d' > src/main.rs && \
    chown -R judge: /tmp/rust && \
    runuser -u judge -- /home/judge/.cargo/bin/cargo fetch && \
    rm -rf /tmp/rust

# DMD (x86_64 only), same installer
RUN if [ "$(dpkg --print-architecture)" = "amd64" ]; then \
      curl -fsS https://dlang.org/install.sh | bash -s dmd --path /opt/dlang && \
      mv /opt/dlang/dmd-*/* /opt/dlang && rmdir /opt/dlang/dmd-*; \
    fi

# Mono + Roslyn + v8dmoj repo with chicken conflict workaround
RUN gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF && \
    gpg --export --armor 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF > /usr/share/keyrings/mono-official-archive-keyring.asc && \
    echo "deb [signed-by=/usr/share/keyrings/mono-official-archive-keyring.asc] https://download.mono-project.com/repo/debian stable-buster main" \
      > /etc/apt/sources.list.d/mono-official-stable.list && \
    curl -fsSL https://dmoj.ca/dmoj-apt.key -o /usr/share/keyrings/dmoj-keyring.asc && \
    echo 'deb [signed-by=/usr/share/keyrings/dmoj-keyring.asc] https://apt.dmoj.ca/ bullseye main' > /etc/apt/sources.list.d/dmoj.list && \
    (echo 'Package: *'; echo 'Pin: origin download.mono-project.com'; echo 'Pin-Priority: 990') > /etc/apt/preferences.d/mono && \
    apt-get update && \
    dpkg-divert --package mono-roslyn --divert /usr/bin/chicken-csc --rename /usr/bin/csc && \
    dpkg-divert --package mono-roslyn --divert /usr/bin/chicken-csi --rename /usr/bin/csi && \
    (cd /tmp && \
      apt download mono-roslyn && \
      dpkg-deb -R mono-roslyn_*.deb roslyn/ && \
      sed -i -e '/^Conflicts: chicken-bin/d' roslyn/DEBIAN/control && \
      dpkg-deb -b roslyn mono-roslyn_no_conflict.deb && \
      rm -rf roslyn/ && \
      apt-get install -y --no-install-recommends mono-devel ./mono-roslyn_no_conflict.deb mono-vbnc fsharp v8dmoj && \
      rm -f *.deb) && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

ENV PATH="/opt/dlang/linux/bin64:/home/judge/.cargo/bin:/opt/pypy2/bin:/opt/pypy3/bin:/opt/dart-sdk/bin:${PATH}"
