# AWS DevOps Readiness Exam
## SDLC Automation
- CI/CD Pipeline
- SCM
- Automation and Testing
- Build and Manage Artifacts
- Deployment and Delivery Strategies, and how to implement them using AWS

### CI:
- Manual step
- Automatically kick on a new release when code is checked in
- build and test code, repeatable
- have an artifact ready
- have feedback when build fails

### Delivery
End result, ends up having a an artifact after its being packaged

### Deployment
Automatically deploy new changes to stage for test
(Ops Work, Elastic Beanstalk, Code Deploy)
Deploy to production without impacting customers
Delivery to customers faster

### Immutable Infra
- need to be able to trash and redo as much as possible

- - - -
### Services
#### (SCM) - Code Commit
	- Integrate with IAM Services
	- Can trigger next step of the CI Pipeline
	- Encrypted by default
	- Other options include Bitbucket and Github
	- Multiple Branches (best practices)
	- Automatically integrated to CW Events
	- 
#### Code Build
	- Fully managed build Service (build and test code)
	- Pay as you go
	- It defines the code build build_spec.yml
	- Can connect to external dependencies no only Code Commit
	- all of them need a build_spec.yml file to actually build

Steps:
	1. Build Project (CPU Ram, Env, Custom Image, The environment information)
	2. This is a serverless option (all logs get to CW Logs)

#### Jenkins
	- Need to know 3 plugins
		- EC2
			- Spins slaves 
			- its an auto scaling
		- Jenkins triggers Code Build (offload) - Jenkins as Organization
		- Code Commit trigger Jenkins
#### Code Deploy
	- Deploys the code ( for EC2 / Lambda / ECS and also on Premise, needs an agent)
	- Need to have an instruction file (AppSpec file)
	- Don’t assume the agent is installed
	- SCM + AppSpect file -> create a revision
	- It has automatic rollback
	- Supports Deployment strategies (in place, rolling, blue/green)
	- Supports Linux + Windows
	- It can deploy to ASG or Group fo Tagged Instances

#### Code Pipeline
	- Structured the pipeline. Can use multiple tools
	- Calls to other resources
	- Manual intervention can happen in the Code Pipeline
	- Unit and Integration testing are key steps 
	- Stages:
		- Source (where is it coming from)
		- Build (how to build the app)
		- Test (How to test the app)
		- Deploy (how should be deployed)
		- Invoke (Custom functions to invoke)
		- Approval (Send SNS to approve or deny the pipeline)
	- Can work multiple regions / multiple accounts
		- need the right role

#### Automated Testing
	- Unit 
	- Service / Integration
	- Performance
	- UI

#### fault Tolerance
	- Highly available
	- Mission Critical Apps 

#### Deployment Strategies
	- (this is the main area)
	- In Place Deployments
		- Remove and Deploy (very impactful)
		- Very quickly
		- *HAS* Downtime
		- Least expensive
	- Rolling Updates
		- Validation happens after creating the instances
		- Code Deploy *talks* to ELB
		- Costs (Price) is a factor
		- There is no downtime
	- Canary
		- Mainly (for test)
		- Use only a handful or one to validate
		- Testing strategy
		- Resizing Architecture
	- Blue / Green
		- Using Route 53 (mainly)
		- Easy Roll Back
	- Red / Black
		- Faster than Blue/Green
		- No Scaling Period
		- Speed**** is why this is it
		- Cost ** -> this is not it
		- This is it for performance
	- Immutable Upgrades
		- Options to use
			- Elastic Beanstalk
			- AWS Opswork
		-  Similar to Red / Black but this is for the entire infrastructure not only for the instances
	- Rolling with Additional Batch
		- Only for Elastic Beanstalk
		- Scales out first before doing anything else

### Test Axioms
	- Know the code services
	- Update and Recover from issues
	- Deployment strategy
	- Remember to test (unit) everything


## Config Management and IAC
- Determine deployment services depending on needs
- App and Infra models based on business needs
- Security

