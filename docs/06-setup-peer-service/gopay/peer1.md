### gopay peer1
ssh to the nodes
```shell
vagrant ssh gopay-peer-1
```

Create directory for msp & tls certificate
```
sudo mkdir -p /etc/hyperledger/peer/organizations/PeerOrganizations/gopay/peers/peer1/tls
sudo mkdir -p /etc/hyperledger/peer/organizations/PeerOrganizations/gopay/peers/peer1/msp

sudo chmod 755 /etc/hyperledger/peer/organizations/PeerOrganizations/dana/peers/peer1/tls
sudo chmod 755 /etc/hyperledger/peer/organizations/PeerOrganizations/dana/peers/peer1/msp
```

create directory for peers
```
sudo mkdir -p /var/hyperledger/production/snapshots
```

get the msp certificate
```
sudo fabric-ca-client enroll -d -u https://peer1@enrollment.gopay.co.id:peer1-password@10.250.251.10:7055 --tls.certfiles ${HOME}/organizations/PeerOrganizations/gopay/msp/tlsintermediatecerts/intermediate-cert.pem --csr.names C=id,O=gopay,ST=jakarta --mspdir /etc/hyperledger/peer/organizations/PeerOrganizations/gopay/peers/peer1/msp
```

check the msp certificate
```
sudo tree /etc/hyperledger/peer/organizations/PeerOrganizations/gopay/peers/peer1/msp
/etc/hyperledger/peer/organizations/PeerOrganizations/gopay/peers/peer1/msp
├── IssuerPublicKey
├── IssuerRevocationPublicKey
├── cacerts
│   └── 10-250-251-10-7055.pem
├── intermediatecerts
│   └── 10-250-251-10-7055.pem
├── keystore
│   └── 078384dffcb39a420dbdac413227febba14106c9d0f2bc3f22ba24b89e597218_sk
├── signcerts
│   └── cert.pem
└── user
```

get the TLS certificate
```
sudo fabric-ca-client enroll -d -u https://peer1@tls.gopay.co.id:peer1-password@10.250.251.10:7054 --tls.certfiles ${HOME}/organizations/PeerOrganizations/gopay/msp/tlsintermediatecerts/intermediate-cert.pem --enrollment.profile tls --csr.cn peer --csr.hosts 'localhost,127.0.0.1,10.250.251.21' --csr.names C=id,O=gopay,ST=jakarta --mspdir /etc/hyperledger/peer/organizations/PeerOrganizations/gopay/peers/peer1/tls
```

check the TLS certificate
```
sudo tree /etc/hyperledger/peer/organizations/PeerOrganizations/gopay/peers/peer1/tls
/etc/hyperledger/peer/organizations/PeerOrganizations/gopay/peers/peer1/tls
├── IssuerPublicKey
├── IssuerRevocationPublicKey
├── cacerts
├── keystore
│   └── b3d0194d963328b8e8e82461ad24cf130ba281bbec7898c5b35495eca709c18e_sk
├── signcerts
│   └── cert.pem
├── tlscacerts
│   └── tls-10-250-251-10-7054.pem
├── tlsintermediatecerts
│   └── tls-10-250-251-10-7054.pem
└── user
```

rename the TLS secret key for better usage by peer nodes
```
sudo mv /etc/hyperledger/peer/organizations/PeerOrganizations/gopay/peers/peer1/tls/keystore/b3d0194d963328b8e8e82461ad24cf130ba281bbec7898c5b35495eca709c18e_sk /etc/hyperledger/peer/organizations/PeerOrganizations/gopay/peers/peer1/tls/keystore/key.pem
```

create `config.yaml` in msp directory, this is used to identify each roles from certificate OU, read more here https://hyperledger-fabric-ca.readthedocs.io/en/latest/deployguide/use_CA.html#nodeous
```
cat <<EOF | sudo tee /etc/hyperledger/peer/organizations/PeerOrganizations/gopay/peers/peer1/msp/config.yaml
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

create core configuration 
```
cat <<'EOF' | sudo tee /etc/hyperledger/peer/core.yaml
# https://github.com/hyperledger/fabric/blob/main/sampleconfig/core.yaml
peer:
  id: peer1.gopay
  networkId: production
  address: 10.250.251.21:7051
  listenAddress: 0.0.0.0:7051
  chaincodeAddress: 10.250.251.21:7052
  chaincodeListenAddress: 0.0.0.0:7052
  mspConfigPath: /etc/hyperledger/peer/organizations/PeerOrganizations/gopay/peers/peer1/msp
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
      file: /etc/hyperledger/peer/organizations/PeerOrganizations/gopay/peers/peer1/tls/signcerts/cert.pem
    key:
      file: /etc/hyperledger/peer/organizations/PeerOrganizations/gopay/peers/peer1/tls/keystore/key.pem
    rootcert:
      file: /etc/hyperledger/peer/organizations/PeerOrganizations/gopay/peers/peer1/tls/tlsintermediatecerts/tls-10-250-251-10-7054.pem

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

create peer systemd unit file
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
Environment=FABRIC_CFG_PATH=/etc/hyperledger/peer
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
