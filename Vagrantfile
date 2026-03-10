# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.hostname = "rundeck"

  config.vm.network "private_network", ip: "192.168.56.10"
  config.vm.network "forwarded_port", guest: 4440, host: 4440, host_ip: "127.0.0.1"

  config.vm.provider "virtualbox" do |vb|
    vb.name = "rundeck-test"
    vb.memory = 2048
    vb.cpus = 2
  end

  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y python3-pip git
    pip3 install ansible
  SHELL

  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "tests/test.yml"
    ansible.install = false
    ansible.verbose = "v"
  end
end
