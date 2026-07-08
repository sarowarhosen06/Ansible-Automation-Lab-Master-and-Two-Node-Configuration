# Ansible Setup Guide (1 Master + 2 Managed Nodes)

This guide explains how to install and configure **Ansible** using:

* **1 Master Node** (Ansible Control Node)
* **2 Managed Nodes** (Target Servers)

## Network Configuration

| Host    | IP Address   | Role                 |
| ------- | ------------ | -------------------- |
| Master  | `172.17.0.1` | Ansible Control Node |
| Server1 | `172.17.0.2` | Managed Node         |
| Server2 | `172.17.0.3` | Managed Node         |

---

# Step 1: Update All Servers

Run the following command on **all machines** (Master, Server1, and Server2):

```bash
sudo apt update && sudo apt upgrade -y
```

---

# Step 2: Install OpenSSH Server

Install the OpenSSH server on **all machines**.

```bash
sudo apt install openssh-server -y
```

Enable and start the SSH service.

```bash
sudo systemctl enable ssh
sudo systemctl start ssh
```

Verify that SSH is running.

```bash
sudo systemctl status ssh
```

---

# Step 3: Install Ansible

Run the following commands **only on the Master Node**.

```bash
sudo apt install ansible -y
```

Verify the installation.

```bash
ansible --version
```

---

# Step 4: Configure Passwordless SSH

Ansible communicates with managed nodes through SSH. Configure passwordless authentication using one of the following methods.

## Method 1: Using `ssh-copy-id` (Recommended)

### Generate an SSH Key

On the **Master Node**, generate a new SSH key pair.

```bash
ssh-keygen
```

Press **Enter** to accept the default file location and leave the passphrase empty if you want fully automated access.

### Copy the Public Key

Copy the SSH public key to each managed node.

```bash
ssh-copy-id root@172.17.0.2
```

```bash
ssh-copy-id root@172.17.0.3
```

### Verify the Connection

```bash
ssh root@172.17.0.2
```

```bash
ssh root@172.17.0.3
```

If you can log in without entering the root password, passwordless SSH has been configured successfully.

---

## Method 2: Manual SSH Key Configuration

If `ssh-copy-id` is unavailable, configure SSH manually.

### Generate an SSH Key

On the **Master Node**, run:

```bash
ssh-keygen -t rsa -b 4096
```

Accept the default location:

```text
~/.ssh/id_rsa
```

Leave the passphrase empty by pressing **Enter** twice if passwordless automation is desired.

### Display the Public Key

```bash
cat ~/.ssh/id_rsa.pub
```

Copy the entire output.

### Configure Each Managed Node

On **Server1** and **Server2**, create the `.ssh` directory if it does not exist.

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

Open the `authorized_keys` file.

```bash
nano ~/.ssh/authorized_keys
```

Paste the copied public key into the file, then save and exit.

Set the correct permissions.

```bash
chmod 600 ~/.ssh/authorized_keys
```

### Verify Passwordless SSH

From the Master Node, test the connection.

```bash
ssh root@172.17.0.2
```

```bash
ssh root@172.17.0.3
```

If the connection opens without prompting for a password, the configuration is complete.

---

# Step 5: Create the Inventory File

Create a new inventory file.

```bash
nano inventory.ini
```

Add the following configuration.

```ini
# inventory.ini

[webservers]
server1 ansible_host=172.17.0.2 ansible_user=root
server2 ansible_host=172.17.0.3 ansible_user=root

[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

Save and close the file.

---

# Step 6: Test Connectivity

Verify that Ansible can communicate with all managed nodes.

```bash
ansible all -i inventory.ini -m ping
```

Expected output:

```text
server1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

server2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

---

# Step 7: Run Ad-Hoc Commands

### Check System Uptime

```bash
ansible all -i inventory.ini -m command -a "uptime"
```

### Check Disk Usage

```bash
ansible all -i inventory.ini -m shell -a "df -h"
```

### Update Package Cache

```bash
ansible all -i inventory.ini -m apt -a "update_cache=yes" --become
```

---

# Project Directory Structure

```text
ansible-project/
│
├── inventory.ini
├── playbook.yml
└── README.md
```

---

# Useful Commands

### Show Ansible Version

```bash
ansible --version
```

### List Inventory Hosts

```bash
ansible-inventory -i inventory.ini --list
```

### Ping All Managed Nodes

```bash
ansible all -i inventory.ini -m ping
```

### Run a Command on All Hosts

```bash
ansible all -i inventory.ini -a "hostname"
```

---

# Requirements

* Ubuntu/Debian Linux
* Python 3
* OpenSSH Server
* Ansible
* Passwordless SSH authentication

---

# Summary

After completing this guide, you will have:

* One Ansible Control Node
* Two Managed Nodes
* Passwordless SSH authentication
* A working Ansible inventory
* Verified connectivity using the `ping` module
* The ability to execute ad-hoc Ansible commands across all managed nodes

---

# Author

Created as a beginner-friendly guide for learning and deploying **Ansible** in a **1 Master + 2 Managed Nodes** environment.
