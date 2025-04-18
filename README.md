# MySQL InnoDB Cluster Deployment with Ansible

![MySQL Cluster](https://img.shields.io/badge/MySQL-Cluster-blue)
![Ansible](https://img.shields.io/badge/Ansible-Automation-green)

Automated deployment of a highly available MySQL InnoDB Cluster across three nodes using Ansible.

## 📌 Table of Contents

- [Features](#-features)
- [Architecture](#-architecture)
- [Prerequisites](#-prerequisites)
- [Quick Start](#-quick-start)
- [Detailed Usage](#-detailed-usage)
- [Customization](#-customization)
- [Verification](#-verification)
- [Troubleshooting](#-troubleshooting)
- [Contributing](#-contributing)
- [License](#-license)

## ✨ Features

- **Automated Cluster Setup**: Fully automated deployment of 3-node MySQL cluster
- **High Availability**: Automatic failover and recovery
- **Secure Configuration**: Built-in security hardening
- **Idempotent Operations**: Safe to run multiple times
- **Customizable**: Easily adjust configuration parameters

## 🏗 Architecture

```mermaid
graph TD
    A[Primary Node\nnode1:192.168.0.1] -->|Replication| B[Secondary Node\nnode2:192.168.0.2]
    A -->|Replication| C[Secondary Node\nnode3:192.168.0.3]
    B -->|Replication| C
