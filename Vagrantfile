# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "vStone/centos-7.x-puppet.3.x"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.cpus   = 2
  end

  config.vm.define "node", autostart: false do |node|
    node.vm.hostname = "vagrant-node.skyguide.ch"
    node.vm.network "private_network", ip: '10.0.1.10'
    node.vm.network "forwarded_port", guest: 8080, host: 9080, auto_correct: true
  end

  config.vm.provision :puppet do |puppet|
      puppet.options = '--verbose'
      puppet.manifest_file = 'infra.pp'
      puppet.module_path = 'modules'
      puppet.working_directory = "/tmp/vagrant-puppet"
  end

end
