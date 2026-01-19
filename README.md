# 🦅 Secure FTP Server Deployment (vsftpd + Ansible)

![Project Status](https://img.shields.io/badge/status-active-success.svg)
![Ansible](https://img.shields.io/badge/ansible-%231A1918.svg?style=flat&logo=ansible&logoColor=white)
![Vagrant](https://img.shields.io/badge/vagrant-%231589F0.svg?style=flat&logo=vagrant&logoColor=white)
![Debian](https://img.shields.io/badge/Debian-D70A53?style=flat&logo=debian&logoColor=white)

Comprehensive Infrastructure as Code (IaC) project to deploy a secure, high-performance FTP server using **vsftpd** (Very Secure FTP Daemon), automated with **Ansible** and virtualized with **Vagrant**.

---

## 📋 Table of Contents
- [Project Overview](#-project-overview)
- [Architecture](#-architecture)
- [Features](#-features)
- [Prerequisites](#-prerequisites)
- [Installation & Usage](#-installation--usage)
- [Configuration Details](#-configuration-details)
- [Testing & Validation](#-testing--validation)
- [Authors](#--authors)

---

## 📖 Project Overview

This project simulates a real-world enterprise environment where a secure file transfer service is required. It uses **Vagrant** to provision the infrastructure and **Ansible** to configure the services, ensuring a reproducible and "zero-touch" deployment.

The main objective is to provide an FTP service that supports:
- **Anonymous Access:** For public files (Read-Only).
- **Authenticated Access:** For internal users (Read/Write).
- **Security:** Mandatory SSL/TLS encryption.
- **QoS:** Bandwidth shaping to prioritize traffic.

---

## 🏗 Architecture

The infrastructure consists of two virtual machines connected via a generic private network (`192.168.56.0/24`) and bridged to the physical network for internet access.

| Role | Hostname | IP Address | OS | Description |
|------|----------|------------|----|-------------|
| **Server** | `mirror.sistema.sol` | `192.168.56.10` | Debian 12 | Hosting `vsftpd` and `bind9`. |
| **Client** | `client` | `192.168.56.11` | Debian 12 | Testing node with `lftp`, `nmap`. |

**Network Topology:**
- **Public Network:** Bridged (DHCP) for package installation and external connectivity.
- **Private Network:** Static IPs for internal communication between Client and Server.

---

## ✨ Features

### 1. 🔒 Security & Encryption
- **SSL/TLS Enforcement:** All authenticated connections MUST use SSL (`explicit ftps`). Unencrypted login attempts are rejected for local users.
- **Certificates:** Self-signed 2048-bit RSA certificate automatically generated during provisioning.

### 2. 👤 User Management & Jails (Chroot)
- **Luis (`luis`):** Authenticated user. **Chrooted** (Jailed) to home directory `/home/luis`. Cannot browse the entire filesystem.
- **Maria (`maria`):** Authenticated user. **Non-Chrooted** (Exception list). Can browse the system (permissions permitting).
- **Anonymous:** Restricted to `/srv/ftp`. No upload permissions.

### 3. 🚦 Quality of Service (Bandwidth Limits)
- **Anonymous Users:** Capped at **2 MB/s** (`anon_max_rate=2097152`).
- **Local Users:** Capped at **5 MB/s** (`local_max_rate=5242880`).

### 4. 🌐 DNS Automation
- The server acts as a primary DNS for `sistema.sol`.
- The client receives automatic DNS configuration to resolve `ftp.sistema.sol` to `192.168.56.10`.

---

## 🛠 Prerequisites

Ensure you have the following installed on your host machine:

- **Vagrant** (v2.2+)
- **VirtualBox** (v6.1+)
- **Ansible** (Usually installed on the local machine or provisioned within the VM, in this project `ansible_local` is used, so Ansible runs *inside* the server VM).

---

## 🚀 Installation & Usage

1.  **Clone the Repository**
    ```bash
    git clone https://github.com/dexplop2602/FTP-Anonymous-y-FTP-Seguro.git
    cd FTP-Anonymous-y-FTP-Seguro
    ```

2.  **Launch Infrastructure**
    This command downloads the Debian box, spins up the VMs, and triggers the Ansible playbook.
    ```bash
    vagrant up
    ```

3.  **Access the Client**
    To perform tests, SSH into the client machine:
    ```bash
    vagrant ssh client
    ```

---

## ⚙ Configuration Details

### Ansible Playbook (`ansible/playbook.yml`)
The core automation logic. Key tasks include:
- Installing `vsftpd`, `openssl`, `bind9`.
- Generating SSL certificates (`/etc/ssl/private/vsftpd.key`).
- Creating users (passwords set to username).
- Configuring `/etc/vsftpd.conf` and DNS zones.

### VSFTPD Configuration (`vsftpd.conf`)
Located in `ansible/files/vsftpd.conf`.
- `ssl_enable=YES`: Activates SSL.
- `chroot_local_user=YES`: Jails users by default.
- `chroot_list_file=/etc/vsftpd.chroot_list`: Defines exceptions (Maria).

---

## 🧪 Testing & Validation

### 1. DNS Resolution
The client automatically resolves the domain.
```bash
ping -c 2 ftp.sistema.sol
# Reply from 192.168.56.10...
```

### 2. Secure Connectivity (FTPS)
Connecting as user **Luis** (Password: `luis`). Note that typical FTP clients might fail without SSL configuration. Use `lftp`.

```bash
lftp -u luis,luis -e 'set ssl:verify-certificate no' ftp.sistema.sol
```
![Login Luis](Capturas/15%20-%20Captura%20Luis.png)

### 3. Chroot Verification
- **Luis** is locked in `/home/luis`.
- **Maria** can wander.

![Login Maria](Capturas/15%20-%20Captura%20Maria.png)

### 4. Bandwidth Limiting
Downloading a 100MB test file generated by the playbook.

**Anonymous (Limited to ~2MB/s):**
```bash
lftp -e 'pget /test_100MB.dat; exit' ftp.sistema.sol
```
![Speed Anon](Capturas/17%20-%20Captura%20Velocidad%20An%C3%B3nima.png)

**Authenticated (Limited to ~5MB/s):**
```bash
lftp -u luis,luis -e 'set ssl:verify-certificate no; pget /home/luis/test_100MB.dat; exit' ftp.sistema.sol
```
![Speed Luis](Capturas/17%20-%20Captura%20Velocidad%20Luis.png)

### 5. Wireshark Evidence
Packets are encrypted, preventing credential sniffing.
![Encryption](Capturas/16%20-%20Captura.png)

---

## 👥 Authors

**David Expósito López** & **Ángela Sánchez Robles**
*System Administration Students*