FROM debian:12-slim

ENV DEBIAN_FRONTEND=noninteractive

RUN { if [ "$(uname -m)" = "x86_64" ]; then SYSTEM=linux; elif [ "$(uname -m)" = "aarch64" ]; then SYSTEM=raspberrypi; else exit 1; fi; } || exit 1 && \
    apt update -yq && apt install -yq ca-certificates curl gpg && \
    curl -SL https://repo-feed.flightradar24.com/flightradar24.pub | gpg --dearmor - > /etc/apt/keyrings/flightradar24.gpg && \
    echo "deb [signed-by=/etc/apt/keyrings/flightradar24.gpg] https://repo-feed.flightradar24.com flightradar24 ${SYSTEM}-stable" > /etc/apt/sources.list.d/fr24feed.list && \
    apt purge -yq curl gpg && apt autoremove --purge -yq

WORKDIR /tmp

RUN apt update -yq && apt download -yq fr24feed && \
    dpkg-deb --raw-extract fr24feed*.deb pkg && \
    rm fr24feed*.deb pkg/DEBIAN/preinst pkg/DEBIAN/postinst pkg/etc/cron.d/fr24feed_updater && \
    sed -i -e '/fr24feed_updater/d' -e '/public_html/d' pkg/DEBIAN/conffiles && \
    dpkg-deb --build pkg && dpkg --install pkg.deb && apt install -yq --fix-broken && \
    rm pkg.deb

RUN useradd -M -r -u 999 fr24
USER fr24

CMD ["/usr/bin/fr24feed"]
