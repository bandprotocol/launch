# Setup Cylinder

The Cylinder program is tailored for specific members participating in threshold signature scheme (TSS) message signing. It simplifies the signing process, enabling secure and efficient collaboration among group members.

If you’re not a selected member yet, you can still run the Cylinder program in advance to be prepared.

The document is written based on the assumption that the program runs on Ubuntu 24.04 LTS.

Before beginning instructions, the following variables should be set to be used in further instructions. Please make sure that these variables are set every time when using the new shell session.

```bash
# Chain ID of Band V3 Testnet #1
export CHAIN_ID="band-v3-testnet-1"
# Wallet name to be used as a granter account, please change this into your name (no whitespace).
export WALLET_NAME=<YOUR_WALLET_NAME>
# url for connecting to the target chain, e.g. tcp://localhost:26657
export RPC_URL=<YOUR_NODE_RPC_URL>
```

## Step 1: Provide granter account to the system

Create a new account using the command below.

```bash
bandd keys add $WALLET_NAME
```

If you choose to use an existing account, add the `--recover` flag to the command mentioned above.

whether a granter account is created or recovered, please ensure that the account has sufficient tokens to cover transaction fees.

To verify that the new account has been added to the system, run `bandd keys show $WALLET_NAME -a` to display the wallet address associated with the account.

## Step 2: Configure general settings

Run the following command to set the configuration of the cylinder program and add signer account to the program.

```bash
cylinder config chain-id $CHAIN_ID
cylinder config node $RPC_URL
cylinder config granter $(bandd keys show $WALLET_NAME -a)
cylinder config gas-prices "0uband"
cylinder config max-messages 20
cylinder config broadcast-timeout "5m"
cylinder config rpc-poll-interval "1s"
cylinder config max-try 5
cylinder config min-de 100
cylinder config gas-adjust-start 1.6
cylinder config gas-adjust-step 0.2
cylinder config random-secret "$(openssl rand -hex 32)"
cylinder config checking-de-interval "1m"

cylinder keys add signer1
cylinder keys add signer2
cylinder keys add signer3
cylinder keys add signer4
```

below is the meaning of the configuration of the system

```go
type Config struct {
	ChainID          	string        		// ChainID of the target chain
	NodeURI          	string        		// Remote RPC URI of BandChain node to connect to
	Granter          	string        		// The granter address
	GasPrices        	string        		// Gas prices of the transaction
	LogLevel         	string        		// Log level of the logger
	MaxMessages      	uint64        		// The maximum number of messages in a transaction
	BroadcastTimeout 	time.Duration 		// The time that cylinder will wait for tx commit
	RPCPollInterval  	time.Duration 		// The duration of rpc poll interval
	MaxTry           	uint64        		// The maximum number of tries to submit a report transaction
	MinDE            	uint64        		// The minimum number of DE
	GasAdjustStart   	float64       		// The start value of gas adjustment
	GasAdjustStep    	float64       		// The increment step of gas adjustment
	RandomSecret     	tss.Scalar    		// The secret value that is used for random D,E
	CheckingDEInterval 	time.Duration  		// The interval for updating DE
}
```

To check that if the signer account is added into the program, run the following command
`cylinder keys list`. The configuration is updated in the `$CYLINDER_HOME_PATH/config.yaml`

## Step 3: Set grantee and send tokens to the signer account.

Run the following commands to send 1 BAND to the predefined signer accounts and designate them as grantees of the granter account.

```bash
bandd tx bank multi-send 1000000uband $(cylinder keys list -a) --gas-prices 0.0025uband --chain-id $CHAIN_ID --from $WALLET_NAME -b sync -y --node $RPC_URL
```

```bash
bandd tx tss add-grantees $(cylinder keys list -a) --gas-prices 0.0025uband --chain-id $CHAIN_ID --gas 350000 --from $WALLET_NAME -b sync -y --node $RPC_URL
```

## Step 4: Register Cylinder service

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

## Step 5: Start Cylinder

```bash
# Start cylinder daemon
sudo systemctl start cylinder
```

After a service has been started, logs can be queried by running `journalctl -u cylinder.service -f` command. Log should be similar to the following log example below. Once verified, you can stop tailing the log by typing `Control-C`.

And now you have become a TSS member on Band V3 Testnet #1.
