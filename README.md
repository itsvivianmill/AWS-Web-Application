# AWS-Web-Application

## Overview and Objectives 
The hypothetical scenario is about a school called XYZ University that is preparing for the new school year and there have been complaints that their web application is slow or not available during the admissions period. XYZ University wants their framework to be able to handle peak admissions periods, support thousands of users, and be highly available, scalable, load balanced, secure, and high performing. 

To preface, my web application could be expanded and more sophisticated but my framework is very barebones with only the necessary EC2 instances, autoscaling, and load balancing services. My first vision of my framwork looked like this:
![og](https://github.com/itsvivianmill/AWS-Web-Application/assets/116047994/d7f55353-826f-4c33-8f75-04f15b48bab0)
I also had an estimated price that included all of thes services, however they were most definitely an over estimate of how it actually turned out.
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
![vpc](https://github.com/itsvivianmill/AWS-Web-Application/assets/116047994/23413e16-acde-41fd-a0bb-81e689b696e8)

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
![web app](https://github.com/itsvivianmill/AWS-Web-Application/assets/116047994/4f992af3-7924-4858-8744-36a144989555)

I launched the instance and waited for it to initialize. 

## Task 4 Create Load Balancer
After creating the EC2 instance, I went and created a load balancer by scrolling down to ‘Load Balancing’ and to ‘Load Balancers’. I clicked ‘Create load balancer’ and chose the Application Load Balancer to configure it as the following:
* **Name**: WebELB
* **Scheme**: Internet-facing
* **IP address Type**: IPv4
* **VPC**: WebServer VPC
* **Mappings**: us-east-1a & us-east-1b
* **Security groups**: WebServerSecurityGroup
* **Listeners and routing**: 
Protocol: HTTP
Port: 80

## Task 5 Create Autoscaling 
After, I clicked on the instance that was running my application to create an image for the auto scaling group. I configured it as the following:
* **Name**: WebImage

I was basically copying the configurations of the instance for duplication when the web application scales out. 

Then I created a launch template and configured it as the following:
* **Launch Template**: Create a launch template
* **Launch Template Name**: PLEASESHOWUP
* **Instance Type**: t2.micro
* **Key Pair**: vockey
* **Network Setting**:
Subnet: WebServer-subnet-public1-us-east-1a
Security Group: WebServerSecurityGroup

Now that I have a launch template and an application load balancer, it helps creating the auto scaling group easier because I will be able to insert it into the configuration. I went and scrolled down to ‘Auto Scaling Groups’ to configure it as the following:

* **Name**: pls work
* **Application and OS Images**: WebServerSecurityGroup
* **Security Group**: WebServerSecurityGroup

## Testing the Instances

Originally, the biggest struggle was to check whether or not my load balancer and scaling group worked. To test the load of my framework, I used a method that utilized EC2 Instance Connect where I would connect into the instance and use the command line. The following commands was what I used:
* sudo apt install stress -y
* sudo stress --cpu 24 --vm-bytes $(awk '/MemAvailable/{printf "%d\n", $2 * 0.9;}' < /proc/meminfo)k --vm-keep -m 1
* Enter Ctrl-C to cancel the session

However, this method did not work for me because even though it raised my CPU usage, it did not activate my auto scaling group. Since I thought the problem was my configuration, I spent more time troubleshooting and then I realized that I simply needed a new method to load test my architecture. The second method that I tried was to use AWS Cloud 9. 
![cloud9](https://github.com/itsvivianmill/AWS-Web-Application/assets/116047994/7c571364-3d8c-4d1c-8d52-8002469a5f95)
In Cloud 9, I created an environment which created a new EC2 instance so that it could be the designated loader. As the terminal loads in the service, I put in the following commands:


* npm install -g loadtest
* loadtest --rps 2000 -c 1000 -k http://<MyLoadBalancerDNS>
  * Enter Ctrl-C to cancel the session
![code](https://github.com/itsvivianmill/AWS-Web-Application/assets/116047994/9a951e2c-cb20-4f5e-bdff-c796c6851a64)
![scaled](https://github.com/itsvivianmill/AWS-Web-Application/assets/116047994/510350d2-73e9-4dd6-ab17-2ec16bb7763e)
![Screenshot 2023-06-05 141034](https://github.com/itsvivianmill/AWS-Web-Application/assets/116047994/35326f4f-bd59-4486-9be1-b10f6d9386a5)

Testing the architecture that I had built was a pain because I would have a graveyard of terminated instances. These instances existed because I had trouble getting the actual web application to show up and I would constantly terminate them and then do a load test again. So I made sure to have the most updated information such as the newest images. 
![grave](https://github.com/itsvivianmill/AWS-Web-Application/assets/116047994/59dd2b22-3c3d-4dd0-a1b7-76c83fedd96d)

In the end, it was a more difficult project than I thought that it was going to be. I plan to expand the framework later by adding AWS RDS and learn to decouple databases, but building a bare bone architecture has been a good start to my learning. As a result, this was what my framework looked like. 
![AWS Capstone Project Lucidchart](https://github.com/itsvivianmill/AWS-Web-Application/assets/116047994/cfbc708a-7db2-4a28-9eb5-5b53ef7eae5e)
