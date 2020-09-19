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
        wget \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV ELEMENTS_VERSION=0.18.1.7
ENV ELEMENTS_SRC=/usr/local/src/elements
ENV ELEMENTS_BIN=/elements

RUN git clone -n https://github.com/ElementsProject/elements $ELEMENTS_SRC \
    && cd $ELEMENTS_SRC \
    && git checkout tags/elements-$ELEMENTS_VERSION

WORKDIR $ELEMENTS_SRC

RUN ./contrib/install_db4.sh `pwd`

RUN ./autogen.sh

ENV BDB_PREFIX="$ELEMENTS_SRC/db4"

RUN mkdir $ELEMENTS_BIN

RUN  ./configure BDB_LIBS="-L${BDB_PREFIX}/lib -ldb_cxx-4.8" BDB_CFLAGS="-I${BDB_PREFIX}/include" --disable-tests --disable-bench --without-gui --prefix=$ELEMENTS_BIN

RUN V=1 make clean

RUN V=1 make -j4

RUN make install




FROM debian:buster-slim

LABEL maintainer.0="Dr Maxim Orlovsky <orlovsky@pandoracore.com>" \
    maintainer.1="Alekos Filini"

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

COPY --from=builder /elements/bin /usr/local/bin

VOLUME "/var/lib/elementsd"

ENV ELEMENTS_DATA=/var/lib/elementsd

RUN groupadd -r elements && useradd -r -m -g elements elements \
    && mkdir -p "$ELEMENTS_DATA" \
    && chmod 700 "$ELEMENTS_DATA" \
    && chown -R elements:elements "$ELEMENTS_DATA"

EXPOSE 7041 7042

ENTRYPOINT elementsd -txindex -datadir=$ELEMENTS_DATA
