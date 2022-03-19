# Setup Hyperledger TLS CA
This CA generates the certificates used to secure communications on Transport Layer Security (TLS). For this reason, this CA is often referred to as a “TLS CA”. These TLS certificates are attached to actions as a way of preventing “man in the middle” attacks. 

### Setup Database
for the sake of simplicity, we will deploy database on docker

ssh to the nodes
```shell
vagrant ssh dana-ca-server-0
```

create directory for docker related files
```shell
mkdir -p docker-compose/tls/
cat <<EOF | tee docker-compose/tls/docker-compose.yaml
version: "2"
services:
  tls-fabric-dana-ca-postgres:
    ports:
      - "5432:5432"
    container_name: tls-fabric-dana-ca-postgres
    environment:
      POSTGRES_DB: tls_fabric_dana_ca
      POSTGRES_USER: tls-fabric-dana-ca-user
      POSTGRES_PASSWORD: tls-fabric-dana-ca-password
    image: postgres:12-alpine
    restart: always
    volumes:
      - tls-fabric-dana-ca-postgres:/var/lib/postgresql/data
volumes:
  tls-fabric-dana-ca-postgres:
    driver: local
EOF
sudo docker-compose --file docker-compose/tls/docker-compose.yaml up --build -d 
```

### Setup Fabric CA
create config directory
```shell
sudo mkdir -p /etc/hyperledger/tls-fabric-ca-server
```

copy the needed certificate to the directory
```
sudo cp certificates/tls/intermediate/ca.crt /etc/hyperledger/tls-fabric-ca-server/
sudo cp certificates/tls/intermediate/ca.key /etc/hyperledger/tls-fabric-ca-server/
sudo cp certificates/tls/intermediate/fullchain.crt /etc/hyperledger/tls-fabric-ca-server/
sudo cp certificates/tls/client/tls-service/tls.crt /etc/hyperledger/tls-fabric-ca-server/
sudo cp certificates/tls/client/tls-service/tls.key /etc/hyperledger/tls-fabric-ca-server/
```

create TLS fabric CA configuration
```shell
cat <<EOF | sudo tee /etc/hyperledger/tls-fabric-ca-server/fabric-ca-server-config.yaml
# Service definition for Hyperledger fabric-ca server

version: 1.5.2
port: 7054

cors:
    enabled: false
    origins:
      - "*"

debug: false
crlsizelimit: 512000

tls:
  enabled: true
  certfile: /etc/hyperledger/tls-fabric-ca-server/tls.crt
  keyfile: /etc/hyperledger/tls-fabric-ca-server/tls.key
  clientauth:
    type: NoClientCert

ca:
  name: dana
  keyfile: /etc/hyperledger/tls-fabric-ca-server/ca.key
  certfile: /etc/hyperledger/tls-fabric-ca-server/ca.crt
  chainfile: /etc/hyperledger/tls-fabric-ca-server/fullchain.crt
  reenrollIgnoreCertExpiry: false

crl:
  expiry: 24h

registry:
  maxenrollments: -1

  identities:
     - name: admin@dana
       pass: adminpasswd
       type: client
       affiliation: ""
       attrs:
          hf.Registrar.Roles: "*"
          hf.Registrar.DelegateRoles: "*"
          hf.Revoker: true
          hf.IntermediateCA: true
          hf.GenCRL: true
          hf.Registrar.Attributes: "*"
          hf.AffiliationMgr: true

db:
  type: postgres
  datasource: host=localhost port=5432 user=tls-fabric-dana-ca-user password=tls-fabric-dana-ca-password dbname=tls_fabric_dana_ca sslmode=disable
  tls:
      enabled: false

ldap:
   enabled: false

affiliations:
   org1:
      - department1
      - department2
   org2:
      - department1

signing:
    default:
      usage:
        - digital signature
      expiry: 8760h
    profiles:
      ca:
         usage:
           - cert sign
           - crl sign
         expiry: 43800h
         caconstraint:
           isca: true
           maxpathlen: 0
      tls:
         usage:
            - signing
            - key encipherment
            - server auth
            - client auth
            - key agreement
         expiry: 8760h

csr:
   cn:
   keyrequest:
     algo: ecdsa
     size: 256
   names:
      - C: id
        ST: jakarta
        L:
        O: dana
        OU:
   hosts:
     - localhost
   ca:
      expiry: 131400h
      pathlength: 1

idemix:
  rhpoolsize: 1000
  nonceexpiration: 15s
  noncesweepinterval: 15m

bccsp:
    default: SW
    sw:
        hash: SHA2
        security: 256
        filekeystore:
            keystore: msp/keystore

cacount:
cafiles:

intermediate:
  parentserver:
    url:
    caname:

  enrollment:
    hosts:
    profile:
    label:

  tls:
    certfiles:
    client:
      certfile:
      keyfile:

cfg:
  identities:
    passwordattempts: 10

operations:
    listenAddress: 127.0.0.1:9443
    tls:
        enabled: false
        cert:
            file:
        key:
            file:
        clientAuthRequired: false
        clientRootCAs:
            files: []

metrics:
    provider: prometheus
EOF
```

