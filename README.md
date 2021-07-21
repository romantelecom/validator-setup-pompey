
# Pompey Finance
```
 ____                                    _____ _                            
|  _ \ ___  _ __ ___  _ __   ___ _   _  |  ___(_)_ __   __ _ _ __   ___ ___ 
| |_) / _ \| '_ ` _ \| '_ \ / _ \ | | | | |_  | | '_ \ / _` | '_ \ / __/ _ \
|  __/ (_) | | | | | | |_) |  __/ |_| | |  _| | | | | | (_| | | | | (_|  __/
|_|   \___/|_| |_| |_| .__/ \___|\__, | |_|   |_|_| |_|\__,_|_| |_|\___\___|
                     |_|         |___/                                      
Pompey Finance <hello@pompey.finance>
```

## Validator Setup Steps - Tendermint
This is my general cheatsheet for setup of an instance on Tendermint Blockchains. We are migrating to Red Hat but most of these steps are for Ubuntu and some are universal. If you know any commands we missed please reach out on Discord. 

<br /><br />
### Generate private/public ssh keys and push to validator/node

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
If using Rocky Linux probably need to setup additional not root user here:
https://www.digitalocean.com/community/tutorials/how-to-create-a-new-sudo-enabled-user-on-centos-8-quickstart
<br /><br />


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

Old Laozi-Testnet-1 
https://hackmd.io/@ntchjb/H1MWbZKOu

Current Laozi-Testnet-2
https://hackmd.io/xrOA-9FSRM6kxCMX9uyv9g# 

Guanyu-Mainnet Test 1
https://github.com/bandprotocol/chain/wiki/Migration-of-GuanYu-Mainnet-to-Laozi-Mainnet

Prometheus Yoda 
https://www.forbole.com/blog/prometheus-exporter-for-bandchain

Outdated - Laozi-Testnet-1
https://hackmd.io/@ntchjb/H1MWbZKOu

<br /><br />

### Band Protocol / Tendermint Useful Commands

Add Your Identity (Add your address here $BAND)
```
bandd tx staking edit-validator --identity $KEY --from $BAND --chain-id band-laozi-testnet2
```
Add Your Website (Add your address here $BAND)
```
bandd tx staking edit-validator --website http://pompey.finance --from $BAND --chain-id band-laozi-testnet2
```
Get Node Details (Useful because NODE-ID is used for presisitance peers) 
```
bandd tendermint show-address 
bandd tendermint show-validator
bandd tendermint show-node-id
```
Unjail yourself (Add your address here $BAND)
```
bandd tx slashing unjail --from $BAND --chain-id laozi-testnet2
bandd query staking validator $BAND --chain-id laozi-testnet2
```
Recover From Seed Phrase - First Remove your existing #CAREFUL!!
```
bandd keys delete $WALLET_NAME
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
