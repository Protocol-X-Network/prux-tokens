
# ProtocolX Token Templates

This repository contains the official templates used to make PRC20 (fungible) and PRC721 (non-fungible) tokens
on the ProtocolX Blockchain.

**How to use**

First, you'll need an Ubuntu Linux 20+ VM or VPS with to install requirements and compile the contracts.

The next packages are required:

- Solidity compiler
- [Solar for ProtocolX tool](https://github.com/protocol-x-network/solar)
- Open Zeppelin Smart Contract templates
- An [ProtocolX wallet](https://github.com/protocol-x-network/protocolx-core/releases) with a handful of PRONX to pay fees

In the steps below, we'll guide you to get the environment ready.

## Step 1: install prerequisites

Change to the root account with

    sudo -i

Run the next commands to install the Solidity Compiler and NPM:

    apt install software-properties-common
    add-apt-repository ppa:ethereum/ethereum
    add-apt-repository ppa:ethereum/ethereum-dev
    apt-get update
    apt-get install solc
    apt install npm

We'll need version 1.17.8 of Golang, and it is installed with the next commands:

    cd /usr/src
    wget https://go.dev/dl/go1.17.8.linux-amd64.tar.gz
    cd /usr/local
    tar -zxvf /usr/src/go1.17.8.linux-amd64.tar.gz
    cd /bin
    ln -s /usr/local/go/bin/go
    ln -s /usr/local/go/bin/gofmt

Note: only the steps above require root privileges.

## Step 2: install the ProtocolX headless wallet

These steps are done using a standard user account (**not root**).

**Important:** the blockchain weights about 26 GB at the time of writing this tutorial.
Make sure you have enough space for it. 50 GB or more are recommended.

Or...

Download the latest snapshot of the Blockchain from the wallet downloads
at [protocolxnetwork.com](https://protocolxnetwork.com/#About) into the `~/.protocolx` directory
after creating it.

Or...

To decrease the size of the download, you can uncomment the `prune=1024` line below (by removing the # symbol)
to only get the latest 1024 MB (1 GB) of the blockchain.

If you already have a wallet running locally or somewhere else but you can connect to its RPC stack,
skip over to the next step.

    mkdir ~/bin
    cd ~/bin
    wget https://github.com/protocol-x-network/protocolx/releases/download/1.0.0/protocolx-1.0.0-linux.tar.gz
    tar -zxvf protocolx-1.0.0-linux.tar.gz
    mv cli/* ./ && rmdir cli
    cd ~
    mkdir .protocolx
    
    echo "listen=1"             >  .protocolx/protocolx.conf
    echo "server=1"             >> .protocolx/protocolx.conf
    echo "bind=127.0.0.1"       >> .protocolx/protocolx.conf
    echo "port=5888"            >> .protocolx/protocolx.conf
    echo "rpcbind=127.0.0.1"    >> .protocolx/protocolx.conf
    echo "rpcport=5889"         >> .protocolx/protocolx.conf
    echo "rpcuser=pronxrpc"      >> .protocolx/protocolx.conf
    echo "rpcpassword=pronxpass" >> .protocolx/protocolx.conf
    echo "rpcallowip=127.0.0.1" >> .protocolx/protocolx.conf
    echo "staking=0"            >> .protocolx/protocolx.conf
    echo "logevents=1"          >> .protocolx/protocolx.conf
    echo "txindex=0"            >> .protocolx/protocolx.conf
    echo "# prune=1024"         >> .protocolx/protocolx.conf

**Note:** you might need to restart your SSH connection in order to have `~/bin` in your path.

If you want to download the bootstrap instead of syncing frin scratch, type the next commands:

    cd ~/.protocolx
    wget https://ipfs.protocolxcoin.io/ipfs/QmekS4dyu6AeYiHMjrmPt4zzkgfaBLboCzWJwmEJn2wdxr/bootstrap-20221208.zip
    unzip bootstrap-20221208.zip
    rm bootstrap-20221208.zip

Depending on your network speed, it may take a couple of hours (or more) to complete.

Now start the wallet with:

    protocolxd -daemon

If you've chosen to 'prune' the download and get any error, edit the `.protocolx/protocolx.conf` file and comment the
`prune=1024` line by adding a # character at the start.

Once the wallet is in sync with the network, get a deposit address:

    protocolx-cli getnewaddress

Then copy the given address and paste it in a text file.

Fund that address with a few ProtocolX. About 10 PRONX will suffice for tests. Let them get at least 10 confirmations
before continuing.

To see the transaction details (and confirmations count), you can use the next command:

    protocolx-cli listtransactions

The output will look like this:

    [
      {
        "address": "Xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
        "category": "receive",
        "amount": 10.00000000,
        "label": "",
        "vout": 1,
        "confirmations": 3,
        "blockhash": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
        "blockheight": xxxxxx,
        "blockindex": x,
        "blocktime": xxxxxxxxxx,
        "txid": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
        "walletconflicts": [
        ],
        "time": xxxxxxxxxx,
        "timereceived": xxxxxxxxxx,
        "bip125-replaceable": "no"
      }
    ]

## Step 3: prepare sources

Run the next commands to instal Solar:

    cd ~
    go get -v -u github.com/protocol-x-network/solar/cli/solar
    cd bin && ln -s ../go/bin/solar && cd ..

Now clone the Open Zeppelin contract templates used for :

    mkdir ~/contracts
    cd ~/contracts
    git clone git@github.com:protocol-x-network/openzeppelin-contracts.git

Now clone **this** repository in place:

    git clone git@github.com:protocol-x-network/protocolx-tokens.git

## Compiling and deploying

At this moment, you have solc, solar, the smart contracts and a wallet with some ProtocolX in it.
You need to define some environment variables.

Before that, we need to know the hex version of the wallet address you'll use to own the tokens.
That can be achieved with the next command:

    protocolx-cli gethexaddress <your_protocolx_address>

E.G.

    protocolx-cli gethexaddress XNP2kdVeF9nbTWp8fGeasWrDa2gvaeQYTS
    79001e7be0e831e256058c76abfb2ad4f8d5dbe8

The string below the command is the hex string we will need.

Copy the next lines, edit them on a text editor and then paste them on the command line:
    
    cd ~/contracts
    export PRONX_RPC=http://protocolxrpc:protocolxpass@127.0.0.1:5889
    SRC_ADDR=<your_protocolx_address>
    SRC_HEX=<hex_of_your_protocolx_address>
    DEPLOYMENT_DIR=`pwd`
    TOKEN_NAME="A name for your token"
    TOKEN_SYMBOL="MYTOKEN"
    DECIMALS="8"
    INITIAL_SUPPLY="100000000000000"

From the lines above:

1. Go into the `contracts` directory
2. Set access to the wallet RPC interface on an environment variable
3. Define the ProtocolX wallet address that will hold the tokens
4. The hex version of the address
5. Set where are we going to get the compiled files (the same dir we're on, ~/contracts)
6. Set a name for your token, E.G. "My marvelous token"
7. Set a symbol for your token, E.G. "MARV"
8. The amount of decimals you want to use. Eight decimals are encouraged for compatibility.
9. The initial supply in satoshis. Just add **one zero for each decimal** to the supply you want, E.G.:  
   For 1 billion tokens (1,000,000,000) and eight decimals, set it to 1000000000**00000000**  
   For 21 million tokens and eight decimals, set it to 21000000**00000000** 

**Important:** think of a good symbol that isn't used anywhere else (surely not BTC, ETH, USDT, etc.).
You'll have to do some research over the web, specially in [CoinMarketCap](https://coinmarketcap.com),
[CoinGecko](https://www.coingecko.com/) or the
[BitcoinTalk Altcoin anns forum](https://bitcointalk.org/index.php?board=159.0)
to check for symbols being used already and avoid them.

Once that's set, just invoke solar using the smart contract of choice:

    solar deploy "protocolx-tokens/PRC20.sol" "[\"$TOKEN_NAME\",\"$TOKEN_SYMBOL\",$INITIAL_SUPPLY,$DECIMALS]" \
          --env="$TOKEN_SYMBOL" --protocolx_sender="$SRC_ADDR"

You'll get an output like this:

    cli gasPrice 1 1
    🚀  All contracts confirmed
       deployed PRC721.sol => xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

Where `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx` is the smart contract address of your token.

That's all! You'll have a file named `solar.MYTOKEN.json` that contains all the information
of the smart contract, including the ABI.

## If you plan to deploy NFTs

In the sample above, the `PRC20.sol` contract was used, and that's for fungible tokens.

For NFTs, you use the `PRC721.sol` contract, and you'll have to mint every token at a time.

To deploy the contract, you set the environment variables as shown below:

    cd ~/contracts
    export PRONX_RPC=http://protocolxrpc:protocolxpass@127.0.0.1:5889
    SRC_ADDR=<your_protocolx_address>
    SRC_HEX=<hex_of_your_protocolx_address>
    DEPLOYMENT_DIR=`pwd`
    TOKEN_NAME="A name for your NFT"
    TOKEN_SYMBOL="MYNFT"

Then invoke solar:

    solar deploy "protocolx-tokens/ORC721.sol" "[\"$TOKEN_NAME\",\"$TOKEN_SYMBOL\"]" \
          --env="$TOKEN_SYMBOL" --protocolx_sender="$SRC_ADDR"
    
Then you'll have a file named `solar.MYNFT.json`.

You now need to interact with the smart contract to mint at least 1 NFT.
And for this purpose, you can install and use our
[PHP helper](https://github.com/protocol-x-network/prc721-php-helper).
