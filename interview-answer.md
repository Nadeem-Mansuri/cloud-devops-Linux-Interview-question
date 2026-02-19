# DevOps Interview - Questions & Answers

**Interviewer:** Ajit (Technology Lead - 14 years experience)  
**Candidate:** Hussein Lokhandwala (Senior DevOps Engineer - 4.5 years experience)  
**Certifications:** AWS, Kubernetes, Terraform, Azure

---

## AWS Architecture & Services

### Q1: How would you architect a fully available and fault tolerant application on AWS?

**Answer:**
Assuming a data analytics application that requires multiple services:

**Networking:**
- Create VPC with private and public subnets
- Place compute and database in private subnets
- Place Application Load Balancer in public subnets
- Configure NACL and security groups appropriately

**Storage:**
- Use S3 with cross-region replication for data redundancy
- Enable versioning and lifecycle policies

**Database:**
- Use RDS with read replicas in cross-region for disaster recovery
- Consider DynamoDB with global tables if application supports it

**Compute:**
- Deploy EKS with cross-region node deployments
- Have primary cluster in main region and secondary in DR region
- Use autoscaling for fault tolerance

**Monitoring:**
- AWS CloudWatch logs for basic monitoring
- Integrate Prometheus and Grafana for custom application and Kubernetes logs

**DNS & Traffic Management:**
- Configure Route 53 with failover policies
- Automatically divert traffic to DR region if primary fails

**Security:**
- Use AWS Secrets Manager for storing secrets
- Implement encryption at rest and in transit

**Automation:**
- Infrastructure as Code using Terraform
- GitOps approach using ArgoCD for Kubernetes deployments
- Automated CI/CD pipelines

---

### Q2: Why would you use EKS and not any other container service like ECS?

**Answer:**
For data analytics applications requiring high security:

**Security Benefits:**
- RBAC (Role-Based Access Control) rules
- Network policies for pod-to-pod communication
- More granular security controls

**Deployment Advantages:**
- GitOps workflow using ArgoCD
- Easier rollbacks
- Better version control

**Flexibility:**
- More customization options
- Rich ecosystem of tools and plugins
- Not locked into AWS-specific implementations

---

### Q3: Can you tell me a scenario where ECS will be better suited as compared to EKS?

**Answer:**
ECS is suitable when:
- Simple deployment with fewer services
- Everything manageable within AWS ecosystem
- Access management can be done using IAM policies
- No need for complex services like Ingress, Istio, etc.
- Routes can be managed via Application Load Balancer
- Task definitions are sufficient for application needs
- Team is more familiar with AWS-native tools

---

### Q4: Which one is cheaper - ECS or EKS? Why?

**Answer:**
**ECS is cheaper** because:
- Only costs for the services/resources you use (EC2 instances, Fargate tasks)
- No control plane charges

**EKS is more expensive** because:
- Charges for the EKS cluster control plane ($0.10 per hour per cluster)
- Additional costs for worker nodes (EC2 instances)
- Add-ons and additional services cost extra

---

## AWS Networking

### Q5: What configuration is needed to make a subnet public or private?

**Answer:**
The configuration lies in the **route tables** assigned to the subnet:

**Public Subnet:**
- Route table has a route directing traffic (0.0.0.0/0) to an Internet Gateway
- Internet Gateway is attached to the VPC (not the subnet)
- Any instance in this subnet can communicate with the internet

**Private Subnet:**
- Route table directs outbound traffic to a NAT Gateway
- NAT Gateway is placed in a public subnet
- Allows outbound internet access but prevents inbound connections from internet

---

### Q6: List down all the choices to connect on-premises network with AWS.

**Answer:**

1. **AWS Direct Connect**
   - For high-volume workload migration
   - Dedicated network connection
   - Higher cost but better performance and security

2. **Site-to-Site VPN Connection**
   - Create Customer Gateway (on-premises side)
   - Create Virtual Private Gateway or Transit Gateway (AWS side)
   - Cheaper for moderate workloads
   - Uses internet for connectivity

3. **Transit Gateway**
   - Hub-and-spoke model
   - Can connect multiple VPCs and on-premises networks
   - Simplifies network architecture

4. **AWS VPN CloudHub**
   - For connecting multiple branch offices to AWS and each other

---

### Q7: Why do you require a transit gateway when we have VPC peering? What problem does transit gateway solve?

**Answer:**

**VPC Peering Limitations:**
- Only connects two VPCs
- Not transitive (if A peers with B, and B peers with C, A cannot reach C)
- For N VPCs, need N*(N-1)/2 peering connections
- Management complexity increases exponentially

**Transit Gateway Benefits:**
- Hub-and-spoke model - one central gateway
- All VPCs connect to Transit Gateway
- All VPCs can communicate through the hub
- Simpler management
- Easier to add new VPCs
- Supports on-premises connectivity

**Cost Consideration:**
- VPC peering has cheaper data transfer rates
- Transit Gateway has additional hourly charges plus data processing fees
- For just 2 VPCs, peering is more cost-effective

---

## Disaster Recovery & Global Architecture

### Q8: How would you design a disaster recovery solution for an e-commerce website?

**Answer:**
The solution depends on the criticality and acceptable downtime:

**1. Backup and Restore (Cheapest - Some Downtime Acceptable)**
- Backup EKS cluster configurations
- Backup RDS databases
- Store backups in S3 with cross-region replication
- During disaster: restore infrastructure and data in DR region
- Route traffic via Route 53 once restored
- **Downtime:** Hours to days

**2. Pilot Light (Warm Standby - Less Downtime)**
- RDS read replica in DR region
- EKS cluster deployed but nodes scaled to zero
- Application ready but not running
- During disaster: scale up nodes, promote read replica, route traffic
- **Downtime:** Minutes to hours

**3. Multi-Site Active/Active (No Downtime - Most Expensive)**
- Fully scaled EKS in both regions
- RDS with cross-region read replicas
- Route 53 health checks and failover policies
- Automatic traffic routing during failures
- **Downtime:** Near zero (seconds)

---

### Q9: How would you design to support a website globally (US and India) with minimum latency and separate databases?

**Answer:**

**Content Delivery:**
- CloudFront (CDN) for serving static content globally
- Reduces latency by caching at edge locations

