ARG MC_VERSION=26.2
FROM ghcr.io/lanslide-team/minecraft-base:v${MC_VERSION}
#FROM minecraft-base:v${MC_VERSION}

RUN apt-get update && apt-get install -y wget

WORKDIR /mc

COPY plugins/ ./plugins/

RUN chown mc:mc /mc -R

ENTRYPOINT ["./entrypoint.sh"]
