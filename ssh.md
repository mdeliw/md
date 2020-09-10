[toc]

# SSH

## Check if SSH Key exists

```bash
ls -al ~/.ssh
# if you have already generated SSH keys, you should see the following files:
# id_rsa, id_rsa.pub, and known_host
```

## Create a new key pair

```bash
cd ~/.ssh
ssh-keygen -t rsa -b 4096 -C your@email.com
# accept the default filename
# optionally create a passphrase or leave is blank

# Two files will be created
ls ~/.ssh
# id_rsa is the private key you need to protect and never reveal to anyone
# id_rsa.pub is the public key you can share

# if ever need to add or change passphrase
ssh-keygen -p
```

## Copying the public SSH key

```bash
# Copies the content of id_rsa.pub to your clipboard
pbcopy < ~/.ssh/id_rsa.pub

# or copy the output to your clipboard
cat ~/.ssh/id_rsa.pub
```

## Add your SSH key your ssh-agent

`ssh-agent` is a program that starts when you log in and stores your private keys. For it to work properly, it needs to be running and have a copy of your private key. By running this, the default private key is used.

```bash
# make sure ssh-agent is running
eval "$(ssh-agent -s)"

# add your private key to ssh-agent
ssh-add ~/.ssh/id_rsa

# check your private key was added to ssh-agent
ssh-add -l
```

## Manage multiple SSH keys

Although its a good practice to have only one key pair per device, you may need to use more than one pair of key pair. The issue now is that you need to specify which key pair to use for which ssh connection. 

```bash
#create a new file
nano ~/..ssh/config

# contents
Host github.com
	HostName github.com
	User mdeliw
	IdentityFile ~/.ssh/id_rsa
	IdentitiesOnly yes
	AddKeysToAgent yes
	UseKeychain yes

Host bitbucket-corporate
    HostName bitbucket.org
    User mdeliw
    IdentityFile ~/.ssh/id_rsa_corp
    IdentitiesOnly yes
	AddKeysToAgent yes
	UseKeychain yes

Host bitbucket-personal
    HostName bitbucket.org
    User mdeliw
    IdentityFile ~/.ssh/id_rsa_personal
    IdentitiesOnly yes
	AddKeysToAgent yes
	UseKeychain yes

Host myserver
    HostName ssh.username.com
    Port 1111
    User username
    IdentityFile ~/.ssh/id_rsa_personal
    IdentitiesOnly yes
	AddKeysToAgent yes
	UseKeychain yes

# delete all keys from ssh-agent
ssh-add -D

# check all keys are deleted from ssh-agent
ssh-add -l
```

```bash
# use id_rsa_corp key
git clone git@bitbucket-corporate:company/project.git

# use id_rsa_personal key
git clone git@bitbucket-personal:username/my-project.git

# you can ssh into myserver without entering port and username
ssh myserver
```

