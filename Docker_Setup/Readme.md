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
```
Replace values with your actual:

IP addresses

SSH username

Path to your SSH private key

## Playbook Contents

Create `install_docker.yml` with the following content:
```bash
---
- name: Install Docker on multiple hosts
  hosts: docker_servers
  become: true
  tasks:
    - name: Update apt package index
      apt:
        update_cache: yes
      when: ansible_pkg_mgr == 'apt'

    - name: Install required system packages
      apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - software-properties-common
          - gnupg-agent
        state: present
      when: ansible_pkg_mgr == 'apt'

    - name: Add Docker GPG key
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: Add Docker repository
      apt_repository:
        repo: deb [arch=amd64] https://download.docker.com/linux/ubuntu {{ ansible_distribution_release }} stable
        state: present

    - name: Install Docker packages
      apt:
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io
        state: present
        update_cache: yes

    - name: Ensure Docker service is running
      service:
        name: docker
        state: started
        enabled: yes

    - name: Add current user to docker group
      user:
        name: "{{ ansible_user }}"
        groups: docker
        append: yes

    - name: Verify Docker installation
      command: docker --version
      register: docker_version
      changed_when: false

    - name: Display Docker version
      debug:
        msg: "Docker installed successfully: {{ docker_version.stdout }}"

```

## Execution

Run the playbook with:

```bash
ansible-playbook -i inventory.ini install_docker.yml
```
## Verification

Verify Docker is working on all nodes:

```bash
ansible docker_servers -i inventory.ini -m command -a "docker run hello-world" -b
```
## Notes

1.For CentOS/RHEL systems, replace apt tasks with equivalent yum tasks

2.Adjust firewall settings if needed for Docker networking

3.For proxy environments, add proxy configuration tasks

4.Check for the latest Docker Compose version before installation
