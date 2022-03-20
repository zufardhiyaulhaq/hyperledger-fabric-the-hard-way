# Setup Root & Intermediate CA
If you already have root CA available, you can directly create intermediate CA from it. In this tutorial, we will create two certificate.

### Root CA
ssh to the nodes
```shell
vagrant ssh dana-ca-server-0
```

create root CA directory
```shell
sudo mkdir -p /etc/secrets/dana/ca/
sudo mkdir -p /etc/secrets/dana/tlsca/
```

create enrollment root CA
```shell
sudo step certificate create root-ledger-dana-ca /etc/secrets/dana/ca/root-cert.pem /etc/secrets/dana/ca/root-key.pem --profile root-ca --kty EC --no-password --insecure --not-after 87600h --san root.enrollment.dana.id
```

create TLS root CA
```shell
sudo step certificate create root-ledger-tls-dana-ca /etc/secrets/dana/tlsca/root-cert.pem /etc/secrets/dana/tlsca/root-key.pem --profile root-ca --kty EC --no-password --insecure --not-after 87600h --san root.tls.dana.id
```

### Intermediate CA
Create enrollment Intermediate CA
```shell
sudo step certificate create intermediate-ledger-dana-ca /etc/secrets/dana/ca/intermediate-cert.pem /etc/secrets/dana/ca/intermediate-key.pem --profile intermediate-ca --kty EC --ca /etc/secrets/dana/ca/root-cert.pem --ca-key /etc/secrets/dana/ca/root-key.pem --no-password --insecure --not-after 43800h --san intermediate.enrollment.dana.id
sudo step certificate bundle /etc/secrets/dana/ca/intermediate-cert.pem /etc/secrets/dana/ca/root-cert.pem /etc/secrets/dana/ca/intermediate-bundle.pem
```

Create TLS Intermediate CA
```shell
sudo step certificate create intermediate-ledger-tls-dana-ca /etc/secrets/dana/tlsca/intermediate-cert.pem /etc/secrets/dana/tlsca/intermediate-key.pem --profile intermediate-ca --kty EC --ca /etc/secrets/dana/tlsca/root-cert.pem --ca-key /etc/secrets/dana/tlsca/root-key.pem --no-password --insecure --not-after 43800h --san intermediate.tls.dana.id
sudo step certificate bundle /etc/secrets/dana/tlsca/intermediate-cert.pem /etc/secrets/dana/tlsca/root-cert.pem /etc/secrets/dana/tlsca/intermediate-bundle.pem
```

starting for now, let's not do anything with this certificate keys, we must not do any operations to certificate keys.

### Client CA
we need to create client for TLS CA, this client certificate will be used to contact to enrollment Fabric CA service as well as TLS Fabric CA service for futher generation of certificate.

create client certificate directory
```shell
sudo mkdir -p /etc/secrets/dana/services/tls-fabric-ca-server/tls/
sudo mkdir -p /etc/secrets/dana/services/enrollment-fabric-ca-server/tls/
```

```shell
sudo step certificate create tls-fabric-ca-server-ledger-tls-dana-ca /etc/secrets/dana/services/tls-fabric-ca-server/tls/cert.pem /etc/secrets/dana/services/tls-fabric-ca-server/tls/key.pem --profile leaf --ca /etc/secrets/dana/tlsca/intermediate-cert.pem --ca-key /etc/secrets/dana/tlsca/intermediate-key.pem --kty EC --no-password --insecure --not-after 47600h --san ca-server.tls.dana.id --san 10.250.252.10 --san localhost --san 127.0.0.1
sudo step certificate bundle /etc/secrets/dana/services/tls-fabric-ca-server/tls/cert.pem /etc/secrets/dana/tlsca/intermediate-cert.pem /etc/secrets/dana/services/tls-fabric-ca-server/tls/bundle.pem

sudo step certificate create enrollment-fabric-ca-server-ledger-tls-dana-ca /etc/secrets/dana/services/enrollment-fabric-ca-server/tls/cert.pem /etc/secrets/dana/services/enrollment-fabric-ca-server/tls/key.pem --profile leaf --ca /etc/secrets/dana/tlsca/intermediate-cert.pem --ca-key /etc/secrets/dana/tlsca/intermediate-key.pem --kty EC --no-password --insecure --not-after 47600h --san ca-server.enrollment.dana.id --san 10.250.252.10 --san localhost --san 127.0.0.1
sudo step certificate bundle /etc/secrets/dana/services/enrollment-fabric-ca-server/tls/cert.pem /etc/secrets/dana/tlsca/intermediate-cert.pem /etc/secrets/dana/services/enrollment-fabric-ca-server/tls/bundle.pem
```

### Validate Root, Intermediate, and Client Certificate
If we check, we will have this certificates
```
sudo tree /etc/secrets/
/etc/secrets/
└── dana
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
mkdir -p organizations/PeerOrganizations/dana/msp/cacerts/
mkdir -p organizations/PeerOrganizations/dana/msp/tlscacerts/
mkdir -p organizations/PeerOrganizations/dana/msp/intermediatecerts/
mkdir -p organizations/PeerOrganizations/dana/msp/tlsintermediatecerts/
```

copy the certificate
```
sudo cp /etc/secrets/dana/ca/root-cert.pem organizations/PeerOrganizations/dana/msp/cacerts/
sudo cp /etc/secrets/dana/ca/intermediate-cert.pem organizations/PeerOrganizations/dana/msp/intermediatecerts/
sudo cp /etc/secrets/dana/tlsca/root-cert.pem organizations/PeerOrganizations/dana/msp/tlscacerts/
sudo cp /etc/secrets/dana/tlsca/intermediate-cert.pem organizations/PeerOrganizations/dana/msp/tlsintermediatecerts/
```

change the owners
```
sudo chown -R vagrant:vagrant organizations/PeerOrganizations/dana/
chmod -R 744 organizations/PeerOrganizations/dana/
```

setup NodeOU
```
cat <<EOF | tee organizations/PeerOrganizations/dana/msp/config.yaml
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
    └── dana
        ├── msp
        │   ├── cacerts
        │   │   └── root-cert.pem
        │   ├── config.yaml
        │   ├── intermediatecerts
        │   │   └── intermediate-cert.pem
        │   ├── tlscacerts
        │   │   └── root-cert.pem
        │   └── tlsintermediatecerts
        │       └── intermediate-cert.pem
```

Copy enrollment & TLS intermediate fullchain to each peer nodes
```
ssh vagrant@10.250.252.20 "mkdir -p ~/organizations/PeerOrganizations/dana/"
scp -r organizations/PeerOrganizations/dana/msp vagrant@10.250.252.20:~/organizations/PeerOrganizations/dana/

ssh vagrant@10.250.252.21 "mkdir -p ~/organizations/PeerOrganizations/dana/"
scp -r organizations/PeerOrganizations/dana/msp vagrant@10.250.252.21:~/organizations/PeerOrganizations/dana/
```
