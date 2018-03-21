# SunCoin (SUN) - Masternode Setup

First Global CO2 Neutral Blockchain Network

With focus on emission neutral mining and an CO2 initiative fund, SunCoin is the thought leader in creating a sustainable distributed ledger system awareness and offer working solution for decarbonizing the market. With your help, SunCoin will balance out impact on our environment and provide new ways powering the future of crypto finance and distributed ledger systems.

**Info:** SunCoin is a proud fork of Dash. Kudos to the Dash team for the hard work and creating the technical foundation for a community driven project as SunCoin.

This guide will describe the steps to setup a SunCoin Masternode.

## Requirements

- 10.000 SUN
- Personal wallet to store your SUN coins
- Linux server, Virtual Private Server (VPS) preferred
- Ubuntu 14.04 or 16.04
- Static public IP address
- 24x7 server uptime

## 1. Masternode Collateral

A SunCoin address with a single transaction of exactly 10.000 SUN is required to operate a masternode (so called collateral).

Open the SunCoin wallet and wait for it to sync with the network.

Click tools > debug console to open the console. Type the following two commands into the console to generate a masternode key and to get the collateral address:

    $ masternode genkey

    $ getaccountaddress 0

Take note of the masternode private key and collateral address, since we will need it later. The next step is to secure your wallet (if you have not already done so). First, encrypt the wallet by selecting settings > encrypt wallet. You should use a strong, new password that you have never used somewhere else. Take note of your password and store it somewhere safe or you will be permanently locked out of your wallet and lose access to your funds. Next, back up your wallet file by selecting file > backup wallet. Save the file to a secure location physically separate to your computer, since this will be the only way you can access our funds if anything happens to your computer.

Now send exactly 10.000 SUN in a single transaction to the account address you generated in the previous step. You will need 15 confirmations before you can start the masternode, but you can continue with the masternode setup in the meantime.

## 2. Setup of Virtual Private Server

Download and install the latest SunCoin linux build.

    $ apt-get install wget nano

    $ cd ~
    $ wget https://github.com/suncoin-network/suncoin/releases/....

Create a working directory for SunCoin, extract the compressed archive, copy the necessary files to the directory and set them as executable:

    $ cd ~
    $ mkdir .suncoincore
    $ tar xfvz XXXXX.tar.gz
    $ cp XXXXX/bin/suncoind .suncoincore/
    $ cp XXXXX/bin/suncoin-cli .suncoincore/
    $ chmod 777 .suncoincore/suncoin*

Clean up unneeded files:

    $ rm XXXXX.tar.gz
    $ rm -r XXXXX/

Create a configuration file using the following command:

    $ nano ~/.suncoincore/suncoin.conf
    
We now need to specifying several variables. Copy and paste the following text to get started, then replace the variables specific to your configuration as follows:

    #----
    rpcuser=XXX
    rpcpassword=XXX
    rpcallowip=127.0.0.1
    #----
    listen=1
    server=1
    daemon=1
    #----
    masternode=1
    masternodeprivkey=XXX
    externalip=XXX.XXX.XXX.XXX
    #----

Replace the fields marked with XXX as follows:

- rpcuser: enter any string of numbers or letters, no special characters allowed
- rpcpassword: enter any string of numbers or letters, no special characters allowed
- masternodeprivkey: this is the private key you generated in the previous step
- externalip: this is the public IP address of your VPS

You can now start running SunCoin to begin synchronisation with the blockchain:

    $ ~/.suncoincore/suncoind
    
You will see a message reading "Suncoin Core server starting". To check the status of synchronisation you can use the command:

    $ ~/.suncoincore/suncoin-cli mnsync status
    
When synchronisation is complete, you should see the following response:  

    {
     "AssetID": 999,
     "AssetName": "MASTERNODE_SYNC_FINISHED",
     "Attempt": 0,
     "IsBlockchainSynced": true,
     "IsMasternodeListSynced": true,
     "IsWinnersListSynced": true,
     "IsSynced": true,
     "IsFailed": false
    }

## 3. Setup of Masternode Sentinel

We will now install sentinel, a piece of software which operates as a watchdog to communicate to the network that your masternode is working properly.

### 3.1. Install Prerequisites

Make sure Python version 2.7.x or above is installed:

    python --version

Update system packages and ensure virtualenv is installed:

    $ apt-get update
    $ apt-get install python-virtualenv

Make sure the local SunCoin daemon is running:

    $ suncoin-cli getinfo | grep version
    
### 3.2. Install Sentinel

Clone the Sentinel repo and install Python dependencies.

    $ cd ~/.suncoincore
    $ git clone https://github.com/suncoin-network/sentinel.git
    $ cd sentinel
    $ virtualenv venv
    $ venv/bin/pip install -r requirements.txt
    $ venv/bin/python bin/sentinel.py

### 3.3. Set up Cron

Add sentinel to crontab to make sure it runs every minute to check on your masternode:

    $ crontab -e
    
Choose nano as your editor and enter the following line at the end of the file:  

    * * * * * cd ~/.suncoincore/sentinel && ./venv/bin/python bin/sentinel.py 2>&1 >> sentinel-cron.log
    
Press enter to make sure there is a blank line at the end of the file.   

## 4. Start Masternode

Please switch back to your personal wallet. 

Now you need to find the "txid" of your collateral (10.000 SUN) transaction. Click tools > debug console and enter the following command:

    $ masternode outputs
  
This will return a string of characters similar to this:

    {
    "06e38868bb8f9958e34d5155437d009b72dff33fc28874c87fd42e51c0f74fdb" : "0",
    }


The first string is your transaction hash, while the last number is the index. We now need to create a file called masternode.conf in order to be able to start your masternode on the network. Open a new text file and enter the following information:

- Label: single word to identify your masternode, e.g. MN1
- IP and port: The IP address and port 10332 configured in the suncoin.conf file
- Masternode private key: This is the result of your masternode genkey command earlier
- Transaction hash: The txid we just identified using masternode outputs
- Index: The index we just identified using masternode outputs

Enter all of this information on a single line with each item separated by a space, for example:

    MN1 52.14.2.67:10332 XrxSr3fXpX3dZcU7CoiFuFWqeHYw83r28btCFfIHqf6zkMp1PZ4 06e38868bb8f9958e34d5155437d009b72dff33fc28874c87fd42e51c0f74fdb 0

Save this file in the SunCoin data folder on the PC running the SunCoin wallet using the filename masternode.conf. 

- Linux: /home/USERNAME/.suncoincore/
- OSX: /Users/USERNAME/Library/Application Support/suncoincore/
- Windows: C:\Users\USERNAME\AppData\Roaming\suncoincore	

You may need to enable "View hidden items" to view this folder. Now close your text editor and restart your SunCoin wallet. The SunCoin wallet will now recognise masternode.conf during startup and is ready to activate your masternode. Go to settings > unlock wallet and enter your wallet passphrase. Then click tools > debug console and enter the following command to start your masternode (replace MN1 with the label for your masternode):

    $ masternode start-alias MN1

At this point you can go back to your VPS and monitor your masternode status, by entering:

    $ ~/.suncoincore/suncoin-cli masternode status 

You will probably need to wait around 30 minutes as the node passes the PRE_ENABLED stage and finally reaches ENABLED. 

**Congratulations! Your SunCoin masternode is now running.**

