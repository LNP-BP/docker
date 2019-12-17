# Docker containers for LNP/BP daemons and tools

This containers are maintained by LNP/BP Standards Association

- **Bitcoin Core**: https://hub.docker.com/repository/docker/lnpbp/bitcoind
  Minimalistic build without wallet (but with ZMQ) on Debian
- **Bitcoin Core** for SigNet: https://hub.docker.com/repository/docker/lnpbp/bitcoind-signet
  Minimalistic build without wallet (but with ZMQ) on Debian

## Quickstart

1. Clone this repository and change to its dir: 
    ```shell script
   git clone https://github.com/lnp-bp/docker
   cd docker
    ```
2. Initialize volume and default config file with this commands:
    ```shell script
    docker volume create blockchain
    docker run --rm -v $PWD:/source -v blockchain:/dest -w /source scratch cp ./bitcoin.conf /dest
    ```
2. Perform `docker-compose up` command

Now you will have bitcoind running for mainnet and signet with JSON-RPC interface opened for the local machine.
You will be able to access it with the user name `bitcoin` and password `bitcoin`.

To modify the default JSON-RPC user name, password, IP addresses and other bitcoind options edit the `bitcoin.conf`
file in the root directory of the repository and each time perform this command:
```shell script
docker-compose down
docker run --rm -v $PWD:/source -v blockchain:/dest -w /source scratch cp ./bitcoin.conf /dest
docker-compose up
```

### Using command-line interface to bitcoind

To connect and execute command for the mainnet use the pattern
```shell script
bitcoin-cli -rpcuser=bitcoin -rpcpassword=bitcoin -rpcport=8332 getblockchaininfo 
```
where `getblockchaininfo` may be replaced with any other JSON-RPC command with its arguments.

For the signet, use
```shell script
bitcoin-cli -rpcuser=bitcoin -rpcpassword=bitcoin -rpcport=38332 getblockchaininfo 
```


## Changing blockchain directory location

If you would like to allocate the blockchain data storage to some specific place, you will need to either edit
`docker-compose.yml` file or create docker container from the command line.

### Editing compose file

Replace all occurrences of `blockchain` string within the volumes block with the desired destination directory. 
To locate volumes look for the following pattern:
```yaml
    volumes:
      - "blockchain:/var/lib/bitcoind"
```
so it will become, for example
```yaml
    volumes:
      - "/Volumes/ExternalDrive/bitcoin:/var/lib/bitcoind"
```

In this case you will need to copy `bitcoin.conf` file to that destination; otherwise bitcoind will run with the
default parameters

### Creating container from the command line

Execute the following command:
```shell script
docker run \
  -p 8332:8332 -p 8333:8333 \
  -v /Volumes/ExternalDrive/bitcoin:/var/lib/bitcoind \
  --name bitcoind \
  lnpbp/bitcoind:latest
```
 with replacing "/Volumes/ExternalDrive/bitcoin" with the desired location for the blockchain data.
 
 You do not need to clone this git repository to execute this command, however you will need to copy `bitcoin.conf` file 
 to that destination beforehand; otherwise bitcoind will run with the default parameters
