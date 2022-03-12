# Setup Root & Intermediate CA
If you already have root CA available, you can directly create intermediate CA from it. In this tutorial, we will create two certificate.

### Root CA
ssh to the nodes
```shell
vagrant ssh gopay-ca-server-0
```

create root CA directory
```shell
mkdir -p certificates/enrollment/root/
mkdir -p certificates/tls/root/
```

create enrollment root CA
```shell
step certificate create root-ledger-gopay-ca certificates/enrollment/root/root.crt certificates/enrollment/root/root.key --profile root-ca --kty EC --no-password --insecure --not-after 87600h --san root.ledger.gopay
```

create TLS root CA
```shell
step certificate create root-ledger-tls-gopay-ca certificates/tls/root/root.crt certificates/tls/root/root.key --profile root-ca --kty EC --no-password --insecure --not-after 87600h --san root.ledger.tls.gopay
```

### Intermediate CA
create Intermediate CA directory
```shell
mkdir -p certificates/enrollment/intermediate
mkdir -p certificates/tls/intermediate
```

Create enrollment Intermediate CA
```shell
step certificate create intermediate-ledger-gopay-ca certificates/enrollment/intermediate/ca.crt certificates/enrollment/intermediate/ca.key --profile intermediate-ca --kty EC --ca certificates/enrollment/root/root.crt --ca-key certificates/enrollment/root/root.key --no-password --insecure --not-after 43800h --san intermediate.ledger.gopay
step certificate bundle certificates/enrollment/intermediate/ca.crt certificates/enrollment/root/root.crt certificates/enrollment/intermediate/fullchain.crt
```

Create TLS Intermediate CA
```shell
step certificate create intermediate-ledger-tls-gopay-ca certificates/tls/intermediate/ca.crt certificates/tls/intermediate/ca.key --profile intermediate-ca --kty EC --ca certificates/tls/root/root.crt --ca-key certificates/tls/root/root.key --no-password --insecure --not-after 43800h --san intermediate.ledger.tls.gopay
step certificate bundle certificates/tls/intermediate/ca.crt certificates/tls/root/root.crt certificates/tls/intermediate/fullchain.crt
```

### Client CA
we need to create client for TLS CA, this client certificate will be used to contact to enrollment Fabric CA service as well as TLS Fabric CA service for futher generation of certificate.

create client certificate directory
```shell
mkdir -p certificates/tls/client/tls-service/
mkdir -p certificates/tls/client/enrollment-service/
```

```shell
step certificate create tls-service-ledger-tls-gopay-ca certificates/tls/client/tls-service/tls.crt certificates/tls/client/tls-service/tls.key --profile leaf --ca certificates/tls/intermediate/ca.crt --ca-key certificates/tls/intermediate/ca.key --kty EC --no-password --insecure --not-after 47600h --san tls-service.ledger.tls.gopay --san 10.250.251.10 --san localhost --san 127.0.0.1
step certificate bundle certificates/tls/client/tls-service/tls.crt certificates/tls/intermediate/ca.crt certificates/tls/client/tls-service/fullchain.crt

step certificate create enrollment-service-ledger-tls-gopay-ca certificates/tls/client/enrollment-service/enrollment.crt certificates/tls/client/enrollment-service/enrollment.key --profile leaf --ca certificates/tls/intermediate/ca.crt --ca-key certificates/tls/intermediate/ca.key --kty EC --no-password --insecure --not-after 47600h --san enrollment-service.ledger.tls.gopay --san 10.250.251.10 --san localhost --san 127.0.0.1
step certificate bundle certificates/tls/client/enrollment-service/enrollment.crt certificates/tls/intermediate/ca.crt certificates/tls/client/enrollment-service/fullchain.crt
```

If we check we will have this all certificates
```shell
tree certificates/
certificates/
├── enrollment
│   ├── intermediate
│   │   ├── ca.crt
│   │   ├── ca.key
│   │   └── fullchain.crt
│   └── root
│       ├── root.crt
│       └── root.key
└── tls
    ├── client
    │   ├── enrollment-service
    │   │   ├── enrollment.crt
    │   │   ├── enrollment.key
    │   │   └── fullchain.crt
    │   └── tls-service
    │       └── fullchain.crt
    │       ├── tls.crt
    │       └── tls.key
    ├── intermediate
    │   ├── ca.crt
    │   ├── ca.key
    │   └── fullchain.crt
    └── root
        ├── root.crt
        └── root.key
```

Copy enrollment & TLS intermediate fullchain to each orderer and peer nodes
```
ssh vagrant@10.250.251.20 "mkdir -p ~/certificates/enrollment/intermediate"
ssh vagrant@10.250.251.20 "mkdir -p ~/certificates/tls/intermediate"

scp certificates/enrollment/intermediate/fullchain.crt vagrant@10.250.251.20:~/certificates/enrollment/intermediate/
scp certificates/tls/intermediate/fullchain.crt vagrant@10.250.251.20:~/certificates/tls/intermediate/

ssh vagrant@10.250.251.21 "mkdir -p ~/certificates/enrollment/intermediate"
ssh vagrant@10.250.251.21 "mkdir -p ~/certificates/tls/intermediate"

scp certificates/enrollment/intermediate/fullchain.crt vagrant@10.250.251.20:~/certificates/enrollment/intermediate/
scp certificates/tls/intermediate/fullchain.crt vagrant@10.250.251.20:~/certificates/tls/intermediate/
```
