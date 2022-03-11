# Setup Hyperledger Orderer Consensus
there are nevertheless several different implementations for achieving consensus on the strict ordering of transactions between ordering service nodes. We will use Raft consensus implemented in etcd for this purpose. This etcd service will be implemented in each of Orderer node.

### payments-orderer-0
ssh to the node
```shell
vagrant ssh payments-orderer-0
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
fabric-ca-client enroll -d -u https://orderer0@payments:orderer0-payments-password@10.250.250.10:7054 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --enrollment.profile tls --csr.cn etcd --csr.hosts 'localhost,127.0.0.1,10.250.250.20' --mspdir ${HOME}/certificates/etcd/client
fabric-ca-client enroll -d -u https://orderer0@payments:orderer0-payments-password@10.250.250.10:7054 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --enrollment.profile tls --csr.cn etcd-peer --csr.hosts 'localhost,127.0.0.1,10.250.250.20,10.250.250.21,10.250.250.22' --mspdir ${HOME}/certificates/etcd/peer
```

copy certificate to the etcd directory
```shell
sudo cp ${HOME}/certificates/etcd/client/signcerts/cert.pem /etc/etcd/cert.pem
sudo cp ${HOME}/certificates/etcd/peer/signcerts/cert.pem /etc/etcd/peer-cert.pem
```

copy certificate key to the etcd directory, everytime you generate, the filename under `keystore` directory is changing, so you must change it by yourself!
```shell
sudo cp ${HOME}/certificates/etcd/client/keystore/068c4be97f717a79367d862d3f40b0c9de553f9a8235b0829701e8197b43ccf7_sk /etc/etcd/key.pem
sudo cp ${HOME}/certificates/etcd/peer/keystore/ff0b1dc14df0e9d9780606143844cc919b70d8bda26cac5f2022b30ae5f65061_sk /etc/etcd/peer-key.pem
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
  --initial-cluster-token payments-orderer \
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

### payments-orderer-1
ssh to the node
```shell
vagrant ssh payments-orderer-1
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
fabric-ca-client enroll -d -u https://orderer1@payments:orderer1-payments-password@10.250.250.10:7054 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --enrollment.profile tls --csr.cn etcd --csr.hosts 'localhost,127.0.0.1,10.250.250.21' --mspdir ${HOME}/certificates/etcd/client
fabric-ca-client enroll -d -u https://orderer1@payments:orderer1-payments-password@10.250.250.10:7054 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --enrollment.profile tls --csr.cn etcd-peer --csr.hosts 'localhost,127.0.0.1,10.250.250.20,10.250.250.21,10.250.250.22' --mspdir ${HOME}/certificates/etcd/peer
```

copy certificate to the etcd directory
```shell
sudo cp ${HOME}/certificates/etcd/client/signcerts/cert.pem /etc/etcd/cert.pem
sudo cp ${HOME}/certificates/etcd/peer/signcerts/cert.pem /etc/etcd/peer-cert.pem
```

copy certificate key to the etcd directory, everytime you generate, the filename under `keystore` directory is changing, so you must change it by yourself!
```shell
sudo cp ${HOME}/certificates/etcd/client/keystore/70202a7423a6338ee2caf904a64f314ee7f0279f5971b51b50cfbf722fab8800_sk /etc/etcd/key.pem
sudo cp ${HOME}/certificates/etcd/peer/keystore/e25b8e37e8421e5becf4ea17ff393b788de585caaea856e46c4f4e17adc61a8a_sk /etc/etcd/peer-key.pem
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
  --initial-cluster-token payments-orderer \
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

### payments-orderer-2
ssh to the node
```shell
vagrant ssh payments-orderer-2
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
fabric-ca-client enroll -d -u https://orderer2@payments:orderer2-payments-password@10.250.250.10:7054 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --enrollment.profile tls --csr.cn etcd --csr.hosts 'localhost,127.0.0.1,10.250.250.22' --mspdir ${HOME}/certificates/etcd/client
fabric-ca-client enroll -d -u https://orderer2@payments:orderer2-payments-password@10.250.250.10:7054 --tls.certfiles ${HOME}/certificates/tls/intermediate/fullchain.crt --enrollment.profile tls --csr.cn etcd-peer --csr.hosts 'localhost,127.0.0.1,10.250.250.20,10.250.250.21,10.250.250.22' --mspdir ${HOME}/certificates/etcd/peer
```

copy certificate to the etcd directory
```shell
sudo cp ${HOME}/certificates/etcd/client/signcerts/cert.pem /etc/etcd/cert.pem
sudo cp ${HOME}/certificates/etcd/peer/signcerts/cert.pem /etc/etcd/peer-cert.pem
```

copy certificate key to the etcd directory, everytime you generate, the filename under `keystore` directory is changing, so you must change it by yourself!
```shell
sudo cp ${HOME}/certificates/etcd/client/keystore/d151d0e0cf704f3decad88cb080889d78f3d45b8c294030759e4e391e9282171_sk /etc/etcd/key.pem
sudo cp ${HOME}/certificates/etcd/peer/keystore/14bca25df8c6d7de982e212e139f2bfae743195892426d8789900ef1d15d3887_sk /etc/etcd/peer-key.pem
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
  --initial-cluster-token payments-orderer \
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
