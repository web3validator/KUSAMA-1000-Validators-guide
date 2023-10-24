# KUSAMA-1000-Validators-guide
Guide how to install kusama node
Steps to take action:
Run a node
Verify the main wallet that will stash through the registrar. To do this, you will need to lock an additional 0.33 KSM
Waited for synchronization and staked at least 10 coins from your stash. Commission no more than 15%. Be sure to use a separate wallet for the controller
Fill out the form and wait at least 6 days before appearing in official and unofficial leaderboards. After 7 days, nominations should begin
Requirements:
Validators should never be slashed
Cannot be placed on Hetzner or Contabo
Validators have a verified wallet account. For this, we use the registrar and this video
Validators must have their own stake of at least 10 KSM
Validators must be connected to private telemetry
Validators have a commission of no more than 15 percent
Validators have separate stash and controller wallets
All nodes must update to the latest version within 12 hours of release if marked as "critical" or "high" priority, and 24 hours if marked as "medium" or "low" priority
Minimum one week of stable validator operation (closer to the end of the week the node will appear in the leaderboard)
To earn points you need to vote and claim rewards
Nomination
nomination is given for 4 eras (1 era 6 hours)
for an era with 15 percent should be about 0.1 KSM. Sometimes more sometimes less. If the validator becomes a Para validator (this is chosen randomly), then for the era there may be a larger reward
at first they can nominate more often, but from the second month it will be about once a week
per month from 1 node is approximately 1 to 1.5 KSM
Up to two nodes can be connected to one validator, but in practice there is a hidden setting that the 2 subnodes will receive a nomination, but without its own additional stake, it will not be enough for the active set
Inclusion
Span-inclusion (inclusion coverage) the earlier the nomination, the more points
Discovered (Node launch time) the earlier the preferable
Nominated (nomination time) the longer there was no nomination, the higher the score
Rank (successful completion of each validation process) the more successful completions, the better
Unclaimed (mint rewards) just do not forget to claim rewards
Bonded ()
Faults (errors) the less, the better
Offline (Offline) the less, the better
Example of score changes
Show Image

Used ports
Copy code

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
Now the node should appear in telemetry - https://telemetry.w3f.community/list/0xb0a8d493285c2df73290dfb7e61f870f17b41801197a149ca93654499ea3dafe

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

https://polkachu.com/snapshots/kusama

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
