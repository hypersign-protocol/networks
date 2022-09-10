# Setting up a Validator Node for Jagrat Testnet

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

You can view the key address information using the command: `hid-noded keys list`

## Validator Setup (Pre Genesis Stage)

> Note: Some directories mentioned in the below steps will be created soon.

### Before Final Genesis Release

- Initialize Node
```
hid-noded init <validator-name> --chain-id jagrat
```
- Run the following to change the coin denom from `stake` to `uhid` in the generated `genesis.json`
```
cat $HOME/.hid-node/config/genesis.json | jq '.app_state["crisis"]["constant_fee"]["denom"]="uhid"' > $HOME/.hid-node/config/tmp_genesis.json && mv $HOME/.hid-node/config/tmp_genesis.json $HOME/.hid-node/config/genesis.json
```
```
cat $HOME/.hid-node/config/genesis.json | jq '.app_state["gov"]["deposit_params"]["min_deposit"][0]["denom" ]="uhid"' > $HOME/.hid-node/config/tmp_genesis.json && mv $HOME/.hid-node/config/tmp_genesis.json $HOME/.hid-node/config/genesis.json
```
```
cat $HOME/.hid-node/config/genesis.json | jq '.app_state["mint"]["params"]["mint_denom"]="uhid"' > $HOME/.hid-node/config/tmp_genesis.json && mv $HOME/.hid-node/config/tmp_genesis.json $HOME/.hid-node/config/genesis.json
```
```
cat $HOME/.hid-node/config/genesis.json | jq '.app_state["staking"]["params"]["bond_denom"]="uhid"' > $HOME/.hid-node/config/tmp_genesis.json && mv $HOME/.hid-node/config/tmp_genesis.json $HOME/.hid-node/config/genesis.json
```
- Specify the chain namespace by running the following command
```
cat $HOME/.hid-node/config/genesis.json | jq '.app_state["ssi"]["chain_namespace"]="jagrat"' > $HOME/.hid-node/config/tmp_genesis.json && mv $HOME/.hid-node/config/tmp_genesis.json $HOME/.hid-node/config/genesis.json
```
- Create a gentx account
```
hid-noded add-genesis-account <key-name> 100000000000uhid
```
- Create a gentx transaction. The `<stake-amount-in-uhid>` should be in `uhid`. Example: `100000000000uhid`
```
hid-noded gentx <key-name> <stake-amount-in-uhid> \
--chain-id jagrat \
--moniker="<validator-name>" \
--commission-max-change-rate=0.01 \
--commission-max-rate=1.0 \
--commission-rate=0.07 \
--min-self-delegation=100000000000 \
--details="XXXXXXXX" \
--security-contact="XXXXXXXX" \
--website="XXXXXXXX"
```
- Fork the [repository](https://github.com/hypersign-protocol/networks)
- Copy the contents of `${HOME}/.hid-node/config/gentx/gentx-XXXXXXXX.json`.
- Create a file `gentx-<validator-name-without-spaces>.json` under the `testnet/jagrat/gentxs` folder in the forked repo and paste the copied text from the last step into the file.
- Create a file `peers-<validator-name>.txt` under the `testnet/jagrat/peers` directory in the forked repo.
- Run `hid-noded tendermint show-node-id` and copy your Node ID.
- Run `ifconfig` or `curl ipinfo.io/ip` and copy your publicly reachable IP address.
- Form the complete node address in the format: `<node-id>@<publicly-reachable-ip>:<p2p-port>`. Example: `31a2699a153e60fcdbed8a47e060c1e1d4751616@<publicly-reachable-ip>:26656`. Note: The default P2P port is 26656. If you want to change the port configuration, open `${HOME}/.hid-node/config/config.toml` and under `[p2p]`, change the port in `laddr` attribute.
- Paste the complete node address from the last step into the file `testnet/jagrat/peers/peer-<validator-name-without-spaces>.txt`.
- Create a Pull Request to the `main` branch of the [repository](https://github.com/hypersign-protocol/networks)
>**NOTE:** Pull Request will be merged by the maintainers to confirm the inclusion of the validator at the genesis. The final genesis file will be published under the file `testnet/jagrat/final_genesis.json`.

### After Final Genesis Release

> Cosmovisor is a tool which will enable automatic upgrade of a blockchain, once a software upgrade governance proposal is passed. More information on Cosmovisor [here](https://docs.cosmos.network/v0.45/run-node/cosmovisor.html).

- Download and Install Cosmovisor
```
wget https://github.com/cosmos/cosmos-sdk/releases/download/cosmovisor%2Fv1.2.0/cosmovisor-v1.2.0-linux-amd64.tar.gz && tar -C /usr/local/bin/ -xzf cosmovisor-v1.2.0-linux-amd64.tar.gz
```
- Export the following environment variables
```
export DAEMON_NAME=hid-noded
export DAEMON_PATH=<complete path of hid-noded binary>
export DAEMON_HOME=$HOME/.hid-node
```
- Create a `cosmovisor` directory and copy the existing blockchain binary to the following location
```
mkdir -p $DAEMON_HOME/cosmovisor/genesis/bin
cp $DAEMON_PATH $DAEMON_HOME/cosmovisor/genesis/bin
```
- Once the `final_genesis.json` file is published, replace the contents of your `${HOME}/.hid-node/config/genesis.json` with `testnet/jagrat/final_genesis.json`.
- Copy all the persistent peers present in `testnet/jagrat/final_peers.json` and paste it in the attribute `persistent_peers`, present in the `${HOME}/.hid-node/config/config.toml` file.
- Set the `minimum-gas-price` in `${HOME}/.hid-node/config/app.toml`. Example value: `0.02uhid` 

**Run Node using Cosmosvisor**

You can run the hid-node in either of the following ways:
- Terminal
   - Run the following to start `hid-node` in terminal
   ```sh
   cosmovisor run start
   ```
- System Service
   - Add the [Cosmosvisor system service file](https://github.com/hypersign-protocol/hid-node/blob/main/contrib/hidnoded-cosmovisor.service) to `/etc/systemd/system` directory.
   - Open the system service file and make necessary changes in line 7, which will specify your `hid-node` config path.
   - Reload service files: `sudo systemctl daemon-reload`
   - To enable your service on every reboot: `sudo systemctl enable hidnoded-cosmovisor.service`
   - To start the service: `sudo systemctl start hidnoded-cosmovisor.service`
   - To check the status of service: `sudo systemctl status hidnoded-cosmovisor.service`
   - To restart the service: `sudo systemctl restart hidnoded-cosmovisor.service`

## Validator Setup (Post Genesis Stage)

- Follow the steps to install `hid-node` binary [here](#installation-steps)
- Initialize node
```sh
hid-noded init <validator-name>
```
- Replace the contents of your `${HOME}/.hid-noded/config/genesis.json` with that of `testnet/jagrat/final_genesis.json` from the `master` branch of [repository](https://github.com/hypersign-protocol/networks).
- Add `persistent_peers` or `seeds` in `${HOME}/.hid-noded/config/config.toml` from `testnet/jagrat/final_peers.json` from the `master` branch of [repository](https://github.com/hypersign-protocol/networks).
- Set the `minimum-gas-price` in `${HOME}/.hid-node/config/app.toml`. Example value: `0.02uhid`
- Download and Install Cosmovisor
```
wget https://github.com/cosmos/cosmos-sdk/releases/download/cosmovisor%2Fv1.2.0/cosmovisor-v1.2.0-linux-amd64.tar.gz && tar -C /usr/local/bin/ -xzf cosmovisor-v1.2.0-linux-amd64.tar.gz
```
- Export the following environment variables
```
export DAEMON_PATH=<complete path of hid-noded binary>
export DAEMON_HOME=$HOME/.hid-node
```
- Create a `cosmovisor` directory and copy the existing blockchain binary to the following location
```
mkdir -p $DAEMON_HOME/cosmovisor/genesis/bin
cp $DAEMON_PATH $DAEMON_HOME/cosmovisor/genesis/bin
```
- Start node using Cosmovisor
```shell
cosmovisor run start
```
- Generate keys by either running `hid-noded keys add <key-name>` or `hid-noded keys add <key-name> --recover` to regenerate keys with your [BIP39](https://github.com/bitcoin/bips/tree/master/bip-0039) mnemonic
- Acquire some $HID
- Send a validator creation transaction
```
hid-noded tx staking create-validator \
--from <key-name> \
--amount 1000000000000uhid \
--pubkey "$(hid-noded tendermint show-validator)" \
--chain-id jagrat \
--moniker="<validator-name>" \
--commission-max-change-rate=0.01 \
--commission-max-rate=1.0 \
--commission-rate=0.07 \
--min-self-delegation="500000000000" \
--details="XXXXXXXX" \
--security-contact="XXXXXXXX" \
--website="XXXXXXXX"
```
