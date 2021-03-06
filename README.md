# Disinfo.quaidorsay.fr-ops

Recipes to setup infrastructure and deploy disinfo.quaidorsay.fr website and API

> Recettes pour mettre en place l'infrastructure et déployer le site web et l'API de disinfo.quaidorsay.fr

## Requirements

- Install [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
- Install required Ansible roles `ansible-galaxy install -r requirements.yml`

### [For developement only] Additionals dependencies

To test the changes without impacting the production server, a Vagrantfile is provided to test the changes locally in a virtual machine. VirtualBox and Vagrant are therefore required.

- Install [VirtualBox](https://www.vagrantup.com/docs/installation/)
- Install [Vagrant](https://www.vagrantup.com/docs/installation/)

## Configuration

### Allow decrypting all sensitive data.

A password is needed to decrypt encrypted files with [`ansible-vault`](https://docs.ansible.com/ansible/latest/user_guide/vault.html).
Get the password from the administrator and copy it in a `vault.key` file at the root of this project, it will avoid entering it every time you run a command.

### [For developement only] Configure your host machine to access the VM.

Edit your hosts file `/etc/hosts`, add the following line so you can connect to the VM to test deployed apps from your host machine's browser:
```
192.168.33.10    vagrant.local
```

Now on your browser you will be able to access deployed app on the VM with the URL `http://vagrant.local`

The server name `vagrant.local` can be changed in the file `/inventories/dev.yml`:
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
[…]
config.vm.network "private_network", ip: "192.168.33.10"
[…]
```

## Usage

To avoid making changes on the production server by mistake, by default all commands will only affect the vagrant developement VM. Note that the VM need to be started before with `vagrant up`.\
To execute commands on the production server you should specify it by adding the option `-i inventories/production.yml` to the following commands.

- **Setup a phoenix server:**

  Before all, following backup steps are required:
  - Create a dump of the Mattermost MySQL database on the remote server with `mysqldump -u root mattermost -p -r /tmp/dump.sql`.
  - Copy the resulting dump to the Mattermost role's `files` directory of this project on your local machine with: `scp -r -p cloud@disinfo.quaidorsay.fr:/tmp/dump.sql ./roles/infra/mattermost/files`.
  - Create a copy of Mattermost `data` files with `sudo cp -a /opt/mattermost/data/ /tmp/mattermost/data`
  - Changes Mattermost `data` files permissions `sudo chown cloud -R /tmp/mattermost/data`
  - Copy Mattermost `data` files to the Mattermost role's `files` directory of this project on your local machine with: `scp -r -p cloud@disinfo.quaidorsay.fr:/tmp/mattermost/data ./roles/infra/mattermost/files`.
  - Copy all scrapped political ads data to the new server directly (there is more than 500Go). Connect to `disinfo.quaidorsay.fr` server and run: `rsync -azP /mnt/data/political-ads-scraper/ cloud@sismo.quaidorsay.fr:/mnt/data/political-ads-scraper`.

```
ansible-playbook playbooks/site.yml
```

- **Setup infrastructure only:**
```
ansible-playbook playbooks/infra.yml
```

- **Setup apps only:**
```
ansible-playbook playbooks/apps.yml
```

- **Setup one app only:**
```
ansible-playbook playbooks/apps/<APP_NAME>.yml
```
_You can find all available apps in `playbooks/apps` directory._

For example, to setup only `media-scale`app on the new server:
```
ansible-playbook playbooks/apps/media-scale.yml
```

- **Setup one sub part of the infra:**
```
ansible-playbook playbooks/infra/<MODULE>.yml
```
_You can find all available modules in `playbooks/infra` directory._

For example, to setup only MongoDB on the new server:
```
ansible-playbook playbooks/infra/mongodb.yml
```

### Options and tags

#### Options

Ansible provide among many others the following useful options:
- `--diff`: to see what changed.
- `--check`: to simulate execution.
- `--check --diff`: to see what will be changed.

For example, if you modify the nginx config and you want to see what will be changed you can run:
```
ansible-playbook playbooks/infra/nginx.yml --check --diff
```

#### Tags

Some tags are available to refine what will happen, use them with `-t`:
 - `setup`: to only setup system dependencies required by the app(s) (cloning repo, installing app dependencies, all config files, and so on…)
 - `start`: to start the app(s)
 - `stop`: to stop the app(s)
 - `restart`: to restart the app(s)
 - `update`: to update the app(s) (pull code, install dependencies and restart app)

For example, you can update all apps by running:
```
ansible-playbook playbooks/apps.yml -t update
```

…or update only `media-scale`:
```
ansible-playbook playbooks/apps/media-scale.yml -t update
```

…or restart only `panoptes`:
```
ansible-playbook playbooks/apps/panoptes.yml -t restart
```

### Troubleshooting

If you have the following error:

```
Failed to connect to the host via ssh: Received disconnect from 127.0.0.1 port 2222:2: Too many authentication failures
```

Modify ansible ssh options to the `inventories/dev.yml` file like this:
```
all:
  children:
    dev:
      hosts:
        '127.0.0.1':
          […]
          ansible_ssh_private_key_file: .vagrant/machines/default/virtualbox/private_key
          ansible_ssh_extra_args: -o StrictHostKeyChecking=no -o IdentitiesOnly=yes
          […]
```
