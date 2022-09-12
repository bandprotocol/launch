# Bandchain Laozi Testnet #5: How to Join as a Validator

This document describes methods on how to join as a validator in Laozi testnet #5.

## Step 1: Set Up Validator Node

This step provides procedures to install Bandchain's executable and sync blocks with other peers.

Assuming to run on Ubuntu 22.04 LTS allowing connection on port `26656` for P2P connection.

Before beginning instructions, following variables should be set to be used in further instructions. **Please make sure that these variables is set everytime when using the new shell session**.

```bash=
# Chain ID of testnet #5
export CHAIN_ID=band-laozi-testnet5
# Wallet name to be used as validator's account, please change this into your name (no whitespace).
export WALLET_NAME=<YOUR_WALLET_NAME>
# Name of your validator node, please change this into your name.
export MONIKER=<YOUR_MONIKER>
# Persistent peers for P2P communication
export SEEDS="472eab3d2a3816fdf63da7fe88b6ae91ba0c88f0@34.87.163.119:26656,d3ea8e468139910e77fcf3a493f3811ca01dd814@34.85.198.8:26656"
# URL of genesis file for Laozi testnet #5
export GENESIS_FILE_URL=https://raw.githubusercontent.com/bandprotocol/launch/master/band-laozi-testnet5/genesis.json
# Data sources/oracle scripts files
export BIN_FILES_URL=https://raw.githubusercontent.com/bandprotocol/launch/master/band-laozi-testnet5/files.tar.gz
# Faucet endpoint
export FAUCET_URL=https://laozi-testnet5.bandchain.org/faucet
```

### Step 1.1: Install Prerequisites

The following application is required for Building and running Bandchain node.

- make, gcc, g++ (can be obtained from `build-essential` package on linux)
- wget, curl for downloading files

```bash=
# install required tools
sudo apt-get update && \
sudo apt-get upgrade -y && \
sudo apt-get install -y build-essential curl wget
```

- Go 1.18.3

```bash=
# Install Go 1.18.3
wget https://go.dev/dl/go1.18.3.linux-amd64.tar.gz
tar xf go1.18.3.linux-amd64.tar.gz
sudo mv go /usr/local/go

# Set Go path to $PATH variable and Cosmovisor variables
echo "export PATH=\$PATH:/usr/local/go/bin:~/go/bin" >> $HOME/.profile
echo "export DAEMON_NAME=bandd" >> $HOME/.profile
echo "export DAEMON_HOME=$HOME/.band" >> $HOME/.profile
source ~/.profile
```

Go binary should be at `/usr/local/go/bin` and any executable compiled by `go install` command should be at `~/go/bin`

### Step 1.2: Clone & Install Bandchain Laozi

```bash=
# Clone Bandchain Laozi version v2.4.0-rc1
git clone https://github.com/bandprotocol/chain
cd chain
git checkout v2.4.0-rc1

# Install binaries to $GOPATH/bin
make install
```

### Step 1.3: Initialize the Bandchain and download genesis file

```bash=
cd $HOME

# Initialize configuration and genesis state
bandd init --chain-id $CHAIN_ID "$MONIKER"

# Replace genesis file with our genesis file
wget $GENESIS_FILE_URL -O $HOME/.band/config/genesis.json

# Download data sources / oracle scripts files, and store in $HOME/.band/files
wget -qO- $BIN_FILES_URL | tar xvz -C $HOME/.band/

# Create new account
bandd keys add $WALLET_NAME

```

### Step 1.4: Setup seeds

```bash=
# Add persistent peers to config.toml
sed -E -i \
  "s/seeds = \".*\"/seeds = \"${SEEDS}\"/" \
  $HOME/.band/config/config.toml
```

### Step 1.5: Setup Cosmovisor

```bash=
# Install Cosmovisor
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@v1.0.0

# Setup folder and provide bandd binary for Cosmovisor
mkdir -p $HOME/.band/cosmovisor/genesis/bin
mkdir -p $HOME/.band/cosmovisor/upgrades
cp $HOME/go/bin/bandd $HOME/.band/cosmovisor/genesis/bin
```

### Step 1.6: Create Bandchain service

We do recommend to run bandchain node as a daemon, which can be setup using `systemctl`. Run the following command to create a new daemon for `bandd` through `cosmovisor` (This script work on non-root user).

```bash=
# Write bandd service file to /etc/systemd/system/bandd.service
export USERNAME=$(whoami)
sudo -E bash -c 'cat << EOF > /etc/systemd/system/bandd.service
[Unit]
Description=BandChain Node Daemon
After=network-online.target

[Service]
Environment="DAEMON_NAME=bandd"
Environment="DAEMON_HOME=${HOME}/.band"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="UNSAFE_SKIP_BACKUP=false"
User=$USERNAME
ExecStart=${HOME}/go/bin/cosmovisor start
Restart=always
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target

EOF'
```

## Step 2: Setup Yoda

Based on design, validator need to send a transaction to submit reports based on certain oracle requests. The validator should send transactions to submit reports within specified timeframe. However, the method is quite tedious. Therefore, we have develop an application called `Yoda`, which is a bot application that help validator automatically listen new oracle requests on Bandchain, execute data sources, and submit report to Bandchain, so validators don't have to send the transactions manually.

### Step 2.1: Prerequisites

There is an update in executor configuration. You can **set up a new executor** by using the instructions on following pages (select either one of these methods):

