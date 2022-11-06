# Go Ethereum PoA Setup Sample 2022

## Description

In this repo I will describe the results I learned to create a private ethereum network using Geth with PoA consensus. In my case, I will use 2 VPS machines from AWS to run and connect each node. If you want on a local machine you can create 2 node folders directly in the project folder and later there will be additional commands when running the node.

## Steps to Start

### Create a Network & Start a Network

1. Make sure you have installed ethereum (to use the available packages), if you can't follow the command below

```
# This will add to the machine repository and install it
$ sudo add-apt-repository -y ppa:ethereum/ethereum && sudo apt update && sudo apt install ethereum
```

2. Create folder for our project and node inside project folder

```
$ mkdir geth-network; cd geth-network; mkdir node0
```

3. Go to inside node0 folder and add account using `geth`. Don't forget to save the address that you got from `geth`

```
$ geth --datadir "./data" account new
```

4. After that we create a Genesis File using `puppeth`

```
$ puppeth
```

5. Follow these steps when creating Genesis File after running `puppeth` (Remember to follow this steps without single quote [''])

```
- Add your network name
- Choose '2' for Configure new genesis
- Choose '1' for Create new genesis from scratch
- Choose '2' for using PoA (Proof-of-Authority) Consensus
- set '5'
- Add node0 address as seal [Obtained from Step 3]
- Add node0 address for be pre-funded
- type 'yes'
- set your chainID whatever you want, ex: 1337
```

6. After finishing creating the genesis file, then we will export the Genesis File that we have created

```
- Choose '2' for Manage existing genesis (menu options will change if you have created a genesis file)
- Choose '2' for Export genesis configuration
- Enter and the file will be in the currently active folder
```

7. Exit `puppeth` with pressing CTRL+D
8. We will initialize our geth with the genesis file that has been created (make sure it's in the same folder as the genesis file)

```
$ geth --datadir ./data init ./genesisfile.json
```

> if successful there will be reading in INFO about the status of genesis that was successfully writen

9. To start our network, here is the command that will run the network (remember without brackets [])

```
geth --networkid [CHANGE WITH YOUR CHAINID] --datadir ./data --port 30303 --ipcdisable --syncmode full --http --allow-insecure-unlock --http.corsdomain "*" --http.port 8545 --http.addr "[CHANGE WITH YOUR PRIVATE IP]" --unlock [CHANGE WITH YOUR NODE ADDRESS] --password ./password.txt --mine --http.api personal,admin,db,eth,net,web3,miner,shh,txpool,debug,clique --ws --ws.addr 0.0.0.0 --ws.port 8546 --ws.origins '*' --ws.api personal,admin,db,eth,net,web3,miner,shh,txpool,debug,clique --maxpeers 25 --miner.etherbase 0 --miner.gasprice 0 --miner.gaslimit 9999999 console
```

> because node0 is a miner, there will be INFO about mining if successful

### Connect to our Private Network

1. We will use Metamask to connect the private ethereum that we have created
2. Go to Settings > Networks > Choose Networks that we will edit
3. Add `Network Name` whatever you want
4. Add `New RPC URL` with public IP from your VPS/machine, ex: http://your-ip-public:8545
5. Add `Chain ID` according to what you entered when creating the network, ex: 1337
6. Save

### Import Account

1. Next we will import the account that was created when creating the node. The file location is in `data/keystore` and the filename prefix is `UTC`
2. Click the profile icon in the right corner and then there will be an `Import Account`
3. Change the type to JSON File, then upload the file in the `data/keystore`
4. Enter the password when creating the account and Import (import process is quite long)
5. If successful, click the profile icon and later there will be a new account imported

### Test private network to Smart Contract

1.  We can test the connection to the smart contract using the [Remix IDE](remix.ethereum.org/)
2.  Add sample code from the [solidity docs](https://docs.soliditylang.org/en/v0.8.17/introduction-to-smart-contracts.html) to test it
3.  After generating the sample code and compiling it, go to `Deploy & run transactions` and change the `Environment` to `Injected Provide - Metamask`
4.  Deploy and choose our imported account then make transaction to see if successfully connected to the private network to continue the transaction or not

## Addition

> some additions to make our work easier

### Create a service on linux (automatically start running node)

1. Create a service in `etc/systemd/system` with root privileges to make it easier (because it requires root access too)
2. Create a file with the name `geth.service`

```
$ nano geth.service
```

3. Add the code below to the file

```
[Unit]
Description=Go Ethereum Client

[Service]
User=root
Type=simple
Restart=always
ExecStart=/usr/bin/geth --networkid 1337 --datadir /opt/ethPoA/node0/data --port 30303 --ipcdisable --syncmode full --http --allow-insecure-unlock --http.corsdomain "*" --http.port 8545 --http.addr "172.31.4.255" --unlock 0x33Ae0945B81ba86748a5dB02c6f6941Cedd8d03C --password /opt/ethPoA/node0/password.txt --mine --http.api personal,admin,db,eth,net,web3,miner,shh,txpool,debug,clique --ws --ws.addr 0.0.0.0 --ws.port 8546 --ws.origins '*' --ws.api personal,admin,db,eth,net,web3,miner,shh,txpool,debug,clique --maxpeers 25 --miner.etherbase 0 --miner.gasprice 0 --miner.gaslimit 9999999

[Install]
WantedBy=default.target
```

4. Save (CTRL+O) then Exit (CTRL+X)
5. Then run the command below to restart, start, and check the service that we have created

```
systemctl daemon-reload <-- this will restart the daemon
systemctl enable geth.service <-- this will active the service
systemctl start geth.service <-- this will start the service
systemctl status geth.service <-- this will stop the service
```

> the above steps are part of node0. But in node1 the same as the steps above except that some commands are removed (like the `mine` part starts the network, because node1 doesn't work as a miner when creating the genesis file)

#### Note

The steps above are the result of learning from the [Blockholic Channel](https://www.youtube.com/c/Blockholic)
