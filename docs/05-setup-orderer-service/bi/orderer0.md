### Bank Indonesia orderer0 node
ssh to the nodes
```shell
vagrant ssh bi-orderer-0
```

Create directory for msp & tls certificate directory for orderer0 node
```
sudo mkdir -p /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer0/tls
sudo mkdir -p /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer0/msp

sudo chmod 755 /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer0/tls
sudo chmod 755 /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer0/msp
```

create directory for etcdraft
```
sudo mkdir -p /var/hyperledger/production/orderer/etcdraft/wal
sudo mkdir -p /var/hyperledger/production/orderer/etcdraft/snapshot
```

get the msp certificate
```
sudo fabric-ca-client enroll -d -u https://orderer0@enrollment.bi.go.id:orderer0-password@10.250.250.10:7055 --tls.certfiles ${HOME}/organizations/OrdererOrganizations/bi/msp/tlsintermediatecerts/intermediate-cert.pem --csr.names C=id,O=bi,ST=jakarta --mspdir /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer0/msp
```

check the msp certificate
```
sudo tree /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer0/msp
/etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer0/msp
├── IssuerPublicKey
├── IssuerRevocationPublicKey
├── cacerts
│   └── 10-250-250-10-7055.pem
├── intermediatecerts
│   └── 10-250-250-10-7055.pem
├── keystore
│   └── 9f3576c7bdde80a62de2638c7eac0af1fd00f6e7f243eade940148eecae4dea0_sk
├── signcerts
│   └── cert.pem
└── user
```

get the TLS certificate
```
sudo fabric-ca-client enroll -d -u https://orderer0@tls.bi.go.id:orderer0-password@10.250.250.10:7054 --tls.certfiles ${HOME}/organizations/OrdererOrganizations/bi/msp/tlsintermediatecerts/intermediate-cert.pem --enrollment.profile tls --csr.cn orderer --csr.hosts 'localhost,127.0.0.1,10.250.250.20' --csr.names C=id,O=bi,ST=jakarta --mspdir /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer0/tls
```

check the TLS certificate
```
sudo tree /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer0/tls
/etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer0/tls
├── IssuerPublicKey
├── IssuerRevocationPublicKey
├── cacerts
├── keystore
│   └── b917ccfdc64a8945f80652e33742760e17cc49828173890e2094178e72a85856_sk
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
sudo mv /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer0/tls/keystore/b917ccfdc64a8945f80652e33742760e17cc49828173890e2094178e72a85856_sk /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer0/tls/keystore/key.pem
```

create `config.yaml` in msp directory, this is used to identify each roles from certificate OU, read more here https://hyperledger-fabric-ca.readthedocs.io/en/latest/deployguide/use_CA.html#nodeous
```
cat <<EOF | sudo tee /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer0/msp/config.yaml
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
        PrivateKey: /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer0/tls/keystore/key.pem
        Certificate: /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer0/tls/signcerts/cert.pem
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

    LocalMSPDir: /etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer0/msp/
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
                └── orderer0
                    ├── msp
                    │   ├── IssuerPublicKey
                    │   ├── IssuerRevocationPublicKey
                    │   ├── cacerts
                    │   │   └── 10-250-250-10-7055.pem
                    │   ├── config.yaml
                    │   ├── intermediatecerts
                    │   │   └── 10-250-250-10-7055.pem
                    │   ├── keystore
                    │   │   └── 9f3576c7bdde80a62de2638c7eac0af1fd00f6e7f243eade940148eecae4dea0_sk
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
