# 🔐 Phase 2: SSH & Remote Access Setup

Configuring secure and convenient SSH access to your Raspberry Pi.

## 🎯 **What We'll Accomplish**

- **✅ Verify SSH is working** properly
- **🔧 Optimize SSH configuration** for better experience  
- **🛡️ Basic security hardening** (optional but recommended)
- **📱 Set up convenient access** from multiple devices

---

## 🔍 **Verify Current SSH Setup**

### **Step 1: Test Your Connection**

From your computer, connect to your Pi:

```bash
ssh media@YOUR_PI_IP
```

You should see something like:
```
Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.8.0-1015-raspi aarch64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

Last login: [timestamp] from [your-computer-ip]
media@ubuntu:~$
```

### **Step 2: Check SSH Service Status**

On your Pi, verify SSH is running properly:

```bash
# Check SSH service status
sudo systemctl status ssh

# Check SSH configuration
sudo sshd -T | grep -E "port|authentication|permitroot"

# View active SSH connections
who
```

---

## ⚙️ **SSH Configuration Optimization**

### **Step 1: Backup Original Configuration**

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

### **Step 2: Edit SSH Configuration**

```bash
sudo nano /etc/ssh/sshd_config
```

**Find and modify these settings** (or add if missing):

```bash
# Basic settings
Port 22                          # Keep default port for simplicity
Protocol 2                       # Use SSH protocol version 2

# Authentication
PermitRootLogin no              # Disable root login for security
PasswordAuthentication yes       # Allow password auth (we're using this)
PubkeyAuthentication yes        # Enable key auth (for future use)
PermitEmptyPasswords no         # Never allow empty passwords

# Connection settings
MaxAuthTries 3                  # Limit login attempts
MaxSessions 10                  # Allow multiple sessions
ClientAliveInterval 300         # Keep connections alive (5 minutes)
ClientAliveCountMax 2           # Disconnect after 2 missed keepalives

# Performance
UseDNS no                       # Skip DNS lookups for faster connections
```

### **Step 3: Apply Configuration**

```bash
# Test configuration syntax
sudo sshd -t

# If no errors, restart SSH service
sudo systemctl restart ssh

# Verify service is running
sudo systemctl status ssh
```

### **Step 4: Test New Configuration**

**Open a NEW terminal** (keep your current SSH session open as backup):

```bash
ssh media@YOUR_PI_IP
```

If successful, you can close the backup session.

---

## 🏠 **Setting Up Convenient Access**

### **Step 1: Create SSH Config on Your Computer**

On your **local computer** (not the Pi), create an SSH config file:

#### **Linux/Mac:**
```bash
mkdir -p ~/.ssh
nano ~/.ssh/config
```

#### **Windows (PowerShell):**
```powershell
mkdir $env:USERPROFILE\.ssh -Force
notepad $env:USERPROFILE\.ssh\config
```

### **Step 2: Add Pi Configuration**

Add this to your SSH config file:

```
Host pi-server
    HostName YOUR_PI_IP
    User media
    Port 22
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

Replace `YOUR_PI_IP` with your Pi's actual IP address.

### **Step 3: Test Convenient Access**

Now you can connect simply with:

```bash
ssh pi-server
```

Much easier than remembering the IP address!

---

## 🛡️ **Basic Security Hardening** (Optional)

### **Step 1: Change Default SSH Port** (Optional)

If you want extra security, change the SSH port:

```bash
sudo nano /etc/ssh/sshd_config
```

Change:
```
Port 22
```
To:
```
Port 2222
```

Then restart SSH:
```bash
sudo systemctl restart ssh
```

Update your SSH config:
```
Host pi-server
    HostName YOUR_PI_IP
    User media
    Port 2222
```

### **Step 2: Install Fail2Ban** (Optional)

Protect against brute force attacks:

```bash
# Install fail2ban
sudo apt install -y fail2ban

# Create local configuration
sudo nano /etc/fail2ban/jail.local
```

Add this configuration:
```ini
[DEFAULT]
bantime = 1h
findtime = 10m
maxretry = 3

