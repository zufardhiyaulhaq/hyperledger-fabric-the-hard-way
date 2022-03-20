### Bank Indonesia orderer1 node
ssh to the nodes
```shell
vagrant ssh bi-orderer-1
```

Create directory for msp & tls certificate directory for orderer1 node
```
sudo mkdir -p /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer1/tls
sudo mkdir -p /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer1/msp
```

create directory for etcdraft
```
sudo mkdir -p /var/hyperledger/production/orderer/etcdraft/wal
sudo mkdir -p /var/hyperledger/production/orderer/etcdraft/snapshot
```

get the msp certificate
```
sudo fabric-ca-client enroll -d -u https://orderer1@enrollment.bi.go.id:orderer1-password@10.250.250.10:7055 --tls.certfiles ${HOME}/organizations/OrdererOrganizations/bi/msp/tlsintermediatecerts/intermediate-cert.pem --csr.names C=id,O=bi,ST=jakarta --mspdir /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer1/msp
```

check the msp certificate
```
sudo tree /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer1/msp
/etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer1/msp
├── IssuerPublicKey
├── IssuerRevocationPublicKey
├── cacerts
│   └── 10-250-250-10-7055.pem
├── intermediatecerts
│   └── 10-250-250-10-7055.pem
├── keystore
│   └── 72742a8403143a520e5345594fa55c4d2cfa2d1e975595a195abb90bc60e793c_sk
├── signcerts
│   └── cert.pem
└── user
```

get the TLS certificate
```
sudo fabric-ca-client enroll -d -u https://orderer1@tls.bi.go.id:orderer1-password@10.250.250.10:7054 --tls.certfiles ${HOME}/organizations/OrdererOrganizations/bi/msp/tlsintermediatecerts/intermediate-cert.pem --enrollment.profile tls --csr.cn orderer --csr.hosts 'localhost,127.0.0.1,10.250.250.21' --csr.names C=id,O=bi,ST=jakarta --mspdir /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer1/tls
```

check the TLS certificate
```
sudo tree /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer1/tls
/etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer1/tls
├── IssuerPublicKey
├── IssuerRevocationPublicKey
├── cacerts
├── keystore
│   └── 83add2b5eea7edbe4e0ac89682818daf16a63c798b8ed0e5fc5181dc4c37afe1_sk
├── signcerts
│   └── cert.pem
├── tlscacerts
│   └── tls-10-250-250-10-7054.pem
├── tlsintermediatecerts
│   └── tls-10-250-250-10-7054.pem
└── user
```

rename the TLS secret key for better usage by orderer nodes
```
sudo mv /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer1/tls/keystore/83add2b5eea7edbe4e0ac89682818daf16a63c798b8ed0e5fc5181dc4c37afe1_sk /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer1/tls/keystore/key.pem
```

create `config.yaml` in msp directory, this is used to identify each roles from certificate OU, read more here https://hyperledger-fabric-ca.readthedocs.io/en/latest/deployguide/use_CA.html#nodeous
```
cat <<EOF | sudo tee /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer1/msp/config.yaml
NodeOUs:
 Enable: true
 ClientOUIdentifier:
   Certificate: intermediatecerts/10-250-250-10-7055.pem
   OrganizationalUnitIdentifier: client
 PeerOUIdentifier:
   Certificate: intermediatecerts/10-250-250-10-7055.pem
   OrganizationalUnitIdentifier: peer
 AdminOUIdentifier:
   Certificate: intermediatecerts/10-250-250-10-7055.pem
   OrganizationalUnitIdentifier: admin
 OrdererOUIdentifier:
   Certificate: intermediatecerts/10-250-250-10-7055.pem
   OrganizationalUnitIdentifier: orderer
EOF
```

create orderer configuration
```
cat <<EOF | sudo tee /etc/hyperledger/orderer/orderer.yaml
General:
    ListenAddress: 0.0.0.0
    ListenPort: 7050
    TLS:
        Enabled: true
        PrivateKey: /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer1/tls/keystore/key.pem
        Certificate: /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer1/tls/signcerts/cert.pem
        ClientAuthRequired: false
    Keepalive:
        ServerMinInterval: 60s
        ServerInterval: 7200s
        ServerTimeout: 20s

    MaxRecvMsgSize: 104857600
    MaxSendMsgSize: 104857600

    Cluster:
        SendBufferSize: 10

    BootstrapMethod: none

    LocalMSPDir: /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer1/msp/
    LocalMSPID: bi

    BCCSP:
        Default: SW
        SW:
            Hash: SHA2
            Security: 256

    Authentication:
        TimeWindow: 15m

FileLedger:
    Location: /var/hyperledger/production/orderer

Operations:
    ListenAddress: 127.0.0.1:11443
    TLS:
        Enabled: false

Metrics:
    Provider: prometheus

Admin:
    ListenAddress: 127.0.0.1:12443

    TLS:
        Enabled: false

ChannelParticipation:
    Enabled: true
    MaxRequestBodySize: 1 MB

Consensus:
    WALDir: /var/hyperledger/production/orderer/etcdraft/wal
    SnapDir: /var/hyperledger/production/orderer/etcdraft/snapshot
EOF
```

check the orderer config directory
```
sudo tree /etc/hyperledger/orderer
/etc/hyperledger/orderer
├── orderer.yaml
└── organizations
    └── OrdererOrganizations
        └── bi
            └── orderers
                └── orderer1
                    ├── msp
                    │   ├── IssuerPublicKey
                    │   ├── IssuerRevocationPublicKey
                    │   ├── cacerts
                    │   │   └── 10-250-250-10-7055.pem
                    │   ├── config.yaml
                    │   ├── intermediatecerts
                    │   │   └── 10-250-250-10-7055.pem
                    │   ├── keystore
                    │   │   └── 72742a8403143a520e5345594fa55c4d2cfa2d1e975595a195abb90bc60e793c_sk
                    │   ├── signcerts
                    │   │   └── cert.pem
                    │   └── user
                    └── tls
                        ├── IssuerPublicKey
                        ├── IssuerRevocationPublicKey
                        ├── cacerts
                        ├── keystore
                        │   └── key.pem
                        ├── signcerts
                        │   └── cert.pem
                        ├── tlscacerts
                        │   └── tls-10-250-250-10-7054.pem
                        ├── tlsintermediatecerts
                        │   └── tls-10-250-250-10-7054.pem
                        └── user
```

create orderer systemd unit file
```
cat <<EOF | sudo tee /etc/systemd/system/fabric-orderer.service
# Service definition for Hyperledger fabric orderer server
[Unit]
Description=hyperledger fabric-orderer server - Orderer for hyperledger fabric
Documentation=https://hyperledger-fabric.readthedocs.io/
Wants=network-online.target
After=network-online.target
[Service]
Type=simple
Restart=on-failure
Environment=CA_CFG_PATH=/etc/hyperledger/orderer
WorkingDirectory=/etc/hyperledger/orderer
ExecStart=/usr/local/bin/orderer start
[Install]
WantedBy=multi-user.target
EOF
```

start orderer service
```
sudo systemctl enable fabric-orderer.service
sudo systemctl start fabric-orderer.service
sudo systemctl status fabric-orderer.service
```
