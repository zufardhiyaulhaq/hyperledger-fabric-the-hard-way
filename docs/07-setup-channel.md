# Setup channel
To setup a channel, we need to generate channel genesis block. To generate this channel genesis block, we need to get this configuration for all organizations:
- Enrollment Root CA certificate
- Enrollment Intermediate CA certificate
- TLS Root CA certificate
- TLS Intermediate CA certificate

and we need to get TLS certificate for all orderer nodes. Let's use bi-ca-server-0 to gather all of the requirements.

*Remember! in real environment, each organization must collaborate each other to share their certificate*

### Generate genesis block
ssh to the nodes
```shell
vagrant ssh bi-ca-server-0
```

currently, BI have only orderer organization MSP
```
tree organizations
organizations
├── OrdererOrganizations
│   └── bi
│       ├── msp
│       │   ├── cacerts
│       │   │   └── root-cert.pem
│       │   ├── config.yaml
│       │   ├── intermediatecerts
│       │   │   └── intermediate-cert.pem
│       │   ├── tlscacerts
│       │   │   └── root-cert.pem
│       │   └── tlsintermediatecerts
│       │       └── intermediate-cert.pem
```

create directory for peer organizations
```shell
mkdir -p organizations/PeerOrganizations/dana
mkdir -p organizations/PeerOrganizations/gopay
```

get certificate GoPay. 
*In the real environment, they will send the certificate to us via email or any secure communication, but let's just get from their CA server directly*
```shell
sshpass -p "vagrant" ssh-copy-id -o StrictHostKeyChecking=no vagrant@10.250.251.10

scp -r vagrant@10.250.251.10:~/organizations/PeerOrganizations/gopay/msp/ organizations/PeerOrganizations/gopay/msp/
```

get certificate Dana. 
*In the real environment, they will send the certificate to us via email or any secure communication, but let's just get from their CA server directly*
```shell
sshpass -p "vagrant" ssh-copy-id -o StrictHostKeyChecking=no vagrant@10.250.252.10

scp -r vagrant@10.250.252.10:~/organizations/PeerOrganizations/dana/msp/ organizations/PeerOrganizations/dana/msp/
```

create directory to host TLS certificate for each orderer service
```
mkdir -p organizations/OrdererOrganizations/bi/orderers/orderer0/tls/signcerts/
mkdir -p organizations/OrdererOrganizations/bi/orderers/orderer1/tls/signcerts/
mkdir -p organizations/OrdererOrganizations/bi/orderers/orderer2/tls/signcerts/
```
We already create TLS certificate for each orderer service, it's should located in `~/certificates/orderer/tls/signcerts/cert.pem`
```shell
scp vagrant@10.250.250.20:/etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer0/tls/signcerts/cert.pem organizations/OrdererOrganizations/bi/orderers/orderer0/tls/signcerts/
scp vagrant@10.250.250.21:/etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer1/tls/signcerts/cert.pem organizations/OrdererOrganizations/bi/orderers/orderer1/tls/signcerts/
scp vagrant@10.250.250.22:/etc/hyperledger/orderer/organizations/OrdererOrganizations/bi/orderers/orderer2/tls/signcerts/cert.pem organizations/OrdererOrganizations/bi/orderers/orderer2/tls/signcerts/
```

