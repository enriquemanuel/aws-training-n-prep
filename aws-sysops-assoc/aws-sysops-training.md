# AWS Training - System Operations

Trainer: Karthik Chandy
E-Mail: chandyK@amazon.com

## Day 1

*TIPS:*

- You can only create 5 vpc in a single region (hard limit)
- Instance Profile are Roles named differently

## Day 2

### Compute (ELB, ASG, Route 53)

- **ELB**
  - The ELB provides:
    - High Availability
    - Health Check
    - Security Features
    - TLS Termination
    - Layer 4 or Layer 7 LB
    - Operational Monitoring
  - ELB has 3 types:
    - Classic (tcp and http) [old]
      - Supports Layer 4 and Layer 7 of OSI Model
    - Application Load Balancer (http)
      - Only Layer 7
    - Network Load Balancer (tcp) [newest]
      - Only Layer 4
  - Benefits
    - The benefit of the ELB is attach it to an Auto Scaling Group (ASG), this way it automagically creates different endpoints in the different Availability Zones.
  - The Load Balancers allow Cross Zone, which can manage different subnets in AZ

- **ASG**
  - Without CW you will need to manually invoke the policies. CW makes the magic of ASG.
  - It provides:
    - Health Check Status
    - User defined policies driven by CW
    - Schedules
    - Other criteria (programmatically)
    - Manually using set-desired-capacity
    - Scale out to meet demands (main reason why we go to the cloud)
  - How does it get configured?
    1. We need a Launch Configuration that contains:
        - AMI
        - Instance type
        - SG
        - Key Pair
        - Storage
        - IAM Roles / Instance Profile
        - User Data
    2. Specifics of the ASG
        - Automatically scale between
          - Min
          - Desired (Optimal value)
          - Max
        - Health Checks to maintain the size (what to monitor)
        - Distribute across the AZ
    3. Policies of the ASG
        - Parameters for performing an EC2 action
        - how to trigger this events
          - (CW) Cloud Watch driven
          - Instance Failure (healths checks)
          - Scheduled
          - Manually
        - It has scale out / in by how much
          - ChangeInCapacity
          - Exact capacity
          - Change in Percentage
        - Cool down period
          - Ignore notifications for a period that it allows me to create my instances.
  - How does terminate any instance?
      1. Defaults: Closest to the next hour | Oldest Launch Configuration
      2. There are other values but the default are the above
  - Creating a Steady State Group
    - A Steady state group allows that if an instance is unhealthy in one zone it automatically will create another instance. _To be set, you need to have Min, Max and Desired state to the same, usually 1._
    - Instance is recreated automatically if the AZ fails.
    - It will have down time, since you only have 1 instance at all times.
  - In the weeds with ASG
    - If there is are two of the **same** CW alarms, it will just  act on the first one.
    - If it already acted on the first alarm, but it comes a **second alarm (different)** it will match the required size, so if the first one was add 1 and the second alarm was to add 2, it will just add 1.

  - **Curious Notes:**
    - Since ASG integrates with CW, you can have a metric to monitor SQS or any other tool inside AWS to trigger an event (scale in our out) the ASG.
    - This means that it doesn't have to be a LB.
    - The benefit of having a LB, it provides certificate manager, some sort of DDOS, Application Firewall, an other tools, but an ASG can exist without a LB.

- **Route 53**
  - Its a service for DNS. Register the Domain Names.
  - FQDN [www.xyz.com] can be routed:
    - A record (IP)
    - CNAME record (DNS / ELB)
  - Top Level (Zone APEX) [xyz.com] can be routed:
    - A Record (IP)
    - CNAME (DNS / ELB) -> THIS DOES NOT WORK, FORBIDDEN by RFC 1033
    - ALIAS Record (DNS / ELB)
  - Routing policies
    - Simple Routing
      - Don't care, just give it to them
      - The only reason it wont give it, if its unhealthy
    - Weighted Routing
      - Giving a weight for each route so it behaves a normal
    - Failover
      - If its not working, don't give it
    - Latency
      - Will check from the endpoints which provides the least latency (its not physical distance)
      - This is the most expensive option
    - Geolocation routing
      - Send the traffic to a route that might have the option to respond that location (language)
    - Geo Proximity routing
      - You have some benefits of Weighted but its very similar to geolocation.
      - I want to allocate weight to this request
    - Multi value answer routing
      - it combines weight with latency.

- **Testing Scalability**
  - The Grinder
  - JMeter
  - Bees with Machines Guns

