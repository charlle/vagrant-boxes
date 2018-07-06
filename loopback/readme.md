### BUILDING LOOPBACK
First we are going to build our LoopBack API server with Vagrant, 
then we are going to build a box and package it.  This way we don't 
lose everyting when we `vagrant destroy`
- vbox min requirments = 1GB RAM 50% CPU
- ref: http://friendsofvagrant.github.io


### Contents
- [Vagrant Install](#vagrant_install)
- [Manual Build](#manual_build)
- [Package Box](#package_box)


#### VAGRANT INSTALL
```
=begin
@author:  Charleston Malkemus
@date:    February 15, 2016
@app:     LoopBack
@descriptions:
Building loopback box

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


# CONFIGURE API SERVER (loopback)
# 
# set hostname, network, and aliases
  config.vm.define "api", primary: true do |api|
    api.ssh.username = "vagrant"
    api.vm.hostname = "loopback"

  # SYNC FOLDERS 
  # 
  # loopback should be in /var/...
  # sync local with vm folders
    api.vm.synced_folder ".", "/var/api/"

  # set pretty url
    api.hostmanager.aliases = %w(loopback)

  # SET RESOURCES
  #
  # box must have min 1GB and 50% CPU
  # set provider
    api.vm.provider "virtualbox" do |vb|
      vb.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
      vb.memory = "1024"
      vb.cpus = "1"
    end
     
  end

# VAGRANT MESSAGE
  config.vm.post_up_message = "You're ready!"
end

```

#### MANUAL BUILD
`vagrant ssh`, then run these commands manually.

```
# ! do not select Y to upgrade
#
  # sudo apt-get -y upgrade
  sudo apt-get -y install vim git curl build-essential openssl libssl-dev pkg-config 
  sudo apt-get -y install software-properties-common python-software-properties
 
  # n 7.x is the highest supprted with sqlite & loopback 2
  sudo curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash -
  sudo apt-get install -y nodejs
  sudo update-alternatives --install /usr/bin/node node /usr/bin/nodejs 99
  sudo apt-get -y update

# check versions
# 
# nodejs -v
# npm -v

  # chown of var directory iot create zip file for deploy
  sudo chown -R $USER /var/
  # sudo chown -R $USER /usr/local/
  sudo chown -R $USER /usr/lib/node_modules/
  # sudo npm install -g node-pre-gyp
  sudo npm install -g node-gyp rebuild
  sudo npm install -g loopback-cli
  sudo npm install --unsafe-perm -g strongloop
  # npm install -g strongloop
  # sudo ln -s /usr/lib/node_modules/strongloop/bin/slc.js /usr/bin/slc
```

#### PACKAGE BOX
```
# clean up and package your box
  sudo apt-get autoremove
  sudo apt-get clean

  sudo dd if=/dev/zero of=/EMPTY bs=1M
  sudo rm -f /EMPTY
  cat /dev/null > ~/.bash_history && history -c && exit

  vagrant package --output loopback.box
  vagrant box add loopback.box --name loopback
  vagrant destroy
```



#### POST VAGRANT FILE
```
=begin
@author:  Charleston Malkemus
@date:    February 15, 2016
@app:     LoopBack
@descriptions:
Building loopback box

=end

# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'virtualbox'
Vagrant.require_version ">= 1.6.0"

Vagrant.configure("2") do |config|
  config.ssh.forward_agent = true

# CONFIGURE HOSTMANAGER
# 
# manage pretty url writing
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.manage_guest = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true


  # CONFIGURE API SERVER (loopback)
  # 
  # set hostname, network, and aliases
    config.vm.define "api", primary: true do |api|
      api.vm.box = "charlle/loopback"
      api.vm.box_version = "1.0"
      api.ssh.username = "vagrant"
      api.vm.hostname = "loopback"
      api.vm.network :private_network, ip: '192.168.42.51'
      api.vm.network :forwarded_port, guest: 3000, host: 3000

    # SYNC FOLDERS 
    # 
    # loopback should be in /var/api/...
    # sync local with vm folders
      api.vm.synced_folder ".", "/var/api/", type: "nfs"


    # PRETTY URL
    #
    # set pretty url
      api.hostmanager.aliases = %w(loopback)

    # SET RESOURCES
    #
    # box must have min 1GB and 50% CPU
    # set provider
      api.vm.provider "virtualbox" do |vb|
        vb.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
        vb.memory = "1024"
        vb.cpus = "1"
      end

    # START UP LOOPBACK
    #  
    # restart loopback on vm after up
      api.trigger.after :up do
        run_remote "npm run vagrant"
      end
      
    end
  #
  # END API SERVER (loopback)


# VAGRANT MESSAGE
  config.vm.post_up_message = "LoopBack is ready!"
end
```

