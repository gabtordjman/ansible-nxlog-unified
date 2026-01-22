# Ansible NXLog Unified Deployment

This repository contains an Ansible role for deploying NXLog-CE on both Windows and Linux systems.

It's designed for infrastructure with no internet access, so every file is available locally.

This role can be used with the "Grafana-CPLEM" project: [https://github.com/gabtordjman/grafana-cplem].

## 📋 Prerequisites

### Control Node (Linux)

- Ansible 2.9+
- Python 3.x
- For Active Directory: `krb5-user` package

### Target Servers (Windows)

- Microsoft Windows OS with WinRM (minimum requirements : Windows Server 2008 R2 or Windows 7)
- PowerShell 5.1+
- User with Administrator access
- WinRM enabled (see configuration below)

### Target Servers (Linux)

- Debian/Ubuntu Linux system
- SSH access and root access (a valid SSH key is required)

## 🚀 Quick Start

### 1. Clone Repository

```bash
git clone https://github.com/gabtordjman/ansible-nxlog-unified.git
cd ansible-nxlog-unified
```

### 2. Prepare Files

Place required files in `nxlog/files/`:

- `nxlog-ce-{version}.msi` - NXLog installer for Windows systems
- `nxlog-ce-{version}_amd64.deb` - NXLog package for Linux-based systems

If the directory doesn't exist, create it.

### 3. Configure Variables

Edit `nxlog/defaults/main.yml`:

```yaml
nxlog_version: "3.2.2329"  # Must match your MSI filename or DEB package
```

Also edit `deploy-nxlog.yml` and change these variables:

```yaml
graylog_server_ip: "your-alloy-ip"
graylog_server_port: "your-alloy-port" # (12201 is the default)
```

## 🔧 Configuration

### WinRM Setup on Windows Targets

Run PowerShell as administrator on each target server:

```powershell
# Enable WinRM
Enable-PSRemoting -Force

# Configure for basic auth (standalone)
winrm set winrm/config/service '@{AllowUnencrypted="true"}'
winrm set winrm/config/service/auth '@{Basic="true"}'

# Or for Kerberos (Active Directory)
winrm set winrm/config/service/auth '@{Kerberos="true"}'
winrm set winrm/config/service/auth '@{Negotiate="true"}'

# Restart service
Restart-Service WinRM

# Add firewall rule
netsh advfirewall firewall add rule name="WinRM HTTP" dir=in action=allow protocol=TCP localport=5985
```

## For Active Directory Environments

### Kerberos Setup on Ansible Control Node

#### 1. Install Kerberos

```bash
apt-get install krb5-user

# Configure krb5.conf
cat > /etc/krb5.conf << 'EOF'
[libdefaults]
default_realm = YOURDOMAIN.tld
dns_lookup_kdc = false

[realms]
YOURDOMAIN.tld = {
    kdc = dc01.yourdomain.tld
    admin_server = dc01.yourdomain.tld
}
EOF
```

This will allow the machine to authenticate with the Active Directory users.

#### 2. Add DNS resolution

```bash
echo "192.168.1.253 dc01.yourdomain.tld" >> /etc/hosts
```

Replace the `ip` fields with the correct IP addresses of your machines that have joined the domain.

#### 3. SSH Configuration

Generate SSH key:

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/ansible_key -N ""
ssh-copy-id -i ~/.ssh/ansible_key.pub user@linux_host
```

#### 🏷️ For Kerberos Authentication

```bash
kinit user@DOMAIN.TLD
klist
```

An account in the Administrator group is recommended.

#### 4. Join an Active Directory domain (client side)

To join an Active Directory domain on Linux, see this detailed guide: [https://neptunet.fr/ubuntu-ad/]

## 🎯 Usage

### Connectivity test

```bash
# For Windows
ansible windows_servers -m win_ping -i inventory.ini

# For Linux
ansible linux_servers -m ping -i inventory.ini
```

`windows_servers` and `linux_servers` are specified in `inventory.ini`

### Deploy NXLog on all the systems

```bash
ansible-playbook deploy-nxlog.yml -i inventory.ini
```

By choosing the `inventory.ini` file, the playbook will detect the operating systems of the machines and use the correct template to generate a valid `nxlog.conf` file.

## 📝 Notes

- For large environments, Active Directory is strongly recommended:
  - Standalone deployment is also possible; however, it requires a bit more precise configuration.
  