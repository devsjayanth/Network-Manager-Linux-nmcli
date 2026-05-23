# 🐧Linux Network Manager Quick Guide

## Installation:

### 🐧 Debian / Ubuntu
**Install & Enable:**
```bash
sudo apt update
sudo apt install -y network-manager
sudo systemctl disable --now systemd-networkd  # avoid conflicts
sudo systemctl enable --now NetworkManager
```
*(Note: Not installed by default on Debian Server. Service name is `NetworkManager` with capital N.)*

---

### 🎩 RHEL / CentOS / Rocky / Alma / Fedora
**Install & Enable:**
```bash
# RHEL 8+/Fedora/Rocky/Alma
sudo dnf install -y NetworkManager

# RHEL 7 / CentOS 7
sudo yum install -y NetworkManager

# Usually already installed & running, but ensure it's active:
sudo systemctl enable --now NetworkManager
```
*(Note: Enabled by default on all modern RHEL-family systems.)*

---

### 🔧 Universal `nmcli` Static IP Setup (Works on Both)
1. **Find your interface & connection name:**
```bash
nmcli device status
nmcli connection show
```
*(Note the `DEVICE` name, e.g., `eth0`, `ens192`, and the `NAME` column for existing connections.)*

## Edit the Existing Network settings

### 🔍 Step 1: Find your existing connection name
```bash
nmcli connection show
```
Look at the `NAME` column. It might be something like `Wired connection 1`, `System eth0`, `ens33`, or `eth0`.  
*(Note: `nmcli` edits the **connection profile**, not the raw interface. The profile is already linked to your NIC.)*

---

### 🛠 Step 2: Modify the existing profile
Replace `"Your-Connection-Name"` with the exact name from Step 1:
```bash
nmcli connection modify "Your-Connection-Name" \
  ipv4.method manual \
  ipv4.addresses 192.168.1.100/24 \
  ipv4.gateway 192.168.1.1 \
  ipv4.dns "8.8.8.8 1.1.1.1" \
  ipv4.ignore-auto-dns yes \
  ipv4.may-fail no
```
🔸 **What each flag does:**
- `ipv4.method manual` → Forces static IP (instead of `auto`/DHCP)
- `ipv4.addresses` → Your static IP + CIDR subnet
- `ipv4.gateway` → Default route
- `ipv4.dns` → Space-separated DNS servers
- `ipv4.ignore-auto-dns yes` → Prevents DHCP from overwriting DNS
- `ipv4.may-fail no` → Stops NM from falling back to DHCP if link is slow

---

### 🔄 Step 3: Apply the changes
```bash
nmcli connection up "Your-Connection-Name"
```
This reloads the profile and pushes it to the physical interface. No reboot needed.

---

### ✅ Step 4: Verify
```bash
nmcli connection show "Your-Connection-Name" | grep -E "ipv4\.(method|address|gateway|dns)"
ip addr show | grep inet
ping -c 3 8.8.8.8
```

---

### 🛑 SSH Warning & Recovery
If you're editing over SSH, a typo can drop your connection.  
**Recover to DHCP if needed:**
```bash
nmcli connection modify "Your-Connection-Name" ipv4.method auto ipv4.addresses "" ipv4.gateway ""
nmcli connection up "Your-Connection-Name"
```

---

### 📋 Quick `nmcli` Edit Reference
| What to change          | Command snippet                                  |
|-------------------------|--------------------------------------------------|
| Change only IP          | `ipv4.addresses 10.0.0.50/24`                    |
| Change only gateway     | `ipv4.gateway 10.0.0.1`                          |
| Add secondary IP        | `+ipv4.addresses 10.0.0.51/24`                   |
| Remove all static IPs   | `ipv4.addresses ""`                              |
| Enable IPv6 SLAAC       | `ipv6.method auto`                               |
| Disable IPv6            | `ipv6.method disabled`                           |

Run `nmcli connection show "Your-Connection-Name"` to dump every current setting before/after editing.
