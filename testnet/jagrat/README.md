### [Important]: Following instructions are not maintained. For updated instructions, check [here](https://docs.hypersign.id/validator/running-a-testnet-validator-node)

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
git checkout v0.1.0
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

## Validator Setup

- Follow the steps to install `hid-node` binary [here](#installation-steps)
- Initialize node
```sh
hid-noded init <validator-name>
```
- Replace the contents of your `${HOME}/.hid-noded/config/genesis.json` with that of `testnet/jagrat/final_genesis.json` from the `master` branch of [repository](https://github.com/hypersign-protocol/networks).
- Add `persistent_peers` in `${HOME}/.hid-noded/config/config.toml` from `testnet/jagrat/final_peers.json` from the `master` branch of [repository](https://github.com/hypersign-protocol/networks).
- Set the `minimum-gas-price` in `${HOME}/.hid-node/config/app.toml` to `0.02uhid`
- Download and Install Cosmovisor
```
wget https://github.com/cosmos/cosmos-sdk/releases/download/cosmovisor%2Fv1.2.0/cosmovisor-v1.2.0-linux-amd64.tar.gz && tar -C /usr/local/bin/ -xzf cosmovisor-v1.2.0-linux-amd64.tar.gz
```
- Export the following environment variables needed by `cosmovisor`
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

**Run Node using Cosmovisor**

- You can run the `hid-node` in either of the following ways:
   - Terminal
      - Run the following to start `hid-node` in terminal
      ```sh
      cosmovisor run start
      ```
      > Note: `cosmovisor` looks for the environment variable `DAEMON_NAME` and `DAEMON_HOME`. Make sure to run the above command in the same terminal window where the said environment variables are set.
   - System Service
      - Change directory: `cd /etc/systemd/system`
      - Add the [Cosmovisor system service file](https://github.com/hypersign-protocol/hid-node/blob/main/contrib/hidnoded-cosmovisor.service) to `/etc/systemd/system` directory.
      - Open the system service file and make necessary changes in line 7, which will specify your `hid-node` config path.
      - Reload service files: `sudo systemctl daemon-reload`
      - To enable your service on every reboot: `sudo systemctl enable hidnoded-cosmovisor.service`
      - To start the service: `sudo systemctl start hidnoded-cosmovisor.service`
      - To check the status of service: `sudo systemctl status hidnoded-cosmovisor.service`
      - To restart the service: `sudo systemctl restart hidnoded-cosmovisor.service`

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
