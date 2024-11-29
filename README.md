# 0G-node Guide

## install dependencies
```console
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

## İnstall Go
```console
cd $HOME
VER="1.21.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

## Settings
```console
echo "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="moniker"" >> $HOME/.bash_profile
echo "export OG_CHAIN_ID="zgtendermint_16600-2"" >> $HOME/.bash_profile
echo "export OG_PORT="29"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Download Binary
```console
cd $HOME
rm -rf 0g-chain
wget -O 0gchaind https://github.com/0glabs/0g-chain/releases/download/v0.4.0/0gchaind-linux-v0.4.0
chmod +x $HOME/0gchaind
sudo mv $HOME/0gchaind $HOME/go/bin
```

## Config and init app
```console
0gchaind config node tcp://localhost:${OG_PORT}657
0gchaind config keyring-backend os
0gchaind config chain-id zgtendermint_16600-2
0gchaind init "moniker" --chain-id zgtendermint_16600-2
```

## Download genesis
```console
rm ~/.0gchain/config/genesis.json
wget -P ~/.0gchain/config https://github.com/0glabs/0g-chain/releases/download/v0.2.3/genesis.json
```

# Set seeds and peers
```console
SEEDS="8f21742ea5487da6e0697ba7d7b36961d3599567@og-testnet-seed.itrocket.net:47656"
PEERS="80fa309afab4a35323018ac70a40a446d3ae9caf@og-testnet-peer.itrocket.net:11656,1306b39656815c4e0037b81b2173e2e260c53b1c@38.242.219.187:12656,922e74b2a646b0549979596bdfbeda426c3adeac@158.220.99.241:12656,c11c97da84b2eab15dc26283e881a490c44cfa25@95.217.78.248:26656,f4ac2e6e31fdd991d590f3541fe688cc62a95fe3@161.97.162.96:47656,7b25cd9823ad344f1b5895690d51b7cb56e473bd@38.242.145.221:47656,fc6bf6f95d34002ba71b2098fa3d69a4f428b2c0@217.76.57.91:12656,9efd0ac7315cbadaf7f488272360741a5b91f28e@62.169.28.60:12656,9a5876bdff97c89ef9fd496dfb7d6df316001e4c@124.63.73.33:40656,11a46e8c69cd7cc3cad346f298be956d608f9065@136.35.205.142:38656,2daf5337b37cd528df76089ebeb86204e64bb9cf@38.242.247.180:12656,3ff3116c7bf258c2e357637466d7590cf1035dfc@164.68.125.48:12656,a47046994182b9c1e71527dee7b3104699cc8024@184.174.32.235:12656,ed2f729fd59bea4c54df189d5a96798910673882@38.242.148.46:47656,b03c66e1c8fe158ad295c84515e1348b9caf5fcd@75.119.129.45:12656,c1af32d9cea6ef0a66227e47ae001871c054608e@213.199.32.178:12656,2d36473d7adec05a321b57e2cf578696117c75a0@62.171.158.42:12656,9c665db23dbbe8a667910fb5e1482908a27ed69e@45.159.229.205:12656,79e773d048f6854a2820209fa31e52773c55ea7e@157.173.99.172:12656,da9ac9d516b1c2f788903b0e3ac7eb75de6eb9a1@144.91.116.117:12656,bad92a950179805d7962fff2edbeed9e85e0e9bb@159.69.72.177:12656"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.0gchain/config/config.toml
```

## Set custom ports in app.toml
```console
sed -i.bak -e "s%:1317%:${OG_PORT}317%g;
s%:8080%:${OG_PORT}080%g;
s%:9090%:${OG_PORT}090%g;
s%:9091%:${OG_PORT}091%g;
s%:8545%:${OG_PORT}545%g;
s%:8546%:${OG_PORT}546%g;
s%:6065%:${OG_PORT}065%g" $HOME/.0gchain/config/app.toml
```

## Set custom ports in config.toml file
```console
sed -i.bak -e "s%:26658%:${OG_PORT}658%g;
s%:26657%:${OG_PORT}657%g;
s%:6060%:${OG_PORT}060%g;
s%:26656%:${OG_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${OG_PORT}656\"%;
s%:26660%:${OG_PORT}660%g" $HOME/.0gchain/config/config.toml
```

## Config pruning
```console
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.0gchain/config/app.toml 
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.0gchain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"19\"/" $HOME/.0gchain/config/app.toml
```

## Set minimum gas price, enable prometheus and disable indexing
```console
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0ua0gi"|g' $HOME/.0gchain/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.0gchain/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.0gchain/config/config.toml
```

## Create service file
```console
sudo tee /etc/systemd/system/0gchaind.service > /dev/null <<EOF
[Unit]
Description=0G node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.0gchain
ExecStart=$(which 0gchaind) start --home $HOME/.0gchain --log_output_console
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

## Enable and start service
```console
sudo systemctl daemon-reload
sudo systemctl enable 0gchaind
sudo systemctl restart 0gchaind && sudo journalctl -u 0gchaind -f
```

## create a new wallet
```console
0gchaind keys add $WALLET
```

## Restore exexuting wallet
```console
0gchaind keys add $WALLET --recover
```

## Check sync status, once your node is fully synced, the output from above will print "false"
```console
0gchaind status 2>&1 | jq 
```

## Create Validator
```console
0gchaind tx staking create-validator \
--amount 1000000ua0gi \
--from $WALLET \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--pubkey $(0gchaind tendermint show-validator) \
--moniker "moniker" \
--identity "" \
--website "" \
--details "Your info" \
--chain-id zgtendermint_16600-2 \
--gas=auto --gas-adjustment=1.6 \
-y
```
