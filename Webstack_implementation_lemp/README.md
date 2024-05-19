# PROJECT DOCUMENTATION :WEBSTACK  IMPLEMENTATION

## OVERVIEW
I deployed a real-life project implementing the LAMP stack on AWS.
The LAMP stack consists of Linux (Ubuntu 20.04), Apache HTTP server, MySQL and PHP.

# TOPICS COVERED
    AWS Account Creation
    Installing Apache and Updating Firewall
    Installing MySQL
    Installing PHP
    Creating a Virtual Host with Apache

# STEPS TAKEN
1.Installing the Nginx Web Server
- update packages
 ```
 sudo apt update
 ```
 - Install nginx
 ```
 sudo apt install nginx -y 
 ```
 - To check if nginx was installed run
 __N/B The -y flag automatically confirms the installation.__
 ```
 sudo systemctl status nginx 
 ```
 Our server is running and we can access it locally and from the internet (Source 0.0.0.0/0 'from any IP address')
 I checked if I can access the ubuntu shell locally using curl,
 ```
 curl http://localhost:80
 ```
 or 
 ```
 curl http://127.0.0.1:80
 ```
 __N/B Both commands do the same thing. The second one uses the IP address while the first uses DNS.__

 - Now to check if nginx responds to requests from the internet use the following url
 ```
 http://13.60.72.1:80
 ```
 Another way to retrieve your public address other than check the AWS console would be by using the command 
 ```
 curl -s http://169.254.169.254/latest/meta-data/public-ipv4
 ```

 2.Installing MYSQL
 - We will use MYSQL as our (Database Management System)DBMS
 - Again using apt 
 ```
 sudo apt install mysql-server -y
 ```
- When finished log into MYSQL console ny typing
```
sudo mysql
```
- As recommended to run a security script that comes with MYSQL.
The script will remove some insecure default settings and lock down access to my database system.
Before running the script the script I set the a password for the root user,using *mysql_native_password* as a default authentication method.
```
 ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Password.1';
 ```
- Exit the MYSQL console using
```
mysql> exit
```
- To start the interactive script run
```
sudo mysql_secure_installation
```
- Log into MySQL with Password
```
 sudo mysql -p
```
- To exit the MYSQL server type 
```
exit
```

3.Installing PHP
- Here we installed *php-fpm* which stands for "PHP fastCGI process manager" and *php-mysql* which allows php to communicate with MYSQL-based databases.
-To run this packages at the same time use
```
sudo apt install php-fpm php-mysql -y 
``` 

4.Configuring Nginx to use PHP processor
- On ubuntu Nginx has one server block enabled by default and is configured to serve documents out of a directory  */var/www/html.*
- Instead of modifying  */var/www/html.* we'll create a directory 0*/var/www/* for the *your_domain* website 
- create the root web directory for  *your_domain* as follows:
```
sudo mkdir /var/www/projectLEMP