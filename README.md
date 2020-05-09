# AWS-devops-prep-notes


# CI & CD:

========
  
2 core software development processes
  
CI process of automating regular code commits followed by an automated build and test process designed to highlight intergration issues early.

Additional tooling and functionality provided by Bamboo, CruiseControl, Jenkins, Go and TeamCity etc.
workflow based

CD takes the form of a workflow based process which accepts a tested software build payload from a CI server. Automates the deployment into a working QA, Pre-prod or Prod environment.

AWS CodeDeploy and CodePipeline provide CI/CD services

Elasticbeanstalk and CFN provide functionality which can be utilized by CI/CD servers.

## Deployment Types:

=================

1. Single Target Deployment - small dev projects, legacy or non-HA infrastructure; outage occurs in case of failure, testing opportunity is limited.

2. All-at-Once Deployment - deployment happens on multiple targets, requires Orchestration tools, suitable for non critical apps in 5-10 range.

3. Minimum in-service Deployment - keeps min in-service targets and deploy in multiple stages, suitable for large environments, allow automated testing, no downtime

4. Rolling Deployments - x targets per stage, happens in multiple stages, after completion of stage 1, next stage begins, orchestration and health check required, can be least efficient if x is smaller, allow automated testing, no downtime if x is not large to impact application, can be paused, allowing multi-version testing.

5. Blue Green Deployment - Deploy to seperate Green environment, update the code on Green, extra cost due to duplicate env during deployment, Deployment is rapid, cutover and migration is clean(DNS Change), Rollback easy(DNS regression), can be fully automates using CFN etc.
Binary, No Traffic Split, not used to feature test

A/B Testing - distribution traffic between blue/green, allows gradual performance/stability/health analysis, allows new feature testing, rollback is quick, end goal of A/B testing is not migration, Uses Route 53 for DNS resolution, 2 records one pointing A, other pointing B, weighted/round robin.

Bootstraping: process starts with base image (ISO/AMI) and via automation build on it to create more complex object.
Cloud Init, CFN Init (complex bootstraping engine)

AMI (Baking) based approach large number of AMIs even for a small system change. Ex: Quickstart
Bootstraping - every option like OS, patches, dependencies, applications can be tweaked. Ex: Quick launch

Immutable Architecture - Replace infra instead of upgrading or repairing faulty components, treat servers as unchangeable objects, don't diagnose and fix, throw away and re-create, Nothing bootstraped except AMI.
Example: Pets vs Cattles

Container/Docker: contains only application and dependencies, achieves higher density, and improved portability by removing per container Guest OS.

Escape for dependency hell, Consistent progression from DEV->TEST->QA->PROD, Isolation, resource scheduling at micro level, Extreme code portability, Micro-services; Docker Image(Read Only), Docker container (hold everything required to run), Layers/UFS (combines layers using UFS), Dockerfile, Docker Daemon/Engine, Docker Client (Interface between user and Docker Engine), Docker Registry/Hub (Private/Public store for Docker images)

## CloudFormation: 

================

Building block service to provision infra within AWS, Elastic Beanstalk uses CFN, JSON format, Stack (CFN unit of grouping infra), Template (JSON doc giving instructions for CFN on how to act and what to create/update), Stack Policy (IAM style policy governs what/who can change, can be added via CLI or UI, updated but can not be removed)

