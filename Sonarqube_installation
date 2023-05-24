In a previous article, I explained how to install and configure SonarQube on a development environment aka on localhost. In this article, I will explain how to install a SonarQube server on an AWS EC2 instance with Amazon Linux from A to Z.

This installation guide will consist of 7 major parts as mentioned follows,

System properties of the EC2 instance we will be using
Download, Install and Configure A PostgreSQL instance
Create a system user for SonarQube
Install OpenJDK 11
Download, Install and Configure SonarQube Server
1. Set up an EC2 instance
We need an EC2 server up and running before starting installing a sonar server. I will be using an Amazon Linux 2 AMI as it comes with AWS CLI pre-installed which will help if we choose to use this instance in a code pipeline and other places. You can use a machine type of t2 medium or larger as we need at least 3GB of RAM to run SonarQube efficiently.

You can go ahead and create an EC2 instance with Amazon Linux 2 AMI with t2 medium machine type now. After you have created ssh into the server using the Ubuntu terminal. You can acquire the connect string by clicking the connect button in the EC2 management console.

2. Setup a PostgreSQL Database
We need a database instance to store SonarQube results, instead of the in-memory database that comes default with SonarQube. SonarQube only supports Oracle, Microsoft SQL Server or PostgreSQL databases only. From three of them, I have chosen PostgreSQL for the convenience of setup. You are welcome to use any of the other two types of databases also.

2.1 Install PostgreSQL 9.6
sudo yum install postgresql96 postgresql96-server #this is a one time command
2.2 Create a new PostgreSQL database cluster
sudo service postgresql96 initdb
2.3 Start PostgreSQL service
sudo service postgresql96 start
2.4 Change password of default PostgreSQL user
sudo passwd postgres
2.5 Login as Postgres user using the new password
su — postgres
2.6 Login to psql shell
psql
2.7 Create a user and database for sonar
CREATE USER sonar WITH ENCRYPTED PASSWORD ‘sonar_password’;
CREATE DATABASE sonarqube;
GRANT ALL PRIVILEGES ON DATABASE sonarqube to sonar; #Don’t exit from psql shell
2.8 Edit authentication methods of PostgreSQL
2.8.1 locate pg_hba.conf file and copy the path into clipboard

SHOW hba_file; #inside psql shell
## default output
/var/lib/pgsql96/data/pg_hba.conf #$HBA_FILE_PATH
2.8.2 exit into ec2-user/root bash

\q #exit from psql shell
exit #exit from postgres user bash
2.8.3 open hba file in an editor

sudo nano $HBA_FILE_PATH
2.8.4 edit authentication methods of local and host connections as follows

OLD:-

# TYPE DATABASE USER ADDRESS METHOD
# “local” is for Unix domain socket connections only
local all all peer
# IPv4 local connections:
host all all 127.0.0.1/32 ident
# IPv6 local connections:
host all all ::1/128 ident
NEW:-

# TYPE DATABASE USER ADDRESS METHOD
# “local” is for Unix domain socket connections only
local all sonar md5
local all all peer
# IPv4 local connections:
host all all 127.0.0.1/32 md5
# IPv6 local connections:
host all all ::1/128 md5
2.8.5 restart PostgreSQL service

sudo service postgresql96 restart
3. Create Sonar System User
We need a non-root system user to run a SonarQube as it uses ElasticSearch as the analytic engine which not allows itself to run as a user with root privileges.

3.1 Create a user group for sonar users
sudo groupadd sonar
3.2 Create a user named sonar without sign in options
sudo useradd -c “Sonar System User” -d /opt/sonarqube -g sonar -s /bin/bash sonar
3.3 Activate sonar user by setting a password to it
sudo passwd sonar
3.4 Add sonar group to ec2-user
sudo usermod -a -G sonar ec2-user
3.5 Exit from ec2-user bash and reconnect to the server to load new group for ec2-user
4. Install OpenJDK 11
We need OpenJDK 11 to run a SonarQube instance of version 7.9 or higher. You can use JDK 8 for SonarQube version 7.8.

4.1 Download zip file
curl -O https://download.java.net/java/GA/jdk11/13/GPL/openjdk-11.0.1_linux-x64_bin.tar.gz
4.2 Unzip and move files into an appropriate path
tar zxvf openjdk-11.0.1_linux-x64_bin.tar.gz
sudo mv jdk-11.0.1 /usr/local/
4.3 Change access to JDK folder
sudo chmod -R 755 /usr/local/jdk-11.0.1
4.4 Add Java to the path
4.4.1 Open /etc/profile in an editor

