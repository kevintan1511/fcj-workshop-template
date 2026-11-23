---
title: "Worklog Tuần 7"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu Tuần 7:

* Set up shared Windows file storage with Amazon FSx
* Build an S3 data lake with lifecycle tiering
* Design DynamoDB tables and indexes
* Analyze cost savings with Savings Plans and Reserved Instances
* Right-size EC2 with Compute Optimizer

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --------- | ------------ | --------------- | ------------------- |
| 2 | - Provision FSx for Windows File Server in a private subnet<br>- Join it to Active Directory and mount on EC2<br>- Test file sharing and daily backups | 26/10/2026 | 26/10/2026 | <https://000025.awsfcaj.com/> |
| 3 | - Create raw, processed and curated bucket structure<br>- Add lifecycle rules to move old data to Glacier<br>- Enable versioning and default encryption | 27/10/2026 | 27/10/2026 | <https://000035.awsfcaj.com/> |
| 4 | - Create a DynamoDB table with partition and sort keys<br>- Add and query items in Console and CLI<br>- Create a global secondary index | 28/10/2026 | 28/10/2026 | <https://000039.awsfcaj.com/> |
| 5 | - Review Cost Explorer Savings Plans recommendations<br>- Compare On-Demand and Reserved Instances<br>- Estimate 1-year vs 3-year commitment impact | 29/10/2026 | 29/10/2026 | <https://000029.awsfcaj.com/> |
| 6 | - Run Compute Optimizer on underused EC2 instances<br>- Right-size a test instance<br>- Watch FCAJ YouTube lessons 161-185 | 30/10/2026 | 30/10/2026 | <https://000032.awsfcaj.com/><br><https://www.youtube.com/@AWSStudyGroup> |

### Kết quả đạt được Tuần 7:

* Provisioned managed Windows file storage with FSx
* Organized S3 data lake tiers with lifecycle archiving
* Designed and queried DynamoDB tables and indexes
* Analyzed savings with Savings Plans and RIs
* Right-sized EC2 using Compute Optimizer

### Các dịch vụ AWS đã sử dụng:

* Amazon FSx
* Amazon S3
* Amazon DynamoDB
* AWS Cost Explorer
* AWS Compute Optimizer

### Kỹ năng đạt được:

* Shared file systems on AWS
* S3 tiering and lifecycle management
* NoSQL data modeling
* Cost analysis and right-sizing
