
# Critical Staking
```
  _____         _   __    _               __
 / ___/  ____  (_) / /_  (_) ____ ___ _  / /
/ /__   / __/ / / / __/ / / / __// _ `/ / /  	Critical Staking
\___/  /_/   /_/  \__/ /_/  \__/ \_,_/ /_/   	You are being logged
                 
Contact Critical Staking <hello@criticalstaking.com>
```
## Resources for employees and administrators


Core Tendermint Command Reference And Wallet Structure
https://docs.cosmos.network/v0.43/run-node/keyring.html#adding-keys-to-the-keyring

Laozi-Mainnet Setup Doc
https://github.com/bandprotocol/chain/wiki/BandChain-Laozi-Mainnet:-How-to-Join-as-a-Validator

## Validator Setup Steps - Tendermint
This is my general cheatsheet for setup of an instance on Tendermint Blockchains. We are migrating to Red Hat but most of these steps are for Ubuntu and some are universal. We are active on Discord please reach out for questions. 

<br />
### Generate private/public ssh keys and push to validator/sentry node

From your local Machine
Generate private & public keys (public key has ".pub" extension)
When prompted, name it something other than "id_rsa" (so you don't overwrite existing keys)
```
ssh-keygen -t rsa -b 4096 -f ~/.ssh/specialname_rsa
```
Lock down private key
```
chmod 400 ~/.ssh/<YOUR KEY>
```
Push key up to your box
See below if using Digital Ocean for vps
```
ssh-copy-id -i ~/.ssh/<YOUR KEYNAME>.pub username@remote_host
```
Login with ssh
```
ssh -i ~/.ssh/<YOUR SSH PRIVATE KEY> username@remote_host
```
If using Rocky Linux probably need to setup additional not-root user as SELinux does not like running programs in user folder:
https://www.digitalocean.com/community/tutorials/how-to-create-a-new-sudo-enabled-user-on-centos-8-quickstart
<br />


### Update the validator/node
```
sudo apt update
sudo apt upgrade
sudo apt install -y build-essential 
```
On GCP and AWS root login may already be diabled but check to be sure. 
```
vim /etc/ssh/sshd_config
```
Alter the appropriate
```
PermitRootLogin without-password
ChallengeResponseAuthentication no
PasswordAuthentication no
AllowUsers root otheruser
PubkeyAuthentication yes   
```
Restart ssh using 
```
sudo systemctl reload sshd
```
<br /><br />


### Firewall Setup For Band Protocol With UFW

Set defaults for incoming/outgoing ports
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
```
Open ssh port (if you changed to non standard port in /etc/ssh/sshd_config use that or you will be locked out) 
```
sudo ufw allow 22 
```
Add access for prometheus but only from friendly IPs
```
sudo ufw allow from $PROMETHEUS to any port 9100
sudo ufw allow from $PROMETHEUS to any port 26660
```
Add access for your sentry nodes OR if public face open the port.
```
sudo ufw allow from $SENTRY_1_IP to any port 26696
sudo ufw allow from $SENTRY_1_IP to any port 26696
```
Re-enable the firewall
```
ufw enable
ufw status verbose
```

Change Hostname
Change this to something useful
```
sudo hostnamectl set-hostname <NEW_HOSTNAME>
```
<br /><br />

### Prometheus Node Exporter
https://www.linuxtechi.com/install-prometheus-monitoring-tool-centos-8-rhel-8/
https://everstake.one/blog/cosmosmon-a-monitor-for-the-cosmos-node

Install the Node Exporter on each node as per the Centos guide above.


<br /><br />

### Band Protocol Specific Stuff
Now Linux is setup we can move on to install ing GO, BAND and YODA using the latest guides.

Laozi-Mainnet
https://github.com/bandprotocol/chain/wiki/BandChain-Laozi-Mainnet:-How-to-Join-as-a-Validator

Prometheus Yoda 
https://www.forbole.com/blog/prometheus-exporter-for-bandchain


<br />

### Band Protocol / Tendermint Useful Commands

Remember to use ```bandcli``` on guanyu and ```bandd``` on Laozi. And double check anything you do tat involves keys twice so you do not accidentally delete. 

