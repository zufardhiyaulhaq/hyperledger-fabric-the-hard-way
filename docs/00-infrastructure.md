# Infrastructure

First, we need to bring up the infrastructure. Clone this repository
```shell
git clone https://github.com/zufardhiyaulhaq/hyperledger-fabric-the-hard-way.git
```

Navigate to the docs directory and start the Virtual Machine (VM)
```shell
cd hyperledger-fabric-the-hard-way/docs
vagrant up
```

You can then check the status of various VMs
```shell
vagrant status
==> vagrant: A new version of Vagrant is available: 2.2.19 (installed version: 2.2.9)!
==> vagrant: To upgrade visit: https://www.vagrantup.com/downloads.html

Current machine states:

bi-ca-server-0            running (virtualbox)
bi-orderer-0              running (virtualbox)
bi-orderer-1              running (virtualbox)
bi-orderer-2              running (virtualbox)
gopay-ca-server-0         running (virtualbox)
gopay-peer-0              running (virtualbox)
gopay-peer-1              running (virtualbox)
dana-ca-server-0          running (virtualbox)
dana-peer-0               running (virtualbox)
dana-peer-1               running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```

To access a VM, you can simply use vagrant ssh. Here's an example:
```shell
vagrant ssh payments-ca-server-0
```
