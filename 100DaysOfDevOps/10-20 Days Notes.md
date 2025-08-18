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


---

## Day 12: Linux Network Services

Our monitoring tool has reported an issue in Stratos Datacenter. One of our app servers has an issue, as its Apache service is not reachable on port 5001 (which is the Apache port). The service itself could be down, the firewall could be at fault, or something else could be causing the issue.
Use tools like telnet, netstat, etc. to find and fix the issue. Also make sure Apache is reachable from the jump host without compromising any security settings. Test using `curl http://stapp01:5001` command from jump host.

### Troubleshooting Process

```bash
# Check Apache service logs for errors
journalctl -u httpd

# Check what's using port 5001
sudo netstat -4 -tlnp | grep 5001

# Find the process ID using the port
ps <PID>

# Stop the conflicting service (sendmail in this case)
systemctl stop sendmail

# Check if iptables firewall is present
sudo iptables -L

# If iptables is active, allow port 5001
sudo iptables -I INPUT -p tcp --dport 5001 -j ACCEPT

# Start Apache service
systemctl start httpd

# Verify Apache is running and listening on port 5001
systemctl status httpd
netstat -tlnp | grep 5001

# Test from jump host
curl http://stapp01:5001
```

### What's Happening - Detailed Analysis

**Error Found in Logs:**
```
Aug 18 17:46:05 stapp01.stratos.xfusioncorp.com httpd[492]: (98)Address already in use: AH00072:
make_sock: could not bind to address 0.0.0.0:5001
Aug 18 17:46:05 stapp01.stratos.xfusioncorp.com httpd[492]: no listening sockets available, shutting down
```

**Root Cause Analysis:**
- **Port Conflict**: Another service (sendmail) was already using port 5001
- **Apache Cannot Start**: HTTP daemon cannot bind to port 5001 because it's occupied
- **Service Failure**: Apache shuts down due to inability to create listening sockets

**Troubleshooting Tools Used:**

| Tool | Purpose | What It Shows |
|------|---------|---------------|
| `journalctl -u httpd` | **System logs** for Apache service | Error messages and startup failures |
| `netstat -tlnp` | **Network connections** and listening ports | Which process is using which port |
| `ps <PID>` | **Process information** | Details about specific running processes |
| `iptables -L` | **Firewall rules** | Current firewall configuration |

---
