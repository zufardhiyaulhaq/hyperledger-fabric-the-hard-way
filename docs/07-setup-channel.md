# Setup channel
Here is the tricky part, to setup a channel, we need to generate channel genesis block. To generate this channel genesis block, we need to get this configuration for all organizations:
- Enrollment Root CA certificate
- Enrollment Intermediate CA certificate
- TLS Root CA certificate
- TLS Intermediate CA certificate

and we need to get TLS certificate for all orderer nodes. Let's use bi-ca-server-0 to gather all of the requirements.

*Remember! in real environment, each organization must collaborate each other to share their certificate*

### Generate genesis block
ssh to the nodes
```shell
vagrant ssh bi-ca-server-0
```

Download binary
```
wget https://github.com/hyperledger/fabric/releases/download/v2.4.3/hyperledger-fabric-linux-amd64-2.4.3.tar.gz
tar xzvf hyperledger-fabric-linux-amd64-2.4.3.tar.gz
sudo cp bin/* /usr/local/bin/
rm -rf bin/
rm -rf config/
rm -rf hyperledger-fabric-linux-amd64-2.4.3.tar.gz
```

create directory for each organizations
```shell
mkdir -p organizations/bi/msp/cacerts
mkdir -p organizations/bi/msp/intermediatecerts
mkdir -p organizations/bi/msp/tlscacerts
mkdir -p organizations/bi/msp/tlsintermediatecerts
mkdir -p organizations/bi/msp/admincerts

mkdir -p organizations/bi/orderer/orderer0/tls
mkdir -p organizations/bi/orderer/orderer1/tls
mkdir -p organizations/bi/orderer/orderer2/tls

mkdir -p organizations/gopay/msp/cacerts
mkdir -p organizations/gopay/msp/intermediatecerts
mkdir -p organizations/gopay/msp/tlscacerts
mkdir -p organizations/gopay/msp/tlsintermediatecerts
mkdir -p organizations/gopay/msp/admincerts

mkdir -p organizations/dana/msp/cacerts
mkdir -p organizations/dana/msp/intermediatecerts
mkdir -p organizations/dana/msp/tlscacerts
mkdir -p organizations/dana/msp/tlsintermediatecerts
mkdir -p organizations/dana/msp/admincerts
```

get certificate BI
```shell
mkdir -p tmp/bi/tls
mkdir -p tmp/bi/enrollment
mkdir -p tmp/bi/enrollment-admin

cp certificates/tls/intermediate/fullchain.crt tmp/bi

fabric-ca-client getcainfo -d -u https://admin@bi0:adminpasswd@10.250.250.10:7054 --tls.certfiles ${HOME}/tmp/bi/fullchain.crt --mspdir ${HOME}/tmp/bi/tls
fabric-ca-client getcainfo -d -u https://admin@bi0:adminpasswd@10.250.250.10:7055 --tls.certfiles ${HOME}/tmp/bi/fullchain.crt --mspdir ${HOME}/tmp/bi/enrollment
fabric-ca-client enroll -d -u https://administrator@bi:administrator-bi-password@10.250.250.10:7055 --tls.certfiles ${HOME}/tmp/bi/fullchain.crt --csr.names C=id,O=bi,ST=jakarta --mspdir ${HOME}/tmp/bi/enrollment-admin

cp tmp/bi/enrollment-admin/signcerts/cert.pem organizations/bi/msp/admincerts

cp tmp/bi/enrollment/cacerts/10-250-250-10-7055.pem organizations/bi/msp/cacerts/root-ca.pem
cp tmp/bi/enrollment/intermediatecerts/10-250-250-10-7055.pem organizations/bi/msp/intermediatecerts/intermediate-ca.pem

cp tmp/bi/tls/cacerts/10-250-250-10-7054.pem organizations/bi/msp/tlscacerts/root-ca.pem
cp tmp/bi/tls/intermediatecerts/10-250-250-10-7054.pem organizations/bi/msp/tlsintermediatecerts/intermediate-ca.pem

rm -rf tmp
```

