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

Imagine Edward Norton from Fight Club against Disney's Scrooge McDuck who likes to share pictures from Scotland

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
* Standard: magnetic HDD (previous generation type)
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
* Create Groups - Assign users to Groups, and assign permissions to the Group rather than the User.
  * **Group permissions are assigned via policy documents**
* Secret Access Key - You will only see this once! You must save it somewhere, otherwise you must generate a new one.
* Do not share access keys - Assign each team member their own IAM user, and each IAM user their own access key
* You can use the CLI on your PC - Your access key works anywhere, not just on AWS servers

### AWS EC2 exam tips

* You can encrypt the root device volume (the volume the OS is installed on) using Operating System level encryption
* You can encrypt the root device volume by first taking a snapshot of that volume and then creating a copy of that snapshot with encryption. You then make an AMI of this snapshot and deploy the encrypted root device volume.
* You can encrypt the additional attached volumes using the console, CLI or API

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


## RDS - Backups, Multi-AZ and Read Replicas - lecture

### backup types
* There are two different types of backups for AWS: Automated Backups and Database Snapshots

#### Automated Backups
* Automated Backups allow you to recover your database to any point in time within a "retention period"
  * retention period for daily backups can be between 1 and 35 days
* Automated Backups will take a full daily snapshot and will also stored transaction logs throughout the day
* When you do a recovery AWS will choose the most recent daily backup and then apply transaction logs relevant to that day
  * This allows you to do a point in time recovery down to a second within the retention period 
* Automated Backups are enabled by default
* Automated Backup data is stored in S3 and you get free S3 storage equal to the size of your database
* Backups are taken within a defined window
  * During the backup windows, storage I/O may be suspended while your data is being backed up and you may experience elevated latency
* Automated Backups are deleted when you delete the RDS instance!

#### Database Snapshots

* Database Snapshots are done manually (user-initiated)
* These are stored even after you delete the original RDS instance, unlike Automated Backups

### Restoring from a backup

Whenever you restore either an Automated Backup or a Database Snapshot, the restored version of the database will be a new RDS instance with a new DNS endpoint name

### Encryption

* All database types support "encryption at rest" using the AWS KMS service
* An encrypted RDS instance will encrypt the underlying data storage, the automated backups, read replicas, and database snapshots
* (At present time) you cannot encrypt a non-encrypted RDS instance - you must first create a snapshot, make a copy of the snapshot and then encrypt the copy

### Multi-AZ enabled databases

Multi-AZ enabled databases will apply all writes to the database to a second instance that resides in a different AZ in that region

**Used for DR, NOT for scaling!**

* This is done for disaster recovery reasons, NOT to improve performance!
* if AZ 1 is lost, then AZ 2 will have an up-to-date copy of the instance and will automatically fail over to AZ 2
* AWS will automatically update the RDS DNS endpoint name with the failover database instance if primary AZ instance goes down
* AWS handles the replication for you - writes are automatically synchronized to the standby database
* In the event of planned database maintenance, instance failure, or AZ failure, RDS will automatically failover to the standby database without manual intervention

### Read Replicas

This is a performance enhancement for databases that are hit with frequent read operations

**Used for scaling, NOT for DR!**

* Read replicas allow you to have a read-only copy of your production database
* Achieved using asynchronous replication from the primary RDS instance to (one or more) read replica database
* Used primarily for very read-heavy database workloads
* Currently available for these database types:
  * MySQL Server
  * PostgreSQL
  * MariaDB
  * Aurora
* **You must have Automatic Backups enabled in order to deploy a read replica!**
* You can have up to 5 read replica copies of any database
* You can have read replicas or read replicas (but latency can be an issue if you do this)
* Each read replica will have its own DNS endpoint
* You can have read replicas that have Multi-AZ
* You can create replicas of Multi-AZ source databases
* Read replicas can be promoted to be their own databases, but this breaks the replication
* You can have a read replica **in a different region** (currently for MySQL and MariaDB)

## Elasticache

Elasticache is a web service that makes it easy to deploy, operate and scale an in-memory cache in the cloud.
The service improves the performance of web applications by allowing you to retrieve info from fast, managed in-memory caches instead of relying entirely on slower disk-based databases.

Amazon ElastiCache can be used to significantly improve latency/throughput for read-heavy application workloads or compute-intensive workloads (such as a recommendation engine).

Caching improves performance by storing critical pieces of data in memory for low-latency access.
Cached information may include the results of I/O-intensive database queries or the results of computationally-intensive calculations.

### Elasticache types

#### Memcached

A widely adopted memory object caching system. Elasticache is protocol compliant with Memcached, so existing tools will work seamlessly with the service.

