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

## Install nginx

Use apt to install nginx at a specific version
```
    - name: install packages
      apt: pkg={{ item }} state=latest
      sudo: yes
      with_items:
        - nginx=1.6.0
```

To provision our Vagrant machine (in this case install nginx in its default state) just type:
```
vagrant provision
```

To access the now running nginx on http://localhost:8080 we need to forward the clients port 80 to the hosts port 8080 in our Vagrantfile.
```
config.vm.network "forwarded_port", guest: 80, host: 8080

```
To reload the Vagrantfile type:
```
vagrant reload
```