get certificate GoPay. 
*In the real environment, they will send the certificate to us via email or any secure communication, but let's just get from their CA server directly*
```shell
sshpass -p "vagrant" ssh-copy-id -o StrictHostKeyChecking=no vagrant@10.250.251.10

mkdir -p tmp/gopay/tls
mkdir -p tmp/gopay/enrollment
mkdir -p tmp/gopay/enrollment-admin

scp vagrant@10.250.251.10:~/certificates/tls/intermediate/fullchain.crt tmp/gopay/

fabric-ca-client getcainfo -d -u https://admin@gopay0:adminpasswd@10.250.251.10:7054 --tls.certfiles ${HOME}/tmp/gopay/fullchain.crt --mspdir ${HOME}/tmp/gopay/tls
fabric-ca-client getcainfo -d -u https://admin@gopay0:adminpasswd@10.250.251.10:7055 --tls.certfiles ${HOME}/tmp/gopay/fullchain.crt --mspdir ${HOME}/tmp/gopay/enrollment
fabric-ca-client enroll -d -u https://administrator@gopay:administrator-gopay-password@10.250.251.10:7055 --tls.certfiles ${HOME}/tmp/gopay/fullchain.crt --csr.names C=id,O=gopay,ST=jakarta --mspdir ${HOME}/tmp/gopay/enrollment-admin

cp tmp/gopay/enrollment-admin/signcerts/cert.pem organizations/gopay/msp/admincerts

cp tmp/gopay/enrollment/cacerts/10-250-251-10-7055.pem organizations/gopay/msp/cacerts/root-ca.pem
cp tmp/gopay/enrollment/intermediatecerts/10-250-251-10-7055.pem organizations/gopay/msp/intermediatecerts/intermediate-ca.pem

cp tmp/gopay/tls/cacerts/10-250-251-10-7054.pem organizations/gopay/msp/tlscacerts/root-ca.pem
cp tmp/gopay/tls/intermediatecerts/10-250-251-10-7054.pem organizations/gopay/msp/tlsintermediatecerts/intermediate-ca.pem

rm -rf tmp
```

get certificate Dana. 
*In the real environment, they will send the certificate to us via email or any secure communication, but let's just get from their CA server directly*
```shell
sshpass -p "vagrant" ssh-copy-id -o StrictHostKeyChecking=no vagrant@10.250.252.10

mkdir -p tmp/dana/tls
mkdir -p tmp/dana/enrollment
mkdir -p tmp/dana/enrollment-admin

scp vagrant@10.250.252.10:~/certificates/tls/intermediate/fullchain.crt tmp/dana/

fabric-ca-client getcainfo -d -u https://admin@dana0:adminpasswd@10.250.252.10:7054 --tls.certfiles ${HOME}/tmp/dana/fullchain.crt --mspdir ${HOME}/tmp/dana/tls
fabric-ca-client getcainfo -d -u https://admin@dana0:adminpasswd@10.250.252.10:7055 --tls.certfiles ${HOME}/tmp/dana/fullchain.crt --mspdir ${HOME}/tmp/dana/enrollment
fabric-ca-client enroll -d -u https://administrator@dana:administrator-dana-password@10.250.252.10:7055 --tls.certfiles ${HOME}/tmp/dana/fullchain.crt --csr.names C=id,O=dana,ST=jakarta --mspdir ${HOME}/tmp/dana/enrollment-admin

cp tmp/dana/enrollment-admin/signcerts/cert.pem organizations/dana/msp/admincerts

cp tmp/dana/enrollment/cacerts/10-250-252-10-7055.pem organizations/dana/msp/cacerts/root-ca.pem
cp tmp/dana/enrollment/intermediatecerts/10-250-252-10-7055.pem organizations/dana/msp/intermediatecerts/intermediate-ca.pem

cp tmp/dana/tls/cacerts/10-250-252-10-7054.pem organizations/dana/msp/tlscacerts/root-ca.pem
cp tmp/dana/tls/intermediatecerts/10-250-252-10-7054.pem organizations/dana/msp/tlsintermediatecerts/intermediate-ca.pem

rm -rf tmp
```

We already create TLS certificate for each orderer service, it's should located in `~/certificates/orderer/tls/signcerts/cert.pem`
```shell
scp vagrant@10.250.250.20:~/certificates/orderer/tls/signcerts/cert.pem organizations/bi/orderer/orderer0/tls/
scp vagrant@10.250.250.21:~/certificates/orderer/tls/signcerts/cert.pem organizations/bi/orderer/orderer1/tls/
scp vagrant@10.250.250.22:~/certificates/orderer/tls/signcerts/cert.pem organizations/bi/orderer/orderer2/tls/
```

