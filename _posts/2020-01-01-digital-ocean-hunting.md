---
layout: post
published: true
featured: true
title: Using Digital Ocean for Bounty Hunting
headline: 'Using Digital Ocean for Bounty Hunting'
description: How to Setup Digital Ocean Droplets for Bounty Hunting
modified: '2020-01-01'
categories:
  - hacking
  - tools
  - bug-bounty
  - ubuntu
tags: hacking ubuntu linux digital-ocean bug-bounty
---

# Using Digital Ocean for Bounty Hunting

These instructions are how I setup a base Ubuntu 18.04 droplet on Digital Ocean for Bug Bounty Hunting.  The steps below walk you through how to configure Ubuntu for secure remote access, update packages and install some common core packages needed for bug hunting tools.  Once complete, we'll create a snapshot to use for building additional droplets, faster as you try different toolsets out.
<p> </p>
## Digital Ocean

Before starting, sign up for Digital Ocean, you can use my referal link below to get a $100 credit for 6 months. 

<a href="https://m.do.co/c/354c7849dfe2" target="_blank">Digital Ocean - Sign-Up with $100 Credit</a>

A droplet is DO's terminology for a virtual machine (VM).   They have pre-built base images (or you can upload your own).  You are billed for the droplet on or off - by the hour.  After creating and using the droplet, you can then destory it and no longer be billed.  

These instructions will give you a repeatable method for building a core snapshot that can then be used to quickly setup, use and destroy future droplets... and minimize your bill.

Other ways exist to completely automate the setup of droplets using platforms like Vagrant or Ansible.

<p> </p>
## 1.0 Create a Droplet

