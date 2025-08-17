## Day 11: Install and Configure Tomcat Server

1. Install Tomcat on App Server 2
2. Configure it to run on port 5002
3. Deploy ROOT.war file from jump host at location `/tmp`

Verify webpage works on base URL: `curl http://stapp02:5002`

```bash
# Copy the ROOT.war file from Jump server to App Server 2
scp /tmp/ROOT.war username@stapp02:/tmp

# On App Server 2
# Install Tomcat package
yum install -y tomcat

# Edit Tomcat configuration file to change port
vi /usr/share/tomcat/conf/server.xml
# Change: <Connector port="5002" protocol="HTTP/1.1"

# Copy WAR file to Tomcat webapps directory
cp /tmp/ROOT.war /usr/share/tomcat/webapps

# Install and enable firewall
sudo yum install firewalld -y       
sudo systemctl enable --now firewalld

# Open port 5002 for HTTP traffic
sudo firewall-cmd --zone=public --add-port=5002/tcp --permanent
sudo firewall-cmd --reload

# Start Tomcat service
systemctl start tomcat

# Check service status
systemctl status tomcat

# Test the deployment
curl http://stapp02:5002

# Verify Tomcat is listening on port 5002
netstat -tlnp | grep 5002
```

**What is `server.xml`?**
- **Main configuration file** for Apache Tomcat
- Defines server components like connectors, hosts, and contexts
- Contains port configurations, SSL settings, and virtual host definitions
- Located in `/usr/share/tomcat/conf/` directory

**What is `/usr/share/tomcat/webapps`?**
- **Deployment directory** for Tomcat web applications
- Tomcat automatically deploys WAR files placed here
- `ROOT.war` becomes the default application accessible at the base URL
- Other WAR files create subdirectories (e.g., `app.war` → `/app` context)

### Key Tomcat Directories:

| Directory | Purpose |
|-----------|---------|
| `/usr/share/tomcat/conf/` | **Configuration files** (server.xml, web.xml, etc.) |
| `/usr/share/tomcat/webapps/` | **Web applications** deployment directory |
| `/usr/share/tomcat/logs/` | **Log files** for troubleshooting |
| `/usr/share/tomcat/work/` | **Temporary files** and compiled JSPs |