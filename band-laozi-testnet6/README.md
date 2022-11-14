# Bandchain Laozi Testnet #6: How to Join as a Validator

This document describes methods on how to join as a validator in Laozi testnet #6 without using State Sync. If you want to join using State Sync please follow this document [How to Join as a Validator using State Sync](https://github.com/bandprotocol/launch/blob/master/band-laozi-testnet6/docs/how-to-join-as-a-validator-using-statesync.md)

## Step 1: Set Up Validator Node

This step provides procedures to install Bandchain's executable and sync blocks with other peers.

Assuming to run on Ubuntu 22.04 LTS allowing connection on port `26656` for P2P connection.

Before beginning instructions, following variables should be set to be used in further instructions. **Please make sure that these variables is set everytime when using the new shell session**.

```bash=
# Chain ID of testnet #6
export CHAIN_ID="band-laozi-testnet6"
# Wallet name to be used as validator's account, please change this into your name (no whitespace).
export WALLET_NAME=<YOUR_WALLET_NAME>
# Name of your validator node, please change this into your name.
export MONIKER=<YOUR_MONIKER>
# Seed and persistent peers for P2P communication
export SEEDS="da61931cbbbb2b62dbe7c470d049126cf365d257@35.213.165.61:26656,fffd730672f04d5dc065fa9afce2eb1d6bc4d150@35.212.60.28:26656"
# URL of genesis file for Laozi testnet #6
export GENESIS_FILE_URL=https://raw.githubusercontent.com/bandprotocol/launch/master/band-laozi-testnet6/genesis.json
# Data sources/oracle scripts files
export BIN_FILES_URL=https://raw.githubusercontent.com/bandprotocol/launch/master/band-laozi-testnet6/files.tar.gz
# Faucet endpoint
export FAUCET_URL=https://laozi-testnet6.bandchain.org/faucet
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

- Go 1.16.7
```bash=
# Install Go 1.16.7
wget https://go.dev/dl/go1.16.7.linux-amd64.tar.gz
tar xf go1.16.7.linux-amd64.tar.gz
sudo mv go /usr/local/go

# Set Go path to $PATH variable
echo "export PATH=\$PATH:/usr/local/go/bin:~/go/bin" >> $HOME/.profile
source ~/.profile
```

Go binary should be at `/usr/local/go/bin` and any executable compiled by `go install` command should be at `~/go/bin`

### Step 1.2: Clone & Install Bandchain Laozi

```bash=
# Clone Bandchain Laozi version v2.3.6
git clone https://github.com/bandprotocol/chain
cd chain
git checkout v2.3.6

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

### Step 1.4: Setup seeds and minimum gas price

```bash=
# Add seeds to config.toml
sed -E -i \
  "s/seeds = \".*\"/seeds = \"${SEEDS}\"/" \
  $HOME/.band/config/config.toml
  
# Add minimum gas price
sed -E -i \
  "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025uband\"/" \
  $HOME/.band/config/app.toml
```
## Step 2: Setup Cosmovisor
This step provides procedures to setup Cosmovisor. Cosmovisor is a small process manager for Cosmos SDK application binaries that monitors the governance module via stdout for incoming chain upgrade proposals

### Step 2.1: Setup environment variables
Add required environment variables for Cosmovisor into your profile

```bash=
cd ~
echo "export DAEMON_NAME=bandd" >> ~/.profile
echo "export DAEMON_HOME=$HOME/.band" >> ~/.profile
source ~/.profile
```
### Step 2.2: Setup Cosmovisor
Install Cosmovisor and provide bandd binary to Cosmovisor

```bash=
# Install Cosmovisor
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@v1.0.0

# Setup folder and provide bandd binary for Cosmovisor
mkdir -p $HOME/.band/cosmovisor/genesis/bin
mkdir -p $HOME/.band/cosmovisor/upgrades
cp $HOME/go/bin/bandd $HOME/.band/cosmovisor/genesis/bin
```

### Step 2.3: Update Bandchain service
In this step, we will write the daemon service to use Cosmovisor instead of Bandd binary.

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
Environment="UNSAFE_SKIP_BACKUP=true"
User=$USERNAME
ExecStart=${HOME}/go/bin/cosmovisor start
Restart=always
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF'
```

## Step 3: Provide bandd v2.4 for Cosmovisor
This Step provides procedures to provide bandd version 2.4 for Cosmovisor  to upgrade when it's reach the upgrade height

### Step 3.1: Update Go version to 1.19.1 and Reinstall Binary
Remove your old go version and install go version 1.19.1 into your system. 

> Note: You can skip this step if you are running Go version 1.19.1

##### Update Go version
```bash=
cd ~
source ~/.profile

# Remove old Go version
sudo rm -rf /usr/local/go
sudo rm -rf ~/go

# Install Go 1.19.1
wget https://go.dev/dl/go1.19.1.linux-amd64.tar.gz
tar xf go1.19.1.linux-amd64.tar.gz
sudo mv go /usr/local/go