Create CFN template -> Add template to CFN -> Create CFNStack -> Resources (200 per template) -> Update template/Stack -> Delete Stack
Template Anatomy: Parameters (pass variables into a template), Mappings (allow processing of hash's by CFN template), Resources, Outputs (results from template); Only Resources is mandatory. CFN can run scripts within instances, expand files within instances, stack id(unique).

Use cases: 
1. Template for manual deployment of bespoke infra. 
2. create repeatable patterned environment (ex:wordpress site with DB)
3. run automated testing for CI/CD environments (dev, test, prod)
4. define an environment once, deployed 
5. manage infra config using software development style versioning and testing concepts.
IMPORTANT: Template should be designed to work 1 or 1000 app in one or more regions.

## CloudFormation Structure:

=========================

Parameters - way of passing data into CFN template one or more values; ex: ip address, instance size, name, password etc;

AWS::EC2::KeyPair:KeyName; Default value, Allowed values, Allowed Patterns, Min & MaxValue, Min & MaxLength;

Outputs - way of displaying results of stack creation; A stack can have many outputs, each output can be constructed value, parameter 
references, pseudo parameters or an output from a function such as fn::GetAtt or Ref; Ref references resource provide primary value such
as instance id; GetAtt provide alternate values such as private ip & public ip.

## Intrinsic & Conditional Functions

=================================

Intrinsic Fn - inbuilt function provided by AWS to help manage, reference, and condtionally act upon resources, situation & inputs to a stack.

Fn::Base64 - Base64 encoding for User Data

Fn::FindInMap - Mapping lookup

Fn::GetAtt - Advanced reference look up

Fn::GetAZs - retrieve list of AZs in a region

Fn::Join - construct complex strings; concatenate strings

Fn::Select - value selection from list (0, 1) 

Ref - default value of resource

Conditional Functions - Fn::And, Fn::Equals, Fn::If, Fn::Not, Fn::Or

## Stack Creation & Depends On:

============================


1.Template Upload/S3 Template reference

2.Template Syntax Check

3.Stack Name & verification & ingestion

4.CFN template processing & stack creation

5.Resource ordering

6.Resource creation

7.Output Generation

8.Stack completion or Rollback

DependsOn - influences automatic dependency checking of CFN; directs CFN how to handle dependencies;

## CFN Resource Deletion Policies:

===============================

A policy/setting which is associated with each resource in a template; A way to control what happens to each resource when a stack is deleted.

Policy value - Delete (Default), Retain, Snapshot

Delete - Useful for testing environment, CI/CD/QA workflows, Presales, Short Lifecycle/Immutable env.

Retain - live beyond lifcycle of stack; Windows Server Platform (AD), Servers with state, SQL, Exchange, File Servers, Non immutable architectures.

Snapshot - restricted policy type only available for EBS volumes; takes snapshot before deleting for recovering data.

## CFN Stack updates:

=================

stack policy is checked, updates can be prevented; absence of stack policy allow all updates; stack policy cannot be deleted once applied.

Once stack policy applied ALL objects are protected, Update is denied; to remove default DENY, explicit allow is required; can be applied, to a single resource(id)/Wild card/NotResource; Has Principal and Action; Condition element (resource type) can also be used.

Stack updates: 4 Types - Update with No Interrupion, Some Interruption, Replacement, Delete
Changing Availability Zone, Changing Image ID, Updating DyanamoDB table name, Autoscaling LC - Replacement

EBS Optimized, Changing Instance type - Some Interruptions

Updating DynamoDB provisioned throughput - No Interruption

## CFN Nesting:

============

to allow huge set of infra to be split over multiple templates, 460k template size limit for S3, 200 resource limit per template,
100 mapping, 60 parameters, 60 outputs limit per stack/template, nesting can overcome these limits.
resource type - AWS::CloudFormation::Stack

## CFN Creation Policies, Wait Conditions & Wait Condition Handlers:

=================================================================

Influence when a resource is marked as competed - delaying until its actually ready.

Creation Policies - only be used on EC2 instance and ASG. Creation Policy Definition & Signal Configuration

WC & WCH can be used in complex scenarios involving complex interaction between resources; two related components;

WCH is a CFN resource with no properties, but it generated signed URL which can be used to communicate SUCCESS or FAILURE.

WC - 4 components 1) they DependOn the resources waiting on 2) a Handle property reference 3) response timeout 4) count (default 1)

WCH - Signed URL

## CFN Custom Resources:

=====================

resource type within CFN that is backed by SNS or Lambda; Custom::ResourceName(SNSTopic or Lambda Function); ServiceToken - arn for SNS topic

Use the presigned URL; provide a response payload; Important elements: status, stack id & request id, physical & logical resource id, data

Use cases: stack linked to on-premise resource creation, stack linked to advanced logic - resource discovery, stack deletion linke to 
advanced tidy operations - backup/montioring deactivation, stacks linked to on-premise CMS, web stack creation - linked to monitoring/penetration,testing system, stack creation/deletion updates a lambda based backup solution - EBS snapshotting, Stack deletion spawns account wide prunning for orphaned EBS volumes.

## OpsWorks Primer:

================

AWS Impelementation of CHEF configuration management & automation system; allows to provision infra, but abstracts some details.

Between CFN (overhead(JSON) and configurability) and Elastic Beanstalk (provides more power & customization).

####OpsWorks Structure: 

OpsWorks Stack - collection of resources for specific purpose

OpsWorks Layer - set of shared functionailty applied to a group of components

OpsWorks Instance - actaul units of compute; inherits some from stack/layer

OpsWorks Application - app deployed into one or more instances

OpsWorks Under the Hood: 

OpsWorks Agent - CHEF, responsible for configuration/reconfiguration of machines as per recipes 

OpsWorks Automation Engine - responsible for creating, deleting, updating EC2 instances and other services; handle load balancing, Auto
scaling, Auto healing, Lifecycle events

Recipes & Cookbooks - Desired state engines; Recipes tell OpsWorks WHAT you want the end result to be; Cookbooks contain recipes and all
associated data to support them.

## OpsWorks Stacks & Layers

=========================

Creating Stack - Stack Name, region cannot be changed, VPC (instances need internet access to communicate with OpsWorks Orchestration Engine), Subnet can be changed, OS cannot be changed (windows/linux), SSH keys, Custom Cookbooks for Git, Advanced options (not changed mostly, can be changed later)

