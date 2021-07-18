# Pompey Finance
* "Pompey Finance Support" <hello@pompey.finance>
<br>

## Validator Setup Steps - Tendermint
Setup Steps Ubuntu Instance Tendermint Blockchain


### Generate private/public ssh keys and push to validator/node

From your local Machine
Generate private & public keys (public key has ".pub" extension)
When prompted, name it something other than "id_rsa" (so you don;t overwrite existing keys)
```
ssh-keygen -t rsa
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
If using Rocky Linux probably need to setup additional not root user here https://www.digitalocean.com/community/tutorials/how-to-create-a-new-sudo-enabled-user-on-centos-8-quickstart

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
### Band Protocol Specific Stuff
Now Linux is setup move to install GO + BANND + YODA Using the latest Guide

Current Laozi-Testnet-2
https://hackmd.io/xrOA-9FSRM6kxCMX9uyv9g# 

Prometheus Yoda 
https://www.forbole.com/blog/prometheus-exporter-for-bandchain

Outdated - Laozi-Testnet-1
https://hackmd.io/@ntchjb/H1MWbZKOu


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
