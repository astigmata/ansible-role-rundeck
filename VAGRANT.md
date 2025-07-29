Vagrantfile

```yaml
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.hostname = "rundeck"

  # Network configuration
  config.vm.network "private_network", ip: "192.168.56.10"
  config.vm.network "forwarded_port", guest: 4440, host: 4440, host_ip: "127.0.0.1"

  # VM resources - optimized for containers
  config.vm.provider "virtualbox" do |vb|
    vb.name = "rundeck"
    vb.memory = "2048"  # 12GB for containers
    vb.cpus = 2
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
    vb.customize ["modifyvm", :id, "--nestedpaging", "on"]
    vb.customize ["modifyvm", :id, "--largepages", "on"]
  end

  # Additional disk for Docker volumes
  config.vm.disk :disk, size: "80GB", name: "docker_storage"

  # Install Docker and Ansible before role installation
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y ca-certificates curl gnupg lsb-release python3-pip git

    # Install Ansible
    pip3 install ansible

    # Install requirements as vagrant user
    sudo -u vagrant ansible-galaxy install -r /vagrant/requirements.yml --force
    sudo -u vagrant ansible-galaxy collection install -r /vagrant/requirements.yml
  SHELL

  # Ansible provisioning
  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "deploy_rundeck.yml"
    ansible.verbose = "v"
    ansible.install = false  # Already installed via shell
    ansible.provisioning_path = "/vagrant"
    ansible.inventory_path = "inventory.yml"
    ansible.limit = "all"
  end
end
```

inventory.yml
```yaml
---
all:
  hosts:
    localhost:
      ansible_connection: local
      ansible_python_interpreter: /usr/bin/python3
```

deploy_rundeck.yml
```yaml
---
- hosts: all
  become: yes

  vars:
    rundeck_server_url: "http://192.168.56.10:4440"

  pre_tasks:
    - name: Wait for system to be ready
      wait_for_connection:
        timeout: 300

    - name: Update apt cache
      apt:
        update_cache: yes
      when: ansible_os_family == "Debian"

  roles:
    - astigmata.ansible-role-rundeck
```

requirements.yml
```yaml
---
# Ansible Collections
collections: []

# Ansible Roles (if needed)
roles:
  - name: astigmata.ansible-role-rundeck
    src: https://github.com/astigmata/ansible-role-rundeck.git
    version: feature/first_version

```