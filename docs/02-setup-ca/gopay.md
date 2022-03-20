# Setup Root & Intermediate CA
If you already have root CA available, you can directly create intermediate CA from it. In this tutorial, we will create two certificate.

### Root CA
ssh to the nodes
```shell
vagrant ssh gopay-ca-server-0
```

create root CA directory
```shell
sudo mkdir -p /etc/secrets/gopay/ca/
sudo mkdir -p /etc/secrets/gopay/tlsca/
```

create enrollment root CA
```shell
sudo step certificate create root-ledger-gopay-ca /etc/secrets/gopay/ca/root-cert.pem /etc/secrets/gopay/ca/root-key.pem --profile root-ca --kty EC --no-password --insecure --not-after 87600h --san root.enrollment.gopay.co.id
```

create TLS root CA
```shell
sudo step certificate create root-ledger-tls-gopay-ca /etc/secrets/gopay/tlsca/root-cert.pem /etc/secrets/gopay/tlsca/root-key.pem --profile root-ca --kty EC --no-password --insecure --not-after 87600h --san root.tls.gopay.co.id
```

### Intermediate CA
Create enrollment Intermediate CA
```shell
sudo step certificate create intermediate-ledger-gopay-ca /etc/secrets/gopay/ca/intermediate-cert.pem /etc/secrets/gopay/ca/intermediate-key.pem --profile intermediate-ca --kty EC --ca /etc/secrets/gopay/ca/root-cert.pem --ca-key /etc/secrets/gopay/ca/root-key.pem --no-password --insecure --not-after 43800h --san intermediate.enrollment.gopay.co.id
sudo step certificate bundle /etc/secrets/gopay/ca/intermediate-cert.pem /etc/secrets/gopay/ca/root-cert.pem /etc/secrets/gopay/ca/intermediate-bundle.pem
```

Create TLS Intermediate CA
```shell
sudo step certificate create intermediate-ledger-tls-gopay-ca /etc/secrets/gopay/tlsca/intermediate-cert.pem /etc/secrets/gopay/tlsca/intermediate-key.pem --profile intermediate-ca --kty EC --ca /etc/secrets/gopay/tlsca/root-cert.pem --ca-key /etc/secrets/gopay/tlsca/root-key.pem --no-password --insecure --not-after 43800h --san intermediate.tls.gopay.co.id
sudo step certificate bundle /etc/secrets/gopay/tlsca/intermediate-cert.pem /etc/secrets/gopay/tlsca/root-cert.pem /etc/secrets/gopay/tlsca/intermediate-bundle.pem
```

starting for now, let's not do anything with this certificate keys, we must not do any operations to certificate keys.

### Client CA
we need to create client for TLS CA, this client certificate will be used to contact to enrollment Fabric CA service as well as TLS Fabric CA service for futher generation of certificate.

create client certificate directory
```shell
sudo mkdir -p /etc/secrets/gopay/services/tls-fabric-ca-server/tls/
sudo mkdir -p /etc/secrets/gopay/services/enrollment-fabric-ca-server/tls/
```

```shell
sudo step certificate create tls-fabric-ca-server-ledger-tls-gopay-ca /etc/secrets/gopay/services/tls-fabric-ca-server/tls/cert.pem /etc/secrets/gopay/services/tls-fabric-ca-server/tls/key.pem --profile leaf --ca /etc/secrets/gopay/tlsca/intermediate-cert.pem --ca-key /etc/secrets/gopay/tlsca/intermediate-key.pem --kty EC --no-password --insecure --not-after 47600h --san ca-server.tls.gopay.co.id --san 10.250.251.10 --san localhost --san 127.0.0.1
sudo step certificate bundle /etc/secrets/gopay/services/tls-fabric-ca-server/tls/cert.pem /etc/secrets/gopay/tlsca/intermediate-cert.pem /etc/secrets/gopay/services/tls-fabric-ca-server/tls/bundle.pem

sudo step certificate create enrollment-fabric-ca-server-ledger-tls-gopay-ca /etc/secrets/gopay/services/enrollment-fabric-ca-server/tls/cert.pem /etc/secrets/gopay/services/enrollment-fabric-ca-server/tls/key.pem --profile leaf --ca /etc/secrets/gopay/tlsca/intermediate-cert.pem --ca-key /etc/secrets/gopay/tlsca/intermediate-key.pem --kty EC --no-password --insecure --not-after 47600h --san ca-server.enrollment.gopay.co.id --san 10.250.251.10 --san localhost --san 127.0.0.1
sudo step certificate bundle /etc/secrets/gopay/services/enrollment-fabric-ca-server/tls/cert.pem /etc/secrets/gopay/tlsca/intermediate-cert.pem /etc/secrets/gopay/services/enrollment-fabric-ca-server/tls/bundle.pem
```

