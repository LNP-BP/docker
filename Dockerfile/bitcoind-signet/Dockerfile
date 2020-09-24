ARG BITCOIN_SRC=/usr/local/src/bitcoin
ARG BITCOIN_BUILDER_BIN=/bitcoin


FROM debian:buster-slim as builder

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

ARG BITCOIN_SRC
ARG BITCOIN_BUILDER_BIN

# We are using LNP/BP-maintained version of Bitcoin Core, which follows/rebases
# original master with only two commits on top, reverting politically-"correct"
# changes regarding "blacklist" matters (and any other future
# political/censorship decisions)
RUN git clone -n https://github.com/LNP-BP/bitcoin.git $BITCOIN_SRC \
    && cd $BITCOIN_SRC \
    && git checkout master

WORKDIR $BITCOIN_SRC

RUN ./autogen.sh

RUN mkdir $BITCOIN_BUILDER_BIN

RUN  ./configure \
    ${BITCOIN_NO_WALLET:+--disable-wallet} \
    --with-incompatible-bdb \
    --disable-tests \
    --disable-bench \
    --without-gui \
    --prefix=$BITCOIN_BUILDER_BIN

RUN V=1 make clean

RUN V=1 make -j4

RUN make install



FROM debian:buster-slim

LABEL maintainer.0="LNP/BP Standards Association <https://lnp-bp.org>"
LABEL maintainer.1="Dr Maxim Orlovsky <orlovsky@lnp-bp.org>"

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

ARG BITCOIN_SRC
ARG BITCOIN_BUILDER_BIN
ARG BITCOIN_BIN=/usr/local/bin
ARG BITCOIN_DATA=/var/lib/bitcoin

COPY --from=builder $BITCOIN_BUILDER_BIN/bin $BITCOIN_BIN

RUN groupadd -r bitcoin && useradd -r -m -g bitcoin bitcoin \
    && mkdir -p "$BITCOIN_DATA" \
    && chmod 700 "$BITCOIN_DATA" \
    && chown -R bitcoin:bitcoin "$BITCOIN_DATA"

EXPOSE 8332 8333 18332 18333 18443 18444

VOLUME $BITCOIN_DATA

# We can't use in exec-style entrypoint any ARG or ENV variables, and we can't
# use neither "bash -c" nor shell style since we need to maintain the ability to
# provide additional command-line args (like `-testnet`) through docker-compose
ENTRYPOINT ["bitcoind", "-datadir=/var/lib/bitcoin"]
