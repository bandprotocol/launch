# Setup Cylinder

The Cylinder program is tailored for specific members participating in threshold signature scheme (TSS) message signing. It simplifies the signing process, enabling secure and efficient collaboration among group members.

The document is written based on the assumption that the program runs on Ubuntu 24.04 LTS.

## Prerequisites

- **Finished** [Joined as a testnet validator](https://github.com/bandprotocol/launch/blob/master/band-v3-testnet-1/README.md)
- **Selected as a TSS member**
- **Latest blocks are fully synced**

## Setup Variables

Before beginning instructions, the following variables should be set to be used in further instructions. Please make sure that these variables are set every time when using the new shell session.

```bash
# Chain ID of Band V3 Testnet #1
export CHAIN_ID="band-v3-testnet-1"
# Wallet name that used as your validator's account.
export WALLET_NAME=<YOUR_WALLET_NAME>
# Reload from the .profile file
source $HOME/.profile
```

## Step 1: Configure general settings

Firstly, configure Cylinder's basic configurations

```bash
cylinder config chain-id $CHAIN_ID
cylinder config node http://localhost:26657
cylinder config granter $(bandd keys show $WALLET_NAME -a)
cylinder config gas-prices "0uband"
cylinder config max-messages 10
cylinder config broadcast-timeout "5m"
cylinder config rpc-poll-interval "1s"
cylinder config max-try 5
cylinder config min-de 100
cylinder config gas-adjust-start 1.6
cylinder config gas-adjust-step 0.2
cylinder config random-secret "$(openssl rand -hex 32)"
cylinder config checking-de-interval "1m"
```

Secondly, add multiple signer accounts.

```bash
cylinder keys add signer1
cylinder keys add signer2
cylinder keys add signer3
cylinder keys add signer4
cylinder keys add signer5
```

To check that if the signer account is added into the program, run the following command
`cylinder keys list`.

## Step 2: Set grantee and send tokens to the signer account.

Firstly, signer accounts must be create on Bandchain by supplying some small amount of BAND tokens.

```bash
bandd tx bank multi-send $WALLET_NAME $(cylinder keys list -a) 1uband \
	--gas-prices 0.0025uband \
	--chain-id $CHAIN_ID \
	-b sync \
	-y
```

Secondly, designate all signer account as grantees of the granter account.

```bash
bandd tx tss add-grantees $(cylinder keys list -a) --gas-prices 0.0025uband --chain-id $CHAIN_ID --gas 800000 --from $WALLET_NAME -b sync -y
```

## Step 3: Register Cylinder service

We also do recommend to use `systemctl` the same as `bandd`.

```bash
# Write cylinder service to /etc/systemd/system/cylinder.service
export USERNAME=$(whoami)
sudo -E bash -c 'cat << EOF > /etc/systemd/system/cylinder.service
[Unit]
Description=Cylinder Daemon
After=network-online.target

[Service]
User=$USERNAME
ExecStart=/home/$USERNAME/go/bin/cylinder run
Restart=always
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF'
```

To register `cylinder` services, run the following commands.

```bash
# Register cylinder to systemctl
sudo systemctl enable cylinder
```

## Step 4: Start Cylinder

```bash
# Start cylinder daemon
sudo systemctl start cylinder
```

After a service has been started, logs can be queried by running `journalctl -u cylinder.service -f` command. Log should be similar to the following log example below. Once verified, you can stop tailing the log by typing `Control-C`.

And now you have become a TSS member on Band V3 Testnet #1.
