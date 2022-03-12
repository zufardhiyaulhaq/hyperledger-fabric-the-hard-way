# Setup Hyperledger Orderer Consensus
there are nevertheless several different implementations for achieving consensus on the strict ordering of transactions between ordering service nodes. We will use Raft consensus implemented in etcd for this purpose. This etcd service will be implemented in each of Orderer node.

***notes**: BI organization handle Orderer nodes*

### bi-orderer-0
ssh to the node
```shell
vagrant ssh bi-orderer-0
```

download etcd binary
```shell
wget https://github.com/coreos/etcd/releases/download/v3.5.2/etcd-v3.5.2-linux-amd64.tar.gz
tar xzvf etcd-v3.5.2-linux-amd64.tar.gz
sudo cp etcd-v3.5.2-linux-amd64/etcdutl /usr/local/bin/etcdutl
sudo cp etcd-v3.5.2-linux-amd64/etcdctl /usr/local/bin/etcdctl
sudo cp etcd-v3.5.2-linux-amd64/etcd /usr/local/bin/etcd
rm -rf etcd-v3.5.2-linux-amd64
rm -rf etcd-v3.5.2-linux-amd64.tar.gz
```

create etcd directory
```shell
sudo mkdir -p /etc/etcd
sudo mkdir -p /var/lib/etcd
```

create certificate directory
```shell
mkdir -p certificates/etcd/client
mkdir -p certificates/etcd/peer
```

get certificate from TLS fabric CA server
```shell
fabric-ca-client enroll -d -u https://orderer0@bi:orderer0-bi-password@10.250.250.10:7054 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --enrollment.profile tls --csr.cn etcd --csr.hosts 'localhost,127.0.0.1,10.250.250.20' --csr.names C=id,O=bi,ST=jakarta --mspdir ${HOME}/certificates/etcd/client
fabric-ca-client enroll -d -u https://orderer0@bi:orderer0-bi-password@10.250.250.10:7054 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --enrollment.profile tls --csr.cn etcd-peer --csr.hosts 'localhost,127.0.0.1,10.250.250.20,10.250.250.21,10.250.250.22' --csr.names C=id,O=bi,ST=jakarta --mspdir ${HOME}/certificates/etcd/peer
```

copy certificate to the etcd directory
```shell
sudo cp ${HOME}/certificates/etcd/client/signcerts/cert.pem /etc/etcd/cert.pem
sudo cp ${HOME}/certificates/etcd/peer/signcerts/cert.pem /etc/etcd/peer-cert.pem
```

copy certificate key to the etcd directory, everytime you generate, the filename under `keystore` directory is changing, so you must change it by yourself!
```shell
sudo cp ${HOME}/certificates/etcd/client/keystore/2eb70e6fa985fb872c11f63b337866fa1554453c940fe058b18e1107b9e9f93e_sk /etc/etcd/key.pem
sudo cp ${HOME}/certificates/etcd/peer/keystore/f7f0334a06657e2c3cc062922ea74c5d31e204062cfd2d07eafcfbcb74e0d6f9_sk /etc/etcd/peer-key.pem
```

copy intermediate CA certificate, everytime you generate, the filename under `tlsintermediatecerts` directory is changing, so you must change it by yourself!
```shell
sudo cp ${HOME}/certificates/etcd/client/tlsintermediatecerts/tls-10-250-250-10-7054.pem /etc/etcd/ca.pem
sudo cp ${HOME}/certificates/etcd/peer/tlsintermediatecerts/tls-10-250-250-10-7054.pem /etc/etcd/peer-ca.pem
```

create etcd configuration
```shell
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos
 
[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \
  --name 10.250.250.20 \
  --cert-file=/etc/etcd/cert.pem \
  --key-file=/etc/etcd/key.pem \
  --peer-cert-file=/etc/etcd/peer-cert.pem \
  --peer-key-file=/etc/etcd/peer-key.pem \
  --trusted-ca-file=/etc/etcd/ca.pem \
  --peer-trusted-ca-file=/etc/etcd/peer-ca.pem \
  --client-cert-auth \
  --peer-client-cert-auth \
  --initial-advertise-peer-urls https://10.250.250.20:2380 \
  --listen-peer-urls https://10.250.250.20:2380 \
  --listen-client-urls https://10.250.250.20:2379,https://127.0.0.1:2379 \
  --advertise-client-urls https://10.250.250.20:2379 \
  --initial-cluster-token bi-orderer \
  --initial-cluster 10.250.250.20=https://10.250.250.20:2380,10.250.250.21=https://10.250.250.21:2380,10.250.250.22=https://10.250.250.22:2380 \
  --initial-cluster-state new \
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5
 
[Install]
WantedBy=multi-user.target
EOF
```

