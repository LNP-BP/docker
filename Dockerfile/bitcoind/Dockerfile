ARG BUILDER_DIR=/bitcoin
ARG DISABLE_WALLET


FROM debian:buster-slim as builder

ARG SRC_DIR=/usr/local/src/bitcoin
ARG BUILDER_DIR
ARG VERSION
ARG DISABLE_WALLET

RUN echo ${DISABLE_WALLET+no wallet}
RUN echo ${NO_WALLET-with wallet}

RUN apt-get update -y \
    && apt-get install -y \
        automake \
        autotools-dev \
        bsdmainutils \
        build-essential \
        git \
        ${DISABLE_WALLET-libdb-dev} \
        ${DISABLE_WALLET-libdb++-dev} \
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

# We are using LNP/BP-maintained version of Bitcoin Core, which follows/rebases
# original master with only two commits on top, reverting politically-"correct"
# changes regarding "blacklist" matters (and any other future politically or
# censorship-motivated decisions)
RUN git clone -n https://github.com/LNP-BP/bitcoin.git $SRC_DIR \
    && cd $SRC_DIR \
    && git checkout ${VERSION:+tags/v}${VERSION:-master}

WORKDIR $SRC_DIR

RUN ./autogen.sh

RUN mkdir $BUILDER_DIR

RUN  ./configure \
    ${DISABLE_WALLET---with-incompatible-bdb} \
    ${DISABLE_WALLET+--disable-wallet} \
    --disable-tests \
    --disable-bench \
    --without-gui \
    --prefix=$BUILDER_DIR

RUN V=1 make clean

RUN V=1 make -j2

RUN make install



FROM debian:buster-slim

LABEL maintainer.0="LNP/BP Standards Association <https://lnp-bp.org>"
LABEL maintainer.1="Dr Maxim Orlovsky <orlovsky@lnp-bp.org>"

ARG BUILDER_DIR
ARG BIN_DIR=/usr/local/bin
ARG DATA_DIR=/var/lib/bitcoin
ARG DISABLE_WALLET

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
        ${DISABLE_WALLET-libdb5.3++} \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY --from=builder $BUILDER_DIR/bin $BIN_DIR

RUN groupadd -r bitcoin && useradd -r -m -g bitcoin bitcoin \
    && mkdir -p "$DATA_DIR" \
    && chmod 700 "$DATA_DIR" \
    && chown -R bitcoin:bitcoin "$DATA_DIR"

EXPOSE 8332 8333 18332 18333 18443 18444

VOLUME $DATA_DIR

USER bitcoin:bitcoin

# We can't use in exec-style entrypoint any ARG or ENV variables, and we can't
# use neither "bash -c" nor shell style since we need to maintain the ability to
# provide additional command-line args (like `-testnet`) through docker-compose
ENTRYPOINT ["bitcoind", "-datadir=/var/lib/bitcoin"]
