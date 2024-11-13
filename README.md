# Amazon Web Services in Action, Third Edition

https://learning.oreilly.com/library/view/amazon-web-services/9781633439160/OEBPS/Text/fm.htm
# AWS - Foundational Knowledge
Date: 2024-10-28
Tags: #foundational-knowledge #aws #hands-on 

## Overview
I'm starting my hands-on morning practice with this AWS book. I selected it, because like many books I'll use in this practice, it encourages hands-on work. We will be working in AWS. We won't cover everything, but that's ok, it's enough to get our hands active and to get some projects developed. I'm creating a repository for it right now, so I have a basic set of coding tools to work with. 

## Core Concepts
1. Cloud offers businesses flexible ways to use compute and storage that specifically fits their needs
2. AWS Provides multiple ways to provision architecture, but the best is probably Infrastructure as Code
3. CloudFormation is the best language and tool for AWS IaC 
4. To better learn AWS, we should learn CloudFormation

## Notes

#### Chapter 1
Chapter 1 is a general introduction to AWS that succinctly defines the nature of the cloud and the market offerings of AWS. The first use case examined is a Web Shop owner who wants to lift-and-and shift from an on-premises data center to AWS. The authors point out the advantages of employing additional services for resiliency and greater efficiency.

The second use case is Maureen who needs to migrate an enterprise Java application to AWS. The focus on this use case is the security a virtual network provides the organization and the reduction in overhead for overall control.

The third use case is for Alexa, who works on at a startup. The focus here is on a highly available system managed through a load balancer and redundant systems. 

The forth illustration is Nick, who needs to process a large amount of data, but do it on a small budget. He does this by taking advantage of spot instances and the ability to start and stop a virtual machine on command. 

Much of the rest of this chapter is a focus on AWS as a cloud service, breaking down exactly what AWS offers users. I'm not attempting to necessarily capture that type of information in these notes, so I'll skip capture much more from Chapter 1.

#### Here's the chapter summary:
Cloud computing, or the cloud, is a metaphor for supply and consumption of IT resources.

Amazon Web Services (AWS) offers Infrastructure as a Service (IaaS), Platform as a Service (PaaS), and Software as a Service (SaaS).

AWS is a platform of web services for computing, storing, and networking that work well together.

Cost savings aren’t the only benefit of using AWS. You’ll also profit from an innovative and fast-growing platform with flexible capacity, fault-tolerant services, and a worldwide infrastructure.

Almost any use case can be implemented on AWS, whether it’s a widely used web application or a specialized enterprise application with an advanced networking setup.

You can interact with AWS in many different ways. Control the different services by using the web-based user interface, use code to manage AWS programmatically from the command line or SDKs, or use blueprints to set up, modify, or delete your infrastructure on AWS.

Pay-per-use is the pricing model for AWS services. Computing power, storage, and networking services are billed similarly to electricity.

To create an AWS account, all you need is a telephone number and a credit card.

Creating budget alerts allows you to keep track of your AWS bill and get notified whenever you exceed the Free Tier.
#### Chapter 2
Looks like we're starting our day off today 2024-10-29 creating a Wordpress server. This is what the architecture looks like that we're building:
```mermaid
flowchart LR
    Users([Users])
    LB[Load Balancer]
    VM1[Virtual Machine 1]
    VM2[Virtual Machine 2]
    VM3[Virtual Machine 3]
    NFS[Network Filesystem]
    DB[(RDS Database)]
    
    subgraph Clients
        Users
    end
    
    subgraph ELB[Elastic Load Balancing]
        LB
    end
    
    subgraph EC2[Virtual Machines]
        VM1
        VM2
        VM3
    end
    
    Users -->|Incoming requests| LB
    LB -->|Distribute traffic| VM1
    LB -->|Distribute traffic| VM2
    LB -->|Distribute traffic| VM3
    VM1 --> NFS
    VM2 --> NFS
    VM3 --> NFS
    VM1 --> DB
    VM2 --> DB
    VM3 --> DB

    %% Styling
    classDef default fill:#f9f9f9,stroke:#333,stroke-width:2px;
    classDef client fill:#f4f4f4,stroke:#666;
    classDef database fill:#f0f0f0,stroke:#333;
    
    class Users client;
    class DB database;
```
For this we will use the following AWS resources:
- Elastic Load Balancer
- EC2 instance
- RDS for MySQL
- EFS (Elastic File System)
- Security groups

