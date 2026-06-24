ARG MC_VERSION=26.2
FROM ghcr.io/lanslide-team/minecraft-base:v${MC_VERSION}
#FROM minecraft-base:v${MC_VERSION}

RUN apt-get update && apt-get install -y yq

WORKDIR /mc

COPY --chown=mc:mc bukkit.yml commands.yml ./
COPY --chown=mc:mc worlds/plotworld/ ./worlds/plotworld/
COPY --chown=mc:mc plugins/ ./plugins/

RUN chown mc:mc /mc -R

ENTRYPOINT ["./entrypoint.sh"]
