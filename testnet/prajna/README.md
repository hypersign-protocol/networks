# Setting up a Validator Node for Prajna Testnet

## Hardware Requirements
* **Minimal**
    * 16 GB RAM
    * 300 GB SSD
    * 1.4 GHz x2 CPU
* **Recommended**
    * 32 GB RAM
    * 600 GB SDD
    * 2.0 GHz x4 CPU

## Operating System

- Linux (Ubuntu 20.04+ Recommended)
- MacOS

## Prerequisite

- Golang 1.21 required. (<a href="https://go.dev/doc/install">Installation Ref</a>)
- git <a href="https://git-scm.com/book/en/v2/Getting-Started-Installing-Git">Installation Ref</a>
- jq <a href="https://lindevs.com/install-jq-on-ubuntu/">Installation Ref</a>
- make <a href="https://linuxhint.com/install-make-ubuntu/">Installation Ref</a>

## Installation Steps

- Clone the repository
```
git clone https://github.com/hypersign-protocol/hid-node.git
```

- Install `hid-noded` binary (tag: `v0.2.0`)

```
cd hid-node
git checkout v0.2.0
make install
```

- Run the following to check if the node is successfully installed

```
hid-noded version
```

## Validator Setup

- Initialize Node:

```
hid-noded init <validator-name> --chain-id prajna-1
```

- Generate keys by either running `hid-noded keys add <key-name>` or `hid-noded keys add <key-name> --recover` to regenerate keys with your [BIP39](https://github.com/bitcoin/bips/tree/master/bip-0039) mnemonic

- Acquire some $HID tokens from the faucet which available on Hypersign's Official Discord Server. The name of the channel is `prajna-faucet-1`.

- Download the genesis from [here](https://github.com/hypersign-protocol/networks/blob/master/testnet/prajna/final_genesis.json) and save it as `genesis.json` in your `.hid-node` directory

- Set the `persistent_peers` in your `.hid-node/config/config.toml` file. Refer [here](https://github.com/hypersign-protocol/networks/blob/master/testnet/prajna/final_peers.txt) for the list of Testnet peers

- Start your node:

```
hid-noded start
```

Since the Foundation node has not enabled state-sync service, we would suggest you to refer other Prajna Testnet validator's state-sync service to sync up your nodes quickly 

- Wait until your node is synced. Once your node, run the following command to promote your node to a validator node:

```
hid-noded tx staking create-validator \
--from <key-name> \
--amount 1000000uhid \
--pubkey "$(hid-noded comet show-validator)" \
--chain-id prajna-1 \
--moniker="<validator-name>" \
--commission-max-change-rate=0.01 \
--commission-max-rate=1.0 \
--commission-rate=0.07 \
--min-self-delegation="1000000" \
--details="XXXXXXXX" \
--security-contact="XXXXXXXX" \
--website="XXXXXXXX"
```