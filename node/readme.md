## NODE JS SERVER WITH NGINX
This gives you an immediate node/nginx local development environment.  If you want to just be up and running, pull down both the `node.box` and the Vagrantfile.  This will give you ability to run it right away.  If you want to start from scracth you will need to follow these steps.  First build a base box with nodeJs and nginx.  Then export that base box to node.box.  Finally use the new `Vagrantfile` to run node.box.

### Contents
- [Building your box](#building-your-box)
- [Deploying Node](#deploying-node)
- [Folders](#folders)
- [Specs](#specs)
- [References](#references)


### Building your box
```
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'virtualbox'
Vagrant.require_version ">= 2.0.0"

Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/trusty64"
  config.ssh.forward_agent = true

# CONFIGURE NODE SERVER
# 
# set hostname, network, and aliases
  config.vm.define "node", primary: true do |node|
    node.ssh.username = "vagrant"
    node.vm.hostname = "node"
    node.vm.network :private_network, ip: '192.168.42.51'
    node.vm.network :forwarded_port, guest: 3000, host: 3000

  # SYNC FOLDERS 
  # 
  # node apps should be in /var/app...
  # sync local with vm folders
    node.vm.synced_folder "app/", "/var/app/", type: "nfs"
    node.vm.synced_folder "app/config/", "/etc/nginx/sites-available/", type: "nfs"

  # set pretty url
    node.hostmanager.aliases = %w(node)

  # SET RESOURCES
  #
  # box must have min 256MB and 25% CPU
  # set provider
    node.vm.provider "virtualbox" do |vb|
      vb.customize ["modifyvm", :id, "--cpuexecutioncap", "25"]
      vb.memory = "256"
      vb.cpus = "1"
    end

  # INSTALL NODE, PM2, NGINX
   node.vm.provision "shell", inline: <<-SHELL
    sudo su -
    cd /var/app/
    apt-get upgrade
    apt-get install -y vim git curl build-essential openssl libssl-dev pkg-config 
    apt-get install -y software-properties-common python-software-properties
    curl -sL https://deb.nodesource.com/setup_6.x -o nodesource_setup.sh
    bash nodesource_setup.sh
    apt-get install nodejs
    npm install -g pm2
    pm2 startup systemd
    apt-get install -y nginx 
    apt-get update 
  SHELL
    
  end

# VAGRANT MESSAGE
  config.vm.post_up_message = "You're ready!"
end
```

*Clean Up*
```
sudo apt-get autoremove
sudo apt-get clean

sudo dd if=/dev/zero of=/EMPTY bs=1M
sudo rm -f /EMPTY
cat /dev/null > ~/.bash_history && history -c && exit
```

*Package Box*
```
vagrant package --output node.box
vagrant destroy
```
You can either upload the box to Vagrant Cloud or use it locally.
- Local: `vagrant box add node node.box`
- Vagrant Cloud: [Signup with Atlas](https://atlas.hashicorp.com/help/vagrant/boxes/create)



### Deploying Node
Use your newly packaged vagrant box with the `Vagrantfile` below.

```
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


  # CONFIGURE WEB SERVER (nginx)
  # 
  # set hostname, network, and aliases
    config.vm.define "node", primary: true do |node|
      node.vm.box = "charlle/node"
      node.vm.box_version = "1.0"
      node.ssh.username = "vagrant"
      node.vm.hostname = "node"
      node.vm.network :private_network, ip: '192.168.42.51'
      node.vm.network :forwarded_port, guest: 1234, host: 1234

    # SYNC FOLDERS 
    # 
    # node should be in /var/...
    # sync local with vm folders
      node.vm.synced_folder "app/", "/var/app/", type: "nfs"
      node.vm.synced_folder "node/config/", "/etc/nginx/sites-available/", type: "nfs"

    # PRETTY URL
    #
    # set pretty url
      node.hostmanager.aliases = %w(node)
    
    # SET RESOURCES
    #
    # box must have min 256MB and 25% CPU
    # set provider
      node.vm.provider "virtualbox" do |vb|
        vb.customize ["modifyvm", :id, "--cpuexecutioncap", "25"]
        vb.memory = "256"
        vb.cpus = "1"
      end

    # START UP NGINX
    #  
    # restart nginx on guest after up
      node.trigger.after :up do
        run_remote "sudo service nginx restart"
        run_remote "sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u vagrant --hp /home/vagrant"
        run_remote "sudo pm2 start app.js"
      end
    end
  #
  # END WEB SERVER (nginx)


# VAGRANT MESSAGE
  config.vm.post_up_message = "You're Node Server is Ready!"
end
```


### Folders 
```
/app (/var/app)
|
|---config/ (/etc/nginx/sites-available/)
/box
|
|---node.box
|---Vagrantfile
```

### Specs
- Installed: `nodejs, npm, pm2, nginx`
- Ram: 256MB
- CPU: 25%

### References
- [Nginx How To](https://www.linode.com/docs/websites/nginx/how-to-configure-nginx)
- [Nginx Config](https://github.com/nginxinc/nginx-wiki/blob/master/source/start/topics/examples/full.rst)


	