sudo nano /etc/profile
4.4.2 Add following lines add the end of the file

export JAVA_HOME=/usr/local/jdk-11.0.1
export PATH=$JAVA_HOME/bin:$PATH
4.4.3 Load new configurations

source /etc/profile
4.4.4 Check java is installed properly for each user

java -version
# Expected Output
openjdk version “11.0.1” 2018–10–16
OpenJDK Runtime Environment 18.9 (build 11.0.1+13)
OpenJDK 64-Bit Server VM 18.9 (build 11.0.1+13, mixed mode)
5. Install SonarQube Server
5.1 Download SonarQube 7.9.1 Developer Edition
wget https://binaries.sonarsource.com/CommercialDistribution/sonarqube-developer/sonarqube-developer-7.9.1.zip
5.2 Unzip the downloaded zip file
unzip sonarqube-developer-7.9.1.zip
5.3 Move SonarQube to a file path of choosing (normally /opt/sonarqube)
sudo mv -v sonarqube-7.9.1/* /opt/sonarqube
5.4 Change ownership of all the sonarqube files to user sonar
sudo chown -R sonar:sonar /opt/sonarqube
5.5 Change file access privileges
sudo chmod -R 775 /opt/sonarqube
5.6 Set SONAR_HOME environment variable for future use (optional)
5.6.1 Open .bashrc file in an editor

nano ~/.bashrc
5.6.2 Enter the following line to the end of the file

export SONAR_HOME=/opt/sonarqube
5.6.3 Load new configurations

source ~/.bashrc #you can check whether variable is added by running echo $SONAR_HOME
6. Configure SonarQube
6.1 Set run as user
6.1.1 Open sonar.sh in an editor

sudo nano $SONAR_HOME/bin/linux-x86–64/sonar.sh
6.1.2 Find the line RUN_AS_USER, uncomment it by removing the pound sign and enter sonar user as the value

RUN_AS_USER=sonar
6.2 Other configurations
6.2.1 Open sonar.properties in an editor

sudo nano $SONAR_HOME/conf/sonar.properties
6.2.2 add jdbc user name and password

sonar.jdbc.username=sonar
sonar.jdbc.password=sonar_password
6.2.3 find and uncomment Postgres driver property and remove current schema param if you are not using a custom schema for SonarQube database

sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube
6.2.5 change web port for SonarQube (default value is 9000) (optional)

sonar.web.port=8080 #must be a value higher than 1000
6.2.6 change web CONTEXT for SonarQube (default is root which means you can access sonar server using Http://<host_ip>:<webport>) (optional)

sonar.web.context=/sonarqube #sonar access url will be http://<host_ip>:<webport>/sonarqube
6.2.7 set a path for dedicated volume with fast io for Elasticsearch data storage

sonar.path.data=/path/to/fast/io/volume/data
sonar.path.temp=/path/to/fast/io/volume/temp
6.3 Add a custom TCP security rule for EC2 instance to allow inbound traffic to the selected SonarQube port (default: 9000)
6.4 Start sonar server
Please note that there are some issues that we have faced while starting the SonarQube server in several EC2 instances. After this section, those issues have been listed with the solutions.

$SONAR_HOME/bin/linux-x86–64/sonar.sh console #enter sonar system user password once prompted
6.5 Connect to SonarQube instance for the local machine
http://<host_ip>:<sonar_web_port>/<web_context> # <web_context> must be entered only if you defined any.
6.6 Login to SonarQube server using default username & password
username : admin
password : admin
6.7 Set a proper password for admin user once logged in
6.8 To obtain a license for SonarQube Developer edition, the SonarQube server-id should be obtained
Navigate to Administration > System and obtain server id
ISSUES:
1. max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]

1.1 Open the /etc/security/limits.conf file

sudo nano /etc/security/limits.conf
1.2 Add the following line to the end of the file

ec2-user — nofile 65535
1.3 Disconnect from server and connect again

2. max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

2.1 Open the /etc/sysctl.conf file

sudo nano /etc/sysctl.conf
2.2 Add the following lines

vm.max_map_count=262144
2.3 Reload system configurations

sudo sysctl -p
Conclusion
There are three main prerequisites that should be satisfied before installing a SonarQube instance in a Linux-production environment. There an up and running supported SQL database, a non-root system user and JDK installed in the local environment. After they are satisfied, SonarQube software can be download and installed. Then SonarQube server should be configured to use the setup database and system user.

Note: It is recommended to use a separate hard drive with fast i/o to store elastic search data as elastic search needs frequent access to those files.
