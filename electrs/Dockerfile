FROM rust:slim as builder

RUN apt-get update -y \
    && apt-get install -y \
        clang \
        cmake \
        git \
        libsnappy-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV ELECTRS_VERSION=0.8.3
ENV ELECTRS_SRC=/usr/local/src/electrs
ENV ELECTRS_BIN=/usr/local/bin

RUN git clone -n https://github.com/romanz/electrs.git $ELECTRS_SRC \
    && cd $ELECTRS_SRC \
    && git checkout tags/v$ELECTRS_VERSION

WORKDIR $ELECTRS_SRC

RUN cargo install --path .



FROM debian:buster-slim

LABEL maintainer.0="Dr Maxim Orlovsky <orlovsky@pandoracore.com>"

RUN apt-get update -y \
    && apt-get install -y \
        clang \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY --from=builder /usr/local/cargo/bin/electrs /usr/local/bin

ENV BITCOIN_DATA=/var/lib/bitcoind
ENV ELECTRS_DATA=/var/lib/electrs
ENV ELECTRS_CONF=/etc/electrs/electrs.toml

RUN mkdir /etc/electrs
RUN echo 'cookie = "bitcoin:bitcoin"' > $ELECTRS_CONF

RUN groupadd -r electrs && useradd -r -m -g electrs electrs \
    && mkdir -p "$ELECTRS_DATA" \
    && chmod 700 "$ELECTRS_DATA" \
    && chown -R electrs:electrs "$ELECTRS_DATA"

VOLUME "/var/lib/bitcoind"
VOLUME "/var/lib/electrs"

# Electrum RPC
EXPOSE 50001

# Prometheus monitoring
EXPOSE 4224

STOPSIGNAL SIGINT

ENTRYPOINT ["/bin/bash", "-c", "/usr/local/bin/electrs -vvvv --daemon-dir $BITCOIN_DATA --db-dir $ELECTRS_DATA --conf $ELECTRS_CONF "]
