# CONTINUOS INTEGRATION PIPELINE USING JENKINS

![](https://miro.medium.com/v2/resize:fit:1200/1*iKuaNfxgZSTe_J2x3PYRUg.png)

DevOps is about Agility, and speedy release of software and web solutions. One of the ways to guarantee fast and repeatable deployments is Automation of routine tasks.

In this project we are going to start automating part of our routine tasks with a free and open source automation server - _**Jenkins**_.  
It is one of the mostl popular CI/CD tools, it was created by a former Sun Microsystems developer Kohsuke Kawaguchi and the project originally had a named "Hudson".

_**Continuous integration (CI)**_ is a software development strategy that increases the speed of development while ensuring the quality of the code that teams deploy.  
Developers continually commit code in small increments (at least daily, or even several times a day), which is then automatically built and tested before it is merged with the shared repository.

# TASKS
## Step 1 
### Install Jenkins server
- Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"
- Install JDK (since Jenkins is a Java-based application)
```
sudo apt update
sudo apt install default-jdk-headless
```
### Install Jenkins
```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
```
```
sudo apt update
sudo apt-get install jenkins
```
- Make sure Jenkins is up and running
```
sudo systemctl status jenkins
```
![Jenkins works](assets/jenkinsWorking.png)
- By default Jenkins server uses TCP port 8080 - open it by creating a new Inbound Rule in your EC2 Security Group
![inbound rules](assets/inboundRules.png)

## Perform initial Jenkins setup.
- From your browser access 
```
http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080
```
- You will be prompted to provide a default admin password
![jenkins login](assets/jenkisLogin.png)
- Retrieve it from your server:
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
- Then you will be asked which plugings to install - choose suggested plugins.

- Once plugins installation is done - create an admin user and you will get your Jenkins server address.
![create an admin](assets/createAdmin.png)
The installation is completed!

## Step 2 
### Configure Jenkins to retrieve source codes from GitHub using Webhooks
In this part, you will learn how to configure a simple Jenkins job/project (these two terms can be used interchangeably).This job will will be triggered by GitHub webhooks and will execute a **_'build'_** task to retrieve codes from GitHub and store it locally on Jenkins server.
- Enable webhooks in your GitHub repository settings
![web hooks](assets/webhooks.png)
- Go to Jenkins web console, click _**"New Item"**_ and create a _**"Freestyle project"**_
- To connect your GitHub repository, you will need to provide its URL, you can copy from the repository itself

- In configuration of your Jenkins freestyle project choose Git repository, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

- Save the configuration and let us try to run the build. For now we can only do it manually. Click _**"Build Now"**_ button, if you have configured everything correctly, the build will be successfull and you will see it under **_#1_** 


- You can open the build and check in `"Console Output"` if it has run successfully.
![console output](assets/consoleOutput.png)

- If so - congratulations! You have just made your very first Jenkins build!

- But this build does not produce anything and it runs only when we trigger it manually. Let us fix it.

- Click _**"Configure"**_ your job/project and add these two configurations

- Configure triggering the job from GitHub webhook:

- Configure _**"Post-build Actions"**_ to archive all the files
- files resulted from a build are called __**"artifacts"**__.


- Now, go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch.

- You will see that a new build has been launched automatically (by webhook) and you can see its results - artifacts, saved on Jenkins server.

- You have now configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as 'push' because the changes are being 'pushed' and files transfer is initiated by GitHub). There are also other methods: trigger one job (downstreadm) from another (upstream), poll GitHub periodically and others.

- By default, the artifacts are stored on Jenkins server locally
```
ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/
```
![](assets/archive.png)

## Step 3
### Configure Jenkins to copy files to NFS server via SSH
- Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to /mnt/apps directory.
- Jenkins is a highly extendable application and there are 1400+ plugins available. We will need a plugin that is called "Publish Over SSH".
- Install _**"Publish Over SSH"**_ plugin.
- On main dashboard select _**"Manage Jenkins"**__ and choose "Manage Plugins" menu item.
- On _**"Available"**_ tab search for _**"Publish Over SSH"**_ plugin and install it
- Configure the job/project to copy artifacts over to NFS server.
- On main dashboard select _**"Manage Jenkins"**_ and choose _**"Configure System"**_ menu item.
- Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:
    - Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)
    - Arbitrary name
    - Hostname - can be private IP address of your NFS server
    - Username - ec2-user (since NFS server is based on EC2 with RHEL 8)
    - Remote directory - /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server
![](assets/installPublish.png)
- Test the configuration and make sure the connection returns Success. Remember, that TCP port 22 on NFS server must be open to receive SSH connections.

- Save the configuration, open your Jenkins job/project configuration page and add another one _**"Post-build Action"**_

- Configure it to send all files produced by the build into our previously define remote directory. In our case we want to copy all files and directories
- so we use **. If you want to apply some particular pattern to define which files to send - use this syntax.
![](assets/post-build-actions.png)
- Save this configuration and go ahead, change something in README.MD file in your GitHub Tooling repository.

- Webhook will trigger a new job and in the "Console Output" of the job you will find something like this:
```
SSH: Transferred 25 file(s)
Finished: SUCCESS
```
- This is what our sever looks like at the  moment

![](https://miro.medium.com/v2/resize:fit:1400/1*q__g9ipApArckVp2Jo-_Ag.png)
# Conclusion
In this project, I set up a basic CI pipeline using Jenkins to automate the process of building and deploying our web infrastructure. The pipeline is triggered whenever a change is made to the codebase of our web infrastructure. The pipeline builds the code using Jenkins and then transfers the code to the NFS server with the help of the Publish Over SSH Jenkins plugin.

