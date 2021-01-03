---
title: "Installing SaltStack Within a Vagrant Environment"
date: 2018-05-20T21:04:50+02:00
draft: false
---
In this blog post, I will describe how to install and use SaltStack in an isolated and safe Vagrant environment. 
This can be useful to get familiar with SaltStack and try its features.
<!--more-->

# Introduction

[SaltStack](https://docs.saltstack.com/en/latest/) is a powerful configuration management tool that lets you automate
provisioning and deployment of your systems and applications. It uses a master-minion paradigm where the master is 
controlling all the minions and applies certain states to minions according to the rules that you can set up.

I'm assuming that you're familiar with Vagrant. I encourage you to check out its 
[official website](https://www.vagrantup.com/).

## Initial set-up of an environment with Vagrant

I'm using an environment that consists of 4 virtual machines:

* `master`
* `web1`
* `web2`
* `loadbalancer`

The latter three are minions. `web1` & `web2` will host and run a basic NodeJS Hello World application, while the 
`loadbalancer` machine will balance an incoming load using Nginx.

With Vagrant you can define the configuration of each of your machines individually, but since both `web1` and `web2`
are identical machines, then you can use some Ruby techniques to simplify it and describe it once.

```ruby
(1..NODES).each do |i|
  config.vm.define "web#{i}" do |minion|
    minion.vm.box = BOX_IMAGE
    minion.vm.network "private_network", ip: "192.168.10.1#{i}"
    minion.vm.network "forwarded_port", guest: 8000, host: "800#{i}",auto_correct: true
    minion.vm.hostname = "web#{i}"

    minion.vm.provision :salt do |salt|
      salt.minion_config = "salt/minion-configs/web-minion"
    end

  end
end
```

What is interesting in this code block is the first line that defines a loop in Ruby. It loops from `1` up to a value 
set in `NODES` variable, that was declared previously. Thus, counter `i` is used within a machine declaration to set 
a VM name, IP-address and exposed HTTP port. The rest that is set within a loop is a simple Vagrant configuration for 
a VM.

Another interesting code block here is: `minion.vm.provision`. I will describe later in the post what it is used for.

The configuration of `master` and `loadbalancer` machines is fairly simple. You can find full Vagrantfile in my 
[GitHub repository](https://github.com/plagov/vagrant-salt/blob/master/Vagrantfile).

## SaltStack set-up

To set up Salt on a new machine, you can use a native package manager or use the bootstrap script provided by the 
SaltStack team with the latest stable version.

### Installing SaltStack with a bootstrap script

To download the script, you will need to run the following command on a new machine.

```sh
curl -L https://bootstrap.saltstack.com -o install_salt.sh
```

Next, you will need to run the script in order to install Salt with a master or minion role. For a master you would 
run the following command:

```sh
sudo sh install_salt.sh -M -A 127.0.0.1
```

`-M` key in this command stands for installing a salt-master. And `-A` configures a minion to reference to a master 
by a provided IP-address or DNS host-name.

To install Salt with a minion role you would run the following command:

```sh
sudo sh install_salt.sh -A 192.168.10.10
```

You don't provide a `-M` key since you're not installing the salt-master role and a value for a `-A` key references 
to an IP-address of a salt-master that was set up a bit earlier.

### Installing SaltStack with a Vagrant salt provisioner

By reading these instructions you might think that it is not a very efficient way to configure salt on your machines. 
SSH-ing to every single machine to curl the script and run it manually is definitely not a good option.

Luckily, Vagrant has a built-in `salt` provisioner, meaning that it provides us options to pre-configure the machine 
with either a salt-master or a salt-minion.

To install a salt-master on a Vagrant VM, provide the following instructions to a VM declaration:

```ruby
master.vm.provision :salt do |salt|
  salt.master_config = "salt/minion-configs/master"
  salt.install_master = true
  salt.no_minion = true
end
```

That instruction will take the file with pre-configured settings from `salt/minion-configs/master` and will put it 
under `/srv/salt/master` on a virtual machine, which is a default location for a salt-master configuration. Vagrant's 
install options `install_master` and `no_minion` are also required for a salt-master installation.

To install a salt-minion, just a file with configurations is required:

```ruby
minion.vm.provision :salt do |salt|
  salt.minion_config = "salt/minion-configs/web-minion"
end
```

This way, instead of SSH-ing into each machine and install salt one-by-one, we specify what parameters we would like 
salt-minion to have and what are for a salt-master. The minimum configuration we have to provide to a salt-minion is 
an address of a salt-master to communicate with. It is done easily with the following configuration in `web-minion` 
file:

```conf
master: 192.168.10.10
```

There are a lot of options that you can predefine for a minion. Check 
[official documentation](https://docs.saltstack.com/en/latest/ref/configuration/minion.html) for more information.

To install salt-master with a default configuration, you can omit a `master_config` option and just stay with 
`install_master` and `no_minion`. And should you need to tune a salt-master as per your needs, you can check an 
[extensive documentation](https://docs.saltstack.com/en/latest/ref/configuration/master.html) on this.

## Summary

In this blog post, I showed an easy option on how you can install SaltStack on a virtual machine with Vagrant. Next, 
when Salt is installed, you can start to try it and get familiar with it.

To check both SaltStack and Vagrant configurations, shown in this blog post, please visit my 
[GitHub repository](https://github.com/plagov/vagrant-salt).
