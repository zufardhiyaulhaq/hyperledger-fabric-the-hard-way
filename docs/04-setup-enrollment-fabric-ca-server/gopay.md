# Setup Hyperledger Enrollment CA
this CA is used to generate (through a process called “enrollment”) the certificates of the admin of an organization, the MSP of that organization, and any nodes owned by that organization. This CA will also generate the certificates for any additional users. Because of its role in “enrolling” identities, this CA is sometimes called the “enrollment CA” or the “ecert CA”.


### Setup Database
ssh to the nodes
```shell
vagrant ssh gopay-ca-server-0
```

for the sake of simplicity, we will deploy database on docker

create directory for docker related files
```shell
mkdir -p docker-compose/enrollment/
cat <<EOF | tee docker-compose/enrollment/docker-compose.yaml
version: "2"
services:
  enrollment-fabric-gopay-ca-postgres:
    ports:
      - "5433:5432"
    container_name: enrollment-fabric-gopay-ca-postgres
    environment:
      POSTGRES_DB: enrollment_fabric_gopay_ca
      POSTGRES_USER: enrollment-fabric-gopay-ca-user
      POSTGRES_PASSWORD: enrollment-fabric-gopay-ca-password
    image: postgres:12-alpine
    restart: always
    volumes:
      - enrollment-fabric-gopay-ca-postgres:/var/lib/postgresql/data
volumes:
  enrollment-fabric-gopay-ca-postgres:
    driver: local
EOF
sudo docker-compose --file docker-compose/enrollment/docker-compose.yaml up --build -d 
```

### Setup Fabric CA
create config directory
```shell
sudo mkdir -p /etc/hyperledger/enrollment-fabric-ca-server
```

copy the needed certificate to the directory
```
sudo cp certificates/enrollment/intermediate/ca.crt /etc/hyperledger/enrollment-fabric-ca-server/
sudo cp certificates/enrollment/intermediate/ca.key /etc/hyperledger/enrollment-fabric-ca-server/
sudo cp certificates/enrollment/intermediate/fullchain.crt /etc/hyperledger/enrollment-fabric-ca-server/
sudo cp certificates/tls/client/enrollment-service/enrollment.crt /etc/hyperledger/enrollment-fabric-ca-server/tls.crt
sudo cp certificates/tls/client/enrollment-service/enrollment.key /etc/hyperledger/enrollment-fabric-ca-server/tls.key
```

create TLS fabric CA configuration
```shell
cat <<EOF | sudo tee /etc/hyperledger/enrollment-fabric-ca-server/fabric-ca-server-config.yaml
# Service definition for Hyperledger fabric-ca server

version: 1.5.2
port: 7055

cors:
    enabled: false
    origins:
      - "*"

debug: false
crlsizelimit: 512000

tls:
  enabled: true
  certfile: /etc/hyperledger/enrollment-fabric-ca-server/tls.crt
  keyfile: /etc/hyperledger/enrollment-fabric-ca-server/tls.key
  clientauth:
    type: NoClientCert

ca:
  name: gopay
  keyfile: /etc/hyperledger/enrollment-fabric-ca-server/ca.key
  certfile: /etc/hyperledger/enrollment-fabric-ca-server/ca.crt
  chainfile: /etc/hyperledger/enrollment-fabric-ca-server/fullchain.crt
  reenrollIgnoreCertExpiry: false

crl:
  expiry: 24h

registry:
  maxenrollments: -1

  identities:
     - name: admin@gopay
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
  datasource: host=localhost port=5433 user=enrollment-fabric-gopay-ca-user password=enrollment-fabric-gopay-ca-password dbname=enrollment_fabric_gopay_ca sslmode=disable
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
        O: gopay
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
    listenAddress: 127.0.0.1:10443
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

create enrollment-fabric-ca-server.service systemd unit file
```shell
cat <<EOF | sudo tee /etc/systemd/system/enrollment-fabric-ca-server.service
# Service definition for Hyperledger fabric-ca server
[Unit]
Description=hyperledger enrollment fabric-ca server - Certificate Authority for enrollment hyperledger fabric
Documentation=https://hyperledger-fabric-ca.readthedocs.io/
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
Restart=on-failure
Environment=FABRIC_CA_HOME=/etc/hyperledger/enrollment-fabric-ca-server
Environment=FABRIC_CA_SERVER_HOME=/etc/hyperledger/enrollment-fabric-ca-server
Environment=CA_CFG_PATH=/etc/hyperledger/enrollment-fabric-ca-server
ExecStart=/usr/local/bin/fabric-ca-server start

[Install]
WantedBy=multi-user.target
EOF
```

start TLS fabric CA server
```shell
sudo systemctl enable enrollment-fabric-ca-server.service
sudo systemctl start enrollment-fabric-ca-server.service
sudo systemctl status enrollment-fabric-ca-server.service
```

### Enrolling Admin Users
By default, Encrollment Fabric CA will create an admin identity
```
- name: admin@gopay
  pass: adminpasswd
```
This admin identity is used for creating another identity for orderer and peer nodes. In order to create another identity, we need to first collect the certificate for this default identity

create directory for admin identity
```
mkdir -p identity/enrollment/admin/
mkdir -p identity/enrollment/admin/msp
```

copy TLS CA chain
```
cp certificates/tls/intermediate/fullchain.crt identity/enrollment/admin/tls-fullchain.crt
```

get admin identity certificate
```
fabric-ca-client enroll -d -u https://admin@gopay:adminpasswd@10.250.251.10:7055 --tls.certfiles ${HOME}/identity/enrollment/admin/tls-fullchain.crt --enrollment.profile tls --csr.hosts 'admin' --csr.names C=id,O=gopay,ST=jakarta --mspdir ${HOME}/identity/enrollment/admin/msp
```

if we check, we will find
```
tree identity/
identity/
├── enrollment
│   └── admin
│       ├── msp
│       │   ├── IssuerPublicKey
│       │   ├── IssuerRevocationPublicKey
│       │   ├── cacerts
│       │   ├── keystore
│       │   │   └── a1b0184cbf48a30c83d961d419abeb717ab9d67d2136a5db518574256a23267c_sk
│       │   ├── signcerts
│       │   │   └── cert.pem
│       │   ├── tlscacerts
│       │   │   └── tls-10-250-251-10-7055.pem
│       │   ├── tlsintermediatecerts
│       │   │   └── tls-10-250-251-10-7055.pem
│       │   └── user
│       └── tls-fullchain.crt
```

### Creating Identity
now, let's use this to register another identity for peers
```
fabric-ca-client register -d --id.name peer0@gopay --id.secret peer0-gopay-password -u https://10.250.251.10:7055  --id.type peer --tls.certfiles ${HOME}/identity/enrollment/admin/tls-fullchain.crt --mspdir ${HOME}/identity/enrollment/admin/msp
fabric-ca-client register -d --id.name peer1@gopay --id.secret peer1-gopay-password -u https://10.250.251.10:7055  --id.type peer --tls.certfiles ${HOME}/identity/enrollment/admin/tls-fullchain.crt --mspdir ${HOME}/identity/enrollment/admin/msp
```
