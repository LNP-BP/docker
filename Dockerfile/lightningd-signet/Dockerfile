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

# Remove this patch once https://github.com/ElementsProject/lightning/pull/4068/files accepted
RUN wget https://gist.githubusercontent.com/dr-orlovsky/0bed924ee8ca7aafd2bc94d436490909/raw/548dd6e7570c16e6340585e384ffc17c45d60bdc/clightning-signet-chainparams.diff \
    && git apply clightning-signet-chainparams.diff


ARG DEVELOPER=0
RUN ./configure --prefix=$LIGHTNING_BIN --enable-static

RUN make -j3 DEVELOPER=${DEVELOPER}

RUN make install




FROM lnpbp/bitcoind:signet as downloader

FROM debian:buster-slim

LABEL maintainer.0="Dr Maxim Orlovsky <orlovsky@pandoracore.com>"

RUN apt-get update -y \
    && apt-get install -y --no-install-recommends \
        socat \
        inotify-tools \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN apt-get update -y \
    && apt-get install -y \
        curl \
        libboost-chrono1.67.0 \
        libboost-filesystem1.67.0 \
        libboost-system1.67.0 \
        libboost-thread1.67.0 \
        libevent-2.1-6 \
        libevent-pthreads-2.1-6 \
        libminiupnpc17 \
        libssl1.1 \
        libzmq5 \
        libdb5.3++ \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY --from=builder /lightning/ /usr/local/
COPY --from=downloader /usr/local/bin/bitcoin-cli /usr/local/bin/

VOLUME "/var/lib/lightningd"

ENV LIGHTNING_DATA=/var/lib/lightningd

RUN groupadd -r lightning && useradd -r -m -g lightning lightning \
    && mkdir -p "$LIGHTNING_DATA" \
    && chmod 700 "$LIGHTNING_DATA" \
    && chown -R lightning:lightning "$LIGHTNING_DATA"

EXPOSE 39735 39835

ENTRYPOINT ["/bin/bash", "-c", "lightningd $LIGHTNING_NETWORK_FLAG --log-level=$LIGHTNING_LOG_LEVEL --lightning-dir=$LIGHTNING_DATA --bitcoin-rpcuser=$BITCOIN_RPC_USER --bitcoin-rpcpassword=$BITCOIN_RPC_PASSWORD --bitcoin-rpcconnect=$BITCOIN_RPC_CONNECT --bitcoin-rpcport=$BITCOIN_RPC_PORT"]