**Load Balancing:**
- Global Load Balancer (Route 53 with geolocation routing)
- Routes users to nearest regional endpoint

**Database:**
- Read replicas in all regions where traffic originates
- DynamoDB global tables for multi-region writes
- Data synchronization between regions

**Caching:**
- ElastiCache or Redis in each region
- Cache frequently accessed data
- Reduces database load and improves response time

**Result:**
- Users access nearest data center
- Lowest latency for both static and dynamic content
- Regional data residency if required

---

## Security Best Practices

### Q10: Security best practices in compute, networking, databases, and storage?

**Answer:**

**Networking:**
- AWS Shield for DDoS protection
- AWS WAF for application layer protection
- VPC Flow Logs to trace all network traffic
- Properly configured Security Groups (least privilege)
- Network ACLs for subnet-level protection

**Compute & Storage:**
- Encrypt RDS and EBS volumes with KMS keys
- Regular automated backups of RDS and EBS
- Enable versioning on S3 buckets
- S3 bucket policies with least privilege
- Use IMDSv2 for EC2 metadata

**Compliance & Monitoring:**
- AWS CloudTrail for API audit logging
- AWS Config for resource configuration tracking
- Config Rules for automated compliance checking
- Lambda functions for automated remediation
- CloudWatch alarms for anomaly detection

**Database Specific:**
- Automated backups with appropriate retention
- Encryption in transit (SSL/TLS)
- Encryption at rest
- Database activity monitoring
- Regular security patching

---

### Q11: For a cost-conscious customer, what are bare minimum security practices?

**Answer:**

**Access Control (No Cost):**
- Security Groups with least privilege (no 0.0.0.0/0 unless necessary)
- Network ACLs configured at subnet level
- Granular IAM policies (principle of least privilege)
- IAM Identity Center with permission sets for team-based access
- No root account usage

**Infrastructure as Code (Best Practice):**
- Deploy everything through automation (Terraform)
- No manual configurations in AWS console
- Version control for all infrastructure changes
- Requires DevOps approval for changes
- Prevents configuration drift

**Basic Monitoring:**
- CloudWatch Logs (basic tier included in free tier)
- Set up alarms for critical metrics
- Monitor failed login attempts

**Network Security:**
- Keep databases in private subnets
- Use VPC endpoints where possible
- Minimize public endpoints

**Key Management:**
- Never hardcode credentials
- Use AWS Secrets Manager or Parameter Store
- Rotate credentials regularly

---

### Q12: How would you extend database access to a third party without compromising security?

**Answer:**

**Authentication Methods:**
1. **IAM Database Authentication** (Recommended)
   - Create IAM role for third party
   - Role can authenticate to database without passwords
   - Temporary credentials with expiration
   - Audit trail through CloudTrail

2. **Traditional Credentials**
   - Strong, unique username/password
   - Stored in Secrets Manager
   - Regular rotation

**Network Security:**
- Security Group with specific IP allowlist (not 0.0.0.0/0)
- Whitelist only third party's IP addresses
- Consider VPN or AWS PrivateLink for additional security

**Encryption:**
- Enable TLS/SSL for connections
- Force SSL connections in database configuration
- Reject non-encrypted connections
- Use SSL certificates for mutual authentication

**Monitoring:**
- Database activity monitoring
- CloudWatch alarms for unusual access patterns
- VPC Flow Logs to track connection sources

---

### Q13: What AWS service can detect vulnerable packages in EC2 instances?

**Answer:**
**AWS Inspector**
- Automated security assessment service
- Scans EC2 instances, container images, and Lambda functions
- Identifies software vulnerabilities (CVEs)
- Provides prioritized list of security findings
- Integrates with Security Hub
- Continuous scanning of resources
- Checks against CIS benchmarks and other compliance standards

---

## AWS Well-Architected Framework

### Q14: List down all the pillars of AWS Well-Architected Framework and summarize each.

**Answer:**
The **six pillars** (initially five, sustainability added later):

**1. Operational Excellence**
- Ability to develop and run workloads effectively
- CI/CD pipelines for automated deployments
- Infrastructure as Code
- Monitoring and logging
- Incident response procedures
- Continuous improvement through feedback loops

**2. Security**
- Protecting information and systems
- Identity and access management
- Detective controls (CloudTrail, Config)
- Infrastructure protection
- Data protection (encryption)
- Incident response capabilities

**3. Reliability**
- Workload performs intended functions correctly and consistently
- Multi-AZ deployments
- Disaster recovery procedures
- Backup and restore capabilities
- Automatic scaling
- Change management processes

**4. Performance Efficiency**
- Using computing resources efficiently
- Selecting right instance types for workload
- Selecting appropriate database engine
- Using serverless where applicable
- Monitoring performance
- Making informed decisions based on data

**5. Cost Optimization**
- Avoiding unnecessary costs
- Right-sizing instances
- Using reserved instances and savings plans
- Monitoring and analyzing spending
- Using appropriate storage classes
- Implementing auto-scaling

**6. Sustainability**
- Minimizing environmental impact
- Using efficient instance types
- Maximizing utilization
- Reducing idle resources
- Using managed services (AWS handles efficiency)
- Selecting regions with renewable energy

---

## Cross-Account Access

### Q15: How do you facilitate cross-account connectivity (e.g., accessing S3 or KMS from another account)?

**Answer:**
Using **AWS STS (Security Token Service)** with role assumption:

**Setup Process:**

**Account A (Source - has EC2):**
1. Create IAM role in Account A
2. Attach policy allowing `sts:AssumeRole` action
3. Specify Account B role ARN in the policy

**Account B (Target - has S3):**
1. Create IAM role in Account B
2. Attach policies for S3 access (GetObject, PutObject, etc.)
3. Configure Trust Policy to allow Account A to assume this role
4. Add Account A role ARN in trust relationship

**Access Flow:**
1. EC2 in Account A uses its IAM role
2. Calls STS AssumeRole for Account B role
3. Receives temporary credentials
4. Uses temporary credentials to access S3 in Account B

**For S3 Specific:**
- Also configure S3 bucket policy to allow the role
- Bucket policy provides additional layer of access control

---

### Q16: What issues have you faced with cross-account access?

**Answer:**

