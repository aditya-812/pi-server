# 🍓 Phase 1: Operating System Setup

Installing Ubuntu Server 24.04.3 LTS on your Raspberry Pi 5.

## 🤔 **Why Ubuntu Server 24.04.3 LTS?**

### **✅ Advantages Over Raspberry Pi OS**
- **🔄 Regular Updates**: Predictable 6-month release cycle with 5-year LTS support
- **📦 Better Package Management**: More up-to-date packages and easier Docker installation
- **🏢 Enterprise Grade**: Same OS used in production servers worldwide
- **🐳 Docker Optimized**: Better container performance and compatibility
- **📚 Documentation**: Extensive community knowledge and tutorials
- **🔧 Standardization**: Consistent with most VPS/cloud environments

### **⚖️ Trade-offs**
- **📈 Slightly Higher Resource Usage**: ~100-200MB more RAM than Pi OS Lite
- **🎮 No Pi-Specific Optimizations**: Some hardware features may need manual setup
- **📱 No Desktop Environment**: Server-only (perfect for headless operation)

### **🆚 Other Options**
- **Raspberry Pi OS Lite**: Lighter but older packages, less Docker-friendly
- **Debian**: Similar to Pi OS but manual Pi 5 hardware support
- **Alpine Linux**: Ultra-lightweight but steeper learning curve
- **DietPi**: Optimized for single-board computers but smaller community

**Verdict**: Ubuntu Server strikes the best balance of **performance, compatibility, and ease of use** for a media server.

---

## 💾 **Installation Process**

### **Step 1: Download Raspberry Pi Imager**