I'm using their Ubuntu 18.04 base image to create a droplet. Typically I'll use the Standard Plan with a $20/month droplet configuration (2 vcpu's and 4 GB ram).  Select the level of resources you need. 

Create a droplet named "Hunter01", wait for it to provision.  Once you receive an email with the IP address, root username and password then proceed.

<p> </p>
## 2.0 Operational Security

First we need to secure our own system before attacking others, this is called operational security (OPSEC) and it is your responsibility as a bug bounty hunter or pentester to ensure you have good OPSEC to protect the data and access you have for targets.

<p> </p>
### 2.1 Configure Hostname

Make note of the assigned IP address for the droplet, we'll use this to create a local hostname.

From an elevated powershell prompt, run the following using the assigned IP address:

```powershell
echo "[IP_ADDRESS] hunter01" >> C:\windows\system32\drivers\etc\hosts
```

Check your changes:

```powershell
cat C:\windows\System32\drivers\etc\hosts
ping hunter01
```

You can manually edit the hosts file to clean it up, but you must be using an elevated prompt to do so.

```powershell
notepad C:\windows\System32\drivers\etc\hosts
```

<p> </p>
### 2.2 Add New User

Connect to the droplet using the root user and password emailed from DO.

```powershell
ssh root@hunter01
```

Once logged in, follow the prompts to set a new password for the root account.

Next add a new user account called "hunter" and add it to the sudo users group.

```bash
adduser hunter
usermod -aG sudo hunter
```

Disconnect and reconnect as the new user.

```powershell
ssh hunter@hunter01
```

<p> </p>
### 2.3 Setup SSH Keys

Assuming you already have a local SSH keypair, we'll add your public SSH key to the new droplet.

Generate keys in the droplet using ssh-keygen, then add an authorized_keys file.

```bash
ssh-keygen
nano ~/.ssh/authorized_keys
```

Paste your public key from your local system, for example ```C:\Users\<username>\.ssh\id_rsa.pub``` , into the editor and save (CTRL-S, CTRL-X).

Set the permissions on the new file.

```bash
chmod -R go= ~/.ssh
```

Now exit your SSH session and reconnect.  You should be prompted for your SSH key password instead this time (if set).

<p> </p>
### 2.4 Secure SSH

While connected as your new user, change SSHD config to block login attempts for root and for passwords.

```bash
sudo nano /etc/ssh/sshd_config
```

Make the following edits, find these entries and change both to "no":

```bash
PermitRootLogin no
PasswordAuthentication no
```

Save with CTRL-S, CTRL-X and restart SSH:

```bash
sudo systemctl restart ssh
```

<p> </p>
### 2.5 Firewall

Enable the UFW firewall to only allow incoming SSH connections.

```bash
 sudo ufw allow OpenSSH
 sudo ufw enable
 sudo ufw status numbered
```

<p> </p>
## 3.0 Updates

Apply the latest system updates using apt.

```bash
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get dist-upgrade -y
sudo apt autoremove -y
sudo reboot

```

In most cases having the latest updates will also increase operation security and ensure you have the latest dependencies for other packages you'll be installing. 

<p> </p>
## 4.0 Networking

Change DNS resolvers to the desired settings, in this example I am using Cloudflares public DNS servers 1.1.1.1 and 1.0.0.1as resolvers.

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

This file has a yaml syntax.  Locate the lines under "nameservers" and replace with your preferred resolvers.  

```
nameservers:
    addresses:
    - 1.1.1.1
    - 1.0.0.1
```

Save the file then apply the changes with:

```bash
sudo netplan apply
```

You can check your current settings with:

```bash
networkctl status
```

<p> </p>
## 5.0 Desktop Environment

In some cases, you may want to use local desktop tools such as Burp Suite or Maltego on the droplet, in which case we need to install a desktop environment and allow remote RDP access securely.

<p> </p>
### 5.1 Install XFCE Desktop

The XFCE desktop is lightweight and less resource demanding than desktops like Gnome or KDE.  Install using apt:

```bash
sudo apt-get install xfce4 -y
sudo apt install xfce4 xfce4-goodies xorg dbus-x11 x11-xserver-utils
```

<p> </p>
### 5.2 Install RDP

Install the  XRDP server:

```bash
sudo apt install xrdp -y
```

Edit the config to start the XFCE desktop on connect:

```bash
echo "exec startxfce4" >> /etc/xrdp/xrdp.ini
```

Restart

```bash
 sudo systemctl restart xrdp
```

Now check the status to make sure it is running:

```bash
sudo systemctl status xrdp
```

<p> </p>
### 5.3 Connect via RDP

Instead of opening a firewall port for RDP (TCP 3389), tunnel the connection through SSH to keep good operational security.  Exit your SSH session if open and reconnect using tunnelling:

```bash
 ssh -L 127.0.0.1:33389:hunter01:3389 hunter@hunter01
```

This command will setup local port 33389 to forward RDP traffic to the droplet on port 3389 once you connect via SSH.  The traffic will be encapsulated inside of the encrypted SSH tunnel and your RDP service cannot be directly attacked.

Once the SSH session is connected, start the RDP client and enter your local address to connect:

```
127.0.0.1:33389
```

<p> </p>
## 6.0 Core Packages

Install some essential packages.  We'll need the python and "go" programming languages because many of the tools we'll use are built with these.

**Golang and Python**

```bash
sudo apt-get install golang -y
sudo apt-get install python python-pip -y
```

**Configure Golang**

Edit your .profile to add exports for go with ```nano ~./profile ```

```bash
# Go
export GOPATH=$HOME/go
export PATH=$PATH:/usr/local/go/bin:$GOPATH/bin
export GO111MODULE=on
```

Now reload your profile after saving the changes:

```bash
source ~/.profile
```

<p> </p>
## 6.0 ZSH

Add ZSH for a better shell and to support plugins. 

```bash
sudo apt-get install fonts-powerline -y
sudo apt-get install zsh -y
```

Add the "oh-my-zsh" framework

```bash
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

Once completed, edit the .zshrc file with ```nano ~/.zshrc``` and change the following values, then save.

```bash
ZSH_THEME="agnoster"

plugins=(git extract)

#Add aliases at the end of the file
```

Reload:

```bash
source ~/.zshrc
```

* If your prompt is missing fonts, it's probably because your Windows terminal application doesn't have the powerline fonts installed.  Clone the powerline git repo and run install.ps1.

<p> </p>
## 7.0 Snapshot

Now you've got the basics all setup, shutdown the droplet:

```bash
sudo poweroff
```

Then go to your DO control panel and take a snapshot.  From this point forward you can install different toolsets for bug bounty hunting, such as the PTF framework or other custom setup scripts and always roll back to this snapshot if something goes awry.
<p> </p>
<p> </p>


