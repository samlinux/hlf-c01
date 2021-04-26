# Hyperledger Fabric (HLF) Installation

## Chapter Overview

In this chapter you can find the detailed step by steps instructions for installing a HLF system according to the official HLF documentation. We walk through all the nessesary steps to install HLF on your virtual machine.

Watch the video in this chapter to complete the following steps.

## Learning Objectives

By the end of this chapter you will be able to:

- Understand what are the building blocks of an HLF installation.
- Install HLF on an Ubuntu 20.04 operating system and run the test-network.

## Prerequisites
As this course is based on Ubuntu 20.04 operating system, please prepare a virtual machine with a running Ubuntu 20.04 operating system. You have various options for doing this. 

You can either use one of the public infrastructure as a service (IaaS) providers such as Digital Ocean, AWS, Google Cloud or Azure or you can use VirtualBox locally on your computer and use an Ubuntu 20.04 image. The basic installation of Ubuntu 20.04 is out of the scope of this course. To learn more about the manual installation of Ubuntu please refer to the official documentation on https://ubuntu.com/server/docs/installation. 

In this course a so-called Digital Ocean **Droplet** is used with the following configuration as a reference: 1 CPU, 2 GB, 50 GB SSD. This virtual machine configuration should be sufficient to perform all examples in this course and to spin up in less then one minute.

## Setup
The following steps show a HLF 2.2.x installation on an Ubuntu 20.04 operating system. Figure 1 gives an overview of the installation process in five steps.

<figure class="image">
  <img src="img/install1.png" alt="Overview Installation">
  <figcaption>Figure 1</figcaption>
</figure>

After finishing these steps you will have a ready HLF installation and a running test-network.

### System update
The following steps are required to prepare the Ubuntu 20.04 system for the HLF installation process. Let's discuss the steps in detail. 

Firstly, execute a system update. Open a terminal window and make sure you are working as root user. Copy and paste the command below into the terminal to update the system. A system restart may be required.

```bash
# update the OS
apt update && apt upgrade
```

Secondly, install some helper tools. The tools are **tree, jq, gcc and make**. 

The bash **tree** command is used to display the contents of any desired directory of your computer system in the form of a tree structure. This is helpful to inspect the content of a folder. 

According to the **jq** homepage, jq is like sed for JSON data - you can use it to slice and filter and map and transform structured data with the same ease that sed, awk, grep and friends let you play with text. In our example we can use jq to parse the json response from our blockchain to get a nicely formatted output in the terminal.

The command line tools **gcc and make** are needed to install the Fabric Node.js SDK to interact with the Fabric system, e.g. if you want to query a peer or invoke some transaction. 

Copy and paste the following command into the terminal to continue.
```bash
# install some useful helpers
apt install tree jq gcc make
```
The third and final step is to setup the right timezone. The authors set the timezone to Europe/Vienna. Feel free to set the timezone according to your needs. If your are unsure what is the correct timezone, check the **timedatectl list-timezones** command for a list of possible timezones and pick the right one.

```bash
# setup the correct timezone to fit your needs
timedatectl set-timezone Europe/Vienna

# check the time
timedatectl
```

As we have completed the system preparation, we can move on to the next important step - the Docker installation.

### Install Docker
HLF is basically built on **Docker** images, but there is also a way to use binaries. For the purposes of this course we will use the prepared Docker images. The foundation for that is running a **Docker** host on your operation system. 

>Docker is an open platform for developing, shipping, and running applications. Docker enables you to separate your applications from your infrastructure so you can deliver software quickly.

Use the commands below and step by step copy and paste them into your terminal to install Docker for Ubuntu 20.04. Reference: https://docs.docker.com/engine/install/ubuntu/

```bash
# set up the repository
apt install \
  apt-transport-https \
  ca-certificates \
  curl \
  gnupg-agent \
  software-properties-common

# add Docker’s official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -

# set up the stable repository
add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"

# install docker engine
apt update
apt install docker-ce docker-ce-cli containerd.io

# check the docker version
docker --version
```
If you see an output similar like **Docker version 20.10.5, build 55c4c88**, congratulations! You have successfully installed Docker on your Linux box.

