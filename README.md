# Story Protocol Validator Node Setup Guide

Story raised $140M from Tier1 investors. Story is a blockchain making IP protection and licensing programmable and efficient. It automates IP management, allowing creators to easily license, remix, and monetize their work. With Story, traditional legal complexities are replaced by on-chain smart contracts and off-chain legal agreements, simplifying the entire process.

## System Requirements

| **Hardware** | **Minimum Requirement** |
|--------------|-------------------------|
| **CPU**      | 4 Cores                 |
| **RAM**      | 8 GB                    |
| **Disk**     | 200 GB                  |
| **Bandwidth**| 10 MBit/s               |




Follow our TG : https://t.me/CryptoBuroOfficial




## Install dependencies

```
sudo apt update
sudo apt-get update
sudo apt install curl git make jq build-essential gcc unzip wget lz4 aria2 pv -y
```


## Install Go

```
cd $HOME && \
ver="1.22.0" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
source ~/.bash_profile && \
go version
```


## Download Story-Geth binary

```
wget https://story-geth-binaries.s3.us-west-1.amazonaws.com/geth-public/geth-linux-amd64-0.9.2-ea9f0d2.tar.gz
tar -xzvf geth-linux-amd64-0.9.2-ea9f0d2.tar.gz
[ ! -d "$HOME/go/bin" ] && mkdir -p $HOME/go/bin
if ! grep -q "$HOME/go/bin" $HOME/.bash_profile; then
  echo 'export PATH=$PATH:$HOME/go/bin' >> $HOME/.bash_profile
fi
sudo cp geth-linux-amd64-0.9.2-ea9f0d2/geth $HOME/go/bin/story-geth
source $HOME/.bash_profile
story-geth version
```


## Download Story binary

```
wget https://story-geth-binaries.s3.us-west-1.amazonaws.com/story-public/story-linux-amd64-0.9.11-2a25df1.tar.gz
tar -xzvf story-linux-amd64-0.9.11-2a25df1.tar.gz
[ ! -d "$HOME/go/bin" ] && mkdir -p $HOME/go/bin
if ! grep -q "$HOME/go/bin" $HOME/.bash_profile; then
  echo 'export PATH=$PATH:$HOME/go/bin' >> $HOME/.bash_profile
fi
sudo cp story-linux-amd64-0.9.11-2a25df1/story $HOME/go/bin/story
source $HOME/.bash_profile
story version
```


## Initiate Iliad node

Replace "Your_moniker_name" with any name you want 
(Ex: story init --network iliad --moniker cryptoburo )

```
story init --network iliad --moniker "Your_moniker_name"
```


### Peers setup
```
PEERS=$(curl -s -X POST https://rpc-story.josephtran.xyz -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"net_info","params":[],"id":1}' | jq -r '.result.peers[] | select(.connection_status.SendMonitor.Active == true) | "\(.node_info.id)@\(if .node_info.listen_addr | contains("0.0.0.0") then .remote_ip + ":" + (.node_info.listen_addr | sub("tcp://0.0.0.0:"; "")) else .node_info.listen_addr | sub("tcp://"; "") end)"' | tr '\n' ',' | sed 's/,$//' | awk '{print "\"" $0 "\""}')

sed -i "s/^persistent_peers *=.*/persistent_peers = $PEERS/" "$HOME/.story/story/config/config.toml"

if [ $? -eq 0 ]; then
    echo -e "Configuration file updated successfully with new peers"
else
    echo "Failed to update configuration file."
fi
```

## Create story-geth service file
```
sudo tee /etc/systemd/system/story-geth.service > /dev/null <<EOF
[Unit]
Description=Story Geth Client
After=network.target

[Service]
User=root
ExecStart=/root/go/bin/story-geth --iliad --syncmode full
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```

## Create story service file

```
sudo tee /etc/systemd/system/story.service > /dev/null <<EOF
[Unit]
Description=Story Consensus Client
After=network.target

[Service]
User=root
ExecStart=/root/go/bin/story run
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```