let's check all of the certificate
```
tree organizations
organizations
├── OrdererOrganizations
│   └── bi
│       ├── msp
│       │   ├── cacerts
│       │   │   └── root-cert.pem
│       │   ├── config.yaml
│       │   ├── intermediatecerts
│       │   │   └── intermediate-cert.pem
│       │   ├── tlscacerts
│       │   │   └── root-cert.pem
│       │   └── tlsintermediatecerts
│       │       └── intermediate-cert.pem
│       ├── orderers
│       │   ├── orderer0
│       │   │   └── tls
│       │   │       └── signcerts
│       │   │           └── cert.pem
│       │   ├── orderer1
│       │   │   └── tls
│       │   │       └── signcerts
│       │   │           └── cert.pem
│       │   └── orderer2
│       │       └── tls
│       │           └── signcerts
│       │               └── cert.pem
│       └── users
└── PeerOrganizations
    ├── dana
    │   └── msp
    │       ├── cacerts
    │       │   └── root-cert.pem
    │       ├── config.yaml
    │       ├── intermediatecerts
    │       │   └── intermediate-cert.pem
    │       ├── tlscacerts
    │       │   └── root-cert.pem
    │       └── tlsintermediatecerts
    │           └── intermediate-cert.pem
    └── gopay
        └── msp
            ├── cacerts
            │   └── root-cert.pem
            ├── config.yaml
            ├── intermediatecerts
            │   └── intermediate-cert.pem
            ├── tlscacerts
            │   └── root-cert.pem
            └── tlsintermediatecerts
                └── intermediate-cert.pem
```

create configuration
```shell
cat <<EOF | tee configtx.yaml
Organizations:
  - &BankIndonesiaOrg
      Name: bi
      ID: bi
      MSPDir: organizations/OrdererOrganizations/bi/msp
      Policies:
        Readers:
          Type: Signature
          Rule: "OR('bi.member')"
        Writers:
          Type: Signature
          Rule: "OR('bi.member')"
        Admins:
          Type: Signature
          Rule: "OR('bi.admin')"
  - &GoPayOrg
      Name: gopay
      ID: gopay
      MSPDir: organizations/PeerOrganizations/gopay/msp
      Policies:
        Readers:
          Type: Signature
          Rule: "OR('gopay.admin', 'gopay.peer', 'gopay.client')"
        Writers:
          Type: Signature
          Rule: "OR('gopay.admin', 'gopay.client')"
        Admins:
          Type: Signature
          Rule: "OR('gopay.admin')"
        Endorsement:
          Type: Signature
          Rule: "OR('gopay.peer')"
      AnchorPeers:
          - Host: 10.250.251.20
            Port: 7051
  - &DANAOrg
      Name: dana
      ID: dana
      MSPDir: organizations/PeerOrganizations/dana/msp
      Policies:
        Readers:
          Type: Signature
          Rule: "OR('dana.admin', 'dana.peer', 'dana.client')"
        Writers:
          Type: Signature
          Rule: "OR('dana.admin', 'dana.client')"
        Admins:
          Type: Signature
          Rule: "OR('dana.admin')"
        Endorsement:
          Type: Signature
          Rule: "OR('dana.peer')"
      AnchorPeers:
          - Host: 10.250.252.20
            Port: 7051

Capabilities:
    Channel: &ChannelCapabilities
        V2_0: true
    Orderer: &OrdererCapabilities
        V2_0: true
    Application: &ApplicationCapabilities
        V2_0: true
  
Channel: &QRISChannel
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"
    Capabilities:
        <<: *ChannelCapabilities

Orderer: &QRISOrderer
  BatchTimeout: 2s
  BatchSize:
    MaxMessageCount: 500
    AbsoluteMaxBytes: 10 MB
    PreferredMaxBytes: 2 MB
  MaxChannels: 0
  OrdererType: etcdraft
  EtcdRaft:
    Consenters:
    - Host: 10.250.250.20
      Port: 7050
      ClientTLSCert: organizations/OrdererOrganizations/bi/orderers/orderer0/tls/signcerts/cert.pem
      ServerTLSCert: organizations/OrdererOrganizations/bi/orderers/orderer0/tls/signcerts/cert.pem
    - Host: 10.250.250.21
      Port: 7050
      ClientTLSCert: organizations/OrdererOrganizations/bi/orderers/orderer1/tls/signcerts/cert.pem
      ServerTLSCert: organizations/OrdererOrganizations/bi/orderers/orderer1/tls/signcerts/cert.pem
    - Host: 10.250.250.22
      Port: 7050
      ClientTLSCert: organizations/OrdererOrganizations/bi/orderers/orderer2/tls/signcerts/cert.pem
      ServerTLSCert: organizations/OrdererOrganizations/bi/orderers/orderer2/tls/signcerts/cert.pem
  Addresses:
    - 10.250.250.20:7050
    - 10.250.250.21:7050
    - 10.250.250.22:7050
  Policies:
    Readers:
      Type: ImplicitMeta
      Rule: "ANY Readers"
    Writers:
      Type: ImplicitMeta
      Rule: "ANY Writers"
    Admins:
      Type: ImplicitMeta
      Rule: "MAJORITY Admins"
    BlockValidation:
      Type: ImplicitMeta
      Rule: "ANY Writers"
  Capabilities:
    <<: *OrdererCapabilities

Application: &QRISApplication
  Policies:
      Readers:
          Type: ImplicitMeta
          Rule: "ANY Readers"
      Writers:
          Type: ImplicitMeta
          Rule: "ANY Writers"
      Admins:
          Type: ImplicitMeta
          Rule: "MAJORITY Admins"
      LifecycleEndorsement:
          Type: ImplicitMeta
          Rule: "MAJORITY Endorsement"
      Endorsement:
          Type: ImplicitMeta
          Rule: "MAJORITY Endorsement"
  Capabilities:
    <<: *ApplicationCapabilities

Profiles:
  QRIS:
    <<: *QRISChannel
    Orderer:
      <<: *QRISOrderer
      Organizations:
        - <<: *BankIndonesiaOrg
      
    Application:
      <<: *QRISApplication
      Organizations:
      - <<: *GoPayOrg
      - <<: *DANAOrg
EOF
```

