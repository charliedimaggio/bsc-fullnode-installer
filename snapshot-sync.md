# Guide to downloading and installing the tarball snapshot provided by BSC:

The reason people have historically struggled with syncing from the snapshot is because no real effort has been made to simplify the process.

This guide will attempt to address that.

The first step is to prepare your environment so that it is ready for the snapshot:

## Step 1 - Become root (only required if you are not currently root)
```
sudo su
```

## Step 2 - Create geth user
```
useradd -m geth
```

## Step 3 - Switch to geth home folder
```
cd /home/geth
```

## Step 4 - Download geth_linux
```
wget -O /home/geth/geth_linux https://github.com/binance-chain/bsc/releases/latest/download/geth_linux
chmod +x /home/geth/geth_linux

```

## Step 5 - Create `start.sh` file.
```
echo "./geth_linux --config ./config.toml --datadir ./mainnet --cache 18000 --rpc.allow-unprotected-txs --txlookuplimit 0 --http --maxpeers 100 --ws --syncmode=full --snapshot=true --diffsync" > /home/geth/start.sh
chmod +x /home/geth/start.sh

```

## Step 6 - Install unzip
```
apt update
apt install unzip
```

## Step 7 - Download mainnet configs and initialize geth with genesis data
```
wget -O /home/geth/mainnet.zip https://github.com/binance-chain/bsc/releases/latest/download/mainnet.zip
unzip /home/geth/mainnet.zip
/home/geth/geth_linux --datadir mainnet init genesis.json

```

## Step 8 - Setup systemd
```
nano /lib/systemd/system/geth.service
```

Then paste the following:

```
[Unit]
Description=BSC Full Node

[Service]
User=geth
Type=simple
WorkingDirectory=/home/geth
ExecStart=/bin/bash /home/geth/start.sh
Restart=on-failure
RestartSec=30
TimeoutSec=300
IOWeight=8000
CPUWeight=8000

[Install]
WantedBy=default.target
```

Save the file by pressing CTRL+O and then press enter. Exit nano by pressing CTRL+X

## Step 9 - Give geth user ownership and enable geth systemd service

```
chown -R geth.geth /home/geth/*
systemctl enable geth
```

## Step 10 - Clean up /home/geth/mainnet/geth/


We need to remove some of the files and folders that were created when we ran the genesis step (7)
```
rm -rf /home/geth/mainnet/geth/*
```

## Step 11.1 - Download the tarball image

**_If you have less than 2TB space or are an advanced user then follow step 11.2_**

This page should contain the latest image: [tarball snapshot](https://github.com/binance-chain/bsc-snapshots) - copy one of the geth.tar.gz URL's for later use.
*For best performance please pick the endpoint that is geographically closest to your server*

Use this command to download the file - remember to **_keep the quotations_** for the URL:
**DO NOT USE THIS EXAMPLE URL AS IT WILL BE SIGNIFICANTLY OUT OF DATE**
```
wget -O /home/geth/mainnet/geth.tar.lz4  "https://tf-dex-prod-public-snapshot.s3.amazonaws.com/geth-20211114.tar.gz?AWSAccessKeyId=AKIAYIN6SBQPUZDDRRO&Signature=xJJw%2BwbS%2B32IMg6KojKGPq1TwKw%3D&Expires=1639516490"
```

If the download only partially downloads then you might be able to continue if you include the -c or --continue option before running wget again.
Example command required to continue download:
```
wget -cO /home/geth/mainnet/geth.tar.lz4  "
https://tf-dex-prod-public-snapshot.s3.amazonaws.com/geth-20211114.tar.gz?AWSAccessKeyId=AKIAYINE6SBQPUZDRRO&Signature=xJJw%2BwbS%2B32IMg6KojKGPq1TwKw%3D&Expires=1639516490"
```

Once the download has finished you need to make sure that it matches the MD5 checksum mentioned on the website: [tarball snapshot](https://github.com/binance-chain/bsc-snapshots)
**_This command can take some time to run and you will not see a progress indicator_**
```
md5sum /home/geth/mainnet/geth.tar.lz4
```

## Step 11.2 - Download and unpack the tarball image (advanced users)
**_Skip this step if you completed step 11.1_**

This page should contain the latest image: [tarball snapshot](https://github.com/binance-chain/bsc-snapshots) - copy one of the geth.tar.gz URL's for later use.
*For best performance please pick the endpoint that is geographically closest to your server*

Use this command to download the file - remember to **_keep the quotations_** for the URL:
**DO NOT USE THIS EXAMPLE URL AS IT WILL BE SIGNIFICANTLY OUT OF DATE**

This command will download and unpack the snapshot at the same time. The output will not reflect the fact we used "strip-component=2" but the result should respect that flag. The only other concern is if the folder structure changes in future updates.
Additionally, this command now utilises a program called `screen`, so the process will not halt if you lose your connection.
```
cd /home/geth/mainnet
apt install screen
screen -dm bash -c 'wget "https://tf-dex-prod-public-snapshot.s3-accelerate.amazonaws.com/geth-20211202.tar.gz?AWSAccessKeyId=AKIAYINE6SBQPUZDDRRO&Signature=e38bDFq%2BTlpgzXpZLkwvsDNyk%3D&Expires=1641059846" -O - | tar --strip-components=2 -I lz4 -zxf -'
```
To attach to screen press `CTRL+r` and to detach `CTRL+a d`

## Step 12 - Unpack tarball

*Skip this step if you completed step 11.2*

We need to remove two redundant patent folders "server/data-seed"
```
cd /home/geth/mainnet && exec nohup tar -I lz4 --strip-components=2 -xzf /home/geth/mainnet/geth.tar.lz4
```

## Step 13 - Set geth as owner

Once the extraction has completed you will need to perform another chown command:
```
chown -R geth.geth /home/geth/*
```

## Step 14 - Start geth

Start the geth service:
```
systemctl start geth
```

## Step 15 - Now what?

If you are unfamiliar with how to monitor the geth service and/or the syncing process

Stop the geth service:
```
systemctl stop geth
```

Check the status of the geth service:
```
systemctl status geth
```

Check the journal log for errors or warnings from the geth service:
```
journalctl -u geth.service
```

Open the most recent geth log file:
```
nano /home/geth/mainnet/bsc.log
```

Monitor the sync process via tail:
```
tail -F /home/geth/mainnet/bsc.og
```

Check sync process via RPC:
This command will return 'false' either when you are fully synced or if the startup process has not completed

```
/home/geth/geth_linux attach /home/geth/mainnet/geth.ipc --exec 'eth.syncing'
```

Check the performance of your storage:
This command will test the read performance of your storage, as this is typically where most systems are weakest.
```
apt install fio -y
fio -direct=1 -iodepth=1 -rw=randread -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Rand_Read_Testing
```

Additional information can be found on our discord server [TheCryptoFarm](https://discord.com/invite/H582fcrrvG) or the official [BSC Discord](https://discord.gg/ukfzpWpTHp)
