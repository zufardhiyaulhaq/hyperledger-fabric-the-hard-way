# Setup Root & Intermediate CA
If you already have root CA available, you can directly create intermediate CA from it. In this tutorial, we will create two certificate.

### Root CA
ssh to the nodes
```shell
vagrant ssh bi-ca-server-0
```

create root CA directory
```shell
sudo mkdir -p /etc/secrets/bi/ca/
sudo mkdir -p /etc/secrets/bi/tlsca/
```

create enrollment root CA
```shell
sudo step certificate create root-ledger-bi-ca /etc/secrets/bi/ca/root-cert.pem /etc/secrets/bi/ca/root-key.pem --profile root-ca --kty EC --no-password --insecure --not-after 87600h --san root.enrollment.bi.go.id
```

create TLS root CA
```shell
sudo step certificate create root-ledger-tls-bi-ca /etc/secrets/bi/tlsca/root-cert.pem /etc/secrets/bi/tlsca/root-key.pem --profile root-ca --kty EC --no-password --insecure --not-after 87600h --san root.tls.bi.go.id
```

### Intermediate CA
Create enrollment Intermediate CA
```shell
sudo step certificate create intermediate-ledger-bi-ca /etc/secrets/bi/ca/intermediate-cert.pem /etc/secrets/bi/ca/intermediate-key.pem --profile intermediate-ca --kty EC --ca /etc/secrets/bi/ca/root-cert.pem --ca-key /etc/secrets/bi/ca/root-key.pem --no-password --insecure --not-after 43800h --san intermediate.enrollment.bi.go.id
sudo step certificate bundle /etc/secrets/bi/ca/intermediate-cert.pem /etc/secrets/bi/ca/root-cert.pem /etc/secrets/bi/ca/intermediate-bundle.pem
```

Create TLS Intermediate CA
```shell
sudo step certificate create intermediate-ledger-tls-bi-ca /etc/secrets/bi/tlsca/intermediate-cert.pem /etc/secrets/bi/tlsca/intermediate-key.pem --profile intermediate-ca --kty EC --ca /etc/secrets/bi/tlsca/root-cert.pem --ca-key /etc/secrets/bi/tlsca/root-key.pem --no-password --insecure --not-after 43800h --san intermediate.tls.bi.go.id
sudo step certificate bundle /etc/secrets/bi/tlsca/intermediate-cert.pem /etc/secrets/bi/tlsca/root-cert.pem /etc/secrets/bi/tlsca/intermediate-bundle.pem
```

starting for now, let's not do anything with this certificate keys, we must not do any operations to certificate keys.

### Client CA
we need to create client for TLS CA, this client certificate will be used to contact to enrollment Fabric CA service as well as TLS Fabric CA service for futher generation of certificate.

create client certificate directory
```shell
sudo mkdir -p /etc/secrets/bi/services/tls-fabric-ca-server/tls/
sudo mkdir -p /etc/secrets/bi/services/enrollment-fabric-ca-server/tls/
```

```shell
sudo step certificate create tls-fabric-ca-server-ledger-tls-bi-ca /etc/secrets/bi/services/tls-fabric-ca-server/tls/cert.pem /etc/secrets/bi/services/tls-fabric-ca-server/tls/key.pem --profile leaf --ca /etc/secrets/bi/tlsca/intermediate-cert.pem --ca-key /etc/secrets/bi/tlsca/intermediate-key.pem --kty EC --no-password --insecure --not-after 47600h --san ca-server.tls.bi.go.id --san 10.250.250.10 --san localhost --san 127.0.0.1
sudo step certificate bundle /etc/secrets/bi/services/tls-fabric-ca-server/tls/cert.pem /etc/secrets/bi/tlsca/intermediate-cert.pem /etc/secrets/bi/services/tls-fabric-ca-server/tls/bundle.pem

sudo step certificate create enrollment-fabric-ca-server-ledger-tls-bi-ca /etc/secrets/bi/services/enrollment-fabric-ca-server/tls/cert.pem /etc/secrets/bi/services/enrollment-fabric-ca-server/tls/key.pem --profile leaf --ca /etc/secrets/bi/tlsca/intermediate-cert.pem --ca-key /etc/secrets/bi/tlsca/intermediate-key.pem --kty EC --no-password --insecure --not-after 47600h --san ca-server.enrollment.bi.go.id --san 10.250.250.10 --san localhost --san 127.0.0.1
sudo step certificate bundle /etc/secrets/bi/services/enrollment-fabric-ca-server/tls/cert.pem /etc/secrets/bi/tlsca/intermediate-cert.pem /etc/secrets/bi/services/enrollment-fabric-ca-server/tls/bundle.pem
```

### Validate Root, Intermediate, and Client Certificate
If we check, we will have this certificates
```
sudo tree /etc/secrets/
/etc/secrets/
└── bi
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
mkdir -p organizations/OrdererOrganizations/bi/msp/cacerts/
mkdir -p organizations/OrdererOrganizations/bi/msp/tlscacerts/
mkdir -p organizations/OrdererOrganizations/bi/msp/intermediatecerts/
mkdir -p organizations/OrdererOrganizations/bi/msp/tlsintermediatecerts/
```

copy the certificate
```
sudo cp /etc/secrets/bi/ca/root-cert.pem organizations/OrdererOrganizations/bi/msp/cacerts/
sudo cp /etc/secrets/bi/ca/intermediate-cert.pem organizations/OrdererOrganizations/bi/msp/intermediatecerts/
sudo cp /etc/secrets/bi/tlsca/root-cert.pem organizations/OrdererOrganizations/bi/msp/tlscacerts/
sudo cp /etc/secrets/bi/tlsca/intermediate-cert.pem organizations/OrdererOrganizations/bi/msp/tlsintermediatecerts/
```

change the owners
```
sudo chown -R vagrant:vagrant organizations/OrdererOrganizations/bi/
chmod -R 744 organizations/OrdererOrganizations/bi/
```

If we check, we will have this certificates
```
tree organizations/
organizations/
└── OrdererOrganizations
    └── bi
        └── msp
            ├── cacerts
            │   └── root-cert.pem
            ├── intermediatecerts
            │   └── intermediate-cert.pem
            ├── tlscacerts
            │   └── root-cert.pem
            └── tlsintermediatecerts
                └── intermediate-cert.pem
```

Copy enrollment & TLS intermediate fullchain to each orderer nodes
```
ssh vagrant@10.250.250.20 "mkdir -p ~/organizations/OrdererOrganizations/bi/"
scp -r organizations/OrdererOrganizations/bi/msp vagrant@10.250.250.20:~/organizations/OrdererOrganizations/bi/

ssh vagrant@10.250.250.21 "mkdir -p ~/organizations/OrdererOrganizations/bi/"
scp -r organizations/OrdererOrganizations/bi/msp vagrant@10.250.250.21:~/organizations/OrdererOrganizations/bi/

ssh vagrant@10.250.250.22 "mkdir -p ~/organizations/OrdererOrganizations/bi/"
scp -r organizations/OrdererOrganizations/bi/msp vagrant@10.250.250.22:~/organizations/OrdererOrganizations/bi/
```
