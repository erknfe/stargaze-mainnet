# Setup Go  
`curl https://dl.google.com/go/go1.17.5.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf -`  
```
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
```
`source $HOME/.profile`  
`go version`    
# Install required software packages  
`sudo apt-get install git curl build-essential make jq -y`    
# Build from source  
`git clone https://github.com/public-awesome/stargaze`  
`cd stargaze`  
`git checkout v2.0.0`  
`make install`    
`starsd version --long`  
```
name: stargaze
server_name: starsd
version: 2.0.0
commit: 62138d79f0b348449d5fb1e7838f9958842f879b
```
# Setup validator node  
`starsd config chain-id stargaze-1`  
`starsd init <MONIKER-NAME> --chain-id stargaze-1`  
`curl -s  https://raw.githubusercontent.com/public-awesome/mainnet/main/stargaze-1/pre-genesis.json >~/.starsd/config/genesis.json`  
`curl -s  https://raw.githubusercontent.com/public-awesome/mainnet/main/stargaze-1/genesis.tar.gz > genesis.tar.gz`  
`tar -C ~/.starsd/config/ -xvf genesis.tar.gz`  
# Running in production  
`sudo nano /etc/systemd/system/starsd.service`  
```
Description=Stargaze daemon
After=network-online.target

[Service]
User=<YOUR_USERNAME>
ExecStart=/home/<YOUR-USERNAME>/go/bin/starsd start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```
`sudo systemctl enable starsd`  
`sudo systemctl start starsd`  
`starsd status`  
`journalctl -u starsd -f`  
`curl -s localhost:26657/status | jq .result | jq .sync_info`  
# Use prunning data    
`sudo systemctl stop starsd`  
`sudo nano .starsd/config/app.toml`  
```
pruning = "custom"
pruning-keep-recent = "5"
pruning-keep-every = "0"
pruning-interval = "10
```
`sudo nano .starsd/config/config.toml`  
```
indexer = "null"
```
# Restore updated db snapshot  
`sudo systemctl stop starsd`  
`cd .starsd`  
`sudo mv data _data`  
`mkdir -p data`  
`cd data`  
`SNAP_NAME=$(curl -s http://135.181.60.250:8086/stargaze/ | egrep -o ">stargaze.*tar" | tr -d ">")`  
`wget -O - http://135.181.60.250:8086/stargaze/${SNAP_NAME} | tar xf -`  
`sudo systemctl daemon-reload`  
`sudo systemctl start starsd`  
`curl -s localhost:26657/status | jq .result | jq .sync_info`  
# Install Cosmovisor  
`cd stargaze`  
`curl -s  https://raw.githubusercontent.com/public-awesome/mainnet/main/stargaze-1/genesis.tar.gz > genesis.tar.gz`  
`tar -C ~/.starsd/config/ -xvf genesis.tar.gz`  
`wget https://raw.githubusercontent.com/public-awesome/mainnet/main/normalize.jq`  
`jq -S -f normalize.jq  ~/.starsd/config/genesis.json | shasum -a 256`  
`cd ..`  
`go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@latest`  
`echo "export DAEMON_NAME=starsd" >> ~/.profile`  
`echo "export DAEMON_HOME=$HOME/.starsd" >> ~/.profile`  
`source ~/.profile`  
`mkdir -p $DAEMON_HOME/cosmovisor/genesis/bin`  
`mkdir -p $DAEMON_HOME/cosmovisor/upgrades`  
`cp $GOPATH/bin/starsd $DAEMON_HOME/cosmovisor/genesis/bin`  

`sudo nano /etc/systemd/system/starsd.service`  
```
[Unit]
Description=Stargaze Daemon
After=network-online.target

[Service]
User=<YOUR-USERNAME>
ExecStart=/home/<YOUR-USERNAME>/go/bin/cosmovisor start
Restart=always
RestartSec=3
LimitNOFILE=4096

Environment="DAEMON_NAME=starsd"
Environment="DAEMON_HOME=/home/<YOUR-USERNAME>/.starsd"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_LOG_BUFFER_SIZE=512"

[Install]
WantedBy=multi-user.target
```
`sudo -S systemctl daemon-reload`  
`sudo -S systemctl enable starsd`  
`sudo systemctl start starsd`  
`sudo systemctl status starsd`  
`journalctl -u starsd -f`  
`curl -s localhost:26657/status | jq .result | jq .sync_info`  
