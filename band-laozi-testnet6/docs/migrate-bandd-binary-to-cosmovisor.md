# Bandchain: How to migrate from Bandd binary to Cosmovisor

This document describes methods on how to migrate running node by Bandd binary to Cosmovisor

### Step 1: Setup environment variables
Add required environment variables for Cosmovisor into your profile

```
cd ~
echo "export DAEMON_NAME=bandd" >> ~/.profile
echo "export DAEMON_HOME=$HOME/.band" >> ~/.profile
source ~/.profile
```
### Step 2: Setup Cosmovisor
Install Cosmovisor and provide bandd binary to Cosmovisor

```
# Install Cosmovisor
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@v1.0.0

# Setup folder and provide bandd binary for Cosmovisor
mkdir -p $HOME/.band/cosmovisor/genesis/bin
mkdir -p $HOME/.band/cosmovisor/upgrades
cp $HOME/go/bin/bandd $HOME/.band/cosmovisor/genesis/bin
```

### Step 3: Update Bandchain service
In this step, we will overwrite the existing daemon service to use Cosmovisor instead of Bandd binary.

```
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

### Step 4: Restart daemon service
You have to reload the new systemctl configuration and restart the daemon service.

```
# Reload daemon configuration
sudo systemctl daemon-reload
# Restart bandd service to use the new configuration
sudo service bandd restart
```

If you are a validator, we recommend restarting the Yoda service after the node is synced up.


```
# Restart yoda
sudo service yoda restart
```