**KMS Key Access Issues:**
- Not just read access to KMS key needed
- Must grant specific KMS permissions (kms:Decrypt, kms:DescribeKey)
- Need to add policies in both KMS key policy AND IAM role
- Key policy must allow the role from other account

**S3 Bucket Policy Issues:**
- Must configure bucket policy in addition to IAM policies
- Bucket policy needs to explicitly allow the role ARN
- Easy to miss when testing cross-account access

**Trust Relationship Issues:**
- Trust policy must be correctly configured
- External ID sometimes required for additional security
- Incorrect ARN format causes failures

**Debugging Tips:**
- Check CloudTrail for access denied errors
- Verify both IAM policies and resource policies
- Ensure all services (KMS, S3, STS) have proper permissions

---

## Static Website Hosting

### Q17: What configurations are needed to host a static website on S3?

**Answer:**

**S3 Configuration:**
1. Create S3 bucket (bucket name should match domain if using custom domain)
2. Upload website files (index.html, CSS, JS, images)
3. Enable static website hosting in bucket properties
4. Specify index document (e.g., index.html)
5. Specify error document (e.g., error.html)
6. Configure bucket policy to allow public read access

**CloudFront (CDN):**
1. Create CloudFront distribution
2. Set S3 bucket as origin
3. Configure Origin Access Identity (OAI) for secure access
4. Update S3 bucket policy to allow CloudFront OAI
5. Set default root object to index.html

**Route 53 (Optional - for custom domain):**
1. Create hosted zone for domain
2. Create A record with alias to CloudFront distribution
3. Update domain registrar's name servers if needed

**Benefits:**
- Global content delivery
- HTTPS support via CloudFront
- Better performance with caching
- DDoS protection

---

## Cost Optimization

### Q18: What cost optimization solutions would you implement?

**Answer:**

**General Approach:**
1. Never compromise on compliance and security
2. Analyze cost by service (identify highest costs)
3. Look for cheaper alternatives that meet requirements

**Compute Optimization:**
- **Reserved Instances** for predictable, long-term workloads (up to 75% savings)
- **Savings Plans** for flexible commitment-based discounts
- **Spot Instances** for fault-tolerant, flexible workloads (up to 90% savings)
- Right-size instances based on actual usage
- Use auto-scaling to match demand
- Switch to Graviton instances (ARM-based) for cost savings

**Storage Optimization:**
- Migrate from GP2 to GP3 EBS volumes (newer, cheaper, better performance)
- Use S3 Intelligent-Tiering or lifecycle policies
- Move infrequently accessed data to Glacier/Deep Archive
- Reduce backup retention periods where appropriate
- Delete old snapshots and AMIs

**Instance Family Optimization:**
- Evaluate if you need specialized instances (GPU, memory-optimized)
- Switch between instance types in same family (e.g., t3 to t3a for cost savings)
- Use T3/T4g burstable instances for variable workloads

**Database Optimization:**
- Right-size RDS instances
- Use RDS Reserved Instances
- Consider Aurora Serverless for variable workloads
- Optimize backup retention

**AWS Tools:**
- **AWS Cost Explorer** - analyze spending patterns
- **AWS Trusted Advisor** - provides cost optimization recommendations
- **AWS Compute Optimizer** - machine learning-based recommendations
- **AWS Budgets** - set up alerts for cost thresholds

---

### Q19: When would you use spot instances for batch jobs?

**Answer:**

**Suitable Scenarios:**
- Job can handle interruptions gracefully
- Retry mechanism is available
- Job runtime is flexible (not time-critical)
- Can checkpoint and resume work
- Significant cost savings outweigh occasional interruptions

**For 11:30 PM Scheduled Job:**
Consider these factors:
1. **Job Duration:** How long does it take?
2. **Retry Capability:** Can it be retried if interrupted?
3. **Time Window:** Is there flexibility if it completes at 11:45 PM vs 11:35 PM?

**Decision:**
- If job is critical and must complete within tight window: **Use On-Demand**
- If job can retry and has buffer time: **Spot is acceptable**
- Spot instances can be interrupted with 2-minute warning
- Implement graceful shutdown and state saving

**Best Practice:**
- Use spot instances with multiple instance types as fallback
- Implement checkpointing to save progress
- Use spot fleet for better availability
- Have retry logic in place

---

### Q20: What if the spot instance type is not available in that region?

**Answer:**

**Fallback Strategy:**
1. **Multiple Instance Types:** Configure Spot Fleet or EC2 Fleet with multiple instance types
2. **Instance Family Flexibility:** Allow different instance families if requirements permit
3. **Cost Analysis:** Higher spot instance type may still be cheaper than on-demand

**Example:**
- Requested: t3.medium spot (not available)
- Fallback: t3.large spot (still cheaper than t3.medium on-demand)
- Alternative: t3a.medium, t2.medium, or other compatible types

**Implementation:**
- Define a prioritized list of instance types
- AWS automatically tries alternatives
- Balance between cost and availability
- Monitor spot interruption rates by instance type

---

## Infrastructure as Code

### Q21: When would you choose CloudFormation versus Terraform?

**Answer:**

**Prefer Terraform When:**
- **Multi-cloud support** needed (AWS, Azure, GCP, etc.)
- **Large community** - easier to find solutions
- **Modular code** - write once, use for multiple environments
- **State management** with S3 backend and DynamoDB locking
- **Multiple accounts/environments** manageable from single directory
- Better variable management and reusability
- Rich provider ecosystem (not just cloud providers)

**Prefer CloudFormation When:**
- **New AWS services** - CloudFormation supports immediately
- **AWS-native integration** is priority
- **Lambda deployment** - easier inline code support
- Team heavily invested in AWS ecosystem
- Want AWS-managed state management
- Using AWS-specific features extensively

**Personal Preference:**
Would still prefer Terraform even for AWS-only projects because:
- Better code organization
- Stronger community support
- More flexible and powerful
- Better tooling and IDE support
- Established patterns and modules

---

### Q22: For dedicated AWS infrastructure, would you still choose Terraform?

**Answer:**
**Yes, still Terraform** because:

