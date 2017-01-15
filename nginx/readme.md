## NGINX Reverse Proxy
First build the nginx server and package the vagrant box.  Then use the nginx.box in your new `Vagrantfile`.

### Contents
- [Building your box](#building-your-box)
- [Deploying nginx](#deploying-nginx)
- [Folders](#folders)
- [Specs](#specs)
- [References](#references)


### Building your box
```
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


# CONFIGURE PROXY SERVER (nginx)
# 
# set hostname, network, and aliases
  config.vm.define "web", primary: true do |web|
    web.ssh.username = "vagrant"
    web.vm.hostname = "nginx"
    web.vm.network :private_network, ip: '192.168.42.51'
    web.vm.network :forwarded_port, guest: 3451, host: 3451

  # SYNC FOLDERS 
  # 
  # nginx should be in /var/...
  # sync local with vm folders
    web.vm.synced_folder "web/", "/var/web/", type: "nfs"
    web.vm.synced_folder "web/config/", "/etc/nginx/sites-available/", type: "nfs"
    web.sync.host_folder = "web/"
    web.sync.guest_folder = "/var/web/"

  # set pretty url
    web.hostmanager.aliases = %w(nginx)

  # SET RESOURCES
  #
  # box must have min 256MB and 15% CPU
  # set provider
    web.vm.provider "virtualbox" do |vb|
      vb.customize ["modifyvm", :id, "--cpuexecutioncap", "15"]
      vb.memory = "256"
      vb.cpus = "1"
    end

  # INSTALL NGINX
    web.vm.provision "shell", inline: <<-SHELL
      sudo su -
      apt-get upgrade
      apt-get install -y vim git curl build-essential openssl libssl-dev pkg-config 
      apt-get install -y software-properties-common python-software-properties
      
      apt-get install -y nginx    

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
vagrant package --output nginx.box
vagrant destroy
vagrant box add nginx nginx.box
```


### Deploying nginx
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
    config.vm.define "web", primary: true do |web|
      web.vm.box = "nginx"
      web.ssh.username = "vagrant"
      web.vm.hostname = "nginx"
      web.vm.network :private_network, ip: '192.168.42.51'
      web.vm.network :forwarded_port, guest: 3451, host: 3451

    # SYNC FOLDERS 
    # 
    # nginx should be in /var/...
    # sync local with vm folders
      web.vm.synced_folder "web/", "/var/web/", type: "nfs"
      web.vm.synced_folder "web/config/", "/etc/nginx/sites-available/", type: "nfs"
      web.sync.host_folder = "web/"
      web.sync.guest_folder = "/var/web/"

    # PRETTY URL
    #
    # set pretty url
      web.hostmanager.aliases = %w(nginx.vm)
    
    # SET RESOURCES
    #
    # box must have min 256MB and 15% CPU
    # set provider
      web.vm.provider "virtualbox" do |vb|
        vb.customize ["modifyvm", :id, "--cpuexecutioncap", "15"]
        vb.memory = "256"
        vb.cpus = "1"
      end

    # START UP NGINX
    #  
    # restart nginx on guest after up
      web.trigger.after :up do
        run_remote "sudo service nginx restart"
      end
    end
  #
  # END WEB SERVER (nginx)


# VAGRANT MESSAGE
  config.vm.post_up_message = "You're ready!"
end
```


### Folders 
```
/web (/var/web)
|
|---config/ (/etc/nginx/sites-available/)
```

### Specs
- Ram: 256MB
- CPU: 15%

### References
- Real Time Setup
- [How To](https://www.linode.com/docs/websites/nginx/how-to-configure-nginx)
- [Config](https://github.com/nginxinc/nginx-wiki/blob/master/source/start/topics/examples/full.rst)


	
