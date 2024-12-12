### Steps:

## 1. Create EC2 Instance:

* Login into server and install jenkins for Amazon linux:
    
```sh
#! /bin/bash

sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io-2023.key
sudo yum upgrade -y
# Add required dependencies for the jenkins package
    sudo yum install java-17-amazon-corretto -y
sudo yum install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```
* Give permission to exectue the above file and run it.

```sh
sudo chmod +x jenkins.sh

# run the script file
./jenkins.sh
```


## 2. Install sonarqube

```sh
#!/bin/bash

# Update system packages
echo "Updating system packages..."
sudo yum update -y

# Add PostgreSQL repository
echo "Configuring PostgreSQL repository..."
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-$(rpm -E %{rhel})-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo yum install -y epel-release

# Install PostgreSQL
echo "Installing PostgreSQL..."
sudo yum install -y postgresql postgresql-server postgresql-contrib

# Initialize PostgreSQL database
echo "Initializing PostgreSQL database..."
sudo postgresql-setup --initdb

# Enable and start PostgreSQL service
echo "Enabling and starting PostgreSQL service..."
sudo systemctl enable postgresql
sudo systemctl start postgresql

# Set password for 'postgres' user
echo "Setting password for postgres user..."
echo "Enter a password for the 'postgres' user:"
sudo passwd postgres

# Switch to postgres user and perform database setup
su - postgres <<EOF
exit
EOF

# Install zip utility
echo "Installing zip utility..."
sudo yum install -y zip

# Download and install SonarQube
echo "Downloading and installing SonarQube..."
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.6.1.59531.zip
sudo unzip sonarqube-9.6.1.59531.zip
sudo mv sonarqube-9.6.1.59531 sonarqube
sudo mv sonarqube /opt/

# Create SonarQube user and group
echo "Creating SonarQube user and group..."
sudo groupadd sonar
sudo useradd -d /opt/sonarqube -g sonar sonar
sudo chown sonar:sonar /opt/sonarqube -R

# Configure SonarQube
echo "Configuring SonarQube..."
sudo touch /opt/sonarqube/conf/sonar.properties
echo "
sonar.jdbc.username=admin
sonar.jdbc.password=admin123 
" | sudo tee /opt/sonarqube/conf/sonar.properties

# Update SonarQube startup script
echo "Updating SonarQube startup script..."
sudo touch /opt/sonarqube/bin/linux-x86-64/sonar.sh
echo "
RUN_AS_USER="sonar"
" | sudo tee /opt/sonarqube/bin/linux-x86-64/sonar.sh

# Create systemd service for SonarQube
echo "Creating SonarQube systemd service..."
sudo touch /etc/systemd/system/sonarqube.service
echo "
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=sonar
Group=sonar
Restart=always

LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
" | sudo tee /etc/systemd/system/sonarqube.service

# Enable and start SonarQube service
echo "Enabling and starting SonarQube service..."
sudo systemctl enable sonarqube
sudo systemctl start sonarqube
sudo systemctl status sonarqube

# Install Maven
echo "Installing Maven..."
sudo yum install -y maven
mvn --version

# Install Java
echo "Installing OpenJDK 11..."
sudo yum install -y java-11-openjdk-devel

# Configure Java alternatives
echo "Configuring Java alternatives..."
sudo alternatives --config java
java --version

# Verify SonarQube service status
echo "Checking SonarQube service status..."
systemctl status sonarqube

# Final message
echo "Installation and configuration completed successfully."
```

* Give permission to exectue the above file and run it.

```sh
sudo chmod +x sonar.sh

# run the script file
./sonar.sh
```





2. Click -> New item

3. click project → pipeline project

4. Go to → configure

5. Pipeline → Select -> Pipeline Script -> select ‘hello world’
(*For sample code purpose we select hello world*) (* but actual way it looks like below image which we need to write our own code, so follow below steps for writing script*)

6. 