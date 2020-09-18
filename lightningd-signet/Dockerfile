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
    && apt-get install -y \
        automake \
        autotools-dev \
        bsdmainutils \
        build-essential \
        git \
        libdb-dev \
        libdb++-dev \
        libboost-chrono-dev \
        libboost-filesystem-dev \
        libboost-system-dev \
        libboost-test-dev \
        libboost-thread-dev \
        libevent-dev \
        libminiupnpc-dev \
        libssl-dev \
        libtool \
        libzmq3-dev \
        pkg-config \
        python3 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV BITCOIN_SRC=/usr/local/src/bitcoin
ENV BITCOIN_BIN=/bitcoin

RUN git clone https://github.com/kallewoof/bitcoin.git --branch 2003-signet-consensus $BITCOIN_SRC

WORKDIR $BITCOIN_SRC

RUN ./autogen.sh

RUN mkdir $BITCOIN_BIN

RUN  ./configure --with-incompatible-bdb --disable-tests --disable-bench --without-gui --prefix=$BITCOIN_BIN

RUN V=1 make clean

RUN V=1 make -j4

RUN make install



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
COPY --from=downloader /bitcoin/bin/bitcoin-cli /usr/local/bin/
COPY ./bin /usr/local/bin

VOLUME "/var/lib/lightningd"

ENV LIGHTNING_VERSION=0.9.1
ENV LIGHTNING_DATA=/var/lib/lightningd

RUN groupadd -r lightning && useradd -r -m -g lightning lightning \
    && mkdir -p "$LIGHTNING_DATA" \
    && chmod 700 "$LIGHTNING_DATA" \
    && chown -R lightning:lightning "$LIGHTNING_DATA"

EXPOSE 39735 39835

ENTRYPOINT ["/bin/bash", "-c", "lightningd $LIGHTNING_NETWORK_FLAG --log-level=$LIGHTNING_LOG_LEVEL --lightning-dir=$LIGHTNING_DATA --bitcoin-rpcuser=$BITCOIN_RPC_USER --bitcoin-rpcpassword=$BITCOIN_RPC_PASSWORD --bitcoin-rpcconnect=$BITCOIN_RPC_CONNECT --bitcoin-rpcport=$BITCOIN_RPC_PORT"]
