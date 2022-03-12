### dana-peer-0 
Download peer binary
```
wget https://github.com/hyperledger/fabric/releases/download/v2.4.3/hyperledger-fabric-linux-amd64-2.4.3.tar.gz
tar xzvf hyperledger-fabric-linux-amd64-2.4.3.tar.gz
sudo cp bin/* /usr/local/bin/
rm -rf bin/
rm -rf config/
rm -rf hyperledger-fabric-linux-amd64-2.4.3.tar.gz
```

install docker
```
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release -y 
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update && sudo apt-get install docker-ce docker-ce-cli containerd.io -y
```

Create directory for msp & tls certificate
```
sudo mkdir -p /etc/peer/msp/cacerts
sudo mkdir -p /etc/peer/msp/intermediatecerts
sudo mkdir -p /etc/peer/msp/keystore
sudo mkdir -p /etc/peer/msp/signcerts
sudo mkdir -p /etc/peer/msp/tlscacerts
sudo mkdir -p /etc/peer/msp/tlsintermediatecerts
sudo mkdir -p /etc/peer/msp/operationscerts
sudo mkdir -p /etc/peer/tls/
```

create directory for peers
```
sudo mkdir -p /var/hyperledger/production/snapshots
```

create directory for tls & enrollment certificate, we will copy certificate in this directory to `/etc/peer`
```
mkdir -p certificates/peer/tls/
mkdir -p certificates/peer/enrollment/
```

get the certificate
```
fabric-ca-client enroll -d -u https://peer0@dana:peer0-dana-password@10.250.252.10:7054 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --enrollment.profile tls --csr.cn peer --csr.hosts 'localhost,127.0.0.1,10.250.252.20' --csr.names C=id,O=dana,ST=jakarta --mspdir ${HOME}/certificates/peer/tls
fabric-ca-client enroll -d -u https://peer0@dana:peer0-dana-password@10.250.252.10:7055 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --csr.names C=id,O=dana,ST=jakarta --mspdir ${HOME}/certificates/peer/enrollment
```

check the certificate
```
tree ${HOME}/certificates/peer/
/home/vagrant/certificates/peer/
├── enrollment
│   ├── IssuerPublicKey
│   ├── IssuerRevocationPublicKey
│   ├── cacerts
│   │   └── 10-250-252-10-7055.pem
│   ├── intermediatecerts
│   │   └── 10-250-252-10-7055.pem
│   ├── keystore
│   │   └── cb287d1c1060a58569ba50fe2136b20e019fd9ab2743679a8b905e019f3f1783_sk
│   ├── signcerts
│   │   └── cert.pem
│   └── user
└── tls
    ├── IssuerPublicKey
    ├── IssuerRevocationPublicKey
    ├── cacerts
    ├── keystore
    │   └── d3024bb3eb8f9e3c978b40bc4d583636a907bb213c40c9a1fe1b26e0814b3fe5_sk
    ├── signcerts
    │   └── cert.pem
    ├── tlscacerts
    │   └── tls-10-250-252-10-7054.pem
    ├── tlsintermediatecerts
    │   └── tls-10-250-252-10-7054.pem
    └── user
```

copy the certificate to `/etc/peer`
```
sudo cp certificates/peer/enrollment/signcerts/cert.pem /etc/peer/msp/signcerts/cert.pem
sudo cp certificates/peer/enrollment/keystore/cb287d1c1060a58569ba50fe2136b20e019fd9ab2743679a8b905e019f3f1783_sk /etc/peer/msp/keystore/key.pem
sudo cp certificates/peer/enrollment/cacerts/10-250-252-10-7055.pem /etc/peer/msp/cacerts/root-ca.pem
sudo cp certificates/peer/enrollment/intermediatecerts/10-250-252-10-7055.pem /etc/peer/msp/intermediatecerts/intermediate-ca.pem

sudo cp certificates/peer/tls/tlscacerts/tls-10-250-252-10-7054.pem /etc/peer/msp/tlscacerts/root-ca.pem
sudo cp certificates/peer/tls/tlsintermediatecerts/tls-10-250-252-10-7054.pem /etc/peer/msp/tlsintermediatecerts/intermediate-ca.pem

sudo cp certificates/peer/tls/tlsintermediatecerts/tls-10-250-252-10-7054.pem /etc/peer/tls/intermediate-ca.pem
sudo cp certificates/peer/tls/signcerts/cert.pem /etc/peer/tls/cert.pem
sudo cp certificates/peer/tls/keystore/d3024bb3eb8f9e3c978b40bc4d583636a907bb213c40c9a1fe1b26e0814b3fe5_sk /etc/peer/tls/key.pem
```

