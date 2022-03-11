# Setup Root & Intermediate CA
If you already have root CA available, you can directly create intermediate CA from it. In this tutorial, we will create two certificate.

### Root CA
ssh to the nodes
```shell
vagrant ssh payments-ca-server-0
```

create root CA directory
```shell
mkdir -p certificates/enrollment/root/
mkdir -p certificates/tls/root/
```

create enrollment root CA
```shell
step certificate create root-ledger-payments-ca certificates/enrollment/root/root.crt certificates/enrollment/root/root.key --profile root-ca --kty EC --no-password --insecure --not-after 87600h --san root.ledger.payments
```

create TLS root CA
```shell
step certificate create root-ledger-tls-payments-ca certificates/tls/root/root.crt certificates/tls/root/root.key --profile root-ca --kty EC --no-password --insecure --not-after 87600h --san root.ledger.tls.payments
```

### Intermediate CA
create Intermediate CA directory
```shell
mkdir -p certificates/enrollment/intermediate
mkdir -p certificates/tls/intermediate
```

Create enrollment Intermediate CA
```shell
step certificate create intermediate-ledger-payments-ca certificates/enrollment/intermediate/ca.crt certificates/enrollment/intermediate/ca.key --profile intermediate-ca --kty EC --ca certificates/enrollment/root/root.crt --ca-key certificates/enrollment/root/root.key --no-password --insecure --not-after 43800h --san intermediate.ledger.payments
step certificate bundle certificates/enrollment/intermediate/ca.crt certificates/enrollment/root/root.crt certificates/enrollment/intermediate/fullchain.crt
```

Create TLS Intermediate CA
```shell
step certificate create intermediate-ledger-tls-payments-ca certificates/tls/intermediate/ca.crt certificates/tls/intermediate/ca.key --profile intermediate-ca --kty EC --ca certificates/tls/root/root.crt --ca-key certificates/tls/root/root.key --no-password --insecure --not-after 43800h --san intermediate.ledger.tls.payments
step certificate bundle certificates/tls/intermediate/ca.crt certificates/tls/root/root.crt certificates/tls/intermediate/fullchain.crt
```

### Client CA
we need to create client for TLS CA, this client certificate will be used to contact to enrollment Fabric CA service as well as TLS Fabric CA service for futher generation of certificate.

create client certificate directory
```shell
mkdir -p certificates/tls/client/admin/
mkdir -p certificates/tls/client/tls-service/
mkdir -p certificates/tls/client/enrollment-service/
```

```shell
step certificate create admin-ledger-tls-payments-ca certificates/tls/client/admin/admin.crt certificates/tls/client/admin/admin.key --profile leaf --ca certificates/tls/intermediate/ca.crt --ca-key certificates/tls/intermediate/ca.key --kty EC --no-password --insecure --not-after 47600h --san admin.ledger.tls.payments
step certificate bundle certificates/tls/client/admin/admin.crt certificates/tls/intermediate/ca.crt certificates/tls/client/admin/fullchain.crt

step certificate create tls-service-ledger-tls-payments-ca certificates/tls/client/tls-service/tls.crt certificates/tls/client/tls-service/tls.key --profile leaf --ca certificates/tls/intermediate/ca.crt --ca-key certificates/tls/intermediate/ca.key --kty EC --no-password --insecure --not-after 47600h --san tls-service.ledger.tls.payments
step certificate bundle certificates/tls/client/tls-service/cert.pem certificates/tls/intermediate/ca.crt certificates/tls/client/tls-service/fullchain.crt

step certificate create enrollment-service-ledger-tls-payments-ca certificates/tls/client/enrollment-service/enrollment.crt certificates/tls/client/enrollment-service/enrollment.key --profile leaf --ca certificates/tls/intermediate/ca.crt --ca-key certificates/tls/intermediate/ca.key --kty EC --no-password --insecure --not-after 47600h --san enrollment-service.ledger.tls.payments
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
    │   ├── admin
    │   │   ├── admin.crt
    │   │   ├── admin.key
    │   │   └── fullchain.crt
    │   ├── enrollment-service
    │   │   ├── enrollment.crt
    │   │   ├── enrollment.key
    │   │   └── fullchain.crt
    │   └── tls-service
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