### CloudFormation
	- IAC
	- Make everything repeatable, this can be used in the CI/CD
	- Templates
		- Parameters
		- Mappings
		- Conditions
		- Resources
		- Outputs
	- Can be added to an architecture Template -> AWS CFN Engine -> Create Architecture
	- Nesting Stacks
		- Parent template can Ref! the values
		- Don’t include hard coded values
		- Outputs create dependency on the names
			- Best option is to use Nested and use Reference so it doesn’t have that dependency  
	- Updating Stacks / Change Sets
		- Original -> Change Set -> View Change -> Execute
	- Drift Detection 
		- Check the env state is, and what it should be
	- Dependency
		- DependsOn
			- If something depends on something you have to *explicitly* call it out
	- Wait Conditions
		- Never chose it if there is a Creation Policy
	- Creation Policy
		- Newer than Wait Condition
	- Lock Down Stack Resources
		- Need to have a generic effect *Allow* to everything, if not the least privileges will occur
	- Custom Resources
		- Creates a Lambda Resources and then reports to CFN like talking to Premise or Peer VPCs
	- StackSets
		- Create Stacks in multiple accounts
	- Updates:
		- Updates with no interruption
			- check if the change produces a change
		- Updates with some interruption
		- Replacement
			- terminate and create new stuff
		- Options:
			- In Place = Red / Black
			- Rolling = Blue / Green
				- Max Batch Size
				- MinInstanceService
				- PauseTime
				- WaitOnResourceSignals
				- Suspend Process
				- Min Successful Instance Percent
	- Helper Scripts
		- cfn-init
			- applies metadata instructiions
			- installs, creates, etc (kind of CM - Puppet, Chef, etc)
			- It runs *only once* even if you update the stack
		- cfn-hup
			- Deamon that pulls from changes of your metadata
			- this will kick the cfn-init if you need
			- Works together with the `init`
			- Interval is by default 15 minutes, number of minutes to poll
		- cfn-signal
			- Alerting the CFN
		- cfn-get-metadata
			- Dumps the metadata of the instance to the host

### Deployment Options 
	- Always wrap everything in CFN
