# ORCHESTRATING CONTAINERS ACROSS MULTIPLE VIRTUAL SERVERS WITH KUBERNETES 1- 101
In previous Project, you have started to work with containerization and have learned how to prepare and deploy a Docker container using Docker Compose.
In this project, you will continue building upon your containerization skills, and begin to work on industry tools that fit for production deployment.
Containers are the most lightweight and easily transferrable workloads, they start faster than VMs, consume less space and memory and, they perfectly fit to accommodate microservice architecture.
It means that number of containers will be usually significantly higher that number of VMs and with number comes complexity of managing containers.

To manage a fleet of containers you can use different software solutions, but the most widely used one is Kubernetes (also referred as K8s).
It was originally developed by Google and now being maintained by [Cloud Native Computing Foundation](https://en.wikipedia.org/wiki/Cloud_Native_Computing_Foundation) (CNCF).

Why has it become so popular? It is a powerful Swiss-army knife when it comes to managing multiple containers in production-grade solutions.

For now we will not list down all the features and benefits of Kubernetes, you can watch this [overview video](https://www.youtube.com/watch?v=VnvRFRk_51k) to get to know more, but you certainly must have this tool in your DevOps arsenal and know when and how to use if for production-grade deployments.

This project is the first of a series Kubernetes-related practice projects, so get ready to become a professional containers' fleet "pilot"*.
_*Kubernetes (κυβερνήτης, Greek for "helmsman" or "pilot" or "governor", and the etymological root of cybernetics)_

#### WHY MIGRATE FROM DOCKER COMPOSE TO K8S

In the previous project you successfully deployed your Docker containers using Docker Compose, it is a great tool that helps avoiding execution of multiple CLI commands by preparing a declarative configuration file.
It is handy when you deploy one or a few containers, but in most cases, it does not fit for production deployments.

Because of the many limitations that Docker Compose has, it is very important for us to consider migrating our solution to more an advanced technology.
The most common alternatives to Compose, amongst a few others, are **Docker Swarm** and **Kubernetes**.

#### WHAT IS WRONG WITH DOCKER COMPOSE?

It is important to understand that, DevOps is about "Culture" and NOT "Tools" Therefore, it is not correct to say that one tool is better than another;
different organizations have different needs and a good tool for one team may be bad for another just because their needs are not the same.
In some teams, Docker Compose fit their needs perfectly, despite the perceived limitations.
The major limitation of Docker Compose is that it can only be used to run workloads on a single computer host.
Now, that is an obvious limitation because if our Tooling Application and its MySQL Database are all running on a single VM, like we did in Project 20, then this host is considered as a **SPOF** (i.e. - Single Point Of Failure).

So, could we say there is something wrong with Docker Compose?
Not exactly, as a matter of fact, it is being used a lot in the industry. It fits well into some use cases that require speedy development and Proof of Concepts. As you will soon see, Kubernetes is a lot more complex technology, and it may be an overkill for some use cases.

## CONTAINER ORCHESTRATION WITH KUBERNETES

#### WHAT IS CONTAINER ORCHESTRATION?

Two important things to remember about Docker containers are:
  1. Unlike virtual machines, they are not designed to run for a very long time.  
  By design, Docker containers are **ephemeral**.
  By “ephemeral” it means that a container can be stopped and destroyed,
  and a new one can be built from the same Docker image and put in place with an absolute minimum set-up and configuration requirement.
  2. To ensure that container workloads are highly scalable, they must be configured to run across multiple compute nodes.
  A compute node can be a physical server or a virtual machine with Docker engine installed to run your containers.

If we had two compute nodes to run our containers, let us consider a following scenarios:
  1. Given the two points mentioned above, if containers are configured to run across 2 computer nodes,
   and a particular container, running on Node 1 dies, how will it know that it can spin up again on Node 2?
  2. Let us imagine that Tooling website container is running on Node 1, and MySQL container is running on Node 2,
  how will both containers be able to communicate with each other?   
  Remember in Project 20,
  we had to create a custom network on the same host and ensure that they can communicate through that network.
  But in the case of 2 separate hosts, this is natively not possible.

`Container orchestration` is a concept that allows to address these two scenarios, it provides automation of all the aspects of coordinating and managing containers.
Container orchestration is focused on managing life cycle of containers and their dynamic environments.

It is about automating the entire lifecycle of containers running across multiple nodes:
- Configuring and scheduling of containers on nodes
- Ensuring the availability of containers, even when they die
- Scaling of containers to equally balance application workloads across infrastructure
- Allocation of resources between containers
- Load balancing, traffic routing and service discovery of containers
- Health monitoring of containers
- Securing the interactions between containers.

**Kubernetes is a tool designed to do Container Orchestration and it does its job very well when correctly configured.**  
As mentioned earlier, there are other alternatives to Docker Compose. But, throughout the entire PBL program, we will not focus on **Docker Swarm**.
We will rather spend more time with Kubernetes.  
Part of the reason for this is because Kubernetes has more functionalities and is widely in use in the industry.

To know when to choose between Docker Swarm and Kubernetes, [Here is an interesting article](https://dzone.com/articles/quotdocker-swarm-or-kubernetesquot-is-it-the-right) to read with some very enlightening stats.

## KUBERNETES ARCHITECTURE
Kubernetes is a not a single package application that you can install with one command,
it is comprised of several components, some of them can be deployed as services, some can be also deployed as separate containers.

Let us take a look at Kubernetes architecture diagram below:

![](assets/architecture.png)

Read about every component in the [official documentation](https://kubernetes.io/docs/concepts/overview/components/).

Make sure you understand the role of each component on the diagram above, without this understanding it will be extremely difficult for you to install and operate a K8s cluster, especially when it comes to troubleshooting and maintenance.

As an IT professional in general, you shall be comfortable using official documentation for tools you use, in case of Kubernetes - it has a very well structured and comprehensive documentation portal with multiple configuration code snippets. We strongly encourage you to add it to your bookmarks and refer to it every time you have a K8s-related question.

## "KUBERNETES FROM-GROUND-UP"
#### K8S INSTALLATION OPTIONS

So far, Kubernetes sounds like a lot of fun, right?  
With its intuitive architecture, and rich configuration options, you may already want to jump right in, spin up a few VMs and begin to install and configure a Kubernetes cluster. BUT HOLD ON FOR A SECOND!.  
Installing and configuring Kubernetes is far from being a walk in the park, i.e., it is very difficult to implement and get it ready for production. Especially, if you want to setup a highly available, and secure Kubernetes cluster.

The good news is, there are open-source tools available today that already has all the hard work done and you can plug into them easily.  
An example of that is `minikube`, which can be used during testing and development.

For a better understanding of each aspect of spinning up a Kubernetes cluster, we will do it without any automated helpers.
You will install each and every component manually from scratch and learn how to make them work together - we call this approach **"K8s From-Ground-Up"**.

To successfully implement _"K8s From-Ground-Up"_, the following and even more will be done by you as a K8s administrator:

  1. Install and configure master (also known as control plane) components and worker nodes (or just nodes).
  2. Apply security settings across the entire cluster (i.e., encrypting the data in transit, and at rest)
      - In transit encryption means encrypting communications over the network using HTTPS
      - At rest encryption means encrypting the data stored on a disk
  3. Plan the capacity for the backend data store etcd
  4. Configure network plugins for the containers to communicate
  5. Manage periodical upgrade of the cluster
  6. Configure observability and auditing

**Note:** _Unless you have any business or compliance restrictions, **ALWAYS** consider to use managed versions of K8s - Platform as a Service offerings, such as Azure Kubernetes Service (AKS), Amazon Elastic Kubernetes Service (Amazon EKS), or Google Kubernetes Engine (GKE) as they usually have better default security settings, and the costs for maintaining the control plane are very low._
You will be able to appreciate automation tools and managed versions of Kubernetes much more after you have experienced all the lessons from the struggles and failures from the "K8s From-Ground-Up".

**Let us begin building out Kubernetes cluster from the ground!**

![](https://media4.giphy.com/media/V6jvsGS832gx387gri/200.webp?cid=790b76111j5au63sc310fy4h9ue8669hs6qlni8ow906bcar&ep=v1_gifs_search&rid=200.webp&ct=g)

**DISCLAIMER:** _The following setup of Kubernetes should be used for learning purpose only, and not to be considered for production._
_This is because setting up a K8s cluster for production use has a lot more moving parts, especially when it comes to planning the nodes, and securing the cluster._
_The purpose of "K8s From-Ground-Up" is to get you much closer to the different components as shown in the architecture diagram and relate with what you have been learning about Kubernetes._

## TOOLS TO BE USED AND EXPECTED RESULT OF THE PROJECT 20
- VM: AWS EC2
- OS: Ubuntu 20.04 lts+
- Docker Engine
- kubectl console utility
- cfssl and cfssljson utilities
- Kubernetes cluster

You will create 6 EC2 Instances, and in the end, we will have the following parts of the cluster properly configured:
- Three Kubernetes Master
- Three Kubernetes Worker Nodes
- Configured SSL/TLS certificates for Kubernetes components to communicate securely
- Configured Node Network
- Configured Pod Network

## STEP 0 INSTALL CLIENT TOOLS BEFORE BOOTSTRAPPING THE CLUSTER.
First, you will need some client tools installed and configurations made on your client workstation:
- **awscli** - is a unified tool to manage your AWS services
- **kubectl** - this command line utility will be your main control tool to manage your K8s cluster.
  You will use this tool so many times, so you will be able to type 'kubetcl' on your keyboard with a speed of light.
  You can always make a shortcut (alias) to just one character 'k'.
  Also, add this extremely useful official kubectl Cheat Sheet to your bookmarks, it has examples of the most used 'kubectl' commands.
- **cfssl** - an open source toolkit for everything TLS/SSL from Cloudflare
- **cfssljson**- a program, which takes the JSON output from the cfssl and writes certificates, keys, CSRs, and bundles to disk.

### INSTALL AND CONFIGURE AWS CLI
Configure AWS CLI to access all AWS services used, for this you need to have a user with programmatic access keys configured in AWS Identity and Access Management (IAM):

![](assets/1.png)

Set permissions

![](assets/3.png)

Generate access keys and store them in a safe place

![](assets/4.png)

On your local workstation download and install the [latest version of AWS CLI](https://aws.amazon.com/cli/)

![](assets/0.png)

To [configure your AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html) - run your shell (or cmd if using Windows) and run:


```bash
$ aws configure --profile %your_username%
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```
Test your AWS CLI by running:
```bash
aws ec2 describe-vpcs
```
![](assets/5.png)

and check if you can see VPC details.

### INSTALL KUBECTL

Kubernetes cluster has a Web API that can receive HTTP/HTTPS requests, but it is quite cumbersome to curl an API each and every time you need to send some command, so kubectl command tool was developed to ease a K8s administrator's life.

With this tool you can easily interact with Kubernetes to deploy applications, inspect and manage cluster resources, view logs and perform many more administrative operations.

#### MAC OS X
Download the binary
```bash
curl -o kubectl https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/darwin/amd64/kubectl
```
Make it executable
```bash
chmod +x kubectl
```
Move to the Bin directory
```bash
sudo mv kubectl /usr/local/bin/
```

#### LINUX OR WINDOWS USING GITBASH OR SIMILAR TOOL
Download the binary
```bash
wget https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl
```
Make it executable
```bash
chmod +x kubectl
```
Move to the Bin directory
```bash
sudo mv kubectl /usr/local/bin/
```
Verify that kubectl version 1.21.0 or higher is installed:
```bash
kubectl version --client
```
Output:
```bash
Client Version: version.Info{Major:"1", Minor:"20+", GitVersion:"v1.20.4-dirty", GitCommit:"e87da0bd6e03ec3fea7933c4b5263d151aafd07c", GitTreeState:"dirty", BuildDate:"2021-03-15T10:03:32Z", GoVersion:"go1.16.2", Compiler:"gc", Platform:"darwin/amd64"}
```

![](assets/6.png)

### INSTALL CFSSL AND CFSSLJSON

cfssl is an open source tool by Cloudflare used to setup a Public Key Infrastructure (PKI Infrastructure) for generating, signing and bundling TLS certificates.
In previous projects you have experienced the use of Letsencrypt for the similar use case. Here, cfssl will be configured as a Certificate Authority which will issue the certificates required to spin up a Kubernetes cluster.

Download, install and verify successful installation of cfssl and cfssljson:

#### MAC OS X
```sh
curl -o cfssl https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/darwin/cfssl
curl -o cfssljson https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/darwin/cfssljson
```
```sh
chmod +x cfssl cfssljson
```
```sh
sudo mv cfssl cfssljson /usr/local/bin/
```
If you have issues using the binaries directly, you should consider using the package manager Homebrew and that might be a better option:
```sh
brew install cfssl
```
Verify that cfssl version 1.4.1 or higher is installed:

Output: cfssl version
```
Version: 1.4.1
Runtime: go1.12.12
```
cfssljson --version
```
Version: 1.4.1
Runtime: go1.12.12
```
#### LINUX OR WINDOWS USING GITBASH OR SIMILAR TOOL
```bash
wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssl \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssljson
```
```bash
chmod +x cfssl cfssljson
```
```bash
sudo mv cfssl cfssljson /usr/local/bin/
```

![](assets/7.png)

### AWS CLOUD RESOURCES FOR KUBERNETES CLUSTER

As we already know, we need some machines to run the control plane and the worker nodes.
In this section, you will provision EC2 Instances required to run your K8s cluster.
You can use Terraform for this. But it is highly recommended to start out first with manual provisioning using awscli and have thorough knowledge about the whole setup.
After that, you can destroy the entire project and start all over again using Terraform.
This manual approach will solidify your skills and give you the opportunity to face more challenges.

## STEP 1 - CONFIGURE NETWORK INFRASTRUCTURE
##### VIRTUAL PRIVATE CLOUD - VPC

1. Create a directory named **k8s-cluster-from-ground-up**
```bash
mkdir -p k8s-cluster-from-ground-up
```
2. Create a VPC and store the ID as a variable:
```bash
VPC_ID=$(aws ec2 create-vpc \
  --cidr-block 172.31.0.0/16 \
  --output text --query 'Vpc.VpcId'
  )

```
![](assets/8.png)

3. Tag the VPC so that it is named:
```bash
NAME=k8s-cluster-from-ground-up

aws ec2 create-tags \
  --resources ${VPC_ID} \
  --tags Key=Name,Value=${NAME}
```

![](assets/10.png)

##### DOMAIN NAME SYSTEM - DNS

4. Enable DNS support for your VPC:
```bash
aws ec2 modify-vpc-attribute \
  --vpc-id ${VPC_ID} \
  --enable-dns-support '{"Value": true}'
```
5. Enable DNS support for hostnames:
```bash
aws ec2 modify-vpc-attribute \
  --vpc-id ${VPC_ID} \
  --enable-dns-hostnames '{"Value": true}'
```

![](assets/9.png)

##### AWS Region
6. Set the required region
```bash
AWS_REGION=us-west-2
```
![](assets/11.png)

##### Subnet
7. Create the Subnet:
```bash
SUBNET_ID=$(aws ec2 create-subnet \
  --vpc-id ${VPC_ID} \
  --cidr-block 172.31.0.0/24 \
  --output text --query 'Subnet.SubnetId')
aws ec2 create-tags \
  --resources ${SUBNET_ID} \
  --tags Key=Name,Value=${NAME}
```

![](assets/12.png)

##### INTERNET GATEWAY - IGW
8. Create the Internet Gateway and attach it to the VPC:
```bash
INTERNET_GATEWAY_ID=$(aws ec2 create-internet-gateway \
  --output text --query 'InternetGateway.InternetGatewayId')
aws ec2 create-tags \
  --resources ${INTERNET_GATEWAY_ID} \
  --tags Key=Name,Value=${NAME}
aws ec2 attach-internet-gateway \
  --internet-gateway-id ${INTERNET_GATEWAY_ID} \
  --vpc-id ${VPC_ID}
```

![](assets/13.png)

##### ROUTE TABLES
9. Create route tables, associate the route table to subnet, and create a route to allow external traffic to the Internet through the Internet Gateway:

```bash
ROUTE_TABLE_ID=$(aws ec2 create-route-table \
  --vpc-id ${VPC_ID} \
  --output text --query 'RouteTable.RouteTableId')
aws ec2 create-tags \
  --resources ${ROUTE_TABLE_ID} \
  --tags Key=Name,Value=${NAME}
aws ec2 associate-route-table \
  --route-table-id ${ROUTE_TABLE_ID} \
  --subnet-id ${SUBNET_ID}
aws ec2 create-route \
  --route-table-id ${ROUTE_TABLE_ID} \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id ${INTERNET_GATEWAY_ID}
```

![](assets/14.png)
Output:
```
{
    "AssociationId": "rtbassoc-07a8877e92504def7",
    "AssociationState": {
        "State": "associated"
    }
}
{
    "Return": true
}
```

![](assets/15.png)

##### SECURITY GROUPS
10. Configure security groups
```bash
# Create the security group and store its ID in a variable
SECURITY_GROUP_ID=$(aws ec2 create-security-group \
  --group-name ${NAME} \
  --description "Kubernetes cluster security group" \
  --vpc-id ${VPC_ID} \
  --output text --query 'GroupId')

# Create the NAME tag for the security group
aws ec2 create-tags \
  --resources ${SECURITY_GROUP_ID} \
  --tags Key=Name,Value=${NAME}
  
# Create Inbound traffic for all communication within the subnet to connect on ports used by the master node(s)
aws ec2 authorize-security-group-ingress \
    --group-id ${SECURITY_GROUP_ID} \
    --ip-permissions IpProtocol=tcp,FromPort=2379,ToPort=2380,IpRanges='[{CidrIp=172.31.0.0/24}]'

# # Create Inbound traffic for all communication within the subnet to connect on ports used by the worker nodes
aws ec2 authorize-security-group-ingress \
    --group-id ${SECURITY_GROUP_ID} \
    --ip-permissions IpProtocol=tcp,FromPort=30000,ToPort=32767,IpRanges='[{CidrIp=172.31.0.0/24}]'

# Create inbound traffic to allow connections to the Kubernetes API Server listening on port 6443
aws ec2 authorize-security-group-ingress \
  --group-id ${SECURITY_GROUP_ID} \
  --protocol tcp \
  --port 6443 \
  --cidr 0.0.0.0/0

# Create Inbound traffic for SSH from anywhere (Do not do this in production. Limit access ONLY to IPs or CIDR that MUST connect)
aws ec2 authorize-security-group-ingress \
  --group-id ${SECURITY_GROUP_ID} \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0

# Create ICMP ingress for all types
aws ec2 authorize-security-group-ingress \
  --group-id ${SECURITY_GROUP_ID} \
  --protocol icmp \
  --port -1 \
  --cidr 0.0.0.0/0
```

![](assets/16.png)

##### NETWORK LOAD BALANCER

11. Create a network Load balancer,

```bash
LOAD_BALANCER_ARN=$(aws elbv2 create-load-balancer \
  --name ${NAME} \
  --subnets ${SUBNET_ID} \
  --scheme internet-facing \
  --type network \
  --output text --query 'LoadBalancers[].LoadBalancerArn')
```

![](assets/17.png)


##### TAGRET GROUP
12. Create a target group: (For now it will be unhealthy because there are no real targets yet.)
```bash
TARGET_GROUP_ARN=$(aws elbv2 create-target-group \
  --name ${NAME} \
  --protocol TCP \
  --port 6443 \
  --vpc-id ${VPC_ID} \
  --target-type ip \
  --output text --query 'TargetGroups[].TargetGroupArn')
```

![](assets/18.png)

13. Register targets: (Just like above, no real targets. You will just put the IP addresses so that, when the nodes become available, they will be used as targets.)

```bash
aws elbv2 register-targets \
  --target-group-arn ${TARGET_GROUP_ARN} \
  --targets Id=172.31.0.1{0,1,2}
```

![](assets/19.png)

14. Create a listener to listen for requests and forward to the target nodes on TCP port 6443
```bash
aws elbv2 create-listener \
  --load-balancer-arn ${LOAD_BALANCER_ARN} \
  --protocol TCP \
  --port 6443 \
  --default-actions Type=forward,TargetGroupArn=${TARGET_GROUP_ARN} \
  --output text --query 'Listeners[].ListenerArn'
```

![](assets/20.png)

##### K8S PUBLIC ADDRESS
15. Get the Kubernetes Public address
```bash
KUBERNETES_PUBLIC_ADDRESS=$(aws elbv2 describe-load-balancers \
  --load-balancer-arns ${LOAD_BALANCER_ARN} \
  --output text --query 'LoadBalancers[].DNSName')
```

![](assets/21.png)

## STEP 2 - CREATE COMPUTE RESOURCES

##### AMI
1. Get an image to create EC2 instances:
```bash
IMAGE_ID=$(aws ec2 describe-images --owners 099720109477 \
  --filters \
  'Name=root-device-type,Values=ebs' \
  'Name=architecture,Values=x86_64' \
  'Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-xenial-16.04-amd64-server-*' \
  | jq -r '.Images|sort_by(.Name)[-1]|.ImageId')
```

Install jq (which is used for processing JSON output) if not already installed.
```bash
sudo snap install jq
```

![](assets/22.png)

##### SSH KEY-PAIR
2. Create SSH Key-Pair

```bash
mkdir -p ssh

aws ec2 create-key-pair \
  --key-name ${NAME} \
  --output text --query 'KeyMaterial' \
  > ssh/${NAME}.id_rsa
chmod 600 ssh/${NAME}.id_rsa
```

![](assets/23.png)

##### EC2 INSTANCES FOR CONTROLE PLANE (MASTER NODES)
3. Create 3 Master nodes: 
_Note - Using t3.micro instead of t3.small as t3.micro is covered by AWS free tier_
```bash
for i in 0 1 2; do
  instance_id=$(aws ec2 run-instances \
    --associate-public-ip-address \
    --image-id ${IMAGE_ID} \
    --count 1 \
    --key-name ${NAME} \
    --security-group-ids ${SECURITY_GROUP_ID} \
    --instance-type t3.micro \
    --private-ip-address 172.31.0.1${i} \
    --user-data "name=master-${i}" \
    --subnet-id ${SUBNET_ID} \
    --output text --query 'Instances[].InstanceId')

  aws ec2 modify-instance-attribute \
    --instance-id ${instance_id} \
    --no-source-dest-check

  aws ec2 create-tags \
    --resources ${instance_id} \
    --tags "Key=Name,Value=${NAME}-master-${i}"
done

```

![](assets/24.png)

### EC2 INSTANCES FOR WORKER NODES
1. Create 3 worker nodes:
```bash
for i in 0 1 2; do
  instance_id=$(aws ec2 run-instances \
    --associate-public-ip-address \
    --image-id ${IMAGE_ID} \
    --count 1 \
    --key-name ${NAME} \
    --security-group-ids ${SECURITY_GROUP_ID} \
    --instance-type t3.micro \
    --private-ip-address 172.31.0.2${i} \
    --user-data "name=worker-${i}|pod-cidr=172.20.${i}.0/24" \
    --subnet-id ${SUBNET_ID} \
    --output text --query 'Instances[].InstanceId')

  aws ec2 modify-instance-attribute \
    --instance-id ${instance_id} \
    --no-source-dest-check

  aws ec2 create-tags \
    --resources ${instance_id} \
    --tags "Key=Name,Value=${NAME}-worker-${i}"
done

```

![](assets/25.png)

![](assets/26.png)

## STEP 3 PREPARE THE SELF-SIGNED CERTIFICATE AUTHORITY AND GENERATE TLS CERTIFICATES

The following components running on the Master node will require TLS certificates.
  - kube-controller-manager
  - kube-scheduler
  - etcd
  - kube-apiserver
  - kubelet
  - kube-proxy

The following components running on the Worker nodes will require TLS certificates.

  - kubelet
  - kube-proxy

Therefore, you will provision a PKI Infrastructure using cfssl which will have a Certificate Authority. The CA will then generate certificates for all the individual components.

### SELF-SIGNED ROOT CERTIFICATE AUTHORITY (CA)

Here, you will provision a CA that will be used to sign additional TLS certificates.

Create a directory and cd into it:
```bash
mkdir ca-authority && cd ca-authority
```
Generate the CA configuration file, Root Certificate, and Private key:
```bash
{

cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF

cat > ca-csr.json <<EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "Kubernetes",
      "OU": "Steghub.com DEVOPS",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca

}
```

![](assets/27.png)
The file defines the following:

```
CN – Common name for the authority

algo – the algorithm used for the certificates

size – algorithm size in bits

C – Country

L – Locality (city)

ST – State or province

O – Organization

OU – Organizational Unit
```
Output:
```
2021/05/16 20:18:44 [INFO] generating a new CA key and certificate from CSR
2021/05/16 20:18:44 [INFO] generate received request
2021/05/16 20:18:44 [INFO] received CSR
2021/05/16 20:18:44 [INFO] generating key: rsa-2048
2021/05/16 20:18:44 [INFO] encoded CSR
2021/05/16 20:18:44 [INFO] signed certificate with serial number 478642753175858256977534824638605235819766817855
```

List the directory to see the created files
```bash
ls -ltr
```
![](assets/28.png)

```
-rw-r--r--  1 james  james   232 16 May 20:18 ca-config.json
-rw-r--r--  1 james  james   207 16 May 20:18 ca-csr.json
-rw-r--r--  1 james  james  1306 16 May 20:18 ca.pem
-rw-------  1 james  james  1679 16 May 20:18 ca-key.pem
-rw-r--r--  1 james  james  1001 16 May 20:18 ca.csr
```


The 3 important files here are:
- **ca.pem** - The Root Certificate
- **ca-key.pem** - The Private Key
- **ca.csr** - The Certificate Signing Request

### GENERATING TLS CERTIFICATES FOR CLIENT AND SERVER

You will need to provision Client/Server certificates for all the components. It is a MUST to have encrypted communication within the cluster. Therefore, the server here are the master nodes running the api-server component. While the client is every other component that needs to communicate with the api-server.

Now we have a certificate for the Root CA, we can then begin to request more certificates which the different Kubernetes components, i.e. clients and server, will use to have encrypted communication.

Remember, the clients here refer to every other component that will communicate with the api-server. These are:
- kube-controller-manager
- kube-scheduler
- etcd
- kubelet
- kube-proxy
- Kubernetes Admin User

### LET US BEGIN WITH THE KUBERNETES API-SERVER CERTIFICATE AND PRIVATE KEY

The certificate for the Api-server must have IP addresses, DNS names, and a Load Balancer address included. Otherwise, you will have a lot of difficulties connecting to the api-server.
1. Generate the Certificate Signing Request (CSR), Private Key and the Certificate for the Kubernetes Master Nodes.
```bash
{
cat > master-kubernetes-csr.json <<EOF
{
  "CN": "kubernetes",
   "hosts": [
   "127.0.0.1",
   "172.31.0.10",
   "172.31.0.11",
   "172.31.0.12",
   "ip-172-31-0-10",
   "ip-172-31-0-11",
   "ip-172-31-0-12",
   "ip-172-31-0-10.${AWS_REGION}.compute.internal",
   "ip-172-31-0-11.${AWS_REGION}.compute.internal",
   "ip-172-31-0-12.${AWS_REGION}.compute.internal",
   "${KUBERNETES_PUBLIC_ADDRESS}",
   "kubernetes",
   "kubernetes.default",
   "kubernetes.default.svc",
   "kubernetes.default.svc.cluster",
   "kubernetes.default.svc.cluster.local"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "Kubernetes",
      "OU": "StegHub.com DEVOPS",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  master-kubernetes-csr.json | cfssljson -bare master-kubernetes
}
```

![](assets/29.png)

### CREATING THE OTHER CERTIFICATES: FOR THE FOLLOWING KUBERNETES COMPONENTS:
- Scheduler Client Certificate
- Kube Proxy Client Certificate
- Controller Manager Client Certificate
- Kubelet Client Certificates
- K8s admin user Client Certificate

2. kube-scheduler Client Certificate and Private Key
```bash
{

cat > kube-scheduler-csr.json <<EOF
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "system:kube-scheduler",
      "OU": "Steghub.com DEVOPS",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-scheduler-csr.json | cfssljson -bare kube-scheduler

}
```

<!-- ![](assets/30.png) -->
<!-- ![](assets/31.png) -->

_If you see any warning message, it is safe to ignore it_

3. kube-proxy Client Certificate and Private Key
```bash
{

cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "system:node-proxier",
      "OU": "Steghub.com DEVOPS",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy

}
```

![](assets/32.png)

4. kube-controller-manager Client Certificate and Private Key
```bash
{
cat > kube-controller-manager-csr.json <<EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "system:kube-controller-manager",
      "OU": "Steghub.com DEVOPS",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager

}
```

![](assets/33.png)

5. kubelet Client Certificate and Private Key

Similar to how you configured the api-server's certificate, Kubernetes requires that the hostname of each worker node is included in the client certificate.

Also, Kubernetes uses a special-purpose authorization mode called Node Authorizer, that specifically authorizes API requests made by kubelet services. In order to be authorized by the Node Authorizer, kubelets must use a credential that identifies them as being in the system:nodes group, with a username of system:node:<nodeName>. Notice the "CN": "system:node:${instance_hostname}", in the below code.

Therefore, the certificate to be created must comply to these requirements. In the below example, there are 3 worker nodes, hence we will use bash to loop through a list of the worker nodes' hostnames, and based on each index, the respective Certificate Signing Request (CSR), private key and client certificates will be generated.

```bash
for i in 0 1 2; do
  instance="${NAME}-worker-${i}"
  instance_hostname="ip-172-31-0-2${i}"
  cat > ${instance}-csr.json <<EOF
{
  "CN": "system:node:${instance_hostname}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "system:nodes",
      "OU": "Steghub.com DEVOPS",
      "ST": "London"
    }
  ]
}
EOF

  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')

  internal_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PrivateIpAddress')

  cfssl gencert \
    -ca=ca.pem \
    -ca-key=ca-key.pem \
    -config=ca-config.json \
    -hostname=${instance_hostname},${external_ip},${internal_ip} \
    -profile=kubernetes \
    ${NAME}-worker-${i}-csr.json | cfssljson -bare ${NAME}-worker-${i}
done
```
![](assets/31.png)

6. Finally, kubernetes admin user's Client Certificate and Private Key

```bash
{
cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "system:masters",
      "OU": "Steghub.com DEVOPS",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin
}
```

![](assets/35.png)

7. Actually, we are not done yet!

![](https://media4.giphy.com/media/v1.Y2lkPTc5MGI3NjExcm9uM2Rlb28xYnhhNG4zMHpna29sc3gybzhyYzJwcWVrbzkxd2lucCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/eeL8EcBBTwSMLACw6F/giphy.gif)

There is one more pair of certificate and private key we need to generate. That is for the Token Controller: a part of the Kubernetes Controller Manager kube-controller-manager responsible for generating and signing service account tokens which are used by pods or other resources to establish connectivity to the api-server. Read more about Service Accounts from the official documentation.

Alright, let us quickly create the last set of files, and we are done with PKIs
```bash
{

cat > service-account-csr.json <<EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "Kubernetes",
      "OU": "Steghub.com DEVOPS",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  service-account-csr.json | cfssljson -bare service-account
}
```

![](assets/36.png)

![](assets/34.png)




## STEP 4 - DISTRIBUTING THE CLIENT AND SERVER CERTIFICATES

Now it is time to start sending all the client and server certificates to their respective instances.

Let us begin with the **worker nodes**:

Copy these files securely to the worker nodes using scp utility
- Root CA certificate - ca.pem
- X509 Certificate for each worker node
- Private Key of the certificate for each worker node

```bash
for i in 0 1 2; do
  instance="${NAME}-worker-${i}"
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
  scp -i ../ssh/${NAME}.id_rsa \
    ca.pem ${instance}-key.pem ${instance}.pem ubuntu@${external_ip}:~/; \
done
```
OUTPUT:
```
    ca.pem ${instance}-key.pem ${instance}.pem ubuntu@${external_ip}:~/; \
done
ca.pem  
100% 1350    48.2KB/s   00:00    
k8s-cluster-from-ground-up-worker-0-key.pem 
100% 1675    52.5KB/s   00:00    
k8s-cluster-from-ground-up-worker-0.pem                                      
100% 1594    48.9KB/s   00:00    
ca.pem                                                                               100% 1350    35.9KB/s   00:00    
k8s-cluster-from-ground-up-worker-1-key.pem  
100% 1675    41.6KB/s   00:00    
k8s-cluster-from-ground-up-worker-1.pem   
100% 1594    44.0KB/s   00:00    
ca.pem     
100% 1350    44.7KB/s   00:00    
k8s-cluster-from-ground-up-worker-2-key.pem  
100% 1679    49.2KB/s   00:00    
k8s-cluster-from-ground-up-worker-2.pem   
```

![](assets/35.png)

**Master or Controller node:** - 
_Note that only the api-server related files will be sent over to the master nodes._

```bash
for i in 0 1 2; do
instance="${NAME}-master-${i}" \
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
  scp -i ../ssh/${NAME}.id_rsa \
    ca.pem ca-key.pem service-account-key.pem service-account.pem \
    master-kubernetes.pem master-kubernetes-key.pem ubuntu@${external_ip}:~/;
done
```
Output:
```
ca.pem                                                                               100% 1350     8.4KB/s   00:00    
ca-key.pem  
100% 1675    44.7KB/s   00:00    
service-account-key.pem
100% 1675    45.3KB/s   00:00    
service-account.pem
100% 1440    42.0KB/s   00:00    
master-kubernetes.pem
100% 1956    58.5KB/s   00:00    
master-kubernetes-key.pem
100% 1671    47.5KB/s   00:00    
ca.pem
100% 1350    42.9KB/s   00:00    
ca-key.pem
100% 1675    46.3KB/s   00:00    
service-account-key.pem
100% 1675    44.1KB/s   00:00    
service-account.pem
100% 1440    46.9KB/s   00:00    
master-kubernetes.pem
100% 1956    54.6KB/s   00:00    
master-kubernetes-key.pem
100% 1671    48.7KB/s   00:00    
ca.pem
100% 1350    41.8KB/s   00:00    
ca-key.pem
100% 1675    45.4KB/s   00:00    
service-account-key.pem
100% 1675    52.5KB/s   00:00    
service-account.pem
100% 1440    45.6KB/s   00:00    
master-kubernetes.pem
100% 1956    48.9KB/s   00:00    
master-kubernetes-key.pem  
```

![](assets/36.png)

The _kube-proxy_,_kube-controller-manager_, _kube-scheduler_, and _kubelet_ client certificates will be used to generate client authentication configuration files later.

## STEP 4 USE KUBECTL TO GENERATE KUBERNETES CONFIGURATION FILES FOR AUTHENTICATION

All the work you are doing right now is ensuring that you do not face any difficulties by the time the Kubernetes cluster is up and running. In this step, you will create some files known as kubeconfig, which enables Kubernetes clients to locate and authenticate to the Kubernetes API Servers.

You will need a client tool called kubectl to do this. And, by the way, most of your time with Kubernetes will be spent using kubectl commands.

Now it's time to generate kubeconfig files for the kubelet, controller manager, kube-proxy, and scheduler clients and then the admin user.

First, let us create a few environment variables for reuse by multiple commands.

```bash
KUBERNETES_API_SERVER_ADDRESS=$(aws elbv2 describe-load-balancers --load-balancer-arns ${LOAD_BALANCER_ARN} --output text --query 'LoadBalancers[].DNSName')
```


For each of the nodes running the kubelet component, it is very important that the client certificate configured for that node is used to generate the kubeconfig. This is because each certificate has the node's DNS name or IP Address configured at the time the certificate was generated. It will also ensure that the appropriate authorization is applied to that node through the Node Authorizer

Below command must be run in the directory where all the certificates were generated.
```bash
for i in 0 1 2; do

instance="${NAME}-worker-${i}"
instance_hostname="ip-172-31-0-2${i}"

# Set the kubernetes cluster in the kubeconfig file
  kubectl config set-cluster ${NAME} \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://$KUBERNETES_API_SERVER_ADDRESS:6443 \
    --kubeconfig=${instance}.kubeconfig

# Set the cluster credentials in the kubeconfig file
  kubectl config set-credentials system:node:${instance_hostname} \
    --client-certificate=${instance}.pem \
    --client-key=${instance}-key.pem \
    --embed-certs=true \
    --kubeconfig=${instance}.kubeconfig

# Set the context in the kubeconfig file
  kubectl config set-context default \
    --cluster=${NAME} \
    --user=system:node:${instance_hostname} \
    --kubeconfig=${instance}.kubeconfig

  kubectl config use-context default --kubeconfig=${instance}.kubeconfig
done
```
Output:
```
Cluster "k8s-cluster-from-ground-up" set.

User "system:node:ip-172-31-0-20.eu-central-1.compute.internal" set.

Context "default" created.

Switched to context "default".

Cluster "k8s-cluster-from-ground-up" set.

User "system:node:ip-172-31-0-21.eu-central-1.compute.internal" set.

Context "default" created.
1. Generate the kubelet kubeconfig file

Switched to context "default".

Cluster "k8s-cluster-from-ground-up" set.

User "system:node:ip-172-31-0-22.eu-central-1.compute.internal" set.

Context "default" created.

Switched to context "default".
```

![](assets/37.png)

List the output
```bash
ls -ltr *.kubeconfig
```
OUTPUT:
```
-rw-------  1 james  staff  6602 22 Jun 20:40 k8s-cluster-from-ground-up-worker-0.kubeconfig
-rw-------  1 james  staff  6602 22 Jun 20:40 k8s-cluster-from-ground-up-worker-1.kubeconfig
-rw-------  1 james  staff  6606 22 Jun 20:40 k8s-cluster-from-ground-up-worker-2.kubeconfig
```

![](assets/40.png)

Open up the kubeconfig files generated and review the 3 different sections that have been configured:
- Cluster
- Credentials
- And Kube Context

Kubeconfig file is used to organize information about clusters, users, namespaces and authentication mechanisms. By default, kubectl looks for a file named config in the $HOME/.kube directory. You can specify other kubeconfig files by setting the KUBECONFIG environment variable or by setting the --kubeconfig flag. To get to know more how to create your own kubeconfig files - read this documentation.

**Context** part of kubeconfig file defines three main parameters: cluster, namespace and user. You can save several different contexts with any convenient names and switch between them when needed.

```bash
kubectl config use-context %context-name%
```
2. Generate the kube-proxy kubeconfig
```bash
{
  kubectl config set-cluster ${NAME} \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_API_SERVER_ADDRESS}:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.pem \
    --client-key=kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=${NAME} \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
}
```
![](assets/41.png)
3. Generate the Kube-Controller-Manager kubeconfig

Notice that the --server is set to use 127.0.0.1. This is because, this component runs on the API-Server so there is no point routing through the Load Balancer.

```bash
{
  kubectl config set-cluster ${NAME} \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.pem \
    --client-key=kube-controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=${NAME} \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
}
```

![](assets/42.png)

4. Generating the Kube-Scheduler Kubeconfig
```bash
{
  kubectl config set-cluster ${NAME} \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.pem \
    --client-key=kube-scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=${NAME} \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
}
```

![](assets/43.png)

5. Finally, generate the kubeconfig file for the admin user
```bash
{
  kubectl config set-cluster ${NAME} \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_API_SERVER_ADDRESS}:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=${NAME} \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default --kubeconfig=admin.kubeconfig
}
```

![](assets/44.png)

![](assets/45.png)

**TASK:**_Distribute the files to their respective servers, using scp and a for loop like we have done previously. This is a test to validate that you understand which component must go to which node._

## STEP 5 PREPARE THE ETCD DATABASE FOR ENCRYPTION AT REST.

Kubernetes uses etcd (A distributed key value store) to store variety of data which includes the cluster state, application configurations, and secrets. By default, the data that is being persisted to the disk is not encrypted. Any attacker that is able to gain access to this database can exploit the cluster since the data is stored in plain text. Hence, it is a security risk for Kubernetes that needs to be addressed.

To mitigate this risk, we must prepare to encrypt etcd at rest. "At rest" means data that is stored and persists on a disk. Anytime you hear "in-flight" or "in transit" refers to data that is being transferred over the network. "In-flight" encryption is done through TLS.

##### GENERATE THE ENCRYPTION KEY AND ENCODE IT USING BASE64
```
ETCD_ENCRYPTION_KEY=$(head -c 64 /dev/urandom | base64) 
```
See the output that will be generated when called. Yours will be a different random string.

```bash
echo $ETCD_ENCRYPTION_KEY
```

OUTPUT:
```
OuxSvV5XUQVid4fNNbeyFEDTUPr1yozZPQ+E6Eqj80m1FSVDB6jOHt9miD/7kMdJIvVshlMgxY80wFajlqItug===$
```

![](assets/46.png)

** Create an encryption-config.yaml file as documented officially by kubernetes
```bash
cat > encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ETCD_ENCRYPTION_KEY}
      - identity: {}
EOF
```

![](assets/47.png)

Send the encryption file to the **Controller nodes** using scp and a for loop.

### BOOTSTRAP ETCD CLUSTER

**TIPS:** _Use a terminal multi-plexer like multi-tabbed putty or tmux to work with multiple terminal sessions simultaneously. It will make your life easier, especially when you need to work on multiple nodes and run the same command across all nodes. Imagine repeating the same commands on 10 different nodes, and you don not intend to start automating with a configuration management tool like Ansible yet._

The primary purpose of the etcd component is to store the state of the cluster. This is because Kubernetes itself is stateless. Therefore, all its stateful data will persist in etcd. Since Kubernetes is a distributed system - it needs a distributed storage to keep persistent data in it. etcd is a highly-available key value store that fits the purpose. All K8s cluster configurations are stored in a form of key value pairs in etcd, it also stores the actual and desired states of the cluster. etcd cluster is intelligent enough to watch for changes made on one instance and almost instantly replicate those changes to the rest of the instances, so all of them will be always reconciled.

**NOTE:** _Don not just copy and paste the commands, ensure that you go through each and understand exactly what they will do on your servers. Use tools like tmux to make it easy to run commands on multiple terminal screens at once._

Installing tmux
```brew
# On macOS
brew install tmux
```
```bash
# On Linux
sudo apt update
sudo apt install tmux
```
##### Using tmux
- Start a new session: `tmux`
- To start a session with a name: `tmux new -s mysession`

Detaching and Reattaching Sessions
- To detach from a session (leave it running in the background): Press Ctrl-b then d
- To list all sessions: tmux ls
- To reattach to a session: tmux attach -t mysession

Windows and Panes
- To create a new window: `Press Ctrl-b then c`
- To switch between windows: `Press Ctrl-b then a number (0, 1, 2, etc.)`
- To split a window into panes:
    - Vertically: `Ctrl-b then %`
    - Horizontally: `Ctrl-b then "`
- To switch between panes: `Press Ctrl-b then use the arrow keys`

Exiting tmux
- To close a pane or window, simply exit the shell running in it (type exit or use Ctrl-d).
- To kill a session: tmux kill-session -t mysession

##### Enabling Input Broadcasting

Broadcast to All Panes in a Window:
- To broadcast input to all panes in the current window
  - `Press Ctrl-b then` : to bring up the command prompt.
  - Type `setw synchronize-panes on` and `press Enter`.

With input synchronization enabled, any command you type in one pane will be replicated in all panes in the current window.

Disable Broadcasting
- To stop broadcasting input to all panes, use the same steps:
  - `Press Ctrl-b the`n `:`.
  - Type `setw synchronize-panes off` and press `Enter`.

1. SSH into the controller server

  - Master-1
```bash
  master_1_ip=$(aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=${NAME}-master-0" \
  --output text --query 'Reservations[].Instances[].PublicIpAddress')
  ssh -i k8s-cluster-from-ground-up.id_rsa ubuntu@${master_1_ip}
```
  - Master-2
```bash
  master_2_ip=$(aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=${NAME}-master-1" \
  --output text --query 'Reservations[].Instances[].PublicIpAddress')
  ssh -i k8s-cluster-from-ground-up.id_rsa ubuntu@${master_2_ip}
```
  - Master-3
```bash
  master_3_ip=$(aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=${NAME}-master-2" \
  --output text --query 'Reservations[].Instances[].PublicIpAddress')
  ssh -i k8s-cluster-from-ground-up.id_rsa ubuntu@${master_3_ip}
```
You should have a a similar pane like below. 
You should be able to see all the files that have been sent to the nodes.

![](assets/48.png)

2. Download and install etcd
```bash
  wget -q --show-progress --https-only --timestamping \
  "https://github.com/etcd-io/etcd/releases/download/v3.4.15/etcd-v3.4.15-linux-amd64.tar.gz"
```
3. Extract and install the etcd server and the etcdctl command line utility:
```bash
{
  tar -xvf etcd-v3.4.15-linux-amd64.tar.gz
  sudo mv etcd-v3.4.15-linux-amd64/etcd* /usr/local/bin/
}
```
4. Configure the etcd server
```bash
{
  sudo mkdir -p /etc/etcd /var/lib/etcd
  sudo chmod 700 /var/lib/etcd
  sudo cp ca.pem master-kubernetes-key.pem master-kubernetes.pem /etc/etcd/
}
```

5. The instance internal IP address will be used to serve client requests and communicate with etcd cluster peers. Retrieve the internal IP address for the current compute instance:
```bash
export INTERNAL_IP=$(curl -s http://169.254.169.254/latest/meta-data/local-ipv4)
```

6. Each etcd member must have a unique name within an etcd cluster. Set the etcd name to node Private IP address so it will uniquely identify the machine:
```bash
ETCD_NAME=$(curl -s http://169.254.169.254/latest/user-data/ \
  | tr "|" "\n" | grep "^name" | cut -d"=" -f2)
  
echo ${ETCD_NAME}
```

![](assets/49.png)

7. Create the etcd.service systemd unit file:

The flags are well documented [here](https://www.bookstack.cn/read/etcd-3.2.17-en/717bafd59fa87192.md)

```bash
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster master-0=https://172.31.0.10:2380,master-1=https://172.31.0.11:2380,master-2=https://172.31.0.12:2380 \\
  --cert-file=/etc/etcd/master-kubernetes.pem \\
  --key-file=/etc/etcd/master-kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/master-kubernetes.pem \\
  --peer-key-file=/etc/etcd/master-kubernetes-key.pem \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
 
 ![](assets/50.png)   

8. Start and enable the etcd Server
```bash
{
  sudo systemctl daemon-reload
  sudo systemctl enable etcd
  sudo systemctl start etcd
}
```

![](assets/51.png)

9. Verify the etcd installation
```bash
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/master-kubernetes.pem \
  --key=/etc/etcd/master-kubernetes-key.pem
```
Output:
```
6709c481b5234095, started, master-0, https://172.31.0.10:2380, https://172.31.0.10:2379, false
ade74a4f39c39f33, started, master-1, https://172.31.0.11:2380, https://172.31.0.11:2379, false
ed33b44c0b153ee3, started, master-2, https://172.31.0.12:2380, https://172.31.0.12:2379, false
```

```bash
systemctl status etcd
```

![](assets/52.png)

### BOOTSTRAP THE CONTROL PLANE

In this section, you will configure the components for the control plane on the master/controller nodes.
1. Create the Kubernetes configuration directory:
```bash
sudo mkdir -p /etc/kubernetes/config
```
2. Download the official Kubernetes release binaries:
```bash
wget -q --show-progress --https-only --timestamping \
  "https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kube-apiserver" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kube-controller-manager" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kube-scheduler" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl"
```
![](assets/53.png)

3. Install the Kubernetes binaries:
```bash
{
  chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl
  sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
}
```

4. Configure the Kubernetes API Server:
```bash
{
  sudo mkdir -p /var/lib/kubernetes/

  sudo mv ca.pem ca-key.pem master-kubernetes-key.pem master-kubernetes.pem \
    service-account-key.pem service-account.pem \
    encryption-config.yaml /var/lib/kubernetes/
}
```

The instance internal IP address will be used to advertise the API Server to members of the cluster. Retrieve the internal IP address for the current compute instance:

```bash
export INTERNAL_IP=$(curl -s http://169.254.169.254/latest/meta-data/local-ipv4)
export export KUBERNETES_API_SERVER_ADDRESS="k8s-api-server.svc.steghub.com"
```

![](assets/54.png)

Create the kube-apiserver.service systemd unit file: Ensure to read each startup flag used in below systemd file from the documentation here

```bash
cat <<EOF | sudo tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\
  --enable-admission-plugins=NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
  --etcd-cafile=/var/lib/kubernetes/ca.pem \\
  --etcd-certfile=/var/lib/kubernetes/master-kubernetes.pem \\
  --etcd-keyfile=/var/lib/kubernetes/master-kubernetes-key.pem\\
  --etcd-servers=https://172.31.0.10:2379,https://172.31.0.11:2379,https://172.31.0.12:2379 \\
  --event-ttl=1h \\
  --encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
  --kubelet-client-certificate=/var/lib/kubernetes/master-kubernetes.pem \\
  --kubelet-client-key=/var/lib/kubernetes/master-kubernetes-key.pem \\
  --runtime-config='api/all=true' \\
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \\
  --service-account-signing-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-account-issuer=https://${INTERNAL_IP}:6443 \\
  --service-cluster-ip-range=172.32.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/master-kubernetes.pem \\
  --tls-private-key-file=/var/lib/kubernetes/master-kubernetes-key.pem \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

![](assets/55.png)

5. Configure the Kubernetes Controller Manager:
Move the kube-controller-manager kubeconfig into place:
```bash
sudo mv kube-controller-manager.kubeconfig /var/lib/kubernetes/
```
Export some variables to retrieve the vpc_cidr - This will be required for the bind-address flag:
```bash
export AWS_METADATA="http://169.254.169.254/latest/meta-data"
export EC2_MAC_ADDRESS=$(curl -s $AWS_METADATA/network/interfaces/macs/ | head -n1 | tr -d '/')
export VPC_CIDR=$(curl -s $AWS_METADATA/network/interfaces/macs/$EC2_MAC_ADDRESS/vpc-ipv4-cidr-block/)
export NAME=k8s-cluster-from-ground-up
```

Create the kube-controller-manager.service systemd unit file:
```bash
cat <<EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --bind-address=0.0.0.0 \\
  --cluster-cidr=${VPC_CIDR} \\
  --cluster-name=${NAME} \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \\
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --authentication-kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --authorization-kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.pem \\
  --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-cluster-ip-range=172.32.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

6. Configure the Kubernetes Scheduler:

Move the kube-scheduler kubeconfig into place:
```bash
sudo mv kube-scheduler.kubeconfig /var/lib/kubernetes/
sudo mkdir -p /etc/kubernetes/config
```

Create the kube-scheduler.yaml configuration file:
```bash
cat <<EOF | sudo tee /etc/kubernetes/config/kube-scheduler.yaml
apiVersion: kubescheduler.config.k8s.io/v1beta1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
leaderElection:
  leaderElect: true
EOF
```

![](assets/56.png)

Create the kube-scheduler.service systemd unit file:
```bash
cat <<EOF | sudo tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --config=/etc/kubernetes/config/kube-scheduler.yaml \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

![](assets/57.png)

7. Start the Controller Services
```bash
{
  sudo systemctl daemon-reload
  sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
  sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
}
```

![](assets/58.png)

Check the status of the services. Start with the kube-scheduler and kube-controller-manager. It may take up to 20 seconds for kube-apiserver to be fully loaded.
```bash
{ 
  sudo systemctl status kube-apiserver 
  sudo systemctl status kube-controller-manager 
  sudo systemctl status kube-scheduler 
}
```

![](assets/59.png)

**NOTE:** _There is a trap in the entire setup you have been going through, and so the api-server will not start up on your server if you have followed the exact steps so far. As a DevOps engineer, you must be able to solve problems._

**HINTS:**
  1. The problem relates to etcd configuration.
  2. Check the systemd logs for the api-server. The problem will be clearly logged, and it will give you an idea what is wrong. Find out how to fix it.

![](assets/59.png)

![](https://media0.giphy.com/media/v1.Y2lkPTc5MGI3NjExNXVnZnlyYmMycGxhbjZuczFtNWpwcHJwZ3d5aTR4eWFoNmhnZXBuMyZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/26Ff8xSyYAZtNsDaE/giphy.gif)

## TEST THAT EVERYTHING IS WORKING FINE
1. To get the cluster details run:
```bash
kubectl cluster-info  --kubeconfig admin.kubeconfig
```
OUTPUT:
```
Kubernetes control plane is running at https://k8s-api-server.svc.steghub.com:6443
To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

![](assets/61.png)

2. To get the current namespaces:
```bash
kubectl get namespaces --kubeconfig admin.kubeconfig
```


```
OUTPUT:

NAME              STATUS   AGE
default           Active   22m
kube-node-lease   Active   22m
kube-public       Active   22m
kube-system       Active   22m
```

![](assets/62.png)

3. To reach the Kubernetes API Server publicly
```bash
curl --cacert /var/lib/kubernetes/ca.pem https://$INTERNAL_IP:6443/version
```


```
OUTPUT:

{
  "major": "1",
  "minor": "21",
  "gitVersion": "v1.21.0",
  "gitCommit": "cb303e613a121a29364f75cc67d3d580833a7479",
  "gitTreeState": "clean",
  "buildDate": "2021-04-08T16:25:06Z",
  "goVersion": "go1.16.1",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

![](assets/63.png)

4. To get the status of each component:
```bash
kubectl get componentstatuses --kubeconfig admin.kubeconfig
```

![](assets/64.png)

![](assets/65.png)

![](assets/66.png)

5. On one of the controller nodes, configure Role Based Access Control (RBAC) so that the api-server has necessary authorization for for the kubelet.

Create the **ClusterRole**:
```bash
cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
EOF
```

![](assets/67.png)

Create the **ClusterRoleBinding** to bind the kubernetes user with the role created above:
```bash
cat <<EOF | kubectl --kubeconfig admin.kubeconfig  apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF
```

![](assets/68.png)

## CONFIGURING THE KUBERNETES WORKER NODES

Before we begin to bootstrap the worker nodes, it is important to understand that the K8s API Server authenticates to the kubelet as the kubernetes user using the same kubernetes.pem certificate.

We need to configure Role Based Access (RBAC) for Kubelet Authorization:
1. Configure RBAC permissions to allow the Kubernetes API Server to access the Kubelet API on each worker node. Access to the Kubelet API is required for retrieving metrics, logs, and executing commands in pods.

Create the *system:kube-apiserver-to-kubelet* ClusterRole with permissions to access the Kubelet API and perform most common tasks associated with managing pods on the worker nodes:

Run the below script on the Controller node:
```bash
cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
EOF
```

![](assets/69.png)

2. Bind the system:kube-apiserver-to-kubelet ClusterRole to the kubernetes user so that API server can authenticate successfully to the kubelets on the worker nodes:

```bash
cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF
```

![](assets/70.png)

### BOOTSTRAPING COMPONENTS ON THE WORKER NODES

The following components will be installed on each node:
- kubelet
- kube-proxy
- Containerd or Docker
- Networking plugins

1. SSH into the worker nodes
  - Worker-1

```bash
  worker_1_ip=$(aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=${NAME}-worker-0" \
  --output text --query 'Reservations[].Instances[].PublicIpAddress')
  ssh -i k8s-cluster-from-ground-up.id_rsa ubuntu@${worker_1_ip}
```
  - Worker-2
```bash
  worker_2_ip=$(aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=${NAME}-worker-1" \
  --output text --query 'Reservations[].Instances[].PublicIpAddress')
  ssh -i k8s-cluster-from-ground-up.id_rsa ubuntu@${worker_2_ip}
```
  - Worker-3
```bash
  worker_3_ip=$(aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=${NAME}-worker-2" \
  --output text --query 'Reservations[].Instances[].PublicIpAddress')
  ssh -i k8s-cluster-from-ground-up.id_rsa ubuntu@${worker_3_ip}
```

![](assets/71.png)

2. Install OS dependencies:
```bash
{
  sudo apt-get update
  sudo apt-get -y install socat conntrack ipset
}
```

##### MORE ABOUT THE DEPENDENCIES:
- **socat.** Socat is the default implementation for Kubernetes port-forwarding when using dockershim for the kubelet runtime. You will get to experience port-forwarding with Kubernetes in the next project. But what is Dockershim?
- **Dockershim** was a temporary solution proposed by the Kubernetes community to add support for Docker so that it could serve as its container runtime. You should always remember that Kubernetes can use different container runtime to run containers inside its pods. For many years, Docker has been adopted widely and has been used as the container runtime for kubernetes. Hence the implementation that allowed docker is called the Dockershim. If you check the source code of Dockershim, you will see that socat was used to implement the port-forwarding functionality.
- **conntrack** Connection tracking (“conntrack”) is a core feature of the Linux kernel's networking stack. It allows the kernel to keep track of all logical network connections or flows, and thereby identify all of the packets which make up each flow so they can be handled consistently together. It is essential for performant complex networking of Kubernetes where nodes need to track connection information between thousands of pods and services.
- **ipset** is an extension to iptables which is used to configure firewall rules on a Linux server. ipset is a module extension to iptables that allows firewall configuration on a "set" of IP addresses. Compared with how iptables does the configuration linearly, ipset is able to store sets of addresses and index the data structure, making lookups very efficient, even when dealing with large sets. Kubernetes uses ipsets to implement a distributed firewall solution that enforces network policies within the cluster. This can then help to further restrict communications across pods or namespaces. For example, if a namespace is configured with DefaultDeny isolation type (Meaning no connection is allowed to the namespace from another namespace), network policies can be configured in the namespace to whitelist the traffic to the pods in that namespace.

##### QUICK OVERVIEW OF KUBERNETES NETWORK POLICY AND HOW IT IS IMPLEMENTED

Kubernetes network policies are application centric compared to infrastructure/network centric standard firewalls. There are no explicit CIDR or IP used for matching source or destination IP’s. Network policies build up on labels and selectors which are key concepts of Kubernetes that are used for proper organization (for e.g dedicating a namespace to data layer and controlling which app is able to connect there). A typical network policy that controls who can connect to the database namespace will look like below:
```bash
apiVersion: extensions/v1beta1
kind: NetworkPolicy
metadata:
  name: database-network-policy
  namespace: tooling-db
spec:
  podSelector:
    matchLabels:
      app: mysql
  ingress:
   - from:
     - namespaceSelector:
       matchLabels:
         app: tooling
     - podSelector:
       matchLabels:
       role: frontend
   ports:
     - protocol: tcp
     port: 3306
```

**NOTE:** _Best practice is to use solutions like RDS for database implementation. So the above is just to help you understand the concept._

3. Disable Swap

If **(swap)** is not disabled, kubelet will not start. It is highly recommended to allow Kubernetes to handle resource allocation.

Test if swap is already enabled on the host:
```bash
sudo swapon --show
```
If there is no output, then you are good to go.
Otherwise, run below command to turn it off
```bash
sudo swapoff -a
```
4. Download and install a container runtime. (Docker Or Containerd)

Before you install any container runtime, you need to understand that Docker is now deprecated, and Kubernetes no longer supports the Dockershim codebase from v1.20 release

[Read more about this notice here](https://kubernetes.io/blog/2020/12/02/dockershim-faq/)
If you install Docker, it will work. But be aware of this huge change.

- Docker
```bash
sudo apt update -y && \
sudo apt -y install apt-transport-https ca-certificates curl software-properties-common && \
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - && \
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable" && \
sudo apt update -y && \
apt-cache policy docker-ce && \
sudo apt -y install docker-ce && \
sudo usermod -aG docker ${USER} && \
sudo systemctl status docker
```

![](assets/72.png)

**NOTE:** _exit the shell and log back in. Otherwise, you will face a permission denied error. Alternatively, you can run newgrp docker without exiting the shell. But you will need to provide the password of the logged in user_

- Containerd

Download binaries for runc, cri-ctl, and containerd
```bash
  wget https://github.com/opencontainers/runc/releases/download/v1.0.0-rc93/runc.amd64 \
  https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.21.0/crictl-v1.21.0-linux-amd64.tar.gz \
  https://github.com/containerd/containerd/releases/download/v1.4.4/containerd-1.4.4-linux-amd64.tar.gz 
```


Configure containerd:
```bash
{
  mkdir containerd
  tar -xvf crictl-v1.21.0-linux-amd64.tar.gz
  tar -xvf containerd-1.4.4-linux-amd64.tar.gz -C containerd
  sudo mv runc.amd64 runc
  chmod +x  crictl runc  
  sudo mv crictl runc /usr/local/bin/
  sudo mv containerd/bin/* /bin/
}
```

![](assets/73.png)

```bash
sudo mkdir -p /etc/containerd/
```
```bash
cat << EOF | sudo tee /etc/containerd/config.toml
[plugins]
  [plugins.cri.containerd]
    snapshotter = "overlayfs"
    [plugins.cri.containerd.default_runtime]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runc"
      runtime_root = ""
EOF
```

![](assets/74.png)

Create the containerd.service systemd unit file:
```bash
cat <<EOF | sudo tee /etc/systemd/system/containerd.service
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target

[Service]
ExecStartPre=/sbin/modprobe overlay
ExecStart=/bin/containerd
Restart=always
RestartSec=5
Delegate=yes
KillMode=process
OOMScoreAdjust=-999
LimitNOFILE=1048576
LimitNPROC=infinity
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF
```

![](assets/75.png)

5. Create directories for to configure kubelet, kube-proxy, cni, and a directory to keep the kubernetes root ca file:

```bash
sudo mkdir -p \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

6. Download and Install CNI

CNI (Container Network Interface), a Cloud Native Computing Foundation project, consists of a specification and libraries for writing plugins to configure network interfaces in Linux containers. It also comes with a number of plugins.

Kubernetes uses CNI as an interface between network providers and Kubernetes Pod networking. Network providers create network plugin that can be used to implement the Kubernetes networking, and includes additional set of rich features that Kubernetes does not provide out of the box.

Download the plugins available from containernetworking's GitHub repo and read more about CNIs and why it is being developed.

```bash
wget -q --show-progress --https-only --timestamping \
  https://github.com/containernetworking/plugins/releases/download/v0.9.1/cni-plugins-linux-amd64-v0.9.1.tgz
```

Install CNI into /opt/cni/bin/
```bash
sudo tar -xvf cni-plugins-linux-amd64-v0.9.1.tgz -C /opt/cni/bin/
```
The output shows the plugins that comes with the CNI.
```bash
./
./macvlan
./flannel
./static
./vlan
./portmap
./host-local
./vrf
./bridge
./tuning
./firewall
./host-device
./sbr
./loopback
./dhcp
./ptp
./ipvlan
./bandwidth
```

![](assets/76.png)

There are few other plugins that are not included in the CNI, which are also widely used in the industry. They all have their unique implementation approach and set of features.

Click to read more about each of the network plugins below:
- [Calico](https://www.projectcalico.org/)
- [Weave Net](https://www.projectcalico.org/)
- [flannel](https://github.com/flannel-io/flannel)

#### SOURCE

Sometimes you can combine more than one plugin together to maximize the use of features from different providers. Or simply use a CNI network provider such as canal that gives you the best of Flannel and Calico.

7. Download binaries for kubectl, kube-proxy, and kubelet

```bash
wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubelet
```

![](assets/77.png)

8. Install the downloaded binaries
```bash
{
  chmod +x  kubectl kube-proxy kubelet  
  sudo mv  kubectl kube-proxy kubelet /usr/local/bin/
}
```



##### CONFIGURE THE WORKER NODES COMPONENTS

9. Configure kubelet:

In the home directory, you should have the certificates and kubeconfig file for each node. A list in the home folder should look like below:

##### CONFIGURING THE NETWORK

Get the `POD_CIDR` that will be used as part of network configuration

```bash
POD_CIDR=$(curl -s http://169.254.169.254/latest/user-data/ \
  | tr "|" "\n" | grep "^pod-cidr" | cut -d"=" -f2)
echo "${POD_CIDR}"
```

![](assets/79.png)

![](assets/78.png)

In case you are wondering where this $POD_CIDR is coming from. Well, this was configured at the time of creating the worker nodes. Remember the for loop below? The --user-data flag is where we specified what we want the POD_CIDR to be. It is very important to ensure that the CIDR does not overlap with EC2 IPs within the subnet. In the real world, this will be decided in collaboration with the Network team.

Why do we need a network plugin? And why network configuration is crucial to implementing a Kubernetes cluster?

First, let us understand the Kubernetes networking model:

The networking model assumes a flat network, in which containers and nodes can communicate with each other. That means, regardless of which node is running the container in the cluster, Kubernetes expects that all the containers must be able to communicate with each other. Therefore, any network interface used for a Kubernetes implementation must follow this requirement. Otherwise, containers running in pods will not be able to communicate. Of course, this has security concerns. Because if an attacker is able to get into the cluster through a compromised container, then the entire cluster can be exploited.

To mitigate security risks and have a better controlled network topology, Kubernetes uses CNI (Container Network Interface) to manage Network Policies which can be used to operate the Pod network through external plugins such as Calico, Flannel or Weave Net to name a few. With these, you can set policies similar to how you would configure segurity groups in AWS and limit network communications through either cidr ipBlock, namespaceSelectors, or podSelectors, you will see more of these concepts further on.

To really understand Kubernetes further, let us explore some basic concepts around its networking:

##### PODS:

A Pod is the basic building block of Kubernetes; it is the smallest and simplest unit in the Kubernetes object model that you create or deploy. A Pod represents a running process on your cluster. It encapsulates a container running an application such as the Tooling website (or, in some cases, multiple containers), storage resources, a unique network IP, and options that govern how the container(s) should run. All the containers running inside a Pod can reach each other on localhost.

For example, if you deploy both Tooling and MySQL containers inside the same pod, then both of them are considered running on localhost. Although this design pattern is not ideal. Most likely they will run in separate Pods. In most cases one Pod contains just one container, but there are some design patterns that imply multi-container pods (e.g. sidecar, ambassador, adapter) - you can read more about them in this article.

For a better understanding, of Kubernetes networking, let us assume that we have 2-containers in a single Pod and we have 2 such Pods (we can actually have as many pods of the same composition as our node resources would allow).

Network configuration will look like this:

![](assets/k8s-network-config.png)

Notice, that both containers share a single virtual network interface veth0 that belongs to a virtual network within a single node. This virtual interface veth0 is used to allow communication from a pod to the outer world through a bridge cbr0 (custom bridge). This bridge is an interface that forwards the traffic from the Pods on one node to other nodes through a physical network interface eth0. Routing between the nodes is done by means of a router with the routing table.

For more detailed explanation of different aspects of Kubernetes networking - watch this video.

##### POD NETWORK

You must decide on the Pod CIDR per worker node. Each worker node will run multiple pods, and each pod will have its own IP address. IP address of a particular Pod on worker node 1 should be able to communicate with the IP address of another particular Pod on worker node 2. For this to become possible, there must be a bridge network with virtual network interfaces that connects them all together. Here is an interesting read that goes a little deeper into how it works Bookmark that page and read it over and over again after you have completed this project

10. Configure the bridge and loopback networks

Bridge:
```bash
cat > 172-20-bridge.conf <<EOF
{
    "cniVersion": "0.3.1",
    "name": "bridge",
    "type": "bridge",
    "bridge": "cnio0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "ranges": [
          [{"subnet": "${POD_CIDR}"}]
        ],
        "routes": [{"dst": "0.0.0.0/0"}]
    }
}
EOF
```

Loopback:
```bash
cat > 99-loopback.conf <<EOF
{
    "cniVersion": "0.3.1",
    "type": "loopback"
}
EOF
```

![](assets/80.png)

11. Move the files to the network configuration directory:

```bash
sudo mv 172-20-bridge.conf 99-loopback.conf /etc/cni/net.d/
```


12. Store the worker's name in a variable:
```bash
NAME=k8s-cluster-from-ground-up
WORKER_NAME=${NAME}-$(curl -s http://169.254.169.254/latest/user-data/ \
  | tr "|" "\n" | grep "^name" | cut -d"=" -f2)
echo "${WORKER_NAME}"
```

![](assets/81.png)

13. Move the certificates and kubeconfig file to their respective configuration directories:
```bash
sudo mv ${WORKER_NAME}-key.pem ${WORKER_NAME}.pem /var/lib/kubelet/
sudo mv ${WORKER_NAME}.kubeconfig /var/lib/kubelet/kubeconfig
sudo mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
sudo mv ca.pem /var/lib/kubernetes/
```

14. Create the kubelet-config.yaml file

Ensure the needed variables exist:
```bash
NAME=k8s-cluster-from-ground-up
WORKER_NAME=${NAME}-$(curl -s http://169.254.169.254/latest/user-data/ \
  | tr "|" "\n" | grep "^name" | cut -d"=" -f2)
echo "${WORKER_NAME}"
```
```bash
cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.32.0.10"
resolvConf: "/etc/resolv.conf"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/${WORKER_NAME}.pem"
tlsPrivateKeyFile: "/var/lib/kubelet/${WORKER_NAME}-key.pem"
EOF
```

![](assets/82.png)

Let us talk about the configuration file kubelet-config.yaml and the actual configuration for a bit. Before creating the systemd file for kubelet, it is recommended to create the kubelet-config.yaml and set the configuration there rather than using multiple startup flags in systemd. You will simply point to the yaml file.

The config file specifies where to find certificates, the DNS server, and authentication information. As you already know, kubelet is responsible for the containers running on the node, regardless if the runtime is Docker or Containerd; as long as the containers are being created through Kubernetes, kubelet manages them. If you run any docker or cri commands directly on a worker to create a container, bear in mind that Kubernetes is not aware of it, therefore kubelet will not manage those. Kubelet's major responsibility is to always watch the containers in its care, by default every 20 seconds, and ensuring that they are always running. Think of it as a process watcher.

The clusterDNS is the address of the DNS server. As of Kubernetes v1.12, CoreDNS is the recommended DNS Server, hence we will go with that, rather than using legacy kube-dns.

**Note:** _The CoreDNS Service is named kube-dns(When you see kube-dns, just know that it is using CoreDNS). This is more of a backward compatibility reasons for workloads that relied on the legacy kube-dns Service name._

In Kubernetes, Pods are able to find each other using service names through the internal DNS server. Every time a service is created, it gets registered in the DNS server.

In Linux, the /etc/resolv.conf file is where the DNS server is configured. If you want to use Google's public DNS server (8.8.8.8) your /etc/resolv.conf file will have following entry:

_nameserver 8.8.8.8_

In Kubernetes, the kubelet process on a worker node configures each pod. Part of the configuration process is to create the file /etc/resolv.conf and specify the correct DNS server.

15. Configure the kubelet systemd service

```bash
cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service
[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --cluster-domain=cluster.local \\
  --container-runtime=remote \\
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5
[Install]
WantedBy=multi-user.target
EOF
```

![](assets/83.png)

16. Create the kube-proxy.yaml file
```bash
cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "172.31.0.0/16"
EOF
```

17. Configure the Kube Proxy systemd service

```bash
cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes
[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5
[Install]
WantedBy=multi-user.target
EOF
```

![](assets/84.png)

18. Reload configurations and start both services
```bash
{
  sudo systemctl daemon-reload
  sudo systemctl enable containerd kubelet kube-proxy
  sudo systemctl start containerd kubelet kube-proxy
}
```

![](assets/85.png)

![](https://media2.giphy.com/media/glvNGHmbZwgrKH4YYA/200.webp?cid=790b7611b227y6aq137ub0bqlikv7vi4vdmfy9uoez4dumcx&ep=v1_gifs_search&rid=200.webp&ct=g)

Now you should have the worker nodes joined to the cluster, and in a READY state.

**Troubleshooting Tips**: If you have issues at this point. Consider the below:

1. Use _journalctl -u <service name>_ to get the log output and read what might be wrong with starting up the service. You can redirect the output into a file and analyse it.
2. Review your PKI setup again. Ensure that the certificates you generated have the hostnames properly configured.
3. It is okay to start all over again. Each time you attempt the solution is an opportunity to learn something.

# CONGRATULATIONS!

We have built our first Kubernetes cluster from the ground up! It wasn’t easy, but we gained a deep understanding of how K8s components work together. This experience not only helps with setting up clusters but also with maintaining and troubleshooting them in real-world scenarios.

On to the next project—more Kubernetes and exciting technologies ahead!

