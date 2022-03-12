# Setup Hyperledger Orderer Servoce

***notes**: BI organization handle Orderer nodes*

### bi-orderer-0
Download orderer binary
```
wget https://github.com/hyperledger/fabric/releases/download/v2.4.3/hyperledger-fabric-linux-amd64-2.4.3.tar.gz
tar xzvf hyperledger-fabric-linux-amd64-2.4.3.tar.gz
sudo cp bin/* /usr/local/bin/
rm -rf bin/
rm -rf config/
rm -rf hyperledger-fabric-linux-amd64-2.4.3.tar.gz
```

Create directory for msp & tls certificate
```
sudo mkdir -p /etc/orderer/msp/cacerts
sudo mkdir -p /etc/orderer/msp/intermediatecerts
sudo mkdir -p /etc/orderer/msp/keystore
sudo mkdir -p /etc/orderer/msp/signcerts
sudo mkdir -p /etc/orderer/msp/tlscacerts
sudo mkdir -p /etc/orderer/msp/tlsintermediatecerts
sudo mkdir -p /etc/orderer/msp/operationscerts
sudo mkdir -p /etc/orderer/tls/
```

create directory for etcdraft
```
sudo mkdir -p /var/hyperledger/production/orderer/etcdraft/wal
sudo mkdir -p /var/hyperledger/production/orderer/etcdraft/snapshot
```

create directory for tls & enrollment certificate, we will copy certificate in this directory to `/etc/orderer`
```
mkdir -p certificates/orderer/tls/
mkdir -p certificates/orderer/enrollment/
```

get the certificate
```
fabric-ca-client enroll -d -u https://orderer0@bi:orderer0-bi-password@10.250.250.10:7054 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --enrollment.profile tls --csr.cn orderer --csr.hosts 'localhost,127.0.0.1,10.250.250.20' --csr.names C=id,O=bi,ST=jakarta --mspdir ${HOME}/certificates/orderer/tls
fabric-ca-client enroll -d -u https://orderer0@bi:orderer0-bi-password@10.250.250.10:7055 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --csr.names C=id,O=bi,ST=jakarta --mspdir ${HOME}/certificates/orderer/enrollment
```

check the certificate
```
tree ${HOME}/certificates/orderer/
/home/vagrant/certificates/orderer/
├── enrollment
│   ├── IssuerPublicKey
│   ├── IssuerRevocationPublicKey
│   ├── cacerts
│   │   └── 10-250-250-10-7055.pem
│   ├── intermediatecerts
│   │   └── 10-250-250-10-7055.pem
│   ├── keystore
│   │   └── 70aab89c9249a1e12277d7e0b2d8f060a5ef43c52d465b47b20b3aeae97aec22_sk
│   ├── signcerts
│   │   └── cert.pem
│   └── user
└── tls
    ├── IssuerPublicKey
    ├── IssuerRevocationPublicKey
    ├── cacerts
    ├── keystore
    │   └── a3fd42446cf4e72f7481296a146c72156e13bd11cc8f11f6ee0d8eb8d30289bb_sk
    ├── signcerts
    │   └── cert.pem
    ├── tlscacerts
    │   └── tls-10-250-250-10-7054.pem
    ├── tlsintermediatecerts
    │   └── tls-10-250-250-10-7054.pem
    └── user
```

copy the certificate to `/etc/orderer`
```
sudo cp certificates/orderer/enrollment/signcerts/cert.pem /etc/orderer/msp/signcerts/cert.pem
sudo cp certificates/orderer/enrollment/keystore/70aab89c9249a1e12277d7e0b2d8f060a5ef43c52d465b47b20b3aeae97aec22_sk /etc/orderer/msp/keystore/key.pem
sudo cp certificates/orderer/enrollment/cacerts/10-250-250-10-7055.pem /etc/orderer/msp/cacerts/root-ca.pem
sudo cp certificates/orderer/enrollment/intermediatecerts/10-250-250-10-7055.pem /etc/orderer/msp/intermediatecerts/intermediate-ca.pem

sudo cp certificates/orderer/tls/tlscacerts/tls-10-250-250-10-7054.pem /etc/orderer/msp/tlscacerts/root-ca.pem
sudo cp certificates/orderer/tls/tlsintermediatecerts/tls-10-250-250-10-7054.pem /etc/orderer/msp/tlsintermediatecerts/intermediate-ca.pem
sudo cp certificates/orderer/tls/signcerts/cert.pem /etc/orderer/tls/cert.pem
sudo cp certificates/orderer/tls/keystore/a3fd42446cf4e72f7481296a146c72156e13bd11cc8f11f6ee0d8eb8d30289bb_sk /etc/orderer/tls/key.pem
```

