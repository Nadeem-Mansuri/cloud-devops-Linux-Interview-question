# Cloud DevOps & Linux Interview Questions

## Table of Contents
- [AWS Questions](#aws-questions)
- [Docker Questions](#docker-questions)
- [Git Questions](#git-questions)
- [Jenkins Questions](#jenkins-questions)
- [Kubernetes Questions](#kubernetes-questions)
- [Terraform Questions](#terraform-questions)
- [Linux Questions](#linux-questions)
- [DevOps General Questions](#devops-general-questions)

---

## AWS Questions

### Networking & VPC
1. What EC2 family instance types have you worked on? Could you please name them?
2. How do you make decisions while choosing instances?
3. How do you ensure high availability while designing a subnet?
4. Is it possible to create multiple subnets in one AZ?
5. Can we have a single subnet spread across multiple AZs?
6. How can you block certain IPs from your subnet?
7. What is the best way to interconnect 10 VPCs?

### Load Balancers & Routing
8. What is path-based routing in Load Balancer?
9. What is host-based routing?
10. How do you implement encryption in your load balancer?
11. What is the difference between encryption in transit and encryption at rest?
12. Is implementing an SSL certificate in LB encryption in transit or at rest?

### Auto Scaling & API Gateway
13. What is the use of lifecycle hooks in Auto Scaling?
14. Why do you need API Gateway? What additional features does API Gateway provide that are not in Load Balancer?

### Route 53
15. What are the different types of records in Route53 (CNAME & Alias)?
16. Write an example of CNAME mapping in example.com
17. When you make DNS changes in Route 53, how much time does propagation take?

### Storage
18. What is the difference between EBS and S3?
19. What are the different kinds of encryption in S3?
20. What is lifecycle policy in S3? Is reverse order possible?
21. What is the significance of S3 lifecycle policy and how do you utilize it?

### Lambda
22. What is Lambda Layer?
23. What is synchronous and asynchronous invocation?
24. What are the limitations of AWS Lambda (like concurrency)?

### IAM & Security
25. What is an IAM role and IAM policy?
26. How many kinds of IAM policies are there?
27. What does "principal" mean in IAM policy?
28. What is identity-based and resource-based policy in IAM?
29. **Cross-Account Access Scenario:** Account A has an EC2 instance and Account B has an S3 bucket. How do you configure it so EC2 can access the S3 bucket in Account B?
30. What is the difference between permission boundary and SCP?
31. Where can you apply policies?
32. What are the types of policies in SCP?
33. Where does SCP affect?

### Security Services
34. Explain WAF, Guardrail, GuardDuty, and CloudFront
35. What security services have you used in AWS?
36. How do you set GuardDuty for an Organization to implement in all AWS accounts?
37. How do you integrate GuardDuty with Security Hub?

### RDS
38. What is Multi-AZ in RDS?
39. What is the use of Multi-AZ and Read Replica in RDS?
40. What is authentication and authorization in RDS?
41. **High Latency in RDS:** How can you resolve high latency in an RDS instance?
42. How do you increase IOPS in RDS?
43. If you change instance type in RDS, is there any downtime?
44. What are the different DR strategies for RDS that AWS recommends?
45. What is the switchover strategy when your writer instance is down in terms of DR?
46. How do you implement failover where replication is configured?
47. Do read replicas help with write performance?

### CloudWatch & CloudTrail
48. What is log group & log stream in CloudWatch?
49. What is the difference between CloudWatch and CloudTrail?
50. What is custom metric in CloudWatch?
51. **Scenario:** Your instance was deleted 1 year ago. You have CloudTrail logs in an S3 bucket. Which AWS tools will you use to analyze that data?

### Organizations & SSO
52. What is the first SCP you can think of applying at the root level?
53. What are the benefits of using AWS Organizations?
54. What is a permission set in SSO?

### Transit Gateway & Direct Connect
55. What is the functionality of Transit Gateway?
56. What is dynamic and static route in Transit Gateway?
57. What is DX Location?
58. How does Direct Connect work?
59. How do you encrypt Direct Connect traffic?
60. Which Direct Connect partner are you collaborating with?
61. What are the types of Direct Connect (Dedicated connection and Hosted connection)?
62. What is LOA in Direct Connect?

### Advanced Topics
63. What is a StackSet in AWS CloudFormation?
64. How do you enable cross-account access to an S3 bucket?
65. **S3 Bucket Policy Scenario:**
    ```
    S3 bucket with VPCE policy: allow all traffic: principalORG: org_id
    No connectivity between system and AWS VPC
    
    Bucket policy:
    allow: s3:*
    resource: [arn:aws::s3::mybucket, arn:aws::s3::mybucket/*]
    condition:
      aws:sourceVpce: vpce-s3endpointid
    
    Q1: Can I view objects when logged into this AWS account?
    Q2: How can I view the objects?
    ```

66. **Cross-Account S3 Access:** Account A has s3://epam/demo.txt. Account B has IAM user mohammad. Configure CLI user to get/list demo.txt. Both accounts are not in same OU. Cannot use secret/access keys.

### Boto3 & Python SDK
67. What is the session in boto3, and why do we create it?
68. What is the difference between boto3 client and resources?

### SNS & SQS
69. For SNS, what endpoints are available to send notifications?
70. How do you configure SNS so it sends notifications in order?
71. What is dead-letter-queue in SQS?

### STS
72. STS has 2 endpoints: global and regional. How do they differ and where do you use them?

---

## Docker Questions

1. What is a multi-stage docker container?
2. What is the difference between ADD and COPY?
3. Your container is writing a log to some path inside the container. How do you view it from your host machine?
4. What are tasks & task definitions in ECS?
5. What kind of Dockerfiles have you created?
6. What is the main difference between CMD and ENTRYPOINT? Can we use both in a single Dockerfile?
7. If you use multiple CMD in a single Dockerfile, what will happen?
8. How many container runtimes are available?
9. What is the main difference between a virtual machine and a container?
10. Create a Dockerfile to deploy nginx web server on port 8080. Write commands to build the image and run it as a container.

---

## Git Questions

### Fundamentals
1. Could you clarify the distinction between monolithic and microservices architectures?
2. How do you manage code repositories when transitioning from monolithic to microservices?
3. What is the difference between `git merge` and `git rebase`?
4. What is `git checkout`?
5. What is the HEAD in Git?
6. How do you ignore changes in a tracked file?
7. Which git workflow are you using in your current project?

### Commit Management
8. How do you change the last commit? Use `git commit --amend`
9. **Scenario:** After making 10 commits, you want the 11th commit merged into the previous commit. What steps would you take?
10. **Scenario:** You want to make another commit with the same commit message as before without creating a new message for each push. How?

### Conflict Resolution
11. You encounter a merge conflict in your branch while another team member is working on the same codebase. How do you handle and resolve this conflict?

---

## Jenkins Questions

### Pipeline Basics
1. What kind of pipelines have you created? What kinds are available in Jenkins?
2. What is the difference between scripted and declarative pipelines?
3. What is a multi-branch pipeline? Why do we use it?
4. How do you integrate different tools (GitHub, Maven, Docker, etc.) with Jenkins?

### Architecture & Setup
5. Have you set up Jenkins architecture? How many Jenkins masters and nodes are in your setup?
6. Which plugins have you used for Jenkins?

### Pipeline Design
7. How do you design your pipeline? What are the steps in the pipeline?
8. Have you worked on any application-based pipelines (like Java)?

---

## Kubernetes Questions

### Services & Networking
1. What kinds of services are available in Kubernetes?
2. What is a headless service in Kubernetes? When do you use it?
3. How is traffic routed to Pods? Please explain.
4. What is the main purpose of services in Kubernetes?
5. How does pod networking work in an EKS cluster?

### Components
6. What are the master components in Kubernetes?
7. How does etcd communicate with API servers?
8. Is kubelet a master component or node component?

### Probes & Health Checks
9. When do we use Liveness probe and Readiness probes?

### Deployment & Scaling
10. What are deployment strategies in Kubernetes? If deployment fails, how do you handle it?
11. Which Kubernetes version are you using?
12. How would you implement zero-downtime deployments in Kubernetes?
13. How do you handle stateful applications in Kubernetes?

### Troubleshooting & Management
14. How would you optimize resource usage in a Kubernetes cluster?
15. How would you secure a Kubernetes cluster?
16. How can you ensure high availability of the etcd cluster?
17. How would you approach capacity planning for a Kubernetes cluster?
18. How do you handle logging and monitoring in a large-scale Kubernetes environment?

### Pod Management
19. How to get the following info about pods:
    - a. List of pods in a cluster
    - b. Description of a specific pod
    - c. Detailed info about pods in a namespace
    - d. Drain a Node

### EKS Specific
20. How do you configure a Kubernetes application to be available outside the cluster?
21. Upgrade process of EKS Cluster and Worker Node with zero downtime
22. How to avoid using NAT during the upgrade of an Amazon EKS cluster?
23. How to upgrade worker nodes in EKS with zero downtime?
24. How to add a new node group in a different AZ to an existing EKS cluster?

### IRSA (IAM Roles for Service Accounts)
25. What is IRSA in Kubernetes (Amazon EKS)?
26. What problems does IRSA solve?
27. How does IRSA work?
28. How to configure OIDC in Kubernetes EKS?

### Monitoring
29. What are the steps to install Datadog agent into worker nodes?

---

## Terraform Questions

### Basics

1. **Why do we use providers in Terraform?**
   - **Answer:**
     - Providers are plugins that interact with cloud platforms, SaaS providers, and APIs
     - They translate Terraform configuration into API calls
     - Manage authentication, API versions, and resource lifecycle
     ```hcl
     terraform {
       required_providers {
         aws = {
           source  = "hashicorp/aws"
           version = "~> 5.0"
         }
       }
     }
     
     provider "aws" {
       region  = "us-east-1"
       profile = "default"
     }
     ```
     **Common providers:** AWS, Azure, GCP, Kubernetes, Docker, GitHub, Datadog

2. **Why do we use provisioners?**
   - **Answer:**
     - Execute scripts/commands on local or remote machines after resource creation
     - Types:
       - **local-exec**: Runs on machine executing Terraform
       - **remote-exec**: Runs on created resource
       - **file**: Copies files to resource
     ```hcl
     resource "aws_instance" "web" {
       ami           = "ami-12345678"
       instance_type = "t2.micro"
       
       provisioner "remote-exec" {
         inline = [
           "sudo apt-get update",
           "sudo apt-get install -y nginx"
         ]
       }
       
       provisioner "local-exec" {
         command = "echo ${self.private_ip} >> private_ips.txt"
       }
     }
     ```
     **Note:** Use as last resort; prefer user_data, cloud-init, or configuration management tools

3. **How does Terraform manage state?**
   - **Answer:**
     - **State file** (`terraform.tfstate`) tracks real infrastructure vs configuration
     - Maps resources to real-world objects
     - Stores metadata and resource dependencies
     
     **Local Backend (default):**
     ```bash
     terraform.tfstate  # Stored locally
     ```
     
     **Remote Backend (recommended):**
     ```hcl
     terraform {
       backend "s3" {
         bucket         = "my-terraform-state"
         key            = "prod/terraform.tfstate"
         region         = "us-east-1"
         encrypt        = true
         dynamodb_table = "terraform-locks"
       }
     }
     ```
     
     **State management:**
     - Locking prevents concurrent modifications (DynamoDB)
     - Versioning for state history (S3 versioning)
     - Encryption for sensitive data
     - Regular backups

4. **What kind of modules have you worked on?**
   - **Answer:** (Example response)
     - **VPC module**: Reusable VPC with subnets, IGW, NAT, route tables
     - **EC2 module**: Standard instance configurations with security groups
     - **RDS module**: Database with backups, Multi-AZ, parameter groups
     - **EKS module**: Complete EKS cluster with node groups
     - **Security module**: Security groups, NACLs, IAM roles/policies
     - **Network module**: Transit Gateway, VPC peering, endpoints
     - Public registry modules: terraform-aws-modules/vpc/aws

### Functions & Logic

5. **Can you explain the scenario-based usage of `for_each` and `count`? How are they different?**
   - **Answer:**
     
     **Comparison:**
     | Feature | count | for_each |
     |---------|-------|----------|
     | Type | Integer | Map or Set |
     | Index | Numeric (0,1,2) | Key-based |
     | Removal Impact | Recreates resources | Stable references |
     | Use Case | Identical resources | Unique identifiers |
     
     **count example:**
     ```hcl
     resource "aws_instance" "server" {
       count         = 3
       ami           = "ami-12345678"
       instance_type = "t2.micro"
       
       tags = {
         Name = "server-${count.index}"
       }
     }
     # Creates: server-0, server-1, server-2
     # Problem: Removing server-1 recreates server-2 as server-1
     ```
     
     **for_each example (preferred):**
     ```hcl
     variable "servers" {
       default = {
         web = "t2.micro"
         app = "t2.small"
         db  = "t2.medium"
       }
     }
     
     resource "aws_instance" "server" {
       for_each      = var.servers
       ami           = "ami-12345678"
       instance_type = each.value
       
       tags = {
         Name = each.key
       }
     }
     # Creates: web, app, db
     # Removing app doesn't affect web or db (stable)
     ```
     
     **When to use:**
     - **count**: Fixed number of identical resources, conditional creation
     - **for_each**: Resources with unique identifiers, stable resource management

6. **What is the difference between locals and variables in Terraform?**
   - **Answer:**
     
     **Comparison:**
     | Feature | Variables | Locals |
     |---------|-----------|--------|
     | Input | External (CLI, tfvars) | Internal computed |
     | Purpose | User configuration | Derived values |
     | Override | Yes (runtime) | No |
     | Validation | Supported | Not supported |
     | Reference | `var.name` | `local.name` |
     
     **Variables:**
     ```hcl
     variable "environment" {
       type        = string
       description = "Environment name"
       default     = "dev"
       
       validation {
         condition     = contains(["dev", "staging", "prod"], var.environment)
         error_message = "Must be dev, staging, or prod"
       }
     }
     
     # Usage
     resource "aws_instance" "web" {
       tags = {
         Environment = var.environment
       }
     }
     ```
     
     **Locals:**
     ```hcl
     locals {
       common_tags = {
         Environment = var.environment
         ManagedBy   = "Terraform"
         Project     = var.project_name
       }
       
       db_name = "${var.project_name}-${var.environment}-db"
       
       # Complex expressions
       subnet_cidrs = [
         for i in range(3) : cidrsubnet(var.vpc_cidr, 8, i)
       ]
     }
     
     # Usage
     resource "aws_db_instance" "main" {
       identifier = local.db_name
       tags       = local.common_tags
     }
     ```

7. **Is there anything you can do with locals but not with variables?**
   - **Answer:** **Yes, locals provide capabilities variables don't:**
     
     **1. Compute values from resources:**
     ```hcl
     locals {
       vpc_cidr_parts = split(".", var.vpc_cidr)
       first_octet    = local.vpc_cidr_parts[0]
     }
     ```
     
     **2. Reference other locals:**
     ```hcl
     locals {
       env       = var.environment
       full_name = "${var.project}-${local.env}"
       db_name   = "${local.full_name}-database"
     }
     ```
     
     **3. Complex expressions and functions:**
     ```hcl
     locals {
       # Filter and transform
       prod_instances = [
         for k, v in var.instances : v
         if v.environment == "production"
       ]
       
       # Conditional logic
       use_multi_az = var.environment == "prod" ? true : false
       
       # Map transformations
       instance_ids = { for k, v in aws_instance.servers : k => v.id }
     }
     ```
     
     **4. Avoid repetition (DRY principle):**
     ```hcl
     locals {
       base_tags = {
         ManagedBy   = "Terraform"
         CostCenter  = "Engineering"
         Compliance  = "PCI-DSS"
       }
     }
     
     resource "aws_instance" "web" {
       tags = merge(local.base_tags, {
         Name = "web-server"
         Role = "webserver"
       })
     }
     ```

### Practical Scenarios

8. **Task:** Write Terraform code to provision an AWS EC2 instance with specified requirements
   - **Answer:**
     ```hcl
     # filepath: main.tf
     terraform {
       required_version = ">= 1.0"
       required_providers {
         aws = {
           source  = "hashicorp/aws"
           version = "~> 5.0"
         }
       }
     }
     
     provider "aws" {
       region = "us-east-1"
     }
     
     # Data source to get latest Ubuntu 20.04 AMI
     data "aws_ami" "ubuntu" {
       most_recent = true
       owners      = ["099720109477"] # Canonical
       
       filter {
         name   = "name"
         values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
       }
       
       filter {
         name   = "virtualization-type"
         values = ["hvm"]
       }
     }
     
     # Security group allowing SSH from anywhere
     resource "aws_security_group" "allow_ssh" {
       name        = "allow_ssh"
       description = "Allow SSH inbound traffic from anywhere"
       
       ingress {
         description = "SSH from anywhere"
         from_port   = 22
         to_port     = 22
         protocol    = "tcp"
         cidr_blocks = ["0.0.0.0/0"]
       }
       
       egress {
         description = "Allow all outbound"
         from_port   = 0
         to_port     = 0
         protocol    = "-1"
         cidr_blocks = ["0.0.0.0/0"]
       }
       
       tags = {
         Name = "allow-ssh-sg"
       }
     }
     
     # EC2 Instance
     resource "aws_instance" "my_instance" {
       ami                    = data.aws_ami.ubuntu.id
       instance_type          = "t2.micro"
       key_name               = "my-key-pair"
       vpc_security_group_ids = [aws_security_group.allow_ssh.id]
       
       tags = {
         Name = "MyInstance"
       }
     }
     
     # Outputs
     output "instance_id" {
       value = aws_instance.my_instance.id
     }
     
     output "instance_public_ip" {
       value = aws_instance.my_instance.public_ip
     }
     
     output "instance_public_dns" {
       value = aws_instance.my_instance.public_dns
     }
     ```
     
     **Commands to apply:**
     ```bash
     terraform init
     terraform plan
     terraform apply
     terraform output
     ```

9. **What are all mandatory parameters to create an EBS volume using Terraform?**
   - **Answer:** Mandatory parameters:
     ```hcl
     resource "aws_ebs_volume" "example" {
       availability_zone = "us-east-1a"  # Required
       size              = 40             # Required (in GiB)
       type              = "gp3"          # Optional but recommended (default: gp2)
       
       # Optional but commonly used
       encrypted         = true
       iops              = 3000           # For io1/io2/gp3
       throughput        = 125            # For gp3 only
       
       tags = {
         Name = "my-ebs-volume"
       }
     }
     ```
     **Three mandatory parameters:**
     1. **availability_zone**: Must specify AZ
     2. **size**: Volume size in GiB
     3. **type**: gp2, gp3, io1, io2, st1, sc1 (optional but recommended)

10. **Dynamic Block Example:** Create security group with rules using dynamic block
    - **Answer:**
      ```hcl
      # filepath: security_group.tf
      locals {
        allowed_cidrs = {
          https = "0.0.0.0/0"
          ssh   = "1.1.1.1/32"
        }
        
        port_config = {
          https = 443
          ssh   = 22
        }
      }
      
      resource "aws_security_group" "dynamic_sg" {
        name        = "dynamic-security-group"
        description = "Security group with dynamic rules"
        
        # Dynamic ingress rules
        dynamic "ingress" {
          for_each = local.allowed_cidrs
          
          content {
            description = "Allow ${ingress.key}"
            from_port   = local.port_config[ingress.key]
            to_port     = local.port_config[ingress.key]
            protocol    = "tcp"
            cidr_blocks = [ingress.value]
          }
        }
        
        egress {
          description = "Allow all outbound"
          from_port   = 0
          to_port     = 0
          protocol    = "-1"
          cidr_blocks = ["0.0.0.0/0"]
        }
        
        tags = {
          Name = "dynamic-sg"
        }
      }
      
      # Alternative approach with list of objects
      locals {
        ingress_rules = [
          {
            description = "HTTPS from anywhere"
            port        = 443
            cidr        = "0.0.0.0/0"
          },
          {
            description = "SSH from specific IP"
            port        = 22
            cidr        = "1.1.1.1/32"
          }
        ]
      }
      
      resource "aws_security_group" "dynamic_sg_v2" {
        name        = "dynamic-security-group-v2"
        description = "Security group with dynamic rules v2"
        
        dynamic "ingress" {
          for_each = local.ingress_rules
          
          content {
            description = ingress.value.description
            from_port   = ingress.value.port
            to_port     = ingress.value.port
            protocol    = "tcp"
            cidr_blocks = [ingress.value.cidr]
          }
        }
        
        egress {
          from_port   = 0
          to_port     = 0
          protocol    = "-1"
          cidr_blocks = ["0.0.0.0/0"]
        }
      }
      ```

11. **What is the difference between `count` and `for_each` in Terraform?**
    - **Answer:** (Covered in detail in question 5 above)
      
      **Key differences:**
      - **count**: Integer-based, uses index (0,1,2), resource recreation on changes
      - **for_each**: Map/Set based, uses keys, stable resource references
      
      **Example showing the problem with count:**
      ```bash
      # Initial: 3 instances (web-0, web-1, web-2)
      # Remove web-1: web-2 becomes web-1 (recreation)
      
      # With for_each: No recreation, stable keys
      ```

### State Management

12. **How do you validate Terraform templates? What system do you have to check your template?**
    - **Answer:**
      
      **Built-in validation commands:**
      ```bash
      # 1. Format check
      terraform fmt -check -recursive
      terraform fmt -diff
      
      # 2. Validation
      terraform validate
      
      # 3. Plan (dry-run)
      terraform plan -out=tfplan
      
      # 4. Show plan in JSON
      terraform show -json tfplan
      ```
      
      **External validation tools:**
      ```bash
      # 1. TFLint - Linter
      tflint --init
      tflint
      
      # 2. Checkov - Security scanner
      checkov -d .
      checkov -f main.tf
      
      # 3. Terraform Compliance - Policy as Code
      terraform-compliance -f tests/ -p tfplan.json
      
      # 4. Terrascan - Security scanner
      terrascan scan -t aws
      
      # 5. tfsec - Security scanner
      tfsec .
      
      # 6. Sentinel - Policy as Code (Terraform Cloud)
      sentinel test
      ```
      
      **CI/CD Pipeline validation:**
      ```yaml
      # .gitlab-ci.yml
      stages:
        - validate
        - plan
        - apply
      
      terraform-validate:
        stage: validate
        script:
          - terraform fmt -check -recursive
          - terraform init
          - terraform validate
          - tflint
          - checkov -d .
      ```
      
      **Pre-commit hooks:**
      ```yaml
      # .pre-commit-config.yaml
      repos:
        - repo: https://github.com/antonbabenko/pre-commit-terraform
          rev: v1.83.0
          hooks:
            - id: terraform_fmt
            - id: terraform_validate
            - id: terraform_tflint
            - id: terraform_checkov
      ```

13. **What about drifting?**
    - **Answer:**
      
      **What is drift?**
      - Difference between Terraform state and actual infrastructure
      - Caused by: Manual changes, external automation, console modifications
      
      **Detect drift:**
      ```bash
      # 1. Terraform refresh (updates state)
      terraform refresh
      
      # 2. Terraform plan (shows differences)
      terraform plan -refresh-only
      
      # 3. Show detailed diff
      terraform plan -detailed-exitcode
      
      # 4. Drift detection automation
      terraform plan -out=plan.tfplan
      terraform show -json plan.tfplan | jq '.resource_changes'
      ```
      
      **Handle drift:**
      ```bash
      # Option 1: Import manual changes
      terraform import aws_instance.example i-1234567890abcdef0
      
      # Option 2: Update Terraform config to match reality
      terraform refresh
      terraform plan
      # Update .tf files to match
      
      # Option 3: Force infrastructure to match config
      terraform apply -auto-approve
      
      # Option 4: Remove from state and re-import
      terraform state rm aws_instance.example
      terraform import aws_instance.example i-1234567890abcdef0
      ```
      
      **Prevent drift:**
      - Use RBAC to restrict console access
      - Enable CloudTrail/audit logging
      - Regular drift detection (scheduled CI/CD)
      - Infrastructure as Code policies
      - Read-only console access
      
      **Automated drift detection:**
      ```bash
      # Script for drift detection
      #!/bin/bash
      terraform plan -detailed-exitcode
      if [ $? -eq 2 ]; then
        echo "Drift detected!"
        # Send alert (Slack, email, etc.)
        exit 1
      fi
      ```
      
      **Tools for drift detection:**
      - **Terraform Cloud**: Built-in drift detection
      - **driftctl**: Specialized drift detection tool
      ```bash
      driftctl scan --from tfstate://terraform.tfstate
      ```
      - **AWS Config**: Track resource configuration changes
      - **Custom scripts**: Scheduled terraform plan

---

## Linux Questions

### System Information

1. **How to check kernel version?**
   - **Answer:**
     ```bash
     # Method 1: uname command
     uname -r              # Kernel release
     uname -rsv            # Release, name, version
     uname -a              # All information
     
     # Output example: 5.15.0-89-generic
     
     # Method 2: Check kernel files
     cat /proc/version
     cat /proc/sys/kernel/osrelease
     
     # Method 3: Distribution specific
     hostnamectl           # System info including kernel
     
     # Method 4: RPM/DEB packages
     rpm -q kernel         # RHEL/CentOS
     dpkg -l | grep linux-image  # Ubuntu/Debian
     ```

2. **What is initramfs and where is it located?**
   - **Answer:**
     - **initramfs** (Initial RAM Filesystem): Temporary root filesystem loaded into memory during boot
     - Contains drivers and scripts needed to mount real root filesystem
     
     **Location:**
     ```bash
     # RHEL/CentOS/Amazon Linux
     /boot/initramfs-$(uname -r).img
     /boot/initrd.img-$(uname -r)
     
     # Ubuntu/Debian
     /boot/initrd.img-$(uname -r)
     
     # List all initramfs files
     ls -lh /boot/init*
     
     # Example
     /boot/initramfs-5.15.0-89-generic.img
     ```
     
     **Purpose:**
     - Load essential drivers (storage, filesystem)
     - Mount root filesystem
     - Transition to actual root filesystem

3. **If initramfs gets corrupted, how do you repair it?**
   - **Answer:**
     
     **Recovery steps:**
     ```bash
     # Step 1: Boot into rescue/recovery mode
     # At GRUB menu, select "Recovery Mode" or "Rescue Mode"
     
     # Step 2: Check current kernel version
     uname -r
     
     # Step 3: Regenerate initramfs
     
     # For RHEL/CentOS/Amazon Linux
     dracut -f /boot/initramfs-$(uname -r).img $(uname -r)
     # Or
     mkinitrd -f /boot/initramfs-$(uname -r).img $(uname -r)
     
     # For Ubuntu/Debian
     update-initramfs -u
     # Or specific kernel
     update-initramfs -c -k $(uname -r)
     
     # Step 4: Verify file was created
     ls -lh /boot/initramfs-$(uname -r).img
     
     # Step 5: Update GRUB
     grub2-mkconfig -o /boot/grub2/grub.cfg  # RHEL/CentOS
     update-grub                              # Ubuntu/Debian
     
     # Step 6: Reboot
     reboot
     ```
     
     **If unable to boot at all:**
     ```bash
     # Boot from Live CD/USB
     # Mount root partition
     mount /dev/sda1 /mnt
     
     # Chroot into system
     chroot /mnt
     
     # Regenerate initramfs
     dracut -f
     
     # Exit and reboot
     exit
     umount /mnt
     reboot
     ```

4. **How to identify if you're working on a physical or virtual server?**
   - **Answer:**
     ```bash
     # Method 1: dmidecode (most reliable)
     sudo dmidecode -t system | grep -i manufacturer
     sudo dmidecode -t system | grep -i product
     
     # Output examples:
     # Physical: Dell Inc., HP, Supermicro
     # Virtual: VMware, Inc., QEMU, Xen, Microsoft Corporation
     
     # Method 2: systemd-detect-virt
     systemd-detect-virt
     # Returns: kvm, vmware, xen, microsoft, oracle, or "none" for physical
     
     # Method 3: Check for hypervisor
     cat /proc/cpuinfo | grep hypervisor
     # If present: Virtual machine
     
     # Method 4: lscpu
     lscpu | grep Hypervisor
     
     # Method 5: virt-what (requires installation)
     sudo virt-what
     
     # Method 6: dmesg
     dmesg | grep -i hypervisor
     dmesg | grep -i virtual
     
     # Method 7: Check specific files
     cat /sys/class/dmi/id/product_name
     cat /sys/class/dmi/id/sys_vendor
     
     # AWS EC2 specific
     ec2-metadata --instance-id  # Only works on EC2
     ```

### Storage Management

5. **How to create an LVM?**
   - **Answer:**
     ```bash
     # Step 1: Identify disks
     lsblk
     fdisk -l
     
     # Step 2: Create Physical Volume (PV)
     sudo pvcreate /dev/sdb
     sudo pvcreate /dev/sdc
     
     # Verify
     sudo pvs
     sudo pvdisplay
     
     # Step 3: Create Volume Group (VG)
     sudo vgcreate my_volume_group /dev/sdb /dev/sdc
     
     # Verify
     sudo vgs
     sudo vgdisplay my_volume_group
     
     # Step 4: Create Logical Volume (LV)
     sudo lvcreate -n my_logical_volume -L 50G my_volume_group
     # Or use percentage
     sudo lvcreate -n my_lv -l 100%FREE my_volume_group
     
     # Verify
     sudo lvs
     sudo lvdisplay
     
     # Step 5: Create filesystem
     sudo mkfs.ext4 /dev/my_volume_group/my_logical_volume
     # Or XFS
     sudo mkfs.xfs /dev/my_volume_group/my_logical_volume
     
     # Step 6: Mount
     sudo mkdir -p /mnt/mylvm
     sudo mount /dev/my_volume_group/my_logical_volume /mnt/mylvm
     
     # Step 7: Make permanent (add to /etc/fstab)
     echo '/dev/my_volume_group/my_logical_volume /mnt/mylvm ext4 defaults 0 0' | sudo tee -a /etc/fstab
     
     # Verify
     df -h /mnt/mylvm
     ```

6. **Commands: `lsblk`, `fdisk -l`**
   - **Answer:**
     ```bash
     # lsblk - List block devices
     lsblk
     lsblk -f          # Show filesystem type
     lsblk -o NAME,SIZE,TYPE,MOUNTPOINT
     
     # Output example:
     # NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
     # sda      8:0    0  100G  0 disk
     # ├─sda1   8:1    0   50G  0 part /
     # └─sda2   8:2    0   50G  0 part /data
     
     # fdisk -l - List disk partitions
     sudo fdisk -l
     sudo fdisk -l /dev/sda
     
     # Interactive mode
     sudo fdisk /dev/sdb
     # Commands: n (new), d (delete), p (print), w (write), q (quit)
     
     # Alternative tools
     parted -l         # GNU parted
     lsscsi            # List SCSI devices
     ```

7. **Disk is not reflecting using lsblk or fdisk command. How to troubleshoot?**
   - **Answer:**
     ```bash
     # Step 1: Rescan SCSI bus
     echo "- - -" | sudo tee /sys/class/scsi_host/host*/scan
     
     # Step 2: Check dmesg for errors
     dmesg | tail -50
     dmesg | grep -i sd
     
     # Step 3: List all SCSI devices
     lsscsi
     ls -l /sys/block/
     
     # Step 4: Check kernel messages
     cat /proc/partitions
     
     # Step 5: Rescan specific device
     echo 1 | sudo tee /sys/block/sda/device/rescan
     
     # Step 6: For multipath devices
     sudo multipath -ll
     sudo multipathd reconfigure
     
     # Step 7: Reload udev rules
     sudo udevadm control --reload-rules
     sudo udevadm trigger
     
     # Step 8: Check hardware
     sudo smartctl -a /dev/sda  # Requires smartmontools
     
     # For AWS EBS volumes
     # Attach volume via console, then:
     aws ec2 describe-volumes --volume-ids vol-xxxxx
     lsblk  # Should now appear
     ```

8. **How to reduce disk size in LVM?**
   - **Answer:**
     ```bash
     # WARNING: Always backup data first! Risk of data loss!
     
     # Step 1: Unmount the filesystem
     sudo umount /mnt/mylvm
     
     # Step 2: Check filesystem (REQUIRED!)
     sudo e2fsck -f /dev/my_vg/my_lv  # For ext4
     sudo xfs_repair /dev/my_vg/my_lv  # For XFS (read note below)
     
     # Step 3: Resize filesystem (must be smaller than new LV size)
     sudo resize2fs /dev/my_vg/my_lv 40G  # For ext4 only
     # Note: XFS cannot be shrunk! See next question.
     
     # Step 4: Reduce logical volume
     sudo lvreduce -L 45G /dev/my_vg/my_lv
     # Add 5G buffer to avoid data loss
     
     # Or reduce by amount
     sudo lvreduce -L -10G /dev/my_vg/my_lv
     
     # Step 5: Remount
     sudo mount /dev/my_vg/my_lv /mnt/mylvm
     
     # Verify
     df -h /mnt/mylvm
     sudo lvs
     ```

9. **RHEL 8 uses XFS. How to reduce XFS filesystem?**
   - **Answer:**
     **XFS CANNOT BE SHRUNK!** This is a fundamental limitation.
     
     **Workarounds:**
     ```bash
     # Option 1: Backup and recreate (recommended)
     # Step 1: Backup data
     sudo tar -czf /backup/data.tar.gz /mnt/mylvm/
     
     # Step 2: Unmount
     sudo umount /mnt/mylvm
     
     # Step 3: Remove LV
     sudo lvremove /dev/my_vg/my_lv
     
     # Step 4: Create smaller LV
     sudo lvcreate -n my_lv -L 40G my_vg
     
     # Step 5: Create new XFS filesystem
     sudo mkfs.xfs /dev/my_vg/my_lv
     
     # Step 6: Mount and restore
     sudo mount /dev/my_vg/my_lv /mnt/mylvm
     sudo tar -xzf /backup/data.tar.gz -C /
     
     # Option 2: Use rsync to migrate
     # Create new smaller LV
     sudo lvcreate -n my_lv_new -L 40G my_vg
     sudo mkfs.xfs /dev/my_vg/my_lv_new
     sudo mount /dev/my_vg/my_lv_new /mnt/temp
     
     # Sync data
     sudo rsync -avxHAX /mnt/mylvm/ /mnt/temp/
     
     # Swap mounts
     sudo umount /mnt/mylvm
     sudo umount /mnt/temp
     sudo lvremove /dev/my_vg/my_lv
     sudo lvrename my_vg my_lv_new my_lv
     sudo mount /dev/my_vg/my_lv /mnt/mylvm
     
     # Alternative: Convert to ext4 (allows shrinking)
     # But involves downtime and migration
     ```

10. **How to extend volume if it is filled up?**
    - **Answer:**
      ```bash
      # Step 1: Check current usage
      df -h
      lsblk
      
      # Step 2: Identify partition
      lsblk
      # Example: /dev/nvme0n1p1
      
      # For AWS EBS volumes:
      # First, extend volume in AWS Console or CLI
      aws ec2 modify-volume --volume-id vol-xxxxx --size 100
      
      # Step 3: Rescan to detect new size
      sudo partprobe /dev/nvme0n1
      # Or
      echo 1 | sudo tee /sys/class/block/nvme0n1/device/rescan
      
      # Step 4: Extend partition (if needed)
      sudo growpart /dev/nvme0n1 1
      # Or use parted/fdisk to extend
      
      # Step 5: Extend filesystem
      
      # For XFS filesystem:
      sudo xfs_growfs -d /
      # Or specific mount point
      sudo xfs_growfs /mnt/data
      
      # For ext4 filesystem:
      sudo resize2fs /dev/nvme0n1p1
      
      # For LVM:
      # Extend physical volume
      sudo pvresize /dev/nvme0n1p1
      
      # Extend logical volume
      sudo lvextend -l +100%FREE /dev/my_vg/my_lv
      # Or specific size
      sudo lvextend -L +20G /dev/my_vg/my_lv
      
      # Extend filesystem
      sudo xfs_growfs /mnt/data        # XFS
      sudo resize2fs /dev/my_vg/my_lv  # ext4
      
      # Step 6: Verify
      df -h
      ```

### NFS & Filesystem

11. **How do you export NFS?**
    - **Answer:**
      ```bash
      # Step 1: Install NFS server
      # RHEL/CentOS
      sudo yum install nfs-utils
      # Ubuntu/Debian
      sudo apt install nfs-kernel-server
      
      # Step 2: Start and enable NFS service
      sudo systemctl start nfs-server
      sudo systemctl enable nfs-server
      
      # Step 3: Create directory to export
      sudo mkdir -p /export/data
      sudo chmod 755 /export/data
      
      # Step 4: Configure exports in /etc/exports
      sudo vi /etc/exports
      
      # Add export entries:
      /export/data 192.168.1.0/24(rw,sync,no_root_squash,no_subtree_check)
      /export/share *(ro,sync,no_subtree_check)
      /export/home 192.168.1.100(rw,sync) 192.168.1.101(ro,sync)
      
      # Export options:
      # rw: Read-write
      # ro: Read-only
      # sync: Synchronous writes
      # async: Asynchronous writes (faster but risky)
      # no_root_squash: Root on client = root on server
      # root_squash: Root on client = nobody on server (default)
      # no_subtree_check: Disable subtree checking
      # all_squash: Map all users to anonymous
      
      # Step 5: Export the filesystem
      sudo exportfs -arv
      # -a: Export all
      # -r: Re-export all
      # -v: Verbose
      
      # Step 6: Verify exports
      sudo exportfs -v
      showmount -e localhost
      
      # Step 7: Configure firewall
      sudo firewall-cmd --permanent --add-service=nfs
      sudo firewall-cmd --permanent --add-service=mountd
      sudo firewall-cmd --permanent --add-service=rpc-bind
      sudo firewall-cmd --reload
      
      # Client-side mount:
      sudo mount -t nfs 192.168.1.10:/export/data /mnt/nfs
      
      # Permanent mount (/etc/fstab):
      192.168.1.10:/export/data /mnt/nfs nfs defaults 0 0
      ```

12. **Filesystem is going to readonly mode. How did you fix it?**
    - **Answer:**
      **Common causes:**
      - Filesystem corruption
      - Disk errors/bad sectors
      - Out of space (inodes or blocks)
      - I/O errors
      - Hardware failure
      
      **Troubleshooting steps:**
      ```bash
      # Step 1: Check dmesg for errors
      dmesg | tail -50
      dmesg | grep -i error
      dmesg | grep -i "read-only"
      
      # Step 2: Check mount status
      mount | grep "ro,"
      
      # Step 3: Check disk space
      df -h
      df -i  # Check inodes (often overlooked!)
      
      # Step 4: Check for disk errors
      sudo smartctl -a /dev/sda
      
      # Step 5: Remount as read-write (temporary)
      sudo mount -o remount,rw /
      sudo mount -o remount,rw /dev/sda1 /
      
      # Step 6: Check filesystem (requires unmount or read-only)
      # Boot into rescue mode or single-user mode
      
      # For ext4
      sudo e2fsck -f -y /dev/sda1
      
      # For XFS
      sudo xfs_repair /dev/sda1
      # If still issues:
      sudo xfs_repair -L /dev/sda1  # Clear log (last resort!)
      
      # Step 7: Check for failed services
      systemctl --failed
      
      # Step 8: Review system logs
      sudo journalctl -xb | grep -i error
      
      # Step 9: If AWS EBS:
      # Check CloudWatch metrics
      # Consider replacing volume from snapshot
      
      # Step 10: Reboot if necessary
      sudo reboot
      
      # Prevention:
      # - Monitor disk health (SMART)
      # - Regular backups
      # - Monitor disk space
      # - Use RAID for redundancy
      ```

13. **Several NFS modes exist in Linux. How to check?**
    - **Answer:**
      ```bash
      # Check NFS version and mount options
      mount | grep nfs
      nfsstat -m  # Mount statistics
      
      # NFS versions:
      # NFSv2: Legacy, rarely used
      # NFSv3: Most common, stateless
      # NFSv4: Modern, stateful, better security
      # NFSv4.1: Parallel NFS (pNFS)
      # NFSv4.2: Latest features
      
      # Check server NFS versions
      cat /proc/fs/nfsd/versions
      rpcinfo -p | grep nfs
      
      # Mount with specific version
      sudo mount -t nfs -o vers=4.2 server:/export /mnt
      
      # Common mount options:
      # hard/soft: Hard (retry forever) vs soft (timeout)
      # timeo: Timeout value
      # retrans: Number of retransmissions
      # rsize/wsize: Read/write buffer size
      # ac/noac: Attribute caching
      # sec: Security flavor (sys, krb5, krb5i, krb5p)
      
      # Check NFS statistics
      nfsstat -s  # Server stats
      nfsstat -c  # Client stats
      nfsstat -r  # RPC stats
      
      # Check active connections
      sudo ss -tan | grep :2049
      sudo netstat -an | grep :2049
      ```

### User Management

14. **How do you set maximum number of files for a specific user?**
    - **Answer:**
      ```bash
      # Edit /etc/security/limits.conf
      sudo vi /etc/security/limits.conf
      
      # Add entries:
      # Format: <domain> <type> <item> <value>
      
      # For specific user
      john soft nofile 4096
      john hard nofile 8192
      
      # For specific group
      @developers soft nofile 4096
      @developers hard nofile 8192
      
      # For all users
      * soft nofile 4096
      * hard nofile 8192
      
      # Limits types:
      # soft: Warning limit, can be increased by user
      # hard: Maximum limit, cannot exceed
      
      # Common limits:
      # nofile: Max number of open files
      # nproc: Max number of processes
      # memlock: Max locked memory
      # core: Core file size
      # stack: Stack size
      # cpu: CPU time (minutes)
      # fsize: Maximum file size
      
      # Apply limits immediately (for new sessions)
      # User needs to logout and login
      
      # Check current limits
      ulimit -a          # All limits
      ulimit -n          # File descriptors (nofile)
      ulimit -u          # Max processes (nproc)
      
      # Set limits temporarily
      ulimit -n 8192     # Current session only
      
      # For systemd services
      # Edit service file:
      sudo systemctl edit myservice
      
      [Service]
      LimitNOFILE=8192
      LimitNPROC=4096
      
      # Check limits for running process
      cat /proc/<PID>/limits
      
      # System-wide limits
      cat /proc/sys/fs/file-max
      sudo sysctl fs.file-max
      
      # Set system-wide limit
      echo "fs.file-max = 2097152" | sudo tee -a /etc/sysctl.conf
      sudo sysctl -p
      ```

15. **User is not able to create a file or directory. What is the issue?**
    - **Answer:**
      **Troubleshooting checklist:**
      
      ```bash
      # 1. Check disk space
      df -h
      df -i  # Check inodes (often overlooked!)
      
      # 2. Check permissions
      ls -ld /target/directory
      # User needs write permission on parent directory
      
      # 3. Check ownership
      ls -l /target/directory
      stat /target/directory
      
      # 4. Check SELinux context (if enabled)
      getenforce
      ls -Z /target/directory
      
      # Fix SELinux context
      sudo chcon -R -t user_home_t /target/directory
      sudo restorecon -R /target/directory
      
      # 5. Check AppArmor (Ubuntu)
      sudo aa-status
      
      # 6. Check user quotas
      quota -u username
      sudo repquota -a
      
      # 7. Check filesystem status
      mount | grep /target
      # Look for "ro" (read-only)
      
      # 8. Check ulimits
      su - username
      ulimit -a
      ulimit -n  # File descriptors
      
      # 9. Check immutable flag
      lsattr /target/directory
      # If shows 'i', remove it:
      sudo chattr -i /target/file
      
      # 10. Check parent directory permissions
      namei -l /target/directory/file
      
      # 11. Check for filesystem errors
      dmesg | grep -i error
      dmesg | grep -i "read-only"
      
      # 12. Check ACLs
      getfacl /target/directory
      
      # 13. Verify user exists and is not locked
      id username
      passwd -S username
      
      # Common fixes:
      
      # Fix permissions
      sudo chmod 755 /target/directory
      sudo chown username:group /target/directory
      
      # Increase quotas
      sudo edquota username
      
      # Free up inodes
      find /target -type f -size 0 -delete
      
      # Remount filesystem
      sudo mount -o remount,rw /target
      ```

### Networking

16. **How to check what ports are listening on a remote host?**
    - **Answer:**
      ```bash
      # Primary tool: nmap
      nmap <remote-host>
      nmap 192.168.1.100
      nmap example.com
      
      # Scan specific ports
      nmap -p 80,443,3306 192.168.1.100
      nmap -p 1-65535 192.168.1.100  # All ports (slow)
      
      # Fast scan (top 100 ports)
      nmap -F 192.168.1.100
      
      # Service version detection
      nmap -sV 192.168.1.100
      
      # OS detection
      nmap -O 192.168.1.100
      
      # Aggressive scan
      nmap -A 192.168.1.100
      
      # TCP SYN scan (stealth)
      sudo nmap -sS 192.168.1.100
      
      # UDP scan
      sudo nmap -sU 192.168.1.100
      
      # Other tools:
      
      # netcat
      nc -zv 192.168.1.100 80
      nc -zv 192.168.1.100 1-1000
      
      # telnet
      telnet 192.168.1.100 80
      
      # curl (for HTTP/HTTPS)
      curl -I http://192.168.1.100
      
      # ncat (nmap version of netcat)
      ncat -zv 192.168.1.100 80-443
      
      # masscan (very fast, requires root)
      sudo masscan -p1-65535 192.168.1.100 --rate=1000
      
      # For specific services:
      # MySQL
      mysql -h 192.168.1.100 -u user -p
      
      # PostgreSQL
      psql -h 192.168.1.100 -U user
      
      # Redis
      redis-cli -h 192.168.1.100 ping
      
      # AWS-specific (from instance)
      # Check security groups via AWS CLI
      aws ec2 describe-security-groups --group-ids sg-xxxxx
      ```

17. **How do you check network bandwidth between 2 Linux servers when copying a file?**
    - **Answer:**
      ```bash
      # Method 1: iperf3 (recommended)
      # Server side (192.168.1.100)
      iperf3 -s
      
      # Client side (192.168.1.101)
      iperf3 -c 192.168.1.100
      iperf3 -c 192.168.1.100 -t 60  # Run for 60 seconds
      iperf3 -c 192.168.1.100 -P 10  # 10 parallel streams
      
      # Method 2: scp with pv (pipe viewer)
      # Install pv
      sudo yum install pv  # RHEL/CentOS
      sudo apt install pv  # Ubuntu
      
      # Copy with progress
      pv largefile.iso | ssh user@192.168.1.100 "cat > /tmp/largefile.iso"
      
      # Method 3: rsync with progress
      rsync -avz --progress file.iso user@192.168.1.100:/tmp/
      
      # Method 4: dd with progress
      dd if=largefile.iso | pv | ssh user@192.168.1.100 "dd of=/tmp/largefile.iso"
      
      # Method 5: Monitor with iftop
      sudo iftop -i eth0
      
      # Method 6: Monitor with nethogs
      sudo nethogs eth0
      
      # Method 7: Monitor with nload
      nload eth0
      
      # Method 8: bwm-ng (bandwidth monitor)
      bwm-ng
      
      # Method 9: speedtest-cli (internet speed)
      speedtest-cli
      
      # Method 10: Monitor with sar
      sar -n DEV 1 10  # Network stats every 1 sec, 10 times
      
      # Method 11: netstat continuous monitoring
      while true; do
        netstat -i
        sleep 1
      done
      
      # Method 12: ip command
      watch -n 1 'ip -s link show eth0'
      ```

18. **How to set permanent route?**
    - **Answer:**
      ```bash
      # Method 1: Using ip route (RHEL 8+/Ubuntu)
      # Temporary
      sudo ip route add 10.0.0.0/24 via 192.168.1.1 dev eth0
      
      # Permanent - RHEL/CentOS 7
      # Edit /etc/sysconfig/network-scripts/route-eth0
      sudo vi /etc/sysconfig/network-scripts/route-eth0
      # Add:
      10.0.0.0/24 via 192.168.1.1 dev eth0
      
      # Method 2: RHEL/CentOS 8+ (NetworkManager)
      sudo nmcli connection modify eth0 +ipv4.routes "10.0.0.0/24 192.168.1.1"
      sudo nmcli connection up eth0
      
      # Method 3: Ubuntu/Debian (netplan)
      sudo vi /etc/netplan/01-netcfg.yaml
      # Add under network interface:
      network:
        version: 2
        ethernets:
          eth0:
            routes:
              - to: 10.0.0.0/24
                via: 192.168.1.1
      
      # Apply
      sudo netplan apply
      
      # Method 4: Ubuntu/Debian (old style)
      sudo vi /etc/network/interfaces
      # Add:
      up route add -net 10.0.0.0/24 gw 192.168.1.1 dev eth0
      
      # Method 5: Using iptables (not recommended for routing)
      # iptables is for firewall rules, not routing
      # But can be used for NAT/masquerading:
      sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
      sudo iptables-save > /etc/sysconfig/iptables
      
      # Method 6: Amazon Linux 2/AWS
      sudo vi /etc/sysconfig/network-scripts/route-eth0
      10.0.0.0/24 via 192.168.1.1 dev eth0
      
      # Verify routes
      ip route show
      route -n
      netstat -rn
      
      # Delete route
      sudo ip route del 10.0.0.0/24
      ```

19. **How to check active routes?**
    - **Answer:**
      ```bash
      # Method 1: ip route (modern)
      ip route show
      ip route list
      ip route get 8.8.8.8  # Show route to specific destination
      
      # Method 2: route command (legacy)
      route -n
      
      # Method 3: netstat
      netstat -rn
      
      # Method 4: Check routing table
      cat /proc/net/route
      
      # Method 5: For IPv6
      ip -6 route show
      route -A inet6
      
      # Method 6: Detailed routing info
      ip route show table all
      
      # Method 7: Check specific interface routes
      ip route show dev eth0
      
      # Method 8: NetworkManager routes
      nmcli device show eth0
      
      # Method 9: Netplan config (Ubuntu)
      cat /etc/netplan/*.yaml
      
      # Method 10: traceroute to see path
      traceroute 8.8.8.8
      tracepath 8.8.8.8
      
      # Method 11: mtr (better than traceroute)
      mtr 8.8.8.8
      
      # Common output columns:
      # Destination: Target network
      # Gateway: Next hop router
      # Genmask: Netmask
      # Flags: U=up, G=gateway, H=host
      # Metric: Route priority
      # Iface: Interface
      ```

### DNS

20. **Explain DNS - port, `/etc/resolv.conf`, nslookup**
    - **Answer:**
      
      **DNS Basics:**
      - **Port**: 53 (UDP for queries, TCP for zone transfers)
      - **Purpose**: Translates domain names to IP addresses
      
      **`/etc/resolv.conf` - DNS resolver configuration:**
      ```bash
      cat /etc/resolv.conf
      
      # Example content:
      nameserver 8.8.8.8          # Google DNS
      nameserver 8.8.4.4          # Google DNS (backup)
      nameserver 1.1.1.1          # Cloudflare DNS
      search example.com          # Default domain to append
      options timeout:2 attempts:3
      
      # Edit (temporary, may be overwritten)
      sudo vi /etc/resolv.conf
      
      # Make permanent (RHEL/CentOS)
      sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0
      DNS1=8.8.8.8
      DNS2=8.8.4.4
      
      # Make permanent (Ubuntu netplan)
      sudo vi /etc/netplan/01-netcfg.yaml
      network:
        version: 2
        ethernets:
          eth0:
            nameservers:
              addresses: [8.8.8.8, 8.8.4.4]
      ```
      
      **nslookup - DNS query tool:**
      ```bash
      # Basic lookup
      nslookup example.com
      
      # Specify DNS server
      nslookup example.com 8.8.8.8
      
      # Reverse lookup
      nslookup 8.8.8.8
      
      # Interactive mode
      nslookup
      > server 8.8.8.8
      > example.com
      > set type=MX
      > example.com
      > exit
      
      # Query types
      nslookup -type=A example.com      # IPv4
      nslookup -type=AAAA example.com   # IPv6
      nslookup -type=MX example.com     # Mail servers
      nslookup -type=NS example.com     # Name servers
      nslookup -type=TXT example.com    # Text records
      nslookup -type=CNAME www.example.com
      ```
      
      **Alternative tools:**
      ```bash
      # dig (more detailed)
      dig example.com
      dig example.com @8.8.8.8
      dig example.com MX
      dig example.com +short
      dig -x 8.8.8.8  # Reverse lookup
      
      # host
      host example.com
      host 8.8.8.8
      
      # getent (uses system resolver)
      getent hosts example.com
      ```

21. **Using nslookup not getting anything. How to troubleshoot?**
    - **Answer:**
      ```bash
      # Step 1: Check /etc/resolv.conf
      cat /etc/resolv.conf
      # Should have valid nameserver entries
      
      # Step 2: Test connectivity to DNS server
      ping 8.8.8.8
      ping -c 4 8.8.8.8
      
      # Step 3: Check if DNS port is open
      nc -zv 8.8.8.8 53
      telnet 8.8.8.8 53
      
      # Step 4: Try different DNS server
      nslookup example.com 8.8.8.8
      nslookup example.com 1.1.1.1
      
      # Step 5: Check firewall rules
      sudo iptables -L -n | grep 53
      sudo firewall-cmd --list-all
      
      # Step 6: Check if systemd-resolved is running (Ubuntu)
      systemctl status systemd-resolved
      sudo systemctl restart systemd-resolved
      
      # Step 7: Test with dig
      dig example.com
      dig example.com @8.8.8.8
      
      # Step 8: Check /etc/nsswitch.conf
      cat /etc/nsswitch.conf
      # Should have: hosts: files dns
      
      # Step 9: Check /etc/hosts
      cat /etc/hosts
      # Make sure no conflicts
      
      # Step 10: Flush DNS cache
      # Ubuntu
      sudo systemd-resolve --flush-caches
      # RHEL/CentOS
      sudo systemctl restart NetworkManager
      
      # Step 11: Check SELinux (if enabled)
      getenforce
      sudo setenforce 0  # Temporary disable
      
      # Step 12: Check routing
      ip route get 8.8.8.8
      traceroute 8.8.8.8
      
      # Step 13: Test raw DNS query
      dig +trace example.com
      
      # Step 14: Check for DNS blocking
      curl -I http://example.com
      
      # Step 15: Verify internet connectivity
      ping 8.8.8.8
      curl -I https://www.google.com
      
      # Common fixes:
      
      # Fix resolv.conf
      echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
      
      # Restart network
      sudo systemctl restart NetworkManager
      
      # For AWS EC2
      # Check VPC DNS settings
      # Check security group rules (allow UDP 53 outbound)
      # Check NACL rules
      ```

### System Logs

22. **How to check kernel messages?**
    - **Answer:**
      ```bash
      # Method 1: dmesg (primary tool)
      dmesg
      dmesg | less
      dmesg | tail -50
      dmesg | grep -i error
      dmesg | grep -i warning
      dmesg | grep -i fail
      
      # With human-readable timestamps
      dmesg -T
      dmesg -T | tail
      
      # Follow new messages (like tail -f)
      dmesg -w
      
      # Filter by facility/level
      dmesg -l err,warn     # Errors and warnings
      dmesg -l emerg,alert,crit,err
      
      # Filter by facility
      dmesg -f kern         # Kernel messages
      dmesg -f daemon       # Daemon messages
      
      # Clear ring buffer (requires root)
      sudo dmesg -C
      
      # Method 2: journalctl (systemd)
      # Boot messages
      journalctl -k
      journalctl -kb        # Current boot
      journalctl -kb -1     # Previous boot
      
      # Kernel messages with priority
      journalctl -k -p err
      
      # Since specific time
      journalctl -k --since "2024-01-01"
      journalctl -k --since "1 hour ago"
      
      # Follow kernel messages
      journalctl -kf
      
      # Method 3: /var/log files
      sudo tail -f /var/log/kern.log      # Ubuntu
      sudo tail -f /var/log/messages      # RHEL/CentOS
      sudo cat /var/log/dmesg             # Boot messages
      
      # Method 4: Check specific hardware
      # USB devices
      dmesg | grep -i usb
      
      # Disk/SATA
      dmesg | grep -i sata
      dmesg | grep -i ata
      
      # Network
      dmesg | grep -i eth
      dmesg | grep -i network
      
      # PCI devices
      dmesg | grep -i pci
      
      # Method 5: System info
      uname -a
      cat /proc/cmdline      # Kernel boot parameters
      cat /proc/version
      
      # Method 6: Hardware errors
      dmesg | grep -i "hardware error"
      dmesg | grep -i "Machine check"
      
      # Method 7: OOM (Out of Memory) killer
      dmesg | grep -i "out of memory"
      dmesg | grep -i "oom"
      
      # Common use cases:
      
      # Check after system hang/crash
      dmesg -T | grep -E "error|fail|warn" | tail -100
      
      # Monitor for hardware issues
      dmesg -w | grep -i error
      
      # Check after driver installation
      dmesg | tail -50
      
      # Find device information
      dmesg | grep -i "device attached"
      ```

### Shell Scripting

23. **Shell command to remotely sync large data from /data1 on server1 to /data2 on server2. Consider main folder has subfolders.**
    - **Answer:**
      ```bash
      # Method 1: rsync (recommended)
      # Basic sync
      rsync -avz /data1/ user@server2:/data2/
      
      # With progress and stats
      rsync -avzP /data1/ user@server2:/data2/
      
      # Comprehensive options
      rsync -avzP --delete --exclude='*.tmp' /data1/ user@server2:/data2/
      
      # Options explained:
      # -a: Archive mode (preserves permissions, timestamps, etc.)
      # -v: Verbose
      # -z: Compress during transfer
      # -P: Progress + partial (resume capability)
      # --delete: Delete files on destination not in source
      # --exclude: Exclude patterns
      
      # Resume interrupted transfer
      rsync -avzP --partial /data1/ user@server2:/data2/
      
      # Limit bandwidth (KB/s)
      rsync -avz --bwlimit=10000 /data1/ user@server2:/data2/
      
      # Dry run (test without changes)
      rsync -avzn /data1/ user@server2:/data2/
      
      # With SSH key
      rsync -avzP -e "ssh -i /path/to/key.pem" /data1/ user@server2:/data2/
      
      # Exclude large files
      rsync -avzP --max-size=100M /data1/ user@server2:/data2/
      
      # Include/Exclude patterns
      rsync -avzP --include='*.txt' --exclude='*' /data1/ user@server2:/data2/
      
      # Method 2: scp (simpler but no resume)
      scp -r /data1/ user@server2:/data2/
      
      # With progress (requires pv)
      tar czf - /data1 | pv | ssh user@server2 "tar xzf - -C /data2"
      
      # Method 3: tar over SSH
      tar czf - /data1 | ssh user@server2 "tar xzf - -C /data2"
      
      # Method 4: Using screen/tmux for long transfers
      screen
      rsync -avzP /data1/ user@server2:/data2/
      # Press Ctrl+A, then D to detach
      # screen -r to reattach
      
      # Method 5: Parallel rsync (for multiple directories)
      #!/bin/bash
      for dir in /data1/*/; do
        rsync -avzP "$dir" user@server2:/data2/ &
      done
      wait
      
      # Method 6: With error handling
      #!/bin/bash
      rsync -avzP /data1/ user@server2:/data2/
      if [ $? -eq 0 ]; then
        echo "Sync completed successfully"
      else
        echo "Sync failed with exit code $?"
        exit 1
      fi
      
      # Method 7: Incremental backup script
      #!/bin/bash
      SOURCE="/data1/"
      DEST="user@server2:/data2/"
      LOG="/var/log/rsync-$(date +%Y%m%d).log"
      
      rsync -avzP \
        --delete \
        --backup \
        --backup-dir=/data2/backup-$(date +%Y%m%d) \
        --log-file="$LOG" \
        "$SOURCE" "$DEST"
      
      # Method 8: Using AWS S3 as intermediate (for AWS)
      # Server 1
      aws s3 sync /data1/ s3://my-bucket/data/
      
      # Server 2
      aws s3 sync s3://my-bucket/data/ /data2/
      
      # Performance tips:
      # 1. Use compression (-z) for slow networks
      # 2. Skip compression for fast networks (remove -z)
      # 3. Use multiple parallel rsync for large datasets
      # 4. Consider using GNU parallel
      # 5. Increase SSH cipher speed: -e "ssh -c aes128-ctr"
      ```

---

## DevOps General Questions

### Architecture & Design
1. What different types of pipelines are you involved with?
2. What is the end-to-end pipeline structure? How does code deploy to production?
3. How do you monitor after moving to production? What tools do you use?
4. How is the infrastructure for CI/CD deployed?
5. How is provisioning done for the infrastructure?

### Terraform & GitOps
6. What is Terragrunt? How does Terragrunt work? Is it applied manually or automatically?
7. How did you validate the Terraform template?

### Multi-Cloud
8. If you have services in AWS and Azure, how do you integrate them? How does your system get authenticated? How do you ensure data transfer?

### CI/CD
9. What is CI (Continuous Integration) and CD (Continuous Delivery/Deployment)?
10. How do you implement CI/CD for Kubernetes?
11. GitLab CI YAML explanation

### Deployment Strategies
12. How would you rollback your deployment? (What went wrong, how did you fix it, and how did rollback happen)
13. What deployment strategies have you used?

### AWS Specific Scenarios
14. What are a couple of use-cases you recently implemented in AWS?
15. How to use SSM for patching? How does the entire pipeline work for patching?
16. How will you design a VPC for a 3-tier architecture?
17. Transit Gateway, VPC peering insights, pre-checks for VPC peering
18. Multi-account connectivity and On-Prem connectivity
19. NACL rule understanding
20. Have you worked on handling multiple accounts, Organizations, SCP, etc.?

### High Availability & DR
21. How do you make your application highly available?
22. DR Strategies in AWS:
    - Backup & Restore: Lowest cost
    - Pilot Light: Moderate cost
    - Warm Standby: Higher cost
    - Multi-Site Active/Active: Highest cost

---

## AWS Architecture Questions (From Amazon Interview)

### Performance Optimization
1. **Web application with two web servers behind a load balancer talking to a single database is slow. What steps can be taken to improve latency?**
   - Analyze bottlenecks
   - Implement caching (ElastiCache, CloudFront)
   - Optimize database (read replicas, query optimization)
   - Use autoscaling

2. **Issue in database query on AWS. What steps to improve latency?**
   - Use Performance Insights
   - Create appropriate indexes
   - Use connection pooling
   - Implement caching
   - Scale database (read replicas, instance class)

3. **How to improve write speed in a database on AWS?**
   - Scale up instance class
   - Optimize schema design
   - Batch write operations
   - Use Provisioned IOPS storage
   - Shard or partition database

### High-Traffic Applications
4. **Building a high-traffic web application in AWS. What are the key steps?**
   - Use Elastic Load Balancers (ALB/NLB)
   - Enable autoscaling
   - Cache with CloudFront CDN
   - Store sessions outside webservers (Redis/DynamoDB)
   - Optimize backend storage
   - Secure with IAM, WAF, Shield
   - Monitor with CloudWatch

### Container vs VM
5. **What is the difference between using containers and EC2 instances?**
   - Containers: Lightweight, portable, share host OS, faster start times
   - EC2: Complete VM, full OS control, better for monolithic apps

6. **What is the difference between ECS and EKS?**
   - ECS: AWS native, easier setup, tight AWS integration
   - EKS: Managed Kubernetes, cross-platform, multi-cloud capable

### Architecture Design
7. **How would you architect a shopping cart functionality in AWS?**
   - API Gateway + Lambda for cart operations
   - ElastiCache (Redis) for guest users
   - DynamoDB/RDS for logged-in users
   - CloudFront for static assets
   - AWS Cognito for authentication

### Storage
8. **What is the difference between SAN, NAS, and DAS?**
   - SAN: Block-level storage, high scalability, expensive
   - NAS: File-based storage, network overhead
   - DAS: Direct attached, high-speed, not shared

### CDN & Load Balancing
9. **What is a CDN in AWS?**
   - Amazon CloudFront delivers content via global edge locations

10. **What is the difference between Layer 4 and Layer 7 load balancing?**
    - Layer 4: Transport layer (TCP/UDP), IP/port based, faster (NLB)
    - Layer 7: Application layer (HTTP/HTTPS), content-based routing (ALB)

### Security
11. **What is "Federation" and "Identity" in AWS security?**
    - Identity: Users/devices requiring authentication (IAM)
    - Federation: Third-party IdP authentication with trust relationships

---

## Python Questions

### Data Types
1. **What are the built-in data types in Python?**
   - Numeric: int, float, complex
   - Sequence: str, list, tuple, range
   - Mapping: dict
   - Set: set, frozenset
   - Boolean: bool
   - None: NoneType
   - Binary: bytes, bytearray, memoryview

---

## Troubleshooting Scenarios

### ECR
1. Docker login command for ECR
2. Getting "unauthorized" error while pulling image from ECR - How to fix?

### Bastion Host
3. Root key of bastion host is lost. How to fix and login?

### GitLab
4. GitLab CI YAML explanation

---

## Additional Questions to Prepare

1. What exposures are available in a cluster?
2. RDS replicas (if one replica fails, what happens?)
3. What is Terragrunt? How does it work?
4. Scenario: depend_on usage in Terraform

---

*Note: This document is a compilation of interview questions from various DevOps, Cloud, and Linux roles. Regular practice and hands-on experience with these topics is recommended.*
