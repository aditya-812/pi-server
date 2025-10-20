# 🐳 Phase 5: Portainer Deployment

Installing Portainer for easy Docker container management through a web interface.

## 🎯 **What We'll Accomplish**

- **📦 Deploy Portainer** using Docker Compose
- **🔐 Set up admin account** and secure access
- **🌐 Access web interface** and explore features
- **📋 Prepare for stack deployment** of our media services
- **⚙️ Configure Portainer** for optimal use

---

## 🤔 **Why Portainer?**

### **✅ Advantages**
- **🖥️ Web Interface**: Manage Docker without command line
- **📊 Visual Monitoring**: See container status, logs, and stats
- **📋 Stack Management**: Deploy multi-container applications easily
- **👥 User Management**: Multiple users with different permissions
- **📱 Mobile Friendly**: Access from any device with a browser
- **🔧 Easy Updates**: Update containers with a few clicks

### **🆚 Alternatives**
- **Portainer vs Command Line**: More user-friendly, visual feedback
- **Portainer vs Docker Desktop**: Lighter, server-focused
- **Portainer vs Yacht**: More mature, better documentation

---

## 📦 **Portainer Installation**

### **Step 1: Create Portainer Directory**

```bash
# Create Portainer configuration directory
mkdir -p /home/media/docker/portainer

# Create docker-compose file
nano /home/media/docker/portainer/docker-compose.yml
```

### **Step 2: Portainer Docker Compose Configuration**

Add this configuration to the docker-compose.yml file:

```yaml
version: "3.9"

services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: unless-stopped
    ports:
      - "9000:9000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - portainer_data:/data
    networks:
      - mediacenter_default
    command: --admin-password-file /tmp/portainer_password
    environment:
      - TZ=${TZ:-UTC}
    labels:
      - "com.centurylinklabs.watchtower.enable=true"

volumes:
  portainer_data:
    driver: local

networks:
  mediacenter_default:
    external: true
```

### **Step 3: Set Admin Password**

```bash
# Create a secure admin password
echo "your-secure-admin-password" | sudo tee /tmp/portainer_password

# Secure the password file
sudo chmod 600 /tmp/portainer_password
sudo chown root:root /tmp/portainer_password
```

**⚠️ Important**: Replace `your-secure-admin-password` with a strong password you'll remember!

### **Step 4: Deploy Portainer**

```bash
# Navigate to Portainer directory
cd /home/media/docker/portainer

# Create .env file with timezone
echo "TZ=$(timedatectl show --property=Timezone --value)" > .env

# Deploy Portainer
docker compose up -d

# Check if container is running
docker ps | grep portainer

# Check logs
docker logs portainer
```

---

## 🌐 **Initial Portainer Setup**

### **Step 1: Access Portainer Web Interface**

1. **Open your web browser**
2. **Navigate to**: `http://YOUR_PI_IP:9000`
3. **You should see the Portainer login screen**

### **Step 2: First Login**

1. **Username**: `admin`
2. **Password**: The password you set in `/tmp/portainer_password`
3. **Click** "Login"

### **Step 3: Environment Setup**

After first login, Portainer will ask you to choose an environment:

1. **Select** "Get Started" 
2. **Choose** "Docker" (should be auto-detected)
3. **Click** "Connect"

You should now see the Portainer dashboard with your local Docker environment.

### **Step 4: Explore the Dashboard**

Take a moment to explore:
- **📊 Dashboard**: Overview of containers, images, volumes
- **📦 Containers**: List of running containers (you should see Portainer itself)
- **🖼️ Images**: Downloaded Docker images
- **📋 Stacks**: Multi-container applications (empty for now)
- **🌐 Networks**: Docker networks (you should see mediacenter_default)

---

## 🔧 **Portainer Configuration**

### **Step 1: Configure Settings**

1. **Click** on "Settings" in the left sidebar
2. **Update these settings**:
   - **App Templates**: Enable "Use external templates"
   - **Edge Compute**: Disable (not needed for local setup)
   - **Analytics**: Disable for privacy

### **Step 2: Set Up Environment Variables**

We'll configure global environment variables for our stacks:

1. **Go to** "Stacks" in the sidebar
2. **Note**: We'll use environment variables when deploying stacks

### **Step 3: Security Settings** (Optional)

For additional security:

1. **Go to** "Settings" → "Authentication"
2. **Enable** "Hide labels" (optional)
3. **Set** "Session timeout" to reasonable value (e.g., 8 hours)

---

## 📋 **Prepare for Media Stack Deployment**

### **Step 1: Download Stack Configurations**

We'll use the stack configurations from this repository:

```bash
# Create directory for stack files
mkdir -p /home/media/stacks

# We'll copy our stack files here when ready
# For now, just create the directory structure
```

### **Step 2: Prepare Environment Variables**

Update your `.env` file with all required variables:

```bash
nano /home/media/.env
```

