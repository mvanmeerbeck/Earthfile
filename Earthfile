VERSION 0.7

# https://www.debian.org/releases/
# version 12.1, was released on July 22nd, 2023
ARG --global DEBIAN_VERSION=12

alpine:
    FROM alpine:3.18.4
    DO +ALPINE_TZ_FR

ALPINE_TZ_FR:
    COMMAND
    # setup time-zone https://wiki.alpinelinux.org/wiki/Setting_the_timezone
    RUN apk add --update tzdata && cp /usr/share/zoneinfo/Europe/Paris /etc/localtime && echo "Europe/Paris" > /etc/timezone && apk del tzdata

alpine-python:
    FROM +alpine
    RUN apk add --update python3 py3-pip

debian:
   FROM debian:${DEBIAN_VERSION}-slim
   DO +DEBIAN_NO_AUTO_INSTALL
   DO +DEBIAN_TZ_FR

DEBIAN_TZ_FR:
   COMMAND
   RUN rm /etc/localtime
   RUN ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime

debian-europe-paris:
    FROM debian:${DEBIAN_VERSION}-slim
    SAVE ARTIFACT /usr/share/zoneinfo/Europe/Paris localtime

distroless-python3-debian:
    FROM gcr.io/distroless/python3-debian${DEBIAN_VERSION}
    COPY +debian-europe-paris/localtime /etc/localtime

pip-tools:
  FROM python:3.8-alpine
  DO +USE_USER
  RUN pip install pip-tools

SETUP_USER:
	COMMAND
    ARG uid=1000
    ARG username=sam
    ARG shell=/bin/sh
    ENV HOME=/home/$username
    RUN addgroup --gid $uid --system $username \
    	&& adduser --uid $uid --shell ${shell} --disabled-password $username --ingroup $username \
		&& chown -R $username:$username $HOME
    # audio video disk lp dialout users
	ARG groups
	IF  [ -n "$groups" ]
		RUN for group in $groups ; do { grep -q -E "^${group}:" /etc/group || addgroup --system $group ; } && adduser $username $group ; done
	END
    ENV PATH=$HOME/.local/bin:$PATH

AS_USER:
	COMMAND
    ARG uid=1000
    ARG username=sam
    ENV HOME=/home/$username
    USER $username
    WORKDIR $HOME

USE_USER:
	COMMAND
	ARG groups
    ARG uid=1000
    ARG username=sam
    DO +SETUP_USER --groups="$groups" --uid="${uid}" --username="${username}"
    DO +AS_USER --uid="${uid}" --username="${username}"

DEBIAN_NO_AUTO_INSTALL:
  COMMAND
  RUN echo 'APT::Install-Recommends "0";' > /etc/apt/apt.conf.d/01norecommend
  RUN echo 'APT::Install-Suggests "0";' >> /etc/apt/apt.conf.d/01norecommend

USER_WRITE_ENV:
    COMMAND
    ARG --required name
    RUN bash -c 'echo export ${name}="${!name}"' >> ${HOME}/.profile