Stack Options:
Resources tab - allows registration of exisiting resource with stack(EIPs, Volumes, RDS)
Layers - logical group of instances share common config elements; Ex: General settings, Auto healing enabled switch; Recipes tab can add recipes,Network - allows ELB, associate EIPs; EBS Voumes - allow EBS optimized instances; Security - allows to select instance profiles/SGs

Layer Types: OpsWorks Layer, ECS, RDS
AN RDS INSTANCE CAN ONLY BE ASSOCIATED WITH ONE OPSWORKS STACK.
A STACK CLONE OPERATION DOESNT COPY AN EXISTING RDS INSTANCE.

## OpsWorks Lifecycle Events

==========================

Events can run manually using events stack run command functionality; When event occurs it runs set of recipes assigned to that event.
Each layer has own recipes for that event. 

SETUP - Occurs when an instance has finished booting

CONFIGURE - Occurs when instance enters/leaves online, associate/disassociate EIP, attach/detach LB to a layer 

DEPLOY - Occurs when run the deploy command on an instance

UNDEPLOY - Occurs when delete an application or run Undeploy command

SHUTDOWN - Occurs when an instance is shutdown, but before its terminated; allows cleanup by running recipes

## OpsWorks Instances

==================

Instances can be added to the layer or the stack instances menu; OpsWorks Instance Type - 24/7 (provisioned manually, started/stopped
by admin), Time-based (intially provisioned and configured to power on/off at certains times during the day), Load-based (intially 
provisioned and configured to power on/off based on configurable criteria; per layer scaling config)

#### OpsWorks Applications

=====================

Applications - object which represent app and its meta data; App name, Doc root, Data Source(RDS/None), App source (Git/HTTP/S3),Env variables, Domain names, SSL enabled & settings

Deploying an Application - 

1.Executed deployment recipes on instances targeted by the cmd.

2.Application id passed to the cmd.

3.App parameters passed into Chef Env within Databags(JSON objects contain data)

4.Deploy recipe access the app source info and pulls app payload onto the instance

5. 5 versions of application maintained; current + previous 4 deployments

#### OpsWorks Create Deployment cmd

===============================

create-deployment: creates app deployments; allows stack level commands to be executed against stack
Syntax: aws opsworks --region us-east-1 create-deployment

--stack-id, --app-id, --instance-ids, --comment, --custom-json, generate-cli-skeleton, cli-input-json, --command (install_dependencies,
update_custom_cookbooks, execute_recipes, configure, setup, deploy, rollback, start, stop, restart & undeploy)

#### Works Databags & berkself

=============================

Berkself - system which address one of Chef's initial shortcoming; OpsWorks < 11.10 version allows only one custom cookbook repo;
11.10 and newer allow to install cookbook from multiple repo. Enable Berkshelf by selecting Use Custom Chef Cookbooks at stack level, 
create berksfile at the root folder of custom repo.
Berksfile sample -
source "https://supermarket.chef.io"
cookbook 'apt'
cookbook 'bleh', git: 'git://somewhere/bleh.git'
Databags - allow access to contextual info within recipes; Global JSON objects assessible from within CHEF framework; 
multiple databags including STACK, LAYER, APP, INSTANCE; Data assessed via Chef data_bag_item & search methods within compute assets; 
updated vi CUSTOM_JSON dialogue

#### OpsWorks Autohealing

====================

Each OpsWorks instance has agent installd; Instances perform ongoing heartbeat style healthchecks with OpsWorks Orchestartion engine;

If heartbeat fails, OpsWorks treat instance as unhealthy and perform auto-heal; 

Auto-Heal workflow - OpsWorks loose communication with engine for more than 5 min, marks as unhealthy.

EBS backed - Online -> stopping -> stopped -> requested -> pending -> booting -> online

Instance Store backed - Online -> Shuting down -> requested -> pending -> booting -> online

Auto-healing wont recover serious instance corruption (start-failed error), requires manual intervention.

Auto-healing wont upgrade os of instances

Auto-heal isn't a performance response, its a failure response.

#### EB primer

===========

allow to deploy, monitor and scale service quickly; PaaS; 

Component Structure: 

Applications - High level structure

Environments - single instance or scalable; web server env or worker env; urls can be swapped (Blue/Green deployment)

Application versions - App bundled in zip file; can be deployed one or more env.

Type of App Env - Web server Env (apps interact with user/service, generate output), Worker Env(process output generated by other

env and act upon); these web server and worker env should be uncoupled or loosely coupled

Application Tier, Network Tier and Data Tier (Create DB outside Elastic Beanstalk env)

Use Elastic Beanstalk for non supported platform - via Docker

#### Extending Beanstalk using Extensions

====================================

.ebextensions is config folder within Elastic Beanstalk app source bundle; allows granular config of beanstalk env & customization of resources. Files are YAML formatted end with .config extension; .config file sections: option_settings(allows Global Config), resources, packages, sources, files, users, groups, commands, container_commands, service (allows customization of EC2 - similar to AWS::CloudFormation::Init)

