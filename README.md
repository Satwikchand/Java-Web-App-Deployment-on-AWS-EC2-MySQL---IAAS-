🚀 Java Web App Deployment on AWS EC2 + MySQL
Step 1: Understand the Architecture
App Server (app-vm) → Runs Tomcat + Java App
DB Server (db-vm) → Runs MySQL

Flow:
Browser → App VM (Port 8080) → DB VM (Port 3306)

Step 2: Launch EC2 Instances
🔹 2.1 Create DB VM
Name: db-vm
Security Group:
SSH (22) → Your IP
MySQL (3306) → App VM Security Group
🔹 2.2 Create App VM
Name: app-vm
Security Group:
SSH (22) → Your IP
HTTP (8080) → Anywhere (0.0.0.0/0)
Step 3: Configure DB VM (MySQL Server)
🔹 3.1 Install MySQL
sudo apt update -y
sudo apt install mysql-server -y
sudo systemctl start mysql
sudo systemctl enable mysql
🔹 3.2 Login to MySQL
sudo mysql
🔹 3.3 Create Database
CREATE DATABASE jet;
USE jet;
🔹 3.4 Create Table
CREATE TABLE USER (
id INT AUTO_INCREMENT PRIMARY KEY,
first_name VARCHAR(50),
last_name VARCHAR(50),
email VARCHAR(100),
username VARCHAR(50),
password VARCHAR(50),
regdate DATE
);
🔹 3.5 Create DB User
CREATE USER 'appuser'@'%' IDENTIFIED BY 'YourPassword123!';
GRANT ALL PRIVILEGES ON jet.* TO 'appuser'@'%';
FLUSH PRIVILEGES;
🔹 3.6 Enable Remote Access

Edit MySQL config:

sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf

Change:

bind-address = 127.0.0.1

To:

bind-address = 0.0.0.0
🔹 3.7 Restart MySQL
sudo systemctl restart mysql
🔹 3.8 Verify MySQL Port
sudo ss -tlnp | grep 3306
Step 4: Configure App VM (Tomcat + Java App)
🔹 4.1 Install Java & Maven
sudo apt update -y
sudo apt install openjdk-11-jdk -y
sudo apt install maven -y
🔹 4.2 Install Tomcat
cd /opt
wget https://downloads.apache.org/tomcat/tomcat-9/v9.0.91/bin/apache-tomcat-9.0.91.tar.gz
tar -xvf apache-tomcat-9.0.91.tar.gz
mv apache-tomcat-9.0.91 tomcat9
chmod +x /opt/tomcat9/bin/*.sh
🔹 4.3 Fix Permissions
sudo chown -R ubuntu:ubuntu /opt/tomcat9
🔹 4.4 Start Tomcat
sudo /opt/tomcat9/bin/startup.sh
🔹 4.5 Clone Project
git clone https://github.com/Akiranred/aws-rds-java.git
cd aws-rds-java
🔹 4.6 Update DB Connection

Edit your JSP/Java file:

Class.forName("com.mysql.cj.jdbc.Driver");

Connection con = DriverManager.getConnection(
"jdbc:mysql://<DB-PRIVATE-IP>:3306/jet?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC",
"appuser",
"YourPassword123!"
);

👉 Replace <DB-PRIVATE-IP> with your DB VM private IP.

🔹 4.7 Build Project
mvn clean package
🔹 4.8 Deploy WAR File
sudo /opt/tomcat9/bin/shutdown.sh
sudo rm -rf /opt/tomcat9/webapps/LoginWebApp*
sudo cp target/LoginWebApp.war /opt/tomcat9/webapps/
sudo /opt/tomcat9/bin/startup.sh
Step 5: Test Application

Open in browser:

http://<APP-PUBLIC-IP>:8080/LoginWebApp
Step 6: Verify Database Entries

Run on DB VM:

mysql -u appuser -p -e "SELECT * FROM jet.USER;"