Memcached is designed as a pure caching solution with no persistence.
Elasticache manages Memcached nodes as a pool that can grow or shrink, similar to an AWS EC2 auto-scaling group.
Individual nodes are expendable, and Elasticache provides additional capabilities here such as automatic node replacement and Auto Discovery.

**Use Cases for Memcached**
* Object caching to reduce load on your database
* Want to use as simple a caching model as possible
* Want to run large cache nodes and require multithreaded performance using multiple cores
* Want ability to scale the cache horizontally as you grow

#### Redis

A popular open-source in-memory key-value store that supports data structures such as sorted sets and lists.

Elasticache supports Master/Slave replication and Multi-AZ which can be used to achieve cross-AZ redundancy (Memcached cannot do this).

AWS handles Redis more like a relational database, since it supports replication and persistence.
Redis Elasticache clusters are managed as stateful entities that include failover, similar to how RDS manages database failover.

**Use Cases for Redis**
* Want to use more advanced data types such as lists, hashes and sets
* **If sorting and ranking datasets in memory helps you, such as with leaderboards**
* If persistence of your key store is important
* If you want to run in multiple AZs with failover 

#### Elasticache exam tips

Typically you will be given a scenario where a particular database is under a lot of stress/load.
You may be asked which service you should use to alleviate this.

* Elasticache is a good choice if your database is particularly read-heavy and not prone to frequent changing
* Redshift is a good answer if the reason your database is under stress is bacause management keeps running OLAP transactions on it
* Use **Memcached** if:
  * Object caching is your primary goal
  * You want to keep things as simple as possible
  * You want to scale your cache horizontally (scale out)
* Use **Redis** if: 
  * You have advanced data types, such as lists, hashes and sets
  * You are doing data sorting and ranking (such as leaderboards)
  * Data persistence
  * Multi-AZ
  * Pub/Sub capabilities are needed

# S3 (Simple Storage Service)

S3 provides secure, durable, highly-scalable object storage.

* A safe place to storage your files
* Object-based storage (files, etc) - NOT BLOCK STORAGE!
* The data is spread across multiple devices and facilities

## S3 basics

