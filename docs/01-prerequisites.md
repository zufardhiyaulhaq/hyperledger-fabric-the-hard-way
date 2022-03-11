# Prerequisites

There several different prerequisites for different ledger sub system.

# CA server
ssh to the nodes
```shell
$ vagrant ssh payments-ca-server-0
```

Install step certificate
```shell
$ wget https://dl.step.sm/gh-release/cli/docs-cli-install/v0.18.2/step-cli_0.18.2_amd64.deb
$ sudo dpkg -i step-cli_0.18.2_amd64.deb
$ rm -rf https://dl.step.sm/gh-release/cli/docs-cli-install/v0.18.2/step-cli_0.18.2_amd64.deb
```

Install Hyperledger Fabric CA binary
```shell
$ wget https://github.com/hyperledger/fabric-ca/releases/download/v1.5.2/hyperledger-fabric-ca-darwin-amd64-1.5.2.tar.gz
$ tar xzvf hyperledger-fabric-ca-darwin-amd64-1.5.2.tar.gz
$ cp bin/fabric-ca-server /usr/local/bin/fabric-ca-server
$ cp bin/fabric-ca-client /usr/local/bin/fabric-ca-client
$ rm -rf bin/
$ rm -rf hyperledger-fabric-ca-darwin-amd64-1.5.2.tar.gz
```
