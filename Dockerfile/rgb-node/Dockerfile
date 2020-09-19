FROM rustlang/rust:nightly-slim as builder

RUN apt-get update \
    && apt-get -y install --no-install-recommends \
        build-essential cmake git libpq-dev libssl-dev libzmq3-dev pkg-config

WORKDIR /srv/app

ENV RGBNODE_VERSION=v0.1.0-beta.4

RUN git clone -n https://github.com/LNP-BP/rgb-node.git . \
    && git checkout ${RGBNODE_VERSION} \
    && cargo build --release


FROM debian:buster-slim

ENV APP_DIR=/srv/app USER=rgbnode

RUN adduser --home ${APP_DIR} --shell /bin/bash --disabled-login \
        --gecos "${USER} user" ${USER}

COPY --from=builder --chown=${USER}:${USER} \
        ${APP_DIR} ${APP_DIR}

RUN apt-get update \
    && apt-get -y install --no-install-recommends \
        libssl1.1 tini \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

WORKDIR ${APP_DIR}
USER ${USER}

RUN mkdir data

VOLUME ["${APP_DIR}/data"]
ENTRYPOINT ["/usr/bin/tini", "-g", "--", "/usr/local/bin/rgbd", \
			"--bin-dir", "/usr/local/bin", "--data-dir", "./data"]
CMD ["-vvvv"]
