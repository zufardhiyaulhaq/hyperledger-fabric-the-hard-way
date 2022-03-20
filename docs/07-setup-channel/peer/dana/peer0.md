### DANA peer0 node
ssh to the nodes
```shell
vagrant ssh dana-peer-0
```

copy DANA MSP organizations
```
sudo cp -R organizations/PeerOrganizations/dana/msp /etc/hyperledger/peer/organizations/PeerOrganizations/dana/msp
```

create directory for orderer CA
```
sudo mkdir -p /etc/hyperledger/peer/organizations/OrdererOrganizations/bi/msp/tlsintermediatecerts
```

fetch Orderer TLS intermediate certificate (BI TLS intermediate certificate) to each of the peer nodes
*Remember! in real environment, each organizations must collaborate each other to share their certificate*
```
ssh-keygen
sshpass -p "vagrant" ssh-copy-id -o StrictHostKeyChecking=no vagrant@10.250.250.10

scp vagrant@10.250.250.10:~/organizations/OrdererOrganizations/bi/msp/tlsintermediatecerts/intermediate-cert.pem ~/
sudo mv ~/intermediate-cert.pem /etc/hyperledger/peer/organizations/OrdererOrganizations/bi/msp/tlsintermediatecerts/
```

create directory for admin
```
sudo mkdir -p /etc/hyperledger/peer/organizations/PeerOrganizations/dana/users/administrator@enrollment.dana.id/msp
```

fetch certificate admin
```
sudo fabric-ca-client enroll -d -u https://administrator@enrollment.dana.id:administrator-password@10.250.252.10:7055 --tls.certfiles ${HOME}/organizations/PeerOrganizations/dana/msp/tlsintermediatecerts/intermediate-cert.pem --csr.names C=id,O=dana,ST=jakarta --mspdir /etc/hyperledger/peer/organizations/PeerOrganizations/dana/users/administrator@enrollment.dana.id/msp

cat <<EOF | sudo tee /etc/hyperledger/peer/organizations/PeerOrganizations/dana/users/administrator@enrollment.dana.id/msp/config.yaml
NodeOUs:
 Enable: true
 ClientOUIdentifier:
   Certificate: intermediatecerts/10-250-252-10-7055.pem
   OrganizationalUnitIdentifier: client
 PeerOUIdentifier:
   Certificate: intermediatecerts/10-250-252-10-7055.pem
   OrganizationalUnitIdentifier: peer
 AdminOUIdentifier:
   Certificate: intermediatecerts/10-250-252-10-7055.pem
   OrganizationalUnitIdentifier: admin
 OrdererOUIdentifier:
   Certificate: intermediatecerts/10-250-252-10-7055.pem
   OrganizationalUnitIdentifier: orderer
EOF
```

check the certificate
```
sudo tree /etc/hyperledger/peer/organizations
/etc/hyperledger/peer/organizations
├── OrdererOrganizations
│   └── bi
│       └── msp
│           └── tlsintermediatecerts
│               └── intermediate-cert.pem
└── PeerOrganizations
    └── dana
        ├── msp
        │   ├── cacerts
        │   │   └── root-cert.pem
        │   ├── intermediatecerts
        │   │   └── intermediate-cert.pem
        │   ├── tlscacerts
        │   │   └── root-cert.pem
        │   └── tlsintermediatecerts
        │       └── intermediate-cert.pem
        ├── peers
        │   └── peer0
        └── users
            └── administrator@enrollment.dana.id
                └── msp
                    ├── IssuerPublicKey
                    ├── IssuerRevocationPublicKey
                    ├── cacerts
                    │   └── 10-250-252-10-7055.pem
                    ├── config.yaml
                    ├── intermediatecerts
                    │   └── 10-250-252-10-7055.pem
                    ├── keystore
                    │   └── d6395dc52e781de250a2c24fa83432b88c688d6ddefe751f23597954b4ece665_sk
                    ├── signcerts
                    │   └── cert.pem
                    └── user
```

fetch the genesis.block from orderer
```
sudo su
cd /etc/hyperledger/peer

peer channel fetch newest genesis.block -c qris -o 10.250.250.20:7050 --tls --cafile /etc/hyperledger/peer/organizations/OrdererOrganizations/bi/msp/tlsintermediatecerts/intermediate-cert.pem

2022-03-20 11:19:41.639 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2022-03-20 11:19:41.641 UTC 0002 INFO [cli.common] readBlock -> Received block: 0
```

join peers to the channel
```
CORE_PEER_LOCALMSPID=dana CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/peer/organizations/PeerOrganizations/dana/users/administrator@enrollment.dana.id/msp peer channel join -b genesis.block -o 10.250.250.20:7050 --tls --cafile /etc/hyperledger/peer/organizations/OrdererOrganizations/bi/msp/tlsintermediatecerts/intermediate-cert.pem
```

you should see responses like this
```json
2022-03-20 11:21:41.074 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2022-03-20 11:21:41.099 UTC 0002 INFO [channelCmd] executeJoin -> Successfully submitted proposal to join channel
```

check peers connected to channels
```
CORE_PEER_LOCALMSPID=dana CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/peer/organizations/PeerOrganizations/dana/users/administrator@enrollment.dana.id/msp peer channel list -o 10.250.250.20:7050 --tls --cafile /etc/hyperledger/peer/organizations/OrdererOrganizations/bi/msp/tlsintermediatecerts/intermediate-cert.pem

2022-03-20 11:33:07.026 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
Channels peers has joined:
qris
```

check qris channel information
```
CORE_PEER_LOCALMSPID=dana CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/peer/organizations/PeerOrganizations/dana/users/administrator@enrollment.dana.id/msp peer channel getinfo -c qris -o 10.250.250.20:7050 --tls --cafile /etc/hyperledger/peer/organizations/OrdererOrganizations/bi/msp/tlsintermediatecerts/intermediate-cert.pem

Blockchain info: {"height":1,"currentBlockHash":"qsZSKqF57WxZoQQxW/jpacVCa2trmT+vQKRLtc+Fq5A="}
```
