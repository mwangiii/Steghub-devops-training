# Project Documentation: LAMP Stack Implementation in AWS

## Overview
I deployed a real-life project implementing the LAMP stack on AWS.  
The LAMP stack consists of Linux (Ubuntu 20.04), Apache HTTP server, MySQL and PHP.

## TOPICS COVERED 
- AWS Account Creation
- Installing Apache and Updating Firewall
- Installing MySQL
- Installing PHP
- Creating a Virtual Host with Apache

## Steps Taken
1. **Registering a New AWS Account**:
   - Followed the step-by-step guide provided by AWS: [AWS Account Creation Guide](https://repost.aws/knowledge-center/create-and-activate-aws-account)

2. **Installing Apache and Updating the Firewall**:
   - I started the process by connecting to  __EC2 to SSH__ using the terminal  
   ``` ssh -i <private-key-name>.pem ubuntu@<public-IP-address>```
   - To make the process a bit easier I created a bash script ```ssh-steghub.sh``` to ease the process
   - The next step was to change the permission of the private-key.pem  
   ```sudo chmod 0400 <private-key-name>.pem```
   - Install Apache using Ubuntu's package manager __'apt'__
``` bash
  #update a list of packages in package manager  
   sudo apt update  
  #run apache2 package installation  
   sudo apt install apache2
```
 - To verify that apache is working  
 ``` sudo systemctl status apache2```  

 ![APACHE2 RUNNING](apacheWorking.jpeg)  
 - I checked if it can be accessed locally using curl  
 ```curl http://localhost:80```  
 or  
 ```curl http://127.0.0.1:80```  
 N/B __second once uses ip address while the first DNS__

 - I checked if the APACHE HTTP can respond to requests from the internet        
 ``` http://<Public-ip-Address>:80```  
  __I used the google.com to check__  

   ![APACHE2 IS RESPONDING](google.com.png)  

3. **Installing MySQL**:
   - Configuring MySQL server and databases for the project.
   - Again use apt to install MySQL
   ``` sudo apt install mysql-server```
   
4. **Installing PHP**:
   - Setting up PHP environment and dependencies.

5. **Creating a Virtual Host for the Website Using Apache**:
   - Assigned the project domain "projectlamp" to the virtual host.
   - Utilized Emacs for editing configuration files instead of the required Vim.
   - Implemented '#' character usage as per project requirements.

## Challenges Faced
[Describe any challenges encountered during the project.]

## Solutions Implemented
[Explain the solutions implemented for the challenges.]


## DAILY UPDATES
### [Date]
- Yesterday: [Brief summary of what you worked on yesterday.]
- Today: [Outline what you plan to accomplish today.]

## Screenshots
<picture>
  <!-- Include relevant screenshots here -->
</picture>

## FEEDBACK AND REVIEWS
[Link to the Word document containing the GitHub link for submission.]

## KEYWORDS
- TCP(Transmission Control Protocol) : TCP is a communication protocol used for transmitting data reliably and accurately across networks.  
It ensures that data packets arrive in the correct order and without errors by establishing a connection between sender and receiver and managing data flow.
- UDP(User Datagram Protocol) : UDP is another communication protocol used for transmitting data across networks. Unlike TCP, UDP is connectionless and does not guarantee delivery or order of data packets.  
It is often used for applications where speed and efficiency are prioritized over reliability, such as real-time streaming or online gaming.
- DBMS : A DBMS is software that manages databases, providing an interface for users and applications to interact with data stored in the database.  
It handles tasks like data organization, storage, retrieval, and security, allowing multiple users to access and manipulate data concurrently.
- SSH  (Secure Shell): SSH is a protocol used for securely accessing and managing remote systems over a network.  
 It provides encrypted communication between the client and server, allowing users to securely log in to a remote machine, execute commands, transfer files, and perform administrative tasks.
- DNS(Domain Name System): DNS is a system used to translate domain names (e.g., example.com) into IP addresses (e.g., 192.168.1.1) that computers can understand.   
It acts as a distributed database that maps domain names to their corresponding IP addresses, facilitating the routing of network traffic on the internet.
- IP (Internet Protocol): IP is a fundamental protocol used for routing and addressing data packets across networks.  
 It assigns unique IP addresses to devices on a network, allowing them to communicate with each other. IP is part of the TCP/IP protocol suite and is essential for internet communication.
