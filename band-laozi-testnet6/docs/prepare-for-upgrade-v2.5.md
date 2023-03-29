# Bandchain: How to prepare for upgrade v2.5

This document describes methods how to prepare for upgrade v2.5

> Note: If you are not running node by Cosmovisor, please follow this guide first (Bandchain: [How to migrate from Bandd binary to Cosmovisor](https://github.com/bandprotocol/launch/blob/master/band-laozi-testnet6/docs/migrate-bandd-binary-to-cosmovisor.md))

### Step 1: Clone & Install the new BandChain Laozi
Make new bandd binary from chain v2.5.1

```
# Clone Bandchain Laozi version 2.5.1
source ~/.profile
cd ~
git clone https://github.com/bandprotocol/chain
cd chain
git fetch
git checkout v2.5.1

# Install binaries to $GOPATH/bin
make install

# Verify bandd version
bandd version
```

### Step 2: Provide new binary to Cosmovisor
Provide bandd binary to Cosmovisor

```
# Setup folder and provide new bandd binary for Cosmovisor
mkdir -p $HOME/.band/cosmovisor/upgrades/v2_5/bin
cp $HOME/go/bin/bandd $HOME/.band/cosmovisor/upgrades/v2_5/bin
```

That’s all! Now, you just have to wait until the upgrade comes.

## After the upgraded block

### Step 1: Restart Yoda
If you are a validator, we recommend restarting the Yoda service after the node is synced up.

```
# Restart yoda with gap time
sudo service yoda stop
sleep 10
sudo service yoda start
```