create `config.yaml` in msp directory, this is used to identify each roles from certificate OU, read more here https://hyperledger-fabric-ca.readthedocs.io/en/latest/deployguide/use_CA.html#nodeous
```
cat <<EOF | sudo tee /etc/orderer/msp/config.yaml
NodeOUs:
 Enable: true
 ClientOUIdentifier:
   Certificate: cacerts/root-ca.pem
   OrganizationalUnitIdentifier: client
 PeerOUIdentifier:
   Certificate: cacerts/root-ca.pem
   OrganizationalUnitIdentifier: peer
 AdminOUIdentifier:
   Certificate: cacerts/root-ca.pem
   OrganizationalUnitIdentifier: admin
 OrdererOUIdentifier:
   Certificate: cacerts/root-ca.pem
   OrganizationalUnitIdentifier: orderer
EOF
```

create orderer configuration
```
cat <<EOF | sudo tee /etc/orderer/orderer.yaml
General:
    ListenAddress: 0.0.0.0
    ListenPort: 7050
    TLS:
        Enabled: true
        PrivateKey: /etc/orderer/tls/key.pem
        Certificate: /etc/orderer/tls/cert.pem
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

    LocalMSPDir: /etc/orderer/msp/
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
Environment=CA_CFG_PATH=/etc/orderer
WorkingDirectory=/etc/orderer
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

### bi-orderer-1
Download orderer binary
```
wget https://github.com/hyperledger/fabric/releases/download/v2.4.3/hyperledger-fabric-linux-amd64-2.4.3.tar.gz
tar xzvf hyperledger-fabric-linux-amd64-2.4.3.tar.gz
sudo cp bin/* /usr/local/bin/
rm -rf bin/
rm -rf config/
rm -rf hyperledger-fabric-linux-amd64-2.4.3.tar.gz
```

Create directory for msp & tls certificate
```
sudo mkdir -p /etc/orderer/msp/cacerts
sudo mkdir -p /etc/orderer/msp/intermediatecerts
sudo mkdir -p /etc/orderer/msp/keystore
sudo mkdir -p /etc/orderer/msp/signcerts
sudo mkdir -p /etc/orderer/msp/tlscacerts
sudo mkdir -p /etc/orderer/msp/tlsintermediatecerts
sudo mkdir -p /etc/orderer/msp/operationscerts
sudo mkdir -p /etc/orderer/tls/
```

create directory for etcdraft
```
sudo mkdir -p /var/hyperledger/production/orderer/etcdraft/wal
sudo mkdir -p /var/hyperledger/production/orderer/etcdraft/snapshot
```

create directory for tls & enrollment certificate, we will copy certificate in this directory to `/etc/orderer`
```
mkdir -p certificates/orderer/tls/
mkdir -p certificates/orderer/enrollment/
```

get the certificate
```
fabric-ca-client enroll -d -u https://orderer1@bi:orderer1-bi-password@10.250.250.10:7054 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --enrollment.profile tls --csr.cn orderer --csr.hosts 'localhost,127.0.0.1,10.250.250.21' --csr.names C=id,O=bi,ST=jakarta --mspdir ${HOME}/certificates/orderer/tls
fabric-ca-client enroll -d -u https://orderer1@bi:orderer1-bi-password@10.250.250.10:7055 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --csr.names C=id,O=bi,ST=jakarta --mspdir ${HOME}/certificates/orderer/enrollment
```

check the certificate
```
tree ${HOME}/certificates/orderer/
/home/vagrant/certificates/orderer/
├── enrollment
│   ├── IssuerPublicKey
│   ├── IssuerRevocationPublicKey
│   ├── cacerts
│   │   └── 10-250-250-10-7055.pem
│   ├── intermediatecerts
│   │   └── 10-250-250-10-7055.pem
│   ├── keystore
│   │   └── 01dcbfb6aa05c0a3f48d4e430c63843540c3b9d00ba618d3e20b4e824ef851aa_sk
│   ├── signcerts
│   │   └── cert.pem
│   └── user
└── tls
    ├── IssuerPublicKey
    ├── IssuerRevocationPublicKey
    ├── cacerts
    ├── keystore
    │   └── a31109dd5f916b00c5a3e99eaf3ce8ab346acb8ed39333ba54c12e49c8081208_sk
    ├── signcerts
    │   └── cert.pem
    ├── tlscacerts
    │   └── tls-10-250-250-10-7054.pem
    ├── tlsintermediatecerts
    │   └── tls-10-250-250-10-7054.pem
    └── user
```

copy the certificate to `/etc/orderer`
```
sudo cp certificates/orderer/enrollment/signcerts/cert.pem /etc/orderer/msp/signcerts/cert.pem
sudo cp certificates/orderer/enrollment/keystore/01dcbfb6aa05c0a3f48d4e430c63843540c3b9d00ba618d3e20b4e824ef851aa_sk /etc/orderer/msp/keystore/key.pem
sudo cp certificates/orderer/enrollment/cacerts/10-250-250-10-7055.pem /etc/orderer/msp/cacerts/root-ca.pem
sudo cp certificates/orderer/enrollment/intermediatecerts/10-250-250-10-7055.pem /etc/orderer/msp/intermediatecerts/intermediate-ca.pem

sudo cp certificates/orderer/tls/tlscacerts/tls-10-250-250-10-7054.pem /etc/orderer/msp/tlscacerts/root-ca.pem
sudo cp certificates/orderer/tls/tlsintermediatecerts/tls-10-250-250-10-7054.pem /etc/orderer/msp/tlsintermediatecerts/intermediate-ca.pem
sudo cp certificates/orderer/tls/signcerts/cert.pem /etc/orderer/tls/cert.pem
sudo cp certificates/orderer/tls/keystore/a31109dd5f916b00c5a3e99eaf3ce8ab346acb8ed39333ba54c12e49c8081208_sk /etc/orderer/tls/key.pem
```

create `config.yaml` in msp directory, this is used to identify each roles from certificate OU, read more here https://hyperledger-fabric-ca.readthedocs.io/en/latest/deployguide/use_CA.html#nodeous
```
cat <<EOF | sudo tee /etc/orderer/msp/config.yaml
NodeOUs:
 Enable: true
 ClientOUIdentifier:
   Certificate: cacerts/root-ca.pem
   OrganizationalUnitIdentifier: client
 PeerOUIdentifier:
   Certificate: cacerts/root-ca.pem
   OrganizationalUnitIdentifier: peer
 AdminOUIdentifier:
   Certificate: cacerts/root-ca.pem
   OrganizationalUnitIdentifier: admin
 OrdererOUIdentifier:
   Certificate: cacerts/root-ca.pem
   OrganizationalUnitIdentifier: orderer
EOF
```

create orderer configuration
```
cat <<EOF | sudo tee /etc/orderer/orderer.yaml
General:
    ListenAddress: 0.0.0.0
    ListenPort: 7050
    TLS:
        Enabled: true
        PrivateKey: /etc/orderer/tls/key.pem
        Certificate: /etc/orderer/tls/cert.pem
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

    LocalMSPDir: /etc/orderer/msp/
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
Environment=CA_CFG_PATH=/etc/orderer
WorkingDirectory=/etc/orderer
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

### bi-orderer-2
Download orderer binary
```
wget https://github.com/hyperledger/fabric/releases/download/v2.4.3/hyperledger-fabric-linux-amd64-2.4.3.tar.gz
tar xzvf hyperledger-fabric-linux-amd64-2.4.3.tar.gz
sudo cp bin/* /usr/local/bin/
rm -rf bin/
rm -rf config/
rm -rf hyperledger-fabric-linux-amd64-2.4.3.tar.gz
```

Create directory for msp & tls certificate
```
sudo mkdir -p /etc/orderer/msp/cacerts
sudo mkdir -p /etc/orderer/msp/intermediatecerts
sudo mkdir -p /etc/orderer/msp/keystore
sudo mkdir -p /etc/orderer/msp/signcerts
sudo mkdir -p /etc/orderer/msp/tlscacerts
sudo mkdir -p /etc/orderer/msp/tlsintermediatecerts
sudo mkdir -p /etc/orderer/msp/operationscerts
sudo mkdir -p /etc/orderer/tls/
```

create directory for etcdraft
```
sudo mkdir -p /var/hyperledger/production/orderer/etcdraft/wal
sudo mkdir -p /var/hyperledger/production/orderer/etcdraft/snapshot
```

create directory for tls & enrollment certificate, we will copy certificate in this directory to `/etc/orderer`
```
mkdir -p certificates/orderer/tls/
mkdir -p certificates/orderer/enrollment/
```

get the certificate
```
fabric-ca-client enroll -d -u https://orderer2@bi:orderer2-bi-password@10.250.250.10:7054 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --enrollment.profile tls --csr.cn orderer --csr.hosts 'localhost,127.0.0.1,10.250.250.22' --csr.names C=id,O=bi,ST=jakarta --mspdir ${HOME}/certificates/orderer/tls
fabric-ca-client enroll -d -u https://orderer2@bi:orderer2-bi-password@10.250.250.10:7055 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --csr.names C=id,O=bi,ST=jakarta --mspdir ${HOME}/certificates/orderer/enrollment
```

check the certificate
```
tree ${HOME}/certificates/orderer/
/home/vagrant/certificates/orderer/
├── enrollment
│   ├── IssuerPublicKey
│   ├── IssuerRevocationPublicKey
│   ├── cacerts
│   │   └── 10-250-250-10-7055.pem
│   ├── intermediatecerts
│   │   └── 10-250-250-10-7055.pem
│   ├── keystore
│   │   └── 70aab89c9249a1e12277d7e0b2d8f060a5ef43c52d465b47b20b3aeae97aec22_sk
│   ├── signcerts
│   │   └── cert.pem
│   └── user
└── tls
    ├── IssuerPublicKey
    ├── IssuerRevocationPublicKey
    ├── cacerts
    ├── keystore
    │   └── a3fd42446cf4e72f7481296a146c72156e13bd11cc8f11f6ee0d8eb8d30289bb_sk
    ├── signcerts
    │   └── cert.pem
    ├── tlscacerts
    │   └── tls-10-250-250-10-7054.pem
    ├── tlsintermediatecerts
    │   └── tls-10-250-250-10-7054.pem
    └── user
```

copy the certificate to `/etc/orderer`
```
sudo cp certificates/orderer/enrollment/signcerts/cert.pem /etc/orderer/msp/signcerts/cert.pem
sudo cp certificates/orderer/enrollment/keystore/372b2644baa163216722eb13d21c79e617325b3cc27c0b736929c81b53d5a44e_sk /etc/orderer/msp/keystore/key.pem
sudo cp certificates/orderer/enrollment/cacerts/10-250-250-10-7055.pem /etc/orderer/msp/cacerts/root-ca.pem
sudo cp certificates/orderer/enrollment/intermediatecerts/10-250-250-10-7055.pem /etc/orderer/msp/intermediatecerts/intermediate-ca.pem

sudo cp certificates/orderer/tls/tlscacerts/tls-10-250-250-10-7054.pem /etc/orderer/msp/tlscacerts/root-ca.pem
sudo cp certificates/orderer/tls/tlsintermediatecerts/tls-10-250-250-10-7054.pem /etc/orderer/msp/tlsintermediatecerts/intermediate-ca.pem
sudo cp certificates/orderer/tls/signcerts/cert.pem /etc/orderer/tls/cert.pem
sudo cp certificates/orderer/tls/keystore/7feb6c8f48381b050e86602f2aa77e14a56cc062e366eeef0108a84f9481b1da_sk /etc/orderer/tls/key.pem
```

create `config.yaml` in msp directory, this is used to identify each roles from certificate OU, read more here https://hyperledger-fabric-ca.readthedocs.io/en/latest/deployguide/use_CA.html#nodeous
```
cat <<EOF | sudo tee /etc/orderer/msp/config.yaml
NodeOUs:
 Enable: true
 ClientOUIdentifier:
   Certificate: cacerts/root-ca.pem
   OrganizationalUnitIdentifier: client
 PeerOUIdentifier:
   Certificate: cacerts/root-ca.pem
   OrganizationalUnitIdentifier: peer
 AdminOUIdentifier:
   Certificate: cacerts/root-ca.pem
   OrganizationalUnitIdentifier: admin
 OrdererOUIdentifier:
   Certificate: cacerts/root-ca.pem
   OrganizationalUnitIdentifier: orderer
EOF
```

create orderer configuration
```
cat <<EOF | sudo tee /etc/orderer/orderer.yaml
General:
    ListenAddress: 0.0.0.0
    ListenPort: 7050
    TLS:
        Enabled: true
        PrivateKey: /etc/orderer/tls/key.pem
        Certificate: /etc/orderer/tls/cert.pem
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

    LocalMSPDir: /etc/orderer/msp/
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
Environment=CA_CFG_PATH=/etc/orderer
WorkingDirectory=/etc/orderer
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
