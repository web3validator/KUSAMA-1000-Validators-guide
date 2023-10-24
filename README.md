# KUSAMA-1000-Validators-guide

Guide on how to install a Kusama node.

## Steps to Take Action:

1. **Run a Node**: Set up and run your Kusama node.
2. **Verify the Main Wallet**: You need to verify the main wallet that will stash through the registrar. This requires locking an additional 0.33 KSM.
3. **Synchronize and Stake**: Wait for synchronization and stake at least 10 coins from your stash. Ensure the commission is no more than 15%. Use a separate wallet for the controller.
4. **Fill Out the Form**: Complete the required form and wait at least 6 days before appearing in official and unofficial leaderboards. Nominations should begin after 7 days.

## Requirements:

- Validators should never be slashed.
- Cannot be hosted on Hetzner or Contabo.
- Validators must have a verified wallet account. Refer to [this video](<link-to-the-video>) for guidance.
- Validators must have their own stake of at least 10 KSM.
- Validators must be connected to private telemetry.
- Validators should have a commission of no more than 15 percent.
- Validators must have separate stash and controller wallets.
- All nodes must update to the latest version within 12 hours of release if marked as "critical" or "high" priority, and 24 hours if marked as "medium" or "low" priority.
- A minimum of one week of stable validator operation is required (your node will appear in the leaderboard closer to the end of the week).
- To earn points, you need to vote and claim rewards.

## Nomination:

- Nomination is given for 4 eras (1 era = 6 hours).
- For an era with 15 percent commission, the reward should be about 0.1 KSM (this can vary).
- Validators may receive more frequent nominations initially, but this will likely reduce to about once a week from the second month onwards.
- Expect to earn approximately 1 to 1.5 KSM per month from one node.
- Up to two nodes can be connected to one validator. However, without its own additional stake, the second subnode may not receive enough for the active set.

## Scoring:

- **Span-inclusion (Inclusion Coverage)**: Earlier nominations score more points.
- **Discovered (Node Launch Time)**: Earlier is preferable.
- **Nominated (Nomination Time)**: The longer since the last nomination, the higher the score.
- **Rank (Successful Validation Completion)**: More successful completions lead to a better score.
- **Unclaimed (Mint Rewards)**: Don't forget to claim rewards.
- **Bonded**: TBD
- **Faults (Errors)**: Fewer is better.
- **Offline**: Being online is crucial; less offline time is better.

Used ports

# for the first node
```bash
--port 30333 \ #RPC   
--ws-port 9944 \
--rpc-port 9933 \ 
--prometheus-port 9615 \ #Prometeus
```
# for the SECOND node  
```bash
--port 30533 \ #RPC
--ws-port 9954 \   
--rpc-port 9953 \
--prometheus-port 9655 \ #Prometeus
```
Server preparation
Copy code

# update repositories  
```bash
apt update && apt upgrade -y
```
# install required utilities 
```bash
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```
Install Docker:

Copy code
```bash
. <(wget -qO- https://raw.githubusercontent.com/SecorD0/utils/main/installers/docker.sh)
```
New node installation
IMPORTANT - in the commands below, replace everything in <> with your own value and remove the <> themselves

Copy code

# create directory
```bash
mkdir -p $HOME/.kusama  
```
# give the right permissions
```bash
chown -R $(id -u):$(id -g) $HOME/.kusama
```
# open used ports. It also works without opening ports   
```bash
ufw allow 30533
```
# run docker, pre-specifying the validator name <moniker>
```bash
docker run -dit \
--name kusama_node \ 
--restart always \  
--network host \
-v $HOME/.kusama:/data -u $(id -u ${USER}):$(id -g ${USER}) \
parity/polkadot --base-path /data --chain kusama \   
--validator --name "<moniker>" \ 
--port 30533 \
--ws-port 9954 \
--rpc-port 9953 \
--prometheus-port 9655 \
--telemetry-url 'wss://telemetry.polkadot.io/submit/ 1' \
--telemetry-url 'wss://telemetry-backend.w3f.community/submit 1'
```
Now the node should appear in telemetry - https://telemetry.w3f.community/