**Beyond Multi-Cloud:**
- **Modularity:** Reusable modules across environments (dev, staging, prod)
- **Multiple Accounts:** Easier to manage multi-account setups
- **Community:** Huge community, extensive documentation
- **Tooling:** Better IDE support, linting, validation tools
- **State Management:** Explicit and flexible state handling
- **Variables & Outputs:** More powerful than CloudFormation
- **Plan/Apply Workflow:** Better visibility into changes before applying
- **Debugging:** Easier to troubleshoot issues

**Additional Benefits:**
- Non-AWS resources (GitHub, Datadog, PagerDuty) in same codebase
- Consistent workflow regardless of provider
- Better abstractions and conditionals
- Extensive module registry (Terraform Registry)

**When CloudFormation Makes Sense:**
- Specific requirement for AWS-native tooling
- Need immediate access to newly released AWS services
- Team expertise lies heavily in CloudFormation

---

## Load Balancers & API Gateway

### Q23: What are the different types of load balancers in AWS?

**Answer:**

**1. Application Load Balancer (ALB)**
- **OSI Layer:** Layer 7 (Application Layer)
- **Use Cases:**
  - HTTP/HTTPS traffic
  - Path-based routing (/api, /images)
  - Host-based routing (api.example.com, www.example.com)
  - Web applications
  - Microservices architectures
- **Features:**
  - WebSocket support
  - HTTP/2 support
  - Native authentication support
  - Integration with AWS WAF

**2. Network Load Balancer (NLB)**
- **OSI Layer:** Layer 4 (Transport Layer)
- **Use Cases:**
  - Ultra-low latency requirements (milliseconds matter)
  - High-throughput workloads
  - TCP/UDP traffic
  - Trading platforms
  - Gaming applications
  - IoT applications
- **Features:**
  - Static IP address support (Elastic IP)
  - Preserves source IP
  - Millions of requests per second
  - Connection draining

**3. Gateway Load Balancer (GWLB)**
- **OSI Layer:** Layer 3 (Network Layer) + Layer 4
- **Use Cases:**
  - Security appliances (firewalls, IDS/IPS)
  - Financial sector with strict authorization requirements
  - Third-party virtual appliances
  - Traffic inspection before reaching application
- **Features:**
  - Integration with API Gateway
  - Integration with AWS WAF
  - Deploy and scale third-party appliances
  - Transparent to source and destination

**4. Classic Load Balancer (CLB)** - Deprecated
- Legacy load balancer
- Being phased out
- Not recommended for new applications

---

### Q24: When would you use API Gateway versus Application Load Balancer?

**Answer:**

**Use Application Load Balancer When:**
- Simple routing within VPC
- Internal APIs (private communication)
- Path-based or host-based routing sufficient
- Direct integration with ECS/EKS/EC2
- Cost optimization for VPC-internal traffic
- Lower latency for internal communications

**Use API Gateway When:**
- **External APIs** requiring authentication
- **Third-party integrations** (public APIs)
- **Security features needed:**
  - AWS WAF integration (DDoS protection)
  - TLS/SSL by default
  - API keys and usage plans
  - Request/response transformation
  - Request throttling
- **Additional features:**
  - Request validation
  - Canary deployments
  - Stage management (dev, staging, prod)
  - SDK generation
  - API versioning
  - Detailed CloudWatch metrics per API
  - Custom domain names with SSL
  - Response caching
  - CORS configuration

**Decision Matrix:**
- Internal VPC traffic → ALB
- Public APIs with authentication → API Gateway
- Simple HTTP routing → ALB
- Rate limiting and throttling needed → API Gateway
- Cost-sensitive internal traffic → ALB
- Complex API management requirements → API Gateway

---

## Troubleshooting

### Q25: Users report application failure. What are your troubleshooting steps?

**Answer:**

**Step 1: Identify What Changed**
- Check deployment pipelines - was any code deployed?
- Review infrastructure changes - any Terraform/CloudFormation runs?
- Check AWS console for manual changes (CloudTrail)
- Review recent configuration updates
- **If recent deployment found:** Consider immediate rollback to stabilize

**Step 2: Check Application Layer (EKS/ECS)**
- Verify pods/tasks are running: `kubectl get pods`
- Check pod status: CrashLoopBackOff, ImagePullBackOff, Pending?
- Review application logs: `kubectl logs <pod-name>`
- Check resource utilization (CPU, memory)
- Verify deployment/replicaset status
- Check for recent restarts

**Step 3: Check Load Balancer & Health Checks**
- Review ALB target group health
- Check health check configuration
- Verify listener rules and target groups
- Review ALB access logs
- Check security groups on ALB

**Step 4: Check Database Connectivity**
- Verify RDS/database instances are running
- Check database connection pool
- Review database performance metrics
- Check for locks or slow queries
- Verify security group rules allow traffic

**Step 5: Check Network Connectivity**
- Review VPC Flow Logs
- Verify security group rules
- Check NACL configurations
- Verify DNS resolution (Route 53)
- Check NAT gateway functionality

**Step 6: Review Monitoring & Observability**
- CloudWatch metrics for all components
- Custom Prometheus/Grafana dashboards
- Check for alerts that may have been triggered
- Review distributed tracing (X-Ray if enabled)

**Priority: Stabilize First**
1. **Immediate action:** Rollback to last known good state
2. **Stabilize:** Ensure application is serving users
3. **Investigate:** Detailed root cause analysis
4. **Fix:** Address the actual issue
5. **Test:** Verify fix in lower environments (dev, staging)
6. **Deploy:** Promote to production with monitoring

**Communication:**
- Notify stakeholders immediately
- Provide regular updates
- Document incident and resolution in post-mortem

---

## Docker

### Q26: What are Docker best practices for image creation?

**Answer:**

**1. Multi-Stage Builds**
```dockerfile
# Build stage
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM node:16-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/main.js"]
```
- Separates build dependencies from runtime
- Reduces final image size significantly
- Only includes necessary artifacts

**2. Use Lightweight Base Images**
- Prefer `alpine` variants (e.g., `node:16-alpine`)
- Smaller attack surface
- Faster pulls and deployments
- **Caution:** Use specific tags, not `latest`

**3. Specific Image Tags**
- ❌ Don't use: `FROM node:alpine` or `FROM node:latest`
- ✅ Use: `FROM node:16.20-alpine3.18`
- Ensures reproducibility
- Prevents unexpected breaking changes
- Allows controlled upgrades

