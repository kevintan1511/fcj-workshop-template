---
title: "Worklog Tuần 10"
date: 2024-01-01
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

{{% notice info %}}
**Tuần kế hoạch** — Các công việc và kết quả trong Tuần 10 là kế hoạch dự kiến.
{{% /notice %}}

### Mục tiêu Tuần 10:

* Learn Kubernetes basics and create an Amazon EKS cluster
* Build CI/CD pipelines with CodePipeline
* Run serverless containers on AWS Fargate
* Deploy image updates to EKS from GitHub
* Connect a serverless frontend to API Gateway and Lambda

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --------- | ------------ | --------------- | ------------------- |
| 2 | - Learn Kubernetes concepts on EKS<br>- Create a cluster with eksctl<br>- Deploy a sample Nginx pod and service with kubectl | 16/11/2026 | 16/11/2026 | <https://000126.awsfcaj.com/> |
| 3 | - Create a CodePipeline with a GitHub source<br>- Add a CodeBuild step for tests and artifacts<br>- Test the build on commit | 17/11/2026 | 17/11/2026 | <https://000017.awsfcaj.com/> |
| 4 | - Compare ECS EC2 launch type and Fargate<br>- Create a Fargate task definition<br>- Run a container without managing hosts | 18/11/2026 | 18/11/2026 | <https://000067.awsfcaj.com/> |
| 5 | - Build a CI/CD workflow to update the image on EKS<br>- Connect GitHub to deploy updated pods<br>- Verify zero-downtime rolling replacement | 19/11/2026 | 19/11/2026 | <https://000062.awsfcaj.com/> |
| 6 | - Build an HTML/JS frontend on S3<br>- Create a REST API in API Gateway that triggers Lambda<br>- Connect frontend requests to the API<br>- Watch FCAJ YouTube lessons 236-260 | 20/11/2026 | 20/11/2026 | <https://000079.awsfcaj.com/><br><https://www.youtube.com/@AWSStudyGroup> |

### Dự kiến kết quả đạt được Tuần 10 (kế hoạch):

* Create an EKS cluster and deploy a sample workload
* Automate builds and tests with CodePipeline
* Run serverless containers on Fargate
* Set up deployment from GitHub to EKS
* Connect a frontend to API Gateway and Lambda

### Các dịch vụ AWS dự kiến sử dụng:

* Amazon EKS
* AWS CodePipeline and CodeBuild
* AWS Fargate
* Amazon API Gateway
* Amazon S3

### Kỹ năng dự kiến đạt được:

* Kubernetes fundamentals on EKS
* Continuous integration pipelines
* Serverless container execution
* Frontend-to-API serverless integration
