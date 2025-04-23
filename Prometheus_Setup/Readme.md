---

# 📊 Ansible Playbook for Installing Prometheus + Thanos on 3 Ubuntu VMs

This project automates the installation and configuration of **Prometheus** and **Thanos** on 3 Ubuntu-based virtual machines. It sets up Prometheus nodes with Thanos sidecars, and optionally a Thanos Store and Query component.

---

## 🧱 Architecture

![ChatGPT Image Apr 23, 2025, 12_40_47 PM](https://github.com/user-attachments/assets/af0df287-1494-45c2-ab01-1ae5c277cb74)

---

## 📋 Prerequisites

- 3 Ubuntu 22.04 VMs with SSH access
- Ansible control node
- Public or private S3-compatible object storage (MinIO or AWS S3)
- Domain or static IPs for VM access
- Port 9090 (Prometheus), 10902 (Thanos gRPC), 10903 (Thanos HTTP) open

---

## 🗂️ Inventory Setup

### `inventory.ini`

```ini
[prometheus_nodes]
vm1 ansible_host=192.168.1.101
vm2 ansible_host=192.168.1.102
vm3 ansible_host=192.168.1.103

[prometheus_nodes:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/ansible_key
ansible_python_interpreter=/usr/bin/python3
```

---

## 📦 Playbook: `install_prometheus_thanos.yml`

```yaml
---
- name: Install Prometheus and Thanos Sidecar
  hosts: prometheus_nodes
  become: true
  vars:
    prometheus_version: "2.52.0"
    thanos_version: "0.34.1"
    prometheus_dir: /opt/prometheus
    thanos_dir: /opt/thanos
    s3_bucket: "thanos-bucket"
    s3_endpoint: "http://minio.local:9000"
    s3_access_key: "minioadmin"
    s3_secret_key: "minioadmin"
  tasks:
    - name: Create directories
      file:
        path: "{{ item }}"
        state: directory
        mode: '0755'
      loop:
        - "{{ prometheus_dir }}"
        - "{{ thanos_dir }}"
        - "/etc/prometheus"
        - "/var/lib/prometheus"

    - name: Download and install Prometheus
      unarchive:
        src: "https://github.com/prometheus/prometheus/releases/download/v{{ prometheus_version }}/prometheus-{{ prometheus_version }}.linux-amd64.tar.gz"
        dest: "{{ prometheus_dir }}"
        remote_src: yes
        extra_opts: [--strip-components=1]

    - name: Download and install Thanos
      unarchive:
        src: "https://github.com/thanos-io/thanos/releases/download/v{{ thanos_version }}/thanos-{{ thanos_version }}.linux-amd64.tar.gz"
        dest: "{{ thanos_dir }}"
        remote_src: yes
        extra_opts: [--strip-components=1]

    - name: Copy Prometheus config
      template:
        src: templates/prometheus.yml.j2
        dest: /etc/prometheus/prometheus.yml
        mode: '0644'

    - name: Setup systemd service for Prometheus
      template:
        src: templates/prometheus.service.j2
        dest: /etc/systemd/system/prometheus.service

    - name: Setup systemd service for Thanos Sidecar
      template:
        src: templates/thanos_sidecar.service.j2
        dest: /etc/systemd/system/thanos-sidecar.service

    - name: Reload and start services
      systemd:
        daemon_reload: yes
        name: "{{ item }}"
        state: started
        enabled: yes
      loop:
        - prometheus
        - thanos-sidecar
```

---

## 📄 Template Files

### `templates/prometheus.yml.j2`

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
```

---

### `templates/prometheus.service.j2`

```ini
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
ExecStart={{ prometheus_dir }}/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.listen-address=:9090 \
  --web.enable-lifecycle

Restart=on-failure

[Install]
WantedBy=multi-user.target
```

---

### `templates/thanos_sidecar.service.j2`

```ini
[Unit]
Description=Thanos Sidecar
After=network-online.target

[Service]
ExecStart={{ thanos_dir }}/thanos sidecar \
  --tsdb.path /var/lib/prometheus \
  --prometheus.url http://localhost:9090 \
  --objstore.config="{{ lookup('file', 'thanos-s3-config.yaml') | replace('\n', ' ') }}" \
  --http-address 0.0.0.0:10903 \
  --grpc-address 0.0.0.0:10902

Restart=always

[Install]
WantedBy=multi-user.target
```

---

### 📁 `thanos-s3-config.yaml` (keep in `files/` directory)

```yaml
type: s3
config:
  bucket: "thanos-bucket"
  endpoint: "minio.local:9000"
  access_key: "minioadmin"
  secret_key: "minioadmin"
  insecure: true
```

---

## ▶️ Run the Playbook

```bash
ansible-playbook -i inventory.ini install_prometheus_thanos.yml
```

---

## ✅ What You Get

- Prometheus running on all 3 nodes
- Thanos sidecar on each node pushing data to object storage
- Ready to connect a Thanos **Query** or **Store** component

---

