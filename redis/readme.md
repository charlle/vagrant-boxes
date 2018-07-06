### BUILDING REDISDB
First we are going to build our REDIS DB server with Vagrant, 
then we are going to build a box and package it.  This way we don't 
lose everyting when we `vagrant destroy`
- vbox min requirments = 1GB RAM 25% CPU
- ref: http://friendsofvagrant.github.io


### Contents
- [Vagrant Install](#install_ubuntu)
- [Manual Build](#manual_build)
- [Package Box](#package_box)


#### INSTALL UBUNTU
```
=begin
@author:  Charleston Malkemus
@date:    September 13, 2017
@app:     Redis Database
@descriptions:
Building redis box

=end

# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'virtualbox'
Vagrant.require_version ">= 1.6.0"

Vagrant.configure("2") do |config|

  config.vm.box = "bento/ubuntu-16.04"
  config.ssh.forward_agent = true

# CONFIGURE HOSTMANAGER
# 
# manage pretty url writing
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.manage_guest = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true


# CONFIGURE REDIS SERVER (redis)
# 
# set hostname, network, and aliases
  config.vm.define "redis", primary: true do |redis|
    redis.ssh.username = "vagrant"
    redis.vm.hostname = "redis"
    redis.vm.network :private_network, ip: '123.123.55.100'
    redis.vm.network :forwarded_port, guest: 6379, host: 6379

  # SYNC FOLDERS 
  # 
  # redis should be in /var/...
  # sync local with vm folders
    redis.vm.synced_folder ".", "/var/data/"

  # set pretty url
    redis.hostmanager.aliases = %w(redis)

  # SET RESOURCES
  #
  # box must have min 1GB and 25% CPU
  # set provider
    redis.vm.provider "virtualbox" do |vb|
      vb.customize ["modifyvm", :id, "--cpuexecutioncap", "25"]
      vb.memory = "1024"
      vb.cpus = "1"
    end
     
  end

# VAGRANT MESSAGE
  config.vm.post_up_message = "Ubuntu is ready!"
end

```


#### MANUAL BUILD
`vagrant ssh`, then run these commands manually.

```

sudo apt-get install build-essential tcl vim git
cd /tmp
curl -O http://download.redis.io/redis-stable.tar.gz
tar xzvf redis-stable.tar.gz
cd redis-stable
make
make test
sudo make install
sudo apt-get update
sudo mkdir /etc/redis
sudo mkdir /var/data/redis
sudo cp /tmp/redis-stable/redis.conf /etc/redis
sudo vim /etc/redis/redis.conf
  bind 127.0.0.1 
  change supervised to systemd
  dir /var/data/redis
  requirepass 239swATLVSwinton
sudo vim /etc/systemd/system/redis.service
  [Unit]
  Description=Redis In-Memory Data Store
  After=network.target

  [Service]
  User=redis
  Group=redis
  ExecStart=/usr/local/bin/redis-server /etc/redis/redis.conf
  ExecStop=/usr/local/bin/redis-cli shutdown
  Restart=always

  [Install]
  WantedBy=multi-user.target

sudo adduser --system --group --no-create-home redis
sudo chown redis:redis /var/data/redis
sudo chmod 770 /var/data/redis

# start redis
sudo systemctl start redis

# check redis status
sudo systemctl status redis

# enable redis @ startup
sudo systemctl enable redis

# redis config is here
sudo vim /etc/redis/redis.conf
```


#### PACKAGE BOX
```
# clean up and package your box
sudo apt-get autoremove
sudo apt-get clean

sudo dd if=/dev/zero of=/EMPTY bs=1M
sudo rm -f /EMPTY
cat /dev/null > ~/.bash_history && history -c && exit

vagrant package --output redis.box
vagrant box add redis.box --name redis
vagrant destroy
```



