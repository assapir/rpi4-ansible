# rpi4-ansible
[![GitHub license](https://img.shields.io/github/license/assapir/rpi4-ansible)](https://github.com/assapir/rpi4-ansible/blob/master/LICENSE)
## **Forked from https://github.com/pnavais/rpi4-ansible**


### Ansible config &amp; playbooks for Raspberry Pi 4 K3S cluster

## Rationale
This project contains Ansible playbooks allowing to provision from the ground up a K3S cluster of Raspberry Pi 4 based on ArchLinux ARM with both 64-bit Kernel and userland (see [rpi4-arch](https://github.com/assapir/rpi4-arch) for OS installation).

## How to use it

### Configure the cluster
Modify the file inventory hosts file `inventory/hosts.ini` with the configuration (ip addresses/hosts) to suit your needs.

The default configuration is based on 2 workers:
```ini
[k3s_rpi:children]
k3s_rpi_master
k3s_rpi_workers

[k3s_rpi_master]
master-pi ansible_host=192.168.0.101

[k3s_rpi_workers]
worker-01 ansible_host=192.168.0.102
worker-02 ansible_host=192.168.0.103
```

To check everything is working fine execute:

```shell
ansible -m ping k3s_rpi
```

Which should acknowledge something similar to:
```shell
3s-master | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

Modify the hosts configuration variables file `inventory/group_vars/k3s_rpi.yml` to change default user information (`dev_user`) and custom configuration (e.g. Timezone, preferred static IP addresses, ...).

### Pre-requirements
Provided a sane environment with Python3/pip/Ansible already in place, install external roles from galaxy with :

```bash
> ansible-galaxy install -r requirements.yml
```

### Execute the playbook
The following command with launch the provisioning playbooks :

```bash
> ansible-playbook playbooks/main.yml --ask-become-pass
```

which will execute in order:
1. prepare-system: Performs basic ArchLinux customizations (NTP, GPIO/I2C rules, Yay...) and system packages installation (avahi, docker, ...)
2. setup-packages: Installs auxiliary system packages (Pacman & AUR) and Python modules.
3. setup-users: Create users & groups
4. network: Setup network settings (hostname, static IP, MDNS, ...)
5. k3s-setup: Do all the hard work

After a couple of minutes, we should have a working Kubernetes cluster up and running!

## Auxiliary utilities

### Shutting down the cluster
Execute the following playbook :

```bash
> ansible-playbook playbooks/shutdown.yml
```

The shutdown process will start with worker nodes and end up with the master host. This allows to visually display some info in the LCD display and LED before shutdown.

### Uninstalling K3S from the cluster
Execute the following playbook :

```bash
> ansible-playbook playbooks/k3s-reset.yml
```

This stops K3S services and remove dependencies from all hosts.
