
# Ansible NXLog Deployment

This repository contains an Ansible role for deploying NXLog Community Edition on Windows and Linux systems in both standalone and Active Directory environments. It's designed for infrastructure with no internet connection, so every file will be available on the Ansible server.

## 📋 Prerequisites

### Control Node (Linux)
- Ansible 2.9+
- Python 3.x
- For Active Directory: `krb5-user` package

### Target Servers (Windows)
- Windows Server 2012 R2+ or Windows 10/11
- PowerShell 5.1+
- Administrator access
- WinRM enabled (see configuration below)

## 🚀 Quick Start

### 1. Clone Repository
```bash
git clone https://github.com/gabtordjman/ansible-nxlog-unified.git
cd ansible-nxlog
```
### 2. Prepare Files

Place required files in `nxlog/files/`:

-   `nxlog-ce-{version}.msi` - NXLog installer for Windows systems

-   `nxlog-ce-{version}_amd64.deb` - NXLog package for Linux based systems

### 3. Configure Variables

Edit `nxlog/defaults/main.yml`:
```bash
nxlog_version: "3.2.2329"  # Must match your MSI filename or DEB pacakge
```
## 🔧 Configuration

### WinRM Setup on Windows Targets

Run PowerShell as **Administrator** on each target server:

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
# For Active Directory Environments

### 1. **Kerberos Setup on Ansible Control Node**

## Install Kerberos on Debian

```
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
This will allow the machine to authenticate with the Active Directory users

## Add DNS resolution
```
echo "192.168.1.253 dc01.yourdomain.tld" >> /etc/hosts
```
Replace the `ip` fields with the correct ip addresses of your machines that have joined the domain

### SSH Configuration

Generate SSH key:
```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/ansible_key -N ""
ssh-copy-id -i ~/.ssh/ansible_key.pub user@linux_host
```
### **🏷️ For Kerberos Authentication**
```bash
kinit user@DOMAIN.TLD
klist
```
Account in the Administrator group is recommended

## 🎯 Usage

### Connectivity test
```bash
# For Windows
ansible windows_servers -m win_ping -i inventory.ini

# For Linux
ansible linux_servers -m win_ping -i inventory.ini
```
`windows_servers` and `linux_servers` are specified in `inventory.ini`
### Deploy NXLog on all the systems
```
ansible-playbook deploy-nxlog.yml -i inventory.ini
```
By choosing the `inventory.ini` file, the playbook will detect the operating systems of the machines and use the correct template for the `nxlog.conf` file.