### Validate Root, Intermediate, and Client Certificate
If we check, we will have this certificates
```
sudo tree /etc/secrets/
/etc/secrets/
└── gopay
    ├── ca
    │   ├── intermediate-bundle.pem
    │   ├── intermediate-cert.pem
    │   ├── intermediate-key.pem
    │   ├── root-cert.pem
    │   └── root-key.pem
    ├── services
    │   ├── enrollment-fabric-ca-server
    │   │   └── tls
    │   │       ├── bundle.pem
    │   │       ├── cert.pem
    │   │       └── key.pem
    │   └── tls-fabric-ca-server
    │       └── tls
    │           ├── bundle.pem
    │           ├── cert.pem
    │           └── key.pem
    └── tlsca
        ├── intermediate-bundle.pem
        ├── intermediate-cert.pem
        ├── intermediate-key.pem
        ├── root-cert.pem
        └── root-key.pem
```
starting for now, let's not do anything with this certificate keys, we must not do any operations to certificate keys.

### Organizations MSP
create organizations MSP directory
```
mkdir -p organizations/PeerOrganizations/gopay/msp/cacerts/
mkdir -p organizations/PeerOrganizations/gopay/msp/tlscacerts/
mkdir -p organizations/PeerOrganizations/gopay/msp/intermediatecerts/
mkdir -p organizations/PeerOrganizations/gopay/msp/tlsintermediatecerts/
```

copy the certificate
```
sudo cp /etc/secrets/gopay/ca/root-cert.pem organizations/PeerOrganizations/gopay/msp/cacerts/
sudo cp /etc/secrets/gopay/ca/intermediate-cert.pem organizations/PeerOrganizations/gopay/msp/intermediatecerts/
sudo cp /etc/secrets/gopay/tlsca/root-cert.pem organizations/PeerOrganizations/gopay/msp/tlscacerts/
sudo cp /etc/secrets/gopay/tlsca/intermediate-cert.pem organizations/PeerOrganizations/gopay/msp/tlsintermediatecerts/
```

change the owners
```
sudo chown -R vagrant:vagrant organizations/PeerOrganizations/gopay/
chmod -R 744 organizations/PeerOrganizations/gopay/
```

setup NodeOU
```
cat <<EOF | tee organizations/PeerOrganizations/gopay/msp/config.yaml
NodeOUs:
 Enable: true
 ClientOUIdentifier:
   Certificate: intermediatecerts/intermediate-cert.pem
   OrganizationalUnitIdentifier: client
 PeerOUIdentifier:
   Certificate: intermediatecerts/intermediate-cert.pem
   OrganizationalUnitIdentifier: peer
 AdminOUIdentifier:
   Certificate: intermediatecerts/intermediate-cert.pem
   OrganizationalUnitIdentifier: admin
 OrdererOUIdentifier:
   Certificate: intermediatecerts/intermediate-cert.pem
   OrganizationalUnitIdentifier: orderer
EOF
```

If we check, we will have this certificates
```
tree organizations/
organizations/
└── PeerOrganizations
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

Copy enrollment & TLS intermediate fullchain to each peer nodes
```
ssh vagrant@10.250.251.20 "mkdir -p ~/organizations/PeerOrganizations/gopay/"
scp -r organizations/PeerOrganizations/gopay/msp vagrant@10.250.251.20:~/organizations/PeerOrganizations/gopay/

ssh vagrant@10.250.251.21 "mkdir -p ~/organizations/PeerOrganizations/gopay/"
scp -r organizations/PeerOrganizations/gopay/msp vagrant@10.250.251.21:~/organizations/PeerOrganizations/gopay/
```
