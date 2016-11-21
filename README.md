# vagrant-labo

#### Table of Contents

1. [Description](#description)
2. [Setup - The basics of getting started with [modulename]](#setup)
3. [Labo](#labo)

## Description

This module is a labo running on Vagrant with Virtualbox. 


## Setup
1. git clone repo
2. go into the directory
3. run r10k puppetfile install [-v]
4. run vagrant up node


## Labo
The goal of this labo is to have a nodejs server responding with the message declared in the mynode class.

Step 1:
Take the node.js archive from the net and extract it to /opt.
Source: https://nodejs.org/dist/v6.9.1/node-v6.9.1-linux-x64.tar.xz
sha256: d4eb161e4715e11bbef816a6c577974271e2bddae9cf008744627676ff00036a
The group and user for that must be nodejs:
home: /srv/node

Step 2:
Deploy the server file

Step 3:
Add the systemd startup scripts and reload systemctl

Step 4:
Ensure that the nodejs service is running
