# DevOps Interview Questions

## AWS Architecture & Services

1. How would you architect a fully available and fault tolerant application on AWS?

2. Why would you use EKS and not any other container service like ECS?

3. Why do you think ECS won't be a good choice for highly available and fault tolerant applications?

4. Can you tell me a scenario where ECS will be better suited as compared to EKS?

5. Which one is cheaper - ECS or EKS? Why?

## AWS Networking

6. What configuration is needed to make a subnet public or private?

7. Is internet gateway attached to a subnet or VPC?

8. Have you worked with hybrid networking in AWS?

9. List down all the choices to connect on-premises network with AWS.

10. Why do you require a transit gateway when we have VPC peering? What problem does transit gateway solve?

11. Why is VPC peering cheaper than transit gateway for connecting only two VPCs?

## Disaster Recovery & Global Architecture

12. How would you design a disaster recovery solution for an e-commerce website where the DR region should be geographically isolated from the primary one?

13. If you want to support an e-commerce website globally (US and India) with minimum latency and separate databases for each region, how will you design the solution?

## Security Best Practices

14. Can you name few security best practices in compute, networking, databases, and storage?

15. For a cost-conscious customer who wants bare minimum security to keep lights on, what practices would you implement?

16. How would you extend database access to a third party without compromising security when the database needs to be in a public subnet?

17. What AWS service can detect vulnerable packages in EC2 instances?

## AWS Well-Architected Framework

18. Can you list down all the pillars of AWS Well-Architected Framework and summarize each one?

## Cross-Account Access

19. How do you facilitate cross-account connectivity (e.g., accessing KMS key or S3 bucket from another account)?

20. While establishing cross-account connectivity, what issues have you faced and how did you resolve them?

## Static Website Hosting

21. What configurations or steps are needed to host a static website on S3 and make it publicly available?

## Cost Optimization

22. As a cloud engineer, what cost optimization solutions would you implement across compute, networking, databases, and storage?

23. When would you use spot instances for batch jobs? What if a batch job needs to run at a specific time (e.g., 11:30 PM)?

24. What if the spot instance type is not available in that region?

## Infrastructure as Code

25. When would you choose CloudFormation versus Terraform, or Terraform versus CloudFormation?

26. If you have dedicated infrastructure only on AWS with no other cloud provider, would you still choose Terraform or CloudFormation?

## Load Balancers & API Gateway

27. What are the different types of load balancers in AWS and under what circumstances would you use each?

28. When would you use API Gateway versus Application Load Balancer?

## Troubleshooting

29. If users report failures in an application deployed on AWS (with Route 53, load balancers, autoscaling groups, EKS/ECS, RDS), what would be your troubleshooting steps?

## Docker

30. What are some Docker best practices you follow for image creation?

31. Where do you host your Docker images and how do you control the number of images to avoid cost issues?

32. How do you ensure that Docker images working on developer machines work the same way on ECS/EKS clusters?

33. How do you handle security scanning of Docker images before pushing to ECR?

34. How do you reduce the overall size of Docker images?

## Kubernetes

35. How do you update EKS clusters with zero downtime?

36. What happens if the EKS upgrade fails?

37. Can you give an example of a stateful application you worked on in Kubernetes?

38. Does Kubernetes support blue-green deployment? If yes, how would you configure it?

39. List down few best practices with respect to Kubernetes (cost, security, etc.)?

40. Are Kubernetes secrets really secure? How do you make database credentials more secure in Kubernetes?

41. What troubleshooting scenarios have you faced in Kubernetes?
