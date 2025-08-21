FROM vuthanhtrung2010/runtimes-tier2

USER root
ENV DEBIAN_FRONTEND=noninteractive

# Install extra system deps first (jq, unzip, ncurses, etc.)
RUN apt-get update && apt-get install -y --no-install-recommends \
    unzip xz-utils curl jq gnupg ca-certificates \
    libtinfo5 libncurses5 \
    gnucobol4 gnat gfortran tcl lua5.3 intercal php-cli gforth swi-prolog \
    pike8.0 sbcl gnustep-devel algol68g \
  && rm -rf /var/lib/apt/lists/*

# --- Dart SDK (stable latest) ---
RUN set -eux; \
    case "$(arch)" in \
      x86_64) DART_ARCH=x64 ;; \
      aarch64) DART_ARCH=arm64 ;; \
      *) echo "Unsupported arch $(arch)" && exit 1 ;; \
    esac; \
    VER=$(curl -sSL https://storage.googleapis.com/dart-archive/channels/stable/release/latest/VERSION | jq -r .version); \
    curl -fsSL -o /tmp/dart.zip "https://storage.googleapis.com/dart-archive/channels/stable/release/${VER}/sdk/dartsdk-linux-${DART_ARCH}-release.zip"; \
    unzip -q /tmp/dart.zip -d /opt; \
    rm /tmp/dart.zip; \
    chmod -R go+rx /opt/dart-sdk

# --- Node.js 20 LTS ---
RUN mkdir -p /etc/apt/keyrings && \
    curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key \
      | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg && \
    echo 'deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_20.x nodistro main' \
      > /etc/apt/sources.list.d/nodesource.list && \
    apt-get update && apt-get install -y --no-install-recommends nodejs && \
    rm -rf /var/lib/apt/lists/*

# --- Groovy (latest stable zip) ---
RUN curl -fsSL -o /tmp/groovy.zip "$(curl -s https://groovy.apache.org/download.html | \
      perl -ne 'if(/(["'\''])(https:[^"'\''"]+-binary-[\d.]+\.zip)\1/){print $2; exit}')" && \
    unzip -q /tmp/groovy.zip -d /opt && \
    mv /opt/groovy-* /opt/groovy && \
    rm -rf /tmp/groovy.zip

# --- Kotlin (compiler latest stable) ---
RUN curl -fsSL -o /tmp/kotlin.zip "$(curl -s https://api.github.com/repos/JetBrains/kotlin/releases | \
      jq -r '[.[] | select(.prerelease|not)][0].assets | map(select(.name|startswith("kotlin-compiler") and endswith(".zip")))[0].browser_download_url')" && \
    unzip -q /tmp/kotlin.zip -d /opt && \
    mv /opt/kotlinc /opt/kotlin && \
    rm -f /tmp/kotlin.zip

# --- TProlog (x86_64 only) ---
RUN if [ "$(arch)" = x86_64 ]; then \
      curl -fsSL -o /tmp/tprolog.zip "https://github.com/yingted/OpenTuring/releases/download/v1.0.0-beta/tprolog-v1.0.0-beta.zip" && \
      unzip -q /tmp/tprolog.zip -d /opt && \
      rm /tmp/tprolog.zip; \
    fi

# --- Zig 0.6.0 (very old, pinned) ---
RUN mkdir -p /opt/zig && \
    curl -fsSL "https://ziglang.org/download/0.6.0/zig-linux-$(arch)-0.6.0.tar.xz" \
      | tar -xJ -C /opt/zig --strip-components=1

# --- Lean4 (x86_64 only) ---
RUN if [ "$(arch)" = x86_64 ]; then \
      curl -fsSL -o /tmp/lean.zip "$(curl -s https://api.github.com/repos/leanprover/lean4/releases | \
        jq -r '[.[] | select(.prerelease|not)][0].assets | map(select(.name|endswith("-linux.zip")))[0].browser_download_url')" && \
      unzip -q /tmp/lean.zip -d /opt && \
      mv /opt/lean-* /opt/lean && \
      rm -rf /tmp/lean.zip; \
    fi

# Final PATH
ENV PATH="/opt/dart-sdk/bin:/opt/tprolog:/opt/groovy/bin:/opt/kotlin/bin:/opt/zig:/opt/lean/bin:${PATH}"