Configuring the validator
After the node is synchronized, we extract the key from our node by entering the command. If the node is not on the standard ports, then change the RPC port at the end of the command

Copy code

# below is a command with a non-standard RPC port  
```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "author_rotateKeys", "params":[]}' http://localhost:9953
```
If you got a similar result, then everything is great
{"jsonrpc":"2.0","result":"0xa0very0long0hex0string","id":1} - copy the key (highlighted in bold), we will need it soon

Copy code

# check if keys were created   
```bash
ls -a $HOME/.kusama/chains/ksmcc3/keystore/
```
Don't forget to save the keys!!!

Go to the site and select Network - Staking - Accounts - Validator
Select the stash and controller accounts, the stake amount and click next
Stake at least 10 coins on your validator
Next, insert our key obtained from the validator node, select the commission reward percentage (set to 15 percent) and sign the transaction
Moving the validator
Start the node on the new server as usual and fully synchronize
After synchronization on the new server, run the command and copy the new key
Copy code
```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "author_rotateKeys", "params":[]}' http://localhost:9953
```
Go to Network - Accounts - Change session keys and change our key. Sign the transaction
Wait for the end of the era
After the end of the era, stop the old node
Updating (Manually)
Copy code

# update image
```bash
docker pull parity/polkadot
```
# stop node
```bash
docker stop kusama_node
```
# delete container 
```bash
docker rm kusama_node
```
Copy code

# run docker, pre-specifying the validator name <moniker>
```bash
docker run -dit \  
--name kusama_node \
--restart always \ 
--network host \
-v $HOME/.kusama:/data -u $(id -u ${USER}):$(id -g ${USER}) \   
parity/polkadot --base-path /data --chain kusama \
--validator --name "<moniker>" \
--port 30533 \  
--ws-port 9954 \
--rpc-port 9953 \
--prometheus-port 9655 \
--telemetry-url 'wss://telemetry.polkadot.io/submit/ 1' \
--telemetry-url 'wss://telemetry-backend.w3f.community/submit 1'
```
Updating (Auto)
Snapshot

[https://polkachu.com/snapshots/kusama](https://polkachu.com/substrate_snapshots/kusama)

Copy code

# snapshot from polkachu
```bash
docker stop kusama_node
```
```bash
docker rm kusama_node
```
```bash
curl -o - -L https://snapshots.polkachu.com/snapshots/kusama/kusama_16151324.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.kusama/chains/ksmcc3/
```

# run the node
Useful commands
Copy code

# view logs
```bash
docker logs kusama_node -fn 100  
```
# restart node
```bash
docker restart kusama_node
```
Leaderboards
https://vegas1kv.com/

https://1k.hirish.net/kusama
https://thousand-validators.kusama.network/#/leaderboard

Monitoring
https://kusama.w3f.community/score/D5khA3qGvd8SDXepSrCGmYRWbNUzdJpjEyg6m1mFT7VtHpw

https://insights.math-crypto.com/kusama/D5khA3qGvd8SDXepSrCGmYRWbNUzdJpjEyg6m1mFT7VtHpw

https://kusama.polkastats.io/validator/D5khA3qGvd8SDXepSrCGmYRWbNUzdJpjEyg6m1mFT7VtHpw

https://kusama.w3f.community/score/DwZmVxujvVZmzmLZJ3wNTqyxBYTPDstCxayK6nwSR9HC1tS

https://insights.math-crypto.com/kusama/DwZmVxujvVZmzmLZJ3wNTqyxBYTPDstCxayK6nwSR9HC1tS

https://kusama.polkastats.io/validator/DwZmVxujvVZmzmLZJ3wNTqyxBYTPDstCxayK6nwSR9HC1tS

Deleting a node

```bash
docker stop kusama_node
```
```bash
docker rm kusama_node
```
```bash
cd $HOME   
rm -rf .kusama/
```
