# Setup Cylinder

The Cylinder program is tailored for specific members participating in threshold signature scheme (TSS) message signing. It simplifies the signing process, enabling secure and efficient collaboration among group members.

If you’re not a selected member yet, you can still run the Cylinder program in advance to be prepared.

## Step 1: Configure Cylinder

Firstly, configure Cylinder's basic configurations

```bash=
cylinder config chain-id $CHAIN_ID
cylinder config granter $(bandd keys show $WALLET_NAME -a)
cylinder config max-messages 20
cylinder config broadcast-timeout "5m"
cylinder config rpc-poll-interval "1s"
cylinder config max-try 5
cylinder config gas-prices "0uband"
cylinder config min-de 100
cylinder config gas-adjust-start 1.6
cylinder config gas-adjust-step 0.2
cylinder config random-secret "$(openssl rand -hex 32)"
cylinder config checking-de-interval "1m"
```

Secondly, add multiple signer accounts to allow Cylinder submitting transactions concurrently.

```bash=
cylinder keys add SIGNER_1
cylinder keys add SIGNER_2
cylinder keys add SIGNER_3
cylinder keys add SIGNER_4
cylinder keys add SIGNER_5
```

## Step 2: Start Cylinder

```bash=
# Start cylinder daemon
sudo systemctl start cylinder
```

After a service has been started, logs can be queried by running `journalctl -u cylinder.service -f` command. Log should be similar to the following log example below. Once verified, you can stop tailing the log by typing `Control-C`.

## Step 3: Register Cylinder grantees

Lastly, grantees accounts must be create on BandChain by supplying some small amount of BAND tokens.

```bash=
bandd tx bank multi-send $WALLET_NAME $(cylinder keys list -a --home $HOME_PATH) 1uband \
  --chain-id $CHAIN_ID \
  --gas-prices 0.0025uband \
  -b sync \
  -y
```

Then, grant all grantees for the validator.

```bash=
bandd tx tss add-grantees $(cylinder keys list -a --home $HOME_PATH) \
  --from $WALLET_NAME \
  --chain-id $CHAIN_ID \
  --gas-prices 0.0025uband \
  --gas 1000000 \
  -b sync \
  -y
```

And now you have become a TSS member on Bandchain Laozi testnet #7.
