# Setting up a Validator Node

> **Note**: This steps suggested in this document are tentative and subject to change.

## Hardware Requirements
* **Minimal**
    * 4 GB RAM
    * 250 GB SSD
    * 1.4 GHz x2 CPU
* **Recommended**
    * 8 GB RAM
    * 500 GB SDD
    * 2.0 GHz x4 CPU

## Operating System

- Linux (Ubuntu 20.04+ Recommended)
- MacOS

## Prerequisite

- Golang 1.18+ required. (<a href="https://go.dev/doc/install">Installation Ref</a>)
- git <a href="https://git-scm.com/book/en/v2/Getting-Started-Installing-Git">Installation Ref</a>
- jq <a href="https://lindevs.com/install-jq-on-ubuntu/">Installation Ref</a>
- make <a href="https://linuxhint.com/install-make-ubuntu/">Installation Ref</a>

## Installation Steps

- Clone the repository
```
git clone https://github.com/hypersign-protocol/hid-node.git
```

- Build the node
```
cd hid-node
make install
```

- Run the following to check if the node is successfully installed
```
hid-noded version
```

## Generate Keys

`hid-noded keys add <key-name>`

or

`hid-noded keys add <key-name> --recover` to regenerate keys with your [BIP39](https://github.com/bitcoin/bips/tree/master/bip-0039) mnemonic

## Validator setup

> Note: Some directories mentioned in the below steps will be created soon.

### Pre Genesis Stage

- Initialize Node
```
hid-noded init <validator-name> --chain-id <testnet-chain-id>
```
- Run the following to change the coin denom from `stake` to `uhid` in the generated `genesis.json`
```
cat $HOME/.hid-node/config/genesis.json | jq '.app_state["crisis"]["constant_fee"]["denom"]="uhid"' > $HOME/.hid-node/config/tmp_genesis.json && mv $HOME/.hid-node/config/tmp_genesis.json $HOME/.hid-node/config/genesis.json

cat $HOME/.hid-node/config/genesis.json | jq '.app_state["gov"]["deposit_params"]["min_deposit"][0]["denom" ]="uhid"' > $HOME/.hid-node/config/tmp_genesis.json && mv $HOME/.hid-node/config/tmp_genesis.json $HOME/.hid-node/config/genesis.json

cat $HOME/.hid-node/config/genesis.json | jq '.app_state["mint"]["params"]["mint_denom"]="uhid"' > $HOME/.hid-node/config/tmp_genesis.json && mv $HOME/.hid-node/config/tmp_genesis.json $HOME/.hid-node/config/genesis.json

cat $HOME/.hid-node/config/genesis.json | jq '.app_state["staking"]["params"]["bond_denom"]="uhid"' > $HOME/.hid-node/config/tmp_genesis.json && mv $HOME/.hid-node/config/tmp_genesis.json $HOME/.hid-node/config/genesis.json
```
- Create a gentx transaction
```
hid-noded add-genesis-account <key-name> 10000000000000uhid
hid-noded gentx <key-name> 1000000000000uhid \
--chain-id <testnet-chain-id> \
--moniker="<validator-name>" \
--commission-max-change-rate=0.01 \
--commission-max-rate=1.0 \
--commission-rate=0.07 \
--min-self-delegation=500000000000 \
--details="XXXXXXXX" \
--security-contact="XXXXXXXX" \
--website="XXXXXXXX"
```
- Copy the contents of `${HOME}/.hid-node/config/gentx/gentx-XXXXXXXX.json`.
- Fork the [repository](https://github.com/hypersign-protocol/networks)
- Create a file `gentx-<validator-name>.json` under the `testnet/<testnet-chain-id>/gentxs` folder in the forked repo and paste the copied text into the file.
- Run `hid-noded tendermint show-node-id` and copy your Node ID.
- Run `ifconfig` or `curl ipinfo.io/ip` and copy your publicly reachable IP address.
- Form the complete node address in the format: `<node-id>@<publicly-reachable-ip>:<p2p-port>`
- Create a file `peers-<validator-name>.json` under the `testnet/<testnet-chain-id>/peers` directory in the forked repo and paste the copied text from the last step into the file.
- Create a Pull Request to the `master` branch of the [repository](https://github.com/hypersign-protocol/networks)
>**NOTE:** Pull Request will be merged by the maintainers to confirm the inclusion of the validator at the genesis. The final genesis file will be published under the file `testnet/<testnet-chain-id>/final_genesis.json`.
- Once the `final_genesis.json` file is published, replace the contents of your `${HOME}/.hid-node/config/genesis.json` with that of `testnet/<testnet-chain-id>/final_genesis.json`.
- Add `persistent_peers` or `seeds` in `${HOME}/.hid-node/config/config.toml` from `testnet/<testnet-chain-id>/final_peers.json`.
- Set the `minimum-gas-price` in `${HOME}/.hid-node/config/app.toml`. Example value: `10uhid` 
- Start node
```sh
hid-noded start
```

### Post Genesis Stage

- Follow the steps to install `hid-node` binary [here](#installation-steps)
- Initialize node
```sh
hid-noded init <validator-name>
```
- Replace the contents of your `${HOME}/.hid-noded/config/genesis.json` with that of `testnet/<testnet-chain-id>/final_genesis.json` from the `master` branch of [repository](https://github.com/hypersign-protocol/networks).
- Add `persistent_peers` or `seeds` in `${HOME}/.hid-noded/config/config.toml` from `testnet/<testnet-chain-id>/final_peers.json` from the `master` branch of [repository](https://github.com/hypersign-protocol/networks).
- Set the `minimum-gas-price` in `${HOME}/.hid-node/config/app.toml`. Example value: `10uhid`
- Start node
```shell
hid-noded start
```
- Generate keys by either running `hid-noded keys add <key-name>` or `hid-noded keys add <key-name> --recover` to regenerate keys with your [BIP39](https://github.com/bitcoin/bips/tree/master/bip-0039) mnemonic
- Acquire some $HID
- Send a validator creation transaction
```
hid-noded tx staking create-validator \
--from <key-name> \
--amount 1000000000000uhid \
--pubkey "$(hid-noded tendermint show-validator)" \
--chain-id <testnet-chain-id> \
--moniker="<validator-name>" \
--commission-max-change-rate=0.01 \
--commission-max-rate=1.0 \
--commission-rate=0.07 \
--min-self-delegation="500000000000" \
--details="XXXXXXXX" \
--security-contact="XXXXXXXX" \
--website="XXXXXXXX"
```