Leader Instance - EC2 instance within a Load balancing, Autoscaling Env, choosen to be leader/master; applied only during env creation;
once env is established, all nodes are equal; leader_only can be used with in container_commands section of .config file to ensure command run once on leader node; 


#### Docker in Elastic Beanstalk

===========================

Use Elastic Beanstlk to host and run docker containers.
Docker+EB Components: 

App Source bundle: App source, Dependencies, Scripts, .ebextenstion

Dockerfile: contains instructions on building the image from source image

Dockerrun.aws.json file - AWS EB specific file and isn't involved in traditional docker process; defines how to deploy an existing registry stored container as an ElasticBeanstalk application; defines port mapping; contains auth info to a .dockercfg file for private docker registry;

allows EC2 <-> container mapping; points EB at where your application logs; .dockercfg - stored in S3; can be created using docker login registry-url; use this file as base and create auth file in S3 bucket;

#### CloudWatch: 

============

Metric gathering service, Monitoring/alerting service, A graphing service

CMetrics data will be available only for 2 weeks. Anything over 2 weeks should be stored outside.
Namespaces - Ex: EBS, EC2, Check CloudWatch Developer Guide for all Namespaces. Enable detailed monitoring for additional metrics under Namespaces.

Metrics can be aggregated across Autoscaling Services.

Custom Metrics: 

Create an IAM role -> CloudWatch -> EC2 -> CloudWatch Full Access 

Create EC2 instance with CloudWatch Role -> SSH -> Install Python, Pip, Git and AWS CLI -> git clone https://github.com/ACloudGuru/resources; cd resources;  cat time.sh; chmod a+x time.sh -> setup a cron job (crontab -e) */1 * * * * /home/ubuntu/resources/time.sh -> save -> CloudWatch Min granualrity 1 min -> 

Check the CloudWatch for new namespace with Unixtime metrics

CloudWatch Alarms:

Initiate actions (SNS notification) based on parameters specified against metrics available. Alarm period equal to metric frequency (5 min or 1 min).

Alarm can't invoke actions unless state change. Alarms actions must be in same region as alarm. AWS resource wont send data under certain,conditions (ex: EBS not attached to EC2 wont send data to CloudWatch)

Alarm States: OK (metric match thresold) ALARM (metric outside thresold) INSUFFICIENT_DATA (metric not available/not enough)

Limits: 5000 alarms per AWS account, create or update alarm using mon-put-metric-alarm, enable/disable using mon-[enable/disable]-alarm, 

Describe alarm using mon-describe-alarm. Alarm can be created before metrics created.

date +%s -> Get Unix TimeAWS Alarm can be created using CLI aws cloudwatch put-metric-alarm.

Autoscale based on CloudWatch Metrics: CPU/Network/Memory too high/low.

Attach EC2 Full access policy to CloudWatch IAM role.

Using aws autoscaling CLI, create autoscaling launch config and Auto Scaling group.

Use aws autoscaling put-scaling-policy to scale in and out.

Use aws cloudwatch put-metric-alarm to create an alarm for Add capacity using Scale out ARN.

Use aws cloudwatch put-metric-alarm to create an alarm for Remove capacity using Scale in ARN.

sudo apt-get install stress -> to stress CPU

stress --cpu 2 --timeout 600

CloudWatch Logs: Monitor your existing system, application and custom logs in real time.

Send your existing logs to CloudWatch; Create patterns to look for in your logs; Alert based on finding of these patterns.

Free agents for Ubuntu, Amazon Linux, Windows.

Main Purpose
1. Monitor logs from EC2 instances in realtime. (track number of errors in application logs and send notification if exceed thresold)

2. Monitor AWS CloudTrail logged events (API Activity such as manual EC2 instance termination)

3. Archive log data (change log retention setting to automatically delete)

Log events - A record sent to CloudWatch Logs to be stored. Timestamp and Message.

Log Streams - this is a sequence of log events that share the same resource (Apache access logs, automatically deleted after 2 months).

Log Groups - group of log stream that share same retention, monitoring and access control settings.

CMetric Filters - define how a service would extract metric observations from events and turn them into data points for a CloudWatch metric.

Retention Settings - how long events are kept; Expired logs are deleted automatically.

Download the logs agent (cloudwatchlogs-commands.txt) and run the command.

Log Group Retention can be set to 1 day to 10 years.

CloudWatch Log Filters: filter log data pushed to CloudWatch; won't work on existing log data, only work after log filter created, only 
returns
first 50 results. Metric contains 1. Filter Pattern 2. Metric Name 3. Metric NameSpace 4. Metric value

Modify rsyslog (/etc/rsyslog.d/50-default.conf) and remove auth on line number 9, sudo service rsyslog restart

while [ 1 ] ; do ssh xx.xx.xx.xx ; sleep 2 ; done

Real-Time Log processing: Subscription Filters for RTLP - Kinesis Streams, Lambda, Kinesis Firehouse

