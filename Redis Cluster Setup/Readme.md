# Redis Cluster Setup Project

## Overview
This project demonstrates how to set up a production-style Redis cluster across 3 nodes using both manual configuration and automated deployment with Ansible. The cluster provides high availability and data partitioning across multiple Redis instances.

## Project Structure

```
redis-cluster-setup/
├── manual-setup/               # Manual configuration instructions
│   ├── node1-setup.md
│   ├── node2-setup.md
│   └── node3-setup.md
├── ansible/                    # Ansible automation files
│   ├── inventory.ini
│   ├── playbook.yml
│   ├── roles/
│   │   └── redis-cluster/
│   │       ├── tasks/
│   │       │   └── main.yml
│   │       ├── templates/
│   │       │   └── redis.conf.j2
│   │       └── vars/
│   │           └── main.yml
│   └── requirements.yml
├── tests/                      # Test scripts
│   └── cluster-test.py
└── README.md                   # This file
```

## Prerequisites

- 3 Linux servers (Ubuntu 20.04/22.04 recommended)
- Redis 6.x or later
- Python 3.6+
- Ansible 2.9+ (for automated setup)
- SSH access between nodes

## Manual Setup Instructions

1. **Install Redis on all nodes**:
   ```bash
   sudo apt update
   sudo apt install -y redis-server
   ```

2. **Configure each Redis instance**:
   Edit `/etc/redis/redis.conf` on each node with cluster-specific settings:
   ```
   cluster-enabled yes
   cluster-config-file nodes.conf
   cluster-node-timeout 5000
   appendonly yes
   ```

3. **Create the Redis cluster**:
   On one of the nodes, run:
   ```bash
   redis-cli --cluster create node1:6379 node2:6379 node3:6379 \
     --cluster-replicas 1
   ```

4. **Verify cluster status**:
   ```bash
   redis-cli -c -h <any-node> cluster nodes
   ```

## Automated Setup with Ansible

1. **Install Ansible**:
   ```bash
   sudo apt install -y ansible
   ```

2. **Configure inventory**:
   Edit `ansible/inventory.ini` with your node IPs/hostnames.

3. **Run the playbook**:
   ```bash
   ansible-playbook -i inventory.ini playbook.yml
   ```

4. **Verify the cluster**:
   The playbook includes a verification step that checks cluster health.

## Testing the Cluster

Run the test script to verify data distribution and failover:
```bash
python3 tests/cluster-test.py
```

## Maintenance

- **Adding nodes**: Use `redis-cli --cluster add-node`
- **Resharding**: Use `redis-cli --cluster reshard`
- **Backup**: Use `redis-cli --cluster backup`

## Troubleshooting

- Check Redis logs: `/var/log/redis/redis-server.log`
- Verify cluster state: `redis-cli cluster info`
- Ensure ports 6379 and 16379 are open between nodes
