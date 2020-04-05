# aws-cert-dev-assoc-2020
Repo of misc files from my AWS Certified Developer Associate 2020 certification prep course

# IAM
* Users
* Groups
* Roles (useful for server instances and IAM users from other accounts)
* Policies (can be associated with Users, Groups or Roles)

You should create at least one IAM user in the IAM lab for this section.
You'll need this later on during other labs. For example, the AWS CLI lab requires an IAM user with a programmatic access key.

# EC2

## pricing models
* On Demand: Fixed rate by the hour (or second) with no commitment
* Reserved: Requires a contract for 1 or 3 years, but gives huge discount compared to On Demand
* Spot: Bid for instance capacity for potentially huge savings, but start/end run times must be flexible
  * If a spot instance is terminated by Amazon, you will not be charged for a partial hour of use. However, if you terminate the instance yourself, then you are charged for the complete hour.
* Dedicated Hosts: Physical EC2 dedicated for your use. Allows "bring your own license".

## instance types

**FIGHTDRMCPX** "Fight Dr. McPX" - pneumonic for remembering the different types (as of 2018)

Imagine Edward Norton from Fight Club against Disney's Scrooge McDuck

* F = FPGA
* I = IOPS
* G = Graphics
* H = High Disk Throughput
* T = "tight budget" cheap general purpose
* D = Density
* R = RAM
* M = "main choice" for general purpose apps
* C = Compute
* P = pics (graphics intensive)
* X = Xtreme memory intensive

## EBS

Elastic Block Storage

Block storage devices on which you can create storage volume that can be attached to an Amazon EC2 instance. 
Once attached, you can create a file system on top of these volumes.

Each EBS volume is placed in a specific AWS Availability Zone.
* EBS is automatically replicated to protect from failure of a single physical device   

### EBS types
* GP2: General Purpose SSD
  * best balance of price and performance
* IO1: Provisioned IOPS SSD
  * Extreme I/O performance (over 10000 IOPS)
  * Good for hosting a high-perf database, for example
* ST1: Throughput optimized HDD (magnetic hard drives)
  * big data
  * data warehouse
  * log processing
  * NOTE: CANNOT be a boot volume!
* SC1: Cold HDD
  * lowest cost storage for infrequently accessed workloads
  * file server
  * NOTE: CANNOT be a boot volume!
* Standard: magnetic HDD
  * Lowest cost per gigabyte of all EBS volume types that IS BOOTABLE
  * magnetic volumes are ideal for infrequently accessed data where lowest storage cost is important

## EC2 Linux server lab (and CLI stuff)

* Create a new (cheap) EC2 AWS Linux 2 instance via the Management Console page
* After logging on to the instance (usually as user 'ec2-user')
  * `sudo su` (elevate session access to act as 'root' user)
  * `yum update -y` (do this as 'root' to install updates from yum)
  * `aws s3 ls`
    * NOTE: The CLI command above **should** fail because CLI rights are not configured yet
  * `aws configure`
    * Enter you IAM user's secret access key id and key value when prompted
    * Default region name and output format are optional (can be left blank)
  * `aws s3 ls`
  * `aws s3 mb s3://test-bucket-delete-me-amh`
  * `aws s3 ls`
  * `echo 'Hello, World!' > hello.txt`
  * `ls`
  * `aws s3 cp hello.txt s3://test-bucket-delete-me-amh`
  * `aws s3 ls s3://test-bucket-delete-me-amh`
  * NOTE: If you want to disable CLI access via secret access key, then...
    * `cd ~/.aws`
    * `rm config`
    * `rm credentials`
    * You'll likely want to do this before labs that come later where you assign an instance profile to your EC2 instance
  

### AWS CLI tips

* Least Privilege - Always give users the minimum amount of access required
* Create Groups - Assign users to Groups, and assign permissions to the Group rather than the User. Group permissions are assigned via policy documents.
* Secret Access Key - You will only see this once! You must save it somewhere, otherwise you must generate a new one.
* Do not share access keys - Assign each team member their own IAM user, and each IAM user their own access key
* You can use the CLI on your PC - Your access key works anywhere, not just on AWS servers

## Elastic Load Balancer (ELB)

There are three types:
* Application Load Balancer (OSI layer 7)
  * Best suited for load balancing of HTTP and HTTPS traffic
  * intelligent, and can create advanced routing requests (ex: send specified requests to a specific web server)
* Network Load Balancer (OSI layer 4)
  * best suited for load balancing of TCP traffic where extreme performance is required
  * highest performance, but most expensive AWS load balancer type
* Classic Load Balancer
  * deprecated type
  * Can do some of what the above types do, but not as well

REFERENCE: https://en.wikipedia.org/wiki/OSI_model

### Elastic Load Balancer tips
* If your application stops responding, the ELB (Classic Load Balancer) repsonds with a Error 504 (Gateway timeout)
  * This means the application has not responded within the idle timeout period
  * You must identify where the application is failing, and scale it up or out where possible
* The ELB will forward traffic to your application, so the application will only see the ELB's private IP address
  * If your app needs to know the original source IP address, check the 'X-Forwarded-For' header value
 
## Route53

Route53 is Amazon's DNS service.

Allows you to map your domain names to:
* EC2 instances
* load balancers
* S3 buckets

