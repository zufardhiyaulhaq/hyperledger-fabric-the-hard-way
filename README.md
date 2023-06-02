# Hyperledger Fabric The Hard Way
This tutorial walks you through setting up Hyperledger Fabric the hard way. There is no script needed.

*The results of this tutorial should not be viewed as production ready, but the writter is following Production Network approach, may receive limited support from the community, but don't let that stop you from learning!*

## Cluster Details
- CA Fabric 1.5.2
- Fabric 2.4.3

## Labs
This lab requires you to have an understanding of, and have installed, both Vagrant and Virtualbox.
- [Provisioning Infrastructure](docs/00-infrastructure.md)
- [Installing prerequisites tools](docs/01-prerequisites.md)
- [Setup Certificate Authority](docs/02-setup-ca.md)
- [Setup TLS Certificate Authority Server](docs/03-setup-tls-fabric-ca-server.md)
- [Setup Enrollment Certificate Authority Server](docs/04-setup-enrollment-fabric-ca-server.md)
- [Setup Orderer Service](docs/05-setup-orderer-service.md)
- [Setup Peer Service](docs/06-setup-peer-service.md)
- [Setup Channel](docs/07-setup-channel.md)
- [Setup & Testing Chaincode](docs/08-setup-chaincode.md)

## Architecture
There are three organizations in this tutorial, all of which will join a channel called 'QRIS'.
- BI (Bank Indonesia), will host 3 Orderer services
- GoPay, will host 2 peer services
- DANA, will host 2 peer services

![High Level Architecture](docs/assets/images/hyperledger-highlevelinfra.drawio.png?raw=true "High Level Architecture")
