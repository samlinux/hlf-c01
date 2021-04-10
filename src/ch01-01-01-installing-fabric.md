# Hyperledger Fabric (HLF) Installation
Below you can find the detailed step by steps instructions for installating a HLF system according to the official HLF documentation.

Watch the video in this chapter to complete the following steps.

## Setup
These steps describe a HLF 2.2.x installation on an Ubuntu 20.04 operating system.

First, I would like to give you an overview of the installation steps. We can divide the installation process into 5 steps. Figure 1 shows these steps as a whole.

<figure class="image">
  <img src="img/install1.png" alt="Overview Installation">
  <figcaption>Figure 1</figcaption>
</figure>

After finishing these steps you will have a ready HLF Installation and a running test-network.

## Preparations
### Droplet or virtuel private server 
In this course we will use a so called Digital Ocean Droplet with the following configuration: 1 CPU, 2 GB, 50 GB SSD. This server setup should be sufficient to perform all of the examples in this course.

If your are more familare with a VirtualBox setup then you are good to go as well. The starting point for the installation is in any case a basic installation of Ubuntu 20.04.

### System update
The following steps are required to prepare your ubuntu system for the HLF installation process. Let's discuss the steps in detail. 

First, we do a system update. Open a terminal and make sure you are working as root user.

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
The third and final step is to setup the right timezone according your needs. Because the authors of this course are located in Austria we set the timezone to Europe/Vienna. Feel free to set the timezone according your needs. If your unsure what is the correct timezone check the **timedatectl list-timezones** command for a list of possible timezones and pick the right one.

```bash
# setup the correct timezone to fit your needs
timedatectl set-timezone Europe/Vienna

# check the time
timedatectl
```
Now that we have completed the system prep, we can move on to the next important item. The Docker installation.

### Install Docker
HLF is basically build on **Docker** images, but there is also a way to use binaries. For the purposes of this course, we will use the prepared Docker images. The foundation for that is running a **Docker** host on your operation system. 

>Docker is an open platform for developing, shipping, and running applications. Docker enables you to separate your applications from your infrastructure so you can deliver software quickly.

Use the commands below and step by step copy and paste these into your terminal to install Docker for Ubuntu 20.04. Reference: https://docs.docker.com/engine/install/ubuntu/

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
If you see a output similar like **Docker version 20.10.5, build xxx** you have successfully installed Docker.

Congratulations at this point you have successfully installed Docker on your linux box.

### Install Docker Compose

>**Compose** is a tool for defining and running multi-container Docker applications. With **Compose**, we use a YAML file to configure our Fabric network.

Use the commands below and step by step copy and paste these into your terminal to install Docker Compose. Reference https://docs.docker.com/compose/install/

```bash
# install docker-compose
curl -L "https://github.com/docker/compose/releases/download/1.29.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# apply executable permissions to the binary
chmod +x /usr/local/bin/docker-compose

# check the docker-compose version
docker-compose --version
```
If you see a output like **docker-compose version 1.29.0, build xxxx** you have successfully installed Docker Compose.

Congratulations at this point you have also successfully installed Docker Compose on your linux box. We are now ready to use Docker and Docker Compose for our HLF journey.

### Install Go Programming Language

HLF uses the Go Programming Language for many of its components. 

>Go is an open source programming language that makes it easy to build simple, reliable, and efficient software. 

Although this course is based on Node.js for chaincode as well as for client development, we should install **Go** on our training machine.

Use the commands below and step by step copy and paste these into your terminal to install Go. For this we follow the official documentation. Referene https://golang.org/doc/install

First, we download and unzip the go binary for our system. 
```bash 
# download and extract go
# latest 1.14.x version 08.04.21 1.14.9
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.14.9.linux-amd64.tar.gz
```

Second, we add the path **/usr/local/go/bin** to the PATH environment variable.
```bash
# add the go binary to the path
echo 'export PATH="$PATH:/usr/local/go/bin:"' >> $HOME/.profile
```

Third, to apply the changes immediately, just run the shell command **source $HOME/.profile**.

>**Note**: A profile file is a start-up file of an UNIX user, like the autoexec. bat file of DOS. When a UNIX user tries to login to his account, the operating system executes this file to set up the user account before returning the prompt to the user.

```bash
# reload the profile
source $HOME/.profile
```

Now we can check the go version.
```bash
# check the go version
go version
```

If you see a output like **go version go1.14.15 linux/amd64** you have successfully installed Go.

### Install Node.js
In this course we uses Node.js for chaincode and client development. According to the HLF documentation Node.js version 1.12.x is required.

>Node.js is a JavaScript runtime built on Chrome's V8 JavaScript engine.

Use the commands below and step by step copy and paste these into your terminal to install Node.js.

First, download the install script.
```bash
# add Personal Package Archives (PPA) from NodeSource
curl -sL https://deb.nodesource.com/setup_12.x -o nodesource_setup.sh
```

Second, execute the script. The script will prepare the apt source on your ubuntu maschine to install Node.js via the ubuntu default install tool apt.

>**Note**: The period (dot) is short hand for the bash built in source. It will read and execute commands from a file in the current environment and return the exit status of the last command executed. It does not need to be executable.

```bash
# execute the install script
. nodesource_setup.sh
```

Third, install Node.js. 

>**Note**: What is the meaning of -y ? Automatic yes to prompts; assume "yes" as answer to all prompts and run non-interactively. 

```bash
# install node.js
apt install -y nodejs
```

Finally check the installed Node.js version.
```bash
# check the version
node -v
```

If you see a output like **v12.22.1** you have successfully installed Node.js.

At this point we have made all preparations in place to finish the HLF setup.

## Install Fabric Samples, Binaries and Docker Images

Determine a location on your machine where you want to place the base folder for the fabric-samples repository. In this course we call that folder fabric. The command that follows will perform the following steps:

* Clone the hyperledger/fabric-samples repository
* Checkout the appropriate version tag
* Install the Hyperledger Fabric platform-specific binaries and config files for the version specified into the /bin and /config directories of fabric-samples
* Download the Hyperledger Fabric docker images for the version specified

First, create your base folder and switch onto that folder.

```bash
mkdir fabric && cd fabric
```

Second, execute the following **curl** command to install HLF.

We want a specific release for our course, pass a version identifier for Fabric and Fabric-CA docker images. The command below demonstrates how to download the latest production releases - Fabric v2.2.2 and Fabric CA v1.4.9

>Note: curl -sSL https://bit.ly/2ysbOFE | bash -s -- <fabric_version> <fabric-ca_version>

This command downloads and executes a bash script that will download and extract all of the platform-specific binaries you will need to set up your network and place them into the cloned repo you created above.

```bash
# this course uses HLF version 2.2.2
curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.2.2 1.4.9
```
After the sucessfully installation you should have the folowing binaries under the **bin** sub-directory of the current working directory.

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

In order to be able to access these binaries quickly and easily, we add the path to the **bin** sub-folder to the PATH environment variable and reload the **.profile** file.

```bash
echo 'export PATH="$PATH:/root/fabric/fabric-samples/bin"' >> $HOME/.profile
source $HOME/.profile
```

Finally we can check our installation with the following command.
```bash
# check the bin cmd
peer version
```

If you see a output like below then you have successfully installed HLF.

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