- [AWS Lambda Function Setup](https://github.com/bandprotocol/data-source-runtime/wiki/Setup-Yoda-Executor-Using-AWS-Lambda)
- [Google Cloud Function Setup](https://github.com/bandprotocol/data-source-runtime/wiki/Setup-Yoda-Executor-Using-Google-Cloud-Function)

On the other hand, you can **update the executor** with the latest configuration:

- [AWS Lambda Function Update](https://github.com/bandprotocol/data-source-runtime/wiki/Update-Yoda-Executor-on-AWS-Lambda)
- [Google Cloud Function Update](https://github.com/bandprotocol/data-source-runtime/wiki/Update-Yoda-Executor-on-Google-Cloud-Function)

Then, check the Yoda version that we have compiled. It should be `v2.4.0-rc0`.

```bash=
yoda version
# v2.4.0-rc0
```

### Step 2.2: Configure Yoda

Firstly, configure Yoda's basic configurations

```bash=
rm -rf ~/.yoda # clear old config if exist
yoda config chain-id $CHAIN_ID
yoda config node http://localhost:26657
yoda config broadcast-timeout "5m"
yoda config rpc-poll-interval "1s"
yoda config max-try 5
yoda config validator $(bandd keys show $WALLET_NAME -a --bech val)
```

Secondly, add multiple reporter accounts to allow Yoda submitting transactions concurrently.

```bash=
yoda keys add REPORTER_1
yoda keys add REPORTER_2
yoda keys add REPORTER_3
yoda keys add REPORTER_4
yoda keys add REPORTER_5
```

Thirdly, config Lambda Executor endpoint

```bash=
export EXECUTOR_URL=<YOUR_EXECUTOR_URL>
yoda config executor "rest:${EXECUTOR_URL}?timeout=10s"
```

### Step 2.3: Start Yoda

We also do recommend to use `systemctl` the same as `bandd`.

```bash=
# Write yoda service to /etc/systemd/system/yoda.service
export USERNAME=$(whoami)
sudo -E bash -c 'cat << EOF > /etc/systemd/system/yoda.service
[Unit]
Description=Yoda Daemon
After=network-online.target

[Service]
User=$USERNAME
ExecStart=/home/$USERNAME/go/bin/yoda run
Restart=always
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF'
```

To register both `bandd` and `yoda` services, run the following commands.

```bash=
# Register bandd daemon to systemctl
sudo systemctl enable bandd
# Register yoda to systemctl
sudo systemctl enable yoda
```

Then start `bandd` and `yoda` services

```bash=
# Start bandd daemon
sudo systemctl start bandd
# Start yoda daemon
sudo systemctl start yoda
```

Once `bandd` service has been started, logs can be queried by running `journalctl -u bandd.service -f` command. You will see your node beginning to sync.

After `yoda` service has been started, logs can be queried by running `journalctl -u yoda.service -f` command. Log should be similar to the following log example below. Once verified, you can stop tailing the log by typing `Control-C`.

```bash=
... systemd[...]: Started Yoda Daemon.
... yoda[...]: I[...] ⭐  Creating HTTP client with node URI: tcp://localhost:26657
... yoda[...]: I[...] 🚀  Starting WebSocket subscriber
... yoda[...]: I[...] 👂  Subscribing to events with query: tm.event = 'Tx'...
```

### Step 2.4: Wait for latest blocks to be synced

**This is an important step.** We should wait for newly started Bandchain node to sync their blocks until the latest block is reached. The latest block can be checked on [this Block Explorer](https://laozi-testnet5.cosmoscan.io).

## Step 3: Become a Validator

This step provides procedures to register the node as a validator.

### Step 3.1: Create new Account to be Used as Validator

```bash=
# Request new coins from faucet
curl --location --request POST "${FAUCET_URL}" \
--header 'Content-Type: application/json' \
--data-raw "{
 \"address\": \"$(bandd keys show $WALLET_NAME -a)\"
}"
```

### Step 3.2: Stake Tokens with the Validator Account

```bash=
bandd tx staking create-validator \
    --amount 3000000uband \
    --commission-max-change-rate 0.01 \
    --commission-max-rate 0.2 \
    --commission-rate 0.1 \
    --from $WALLET_NAME \
    --min-self-delegation 1 \
    --moniker "$MONIKER" \
    --pubkey $(bandd tendermint show-validator) \
    --chain-id $CHAIN_ID
```

After became a validator, the validator node will be shown on Block Explorer [here](https://laozi-testnet5.cosmoscan.io/validators).

### Step 3.3: Register Reporters and Become Oracle Provider

Now, Yoda have multiple reporters. In order to grant the reporters to report data for the validator, the following commands should be run.

Firstly, reporter accounts must be create on Bandchain by supplying some small amount of BAND tokens.

```bash=
# Send 1uband from a wallet to each reporter.
bandd tx multi-send 1uband $(yoda keys list -a) \
  --from $WALLET_NAME \
  --chain-id $CHAIN_ID
```

Secondly, grant all reporters for the validator, so that oracle requests for validator can be sent by the reporters.

```bash=
bandd tx oracle add-reporters $(yoda keys list -a) \
  --from $WALLET_NAME \
  --chain-id $CHAIN_ID
```

Finally, activate the validator to become an oracle provider

```bash=
bandd tx oracle activate \
  --from $WALLET_NAME \
  --chain-id $CHAIN_ID
```

If all procedures are successful, then oracle provider status for the validator should be `active`.

```bash=
bandd query oracle validator $(bandd keys show -a $WALLET_NAME --bech val)

# {
#   "is_active": true,
#   "since": ...
# }
```

And now you have become a validator on Bandchain Laozi testnet #5.

Happy staking :chart_with_upwards_trend:, and may the **HODL** be with you.
