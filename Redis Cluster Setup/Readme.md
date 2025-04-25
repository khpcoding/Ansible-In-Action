# Redis Cluster Setup 🔴

This project simulates a **production-style Redis Cluster** running across three nodes. It demonstrates how to set up and manage a distributed Redis environment, both **manually** and using **Ansible automation**.

Redis Cluster provides high availability and horizontal scalability by partitioning data across multiple nodes with replication for fault tolerance.

---

## 📊 Diagrams

### 🖧 Cluster Network Architecture

This shows how the 3 Redis nodes communicate and form the cluster:

![Redis Cluster Network Architecture](images/redis-cluster-network-architecture.png)

- Redis nodes communicate over ports `6379` and `16379`
- Cluster setup uses gossip protocol for node discovery
- `redis-cli` client can connect to any node and gets redirected appropriately

---

### 🧩 Data Partitioning and Slot Distribution

Redis Cluster uses **hash slots** to distribute data. There are 16,384 slots split among master nodes:

![Redis Hash Slot Sharding](images/redis-hashslot-sharding.png)

- Node 1 handles hash slots 0–5460
- Node 2 handles 5461–10922
- Node 3 handles 10923–16383
- Writes/reads are directed to the correct node automatically

---

## ⚙️ Project Structure
---

## 🛠️ Prerequisites

- 3 VMs or containers with Redis installed
- Passwordless SSH (for Ansible)
- Open ports: `6379`, `16379`

---

## 🚀 1. Manual Setup

> For learning Redis cluster mechanics.

1. Install Redis
2. Configure `redis.conf` with cluster options
3. Start Redis on all nodes
4. Create the cluster using:
   ```bash
   redis-cli --cluster create \
     node1:6379 node2:6379 node3:6379 \
     --cluster-replicas 0
