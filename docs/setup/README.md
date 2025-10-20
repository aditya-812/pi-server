# 🚀 Complete Setup Guide - From Zero to Media Server

This guide takes you from a blank Raspberry Pi to a fully functional self-hosted media center. Follow these steps in order for the best experience.

## 📋 **What You'll Build**

By the end of this guide, you'll have:
- **🍓 Ubuntu Server** running on Raspberry Pi 5
- **🐳 Docker & Portainer** for container management
- **🎬 Complete Media Stack** (Jellyfin, *arr apps, qBittorrent)
- **📊 Monitoring Dashboard** (Glance, Pi-hole, Netdata)
- **🌐 Remote Access** via Tailscale VPN
- **🔒 Secure Setup** with proper user permissions

## 🛠️ **Hardware Requirements**

- **Raspberry Pi 5** (4GB+ RAM recommended)
- **MicroSD Card** (32GB+ Class 10)
- **External Storage** (2TB+ USB drive for media)
- **Official Pi 5 Case** with active cooling (recommended)
- **Reliable Power Supply** (Official Pi 5 adapter)
- **Ethernet Connection** (for initial setup)

---

## 📚 **Setup Phases**

### **Phase 1: Foundation** 🏗️
1. [**Operating System Setup**](01-operating-system.md) - Ubuntu Server installation and initial configuration
2. [**SSH & Remote Access**](02-ssh-setup.md) - Secure headless access to your Pi
3. [**User & Permissions Setup**](03-user-permissions.md) - Creating the media user and proper permissions

### **Phase 2: Core Services** 🐳
4. [**Docker Installation**](04-docker-setup.md) - Container platform setup
5. [**Portainer Deployment**](05-portainer-setup.md) - Web-based Docker management
6. [**Storage Configuration**](06-storage-setup.md) - External drive setup and mounting

### **Phase 3: Media Stack** 🎬
7. [**Media Services Deployment**](07-media-stack.md) - Jellyfin, *arr apps, qBittorrent
8. [**Service Configuration**](08-service-config.md) - API keys, connections, and basic setup
9. [**Quality Management**](09-recyclarr-setup.md) - Automated quality profiles with Recyclarr

### **Phase 4: Monitoring & Dashboard** 📊
10. [**Monitoring Stack**](10-monitoring-stack.md) - Pi-hole, Netdata, Glances
11. [**Glance Dashboard**](11-glance-dashboard.md) - Beautiful unified dashboard
12. [**Tailscale VPN**](12-tailscale-setup.md) - Secure remote access

### **Phase 5: Optimization** ⚡
13. [**Performance Tuning**](13-performance.md) - Hardware acceleration and optimization
14. [**Backup Strategy**](14-backup.md) - Protecting your configuration
15. [**Troubleshooting**](15-troubleshooting.md) - Common issues and solutions

---

## ⏱️ **Time Estimates**

- **Phase 1-2** (Foundation + Core): ~2-3 hours
- **Phase 3** (Media Stack): ~2-4 hours  
- **Phase 4** (Monitoring): ~1-2 hours
- **Phase 5** (Optimization): ~1-2 hours

**Total Setup Time**: 6-11 hours (spread over multiple sessions)

---

## 🎯 **Before You Start**

### **📝 Preparation Checklist**
- [ ] Raspberry Pi 5 assembled with case and cooling
- [ ] MicroSD card ready for flashing
- [ ] External USB drive available
- [ ] Ethernet cable connected to router
- [ ] Computer for SSH access ready
- [ ] Router admin access (for finding Pi's IP)

### **🔧 Tools You'll Need**
- **Raspberry Pi Imager** (for OS installation)
- **SSH Client** (Terminal on Mac/Linux, PuTTY on Windows)
- **Text Editor** (for configuration files)
- **Web Browser** (for Portainer and service setup)

### **📚 Accounts to Create**
- **Tailscale Account** (free tier available)
- **Indexer Accounts** (for torrent/usenet indexers)

---

## 🚨 **Important Notes**

### **🔒 Security Considerations**
- This guide prioritizes **functionality over security** for learning purposes
- For production use, consider additional hardening (SSH keys, fail2ban, etc.)
- **Never expose services directly to the internet** - use Tailscale for remote access

### **⚖️ Legal Disclaimer**
- This setup is for **personal use** and **legally owned content** only
- Ensure compliance with your local laws regarding media downloading
- Respect copyright and content creators

### **🆘 Getting Help**
- Each phase includes troubleshooting sections
- Check the [main troubleshooting guide](../troubleshooting/) for common issues
- Join the community discussions for additional support

---

## 🎉 **Ready to Start?**

**Begin with [Phase 1: Operating System Setup](01-operating-system.md)**

> **💡 Pro Tip**: Read through each entire phase before starting - understanding the full process helps avoid common mistakes!

---

*Estimated completion time: 6-11 hours | Difficulty: Intermediate | Support: Community*