[sshd]
enabled = true
port = 22
filter = sshd
logpath = /var/log/auth.log
```

Start and enable fail2ban:
```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# Check status
sudo fail2ban-client status sshd
```

---

## 📱 **Multi-Device Access Setup**

### **Step 1: Mobile SSH Apps**

For smartphone access, install:
- **iOS**: Termius, Prompt 3
- **Android**: Termius, JuiceSSH, ConnectBot

### **Step 2: Configure Mobile Apps**

In your mobile SSH app, create a connection with:
- **Name**: Pi Server
- **Host**: YOUR_PI_IP
- **Port**: 22 (or 2222 if changed)
- **Username**: media
- **Password**: Your media user password

### **Step 3: Tablet/Laptop Access**

For additional computers, repeat the SSH config setup:

1. **Install SSH client** (usually pre-installed on Linux/Mac)
2. **Create SSH config** with the pi-server host entry
3. **Test connection** with `ssh pi-server`

---

## 🔧 **Useful SSH Commands & Tips**

### **File Transfer Commands**

```bash
# Copy file TO Pi
scp /local/file.txt pi-server:/home/media/

# Copy file FROM Pi  
scp pi-server:/home/media/file.txt /local/destination/

# Copy directory recursively
scp -r /local/directory/ pi-server:/home/media/

# Using rsync (better for large transfers)
rsync -avz /local/directory/ pi-server:/home/media/directory/
```

### **Useful SSH Session Commands**

```bash
# Keep session alive when closing laptop
screen -S media-server
# Do your work, then detach with Ctrl+A, D
# Reconnect later with: screen -r media-server

# Alternative: tmux
tmux new-session -s media-server
# Detach with Ctrl+B, D
# Reconnect with: tmux attach -t media-server

# Run command in background
nohup long-running-command &

# Monitor system resources
htop
# Or
top
```

### **SSH Tunneling** (Advanced)

```bash
# Access Pi's web services through SSH tunnel
ssh -L 8080:localhost:8080 pi-server
# Now access http://localhost:8080 on your computer

# Multiple ports
ssh -L 8080:localhost:8080 -L 9696:localhost:9696 pi-server
```

---

## ✅ **Verification Checklist**

Before proceeding, verify:

- [ ] **SSH Connection**: Can connect with `ssh pi-server`
- [ ] **Multiple Sessions**: Can open multiple SSH sessions
- [ ] **File Transfer**: Can copy files to/from Pi with `scp`
- [ ] **Service Status**: SSH service is active and enabled
- [ ] **Mobile Access**: Can connect from phone/tablet (if needed)

---

## 🆘 **Troubleshooting**

### **🚫 Connection Refused After Config Changes**

```bash
# Check SSH service status
sudo systemctl status ssh

# Check configuration syntax
sudo sshd -t

# View SSH logs
sudo journalctl -u ssh -f

# Restart SSH service
sudo systemctl restart ssh
```

### **🐌 Slow SSH Connections**

```bash
# Add to /etc/ssh/sshd_config
UseDNS no
GSSAPIAuthentication no

# Restart SSH
sudo systemctl restart ssh
```

### **🔐 Permission Denied Errors**

```bash
# Check user exists
id media

# Check SSH directory permissions
ls -la /home/media/.ssh/

# Fix permissions if needed
chmod 700 /home/media/.ssh/
chmod 600 /home/media/.ssh/authorized_keys
```

### **📱 Mobile App Connection Issues**

1. **Double-check IP address** - Pi might have gotten new DHCP lease
2. **Verify port number** - Default is 22
3. **Check firewall** - Ubuntu firewall is disabled by default
4. **Test from computer first** - Ensure SSH is working

---

## 🎯 **Next Steps**

✅ **Great!** You now have reliable SSH access to your Pi from multiple devices.

**Continue to**: [Phase 3: User & Permissions Setup](03-user-permissions.md)

---

*Phase 2 Complete | Next: User & Permissions | Estimated time: 20-30 minutes*
