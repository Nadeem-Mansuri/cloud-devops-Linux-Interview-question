
Introduction, Techinical Expertise.

1. what all ec2 family instance type you worked on, could you please named them. - 
2. How you taking a decision while choosing a instances.
3. How do you take care a High availailabilty while designing a subnet.  - No idea
4. Is is possible to create multiple subnet into 1 az.                   
5. Can we have single subnet spread to multiple az.
6. I want to block cetain IP to my Subnet, how can you do that.
7. 10 vpc to interconnect, what is the best way to get it done.
8. what is path based routing in LB.
9. what is host based routing.
10. how to you implement a incryption into your loadbalancer. - 
11. What is difference between Incryption in Trasit and Incryption at rest.
12. Implementing a SSL certificate in LB, that is  Incryption in Trasit or Incryption at rest.
13. what is the use of lifecycle hook in autoscaling.
14. Why do you need API gateway, what all the additonal features you get in API gateway which is not there in Loadbalancer.
15. What are the different types of records present into a route53. Cname & Alias.
16. write any cname mapping in example.com.
17. What is the difference between EBS and S3.
18. What is Lambda Layer.
19. what is syncronous and asyncronous invocation.
20. what is the different kind of encryption is present into S3.
21. What is lifecyscle policy in S3. reverce order is possible.?
22. what is IAM role and I am policy.
23. How many kind of IAM policy are there.
24. what is mean by principal in IAM policy
25. what is identity based and resources based policy in IAM.
26. There is 2 account Account A and Account B, EC2 instance is present into account A and S3 bukcet is present itnt
account B, How to you configure it so ec2 can access S3 bucket in account B.
27. WAF, Guardrail, Guardduty, what is cloudfront.
29. What is multi Az in RDS.
30. what is use of multli AZ and Read replica in RDS.
31. what is authentication and authrozation in RDS.
32. What is log group & Log stream in CLoudwatch.
33. what is the difference between CW and Cloudtrail.
34. what is custom metric in cloudwatch.
35. where you can apply policy.
36. Types of policy in scp.
37. Where did SCP affect
38. What is Dynamic and Static Route in Transit-Gateway
39. what is DX Location.
40. How Directconnect Work
41. How to did you encrypt direct connect traffic
42. In whcih direct connect partner you are collobarating. Types of diredt connect - Dedicated connection and Hostsed connection.
43. What is LOA in Direct Connect

------------DOCKER----------
1. What is multi staged docker container.
2. what is the differnce between add and copy.
3. Your container is writing a log in some path inside a container and you want to see from your host machine.
4. what is tasks & task defination in ECS.

why do we use provider in terrafrom.
Why do we provisioner.
scenerio is depend on.

Git.
How do you change the last commit?
Use `git commit --amend` to modify the most recent commit. This can change the commit's message or include new changes.

What is the difference between `git merge` and `git rebase`?
The main difference is in how the branch history is presented. 
`git merge` preserves the history of a feature branch by creating a new merge commit. 
`git rebase` rewrites the feature branch's history to appear as if it was developed from the latest main branch, creating a linear history.
23552 - jira
What is `git checkout`?

What is the HEAD in Git? HEAD is a reference to the last commit in the currently checked-out branch.

How do you ignore changes in a tracked file?
Use `git update-index --assume-unchanged <file>` to ignore changes in a tracked file temporarily.

What are master component in kubernetes.
how etcd communicate to API servers.
kubelet is a master component or node component.
How the traffic is routed to the Pods, could you please explain.
What is main purpose of services in kubernetes.
When we use Liveness probe and Rediness probes.

Root keys lossed of a bastion, how you can fix and login the issue.

ECR -> 
docker login command
I am getting error unauthorized while pulling the image from ECR

what is the use of lifecycle hook in autoscaling.

How Would You Implement Zero-Downtime Deployments in Kubernetes?
Discuss Strategies for Managing Stateful Applications in Kubernetes.
Explain How You Would Optimize Resource Usage in a Kubernetes Cluster.
Describe How You Would Secure a Kubernetes Cluster.
How Can You Ensure High Availability of the etcd Cluster Used by Kubernetes?
How Would You Approach Capacity Planning for a Kubernetes Cluster?
How Do You Handle Logging and Monitoring in a Large-scale Kubernetes Environment?

