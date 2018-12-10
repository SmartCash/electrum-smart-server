## Create electrumx user

```
adduser electrumx
usermod -aG sudo electrumx
```

## Install SmartCash Core

```
mkdir ~/.smartcash/
nano ~/.smartcash/smartcash.conf
```

Paste at least the following minimum options in smartcash.conf:

```
server=1
listen=1
daemon=1
txindex=1
addressindex=1
timestampindex=1
spentindex=1
rpcuser=<random username>
rpcpassword=<strong password>

# For Testnet Only
# testnet=1
# addnode=45.77.209.202:19678
```

Instal from apt

```
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository ppa:smartcash/ppa
sudo apt update
sudo apt install smartcashd
```

## Install electrumx database

```
sudo apt install python3-pip python3-leveldb libleveldb-dev
```

## Install required Python packages

```
pip3 install --upgrade pip setuptools wheel && reboot
pip3 install --upgrade aiorpcX attrs plyvel pylru aiohttp pycryptodomex
```

## Install and set up ElectrumX

Clone the ElectrumX code from a GitHub repository using git:

```
mkdir ~/source && cd ~/source
git clone https://github.com/SmartCash/electrum-smart-server.git
cd electrumx
```

Run the installation script (use sudo if it doesn't work):

```
python3 setup.py install
```

Next, create a data folder where the blockchain data will be stored:

```
mkdir ~/.electrumx && mkdir ~/.electrumx/db
```


## Create a self-signed certificate

To allow Electrum wallets to connect to your server over SSL you need to create a self-signed certificate.

```
cd ~/.electrumx
```

and generate your key:

```
openssl genrsa -des3 -passout pass:x -out server.pass.key 2048
openssl rsa -passin pass:x -in server.pass.key -out server.key
rm server.pass.key
openssl req -new -key server.key -out server.csr
```

Follow the on-screen information. It will ask for certificate details such as your country and password. You can leave those fields empty.
When done, create a certificate:

```
openssl x509 -req -days 1825 -in server.csr -signkey server.key -out server.crt
```

These commands will create 2 files: server.key and server.crt.


## Launch ElectrumX as service

First, make sure a fully validating SmartCash node is running:

```
smartcash-cli getinfo
```
```
curl --user <rpcuser>:<rpcpassword> --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "getinfo", "params": [] }' -H 'content-type: text/plain;' http://127.0.0.1:9679/
```

Open a sudo session and copy a service file from the ElectrumX repo to your Systemd directory:

```
sudo -s
nano /lib/systemd/system/electrumx.service
```

Edit the file to match your setup:

```
[Unit]
Description=Electrumx
After=network.target

[Service]
EnvironmentFile=/etc/electrumx.conf
ExecStart=/home/electrumx/source/electrumx/electrumx_server
User=electrumx
LimitNOFILE=8192
TimeoutStopSec=30min

[Install]
WantedBy=multi-user.target
```


You need to edit at least ExecStart and User variables.

```
ExecStart=/home/electrumx/source/electrumx/electrumx_server
User=electrumx
```

Create a configuration file for the server:

```
nano /etc/electrumx.conf
```

and configure it according to your environment.

```
COIN = SmartCash   
DB_DIRECTORY = ~/.electrumx/db
DB_ENGINE = leveldb
USERNAME = electrumx
ELECTRUMX = ~/source/electrumx/electrumx_server
HOST = 0.0.0.0
BANNER_FILE = ~/.electrumx/banner.txt
CACHE_MB = 2048
MAX_SESSIONS = 1024
SSL_CERTFILE = ~/.electrumx/server.crt
SSL_KEYFILE = ~/.electrumx/server.key

# Mainet
Net = Mainet
DAEMON_URL = user:pass@127.0.0.1:9679
DONATION_ADDRESS = <your_smartcash_address>
SSL_PORT = 50002 
TCP_PORT = 50001

# Testnet
# Net = Testnet
# DAEMON_URL = user:pass@127.0.0.1:19679
# DONATION_ADDRESS = <your_tSmartcash_address>
# SSL_PORT = 51002 
# TCP_PORT = 51001

```

Create a banner File

```
nano ~/.electrumx/banner
```

Example

```
Welcome to electrumX.smartcash.cc
SmartCash Core: $DAEMON_VERSION
ElectrumX Server: $SERVER_VERSION
Support: https://discord.gg/BDUh8jr
```

Start the service:

```
systemctl start electrumx
```

and check the output:

```
journalctl /usr/bin/python3 -f -n100
```

If it gives you no errors, enable the service:

```
systemctl enable electrumx
```

You can exit the sudo session now:

```
exit
```
