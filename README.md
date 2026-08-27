# Cloud-1: Infrastructure as Code (IaC)

Automated deployment of a containerized infrastructure (Traefik, WordPress, MariaDB, phpMyAdmin) on a remote host using **Ansible** and **Docker Compose**.

## 🚀 Overview

This project provisions and configures a server from scratch. It utilizes Ansible roles to separate system-level dependencies from application orchestration, ensuring **idempotency**, **modularity**, and **zero hardcoded secrets**.

### Architecture Overview

* **Ansible**: Infrastructure setup, package management, secret decryption (Vault), and Jinja2 template rendering.
* **Traefik**: Reverse proxy with automatic SSL/TLS configuration and path routing.
* **WordPress & MariaDB**: Core web service and persistent relational database.
* **phpMyAdmin**: Database management GUI exposed strictly via `/pma/`.

## 🛠️ Project Structure

```text
.
├── inventory/
│   └── hosts.ini              # Inventory file (target IP & SSH user)
├── group_vars/
│   └── web.yml                # Group variables & encrypted Vault secrets
├── roles/
│   ├── docker/                # System role: Installs Docker & Compose V2
│   │   └── tasks/
│   │       └── main.yml
│   └── app/                   # App role: Renders template & deploys services
│       ├── tasks/
│       │   └── main.yml
│       └── templates/
│           └── docker-compose.yml.j2
├── ansible.cfg                # Ansible configuration file
├── cleanup.yml                # Playbook to remove containers & directory
├── playbook.yml               # Main deployment playbook
├── .gitignore
└── README.md
```

## 📋 Requirements

* **Local Machine**: Ansible (`ansible-playbook`), `ansible-vault`
* **Target Host**: Debian/Ubuntu OS with SSH access and `sudo` privileges

## ⚙️ Configuration & Secrets

### Inventory Configuration
Update `inventory/hosts.ini` with your target host IP address and user:

```ini
[web]
<TARGET_IP> ansible_user=<SSH_USER>
```

### Secrets Management
All database credentials are encrypted using **Ansible Vault** inside `group_vars/web.yml`.
* To edit secrets:
  ```bash
  ansible-vault edit group_vars/web.yml
  ```
* To view encrypted secrets:
  ```bash
  ansible-vault view group_vars/web.yml
  ```

## 🏃 Deployment
To execute the deployment playbook on the target host:
```bash
ansible-playbook playbook.yml --ask-vault-pass
```

### Idempotency & Re-deployment
Running the same playbook multiple times will result in `changed=0`, confirming that the system state is preserved without unexpected side effects or downtime.

## 🌐 Endpoints
Once deployed, access the services via HTTPS (accept self-signed certificate if prompted):
| Service | Access URL | Description |
| :--- | :--- | :--- |
| **WordPress** | `https://<TARGET_IP>/` | Main web application |
| **phpMyAdmin** | `https://<TARGET_IP>/pma/` | Database management tool |

### SSH Access to Target VM
To connect directly to the target virtual machine via SSH from any local machine (using a custom SSH key located in your project directory):

```bash
ssh -i ./id_rsa <SSH_USER>@<TARGET_IP>
```
> [!NOTE]
> *Make sure the key file has restricted permissions (`chmod 600 ./id_rsa`), otherwise SSH will reject the connection.*

## 🧹 Cleanup
To safely stop containers, prune unused Docker resources, and clean up the project directory on the remote host, run:
```bash
ansible-playbook cleanup.yml --ask-vault-pass
```