In addition to standard DNS features, you can also create a record set that is an 'alias' for an AWS resource, such as a load balancer

## LAB: EC2 with S3 Role

* Go to IAM in Management Console, then select Roles
* Click "Create Role"
* Choose "AWS service" role type, and choose "EC2" use case, then click "Next"
* We want to assign only S3 privileges to this role, so...
  * Type "s3" into the policy type filter
  * Check the box for "AmazonS3FullAccess", then click "Next"
* Skip "Tags" stage by clicking "Next"
* Name it "MyS3AdminAccess"
  * Edit the role description if you wish
* Click "Create role"
* Go to EC2 in Management Console, then select Instances
* Select my EC2 test instance
* Click "Actions" -> "Instance Settings" -> "Attach/Replace IAM Role"
* Select "MyS3AdminAccess" from the dropdown list, and click "Apply"
* Log on to your EC2 instance via SSH
* `aws s3 ls`
  * You should now be able to issue S3 commands via CLI without a secret access key
  * See EC2 and CLI lab notes above for more S3 commands
* **NOTE:** Remember to terminate your EC2 instance when you're done, if you created it just for this lab

### exam tips related to this lab

* Roles allow you to **not** use Access Key IDs and Secret Access Keys
* **Roles are preferred** over access keys from a security perspective
* Roles are controlled by policies
* You can change a policy on a Role and it will take immediate affect
* You can attach and detact roles to running EC2 instances without having to stop or terminate these instances

# RDS (Relational Database Service)

## RDS 101 lecture

AWS Database Types summary
* RDS - OLTP (Online Transaction Processing)
  * SQL Server
  * Oracle
  * MySQL
  * PostgreSQL
  * Aurora
  * MariaDB
 * NoSQL
   * DynamoDB
 * OLAP (Online Analytics Processing)
   * RedShift
 * In-memory caching
   * Elasticache
      * Memcached
      * Redis

## LAB: RDS

**Creating the RDS database instance**
* Go to RDS in Management Console, then click "Create database" button
* Choose "Standard create", and select "MySQL" as the database engine type
* Choose "Free Tier" template type
* Set a DB instance identifier value (I used "mysqltest")
* Set 'Master username' and 'Master password' to appropriate values
* Use cheapest option for DB instance size (probably "db.t2.micro")
* Leave default values for Storage type
* Availability and durability options will be unavailable on Free Tier
* Connectivity
  * Use your default VPC
  * Choose "No" for Publicly accessible (no need for public access in this lab)
  * Choose "Create new" for VPC security group
    * name it something obvious such as "RDS-LAB-SG"
  * No preference for AZ
  * Keep default port number
* Database authentication
  * Password authentication
* Additional configuration
  * Initial database name: "mysqltest"
* Finally... click "Create database" button at bottom of page

**Creating the EC2 Linux 2 instance that will interact with the RDS database**
* You want to create a new (cheap) EC2 AWS Linux 2 instance via the Management Console page as done in earlier lab, but...
  * You'll need to paste the following into the "User data" field under "Advanced Details" when creating the instance
  
```
#!/bin/bash  
yum install httpd php php-mysql -y  
yum update -y  
chkconfig httpd on  
service httpd start  
echo "<?php phpinfo();?>" > /var/www/html/index.php
cd /var/www/html  
wget https://s3.amazonaws.com/acloudguru-production/connect.php
```

* When creating your instance, create a new Security Group with inbound access to SSH (22) and HTTP (80)
  * You'll need to mess with this security group during the lab
* Create your EC2 instance, then login via SSH, and `sudo su`
* You'll need to edit the template 'connect.php' file that 'wget' pulled from the instructor's S3 bucket
  * `cd /var/www/html`
  * `nano connect.php`
  * Replace the 'username', 'password' and 'dbname' values with the values you used to create your RDS database
  * Replace the 'hostname' value with the DNS name of your RDS database
    * You can copy this value from the Management Console. Look at your instance details in RDS and find 'Endpoint'
      * example: `mysqltest.cmcvhw9a8kfe.us-east-1.rds.amazonaws.com`
* In a web browser, go here first: `http://1.2.3.4/`
  * Replace the IP address with the **public** IP address of your EC2 test instance
  * You should get a diagnostic page for the PHP web server on your EC2 test instance
* In the web browser, change the URL to this: `http://1.2.3.4/connect.php`
  * The browser will hang, and eventually display an error. This is expected because we now need to make one more Security Groups config change.
* In the Management Console, go back to RDS and get the instance details for your test database
* Find the 'Security group rules' section, and select the 'Inbound' security group for your test database
* Edit the inbound security group to allow your EC2 test instance to make database connections
  * Select 'Inbound rules', and click 'Edit inbound rules'
  * Click 'Add rule', and select "MYSQL/Aurora" (port 3306)
  * Leave 'Source' as 'Custom', click in the search box, and start typing 'sg-'
  * Auto-complete should present some options. Choose the SG used by your EC2 test instance.
  * Click 'Save rules'. Your EC2 test instance SG should now be allowed inbound access to your RDS test DB SG.
* In the web browser, try this URL again: `http://1.2.3.4/connect.php`
  * You should now get the expected response with 'Connected to MySQL' at the start


  







