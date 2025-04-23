FROM debian:12-slim

ENV DEBIAN_FRONTEND=noninteractive
RUN apt update -yq && apt install -yq ca-certificates

ADD https://www.flightaware.com/adsb/piaware/files/packages/pool/piaware/f/flightaware-apt-repository/flightaware-apt-repository_1.2_all.deb /

RUN dpkg -i /flightaware-apt-repository_1.2_all.deb
RUN apt update -yq && apt install -yq dump1090-fa

CMD ["/usr/share/dump1090-fa/start-dump1090-fa"]
