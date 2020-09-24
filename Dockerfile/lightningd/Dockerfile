ARG BUILDER_DIR=/lightning
ARG VERSION
ARG DEVELOPER
ARG BITCOIN_VERSION=latest


FROM debian:buster-slim as builder

ARG SRC_DIR=/usr/local/src/lightning
ARG BUILDER_DIR
ARG VERSION
ARG DEVELOPER

RUN apt-get update -y \
    && apt-get install -y --no-install-recommends \
        ca-certificates \
        autoconf \
        automake \
        build-essential \
        dirmngr \
        gettext \
        git \
        gnupg \
        libgmp-dev \
        libsqlite3-dev \
        libtool \
        python \
        python3 \
        python3-mako \
        tclsh \
        unzip \
        wget \
        zlib1g-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

WORKDIR $SRC_DIR

RUN git clone --recursive https://github.com/ElementsProject/lightning.git $SRC_DIR \
    && cd $SRC_DIR \
    && git checkout ${VERSION:+tags/v}${VERSION:-master}

RUN mkdir $BUILDER_DIR

RUN ./configure \
    --prefix=$BUILDER_DIR \
    ${DEVELOPER+--enable-developer} \
    --enable-static

RUN V=1 make clean

RUN V=1 make -j3

RUN make install


FROM lnpbp/bitcoind:${BITCOIN_VERSION} as downloader
FROM debian:buster-slim

LABEL maintainer.0="LNP/BP Standards Association <https://lnp-bp.org>"
LABEL maintainer.1="Dr Maxim Orlovsky <orlovsky@lnp-bp.org>"
LABEL maintainer.2="Zoe Faltiba"

ARG BUILDER_DIR
ARG DEST_DIR=/usr/local
ARG DATA_DIR=/var/lib/lightning

RUN apt-get update -y \
    && apt-get install -y --no-install-recommends \
        libboost-chrono1.67.0 \
        libboost-filesystem1.67.0 \
        libboost-system1.67.0 \
        libboost-thread1.67.0 \
        libevent-2.1-6 \
        libevent-pthreads-2.1-6 \
        socat \
        inotify-tools \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY --from=builder $BUILDER_DIR/ $DEST_DIR/
COPY --from=downloader $DEST_DIR/bin/bitcoin-cli $DEST_DIR/bin/

RUN groupadd -r lightning && useradd -r -m -g lightning lightning \
    && mkdir -p "$DATA_DIR" \
    && chmod 700 "$DATA_DIR" \
    && chown -R lightning:lightning "$DATA_DIR"

EXPOSE 9735 9835

VOLUME $DATA_DIR

USER lightning:lightning

ENTRYPOINT ["lightningd", "--lightning-dir=/var/lib/lightning"]