Add Your Identity (Add your address here $BAND)
```
bandd tx staking edit-validator --identity $KEY --from $BAND --chain-id band-laozi-testnet2
```
Add Your Website (Add your address here $BAND)
```
bandd tx staking edit-validator --website http://pompey.finance --from $BAND --chain-id band-laozi-testnet2
```
Check how many nodes you are connected to.
```
curl -sS http://localhost:26657/net_info | jq -r '.result.n_peers'
```
Get Node Details (Useful because NODE-ID is used for presisitance peers) 
```
bandd tendermint show-address 
bandd tendermint show-validator
bandd tendermint show-node-id
```
Delegate 50 band from wallet to your validator
```
bandcli tx staking delegate bandvaloper<YOUR_ACCOUNT> 50000000uband --from band<YOUR_WALLET> --chain-id band-guanyu-mainnet
```
Withdraw staking rewards validator 
```
bandd tx distribution withdraw-rewards $vali_address  --from $vali_wallet --chain-id laozi-mainnet
```
Unjail yourself (Add your address here $BAND)
```
bandd tx slashing unjail --from $BAND --chain-id laozi-testnet2
bandd query staking validator $BAND --chain-id laozi-testnet2
```
Recover From Seed Phrase - First Remove your existing #CAREFUL!!
```
bandd keys delete $WALLET_NAME  #see warning above
bandd keys add $WALLET_NAME --recover

```
Export and import keys 
```
bandd keys export $WALLETNAME > privatekey.txt
bandd keys import $WALLETNAME privatekey.txt 
```

<br /><br />


### Some Useful Yoda Commands
Test the Lambda endpoint is working
```
curl --location --request POST 'URL_OF_YOUR_LAMBDA' \
--header 'Content-Type: application/json' \
--data-raw '{
    "executable": "IyEvdXNyL2Jpbi9lbnYgcHl0aG9uMwoKaW1wb3J0IHN5cwoKZGVmIG1haW4oZGF0YSk6CiAgICByZXR1cm4gZGF0YQoKCmlmIF9fbmFtZV9fID09ICJfX21haW5fXyI6CiAgICB0cnk6CiAgICAgICAgcHJpbnQobWFpbigqc3lzLmFyZ3ZbMTpdKSkKICAgIGV4Y2VwdCBFeGNlcHRpb24gYXMgZToKICAgICAgICBwcmludChzdHIoZSksIGZpbGU9c3lzLnN0ZGVycikKICAgICAgICBzeXMuZXhpdCgxKQo=",
    "calldata": "\"Hello lambda\"",
    "timeout": 3000
}'
```
Re-activate Yoda
```
export CHAIN_ID=band-laozi-testnet2
export WALLET_NAME=$WALLET_NAME

bandd tx oracle activate \
  --from $WALLET_NAME \
  --chain-id $CHAIN_ID
```

Check yoda is active:
```
bandd query oracle validator OPERATOR_ADDRESS --node http://127.0.0.1:26657
bandd query oracle params --node http://127.0.0.1:26657
yoda version
bandd query oracle counts --node http://127.0.0.1:26657
```

### Sentry Node Architecture Settings
Please read more about senry node architecture here: 
https://hub.cosmos.network/main/validators/security.html
https://forum.cosmos.network/t/sentry-node-architecture-overview/454

Config.toml settings are different for the validator and sentry. This assumes you have configured the firwall and VPN settings and nodes are connected.

Validators Nodes should edit their config.toml:

| Config Option  | Setting |
| ------------- | ------------- |
| pex  | false  |
| persistent_peers |  list of sentry nodes |
| private_peer_ids	|  null |
| addr_book_strict |  false |

Sentry Nodes should edit their config.toml:

| Config Option  | Setting |
| ------------- | ------------- |
| pex  | true |
| persistent_peers |  validator node, optionally other sentry nodes |
| private_peer_ids	|  validator node id |
| addr_book_strict |  false |


### Configure Yubikey 

### Migrate Keys & Yoda