**4. Non-Root User**
```dockerfile
# Create application user
RUN addgroup -g 1001 -S appuser && \
    adduser -S appuser -u 1001 -G appuser

# Change ownership
RUN chown -R appuser:appuser /app

# Switch to non-root user
USER appuser
```
- Security best practice
- Limits damage if container is compromised
- Required for many security policies

**5. Optimize Layer Caching**
```dockerfile
# Copy dependency files first
COPY package*.json ./
RUN npm install

# Copy source code after
COPY . .
```
- Leverage Docker build cache
- Faster subsequent builds
- Copy frequently changing files last

**6. Documentation**
- Document each command's purpose
- Explain non-obvious configuration choices
- Include maintainer information
- Document exposed ports and volumes

**7. .dockerignore File**
```
node_modules
.git
.env
*.md
.dockerignore
Dockerfile
```
- Reduces build context size
- Faster builds
- Prevents sensitive files from being included

**8. Minimize Layers**
- Combine related RUN commands with `&&`
- Clean up in same layer as installation
```dockerfile
RUN apt-get update && \
    apt-get install -y package && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

---

### Q27: How do you control ECR image storage to avoid cost issues?

**Answer:**

**Problem:**
- Developers push multiple images for testing
- Images accumulate over time
- ECR storage costs increase significantly
- Orphaned images remain indefinitely

**Solution 1: Lambda-based Cleanup**
Create Lambda functions that:
- **Trigger:** Daily on CloudWatch Events/EventBridge schedule
- **Logic:**
  - List all images in ECR repository
  - Keep latest 5-7 images based on push date
  - Delete older images
  - Identify untagged images and remove them
- **Benefits:** 
  - Automated cleanup
  - Customizable retention logic
  - Can extend to other resources

**Solution 2: ECR Lifecycle Policies**
```json
{
  "rules": [
    {
      "rulePriority": 1,
      "description": "Keep last 7 images",
      "selection": {
        "tagStatus": "any",
        "countType": "imageCountMoreThan",
        "countNumber": 7
      },
      "action": {
        "type": "expire"
      }
    }
  ]
}
```
- **Native ECR feature**
- No custom code needed
- Multiple rule support (tagged vs untagged)

**Solution 3: Automated Resource Management**
Don't limit cleanup to just ECR, also automate:
- **EC2 instances:** Stop/terminate unused instances at night
- **RDS instances:** Scale down or stop non-production databases during off-hours
- **EBS volumes:** Delete unattached volumes
- **Old snapshots:** Cleanup based on retention policy
- **S3 lifecycle policies:** Move to cheaper storage tiers

**Implementation:**
- Combine Lambda functions for various resource types
- Use tags to identify resources safe to clean up
- Implement allowlist for critical resources
- Send notifications before cleanup (Slack/SNS)

---

### Q28: How do you ensure Docker images work the same locally and on AWS?

**Answer:**

**Common Issues:**

**1. Port Conflicts**
- **Problem:** Port already in use on EC2/EKS but works locally
- **Solution:**
  - Standardize port assignments
  - Use environment variables for port configuration
  - Check security group rules allow traffic on required ports
  - Verify service mesh/ingress configurations

**2. Image Tag Mismatch**
- **Problem:** Pushing image with wrong tag, pulling different version
- **Solution:**
  - Use CI/CD pipeline to manage tags consistently
  - Never use `latest` in production
  - Use semantic versioning or commit SHAs
  - Implement image digest verification

**3. Environment Variables**
- **Problem:** Different env vars locally vs production
- **Solution:**
  - Use `.env` files for local development
  - Kubernetes Secrets/ConfigMaps for production
  - AWS Secrets Manager for sensitive values
  - Document all required environment variables

**4. Security Group & Network Rules**
- **Problem:** Connectivity works locally but not on AWS
- **Solution:**
  - Verify security groups allow required ports
  - Check NACL rules
  - Verify VPC endpoints if using PrivateLink
  - Test network connectivity separately

**5. Resource Limits**
- **Problem:** Works locally but OOMKilled on EKS
- **Solution:**
  - Set appropriate resource requests/limits
  - Monitor actual usage patterns
  - Right-size based on metrics

**Best Practices:**
- **Consistent deployment method:** Use same docker-compose or Kubernetes manifests locally
- **Document assumptions:** Platform-specific configurations
- **Peer review:** Have another engineer verify deployment works
- **Integration tests:** Automated tests in CI/CD pipeline
- **Staging environment:** Mirror production closely for testing

---

### Q29: How do you handle security scanning of Docker images?

**Answer:**

**CI/CD Pipeline Integration:**

**Pipeline Stages:**
1. **Code Checkout:** Developer pushes code to GitHub
2. **Code Quality:** SonarQube scans source code
3. **Build Image:** Docker build creates container image
4. **Security Scan:** Trivy/Aqua scans image for vulnerabilities
5. **Report Generation:** Generate and display scan results
6. **Gate:** Only push to ECR if scan passes acceptable threshold
7. **Deploy:** Proceed with deployment if approved

**Scanning Tools:**

**Trivy (Most Common)**
```bash
# Scan image
trivy image --severity HIGH,CRITICAL myimage:tag

# Output as JSON
trivy image --format json --output results.json myimage:tag

# Fail build on vulnerabilities
trivy image --exit-code 1 --severity CRITICAL myimage:tag
```
- Open source
- Fast scanning
- Detects vulnerabilities, secrets, IaC issues
- CI/CD friendly

**Alternative Tools:**
- **Aqua Security**
- **Snyk Container**
- **AWS ECR Image Scanning** (native)
- **Clair**

**Best Practices:**

**1. Local Scanning First**
- Developers should scan locally before pushing
- Saves CI/CD pipeline costs
- Faster feedback loop
- Reduces failed builds

**2. Multiple Scan Points**
- Developer workstation (optional but recommended)
- CI/CD pipeline (mandatory)
- ECR automatic scanning (additional layer)
- Runtime scanning (for drift detection)

**3. Vulnerability Management**
- Set severity thresholds (block on CRITICAL, warn on HIGH)
- Regular base image updates
- Vulnerability tracking and remediation workflow
- False positive management

**4. Reporting**
- Generate HTML/JSON reports
- Store in S3 for compliance
- Integrate with security dashboard
- Alert security team on critical findings

**Developer Workflow:**
Despite pipeline checks, encourage developers to:
```bash
# Before commit
trivy image myimage:tag

