# Disinfo.quaidorsay.fr-ops

Recipes to setup infrastructure and deploy disinfo.quaidorsay.fr website and API

> Recettes pour mettre en place l'infrastructure et déployer le site web et l'API de disinfo.quaidorsay.fr

## Requirements

- Install [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
- Install required Ansible roles `ansible-galaxy install -r requirements.yml`

### [For developement only] Additionals dependencies

To test the changes without impacting the actual server, a Vagrantfile is provided to test the changes locally in a virtual machine. VirtualBox and Vagrant are therefore required.

- Install [VirtualBox](https://www.vagrantup.com/docs/installation/)
- Install [Vagrant](https://www.vagrantup.com/docs/installation/)

## Configuration

### Allow decrypting all sensitive data.

A password is needed to decrypt encrypted files with ansible-vault.
Get the password from the administrator and copy it in a `vault.key` file at the root of this project, it will avoid entering it every time you run a command.

### [For developement only] Configure your host machine to access the VM.

Edit your hosts file `/etc/hosts`, add the following line so you can connect to the VM to test deployed apps from your host machine's browser:
```
192.168.33.10    vagrant.local
```

Now on your browser you can access the VM with http://vagrant.local

The server name (`vagrant.local`) can be changed in the file `/inventories/dev.yml`:.
```
[…]
    dev:
      hosts:
        '127.0.0.1':
          […]
          base_url: <SERVER_NAME>
```

The guest VM's IP can be changed in the `VagrantFile`:
```
# Create a private network, which allows host-only access to the machine
# using a specific IP.
config.vm.network "private_network", ip: "192.168.33.10"
```

## Usage

_Note: To avoid making changes on the production server by mistake, all commands will be run on the vagrant dev VM, which need to be started before with `vagrant up`. To execute command on the production server you should specify it by adding the option `-i inventories/production.yml`_

- To setup a phoenix server:
```
ansible-playbook site.yml
```

- To setup infrastructure only:
```
ansible-playbook infra.yml
```

- To setup apps only:
```
ansible-playbook apps.yml
```

By adding the following options you can:
- see what changed with `--diff`
- simulate execution with `--check`
- see what will be changed with `--check --diff`

There are some availables tags to finetune what will happen:
 - `setup`: to only setup system dependencies required by the app(s) (cloning repo, installing app dependencies, all config files, and so on…)
 - `start`: to start the app(s)
 - `stop`: to stop the app(s)
 - `restart`: to restart the app(s)
 - `update`: to update the app(s) (pull code, install dependencies and restart app)

For example you can update all apps by running:
```
ansible-playbook apps.yml -t update
```

There are also a tag available for each app to allow working only on one (or multiple but not all) app(s).

For example you can update only `media-scale` and `panoptes` apps by running:
```
ansible-playbook apps.yml -t media-scale,panoptes,update
```
- - -
