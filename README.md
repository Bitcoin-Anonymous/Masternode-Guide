Bitcoin Anonymous Advance CLI Masternode Setup Guide
==========================

## Introduction

This guide is for a single masternode, on a Ubuntu 18.04 64-bit server (VPS) running headless and will be controlled from the wallet on your local computer (Control wallet). The wallet on the the VPS will be referred to as the Remote wallet.

You will need your masternode server details for progressing through this guide including the public IP address.

### VPS Server Recommendations ###

[Digital Ocean](https://www.digitalocean.com/products/droplets/)

[Vultr](https://www.vultr.com/products/cloud-compute/#compute)

**For either of the above VPS services, the economical $5 plan will be sufficient for your masternode.**

---

First the basic requirements:

1. 100 BTCA
1. A main computer (your everyday computer) – This will run the Bitcoin Anonymous Node control wallet, hold your collateral 100 BTCA and can be opened and closed without affecting the masternode.
1. Masternode Server (VPS Suggested – The wallet daemon that will be on 24/7).
1. A unique IP address for your VPS / Remote wallet.

**For security reasons, you’re are going to need a different IP for each masternode you plan to host.**

## Configuration

**Step 1:** Using the control wallet, enter the debug console `Settings > Debug > Console` and type the following command:

```
createmasternodekey
```

*(This will be the masternode’s privkey variable. We will need this later…)*

**Step 2:** Using the control wallet, enter the following command to create a new address:

```
getnewaddress
```

**Step 3:** Still in the control wallet, send 100 BTCA to the wallet address you generated in Step 2. 

Be 100% sure that you entered the address correctly. 

Also make sure this is exactly **100 BTCA**; No less, no more.

**Be absolutely sure the send to address is copied correctly and then check it again. We cannot help you if you send 100 BTCA to an incorrect address.**

Please allow at least 1 block confirmation to complete before moving on.

**Step 4:** Still in the control wallet, enter the command into the console:

```
getmasternodeoutputs
```

*This gets the proof of transaction of sending 100 BTCA*

**Step 5:** Still on the main computer, go into the BTCA data directory:

OS | Path to BTCA
------------ | -------------
Windows | `%Appdata%/BTCA/`
macOS | `~/Library/Application\ Support/BTCA/`
Linux | `~/.btca/`

Find the masternode.conf file, edit it in your favorite text editor and add the following line to it:

```
<Name/Alias of Masternode> <Unique VPS Public IP address>:34472 <The result of Step 1> <Result of Step 4> <The number after the long line in Step 4>
```

Example:

```
MN1 34.24.122.10:34472 894FPpFdbr7sr6Si4fdsfssjjapuFzAXwEVCrpPJubnrmU6aKzh c8f4865da57a68d0e6ddd84324dfd28cfbe0c901015b973e7331bb8ce018716999 1
```

Substitute with your own values and without the "<>"s.

Lastly, close the control wallet and open again to load the new configuration file.

## VPS Remote Wallet Install

Install the latest version of the Bitcoin Anonymous Node wallet onto your masternode. The latest version can be found here: [Bitcoin Anonymous Node Releases](https://github.com/Bitcoin-Anonymous/BTCA-Node/releases).

**Step 1:** Log in to your VPS via SSH:

```
cd ~
```

**Step 2:** From your home directory, download the latest version from the BTCA GitHub repository:

```
wget https://github.com/Bitcoin-Anonymous/BTCA-Node/releases/download/v1.0.0/btca-x86_64-linux-gnu.tar.gz
```

Always check the releases page for the latest version and update the URL to reflect the most current version.

**Step 3:** Unzip & Extract:

```
tar -zxvf btca-x86_64-linux-gnu.tar.gz
```

**Step 4:** Copy the files to the local bin. Run privacy install script. **Requires sudo**

```
sudo cp -R btca/bin/* /usr/local/bin/
```

```
sudo ./btca/bin/fetch-params.sh
```

**Step 5:** Note: If this is the first time running the wallet in the VPS, you’ll need to attempt to start the wallet:

```
btcad --daemon
```

**Step 6:** Stop the daemon after the blockchain download completely:

You can verify you have the entire blockchain by comparing the results of `btca-cli getinfo` with the [BTCA Block Explorer](http://explorer.bitcoinanonymous.info)

Once verified:

```
btca-cli stop
```

**Step 7:** Navigate to the BTCA data directory:

```
cd ~/.btca
```

**Step 8:** Open the configuration file by typing:

```
nano btca.conf
```

**Step 9:** Make the config look like this with your values substituted where needed:

```
rpcuser=long_random_username
rpcpassword=longer_random_password
rpcallowip=127.0.0.1
server=1
daemon=1
logtimestamps=1
maxconnections=256
masternode=1
externalip=VPS_Public_IP_Address
masternodeaddr=VPS_Public_IP_Address:34472
masternodeprivkey=Result of Step 1
```

*Make sure to replace rpcuser and rpcpassword with your own.*

**Step 10:** Save and exit the file:

```
Ctr+x to exit and press Y to save changes and press enter to close
```

**Please be sure to have port 34472 open on your server firewall if applicable for your control wallet to be able start the masternode remotely.**

## Start the Masternode

**Step 1:** Navigate back to the Bitcoin Anonymous VPS server:


**Step 2:** Start the wallet daemon:

```
btcad
```

**Step 3:** From the Control wallet debug console issue the command:

```
startmasternode missing y
```

**The following should appear.**

```
"overall" : "Successfully started 1 masternodes, failed to start 0, total 1"
```

**Step 4:** Back in the VPS (remote wallet), start the masternode:

```
btca-cli startmasternode local false
```

**Step 5:** Use the following command to check status:

```
btca-cli masternode status
```

You should see something like:

```
{
  "txhash": "c2f85b71a04d111a1ca337fbc3aed1168856e3365b4e846ac9b89a9908c15b1d",
  "outputidx": 1,
  "netaddr": "34.179.22.255:34472",
  "addr": "AH4bvJto5ZR7UWpSqb4zSP8o82GHindhP5",
  "status": 4,
  "message": "Masternode successfully started"
}
```

If you see status Not capable masternode: Hot node, waiting for remote activation, you need to wait a bit longer for the blockchain to reach consensus. It's possible it may take 60 to 120 minutes before the activation can be done. You can also try restarting the VPS wallet `btca-cli stop` and then `btcad` and trying the `btca-cli startmasternode local false` command again.

Bitcoin Anonymous Masternode Setup is Complete!


## Tearing down a Masternode

1. `btca-cli stop` from the masternode to stop the wallet.
1. Then from your control wallet, edit your masternode.conf, delete the MN1 masternode line entry.
1. Restart the control wallet.
1. Your 100 BTCA will now be unlocked.