# Or during development
docker build -t myimage:test .
trivy image myimage:test
```
This prevents multiple failed pipeline runs and reduces costs.

---

### Q30: How do you reduce Docker image size?

**Answer:**

**1. Multi-Stage Builds**
- Build in one stage with all dev dependencies
- Copy only runtime artifacts to final stage
- Can reduce image size by 70-90%

**2. Use Lightweight Base Images**
- Alpine Linux based images (5MB vs 100MB+)
- Distroless images (Google)
- Scratch (for compiled languages like Go)

**Example comparison:**
- `node:16` → ~900MB
- `node:16-slim` → ~180MB
- `node:16-alpine` → ~110MB

**3. Minimize Layers & Clean Up in Same Layer**
```dockerfile
# ❌ Bad - creates multiple layers
RUN apt-get update
RUN apt-get install -y package
RUN apt-get clean

# ✅ Good - single layer, cleaned up
RUN apt-get update && \
    apt-get install -y package && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

**4. .dockerignore File**
- Exclude unnecessary files from build context
- Prevents copying large files (node_modules, .git)

**5. Remove Build Dependencies**
```dockerfile
RUN apk add --no-cache --virtual .build-deps gcc musl-dev && \
    pip install package && \
    apk del .build-deps
```

**6. Copy Only What's Needed**
```dockerfile
# ❌ Copies everything
COPY . .

# ✅ Selective copying
COPY package*.json ./
COPY src/ ./src/
```

**7. Optimize Dependencies**
- Use `--production` flag for npm/yarn
- Remove dev dependencies
- Audit for unused dependencies

**Measurement:**
```bash
docker images myimage:tag
docker history myimage:tag
```

---

## Kubernetes

### Q31: How do you update EKS clusters with zero downtime?

**Answer:**

**Prerequisites & Planning:**

**1. Preparation**
- Create detailed runbook documenting all steps
- Review Kubernetes version release notes
- Check for deprecated APIs
- Verify application compatibility with new K8s version
- Ensure all deployments use proper liveness/readiness probes
- Have Terraform/IaC ready for quick rollback
- Take backups using Velero or similar tool

**2. Testing Strategy**
- Test upgrade process in dev environment first
- Then staging environment (production-like)
- Only proceed to production after successful lower env upgrades
- Document any issues and solutions

**Upgrade Process:**

**Step 1: Control Plane Upgrade**
- EKS manages control plane upgrade automatically
- Minimal downtime for control plane (managed by AWS)
- Worker nodes continue serving traffic during this

**Step 2: Node Group Upgrade (Zero Downtime)**
For each node:
1. **Cordon the node**
   ```bash
   kubectl cordon <node-name>
   ```
   - Marks node as unschedulable
   - Prevents new pods from being scheduled

2. **Drain the node**
   ```bash
   kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
   ```
   - Gracefully evicts pods
   - Pods are rescheduled on other nodes
   - Respects PodDisruptionBudgets

3. **Upgrade the node**
   - Update AMI or node group
   - Launch new node with updated version

4. **Verify node health**
   ```bash
   kubectl get nodes
   kubectl describe node <node-name>
   ```

5. **Uncordon the node**
   ```bash
   kubectl uncordon <node-name>
   ```
   - Allows new pods to be scheduled

6. **Repeat** for all nodes one by one

**Step 3: Add-ons Upgrade**
- Update EKS add-ons (VPC CNI, kube-proxy, CoreDNS)
- Update cluster autoscaler
- Update monitoring tools (Prometheus, Grafana)
- Update ingress controllers

**Key Practices:**
- **PodDisruptionBudgets:** Ensure minimum replicas available
- **Multiple replicas:** Never run single pod of critical apps
- **Health probes:** Proper liveness and readiness checks
- **Graceful shutdown:** Applications handle SIGTERM properly
- **Rolling strategy:** Update one node at a time
- **Monitoring:** Watch metrics throughout upgrade

---

### Q32: What if the EKS upgrade fails?

**Answer:**

**Immediate Response:**

**1. Troubleshoot the Issue**
- Check which component failed (control plane, nodes, add-ons, application)
- Review Kubernetes events: `kubectl get events --all-namespaces`
- Check pod status: `kubectl get pods --all-namespaces`
- Identify if it's API deprecation issue
- Check if specific service/application is causing problem
- Review CloudWatch logs for EKS

**2. Assess Impact**
- Is the application completely down or partially working?
- Are new deployments affected while existing ones work?
- Is it a specific service or entire cluster?

**3. Quick Fix Attempts**
- If small service issue, try to fix/rollback that service
- Update deployment manifests for API version compatibility
- Temporarily disable problematic service if non-critical

**Rollback Strategy:**

**Important:** You cannot downgrade EKS cluster or node versions!

**If Application Breaking:**
1. **Restore from Backup**
   - Use Velero to restore cluster state
   - Restore namespaces, configmaps, secrets from backup

2. **Spin Up New Cluster**
   - Create new EKS cluster with old version (from backup)
   - Deploy applications using IaC/GitOps
   - Restore data from backups
   - Test functionality

3. **Traffic Migration**
   - Update Route 53 or load balancer
   - Switch traffic to new (old-version) cluster
   - Verify users can access application

4. **Stabilize First, Debug Later**
   - Priority: Get users back online
   - Then investigate root cause
   - Fix issues in lower environments
   - Plan retry with comprehensive testing

**Prevention:**
- Thorough testing in non-production
- Gradual rollout strategy
- Have rollback cluster ready (blue-green at cluster level)
- Comprehensive monitoring during upgrade
- Clear rollback criteria defined upfront

---

### Q33: Give an example of stateful application on Kubernetes.

**Answer:**

**Use Case: MongoDB on EKS**

**Why StatefulSet:**
- MongoDB requires persistent storage
- Each replica needs stable identity
- Data must persist across pod restarts
- Ordered deployment and scaling needed

**Configuration:**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongodb
spec:
  serviceName: mongodb
  replicas: 3
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo:5.0
        ports:
        - containerPort: 27017
        volumeMounts:
        - name: mongodb-data
          mountPath: /data/db
  volumeClaimTemplates:
  - metadata:
      name: mongodb-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: gp3
      resources:
        requests:
          storage: 10Gi
