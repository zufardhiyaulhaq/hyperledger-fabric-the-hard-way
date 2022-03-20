### GoPay peer1 node
ssh to the nodes
```shell
vagrant ssh gopay-peer-1
```

copy DANA MSP organizations
```
sudo cp -R organizations/PeerOrganizations/gopay/msp /etc/hyperledger/peer/organizations/PeerOrganizations/gopay/msp
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
sudo mkdir -p /etc/hyperledger/peer/organizations/PeerOrganizations/gopay/users/administrator@enrollment.gopay.co.id/msp
```

fetch certificate admin
```
sudo fabric-ca-client enroll -d -u https://administrator@enrollment.gopay.co.id:administrator-password@10.250.251.10:7055 --tls.certfiles ${HOME}/organizations/PeerOrganizations/gopay/msp/tlsintermediatecerts/intermediate-cert.pem --csr.names C=id,O=gopay,ST=jakarta --mspdir /etc/hyperledger/peer/organizations/PeerOrganizations/gopay/users/administrator@enrollment.gopay.co.id/msp

cat <<EOF | sudo tee /etc/hyperledger/peer/organizations/PeerOrganizations/gopay/users/administrator@enrollment.gopay.co.id/msp/config.yaml
NodeOUs:
 Enable: true
 ClientOUIdentifier:
   Certificate: intermediatecerts/10-250-251-10-7055.pem
   OrganizationalUnitIdentifier: client
 PeerOUIdentifier:
   Certificate: intermediatecerts/10-250-251-10-7055.pem
   OrganizationalUnitIdentifier: peer
 AdminOUIdentifier:
   Certificate: intermediatecerts/10-250-251-10-7055.pem
   OrganizationalUnitIdentifier: admin
 OrdererOUIdentifier:
   Certificate: intermediatecerts/10-250-251-10-7055.pem
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
    └── gopay
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
        │   └── peer1
        └── users
            └── administrator@enrollment.gopay.co.id
                └── msp
                    ├── IssuerPublicKey
                    ├── IssuerRevocationPublicKey
                    ├── cacerts
                    │   └── 10-250-251-10-7055.pem
                    ├── config.yaml
                    ├── intermediatecerts
                    │   └── 10-250-251-10-7055.pem
                    ├── keystore
                    │   └── f0d5293a28fb9f9e179667e5f015527e34fa21c051f4cb949175a65415ac7ea3_sk
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
CORE_PEER_LOCALMSPID=gopay CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/peer/organizations/PeerOrganizations/gopay/users/administrator@enrollment.gopay.co.id/msp peer channel join -b genesis.block -o 10.250.250.20:7050 --tls --cafile /etc/hyperledger/peer/organizations/OrdererOrganizations/bi/msp/tlsintermediatecerts/intermediate-cert.pem
```

you should see responses like this
```json
2022-03-20 11:21:41.074 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2022-03-20 11:21:41.099 UTC 0002 INFO [channelCmd] executeJoin -> Successfully submitted proposal to join channel
```

check peers connected to channels
```
CORE_PEER_LOCALMSPID=gopay CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/peer/organizations/PeerOrganizations/gopay/users/administrator@enrollment.gopay.co.id/msp peer channel list -o 10.250.250.20:7050 --tls --cafile /etc/hyperledger/peer/organizations/OrdererOrganizations/bi/msp/tlsintermediatecerts/intermediate-cert.pem

2022-03-20 11:33:07.026 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
Channels peers has joined:
qris
```

check qris channel information
```
CORE_PEER_LOCALMSPID=gopay CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/peer/organizations/PeerOrganizations/gopay/users/administrator@enrollment.gopay.co.id/msp peer channel getinfo -c qris -o 10.250.250.20:7050 --tls --cafile /etc/hyperledger/peer/organizations/OrdererOrganizations/bi/msp/tlsintermediatecerts/intermediate-cert.pem

Blockchain info: {"height":1,"currentBlockHash":"qsZSKqF57WxZoQQxW/jpacVCa2trmT+vQKRLtc+Fq5A="}
```