generate genesis block
```
configtxgen -profile QRIS -outputBlock genesis_block.pb -channelID qris

2022-03-20 11:12:45.412 UTC 0001 INFO [common.tools.configtxgen] main -> Loading configuration
2022-03-20 11:12:45.424 UTC 0002 INFO [common.tools.configtxgen.localconfig] completeInitialization -> orderer type: etcdraft
2022-03-20 11:12:45.426 UTC 0003 INFO [common.tools.configtxgen.localconfig] completeInitialization -> Orderer.EtcdRaft.Options unset, setting to tick_interval:"500ms" election_tick:10 heartbeat_tick:1 max_inflight_blocks:5 snapshot_interval_size:16777216
2022-03-20 11:12:45.426 UTC 0004 INFO [common.tools.configtxgen.localconfig] Load -> Loaded configuration: configtx.yaml
2022-03-20 11:12:45.428 UTC 0005 INFO [common.tools.configtxgen] doOutputBlock -> Generating genesis block
2022-03-20 11:12:45.428 UTC 0006 INFO [common.tools.configtxgen] doOutputBlock -> Creating application channel genesis block
2022-03-20 11:12:45.429 UTC 0007 INFO [common.tools.configtxgen] doOutputBlock -> Writing genesis block
```

copy this to each orderer service
```
scp genesis_block.pb vagrant@10.250.250.20:~/
scp genesis_block.pb vagrant@10.250.250.21:~/
scp genesis_block.pb vagrant@10.250.250.22:~/
```

### Orderer services
***notes**: BI organization handle Orderer nodes*

- [Bank Indonesia Orderer0](07-setup-channel/orderer/bi/orderer0.md)
- [Bank Indonesia Orderer1](07-setup-channel/orderer/bi/orderer1.md)
- [Bank Indonesia Orderer2](07-setup-channel/orderer/bi/orderer2.md)

### Peer Services

- [DANA Peer0](07-setup-channel/peer/dana/peer0.md)
- [DANA Peer1](07-setup-channel/peer/dana/peer1.md)
- [GoPay Peer0](07-setup-channel/peer/gopay/peer0.md)
- [GoPay Peer1](07-setup-channel/peer/gopay/peer1.md)