```

**Components:**
- **StatefulSet:** Manages MongoDB pods
- **PersistentVolume:** EBS volumes for data storage
- **PersistentVolumeClaim:** Dynamically provisioned per pod
- **Headless Service:** Stable network identity for each pod

**Why Not Use AWS DocumentDB:**
- Was a PoC requirement to test everything on EKS
- Wanted to evaluate self-managed vs managed service
- Testing disaster recovery scenarios
- **Note:** For production, AWS DocumentDB (managed service) is recommended

**Best Practice Opinion:**
- Kubernetes/EKS should primarily be stateless
- Stateful workloads better suited for managed services:
  - RDS for relational databases
  - DocumentDB for MongoDB
  - ElastiCache for Redis
  - MSK for Kafka
- Managed services provide:
  - Better reliability
  - Automated backups
  - Easier scaling
  - Less operational overhead

---

### Q34: Does Kubernetes support blue-green deployment? How would you configure it?

**Answer:**

**Yes, Kubernetes supports blue-green deployment.**

**Method 1: Using Service Label Selectors**

**Setup:**
1. Deploy new version (green) alongside old version (blue)
2. Use same namespace or separate namespaces
3. Both deployments run simultaneously

```yaml
# Blue Deployment (current version)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: blue
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
      - name: myapp
        image: myapp:v1.0
```

```yaml
# Green Deployment (new version)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: green
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
      - name: myapp
        image: myapp:v2.0
```

**Traffic Switching via Ingress:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/canary: "false"
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-blue  # Change to myapp-green for cutover
            port:
              number: 80
```

**Cutover Process:**
1. Deploy green version (new)
2. Test green deployment internally
3. Update Ingress to point to green service
4. Monitor for issues
5. Keep blue running for quick rollback
6. After validation, remove blue deployment

**Method 2: Using Argo Rollouts (Recommended)**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp
spec:
  replicas: 3
  strategy:
    blueGreen:
      activeService: myapp-active
      previewService: myapp-preview
      autoPromotionEnabled: false
      scaleDownDelaySeconds: 30
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:v2.0
```

**Argo Rollouts Benefits:**
- Built-in blue-green strategy
- Automated rollback on failure
- Progressive delivery
- Metrics-based promotion
- Better visualization
- Canary deployments support

**Traffic Management:**
- Use Ingress annotations to control traffic percentage
- Can send 100% to blue, then instant switch to 100% green
- Or use canary approach (10%, 25%, 50%, 100%)

---

### Q35: List Kubernetes best practices.

**Answer:**

**Resource Management:**

**1. Define Resource Requests and Limits**
```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```
- Ensures proper scheduling
- Prevents resource contention
- Enables cluster autoscaling
- Protects against noisy neighbors

**Configuration Management:**

**2. Use ConfigMaps and Secrets**
```yaml
# Don't hardcode
env:
  - name: DATABASE_URL
    value: "postgres://..." # ❌ Bad

# Use ConfigMap/Secret
env:
  - name: DATABASE_URL
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: url # ✅ Good
```
- Separation of configuration from code
- Environment-specific configs
- Easier updates without rebuilding images

**Monitoring & Observability:**

**3. Implement Comprehensive Monitoring**
- **CloudWatch:** Basic AWS metrics
- **Prometheus + Grafana:**
  - Custom application metrics
  - Kubernetes cluster metrics
  - Pod and node statistics
  - Easy deployment via Helm
- **Custom Dashboards:** Visualize key metrics
- **PromQL Queries:** Create specific metric queries

**4. Set Up Alerts**
```yaml
# Example Prometheus alert
- alert: HighNodeCPU
  expr: node_cpu_usage > 80
  for: 5m
  annotations:
    summary: "Node CPU usage above 80%"
```
- CPU/memory thresholds
- Pod restart counts
- Failed deployments
- Integration with Slack/PagerDuty/SNS

**Security:**

**5. RBAC (Role-Based Access Control)**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```
- Principle of least privilege
- Separate roles for teams
- Audit access regularly

**6. Pod Security**
- Run as non-root user
- Read-only root filesystem where possible
- Drop unnecessary capabilities
- Use Pod Security Policies/Standards

**7. Network Policies**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-allow
spec:
  podSelector:
    matchLabels:
      app: api
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```
- Control pod-to-pod communication
- Implement zero-trust networking

**8. IRSA (IAM Roles for Service Accounts)**
- Grant AWS permissions to specific pods
- No need for node-level permissions
- Better audit trail
- Principle of least privilege

**High Availability:**

**9. Multiple Replicas**
```yaml
spec:
  replicas: 3  # Minimum for HA
```
- Never run single instance of critical apps
- Distribute across availability zones

**10. PodDisruptionBudgets**
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: myapp
```
- Ensures minimum availability during disruptions
- Protects during node maintenance

**11. Health Probes**
```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```
- Liveness: Restart unhealthy pods
- Readiness: Control traffic routing

**Deployment Strategy:**

**12. Rolling Updates**
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```
- Zero-downtime deployments
- Gradual rollout

**Cost Optimization:**

**13. Cluster Autoscaler**
- Automatically scale nodes based on demand
- Set appropriate min/max node counts
- Combine with Horizontal Pod Autoscaler

**14. Resource Optimization**
- Right-size requests/limits based on actual usage
- Use VPA (Vertical Pod Autoscaler) for recommendations
- Consider spot instances for non-critical workloads

---

### Q36: Are Kubernetes secrets really secure? How do you make them more secure?

**Answer:**

**Problem with K8s Secrets:**
- Secrets are **Base64 encoded**, not encrypted
- Anyone with cluster access can decode:
  ```bash
  kubectl get secret mysecret -o jsonpath='{.data.password}' | base64 -d
  ```
- Stored in etcd (may or may not be encrypted)
- Visible to anyone with kubectl access

**Security Layers:**

**Layer 1: RBAC (Basic Protection)**
```yaml
# Restrict access to secrets
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: no-secrets
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list"]
# Explicitly NOT allowing secrets access
```
- Limit who can view secrets
- Only give access to necessary personnel (DevOps leads, managers)
- **Limitation:** Still can be decoded by authorized users

**Layer 2: External Secrets Manager (Recommended)**

**AWS Secrets Manager Integration:**

**Option A: External Secrets Operator**
```yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secrets-manager
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-east-1
      auth:
        serviceAccount:
          name: external-secrets-sa

