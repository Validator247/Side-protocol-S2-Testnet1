# Side-protocol-S2-Testnet1

Update system

    sudo apt update
    sudo apt-get install git curl build-essential make jq gcc snapd chrony lz4 tmux unzip bc -y

 Install Go

     rm -rf $HOME/go
     sudo rm -rf /usr/local/go
     cd $HOME
     curl https://dl.google.com/go/go1.22.1.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf -
     cat <<'EOF' >>$HOME/.profile
     export GOROOT=/usr/local/go
     export GOPATH=$HOME/go
     export GO111MODULE=on
     export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
     EOF
     source $HOME/.profile
     go version

 Build sided binary

     cd $HOME
     rm -rf sidechain
     git clone -b dev https://github.com/sideprotocol/sidechain.git
     cd sidechain
     git checkout v0.8.0
     make install
     sided version

 Set up variables

     sided init NodeName --chain-id=S2-testnet-1

Download Genesis:

        wget https://github.com/Validator247/Side-protocol-S2-Testnet1/raw/main/genesis.json -O genesis.json

Download addrbook

        wget https://github.com/Validator247/Side-protocol-S2-Testnet1/raw/main/addrbook.json -O addrbook.json

Create Service

    sudo tee /etc/systemd/system/sided.service > /dev/null <<EOF
    [Unit]
    Description=sided Daemon
    After=network-online.target
    [Service]
    User=$USER
    ExecStart=$(which sided) start
    Restart=always
    RestartSec=3
    LimitNOFILE=65535
    [Install]
    WantedBy=multi-user.target
    EOF
    sudo systemctl daemon-reload
    sudo systemctl enable sided  

State-sync

    sudo systemctl stop sided

    SNAP_RPC="https://side-rpc.validatorvn.com:443"

    cp $HOME/.side/data/priv_validator_state.json $HOME/.side/priv_validator_state.json.backup
    sided tendermint unsafe-reset-all --home ~/.side/ --keep-addr-book

    LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
    BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
    TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
    echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

    sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
    s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
    s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
    s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" ~/.side/config/config.toml
    more ~/.side/config/config.toml | grep 'rpc_servers'
    more ~/.side/config/config.toml | grep 'trust_height'
    more ~/.side/config/config.toml | grep 'trust_hash'

    sudo mv $HOME/.side/priv_validator_state.json.backup $HOME/.side/data/priv_validator_state.json

    sudo systemctl restart sided && journalctl -u sided -f -o cat

 Create a wallet

         sided keys add wallet

Recover existing key

        sided keys add wallet --recover

Query Wallet Balance

    sided q bank balances $(sided keys show wallet -a)

Check sync status

    sided status 2>&1 | jq .SyncInfo.catching_up

Create Validator

    sided tx staking create-validator \
    --amount=1000000uside \
    --pubkey=$(sided tendermint show-validator) \
    --moniker=noname \
    --chain-id=S2-testnet-1 \
    --commission-rate=0.05 \
    --commission-max-rate=0.10 \
    --commission-max-change-rate=0.01 \
    --min-self-delegation=1 \
    --from=wallet \
    --identity="keybase" \
    --website="website" \
    --security-contact="email" \
    --details="i love validator247" \
    --fees=2000uside --gas=200000 \
    -y


Delegate tokens to your validator

        sided tx staking delegate $(sided keys show wallet --bech val -a)  <AMOUNT>uside --from wallet --fees 2000uside --gas 200000 -y

                
