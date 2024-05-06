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

         
