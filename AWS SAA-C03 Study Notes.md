# My study notes for the AWS SAA-C03 exam


Resources: 
- [Udemy: Ultimate AWS Certified Solutions Architect Associate SAA-C03 by Stephane Maarek](https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03/)
- [Youtube Channel: Be A Better Dev](https://www.youtube.com/@BeABetterDev)

# Virtual Private Cloud (VPC)

It’s like a data center. A secure, isolated private cloud hosted within a public cloud.

You can have multiple VPCs in an AWS region (max. 5 per region – soft limit)

Max. CIDR per VPC is 5, for each CIDR:



* Min. size is /28 (16 IP addresses)
* Max. size is /16 (65536 IP addresses)


## Subnet (IPv4)

Tied to AZ

AWS reserves 5 IP addresses (first 4 & last 1) in each subnet

These 5 IP addresses are not available for use and can’t be assigned to an EC2 instance

Exam Tip, if you need 29 IP addresses for EC2 instances:



* You can’t choose a subnet of size /27 (32 IP addresses, 32 – 5 = 27 &lt; 29)
* You need to choose a subnet of size /26 (64 IP addresses, 64 – 5 = 59 > 29)


## Internet Gateway (IGW)

Allows resources (e.g., EC2 instances) in a VPC connect to the Internet

Must be created separately from a VPC

One VPC can only be attached to one IGW and vice versa

Internet Gateways on their own do not allow Internet access, route tables must also be edited.


## Bastion Hosts

We can use a Bastion Host to **SSH** into our private EC2 instances

The bastion is in the public subnet which is then connected to all other private subnets

Bastion Host security group must allow inbound from the internet on port 22 from restricted CIDR, for example the public CIDR of your corporation.


## NAT Gateway (Network Address Translation)

**Gives Internet access to EC2 instances in private subnets. **

Requires an IGW (Private Subnet => NATGW => IGW) 


## Network Access Control List (NACL)

NACL are like a firewall which control traffic from and to subnets

One NACL per subnet, new subnets are assigned the Default NACL

Default NACL accepts everything inbound/outbound with the subnets it’s associated with

Do NOT modify the Default NACL, instead create custom NACLs

Supports allow rules and deny rules

**Stateless**: return traffic must be explicitly allowed by rules (think of ephemeral ports)


## Security Group

Operates at the instance level

Supports allow rules only

**Stateful**: return traffic is automatically allowed, regardless of any rules


## VPC Peering

You must **update route tables in each VPC’s subnets** to ensure EC2 instances can communicate with each other.

Connect two VPCs with non overlapping CIDR, non-transitive


## VPC Endpoints

VPC Endpoints (powered by AWS PrivateLink) **allows you to connect to AWS services using a private network instead of using the public Internet**

Types:



* Interface Endpoints (powered by PrivateLink)
    * Provisions an ENI (private IP address) as an entry point (must attach a Security Group)
    * Supports most AWS services
* **Gateway Endpoints**
    * Provisions a gateway and must be used as a target in a route table (does not use security groups)
    * Supports both **S3 and DynamoDB**
    * Free


## Flow Logs

Capture information about IP traffic going into your interfaces



* VPC Flow Logs
    * A VPC feature that enables you to capture information about the IP traffic going to and from network interfaces in your VPC.
* Subnet Flow Logs
* Elastic Network Interface (ENI) Flow Logs


## Site-to-Site VPN



* Virtual Private Gateway (VGW)
    * VPN concentrator on the **AWS side** of the VPN connection
    * VGW is created and attached to the VPC from which you want to create the Site-to-Site VPN connection
* Customer Gateway (CGW)
* Software application or physical device on **customer side** of the VPN connection

Customer Gateway Device (On-premises)



* What IP address to use?
    * Public Internet-routable IP address for your Customer Gateway device
    * If it’s behind a NAT device that’s enabled for NAT traversal (NAT-T), use the public IP address of the NAT device

Important step: enable Route Propagation for theVirtual Private Gateway in the route table that is associated with your subnets

If you need to ping your EC2 instances from on-premises, make sure you add the ICMP protocol on the inbound of your security groups


## AWS VPN CloudHub

AWS VPN CloudHub allows you to securely communicate with multiple sites using AWS VPN. It operates on a simple hub-and-spoke model that you can use with or without a VPC.


## Direct Connect (DX)

Provides a **dedicated private connection** from a **remote network** to your **VPC**

Using a Direct Connect connection, you can access both public and private AWS resources.

Dedicated connection must be setup between your DC and AWS Direct Connect locations

You need to setup aVirtual Private Gateway on yourVPC

Access public resources (S3) and private (EC2) on same connection



* Dedicated Connections: 1Gbps,10 Gbps and 100 Gbps capacity
* Hosted Connections: 50Mbps, 500 Mbps, to 10 Gbps


### Direct Connect Gateway

To **set up a Direct Connect to one or more VPC in many different regions** (same account), 


## Transit Gateway

For having **transitive peering between thousands of VPC and on-premises**, hub-and-spoke (star) connection

ECMP = Equal-cost multi-path routing: Routing strategy to allow to forward a packet over multiple best path


## VPC – Traffic Mirroring

Allows you to capture and inspect network traffic in your VPC, copy network traffic from ENIs for further analysis.


## Egress-only Internet Gateway

Used for IPv6 only: similar to a NAT Gateway but for IPv6

Allows instances in your VPC outbound connections over IPv6 while preventing the internet to initiate an IPv6 connection to your instances

You must update the Route Tables


## Network Firewall

Protect your entire Amazon VPC, from Layer 3 to Layer 7 protection


# Elastic Beanstalk

A developer centric view of **deploying** an application.

Single instance mode is used in the development stage and can reduce costs. High availability mode is used in the production stage.


# CloudFormation

A declarative way of **outlining** your AWS Infrastructure, for any resources


# Route 53

It is a domain registrar, also a DNS service provider.

Route 53 health checkers are outside the VPC. Can’t access private endpoints. If we want to check private resources we need to use it with a CloudWatch Alarm.

Weighted Routing Policy allows you to redirect part of the traffic based on weight (e.g., percentage). It's a common use case to send part of traffic to a new version of your application.

Each DNS record has a TTL (Time To Live) which orders clients for how long to cache these values and not overload the DNS Resolver with DNS requests. The TTL value should be set to strike a balance between how long the value should be cached vs. how many requests should go to the DNS Resolver.

Latency Routing Policy will evaluate the latency between your users and AWS Regions, and help them get a DNS response that will minimize their latency (e.g. response time)

Public Hosted Zones are meant to be used for people requesting your website through the Internet. Finally, NS records must be updated on the 3rd party Registrar.


# Identity & Access Management (IAM)

Region



*  Cluster of data centers. Has a few AZs.

Availability Zones



* Isolated from disasters.

Define permissions. Least privilege principle.

Best Practices:



* Don’t use the root account except for AWS account setup
* One physical user = One AWS user
* Assign users to groups and assign permissions to groups
* Create a strong password policy
* Use and enforce the use of Multi Factor Authentication (MFA)
* Create and use Roles for giving permissions to AWS services
* Use Access Keys for Programmatic Access (CLI / SDK)
* Audit permissions of your account using IAM Credentials Report & IAM Access Advisor
* Never share IAM users & Access Keys

IAM Roles vs Resource-Based Policies



* When you assume a role (user, application or service), you give up your original permissions and take the permissions assigned to the role
    * Kinesis stream, Systems Manager Run Command, ECS task
* When using a resource-based policy, the principal doesn’t have to give up his permissions
    * SNS, SQS, CloudWatch Logs, API Gateway, EventBridge

 IAM Permission Boundaries are supported for users and roles (not groups)


## Organizations

Security: Service Control Policies (SCP)

Blocklist and Allowlist strategies


## Control Tower

Easy way to set up and govern a secure and compliant multi-account AWS environment based on best practices

AWS Control Tower uses **AWS Organizations** to create accounts


### Guardrails



* Provides ongoing governance for your Control Tower environment (AWS Accounts) Preventive Guardrail – using SCPs (e.g., Restrict Regions across all your accounts)
* Detective Guardrail – using AWS Config (e.g., identify untagged resources)


## IAM Conditions

aws:SourceIp

aws:RequestedRegion

ec2:ResourceTag

aws:MultiFactorAuthPresent


## IAM Identity Center

One login (single sign-on) for all your AWS accounts in AWS Organizations, Business cloud applications (e.g., Salesforce, Box, Microsoft 365, ...), SAML2.0-enabled applications, EC2 Windows Instances


## Directory Services

3 Way to connect



* AWS Managed Microsoft AD
* AD Connector
* Simple AD


# Security

In-flight Encryption = HTTPS, and HTTPS can not be enabled without an SSL certificate.

Server-Side Encryption means the server will encrypt the data for us. Both Encryption and Decryption happen on the server. We can't do encryption/decryption ourselves as we don't have access to the corresponding encryption key.

With Client-Side Encryption, the server doesn't need to know any information about the encryption scheme being used, as the server will not perform any encryption or decryption operations.


## Key Management Service (KMS)

Able to audit KMS Key usage using CloudTrail.

You can use the AWS Managed Service keys in KMS, therefore we don't need to create our own KMS keys.

Backing key for Automatic Rotation is rotated every year.

Use KMS Key Policies to control access to KMS CMKs.

Types:



* KMS keys can be symmetric or asymmetric. A symmetric KMS key represents a 256-bit key used for encryption and decryption. An asymmetric KMS key represents an RSA key pair used for encryption and decryption or signing and verification, but not both. Or it represents an elliptic curve (ECC) key pair used for signing and verification.
* Symmetric (AES-256 keys)
    * Single encryption key that is used to Encrypt and Decrypt
    * AWS services that are integrated with KMS use Symmetric CMKs
    * You never get access to the KMS Key unencrypted (must call KMS API to use)
* Asymmetric (RSA & ECC key pairs)
    * Public (Encrypt) and Private Key (Decrypt) pair
    * Used for Encrypt/Decrypt, or Sign/Verify operations
    * The public key is downloadable, but you can’t access the Private Key unencrypted
    * Use case: encryption outside of AWS by users who can’t call the KMS API

KMS Multi-Region Keys



* Use cases: global client-side encryption, encryption on Global DynamoDB, Global Aurora


## SSM (Systems Manager) Parameter Store

Secure storage for configuration and secrets

SSM Parameters Store can be used to store secrets and has built-in version tracking capability. Each time you edit the value of a parameter, SSM Parameter Store creates a new version of the parameter and retains the previous versions. You can view the details, including the values, of all versions in a parameter's history.


## Secrets Manager

Mostly meant for **RDS** integration. 

It is a newer service than the parameter store.


## Cost Explorer

Visualize, understand, and manage your AWS costs and usage over time


## SSM Session Manager

Allows you to start a secure shell on your EC2 and on-premises servers

No SSH access, bastion hosts, or SSH keys needed

No port 22 needed (better security)


## Certificate Manager (ACM)



* Easily provision, manage, and deploy TLS Certificates
* Provide in-flight encryption for websites (HTTPS)
* Clients can use Server Name Indication (SNI) to specify the hostname they reach.
* SNI solves the problem of loading multiple SSL certificates on 1 web server (to serve multiple websites).


## Web Application Firewall (WAF)

Protects your web applications from common web exploits (Layer 7)

Layer 7 is HTTP (vs Layer 4 is TCP/UDP)fs


## Shield

DDoS: Distributed Denial of Service – many requests at the same time


## Firewall Manager

Manage rules in all accounts of an AWS Organization.

AWS Firewall Manager is a security management service that allows you to centrally configure and manage firewall rules across your accounts and applications in AWS Organizations. It is integrated with AWS Organizations so you can enable AWS WAF rules, AWS Shield Advanced protection, security groups, AWS Network Firewall rules, and Amazon Route 53 Resolver DNS Firewall rules.


## GuardDuty

Intelligent Threat discovery to protect your AWS Account

Can protect against CryptoCurrency attacks (has a dedicated “finding” for it)

Input data includes:



* CloudTrail Events Logs
* VPC Flow Logs
* DNS Logs


## Inspector

Automated Security Assessments



* For EC2 instances
* For Container Images push to Amazon ECR
* For Lambda Functions

Reporting & integration with AWS Security Hub

Send findings to Amazon Event Bridge


## Macie

Amazon Macie is a fully managed data security and data privacy service that uses machine learning and pattern matching to discover and protect your sensitive data in AWS. Macie helps identify and alert you to sensitive data, such as personally identifiable information (PII)

Amazon Macie discovers and protects your sensitive data stored in S3 buckets. It automatically provides an inventory of S3 buckets including a list of unencrypted buckets, publicly accessible buckets, and buckets shared with other AWS accounts.


# Elastic Compute Cloud (EC2)



* Have lots of control. Hard to maintain.
* Security Group: Firewall.
* Good to maintain one separate security group for SSH (Port 22) access
* EC2 User Data is used to bootstrap your EC2 instances using a bash script.
* Elastic IP is a public IPv4 that you own as long as you want and you can attach it to one EC2 instance at a time.
* Spot Fleet is a set of Spot Instances and optionally On-demand Instances.
* Placement Groups
    * Cluster, Spread, Partition
* Elastic Network Interfaces (ENI)
    * Represent a Virtual Network Card.
    * Are bounded to a specific AZ.
    * Used in a VPC
* To enable EC2 Hibernate, the EC2 Instance Root Volume type must be an EBS volume and must be encrypted to ensure the protection of sensitive content.


# Elastic Load Balancer (ELB)

3 Types:



* Application (L7: HTTP), great fit for micro services & container-based application (eg. Docker & Amazon ECS).
    * Cross-Zone LB is enabled by default.
    * Application Load Balancers support HTTP, HTTPS and WebSocket.
    * ALBs can route traffic to different Target Groups based on URL Path, Hostname, HTTP Headers, and Query Strings.
* Network (L4: TCP/UDP). Less latency, used for extreme performance.
    * Network Load Balancer has one static IP address per AZ and you can attach an Elastic IP address to it. Application Load Balancers and Classic Load Balancers have a static DNS name.
    * The NLB supports HTTP health checks as well as TCP and HTTPS.
* Gateway (L3: IP Packets). For managing a fleet of **3rd party network virtual appliances in AWS, like a firewall**, etc. For Transparent Network Gateway.

Only Network Load Balancer provides both static DNS name and static IP. Application Load Balancer provides a static DNS name but it does NOT provide a static IP. The reason being that AWS wants your Elastic Load Balancer to be accessible using a static endpoint, even if the underlying infrastructure that AWS manages changes.

ELB Sticky Session feature ensures traffic for the same client is always redirected to the same target (e.g., EC2 instance). This helps that the client does not lose his session data.

When using an Application Load Balancer to distribute traffic to your EC2 instances, the IP address you'll receive requests from will be the ALB's private IP addresses. To get the client's IP address, ALB adds an additional header called "X-Forwarded-For" containing the client's IP address.


## Auto Scaling Group (ASG)

Can scale based on CloudWatch alarms.

You can configure the Auto Scaling Group to determine the EC2 instances' health based on Application Load Balancer Health Checks instead of EC2 Status Checks (default). When an EC2 instance fails the ALB Health Checks, it is marked unhealthy and will be terminated while the ASG launches a new EC2 instance.


# Storage


## S3

Raw Data Store / Object Storage.

Store objects in buckets.

Great for bigger objects, not so great for many small objects.

Versioning is enabled at the bucket level. Original files are version “null”.

Versioning is required for MFA Delete.

Multi-Part Upload is recommended for files > 100 MB. Must be used for files > 5GB.

Explicit DENY in an IAM Policy will take precedence over an S3 bucket policy.

S3 Replication allows you to replicate data from an S3 bucket to another in the same/different AWS Region.

For archive objects that don’t need fast access to, move them to **Glacier** or Glacier Deep Archive. Use **Lifecycle** Rules for this. Lifecycle is also used to transfer from Snowball into S3.

**Transfer Acceleration**: using AWS edge location.

Object Encryption (4 Types)



* Server-Side
    * Amazon S3-Managed Keys (SSE-S3)
        * Enabled by default
        * With SSE-S3, the encryption happens in AWS and you the encryption keys managed by AWS. Encryption keys stored in AWS.
    * KMS Keys (SSE-KMS)
        * With SSE-KMS, the encryption happens in AWS, and the encryption keys are managed by AWS but you have **full control over the rotation policy** of the encryption key. Encryption keys stored in AWS.
    * Customer-Provided Keys (SSE-C): 
        * HTTPS is mandatory.
        * With SSE-C, the encryption happens in AWS and you have full control over the encryption keys.
* Client-Side
    * With Client-Side Encryption, you have to do the encryption yourself and you have full control over the encryption keys. You perform the encryption yourself and send the encrypted data to AWS. AWS does not know your encryption keys and cannot decrypt your data.
* Bucket Policies are evaluated before “Default Encryption”.

Access Logs



* Do not set the logging bucket to be the monitored bucket.
* The target logging bucket must be in the same AWS region. 
* S3 Access Logs log all the requests made to S3 buckets and Amazon Athena can then be used to run serverless analytics on top of the log files.

S3 Pre-Signed URLs are temporary URLs that you generate to grant time-limited access to some actions in your S3 bucket.

S3 Glacier Vault Lock



* Adopt a WORM (Write Once Read Many) model
* Lock the policy for future edits
* Helpful for compliance and data retention

S3 Object Lock



* Versioning enabled is required
* Adopt a WORM (Write Once Read Many) model
* Applied for **a single object**, and a certain version.
* Retention mode - Compliance:
    * Object versions can't be overwritten or deleted by any user, including the root user
    * Objects retention modes can't be changed, and retention periods can't be shortened
* Retention mode - Governance:
    * Some users have special permissions to change the retention or delete the object


## Elastic Block Store (EBS)

A network drive can be attached to the EC2 instances while they run.

“Network USB Stick”

Locked to an AZ. 

It is possible to migrate them between different AZs using EBS Snapshots.

Snapshots: A backup of EBS volume at a point in time.

One instance, except **multi-attach in the same AZ**. Each EC2 instance has full read/write permissions.

Copying an unencrypted snapshot allows encryption

Snapshots of encrypted volumes are encrypted

When creating EC2 instances, you can only use the following EBS volume types as boot volumes: gp2, gp3, io1, io2, and Magnetic (Standard).


## Elastic File System (EFS)

Only compatible with **Linux** based AMI not Windows

Mounting 100s of instances across AZs.

Managed NFS (network file system) that can be mounted on many EC2.

No capacity planning

EFS is a network file system (NFS) that allows you to mount the same file system to many EC2 instances. Storing software updates on an EFS allows each EC2 instance to access them.


## FSx: Launch 3rd party high-performance file systems on AWS

FSx for Windows

FSx for Lustre:** High Performance Computing (HPC)** Linux file system

FSx for NetApp ONTAP: High OS Compatibility, not compatible with FTP

FSx for OpenZFS: Managed ZFS file system

Scratch File System: Provides temporary storage where data is not replicated.

Persistent File System: Provides long-term storage where data is replicated within the same AZ. Failed files were replaced within minutes.


## EC2 Instance Store



* High-performance hardware disk.
* Provides the best disk I/O performance.
* Physically attached to the EC2 instance.
* Ephemeral storage.


## Amazon Machine Image (AMI)

A customization of an EC2 instance

AMIs are built for a specific AWS Region, they're unique for each AWS Region. You can't launch an EC2 instance using an AMI in another AWS Region, but you can copy the AMI to the target AWS Region and then use it to create your EC2 instances.

**Golden AMI** is an image that contains all your software installed and configured so that future EC2 instances can boot up quickly from that AMI.


## Storage Gateway

**Bridge** between on-premises data and cloud data

Need on-premises virtualization

S3 & FSx File Gateway



* Use NFS / SMB

Volume Gateway (cache & stored)



* Use iSCSI protocol

Tape Gateway


## Data Migration/Edge Computing

Move large amount of data to the cloud, physically

Snowball: 80 TB



* Snowball Edge is the right answer as it comes with computing capabilities and allows you to pre-process the data while it's being moved into Snowball.

Snowcone: 14 TB, comes with data sync agent bundled in it already

Snowmobile: 100 PB capacity

Edge Computing: Snowball / Snowcone, can run EC2 or Lambda functions.


## DataSync

Schedule data sync from on-premises to AWS, or AWS to AWS

File permissions and metadata are preserved when moved. \
Support: S3, EFS, FSx for Windows File Server.


## Transfer Family

FTP, FTPS, SFTP **interface** on top of S3 or EFS


# Database


## RDS

RDS supports MySQL, PostgreSQL, MariaDB, Oracle, MS SQL Server, and Amazon Aurora.

**Multi-AZ** keeps the same connection string regardless of which database is up.



* Multi-AZ helps when you plan a disaster recovery for an entire AZ going down. If you plan against an entire AWS Region going down, you should use backups and replication across AWS Regions.

**Read Replica** uses Asynchronous Replication and Multi-AZ uses Synchronous Replication.



* RDS can have up to 15 Read Replicas.
* Read Replicas: for SELECT kind of statements
* You can not create encrypted Read Replicas from an unencrypted RDS DB instance.

Storing Session Data in ElastiCache is a common pattern to ensure different EC2 instances can retrieve your user's state if needed.

**Aurora**: In-house



* Aurora Global Databases allows you to have an Aurora Replica in another AWS Region, with up to 5 secondary regions.
* Aurora Global: up to 16 DB Read Instances in each region, &lt; 1 second storage replication

Storage auto scaling

Cross-region replication takes less than 1s.

**RDS Proxy** reduces the failover time by up to 66% and keeps connection active for your applications.


## DynamoDB

NoSQL

Made of Tables, each table has a Primary Key (Decided at creation time), each table can have an infinite number of items (=rows), each item has attributes (=columns) (can be added over time - can be null)

**In DynamoDB you can rapidly evolve schemas.**

On-Demand Capacity Mode is much more expensive than Provisioned Capacity Mode (has optional auto-scaling).

DynamoDB Accelerator (DAX): in-memory cache



* Help solve read congestion by caching
* **Microseconds** latency for cached data
* Don’t require application logic modification, compatible with existing DynamoDB APIs.
* DynamoDB Accelerator (DAX) is a fully managed, highly available, in-memory cache for DynamoDB that delivers up to 10x performance improvement. It caches the most frequently used data, thus offloading the heavy reads on hot keys off your DynamoDB table.

DynamoDB Streams



* DynamoDB Streams allows you to capture a time-ordered sequence of item-level modifications in a DynamoDB table. It's integrated with AWS Lambda so that you create triggers that automatically respond to events in real-time.
* DynamoDB Streams enable DynamoDB to get a changelog and use that changelog to replicate data across replica tables in other AWS Regions.

Can replace ElastiCache as a key/value store (storing session data for example, using TTL feature)

TTL use case: web session handling

DynamoDB is serverless with no servers to provision, patch, or manage and no software to install, maintain or operate. It automatically scales tables up and down to adjust for capacity and maintain performance. It provides both provisioned (specify RCU & WCU) and on-demand (pay for what you use) capacity modes.

Read Capacity Units (RCU)/Write Capacity Units (WCU): RCU and WCU are decoupled, so you can increase/decrease each value separately.

Export to S3 without using RCU within the PITR window, import from S3 without using WCU

Maximum size of an item in a DynamoDB table is 400kb.


## ElastiCache

Managed Redis / Memcached (similar offering as RDS, but for caches)

In-memory data store, sub-millisecond latency

Support for Clustering (Redis) and Multi AZ, Read Replicas (sharding)

Security through IAM, Security Groups, KMS, Redis Auth

Backup / Snapshot / Point in time restore feature

**Requires some application code changes to be leveraged**

Use Case: Key/Value store, Frequent reads, less writes, cache results for DB queries, store session data for websites, cannot use SQL.


## DocumentDB

DocumentDB is an “AWS-implementation” for **MongoDB** (which is a NoSQL database)


## Neptune

Fully managed graph database


## Amazon Keyspaces

Apache Cassandra is an open-source NoSQL distributed database

A managed Apache Cassandra-compatible database service


## Amazon QLDB

QLDB stands for ”Quantum Ledger Database”

A ledger is a book recording financial transactions

Difference with Amazon Managed Blockchain: no decentralization component, in accordance with financial regulation rules


## Amazon Timestream

Fully managed, fast, scalable, serverless time series database


# Message Orchestration


## SNS

Push data to many subscribers

pub/sub model

Ex. Kafka.

Topic - subscribers.

Fan Out: Push once in SNS, receive in all SQS queues that are subscribers. A common pattern where only one message is sent to the SNS topic and then "fan-out" to multiple SQS queues. This approach has the following features: it's fully decoupled, no data loss, and you have the ability to add more SQS queues (more applications) over time.


## SQS

FIFO/Non-FIFO queue

Consumer “pull data”

Can be integrated with Lambda.

Long Polling: “wait” if there are no messages in the queue

SQS Visibility Timeout is a period of time during which Amazon SQS prevents other consumers from receiving and processing the message again. In Visibility Timeout, a message is hidden only after it is consumed from the queue. Increasing the Visibility Timeout gives more time to the consumer to process the message and prevent duplicate reading of the message. (default: 30 sec., min.: 0 sec., max.: 12 hours)

SQS FIFO (First-In-First-Out) Queues have all the capabilities of the SQS Standard Queue, plus the following two features. First, The order in which messages are sent and received are strictly preserved and a message is delivered once and remains available until a consumer processes and deletes it. Second, duplicated messages are not introduced into the queue.


## MQ

Managed message broker service for RabbitMQ and ActiveMQ

Amazon MQ supports industry-standard APIs such as JMS and NMS, and protocols for messaging, including AMQP, STOMP, MQTT, and WebSocket.


## Kinesis: Real-time streaming

Standard: pull data

Enhanced fan out: push data

Data Streams (KDS): capture, process and store data streams.



* Amazon Kinesis Data Streams (KDS) is a massively scalable and durable real-time data streaming service. It can continuously capture gigabytes of data per second from hundreds of sources such as website clickstreams, database event streams, financial transactions, social media feeds, IT logs, and location-tracking events.
* Streaming service for ingest at scale
* Has a replay feature
* Provisioned mode
* On-demand mode

Data Firehose: load data streams into AWS data stores



* Load streaming data into S3/Redshift/OpenSearch/3rd party/custom HTTP
* Data Streams + Data Firehose: A perfect combo of technology for loading data near real-time data into S3 and Redshift. Kinesis Data Firehose supports custom data transformations using AWS Lambda.

Data Analytics

Video Streams

The capacity limits of a Kinesis data stream are defined by the number of shards within the data stream. The limits can be exceeded by either data throughput or the number of reading data calls. Each shard allows for 1 MB/s incoming data and 2 MB/s outgoing data. You should increase the number of shards within your data stream to provide enough capacity.

Kinesis Data Stream uses the partition key associated with each data record to determine which shard a given data record belongs to. When you use the identity of each user as the partition key, this ensures the data for each user is ordered hence sent to the same shard.


## Batch

AWS Batch supports multi-node parallel jobs, which enables you to run single jobs that span multiple EC2 instances.

Batch vs Lambda

Lambda:



* Time limit
* Limited runtimes
* Limited temporary disk space • Serverless

Batch:



* No time limit
* Any runtime as long as it’s packaged as a Docker image
* Rely on EBS / instance store for disk space
* Relies on EC2 (can be managed by AWS)


## ParallelCluster

Open-source cluster management tool to deploy HPC on AWS

Automate creation of VPC, Subnet, cluster type and instance types

Ability to enable EFA (Elastic Fabric Adapter) on the cluster (improves network performance)


## Simple Email Service (Amazon SES)


## Pinpoint

Scalable 2-way (outbound/inbound) marketing communications service

Versus Amazon SNS or Amazon SES



* In SNS & SES you managed each message's audience
* In Amazon Pinpoint, you create message templates, delivery schedules, highly-targeted segments, and full campaigns


# Containers


## Elastic Container Service (ECS)

EC2 Launch Type



* Launch Docker containers on AWS = Launch ECS Tasks on ECS Clusters
* Each EC2 Instance must run the ECS Agent to register in the ECS Cluster
* EC2 Instance Profile is the IAM Role used by the ECS Agent on the EC2 instance to execute ECS-specific actions such as pulling Docker images from ECR and storing the container logs into CloudWatch Logs.
* ECS Task Role is the IAM Role used by the ECS task itself. Use when your container wants to call other AWS services like S3, SQS, etc.

**Fargate** Launch Type



* **Serverless** container platform 
* Do not provision the infrastructure, no EC2 to manage.
* Fargate + EFS = Serverless
* AWS Fargate allows you to run your containers on AWS without managing any servers.

Amazon S3 cannot be mounted as a file system.

EFS volume can be shared between different EC2 instances and different ECS Tasks. It can be used as a persistent multi-AZ shared storage for your containers.


## Elastic Container Registry (ECR)

Amazon ECR is a fully managed container registry that makes it easy to store, manage, share, and deploy your container images. It won't help in running your Docker-based applications.

Private/Public **repository**


## Elastic Kubernetes Service (EKS)

Open-source

EKS supports EC2 if you want to deploy worker nodes (managed node groups or self-managed nodes) or Fargate to deploy serverless containers.

Kubernetes is cloud-agnostic

EKS pods run on EKS nodes


## App Runner

Deploy web apps and APIs at scale


# Serverless


## Lambda

Virtual functions - no servers to manage

Lambda's maximum execution time is 15 minutes.

Run **on-demand**

Scaling is automated

Edge function: A code that you write and attach to CloudFront distributions

CloudFront Functions



* Use cases: Cache key normalization, Header manipulation, URL rewrites or redirects, Request authentication & authorization

Lambda@Edge



* Longer execution time (several ms)
* Adjustable CPU or memory
* Your code can depend on 3rd party libraries (AWS SDK to access other AWS services)
* Have network access to use external services for processing
* File system access or access to the body of HTTP requests
* Lambda@Edge is a feature of CloudFront that lets you run code closer to your users, which improves performance and reduces latency.

By default Lambda function is launched outside your own VPC


## API Gateway

Easy way to expose REST API backed by AWS Lambda

Endpoint Types



* Edge-Optimized (default): An Edge-Optimized API Gateway is best for geographically distributed clients. API requests are routed to the nearest CloudFront Edge Location which improves latency. The API Gateway still lives in one AWS Region.
    * As the Edge-Optimized API Gateway is using a custom AWS managed CloudFront distribution behind the scene to route requests across the globe through CloudFront Edge locations, the ACM certificate must be created in us-east-1.
* Reginal
* Private: Can only be accessed from VPC using ENI


## Step Functions

**Workflow Orchestration** - in sequence.

Build a serverless visual workflow to orchestrate your Lambda functions.


## Cognito

Identity provider

Amazon Cognito can be used to federate mobile user accounts and provide them with their own IAM permissions, so they can be able to access their own personal space in the S3 bucket.

Amazon Cognito lets you add user sign-up, sign-in, and access control to your web and mobile apps quickly and easily. Amazon Cognito scales to millions of users and supports sign-in with social identity providers, such as Apple, Facebook, Google, and Amazon, and enterprise identity providers via SAML 2.0 and OpenID Connect.

For hundreds of web/mobile application users who sit **outside** of AWS

Cognito User Pools (CUP)



* A serverless database of users for your web/mobile apps

Cognito Identity Pools (Federated Identities)



* Get identities for “users” so they obtain temporary AWS credentials


# Global Network/Edge Locations


## CloudFront

Content Delivery Network (CDN)

Content is cached at the edge.

Can be used with S3 bucket with **Origin Access Control (OAC)**.

CloudFront is a fast content delivery network (CDN) service that securely delivers data, videos, applications, and APIs to customers globally with low latency, high transfer speeds.

Amazon CloudFront can be used in front of an Application Load Balancer.


## Global Accelerator (newer)

Use Anycast IP concept to work

No caching, requests still make it to the application end.

Improves performance for a wide range of applications over **TCP or UDP**.

**Good fit for non-HTTP use cases.**


# Monitoring


## CloudWatch

Performance monitoring (metrics, CPU, network, etc...) & dashboards

Events & Alerting

Log Aggregation & Analysis


### CloudWatch Metrics

CloudWatch Metric Streams


### CloudWatch Logs

CloudWatch Logs Insights: It’s a query engine, not a real-time engine


### CloudWatch Unified Agent



* By default, no logs from your EC2 machine will go to CloudWatch, you need to run a CloudWatch agent on EC2 to push the log files you want.
* Collect metrics and logs


### Cloudwatch Alarms

Alarms are used to trigger notifications for any metric

CloudWatch Alarm Targets:



* Stop,Terminate, Reboot, or Recover an EC2 Instance
* Trigger Auto Scaling Action
* Send notification to SNS (from which you can do pretty much anything)

CloudWatch Alarms are on a single metric

Composite Alarms are monitoring the states of multiple other alarms


### CloudWatch Insights

CloudWatch Container Insights



* ECS, EKS, Kubernetes on EC2, Fargate, needs agent for Kubernetes • Metrics and logs

CloudWatch Lambda Insights



* Detailed metrics to troubleshoot serverless applications

CloudWatch Contributors Insights



* Find “Top-N” Contributors through CloudWatch Logs

CloudWatch Application Insights



* Automatic dashboard to troubleshoot your application and related AWS services


## CloudTrail

AWS CloudTrail allows you to log, continuously monitor, and retain account activity related to actions across your AWS infrastructure. It provides the event history of your AWS account activity, audit API calls made through the AWS Management Console, AWS SDKs, AWS CLI. So, the EC2 instance termination API call will appear here. You can use CloudTrail to detect unusual activity in your AWS accounts.

You can use the CloudTrail Console to view the last** 90 days **of recorded API activity. For events older than 90 days, use Athena to analyze CloudTrail logs stored in S3.

Provides governance, compliance and audit for your AWS Account.

Record **API calls** made within your Account by everyone

Can define trails for specific resources

Global Service

Events:



* Management Events
* Data Events
* CloudTrail Insights Event: Detect unusual activity


## EventBridge

Default Event Bus

Use case:



* Schedule Cron jobs
* React to an event pattern
* Trigger Lambda functions

Archive and Replay feature


## Config

Helps with auditing and recording **compliance** of your AWS resources

Record **configuration** changes

Evaluate resources against compliance rules

Get timeline of changes and compliance


# Disaster Recovery

RPO: Recovery Point Objective

RTO: Recovery Time Objective

Disaster Recovery Strategies



* Backup and Restore
* Pilot Light: Have only the critical infrastructure up and running in AWS.
* Warm Standby: Full system is up and running, but at minimum size.
* Hot Site /  Multi Site Approach: Full Production Scale is running AWS and On Premise


## Database Migration Service (DMS)

Quickly and securely migrate databases to AWS, resilient, self healing

AWS Schema ConversionTool (SCT): Convert your Database’s Schema from one engine to another


## Backup

Centrally manage and automate backups across AWS services

AWS Backup Vault Lock: Enforce a WORM (Write Once Read Many) state for all the backups that you store in your AWS Backup Vault


## Application Discovery Service

Plan migration projects by gathering information about on-premises data centers

Server utilization data and dependency mapping are important for migrations



* Agentless Discovery (AWS Agentless Discovery Connector): VM inventory,configuration,and performance history such as CPU,memory, and disk usage
* Agent-based Discovery (AWS Application Discovery Agent): System configuration, system performance, running processes, and details of the network connections between systems


## Application Migration Service (MGN)

Converts your physical, virtual, and cloud-based servers to run natively on AWS 


# Data Analytics


## Athena

Data can be stored in S3, only paying for the data that it has access to. 

Serverless query service to **analyze data stored in Amazon S3**

Suitable for something to run once in a while.


## Redshift

Not serverless

Analytics Queries/Data Mining.

Snowflake’s competitor.

vs Athena: faster queries / joins / aggregations thanks to indexes

Enhanced VPC Routing in Redshift forces all COPY and UNLOAD traffic moving between your cluster and data repositories through your VPCs


## OpenSearch

In DynamoDB, queries only exist by primary key or indexes. With OpenSearch, you can search any field, even partial matches. It’s common to use OpenSearch as a complement to another database.

Centrally store data and efficiently search and analyze them in real-time.


## Elastic MapReduce (EMR)

EMR helps create Hadoop clusters (Big Data) to analyze and process vast amounts of data.

EMR makes it easy and cost-effective for data engineers and analysts to run applications built using open source big data frameworks such as Apache Spark, Hive, or Presto without having to operate or manage clusters.


## QuickSight

Serverless machine learning-powered business intelligence service to create interactive dashboards.


## Glue

Managed extract, transform, and load (ETL) service

Useful to **prepare and transform data for analytics**

Fully serverless service

Glue Data Catalog: catalog of datasets

**Glue Job Bookmarks** allows you to save and track the data that has already been processed during a previous run of a Glue ETL job.

Gets data from S3


## Lake Formation

Data lake = central place to have all your data for analytics purposes

Fully managed service that makes it easy to setup a data lake in days

Discover, cleanse, transform, and ingest data into your Data Lake


## Kinesis Data Analytics

For SQL applications



* Read from Kinesis Data Streams or Data Firehose, add reference data from S3, perform real-time analytics, send to Data Streams and Data Firehose.

For Apache Flink


## Managed Streaming for Apache Kafka (MSK)


# Machine Learning


## Rekognition

Find objects, people, text, scenes in images and videos using ML


## Transcribe

Uses a deep learning process called automatic speech recognition (ASR) to **convert speech to text** quickly and accurately 


## Polly

Turn text into lifelike speech using deep learning

Amazon Polly allows you to turn text into speech. It has two important features. First is Pronunciation Lexicons which allows you to customize the pronunciation of words (e.g., “Amazon EC2” will be “Amazon Elastic Compute Cloud”). The second is Speech Synthesis Markup Language (SSML) which allows you to emphasize words, including breathing sounds, whispering, and more.


## Translate


## Lex & Connect

Lex



* Same technology that powers Alexa
* **Automatic Speech Recognition (ASR) **to convert speech to text, chatbot

Connect



* Receive calls, create contact flows, cloud-based virtual **contact center**


## Comprehend

For Natural Language Processing – NLP

Comprehend Medical


## Textract

Automatically extracts text, handwriting, and data from any scanned documents using AI and ML

Textract and Comprehend are HIPPA compliant.


## SageMaker

Fully managed service for developers / data scientists to build ML models


## Forecast

Fully managed service that uses ML to deliver highly accurate forecasts


## Kendra

Fully managed **document search service** powered by Machine Learning 


## Personalize

Fully managed ML-service to build apps with real-time personalized recommendations

Same technology used by Amazon.com


# WhitePaper

Well Architected Framework 6 Pillars



* Operational Excellence
* Security
* Reliability
* Performance Efficiency
* Cost Optimization
* Sustainability


## Trusted Advisor

Analyze your AWS accounts and provides recommendation on 5 categories



* Cost optimization
* Performance
* Security
* Fault tolerance
* Service limits