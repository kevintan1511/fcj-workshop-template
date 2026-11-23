---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# ImageGuard – Automated Image Moderation and Labeling on AWS Serverless
This workshop builds **ImageGuard** from scratch: users sign in and upload images; the system uses **Amazon Rekognition** to moderate content (violence, explicit content, drugs, hate symbols…), approved images are labeled automatically, violating images are blocked and an administrator receives an email alert.
The whole system is **serverless**, runs in **us-east-1**, and costs about **USD 8.37/month** for 3,000 images.
| Item | Details |
| --- | --- |
| **Duration** | 3–4 hours |
| **Level** | 200 – Intermediate |
| **Main services** | Amazon S3, AWS Lambda, Amazon Rekognition, Amazon DynamoDB, Amazon API Gateway, Amazon Cognito, Amazon CloudFront, Amazon SNS, Amazon SQS, Amazon CloudWatch, AWS IAM |
| **Source code** | [static/code/imageguard](https://github.com/Pham-Nhut-An/fcj-workshop/tree/main/static/code/imageguard) |
1. [Workshop Overview](5.1-workshop-overview/)
2. [Prerequisites](5.2-prerequisites/)
3. [Architecture](5.3-architecture/)
4. [Step-by-step Lab Guide](5.4-lab-guide/)
5. [Testing and Troubleshooting](5.5-testing/)
6. [Cost](5.6-cost/)
7. [Clean-up](5.7-cleanup/)
8. [Demo Video](5.8-demo-video/)
