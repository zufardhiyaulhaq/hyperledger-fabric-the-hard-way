# Prerequisites
There several different prerequisites for different ledger subsystem.

# CA server
ssh to the nodes
```shell
vagrant ssh bi-ca-server-0
```

Install step certificate
```shell
wget https://dl.step.sm/gh-release/cli/docs-cli-install/v0.18.2/step-cli_0.18.2_amd64.deb
sudo dpkg -i step-cli_0.18.2_amd64.deb
rm -rf step-cli_0.18.2_amd64.deb
```

Install Hyperledger Fabric CA binary
```shell
wget https://github.com/hyperledger/fabric-ca/releases/download/v1.5.2/hyperledger-fabric-ca-linux-amd64-1.5.2.tar.gz
tar xzvf hyperledger-fabric-ca-linux-amd64-1.5.2.tar.gz
sudo cp bin/fabric-ca-server /usr/local/bin/fabric-ca-server
sudo cp bin/fabric-ca-client /usr/local/bin/fabric-ca-client
rm -rf bin/
rm -rf hyperledger-fabric-ca-linux-amd64-1.5.2.tar.gz
```

For simplicity to distribute the TLS CA certificate later, we can simply add CA server public key to orderer and peer nodes.
```shell
sudo apt install sshpass -y
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

Install Hyperledger Fabric CA binary
```shell
wget https://github.com/hyperledger/fabric-ca/releases/download/v1.5.2/hyperledger-fabric-ca-linux-amd64-1.5.2.tar.gz
tar xzvf hyperledger-fabric-ca-linux-amd64-1.5.2.tar.gz
sudo cp bin/fabric-ca-server /usr/local/bin/fabric-ca-server
sudo cp bin/fabric-ca-client /usr/local/bin/fabric-ca-client
rm -rf bin/
rm -rf hyperledger-fabric-ca-linux-amd64-1.5.2.tar.gz
```
