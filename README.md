# AWS-Web-Application

## Overview and Objectives 
The hypothetical scenario is about a school called XYZ University that is preparing for the new school year and there have been complaints that their web application is slow or not available during the admissions period. XYZ University wants their framework to be able to handle peak admissions periods, support thousands of users, and be highly available, scalable, load balanced, secure, and high performing. 

To preface, my web application could be expanded and more sophisticated but my framework is very barebones with only the necessary EC2 instances, autoscaling, and load balancing services.  

## Task 1 Create a VPC
When launching into the AWS console, the first thing I did was create a VPC. By going to the search bar and looking up VPC, I clicked on ‘Create VPC’ which led me to the different options. I configured the VPC as the following:
* **Resources to create**: VPC and more
* **Name tag auto-generation**: Enable Auto-generate and named the VPC "*WebServer*"
* **Number of Availability Zones (AZs)**: 2
* **Number of public subnets**: 2
* **Number of private subnets**: 2
* **NAT gateways ($)**: None
* **VPC endpoints**: S3 Gateway
* **DNS settings**: Enable DNS resolution and hostnames
* **Tags**: Key- Name | Value- WebServer

The VPC will be the foundation of the web application framework that will be holding the EC2 instances. I used two AZs for reliability and fault tolerance so that when one instance in an AZ is down, then there is another one to keep the application running. There will also be no NAT gateway since I intend to keep the web application public-facing in a public subnet. 


## Task 2 Create Security Groups for HTTP/S and SSH traffic inbound and outbound
I created a security group which will be applied to my EC2 instances. This group will allow traffic through HTTP/HTTPs and SSH ports. I clicked ‘Create security group’ and configured it as the following:
* **Security group name**: WebServerSecurityGroup
* **Description**: Allow Web Server Traffic
* **VPC**: WebServer
* **Inbound rules**: 
  - Type: HTTPS 
  - Source: 0.0.0.0/0 
  - Description: Allow HTTPS inbound from anywhere (Apply the other protocols the same fashion)
* **Outbound rules**: Copy of inbound rules

## Task 3 Launch an EC2 instances in an AZ in a public subnet
The next thing that I did was create an EC2 instance, originally I created two but then I later changed it to one EC2 instance. I went to ‘Instances’ and clicked ‘Launch instances’ to configure it as the following:
* **Name**: WebServer
* **Amazon Machine Image**: Ubuntu Server 64-bit (latest version)
* **Instance Type**: t2.micro
* **Key pair**: vockey
* **Network settings**: 
  - VPC: WebServer
  - Subnet: WebServer-subnet-public1-us-east-1a
  - Auto-assign public IP: Enable
  - Security Group: Select existing security group (WebServerSecurityGroup)
* **Advanced details**:
  - User data: The website file of the University
I launched the instance and waited for it to initialize. 

