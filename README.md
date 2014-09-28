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

## Create our own nginx site

Disable default nginx site in salad.yml:
```
    - name: disable nginx default site
      file: path=/etc/nginx/sites-available/default state=absent
      sudo: yes
      notify: restart nginx

```

Create index.html and nginx_site in templates

Add lines to salad.yml:
```

    - name: enable nginx salad site
      file: src=/etc/nginx/sites-available/salad dest=/etc/nginx/sites-enabled/salad owner={{ whoami.stdout }} group=www-data state=link
      sudo: yes
      notify: restart nginx

    - name: ensure /var/www/salad/ exists
      file: path=/var/www/salad state=directory recurse=yes owner={{ whoami.stdout }} group=www-data
      sudo: yes

    - name: add default index.html
      template: src=templates/index.html dest=/var/www/salad/index.html

  handlers:

    - name: restart nginx
      sudo: yes
      action: service name=nginx state=restarted enabled=yes

```

After provisioning we can visit our new nginx site on localhost:8080
```
vagrant provision
```
