# Setup Chaincode
if we check the policy, the transaction for QRIS need to be endorsed by majority of the peer. Let's write the chaincode and deploy in in all peers. 

In each peer, install golang
```shell
sudo su

bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
source /root/.gvm/scripts/gvm

sudo apt-get install binutils bison gcc make

gvm install go1.4 -B
gvm use go1.4
export GOROOT_BOOTSTRAP=$GOROOT

gvm install go1.16.3
gvm use go1.16.3
```

In each peer, clone the repository to get the chaincode.
```shell
sudo su

git clone https://github.com/zufardhiyaulhaq/hyperledger-fabric-the-hard-way
mkdir -p /etc/peer/chaincode/qris/
cp -R hyperledger-fabric-the-hard-way/chaincode/* /etc/peer/chaincode/qris/

rm -rf hyperledger-fabric-the-hard-way
```

In each peer, build the chaincode package
```shell
sudo su

cd /etc/peer/chaincode/qris
GO111MODULE=on go mod vendor

cd /etc/peer
peer lifecycle chaincode package qris.tar.gz --path ./chaincode/qris --lang golang --label qris_1.0
```

In each peer, install the chaincode
```shell
#execute in gopay peers node only
CORE_PEER_LOCALMSPID=gopay CORE_PEER_MSPCONFIGPATH=/etc/peer/users/admin peer lifecycle chaincode install qris.tar.gz

#execute in dana peers node only
CORE_PEER_LOCALMSPID=dana CORE_PEER_MSPCONFIGPATH=/etc/peer/users/admin peer lifecycle chaincode install qris.tar.gz

2022-03-13 16:03:00.690 UTC 0001 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Installed remotely: response:<status:200 payload:"\nIqris_1.0:490871442eec37bc5d797c811155a74c7c551224bd386ccca18bb5bd0fefce15\022\010qris_1.0" >
2022-03-13 16:03:00.690 UTC 0002 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Chaincode code package identifier: qris_1.0:490871442eec37bc5d797c811155a74c7c551224bd386ccca18bb5bd0fefce15
```

check the chaincode
```shell
#execute in gopay peers node only
CORE_PEER_LOCALMSPID=gopay CORE_PEER_MSPCONFIGPATH=/etc/peer/users/admin peer lifecycle chaincode queryinstalled

#execute in dana peers node only
CORE_PEER_LOCALMSPID=dana CORE_PEER_MSPCONFIGPATH=/etc/peer/users/admin peer lifecycle chaincode queryinstalled

Installed chaincodes on peer:
Package ID: qris_1.0:490871442eec37bc5d797c811155a74c7c551224bd386ccca18bb5bd0fefce15, Label: qris_1.0
```

approve chaincode definition

*only execute in one peer node in each organizations*

##### gopay
```shell
export CC_PACKAGE_ID=qris_1.0:490871442eec37bc5d797c811155a74c7c551224bd386ccca18bb5bd0fefce15
export CORE_PEER_LOCALMSPID=gopay
export CORE_PEER_MSPCONFIGPATH=/etc/peer/users/admin

peer lifecycle chaincode approveformyorg -o 10.250.250.20:7050 --channelID qris --name qris --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile /etc/peer/orderer/intermediate-ca.pem

2022-03-13 17:07:35.728 UTC 0001 INFO [chaincodeCmd] ClientWait -> txid [49e8c208e66d8c48f3c81a907162ce59c154df04eeb54f70f4750e9926697f52] committed with status (VALID) at 10.250.251.20:7051

peer lifecycle chaincode checkcommitreadiness --channelID qris --name qris --version 1.0 --sequence 1 --tls --cafile /etc/peer/orderer/intermediate-ca.pem --output json

{
	"approvals": {
		"dana": false,
		"gopay": true
	}
}
```

