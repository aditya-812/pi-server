# 👤 Phase 3: User & Permissions Setup

Setting up proper user accounts and permissions for your media server.

## 🎯 **What We'll Accomplish**

- **✅ Understand the `media` user** we created during OS installation
- **🔧 Configure proper permissions** for Docker and media files
- **📁 Set up directory structure** for our services
- **🛡️ Ensure security** while maintaining functionality

---

## 👥 **Understanding Users in Our Setup**

### **The `media` User**

During OS installation, we created a user called `media`. This user will:
- **🏠 Own all Docker configurations** and data
- **📁 Have access to media files** on the external drive
- **🐳 Run Docker containers** with proper permissions
- **🔐 Be our main SSH user** for system administration

### **Why Not Use `root` or `pi`?**

- **🚫 Root**: Too powerful, security risk if compromised
- **🚫 Pi**: Default user, predictable for attackers
- **✅ Media**: Custom user, dedicated purpose, easier to manage

---

## 🔍 **Current User Status Check**

### **Step 1: Verify Current User**

```bash
# Check current user
whoami

# Check user details
id media

# Check groups
groups media

# Check home directory
ls -la /home/media/
```

You should see output like:
```
uid=1000(media) gid=1000(media) groups=1000(media),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),100(users)
```

### **Step 2: Check Sudo Access**

```bash
# Test sudo access
sudo whoami
# Should return: root

# Check sudo configuration
sudo -l
```

---

## 🐳 **Docker Permissions Setup**

### **Step 1: Add Media User to Docker Group**

When we install Docker later, we'll need the `media` user to manage containers without `sudo`:

```bash
# Note: We'll run this command after Docker installation
# Adding here for reference - Docker group doesn't exist yet
# sudo usermod -aG docker media
```

### **Step 2: Understanding Docker User Mapping**

