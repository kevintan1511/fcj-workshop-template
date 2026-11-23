---
title: "Worklog Tuần 4"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu Tuần 4:

* Connect local storage to S3 with Storage Gateway
* Deliver content with low latency using CloudFront
* Connect VPCs with VPC Peering and Transit Gateway
* Manage resources and costs with tags and Resource Groups

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --------- | ------------ | --------------- | ------------------- |
| 2 | - Deploy Storage Gateway in File Gateway mode<br>- Mount local shares backed by S3<br>- Verify file sync to the S3 bucket | 05/10/2026 | 05/10/2026 | <https://000024.awsfcaj.com/> |
| 3 | - Create a CloudFront distribution for an S3 website<br>- Configure an SSL certificate with ACM<br>- Test caching and edge delivery speed | 06/10/2026 | 06/10/2026 | <https://000094.awsfcaj.com/> |
| 4 | - Connect two VPCs with VPC Peering<br>- Update route tables and security groups<br>- Test private traffic between EC2 instances | 07/10/2026 | 07/10/2026 | <https://000019.awsfcaj.com/> |
| 5 | - Set up Transit Gateway to link multiple VPCs<br>- Configure route tables and attachments<br>- Compare Transit Gateway with VPC Peering | 08/10/2026 | 08/10/2026 | <https://000020.awsfcaj.com/> |
| 6 | - Apply tags to EC2, S3 and RDS<br>- Group tagged resources with Resource Groups<br>- Enable cost allocation tags<br>- Watch FCAJ YouTube lessons 81-110 | 09/10/2026 | 09/10/2026 | <https://000027.awsfcaj.com/><br><https://000028.awsfcaj.com/><br><https://www.youtube.com/@AWSStudyGroup> |

### Kết quả đạt được Tuần 4:

* Linked local file storage to S3 via Storage Gateway
* Sped up content delivery with CloudFront
* Built private cross-VPC networking with VPC Peering
* Centralized VPC connections with Transit Gateway
* Organized resources with tags and Resource Groups

### Các dịch vụ AWS đã sử dụng:

* AWS Storage Gateway
* Amazon CloudFront
* AWS Certificate Manager
* Amazon VPC Peering
* AWS Transit Gateway
* AWS Resource Groups

### Kỹ năng đạt được:

* Hybrid cloud storage and file sync
* CDN distribution and edge caching
* Inter-VPC routing
* Resource tagging and grouping strategy