#### Elastic Beanstalk
	- Automated Infra management and code deployment for your application
	- Includes 
		- Load Balancing
		- Health Monitoring
		- ASG
		- App Management
		- Code Deployment
	- Terms
		- Environment 
			- Versions
			- Types:
				- Web Tier (Public Facing)
				- Worker (Non Public - Back End), Its subscribes to SQS
	- Everything is contained in one *Application*
		- Multiple versions
		- Different Environments
		- EB CLI is the cli for ElasticBeanstalk and its separate from the aws cli
	- .ebextensions
		- It has the configuration of the entire ElasticBeanstalk
		- it gets overrides if you select the value t hat contradicts it (it can be in the UI or the restoring 
		- ebextensions are done alphabetically
			- one can override the first one
	- EB wraps ECS
		- if its not in the preferred languages, you can put it in a container and then it works
#### Opswork
	- Chef or Puppet
	- Stacks
		- Supports Chef Recipes, 
		- Bash / PowerShell
		- Automatic Instance Scaling and auto Healing
		- Supports Any Server
		- Creates Layers
			- ELB
			- App
			- ETC
	- Chef 
		- Only Chef
	- Puppet
		- Only Puppet

#### Containers
	- EKS
	- ECS
	- EC2 (long running tasks)
	- Fargate (short running tasks)
	- ECR

#### Serverless
	- Stateless Code
	- Custom Runtimes
	- Supports Node, Java, Python, C#, Go and Ruby
	- Runs code on a schedule or response to events
	- Sync or Async

#### Api Gateway
	- Protection from DDOS and injection attacks
	- Deeply integrates with Lambda
	- Endpoint to your VPC

### Test Axioms
Know the CFN in depth
Know when and where to use EB/CNF/Opswork

## Monitoring and Logging
	- Determine how to set up aggregation, storage and analysis of log and metrics
	- apply concepts to automate monitoring and event management 
	- audit/log, monitoring OS
	- Tagging
	- no manual step here. It should always be automatic with no manual impact
Metrics kept for a period of 15 months

	- Metrics 
	- Logs
		- never expire
		- Need to install and configure aws logs  agent
		- Compared to syslog service
		- Analyze logs thru filters
		- Logs Examples ELB Logs
			- Client:port
			- request_processing_time
			- response_processing_time
			- request
			- chosen_cert_arn
	- Alarms
	- Events
	- Rules
	- Targets

### Services
	- CLoudwatch
	- CloudTrail
		- it takes on avg 10/15 mins to show in the logs
		- Best Practices
			- Enable in all regions
			- enable logs file
			- Enable encryption
			- Enable to Kinesis or S3 so you can query
	- XRay
		- Identify performance bottlenecks and errors
	- Logging in AWS
		- CW Agent -> Metrics / Streams 
	- Log Event -> Log Group -> Log Stream 
	- VPC Flow Logs
		- Capture Ip Traffic Flow details in your VPC
		- Enabled for VPC/Subnet/ENI
		- Publish to CW Logs
	- Kinesis 
		- Collect, process and analyze real time or near real time streaming data for quick Incident Reponse
		- Options:
			- Data Stream
				- for custom built apps to process data
				- real time dashboards
				- generate alerts and dynamic pricing
			- Data Firehouse
				- Loading streaming data directly into was storage and processing data to another was service
			- Data Analytics
				- Run SQL Queries in the logs
### Tagging
	- Enforcing

## Policies and Standards Automation
	- Enforce standards for logging, metrics, monitoring, testing and security
	- How to optimize cost via automation
	- Concepts required to implement governance strategies

### Security
- USers 
- Groups
- Role
	- Trust Policy (who is allowed to assume this role)
	- Access Permission (what actions)
- Policies
	- MFA is mostly available when doing destructive things
	- Conditions:
		- IP Range
		- Tags
		- MFA
	- Service Controlled Policies
		- Turn off services / feature
		- Can be done in the Root Account, 
		- The root account can do anything
	- Permissions Boundaries
		- Similar to Service Controller Policies, but only for the individual

### Data Protection 
	- Encryption at Rest
	- Certificate manager
	- IPSEC VPNS Encapsulate traffic
	- SSL Termination at the ELB 
	- Cloudfront
		- SSL Termination
		- CDN
	- Stateful = SG
	- Stateless = ACL
	- Data Protection
		- Automatically encrypt data
			- Glacier
		- S3, EBS, EFS its not encrypted by default, it gives you the option
			- You can use into KMS for encryption keys
	- Guard Duty
		- Network Protector
		- Protects AWS accounts and workload
		- Monitors was env for suspicious 
		- tells you what’s wrong *doesn’t solve it*
		- Scans:
			- DNS Query logs
			- VPC Flow Logs
			- CW Events
	- Inspector
		- Detects Vulnerabilities
		- OS Level
		- tells you what’s wrong *doesn’t solve it*
		- Agent based solution
		- Security Best Practice
		- Automated Assessment that helps improve security and compliance of apps
		- Agent that you have to install 
	- System Manager
		- On Cloud and On Prem
		- Automate Admin tasks 
			- Collect inventory
			- Apply Patches
			- Create system images
			- Config Win/Linux OS
			- Patch Management
				- have a base line
				- Create a maintainability window for patching
				- Apply Patches
				- Audit and have compliance
				- Its done by tagging
	- Parameter Store 
		- free
	- Secrets Manager
		- values encrypted by default
		- strictly for creds
		- automated credentials rotation
	- AWS License Manager
		- license tracking and enforce usage roles
	- Config 
		- Track Resource config changes
		- Compliance and Analyze Security
		- Evaluate against policies defined in the roles
	- Trusted Advisor
		- Reduce Cost
		- Increase Performance
		- Improve Security 
	- Tagging
	- Service Catalog
		- Limit Access to underlying AWS Service
		- Create and manage catalogs of approved IT services
		- Wrapper of CFN for services 
		- One stop shop of applications
## Incident and Event Response
	- Automated healing
	- Event driven automated actions
	- automate event management and alerting
	- Logging + Xray
Steady State ASG (same number in min, max, required)



## High Availability, Fault Tolerance and DR
	- RDS - cross region snapshot
	- cross region read replica
	- Amazong DDB global tables
	- S3 -> cross region replicate
		- Only updates not deletes
	- Copy EBS snapshots
	- Scaling
		- Lambda@Edge
		- Scale not only from CPU but other metrics as well like SQS
	- DR  
		- RTO (recovery time objective)
			- how much data can you afford to recreate or lose
			- how quickly  must you recover / cost of downtime
			- Backup and restore
				- s3
			- Pilot Light
				- asg in 0, just scale it up
			- Warm Standby
				- things are running
			- Hot Standby
				- entire iac is running
	- Failover
		- Eliminate all single point of failures
		- 
