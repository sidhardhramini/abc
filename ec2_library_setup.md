# AWS EC2 Ubuntu – Library Management Application Setup

> **Important:** The supplied repository (`https://github.com/deepthisagar7/library-management.git`) is a console-based Java application that builds a JAR. It is not a browser-based WAR application. The commands below build and run that console application on EC2.

## 1. Update Ubuntu and install prerequisites

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y git openjdk-17-jdk maven
```

## 2. Verify Java and Maven

```bash
java -version
mvn -version
```

## 3. Clone the GitHub repository

```bash
git clone https://github.com/deepthisagar7/library-management.git
cd library-management
```

If the repository is already cloned, update it instead:

```bash
git pull
```

## 4. Build the application

```bash
mvn clean
mvn package
```

Check the generated files:

```bash
ls target/
```

## 5. Run the generated JAR

```bash
java -jar target/library-management-1.0-SNAPSHOT.jar
```

The application should start in the terminal. Follow its on-screen menu.

---

## Optional: Tomcat deployment for a WAR-based web application

Use this section only if your exam provides a **web application that generates a compatible `.war` file**. The console-based repository above does not produce a WAR file.

### Install and start Tomcat 10

```bash
sudo apt install tomcat10 -y
sudo systemctl start tomcat10
sudo systemctl enable tomcat10
sudo systemctl status tomcat10
```

Test Tomcat locally:

```bash
curl http://localhost:8080
```

### Deploy a WAR file

First confirm that the project generated a WAR:

```bash
ls target/*.war
```

Copy the actual WAR file to Tomcat:

```bash
sudo cp target/your-application.war /var/lib/tomcat10/webapps/
sudo systemctl restart tomcat10
```

Replace `your-application.war` with the real filename.

Access it using:

```text
http://YOUR_EC2_PUBLIC_IP:8080/your-application
```

Tomcat 10 requires an application compatible with Jakarta Servlet APIs. An older Java EE application may require Tomcat 9.

---

## AWS Security Group

In AWS Console, open **EC2 → Instances → select your instance → Security → Security groups → Edit inbound rules**.

Add only the rules needed:

| Type | Port | Source |
|---|---:|---|
| SSH | 22 | My IP |
| HTTP | 80 | 0.0.0.0/0, if serving HTTP |
| Custom TCP | 8080 | My IP, for Tomcat testing |

Do not expose SSH (22) or MySQL (3306) to the entire internet. For production, use HTTPS and place Tomcat behind a reverse proxy or load balancer rather than publicly exposing port 8080.

## Database setup (only if the application requires MySQL)

```bash
sudo apt install mysql-server -y
sudo systemctl start mysql
sudo systemctl enable mysql
sudo mysql
```

Inside the MySQL prompt:

```sql
CREATE DATABASE library_db;
CREATE USER 'libraryuser'@'localhost' IDENTIFIED BY 'Choose-A-Strong-Password';
GRANT ALL PRIVILEGES ON library_db.* TO 'libraryuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

The application must also be configured with the matching database URL, database name, username and password. Installing MySQL alone does not connect it to the application.

## Monitoring and troubleshooting commands

```bash
# Tomcat status (for a WAR-based deployment)
sudo systemctl status tomcat10

# Restart Tomcat
sudo systemctl restart tomcat10

# MySQL status
sudo systemctl status mysql

# CPU and processes
top

# Memory
free -h

# Disk space
df -h

# Recent Tomcat service logs
sudo journalctl -u tomcat10 -n 50
```

## Scenario-based revision

1. **EC2 application inaccessible:** Check the public IP, Security Group inbound rules, application status, listening port, firewall and deployment logs.
2. **Restrict SSH:** Limit port 22 to trusted IP addresses and use key-based authentication.
3. **Tomcat stops while EC2 runs:** EC2 remains running, but the web application is unavailable until Tomcat is restarted.
4. **Improve availability:** Use service restart policies, monitoring, backups, health checks and multiple EC2 instances.
5. **ALB and multiple EC2 instances:** An Application Load Balancer distributes traffic across healthy instances and improves fault tolerance and scalability.

## Quick command sequence

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y git openjdk-17-jdk maven
java -version
mvn -version
git clone https://github.com/deepthisagar7/library-management.git
cd library-management
mvn clean
mvn package
ls target/
java -jar target/library-management-1.0-SNAPSHOT.jar
```
