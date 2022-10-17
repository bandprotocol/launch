# Bandchain: How to prepare for upgrade v2.4

This document describes methods how to prepare for upgrade v2.4

> Note: If you are not running node by Cosmovisor, please follow this guide first (Bandchain: [How to migrate from Bandd binary to Cosmovisor](https://github.com/bandprotocol/launch/blob/master/band-laozi-testnet6/docs/migrate-bandd-binary-to-cosmovisor.md))


### Step 1: Update Go version to 1.19.1 and reinstall binary
Remove your old go version and install go version 1.19.1 into your system. 

> Note: You can skip this step if you are running Go version 1.19.1

```
cd ~
source ~/.profile

# Remove old Go version
sudo rm -rf /usr/local/go
sudo rm -rf ~/go

# Install Go 1.19.1
wget https://go.dev/dl/go1.19.1.linux-amd64.tar.gz
tar xf go1.19.1.linux-amd64.tar.gz
sudo mv go /usr/local/go

# Reinstall cosmovisor
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@v1.0.0

# Reinstall binary for current version (bandd, yoda)
cd ~
rm -rf chain
git clone https://github.com/bandprotocol/chain
cd chain
git checkout v2.3.6
make install
```

### Step 2: Clone & Install the new Bandchain Laozi
Make new bandd binary from chain v2.4.0

```
# Clone Bandchain Laozi version 2.4.0
cd ~
rm -rf chain
git clone https://github.com/bandprotocol/chain
cd chain
git checkout v2.4.0

# Install binaries to $GOPATH/bin
make install

# Verify bandd version
bandd version
```

### Step 3: Install and provide new binary to Cosmovisor
Provide bandd binary to Cosmovisor

```
# Setup folder and provide new bandd binary for Cosmovisor
mkdir -p $HOME/.band/cosmovisor/upgrades/v2_4/bin
cp $HOME/go/bin/bandd $DAEMON_HOME/cosmovisor/upgrades/v2_4/bin
```

### Step 4: Restart daemon service

```
# Restart bandd
sudo systemctl restart bandd
```

If you are a validator, we recommend restarting the Yoda service after the node is synced up.

```
# Restart yoda
sudo systemctl restart yoda
```

That’s all! Now, you just have to wait until the upgrade comes.

## After the upgraded block

#### Step 1: Restart Yoda
If you are a validator, we recommend restarting the Yoda service after the node is synced up.

```
# Restart yoda
sudo systemctl restart yoda
```