##### dana
```shell
export CC_PACKAGE_ID=qris_1.0:490871442eec37bc5d797c811155a74c7c551224bd386ccca18bb5bd0fefce15
export CORE_PEER_LOCALMSPID=dana
export CORE_PEER_MSPCONFIGPATH=/etc/peer/users/admin

peer lifecycle chaincode approveformyorg -o 10.250.250.20:7050 --channelID qris --name qris --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile /etc/peer/orderer/intermediate-ca.pem

2022-03-13 17:11:39.410 UTC 0001 INFO [chaincodeCmd] ClientWait -> txid [24a51fe90999fbf49ff8c609296470766499bd493ed117a904ad0ee4dbac6d8f] committed with status (VALID) at 10.250.252.20:7051

peer lifecycle chaincode checkcommitreadiness --channelID qris --name basic --version 1.0 --sequence 1 --tls --cafile /etc/peer/orderer/intermediate-ca.pem --output json

{
	"approvals": {
		"dana": true,
		"gopay": true
	}
}
```

We need to commit the chaincode, this only need to be done from 1 peer node from 1 organizations only. Let's use GoPay peer node to do this commit.

*In the real environment, they will send the certificate to us via email or any secure communication, but let's just get from their CA server directly*
```shell
export CORE_PEER_LOCALMSPID=gopay
export CORE_PEER_MSPCONFIGPATH=/etc/peer/users/admin
export ORDERER_CA=/etc/peer/orderer/intermediate-ca.pem
export GOPAY_CA=/etc/peer/msp/tlsintermediatecerts/intermediate-ca.pem
export DANA_CA=/etc/peer/organizations/dana/msp/tlsintermediatecerts/intermediate-ca.pem

ssh-keygen
sshpass -p "vagrant" ssh-copy-id -o StrictHostKeyChecking=no vagrant@10.250.252.10

mkdir -p /etc/peer/organizations/dana/msp/tlsintermediatecerts/
scp vagrant@10.250.252.10:~/certificates/tls/intermediate/fullchain.crt /etc/peer/organizations/dana/msp/tlsintermediatecerts/intermediate-ca.pem

peer lifecycle chaincode commit -o 10.250.250.20:7050 --channelID qris --name qris --version 1.0 --sequence 1 --tls --cafile $ORDERER_CA --peerAddresses 10.250.251.20:7051 --tlsRootCertFiles $ORG_CA --peerAddresses 10.250.252.20:7051 --tlsRootCertFiles $DANA_CA

2022-03-13 17:46:02.932 UTC 0001 INFO [chaincodeCmd] ClientWait -> txid [3ff6eaa8a9e621180ddefd83ae686a80f53e3ee12c9135808763787efa6325e6] committed with status (VALID) at 10.250.251.20:7051
2022-03-13 17:46:02.944 UTC 0002 INFO [chaincodeCmd] ClientWait -> txid [3ff6eaa8a9e621180ddefd83ae686a80f53e3ee12c9135808763787efa6325e6] committed with status (VALID) at 10.250.252.20:7051
```

Verify chaincode

##### Dana
```shell
export CORE_PEER_LOCALMSPID=dana
export CORE_PEER_MSPCONFIGPATH=/etc/peer/users/admin
peer lifecycle chaincode querycommitted --channelID qris --name qris -o 10.250.250.20:7050 --cafile /etc/peer/orderer/intermediate-ca.pem

Committed chaincode definition for chaincode 'qris' on channel 'qris':
Version: 1.0, Sequence: 1, Endorsement Plugin: escc, Validation Plugin: vscc, Approvals: [dana: true, gopay: true]
```

##### GoPay
```shell
export CORE_PEER_LOCALMSPID=gopay
export CORE_PEER_MSPCONFIGPATH=/etc/peer/users/admin
peer lifecycle chaincode querycommitted --channelID qris --name qris -o 10.250.250.20:7050 --cafile /etc/peer/orderer/intermediate-ca.pem

Committed chaincode definition for chaincode 'qris' on channel 'qris':
Version: 1.0, Sequence: 1, Endorsement Plugin: escc, Validation Plugin: vscc, Approvals: [dana: true, gopay: true]
```

We already install chaincode in our channel. let's test this chaincode with admin users. Let's use GoPay peer node to do this 

*In the real environment, they will send the certificate to us via email or any secure communication, but let's just get from their CA server directly*

