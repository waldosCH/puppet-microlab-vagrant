# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "vStone/centos-7.x-puppet.3.x"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.cpus   = 1
  end

  # comment these 2 lines as we don't want to have hiera "applied" for this step of the labo
  #config.vm.provision "shell", inline: "ln -fs /vagrant/modules/hiera/hiera.yaml /etc/puppet/hiera.yaml"
  #config.vm.provision "shell", inline: "ln -fs /vagrant/modules/hiera /etc/puppet/hiera"
  config.vm.provision "file", source: "puppet-facters/environment.rb", destination: "/tmp/environment.rb"
  config.vm.provision "shell", inline: "cp /tmp/environment.rb /usr/share/ruby/vendor_ruby/facter/environment.rb" 

  config.vm.define "node1", autostart: false do |node1|
    node1.vm.hostname = "vagrant-node-1"
    node1.vm.network "private_network", ip: '10.0.1.11'
    node1.vm.network "forwarded_port", guest: 8080, host: 9080, auto_correct: true
  end

  config.vm.define "node2", autostart: false do |node2|
    node2.vm.hostname = "vagrant-node-2"
    node2.vm.network "private_network", ip: '10.0.1.12'
    node2.vm.network "forwarded_port", guest: 8080, host: 9090, auto_correct: true
  end

  config.vm.provision :puppet do |puppet|
      puppet.options = '--verbose'
      puppet.manifest_file = 'infra.pp'
      puppet.module_path = 'modules'
      puppet.working_directory = "/tmp/vagrant-puppet"
  end

end