---
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: database-secret
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
  target:
    name: db-credentials
  data:
  - secretKey: password
    remoteRef:
      key: prod/database/password
```

**Option B: Secrets Store CSI Driver**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  serviceAccountName: myapp-sa
  containers:
  - name: myapp
    image: myapp:latest
    volumeMounts:
    - name: secrets-store
      mountPath: "/mnt/secrets"
      readOnly: true
  volumes:
  - name: secrets-store
    csi:
      driver: secrets-store.csi.k8s.io
      readOnly: true
      volumeAttributes:
        secretProviderClass: "aws-secrets"
```

**Benefits:**
- Secrets never stored in Kubernetes
- Centralized secret management
- Automatic rotation support
- Audit trail in AWS CloudTrail
- Encryption at rest in AWS
- Fine-grained IAM permissions

**Layer 3: Encryption at Rest**
- Enable etcd encryption in EKS
- Encrypt using AWS KMS
- Protects secrets if etcd backup is compromised

**Layer 4: IRSA (IAM Roles for Service Accounts)**
- Pods can directly access AWS Secrets Manager
- No need to store AWS credentials in K8s
- Each pod gets only required permissions

**Best Practice Recommendation:**
1. **Never** store sensitive data directly in K8s secrets for production
2. **Always** use external secrets manager (AWS Secrets Manager, HashiCorp Vault, etc.)
3. Combine with RBAC for defense in depth
4. Rotate secrets regularly
5. Audit secret access

---

### Q37: What troubleshooting scenarios have you faced in Kubernetes?

**Answer:**

**Common Errors:**

**1. CrashLoopBackOff**
- **Cause:** Application crashes repeatedly after starting
- **Troubleshooting:**
  ```bash
  kubectl describe pod <pod-name>
  kubectl logs <pod-name>
  kubectl logs <pod-name> --previous  # Previous container logs
  ```
- **Common Reasons:**
  - Application error in code
  - Missing environment variables
  - Cannot connect to database
  - Wrong configuration in ConfigMap
  - Insufficient resources
- **Resolution:** Fix application code or configuration

**2. ImagePullBackOff**
- **Cause:** Cannot pull container image
- **Troubleshooting:**
  ```bash
  kubectl describe pod <pod-name>  # Check Events section
  ```
- **Common Reasons:**
  - Image doesn't exist (typo in image name/tag)
  - Registry authentication failure (ECR permissions)
  - Network issues reaching registry
  - Rate limiting (Docker Hub)
- **Resolution:**
  - Verify image name and tag
  - Check ECR permissions/IRSA
  - Create imagePullSecret if needed

**3. Pending State**
- **Cause:** Pod cannot be scheduled to any node
- **Troubleshooting:**
  ```bash
  kubectl describe pod <pod-name>
  kubectl get nodes
  kubectl describe node <node-name>
  ```
- **Common Reasons:**
  - Insufficient resources (CPU/memory) on nodes
  - Node selector/affinity rules not matching
  - Taints on nodes without tolerations
  - PersistentVolumeClaim not bound
  - Resource quotas exceeded
- **Resolution:**
  - Scale up cluster (add nodes)
  - Adjust resource requests
  - Fix node selectors/taints
  - Check PVC status

**4. Service Not Reachable**
- **Cause:** Cannot access application via Service
- **Troubleshooting:**
  ```bash
  kubectl get svc
  kubectl get endpoints <service-name>
  kubectl describe svc <service-name>
  ```
- **Common Reasons:**
  - Label selector mismatch
  - No pods matching selector
  - Pods not passing readiness probe
  - Wrong service port configuration
- **Resolution:**
  - Verify labels on pods and service selector
  - Check readiness probe configuration
  - Verify container port matches service targetPort

**5. OOMKilled (Out of Memory)**
- **Cause:** Pod exceeds memory limits
- **Troubleshooting:**
  ```bash
  kubectl describe pod <pod-name>  # Check Last State
  kubectl top pod <pod-name>
  ```
- **Resolution:**
  - Increase memory limits
  - Fix memory leak in application
  - Optimize application memory usage

**6. PersistentVolumeClaim Not Binding**
- **Cause:** PVC stuck in Pending state
- **Troubleshooting:**
  ```bash
  kubectl describe pvc <pvc-name>
  kubectl get pv
  kubectl get storageclass
  ```
- **Common Reasons:**
  - No matching PersistentVolume
  - StorageClass doesn't exist
  - Insufficient storage quota
  - Access mode mismatch
- **Resolution:**
  - Verify StorageClass exists and is correct
  - Check dynamic provisioning is working
  - Verify sufficient storage quota

**Debugging Tools:**
```bash
# Get into running container
kubectl exec -it <pod-name> -- /bin/sh

# Port forward for local testing
kubectl port-forward <pod-name> 8080:80

# View resource usage
kubectl top nodes
kubectl top pods

# Check cluster events
kubectl get events --sort-by='.lastTimestamp'
```

---

## Summary

This interview covered comprehensive DevOps topics including:

- ✅ AWS architecture for high availability and fault tolerance
- ✅ Container orchestration (ECS vs EKS)
- ✅ Networking (VPC, hybrid connectivity, load balancers)
- ✅ Disaster recovery and global architecture
- ✅ Security best practices across all layers
- ✅ Cost optimization strategies
- ✅ Infrastructure as Code (Terraform vs CloudFormation)
- ✅ Docker best practices and security
- ✅ Kubernetes operations and troubleshooting
- ✅ Blue-green deployments
- ✅ Monitoring and observability

**Candidate Strengths:**
- Strong hands-on experience with AWS and Kubernetes
- Good understanding of security principles
- Practical knowledge of CI/CD pipelines
- Experience with infrastructure automation
- Problem-solving approach to troubleshooting

**Areas for Improvement:**
- Deeper understanding of managed services vs self-hosted trade-offs
- More experience with cost analysis tools
- Enhanced knowledge of specific AWS service configurations
- Documentation and runbook creation practices
