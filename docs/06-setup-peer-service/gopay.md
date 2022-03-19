### gopay-peer-0 
ssh to the nodes
```shell
vagrant ssh gopay-peer-0
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
sudo mkdir -p /etc/peer/msp/admincerts

sudo mkdir -p /etc/peer/users/admin/cacerts
sudo mkdir -p /etc/peer/users/admin/intermediatecerts
sudo mkdir -p /etc/peer/users/admin/keystore
sudo mkdir -p /etc/peer/users/admin/signcerts
sudo mkdir -p /etc/peer/users/admin/tlscacerts
sudo mkdir -p /etc/peer/users/admin/tlsintermediatecerts
sudo mkdir -p /etc/peer/users/admin/operationscerts
sudo mkdir -p /etc/peer/users/admin/admincerts

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
mkdir -p certificates/peer/enrollment-admin/
```

get the certificate
```
fabric-ca-client enroll -d -u https://peer0@gopay:peer0-gopay-password@10.250.251.10:7054 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --enrollment.profile tls --csr.cn peer --csr.hosts 'localhost,127.0.0.1,10.250.251.20' --csr.names C=id,O=gopay,ST=jakarta --mspdir ${HOME}/certificates/peer/tls
fabric-ca-client enroll -d -u https://peer0@gopay:peer0-gopay-password@10.250.251.10:7055 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --csr.names C=id,O=gopay,ST=jakarta --mspdir ${HOME}/certificates/peer/enrollment
fabric-ca-client enroll -d -u https://administrator@gopay:administrator-gopay-password@10.250.251.10:7055 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --csr.names C=id,O=gopay,ST=jakarta --mspdir ${HOME}/certificates/peer/enrollment-admin
```

check the certificate
```
tree ${HOME}/certificates/peer/
/home/vagrant/certificates/peer/
├── enrollment
│   ├── IssuerPublicKey
│   ├── IssuerRevocationPublicKey
│   ├── cacerts
│   │   └── 10-250-251-10-7055.pem
│   ├── intermediatecerts
│   │   └── 10-250-251-10-7055.pem
│   ├── keystore
│   │   └── 3d57c1e499318b45808db864bbb7802823e5497fb5c3cc28c98f2224c5d52a6a_sk
│   ├── signcerts
│   │   └── cert.pem
│   └── user
├── enrollment-admin
│   ├── IssuerPublicKey
│   ├── IssuerRevocationPublicKey
│   ├── cacerts
│   │   └── 10-250-251-10-7055.pem
│   ├── intermediatecerts
│   │   └── 10-250-251-10-7055.pem
│   ├── keystore
│   │   └── 8f5aa65b05c253d00070067df0520873b17cf9a85b4eb1093c929a6f93f6f979_sk
│   ├── signcerts
│   │   └── cert.pem
│   └── user
└── tls
    ├── IssuerPublicKey
    ├── IssuerRevocationPublicKey
    ├── cacerts
    ├── keystore
    │   └── 483c6d7b037aa0a038c8645e4bbf7fdad8f822a7ae9afa1fb0e0847702f070f2_sk
    ├── signcerts
    │   └── cert.pem
    ├── tlscacerts
    │   └── tls-10-250-251-10-7054.pem
    ├── tlsintermediatecerts
    │   └── tls-10-250-251-10-7054.pem
    └── user
```