Task: Write Terraform code to provision an AWS EC2 instance in a specific region with the 
following specifications:

Instance Type: t2.micro
AMI: Only Latest AMI's of Ubuntu 20.04.
Availability Zone: Any
Security Group: Allow SSH access from anywhere (port 22)
Key Pair: Use an existing key pair named "my-key-pair"
Tags: Add a tag with the key "Name" and value "MyInstance"



Can you explain the scenario-based usage of the Terraform functions for_each and count? 
How are they different, and can you provide examples of when you might use each?

Flexibility and Change Management:

resource "aws_kms_key" "session_manager_kms_key" {
  count                   = local.create_kms_key ? 1 : 0
  description             = "KMS keys"
  deletion_window_in_days = 7
  policy                  = data.aws_iam_policy_document.kms_key_policy[0].json
}



-------GIT-------

1. Could you clarify the distinction between monolithic and microservices architectures.
2. could you provide more details about the specific monolithic architecture you have implemented? Lastly, have you migrated to a microservices architecture within the same project?
3. If you are working with a monolithic architecture, you typically store all the code in a single repository.
  However, if you are transitioning to a microservices architecture, will you continue to maintain the code in a single repository,
  or will you distribute it across multiple repositories? What would be the steps you follow in that scenario?
4. what is the difference between git merge and git-rebase
5.  Let’s consider a scenario where you are working on code and have made multiple commits. Now, you want to make another commit, but you want it to use the same commit message as before and avoid creating a new commit message for each push.
6.  After making 10 commits, you decide that the 11th commit should not be added as a separate commit but instead merged into the previous commit. What would be the steps or commands required to accomplish this?
7. You are part of a team working on the same codebase, where you are adding and committing changes.
   However, you encounter a merge conflict in your branch. How would you handle and resolve this conflict,
   especially since another team member is also working on the same codebase?
7. which git workflow you are using in your current project.

----------Jenkins----------

1. what kind of pipeline you have created? Would you have knowledge about it? What kind of pipelines are available in the Jenkins.
2. What is the difference between scripted and declarative pipeline.
3. what does mean by multi-branch branch pipeline? Why we are using it.
4. for any pipeline? And you want to integrate all the different tools with Jenkins. For example, you want to pick the code from Github right?
And then you want to build the code with the enable tool.
Let's take the meven itself. Okay? Then you need to scan that code with the some scanner tool. 
Then you want to create some image or something you need to deploy somewhere, right? But these tools, like Docker communities, or the Github,
or the the Memo itself, how you integrate with the Jenkins

7. Have you set up the Jenkins architecture? Have you? You used right, Jenkins? But you could like install Jenkins or you the slaves, or how the things like architecture means how many
 Jenkins ma master will be there. Notes will be there right? So which architecture you use like one genuine master, one node. How? How do architecture in your case?
 
8.  which plugins you have used for Jenkins.
9.  so how you design your pipeline? And what are the steps in the pipeline like? What kind of steps you have taken? I think you should you have written some pipeline for the github? 
Right? If you want to do some jobs in the through the pipeline itself? So what are the steps in that.
10. have you worked on any application based pipeline like Java.

Docker:-

1.  what kind of docker file you have created?
2. main difference between CMD and Entrypoint, can we use both CMD and Entrypoint in a single docker file.
3. If you use multiple CMD in a single DockerFile, then, what will happen?
4. How many container runtime available.
5. what is the main differnce between virtual Machine and container.

Kuberetes: -
1. What kind of services available in Kubernetes.
2. Headless service in kubernetes - if only database application will be there, that in that case only we will use the headless service
4. Deployment stratigic in Kubernetes. if you are deploying and it got failed how do you handle this situation.
5. Which kubernetes version you are using.

Terrafrom:-