Export the keyfile
```
bandcli keys export $WALLET_NAME
```
After entering your passsword a private key will be displayed on terminal, similar to the example below.
```
# 
# -----BEGIN TENDERMINT PRIVATE KEY-----

# ...
# -----END TENDERMINT PRIVATE KEY-----
# 
```
Copy the result to a file 
```
vim $HOME/${WALLET_NAME}_privkey.txt
```
On the new server install this key
```
bandd keys import $WALLET_NAME $HOME/${WALLET_NAME}_privkey.txt

# Please make sure that the key has been imported successfully
bandd keys list

# If key has been import successfully, then delete the file
rm $HOME/${WALLET_NAME}_privkey.txt
```
The keyring folder of Yoda should be backed up
```
# PLEASE MAKE SURE THAT NO TRALING SLASH IN BOTH PATHS
$HOME/.yoda/keyring-test
```


### How to setup QEMU server
Bridged networking:
https://levelup.gitconnected.com/how-to-setup-bridge-networking-with-kvm-on-ubuntu-20-04-9c560b3e3991


### Old Tendermnt notes for myself from a post by greg-szabo 

Adrian is right, I'm relying on unsafe_reset_all not removing the genesis file. I had a hard time understanding the logic and the relations among genesis.json/config.toml/priv_validator.json. Here are the current pieces I'm aware of:

1. ```tendermint init``` generates the priv_validator.json and that's its primary role.
2. Secondly it's generating an example genesis.json, that assumes that the node is a validator node. Since the genesis.json (with public keys of validator nodes) is probably coming from the owners of a network, this is more of an example than anything and it should not be taken too seriously.
As an afterthought it also creates a default config.toml that hooks onto all IP addresses on the default ports and can be overwritten on the command-line. As an example this is fine but for a network, the seeds part will be definitely overwritten and usually the IP addresses will be corrected too on the command-line or in the file. It is only valuable as-is if it is used as an example.
```tendermint gen_validator``` will only generate the priv_validator.json content and print it to the screen. Together with tendermint show_validator it can be used to imitate the ```init``` command.
3. ```tendermint unsafe_reset_all``` will not touch the genesis.json or the config.toml file. It will remove the data folder and reset all counters in the priv_validator.json. It is considered unsafe because it destroys the database but it does not destroy the configuration. ```unsafe_reset_priv_validator``` will not remove the data folder, only reset counters. (I'm not sure in what scenario this is useful.)

### General GCP / AWS Type Stuff

## Resize drive
```
lsblk or df -hT
sudo file -s /dev/nvme01
sudo growpart /dev/nvme0n1 1
sudo resize2fs /dev/nvme0n1p1
df -h
```

## Bastion Hosts 
```
# Shell 1 Bastion Key
ssh -i "~/keys/aws-key.pem" user@public-ip-of-bastion -L 9999:ip-of-the-target:22
# Shell 2 Target Key
ssh -i "~/keys/target-key.pem" user@localhost -p 9999
```
## Copy file up to a node
```
scp file.tar.xz remote_username@10.10.0.2:/remote/directory
```

## Migration Guanyu to New Server

### Snapshot

```
tar czvf /data/snapshot.tar.gz $HOME/.bandd/data/ $HOME/.bandd/files/
```
### or rsync
```
sudo rsync -av --delete --dry-run $HOME/.bandd/data/ /data/.bandd/data
sudo rsync -av --delete --dry-run $HOME/.bandd/files/ /data/.bandd/files
sudo rsync -av --delete $HOME/.bandd/data/ /data/.bandd/data
sudo rsync -av --delete $HOME/.bandd/files/ /data/.bandd/files
```
###  After sync

```
# Target and prrmary systems
sudo systemctl diable bandd
sudo systemctl stop band
```
Check it worked
```
journalctl -f -u bandd.service 
```

### Key data import
```
/home/username/go/bin/bandcli  keys import key.txt
cp priv_validator_key.json $HOME/.bandd/config
cp node_key.json $HOME/.band/config
/home/username/go/bin/bandcli keys list keys import #WALLET_NAME $HOME/key.txt
```
### Check it worked
```
/home/username/go/bin/bandcli keys list
```

### Yoda
```
cd .yoda
tar -xvf yoda.tar.xf
#if lz4's
lz4 -d "$PATH" | tar xvf -
```