Docker containers can run as different users. We'll configure our containers to use:
- **User ID (UID)**: 1000 (media user's ID)
- **Group ID (GID)**: 1000 (media group's ID)

This ensures files created by containers are owned by the `media` user.

---

## 📁 **Directory Structure Setup**

### **Step 1: Create Docker Configuration Directory**

```bash
# Create main docker directory
mkdir -p /home/media/docker

# Create subdirectories for each service
mkdir -p /home/media/docker/{jellyfin,sonarr,radarr,prowlarr,qbittorrent,jellyseerr}
mkdir -p /home/media/docker/{glance-local,glance-ts,recyclarr,filebrowser}
mkdir -p /home/media/docker/{pihole,netdata,jellystat}

# Create data directories
mkdir -p /home/media/docker/jellyfin/{config,cache}
mkdir -p /home/media/docker/sonarr/config
mkdir -p /home/media/docker/radarr/config
mkdir -p /home/media/docker/prowlarr/config
mkdir -p /home/media/docker/qbittorrent/config
mkdir -p /home/media/docker/jellyseerr/config
mkdir -p /home/media/docker/glance-local/config
mkdir -p /home/media/docker/glance-ts/config
mkdir -p /home/media/docker/recyclarr/config
mkdir -p /home/media/docker/filebrowser/{database,settings}
mkdir -p /home/media/docker/pihole/{etc-pihole,etc-dnsmasq.d}
mkdir -p /home/media/docker/netdata/{config,lib}
mkdir -p /home/media/docker/jellystat/{config,db}
```

### **Step 2: Set Proper Ownership**

```bash
# Ensure media user owns all docker directories
sudo chown -R media:media /home/media/docker/

# Set proper permissions
chmod -R 755 /home/media/docker/
```

### **Step 3: Verify Directory Structure**

```bash
# Check directory structure
tree /home/media/docker/ -L 2

# Check ownership
ls -la /home/media/docker/
```

---

## 💾 **External Storage Preparation**

### **Step 1: Create Mount Point**

```bash
# Create mount point for external drive
sudo mkdir -p /mnt/media

# Create media subdirectories (we'll use these after mounting)
# Note: We'll create these after mounting the drive
```

### **Step 2: Identify External Drive**

Connect your external USB drive and identify it:

```bash
# List all storage devices
lsblk

# Check for new devices
sudo fdisk -l

# Look for USB devices
dmesg | grep -i usb | tail -10
```

You should see something like:
```
sda      8:0    0  1.8T  0 disk 
└─sda1   8:1    0  1.8T  0 part
```

### **Step 3: Format Drive (if needed)**

**⚠️ Warning**: This will erase all data on the drive!

```bash
# Check if drive has existing filesystem
sudo file -s /dev/sda1

# If it shows "data" or you want to reformat:
sudo mkfs.ext4 /dev/sda1 -L MediaDrive

# Create filesystem with label
sudo e2label /dev/sda1 MediaDrive
```

### **Step 4: Mount External Drive**

```bash
# Mount temporarily to test
sudo mount /dev/sda1 /mnt/media

# Check if mounted
df -h | grep media

# Create media directories
sudo mkdir -p /mnt/media/{Movies,TV-Shows,Downloads,Music,Books}

# Set ownership to media user
sudo chown -R media:media /mnt/media/
```

### **Step 5: Configure Automatic Mounting**

```bash
# Get drive UUID
sudo blkid /dev/sda1

# Edit fstab for automatic mounting
sudo nano /etc/fstab
```

Add this line (replace UUID with your actual UUID):
```
UUID=your-drive-uuid /mnt/media ext4 defaults,nofail,user,exec 0 2
```

Test the fstab entry:
```bash
# Unmount first
sudo umount /mnt/media

# Test automatic mounting
sudo mount -a

# Verify it worked
df -h | grep media
```

---

## 🔐 **Permission Configuration for Docker**

### **Step 1: Understanding Docker User Mapping**

Our Docker containers will use these environment variables:
- **PUID=1000** (media user ID)
- **PGID=1000** (media group ID)

### **Step 2: Check User/Group IDs**

```bash
# Get media user ID and group ID
id media

# Should show: uid=1000(media) gid=1000(media)
```

If your IDs are different, note them down - you'll need to adjust the Docker configurations.

### **Step 3: Create Environment Variables File**

```bash
# Create .env file for Docker
nano /home/media/.env
```

Add these variables:
```bash
# User and Group IDs
PUID=1000
PGID=1000

# Timezone
TZ=Your/Timezone

# Network settings (we'll fill these later)
# RADARR_API_KEY=
# SONARR_API_KEY=
# PROWLARR_API_KEY=
# JELLYFIN_API_KEY=
```

---

## 🔍 **Understanding Docker User Behavior**

### **Why Some Services Need User Configuration**

Different Docker containers handle users differently:

#### **✅ LinuxServer.io Images** (Sonarr, Radarr, etc.)
```yaml
environment:
  - PUID=1000
  - PGID=1000
```
These containers **automatically** switch to the specified user inside the container.

#### **⚙️ Official Images** (Jellyfin, etc.)
```yaml
user: "1000:1000"
```
These need the `user:` directive to run as the correct user.

#### **🔧 Custom Configuration** (Recyclarr, etc.)
```yaml
user: "${PUID}:${PGID}"
```
Uses environment variables for flexibility.

### **Why This Matters**

- **📁 File Ownership**: Files created by containers are owned by `media` user
- **🔐 Permissions**: No permission denied errors when accessing files
- **🛡️ Security**: Containers don't run as root unnecessarily
- **🔧 Maintenance**: Easy to backup and manage files

---

## ✅ **Verification Checklist**

Before proceeding, verify:

- [ ] **Media User**: Can SSH as media user
- [ ] **Sudo Access**: Media user can use sudo
- [ ] **Docker Directories**: All service directories created and owned by media
- [ ] **External Drive**: Mounted at `/mnt/media` with proper permissions
- [ ] **Auto-Mount**: Drive mounts automatically after reboot
- [ ] **Environment File**: `.env` file created with PUID/PGID

### **Quick Verification Commands**

```bash
# Check user
whoami && id

# Check docker directory
ls -la /home/media/docker/ | head -5

# Check external drive
df -h | grep media && ls -la /mnt/media/

# Check environment file
cat /home/media/.env

# Test reboot and auto-mount
sudo reboot
# After reboot, SSH back in and check:
df -h | grep media
```

---

## 🆘 **Troubleshooting**

### **🚫 Permission Denied on External Drive**

```bash
# Check mount options
mount | grep media

# Remount with proper options
sudo umount /mnt/media
sudo mount -o defaults,user,exec /dev/sda1 /mnt/media

# Fix ownership
sudo chown -R media:media /mnt/media/
```

### **📁 Directory Creation Errors**

```bash
# Check available space
df -h

# Check parent directory permissions
ls -la /home/media/

# Recreate with proper permissions
sudo mkdir -p /home/media/docker
sudo chown media:media /home/media/docker
```

### **🔄 Auto-Mount Not Working**

```bash
# Check fstab syntax
sudo mount -a

# View mount errors
sudo journalctl -u systemd-remount-fs

# Test manual mount
sudo mount UUID=your-uuid /mnt/media
```

### **🆔 Wrong User/Group IDs**

If your media user doesn't have UID 1000:

```bash
# Check current ID
id media

# Update .env file with correct values
nano /home/media/.env
# Change PUID and PGID to match your user's actual IDs
```

---

## 🎯 **Next Steps**

✅ **Excellent!** Your user permissions and directory structure are properly configured.

**Continue to**: [Phase 4: Docker Installation](04-docker-setup.md)

---

*Phase 3 Complete | Next: Docker Setup | Estimated time: 30-45 minutes*
