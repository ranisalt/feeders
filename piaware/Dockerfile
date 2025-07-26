FROM debian:12-slim AS build

ENV DEBIAN_FRONTEND=noninteractive
RUN apt update -yq && apt install -yq build-essential git devscripts debhelper tcl8.6-dev autoconf \
  python3-dev python3-wheel python3-build python3-pip python3-venv python3-setuptools libz-dev openssl \
  libboost-system-dev libboost-program-options-dev libboost-regex-dev \
  libboost-filesystem-dev patchelf wget

ADD https://github.com/flightaware/piaware_builder.git#v10.2 /src

WORKDIR /src
RUN ./sensible-build.sh bookworm && cd package-bookworm && dpkg-buildpackage -b --no-sign

FROM debian:12-slim

RUN --mount=type=bind,from=build,source=/src,target=/src apt update -yq && apt install -yq /src/piaware_10.2_*.deb

USER piaware

CMD ["/usr/bin/piaware", "-debug", "-plainlog"]
