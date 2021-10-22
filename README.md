# Become Root
```
sudo su
```

# Create GETH User
```
sudo useradd -m geth
```

# Switch To The New User's Home
```
cd /home/geth
```

# Download The GETH
```
wget https://github.com/binance-chain/bsc/releases/download/v1.1.3/geth_linux
chmod +x geth_linux
```

# Create `start.sh` File.
```
echo "./geth_linux --config ./config.toml --datadir ./mainnet --cache 18000 --rpc.allow-unprotected-txs --txlookuplimit 0 --http --maxpeers 100 --ws --syncmode=snap --snapshot=false" > start.sh
chmod +x start.sh
```

# Install Unzip
```
apt install unzip
```

# Download Mainnet Configs
```
wget https://github.com/binance-chain/bsc/releases/download/v1.1.2/mainnet.zip
unzip mainnet.zip
./geth_linux --datadir mainnet init genesis.json
```

# Setup systemd
```
sudo nano /lib/systemd/system/geth.service
```

Then paste the following;

```
[Unit]
Description=BSC Full Node

[Service]
User=geth
Type=simple
WorkingDirectory=/home/geth
ExecStart=/bin/bash /home/geth/start.sh
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
```

After that;

```
chown -R geth.geth /home/geth/*
systemctl enable geth
systemctl start geth
```

# Show logs
```
tail -f /home/geth/mainnet/bsc.log
```

# How to Prune
```
systemctl stop geth 
cd /home/geth
./geth_linux snapshot prune-state --datadir ./mainnet
chown -R geth.geth ./mainnet
systemctl start geth
```

# Credits
The all procedures was written by PhatJay#4958.