create core configuration 
```
cat <<EOF | sudo tee /etc/peer/core.yaml
# https://github.com/hyperledger/fabric/blob/main/sampleconfig/core.yaml
peer:
  id: peer0.dana
  networkId: production
  address: 10.250.252.20:7051
  listenAddress: 0.0.0.0:7051
  chaincodeAddress: 10.250.252.20:7052
  chaincodeListenAddress: 0.0.0.0:7052
  mspConfigPath: /etc/peer/msp
  localMspId: dana
  fileSystemPath: /var/hyperledger/production

  gossip:
    endpoint: 10.250.252.20:7051
    externalEndpoint: 10.250.252.20:7051
    bootstrap: 10.250.252.21:7051

  tls:
    enabled: true
    clientAuthRequired: false
    cert:
      file: tls/cert.pem
    key:
      file: tls/key.pem
    rootcert:
      file: tls/intermediate-ca.pem

  BCCSP:
    Default: SW
    SW:
      Hash: SHA2
      Security: 256

  gateway:
    enabled: true
    endorsementTimeout: 30s
    dialTimeout: 2m

  discovery:
    enabled: true
    authCacheEnabled: true
    authCacheMaxSize: 1000
    authCachePurgeRetentionRatio: 0.75
    orgMembersAllowedAccess: false

ledger:
  state:
    stateDatabase: goleveldb
    totalQueryLimit: 100000
  snapshots:
    rootDir: /var/hyperledger/production/snapshots

operations:
  listenAddress: 127.0.0.1:9443
  tls:
    enabled: false

metrics:
  provider: prometheus

vm:
  endpoint: unix:///var/run/docker.sock

chaincode:
  externalBuilders: []
EOF
```