### Install Docker Compose

>**Compose** is a tool for defining and running multi-container Docker applications. With **Compose** we use a YAML file to configure our Fabric network.

Use the commands below and step by step copy and paste them into your terminal to install Docker Compose. Reference https://docs.docker.com/compose/install/

```bash
# install docker-compose
curl -L "https://github.com/docker/compose/releases/download/1.29.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# apply executable permissions to the binary
chmod +x /usr/local/bin/docker-compose

# check the docker-compose version
docker-compose --version
```
If you see an output like **docker-compose version 1.29.0, build 5db8d86f**, congratulations! You have successfully installed Docker Compose on your Linux box. We are now ready to use Docker and Docker Compose for our HLF journey.

### Install Go Programming Language

HLF uses the Go Programming Language for many of its components. 

>Go is an open source programming language that makes it easy to build simple, reliable, and efficient software. 

Although this course is based on Node.js for chaincode as well as for client development, we should install **Go** on our training machine.

Use the commands below and step by step copy and paste these into your terminal to install Go. For this process we follow the official documentation. Reference https://golang.org/doc/install

Firstly, download and unzip the go binary for your system. 
```bash 
# download and extract go
# latest 1.14.x version 08.04.21 1.14.9
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.14.9.linux-amd64.tar.gz
```

Secondly, add the path **/usr/local/go/bin** to the PATH environment variable.
```bash
# add the go binary to the path
echo 'export PATH="$PATH:/usr/local/go/bin:"' >> $HOME/.profile
```

Thirdly, to apply the changes immediately, just run the shell command **source $HOME/.profile**.

>**Note**: A profile file is a start-up file of an UNIX user, like the autoexec. bat file of DOS. When a UNIX user tries to login to his account, the operating system executes this file to set up the user account before returning the prompt to the user.

```bash
# reload the profile
source $HOME/.profile
```

Now, check the go version.
```bash
# check the go version
go version
```

If you see an output like **go version go1.14.15 linux/amd64**, congratulations! You have successfully installed Go.

### Install Node.js
In this course we use Node.js for chaincode and client development. According to the HLF documentation, Node.js version 1.12.x is required.

>Node.js is a JavaScript runtime built on Chrome's V8 JavaScript engine.

Use the commands below and step by step copy and paste these commands into your terminal to install Node.js.

Firstly, download the install script.
```bash
# add Personal Package Archives (PPA) from NodeSource
curl -sL https://deb.nodesource.com/setup_12.x -o nodesource_setup.sh
```

Secondly, execute the script. The script prepares the apt source on your Ubuntu machine to install Node.js using standard Ubuntu installer apt.

>**Note**: The period (dot) is short hand for the bash built in source. It will read and execute commands from a file in the current environment and return the exit status of the last command executed. It does not need to be executable.

```bash
# execute the install script
. nodesource_setup.sh
```

Thirdly, install Node.js. 

>**Note**: What is the meaning of -y ? Automatic yes to prompts; assume "yes" as answer to all prompts and run non-interactively. 

```bash
# install node.js
apt install -y nodejs
```

Finally, check the installed Node.js version.
```bash
# check the version
node -v
```

If you see an output like **v12.22.1**, congratulations! You have successfully installed Node.js.

At this point you have all preparations in place to finish the HLF setup.

## Install Fabric Samples, Binaries and Docker Images

Determine a location on your machine where you want to place the base folder for the fabric-samples repository. In this course we call that folder **fabric**. The command that follows performs the required steps:

* Clone the hyperledger/fabric-samples repository
* Checkout the appropriate version tag
* Install the Hyperledger Fabric platform-specific binaries and config files for the version specified into the /bin and /config directories of fabric-samples
* Download the Hyperledger Fabric docker images for the version specified

