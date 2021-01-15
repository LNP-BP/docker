ARG BUILDER_DIR=/srv/rgb


FROM rust:1.47.0-slim-buster as builder

ARG SRC_DIR=/usr/local/src/lnp
ARG BUILDER_DIR
ARG VERSION

RUN apt-get update \
    && apt-get -y install --no-install-recommends \
        build-essential cmake git pkg-config \
        libpq-dev libssl-dev libzmq3-dev libsqlite3-dev

WORKDIR "$SRC_DIR"

RUN git clone -n https://github.com/LNP-BP/lnp-node.git . \
    && git checkout ${VERSION:+tags/v}${VERSION:-master}


RUN mkdir "${BUILDER_DIR}"

RUN cargo install --path . --locked --root "${BUILDER_DIR}" --bins --all-features


FROM debian:buster-slim

ARG BUILDER_DIR
ARG BIN_DIR=/usr/local/bin
ARG DATA_DIR=/var/lib/lnp
ARG USER=lbpd

RUN apt-get update \
    && apt-get -y install --no-install-recommends \
       libsqlite3-0 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN adduser --home "${DATA_DIR}" --shell /bin/bash --disabled-login \
        --gecos "${USER} user" ${USER}

COPY --from=builder --chown=${USER}:${USER} \
     "${BUILDER_DIR}/bin/" "${BIN_DIR}"

WORKDIR "${BIN_DIR}"
USER ${USER}

VOLUME "$DATA_DIR"

EXPOSE 9735

ENTRYPOINT ["lnpd", "--data-dir=/var/lib/lnp"]

CMD ["-vvvv"]

