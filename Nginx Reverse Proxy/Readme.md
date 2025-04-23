
---

# 🌐 Ansible Playbook for Nginx Reverse Proxy with Let's Encrypt TLS

This Ansible project automates the deployment of an **Nginx reverse proxy** with **TLS encryption** via **Let's Encrypt (Certbot)** on an Ubuntu server. It's useful for securing web services behind a central reverse proxy.

## 📋 Prerequisites

- 🖥️ Ubuntu-based server (VM or physical)
- 🔧 Ansible control node
- 🔐 DNS A record pointing to your server's public IP
- 🌍 A valid domain name
- 📬 Email address for Let's Encrypt registration
- 🚪 Port 80 and 443 open on the firewall

## 🗂️ Inventory Setup

Create an `inventory.ini` file:

```ini
[nginx_proxy]
proxy ansible_host=your.server.ip

[nginx_proxy:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/ansible_key
ansible_python_interpreter=/usr/bin/python3
domain_name=yourdomain.com
certbot_email=you@example.com
```

## 📦 Playbook Overview

### `nginx_tls_proxy.yml`

```yaml
---
- name: Configure Nginx Reverse Proxy with TLS
  hosts: nginx_proxy
  become: true
  vars:
    web_backend: "http://localhost:3000"
  tasks:
    - name: Install required packages
      apt:
        name:
          - nginx
          - certbot
          - python3-certbot-nginx
        state: present
        update_cache: yes

    - name: Configure Nginx as reverse proxy
      template:
        src: nginx_proxy.j2
        dest: /etc/nginx/sites-available/{{ domain_name }}
      notify: Reload nginx

    - name: Enable Nginx config
      file:
        src: /etc/nginx/sites-available/{{ domain_name }}
        dest: /etc/nginx/sites-enabled/{{ domain_name }}
        state: link
        force: yes
      notify: Reload nginx

    - name: Remove default Nginx site
      file:
        path: /etc/nginx/sites-enabled/default
        state: absent
      notify: Reload nginx

    - name: Obtain Let's Encrypt certificate
      command: >
        certbot --nginx -d {{ domain_name }}
        --non-interactive --agree-tos
        --email {{ certbot_email }}
        --redirect
      args:
        creates: /etc/letsencrypt/live/{{ domain_name }}/fullchain.pem

  handlers:
    - name: Reload nginx
      service:
        name: nginx
        state: reloaded
```

---

## 🧩 `templates/nginx_proxy.j2`

```nginx
server {
    listen 80;
    server_name {{ domain_name }};

    location / {
        proxy_pass {{ web_backend }};
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

---

## ▶️ Execution

```bash
ansible-playbook -i inventory.ini nginx_tls_proxy.yml
```

---

## ✅ Verification

After the playbook runs:

- Visit `https://yourdomain.com` in a browser — it should:
  - Have a valid HTTPS certificate
  - Proxy traffic to your backend app (e.g., on port 3000)

---

## 📝 Notes

1. ⚠️ Certbot will **fail if DNS is not set up correctly** or ports 80/443 are blocked.  
2. 🔁 Certificates auto-renew via cron — no manual renewal needed.  
3. 📦 You can adapt this playbook for multiple domains or to use static HTML as a backend.  
4. 🛡️ Consider enabling rate limits or Web Application Firewall (WAF) in production.

---

