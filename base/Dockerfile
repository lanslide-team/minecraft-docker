ARG MC_VERSION=26.3
ARG JAVA_MAJOR=26

# ---------- Build Stage ----------
FROM eclipse-temurin:${JAVA_MAJOR}-jdk AS build
ARG MC_VERSION
ENV MC_VERSION=${MC_VERSION}

WORKDIR /build

# Install dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
      git curl ca-certificates build-essential pkg-config jq && \
    rm -rf /var/lib/apt/lists/*

# Build mcrcon (optional RCON CLI tool)
RUN git clone --depth=1 https://github.com/Tiiffi/mcrcon.git /tmp/mcrcon && \
    make -C /tmp/mcrcon && strip /tmp/mcrcon/mcrcon && \
    mv /tmp/mcrcon/mcrcon /usr/local/bin/mcrcon && rm -rf /tmp/mcrcon

RUN USER_AGENT="LAN-slide-Minecraft-Server/1.0 (https://lanslide.com.au)" && \
    PAPER_URL=$(curl -fsSL \
      -H "User-Agent: ${USER_AGENT}" \
      "https://fill.papermc.io/v3/projects/paper/versions/${MC_VERSION}/builds" \
      | jq -r '.[0].downloads."server:default".url') && \
    echo "Downloading Paper ${MC_VERSION}: ${PAPER_URL}" && \
    test "${PAPER_URL}" != "null" && \
    curl -fsSL \
      -H "User-Agent: ${USER_AGENT}" \
      -o /build/paper.jar \
      "${PAPER_URL}"

RUN test -s /build/paper.jar || (echo "Paper ${MC_VERSION} download failed" && exit 1)

# ---------- Runtime Stage ----------
FROM eclipse-temurin:${JAVA_MAJOR}-jre

# Re-declare MC_VERSION in this stage so it's usable in LABEL/ENV/COPY
ARG MC_VERSION
ENV MC_VERSION=${MC_VERSION}

LABEL org.opencontainers.image.title="Paper Base" \
      org.opencontainers.image.version="${MC_VERSION}" \
      maintainer="LAN-slide"

ARG PUID=1000
ARG PGID=1000

# Create non-root user & directories
RUN usermod -l mc ubuntu && \
    groupmod -n mc ubuntu && \
    mkdir -p /mc && chown -R $PUID:$PGID /mc && \
    apt-get update && apt-get install -y --no-install-recommends \
        bash iproute2 sed gawk python3 unzip wget yq && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /mc

# Copy artifacts from build stage
COPY --from=build /build/paper*.jar /mc/paper.jar
COPY --from=build /usr/local/bin/mcrcon /usr/local/bin/mcrcon

# Entrypoint + optional helpers (present in your repo)
COPY --chown=mc:mc advertise.py entrypoint.sh server.properties /mc/
COPY --chown=mc:mc config /mc/config/
RUN chmod +x /mc/entrypoint.sh

HEALTHCHECK --interval=30s --timeout=3s --retries=20 CMD bash -lc 'ss -lnt | grep -q ":25565"'

ENTRYPOINT ["/mc/entrypoint.sh"]