copy the certificate to `/etc/peer`
```
sudo cp certificates/peer/enrollment-admin/signcerts/cert.pem /etc/peer/msp/admincerts/cert.pem
sudo cp certificates/peer/enrollment/signcerts/cert.pem /etc/peer/msp/signcerts/cert.pem
sudo cp certificates/peer/enrollment/keystore/3d57c1e499318b45808db864bbb7802823e5497fb5c3cc28c98f2224c5d52a6a_sk /etc/peer/msp/keystore/key.pem
sudo cp certificates/peer/enrollment/cacerts/10-250-251-10-7055.pem /etc/peer/msp/cacerts/root-ca.pem
sudo cp certificates/peer/enrollment/intermediatecerts/10-250-251-10-7055.pem /etc/peer/msp/intermediatecerts/intermediate-ca.pem

sudo cp certificates/peer/tls/tlscacerts/tls-10-250-251-10-7054.pem /etc/peer/msp/tlscacerts/root-ca.pem
sudo cp certificates/peer/tls/tlsintermediatecerts/tls-10-250-251-10-7054.pem /etc/peer/msp/tlsintermediatecerts/intermediate-ca.pem

sudo cp certificates/peer/enrollment-admin/signcerts/cert.pem /etc/peer/users/admin/admincerts/cert.pem
sudo cp certificates/peer/enrollment-admin/signcerts/cert.pem /etc/peer/users/admin/signcerts/cert.pem
sudo cp certificates/peer/enrollment-admin/keystore/8f5aa65b05c253d00070067df0520873b17cf9a85b4eb1093c929a6f93f6f979_sk /etc/peer/users/admin/keystore/key.pem
sudo cp certificates/peer/enrollment-admin/cacerts/10-250-251-10-7055.pem /etc/peer/users/admin/cacerts/root-ca.pem
sudo cp certificates/peer/enrollment-admin/intermediatecerts/10-250-251-10-7055.pem /etc/peer/users/admin/intermediatecerts/intermediate-ca.pem

sudo cp certificates/peer/tls/tlscacerts/tls-10-250-251-10-7054.pem /etc/peer/users/admin/tlscacerts/root-ca.pem
sudo cp certificates/peer/tls/tlsintermediatecerts/tls-10-250-251-10-7054.pem /etc/peer/users/admin/tlsintermediatecerts/intermediate-ca.pem

sudo cp certificates/peer/tls/tlsintermediatecerts/tls-10-250-251-10-7054.pem /etc/peer/tls/intermediate-ca.pem
sudo cp certificates/peer/tls/signcerts/cert.pem /etc/peer/tls/cert.pem
sudo cp certificates/peer/tls/keystore/483c6d7b037aa0a038c8645e4bbf7fdad8f822a7ae9afa1fb0e0847702f070f2_sk /etc/peer/tls/key.pem
```

create core configuration 
```
cat <<EOF | sudo tee /etc/peer/core.yaml
# https://github.com/hyperledger/fabric/blob/main/sampleconfig/core.yaml
peer:
  id: peer0.gopay
  networkId: production
  address: 10.250.251.20:7051
  listenAddress: 0.0.0.0:7051
  chaincodeAddress: 10.250.251.20:7052
  chaincodeListenAddress: 0.0.0.0:7052
  mspConfigPath: /etc/peer/msp
  localMspId: gopay
  fileSystemPath: /var/hyperledger/production

  gossip:
    endpoint: 10.250.251.20:7051
    externalEndpoint: 10.250.251.20:7051
    bootstrap: 10.250.251.21:7051
    useLeaderElection: false
    orgLeader: true
    state:
        enabled: true
    pvtData:
      pushAckTimeout: 3s
      implicitCollectionDisseminationPolicy:
          requiredPeerCount: 0
          maxPeerCount: 1

  handlers:
    authFilters:
      - name: DefaultAuth
      - name: ExpirationCheck   
    decorators:
      - name: DefaultDecorator
    endorsers:
      escc:
        name: DefaultEndorsement
        library:
    validators:
      vscc:
        name: DefaultValidation
        library:

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
  builder: $(DOCKER_NS)/fabric-ccenv:$(TWO_DIGIT_VERSION)
  pull: false
  golang:
      runtime: $(DOCKER_NS)/fabric-baseos:$(TWO_DIGIT_VERSION)
      dynamicLink: false
  java:
      runtime: $(DOCKER_NS)/fabric-javaenv:$(TWO_DIGIT_VERSION)
  node:
      runtime: $(DOCKER_NS)/fabric-nodeenv:$(TWO_DIGIT_VERSION)
  system:
    _lifecycle: enable
    cscc: enable
    lscc: enable
    qscc: enable
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

### gopay-peer-1
ssh to the nodes
```shell
vagrant ssh gopay-peer-1
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
sudo mkdir -p /etc/peer/msp/admincerts

