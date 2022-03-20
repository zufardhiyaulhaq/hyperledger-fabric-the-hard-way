# Setup Chaincode
if we check the policy, the transaction for QRIS need to be endorsed by majority of the peer. Let's write the chaincode and deploy in in all peers. 

## Building Chaincode
In each peer, change user to root
```
sudo su
```

In each peer, clone the repository to get the chaincode.
```shell

git clone https://github.com/zufardhiyaulhaq/hyperledger-fabric-the-hard-way
mkdir -p /etc/hyperledger/peer/chaincode/qris/
cp -R hyperledger-fabric-the-hard-way/chaincode/* /etc/hyperledger/peer/chaincode/qris/

rm -rf hyperledger-fabric-the-hard-way
```

In each peer, build the chaincode package
```shell
cd /etc/hyperledger/peer/chaincode/qris
GO111MODULE=on go mod vendor

cd /etc/hyperledger/peer
peer lifecycle chaincode package qris.tar.gz --path ./chaincode/qris --lang golang --label qris_1.0
```

## Installing Chaincode
#### GoPay
In each peer, change user to root  & change directory
```
sudo su
cd /etc/hyperledger/peer
```

In each peer, install the chaincode
```shell
export CORE_PEER_LOCALMSPID=gopay
export CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/peer/organizations/PeerOrganizations/gopay/users/administrator@enrollment.gopay.co.id/msp

peer lifecycle chaincode install qris.tar.gz

2022-03-20 11:45:49.590 UTC 0001 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Installed remotely: response:<status:200 payload:"\nIqris_1.0:2978b25484cf84dcf528c52a5c4ef894ca2dbc3558fe842b0c7fa4b33d03acaa\022\010qris_1.0" >
2022-03-20 11:45:49.590 UTC 0002 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Chaincode code package identifier: qris_1.0:2978b25484cf84dcf528c52a5c4ef894ca2dbc3558fe842b0c7fa4b33d03acaa
```

In each peer, check the chaincode
```shell
export CORE_PEER_LOCALMSPID=gopay
export CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/peer/organizations/PeerOrganizations/gopay/users/administrator@enrollment.gopay.co.id/msp

peer lifecycle chaincode queryinstalled

Installed chaincodes on peer:
Package ID: qris_1.0:2978b25484cf84dcf528c52a5c4ef894ca2dbc3558fe842b0c7fa4b33d03acaa, Label: qris_1.0
```

#### DANA
In each peer, change user to root & change directory
```
sudo su
cd /etc/hyperledger/peer
```

In each peer, install the chaincode
```shell
export CORE_PEER_LOCALMSPID=dana
export CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/peer/organizations/PeerOrganizations/dana/users/administrator@enrollment.dana.id/msp 

peer lifecycle chaincode install qris.tar.gz

2022-03-20 11:45:49.590 UTC 0001 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Installed remotely: response:<status:200 payload:"\nIqris_1.0:2978b25484cf84dcf528c52a5c4ef894ca2dbc3558fe842b0c7fa4b33d03acaa\022\010qris_1.0" >
2022-03-20 11:45:49.590 UTC 0002 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Chaincode code package identifier: qris_1.0:2978b25484cf84dcf528c52a5c4ef894ca2dbc3558fe842b0c7fa4b33d03acaa
```

In each peer, check the chaincode
```shell
export CORE_PEER_LOCALMSPID=dana
export CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/peer/organizations/PeerOrganizations/dana/users/administrator@enrollment.dana.id/msp 

peer lifecycle chaincode queryinstalled

Installed chaincodes on peer:
Package ID: qris_1.0:2978b25484cf84dcf528c52a5c4ef894ca2dbc3558fe842b0c7fa4b33d03acaa, Label: qris_1.0
```

## Approving Chaincode
*only execute in one peer node in each organizations*

#### GoPay
Let's use `gopay-peer-0`.

change user to root & change directory
```
sudo su
cd /etc/hyperledger/peer
```

