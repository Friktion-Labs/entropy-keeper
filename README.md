# Deploying Entropy Keeper on Google Cloud Platform

https://forum.friktion.fi/t/more-keepers-for-entropy/73


## Server Creation

Create a new Compute Engine instance
1. Series: N1
2. Machine Type: f1-micro
<img width="1022" alt="Screen Shot 2022-04-22 at 09 34 03" src="https://user-images.githubusercontent.com/219298/164745641-1bf81dff-910d-4a5f-b61b-0723af47d46d.png">

4. Change Boot Disk to Ubuntu 20.04 LTS

<img width="607" alt="Screen Shot 2022-04-22 at 09 33 52" src="https://user-images.githubusercontent.com/219298/164745639-121d29f7-a3ed-4c83-974f-23b2907730d7.png">

## Run as Root
We'll run all these commands as a super user
```sh
sudo su
```

## Setup GIT

```sh
eval "$(ssh-agent -s)"
ssh-keygen -t rsa
ssh-add
ssh-add -l -E sha256
```

Follow this guide to add your SSH key to Github https://docs.github.com/en/authentication/troubleshooting-ssh/error-permission-denied-publickey#verify-the-public-key-is-attached-to-your-account

On step 6, you can easily get your key by running:
```sh
tail /root/.ssh/id_rsa.pub
```

## Install Yarn & NPM

```
apt update
apt remove cmdtest
apt remove yarn
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
apt-get update && apt-get install yarn
apt install node-typescript
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
nvm install node
npm install -g ts-node
```

## Clone Entropy

``` 
cd /home/ubuntu
git clone git@github.com:Friktion-Labs/entropy-client.git
cd /home/ubuntu/entropy-client
yarn
cd /home/ubuntu
```

## Install Solana CLI

https://docs.solana.com/cli/install-solana-cli-tools

```sh
sh -c "$(curl -sSfL https://release.solana.com/stable/install)"
export PATH="/root/.local/share/solana/install/active_release/bin:$PATH"
solana --version
```

## Add Phantom Wallet into Solana CLI

Ensure a SOL funded private keypair is in this file location:

```sh
solana-keygen recover 'prompt:?key=0/0' --outfile ~/.config/solana/entropy-mainnet-authority.json
```


## Running the keeper as a Daemon
You'll want it running in the background on boot

```
nano /usr/local/bin/keeper.sh
```

Paste:
```
#!/bin/bash

echo "running keeper"
cd /home/ubuntu/entropy-client
yarn keeper
```

```
nano /etc/systemd/system/keeper.service
```

Paste:
```sh
[Unit]
Description=keeper
After=network.target
StartLimitIntervalSec=20
[Service]
Type=simple
Restart=always
RestartSec=1
User=root
ExecStart=/usr/local/bin/keeper.sh

[Install]
WantedBy=multi-user.target
```

```sh
chmod 744 /usr/local/bin/keeper.sh 
chmod 644 /etc/systemd/system/keeper.service
systemctl enable keeper.service
```


## Reboot

```
reboot
```

## Verify Service is running
```
sudo journalctl -u keeper.service -f
```
