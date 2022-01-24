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
`git checkout v1.0.0`  
`make install`    
`starsd version --long`  
```
name: stargaze
server_name: starsd
version: 1.0.0
commit: bee49997775a45f9f6383d6ba8c1dbc67439a6b6
```
# Setup validator node  
`starsd config chain-id stargaze-1`  
`starsd init <MONIKER-NAME> --chain-id stargaze-1`  
`curl -s  https://raw.githubusercontent.com/public-awesome/mainnet/main/stargaze-1/pre-genesis.json >~/.starsd/config/genesis.json`  
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
# Continue with restored db  
`cd .starsd`  
`sudo apt install aria2`  
`wget http://amsterdamstake.club/files/stargaze-data-29Dec20201-899044.torrent`  
when the download status is 100%  
`ctrl+c`  
`sudo apt remove aria2`  
`sudo mv data _data`  
`sudo mv stargaze-data-29Dec20201-899044 data`  
`sudo systemctl daemon-reload`  
`sudo systemctl start starsd`  
`curl -s localhost:26657/status | jq .result | jq .sync_info`  
