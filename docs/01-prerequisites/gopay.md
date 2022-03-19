# Prerequisites
There several different prerequisites for different ledger subsystem.

# CA server
ssh to the nodes
```shell
vagrant ssh gopay-ca-server-0
```

Install step certificate binary, this is used to generate certificate authority on GoPay organization.
```shell
wget https://dl.step.sm/gh-release/cli/docs-cli-install/v0.18.2/step-cli_0.18.2_amd64.deb
sudo dpkg -i step-cli_0.18.2_amd64.deb
rm -rf step-cli_0.18.2_amd64.deb
```

Install Hyperledger Fabric CA binary, this is used to run Fabric CA server that serve creation of certificate under GoPay organization.
```shell
wget https://github.com/hyperledger/fabric-ca/releases/download/v1.5.2/hyperledger-fabric-ca-linux-amd64-1.5.2.tar.gz
tar xzvf hyperledger-fabric-ca-linux-amd64-1.5.2.tar.gz
sudo cp bin/fabric-ca-server /usr/local/bin/fabric-ca-server
sudo cp bin/fabric-ca-client /usr/local/bin/fabric-ca-client
rm -rf bin/
rm -rf hyperledger-fabric-ca-linux-amd64-1.5.2.tar.gz
```

install Docker & docker-compose, this is used to run postgresql that is used by Fabric CA server
```
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release -y 
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update && sudo apt-get install docker-ce docker-ce-cli containerd.io -y

sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

For simplicity to distribute the TLS CA certificate later, we can simply add CA server public key to peer nodes that managed by GoPay.
```shell
ssh-keygen

# copy to peer
sshpass -p "vagrant" ssh-copy-id -o StrictHostKeyChecking=no vagrant@10.250.251.20
sshpass -p "vagrant" ssh-copy-id -o StrictHostKeyChecking=no vagrant@10.250.251.21
```

# Peer
for each peer nodes, do this following things

```shell
vagrant ssh gopay-peer-0
vagrant ssh gopay-peer-1
```

Install Hyperledger Fabric CA binary, this is used to communicate with Fabric CA server that GoPay managed.
```shell
wget https://github.com/hyperledger/fabric-ca/releases/download/v1.5.2/hyperledger-fabric-ca-linux-amd64-1.5.2.tar.gz
tar xzvf hyperledger-fabric-ca-linux-amd64-1.5.2.tar.gz
sudo cp bin/fabric-ca-server /usr/local/bin/fabric-ca-server
sudo cp bin/fabric-ca-client /usr/local/bin/fabric-ca-client
rm -rf bin/
rm -rf hyperledger-fabric-ca-linux-amd64-1.5.2.tar.gz
```

Install Hyperledger Fabric binary, this is used to run peer service.
```
wget https://github.com/hyperledger/fabric/releases/download/v2.4.3/hyperledger-fabric-linux-amd64-2.4.3.tar.gz
tar xzvf hyperledger-fabric-linux-amd64-2.4.3.tar.gz
sudo cp bin/* /usr/local/bin/
rm -rf bin/
rm -rf config/
rm -rf hyperledger-fabric-linux-amd64-2.4.3.tar.gz
```

Install Golang to package the chaincode if needed
```shell
sudo su

bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
source /root/.gvm/scripts/gvm

sudo apt-get install binutils bison gcc make

gvm install go1.4 -B
gvm use go1.4
export GOROOT_BOOTSTRAP=$GOROOT

gvm install go1.16.3
gvm use go1.16.3
```

install Docker, this is used to build the chaincode in the peer server
```
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release -y 
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update && sudo apt-get install docker-ce docker-ce-cli containerd.io -y
```
