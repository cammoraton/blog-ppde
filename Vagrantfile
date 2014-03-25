# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"
 
# Deprecated in 1.5
# Vagrant.require_plugin "vagrant-vbguest"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # Don't share the default synced folder
  config.vm.synced_folder "", "/vagrant", disabled: true

  # Replace with your puppet folder
  # config.vm.synced_folder "~/Workspace/puppet", "/puppet"

  # Common puppet manifests and modules (default.pp)
  config.vm.provision :puppet do |puppet|
    puppet.module_path    = "./puppet/modules"
    puppet.manifests_path = "./puppet/manifests"
  end

  config.vm.define "puppet" do |puppetmaster|
    puppetmaster.vm.box = "example-PuppetMaster"
    puppetmaster.vm.box_url = "./boxes/puppetmaster.box",
                              "./boxes/centos.box"
    
    puppetmaster.vm.hostname = "puppet.example.com"
    puppetmaster.vm.network "private_network", 
      ip: "203.0.113.2",
      virtualbox__intnet: "example"

    puppetmaster.vm.synced_folder "puppet/ssl/puppetmaster", 
      "/var/lib/puppet/ssl", 
      owner: "puppet", 
      group: "puppet"

    puppetmaster.vm.provision :puppet do |puppet|
      puppet.manifest_file  = "puppetmaster.pp"
      puppet.module_path    = "./puppet/modules"
      puppet.manifests_path = "./puppet/manifests"
    end
  end

  config.vm.define "centos" do |centos|
    centos.vm.box = "example-CentOS6.5"
    centos.vm.box_url = "./boxes/centos.box"

    centos.vm.hostname = "centos6.example.com"
    centos.vm.network "private_network", 
      ip: "203.0.113.3", 
      virtualbox__intnet: "example"

    centos.vm.synced_folder "./puppet/ssl/centos", 
      "/var/lib/puppet/ssl", 
      owner: "puppet", 
      group: "puppet"

    centos.vm.provision :puppet do |puppet|
      puppet.manifest_file  = "centos.pp"
      puppet.module_path    = "./puppet/modules"
      puppet.manifests_path = "./puppet/manifests"
    end
  end
 
  config.vm.define "ubuntu" do |ubuntu|
    ubuntu.vm.box = "example-Ubuntu12.04"
    ubuntu.vm.box_url = "boxes/ubuntu.box"

    ubuntu.vm.hostname = "ubuntu12.example.com"
    ubuntu.vm.network "private_network", 
      ip: "203.0.113.4", 
      virtualbox__intnet: "example"

    ubuntu.vm.synced_folder "./puppet/ssl/ubuntu", 
      "/var/lib/puppet/ssl", 
      owner: "puppet", 
      group: "puppet"

    ubuntu.vm.provision :puppet do |puppet|
      puppet.manifest_file  = "ubuntu.pp"
      puppet.module_path    = "./puppet/modules"
      puppet.manifests_path = "./puppet/manifests"
    end
  end
end
