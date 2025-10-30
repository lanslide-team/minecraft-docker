ARG MC_VERSION=1.21.9
FROM ghcr.io/lanslide-team/minecraft-base:v${MC_VERSION}
#FROM minecraft-base:v${MC_VERSION}

WORKDIR /mc

COPY --chown=mc:mc bukkit.yml commands.yml ./
COPY --chown=mc:mc worlds/plotworld/ ./worlds/plotworld/
COPY --chown=mc:mc plugins/ ./plugins/

USER mc
ENTRYPOINT ["./entrypoint.sh"]