```shell
export CORE_PEER_LOCALMSPID=gopay
export CORE_PEER_MSPCONFIGPATH=/etc/peer/users/admin
export ORDERER_CA=/etc/peer/orderer/intermediate-ca.pem
export GOPAY_CA=/etc/peer/msp/tlsintermediatecerts/intermediate-ca.pem
export DANA_CA=/etc/peer/organizations/dana/msp/tlsintermediatecerts/intermediate-ca.pem

# invoke InitLedger function to create initial assets
peer chaincode invoke -o 10.250.250.20:7050 --channelID qris --name qris --tls --cafile $ORDERER_CA --peerAddresses 10.250.251.20:7051 --tlsRootCertFiles $ORG_CA --peerAddresses 10.250.252.20:7051 --tlsRootCertFiles $DANA_CA -c '{"function":"InitLedger","Args":[]}'

2022-03-13 18:12:22.660 UTC 0001 INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200

# check all assets created
peer chaincode invoke -o 10.250.250.20:7050 --channelID qris --name qris --tls --cafile $ORDERER_CA --peerAddresses 10.250.251.20:7051 --tlsRootCertFiles $ORG_CA --peerAddresses 10.250.252.20:7051 --tlsRootCertFiles $DANA_CA -c '{"Args":["GetAllAssets"]}'

2022-03-13 18:13:16.749 UTC 0001 INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200 payload:"[{\"ID\":\"8b8c7df1-48f8-4aba-9990-57a99d70e010\",\"Owner\":\"Bank Indonesia\",\"Producer\":\"Bank Indonesia\",\"Value\":0}]"

# create new assets
peer chaincode invoke -o 10.250.250.20:7050 --channelID qris --name qris --tls --cafile $ORDERER_CA --peerAddresses 10.250.251.20:7051 --tlsRootCertFiles $ORG_CA --peerAddresses 10.250.252.20:7051 --tlsRootCertFiles $DANA_CA -c '{"function":"CreateAsset","Args":["8b8c7df1-48f8-4aba-9990-57a99d70e011", "Zufar Dhiyaulhaq", "GoPay", "50"]}'

# check all assets created
peer chaincode invoke -o 10.250.250.20:7050 --channelID qris --name qris --tls --cafile $ORDERER_CA --peerAddresses 10.250.251.20:7051 --tlsRootCertFiles $ORG_CA --peerAddresses 10.250.252.20:7051 --tlsRootCertFiles $DANA_CA -c '{"Args":["GetAllAssets"]}'

2022-03-13 18:35:26.961 UTC 0001 INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200 payload:"[{\"ID\":\"8b8c7df1-48f8-4aba-9990-57a99d70e010\",\"Owner\":\"Bank Indonesia\",\"Producer\":\"Bank Indonesia\",\"Value\":0},{\"ID\":\"8b8c7df1-48f8-4aba-9990-57a99d70e011\",\"Owner\":\"Zufar Dhiyaulhaq\",\"Producer\":\"GoPay\",\"Value\":50}]"

# update assets
peer chaincode invoke -o 10.250.250.20:7050 --channelID qris --name qris --tls --cafile $ORDERER_CA --peerAddresses 10.250.251.20:7051 --tlsRootCertFiles $ORG_CA --peerAddresses 10.250.252.20:7051 --tlsRootCertFiles $DANA_CA -c '{"function":"UpdateAsset","Args":["8b8c7df1-48f8-4aba-9990-57a99d70e011", "Zufar Dhiyaulhaq", "GoPay", "500000000"]}'

# check specific assets 
peer chaincode invoke -o 10.250.250.20:7050 --channelID qris --name qris --tls --cafile $ORDERER_CA --peerAddresses 10.250.251.20:7051 --tlsRootCertFiles $ORG_CA --peerAddresses 10.250.252.20:7051 --tlsRootCertFiles $DANA_CA -c '{"function":"ReadAsset","Args":["8b8c7df1-48f8-4aba-9990-57a99d70e011"]}'

2022-03-13 18:38:50.794 UTC 0001 INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200 payload:"{\"ID\":\"8b8c7df1-48f8-4aba-9990-57a99d70e011\",\"Owner\":\"Zufar Dhiyaulhaq\",\"Producer\":\"GoPay\",\"Value\":500000000}"
```