```shell
export CC_PACKAGE_ID=qris_1.0:2978b25484cf84dcf528c52a5c4ef894ca2dbc3558fe842b0c7fa4b33d03acaa
export CORE_PEER_LOCALMSPID=gopay
export CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/peer/organizations/PeerOrganizations/gopay/users/administrator@enrollment.gopay.co.id/msp

peer lifecycle chaincode approveformyorg -o 10.250.250.20:7050 --channelID qris --name qris --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile /etc/hyperledger/peer/organizations/OrdererOrganizations/bi/msp/tlsintermediatecerts/intermediate-cert.pem

2022-03-20 11:54:02.512 UTC 0001 INFO [chaincodeCmd] ClientWait -> txid [0ca3d0a7a37fe19a6702053fc4057b967c0ae6e417447f291a51f2069e4f2355] committed with status (VALID) at 10.250.251.20:7051
```

check the chaincode
```shell
export CORE_PEER_LOCALMSPID=gopay
export CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/peer/organizations/PeerOrganizations/gopay/users/administrator@enrollment.gopay.co.id/msp

peer lifecycle chaincode checkcommitreadiness --channelID qris --name qris --version 1.0 --sequence 1 --tls --cafile /etc/hyperledger/peer/organizations/OrdererOrganizations/bi/msp/tlsintermediatecerts/intermediate-cert.pem

Chaincode definition for chaincode 'qris', version '1.0', sequence '1' on channel 'qris' approval status by org:
dana: false
gopay: true
```

#### DANA
Let's use `dana-peer-0`.

change user to root & change directory
```
sudo su
cd /etc/hyperledger/peer
```

```shell
export CC_PACKAGE_ID=qris_1.0:2978b25484cf84dcf528c52a5c4ef894ca2dbc3558fe842b0c7fa4b33d03acaa
export CORE_PEER_LOCALMSPID=dana
export CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/peer/organizations/PeerOrganizations/dana/users/administrator@enrollment.dana.id/msp 

peer lifecycle chaincode approveformyorg -o 10.250.250.20:7050 --channelID qris --name qris --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile /etc/hyperledger/peer/organizations/OrdererOrganizations/bi/msp/tlsintermediatecerts/intermediate-cert.pem

2022-03-20 11:56:10.114 UTC 0001 INFO [chaincodeCmd] ClientWait -> txid [0b1b3cbee4ebefa8bcbbb2cc13f2cdc9de02d3649b897087e24156fc9038140d] committed with status (VALID) at 10.250.252.20:7051
```

check the chaincode
```shell
export CORE_PEER_LOCALMSPID=dana
export CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/peer/organizations/PeerOrganizations/dana/users/administrator@enrollment.dana.id/msp 

peer lifecycle chaincode checkcommitreadiness --channelID qris --name qris --version 1.0 --sequence 1 --tls --cafile /etc/hyperledger/peer/organizations/OrdererOrganizations/bi/msp/tlsintermediatecerts/intermediate-cert.pem

Chaincode definition for chaincode 'qris', version '1.0', sequence '1' on channel 'qris' approval status by org:
dana: true
gopay: true
```

## Commit Chaincode
We need to commit the chaincode, this only need to be done from 1 peer node from 1 organizations only. Let's use GoPay peer node (gopay-peer-0) to do this commit. We need TLS Intermediate Certificate from all organizations.

change user to root & change directory
```
sudo su
cd /etc/hyperledger/peer
```

check the intermediate certificate
```shell
tree /etc/hyperledger/peer/organizations
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
```

we stil miss DANA certificate, let's get this certificate
*In the real environment, they will send the certificate to us via email or any secure communication, but let's just get from their CA server directly*

```
mkdir -p /etc/hyperledger/peer/organizations/PeerOrganizations/dana/msp/tlsintermediatecerts

ssh-keygen
sshpass -p "vagrant" ssh-copy-id -o StrictHostKeyChecking=no vagrant@10.250.252.10

scp vagrant@10.250.252.10:~/organizations/PeerOrganizations/dana/msp/tlsintermediatecerts/intermediate-cert.pem /etc/hyperledger/peer/organizations/PeerOrganizations/dana/msp/tlsintermediatecerts
```

