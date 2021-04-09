# Hyperledger Fabric Installation
Below you can find the detailed step by steps instructions for installating a Hyperledger fabric system according to the official Hyperledger fabric documentation.

Watch the video of this chapter to follow the steps below.

## Setup
These steps describe a HLF 2.2.x installation on an Ubuntu 20.04 operating system.

First, I would like to give you an overview of the installation steps. We can divide the installation process into 5 steps. Figure 1 shows these steps as a whole.

<figure class="image">
  <img src="img/install1.png" alt="Overview Installation">
  <figcaption>Figure 1</figcaption>
</figure>

After finishing these steps you will have a ready Hyperledger Fabric Installation and a running test-network.

## Droplet or virtuel private server 
In this course we will use a so called Digital Ocean Droplet with the following configuration: 1 CPU, 2 GB, 50 GB SSD. This server setup should be sufficient to perform all of the examples in this course.

If your are more familare with a VirtualBox setup then you are good to go as well. The starting point for the installation is in any case a basic installation of Ubuntu 20.04.

## System update
The following steps are required to prepare your ubuntu system for the fabric installation process. Let's discuss the steps in detail. 

First, we do a system update. Make sure you are working as root user.

```bash
# update the OS
apt update && apt upgrade
```

Second, we install some helper tools. The tools are tree, jq, gcc and make. 

The bash **tree** command is used to display the contents of any desired directory of your computer system in the form of a tree structure. This is helpful to inspect the content of a folder. 

According to the **jq** homepage, jq is like sed for JSON data - you can use it to slice and filter and map and transform structured data with the same ease that sed, awk, grep and friends let you play with text. In our example we can use jq to parse the json response from our blockchain to get to get a nicely formatted output in the terminal.

Finally the command line tools **gcc and make** are needed to install the Fabric Node.js SDK to interact with the Fabric system e.g. if you want to query a peer or invoke some transaction. 

```bash
# install some useful helpers
apt install tree jq gcc make
```
The third and final step is to setup the right timezone according your needs. Because the authors of this course are located in Austria we set the timeuzone to Europe/Vienna. Feel free to set you timezone. If your unsure what is the correct timezone check the **timedatectl list-timezones** command for a list of possible timezones.

```bash
# setup the correct timezone to fit your needs
timedatectl set-timezone Europe/Vienna

# check the time
timedatectl
```
Now that we have completed the system prep, we can move on to the next important item. The Docker installation.

## Install Docker
The following steps are required to install docker on the Droplet. Reference: https://docs.docker.com/engine/install/ubuntu/

```bash
# set up the repository
sudo apt install \
  apt-transport-https \
  ca-certificates \
  curl \
  gnupg-agent \
  software-properties-common

# add Docker’s official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -


# set up the stable repository
sudo add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"

# install docker engine
apt update
apt install docker-ce docker-ce-cli containerd.io

# check the docker version
docker --version
```

## Install Docker-Compose

Reference https://docs.docker.com/compose/install/

```bash
# install docker-compose
curl -L "https://github.com/docker/compose/releases/download/1.29.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# apply executable permissions to the binary
sudo chmod +x /usr/local/bin/docker-compose

# check the docker-compose version
docker-compose --version
```

## Install Go Programming Language
Hyperledger Fabric uses the Go Programming Language for many of its components. Go version 1.14.x is required.

```bash 
# download and extract go
# latest version 08.04.21 1.14.9
wget -c https://dl.google.com/go/go1.14.9.linux-amd64.tar.gz -O - | tar -xz -C /usr/local

# add the go binary to the path
echo 'export PATH="$PATH:/usr/local/go/bin:/root/fabric/fabric-samples/bin"' >> $HOME/.profile

# point the GOPATH env var to the base fabric workspace folder
echo 'export GOPATH="$HOME/fabric"' >> $HOME/.profile

# reload the profile
source $HOME/.profile

# check the go version
go version

# check the vars
printenv | grep PATH
```

## Install node.js
Hyperledger Fabric uses Node.js for Chaincode and Client development. Node.js version 1.12.x is required.

```bash
# add PPA from NodeSource
curl -sL https://deb.nodesource.com/setup_12.x -o nodesource_setup.sh

# call the install script
. nodesource_setup.sh

# install node.js
apt-get install -y nodejs

# check the version
node -v
```

## Install Samples, Binaries and Docker Images

```bash
mkdir fabric
cd fabric

# install the latest production release from the 1.4.x branch
# we use 2.2 in our examples
curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.2.1 1.4.9

# check downloaded images
docker images

# check the bin cmd
peer version
```

## Try the installation
The fabric-samples provisions a sample Hyperledger Fabric test-network consisting of two organizations, each maintaining one peer nodes. It also will deploy a single RAFT ordering service by default. 

To test your installationen we can start interacting with the network.

```bash
# switch to the base folder
cd fabric-samples/test-network

# print some help
./network.sh --help

# bring up the network
./network.sh up createChannel -c channel1

# install default CC - asset-transfer (basic) chaincode
./network.sh deployCC -c channel1

# show if some containers are running
docker ps
docker-compose -f docker/docker-compose-test-net.yaml ps
```

## Interacting with the network

tmux control
```bash
# start a new tmux session
tmux new -s fabric

# attach to existing session
tmux add -t fabric

# show all logs
docker-compose -f docker/docker-compose-test-net.yaml logs -f -t

# open a new panel
CTRL + b (release pressed keys) + \""

# jump between panels
CTRL + b + q 1

# detach from session
CTRL + b + d

```

## Environment variables for peer Org1

```bash
# create an env file
. scripts/envVars.sh
setGlobals 1

# check env vars
printenv | grep CORE
```

### Initialize the leder (sample data)
Run the following command to initialize the ledger with assets:
```bash

# for explanation
peer chaincode invoke 
  -o localhost:7050 
  --ordererTLSHostnameOverride orderer.example.com 
  --tls 
  --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem 
  -C $CHANNEL_NAME 
  -n basic 
  --peerAddresses localhost:7051 
  --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt 
  --peerAddresses localhost:9051 
  --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt 
  -c '{"function":"InitLedger","Args":[]}'


# for copy and paste
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"InitLedger","Args":[]}'
```

### Query the leder

```bash
# Read the last state of all assets
peer chaincode query -C $CHANNEL_NAME -n basic -c '{"Args":["GetAllAssets"]}' | jq .

# Read an asset 
peer chaincode query -C $CHANNEL_NAME -n basic -c '{"Args":["ReadAsset","asset1"]}' | jq .
```

### Create an asset
```bash

peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"CreateAsset","Args":["asset7","green","10","Roland","500"]}'
```

### Update an asset
```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"UpdateAsset","Args":["asset7","green","10","Roland","600"]}'
```

### Transfer an asset
```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"TransferAsset","Args":["asset7","Joana"]}'
```

### Delete an asset
```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"DeleteAsset","Args":["asset1"]}'
```

### Switch to peer Org2
We can switch to work with peer Org2 peer0.org2.example.com with changeing the following evironment variables. 
```bash 

# create an env file
setGlobals 2
```

## Bring down the network
```bash
./network.sh down
```
[Index](./index.md)