1. How does Terrafrom manages states.
2. What kind of modules you have worked on.
3. How you make your application highly available.
4. RDS replicas. (if one replicas is there and it failed.

what is the functionality of Trasit Gateway.
what is the significance of s3 life cycle policy. How to you utilized.

Account A: s3://epam/demo.txt
Account B: IAM user - mohammad
CLI - user - get, list - demo.txt
- both accounts are not in same OU
- Cannot use secret/access keys

GitLab.ci yaml explanation.

How would you rollback you deployment.( What went wrong, How did you fix it and how did the rollback happen)
What all the deployment strategies you have used.
What all mandatory paratmeter to create a EBS volume using terrafrom.(az,size,type).
Create a docker file to deploy nginx web server on port 8080. Write commands to build the image and run it as a container.

How to get the following info about pods: 
a. list of pods in a cluster
b. description of a specific pod. 
c. detailed info about pods in a namespace
d. drain a Node

Shell: 
Shell command to remotely sync large data from /data1 on server1 to /data2 on server2. Consider main folder to have sub folders.


1. couple of use-cases that you recently implement in AWS.
2. How to use SSM for patching, how does the entire pipeline use in terms of patching.
3. How will you design a VPC for a 3 tier Acrh.
4. Transit Gateway, VPC peering Insight, Pre-check for VPC peering. 
5. Multi account connectivity and On-Prem connectivity.
6. NACL rule understanding.
7. Have you ever worked on handeling multiple accounts Organisation, SCP etc.
8. What is the first SCP you can think of applying into the root level.
9. What is the benifits of using ORG.
10. WHat is the permission set on SSO.
11. what are the security services you have used in AWS.
12. How do you set guard duty for an ORG to implement in all the AWS account.
13. How to integrate Guardduty with security-hub.
14. What are the different ways will use will RDS in terms of DR, that AWS recommend.
15. What the switch over strategy when you writer instance is down in terms of DR.
16. How do you really failover where your replication is configured.
17. What are limitation of using AWS lambda function like concurrency.
18. What is the session in boto3, what do we create.
19. what is the differnce between boto3 client and resources.
20. For SNS what the endpoint present today to send your notification.
21. How do you configure SNS in order so it will send notification in order.
22. what is dead-letter-ques in SQS.
23. Lets say your instance is deleted 1 year backup, you have a backup of cloudtrail logs present into
     s3 bucket. Which AWS tools you are going to use to analyis those data.
24. STS have 2 endpoints, global and regional. how do they differnces where do you use them.
25. Permission boundry question.
26. What is the difference between Local and Variables in terrafrom.
27. Any thing that you can think of we can with locals but not using variable.
28. Create security group with rules using dynamic block?
locals{
allowed cidrs =
[{"https":"0.0.0.0/0"}, {"ssh":"1.1.1.1/32"}]
}
29. What is the differnce between count and for_each in terrafrom.
30. 

============
s3 bucket
vpce policy: allow all traffic: principalORG: org_id

No connectivity between my system and AWS VPC

allow: s3:*
resource: [arn:aws::s3::mybucket,
arn:aws::s3::mybucket/*]
condition:
  aws:sourceVpce: vpce-s3endpointid

1. Can I view objects of this bucket when logged into this aws account?

2. How can I view the objects?
================

locals{
allowed cidrs =
[{"https":"0.0.0.0/0"}, {"ssh":"1.1.1.1/32"}]
}

Create security group with rules using dynamic block?


Linux -> 


==PROXY LINUX INTERVIEW QUES==

uname -rsv - kernel.
Initramfs, where it is located.
If initramfs got correcpted. how to repair.
how to identity you are working in a physical or a virtual server - dmidecode -t 
How to create a LVM.
lsblk - fdisk -l.
disk is not reflecting using lsblk or fdisk command.
how to reduce the disk size to LVM.
RHEL 8 - XFS. How to reduce 
How do you export NFS - export /dir - 
Filesystem is going to readonly mode, how did you fixed it.
several nfs modes are ther ein linux - how to check 
How to you set maximum number of files for a specifc user. /etc/security/custom.conf
how to check what all port are listen for the remote host - nmap is the correct answer.
How do you check netwirk bandwith between 2 linux server, if you copy a file from 1 to another
How to set permanent route - > Using Iptables.
How to check the active routes. -> /etc/netplan.
How to check kernel messages -> dmesg
User is not able to create a file ot directory what is the issue. 
DNS - > explain - port - /etc/resolve, nslookup
using nslookup not getting anything - How to trouble shoot.

AWS - What is VPC., What are all the prerequisted in order to create a ec2 instance.
what NAT gateway do.
security grp and NACl difference
what kind to exposer in cluster.


=== DevOps QUES ==

 what is the different types of pipiline you are involved with.?
 what are end to end pipeline structure, how the code deploying to the production.
 you have moved to production , how did you monitor, what all tools you are using to monitor that.
 How did the infra for cicd, how it is deployed.
 How did the provisioing done for the infrastructure.
 What is terragunt, how terragrunt work, is it apply manuuly or automatics.
 what about drifting.
 How did you validate the terraform template, what system you have to check your template -> terraform validate, terraform fmt.
 lets say you couple of services in AWS and Azure, How did you integrate, how did you r system get authenticated, how did you ensure data transfer
 

----------DEVOPS-----------
1. How you can configure a kubernetes application to available outside world.?
   To make a Kubernetes application available to the outside world, you need to expose the application's service
   outside the cluster. Kubernetes provides multiple mechanisms to expose an application, such as using NodePort,
   LoadBalancer, or Ingress.

2. There is an high latency expericing in RDS instance, How you can resolve it.?
   To resolve high write latency in RDS:-

	- Analyze metrics using CloudWatch (identify CPU, I/O, memory, and connection bottlenecks).
	- Optimize slow queries and reduce locking contention.
	- Improve storage performance by using Provisioned IOPS and increasing storage capacity.
	- Scale up the instance class for better resources (CPU and memory).
	- Use read replicas to offload read traffic.
	- Adjust database engine parameters, such as transaction handling and caching.
	- Ensure proper application-level optimizations, such as connection pooling and batch operations.
		
These steps should help mitigate the write latency issues. If you're still experiencing issues,
consider testing your workload on Amazon Aurora where write scaling may be more efficient for high-write applications.

3. How to increase IOPS in RDS.
Using AWS Console:-
	1. Navigate to RDS in the AWS Management Console.
	2. Select your database instance.
	3. Click Modify.
	4. Change Storage Type to Provisioned IOPS and specify the desired IOPS.
	5. Adjust the instance type if necessary.
	6. Using AWS CLI

**Using bash command**

aws rds modify-db-instance \
  --db-instance-identifier mydbinstance \
  --allocated-storage 1000 \
  --iops 20000 \
  --apply-immediately

4. If you change instance type in RDS, is there any downtime.
Changes That Typically Cause Downtime
These types of changes require the database instance to reboot, which results in downtime. During the reboot, connectivity to the instance will be interrupted.

	1. Scaling Instance Class
	Changing the instance type/class (e.g., upgrading from db.t2.micro to db.m5.large).
	Downtime: Yes, a reboot is required, and downtime will occur while the database instance switches to the new class.
	How to Minimize Downtime: You can schedule the change during the preferred maintenance window to avoid disrupting live applications:

aws rds modify-db-instance \
  --db-instance-identifier <your-db-instance> \
  --db-instance-class <new-instance-class> \
  --no-apply-immediately

By using --no-apply-immediately, the change will be applied during the maintenance window rather than immediately.

2. Storage Type Changes
Changing the storage type (e.g., switching from gp2 to io1/io2 or gp3).
Downtime: Yes, as this may involve stopping the instance temporarily to migrate the storage.
How to Minimize Downtime: Apply changes during the maintenance window.

3. Enabling Multi-AZ Deployment
Enabling Multi-AZ requires setting up a secondary replica in another availability zone.
Downtime: Yes, a brief service disruption occurs when enabling Multi-AZ.
How to Minimize Downtime: Perform this change during the maintenance window.

4. Parameter Group Changes
If modifying parameters requires a reboot, such as changing instance-level kernel settings or database engine configuration (e.g., innodb_flush_log_at_trx_commit, max_connections on MySQL or PostgreSQL).
Downtime: Reboot required for certain parameter modifications.
How to Minimize Downtime: Schedule the reboot during the maintenance window.

5. Switch to Dedicated Network or Encryption Changes
Switching to a new VPC/subnet group or enabling encryption on storage.
Downtime: Yes, changes require migration or reboot.
How to Minimize Downtime: Perform these operations outside peak hours or during the maintenance window.

Changes That Typically Do NOT Cause Downtime:-
	1. Increasing Storage Size - Increasing allocated storage does not cause downtime.
	2. Updating Instance Tags - Modifying tags to label or organize instances does not affect services.
	3. Adding or Modifying Read Replicas - Adding read replicas or modifying existing replicas does not affect the primary instance.
	4. Performance Features - Changing IOPS values for io1/io2 or switching from gp2 to gp3 does not cause downtime as long as apply-immediately is used.
	5. Monitoring or Logging Features - Turning on options like Enhanced Monitoring, Performance Insights, or Log Exports typically does not cause downtime.
	
6. DR Strategies in AWS	
	1.Backup & Restore: Lowest cost (pay only for backup storage until you restore).
	2.Pilot Light: Moderate cost (resources used minimally).
	3.Warm Standby: Higher cost (scaled-down environments remain operational).
	4.Multi-Site Active/Active: Highest cost ($$$) due to resource duplication across regions.
	
7. When you make DNS changes in Amazon Route 53, how much time it will take for propagation.

A. Key Factors Affecting Propagation Time-
   What is TTL?
	TTL specifies how long DNS resolvers (e.g., ISPs and caching servers) cache a DNS record before checking for an update.
	Each DNS record in Route 53 has a TTL value in seconds. For example, if the TTL is 300 seconds (5 minutes), DNS resolvers will use the cached value for 5 minutes before querying Route 53 for the latest record.
	
	What happens if TTL is high?
	If the TTL is long (e.g., 24 hours = 86,400 seconds), resolvers may continue to use the outdated IP address or value until the TTL expires.
	
	What happens if TTL is low?
	Setting a shorter TTL (e.g., 60 seconds) reduces propagation time because resolvers check for changes more frequently.
		
8. How you can implement CICD for Kubernetes. 
	1. Set Up the Kubernetes Cluster
	2. Set Up a Version Control System (GitHub, GitLab, etc.)
	3. Automate the Build Process in CI/CD Tool
	4. Push the Docker Image to a Container Registry
	5. Update Kubernetes Manifests
	6.  Deploy to Kubernetes
		
	7. Set Up CI/CD Pipelines using Gitlab or Github action
	8. GitOps for Kubernetes using ArgoCD
	9. Monitor and Automate Rollbacks
	
9. Suppose If your existing Amazon EKS Cluster is configured to use a single Availability Zone (AZ) 
	with a managed node group, how you can add a new node group in a different AZ in the existing cluster.
	1. Identify Available AZs in the Region: Check the available AZs for your region:
	2. Create a New Subnet in the Desired AZ: Choose an AZ that is different from the one where your existing node group resides.
	3. Ensure the Subnet Has a Route to the Internet (if required):
	4. Modify Subnet Tags for EKS (Optional):
	5. Associate the New Subnet with the EKS Cluster
	6. Create a New Managed Node Group in the New AZ
	
	
10. How to extend volume if it is filled up.
    	1. lsblk
	2. sudo growpart /dev/nvme0n1 1 (for XFS filesystem)
	3. sudo xfs_growfs -d /  (for XFS filesystem)
	4. resize2fs /dev/nvme0n1p1 (For ext4 filesystem)

=======AWS QUEST======
 
1. Difference between persission boundry and SCP?

When to Use Permissions Boundary vs SCP - 
Use Permissions Boundary:
For individual IAM users or roles where you want to limit the maximum permissions.
Delegating administrative control while ensuring they can’t exceed specific permission limits.
Fine-grained control within a single AWS account.

Use Service Control Policy (SCP):
For multi-account environments where you want to enforce compliance or security rules across many AWS accounts.
To standardize service access and restrictions across an organization or prevent unauthorized use of services (e.g., blocking public S3 buckets across all accounts).
Broad control across organizational units or accounts.

2. How to enable cross-account access to an Amazon S3 bucket in AWS?
There are multiple ways to enable cross-account access to S3:

A. Bucket policy is the simplest and most direct technique, useful for simple resource-level permissions.
B. IAM roles provide a fine-grained and secure mechanism for controlling cross-account access, preferred in complex organizational setups.
C. Access Points simplify permissions management for large-scale bucket sharing.

3. What is a StackSet in AWS CloudFormation?
A StackSet in AWS CloudFormation is a feature that allows you to manage CloudFormation stacks across multiple AWS accounts and regions from a centralized location.
It simplifies and automates the deployment and management of AWS resources for multi-account and multi-region architectures, ensuring consistency and scalability
in your infrastructure configuration.

How Does a StackSet Work?
A. Create a CloudFormation template that defines the resources you want to deploy.
B. From your administrator account, create a StackSet and specify the list of target accounts and AWS regions in which the stacks should be deployed.
C. AWS CloudFormation uses service-managed roles or self-managed permissions to deploy stacks into the target accounts and regions.
D. Updates to the StackSet are automatically rolled out across all the associated stacks.

4. Upgrade Process of EKS Cluster and Worker Node with zero downtime.
It can be done via AWS CLI or Terraform.

5. How to avoid using a NATING during the upgrade of an Amazon EKS cluster?
To avoid using a NAT Gateway during the upgrade of an Amazon EKS cluster, you need to ensure that the EKS control plane components and worker nodes
(or managed node groups) can communicate with required AWS services (e.g., amazonaws.com endpoints, container registries, etc.) without routing through
the public internet. This can be achieved by leveraging VPC endpoints to enable private network communication between your EKS cluster and AWS services.


6. What are the steps to install datadog agent into worker node.
The installation of the Datadog Agent on EKS worker nodes will use a DaemonSet.
A DaemonSet ensures that a copy of the Datadog Agent runs on every worker node within your Kubernetes cluster.

A. Ensure you have a Datadog account and obtain your API key. Go to Integrations → APIs in your Datadog dashboard.
B. IAM Role for ServiceAccount (IRSA): To enable the Datadog Agent to appropriately monitor and interact with AWS resources securely, you need to create an IAM Role for ServiceAccount.
C. Step 3: Deploy the Datadog Agent.
	Add Datadog Helm Repository: Datadog provides a Helm chart for easy installation of the Agent.
	# helm repo add datadog https://helm.datadoghq.com
	# helm repo update

Step 4: Verify Datadog Integration
Datadog Dashboard:

Log in to your Datadog dashboard and navigate to the Infrastructure section. You should see metrics populating from the worker nodes and Kubernetes clusters.
Containers and Logs:

Confirm that container-level metrics are collected by Datadog.
If enabled, logs should also be available (e.g., Kubernetes events).

7. How to upgrade worker nodes in an Amazon Elastic Kubernetes Service (EKS) cluster with zero downtime.
By following the rolling upgrade strategy—whether using managed or self-managed node groups—you can upgrade EKS worker nodes with zero downtime.

8. What is IRSA in Kubernetes (Amazon EKS)?
IRSA, or IAM Roles for Service Accounts, is a feature introduced by AWS for Amazon EKS (Elastic Kubernetes Service) that integrates Kubernetes service accounts
with AWS Identity and Access Management (IAM) roles. IRSA allows you to associate IAM roles with Kubernetes service accounts to manage permissions for pods
running within your EKS cluster.
This eliminates the need to attach IAM permissions directly to an EC2 instance (e.g., worker nodes), thereby enabling fine-grained access control for workloads in Kubernetes. 
With IRSA, pods in your Kubernetes cluster can perform secure, scoped actions in your AWS environment by using temporary credentials tied to the associated IAM role.

9. What Problems Does IRSA Solve?

Granular IAM Permissions:
Without IRSA, granting IAM permissions is typically done at the EC2 instance level. All pods on the same node can inherit the same IAM role, leading to over-permissioning.
IRSA allows you to attach specific IAM roles to individual service accounts, ensuring pods using these service accounts only have access to what they need.

Security:
By removing node-level IAM permissions and tying them to individual pods (via service accounts), IRSA reduces the blast radius if a pod is compromised.

IAM Credential Management:
IRSA uses AWS Security Token Service (STS) to manage temporary credentials for pods, ensuring credentials remain dynamic and secure without manual intervention.

10. How IRSA Works?

Here's a high-level summary of how IRSA integrates AWS and Kubernetes:

Service Account in Kubernetes:
You create a Kubernetes service account in a specific namespace where the pod resides.

IAM Role Associated with Service Account:
You create an IAM role in AWS and associate it with the Kubernetes service account. This IAM role includes the permissions the pod requires to interact with AWS services
(e.g., S3, DynamoDB, CloudWatch).

OIDC Identity Provider:
Amazon EKS uses an OpenID Connect (OIDC) identity provider to authenticate the pod for AWS API calls.
You must enable an OIDC identity provider for your cluster.

Pod Uses Service Account:
Pods running in the cluster use the specified service account, which in turn assumes the IAM role associated with it. 
The permissions granted to this IAM role dictate what AWS actions the pod can perform.

**Common Issues and Troubleshooting in ORSA

Role Not Applied to Pod:
Ensure the pod uses the correct serviceAccountName and the service account has the required IAM role annotation.

Invalid OIDC Configuration:
Verify the OIDC provider URL, thumbprint, and IAM role trust relationship.

Insufficient Permissions:
If the pod is unable to access AWS resources, make sure the IAM policy attached to the role provides the necessary actions and resources.

11. How how pods networking works in eks cluster?
Networking for pods in an Amazon EKS (Elastic Kubernetes Service) cluster is primarily based on the Kubernetes networking model,
integrated with AWS's networking infrastructure (such as the VPC, subnets, ENIs, and security groups).

12. how pods networking works in eks cluster?

Pod networking in Amazon EKS is achieved using the AWS VPC CNI plugin, which tightly integrates Kubernetes and AWS networking concepts.
It provides pod-level IPs directly in the VPC, supports advanced routing between pods, and allows secure communication with AWS resources.
By leveraging Kubernetes networking policies and AWS infrastructure, pod networking in EKS ensures scalability, security, and seamless integration with other AWS services.

** High-Level Overview
Pods in EKS use Container Network Interface (CNI) plugins for networking.
AWS provides the vpc-cni plugin (AWS CNI), which allows Kubernetes pods to get IP addresses directly from the Amazon VPC's subnet and communicate using the VPC's networking features.
Each pod in the cluster is assigned a private IP address from the VPC.

Key Components of Pod Networking in EKS:-
1. Amazon VPC

EKS uses an Amazon Virtual Private Cloud (VPC) to facilitate pod networking:

Pods get IP addresses from the VPC's subnets.
These IP addresses allow pods to natively communicate with other AWS services (e.g., S3, DynamoDB) via private IP-based VPC routing.
Internet access is possible through NAT Gateways or VPC endpoints.

2. AWS CNI Plugin
The AWS CNI (vpc-cni) plugin is the default CNI plugin used in EKS clusters:

It assigns Elastic Network Interfaces (ENIs) and private IP addresses from the VPC to Kubernetes pods running on worker nodes.
The IP addresses assigned to the pods come from the worker node's subnet, making it possible for pods to communicate with AWS resources directly.

Key features of AWS CNI:

Pods are treated as native VPC resources.
Supports multiple ENIs for high scalability on nodes.
Pod-to-Node communication occurs via private IP addresses.

3. Elastic Network Interfaces (ENIs)
When a pod is created, the worker node's ENI handles networking for the pod:

The ENI is attached to the underlying EC2 instance (worker node).
The primary and secondary IP addresses on the ENI are assigned to pods.

4. Kubernetes Networking
Kubernetes relies on the networking model where:

Every pod in the cluster gets its own IP address (Pod IP).
Pods communicate with each other regardless of which nodes they reside on (using the overlay network or AWS VPC routing).
Network rules (e.g., NetworkPolicies) control allowed communications between pods.

**Pod Networking Flow in EKS
1. Pod Creation
When Kubernetes schedules a pod:

The AWS CNI plugin requests an IP address for the pod from the VPC subnet associated with the worker node.
The CNI plugin allocates an available secondary IP from the node's ENI and assigns it to the pod.

2. Pod-to-Pod Communication
Pod networking in EKS follows the Kubernetes networking model:

Same Node: If two pods are running on the same node, they communicate locally via the localhost or the worker node's internal network.
Different Nodes: Pods on different nodes communicate via the VPC to route traffic. The worker nodes' ENIs and subnet configuration handle inter-node communication.

3. Pod-to-Service Communication
When pods communicate with a Kubernetes service:

A service has a ClusterIP, which acts as a virtual IP backed by an iptables or IPVS configuration.
Pods reach the service and route traffic to one of its backend pods (using kube-proxy for load balancing).

4. Pod-to-External Communication
For external communication, pods typically:

Route traffic through a NAT gateway if they are in private subnets.
Use AWS VPC endpoints for direct communication with AWS services like S3 or DynamoDB, bypassing the internet.
