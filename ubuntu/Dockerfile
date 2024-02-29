############################################################
# Dockerfile that contains SteamCMD
############################################################
ARG VERSION_OS

FROM ubuntu:${VERSION_OS} as build_stage

LABEL maintainer="admin@rwind.io"
LABEL hostname="steamcmd"
ARG PUID=1000

ENV USER steam
ENV HOMEDIR "/home/${USER}"
ENV STEAMCMDDIR "${HOMEDIR}/steamcmd"

ENV TRACY_NO_INVARIANT_CHECK=1

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN dpkg --add-architecture i386 \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
                ca-certificates \
                locales \
                lib32gcc-s1 \
                libsdl2-2.0-0:i386 \
                curl \
                vim \
                wget \
    && apt-get -y autoremove \
    && apt-get -y clean \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/* \
    && rm -rf /var/tmp/*
# Add unicode support
RUN locale-gen en_US.UTF-8
ENV LANG 'en_US.UTF-8'
ENV LANGUAGE 'en_US:en'

RUN set -x \
        && userdel -r ubuntu \
        && useradd -u "${PUID}" -m "${USER}" \
    	&& su "${USER}" -c \
		"mkdir -p \"${STEAMCMDDIR}\"  \
            && curl -fsSL 'https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz' | tar xvzf - -C \"${STEAMCMDDIR}\" \
            && \"./${STEAMCMDDIR}/steamcmd.sh\" +quit \
            && ln -s \"${STEAMCMDDIR}/linux32/steamclient.so\" \"${STEAMCMDDIR}/steamservice.so\" \
            && mkdir -p \"${HOMEDIR}/.steam/sdk32\" \
            && ln -s \"${STEAMCMDDIR}/linux32/steamclient.so\" \"${HOMEDIR}/.steam/sdk32/steamclient.so\" \
            && ln -s \"${STEAMCMDDIR}/linux32/steamcmd\" \"${STEAMCMDDIR}/linux32/steam\" \
            && mkdir -p \"${HOMEDIR}/.steam/sdk64\" \
            && ln -s \"${STEAMCMDDIR}/linux64/steamclient.so\" \"${HOMEDIR}/.steam/sdk64/steamclient.so\" \
            && ln -s \"${STEAMCMDDIR}/linux64/steamcmd\" \"${STEAMCMDDIR}/linux64/steam\" \
            && ln -s \"${STEAMCMDDIR}/steamcmd.sh\" \"${STEAMCMDDIR}/steam.sh\"" \ 
        # 	# Symlink steamclient.so; So misconfigured dedicated servers can find it
        && ln -s "${STEAMCMDDIR}/linux64/steamclient.so" "/usr/lib/x86_64-linux-gnu/steamclient.so"

FROM build_stage AS ubuntu-root
WORKDIR ${STEAMCMDDIR}

FROM ubuntu-root AS ubuntu-steam
# Switch to user
USER ${USER}