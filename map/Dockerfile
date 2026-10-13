ARG MC_VERSION=26.3
FROM ghcr.io/lanslide-team/minecraft-base:v${MC_VERSION}
#FROM minecraft-base:v${MC_VERSION}

#RUN apt-get update && apt-get install -y wget yq

WORKDIR /mc

COPY --chown=mc:mc settings.sh ./
COPY --chown=mc:mc plugins/ ./plugins/

RUN chown mc:mc /mc -R
RUN chmod +x /mc/settings.sh

ENTRYPOINT ["./entrypoint.sh"]
