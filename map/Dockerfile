ARG MC_VERSION=1.21.10
FROM ghcr.io/lanslide-team/spigot-base:v${MC_VERSION}
#FROM minecraft-base:v${MC_VERSION}

WORKDIR /mc

COPY plugins/ ./plugins/
USER root
RUN chown -R mc:mc /mc
USER mc

ENTRYPOINT ["./entrypoint.sh"]