Firstly, create your base folder and switch to that folder.

```bash
mkdir fabric && cd fabric
```

Secondly, execute the following **curl** command to install HLF.

As we want a specific release for our course, pass a version identifier for Fabric and Fabric-CA docker images. The command below demonstrates how to download the latest production releases - Fabric v2.2.2 and Fabric CA v1.4.9

>Note: curl -sSL https://bit.ly/2ysbOFE | bash -s -- <fabric_version> <fabric-ca_version>

The command below downloads and executes a bash script that downloads and extracts all platform-specific binaries you will need to set up your network and place them into the cloned repo you created above.

This process can take a while, depending on your internet speed.

```bash
# this course uses HLF version 2.2.2
curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.2.2 1.4.9
```
After the sucessful installation you should have the following binaries under the **bin** sub-directory of the current working directory and the following Docker images as well.

```bash
root@fabric:~/fabric: tree fabric-samples/bin/
fabric-samples/bin/
├── configtxgen
├── configtxlator
├── cryptogen
├── discover
├── fabric-ca-client
├── fabric-ca-server
├── idemixgen
├── orderer
└── peer

0 directories, 9 files
```

Check your Docker images with the command: **docker images**

```bash
root@fabric:~: docker images
REPOSITORY                   TAG       IMAGE ID       CREATED        SIZE
couchdb                      3.1.1     6763bc849b36   13 days ago    190MB
busybox                      latest    a9d583973f65   4 weeks ago    1.23MB
hyperledger/fabric-tools     2.2       212bfff67240   2 months ago   436MB
hyperledger/fabric-tools     2.2.2     212bfff67240   2 months ago   436MB
hyperledger/fabric-tools     latest    212bfff67240   2 months ago   436MB
hyperledger/fabric-peer      2.2       0db1edb6ddd8   2 months ago   55MB
hyperledger/fabric-peer      2.2.2     0db1edb6ddd8   2 months ago   55MB
hyperledger/fabric-peer      latest    0db1edb6ddd8   2 months ago   55MB
hyperledger/fabric-orderer   2.2       89cac0d3ab9b   2 months ago   38.5MB
hyperledger/fabric-orderer   2.2.2     89cac0d3ab9b   2 months ago   38.5MB
hyperledger/fabric-orderer   latest    89cac0d3ab9b   2 months ago   38.5MB
hyperledger/fabric-ccenv     2.2       989d60213726   2 months ago   502MB
hyperledger/fabric-ccenv     2.2.2     989d60213726   2 months ago   502MB
hyperledger/fabric-ccenv     latest    989d60213726   2 months ago   502MB
hyperledger/fabric-baseos    2.2       0483f4bff906   2 months ago   6.85MB
hyperledger/fabric-baseos    2.2.2     0483f4bff906   2 months ago   6.85MB
hyperledger/fabric-baseos    latest    0483f4bff906   2 months ago   6.85MB
hyperledger/fabric-ca        1.4       dbbc768aec79   6 months ago   158MB
hyperledger/fabric-ca        1.4.9     dbbc768aec79   6 months ago   158MB
hyperledger/fabric-ca        latest    dbbc768aec79   6 months ago   158MB
```


In order to be able to access these binaries quick and easy, add the path to the **bin** sub-folder to the PATH environment variable and reload the **.profile** file.

```bash
echo 'export PATH="$PATH:/root/fabric/fabric-samples/bin"' >> $HOME/.profile
source $HOME/.profile
```

Finally, check your installation with the following command.
```bash
# check the bin cmd
peer version
```

If you see an output like below, congratulations! You have successfully installed HLF on your Linux box.

```bash
root@fabric:~/fabric: peer version
peer:
 Version: 2.2.2
 Commit SHA: bebb75fd1
 Go version: go1.14.12
 OS/Arch: linux/amd64
 Chaincode:
  Base Docker Label: org.hyperledger.fabric
  Docker Namespace: hyperledger
```

[Index](./index.md)
