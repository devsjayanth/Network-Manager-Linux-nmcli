# 🛠️ Steps: Static IP + Hostname (RedHat/Fedora)

### 🔑 Placeholder Legend
- `<CONNECTION_NAME>` → Exact name from `nmcli connection show` (e.g., `System eth0`)
- `<STATIC_IP>` → Your desired IP (e.g., `10.0.0.50`)
- `<SUBNET_PREFIX>` → CIDR mask (e.g., `24` for `255.255.255.0`)
- `<GATEWAY_IP>` → Your router/gateway (e.g., `10.0.0.1`)
- `<DNS_SERVERS>` → Comma-separated, no spaces (e.g., `8.8.8.8,1.1.1.1`)
- `<NEW_HOSTNAME>` → FQDN (e.g., `server01.example.com`)
- `<SHORT_NAME>` → Hostname without domain (e.g., `server01`)

---

## 🌐 1. Set Static IP (Line-by-Line)
```bash
# Find your connection name first
nmcli connection show

# 1. Set IP & Subnet
sudo nmcli connection modify "<CONNECTION_NAME>" ipv4.addresses <STATIC_IP>/<SUBNET_PREFIX>

# 2. Set Gateway
sudo nmcli connection modify "<CONNECTION_NAME>" ipv4.gateway <GATEWAY_IP>

# 3. Set DNS
sudo nmcli connection modify "<CONNECTION_NAME>" ipv4.dns "<DNS_SERVERS>"

# 4. Switch to static mode
sudo nmcli connection modify "<CONNECTION_NAME>" ipv4.method manual

# 5. Auto-connect on boot
sudo nmcli connection modify "<CONNECTION_NAME>" connection.autoconnect yes

# 6. Apply changes
sudo nmcli connection up "<CONNECTION_NAME>"
```

---

## 🏷️ 2. Change Hostname
```bash
# Set hostname
sudo hostnamectl set-hostname <NEW_HOSTNAME>

# Edit /etc/hosts
sudo nano /etc/hosts
#or
sudo vi /etc/hosts
```
**Add/edit this line inside the file:**
```
127.0.0.1  localhost  <NEW_HOSTNAME>  <SHORT_NAME>
```
💡 **Editor Quick Keys:**
- **`nano`**: Edit → `Ctrl+O` → `Enter` → `Ctrl+X`
- **`vi/vim`**: Press `i` → edit → `Esc` → type `:wq` → `Enter`

---

## 🔁 3. Revert Back to DHCP
```bash
# Switch back to automatic (DHCP)
sudo nmcli connection modify "<CONNECTION_NAME>" ipv4.method auto

# Apply
sudo nmcli connection up "<CONNECTION_NAME>"

# Optional: Reset hostname
sudo hostnamectl set-hostname <OLD_HOSTNAME>
# Edit /etc/hosts and remove the custom line if needed
```

---

## ✅ 4. Verify
```bash
ip -4 addr show       # Check IP
hostnamectl           # Check hostname
ping -c 3 8.8.8.8     # Test internet
```
