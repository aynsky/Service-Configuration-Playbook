# Apache Tomcat 10 Configuration Guide 🚀

Complete setup and configuration guide for Apache Tomcat 10, covering development environments (WSL2) to production deployments on Linux servers.

---

## 📋 Table of Contents
- [Overview](#-overview)
- [Prerequisites](#-prerequisites)
- [Quick Start](#-quick-start)
- [Guide Structure](#-guide-structure)
- [Configuration Files](#️-configuration-files)
- [Common Use Cases](#-common-use-cases)
- [Troubleshooting](#-troubleshooting)
- [Security Best Practices](#️-security-best-practices)
- [Migration Path](#-migration-path)

---

## 🎯 Overview

This directory contains comprehensive guides for setting up and configuring Apache Tomcat 10, the latest version of the popular Java servlet container. Our guides cover everything from initial installation to production deployment and servlet development.

### What's Included:
- **Development Setup** - WSL2/RHEL 9 installation and configuration
- **Servlet Development** - Creating and deploying Java servlets
- **Production Deployment** - Real server setup with systemd integration
- **Security Configuration** - Admin users, roles, and access control
- **Best Practices** - Performance tuning and maintenance

### Target Audience:
- Java developers learning servlet development
- System administrators deploying Tomcat servers
- DevOps engineers standardizing Tomcat configurations
- Students and professionals transitioning from development to production

---

## 📋 Prerequisites

### System Requirements:
- **Operating System**: Linux (RHEL 9, CentOS, Ubuntu, Debian)
- **Java**: OpenJDK 11 or later
- **Memory**: Minimum 512MB RAM (2GB+ recommended for production)
- **Disk Space**: 200MB for Tomcat + space for applications

### Knowledge Prerequisites:
- Basic Linux command line skills
- Understanding of Java concepts
- Familiarity with web applications (helpful but not required)
- Basic understanding of HTTP and web servers

### Tools Needed:
- Text editor (nano, vim, or IDE)
- Java Development Kit (JDK)
- Web browser for testing
- curl or wget for downloads

---

## 🚀 Quick Start

### 1. Choose Your Path:

**For Development/Learning:**
```bash
# Start with WSL2 setup
📖 Read: 01-tomcat-setup-wsl2.md
🎯 Goal: Get Tomcat running locally for development
⏱️ Time: 15-30 minutes
```

**For Servlet Development:**
```bash
# After basic setup, create your first servlet
📖 Read: 02-hello-servlet-wsl2.md  
🎯 Goal: Build and deploy a working servlet
⏱️ Time: 30-45 minutes
```

**For Production Deployment:**
```bash
# Deploy on real Linux servers
📖 Read: 03-tomcat-real-server.md
🎯 Goal: Production-ready Tomcat installation
⏱️ Time: 1-2 hours
```

### 2. Verify Installation:
```bash
# Check if Tomcat is running
curl http://localhost:8080

# Should return Tomcat welcome page HTML
```

---

## 📚 Guide Structure

### [01-tomcat-setup-wsl2.md](./01-tomcat-setup-wsl2.md)
**WSL2 Development Setup**
- ✅ Manual Tomcat 10 installation
- ✅ Environment variable configuration
- ✅ Admin user setup with web interface access
- ✅ Common WSL2-specific challenges and solutions
- ✅ Folder structure explanation

**Key Topics:**
- Download and installation from Apache mirrors
- `$CATALINA_HOME` and `$PATH` configuration
- Admin user configuration in `tomcat-users.xml`
- WSL2 limitations (no systemd)
- Bash history expansion issues

### [02-hello-servlet-wsl2.md](./02-hello-servlet-wsl2.md)
**Servlet Development Guide**
- ✅ Web application directory structure
- ✅ Jakarta EE servlet development
- ✅ Compilation with proper classpath
- ✅ `web.xml` configuration
- ✅ Deployment and testing

**Key Topics:**
- Jakarta vs Javax servlet APIs (Tomcat 10+ requirement)
- Proper `WEB-INF` structure
- Servlet compilation with `servlet-api.jar`
- URL mapping and servlet lifecycle
- Common development pitfalls

### [03-tomcat-real-server.md](./03-tomcat-real-server.md)
**Production Server Deployment**
- ✅ System user creation and security
- ✅ Systemd service configuration
- ✅ Production directory structure
- ✅ Security hardening and user management
- ✅ WAR file deployment strategies

**Key Topics:**
- Dedicated `tomcat` system user
- Systemd integration and auto-startup
- Production security best practices
- Multiple deployment strategies
- Monitoring and log management

---

## 🗂️ Configuration Files

### Core Configuration Templates:

#### `tomcat-users.xml` - User Management
```xml
<!-- Admin users with different role levels -->
<tomcat-users>
  <role rolename="manager-gui"/>
  <role rolename="admin-gui"/>
  <user username="admin" password="secure_password" roles="manager-gui,admin-gui"/>
</tomcat-users>
```

#### `systemd` Service Unit
```ini
# Production systemd service configuration
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking
User=tomcat
Environment="CATALINA_HOME=/usr/share/tomcat10"
ExecStart=/usr/share/tomcat10/bin/startup.sh
```

#### `web.xml` - Servlet Mapping
```xml
<!-- Modern servlet configuration -->
<web-app xmlns="https://jakarta.ee/xml/ns/jakartaee" version="5.0">
  <servlet>
    <servlet-name>HelloServlet</servlet-name>
    <servlet-class>HelloServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>HelloServlet</servlet-name>
    <url-pattern>/hello</url-pattern>
  </servlet-mapping>
</web-app>
```

---

## 💼 Common Use Cases

### Development Environment
- **Local testing** of Java web applications
- **Servlet development** and debugging  
- **Learning Jakarta EE** concepts
- **Prototyping** web applications

### Production Deployment
- **Enterprise web applications** hosting
- **Microservices** backend deployment
- **API server** implementation
- **Load-balanced** application clusters

### Educational Purposes
- **Computer science** coursework
- **Java certification** preparation
- **Web development** learning
- **DevOps** skill development

---

## 🔧 Troubleshooting

### Common Issues and Solutions:

#### 🚨 Port 8080 Already in Use
```bash
# Find process using port 8080
sudo netstat -tulpn | grep 8080
# Kill the process or change Tomcat port
```

#### 🚨 Permission Denied Errors
```bash
# Fix ownership and permissions
sudo chown -R tomcat:tomcat /opt/tomcat10
sudo chmod +x /opt/tomcat10/bin/*.sh
```

#### 🚨 Servlet Compilation Fails
```bash
# Verify servlet-api.jar path
ls $CATALINA_HOME/lib/servlet-api.jar
# Use correct classpath in javac command
```

#### 🚨 Manager App Access Denied
```bash
# Check tomcat-users.xml configuration
# Verify remote access valve settings
# Restart Tomcat after configuration changes
```

#### 🚨 Jakarta vs Javax Issues
```java
// Wrong (Tomcat 9 and earlier)
import javax.servlet.*;

// Correct (Tomcat 10+)
import jakarta.servlet.*;
```

### Debug Commands:
```bash
# View Tomcat logs
tail -f $CATALINA_HOME/logs/catalina.out

# Check Tomcat process
ps aux | grep tomcat

# Test connectivity
curl -I http://localhost:8080
```

---

## 🛡️ Security Best Practices

### Essential Security Measures:

#### 1. User Management
- ✅ Use **strong passwords** for admin accounts
- ✅ Implement **role-based access control**
- ✅ Remove or secure **default applications**
- ✅ Regularly **audit user accounts**

#### 2. System Security
- ✅ Run Tomcat as **dedicated system user** (not root)
- ✅ Configure **firewall rules** appropriately
- ✅ Use **reverse proxy** (NGINX/Apache) when possible
- ✅ Enable **security headers** and **SSL/TLS**

#### 3. Application Security
- ✅ Validate and sanitize **user input**
- ✅ Implement proper **session management**
- ✅ Use **parameterized queries** for database access
- ✅ Follow **OWASP security guidelines**

#### 4. Monitoring and Maintenance
- ✅ Monitor **access logs** regularly
- ✅ Set up **log rotation** for disk space management
- ✅ Keep Tomcat and Java **updated**
- ✅ Implement **intrusion detection**

---

## 🔄 Migration Path

### Development to Production Journey:

#### Phase 1: Local Development (WSL2)
1. Follow `01-tomcat-setup-wsl2.md`
2. Set up basic admin access
3. Create simple HTML/JSP pages
4. **Estimated Time:** 1-2 days

#### Phase 2: Servlet Development
1. Follow `02-hello-servlet-wsl2.md`
2. Learn Jakarta servlet APIs
3. Build and test servlets locally
4. **Estimated Time:** 1 week

#### Phase 3: Production Deployment
1. Follow `03-tomcat-real-server.md`
2. Set up production server environment
3. Implement security hardening
4. Deploy and monitor applications
5. **Estimated Time:** 2-3 days

#### Phase 4: Advanced Topics (Future)
- Load balancing and clustering
- Performance tuning and optimization
- Continuous deployment pipelines
- Container orchestration

---

## 📊 Version Compatibility

| Tomcat Version | Java Version | Servlet API | JSP API |
|---------------|-------------|-------------|---------|
| Tomcat 10.1.x | Java 11+ | Jakarta Servlet 6.0 | Jakarta JSP 3.1 |
| Tomcat 10.0.x | Java 8+ | Jakarta Servlet 5.0 | Jakarta JSP 3.0 |
| Tomcat 9.0.x | Java 8+ | Servlet 4.0 | JSP 2.3 |

> ⚠️ **Important:** These guides are specifically for **Tomcat 10.x** which uses **Jakarta EE** namespaces, not the older **Java EE** namespaces.

---

## 🤝 Contributing

### How to Improve These Guides:
1. **Report Issues** - Found something unclear or incorrect?
2. **Suggest Improvements** - Better ways to explain concepts?
3. **Add Examples** - More servlet examples or use cases?
4. **Update Content** - New Tomcat versions or features?

### Contribution Areas:
- Additional servlet examples and patterns
- Performance tuning guides
- Security hardening procedures
- Docker and containerization examples
- Integration with popular frameworks

---

## 📚 Additional Resources

### Official Documentation:
- [Apache Tomcat 10 Documentation](https://tomcat.apache.org/tomcat-10.1-doc/)
- [Jakarta EE Specifications](https://jakarta.ee/specifications/)
- [Java Servlet Tutorial](https://docs.oracle.com/javaee/7/tutorial/servlets.htm)

### Community Resources:
- [Apache Tomcat Users Mailing List](https://tomcat.apache.org/lists.html)
- [Stack Overflow Tomcat Tag](https://stackoverflow.com/questions/tagged/tomcat)
- [Tomcat Security Guide](https://tomcat.apache.org/tomcat-10.1-doc/security-howto.html)

---

## 📝 Notes

- These guides are **actively maintained** and tested on RHEL 9/CentOS
- All configurations prioritize **security and best practices**
- Examples are **production-ready** with proper error handling
- Guides assume **basic Linux knowledge** but provide detailed explanations

---

*Happy coding! 🚀 If these guides helped you, please consider starring the repository and sharing with others.*