## Reload and start story-geth
```
sudo systemctl daemon-reload && \
sudo systemctl enable story-geth && \
sudo systemctl enable story && \
sudo systemctl start story-geth && \
sudo systemctl start story && \
sudo systemctl status story-geth
```



# Check logs

### Geth logs
```
sudo journalctl -u story-geth -f -o cat
```
### Story logs
```
sudo journalctl -u story -f -o cat
```
### Check sync status

```
curl localhost:26657/status | jq
```



# SYNC using snapshot File

**Apply Mandragora snapshots (story client+story-geth)**

Check the height of the snapshot (v0.10.1): Block Number -> 1016207


### install lz4
```
sudo apt-get install wget lz4 -y
```


### Stop node
```
sudo systemctl stop story
sudo systemctl stop story-geth
```


### Backup priv_validator_state.json:
```
sudo cp $HOME/.story/story/data/priv_validator_state.json $HOME/.story/priv_validator_state.json.backup
```

### Download Geth-data
```
cd $HOME && rm -f Geth_snapshot.lz4 && wget -O Geth_snapshot.lz4 https://snapshots.mandragora.io/geth_snapshot.lz4
```

### Download Story-data
```
cd $HOME && rm -f Story_snapshot.lz4 && wget -O story_snapshot.lz4 https://snapshots.mandragora.io/story_snapshot.lz4

```

### Unzip Geth Snapshot file 
```
lz4 -c -d Geth_snapshot.lz4 | tar -xv -C $HOME/.story/geth/iliad/geth
```

### Unzip Story-data Snapshot file 
```
lz4 -c -d story_snapshot.lz4 | tar -xv -C $HOME/.story/story
```


### Backup priv_validator_state.json:
```
sudo cp $HOME/.story/priv_validator_state.json.backup $HOME/.story/story/data/priv_validator_state.json
```

### Restart node 
```
sudo systemctl start story
sudo systemctl start story-geth
```


# Upgrade to Story v0.11.1


## Download Story v0.11.1 File : 
    
    cd $HOME && wget https://raw.githubusercontent.com/CryptoBuroMaster/Story-Node/main/story-v0.11.1.sh
    


## Make the script executable :
    
    chmod +x story-v0.11.0.sh
    

## Run the script to update the Story version :
    
    ./story-v0.11.0.sh
    

### Ensure your node is running correctly by checking the logs:
```
journalctl -u story -f
```


# Upgrade to Story-Geth v0.9.3 version


## Download Story-Geth v0.9.3 File : 
    
    cd $HOME && wget https://raw.githubusercontent.com/CryptoBuroMaster/Story-Node/main/story-geth-v0.9.3.sh

    


## Make the script executable :
    
    chmod +x story-geth-v0.9.3.sh

    

## Run the script to update the Story version :
    
    ./story-geth-v0.9.3.sh
    
    

### Ensure your node is running correctly by checking the logs:
```
journalctl -u story -f
```




# Register your Validator

### 1. Export wallet:
```
story validator export --export-evm-key
```
### 2. Private key preview
```
sudo nano ~/.story/story/config/private_key.txt
```
### 3. Import Key to Metamask 

Get the wallet address for faucet

### 4. You need at least have 1 IP on wallet

Get it from faucet : https://faucet.story.foundation/

Check the sync, the catching up must be 'false'
```
curl -s localhost:26657/status | jq
```
Stake only after "catching_up": false

### 5. Validator registering

Replace "your_private_key" with your key from the step2

```
story validator create --stake 1000000000000000000 --private-key "your_private_key"
```

### 6. Check your validator INFO
```
curl -s localhost:26657/status | jq -r '.result.validator_info' 
```

### 7. check your validator

Explorer: https://testnet.story.explorers.guru/

## BACK UP FILE

### 1. Wallet private key:
```
sudo nano ~/.story/story/config/private_key.txt
```
### 2. Validator key:

```
sudo nano ~/.story/story/config/priv_validator_key.json
```

Join our TG : https://t.me/CryptoBuroOfficial
