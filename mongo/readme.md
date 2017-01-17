### BUILDING MONGODB
First we are going to build our LoopBack API server with Vagrant, 
then we are going to build a box and package it.  This way we don't 
lose everyting when we `vagrant destroy`
- vbox min requirments = 1GB RAM 50% CPU
- ref: http://friendsofvagrant.github.io


### Contents
- [Vagrant Install](#install_ubuntu)
- [Manual Build](#manual_build)
- [Package Box](#package_box)


#### INSTALL UBUNTU
```
=begin
@author:  Charleston Malkemus
@date:    February 15, 2016
@app:     Mongo Database
@descriptions:
Building mongo box

=end

# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'virtualbox'
Vagrant.require_version ">= 1.6.0"

Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/trusty64"
  config.vm.box_url = "https://github.com/kraksoft/vagrant-box-ubuntu/releases/download/14.04/ubuntu-14.04-amd64.box"
  config.ssh.forward_agent = true

# CONFIGURE HOSTMANAGER
# 
# manage pretty url writing
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.manage_guest = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true


# CONFIGURE API SERVER (mongo)
# 
# set hostname, network, and aliases
  config.vm.define "mongo", primary: true do |mongo|
    mongo.ssh.username = "vagrant"
    mongo.vm.hostname = "mongo"
    mongo.vm.network :private_network, ip: '188.123.55.100'
    mongo.vm.network :forwarded_port, guest: 27017, host: 27017

  # SYNC FOLDERS 
  # 
  # mongo should be in /var/...
  # sync local with vm folders
    mongo.vm.synced_folder ".", "/var/data/"

  # set pretty url
    mongo.hostmanager.aliases = %w(mongo)

  # SET RESOURCES
  #
  # box must have min 1GB and 25% CPU
  # set provider
    mongo.vm.provider "virtualbox" do |vb|
      vb.customize ["modifyvm", :id, "--cpuexecutioncap", "25"]
      vb.memory = "1024"
      vb.cpus = "1"
    end

  # INSTALL MONGO
  #  mongo.vm.provision "shell", inline: <<-SHELL
  #   sudo su -

  #  SHELL
     
  end

# VAGRANT MESSAGE
  config.vm.post_up_message = "Ubuntu is ready!"
end

```


#### MANUAL BUILD
`vagrant ssh`, then run these commands manually.

```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
echo "deb [ arch=amd64 ] http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
sudo apt-get update
sudo apt-get install -y mongodb-org


# mongo files are in this folder
sudo chmod 777 /var/lib/mongodb/

# mongo config is here
sudo vim /etc/mongod.conf
# comment out bind ip address

# check service is working
sudo service mongod start
sudo service mongod stop
```

If you run into problems either start over or attempt to repair the database. `mongod --repair --dbpath /var/lib/mongodb --storageEngine wiredTiger`

#### PACKAGE BOX
```
# clean up and package your box
sudo apt-get autoremove
sudo apt-get clean

sudo dd if=/dev/zero of=/EMPTY bs=1M
sudo rm -f /EMPTY
cat /dev/null > ~/.bash_history && history -c && exit

vagrant package --output mongo.box
vagrant box add mongo.box --name mongo
vagrant destroy
```



