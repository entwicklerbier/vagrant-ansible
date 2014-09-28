vagrant-ansible
===============

A simple tutorial in using Vagrant and Ansible for development and production

## Requirements:

### Ansible
http://docs.ansible.com/intro_installation.html#installing-the-control-machine
```
brew install ansible
```

### Vagrant
https://docs.vagrantup.com/v2/installation/

## Vagrant

Init a new Vagrant machine with Ubuntu 14.04 64bit:
```
vagrant init ubuntu/trusty64
```

## Create first playbook
Create a file named salad.yml and add it to Vagrant:
```
  config.vm.provision "ansible" do |ansible|
    ansible.extra_vars = {
    }

    ansible.playbook = "salad.yml"
  end
```

## First boot of Vagrant machine
```
vagrant up
```
