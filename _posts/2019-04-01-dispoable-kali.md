---
layout: post
published: true
featured: false
title: Disposable Kali Linux
headline: 'Disposable Kali Linux'
description: How to effortlessly spin up Kali Linux with Vagrant
modified: '2019-04-01'
categories:
  - hacking
  - tools
tags: hacking kali linux vagrant virtualbox oscp pentest
---
## The Problem

I use Kali Linux on an almost daily basis, whether it's for penetration testing, studying, practice or research.  Because Kali is specifically built for security testing and auditing, best practices are to destroy the installation and start fresh after each use or engagement.  

Without significant modification, Kali is not meant to be your long term, primary operating system.
My problem is that I spend a lot of time setting up Kali from scratch, even when importing the OVA appliances from Offensive Security and I have learned to automate as many steps of my workflow as possible as a security researcher and tester.  Time is one of my most limiting constraints.

## The Solution

I run Kali on virtual machines locally on my Ubuntu laptop mostly using Oracle's VirtualBox.  This works well for me, it's sufficiently performant with hardware virtualization enabled in the BIOS, an SSD and plenty of RAM.  In fact, I can spin up a small lab including Windows machines for setting up a scenario.  But again, this can all be time consuming even with snapshots and exporting OVA's.

Enter Hashicorp's Vagrant tool, which provides an abstracted platform for managing virtual machines (or even containers and cloud services).   Using Vagrant for my Kali Linux virtual machines now let's me spin up an instance, ready for use with all of my customizations, updates and preferred settings.  I stuck my solution in a Github repo to share and evolve.

## Installing

### Prerequisites
Ensure you have the following installed and working:

-VirtualBox 6.0 or higher
-Vagrant 2.0 or higher

_Note that I am using this solution on Ubuntu 19, your experience may be different on other distributions or using Windows._ 

### Download

- Git clone the Disposable Kali repo on Github or just download the Vagrantfile to a directory (see Resources below)
- Open a terminal window and navigate to the directory containing your Vagrantfile
- Use _vagrant validate_ to ensure the file is intact

## Configuration

Before provisioning, open the Vagrant file in a text editor and check through the settings, especially these.

### Synced Folder

I like having a folder shared between my host operating system and the Kali virtual machine so that I can easily access logs, commands, scripts and screenshots between both.  Customize the paths here or comment out the line to prevent the shared folder from being setup.

```ruby

# [OPTIONAL] comment out to skip
# create a shared folder between host and vm 
kali.vm.synced_folder "~/shared", "/root/shared", create: true, owner: "root", group: "root", automount: true

```
<p> </p>
### Firewall Rules

This one is really important.  In this script I install UFW, enable it (which blocks incoming traffic) and then open up some ports I use commonly for downloading tools and catching reverse shells.  Customize this or comment out the script.  
There's nothing worse than working on an exploit for an hour and realizing the reverse shell isn't working because your firewall is blocking it!!!

```ruby

#customize to ports your liking... 
$script_network_ufw = <<-SCRIPT
  echo "--- network_ufw running... "
  apt-get install ufw -y
  apt-get install gufw -y
  ufw allow 22/tcp
  ufw allow 80/tcp
  ufw allow 9021/tcp    
  ufw allow 9022/tcp
  echo yes | ufw enable
  ufw status verbose
  echo "--- network_ufw completed. "
SCRIPT

```
<p> </p>
## First Use

The first time you provision the virtual machine using "vagrant up", the process will take longer than subsequent use because you will be downloading the base box, configuring the VM and running scripts.  

```shell

vagrant up

```
<p> </p>
Once complete, do the following:

- Use "vagrant ssh" to login as root
- The default password for the Offec Kali box is "toor"
- Use "passwd" to change the password immediately

## Usage

From this point forwards, you can just "vagrant up", "vagrant ssh" to login and "vagrant halt" to shut down the VM.

### Common Vagrant commands

```shell

# Login (as root)
vagrant ssh

# Stop the VM gracefully
vagrant halt

# To destroy the VM
vagrant destroy

# Start the VM (or re-provision if it was destroyed)
vagrant up

```
<p> </p>
## Headless Usage

My favorite aspect of this, especially with Ubuntu as my host operating system is that I can use the VM completely headless without having to start the VirtualBox GUI and get a full desktop to just use a bunch of terminal windows.  I can also start X applications forwarded over the SSH connection such as Firefox and Burp Suite.

If you find this awkward and need the Kali desktop, then open up VirtualBox, choose your VM and select "Show" to get a full desktop.

## Roadmap

I have a lot of ideas to make this better, especially centralizing and making customization easier.

Some future ideas:

- Improve and centralize customization settings
- Add a Windows host for exploit compilation, debugging and testing and malware analysis
- Add lab hosts like Metasploitable and Vulnhubs

## Resources

- <a href="https://www.virtualbox.org/" target="_blank">Virtual Box</a>
- <a href="https://www.offensive-security.com/kali-linux-vm-vmware-virtualbox-image-download/" target="_blank">Offensive Security Kali Appliances</a>
- <a href="https://www.kali.org/downloads/" target="_blank">Kali.org ISO Images</a>
- <a href="https://www.vagrantup.com/" target="_blank">Hashicorp Vagrant</a>
- <a href="https://app.vagrantup.com/kalilinux/boxes/rolling" target="_blank">Kali Vagrant Box</a>
- <a href="https://github.com/stevemcilwain/Disposable-Kali" target="_blank">Disposable Kali on Github</a>

Please contribute by forking on Github and issuing pull requests.

<p>&nbsp;</p>
<p>&nbsp;</p>