create configuration
```shell
cat <<EOF | tee organizations/configtx.yaml
Organizations:
  - &BankIndonesiaOrg
      Name: bi
      ID: bi
      MSPDir: bi/msp
      Policies:
        Readers:
          Type: Signature
          Rule: "OR('bi.member')"
        Writers:
          Type: Signature
          Rule: "OR('bi.member')"
        Admins:
          Type: Signature
          Rule: "OR('bi.admin')"
  - &GoPayOrg
      Name: gopay
      ID: gopay
      MSPDir: gopay/msp
      Policies:
        Readers:
          Type: Signature
          Rule: "OR('gopay.admin', 'gopay.peer', 'gopay.client')"
        Writers:
          Type: Signature
          Rule: "OR('gopay.admin', 'gopay.client')"
        Admins:
          Type: Signature
          Rule: "OR('gopay.admin')"
        Endorsement:
          Type: Signature
          Rule: "OR('gopay.peer')"
      AnchorPeers:
          - Host: 10.250.251.20
            Port: 7051
  - &DANAOrg
      Name: dana
      ID: dana
      MSPDir: dana/msp
      Policies:
        Readers:
          Type: Signature
          Rule: "OR('dana.admin', 'dana.peer', 'dana.client')"
        Writers:
          Type: Signature
          Rule: "OR('dana.admin', 'dana.client')"
        Admins:
          Type: Signature
          Rule: "OR('dana.admin')"
        Endorsement:
          Type: Signature
          Rule: "OR('dana.peer')"
      AnchorPeers:
          - Host: 10.250.252.20
            Port: 7051

Capabilities:
    Channel: &ChannelCapabilities
        V2_0: true
    Orderer: &OrdererCapabilities
        V2_0: true
    Application: &ApplicationCapabilities
        V2_0: true
  
Channel: &QRISChannel
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"
    Capabilities:
        <<: *ChannelCapabilities

Orderer: &QRISOrderer
  BatchTimeout: 2s
  BatchSize:
    MaxMessageCount: 500
    AbsoluteMaxBytes: 10 MB
    PreferredMaxBytes: 2 MB
  MaxChannels: 0
  OrdererType: etcdraft
  EtcdRaft:
    Consenters:
    - Host: 10.250.250.20
      Port: 7050
      ClientTLSCert: bi/orderer/orderer0/tls/cert.pem
      ServerTLSCert: bi/orderer/orderer0/tls/cert.pem
    - Host: 10.250.250.21
      Port: 7050
      ClientTLSCert: bi/orderer/orderer1/tls/cert.pem
      ServerTLSCert: bi/orderer/orderer1/tls/cert.pem
    - Host: 10.250.250.22
      Port: 7050
      ClientTLSCert: bi/orderer/orderer2/tls/cert.pem
      ServerTLSCert: bi/orderer/orderer2/tls/cert.pem
  Addresses:
    - 10.250.250.20:7050
    - 10.250.250.21:7050
    - 10.250.250.22:7050
  Policies:
    Readers:
      Type: ImplicitMeta
      Rule: "ANY Readers"
    Writers:
      Type: ImplicitMeta
      Rule: "ANY Writers"
    Admins:
      Type: ImplicitMeta
      Rule: "MAJORITY Admins"
    BlockValidation:
      Type: ImplicitMeta
      Rule: "ANY Writers"
  Capabilities:
    <<: *OrdererCapabilities

Application: &QRISApplication
  Policies:
      Readers:
          Type: ImplicitMeta
          Rule: "ANY Readers"
      Writers:
          Type: ImplicitMeta
          Rule: "ANY Writers"
      Admins:
          Type: ImplicitMeta
          Rule: "MAJORITY Admins"
      LifecycleEndorsement:
          Type: ImplicitMeta
          Rule: "MAJORITY Endorsement"
      Endorsement:
          Type: ImplicitMeta
          Rule: "MAJORITY Endorsement"
  Capabilities:
    <<: *ApplicationCapabilities

Profiles:
  QRIS:
    <<: *QRISChannel
    Orderer:
      <<: *QRISOrderer
      Organizations:
        - <<: *BankIndonesiaOrg
      
    Application:
      <<: *QRISApplication
      Organizations:
      - <<: *GoPayOrg
      - <<: *DANAOrg
EOF
```

setup NodeOU for each organizations
```
cat <<EOF | tee organizations/dana/msp/config.yaml
NodeOUs:
 Enable: true
 ClientOUIdentifier:
   Certificate: intermediatecerts/intermediate-ca.pem
   OrganizationalUnitIdentifier: client
 PeerOUIdentifier:
   Certificate: intermediatecerts/intermediate-ca.pem
   OrganizationalUnitIdentifier: peer
 AdminOUIdentifier:
   Certificate: intermediatecerts/intermediate-ca.pem
   OrganizationalUnitIdentifier: admin
 OrdererOUIdentifier:
   Certificate: intermediatecerts/intermediate-ca.pem
   OrganizationalUnitIdentifier: orderer
EOF

cat <<EOF | tee organizations/gopay/msp/config.yaml
NodeOUs:
 Enable: true
 ClientOUIdentifier:
   Certificate: intermediatecerts/intermediate-ca.pem
   OrganizationalUnitIdentifier: client
 PeerOUIdentifier:
   Certificate: intermediatecerts/intermediate-ca.pem
   OrganizationalUnitIdentifier: peer
 AdminOUIdentifier:
   Certificate: intermediatecerts/intermediate-ca.pem
   OrganizationalUnitIdentifier: admin
 OrdererOUIdentifier:
   Certificate: intermediatecerts/intermediate-ca.pem
   OrganizationalUnitIdentifier: orderer
EOF

cat <<EOF | tee organizations/bi/msp/config.yaml
NodeOUs:
 Enable: true
 ClientOUIdentifier:
   Certificate: intermediatecerts/intermediate-ca.pem
   OrganizationalUnitIdentifier: client
 PeerOUIdentifier:
   Certificate: intermediatecerts/intermediate-ca.pem
   OrganizationalUnitIdentifier: peer
 AdminOUIdentifier:
   Certificate: intermediatecerts/intermediate-ca.pem
   OrganizationalUnitIdentifier: admin
 OrdererOUIdentifier:
   Certificate: intermediatecerts/intermediate-ca.pem
   OrganizationalUnitIdentifier: orderer
EOF
```

