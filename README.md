# 🚀 Production-Grade HPC Cluster: Automated Infrastructure & Security

This project demonstrates the ground-up deployment of a scalable **High-Performance Computing (HPC)** environment. It transforms a standard Linux environment into a high-performance infrastructure featuring automated configuration management, centralized authentication, and a robust observability stack.

---

## 🏗️ Technical Architecture
The cluster is divided into three functional layers to ensure a professional separation of concerns:

* **Management Layer (Master Node):** Hosts the NFS Server for shared storage, the LDAP Directory for centralized users, and the Slurm Control Daemon (`slurmctld`).
* **Infrastructure & Gateway Layer (Login Node):** Acts as the cluster's "Internal ISP." Manages DNS, DHCP, and NTP for the private subnet (`192.168.100.x`) and provides secure external access via **WireGuard VPN**.
* **Execution Layer (Compute Nodes):** High-performance nodes (8 Cores / 16 Threads each) dedicated to running workloads assigned by the Slurm scheduler.



---

## 🛠️ Technical Stack
* **Cluster Management:** Slurm (Workload Manager), Munge (Authentication)
* **Networking:** WireGuard (VPN), ISC-DHCP-Server, BIND9 (DNS)
* **Automation:** Ansible (Configuration Management)
* **Storage:** NFSv4 with systemd-automount
* **Security & SIEM:** **Wazuh** (Endpoint Security), **Suricata** (IDS)
* **Monitoring:** **Zabbix** (Infrastructure Health), Prometheus, Grafana, Loki

---

## ⚙️ Key Implementation Details

### 1. The "Nervous System" (Networking & Core Services)
* **DHCP & DNS:** Static IP leases based on MAC addresses ensure consistency. A local DNS zone (`cluster.local`) allows seamless hostname resolution.
* **NTP Stratum Server:** The Login Node acts as the time source to prevent "Time Drift," critical for Slurm stability and LDAP authentication.

### 2. Automated Configuration (Ansible)
* **Slurm Deployment:** Automated installation of `slurmctld` and `slurmd` across the cluster.
* **LDAP Integration:** Playbooks automate client-side configuration for Single Sign-On (SSO).

### 3. Storage Networking (NFS Tuning)
* **Stateless Mounting:** Resolved "90-second boot hangs" using NFSv4 and the `nolock` flag.
* **Systemd Automounting:** Implemented `x-systemd.automount` to decouple storage from the networking boot sequence.

---

## 🛡️ Security & Observability

### Endpoint Security & IDS
* **Wazuh SIEM:** Deployed Wazuh agents across all nodes for file integrity monitoring (FIM), rootkit detection, and centralized log analysis.
* **Suricata IDS:** Configured to monitor physical interfaces and the `wg0` (WireGuard) tunnel with custom rules for reconnaissance detection.

### The Monitoring Pipeline
* **Zabbix:** Utilized for proactive infrastructure monitoring, tracking hardware health (CPU temp, disk SMART status), and service availability across the cluster.
* **Prometheus & Grafana:** Scrapes `node_exporter` for real-time performance metrics during HPL benchmarks.
* **Loki:** Aggregates `eve.json` logs from Suricata for security forensics.
* **Telegram Integration:** Alertmanager sends instant notifications for high CPU usage or security breaches.



---

## 📊 Benchmarking (HPL)
The cluster performance was validated using the **High-Performance Linpack (HPL)** benchmark.

* **Process Grid:** Optimized $2 \times 16$ grid ($P \times Q$) to utilize 32 total threads across compute nodes.
* **Job Submission:** Managed via `sbatch` scripts to ensure job persistence and automated logging.

---

## 🔍 Challenges Overcome
* **The "Silent Drop" Issue:** Fixed a visibility gap where Suricata ignored WireGuard traffic by using `ethtool` to disable hardware checksum offloading on the `wg0` interface.
* **Network-Storage Deadlocks:** Fixed critical boot failures where the OS would hang waiting for NFS mounts by implementing lazy mounting.
* **MPI Integration:** Resolved Open MPI "PMI Support" errors by properly configuring the environment within Slurm allocations.

---

## 📂 Repository Structure
```plaintext
├── ansible/            # LDAP setup and Slurm deployment playbooks
├── configs/            # DHCPd, BIND9, and WireGuard configurations
├── monitoring/         # Zabbix templates, Prometheus rules, Grafana JSONs
├── slurm/              # slurm.conf and sbatch submission scripts
├── suricata/           # Custom local.rules and suricata.yaml
└── benchmarks/         # HPL.dat configuration and performance results
