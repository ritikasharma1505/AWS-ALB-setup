In this demo project - We create a VPC, with two public subnets in two different availability zones, attached with Internet gateway, an Application load balancer managing incoming traffic backed by Autoscaling group which scales as per traffic.

- Create VPC only (CIDR IP RANGE: 12.0.0.0/16)
- Create Internet Gateway and attach Internet Gateway to VPC
- Create two public subnets in different Availability-zones (for example: 1a and 1b)
  (CIDR IP RANGE for 1a : 12.0.1.0/24) & (CIDR IP RANGE for 1b : 12.0.2.0/24)
- Create route table
- Associate both public subnets with route table
- Edit route: Destination -> 0.0.0/0 and target -> Internet Gateway

- Create EC2 instances in each public subnet 

Select AMI
Select Instance type
Create keypair 
Select VPC 
Select subnet 
Select firewall(Security Group-sg) or create a new one
Enable Auto-assign pulbic IP *IMPORTANT*


## *IMPORTANT* For EC2 instances inbound rule:

port - 80, HTTP, custom from security group of alb security group( to allow http traffic via application load balancer(alb) only)
port - 22, SSH from anywhere(not suggested, recommended your IP)

## *IMPORTANT* For Application load balancer

port - 80, HTTP from anywhere
port - 443, HTTPS from anywhere

Add User data - from advanced settings:
User script for Nginx server Installation for Ubuntu 24.04

```
#!/bin/bash
apt update -y
apt install -y nginx
systemctl enable nginx
systemctl start nginx
echo "<h1>Hello from ASG behind ALB</h1>" > /var/www/html/index.nginx-debian.html
```

- Create Target group

Choose a target type: Instances
Target group name: Name
VPC
Select instances from available instances as target and include as pending below
Ports for the selected instances : 80


- Create Application Load balancer(ALB)

Load balancer name
Scheme - Internet-facing
VPC
Availability Zones and subnets
Security groups
Listeners and routing : select Target Group 
Create load balancer

- Use launch template for Autoscaling group (Automatically scale your EC2 instance as per traffic)

Launch template name - required
Template version description
Application and OS Images (Amazon Machine Image) 
Instance type 
Key pair (login) 
Firewall (security groups)
User data - optional 
Create launch template

- Create Autoscaling group

Auto Scaling group name
Launch template
VPC
Availability Zones and subnets
Load balancing : Choose 'No load balancer' (create after it) or if you have already create one before then 'Attach to an existing load balancer'
Existing load balancer target groups : if then select
Choose Desired capacity 
Min desired capacity
Max desired capacity
Create Autoscaling group


Copy the dns name and hit on browser: 
Example dns name below: 

```
http://alb-test-1415501060.us-east-2.elb.amazonaws.com/
```
You will see this output on browser if everything went well

```
Hello from ASG behind ALB
```

- After use donot forget to cleanup resources 

NOTE: Launch template and Target Groups doesnot incur charges. Only resources created by them does.


Troubleshooting common error:
 
If DNS name doesnt show desired output, this means there is something wrong with security group setting 
Autoassign Public IP is enabled (In launch template, finding this setting is tricky, its under Advanced network configuration -> Add Network Interface -> Auto-assign public IP )
The user data script is correct 
DNS name starts with http 