Ensure it contains all these variables (we'll fill in the API keys later):

```bash
# System Configuration
PUID=1000
PGID=1000
TZ=America/New_York  # Your timezone

# Network Configuration
DOCKER_NETWORK=mediacenter_default

# API Keys (to be filled later)
RADARR_API_KEY=
SONARR_API_KEY=
PROWLARR_API_KEY=
JELLYFIN_API_KEY=
JELLYSTAT_API_KEY=

# Service Passwords (set these now)
CHANGEME_PIHOLE_PASSWORD=your-pihole-password
CHANGEME_FILEBROWSER_PASSWORD=your-filebrowser-password
CHANGEME_CODE_SERVER_PASSWORD=your-code-server-password
JELLYSTAT_DB_PASSWORD=your-jellystat-db-password
JELLYSTAT_JWT_SECRET=your-jwt-secret-key

# Tailscale (to be configured later)
TS_HOST=
TAILSCALE_API_KEY=

# Jellyfin
JELLYFIN_PUBLISHED_URL=http://YOUR_PI_IP:8096
```

### **Step 3: Test Stack Deployment**

Let's test Portainer's stack functionality with a simple example:

1. **In Portainer**, go to "Stacks"
2. **Click** "Add stack"
3. **Name**: `test-stack`
4. **Build method**: "Web editor"
5. **Paste this test configuration**:

```yaml
version: "3.9"
services:
  whoami:
    image: traefik/whoami
    container_name: whoami-test
    ports:
      - "8080:80"
    networks:
      - mediacenter_default

networks:
  mediacenter_default:
    external: true
```

6. **Click** "Deploy the stack"
7. **Test**: Visit `http://YOUR_PI_IP:8080`
8. **Clean up**: Delete the stack after testing

---

## 🔍 **Portainer Management Tips**

### **Container Management**

- **📊 View Logs**: Click container → "Logs" tab
- **📈 Monitor Stats**: Click container → "Stats" tab  
- **🔧 Execute Commands**: Click container → "Console" tab
- **🔄 Restart Services**: Click container → "Restart" button

### **Stack Management**

- **📋 Deploy Stacks**: Use "Stacks" → "Add stack"
- **🔄 Update Stacks**: Edit stack and redeploy
- **📊 Monitor Stack**: View all containers in a stack
- **🗑️ Remove Stacks**: Cleanly removes all containers and networks

### **Image Management**

- **📥 Pull Images**: "Images" → "Pull image"
- **🗑️ Clean Up**: "Images" → Select unused → "Remove"
- **🔍 Inspect Images**: Click image for details

---

## 🛡️ **Security Considerations**

### **Network Security**

Portainer is accessible on port 9000. For security:

1. **Firewall**: Only allow access from your local network
2. **Strong Password**: Use a complex admin password
3. **Regular Updates**: Keep Portainer updated
4. **HTTPS**: Consider setting up SSL/TLS for production use

### **Access Control**

For multi-user environments:

1. **Create Users**: "Users" → "Add user"
2. **Set Roles**: Assign appropriate permissions
3. **Team Management**: Organize users into teams
4. **Environment Access**: Control which environments users can access

---

## ✅ **Verification Checklist**

Before proceeding, verify:

- [ ] **Portainer Running**: Container shows as "running" in `docker ps`
- [ ] **Web Access**: Can access Portainer at `http://YOUR_PI_IP:9000`
- [ ] **Admin Login**: Can log in with admin credentials
- [ ] **Environment Connected**: Local Docker environment is visible
- [ ] **Network Visible**: `mediacenter_default` network appears in Networks
- [ ] **Test Stack**: Successfully deployed and removed test stack
- [ ] **Environment File**: `.env` file updated with all required variables

### **Quick Verification Commands**

```bash
# Check Portainer is running
docker ps | grep portainer

# Check Portainer logs
docker logs portainer | tail -10

# Test web access
curl -I http://localhost:9000

# Check environment file
cat /home/media/.env | grep -E "PUID|PGID|TZ"
```

---

## 🆘 **Troubleshooting**

### **🚫 Can't Access Portainer Web Interface**

```bash
# Check if Portainer is running
docker ps | grep portainer

# Check Portainer logs
docker logs portainer

# Check if port 9000 is accessible
sudo netstat -tlnp | grep :9000

# Restart Portainer
cd /home/media/docker/portainer
docker compose restart
```

### **🔐 Admin Password Issues**

```bash
# Reset admin password
docker stop portainer
sudo rm /tmp/portainer_password
echo "new-password" | sudo tee /tmp/portainer_password
sudo chmod 600 /tmp/portainer_password
docker start portainer
```

### **🌐 Network Connection Issues**

```bash
# Check if mediacenter_default network exists
docker network ls | grep mediacenter

# Recreate network if needed
docker network rm mediacenter_default
docker network create mediacenter_default

# Restart Portainer
docker compose restart
```

### **📦 Stack Deployment Failures**

1. **Check YAML syntax** - Use a YAML validator
2. **Verify network names** - Ensure networks exist
3. **Check environment variables** - Verify all required variables are set
4. **Review logs** - Check container logs for specific errors

### **🔄 Portainer Update Issues**

```bash
# Update Portainer
cd /home/media/docker/portainer
docker compose pull
docker compose up -d

# Check if update was successful
docker logs portainer | tail -5
```

---

## 🎯 **Next Steps**

✅ **Perfect!** Portainer is now running and ready to manage your Docker containers.

**Continue to**: [Phase 6: Storage Configuration](06-storage-setup.md)

---

*Phase 5 Complete | Next: Storage Setup | Estimated time: 20-30 minutes*
