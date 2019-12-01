---
layout: post
published: true
featured: true
title: Disposable Kali Linux 0.2
headline: 'Disposable Kali Linux 0.2'
description: Updates and improvements to Disposable Kali Linux
modified: '2019-06-01'
categories:
  - hacking
  - tools
tags: hacking kali linux vagrant virtualbox oscp pentest
---
## Version 0.2.0 Released
This new version is a complete refactor adding easier and more versatile customization.

## Features
- changing "vagrant ssh" to use root
- adding swap space
- setting BASH aliases
- adding extra word-lists
- installing wine
- update & dist-upgrade
- prepping Metasploit
- cloning useful git repos like AutoBlue and AutoRecon
- installing common exploit dependencies and mingw
- setting up UFW
- experimental FuzzBunch installer on wine

## External Shell Scripts
All shell scripts have been moved from inline to external files. This means you can also run the shell scripts directly without vagrant.

For example, here is the shell script that installs UFW and sets up an initial ALLOW list as the first argument:

```bash

curl -s https://raw.githubusercontent.com/stevemcilwain/Disposable-Kali/master/scripts/network_ufw.sh | bash -s 80/tcp

```

## Easy Configuration
The new version caters to easy customization with configuration variables in the Vagrantfile.

## Vagrant Box Settings

```ruby

# BOX_PATH:  the name or full url of the base box to use
BOX_PATH = "offensive-security/kali-linux"

# BOX_UPDATE: set to true to check for base box updates 
BOX_UPDATE = true

```
<p> </p>

## Virtual Machine Settings

```ruby

# VM_NAME: set the name of the virtual machine and host name
VM_NAME = "kali"

# VM_MEMORY: specify the amount of memory to allocate to the VM
#VM_MEMORY = "8192"
VM_MEMORY = "4096"
#VM_MEMORY = "2048"

# VM_CPUS: specify the number of CPU cores to allocate to the VM
VM_CPUS = "2"
Script Settings
# SWAP_ADD: if enabled, will add the amount of SWAP_ADD_GB to current swap space
#           which is usually 2GB for the Kali base box
SWAP_ADD = true
SWAP_ADD_GB = 2

# SHELL_ALIASES: if enabled, will add BASH aliases to .bash_aliases.
#                Aliases can be customized in /scripts/shell_aliases.sh
SHELL_ALIASES = true

# UFW_INSTALL: if enabled, will install UFW with an allow rule for the ports
#              in UFW_ALLOW.  UFW will be left disabled, activate manually.
UFW_INSTALL = false
UFW_ALLOW = "22,80,443,4443,4444/tcp"

# PKGS_UPGRADE: if enabled, will run update & upgrade
PKGS_UPGRADE = false
Please report any bugs or problems via Github issues. 

```
<p> </p>
## Resources

- <a href="https://github.com/stevemcilwain/Disposable-Kali" target="_blank">Disposable Kali on Github</a>

Please contribute by forking on Github and issuing pull requests.

<p>&nbsp;</p>
<p>&nbsp;</p>