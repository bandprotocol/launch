# How to update node after the upgrade was successful

**This step should be executed after the upgrade height is reached only**

After the chain reaches upgrade height, your old bandd should be stopped automatically by the upgrade proposal. After that, you have to change bandd binary to the new version by following steps.

**Noted:** If it isn't a validator node, please skip all steps about `yoda`.

## Step 1: Stop Bandd and Yoda daemon service

To prevent systemd from restarting Bandd and Yoda service, we have to stop them first.

```bash
sudo systemctl stop bandd
sudo systemctl stop yoda
```

## Step 2: Update Go version to 1.19.1

Remove your old go version and install go version 1.19.1 into your system.

```bash
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

## Step 3: Install the new Bandchain Laozi

Make new Bandd binary from chain v2.4.1

```bash
cd chain
git fetch && git checkout v2.4.1
make install
```

## Step 4: Start daemon service

### Restart bandd

```bash
sudo systemctl start bandd
```

After the node is synced up, you might need to restart Yoda service.

### Restart yoda

```bash
sudo systemctl start yoda
```

That’s all! Now, you are running the newest Bandchain version (v2.4).