Use aws kinesis command to create/describe stream, get stream ARN. Update permissions.json with stream and role ARN.

Run aws logs put-subscription-filter to start real time log processing.

Check GitHub: awslabs/cloudwatch-logs-subscription-consumer ==> one click deploy for ELK; 

CloudTrail: 

service records all AWS API calls on account and delivers in a log. Log contain identity of whom made the API call, time of call, source 
ip of call, request parameters, response elements returned by AWS service.

Purpose - to enable security analysis, track changes to your account, provide compliance auditing

Types - All regions and one region (logs delivered to S3 bucket and stored using SSE, logs delivered in 15 mins, new logs files every 5 min)

CloudWatch Events: similar to CloudTrail but faster, central nervous system of AWS. Components 1. Events - Created when State Change or API call or 

Own code generated application level events. 2. Rules - match incoming events and route them for processing, not ordered. 3. Targets - 

where events are processed. i) Lambda functions ii) Kinesis streams iii) SNS topics iv) built-in targets

Go to AWS Lambda from console and copy cloudwatchevents.txt in configure function.

Go to CloudWatch, Create rule -> EC2 state change notification(shutting down), Lambda Function(), Configure rule details -> Create rule 

#### Delegation & Federation:
=======================

Identity Federation - Own IdP - IAM, allow users in other AWS accounts access to resources - Delegation, 
allows users from external IdP - Federation. 

w-2 Types of Federation - 

1. Corporate/Enterprise Identity Federation (LADP,AD,SAML,AWS Directory Service)

2. Web Identity Federation (Amazon, Facebook, Google, Twitter, OpenID Connect) allow app or access to your AWS resources.

ROLES: object which contains 2 policy documents. TRUST Policy (who granted - ARN) ACCESS policy (what entity - Action)

SESSIONS: set of temporary credentials; access and secret key with expiration; obtained by STS;

Service Delegation - EC2 or Lambda auto refresh the session which auto refreshes temp credentials managed on your behalf.

#### Console Multi-Account Access:
==============================

oLogin to Prod account -> authenticate with access keys -> STS Service -> STS:AssumeRole -> Temp Credentials -> Dev Account

Login to Dev account -> Create IAM role -> TrustProductionUsersFullAdmin -> Role for Cross account access -> between AWS accounts ->

rProd account AWS Account ID -> Attach Policy -> Create Role

Login to Prod Account -> Switch Role -> Account Name, Role, Display Name, Color

#### Corporate Identity Federation:
==============================
allow to use existing identity store for AWS access. AWS Directory services, SAML, custom federation proxy. Uses role architecture.

Temp access by STS and access obtained via GetFederationToken or STS:AssumeRole operations.

AssumeRole session min 15 minutes, Max 1 hr, Default 1hr; GetFederationToken min 15 min, Max 36 hrs, Default 12 hrs

allows seperation of responsibilities, minimize admin overhead.

#### Custom Proxy - Console - AssumeRole

------------------------------------

1. Corporate User Browse the Fed Proxy domain.com 

2. Fed Proxy authenticates user to LDAP

3. LDAP get groups from Fed Proxy

4. Fed Proxy sends list roles request

5. STS returns list of roles

6. User will select appropriate role

7. Fed Proxy sends STS:AssumeRole

8. STS returns STS:AssumeRole responses

9. Generate URL and redirect to user

10. User access URL and get console access

#### Custom Proxy - API - GetFederationToken

---------------------------------------

1. Corporate App browse Fed Proxy

2. FedProxy authenticats App to LDAP 

3. Directory sends Entitlements to Fed Proxy

4. Fed Proxy send GetFederationToken to STS

5. STS returns GetFederationToken reponse

6. Session

7. Call APIs

Both use cases needs an IAM user. GetFederationToken does not support MFA.

#### SAML - Console - AssumeRoleWithSAML

-----------------------------------

1. Corporate user access AD FS

2. AD FS authenticates user against Directory

3. SAML Token contains membership generated

4. Sigin in with SAML Token to AWS Sign-in Endpoint

5. AssumeRoleWithSAML send to STS

6. STS returns Creds

7. AWS Sign-in endpoint returns Console URL

8. Corporate user Redirected to AWS Console

No need to maintain dedicated Fed proxy for application, proxy doesnt need any IAM permission.

#### Web Identity Federation

=======================

allows trusted third party to authneticate users; avoids to create and manage users; avoid users having multiple id's; simplifies access 
control via roles.

#### Standard Web Identity Federation

================================

1. Mobile user autheticates with Web Identity provider

2. WIP authenticates identity

3. Mobile user AssumeRole with STS via API

4. STS validates with WIP receives success/failure notification

5. success response verifies Role Trust policy

6. STS provide Temp access credentials to Mobile User

7. Mobile User use Temp credentials to use service


Cognito: identity management and sync service; 2 product streams; cognito identity pool - collection of identities; allows 2 roles to
be associated one for authenticated user other for unauthenticated users

