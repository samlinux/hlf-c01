# Installing the dev-network
The developer network is optimized for this course and not suitable for productive use.

The following steps are required to get started with the dev-network.

We need different terminals to run the network. You can use tmux or three different ssh terminals.

## In terminal 1
In terminal 1 we start and run the dev-network. As well some network debug messages will be printed.

```bash
# switch into the base install folder
cd fabric

# clone the git repro
git clone git@sun.samlinux.at:/opt/git/sdg-dev-network.git

# start the network
./devNetwork.sh up

```

## In terminal 2
In terminal 2 we start and stop the chaincode itself.

```bash
# switch to the chaincode folder
cd chaincode/nodejs/sacc

# install all dependencies only for the first time
npm install 

# start the chaincode
CORE_CHAINCODE_LOGLEVEL=debug CORE_PEER_TLS_ENABLED=false CORE_CHAINCODE_ID_NAME=mycc:1.0 ./node_modules/.bin/fabric-chaincode-node start -peer.address 127.0.0.1:7052

```

## In terminal 3
In terminal 3 we use the chaincode with CLI commands.

```bash
# set environment vars
source org1.env
setGlobals

# test the chaincode
CORE_PEER_ADDRESS=127.0.0.1:7051 peer chaincode invoke -o 127.0.0.1:7050 -C ch1 -n mycc -c '{"Args":["set","k1","Hello World!"]}'

CORE_PEER_ADDRESS=127.0.0.1:7051 peer chaincode query -o 127.0.0.1:7050 -C ch1 -n mycc -c '{"Args":["get","k1"]}' | jq .
```

[Index](./index.md)