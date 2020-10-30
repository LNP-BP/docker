ARG VERSION
ARG RUSTC=1.44.1


FROM rust:${RUSTC}-slim-buster as builder

ARG SRC_DIR=/usr/local/src/electrs
ARG VERSION

RUN apt-get update -y \
    && apt-get install -y \
        clang \
        cmake \
        git \
        libsnappy-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

WORKDIR $SRC_DIR

RUN git clone -n https://github.com/romanz/electrs.git $SRC_DIR \
    && cd $SRC_DIR \
    && git checkout ${VERSION:+tags/v}${VERSION:-master}

RUN cargo install --path .



FROM debian:buster-slim

LABEL maintainer.0="LNP/BP Standards Association <https://lnp-bp.org>" \
      maintainer.1="Dr Maxim Orlovsky <orlovsky@lnp-bp.org>" \
      maintainer.2="Zoe Faltiba"

RUN apt-get update -y \
    && apt-get install -y \
        clang \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ARG BIN_DIR=/usr/local/bin
ARG BITCOIN_DIR=/var/lib/bitcoin
ARG DATA_DIR=/var/lib/electrs

COPY --from=builder /usr/local/cargo/bin/electrs $BIN_DIR

RUN groupadd -r electrs && useradd -r -m -g electrs electrs \
    && mkdir -p "$DATA_DIR" \
    && chmod 700 "$DATA_DIR" \
    && chown -R electrs:electrs "$DATA_DIR"

# Electrum RPC: mainnet, testnet and signet
EXPOSE 50001 60001 60601

# Prometheus monitoring
EXPOSE 4224

VOLUME $BITCOIN_DIR
VOLUME $DATA_DIR

USER electrs:electrs

STOPSIGNAL SIGINT

ENTRYPOINT ["electrs", "--daemon-dir", "/var/lib/bitcoin", "--db-dir", "/var/lib/electrs"]

CMD ["-vvvv"]
