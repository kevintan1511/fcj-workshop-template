---
title: "Week 6 Worklog"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Objectives:

* Assess security posture with Security Hub and AWS WAF
* Manage encryption keys with AWS KMS
* Containerize apps with Docker and Amazon ECR
* Deploy containers on ECS behind a load balancer
* Build a container CI/CD pipeline with CodePipeline

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | ---- | ---------- | --------------- | ------------------ |
| 2 | - Enable Security Hub and review the security score<br>- Fix common findings<br>- Create a WAF web ACL on the load balancer | 19/10/2026 | 19/10/2026 | <https://000018.awsfcaj.com/><br><https://000026.awsfcaj.com/> |
| 3 | - Create customer managed keys in KMS<br>- Configure key policies and automatic rotation<br>- Encrypt S3 buckets and EBS volumes with custom keys | 20/10/2026 | 20/10/2026 | <https://000033.awsfcaj.com/> |
| 4 | - Write a Dockerfile for a Node.js app<br>- Build and run the image locally<br>- Create an ECR repository and push the image | 21/10/2026 | 21/10/2026 | <https://000015.awsfcaj.com/> |
| 5 | - Create an ECS cluster on EC2 launch type<br>- Define a task definition from the ECR image<br>- Deploy an ECS service behind an Application Load Balancer | 22/10/2026 | 22/10/2026 | <https://000016.awsfcaj.com/> |
| 6 | - Set up CodePipeline with CodeBuild<br>- Trigger builds and ECS deployments on Git push<br>- Watch FCAJ YouTube lessons 136-160 | 23/10/2026 | 23/10/2026 | <https://000029.awsfcaj.com/><br><https://www.youtube.com/@AWSStudyGroup> |

### Week 6 Achievements:

* Checked security benchmarks with Security Hub and added WAF rules
* Managed keys and encrypted storage with KMS
* Built Docker images and pushed them to ECR
* Deployed load-balanced containers on ECS
* Created a CI/CD pipeline with CodePipeline

### AWS services used:

* AWS Security Hub
* AWS WAF
* AWS KMS
* Amazon ECR
* Amazon ECS
* CodePipeline and CodeBuild

### Skills gained:

* Security auditing and web firewall protection
* Key management and encryption at rest
* Docker packaging and image registries
* Container orchestration and CI/CD
