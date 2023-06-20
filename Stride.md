Update
```
sudo apt update && sudo apt upgrade -y
```

Variables
```
NODENAME=<YOUR_MONIKER_NAME_GOES_HERE>
STRIDE_PORT=16
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export STRIDE_CHAIN_ID=stride-1" >> $HOME/.bash_profile
echo "export STRIDE_PORT=${STRIDE_PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
Install packages
```
sudo apt install curl build-essential git wget jq make gcc tmux chrony -y
```
Install GO
```
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.bash_profile
source ~/.bash_profile
```
Binaries
```
cd $HOME
git clone https://github.com/Stride-Labs/stride.git
cd stride
git checkout v1.0.2
make build
sudo cp $HOME/stride/build/strided /usr/local/bin
```
Change Config and app
```
strided config chain-id $STRIDE_CHAIN_ID
strided config keyring-backend file
strided config node tcp://localhost:${STRIDE_PORT}657
```
Init
```
strided init $NODENAME --chain-id $STRIDE_CHAIN_ID
```
Addrbook and Genesis
```
wget -qO $HOME/.stride/config/genesis.json "https://raw.githubusercontent.com/Stride-Labs/testnet/infra-test/poolparty/infra/genesis.json"
```
Peers and Seeds
```
SEEDS="cb91a11588d66cfd9c01f99541df4978a08e0e39@seedv1.main.stridenet.co:26656"
PEERS="a757fc9ea95a7f643d392ec9fdaa31cbf06e76d9@195.3.221.21:12256,076e97f47762a477f2ae3dd3e798a7970b6bb20d@52.52.110.228:26656,e821acdaf0c7a3c60ea3cd4eb4a98a62dad06f58@43.201.12.41:26656,04dbfff241762b9460b3e23148378fbc8e559a9e@116.203.17.177:26656,74b693b1b0745d250becfbdb550d36504e03bf92@93.115.25.15:26656,b5f9fa874781f975687018ae559f0d952d3a2e24@52.52.208.179:26656,cb0b38aa612e8ac05f704d9b2feb7526607afb77@159.203.191.62:26656,6a1087004245692128a6ad11b812bb3640955b86@162.55.235.69:25656,23180f90318d0003a4e8140a1e67407bf874d69d@78.107.234.44:25656,186b989136983db3ec3147f3e245943d6022e5d4@116.202.227.117:16656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.stride/config/config.toml
```
Service
```
sudo tee /etc/systemd/system/strided.service > /dev/null <<EOF
[Unit]
Description=stride
After=network-online.target

[Service]
User=$USER
ExecStart=$(which strided) start --home $HOME/.stride
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
Register&Start Service
```
sudo systemctl daemon-reload
sudo systemctl enable strided
sudo systemctl restart strided
sudo journalctl -u strided -f -o cat
```