start etcd server
```shell
sudo systemctl enable etcd.service
sudo systemctl start etcd.service
sudo systemctl status etcd.service
```

### bi-orderer-1
ssh to the node
```shell
vagrant ssh bi-orderer-1
```

download etcd binary
```shell
wget https://github.com/coreos/etcd/releases/download/v3.5.2/etcd-v3.5.2-linux-amd64.tar.gz
tar xzvf etcd-v3.5.2-linux-amd64.tar.gz
sudo cp etcd-v3.5.2-linux-amd64/etcdutl /usr/local/bin/etcdutl
sudo cp etcd-v3.5.2-linux-amd64/etcdctl /usr/local/bin/etcdctl
sudo cp etcd-v3.5.2-linux-amd64/etcd /usr/local/bin/etcd
rm -rf etcd-v3.5.2-linux-amd64
rm -rf etcd-v3.5.2-linux-amd64.tar.gz
```

create etcd directory
```shell
sudo mkdir -p /etc/etcd
sudo mkdir -p /var/lib/etcd
```

create certificate directory
```shell
mkdir -p certificates/etcd/client
mkdir -p certificates/etcd/peer
```

get certificate from TLS fabric CA server
```shell
fabric-ca-client enroll -d -u https://orderer1@bi:orderer1-bi-password@10.250.250.10:7054 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --enrollment.profile tls --csr.cn etcd --csr.hosts 'localhost,127.0.0.1,10.250.250.21' --csr.names C=id,O=bi,ST=jakarta --mspdir ${HOME}/certificates/etcd/client
fabric-ca-client enroll -d -u https://orderer1@bi:orderer1-bi-password@10.250.250.10:7054 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --enrollment.profile tls --csr.cn etcd-peer --csr.hosts 'localhost,127.0.0.1,10.250.250.20,10.250.250.21,10.250.250.22' --csr.names C=id,O=bi,ST=jakarta --mspdir ${HOME}/certificates/etcd/peer
```

copy certificate to the etcd directory
```shell
sudo cp ${HOME}/certificates/etcd/client/signcerts/cert.pem /etc/etcd/cert.pem
sudo cp ${HOME}/certificates/etcd/peer/signcerts/cert.pem /etc/etcd/peer-cert.pem
```

copy certificate key to the etcd directory, everytime you generate, the filename under `keystore` directory is changing, so you must change it by yourself!
```shell
sudo cp ${HOME}/certificates/etcd/client/keystore/3a6a285961690801f5d54c27a6bd69b164b1efaa9255af095857d10069fd894c_sk /etc/etcd/key.pem
sudo cp ${HOME}/certificates/etcd/peer/keystore/ed300b0ffe8fd4e763dbdade68721cced3ed4489806883784424553514d5c1e0_sk /etc/etcd/peer-key.pem
```

copy intermediate CA certificate, everytime you generate, the filename under `tlsintermediatecerts` directory is changing, so you must change it by yourself!
```shell
sudo cp ${HOME}/certificates/etcd/client/tlsintermediatecerts/tls-10-250-250-10-7054.pem /etc/etcd/ca.pem
sudo cp ${HOME}/certificates/etcd/peer/tlsintermediatecerts/tls-10-250-250-10-7054.pem /etc/etcd/peer-ca.pem
```

create etcd configuration
```shell
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos
 
[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \
  --name 10.250.250.21 \
  --cert-file=/etc/etcd/cert.pem \
  --key-file=/etc/etcd/key.pem \
  --peer-cert-file=/etc/etcd/peer-cert.pem \
  --peer-key-file=/etc/etcd/peer-key.pem \
  --trusted-ca-file=/etc/etcd/ca.pem \
  --peer-trusted-ca-file=/etc/etcd/peer-ca.pem \
  --client-cert-auth \
  --peer-client-cert-auth \
  --initial-advertise-peer-urls https://10.250.250.21:2380 \
  --listen-peer-urls https://10.250.250.21:2380 \
  --listen-client-urls https://10.250.250.21:2379,https://127.0.0.1:2379 \
  --advertise-client-urls https://10.250.250.21:2379 \
  --initial-cluster-token bi-orderer \
  --initial-cluster 10.250.250.20=https://10.250.250.20:2380,10.250.250.21=https://10.250.250.21:2380,10.250.250.22=https://10.250.250.22:2380 \
  --initial-cluster-state new \
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5
 
[Install]
WantedBy=multi-user.target
EOF
```

