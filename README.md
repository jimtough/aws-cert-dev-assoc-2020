# aws-cert-dev-assoc-2020
Repo of misc files from my AWS Certified Developer Associate 2020 certification prep course

# IAM
* Users
* Groups
* Roles (useful for server instances and IAM users from other accounts)
* Policies (can be associated with Users, Groups or Roles)

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

## EC2 Linux server lab

* After logging on to the instance (usually as user 'ec2-user')
  * sudo su (elevate session access to act as 'root' user)
  * yum update -y (do this as 'root' to install updates from yum)

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










