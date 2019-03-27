sa-packer-helper
================

[![Build Status](https://travis-ci.com/softasap/sa-packer-helper.svg?branch=master)](https://travis-ci.com/softasap/sa-packer-helper)

Original credits to  https://github.com/geerlingguy/ansible-role-packer-debian/

## Requirements

Prior to running this role via Packer, you need to make sure Ansible is installed via a shell provisioner, and that preliminary VM configuration (like adding a vagrant user to the appropriate group and the sudoers file) is complete, generally by using a Kickstart installation file (e.g. `ks.cfg`) or [preseeding](https://help.ubuntu.com/lts/installation-guide/s390x/apbs02.html) with Packer. An example array of provisioners for your Packer .json template looks something like:

```json
"provisioners": [
  {
    "type": "shell",
    "execute_command": "echo 'vagrant' | {{.Vars}} sudo -S -E bash '{{.Path}}'",
    "script": "scripts/ansible.sh"
  },
  {
    "type": "ansible-local",
    "playbook_file": "ansible/main.yml",
    "role_paths": [
      "VMs/roles",
    ]
  }
],
```

The files should contain, at a minimum:

**scripts/ansible.sh**:

An example for Ubuntu 16.04

```bash
#!/bin/bash -eux
# Install Ansible repository and Ansible.
apt -y install software-properties-common
apt-add-repository ppa:ansible/ansible
apt-get update
apt-get install ansible
```

An example for Debian 8.8

```bash
#!/bin/bash -eux
# Install Ansible repository and Ansible.
apt -y install software-properties-common
echo "deb http://ppa.launchpad.net/ansible/ansible/ubuntu trusty main" | tee -a /etc/apt/sources.list
apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 93C4A3FD7BB9C367
apt -y update
apt -y install ansible
```

**ansible/main.yml**:

```yaml
---
- hosts: all
  sudo: yes
  gather_facts: yes
  roles:
    - geerlingguy.packer-debian
```

You might also want to add another shell provisioner to run cleanup, erasing free space using `dd`, but this is not required (it will just save a little disk space in the Packer-produced .box file).
