---
title: "Week 5 Worklog"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 Objectives:

* Connect to EC2 without open SSH ports via Session Manager
* Define infrastructure as code with CloudFormation
* Automate off-hours shutdown with Lambda and Slack alerts
* Set up single sign-on with IAM Identity Center
* Apply guardrails with IAM Permission Boundaries

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | ---- | ---------- | --------------- | ------------------ |
| 2 | - Attach an instance profile with SSM permissions<br>- Connect via Session Manager<br>- Run remote commands with Run Command | 12/10/2026 | 12/10/2026 | <https://000058.awsfcaj.com/> |
| 3 | - Learn template structure: Resources, Parameters, Outputs<br>- Write a YAML template for VPC, subnets and EC2<br>- Create and update stacks via Console and CLI | 13/10/2026 | 13/10/2026 | <https://000037.awsfcaj.com/> |
| 4 | - Write a Python Lambda to stop running EC2 instances<br>- Trigger it on a schedule with EventBridge<br>- Send run status to Slack | 14/10/2026 | 14/10/2026 | <https://000022.awsfcaj.com/> |
| 5 | - Set up IAM Identity Center directory<br>- Create users and assign account permissions<br>- Test single sign-on to the Console | 15/10/2026 | 15/10/2026 | <https://000012.awsfcaj.com/> |
| 6 | - Create and test Permission Boundaries on test users<br>- Confirm delegated users cannot exceed their limits<br>- Watch FCAJ YouTube lessons 111-135 | 16/10/2026 | 16/10/2026 | <https://000030.awsfcaj.com/><br><https://www.youtube.com/@AWSStudyGroup> |

### Week 5 Achievements:

* Managed EC2 without SSH keys using Session Manager
* Provisioned infrastructure with CloudFormation
* Built an automated EC2 shutdown with Lambda and EventBridge
* Configured IAM Identity Center for central login
* Enforced guardrails with Permission Boundaries

### AWS services used:

* AWS Systems Manager
* AWS CloudFormation
* AWS Lambda
* Amazon EventBridge
* IAM Identity Center
* AWS IAM

### Skills gained:

* Secure remote management without public ports
* Infrastructure as Code with YAML
* Serverless automation and scheduling
* Access governance with SSO and boundaries
