### Bank Indonesia orderer1 node
ssh to the nodes
```shell
vagrant ssh bi-orderer-1
```

run this command to join orderer to the channel
```
osnadmin channel join --channelID qris  --config-block genesis_block.pb -o 127.0.0.1:12443
```

you should see responses like this
```json
Status: 201
{
	"name": "qris",
	"url": "/participation/v1/channels/qris",
	"consensusRelation": "consenter",
	"status": "active",
	"height": 1
}
```
