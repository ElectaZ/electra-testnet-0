![Electra (1)](https://github.com/Alkia/electra/raw/master/vue/public/Electra.png)

# The electra-testnet-0 guide

- **Recommended hardware requirements**:

| Network   |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet 0   |  2 | 8GB | 0.25TB SSD/NVMe |
| Mainnet   |  2 | 16GB | 0.5TB SSD/NVMe |

- **testnet-0 official ports**:   

| Service | testnet-0 Port | Description       |
|----------|----------------|-------------------|
| rpc       |      26659      |                   |
| p2p       |      26658      |                   |
| prof      |       6061      |                   |
| grp c     |       9092      |                   |
| grpc-web  |       9093      |                   |
| api       |       1318      |                   |
| rosetta   |       8080      |                   |

### Preparing the server

    sudo apt update && sudo apt upgrade -y && \
    sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
    
## GO 18.7 (one command)
    wget https://golang.org/dl/go1.18.7.linux-amd64.tar.gz; \
    rm -rv /usr/local/go; \
    tar -C /usr/local -xzf go1.18.7.linux-amd64.tar.gz && \
    rm -v go1.18.7.linux-amd64.tar.gz && \
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
    source ~/.bash_profile && \
    go version
      
## Build    (07.11.22)
    git clone https://github.com/alkia/electra
    cd electra
    git checkout v0.1.4
    make install
`electrad version`
+ version 0.1.4

      electrad init <moniker-name> --chain-id electra-testnet-0    

## Create/recover wallet
     electrad keys add <walletname>
     electrad keys add <walletname> --recover
##### when creating, do not forget to write down the seed phrase    
## Genesis
    wget https://raw.githubusercontent.com/Alkia/electra-testnet-0/e9b450f853d64d6972c3b5ba6f71e13c4e705eea/genesis.json
    mv genesis.json ~/.electra/config/    
## Set up the minimum gas price $HOME/.electra/config/app.toml as well as seed and peers
    sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"1uelectra\"/;" ~/.electra/config/app.toml

    external_address=$(wget -qO- eth0.me)
    sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26658\"/" $HOME/.electra/config/config.toml

    peers="d8ac979da3dbe2f796e2344616096160dc5cfdc1@34.213.122.198:26658"
    sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.electra/config/config.toml
    seeds=""
    sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.electra/config/config.toml 
    

    
## Create a service file
    sudo tee /etc/systemd/system/electrad.service > /dev/null <<EOF
    [Unit]
    Description=electra
    After=network-online.target

    [Service]
    User=$USER
    ExecStart=$(which electrad) start
    Restart=on-failure
    RestartSec=3
    LimitNOFILE=65535

    [Install]
    WantedBy=multi-user.target
    EOF
    
 [Snapshot](https://polkachu.com/tendermint_snapshots/electra)    (optional) \
    or \
 [RPC](https://nodejumper.io/electra/sync) (optional)
 
# Start
    sudo systemctl daemon-reload
    sudo systemctl enable electrad
    sudo systemctl restart electrad
    sudo journalctl -u electrad -f -o cat
       
# Notes on Managing Validator

Running a production-quality validator node with a robust architecture and security features requires an extensive setup.

Electra chain is powered by the Tendermint consensus. Validators run full nodes, participate in consensus by broadcasting votes, commit new blocks to the blockchain, and participate in governance of the blockchain.

Validators earn the following fees:
- Gas: Fees added on to each transaction to avoid spamming and pay for computing power. Validators set minimum gas prices and reject transactions that have implied gas prices below this threshold.
- Validators get coins as rewards proportional to their shares

`If validators double sign or frequently offline, their credibility will be slashed. Penalties can vary depending on the severity of the violation.`
    
# References    
    * [Website](https://electra.alkia.net/)       
    * [Block Explorer](https://www.mintscan.io/electra/validators)

# Options

## Pruning (optional)
    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.electra/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.electrad/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.electrad/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.electrad/config/app.toml
    
## Indexer (optional)    
    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.electrad/config/config.toml    
   
## Delete node

    sudo systemctl stop electrad && \
    sudo systemctl disable electrad && \
    rm /etc/systemd/system/electrad.service && \
    sudo systemctl daemon-reload && \
    cd $HOME && \
    rm -rf .electrad && \
    rm -rf electra && \
    rm -rf $(which electrad)    
