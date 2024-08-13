# DEVOPS TOOLING WEBSITE SOLUTION - 101
![devops](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fmiro.medium.com%2Fv2%2Fresize%3Afit%3A904%2F1*AXFR-ZWlB1Du3BrEicdIEg.jpeg&f=1&nofb=1&ipt=f0e05e29fe6ca4ce083d89f78884963be06d7041cafcf5a7dea54872acd18cb4&ipo=images)
For this project we want to introduce a set of DevOps tools that will help our team in day to day activities in managing,developing, testing, deploying and monitoring different projects.  
We will introduce a single DevOps Tooling Solution that will consist of:
- Jenkins - free and open source automation server used to build CI/CD pipelines.
- Kubernetes - an open-source container-orchestration system for automating computer application deployment, scaling, and management.
- Jfrog Artifactory - Universal Repository Manager supporting all major packaging formats, build tools and CI servers. Artifactory.
- Rancher - an open source software platform that enables organizations to run and manage Docker and Kubernetes in production.
- Grafana - a multi-platform open source analytics and interactive visualization web application.
- Prometheus - An open-source monitoring system with a dimensional data model, flexible query language, efficient time series database and modern alerting approach.
- Kibana - Kibana is a free and open user interface that lets you visualize your Elasticsearch data and navigate the Elastic Stack.

**Note**: _Do not feel overwhelmed by all the tools and technologies listed above,we will gradually get ourselves familiar with them in upcoming projects!_
.
## SETUP AND TECHNOLOGY USED
In this project we will implement a solution that consists of following components:  
1.**Infrastructure**: AWS  
2.**Webserver Linux**: Red Hat Enterprise Linux 8  
3.**Database Server**: Ubuntu 24.04 + MySQL   
4.**Storage Server**: Red Hat Enterprise Linux 8 + NFS Server  
5.**Programming Language**: PHP  
6.**Code Repository**: GitHub
# PROCEDURE
## PREPARE NFS SERVER
- Spin up a new EC2 instance with RHEL Linux 8 Operating System.
- Based on your LVM experience from Project 6, Configure LVM on the Server.
- Instead of formatting the disks as ext4 you will have to format them as xfs
- Ensure there are 3 Logical Volumes. __lv-opt__ __lv-apps__ and __lv-logs__
- Create mount points on _/mnt_ directory for the logical volumes as follows:
  -  **Mount lv-apps on /mnt/apps** - To be used by webservers Mount lv-logs on /mnt/
  -  **logs** - To be used by webserver logs 
  - **Mount lv-opt on /mnt/opt** - To be used by Jenkins server in Project 8

- Install NFS server, configure it to start on reboot and make sure it is u and running
```
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
```
- Export the mounts for webservers' subnet cidr to connect as clients. For simplicity, you will install your all three Web Servers inside the same subnet, but in production set up you would probably want to separate each tier inside its own subnet for higher level of security. To check your subnet cidr - open your EC2 details in AWS web console and locate 'Networking' tab and open a Subnet link:

- Make sure we set up permission that will allow our Web servers to read, write and execute files on NFS:
```
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt
sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt
sudo systemctl restart nfs-server.service
```

- Configure access to NFS for clients within the same subnet (example of Subnet CIDR - 172.31.32.0/20 ):
```
sudo vi /etc/exports
```
```
/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
```
```
Esc + :wq!
```
```
sudo exportfs -arv
```

- Check which port is used by NFS and open it using Security Groups (add new Inbound Rule)
```
rpcinfo -p | grep nfs
```
- **Important note**: _In order for NFS server to be accessible from your client, you must also open following ports: TCP 111, UDP 111, UDP 2049_


## CONFIGURE THE DATABSE SERVER
By now you should know how to install and configure a MySQL DBMS to work with remote Web Server
- Install MySQL server
- Create a database and name it tooling
- Create a database user and name it webaccess
- Grant permission to webaccess user on tooling database to do anything only from the webservers subnet cidr

## PREPARE THE WEB SERVERS
- We need to make sure that our Web Servers can serve the same content from shared storage solutions, in our case - NFS Server and MySQL database.
- You already know that one DB can be accessed for reads and writes by multiple clients.
- For storing shared files that our Web Servers will use - we will utilize NFS and mount previously created Logical Volume __lv-apps__ to the folder where Apache stores files to be served to the users (/var/www).
- This approach will make our Web Servers stateless, which means we will be able to add new ones or remove them whenever we need and the integrity of the data (in the database and on NFS) will be preserved.
- During the next steps we will do following:
  - Configure NFS client (this step must be done on all three servers)
  - Deploy a Tooling application to our Web Servers into a shared NFS folder
  - Configure the Web Servers to work with a single MySQL database

- Launch a new EC2 instance with RHEL 8 Operating System
- Install NFS client
```
sudo yum install nfs-utils nfs4-acl-tools -y
```
- Mount /var/www/ and target the NFS server's export for apps
```
sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www
```
- Verify that NFS was mounted successfully by running df -h. Make sure that the changes will persist on Web Server after reboot:
```
sudo vi /etc/fstab
```
- add following line
```
<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0
```
- Install Remi's repository, Apache and PHP
```
sudo yum install httpd -y
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo dnf module reset php
sudo dnf module enable php:remi-7.4
sudo dnf install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
```
### Repeat steps 1-5 for another 2 Web Servers.
- Verify that Apache files and directories are available on the Web Server in /var/www and also on the NFS server in /mnt/apps. If you see the same files - it means NFS is mounted correctly. You can try to create a new file touch test.txt from one server and check if the same file is accessible from other Web Servers.
- Locate the log folder for Apache on the Web Server and mount it to NFS server's export for logs. Repeat step №4 to make sure the mount point will persist after reboot.
- Fork the tooling source code from StegHub Github Account to your Github account. (Learn how to fork a repo here)
- Deploy the tooling website's code to the Webserver. Ensure that the html folder from the repository is deployed to /var/www/html

**Note 1**: _Do not forget to open TCP port 80 on the Web Server._

**Note 2**: _If you encounter **403 Error** - check permissions to your _/var/www/html_ folder and also disable SELinux sudo setenforce 0 To make this change permanent 
- open following config file 
```
sudo vi /etc/sysconfig/selinux
```
- set `SELINUX=disabled` 
- then restart httpd.
- Update the website's configuration to connect to the database (in /var/www/html/functions.php file).
- Apply tooling-db.sql script to your database using this command 
```
mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql
```
- Create in MySQL a new admin user with username: myuser and password: password:

```mysql
INSERT INTO 'users'
 ('id', 'username', 'password', 'email', 'user_type', 'status')
 VALUES -> 
  (1, 'myuser', '5f4dcc3b5aa765d61d8327deb882cf99', 'user@mail.com', 'admin', '1');
```
- Open the website in your browser `http://<Web-Server-Public-IP-Address-or-Public-DNS-Name>/index.php` and make sure you can login into the websute with myuser user.
