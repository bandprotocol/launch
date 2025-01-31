# Export & Import Cylinder groups' keys

Once you have joined the TSS group, you can export the group's keys for future import. In case you set up a new node, these exported keys can be imported to ensure seamless participation in the TSS group.

Note 1: Always export and back up your keys each time you join a new group to prevent data loss.
Note 2: Do not migrate your Cylinder while you are in the group creation process, as this may cause the group creation to fail.

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

## Export Cylinder groups' keys

### Step 1: Stop Cylinder

If your cylinder is running, stop it before exporting keys

```bash
sudo systemctl stop cylinder
```


### Step 2: Export keys

```bash
cylinder export groups --all --output cylinder_key.json
```

Your keys will be at `cylinder_key.json`, save it in the safe place.

### Step 3: Start Cylinder

```bash
# Start cylinder daemon
sudo systemctl start cylinder
```

After a service has been started, logs can be queried by running `journalctl -u cylinder.service -f` command. Once verified, you can stop tailing the log by typing Control-C.

### Step 4: Activate group members status

If you stop and restart Cylinder, you may be deactivated as a TSS group member. You can verify your status by running the following command:

```bash
bandd query bandtss member $(bandd keys show $WALLET_NAME -a) --chain-id $CHAIN_ID
```

If your status `is_active` is `false` (or not shown in the output, which also means `false`), you can activate after **10 minutes** with this command.

```bash
bandd tx bandtss activate --from $WALLET_NAME \
    --gas-prices 0.0025uband \
	--chain-id $CHAIN_ID \
	-b sync \
	-y
```

## Import Cylinder groups' keys

If you are setting up a new node or have lost Cylinder data, you can import the TSS group's keys to resume participation in the TSS group.

### Step 1: Stop Cylinder

If your cylinder is running, stop it before importing keys

```bash
sudo systemctl stop cylinder
```

### Step 2: Import keys

```bash
cylinder import groups <PATH_TO_YOUR_KEYS_FILE>
```

### Step 3: Reset your DE information on chain

```bash
bandd tx tss reset-de --from $WALLET_NAME \
    --gas-prices 0.0025uband \
	--gas 600000
	--chain-id $CHAIN_ID \
	-b sync \
	-y
```

### Step 4: Start Cylinder

```bash
# Start cylinder daemon
sudo systemctl start cylinder
```

After a service has been started, logs can be queried by running `journalctl -u cylinder.service -f` command. Once verified, you can stop tailing the log by typing Control-C.

### Step 5: Activate group members status

After restart Cylinder, you may be deactivated as a TSS group member. You can verify your status by running the following command:

```bash
bandd query bandtss member $(bandd keys show $WALLET_NAME -a) --chain-id $CHAIN_ID
```

If your status `is_active` is `false` (or not shown in the output, which also means `false`), you can activate after **10 minutes** with this command.

```bash
bandd tx bandtss activate --from $WALLET_NAME \
    --gas-prices 0.0025uband \
	--chain-id $CHAIN_ID \
	-b sync \
	-y
```