create `config.yaml` in msp directory, this is used to identify each roles from certificate OU, read more here https://hyperledger-fabric-ca.readthedocs.io/en/latest/deployguide/use_CA.html#nodeous
```
cat <<EOF | sudo tee /etc/peer/msp/config.yaml
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

create orderer systemd unit file
```
cat <<EOF | sudo tee /etc/systemd/system/fabric-peer.service
# Service definition for Hyperledger fabric peer server
[Unit]
Description=hyperledger fabric-peer server - fabric peer for hyperledger fabric
Documentation=https://hyperledger-fabric.readthedocs.io/
Wants=network-online.target
After=network-online.target
[Service]
Type=simple
Restart=on-failure
Environment=FABRIC_CFG_PATH=/etc/peer
ExecStart=/usr/local/bin/peer node start
[Install]
WantedBy=multi-user.target
EOF
```

start fabric peer
```shell
sudo systemctl enable fabric-peer.service
sudo systemctl start fabric-peer.service
sudo systemctl status fabric-peer.service
```

### dana-peer-1
Download peer binary
```
wget https://github.com/hyperledger/fabric/releases/download/v2.4.3/hyperledger-fabric-linux-amd64-2.4.3.tar.gz
tar xzvf hyperledger-fabric-linux-amd64-2.4.3.tar.gz
sudo cp bin/* /usr/local/bin/
rm -rf bin/
rm -rf config/
rm -rf hyperledger-fabric-linux-amd64-2.4.3.tar.gz
```

install docker
```
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release -y 
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update && sudo apt-get install docker-ce docker-ce-cli containerd.io -y
```

Create directory for msp & tls certificate
```
sudo mkdir -p /etc/peer/msp/cacerts
sudo mkdir -p /etc/peer/msp/intermediatecerts
sudo mkdir -p /etc/peer/msp/keystore
sudo mkdir -p /etc/peer/msp/signcerts
sudo mkdir -p /etc/peer/msp/tlscacerts
sudo mkdir -p /etc/peer/msp/tlsintermediatecerts
sudo mkdir -p /etc/peer/msp/operationscerts
sudo mkdir -p /etc/peer/tls/
```

create directory for peers
```
sudo mkdir -p /var/hyperledger/production/snapshots
```

create directory for tls & enrollment certificate, we will copy certificate in this directory to `/etc/peer`
```
mkdir -p certificates/peer/tls/
mkdir -p certificates/peer/enrollment/
```

get the certificate
```
fabric-ca-client enroll -d -u https://peer1@dana:peer1-dana-password@10.250.252.10:7054 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --enrollment.profile tls --csr.cn peer --csr.hosts 'localhost,127.0.0.1,10.250.252.21' --csr.names C=id,O=dana,ST=jakarta --mspdir ${HOME}/certificates/peer/tls
fabric-ca-client enroll -d -u https://peer1@dana:peer1-dana-password@10.250.252.10:7055 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --csr.names C=id,O=dana,ST=jakarta --mspdir ${HOME}/certificates/peer/enrollment
```

check the certificate
```
tree ${HOME}/certificates/peer/
/home/vagrant/certificates/peer/
├── enrollment
│   ├── IssuerPublicKey
│   ├── IssuerRevocationPublicKey
│   ├── cacerts
│   │   └── 10-250-252-10-7055.pem
│   ├── intermediatecerts
│   │   └── 10-250-252-10-7055.pem
│   ├── keystore
│   │   └── bbe0dd3a80d89feb5849dc3615bdd7856f927f9beb65798469452457bc7bc59c_sk
│   ├── signcerts
│   │   └── cert.pem
│   └── user
└── tls
    ├── IssuerPublicKey
    ├── IssuerRevocationPublicKey
    ├── cacerts
    ├── keystore
    │   └── a7d9c96ff4d619843bf372e2c326890266575278090d573a33613eb9af311820_sk
    ├── signcerts
    │   └── cert.pem
    ├── tlscacerts
    │   └── tls-10-250-252-10-7054.pem
    ├── tlsintermediatecerts
    │   └── tls-10-250-252-10-7054.pem
    └── user
```

copy the certificate to `/etc/peer`
```
sudo cp certificates/peer/enrollment/signcerts/cert.pem /etc/peer/msp/signcerts/cert.pem
sudo cp certificates/peer/enrollment/keystore/bbe0dd3a80d89feb5849dc3615bdd7856f927f9beb65798469452457bc7bc59c_sk /etc/peer/msp/keystore/key.pem
sudo cp certificates/peer/enrollment/cacerts/10-250-252-10-7055.pem /etc/peer/msp/cacerts/root-ca.pem
sudo cp certificates/peer/enrollment/intermediatecerts/10-250-252-10-7055.pem /etc/peer/msp/intermediatecerts/intermediate-ca.pem

sudo cp certificates/peer/tls/tlscacerts/tls-10-250-252-10-7054.pem /etc/peer/msp/tlscacerts/root-ca.pem
sudo cp certificates/peer/tls/tlsintermediatecerts/tls-10-250-252-10-7054.pem /etc/peer/msp/tlsintermediatecerts/intermediate-ca.pem

sudo cp certificates/peer/tls/tlsintermediatecerts/tls-10-250-252-10-7054.pem /etc/peer/tls/intermediate-ca.pem
sudo cp certificates/peer/tls/signcerts/cert.pem /etc/peer/tls/cert.pem
sudo cp certificates/peer/tls/keystore/a7d9c96ff4d619843bf372e2c326890266575278090d573a33613eb9af311820_sk /etc/peer/tls/key.pem
```

create core configuration 
```
cat <<EOF | sudo tee /etc/peer/core.yaml
# https://github.com/hyperledger/fabric/blob/main/sampleconfig/core.yaml
peer:
  id: peer1.dana
  networkId: production
  address: 10.250.252.21:7051
  listenAddress: 0.0.0.0:7051
  chaincodeAddress: 10.250.252.21:7052
  chaincodeListenAddress: 0.0.0.0:7052
  mspConfigPath: /etc/peer/msp
  localMspId: dana
  fileSystemPath: /var/hyperledger/production

  gossip:
    endpoint: 10.250.252.21:7051
    externalEndpoint: 10.250.252.21:7051
    bootstrap: 10.250.252.20:7051

  tls:
    enabled: true
    clientAuthRequired: false
    cert:
      file: tls/cert.pem
    key:
      file: tls/key.pem
    rootcert:
      file: tls/intermediate-ca.pem

  BCCSP:
    Default: SW
    SW:
      Hash: SHA2
      Security: 256

  gateway:
    enabled: true
    endorsementTimeout: 30s
    dialTimeout: 2m

  discovery:
    enabled: true
    authCacheEnabled: true
    authCacheMaxSize: 1000
    authCachePurgeRetentionRatio: 0.75
    orgMembersAllowedAccess: false

ledger:
  state:
    stateDatabase: goleveldb
    totalQueryLimit: 100000
  snapshots:
    rootDir: /var/hyperledger/production/snapshots

operations:
  listenAddress: 127.0.0.1:9443
  tls:
    enabled: false

metrics:
  provider: prometheus

vm:
  endpoint: unix:///var/run/docker.sock

chaincode:
  externalBuilders: []
EOF
```

create `config.yaml` in msp directory, this is used to identify each roles from certificate OU, read more here https://hyperledger-fabric-ca.readthedocs.io/en/latest/deployguide/use_CA.html#nodeous
```
cat <<EOF | sudo tee /etc/peer/msp/config.yaml
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

create orderer systemd unit file
```
cat <<EOF | sudo tee /etc/systemd/system/fabric-peer.service
# Service definition for Hyperledger fabric peer server
[Unit]
Description=hyperledger fabric-peer server - fabric peer for hyperledger fabric
Documentation=https://hyperledger-fabric.readthedocs.io/
Wants=network-online.target
After=network-online.target
[Service]
Type=simple
Restart=on-failure
Environment=FABRIC_CFG_PATH=/etc/peer
ExecStart=/usr/local/bin/peer node start
[Install]
WantedBy=multi-user.target
EOF
```

start fabric peer
```shell
sudo systemctl enable fabric-peer.service
sudo systemctl start fabric-peer.service
sudo systemctl status fabric-peer.service
```
