ARG MC_VERSION=1.21.8
ARG JAVA_MAJOR=21

# ---------- Build Stage ----------
FROM eclipse-temurin:${JAVA_MAJOR}-jdk AS build
ARG MC_VERSION
ENV MC_VERSION=${MC_VERSION}

WORKDIR /build

# Install dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
      git curl ca-certificates build-essential pkg-config && \
    rm -rf /var/lib/apt/lists/*

# Build mcrcon (optional RCON CLI tool)
RUN git clone --depth=1 https://github.com/Tiiffi/mcrcon.git /tmp/mcrcon && \
    make -C /tmp/mcrcon && strip /tmp/mcrcon/mcrcon && \
    mv /tmp/mcrcon/mcrcon /usr/local/bin/mcrcon && rm -rf /tmp/mcrcon

# Build Spigot for this exact Minecraft version
RUN curl -L -o BuildTools.jar \
      https://hub.spigotmc.org/jenkins/job/BuildTools/lastSuccessfulBuild/artifact/target/BuildTools.jar && \
    git config --global --add safe.directory /build && \
    java -jar BuildTools.jar --rev ${MC_VERSION}

# Verify build success
RUN test -f /build/spigot-${MC_VERSION}.jar || (echo "spigot-${MC_VERSION}.jar not found" && ls -al && exit 1)


# ---------- Runtime Stage ----------
FROM eclipse-temurin:${JAVA_MAJOR}-jre

# Re-declare MC_VERSION in this stage so it's usable in LABEL/ENV/COPY
ARG MC_VERSION
ENV MC_VERSION=${MC_VERSION}

LABEL org.opencontainers.image.title="Spigot Base" \
      org.opencontainers.image.version="${MC_VERSION}" \
      maintainer="LAN-slide"

# Create non-root user & directories
RUN useradd -m -s /bin/bash mc && \
    mkdir -p /mc && chown -R mc:mc /mc && \
    apt-get update && apt-get install -y --no-install-recommends \
      bash iproute2 sed gawk python3 unzip \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /mc

# Copy artifacts from build stage
COPY --from=build /build/spigot-${MC_VERSION}.jar /mc/spigot.jar
COPY --from=build /usr/local/bin/mcrcon /usr/local/bin/mcrcon

# Entrypoint + optional helpers (present in your repo)
COPY --chown=mc:mc advertise.py entrypoint.sh server.properties /mc/
RUN chmod +x /mc/entrypoint.sh

HEALTHCHECK --interval=30s --timeout=3s --retries=20 CMD bash -lc 'ss -lnt | grep -q ":25565"'

USER mc
ENTRYPOINT ["/mc/entrypoint.sh"]

