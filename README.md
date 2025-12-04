# SSH Key Configuration & Backup Automation

Automate SSH key-based authentication and configuration backup for Cisco IOS devices using Ansible.

## 📋 Project Overview

This project provides Ansible playbooks to:
- **Configure SSH authentication** using RSA key pairs on Cisco routers/switches
- **Automatically backup** running configurations from network devices
- **Manage user provisioning** with elevated privileges

## 🏗️ Project Structure

```
ssh-key-config-automation/
├── README.md                          # This file
├── backup_cisco_config/               # Configuration backup automation
│   ├── ansible.cfg                    # Ansible configuration for backup tasks
│   ├── hosts.ini                      # Inventory with SSH key authentication
│   ├── backup_cisco_router_playbook.yml  # Playbook to backup running-config
│   └── backups/                       # Directory where configs are saved
│
└── ssh_by_key/                        # SSH key configuration automation
    ├── ansible.cfg                    # Ansible configuration for SSH setup
    ├── hosts.ini                      # Inventory with password authentication
    └── ssh_key.yml                    # Playbook to configure SSH keys
```

## 🚀 Quick Start

### Prerequisites

1. **Install Ansible** (minimum version 2.9)
   ```bash
   pip install ansible
   ```

2. **Install Cisco IOS network module** (if not already included)
   ```bash
   ansible-galaxy collection install cisco.ios
   ```

3. **Generate SSH keypair** (if not already done)
   ```bash
   ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa
   ```

4. **Update inventory files** with your network device details

### Step 1: Initial Configuration (SSH Key Setup)

Configure SSH authentication on your Cisco device using the initial playbook:

```bash
cd ssh_by_key/
ansible-playbook ssh_key.yml -i hosts.ini
```

**Prerequisites for this step:**
- Device must be reachable via SSH with default credentials
- Default user: `cisco`
- Default password: `cisco123!`
- Device must have SSH enabled

**What this playbook does:**
- Creates an `ansible` user with privilege level 15
- Adds your public SSH key to the device
- Disables password-based authentication (optional)

### Step 2: Backup Configurations

After SSH keys are configured, backup the running configuration:

```bash
cd ../backup_cisco_config/
ansible-playbook backup_cisco_router_playbook.yml -i hosts.ini
```

**What this playbook does:**
- Connects to the device using SSH keys (no password required)
- Retrieves the running configuration
- Saves it to `backups/show_run_<hostname>.txt`

## ⚙️ Configuration Details

### SSH Key Configuration (`ssh_by_key/`)

**Inventory (`hosts.ini`):**
```ini
CSR1kv ansible_user=cisco ansible_password=cisco123! ansible_host=192.168.200.132 ansible_network_os=ios
```

**Playbook (`ssh_key.yml`):**
- Creates user `ansible` with:
  - No password authentication (SSH keys only)
  - SSH public key from `~/.ssh/id_rsa.pub`
  - Privilege level 15 (full admin access)

### Backup Configuration (`backup_cisco_config/`)

**Inventory (`hosts.ini`):**
```ini
CSR1kv ansible_user=cisco ansible_host=192.168.56.101 ansible_network_os=ios
```

**Playbook (`backup_cisco_router_playbook.yml`):**
- Uses SSH key authentication (configured in `ansible.cfg`)
- Executes `show running-config` command
- Saves output with hostname timestamp

## 📝 Ansible Configuration (`ansible.cfg`)

Both directories contain `ansible.cfg` with:
- Local inventory file reference
- SSH private key location
- Disabled RSA fingerprint checking
- Disabled deprecation warnings
- Disabled retry files creation

## 🔑 SSH Key Authentication

### How it works:

1. **First connection**: Uses password authentication
   ```
   ansible → device (SSH with user/password)
   ```

2. **Key deployment**: Public key is added to device
   ```
   ansible_user=ansible
   sshkey="{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
   ```

3. **Subsequent connections**: Uses SSH key only
   ```
   ansible → device (SSH with private key)
   No password required
   ```

## 📊 Backup Storage

Configurations are backed up to:
```
backup_cisco_config/backups/show_run_<hostname>.txt
```

Example: `show_run_CSR1kv.txt`

## 🛠️ Usage Examples

### Run SSH key configuration:
```bash
cd ssh_by_key
ansible-playbook ssh_key.yml -i hosts.ini -v
```

### Run backup with verbose output:
```bash
cd backup_cisco_config
ansible-playbook backup_cisco_router_playbook.yml -i hosts.ini -vvv
```

### Dry-run (check mode):
```bash
ansible-playbook ssh_key.yml -i hosts.ini --check
```

### Run against specific host:
```bash
ansible-playbook ssh_key.yml -i hosts.ini --limit CSR1kv
```

## 🔐 Security Considerations

1. **SSH Keys**: Keep your private key (`~/.ssh/id_rsa`) secure
2. **Inventory**: Don't commit `hosts.ini` with passwords to version control
3. **Permissions**: Restrict file permissions:
   ```bash
   chmod 600 ~/.ssh/id_rsa
   chmod 644 ~/.ssh/id_rsa.pub
   ```

## 🐛 Troubleshooting

### Connection refused
- Verify device IP address and SSH port (default: 22)
- Ensure SSH is enabled on the Cisco device
- Check firewall rules

### Authentication failed
- Verify credentials in `hosts.ini`
- Confirm SSH key has been properly deployed
- Check device user privileges

### Command timeout
- Increase timeout in `ansible.cfg` or playbook
- Verify network connectivity
- Check device resource utilization

## 📚 Supported Devices

- Cisco IOS (routers, switches)
- Cisco CSR 1000v
- Compatible with `network_cli` connection plugin

## 📖 References

- [Ansible Network Modules](https://docs.ansible.com/ansible/latest/collections/cisco/ios/index.html)
- [Cisco IOS Configuration Guide](https://www.cisco.com/c/en/us/support/ios-nx-os-software/ios-software/products.html)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)

## 📄 License

Project for educational purposes

## 👤 Author

**meissadev** 