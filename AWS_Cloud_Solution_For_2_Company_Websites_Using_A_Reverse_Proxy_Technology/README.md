# AWS CLOUD SOLUTION FOR TWO COMPANY WEBSITE USING A REVERSE PROXY TECHNOLOGY
## LETS US GET STARTED
In previous projects you used basic _Infrastructure as a Service_ (IaaS) offerings from AWS such as _EC2 (Elastic Compute Cloud)_ as rented Virtual Machines and _EBS (Elastic Block Store)_, you have also learned how to configure Key pairs and basic Security Groups.

But the power of Clouds is not only in being able to rent Virtual Machines - it is much more than that. From now on, you will start gradually study different Cloud concepts and tools on example of AWS, but do not be worried, your knowledge will not be limited to only AWS specific concepts - overall principles are common across most of the major Cloud Providers (e.g., Microsoft Azure and Google Cloud Platform).  
**NOTE**: _The next few projects will be implemented manually. Before you begin to automate infrastructure in the cloud, it is very important that you can build the solution manually. Otherwise, programming your automation may become frustrating very quickly._    

You will build a secure infrastructure inside AWS VPC (Virtual Private Cloud) network for a fictitious company (Choose an interesting name for it) that uses WordPress CMS for its main business website, and a Tooling Website (https://github.com/<your-name>/tooling) for their DevOps team.  
As part of the company's desire for improved security and performance, a decision has been made to use a reverse proxy technology from NGINX to achieve this.

Cost, Security, and Scalability are the major requirements for this project. Hence, implementing the architecture designed below, ensure that infrastructure for both websites, WordPress and Tooling, is resilient to Web Server's failures, can accomodate to increased traffic and, at the same time, has reasonable cost.
## STARTING OFF THE AWS CLOUD PROJECT
Here are few requirements that must be met before you begin:
1. Properly configure your AWS account and Organization Unit Watch How To Do This Here
    - Create an AWS Master account. (Also known as Root Account)
    ![](assests/awsOrg.png)
    - Within the Root account, create a sub-account and name it `DevOps`. (You will need another email address to complete this)
    ![](assests/devopsCreated.png)
    - Within the Root account, create an AWS Organization Unit (OU). Name it `Dev`. (We will launch Dev resources in there)
    ![](assests/orgunit.png)
    - Move the DevOps account into the **Dev OU**.
    ![](assests/mvDevOps.png)
    - Login to the newly created AWS account using the new email address.
2. Create a free domain name for your fictitious company at `Freenom` domain registrar [here](https://www.freenom.com/en/index.html?lang=en.)

 **Note**:_Since we already have a domain `mwangiii.online` from the previous project, we shall skip this step._  
3. Create a hosted zone in AWS, and map it to your free domain from Freenom.
Watch how to do that [here](https://www.youtube.com/watch?v=IjcHp94Hq8A&feature=youtu.be)
![](assests/route53.png)  
![](assests/hostedzone.png)
![](assests/hostcreated.png)
- Let's map the hosted zone to our free domain
- Copy the name server (NS) values from AWS, then go to your free domain, edit the default NS values and update them with the values from AWS.
![](assests/hostedDets.png)
![](assests/dnsupdate.png)
**NOTE** : _As you proceed with the configuration, ensure that all resources are appropriately tagged,_
- for example:
    - **Project:** `Project-15`
    - **Environment**: `dev`
    - **Automated**: `No` (If you create a recource using an automation tool, it would be <Yes>)



### SET UP A VIRTUAL PRIVATE NETWORK(VPC)
Always make reference to the architectural diagram and ensure that your configuration is aligned with it.
1. Create a VPC
![](assests/vpcCreate.png)
2. Create subnets as shown in the architecture
![](assests/subnets.png)
3. Create a route table and associate it with public subnets
![](assests/publicRTB.png)
4. Create a route table and associate it with private subnets
![](assests/privateRTB.png)
5. Create an Internet Gateway
![](assests/internetGateway.png)
6. Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accessible from the Internet)
![](assests/publicGateway.png)
7. Create 3 Elastic IPs
![](assests/ips.png)
8. Create a Nat Gateway and assign one of the Elastic IPs (*The other 2 will be used by Bastion hosts)
![](assests/natgateway.png)
9. Create a Security Group for:
    - **Nginx Servers:** Access to Nginx should only be allowed from a Application Load balancer (ALB). At this point, we have not created a load balancer, therefore we will update the rules later. For now, just create it and put some dummy records as a place holder.
    ![](assests/nginxSG.png)
    - **Bastion Servers:** Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your workstation public IP address. To get this information, simply go to your terminal and type curl www.canhazip.com
    ![](assests/bastionServer.png)
    - **Application Load Balancer:** ALB will be available from the Internet
    ![](assests/ext-alb.png)
    - Webservers: Access to Webservers should only be allowed from the Nginx servers. Since we do not have the servers created yet, just put some dummy records as a place holder, we will update it later.
    ![](assests/webserversSG.png)
    - **Data Layer:** Access to the Data layer, which is comprised of Amazon Relational Database Service (RDS) and Amazon Elastic File System (EFS) must be carefully desinged - only webservers should be able to connect to RDS, while Nginx and Webservers will have access to EFS Mountpoint.
    ![](assests/dataLayer.png)

### PROCEED WITH COMPUTE RESOURCES
You will need to set up and configure compute resources inside your VPC. The recources related to compute are:
  - EC2 Instances
  - Launch Templates
  - Target Groups
  - Autoscaling Groups
  - TLS Certificates
  - Application Load Balancers (ALB)

### SET UP COMPUTE RESOURCES FOR NGINX
#### PROVISION EC2 INSTANCES FOR NGINX
1. Create an EC2 Instance based on CentOS Amazon Machine Image (AMI) in any 2 Availability Zones (AZ) in any AWS Region (it is recommended to use the Region that is closest to your customers). Use EC2 instance of T2 family (e.g. t2.micro or similar)
2. Ensure that it has the following software installed:
    - python
    - ntp
    - net-tools
    - vim
    - wget
    - telnet
    - epel-release
    - htop
3. Create an AMI out of the EC2 instance
![](assests/nginxImage.png)

#### PREPARE LAUNCH TEMPLATE FRO NGINX(ONE PER SUBNET)
1. Make use of the AMI to set up a launch template
2. Ensure the Instances are launched into a public subnet
3. Assign appropriate security group
4. Configure Userdata to update yum package repository and install nginx

#### CONFIGURE TARGET GROUPS
1. Select Instances as the target type
2. Ensure the protocol HTTPS on secure TLS port 443
3. Ensure that the health check path is /healthstatus
4. Register Nginx Instances as targets
5. Ensure that health check passes for the target group

#### CONFIGURE AUTOSCALING FOR NGINX
1. Select the right launch template
2. Select the VPC
3. Select both public subnets
4. Enable Application Load Balancer for the AutoScalingGroup (ASG)
5. Select the target group you created before
6. Ensure that you have health checks for both EC2 and ALB
7. The desired capacity is 2
8. Minimum capacity is 2
9. Maximum capacity is 4
10. Set scale out if CPU utilization reaches 90%
11. Ensure there is an SNS topic to send scaling notifications

### SET UP COMPUTE RESOURCES FOR BASTION 
#### PROVISION THE ECS INSTANCES FOR BASTION 
1. Create an EC2 Instance based on CentOS Amazon Machine Image (AMI) per each Availability Zone in the same Region and same AZ where you created Nginx server
2. Ensure that it has the following software installed
    - python
    - ntp
    - net-tools
    - vim
    - wget
    - telnet
    - epel-release
    - htop
3. Associate an Elastic IP with each of the Bastion EC2 Instances
4. Create an AMI out of the EC2 instance

#### PREPARE LAUNCH TEMPLATE FOR BASTION(ONE PER SUBNET)
1. Make use of the AMI to set up a launch template
2. Ensure the Instances are launched into a public subnet
3. Assign appropriate security group
4. Configure Userdata to update yum package repository and install Ansible and git


#### CONFIGURE TARGET GROUPS 
1. Select Instances as the target type
2. Ensure the protocol is TCP on port 22
3. Register Bastion Instances as targets
4. Ensure that health check passes for the target group

#### CONFIGURE AUTOSCALING FOR BASTION 
1. Select the right launch template
2. Select the VPC
3. Select both public subnets
4. Enable Application Load Balancer for the AutoScalingGroup (ASG)
5. Select the target group you created before
6. Ensure that you have health checks for both EC2 and ALB
7. The desired capacity is 2
8. Minimum capacity is 2
9. Maximum capacity is 4
10. Set scale out if CPU utilization reaches 90%
11. Ensure there is an SNS topic to send scaling notifications

### SET UP COMPUTE RESOURCES FOR WEBSERVERS
#### PROVISION THE EC2 INSTANCES FOR WEBSERVERS
Now, you will need to create 2 separate launch templates for both the WordPress and Tooling websites

1. Create an EC2 Instance (Centos) each for WordPress and Tooling websites per Availability Zone (in the same Region).
2. Ensure that it has the following software installed
      - python
      - ntp
      - net-tools
      - vim
      - wget
      - telnet
      - epel-release
      - htop
      - php
3. Create an AMI out of the EC2 instance
![](assests/WebserverImage.png)
#### PREPARE LAUNCH TEMPLATE FOR WEBSERVERS(ONE PER SUBNET)
1. Make use of the AMI to set up a launch template
2. Ensure the Instances are launched into a public subnet
3. Assign appropriate security group
4. Configure Userdata to update yum package repository and install wordpress (Only required on the WordPress launch template)

#### TLS CERTIFICATES FROM AMAZON CERTIFICATE MANAGER(ACM)
You will need TLS certificates to handle secured connectivity to your Application Load Balancers (ALB).
1. Navigate to AWS ACM
2. Request a public wildcard certificate for the domain name you registered in Freenom
3. Use DNS to validate the domain name
4. Tag the resource
![](assests/ACM.png)
![](assests/cnameRoute.png)
![](assests/ACMActive.png)


### CONFIGURE APPLICATION LOAD BALANCER (ALB)
#### APPLICATION LOAD BALANCER TO ROUTE TRAFFIC TO NGINX
Nginx EC2 Instances will have configurations that accepts incoming traffic only from Load Balancers. No request should go directly to Nginx servers. With this kind of setup, we will benefit from intelligent routing of requests from the ALB to Nginx servers across the 2 Availability Zones.  
We will also be able to offload SSL/TLS certificates on the ALB instead of Nginx. Therefore, Nginx will be able to perform faster since it will not require extra compute resources to valifate certificates for every request.

![](assests/pageOneLb.png)
  1. Create an Internet facing ALB
  2. Ensure that it listens on HTTPS protocol (TCP port 443)
  ![](assests/pageThreeLb.png)
  3. Ensure the ALB is created within the appropriate VPC | AZ | Subnets
  4. Choose the Certificate from ACM
![](assests/pageFourLb.png)
  5. Select Security Group
  6. Select Nginx Instances as the target group
![](assests/pageTwoLb.png)
#### APPLICATION LOAD BALANCER TO ROUTE TRAFFIC TO WEB SERVERS
Since the webservers are configured for auto-scaling, there is going to be a problem if servers get dynamically scalled out or in. Nginx will not know about the new IP addresses, or the ones that get removed. Hence, Nginx will not know where to direct the traffic.

To solve this problem, we must use a load balancer. But this time, it will be an internal load balancer. Not Internet facing since the webservers are within a private subnet, and we do not want direct access to them.

  1. Create an Internal ALB
  2. Ensure that it listens on HTTPS protocol (TCP port 443)
  3. Ensure the ALB is created within the appropriate VPC | AZ | Subnets
  4. Choose the Certificate from ACM
  5. Select Security Group
  6. Select webserver Instances as the target group
  7. Ensure that health check passes for the target group
![](assests/internalLbOne.png)
![](assests/internalLbTwo.png)
![](assests/internalLbThree.png)
**NOTE:** _This process must be repeated for both WordPress and Tooling websites._

### SETUP EFS
Amazon Elastic File System (Amazon EFS) provides a simple, scalable, fully managed elastic Network File System (NFS) for use with AWS Cloud services and on-premises resources. In this project, we will utulize EFS service and mount filesystems on both Nginx and Webservers to store data.
  1. Create an EFS filesystem
  2. Create an EFS mount target per AZ in the VPC, associate it with both subnets dedicated for data layer
  3. Associate the Security groups created earlier for data layer.
  4. Create an EFS access point. (Give it a name and leave all other settings as default)
![](assests/EFS.png)
### SETUP RDS
Pre-requisite: Create a KMS key from Key Management Service (KMS) to be used to encrypt the database instance.

Amazon Relational Database Service (Amazon RDS) is a managed distributed relational database service by Amazon Web Services. This web service running in the cloud designed to simplify setup, operations, maintenans & scaling of relational databases. Without RDS, Database Administrators (DBA) have more work to do, due to RDS, some DBAs have become jobless

To ensure that yout databases are highly available and also have failover support in case one availability zone fails, we will configure a multi-AZ set up of RDS MySQL database instance. In our case, since we are only using 2 AZs, we can only failover to one, but the same concept applies to 3 Availability Zones. We will not consider possible failure of the whole Region, but for this AWS also has a solution - this is a more advanced concept that will be discussed in following projects.
![](assests/symetricKey.png)
![](assests/aliasKey.png)
![](assests/fullKey.png)
**To configure RDS, follow steps below:**
  1. Create a subnet group and add 2 private subnets (data Layer)
  ![](assests/redsGroup.png)
  ![](assests/rdsStorage.png)
  2. Create an RDS Instance for mysql 8.*.*
  ![](assests/mysqlDatabase.png)
  3. To satisfy our architectural diagram, you will need to select either Dev/Test or Production Sample Template. But to minimize AWS cost, you can select the Do not create a standby instance option under Availability & durability sample template (The production template will enable Multi-AZ deployment)
  4. Configure other settings accordingly (For test purposes, most of the default settings are good to go). In the real world, you will need to size the database appropriately. You will need to get some information about the usage. If it is a highly transactional database that grows at 10GB weekly, you must bear that in mind while configuring the initial storage allocation, storage autoscaling, and maximum storage threshold.
  5. Configure VPC and security (ensure the database is not available from the Internet)
  ![](assests/connecticity.png)
  6. Configure backups and retention
  7. Encrypt the database using the KMS key created earlier
  8. Enable CloudWatch monitoring and export Error and Slow Query logs (for production, also include Audit)
  ![](assests/passwordRDS.png)
**Note:** _This service is an expensinve one. Ensure to review the monthly cost before creating. (DO NOT LEAVE ANY SERVICE RUNNING FOR LONG)_

#### CONFIGURING DNS WITH ROUTE53
Now let us access our tooling website via a browser using our DNS name
![](assests/php.jpg)
![](assests/tooling-admin.jpg)
**NOTE:** _You can use either CNAME or alias records to achieve the same thing. But alias record has better functionality because it is a faster to resolve DNS record, and can coexist with other records on that name. Read here to get to know more about the differences._
 
Ec2 instance 
vpc subents 
EFS