1. **Visit**: [rpi.org/software](https://www.raspberrypi.org/software/)
2. **Download** for your operating system:
   - Windows: `imager_latest.exe`
   - macOS: `imager_latest.dmg`
   - Linux: `imager_latest_amd64.deb` or AppImage

### **Step 2: Prepare Your SD Card**

1. **Insert** your microSD card (32GB+ recommended)
2. **Launch** Raspberry Pi Imager
3. **Click** "CHOOSE OS"

### **Step 3: Select Ubuntu Server**

1. **Navigate**: Other general-purpose OS → Ubuntu
2. **Select**: **Ubuntu Server 24.04.3 LTS (64-bit)**
3. **Click** "CHOOSE STORAGE" and select your SD card

### **Step 4: Configure Advanced Options** ⚙️

**Click the gear icon** (⚙️) for advanced options:

#### **🔐 Enable SSH**
- ✅ **Check** "Enable SSH"
- 🔑 **Select** "Use password authentication"
- 👤 **Username**: `media` (this will be our main user)
- 🔒 **Password**: Choose a strong password (you'll use this a lot)

#### **📶 Configure WiFi** (Optional)
- If using WiFi instead of ethernet:
- ✅ **Check** "Configure wireless LAN"
- 📡 **SSID**: Your WiFi network name
- 🔑 **Password**: Your WiFi password
- 🌍 **Country**: Your country code (e.g., US, GB, DE)

#### **🌐 Locale Settings**
- ⏰ **Timezone**: Your timezone (e.g., America/New_York, Europe/London)
- ⌨️ **Keyboard Layout**: Your keyboard layout (usually auto-detected)

### **Step 5: Write the Image**

1. **Click** "WRITE"
2. **Confirm** the operation (this will erase the SD card)
3. **Wait** for the process to complete (~5-15 minutes)
4. **Safely eject** the SD card when done

---

## 🚀 **First Boot Setup**

### **Step 1: Insert and Boot**

1. **Insert** the SD card into your Raspberry Pi 5
2. **Connect** ethernet cable to your router
3. **Connect** the power adapter
4. **Wait** 2-3 minutes for initial boot and setup

### **Step 2: Find Your Pi's IP Address**

#### **Method 1: Router Admin Panel**
1. **Access** your router's admin interface (usually `192.168.1.1` or `192.168.0.1`)
2. **Look** for connected devices or DHCP clients
3. **Find** device named "ubuntu" or with MAC starting with `D8:3A:DD`, `DC:A6:32`, or `E4:5F:01`

#### **Method 2: Network Scan**
```bash
# On Linux/Mac
nmap -sn 192.168.1.0/24

# On Windows (install nmap first)
nmap -sn 192.168.1.0/24

# Look for entries with "Raspberry Pi Foundation" in the vendor
```

#### **Method 3: Router Command Line**
```bash
# If you have access to your router's CLI
arp -a | grep -i "d8:3a:dd\|dc:a6:32\|e4:5f:01"
```

### **Step 3: Initial Connection Test**

Once you have the IP address (let's say it's `192.168.1.100`):

```bash
ping 192.168.1.100
```

You should see responses confirming the Pi is online.

---

## 🔧 **Initial System Configuration**

### **Step 1: First SSH Connection**

```bash
ssh media@192.168.1.100
```

- **Accept** the SSH fingerprint when prompted
- **Enter** the password you set during imaging
- You should see the Ubuntu welcome message

### **Step 2: System Update**

```bash
# Update package lists
sudo apt update

# Upgrade all packages
sudo apt upgrade -y

# Install essential tools
sudo apt install -y curl wget git htop tree unzip
```

### **Step 3: System Information**

Check your system details:

```bash
# System info
hostnamectl

# Memory and CPU
free -h
nproc

# Storage
df -h

# Ubuntu version
lsb_release -a
```

You should see:
- **OS**: Ubuntu 24.04.3 LTS
- **Architecture**: aarch64 (ARM64)
- **Memory**: 4GB or 8GB depending on your Pi model

### **Step 4: Set Static IP** (Recommended)

Create a netplan configuration:

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

Replace the contents with (adjust IPs for your network):

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 192.168.1.100/24  # Your desired static IP
      routes:
        - to: default
          via: 192.168.1.1   # Your router's IP
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```

Apply the configuration:

```bash
sudo netplan apply
```

### **Step 5: Verify Network Configuration**

```bash
# Check IP address
ip addr show eth0

# Test internet connectivity
ping -c 4 google.com

# Check DNS resolution
nslookup google.com
```

---

## ✅ **Verification Checklist**

Before proceeding to the next phase, verify:

- [ ] **SSH Access**: Can connect via SSH with media user
- [ ] **Internet Connection**: Can ping external sites
- [ ] **Package Manager**: `apt update` works without errors
- [ ] **System Resources**: At least 3GB free RAM and 25GB free storage
- [ ] **Network Stability**: Static IP configured and working

---

## 🆘 **Troubleshooting**

### **🔍 Can't Find Pi's IP Address**
```bash
# Try different network ranges
nmap -sn 192.168.0.0/24
nmap -sn 192.168.1.0/24
nmap -sn 10.0.0.0/24

# Check router's DHCP range in admin panel
```

### **🚫 SSH Connection Refused**
```bash
# Wait longer - first boot can take 5+ minutes
# Try connecting to port 22 specifically
ssh -p 22 media@IP_ADDRESS

# Check if SSH is running on the Pi (if you have monitor/keyboard)
sudo systemctl status ssh
```

### **🌐 No Internet After Static IP**
```bash
# Check netplan syntax
sudo netplan --debug apply

# Revert to DHCP temporarily
sudo nano /etc/netplan/50-cloud-init.yaml
# Change dhcp4: false to dhcp4: true
# Remove addresses, routes, nameservers sections
sudo netplan apply
```

### **📦 Package Update Errors**
```bash
# Fix broken packages
sudo apt --fix-broken install

# Clear package cache
sudo apt clean && sudo apt autoclean

# Reset package lists
sudo rm -rf /var/lib/apt/lists/*
sudo apt update
```

---

## 🎯 **Next Steps**

✅ **Congratulations!** You now have Ubuntu Server running on your Raspberry Pi 5.

**Continue to**: [Phase 2: SSH & Remote Access Setup](02-ssh-setup.md)

---

*Phase 1 Complete | Next: SSH Setup | Estimated time: 30-60 minutes*