We're going to use CloudFormation for this exercise. The book includes a template. I'm using this template to get started [Cloud Formation Template](https://s3.amazonaws.com/awsinaction-code3/chapter02/template.yaml)

I'm choosing to use the CloudFormation wizard for this one and uploading the yaml file from the link. I'm naming my stack something super original like, Wordpress, and giving it a password that is not password, but something more complex. 

The password format for this is:
- Only letters (A-Z, a-z) and numbers (0-9)
- Length between 8 and 30 characters
- No special characters

Good example: `Wp7aKj9nM4xL5v2q` 

In the book, we basically skip everything else. I added tags, just because I'm trying to get in the habit of always using tags, and I'll be doing a lot of development in our environment and I want the ability to delete anything with the tag {'purpose':'learning'}

It created the Wordpress site. The rest of the chapter reviews the different resources deployed and ends on the with pricing and deleting the resources from the Stack. 

Here's a basic overview of the actual template:

1. **Network Infrastructure**
    - VPC with CIDR block 172.31.0.0/16
    - 2 Subnets across different Availability Zones
    - Internet Gateway
    - Route Tables and Network ACLs
    - Various Security Groups for components
2. **Core Components**
    - Application Load Balancer (ALB)
    - Auto Scaling Group (2-4 instances)
    - RDS MySQL Database
    - EFS (Elastic File System) for shared storage
    - EC2 instances using Amazon Linux 2 AMIs
3. **Security**
    - Separate security groups for:
        - Load Balancer
        - Web Servers
        - Database
        - EFS
    - IAM Role for EC2 instances with SSM capabilities

**Parameters**

- `WordpressAdminPassword`: Admin password (must be 8-30 characters, alphanumeric only)

**Key Features**

1. **High Availability**
    - Multi-AZ deployment
    - Auto Scaling Group with min 2, max 4 instances
    - Load balancer for traffic distribution
    - Shared EFS storage for WordPress files
2. **WordPress Configuration**
    - PHP 7.4
    - WordPress 5.8.2
    - Automated installation using wp-cli
    - Optimized PHP and opcache settings
3. **Database**
    - RDS MySQL instance
    - 5GB storage
    - db.t3.micro instance class
    - Automated backups disabled (retention period: 0)

**Resources Created**

1. Network: VPC, Subnets, IGW, Route Tables, NACLs
2. Compute: Auto Scaling Group, Launch Template
3. Storage: EFS, RDS
4. Load Balancing: Application Load Balancer
5. Security: IAM Roles, Security Groups

**Output**

- WordPress URL (accessible via the Load Balancer DNS name)

**Notable Features**

1. Rolling updates enabled for Auto Scaling Group
2. Health checks configured for instances
3. Automated WordPress installation
4. Shared file system using EFS
5. Systems Manager (SSM) integration for instance management

**Development Notes**

- Uses t3.micro instances for web servers and database (suitable for testing/development)
- Database deletion policy set to "Delete" (should be changed for production)
- Region-specific AMI mappings included for all major AWS regions
#### Chapter Summary
- Creating a cloud infrastructure for WordPress and any other application can be fully automated.
- AWS CloudFormation is a tool provided by AWS for free. It allows you to automate the managing of your cloud infrastructure.
- The infrastructure for a web application like WordPress can be created at any time on demand, without any up-front commitment for how long you’ll use it.
- You pay for your infrastructure based on usage. For example, you pay for a virtual machine per second of usage.
- The infrastructure required to run WordPress consists of several parts, such as virtual machines, load balancers, databases, and network filesystems.
- The whole infrastructure can be deleted with one click. The process is powered by automation.
#### Chapter 3
Today we're working with EC2 instances. We're going to build virtual machines. 
Different EC2 instance types by family:
- T family—Cheap, moderate baseline performance with the ability to burst to higher performance for short periods of time
- M family—General purpose, with a balanced ration of CPU and memory
- C family—Computing optimized, high CPU performance
- R family—Memory optimized, with more memory than CPU power compared to the M family
- X family—Extensive capacity with a focus on memory, up to 1952 GB memory and 128 virtual cores
- D family—Storage optimized, offering huge HDD capacity
- I family—Storage optimized, offering huge SSD capacity
- P, G, and CG family—Accelerated computing based on GPUs (graphics processing units)
- F family—Accelerated computing based on FPGAs (field-programmable gate arrays)
Most of this chapter used the console to set up and build out EC2 instances. I didn't walk through the process of building out an Apache Server, but that was an option. 
Here is a summary of what we covered today:
- When launching a virtual machine on AWS, you chose between a wide variety of operating systems: Amazon Linux, Ubuntu, Windows, and many more.
- Modifying the size of a virtual machine is simple: stop the virtual machine, modify the instance type—which defines the number of CPUs as well as the amount of memory and storage—and start the virtual machine.
- Using logs and metrics can help you to monitor and debug your virtual machine.
- AWS offers data centers all over the world. Starting VMs in Sydney, Australia, works the same as starting a machine in northern Virginia.
- Choose a data center by considering network latency, legal requirements, and costs, as well as available features.
- Allocating and associating a public IP address to your virtual machine gives you the flexibility to replace a VM without changing the public IP address.
- Committing to a certain compute usage for one or three years reduces the cost of virtual machines through buying Savings Plans.
- Use spare capacity at significant discount but with the risk of AWS terminating your virtual machine in case the capacity is needed elsewhere.
#### Chapter 4
The command line, SDKs, and CloudFormation
This is the part of the book I've been looking forward to. We finally start learning the automation tools, which is what I've been wanting to dive deeper into for a while. 
Most of this chapter is related to using the CLI. Here are some of the basics around setting it up
*Linux x86*
```bash
$ curl "https:/ /awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
➥ -o "awscliv2.zip"
$ unzip awscliv2.zip
$ sudo ./aws/install
```
*Linux ARM*
```bash
$ curl "https:/ /awscli.amazonaws.com/awscli-exe-linux-aarch64.zip" \
➥ -o "awscliv2.zip"
$ unzip awscliv2.zip
$ sudo ./aws/install
```
*MacOS and Windows*
Download and install the CLI from [CLI](https://awscli.amazonaws.com/AWSCLIV2.pkg](https://awscli.amazonaws.com/AWSCLIV2.pkg)
*Configure the CLI*
To use the CLI, you'll need to authenticate. This can be done through the command line, but you'll need to create an IAM role to manage permissions.
*Basic information we need to add*
```bash
$ aws configure
 AWS Access Key ID [None]:  AKIAIRUR3YLPOSVD7ZCA   ①
 AWS Secret Access Key [None]: 
➥ SSKIng7jkAKERpcT3YphX4cD87sBYgWVw2enqBj7        ②
 Default region name [None]: us-east-1
 Default output format [None]: json
```
That is not a real secret, but one that was made up.
You can test this by checking on the instances in your region using:
```bash
aws ec2 describe-regions
```
You should get a json object with a list of regions. 
The following will give you a list of running ec2 instances:
```bash
aws ec2 describe-instances --filters "Name=instance-type,Values=t2.micro"
{
  "Reservations": []
}
```
**Getting Help**
- `aws` `help`—Shows all available services
- `aws` `<service>` `help`—Shows all actions available for a certain service
- `aws` `<service>` `<action>` `help`—Shows all options available for the particular service action

**Running a temporary virtual machine**
One of the example scripts will run a virtual machine for us until we want to stop it. This is done through the command line. The IAM role we set up earlier when building out instances is used for this. The script is include in the `cli_scripts/virtualmachine.sh` file.

It solves this use case:
- Creating a virtual machine
- Getting the ID of a virtual machine to connect via the Session Manager
- Terminating the virtual machine if it’s no longer needed

Another useful tool is JMESPath, which allows us to query values.
```bash
$ aws ec2 describe-images --filters \
➥ "Name=name,Values=amzn2-ami-hvm-2.0.202*-x86_64-gp2" \
➥ --query "Images[0].ImageId"
"ami-146e2a7c"
```
This could also be outputted as text, if needed:
```bash
aws ec2 describe-images --filters \
➥ "Name=name,Values=amzn2-ami-hvm-2.0.202*-x86_64-gp2" \
➥ --query "Images[0].ImageId" --output text         ①
ami-146e2a7c
```

#### Working with SDKs
My next step will be to work with `nodecc`, which will require using DevBox on my local machine to install node. 

The goal for today is to get _Node Control Center for AWS_ (nodecc) built and running, I plan to use DevBox for this, so I'm going to do a little DevBox prep first.

Based on our book, it looks like we're safe with whatever version of Node we want, as long as it's greater than `14.*`, so I'll use one of the Node projects using `devbox create --template nodejs-npm`

Ok, so I got everything setup, but it's not launching for me. I'm not too interested in trying to get it working, so I'm skipping this to keep moving forward. I'll check the project in, but probably won't spend time on it. 

#### Infrastructure as Code
In this section of the chapter, we'll work on IaC. 
The following from the book is the anatomy of a Cloud Formation template:
1. _Format version_—The latest template format version is 2010-09-09, and this is currently the only valid value. Specify this version; the default is to use the latest version, which will cause problems if new versions are introduced in the future.
2. _Description_—What is this template about?
3. _Parameters_—Parameters are used to customize a template with values, for example, domain name, customer ID, and database password.
4. _Resources_—A resource is the smallest block you can describe. Examples are a virtual machine, a load balancer, or an Elastic IP address.
5. _Outputs_—An output is comparable to a parameter, but the other way around. An output returns details about a resource created by the template, for example, the public name of an EC2 instance.
Here is a summary of the chapter as we wrap things up here:
- Use the CLI, one of the SDKs, or CloudFormation to automate your infrastructure on AWS.
- Infrastructure as Code describes the approach of programming the creation and modification of your infrastructure, including virtual machines, networking, storage, and more.
- You can use the CLI to automate complex processes in AWS with scripts (Bash and PowerShell).
- You can use SDKs for nine programming languages and platforms to embed AWS into your applications and create applications like nodecc.
- CloudFormation uses a declarative approach in JSON or YAML: you define only the end state of your infrastructure, and CloudFormation figures out how this state can be achieved. The major parts of a CloudFormation template are parameters, resources, and outputs.

_My thoughts on IaC and language selection_
I've so far had experience working with Pulumi, Terraform, Bicep, and CloudFormation. I know there are good reasons to use something like Pulumi and Terraform for the state management, but some of these same types of tools are available for Azure and AWS. Meaning, there are ways to control cloud infrastructure drift that are similar to Pulumi and Terraform. 
As of today 2024-11-08, I'm of the opinion that we shouldn't use Terraform (OpenTofu) or Pulumi, unless the client really wants that as a solution. Otherwise, I think all our IaC should be the native languages  of Bicep and CloudFormation. It's just simpler, and the languages are so well created for their purpose that it doesn't make much sense to me to add another layer of abstract thinking on top of an existing layer of abstract thinking, especially considering that I can't very well write one script and have it execute in both environments. These scripts are tightly coupled to the infrastructure they create, so doesn't it make more sense to use a DSL built for that environment. Anyway, thats my takeaway from today's work. 
#### # Securing your system: IAM, security groups, and VPC
- Who is responsible for security?
- Keeping your software up-to-date
- Controlling access to your AWS account with users and roles
- Keeping your traffic under control with security groups
- Using CloudFormation to create a private network
This first section covers the responsibilities of AWS and the cloud user to security. 
Additionally, the process of keeping EC2 instances updates is covered. A suggestion is to use AWS System Manager for updates.
**Core Features Are:**
- _Agent_—Preinstalled and autostarted on Amazon Linux 2 (also powers the Session Manager).
- _Document_—Think of a document as a script on steroids. We use a prebuild document named `AWS-RunPatchBaseline` to install patches.
- _Run Command_—Executes a document on an EC2 instance.
- _Association_—Sends commands (via Run Command) to EC2 instances on a schedule or during startup (bundled into the capability named State Manager).
- _Maintenance Window_—Sends commands (via Run Command) to EC2 instances on a schedule during a time window.
- _Patch baseline_—Set of rules to approve patches for installation based on classification and severity. Luckily, AWS provides predefined patch baselines for various operating systems including Amazon Linux 2. The predefined patch baseline for Amazon Linux 2 approves all security patches that have a severity level of critical or important and all bug fixes. A seven-day waiting period exists after the release of a patch before approval

**Identity vs. resource policies**

IAM policies come in two types. _Identity policies_ are attached to users, groups, or roles. _Resource policies_ are attached to resources. Very few resource types support resource policies. One common example is the S3 bucket policy attached to S3 buckets.

If a policy contains the property `Principal`, it is a resource policy. The `Principal` defines who is allowed to perform the action. Keep in mind that the principal can be set to public.

[Service Authorization Reference](https://docs.aws.amazon.com/service-authorization/latest/reference/reference_policies_actions-resources-contextkeys.html)

**Types of Policies**
- _Managed policy_—If you want to create identity policies that can be reused in your account, a managed policy is what you’re looking for. There are two types of managed policies:
    - _AWS managed policy_—An identity policy maintained by AWS. There are identity policies that grant admin rights, read-only rights, and so on.
    - _Customer managed_—An identity policy maintained by you. It could be an identity policy that represents the roles in your organization, for example.
- _Inline policy_—An identity policy that belongs to a certain IAM role, user, or group. An inline identity policy can’t exist without the IAM role, user, or group that it belongs to

[An example of create a role and assigning it to EC2](https://s3.amazonaws.com/awsinaction-code3/chapter05/ec2-iam-role.yaml)

**Users for authentication and groups to organize users**
I added this to our source folder for scripts, but here is the code for adding an admin group.
```shell
aws iam create-group --group-name "admin"
aws iam attach-group-policy --group-name "admin" \
➥ --policy-arn "arn:aws:iam::aws:policy/AdministratorAccess"
aws iam create-user --user-name "myuser"
aws iam add-user-to-group --group-name "admin" --user-name "myuser"
aws iam create-login-profile --user-name "myuser" --password '$Password'
```
Enabling MFA for all users:
1. Open the IAM service in the Management Console.
2. Choose Users at the left.
3. Click the myuser user.
4. Select the Security Credentials tab.
5. Click the Manage link near the Assigned MFA Device.
6. The wizard to enable MFA for the IAM user is the same one you used for enabling MFA for the AWS account root user.

**Authenticating AWS resources with roles**

We shouldn't create user IDs for resources or services in AWS, but instead we should assign them roles. These roles will determine what permissions they have. For instance, if an EC2 instance needs to shut itself down, it need permissions to do that. Any interaction with the AWS API requires some type of permission. 

**Controlling Network Traffic**
Most of networking is about controlling the flow of traffic. Where is it allowed to go and what rules are in place to stop or allow traffic. 

**Security Groups to Control Traffic**

A security group consists of a set of rules. Each rule allows network traffic based on the following:
- Direction (inbound or outbound)
- IP protocol (TCP, UDP, ICMP)
- Port
- Source/destination based on IP address, IP address range, or security group (works only within AWS)

**Allowing traffic from a security group**
Probably one of the more interesting ways to manage how traffic is manage is by allow traffic from one security group to another. 
> It is possible to control network traffic based on whether the source or destination belongs to a specific security group. For example, you can say that a MySQL database can be accessed only if the traffic comes from your web servers, or that only your proxy servers are allowed to access the web servers.

**Creating a VPC**

- _Public subnets_—For all resources that need to be reachable from the internet, such as a load balancer of a internet-facing web application
- _Private subnets_—For all resources that should not be reachable from the internet, such as an application server or a database system

_I wanted to add some extra information about Security Groups and NACLs here._
AWS VPC Security Groups and Network Access Control Lists (NACLs) both control network traffic, but they operate at different levels and have some key differences:

1. **Level of Operation**:

• **Security Groups** work at the instance level. They act as a virtual firewall for individual EC2 instances within a VPC.

• **NACLs** work at the subnet level. They control traffic going into and out of entire subnets within a VPC.

2. **Traffic Direction**:

• **Security Groups** are stateful, meaning they automatically allow return traffic for any outgoing request. For example, if an instance sends a request, the response is allowed back automatically.

• **NACLs** are stateless, so both inbound and outbound rules need to be explicitly defined. If you allow inbound traffic, you also need to allow outbound traffic for the return response.

3. **Rules and Scope**:

• **Security Groups** can only have “allow” rules, meaning you can specify what traffic is permitted, but there’s no explicit deny option.

• **NACLs** can have both “allow” and “deny” rules, which means they offer more granular control over what traffic is explicitly permitted or blocked.

In short, **use Security Groups to control instance-specific traffic** and **NACLs for broader subnet-level traffic control** within your AWS network.

> We recommend you start with using security groups to control traffic. If you want to add an extra layer of security, you should use NACLs on top. But doing so is optional, in our opinion.

**Interesting solutions for saving on NAT Gateway costs**
- Moving your EC2 instances from the private subnet to a public subnet allows them to transfer data to the internet without using the NAT gateway. Use firewalls to strictly restrict incoming traffic from the internet.
- If data is transferred over the internet to reach AWS services (such as Amazon S3 and Amazon DynamoDB), use gateway VPC endpoints. These endpoints allow your EC2 instances to communicate with S3 and DynamoDB directly and at no additional charge. Furthermore, most other services are accessible from private subnets via interface VPC endpoints (offered by AWS PrivateLink) with an hourly and bandwidth fee.

**Summary**
- AWS is a shared-responsibility environment in which security can be achieved only if you and AWS work together. You’re responsible for securely configuring your AWS resources and your software running on EC2 instances, whereas AWS protects buildings and host systems.
- Keeping your software up-to-date is key and can be automated.
- The Identity and Access Management (IAM) service provides everything needed for authentication and authorization with the AWS API. Every request you make to the AWS API goes through IAM to check whether the request is allowed. IAM controls who can do what in your AWS account. To protect your AWS account, grant only those permissions that your users and roles need.
- Traffic to or from AWS resources like EC2 instances can be filtered based on protocol, port, and source or destination.
- A VPC is a private network in AWS where you have full control. With VPCs, you can control routing, subnets, NACLs, and gateways to the internet or your company network via a VPN. A NAT gateway enables access to the internet from private subnets.
- You should separate concerns in your network to reduce potential damage, for example, if one of your subnets is hacked. Keep every system in a private subnet that doesn’t need to be accessed from the public internet, to reduce your attackable surface.

#### Automating operational tasks with Lambda

This chapter covers:
- Creating a Lambda function to perform periodic health checks of a website
- Triggering a Lambda function with EventBridge events to automate DevOps tasks
- Searching through your Lambda function’s logs with CloudWatch
- Monitoring Lambda functions with CloudWatch alarms
- Configuring IAM roles so Lambda functions can access other services

> the word serverless is a bit of a misnomer. Whether you use a compute service such as AWS Lambda to execute your code, or interact with an API, there are still servers running in the background. The difference is that these servers are hidden from you. There’s no infrastructure for you to think about and no way to tweak the underlying operating system. Someone else takes care of the nitty-gritty details of infrastructure management, freeing your time for other things.

—Peter Sbarski

Return to here tomorrow: Building a website health check with AWS Lambda

## Relationships to Other Technologies/Concepts
[Mind map or list showing connections to other areas of knowledge]

## Key Takeaways
[Main points to remember about this topic]

## Potential Applications in Solutions Architecture
[Ideas on how this knowledge could be applied in your role]

## Resources for Deeper Dive
[Books, articles, or courses for further study if needed]
- [What's New](https://aws.amazon.com/about-aws/whats-new/)
- [AWS Case Studies](https://aws.amazon.com/solutions/case-studies/)

