# Prerequisites
There are several different prerequisites depending on the ledger subsystem.

# CA server
ssh into the nodes
```shell
vagrant ssh bi-ca-server-0
```

Install step certificate binary, this is used to generate certificate authority on Bank Indonesia organization.
```shell
wget https://dl.step.sm/gh-release/cli/docs-cli-install/v0.18.2/step-cli_0.18.2_amd64.deb
sudo dpkg -i step-cli_0.18.2_amd64.deb
rm -rf step-cli_0.18.2_amd64.deb
```

Install Hyperledger Fabric CA binary, this is used to run Fabric CA server that serve creation of certificate under Bank Indonesia organization.
```shell
wget https://github.com/hyperledger/fabric-ca/releases/download/v1.5.2/hyperledger-fabric-ca-linux-amd64-1.5.2.tar.gz
tar xzvf hyperledger-fabric-ca-linux-amd64-1.5.2.tar.gz
sudo cp bin/fabric-ca-server /usr/local/bin/fabric-ca-server
sudo cp bin/fabric-ca-client /usr/local/bin/fabric-ca-client
rm -rf bin/
rm -rf hyperledger-fabric-ca-linux-amd64-1.5.2.tar.gz
```

Download Hyperledger binary, this is used to generate channel genesis block
```
wget https://github.com/hyperledger/fabric/releases/download/v2.4.3/hyperledger-fabric-linux-amd64-2.4.3.tar.gz
tar xzvf hyperledger-fabric-linux-amd64-2.4.3.tar.gz
sudo cp bin/* /usr/local/bin/
rm -rf bin/
rm -rf config/
rm -rf hyperledger-fabric-linux-amd64-2.4.3.tar.gz
```

Install Docker & Docker-compose, which are used to run PostgreSQL, required by the Fabric CA server.

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

For simplicity to distribute the TLS CA certificate later, we can simply add CA server public key to orderer nodes that managed by Bank Indonesia.
```shell
ssh-keygen

# copy to orderer
sshpass -p "vagrant" ssh-copy-id -o StrictHostKeyChecking=no vagrant@10.250.250.20
sshpass -p "vagrant" ssh-copy-id -o StrictHostKeyChecking=no vagrant@10.250.250.21
sshpass -p "vagrant" ssh-copy-id -o StrictHostKeyChecking=no vagrant@10.250.250.22
```

# Orderer
for each orderer nodes, do this following things

ssh to the nodes
```shell
vagrant ssh bi-orderer-0
vagrant ssh bi-orderer-1
vagrant ssh bi-orderer-2
```

Install the Hyperledger Fabric CA binary, which is used to communicate with the Fabric CA server managed by Bank Indonesia.
```shell
wget https://github.com/hyperledger/fabric-ca/releases/download/v1.5.2/hyperledger-fabric-ca-linux-amd64-1.5.2.tar.gz
tar xzvf hyperledger-fabric-ca-linux-amd64-1.5.2.tar.gz
sudo cp bin/fabric-ca-server /usr/local/bin/fabric-ca-server
sudo cp bin/fabric-ca-client /usr/local/bin/fabric-ca-client
rm -rf bin/
rm -rf hyperledger-fabric-ca-linux-amd64-1.5.2.tar.gz
```

Install the Hyperledger Fabric binary, which is used to run the orderer service.
```
wget https://github.com/hyperledger/fabric/releases/download/v2.4.3/hyperledger-fabric-linux-amd64-2.4.3.tar.gz
tar xzvf hyperledger-fabric-linux-amd64-2.4.3.tar.gz
sudo cp bin/* /usr/local/bin/
rm -rf bin/
rm -rf config/
rm -rf hyperledger-fabric-linux-amd64-2.4.3.tar.gz
```