check the intermediate certificate
```shell
tree /etc/hyperledger/peer/organizations
/etc/hyperledger/peer/organizations
├── OrdererOrganizations
│   └── bi
│       └── msp
│           └── tlsintermediatecerts
│               └── intermediate-cert.pem
└── PeerOrganizations
    ├── dana
    │   └── msp
    │       └── tlsintermediatecerts
    │           └── intermediate-cert.pem
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
        └── users
            └── administrator@enrollment.gopay.co.id
                └── msp
                    ├── cacerts
                    │   └── 10-250-251-10-7055.pem
                    ├── config.yaml
                    ├── intermediatecerts
                    │   └── 10-250-251-10-7055.pem
                    ├── IssuerPublicKey
                    ├── IssuerRevocationPublicKey
                    ├── keystore
                    │   └── 6320c6c196bc29424240e3e07968173f9df34ad279f99addae8a790379016205_sk
                    ├── signcerts
                    │   └── cert.pem
                    └── user
```

```shell
export CORE_PEER_LOCALMSPID=gopay
export CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/peer/organizations/PeerOrganizations/gopay/users/administrator@enrollment.gopay.co.id/msp
export ORDERER_CA=/etc/hyperledger/peer/organizations/OrdererOrganizations/bi/msp/tlsintermediatecerts/intermediate-cert.pem
export GOPAY_CA=/etc/hyperledger/peer/organizations/PeerOrganizations/gopay/msp/tlsintermediatecerts/intermediate-cert.pem
export DANA_CA=/etc/hyperledger/peer/organizations/PeerOrganizations/dana/msp/tlsintermediatecerts/intermediate-cert.pem

peer lifecycle chaincode commit -o 10.250.250.20:7050 --channelID qris --name qris --version 1.0 --sequence 1 --tls --cafile $ORDERER_CA --peerAddresses 10.250.251.20:7051 --tlsRootCertFiles $GOPAY_CA --peerAddresses 10.250.252.20:7051 --tlsRootCertFiles $DANA_CA

2022-03-20 12:10:07.133 UTC 0001 INFO [chaincodeCmd] ClientWait -> txid [0c41401e28869bdce074d3e75994d367849a443650783e536a4323ed69436a3d] committed with status (VALID) at 10.250.251.20:7051
2022-03-20 12:10:07.145 UTC 0002 INFO [chaincodeCmd] ClientWait -> txid [0c41401e28869bdce074d3e75994d367849a443650783e536a4323ed69436a3d] committed with status (VALID) at 10.250.252.20:7051
```

## Verify chaincode

#### Dana
```shell
export CORE_PEER_LOCALMSPID=dana
export CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/peer/organizations/PeerOrganizations/dana/users/administrator@enrollment.dana.id/msp 
peer lifecycle chaincode querycommitted --channelID qris --name qris -o 10.250.250.20:7050 --cafile /etc/hyperledger/peer/organizations/OrdererOrganizations/bi/msp/tlsintermediatecerts/intermediate-cert.pem

Committed chaincode definition for chaincode 'qris' on channel 'qris':
Version: 1.0, Sequence: 1, Endorsement Plugin: escc, Validation Plugin: vscc, Approvals: [dana: true, gopay: true]
```

#### GoPay
```shell
export CORE_PEER_LOCALMSPID=gopay
export CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/peer/organizations/PeerOrganizations/gopay/users/administrator@enrollment.gopay.co.id/msp
peer lifecycle chaincode querycommitted --channelID qris --name qris -o 10.250.250.20:7050 --cafile /etc/hyperledger/peer/organizations/OrdererOrganizations/bi/msp/tlsintermediatecerts/intermediate-cert.pem

Committed chaincode definition for chaincode 'qris' on channel 'qris':
Version: 1.0, Sequence: 1, Endorsement Plugin: escc, Validation Plugin: vscc, Approvals: [dana: true, gopay: true]
```

## Invoking Chaincode
We already install chaincode in our channel. let's test this chaincode with admin users. Let's use GoPay peer node to do this 

*In the real environment, they will send the certificate to us via email or any secure communication, but let's just get from their CA server directly*

