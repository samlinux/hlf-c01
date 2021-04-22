# Installing the sdg-dev-network
The **sdg-dev-network** is optimized for this course and not suitable for productive use. Its based on the test-network but reduced the usage of this course. Figure 4 shows its component. 

The sdg-dev-network consists with one orderer and one peer node. A channel called **ch1** and a predefined chaincode called **basic**. The chaincode stores simply a key and a value. There are two chaincode functions available. A set function to create and update an asset in the blockchain and a get function to query the latest state of an existing asset.

<figure class="image">
  <img src="img/04_sdg-dev-network.png" alt="Overview sdg-dev-network">
  <figcaption>Figure 4</figcaption>
</figure>

The following steps are required to get started with the dev-network.

We need three terminals to run the network. To do so we can use tmux or three different ssh terminals.

>tmux is a terminal multiplexer. It lets you switch easily between several programs in one terminal, detach them (they keep running in the background) and reattach them to a different terminal.

In this course we are going to use tmux because of its ability to detach and reattach a program to the terminal window.

We can use the sdg-dev-network for the following development use-cases:

- Chaincode development, certificates generated with cryptogen
  - Chaincode development with levelDb as state database
  - Chaincode development with CouchDb as state database
- Chaincode development, certificates generated with fabric-ca
  - Chaincode development with levelDb as state database
  - Chaincode development with CouchDb as state database

But before we can use it we have to install it.

## Installation and frist try

The sdg-dev-network can be cloned from the course repository.

```bash
# switch into the base install folder
cd fabric

# clone the repository
git clone git@sun.samlinux.at:/opt/git/sdg-dev-network.git

# switch into the network folder
cd sdg-dev-network
```

The sdg-dev-network can be sumerized as follows:

- We need a total of three different terminals, with tmux we call these panels.
- Panel 1 is running the docker compose network.
- Panel 2 is running the chaincode.
- Panel 3 is used to interact with the sdg-dev-network.

The typical start-up process of the sdg-dev-network can be sumerized as follows:

- Create a new tmux sesion.
- Start the sdg-dev-network.
- Split the panel and create a new panel, the second panel. Now you have two panels in on terminal running.
- Switch into the chaincode folder.
- Start the chaincode. This is also the place to stop and restart the chaincode every time if you make a change to the chaincode. After a restart of the chaincode you have immediate access to your chances you have made.
- Split the panel again and create an another new panel, the third panel.
- The third panel is used for testing the chaincode. To interact with the network we have to set proper environment variables.
- Use the peer chaincode command with the subcommand query or invoke to call the chaincode functions.
- Detach from the tmux session if you have finished your work. But the network is still running in the background and you can come back later to continue work.

After we have understood the basic process, we can take the following steps in sequence.

Before we start let's make us familiar with some tmux commands we are going to use. 

>**Create a new tmux session**<br> 
tmux new -s dev

>**Create a new pane horizontally**<br> 
CTRL + b " (double-quots)

>**Switch/select pane by number**<br> 
CTRL + b q  0 ... 9

>**Detach from session**<br> 
CTRL + b d 

>**Show all sessions**<br> 
tmux ls

>**Attach to session dev**<br> 
tmux att -t dev

>**Delete/kill session dev**<br> 
tmux kill-ses -t mysession

## Panel 1

```bash
# start new tmux session
tmux new -s dev

# start the network
./devNetwork.sh up
```
At this time the sdg-deb-network is running.

Split the panel 1 and create panel 2.
```bash
CTRL + b " 
```

## Panel 2
In panel 2 we start and stop the chaincode itself.

```bash
# switch to the chaincode folder
cd chaincode/nodejs/sacc

# install all dependencies only for the first time
npm install 

# start the chaincode
CORE_CHAINCODE_LOGLEVEL=debug CORE_PEER_TLS_ENABLED=false CORE_CHAINCODE_ID_NAME=mycc:1.0 ./node_modules/.bin/fabric-chaincode-node start --peer.address 127.0.0.1:7052
```

Split the panel 2 and create panel 3.
```bash
CTRL + b " 
```

## Panel 3
In panel 3 we use the chaincode with CLI commands.

```bash
# load and set environment variables for Org1
source org1.sh
setGlobals

### test the chaincode

# Create a new asset
CORE_PEER_ADDRESS=127.0.0.1:7051 peer chaincode invoke -o 127.0.0.1:7050 -C ch1 -n mycc -c '{"Args":["set","k1","Hello World!"]}'

# Query the new asset
CORE_PEER_ADDRESS=127.0.0.1:7051 peer chaincode query -o 127.0.0.1:7050 -C ch1 -n mycc -c '{"Args":["get","k1"]}' | jq .
```

Congratulations you have successfully installed and tested the sdg-dev-network. Try to recap all the steps and play around with that chaincode.


## Detach from the session
The leave the session we can use the following tmux command.

```bash
CTRL + b d
```

## Attach to the dev session again
We can get back to work by attaching to the session.

```bash
tmux att -t dev
```

>One final note. When you restart your machine then all tmux sessions are deleted.


[Index](./index.md)