# How to update node after the upgrade was successful

**This step should be executed after the upgrade height is reached only (This upgrade is expected to occur at block 16562500. Estimate date is April 27, 2023 at 14:00 UTC)**

After the chain reaches upgrade height, your old bandd should be stopped automatically by the upgrade proposal. After that, you have to change bandd binary to the new version by following steps.

**Noted:** If it isn't a validator node, please skip all steps about `yoda`.

## Step 1: Stop Bandd and Yoda daemon service

To prevent systemd from restarting Bandd and Yoda service, we have to stop them first.

```bash
sudo systemctl stop bandd
sudo systemctl stop yoda
```

## Step 2: Install the new Bandchain Laozi

Make new Bandd binary from chain v2.5.1

```bash
source ~/.profile
cd ~
git clone https://github.com/bandprotocol/chain
cd chain
git fetch && git checkout v2.5.1
make install
```

## Step 3: Start daemon service

### Restart bandd

```bash
sudo systemctl start bandd
```

After the node is synced up, you might need to restart Yoda service.

### Restart yoda

```bash
sudo systemctl start yoda
```

That’s all! Now, you are running the newest Bandchain version (v2.5).