sudo mkdir -p /etc/peer/users/admin/cacerts
sudo mkdir -p /etc/peer/users/admin/intermediatecerts
sudo mkdir -p /etc/peer/users/admin/keystore
sudo mkdir -p /etc/peer/users/admin/signcerts
sudo mkdir -p /etc/peer/users/admin/tlscacerts
sudo mkdir -p /etc/peer/users/admin/tlsintermediatecerts
sudo mkdir -p /etc/peer/users/admin/operationscerts
sudo mkdir -p /etc/peer/users/admin/admincerts

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
mkdir -p certificates/peer/enrollment-admin/
```

get the certificate
```
fabric-ca-client enroll -d -u https://peer1@gopay:peer1-gopay-password@10.250.251.10:7054 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --enrollment.profile tls --csr.cn peer --csr.hosts 'localhost,127.0.0.1,10.250.251.21' --csr.names C=id,O=gopay,ST=jakarta --mspdir ${HOME}/certificates/peer/tls
fabric-ca-client enroll -d -u https://peer1@gopay:peer1-gopay-password@10.250.251.10:7055 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --csr.names C=id,O=gopay,ST=jakarta --mspdir ${HOME}/certificates/peer/enrollment
fabric-ca-client enroll -d -u https://administrator@gopay:administrator-gopay-password@10.250.251.10:7055 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --csr.names C=id,O=gopay,ST=jakarta --mspdir ${HOME}/certificates/peer/enrollment-admin
```

check the certificate
```
tree ${HOME}/certificates/peer/
/home/vagrant/certificates/peer/
├── enrollment
│   ├── IssuerPublicKey
│   ├── IssuerRevocationPublicKey
│   ├── cacerts
│   │   └── 10-250-251-10-7055.pem
│   ├── intermediatecerts
│   │   └── 10-250-251-10-7055.pem
│   ├── keystore
│   │   └── d31f1645fce671070f6614846ce8b1f9dd8a810746cc296880594eaeca3123dd_sk
│   ├── signcerts
│   │   └── cert.pem
│   └── user
├── enrollment-admin
│   ├── IssuerPublicKey
│   ├── IssuerRevocationPublicKey
│   ├── cacerts
│   │   └── 10-250-251-10-7055.pem
│   ├── intermediatecerts
│   │   └── 10-250-251-10-7055.pem
│   ├── keystore
│   │   └── 7ad5f89442bf06d8816f717f1adac0a409580bdfb29b0520ba1e2a09fcabe689_sk
│   ├── signcerts
│   │   └── cert.pem
│   └── user
└── tls
    ├── IssuerPublicKey
    ├── IssuerRevocationPublicKey
    ├── cacerts
    ├── keystore
    │   └── 40c78c89b5ae45fe95d18647d5fc12f1c2ca639e57fefe4bc1d4985646d29db5_sk
    ├── signcerts
    │   └── cert.pem
    ├── tlscacerts
    │   └── tls-10-250-251-10-7054.pem
    ├── tlsintermediatecerts
    │   └── tls-10-250-251-10-7054.pem
    └── user
```

copy the certificate to `/etc/peer`
```
sudo cp certificates/peer/enrollment-admin/signcerts/cert.pem /etc/peer/msp/admincerts/cert.pem
sudo cp certificates/peer/enrollment/signcerts/cert.pem /etc/peer/msp/signcerts/cert.pem
sudo cp certificates/peer/enrollment/keystore/d31f1645fce671070f6614846ce8b1f9dd8a810746cc296880594eaeca3123dd_sk /etc/peer/msp/keystore/key.pem
sudo cp certificates/peer/enrollment/cacerts/10-250-251-10-7055.pem /etc/peer/msp/cacerts/root-ca.pem
sudo cp certificates/peer/enrollment/intermediatecerts/10-250-251-10-7055.pem /etc/peer/msp/intermediatecerts/intermediate-ca.pem