```shell
export CORE_PEER_LOCALMSPID=gopay
export CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/peer/organizations/PeerOrganizations/gopay/users/administrator@enrollment.gopay.co.id/msp
export ORDERER_CA=/etc/hyperledger/peer/organizations/OrdererOrganizations/bi/msp/tlsintermediatecerts/intermediate-cert.pem
export GOPAY_CA=/etc/hyperledger/peer/organizations/PeerOrganizations/gopay/msp/tlsintermediatecerts/intermediate-cert.pem
export DANA_CA=/etc/hyperledger/peer/organizations/PeerOrganizations/dana/msp/tlsintermediatecerts/intermediate-cert.pem

# invoke InitLedger function to create initial assets
peer chaincode invoke -o 10.250.250.20:7050 --channelID qris --name qris --tls --cafile $ORDERER_CA --peerAddresses 10.250.251.20:7051 --tlsRootCertFiles $GOPAY_CA --peerAddresses 10.250.252.20:7051 --tlsRootCertFiles $DANA_CA -c '{"function":"InitLedger","Args":[]}'

2022-03-20 18:12:22.660 UTC 0001 INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200

# check all assets created
peer chaincode invoke -o 10.250.250.20:7050 --channelID qris --name qris --tls --cafile $ORDERER_CA --peerAddresses 10.250.251.20:7051 --tlsRootCertFiles $GOPAY_CA --peerAddresses 10.250.252.20:7051 --tlsRootCertFiles $DANA_CA -c '{"Args":["GetAllAssets"]}'

2022-03-20 18:13:16.749 UTC 0001 INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200 payload:"[{\"ID\":\"8b8c7df1-48f8-4aba-9990-57a99d70e010\",\"Owner\":\"Bank Indonesia\",\"Producer\":\"Bank Indonesia\",\"Value\":0}]"

# create new assets
peer chaincode invoke -o 10.250.250.20:7050 --channelID qris --name qris --tls --cafile $ORDERER_CA --peerAddresses 10.250.251.20:7051 --tlsRootCertFiles $GOPAY_CA --peerAddresses 10.250.252.20:7051 --tlsRootCertFiles $DANA_CA -c '{"function":"CreateAsset","Args":["8b8c7df1-48f8-4aba-9990-57a99d70e011", "Zufar Dhiyaulhaq", "GoPay", "50"]}'

# check all assets created
peer chaincode invoke -o 10.250.250.20:7050 --channelID qris --name qris --tls --cafile $ORDERER_CA --peerAddresses 10.250.251.20:7051 --tlsRootCertFiles $GOPAY_CA --peerAddresses 10.250.252.20:7051 --tlsRootCertFiles $DANA_CA -c '{"Args":["GetAllAssets"]}'

2022-03-20 18:35:26.961 UTC 0001 INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200 payload:"[{\"ID\":\"8b8c7df1-48f8-4aba-9990-57a99d70e010\",\"Owner\":\"Bank Indonesia\",\"Producer\":\"Bank Indonesia\",\"Value\":0},{\"ID\":\"8b8c7df1-48f8-4aba-9990-57a99d70e011\",\"Owner\":\"Zufar Dhiyaulhaq\",\"Producer\":\"GoPay\",\"Value\":50}]"

# update assets
peer chaincode invoke -o 10.250.250.20:7050 --channelID qris --name qris --tls --cafile $ORDERER_CA --peerAddresses 10.250.251.20:7051 --tlsRootCertFiles $GOPAY_CA --peerAddresses 10.250.252.20:7051 --tlsRootCertFiles $DANA_CA -c '{"function":"UpdateAsset","Args":["8b8c7df1-48f8-4aba-9990-57a99d70e011", "Zufar Dhiyaulhaq", "GoPay", "500000000"]}'

# check specific assets 
peer chaincode invoke -o 10.250.250.20:7050 --channelID qris --name qris --tls --cafile $ORDERER_CA --peerAddresses 10.250.251.20:7051 --tlsRootCertFiles $GOPAY_CA --peerAddresses 10.250.252.20:7051 --tlsRootCertFiles $DANA_CA -c '{"function":"ReadAsset","Args":["8b8c7df1-48f8-4aba-9990-57a99d70e011"]}'

2022-03-20 18:38:50.794 UTC 0001 INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200 payload:"{\"ID\":\"8b8c7df1-48f8-4aba-9990-57a99d70e011\",\"Owner\":\"Zufar Dhiyaulhaq\",\"Producer\":\"GoPay\",\"Value\":500000000}"
```