# Verify go version
go version
```

##### Reinstall Binary
```bash=
# Reinstall cosmovisor
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@v1.0.0

# Reinstall binary for current version (bandd, yoda)
cd ~
cd chain
git checkout v2.3.6
make install
```

### Step 3.2: Clone & Install the new Bandchain Laozi
Make new bandd binary from chain v2.4.1

```bash=
# Clone Bandchain Laozi version 2.4.1
cd ~
cd chain
git checkout v2.4.1

# Install binaries to $GOPATH/bin
make install

# Verify bandd version
bandd version
```

### Step 3.3: Install and provide new binary to Cosmovisor
Provide bandd binary to Cosmovisor

```bash=
# Setup folder and provide new bandd binary for Cosmovisor
mkdir -p $HOME/.band/cosmovisor/upgrades/v2_4/bin
cp $HOME/go/bin/bandd $DAEMON_HOME/cosmovisor/upgrades/v2_4/bin
```

## Step 4: Setup Yoda

Based on design, validator need to send a transaction to submit reports based on certain oracle requests. The validator should send transactions to submit reports within specified timeframe. However, the method is quite tedious. Therefore, we have develop an application called `Yoda`, which is a bot application that help validator automatically listen new oracle requests on Bandchain, execute data sources, and submit report to Bandchain, so validators don't have to send the transactions manually.

### Step 4.1: Prerequisites

There is an update in the executor configuration. You can **set up a new executor** by using the instructions on following pages (select either one of these methods):

- [AWS Lambda Function Setup](https://github.com/bandprotocol/data-source-runtime/wiki/Setup-Yoda-Executor-Using-AWS-Lambda)
- [Google Cloud Function Setup](https://github.com/bandprotocol/data-source-runtime/wiki/Setup-Yoda-Executor-Using-Google-Cloud-Function)

On the other hand, you can **update the executor** with the latest configuration:
- [AWS Lambda Function Update](https://github.com/bandprotocol/data-source-runtime/wiki/Update-Yoda-Executor-on-AWS-Lambda)
- [Google Cloud Function Update](https://github.com/bandprotocol/data-source-runtime/wiki/Update-Yoda-Executor-on-Google-Cloud-Function)

**Noted** You can use the old executor on laozi-testnet5 (no change from that version)

Then, check Yoda version that we have compiled. It should be `v2.4.1`.

```bash=
yoda version
# v2.4.1
```

### Step 4.2: Configure Yoda

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
### Step 4.3: Start Yoda

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

### Step 4.4: Wait for latest blocks to be synced

**This is an important step.** We should wait for newly started Bandchain node to sync their blocks until the latest block is reached. The latest block can be checked on [this Block Explorer](https://laozi-testnet6.cosmoscan.io/).

## Step 5: Become a Validator

This step provides procedures to register the node as a validator.

### Step 5.1: Create new Account to be Used as Validator

```bash=
# Request new coins from faucet
curl --location --request POST "${FAUCET_URL}" \
--header 'Content-Type: application/json' \
--data-raw "{
 \"address\": \"$(bandd keys show $WALLET_NAME -a)\"
}"
```

### Step 5.2: Stake Tokens with the Validator Account

```bash=
bandd tx staking create-validator \
    --amount 3000000uband \
    --commission-max-change-rate 0.01 \
    --commission-max-rate 0.2 \
    --commission-rate 0.1 \
    --from $WALLET_NAME \
    --gas-prices 0.0025uband \
    --min-self-delegation 1 \
    --moniker "$MONIKER" \
    --pubkey $(bandd tendermint show-validator) \
    --chain-id $CHAIN_ID
```

After became a validator, the validator node will be shown on Block Explorer [here](https://laozi-testnet6.cosmoscan.io/validators).


### Step 5.3: Register Reporters and Become Oracle Provider

Now, Yoda have multiple reporters. In order to grant the reporters to report data for the validator, the following commands should be run.

Firstly, reporter accounts must be create on Bandchain by supplying some small amount of BAND tokens.

```bash=
# Send 1uband from a wallet to each reporter.
bandd tx multi-send 1uband $(yoda keys list -a) \
  --from $WALLET_NAME \
  --gas-prices 0.0025uband \
  --chain-id $CHAIN_ID
```

Secondly, grant all reporters for the validator, so that oracle requests for validator can be sent by the reporters.

```bash=
bandd tx oracle add-reporters $(yoda keys list -a) \
  --from $WALLET_NAME \
  --gas-prices 0.0025uband \
  --chain-id $CHAIN_ID
```

Finally, activate the validator to become an oracle provider

```bash=
bandd tx oracle activate \
  --from $WALLET_NAME \
  --gas-prices 0.0025uband \
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

And now you have become a validator on Bandchain Laozi testnet #6.

Happy staking :chart_with_upwards_trend:, and may the **HODL** be with you.
