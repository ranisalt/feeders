FROM debian:12-slim AS build

ENV DEBIAN_FRONTEND=noninteractive
RUN apt update -yq && apt install -yq build-essential fakeroot debhelper librtlsdr-dev pkg-config libncurses5-dev libbladerf-dev libhackrf-dev liblimesuite-dev libsoapysdr-dev devscripts

ADD https://github.com/flightaware/dump1090.git#v10.2 /src

WORKDIR /src
RUN ./prepare-build.sh bookworm && cd package-bookworm && dpkg-buildpackage -b --no-sign

FROM debian:12-slim

RUN --mount=type=bind,from=build,source=/src,target=/src apt update -yq && apt install -yq /src/dump1090-fa_10.2_*.deb

USER dump1090

CMD ["/usr/share/dump1090-fa/start-dump1090-fa"]