sudo cp certificates/peer/tls/tlscacerts/tls-10-250-251-10-7054.pem /etc/peer/msp/tlscacerts/root-ca.pem
sudo cp certificates/peer/tls/tlsintermediatecerts/tls-10-250-251-10-7054.pem /etc/peer/msp/tlsintermediatecerts/intermediate-ca.pem

sudo cp certificates/peer/enrollment-admin/signcerts/cert.pem /etc/peer/users/admin/admincerts/cert.pem
sudo cp certificates/peer/enrollment-admin/signcerts/cert.pem /etc/peer/users/admin/signcerts/cert.pem
sudo cp certificates/peer/enrollment-admin/keystore/7ad5f89442bf06d8816f717f1adac0a409580bdfb29b0520ba1e2a09fcabe689_sk /etc/peer/users/admin/keystore/key.pem
sudo cp certificates/peer/enrollment-admin/cacerts/10-250-251-10-7055.pem /etc/peer/users/admin/cacerts/root-ca.pem
sudo cp certificates/peer/enrollment-admin/intermediatecerts/10-250-251-10-7055.pem /etc/peer/users/admin/intermediatecerts/intermediate-ca.pem

sudo cp certificates/peer/tls/tlscacerts/tls-10-250-251-10-7054.pem /etc/peer/users/admin/tlscacerts/root-ca.pem
sudo cp certificates/peer/tls/tlsintermediatecerts/tls-10-250-251-10-7054.pem /etc/peer/users/admin/tlsintermediatecerts/intermediate-ca.pem

sudo cp certificates/peer/tls/tlsintermediatecerts/tls-10-250-251-10-7054.pem /etc/peer/tls/intermediate-ca.pem
sudo cp certificates/peer/tls/signcerts/cert.pem /etc/peer/tls/cert.pem
sudo cp certificates/peer/tls/keystore/40c78c89b5ae45fe95d18647d5fc12f1c2ca639e57fefe4bc1d4985646d29db5_sk /etc/peer/tls/key.pem
```

create core configuration 
```
cat <<EOF | sudo tee /etc/peer/core.yaml
# https://github.com/hyperledger/fabric/blob/main/sampleconfig/core.yaml
peer:
  id: peer1.gopay
  networkId: production
  address: 10.250.251.21:7051
  listenAddress: 0.0.0.0:7051
  chaincodeAddress: 10.250.251.21:7052
  chaincodeListenAddress: 0.0.0.0:7052
  mspConfigPath: /etc/peer/msp
  localMspId: gopay
  fileSystemPath: /var/hyperledger/production

  gossip:
    endpoint: 10.250.251.21:7051
    externalEndpoint: 10.250.251.21:7051
    bootstrap: 10.250.251.20:7051
    useLeaderElection: false
    orgLeader: true
    state:
        enabled: true
    pvtData:
      pushAckTimeout: 3s
      implicitCollectionDisseminationPolicy:
          requiredPeerCount: 0
          maxPeerCount: 1

  handlers:
    authFilters:
      - name: DefaultAuth
      - name: ExpirationCheck   
    decorators:
      - name: DefaultDecorator
    endorsers:
      escc:
        name: DefaultEndorsement
        library:
    validators:
      vscc:
        name: DefaultValidation
        library:

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
  builder: $(DOCKER_NS)/fabric-ccenv:$(TWO_DIGIT_VERSION)
  pull: false
  golang:
      runtime: $(DOCKER_NS)/fabric-baseos:$(TWO_DIGIT_VERSION)
      dynamicLink: false
  java:
      runtime: $(DOCKER_NS)/fabric-javaenv:$(TWO_DIGIT_VERSION)
  node:
      runtime: $(DOCKER_NS)/fabric-nodeenv:$(TWO_DIGIT_VERSION)
  system:
    _lifecycle: enable
    cscc: enable
    lscc: enable
    qscc: enable
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