create tls-fabric-ca-server.service systemd unit file
```shell
cat <<EOF | sudo tee /etc/systemd/system/tls-fabric-ca-server.service
# Service definition for Hyperledger fabric-ca server
[Unit]
Description=hyperledger tls fabric-ca server - Certificate Authority for TLS hyperledger fabric
Documentation=https://hyperledger-fabric-ca.readthedocs.io/
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
Restart=on-failure
Environment=FABRIC_CA_HOME=/etc/hyperledger/tls-fabric-ca-server
Environment=FABRIC_CA_SERVER_HOME=/etc/hyperledger/tls-fabric-ca-server
Environment=CA_CFG_PATH=/etc/hyperledger/tls-fabric-ca-server
ExecStart=/usr/local/bin/fabric-ca-server start

[Install]
WantedBy=multi-user.target
EOF
```

start TLS fabric CA server
```shell
sudo systemctl enable tls-fabric-ca-server.service
sudo systemctl start tls-fabric-ca-server.service
sudo systemctl status tls-fabric-ca-server.service
```

### Enrolling Admin Users
By default, TLS Fabric CA will create an admin identity
```
- name: admin@dana
  pass: adminpasswd
```
This admin identity is used for creating another identity for orderer and peer nodes. In order to create another identity, we need to first collect the certificate for this default identity

create directory for admin identity
```
mkdir -p identity/tls/admin/
mkdir -p identity/tls/admin/msp
```

copy TLS CA chain
```
cp certificates/tls/intermediate/fullchain.crt identity/tls/admin/
```

get admin identity certificate
```
fabric-ca-client enroll -d -u https://admin@dana:adminpasswd@10.250.252.10:7054 --tls.certfiles ${HOME}/identity/tls/admin/fullchain.crt --enrollment.profile tls --csr.hosts 'admin' --csr.names C=id,O=dana,ST=jakarta --mspdir ${HOME}/identity/tls/admin/msp
```

if we check, we will find
```
tree identity/
identity/
└── tls
    └── admin
        ├── fullchain.crt
        └── msp
            ├── cacerts
            ├── IssuerPublicKey
            ├── IssuerRevocationPublicKey
            ├── keystore
            │   └── 6216c173c326b4b738ea7e41cc47db5c4a73d42a2d4335ab01ec30f3ea4c69ef_sk
            ├── signcerts
            │   └── cert.pem
            ├── tlscacerts
            │   └── tls-10.250.252-10-7054.pem
            ├── tlsintermediatecerts
            │   └── tls-10.250.252-10-7054.pem
            └── user
```

### Creating Identity
now, let's use this to register another identity for orderer
```
fabric-ca-client register -d --id.name peer0@dana --id.secret peer0-dana-password -u https://10.250.252.10:7054  --id.type client --tls.certfiles ${HOME}/identity/tls/admin/fullchain.crt --mspdir ${HOME}/identity/tls/admin/msp
fabric-ca-client register -d --id.name peer1@dana --id.secret peer1-dana-password -u https://10.250.252.10:7054  --id.type client --tls.certfiles ${HOME}/identity/tls/admin/fullchain.crt --mspdir ${HOME}/identity/tls/admin/msp
```
