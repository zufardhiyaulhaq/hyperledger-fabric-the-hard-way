# Prerequisites
Different ledger subsystems have several distinct prerequisites.

# CA server
ssh into the nodes
```shell
vagrant ssh dana-ca-server-0
```

Install the Step Certificate binary, which is used to generate a Certificate Authority for the DANA organization.
```shell
wget https://dl.step.sm/gh-release/cli/docs-cli-install/v0.18.2/step-cli_0.18.2_amd64.deb
sudo dpkg -i step-cli_0.18.2_amd64.deb
rm -rf step-cli_0.18.2_amd64.deb
```

Install the Hyperledger Fabric CA binary, used to run the Fabric CA server. This server manages the creation of certificates for the DANA organization.
```shell
wget https://github.com/hyperledger/fabric-ca/releases/download/v1.5.2/hyperledger-fabric-ca-linux-amd64-1.5.2.tar.gz
tar xzvf hyperledger-fabric-ca-linux-amd64-1.5.2.tar.gz
sudo cp bin/fabric-ca-server /usr/local/bin/fabric-ca-server
sudo cp bin/fabric-ca-client /usr/local/bin/fabric-ca-client
rm -rf bin/
rm -rf hyperledger-fabric-ca-linux-amd64-1.5.2.tar.gz
```

Install Docker & docker-compose. These are used to run PostgreSQL, which is utilized by the Fabric CA server
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

To simplify the distribution of the TLS CA certificate later, we can add the CA server public key to peer nodes managed by DANA.

```shell
ssh-keygen

# copy to peer
sshpass -p "vagrant" ssh-copy-id -o StrictHostKeyChecking=no vagrant@10.250.252.20
sshpass -p "vagrant" ssh-copy-id -o StrictHostKeyChecking=no vagrant@10.250.252.21
```

# Peer
For each peer node, execute the following steps:

```shell
vagrant ssh dana-peer-0
vagrant ssh dana-peer-1
```

Install the Hyperledger Fabric CA binary. This is used for communication with the Fabric CA server managed by DANA.
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

exit
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
