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
1. Why do we use providers in Terraform?
2. Why do we use provisioners?
3. How does Terraform manage state?
4. What kind of modules have you worked on?

### Functions & Logic
5. Can you explain the scenario-based usage of `for_each` and `count`? How are they different?
6. What is the difference between locals and variables in Terraform?
7. Is there anything you can do with locals but not with variables?

### Practical Scenarios
8. **Task:** Write Terraform code to provision an AWS EC2 instance with:
   - Instance Type: t2.micro
   - AMI: Latest Ubuntu 20.04
   - Availability Zone: Any
   - Security Group: Allow SSH access from anywhere (port 22)
   - Key Pair: Use existing "my-key-pair"
   - Tags: Name = "MyInstance"

9. What are all mandatory parameters to create an EBS volume using Terraform? (az, size, type)

10. **Dynamic Block Example:** Create security group with rules using dynamic block:
    ```hcl
    locals {
      allowed_cidrs = [
        {"https":"0.0.0.0/0"}, 
        {"ssh":"1.1.1.1/32"}
      ]
    }
    ```

11. What is the difference between `count` and `for_each` in Terraform?

### State Management
12. How do you validate Terraform templates? What system do you have to check your template?
13. What about drifting?

---

## Linux Questions

### System Information
1. How to check kernel version? (`uname -rsv`)
2. What is initramfs and where is it located?
3. If initramfs gets corrupted, how do you repair it?
4. How to identify if you're working on a physical or virtual server? (`dmidecode -t`)

### Storage Management
5. How to create an LVM?
6. Commands: `lsblk`, `fdisk -l`
7. Disk is not reflecting using lsblk or fdisk command. How to troubleshoot?
8. How to reduce disk size in LVM?
9. RHEL 8 uses XFS. How to reduce XFS filesystem?
10. How to extend volume if it is filled up?
    - `lsblk`
    - `sudo growpart /dev/nvme0n1 1` (for XFS)
    - `sudo xfs_growfs -d /` (for XFS)
    - `resize2fs /dev/nvme0n1p1` (for ext4)

### NFS & Filesystem
11. How do you export NFS? (`export /dir`)
12. Filesystem is going to readonly mode. How did you fix it?
13. Several NFS modes exist in Linux. How to check?

### User Management
14. How do you set maximum number of files for a specific user? (`/etc/security/limits.conf`)
15. User is not able to create a file or directory. What is the issue?

### Networking
16. How to check what ports are listening on a remote host? (nmap)
17. How do you check network bandwidth between 2 Linux servers when copying a file?
18. How to set permanent route? (Using iptables)
19. How to check active routes? (`/etc/netplan`)

### DNS
20. Explain DNS - port, `/etc/resolv.conf`, nslookup
21. Using nslookup not getting anything. How to troubleshoot?

### System Logs
22. How to check kernel messages? (`dmesg`)

### Shell Scripting
23. Shell command to remotely sync large data from /data1 on server1 to /data2 on server2. Consider main folder has subfolders.

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
