# 🐳 Phase 4: Docker Installation

Installing Docker and Docker Compose for container management.

## 🎯 **What We'll Accomplish**

- **📦 Install Docker Engine** using the convenience script
- **🔧 Configure Docker** for the media user
- **📋 Install Docker Compose** for multi-container applications
- **✅ Verify installation** with test containers
- **⚙️ Configure Docker** for optimal performance

---

## 🤔 **Why Docker for Media Server?**

### **✅ Advantages**
- **🔄 Easy Updates**: Update services without system-wide changes
- **🛡️ Isolation**: Services can't interfere with each other
- **📦 Consistency**: Same environment across different systems
- **🔧 Easy Management**: Start, stop, restart services independently
- **💾 Backup Friendly**: Configuration and data clearly separated
- **🚀 Quick Deployment**: Spin up new services in minutes

### **🐳 Docker vs Alternatives**
- **vs Native Installation**: No dependency conflicts, easier updates
- **vs Snap Packages**: Better performance, more control
- **vs Virtual Machines**: Lower resource usage, faster startup

---

## 📦 **Docker Installation**

### **Step 1: System Preparation**

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install prerequisites
sudo apt install -y curl ca-certificates gnupg lsb-release

# Remove any old Docker installations
sudo apt remove -y docker docker-engine docker.io containerd runc
```

### **Step 2: Install Docker Using Convenience Script**

The convenience script is the easiest method for Ubuntu Server:

```bash
# Download and run Docker installation script
curl -fsSL https://get.docker.com -o get-docker.sh

# Review the script (optional but recommended)
cat get-docker.sh

# Run the installation script
sudo sh get-docker.sh

# Clean up
rm get-docker.sh
```

### **Step 3: Configure Docker for Media User**

```bash
# Add media user to docker group
sudo usermod -aG docker media

# Apply group changes (logout and login, or use newgrp)
newgrp docker

# Verify group membership
groups media
```

### **Step 4: Start and Enable Docker**

```bash
# Start Docker service
sudo systemctl start docker

# Enable Docker to start on boot
sudo systemctl enable docker

# Check Docker status
sudo systemctl status docker
```

---

## 📋 **Docker Compose Installation**

Docker Compose is included with Docker Desktop, but we need to install it separately on Ubuntu Server:

### **Step 1: Install Docker Compose**

```bash
# Install Docker Compose plugin (recommended method)
sudo apt install -y docker-compose-plugin

# Verify installation
docker compose version
```

### **Alternative: Standalone Installation**

If the plugin method doesn't work:

```bash
# Download latest Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Make executable
sudo chmod +x /usr/local/bin/docker-compose

# Create symlink for easier access
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

# Verify installation
docker-compose --version
```

---

## ✅ **Verify Docker Installation**

### **Step 1: Test Docker Without Sudo**

```bash
# Test Docker command (should work without sudo)
docker --version

# Check Docker info
docker info

# List running containers (should be empty)
docker ps
```

### **Step 2: Run Test Container**

```bash
# Run hello-world container
docker run hello-world

# You should see a success message
```

### **Step 3: Test Docker Compose**

```bash
# Test Docker Compose
docker compose version

# Alternative command format
docker-compose --version
```

---

## ⚙️ **Docker Configuration Optimization**

### **Step 1: Configure Docker Daemon**

Create Docker daemon configuration:

```bash
sudo nano /etc/docker/daemon.json
```

Add this configuration:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "storage-driver": "overlay2",
  "default-address-pools": [
    {
      "base": "172.17.0.0/16",
      "size": 24
    }
  ]
}
```

### **Step 2: Apply Configuration**

```bash
# Restart Docker to apply changes
sudo systemctl restart docker

# Verify configuration
docker info | grep -E "Storage Driver|Logging Driver"
```

### **Step 3: Configure Docker Networks**

Create a custom network for our media services:

```bash
# Create custom bridge network
docker network create mediacenter_default

# List networks
docker network ls

# Inspect our network
docker network inspect mediacenter_default
```

---

## 📁 **Docker Directory Structure**

### **Step 1: Organize Docker Files**

```bash
# Create compose files directory
mkdir -p /home/media/docker-compose

# Create individual service directories (already done in previous phase)
ls -la /home/media/docker/

# Create shared volumes directory
mkdir -p /home/media/docker/shared
```

### **Step 2: Set Up Environment Variables**

Update the `.env` file we created earlier:

```bash
nano /home/media/.env
```

Add Docker-specific variables:

```bash
# User and Group IDs
PUID=1000
PGID=1000

# Timezone
TZ=America/New_York  # Change to your timezone

# Docker network
DOCKER_NETWORK=mediacenter_default

# Paths
DOCKER_DIR=/home/media/docker
MEDIA_DIR=/mnt/media

# API Keys (we'll fill these later)
RADARR_API_KEY=
SONARR_API_KEY=
PROWLARR_API_KEY=
JELLYFIN_API_KEY=
JELLYSTAT_API_KEY=

# Passwords (we'll set these later)
CHANGEME_PIHOLE_PASSWORD=
CHANGEME_FILEBROWSER_PASSWORD=
CHANGEME_CODE_SERVER_PASSWORD=

# Network settings
TS_HOST=
TAILSCALE_API_KEY=
```

---

## 🧪 **Test Docker Setup with Sample Service**

### **Step 1: Create Test Compose File**

```bash
nano /home/media/docker-compose/test.yml
```

Add this test configuration:

```yaml
version: "3.9"
services:
  nginx-test:
    image: nginx:alpine
    container_name: nginx-test
    ports:
      - "8080:80"
    volumes:
      - /home/media/docker/shared:/usr/share/nginx/html:ro
    networks:
      - mediacenter_default
    restart: unless-stopped

networks:
  mediacenter_default:
    external: true
```

### **Step 2: Create Test Content**

```bash
# Create test HTML file
echo "<h1>Docker is working!</h1><p>Media server setup in progress...</p>" > /home/media/docker/shared/index.html
```

### **Step 3: Run Test Service**

```bash
# Navigate to compose directory
cd /home/media/docker-compose

# Start test service
docker compose -f test.yml up -d

# Check if container is running
docker ps

# Test web access
curl http://localhost:8080
```

You should see the HTML content we created.

### **Step 4: Clean Up Test**

```bash
# Stop and remove test service
docker compose -f test.yml down

# Remove test files
rm /home/media/docker-compose/test.yml
rm /home/media/docker/shared/index.html
```

---

## 📊 **Docker Monitoring and Management**

### **Useful Docker Commands**

```bash
# Container management
docker ps                    # List running containers
docker ps -a                 # List all containers
docker logs container_name   # View container logs
docker exec -it container_name bash  # Access container shell

# Image management
docker images               # List images
docker pull image_name      # Download image
docker rmi image_name       # Remove image

# System management
docker system df            # Show disk usage
docker system prune         # Clean up unused resources
docker volume ls            # List volumes
docker network ls           # List networks

# Docker Compose commands
docker compose up -d        # Start services in background
docker compose down         # Stop and remove services
docker compose logs         # View logs
docker compose pull         # Update images
```

### **Docker System Information**

```bash
# Check Docker version
docker --version
docker compose version

# Check system resources
docker system df

# Check running processes
docker stats

# View Docker info
docker info
```

---

## ✅ **Verification Checklist**

Before proceeding, verify:

- [ ] **Docker Installed**: `docker --version` works
- [ ] **Docker Compose**: `docker compose version` works
- [ ] **User Permissions**: Can run `docker ps` without sudo
- [ ] **Service Status**: Docker service is active and enabled
- [ ] **Network Created**: `mediacenter_default` network exists
- [ ] **Test Container**: Successfully ran and accessed nginx test
- [ ] **Environment File**: `.env` file configured with basic variables

### **Quick Verification Commands**

```bash
# Check everything is working
docker --version && docker compose version
docker ps
docker network ls | grep mediacenter
docker info | grep -E "Storage Driver|Server Version"
cat /home/media/.env | grep -E "PUID|PGID|TZ"
```

---

## 🆘 **Troubleshooting**

### **🚫 Permission Denied Errors**

```bash
# Check if user is in docker group
groups media

# If not in docker group, add and relogin
sudo usermod -aG docker media
# Then logout and login again, or:
newgrp docker
```

### **🐳 Docker Service Won't Start**

```bash
# Check Docker service status
sudo systemctl status docker

# Check Docker logs
sudo journalctl -u docker -f

# Restart Docker service
sudo systemctl restart docker

# Check for conflicting services
sudo netstat -tlnp | grep :80
```

### **📋 Docker Compose Not Found**

```bash
# Check if plugin is installed
docker compose version

# If not working, try standalone installation
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### **🌐 Network Issues**

```bash
# Recreate the network
docker network rm mediacenter_default
docker network create mediacenter_default

# Check network configuration
docker network inspect mediacenter_default
```

### **💾 Storage Issues**

```bash
# Check disk space
df -h

# Clean up Docker system
docker system prune -a

# Check Docker root directory
sudo du -sh /var/lib/docker/
```

---

## 🎯 **Next Steps**

✅ **Fantastic!** Docker is now installed and configured for your media server.

**Continue to**: [Phase 5: Portainer Deployment](05-portainer-setup.md)

---

*Phase 4 Complete | Next: Portainer Setup | Estimated time: 30-45 minutes*
