# -*- mode: ruby -*-
# vi: set ft=ruby :


Vagrant.configure("2") do |config|

  config.vm.box = "bento/ubuntu-18.04"

  config.vm.network "private_network", ip: "192.168.33.10"



config.vm.provider "virtualbox" do |vb|

  vb.memory = "2048"
end
config.ssh.forward_agent = true

end