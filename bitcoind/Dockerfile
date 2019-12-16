FROM debian:buster-slim as builder

RUN apt-get update -y \
    && apt-get install -y \
        automake \
        autotools-dev \
        bsdmainutils \
        build-essential \
        git \
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

ENV BITCOIN_VERSION=0.19.0.1
ENV BITCOIN_SRC=/usr/local/src/bitcoin
ENV BITCOIN_BIN=/bitcoin

RUN git clone -n https://github.com/bitcoin/bitcoin.git $BITCOIN_SRC \
    && cd $BITCOIN_SRC \
    && git checkout tags/v$BITCOIN_VERSION

WORKDIR $BITCOIN_SRC

RUN ./autogen.sh

RUN mkdir $BITCOIN_BIN

RUN  ./configure --disable-tests --disable-bench --disable-wallet --without-gui --prefix=$BITCOIN_BIN

RUN V=1 make clean

RUN V=1 make -j4

RUN make install




FROM debian:buster-slim

LABEL maintainer.0="Dr Maxim Orlovsky <orlovsky@pandoracore.com>"

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
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY --from=builder /bitcoin/bin /usr/local/bin

VOLUME "/var/lib/bitcoind"

ENV BITCOIN_VERSION=0.19.0.1
ENV BITCOIN_DATA=/var/lib/bitcoind

RUN groupadd -r bitcoin && useradd -r -m -g bitcoin bitcoin \
    && mkdir -p "$BITCOIN_DATA" \
    && chmod 700 "$BITCOIN_DATA" \
    && chown -R bitcoin:bitcoin "$BITCOIN_DATA"

EXPOSE 8332 8333 18332 18333 18443 18444 38332 38333

ENTRYPOINT bitcoind -txindex -datadir=$BITCOIN_DATA