* S3 is Object based (allows you to upload files)
* Files can be from 0 bytes to 5 Tb
* There is unlimited storage
* Files are stored in Buckets
* S3 is a universal namespace (each Bucket name must be unique globally across all of Amazon's customers)
  * The Bucket name is used in URLs, so that is one reason for the uniqueness requirement
* When you upload a file to S3 you will receive a HTTP 200 code if the upload was successful
* Built for 99.99% availability for the S3 platform
* Amazon **guarantee** 99.9% availability
* Amazon guarantees 99.99999999999% (11 9's) **durability** for S3 information
* Tiered storage available
* Lifecycle management
* Versioning
* Encryption
* Secure your data - Access Control Lists and Bucket Policies


## Data Consistency model for S3

* "Read after Write Consistency" for PUTs of **new** Objects
* "Eventual Consistency" for **overwrite** PUTs and DELETEs (can take some time to propagate)

## S3 is a simple key/value store

* S3 is Object-based. Objects consist of the following:
  * Key (simply the name of the object)
  * Value (simply the data, which is made up of a sequence of bytes)
  * Version ID (important for versioning)
    * S3 supports version control
  * Metadata (data about the data you are storing)
  * Subresources (Bucket-specific configuration)
    * Bucket policies, access control lists
    * Cross-origin resource sharing (CORS)
    * Transfer Acceleration

## S3 Storage Tiers and Classes

* S3: 99.9% **availability** and 99.99999999999% (11 9's) **durability**, storage redundantly across multiple devices in multiple facilities
  * Designed to sustain the loss of 2 facilities concurrently
* S3 IA (Infrequently Accessed): For data that is accessed less frequently but requires rapid access when needed
  * Lower fee for storage than S3, but you are charged a fee to retrieve files
* S3 - One Zone IA: Same as IA however data is stored in a single AZ only
  * Still 9 11's durability, but only 99.5% availability
  * Cost is 20% less than regular S3 IA
* Reduced Redundancy Storage: Designed to provide 99.99% durability and 99.99% availability of objects over a given year
  * Used for data that can be recreated is lost (example: thumbnails)
  * NOTE: This option is starting to disappear from AWS documentation by may still appear on the exam
* Glacier: Very cheap, but used for archival only
  * Optimized for data that is infrequently accessed
  * **No real-time access** - takes 3 to 5 hours to restore from Glacier
  * There is a fee to restore files from Glacier

### S3 Intelligent Tiering (new service as of Nov 2018)

* Unknown or unpredictable access patterns
* 2 tiers - frequent and infrequent access
* Automatically moves your data to most cost-effective tier based on how frequently you access each object
* 11 9's durability
* 99.9% availability over a given year
* Optimizes your cost
* No fees for accessing your data, but a small monthly fee for monitoring/automation of $0.0025 per 1000 objects

## S3 Charges

You will be charged for the following
* Storage per Gb of data
* Number of requests (GET, PUT, copy, etc.)
* Storage Management pricing
  * inventory, analytics, object tags
* Data Management pricing
  * Data transferred out of S3 (downloads, etc)
* Transfer Acceleration
  * Use of CloudFront to optimize transfers

## S3 Exam tips

* **NOT suitable to install an operating system or run a database on**
* Files can be from 0 bytes to 5 Tb
* There is unlimited storage
* Files are stored in Buckets
* S3 is a universal/global namespace
* "Read after Write Consistency" for PUTs of **new** Objects
* "Eventual Consistency" for **overwrite** PUTs and DELETEs (can take some time to propagate)
* When you upload a file to S3 you will receive a HTTP 200 code if the upload was successful (when using CLI or API)

Official S3 docs: https://aws.amazon.com/s3/faqs/
* Good to review this just before taking the exam

## S3 Security

* By default, newly created S3 Buckets are PRIVATE
* You can set up access control to your buckets using:
  * Bucket Policies - applied at Bucket level
  * Access Control Lists - applied at Object level
* S3 Buckets can be configured to create access logs which log all requests make to the S3 Bucket
  * The logs can be written to another S3 Bucket

## S3 Encryption

### Encryption types

* In Transit
  * SSL/TLS
* At Rest
  * Server Side Encryption
    * S3 managed keys (SSE-S3)
      * Each object is encrypted with its own unique key
      * Each encryption key is ALSO encrypted with a master key that is regularly rotated
      * Amazon manages all this for you
    * AWS KMS managed keys (SSE-KMS)
      * ??? (what is an 'envelope key'?)
      * Provides an audit trail of who used the key, when it was used, and for what purpose
    * server side encryption with customer provided keys (SSE-C)
      * Amazon managed the encryption and decryption operations, but you manage the keys
  * Client Side Encryption
    * Customer encrypts their own files before they are loaded into S3

### How to enforce encryption on S3 Buckets

* Every time a file is uploaded to S3, a PUT request is initiated
  * The HTTP request header contains this: `Expect: 100-continue`
    * Tells S3 not to send the body of the request until it receives an acknowledgement
    * Allows S3 to reject your PUT request based on the contents of the header
  * If the file is to be encrypted at uploaded time, the `x-amz-server-side-encryption` parameter will be included in the request header
    * Two options are currently available:
      * `x-amz-server-side-encryption: AES256` (SSE-S3 - S3 managed keys)
      * `x-amz-server-side-encryption: ams:kms` (SSE-KMS - KMS managed keys)
    * When this parameter is included in the header of the PUT request, it tells S3 to encrypt the object at upload time using the specified method
    * You can enforce the use of Server Side Encryption by using a Bucket Policy which denies any S3 PUT request that is missing the `x-amz-server-side-encryption` parameter in the request header

### S3 Encryption using a Bucket Policy lab

In the lab, we do this by creating a Bucket Policy, and NOT by changing the 'Default encryption' settings in the management console.

* Create a bucket, or use an existing one
* Go to Permissions->Bucket Policy
* Use the Policy Generator tool
  * S3 Bucket Policy type
  * Effect: Deny
  * Principal: *
  * AWS Service: Amazon S3
  * Actions: PutObject
  * Amazon Resource Name: (ARN of your S3 bucket - shown on Bucket Policy page)
  * Click "Add Conditions"
    * Condition: `StringNotEquals`
    * Key: `s3:x-amz-server-side-encryption`
    * Value: `aws:kms` (or whichever server-side encryption type you want to enforce)
  * Click "Add Statement"
  * Click "Generate Policy"
  * Copy policy JSON, paste into Bucket Policy page, and click Save
  * At this point, I get an Error: "Action does not apply to any resource(s) in statement"
    * Fix this by added a wildcard to the "Resource" in the policy:
      * `"Resource": "arn:aws:s3:::adultmalehuman-encrypted-bucket",`
      * `"Resource": "arn:aws:s3:::adultmalehuman-encrypted-bucket/*",`
      * Click "Save" again and it should work
* Now try to upload a test file to the S3 bucket without encryption enabled, and it should fail with "Forbidden" error
* Now try to upload a test file with S3 KMS encryption enabled, and it should succeed