generate genesis block
```
cd organizations
configtxgen -profile QRIS -outputBlock genesis_block.pb -channelID qris
```

copy this to each orderer service
```
scp genesis_block.pb vagrant@10.250.250.20:~/
scp genesis_block.pb vagrant@10.250.250.21:~/
scp genesis_block.pb vagrant@10.250.250.22:~/
```

### Orderer services
in each orderer services, run this command to join orderer to the channel
```
osnadmin channel join --channelID qris  --config-block genesis_block.pb -o 127.0.0.1:12443
```

you should see responses like this
```json
Status: 200
{
	"systemChannel": null,
	"channels": [
		{
			"name": "qris",
			"url": "/participation/v1/channels/qris"
		}
	]
}
```

### Peer Services
fetch Orderer TLS intermediate certificate (BI TLS intermediate certificate) to each of the peer nodes

*Remember! in real environment, each organization must collaborate each other to share their certificate*

copy orderer CA to each peer nodes
```
ssh-keygen
sshpass -p "vagrant" ssh-copy-id -o StrictHostKeyChecking=no vagrant@10.250.250.10

sudo mkdir -p /etc/peer/orderer
scp vagrant@10.250.250.10:~/organizations/bi/msp/tlsintermediatecerts/intermediate-ca.pem ~/
sudo mv intermediate-ca.pem /etc/peer/orderer/
```

fetch the genesis.block from orderer
```
sudo su
cd /etc/peer

peer channel fetch newest genesis.block -c qris -o 10.250.250.20:7050 --tls --cafile /etc/peer/orderer/intermediate-ca.pem
```

join peers to the channel
```
#execute in gopay peers node only
CORE_PEER_LOCALMSPID=gopay CORE_PEER_MSPCONFIGPATH=/etc/peer/users/admin peer channel join -b genesis.block -o 10.250.250.20:7050 --tls --cafile /etc/peer/orderer/intermediate-ca.pem

#execute in dana peers node only
CORE_PEER_LOCALMSPID=dana CORE_PEER_MSPCONFIGPATH=/etc/peer/users/admin peer channel join -b genesis.block -o 10.250.250.20:7050 --tls --cafile /etc/peer/orderer/intermediate-ca.pem
```

check peers connected to channels
```
#execute in gopay peers node only
CORE_PEER_LOCALMSPID=gopay CORE_PEER_MSPCONFIGPATH=/etc/peer/users/admin peer channel list -o 10.250.250.20:7050 --tls --cafile /etc/peer/orderer/intermediate-ca.pem

#execute in dana peers node only
CORE_PEER_LOCALMSPID=dana CORE_PEER_MSPCONFIGPATH=/etc/peer/users/admin peer channel list -o 10.250.250.20:7050 --tls --cafile /etc/peer/orderer/intermediate-ca.pem

2022-03-13 08:39:50.824 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
Channels peers has joined:
qris
```

check qris channel information
```
#execute in gopay peers node only
CORE_PEER_LOCALMSPID=gopay CORE_PEER_MSPCONFIGPATH=/etc/peer/users/admin peer channel getinfo -c qris -o 10.250.250.20:7050 --tls --cafile /etc/peer/orderer/intermediate-ca.pem

#execute in dana peers node only
CORE_PEER_LOCALMSPID=dana CORE_PEER_MSPCONFIGPATH=/etc/peer/users/admin peer channel getinfo -c qris -o 10.250.250.20:7050 --tls --cafile /etc/peer/orderer/intermediate-ca.pem

2022-03-13 08:31:34.867 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
Blockchain info: {"height":1,"currentBlockHash":"vq3Xj0BdB3pInatCfIcoJTl8eznlPwdJAYfqDgJSgiA="}
```