start etcd server
```shell
sudo systemctl enable etcd.service
sudo systemctl start etcd.service
sudo systemctl status etcd.service
```

### bi-orderer-2
ssh to the node
```shell
vagrant ssh bi-orderer-2
```

download etcd binary
```shell
wget https://github.com/coreos/etcd/releases/download/v3.5.2/etcd-v3.5.2-linux-amd64.tar.gz
tar xzvf etcd-v3.5.2-linux-amd64.tar.gz
sudo cp etcd-v3.5.2-linux-amd64/etcdutl /usr/local/bin/etcdutl
sudo cp etcd-v3.5.2-linux-amd64/etcdctl /usr/local/bin/etcdctl
sudo cp etcd-v3.5.2-linux-amd64/etcd /usr/local/bin/etcd
rm -rf etcd-v3.5.2-linux-amd64
rm -rf etcd-v3.5.2-linux-amd64.tar.gz
```

create etcd directory
```shell
sudo mkdir -p /etc/etcd
sudo mkdir -p /var/lib/etcd
```

create certificate directory
```shell
mkdir -p certificates/etcd/client
mkdir -p certificates/etcd/peer
```

get certificate from TLS fabric CA server
```shell
fabric-ca-client enroll -d -u https://orderer2@bi:orderer2-bi-password@10.250.250.10:7054 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --enrollment.profile tls --csr.cn etcd --csr.hosts 'localhost,127.0.0.1,10.250.250.22' --csr.names C=id,O=bi,ST=jakarta --mspdir ${HOME}/certificates/etcd/client
fabric-ca-client enroll -d -u https://orderer2@bi:orderer2-bi-password@10.250.250.10:7054 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --enrollment.profile tls --csr.cn etcd-peer --csr.hosts 'localhost,127.0.0.1,10.250.250.20,10.250.250.21,10.250.250.22' --csr.names C=id,O=bi,ST=jakarta --mspdir ${HOME}/certificates/etcd/peer
```

copy certificate to the etcd directory
```shell
sudo cp ${HOME}/certificates/etcd/client/signcerts/cert.pem /etc/etcd/cert.pem
sudo cp ${HOME}/certificates/etcd/peer/signcerts/cert.pem /etc/etcd/peer-cert.pem
```

copy certificate key to the etcd directory, everytime you generate, the filename under `keystore` directory is changing, so you must change it by yourself!
```shell
sudo cp ${HOME}/certificates/etcd/client/keystore/ccaa933ec29f608d5b5af135d0dfa96961a3bcdeda2e782790a29c65884b7026_sk /etc/etcd/key.pem
sudo cp ${HOME}/certificates/etcd/peer/keystore/234e6c599e8fdc5f2e78049a261d5e03321f60350846e389c405c7ca87c62e06_sk /etc/etcd/peer-key.pem
```

copy intermediate CA certificate, everytime you generate, the filename under `tlsintermediatecerts` directory is changing, so you must change it by yourself!
```shell
sudo cp ${HOME}/certificates/etcd/client/tlsintermediatecerts/tls-10-250-250-10-7054.pem /etc/etcd/ca.pem
sudo cp ${HOME}/certificates/etcd/peer/tlsintermediatecerts/tls-10-250-250-10-7054.pem /etc/etcd/peer-ca.pem
```

create etcd configuration
```shell
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos
 
[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \
  --name 10.250.250.22 \
  --cert-file=/etc/etcd/cert.pem \
  --key-file=/etc/etcd/key.pem \
  --peer-cert-file=/etc/etcd/peer-cert.pem \
  --peer-key-file=/etc/etcd/peer-key.pem \
  --trusted-ca-file=/etc/etcd/ca.pem \
  --peer-trusted-ca-file=/etc/etcd/peer-ca.pem \
  --client-cert-auth \
  --peer-client-cert-auth \
  --initial-advertise-peer-urls https://10.250.250.22:2380 \
  --listen-peer-urls https://10.250.250.22:2380 \
  --listen-client-urls https://10.250.250.22:2379,https://127.0.0.1:2379 \
  --advertise-client-urls https://10.250.250.22:2379 \
  --initial-cluster-token bi-orderer \
  --initial-cluster 10.250.250.20=https://10.250.250.20:2380,10.250.250.21=https://10.250.250.21:2380,10.250.250.22=https://10.250.250.22:2380 \
  --initial-cluster-state new \
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5
 
[Install]
WantedBy=multi-user.target
EOF
```

start etcd server
```shell
sudo systemctl enable etcd.service
sudo systemctl start etcd.service
sudo systemctl status etcd.service
```
