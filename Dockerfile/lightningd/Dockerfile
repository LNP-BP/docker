FROM debian:buster-slim as builder

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

ENV LIGHTNING_VERSION=0.9.1
ENV LIGHTNING_SRC=/usr/local/src/lightning
ENV LIGHTNING_BIN=/lightning

WORKDIR $LIGHTNING_SRC

RUN mkdir $LIGHTNING_BIN

RUN git clone --recursive https://github.com/ElementsProject/lightning.git $LIGHTNING_SRC \
    && cd $LIGHTNING_SRC \
    && git checkout tags/v$LIGHTNING_VERSION

ARG DEVELOPER=0
RUN ./configure --prefix=$LIGHTNING_BIN --enable-static

RUN make -j3 DEVELOPER=${DEVELOPER}

RUN make install




FROM debian:buster-slim as downloader

RUN apt-get update -y \
    && apt-get install -y --no-install-recommends \
        ca-certificates \
        dirmngr \
        wget \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ARG BITCOIN_VERSION=0.20.1
ENV BITCOIN_TARBALL bitcoin-${BITCOIN_VERSION}-x86_64-linux-gnu.tar.gz
ENV BITCOIN_URL https://bitcoincore.org/bin/bitcoin-core-$BITCOIN_VERSION/$BITCOIN_TARBALL
ENV BITCOIN_ASC_URL https://bitcoincore.org/bin/bitcoin-core-$BITCOIN_VERSION/SHA256SUMS.asc

RUN mkdir /opt/bitcoin && cd /opt/bitcoin \
    && wget -qO $BITCOIN_TARBALL "$BITCOIN_URL" \
    && wget -qO bitcoin.asc "$BITCOIN_ASC_URL" \
    && grep $BITCOIN_TARBALL bitcoin.asc | tee SHA256SUMS.asc \
    && sha256sum -c SHA256SUMS.asc \
    && BD=bitcoin-$BITCOIN_VERSION/bin \
    && tar -xzvf $BITCOIN_TARBALL $BD/bitcoin-cli --strip-components=1 \
    && rm $BITCOIN_TARBALL




FROM debian:buster-slim

LABEL maintainer.0="Dr Maxim Orlovsky <orlovsky@pandoracore.com>"

RUN apt-get update -y \
    && apt-get install -y --no-install-recommends \
        socat \
        inotify-tools \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY --from=builder /lightning/ /usr/local/
COPY --from=downloader /opt/bitcoin/bin/bitcoin-cli /usr/bin
COPY bin /usr/local/bin

VOLUME "/var/lib/lightningd"

ENV LIGHTNING_VERSION=0.9.1
ENV LIGHTNING_DATA=/var/lib/lightningd

RUN groupadd -r lightning && useradd -r -m -g lightning lightning \
    && mkdir -p "$LIGHTNING_DATA" \
    && chmod 700 "$LIGHTNING_DATA" \
    && chown -R lightning:lightning "$LIGHTNING_DATA"

EXPOSE 9735 9835

ENTRYPOINT ["/bin/bash", "-c", "lightningd $LIGHTNING_NETWORK_FLAG --log-level=$LIGHTNING_LOG_LEVEL --lightning-dir=$LIGHTNING_DATA --bitcoin-rpcuser=$BITCOIN_RPC_USER --bitcoin-rpcpassword=$BITCOIN_RPC_PASSWORD --bitcoin-rpcconnect=$BITCOIN_RPC_CONNECT --bitcoin-rpcport=$BITCOIN_RPC_PORT"]