Cognito Unauthenticated flow: 

1. Mobile user create unauthenticated identity

2. Coginto returns OpenID Token

3. Mobile User AssumeRole with STS

4. STS validates with Cognito

5. STS returns AWS Guest credentials

6. Mobile users Write data

Cognito can orchestrate generation of unauthenticated identity, merge unauth identity into auth identity, merge multiple entities into one object

Cognito Authenticated flow: Classic or Basic / Enhanced

First step to Login to Web Identity provider, rest are same as unauthenticated flow.

Enhanced flow, communicate all time with Cognito. pre-cognito auth flow, unautheticated or guest flow, simple cognito flow, enhanced cognito flow. why and when to use web id provider - when you need to publish app or service to thousands of users.

#### High Availability and Elasticity:

=================================

Autoscaling - Autoscaling Group (Min, Desired,Max), Launch Configuration (AMI, Instance type, Keypair etc), Scaling Plan (Conditions/Dynamic, Time/Scheduled) Better fault tolerence, Better availabilty, Better cost management
Limits: 100 LC, 20 ASG, 50 Lifecycle hooks, 50 LB per ASG (10 attached), 20 Step adjustments per scaling policy

Autoscaling Lifecycle: life of EC2 attached to ASG; ASG -> Scaling out -> Instance pending state -> EC2_INSTANCE_LAUNCHING lifecycle hook

-> In Service -> Fail health check or Scale in -> Terminate -> EC2_INSTANCE_TERMINATING lifecycle hook -> Terminated 
              -> Troubleshoot instances -> Enter StandBy (Manually) -> Pending -> In Service    
              -> Detach Instances (Manually) -> Detachhing -> Detached -> Standalone Instance -> Can be attached to ASG.

Autoscaling Lifecycle Hooks: allow custom actions as AS launches or terminate instances (Ex: install software or copy logs before termination)              

How it works: 

1. AS responds to a scale out event by launching instance 

2. AS put the instance into Pending:Wait state 

3. As sends message to  notification,target as defined for the hook along with information and a token 

4. waits until you tell it to continue or timeout ends 

5. perform custom action, install software etc. 

6. By default instance wait for an hour and change state Pending:Proceed, then In Service state.

Change the timeout hook via heartbeat-parameter using AWS CLI; can call complete-lifecycle-action to tell ASG to proceed;
call record-lifecycle-action-heartbeat to add more time to timeout; 48 hrs max in wait state

Cooldowns: ensure ASG doesnt launch/terminate more instances; starts when instance enters In Service state; 

ABANDON - cause AS to terminate and launch new instance

CONTINUE - put instance into service

can use SPOT instances, but cannot prevent terminate when the SPOT price go up.

Launch Configuration Deep Dive: Template used by AutoScaling; must specify LC; only one lc per ASG at a time; can't modify LC after created.

