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

## Upload to production server

You need a running VPS at the provider of your choice. I prepared a server under salad.entwicklerbier.org.
Create a hosts file containing:
```
[salad]
salad.entwicklerbier.org
```

To run your playbook on this group, just specify the hosts file with the -i argument:
```
ansible-playbook -i hosts salad.yml
```

## Different configuration for development and production

Say we want to enable a more verbose debug log on the development machine.
We need to supply default value in our playbook - salad.yml
```
  vars:
    web_server:
      log_flags: ''
```

In our Vagrantfile add the log_flags to the Ansible config:
```
web_server: {
  log_flags: 'debug'
}
```

And alter our nginx-site to use the advanced log:
```
error_log /var/log/nginx/error.log {{ web_server.log_flags }};
```

We can now provision our vagrant machine and our host with these changes.

## Deploying SSL-Certificates

Adapt your nginx-config to load the ssl certs.

You should really not put your unencrypted key files in a public repo. So let's encrypt it:
```
openssl rsa -des -in salad.entwicklerbier.org.key -out salad.entwicklerbier.org.key
```

Add a vars_prompt to salad.yml which lets you query the passphrase from the previous step from the user:
```
vars_prompt:
  - name: ssl_passphrase
    prompt: "Enter SSL Certificate Passphrase"
    private: true
```

There are a couple of new tasks to copy the certificate chain/key in position.
To let the vars_prompt value decrypt your keyfile you need the {{ssl_passphrase}} variable
```
- name: strip ssl keys
  command: openssl rsa -in /etc/ssl/private/salad.entwicklerbier.secured.org.key -out /etc/ssl/private/salad.entwicklerbier.org.key -passin pass:{{ssl_passphrase}} creates=/etc/ssl/private/salad.entwicklerbier.org.key
  sudo: yes
  notify: restart nginx

```

Provision and ssl :)
