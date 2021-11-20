# Guide to downloading and installing the tarball snapshot provided by BSC:

The reason people have historically struggled with syncing from the snapshot is because no real effort has been made to simplify the process.

This guide will attempt to address that.

The first step is to prepare your environment so that it is ready for the snapshot:

## Step 1 - Become Root
```
sudo su
```

## Step 2 - Create GETH User
```
sudo useradd -m geth
```

## Step 3 - Switch To The New User's Home
```
cd /home/geth
```

## Step 4 - Download The GETH
```
wget -O /home/geth/geth_linux https://github.com/binance-chain/bsc/releases/latest/download/geth_linux
chmod +x geth_linux
```

## Step 5 - Create `start.sh` File.
```
echo "./geth_linux --config ./config.toml --datadir ./mainnet --cache 18000 --rpc.allow-unprotected-txs --txlookuplimit 0 --http --maxpeers 100 --ws --syncmode=full --snapshot=false --diffsync" > start.sh
chmod +x start.sh
```

## Step 6 - Install Unzip
```
apt install unzip
```

## Step 7 - Download Mainnet Configs
```
wget https://github.com/binance-chain/bsc/releases/latest/download/mainnet.zip
unzip mainnet.zip
./geth_linux --datadir mainnet init genesis.json
```

## Step 8 - Setup systemd
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

## Step 9 - Enable geth systemd service

```
chown -R geth.geth /home/geth/*
systemctl enable geth
```

## Step 10 - Clean up /home/geth/mainnet/geth/


If you are modifying an existing server you will need to make sure that you remove the geth folder located in /home/geth/mainnet/
```
rm -rf /home/geth/mainnet/geth/*
```

## Step 11 - Prepare to download the tarball image

Move bash to the following directory:
```
cd /home/geth
```

Make the mainnet folder if it does not already exist:
```
mkdir mainnet
```

Move to the mainnet directory:
```
cd mainnet 
```

## Step 12.1 - Download the tarball image

This page should contain the latest image: [tarball snapshot](https://github.com/binance-chain/bsc-snapshots) - copy the URL for later use.
*For best performance please pick the endpoint that is geographically closest to your server*

Use this command to download the file - remember to **keep the quotations** for the URL:
**DO NOT USE THIS EXAMPLE URL AS IT WILL BE SIGNIFICANTLY OUT OF DATE**
```
wget -O geth.tar.gz  "https://tf-dex-prod-public-snapshot.s3.amazonaws.com/geth-20211114.tar.gz?AWSAccessKeyId=AKIAYINE6SBQPUZDDRRO&Signature=xJJw%2BwbS%2B32IMg6KojKGPq1TwKw%3D&Expires=1639516490"
```

If the download only partially downloads then you might be able to continue if you include the -c or --continue option before running wget again.
Example command required to continue download:
```
wget -cO geth.tar.gz  "
https://tf-dex-prod-public-snapshot.s3.amazonaws.com/geth-20211114.tar.gz?AWSAccessKeyId=AKIAYINE6SBQPUZDDRRO&Signature=xJJw%2BwbS%2B32IMg6KojKGPq1TwKw%3D&Expires=1639516490"
```

Once the download has finished you need to make sure that it matches the MD5 checksum mentioned on the website: [tarball snapshot](https://github.com/binance-chain/bsc-snapshots)
```
md5sum geth.tar.gz
```

## Step 12.2 - Download and unpack the tarball image, but skip hash verification

*Skip this step if you completed step 12.1*

This command will download and unpack the snapshot at the same time. The output will not reflect the fact we used "strip-component=2" but the result should respect that flag. The only other concern is if the folder structure changes in future updates. It would be wise to cd /tmp and do a small trial run from there, just to make sure it is working as expected:
```
wget "https://tf-dex-prod-public-snapshot.s3.amazonaws.com/geth-20211114.tar.gz?AWSAccessKeyId=AKIAYINE6SBQPUZDDRRO&Signature=xJJw%2BwbS%2B32IMg6KojKGPq1TwKw%3D&Expires=1639516490" -qO - | tar --strip-components=2 -zxvf -
```

## Step 13 - Unpack tarball

*Skip this step if you completed step 12.2*

We need to remove two redundant patent folders "server/data-seed"
```
tar --strip-components=2 -xzf geth.tar.gz
```

## Step 14 - Set geth as owner

Once the extraction has completed you will need to perform another chown command:
```
chown -R geth.geth /home/geth/*
```

## Step 15 - Start geth

Start the geth service:
```
systemctl start geth
```
