![Tgrade](https://user-images.githubusercontent.com/92199696/186116109-10ee97b5-637a-4e5e-bf9f-2fdebff9791c.png)

# POSTHUMAN ∞ DVS StateSync configuration for TGrade network

In order to setup StateSync for your node please follow this commands

```
#stop the node and reset (this command is for tgrade launched as a service)
sudo systemctl stop tgrade && tgrade tendermint unsafe-reset-all --home ~/.tgrade

#set variable to POSTHUMAN ꝏ DVS RPC
SNAP_RPC="https://rpc.tgrade.posthuman.digital:443"

#set variables $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
#this is one command 
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

#check variables
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
#if output is something like this one, then continue to the next step
#833050 832050 BC5E2ACA26E1C2CD67DCEC953B00333341973EEC9D170BC1A0D15AB9B39C6530

#add POSTHUMAN ꝏ DVS peer to the persisten_peers list. Feel free to add other peers as well.
peers="d18111d2ee3626fcbe48d50b7ea44474f030dd69@rpc.tgrade.posthuman.digital:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.tgrade/config/config.toml

#this is one command
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.tgrade/config/config.toml

#start the node (this command is for tgrade launched as a service)
sudo systemctl restart tgrade

#check logs
sudo journalctl -u tgrade -f
```