- **Compute (Containers and Serverless)**

  - **Containers**
    - Run an application an its dependencies in a resources isolated process
    - Challenges
      - Bin Packing
      - Ownership
      - Versions
      - Zombie containers
      - Disappearing containers
    - ECS Benefits [ Management ]
      - Cluster management made easy
      - Flexible scheduling
      - Integrated and extensible
      - Security
      - Performance at scale
      - task definitions

  - **ECR**
    - Managed container registry (docker hub but AWS's)

  - **EKS**
    - Managed Kubernetes service

  - **Lambda**
    - Bring your own code
    - Completely automated administration / scaling
    - built in fault tolerance
    - orchestrate multiple functions
    - integrated security model
    - pay per use (per ms)
    - flexible resource (1 million per month 4 free)
    - Limitations
      - No more than 3Gb
      - No more than 5 mins
      - Can't specify the CPU, but its proportional to memory

  - **API Gateway**
    - Essentially an URL endpoint that can invoke anything including lambdas

  - **Batch**
    - A way to process a queue in order via job/jobs
    - Components:
      - Jobs (a unit of work)
        - it can be shell script, docker container, python
      - Job definitions how a job should be run
      - Job queue
      - Compute environment
        - set of managed of unmanaged compute resources to run jobs

### Database Services

- Choose the right database for the job, based on purpose and application needs.
- SQL vs NoSQL options - something that needs to be analyzed
- Types:
  - Amazon RDS (SQL)
  - Aurora (SQL)
  - DynamoDB (NoSQL)
- Backups:
  - Manual (creates a full storage volume snapshot of the DB)
  - Automatic (created automatic snapshots incremental)
- High Availability
  - Master syncs data to the secondary
  - You can't connect to the secondary
- Scaling
  - Instance Type (with min downtime)
  - Storage Capacity (done on the fly)
  - Read workloads could use Read Replicas
- **Aurora**
  - Customized variable of MySQL / PostgreSQL, offers some advantages.
  - Created by AWS from the Open Source from Mysql and Posgres. Easier to manage and still be an relational database.
  - This will have new features availables like:
    - Multi Master (High Availability)
    - Serverless (Fully managed service that only needs compute)
  - Features
    1. fast
    2. simple
    3. compatible with everything out of the box
    4. pay as you go
- **DynamoDB**
  - Fast and flexible non relational database service for any scale.
  - Features
    - DynamoDB Accelerator (DAX), in memory caching for the DB
    - Global Tables
    - Encryption at Rest
    - Integrated Monitoring
    - Backup and Restore
    - Zero admin
    - Replication is Async
- **ElastiCache**
  - Managed in memory data store services (redis, memcache, etc)
- **Amazon Neptune**
  - GraphSQL
- **Redshift**
  - Data warehousing (analytics)
- **Database Migration Services (DMS)**
  - Supports migrations the same type or non same type

### Networking

- **VPC**
  - Provision a logically isolated section of the AWS in a virtual network you define.
    - Supports logical separation with subnets
    - Fine grained security
    - Restrict VPC to ranges defined in RFC1918
    - Biggest network will be a /16
    - Smallest network will be a /28
  - As soon as a VPC gets created it creates a route table with default values with the CIDR block and target locally.
- **Subnets**
  - Property of a VPC
  - Specific Zone only
  - You can have multiple subnets in an AZ
  - Use the local route to talk to each other
  - Has its own CIDR
  - 5 Ip's are not usable
    - **First IP** Network Address
    - 2nd Reserved by AWS for the VPC router.
    - 3rd Reserved by AWS for mapping to the Amazon-provided DNS. (Note that the IP address of the DNS server is the base of the VPC network range plus two. For more information, see Amazon DNS Server.)
    - 4th Future use
    - **Last IP** Network broadcast address. We do not support broadcast in a VPC, therefore we reserve this address.
  - Auto Assign Public IP - can be assigned and modify the feature in the Subnet
- **Route Tables**
  - You can create route tables that will define the route that it takes to communicate.
    - The priority is **how direct its to communicate to the other ip**
- **VPC Peering**
  - Communicate between VPC's bidirectional.
  - They can't share the same CIDR range.
  - You have to alter / update the route tables so they know they exist and how it works.
  - **Check Restrictions**
- **VPN**
  - You have to set a Virtual Private Gateway (VGW)
    - this is the configuration of the device, not the device itself
  - You have to set a Customer Gateway (CGW)
    - this is the configuration of the device at the client side
  - Create a VPN connection
    - this is where you link both VGW and CGW
  - CIDR ranges can't overlap
  - You have to alter the route table to use the VGW
  - You can have a VPN Cloud Hub (multiple vpn connections to one point)
- **Direct Connect**
  - A way to have a dedicated line to connect your data center to AWS
- **NAT (network address translation)**
  - NAT Gateway
    - up to 10 Gbps connection
    - needs to be created in the public subnet
    - Support TCP, UDP, ICMP protocols
    - built in redundancy for HA
  - NAT Instance
    - You have to manage the instance and configure route tables
    - Cost is better
    - It can do port translation paths
- **VPC Endpoints**
  - They are mappings of AWS Services to be accessible from inside the VPC and not have to go to the internet.
  - Services like S3 and DynamoDB that are available on the internet, can now behave as intranet devices inside the VPC.
  - This is free
- **VPC Endpoint PrivateLink**
  - This has a cost
  - With this service they can communicate to supported AWS Services without going to the internet
  - Interfaces endpoints are powered by PrivateLink
- **ENI's**
  - Max 15 ENI per instance
  - 50 EIP per ENI
- **Network Defense for VPC**
  - Lower your attack surface and increase your layers of security
- **Security Group**
  - They are stateful
  - Essentially a Firewall
  - They are at the instance level
- **Network ACL**
  - at the subnet level
  - Stateless
  - Hardens the security
  - Default:
    - Allows all inbound and outbound traffic

## Day 3

### Certification Preparation

- aws.amazon.com/certification
- 70% of the SysOps Administration is covered by the class

#### How to best prepare

- read the blueprint (it should already be what you covered in the class)
- Step 4 (The FAQ is **very important**)

### Storage

- **EBS**
  - This is block storage (not a file storage)
  - you define the format of the disk
  - Network Attached
  - persistent block storage
  - boot and data volumes supported
  - Maximum 16TB
  - _They are tied to a AZ, they can't be attached to a different AZ._
    - To workaround this, create a new volume from a snapshot.
  - Families:
    - SSD
      - Provisioned IPS SSD (IO1)
        - Max of 32k IOPS
      - General Purpose SSD (GP2) [ default ]
        - Maximum of 10k IOPS
    - HDD
      - Throughput Optimized (ST1)
      - Cold HDD (SC1)
  - EBS Optimization
    - gives you a dedicate path from the EC2 to the EBS (not shared)
    - there are 2 instances that offer this optimization out of the box
    - you can request on all the other instances if its not offered
    - EC2 Nitro Systems
      - this have been optimized and built by AWS to enable high performance and HA and security
  - EBS Incremental Snapshots
    - Snapshots are stored in AWS S3 behind the scenes (you can't download or see)
    - They are incremental (deltas)
    - they reference the other snapshot so they keep it as minimal
      - if deleted it will copy the reference and data to the snapshot being pointed
  - Restoring Snapshot
    - Create a volume and point the snapshot
  - LifeCycle Management
    - Automate creation, retention and deletion of snapshots

- **Instance Store (ephemeral)**
  - Block level storage on a shared disk
  - It comes with an instance
    - not all instances have it
  - use cases:
    - buffers
    - cache
    - scratch data
  - they are super fast
  - reclaimed when instance stopped or terminated

- **EFS**
  - NAS based service
  - Uses NFS v4
  - Shared storage
  - Linked to a VPC
  - Any member across the VPC can use and shared the volume
  - If you want to use it in windows, you need to install a plugin since it doesn't use NFS

- **S3**
  - Simple storage service
  - Can be accessed anywhere (both inside and outside)
  - highly durable (11s 9s)
  - High available (4s 9s)
  - no limit to the amount of data stored
  - biggest to 5TB per object
  - smallest size is 0b (metadata)
  - you pay for consume capacity
  - its object storage
  - via https
  - S3API Commands (for when the CLI is not enough)
    - multi part upload
      - break files in multi chunks
      - stops, pause, resumes
      - after completes it runs a check sum and joins the data
    - put object ACL
    - put bucket version
  - Versions
    - Create a new object with a different version
    - its not enabled by default
    - adds a delete marker so its not deleted
  - Storage Classes (types)
    - General Purpose (Standard)
      - it costs more to store
      - the retrieval is super cheap
    - Infrequent Access
      - its a the object level
      - it costs the same to store as to request it
    - One Zone infrequent Access
      - Similar to IA but with only one zone

- **Glacier**
  - Very good for store, but it takes long to retrieve
  - There are different types of retrieval but doesn't have to specify at the start
    - Expedite
    - Standard
    - Infrequent
  - Super low cost

### Monitoring

- **CloudWatch**
  - Monitoring and management service that provides data and actionable insight for resources.
  - Features:
    - Monitor
      - Monitors the state and utilization of most of the resources
      - Concepts:
        - Common Metrics (complementary)
          - CPU
        - Custom Metrics (need to install a script to send to CW)
          - Memory
        - Alarms
        - Notifications
        - Metrics of how the instance (application) is behaving is not visible
        - You can create alarms to send the other locations
      - Metrics:
        - Type Standard
          - Group by service name
          - only appear if you have used the service within the past 15 months
          - display graphically so metrics can be compared
          - Reachable via API or CLI
        - Type Custom
          - Grouped by user defined
          - Publish to CW using CLI or API
        - Components:
          - Metric
          - Namespace
          - Dimensions
          - Period
      - Alarms
        - Test against a specific threshold
        - Needs 5 minutes to start
        - States:
          - OK
          - ALARM
          - Insufficient Data
      - Detailed Monitoring
        - Default: every 5 minutes (free)
        - Detailed is every 1 minute (additional cost)
    - Events
      - Create rules and targets
      - Example: Upon Launch Successful, Run SSM in that instance
    - Logs
      - Configure
      - Collect
      - Analyze

- **CloudTrail**
  - AWS logging (auditing AWS actions)
  - Log, continuously monitor and retain account activity related to actions across your AWS infrastructure
  - Automatically pushes logs to S3
  - Examples:
    - Who changed a SG
    - IP denied
    - Access
  - Best practices
    - Defined a trail log to desired events
    - Send to a centralized location from different account to a centralized Bucket
    - Admins shouldn't have access to S3 bucket
    - Turn on log file encryption
    - Add tags to the trails
  - Companies to validate and see the logs
    - Exacta
    - Allgress

- **Config**
  - Gives an idea of what exists in your account.
  - Place to see the 10k view of the usage
  - Provides a monitor of configuration changes
  - Runs compliance checks and can analyze the resources.

- **GuardDuty**
  - Uses AI to protect and workloads
  - Analyzes data sources including logs (dns logs, flow logs, cloudtrail)

### Managing Resource Consumption

- **Tagging**
  - Environment
  - Cost Center
  - Ownership
  - Dept
  - Identification
  - Work with Config to enforce tagging
  - Get creative on how to use tags and what they can be.

- **Cost Management Strategies**
  - start up / shut down instances with a specific tag simultaneously
  - "tag or terminate" [ netflix comformity monkey ]

- **Cost Reduction**
  - Right size isntances
  - consider t2 instances for workloads that occasionally require to burst
  - consider purchasing RI for long running instances
  - batch processing jobs can be run in parallel and shut down when its completed
    - maybe consider lambda
  - AWS Trusted Advisor
    - core checks and recommendations available to all customers
    - additional checks recomendations with business and enterprise support plans
    - checks:
      - cost
      - performance
      - security
      - fault tolerance
      - service limit
  - **TIPS**
  - EBS GP2 IOPS calculation
    - X GB * 3 = Y IOPS
    - 30 GB * 3 = 900 IOPS

### Creating Automated and Repeatable Deployments

- **why**
  - Launch new resources quickly
  - deploy configuration changes to running instances
  - make configurations aurtomated and repeatable
- **Technologies to configure**
  - User data
  - ami
  - opswork
  - cloudformation
- **AMI**
  - You can create an AMI from an isntance
  - For Linux (only) on top of the above, you can create an AMI from a volume
  - For Windows, must run _sysprep_ to strip instance-specific networking information
    - EC2 Launch supports a Shutdown with _sysprep_ option
  - AMI's are specific to a region
  - Copy may fail if AWS cannot find a corresponding Kernel in that region
- **CloudFormation**
  - declarative programming language for deploying AWS resources
  - can be code in YAML or JSON
  - supports many resources
  - Steps:
    - Define a template
    - Create Stack
  - WaitCondition and WaitConditionHandle
    - passes signals between apps when the jobs completes (error or not)
    - are only binaries (success or not)
    - its use when bootstrapping a resource
  - Important to have the outputs
  - Default Behaviour on failure:
    - rollback
      - it removes everything (logs and all)
    - change the default to "do nothing"
      - with this the resources are not deleted and you can check the logs.
    - **Best Practices**
      - do not embed credentials
      - use aws specific parameters
      - use parameters contraints
      - validate templates before using them
      - use aws:cnf:init to deploy software application on ec2 instances
    - **Troubleshooting**
      - if pushing from s3, bucket needs to be accesible
      - permissions
      - missing parameters
      
      
