
---

# 🐳 Ansible Playbook for Docker Swarm Cluster Setup (3 Nodes)

This Ansible playbook automates the setup of a Docker Swarm cluster on three Ubuntu virtual machines. One node acts as the **manager**, and two as **workers**.

## 📋 Prerequisites

- ✅ Docker installed on all VMs 
- 🐧 Ubuntu-based VMs (3 total)
- 🔐 SSH access to all VMs from the Ansible control node
- 🔧 Sudo privileges on target nodes
- 🐍 Python 3 on all hosts

## 🗂️ Inventory Setup

Create an `inventory.ini` file like this:

```ini
[manager]
manager1 ansible_host=192.168.1.101

[workers]
worker1 ansible_host=192.168.1.102
worker2 ansible_host=192.168.1.103

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/ansible_key
ansible_python_interpreter=/usr/bin/python3
```

## 📦 Playbook Structure

You will need two files:

- `swarm_init.yml`: initializes the manager and captures join token
- `swarm_join.yml`: joins the workers using the token

---

## 🚀 `swarm_init.yml`

```yaml
---
- name: Initialize Docker Swarm on Manager
  hosts: manager
  become: true
  tasks:
    - name: Get manager IP address
      command: hostname -I
      register: manager_ip_raw

    - name: Set manager IP as fact
      set_fact:
        manager_ip: "{{ manager_ip_raw.stdout.split()[0] }}"

    - name: Initialize Docker Swarm
      shell: |
        docker swarm init --advertise-addr {{ manager_ip }}
      register: swarm_init_output
      failed_when: "'Error response from daemon: This node is already part of a swarm' not in swarm_init_output.stderr"

    - name: Get join-token for workers
      shell: docker swarm join-token worker -q
      register: swarm_worker_token

    - name: Set worker token as fact
      set_fact:
        worker_join_token: "{{ swarm_worker_token.stdout }}"

    - name: Save swarm variables
      add_host:
        name: swarm_info
        groups: swarm_token_holder
        worker_token: "{{ worker_join_token }}"
        manager_ip: "{{ manager_ip }}"
```

---

## 🔗 `swarm_join.yml`

```yaml
---
- name: Join workers to Docker Swarm
  hosts: workers
  become: true
  vars:
    manager_ip: "{{ hostvars['swarm_info'].manager_ip }}"
    worker_token: "{{ hostvars['swarm_info'].worker_token }}"
  tasks:
    - name: Join Swarm as worker
      shell: |
        docker swarm join --token {{ worker_token }} {{ manager_ip }}:2377
      register: swarm_join_output
      failed_when: "'This node is already part of a swarm' not in swarm_join_output.stderr"
```

---

## ▶️ Execution

Run the playbooks in sequence:

```bash
ansible-playbook -i inventory.ini swarm_init.yml
ansible-playbook -i inventory.ini swarm_join.yml
```

---

## ✅ Verification

Verify that all nodes have joined the swarm:

```bash
ansible -i inventory.ini manager -a "docker node ls" -b
```

Expected output on the manager:

```
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
xxxxxxxxxxxx        manager1           Ready               Active              Leader              20.10.xx
xxxxxxxxxxxx        worker1            Ready               Active                                  20.10.xx
xxxxxxxxxxxx        worker2            Ready               Active                                  20.10.xx
```

---

## 📝 Notes

1. ⚠️ This assumes Docker is already installed — use an install playbook first if needed.  
2. 🌍 The manager must be accessible at the advertised IP from the workers.  
3. 🔁 Re-run `swarm_init.yml` carefully — it may fail if the node is already part of a swarm.  
4. 🔒 For production setups, secure the cluster with TLS and firewall rules.

---

