# Setup Root & Intermediate CA
If you already have root CA available, you can directly create intermediate CA from it. In this tutorial, we will create two certificate.

### Root CA
ssh to the nodes
```shell
$ vagrant ssh payments-ca-server-0
```

create root CA directory
```shell
$ mkdir -p certificates/enrollment/root/
$ mkdir -p certificates/tls/root/
```

create enrollment root CA
```shell
$ step certificate create root-ledger-ca certificates/enrollment/root/root-cert.pem certificates/enrollment/root/root-key.pem --profile root-ca --kty RSA --no-password --insecure --not-after 87600h --san root.ledger
```

create TLS root CA
```shell
$ step certificate create root-ledger-tls-ca certificates/tls/root/root-cert.pem certificates/tls/root/root-key.pem --profile root-ca --kty RSA --no-password --insecure --not-after 87600h --san root.ledger.tls
```

### Intermediate CA
create Intermediate CA directory
```shell
mkdir -p certificates/enrollment/intermediate
mkdir -p certificates/tls/intermediate
```

Create enrollment Intermediate CA
```shell
step certificate create intermediate-ledger-ca certificates/enrollment/intermediate/ca-cert.pem certificates/enrollment/intermediate/ca-key.pem --profile intermediate-ca --kty RSA --ca certificates/enrollment/root/root-cert.pem --ca-key certificates/enrollment/root/root-key.pem --no-password --insecure --not-after 43800h --san intermediate.ledger
step certificate bundle certificates/enrollment/intermediate/ca-cert.pem certificates/enrollment/root/root-cert.pem certificates/enrollment/intermediate/cert-chain.pem
```

Create TLS Intermediate CA
```shell
step certificate create intermediate-ledger-tls-ca certificates/tls/intermediate/ca-cert.pem certificates/tls/intermediate/ca-key.pem --profile intermediate-ca --kty RSA --ca certificates/tls/root/root-cert.pem --ca-key certificates/tls/root/root-key.pem --no-password --insecure --not-after 43800h --san intermediate.ledger.tls
step certificate bundle certificates/tls/intermediate/ca-cert.pem certificates/tls/root/root-cert.pem certificates/tls/intermediate/cert-chain.pem
```