LC created from Scratch or running EC2 instance (can't detect block devices created after LC applied); can't use same LC for SPOT and ON DEMAND.

AutoScaling Group Deep Dive: collection of instances (LU); allows improved scaling and management of instances;

creating ASG from EC2 - Tags will not copied, block device mapping from AMI not from instance, LB name is not copied.

AutoScaling Group Self-Healing: create low cost, self-healing, immutable infrastructure; Good for Bastion/OpenVPN etc.


sudo pip install beeswithmachineguns paramiko -> For Testing Autoscaling

Setup access keys by using -> vi .boto; chmod 600 .boto; ssh-keygen -t rsa -f bees; mv bees ~/.ssh/bees.pem; cat bees.pub and import 

keypair in AWS console; bees up -s 10 -g bees -k bees; creates 10 instances; bees attack -n 1000 -c 250 -u http://elbdns; attacks LB, 

create CloudWatch Alarms, Scale out happens; bees down

Amazon RDS:

RDS provides: Provision Infra, Install DB software, Automatic backup, Patch DB software, sycnronous data replication, Automatic failover 

You provide: Settings, Schema, Performance tuning

RDS can be vertically scaled, Change instance type, Multi-Az double the price, Reserved instances make it cheaper, Storage 5 GB to 6TB can  be scaled live(except MS SQL server)

Horizontal Scaling - via Read Replicas, Master/Slave, Great for High Read/Write ratios, one master, replicate write to slave, upto 5 Read Replicas can create replica of a replica, Shrading - split tables into multiple DB.
GP2 (3 IOPS/GB, upto 3000 IOPS) or IOPS storage (Specify IOPS at creation, Fast, Predictable, consistent, suitable for I/O intensive, DB workloads)

Amazon Aurora:

Developed by Amazon. Fast, Reliable, Simple, Cost effective, 5x throughput of MySQL on same hardware, compatible with MySQL 5.6, storage 
is fault tolernet and self healing, detects crashes and restarts, no crash recovery or cache rebuilding, automatic failover to one of up
to 15 RR, storage Auto-scaling from 10 GB to 64TB.

Backups - Automatic, continuous, incremental backups, point-in-time restore with in a second, 35 days retention, stored in S3, No impact on DB performance.

Snapshots - User initiated stored in S3, Incremental

Database failure - Maintain 6 copies in 3 AZs, Recovery in healthy AZ, PIT/Snapshot restore

Fault Tolerence - Data divided into 10GB across many disks, transparently handles data loss, Self-healing

Replicas - Amazon Aurora Replica (upto 15, no performance impact) and MySQL read replicas (upto 5, high performance impact)

Security - Must be created in VPC, SSL(AES-256) secure data in transit, can encrypt db with KMS, Encrypted storage/backups

CANT ENCRYPT UNENCRYPTED DB, MUST CREATE NEW WITH ENCRYPTION ENABLED.

DynamoDB primer:

Fully managed, NoSQL database service, Predictable fully manageable performance with seamless scalability, Fully resilient and HA,
performance scales in a linear way, fully integrated with IAM, suitable for mobile and web applications.

Collection of Tables, tables highest level structure within DB, WCU number of 1KB blocks per second, RCU number of 4KB blocks per second.

eventually consistent (less cost/RCU)/immediate consistent

Hash Key/Partition Key

Range Key/Sort Key

Partitions: 

Underlying storage and processing nodes of DynamoDB. Initially one table -> one partition, one partition can store 10 GB, handles 3000 RCU and 1000 WCU, data distributed based on Hash/Partition Key, can scale indefinitely, no decrease in performance,
allocated WCU and RCU is split between partitions.

GSI/LSI:

DynamoDB offers 2 data retrieval operations, SCAN (scan entire table) and QUERY (select single/multiple item by partition key value)

Index allows efficient queries

Global Secondary Index - can be created anytime, can have alternative Partition & Sort Key, RCU and WCU are defined on GSI.

GSI only support evetually consistent reads.

Local Secondary Index - can only be created at the time of table creation; contains Partition, Sort key and New Sort Key, Projected values

LSI storage concerns - Beware of ItemCollections, ItemCollections max size is 10 GB, ItemCollectionSizeLimitExceededException - Answer

LSI and capacity exceeded 10GB

Streams & Replications: 

Streams: ordered record of updates to a DynamoDB table, If stream enabled records table and stores for 24 hours. 

AWS guarantee no duplication and real time. can be configured with 4 views; 

KEYS_ONLY (only key attributes are written to the stream)

NEW_IMAGE (entire item POST update)

OLD_IMAGE (entire item PRE update)

NEW_AND_OLD_IMAGES (PRE and POST operation state)

Use cases of Stream: Replication, Triggers, Games or large distributed app with user worldwide, DR, Lambda function triggered when items
are added perform analytics etc

Replication: not built in DynamoDB. Create or select table to be replicated, apply CFN stack and wait, get the location from URL of CFN output

test a simple cross region replication

Use SQS as Management write buffer. 

Increase in RCU is dangerous.

How to structure data based on key-space and temporal load/heat prefix/suffix key additions to improve keyspace load leveling
Buffering read/writes with SQS and Caching

SQS:

Reliable, scalable, hosted queue service for sending, storing and retrieving messages. Queue act as a buffer between sender and receiver.

256kb message, >256kb managed using SQS extended client library which uses S3, deliver message atleast once, not FIFO, can be create in
any region, retained for 14 days, can be sent and read simultaneously, long polling reduces frequent polling (wait 20 secs if queue empty)

First 1 million request free, $0.50 next 1 million + data transfer charges, SQS queues can be scaled.
Amazon SQS Architectures - Priority (2 queues High/Low), Fanout (SNS topic/multiple SQS queues for parallel processing)


Kinesis:

3 separate service, 

Streams (collect & process large streams of data in real time), 

Analytics(run standard SQL queries against streaming data), 

Firehouse(load streaming data into AWS from many sources into S3 and Redshift)
Kinesis Streams vs SQS - Kinesis for data processed at the same time or within 24 hrs by different consumers, data can be reused with in 24 hrs

24 hrs retention

SQS can write multiple queue using fanout, data cannot be reused, 14 days retention


Operations:

API/CLI Hints, Tips, Cheats, Instance types, EBS performance, Snapshots vs AMI, Tags

Instance Types: 

PV (Default) & HVM (Slow)

HVM supports wide selection of instance types/sizes.

Why use older versions of Instance? Answer is very attractive spot pricing

Certain Feature require HVM like enhanced networking

Instance family - T M C R G I D(48TB HDD)

Instance generation - 1, 2, 3, 4

Instance size - Micro, Small, Medium, Large etc

Instance Features:

1.EBS Optimization - dedicated bandwidth, consistency, higher throughput

2.Enhanced Networking - AWS supported SR-IOV, less jitter, higher throughput (traffic bypass hypervisor)

3.Instance Store Volume - No resiliance, high throughput, high IO (Temporary Block level storage - storing Buffers, cache etc)

4.GPU Availability - Media conversion, Graphical or Scientific Compute - Genomics (G family instances)

EBS Performance: 

CAPACITY(GB), THROUGHPUT(MB read/write), BLOCKSIZE(KB read/write), IOPS, LATENCY (delay between read & write)

Magnetic volume (HDD) cold and near archiveal workloads, not suitable for Prod.

GP2 - Base performance of 3 IOPS per GB, burstable up to 3000, up to 160 MB/s throughput, max 10000 IOPS

IO2 - 30 IOPS per GB, Max throughput 320 MB/s, Max IOPS 20000

Storage Performance - Depends on EC2 instance type, IO profile, Network speed, EBS volume

IOPS vs Throughput

RAID - combine multiple EBS volume to overcome single volume limits (use 128 or 256KB for better performace)

GP2 Busrt pool - create small volumes, monitor peak/normal usage over time and use GP2 rather than IO2.

Pre-warming volume is no longer required for new volumes.

Volume created from snapshots are lazy restored from S3 - force a full read of volume to force a restore

If using RAID 0 or LVM striped the Quiesce IO, freeze file system and perform snapshots. Snapshot are incremental, improve RTO and RPO by snaping often (less time and cost)

API/CLI Cheat sheet:

Autoscaling - enter-standby, exit-standby (move instances with in ASG in to/out standby state), create-launch-configuration,  delete-launch-configuration, update-auto-scaling-group, put-lifecycle-hook, put-scaling-policy

CloudWatch - put-metric-data, put-metric-alarm, disable-alarm-actions, enable-alarm-actions, set-alarm-state, list-metrics, get-metric-statistic

DynamoDB- get-item (eventually consistent), batch-get-item (reponse size 16MB & max 100 items), query & scan, put-item, update-item,

delete-item, batch-writitem (multiple items in single request), create-table, update-table

EC2 - run instances (Create instance), stop-instances (stops EBS backed instance), start-instance, terminate-instance, describe-instance,

wait (wait until creation of snapshot), create-image (creates EBS backed AMI from stopped instance), create-snapshots, copy-image, copy-snapshot, create-volume, decribe-tags (list tag pairs ex:backup and prune EBS volume)

S3 & S3 API - s3, mb, rb, mv, rm, sync, website, s3 api, head-object, head-bucket, get/put bucket-versioning, get/put bucket-acl,
put-bucket-notification-configuration

SQS - add-permssion, change-message-visibility(max 12 hrs), set-queue-attribute (changes can take 15 mins), send-message, 
recieve-message (wait-time-sec parameter, non zero enable long polling reduce CPU operations), delete-message

STS - assume-role (return set of temp access credentials/cross account access), assume-role-with-saml, assume-role-with-web-identity,
get-session-token (MFA enabled users)

Creating Snapshots, Pruning and Orphan handling:

Data recovery from snapshot happens in backgroud and data is immediately available; Never loose data from linked snapshot.

Tagging - add custom backup related tags like type, retain until, instanceId

Backup Framework - create snapshots hourly, daily, weekly, monthly.

Lambda_Prune function or EC2 worker instance to look decribe-instance and delete based on expiration.

AWS Data Pipeline is a web service that helps you reliably process and move data between different AWS compute and storage services, 
as well as on-premises data sources, at specified intervals. 

#### AWS Data Pipeline Components:

=============================

- Tasks and Task Runner

- Pipeline

- Pipeline Definition

    -- Data Source

    -- Activities

    -- Schedule

    -- Preconditions

#### Compatible AWS Compute and Storage Services:

===========================================

- EC2

- EMR

- Amazon S3 (Object Storage)

- Amazon RDS (Managed RDBMS)

- Amazon DynamoDB (Managed NoSQL)

- Amazon Redshift (Managed Data Warehouse)

#### AWS Data Pipeline Use cases:

=============================

- Extract, Transform, Load (ETL) Operations

- Database backups

- Amazon EMR Jobs

- Importing Data

####  AWS Data Pipeline Benefits:

=============================

- Reliable

- Easy to use

- Flexible

- Scalable





http://ozaws.com

https://read.acloud.guru

https://serverlesscode.com

https://paulwakeford.info

http://blog.aws.amazon.com


Write a CFN template to deploy a HA wordpress instance

Write a CFN template to deploy a PHP website, inside an ASG, reading from DynamoDB, then deploy a HTTP load-testing application, watch
and manipulate (CLI and UI) the auto scaling.

Write a small Lambda function, use it as a backing for a custom resource in CFN template.

Make changes to a CFN template, update template - become familiar with impacts, replace, update, interrupt

Download EB example app, make changes, create Dev and Prod EB env, make changes, observer updates

Deploy 2 instances, with appropriate roles, bootstrap the cloud watch logs agent and configure detailed log ingestion into CloudWatch.
