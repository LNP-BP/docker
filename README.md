# Docker containers for LNP/BP daemons and tools

This containers are maintained by LNP/BP Standards Association

- **Bitcoin Core**: <https://hub.docker.com/r/lnpbp/bitcoind>
- **c-Lightning**: <https://hub.docker.com/r/lnpbp/lightningd>
- **Electrs**: <https://hub.docker.com/r/lnpbp/electrs>
- **RGB Node**: <https://hub.docker.com/r/lnpbp/rgb-node>


## Quickstart

1. Clone this repository and change to its dir: 
    ```shell script
   git clone https://github.com/LNP-BP/docker
   cd docker/docker-compose
    ```
2. Go to the `mainnet`, `signet` or `testnet` directory and perform 
   `docker-compose up` command

Now you will have bitcoind, lightningd, electrs and elementsd (for mainnet 
version only) running with JSON-RPC interface opened for the local machine.

To modify the default JSON-RPC user name, password, IP addresses and other 
options edit `command` and `env` sections in a used `docker-compose.yml` file


## Detailed instructions

### Using command-line tools

The most simple way of using tools is to create aliases:
```shell script
alias bitcoin-cli='docker exec bitcoind-mainnet bitcoin-cli --datadir=/var/lib/bitcoin -rpcpassword=bitcoin -rpcuser=bitcoin'
alias lightning-cli='docker exec lightningd-mainnet lightning-cli --lightning-dir=/var/lib/lightning --mainnet --lightning-dir /var/lib/lightning'
alias liquid-cli='docker exec elementsd-liquidv1 elements-cli --datadir=/var/lib/elements -chain=liquidv1 -rpcuser=bitcoin -rpcpassword=bitcoin'
alias signet-cli='docker exec bitcoind-signet bitcoin-cli --datadir=/var/lib/bitcoin --signet -rpcpassword=bitcoin -rpcuser=bitcoin'
alias sightning-cli='docker exec lightningd-signet lightning-cli --lightning-dir=/var/lib/lightning --signet --lightning-dir /var/lib/lightning'
```

### Customizing docker images

#### Default build commands

- **Bitcoin Core**: we disable wallet in the release build, but leaving it for
  nightly builds such it will be possible to play with signet & testnet
  transactions.
    - latest & version tagged:
      `docker build Dockerfile/bitcoind --build-arg VERSION=<version> --build-arg DISABLE_WALLET=`
    - nightly build:
      `docker build Dockerfile/bitcoind`
- **c-Lightning**: nightly version has developer features enabled and is built
  with bitcoin-cli coming from the nightly Bitcoin Core build
    - latest & version tagged:
      `docker build Dockerfile/lightningd --build-arg VERSION=<version>`
    - nightly build:
      `docker build Dockerfile/lightningd --build-arg BITCOIN_VERSION=nightly --build-arg DEVELOPER=true`
- **Elements**: since there is not a lot of liquid-enabled wallets, here we
  enable wallet in all builds
    - latest & version tagged:
      `docker build Dockerfile/elementsd --build-arg VERSION=<version>`
    - nightly build:
      `docker build Dockerfile/elementsd`

#### Bitcoin Core

If you are planning to create your own docker image builds, remember the 
following:

- you need to specify `VERSION` arg value to the docker if you'd like to 
  build a specific version; otherwise/by default docker will build the latest  
  master (=nightly build)
- you need to set `DISABLE_WALLET` arg to an empty string if you'd like 
  the build to include wallet functionality:
  `docker build . --build-arg DISABLE_WALLET=`
  If this argument is not provided bitcoin core is built with wallet backed
  by the new (5th) version of BerkleyDB storage, meaning that old Bitcoin Core
  wallets can't be imported.


### Changing blockchain directory location

You can use your existing bitcoin blockchain directory using the following steps:
1. Create docker volume pointing to it with
    ```shell script
    docker volume create --driver local \
                         --opt o=bind \
                         --opt type=none \
                         --opt device=/var/lib/bitcoin \
                         bitcoin
    docker volume create --driver local \
                         --opt o=bind \
                         --opt type=none \
                         --opt device=/var/lib/lightning \
                         lightning 
    docker volume create --driver local \
                         --opt o=bind \
                         --opt type=none \
                         --opt device=/var/lib/elements \
                         elements 
    docker volume create --driver local \
                         --opt o=bind \
                         --opt type=none \
                         --opt device=/var/lib/electrs \
                         electrs 
    ```
   where `/var/lib/bitcoin` etc must be replaced with your destination directories
2. Edit `docker-compose/.env` file paths
3. When starting docker-compose from within an appropriate `docker-compose.yml` 
   file directory provide it with `-env=../.env` option

### Creating container from the command line

Execute the following command:
```shell script
docker run \
  -p 8332:8332 -p 8333:8333 \
  -v /var/lib/bitcoin:bitcoin \
  --name bitcoind \
  lnpbp/bitcoind:latest
```
with replacing `/var/lib/bitcoin` with the desired location for the blockchain
data.
