# SonarQube_Setup_Guide_Linux


# Step 1 - Installing OpenJDK

[See requirements](https://docs.sonarsource.com/sonarqube/latest/requirements/prerequisites-and-overview/#supported-platforms)

Distro info:
```bash
cat /etc/os-release
```

To install OpenJDK, first refresh your server’s local package index:
```bash
sudo apt update
```

Install `openjdk-17-jre` Java Runtime Environment:
```bash
sudo apt install openjdk-17-jre
```

Check the version of Java installed on the system to verify the successful installation of OpenJDK:
```bash
java -version
```

To configure the default Java version. If multiple Java versions are installed on the system, this command allows you to select the default one:
```bash
sudo update-alternatives --config java
```

# Step 2 — Installing PostgreSQL

To install the Postgres package:
```bash
sudo apt install postgresql-14
```

Check it it's running on the background:
```bash
sudo systemctl status postgresql
```

To enable PostgreSQL to start at boot:
```bash
sudo systemctl enable postgresql
```

To start PostgreSQL service:
```bash
sudo systemctl start postgresql
```

To reboot:
```bash
reboot
```

This command switches the user context to the `postgres` user. The `-i` option opens a new shell session (login) with the `-u` user `postgres` user's environment:
```bash
sudo -i -u postgres
```

Open the PostgreSQL command-line interface (CLI), to interact with the PostgreSQL database using SQL queries:
```bash
psql
```

The series of SQL commands create a new PostgreSQL user, a database, and assign privileges to the user for that database. This sets up the necessary database configuration for SonarQube:
```sql
CREATE USER sonar WITH PASSWORD 'sonar123';
CREATE DATABASE sonarqube OWNER sonar;
GRANT ALL PRIVILEGES ON DATABASE sonarqube TO sonar;
```

Lists all databases:
```sql
\l
```

Quits the PostgreSQL CLI and returns to the system shell:
```sql
\q
```

To exits the PostgreSQL user's shell session and returns to the original user's context:
```bash
exit
```

Check Postgres port number:
```bash
sudo netstat -plunt | grep LISTEN
```
Or:
```bash
sudo cat /etc/postgresql/14/main/postgresql.conf | grep port
```

# Step 3 - Configuring Kernel Parameters

[Reference](https://docs.sonarsource.com/sonarqube/latest/requirements/prerequisites-and-overview/#platform-notes)

To open the system control configuration file to edit kernel parameters:
```bash
sudo nano /etc/sysctl.conf
```

These lines are added to the end of the `/etc/sysctl.conf` file. They specify two important kernel parameters:
- `vm.max_map_count` is set to `524288`, which is a requirement for SonarQube. This parameter controls the maximum number of memory map areas that a process may have.
- `fs.file-max` is set to `131072`, another requirement for SonarQube. It determines the maximum number of file handles the system can allocate.
```bash
vm.max_map_count=524288
fs.file-max=131072
```

Now, run the sysctl command below to apply new changes on the '/etc/sysctl.conf' file:
```bash
sudo sysctl --system
```

# Step 4 - Configuring SonarQube

[Dowload SonarQube here](https://www.sonarsource.com/products/sonarqube/downloads/)

Download SonarQube from the SonarQube distribution archive:
```bash
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.2.77730.zip
```

Extract SonarQube zip file:
```bash
sudo unzip sonarqube-9.9.2.77730.zip
```

Move the extracted SonarQube files to the `/opt/sonarqube` directory, where it will be installed:
```bash
sudo mv sonarqube-9.9.2.77730 /opt/sonarqube
```

```bash
sudo useradd -d /opt/sonarqube sonar
```

```bash
sudo chown sonar /opt/sonarqube -R
```

This file is now required for SonarQube to run as a service:
```bash
sudo nano /opt/sonarqube/conf/sonar.properties
```

These configurations must be added to this file:
```bash
sonar.jdbc.username=sonar
sonar.jdbc.password=sonar123
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
```
[Reference](https://docs.sonarsource.com/sonarqube/latest/requirements/prerequisites-and-overview/#platform-notes)

This file is now required for SonarQube to run as a service:
```bash
sudo nano /etc/systemd/system/sonar.service
```

This unit file defines how SonarQube should be managed as a service, including start, stop, and restart actions:
```bash
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
```
[Reference]([Reference](https://docs.sonarsource.com/sonarqube/latest/requirements/prerequisites-and-overview/#platform-notes))

To check the status of the SonarQube service:
```bash
sudo systemctl status sonar
```

To command starts the SonarQube service.
```bash
sudo systemctl start sonar
```

To enable the SonarQube service to start automatically at boot:
```bash
sudo systemctl enable sonar
```

# Step 5 - Downloading and Configuring SonarScanner

[Download SonarScanner here](https://docs.sonarsource.com/sonarqube/latest/analyzing-source-code/scanners/sonarscanner/)

Download the SonarScanner CLI:
```bash
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli-5.0.1.3006-linux.zip
```

Extract the downloaded SonarScanner archive:
```bash
unzip sonar-scanner-cli-5.0.1.3006-linux.zip
```

Move the extracted SonarScanner files to the `/opt/sonar-scanner` directory.
```bash
sudo mv sonar-scanner-cli-5.0.1.3006-linux /opt/sonar-scanner
```

We need to add the sonar-scanner command to the PATH variable. Let’s create a file to automate the required environment variables configuration:
```bash
sudo nano /etc/profile.d/sonar-scanner.sh
```

Add below line of code in the file:
```bash
#!/bin/bash
export PATH="$PATH:/opt/sonar-scanner/bin"
```

Use the source command to add the sonar scanner command to the PATH variable:
```bash
source /etc/profile.d/sonar-scanner.sh
```

Check the variable set for the scanner with below command:
```bash
echo $PATH
```

To check the version of SonarScanner run below line of code:
```bash
sonar-scanner -v
```

# Step 6 - Project Setup and run scanner

[Project link](https://github.com/Mohamed-Boutarbouch/php_store_app)

For the first time, you can scan project 2 ways either using the command prompt directly or using the properties file setup:

## 1- Using Command prompt:
Traverse to your project directory for which you want to run scan. In root of the directory run the below command and replace the detail which you have setup and got from the SonarQube project setup. Here replace the **projectKey** and **sonar.login** value with your detail:
```bash
sonar-scanner \
 -Dsonar.projectKey=next-js-app \
 -Dsonar.sources=. \
 -Dsonar.host.url=http://localhost:9000 \
 -Dsonar.login=sqp_e4e8b003c002843bf602b4b679febdc6fa0de981
```

## 2- Properties File setup:
Traverse to your project directory for which you want to run scan. Create one new file inside project folder with name “sonar-project” and extension will be “properties” as “sonar-project.properties”

Add basic configuration given below:

```bash
sonar.projectKey="myproject"
sonar.projectName="My project"
sonar.sourceEncoding=UTF-8
sonar.sources=. // list of folders which will scan
sonar.host.url=http://localhost:9000
sonar.login=d43e9c85a815359c1f475d49c78f4aab35ca164e
sonar.coverage.exclusions=**/**
sonar.exclusions=database/migrations/**,resources/lang/** // list of folders which will exclude from scan
```
[Reference](https://docs.sonarsource.com/sonarqube/latest/analyzing-source-code/scanners/sonarscanner/#configuring-your-project)

>“sonar.sources” & “sonar.exclusion” property values will be the list of folders or files which you wants to scan or exclude from scan. The list must be separated by comma(,). If you want to include all files or folders, then just mention Dot(.).
>In sample code, I want to exclude migrations, language folders so added in the list. Same I want to scan whole project so mentioned in source as “.”

Run below command to scan your code:
```bash
sonar-scanner
```
