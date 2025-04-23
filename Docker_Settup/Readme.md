# Ansible Playbook for Docker Installation on 3 VMs

This Ansible playbook automates the installation of Docker on three Ubuntu virtual machines.

## Prerequisites

- Ansible control node configured
- SSH access to three target VMs
- sudo privileges on target VMs
- Python 3 installed on all VMs

## Inventory Setup

Create an `inventory.ini` file with your VM details:

```ini
[docker_servers]
vm1 ansible_host=192.168.1.101
vm2 ansible_host=192.168.1.102
vm3 ansible_host=192.168.1.103

[docker_servers:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/ansible_key
ansible_python_interpreter=/usr/bin/python3
