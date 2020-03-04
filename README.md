# Setting up for Fabric EVM

Hyperledger Fabric EVM which allows developers to create blockchain application with EVM (Ethereum Virtual Machine) smart contract languages such as Solidity. This smart contract is deployed onto the EVM chaincode running on the Fabric peers. Fab3 allows for using the Ethereum web3.js library for interacting with the EVM in the Fabric network.

This README is for developers looking to create blockchain applications with Hyperledger Fabric by using Solidity smart contracts and the web3.js library. When the reader has completed this setup, they will understand how to:

* Deploy an instance of Hyperledger Fabric locally with the EVM chaincode
* Start a Fab3 instance to interact with the Fabric network

# Included Components

* [Hyperledger Fabric v1.4](https://hyperledger-fabric.readthedocs.io/en/latest/) Hyperledger Fabric is a platform for distributed ledger solutions, underpinned by a modular architecture delivering high degrees of confidentiality, resiliency, flexibility and scalability
* [Hyperledger Fabric EVM chaincode](https://github.com/hyperledger/fabric-chaincode-evm) This project enables one to deploy Ethereum smart contracts written in an EVM compatible language such as Solidity or Vyper on the Hyperledger Fabric permissioned blockchain platform.
* [Ethereum web3.js](https://web3js.readthedocs.io/en/1.0/) web3.js is a library which allow you to interact with a local or remote nodes that implement the Ethereum JSON RPC API, using a HTTP or IPC connection.

# Setting Up

## Prerequisite
- [Go](https://golang.org/dl/) (version 1.14)
- [Docker](https://www.docker.com/) (version 19.03.x or greater)
- [Node](https://nodejs.org/en/) (version 12.13.x or greater)
- [npm](https://www.npmjs.com/) (version 6.12.x)

## Steps

1. [Deploy Hyperledger Fabric locally with EVM chaincode](#1-deploy-hyperledger-fabric-locally-with-evm-chaincode)
    - [Clean docker and set GOPATH](#clean-docker-and-set-gopath)
    - [Get Fabric Samples and download Fabric images](#get-fabric-samples-and-download-fabric-images)
    - [Mount the EVM Chaincode and start the network](#mount-the-evm-chaincode-and-start-the-network)
    - [Install and Instantiate EVM Chaincode](#install-and-instantiate-evm-chaincode)
2. [Setup Fab3](#2-setup-fab3)

## 1. Deploy Hyperledger Fabric locally with EVM chaincode

In this step, we will setup fabric locally using docker containers, install the EVM chaincode (EVMCC) and instantiate the EVMCC on our fabric peers. This step uses the hyperledger [fabric-sample](https://github.com/hyperledger/fabric-samples) repo to deploy fabric locally and the [fabric-chaincode-evm](https://github.com/hyperledger/fabric-chaincode-evm) repo for the EVMCC, Fab3 and Fab3.  This step follows the [fabric-chaincode-evm tutorial](https://github.com/hyperledger/fabric-chaincode-evm/blob/master/examples/EVM_Smart_Contracts.md) closely.

### Clean docker and set GOPATH

This will remove all your hyperledger docker images!
```
docker rmi -f $(docker images | grep hyperledger | awk '/^.*\s/ {print $3}')
```

Make sure to set GOPATH to your go installation
```
mkdir $HOME/.go
export GOPATH=$HOME/.go
export PATH=$PATH:$GOPATH
```

### Get Fabric Samples and download Fabric images

Clone the [fabric-samples](https://github.com/hyperledger/fabric-samples) repo in your `GOPATH/src/github.com/hyperledger` directory:
```
mkdir -p $GOPATH/src/github.com/hyperledger
cd $GOPATH/src/github.com/hyperledger/
git clone https://github.com/hyperledger/fabric-samples.git
```

Checkout `release-1.4`
```
cd fabric-samples
git checkout release-1.4
```

Make binaries available on your path
```
export PATH=$GOPATH/src/github.com/hyperledger/fabric-samples/bin:$PATH
```

Download the docker images
```
curl -sSL https://bit.ly/2ysbOFE | bash -s -- 1.4.0 1.4.0 0.4.18
```

### Mount the EVM Chaincode and start the network

Clone the `fabric-chaincode-evm` repo in your GOPATH directory.
```
cd $GOPATH/src/github.com/hyperledger/
git clone https://github.com/hyperledger/fabric-chaincode-evm
```

Checkout `release-0.1`
```
cd fabric-chaincode-evm
git checkout release-0.1
```


Now navigate back to your fabric-samples folder.  Here we will use first-network to launch the network.
```
cd $GOPATH/src/github.com/hyperledger/fabric-samples/first-network
```

Update the `docker-compose-cli.yaml` with the volumes to include the fabric-chaincode-evm.

```
  cli:
    volumes:
      - ./../../fabric-chaincode-evm:/opt/gopath/src/github.com/hyperledger/fabric-chaincode-evm
```

Generate certificates and bring up the network
```
./byfn.sh generate
./byfn.sh up
```

### Install and Instantiate EVM Chaincode

Navigate into the cli docker container
```
docker exec -it cli bash
```

If successful, you should see the following prompt
```
root@0d78bb69300d:/opt/gopath/src/github.com/hyperledger/fabric/peer#
```

To change which peer is targeted change the following environment variables:
```
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```

Next install the EVM chaincode on all the peers
```
peer chaincode install -n evmcc -l golang -v 0 -p github.com/hyperledger/fabric-chaincode-evm/evmcc
```

Instantiate the chaincode:

```
peer chaincode instantiate -n evmcc -v 0 -C mychannel -c '{"Args":[]}' -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

Great.  You can exit out of the cli container and return to your terminal.
```
exit
```

You are now ready to setup Fab3.


## 2. Setup Fab3


Execute the following to set certain environment variables required for setting up Fab3.

```
export FABPROXY_CONFIG=${GOPATH}/src/github.com/hyperledger/fabric-chaincode-evm/examples/first-network-sdk-config.yaml # Path to a compatible Fabric SDK Go config file
export FABPROXY_USER=User1 # User identity being used for the proxy (Matches the users names in the crypto-config directory specified in the config)
export FABPROXY_ORG=Org1  # Organization of the specified user
export FABPROXY_CHANNEL=mychannel # Channel to be used for the transactions
export FABPROXY_CCID=evmcc # ID of the EVM Chaincode deployed in your fabric network
export PORT=5000 # Port the proxy will listen on. If not provided default is 5000.
```

Navigate to the `fabric-chaincode-evm` cloned repo:
```
cd $GOPATH/src/github.com/hyperledger/fabric-chaincode-evm/
```
Run the following to build the fab proxy
```
go build -o fab3 ./fabproxy/cmd
```
You can then run the proxy:
```
./fab3
```

This will start Fab3 at `http://localhost:5000`

## 3. Connecting to the Proxy

The following directions require ``node`` and ``web3`` to be installed.
To install the same version of `web3` run:
```
npm install web3@0.20.2
```

After installing the correct version of `web3`, in a node session run the
following to connect to the proxy:

```
  > Web3 = require('web3')
  ...
  > web3 = new Web3(new Web3.providers.HttpProvider('http://localhost:5000'))
```

If successful you should be able to get your account address. The first query or transaction you run with the proxy will take a little
longer than others since the SDK is using the discovery service to find out about all the peers on the network.

```
  > web3.eth.accounts
```

And you should see an single element array with your account address.
In order to run any transactions web3 requires `web3.eth.defaultAccount` to be set

```
  > web3.eth.defaultAccount = web3.eth.accounts[0]
```

## Additional Documentation

* [Solidity contract with Truffle](./docs/truffle-commands.md)
* [Using remix to get ABI and bytecode](./docs/using-remix.md)
* [Setup Fab Proxy and using Web3](./docs/proxy-web3-commands.md)
* [Loyalty Program Use Case](./docs/use-case.md)

## Links
* [Fabric Chaincode EVM](https://github.com/hyperledger/fabric-chaincode-evm)
* [Hyperledger Fabric Docs](http://hyperledger-fabric.readthedocs.io/en/latest/)
* [Solidity](https://solidity.readthedocs.io/en/v0.4.25/index.html)
