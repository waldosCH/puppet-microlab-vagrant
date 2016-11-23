# vagrant-labo

#### Table of Contents

1. [Description](#description)
2. [Requirements](#requirements)
3. [Setup - The basics of getting started with [modulename]](#setup)
4. [Labo](#labo)

## Description

This module is a labo running on Vagrant with Virtualbox. 

The goal is to install a [Node.js](https://nodejs.org/en/) which serves a simple "Hello World" on the 8080 port.

![alt text](https://raw.githubusercontent.com/waldosCH/puppet-microlab-vagrant/master/documentation/helicopterview.png "Labo architecture")


## Requirements

For that labo the following must be present on your machine:

- [VirtualBox](https://www.virtualbox.org/manual/ch02.html)
- [Vagrant](https://www.vagrantup.com/docs/installation/)


## Setup

### 1. Clone repo
```bash
git clone https://github.com/waldosCH/puppet-microlab-vagrant.git
```

### 2. go into the directory
```bash
cd puppet-microlab-vagrant
```

### 3. retrieve the modules that are needed 
```bash
run r10k puppetfile install [-v]
```


## Labo
The goal of this labo is to have a nodejs server responding with the message declared in the mynode class.

### Step 1

Checkout the withouthiera branch. This will just not link hiera on the VMs.

```bash
git checkout withouthiera
```

Here is an overview of the important files in that structure (have a look in them):

- Vagrantfile : this defines your virtual machines
- Puppetfile : this is a description of the module that are pulled to the puppet modules directory (used by the r10k command)
- manifests/infra.pp : this is the puppet description file that is used to assign a class to a node

### Step 2

Start one of the two nodes.

```bash
vagrant up node1
```

When the machine is up ssh into the machine with another terminal.
```bash
vagrant ssh node1
```

### Step 3

Now we are going to code the module mynode (modules/mynode).

[Puppet module documentation](https://docs.puppet.com/puppet/4.7/reference/modules_fundamentals.html)

[Puppet Resource Type Reference](https://docs.puppet.com/puppet/4.7/reference/type.html)

First of all, we will have to declare a group and a user for nodejs (and its home directory)

```puppet
  group { 'nodejs':
    ensure => present,
  }
  user {'nodejs':
    ensure  => present,
    shell   => '/bin/bash',
    home    => '/srv/node',
    groups  => ['users', 'nodejs',],
    require => Group['nodejs'],
  }
  file { '/srv/node':
    ensure  => directory,
    owner   => nodejs,
    group   => nodejs,
    mode    => '0744',
    require => User['nodejs'],
  }
```

You can verify these declaration in the machine

```bash
cat /etc/group | grep nodejs
# users:x:100:nodejs
# nodejs:x:1001:nodejs
cat /etc/passwd | grep nodejs
# nodejs:x:1001:1001::/srv/node:/bin/bash
```

### Step 4

Now we are going to describe the server with an archive that is going to be extracted.
There is a moudle that is furnished for that kind of operation.

[puppet-archive module - documentation](https://forge.puppet.com/puppet/archive#puppet-archive)

```puppet
  archive { '/tmp/node-v6.9.1-linux-x64.tar.xz':
    ensure        => present,
    extract       => true,
    extract_path  => '/opt',
    source        => 'https://nodejs.org/dist/v6.9.1/node-v6.9.1-linux-x64.tar.xz',
    checksum      => 'd4eb161e4715e11bbef816a6c577974271e2bddae9cf008744627676ff00036a',
    checksum_type => 'sha256',
    creates       => '/opt/node-v6.9.1-linux-x64',
    cleanup       => true,
    #notify        => Service['nodejs'],
  }
```

### Step 5

We have to add the server script, its service files and reload systemd.

```puppet
  file { '/srv/node/server.js':
    ensure  => present,
    owner   => nodejs,
    group   => nodejs,
    mode    => '0744',
    content => template('mynode/server.js.erb'),
    #notify  => Service['nodejs'],
  }
  file {'/etc/systemd/system/nodejs.service':
    ensure  => present,
    owner   => root,
    group   => root,
    mode    => '0744',
    content => template('mynode/node_basic_server.service.erb'),
  }
  file {'/etc/default/nodejs':
    ensure  => present,
    owner   => root,
    group   => root,
    mode    => '0744',
    content => template('mynode/node_basic_server_default.erb'),
  }~>
  Exec['systemctl-daemon-reload']
```

Don't hesitate to provision your virtual machine periodically (after changes applied to the module) to check if everything works as desired.


### Step 6

We now need to describe the service as running.

```puppet
  service { 'nodejs':
    ensure => 'running',
  }
```

And uncomment the lines that are triggering the service.

These commented lines are in:
- File ['/srv/node/server.js']
- Archive ['/tmp/node-v6.9.1-linux-x64.tar.xz']

### Step 7
Provision your virtual machine and check the server response.

```bash
curl localhost:8080
# hello from module
```

### Step 8
The goal now is to play with hiera.

This is put as a module in the path.

To make it happen you now have to checkout the master branch of the vagrant repo (root of the project).

```bash
cd <workspace>/puppet-microlab-vagrant
git checkout master
```

Hiera is a structure that just overload the parameters of the classes that you chose.
It is described in the module/hiera/hiera.yaml file.

When the Puppet agent runs it will build what we call the catalog (this is the desired state of the machine - described in the modules). It will just try to replace all the parameters of the classes with what he finds in the hierarchy (if it does not find any parameter, it will just take the default value or fire an error if there is no).


You can now start the second node
```bash
vagrant up node2
```

And play with de different values in the hierarchy files:

For example : 

```yaml
--- cat hieradata/vagrant/hosts/vagrant-node-1.yaml

---
mynode::message: 'hello world from hosts hiera
```

The variable must be declared in that format.

Then change the value and provision your vm again... etc...
