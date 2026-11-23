---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# ImageGuard – AI-Powered Image Moderation and Labeling on AWS Serverless
{{% notice info %}}
**Status:** Proposal v2.0.
{{% /notice %}}

## 1. Overview
**ImageGuard** is a serverless web application for sites where people post pictures, such as classified-ad boards, forums and student clubs. It **checks every uploaded image automatically and tags it with searchable labels**. As soon as a photo arrives, **Amazon Rekognition**, a ready-made computer-vision service, runs two checks:
1. **Moderation:** looks for unsafe content such as violence, explicit material, weapons or drugs. A flagged image is stored as `REJECTED` and the administrator receives an email.
2. **Labeling:** for images that pass, it produces descriptive labels (for example *Laptop*, *Bicycle*, *Book*), so owners can **find their pictures by keyword**.
Outcomes are saved in **Amazon DynamoDB** and appear in the web interface within a few seconds. The design is **event-driven** and **serverless**: there is nothing to patch or keep running, and the bill follows real usage.
## 2. Problem Statement
- Sites with user uploads collect a steady flow of pictures. **Checking them by hand** is slow and costly, and moderators end up looking at harmful material.
- Without a filter, unsafe pictures can go public and damage both users and the platform’s name.
- Most uploads carry **no caption or tags**, so older pictures are almost impossible to find again.
**Our answer:** let AI do both jobs the moment a picture is uploaded. Unsafe images are stopped before anyone sees them, and safe images are labeled so they can be searched.
### 2.1. Who Uses ImageGuard
| Role | What they do | What they get |
| --- | --- | --- |
| Uploader | Signs in, uploads pictures, searches their own images | Instant feedback and automatic labels |
| Administrator | Receives email alerts | Only flagged images need attention |
| Development team | Builds and operates the system | A low-cost design with no servers to maintain |
## 3. Objectives & Scope
### 3.1. Objectives
1. Deliver an automatic flow, **Upload → AI moderation → AI labeling → Storage → Display/Search**, built wholly on AWS.
2. Follow the **AWS Well-Architected Framework**, with extra care for the **Security** pillar (least-privilege IAM, nothing public, no credentials in code).
3. Give the team full visibility through **Amazon CloudWatch** (logs, metrics, alarms) and **Amazon SNS** email alerts.
4. Hold the running cost near **USD 6–7 per month** for the expected workload (Section 6).
5. Produce a **bilingual, step-by-step guide** so anyone can rebuild the system.
### 3.2. Success Criteria
| Criterion | Target |
| --- | --- |
| Time from upload to visible result | ≤ 10 seconds |
| Violating images blocked on the test set | ≥ 90% |
| Valid images blocked by mistake (false positives) on the test set | ≤ 10% |
| Images processed without manual help (none stuck in PENDING or FAILED) | ≥ 95% |
| API call without a valid sign-in | Rejected (HTTP 401) |
| One user reading another user’s images or results | Impossible |
| Violating image found | Email alert reaches the administrator |
| Direct Internet access to the S3 buckets | Blocked (Block Public Access) |
| Monthly cost for the expected workload | ≤ USD 10 |
### 3.3. Scope
| In scope | Out of scope |
| --- | --- |
| JPG and PNG images up to 5 MB | Video and animated GIFs |
| Content moderation and automatic labeling | **Face recognition** (biometric and privacy-sensitive) |
| Searching by label inside a user’s own images | Searching across all users’ images |
| Sign-up and sign-in with Amazon Cognito | A dedicated admin console and appeals against decisions |
| One Region: **us-east-1** | Multi-Region deployment and disaster recovery |
### 3.4. Assumptions and Constraints
- The system runs in a single AWS account covered by credits, in Region **us-east-1**.
- Testing uses a small set of the team’s own or freely licensed images, mixing normal and borderline cases.
- The target is a small community, about 3,000 images per month, not internet scale.
- Moderation uses a **60%** confidence threshold and labeling uses **80%**; both can be tuned.
## 4. Solution Architecture
![ImageGuard architecture diagram](../images/2-proposal/imageguard-architecture.drawio.png)
*Figure 1: ImageGuard architecture diagram, drawn with AWS Architecture Icons.*

### 4.1. AWS Services and Rationale
| Service | Role in the system | Why it was chosen |
| --- | --- | --- |
| **Amazon S3** | Holds uploaded images in a private bucket and also hosts the static website | Extremely durable and cheap; its **S3 Events** start the processing |
| **Amazon CloudFront** | Delivers the site over HTTPS and reaches S3 through **Origin Access Control (OAC)** | Keeps the website bucket private; HTTPS at no extra charge; inside the Always Free allowance (1 TB/month) |
| **Amazon Cognito** (User Pool) | Handles sign-up and sign-in and issues **JWT tokens** | Managed identity, free up to 10,000 MAU, and the app never touches passwords |
| **Amazon API Gateway** (HTTP API) | Exposes the endpoints the web app calls, guarded by a Cognito **JWT authorizer** | About USD 1 per million requests, much cheaper than REST APIs, with built-in Cognito support |
| **AWS Lambda** (Python 3.12) | Runs three functions: `presign-upload`, `process-image` and `get-images` | No servers, billed per call, scales on its own |
| **Amazon Rekognition** | `DetectModerationLabels` for safety checks and `DetectLabels` for tagging | Ready-made AI with no training; reads pictures straight from S3; available on the Free plan |
| **Amazon DynamoDB** (On-demand) | Keeps each user’s moderation results and labels | Serverless NoSQL, pay per request, fast, 25 GB free |
| **Amazon SQS** | Acts as the **Dead-Letter Queue (DLQ)** for events that keep failing | Nothing is lost, and failed events can be replayed |
| **Amazon CloudWatch** | Collects logs and metrics, runs alarms, shows the dashboard | One place to watch everything, as the workshop requires |
| **Amazon SNS** | Emails the administrator about rejected images and fired alarms | Simple setup; 1,000 emails per month free |
| **AWS IAM** | Gives every Lambda its own least-privilege role | The security base of the whole design |
**Why us-east-1:** it is usually the cheapest Region and offers every service we need. The extra ~200 ms of latency from Vietnam hardly matters, because the website is served from CloudFront edge locations (including Hanoi and Ho Chi Minh City) and AI analysis is the slowest step anyway. For real users in Vietnam the team would move to Singapore (ap-southeast-1) for lower latency and personal-data rules; Region and resource names are environment variables, so the code stays the same.
### 4.2. Data Flow
The step numbers below match the arrows in Figure 1.
1. **HTTPS (static web):** the user opens the site through **CloudFront**, which reads the private `imageguard-web` bucket via OAC.
2. **Sign in → JWT:** the user signs in with the **Cognito User Pool** and receives a JWT token.
3. **POST /upload-url:** the web app calls **API Gateway** with the token, and the JWT authorizer checks it.
  - **3a. Invoke:** API Gateway calls the `presign-upload` Lambda.
  - **3b. PutItem (PENDING):** the Lambda writes a `PENDING` record to DynamoDB and returns a **presigned URL**, valid for 5 minutes, for the key `uploads/{userId}/{imageId}.jpg`.
4. **PUT image:** the browser uploads the file **straight to S3** (`imageguard-images`) using that URL, so it never passes through Lambda or API Gateway.
5. **S3 Event (ObjectCreated):** a new object under the `uploads/` prefix triggers the `process-image` Lambda asynchronously.
6. **DetectModerationLabels + DetectLabels:** `process-image` asks **Rekognition** to moderate the image and, if it is clean, to label it.
  - **6a. Read image:** Rekognition reads the picture directly from S3.
  - A moderation label in a blocked category with confidence ≥ 60% means `REJECTED`; otherwise up to 10 labels with confidence ≥ 80% are kept.
7. **UpdateItem (APPROVED / REJECTED):** the result, reason and labels are written to DynamoDB.
8. **Publish alert if REJECTED:** an email goes to the administrator through the **SNS** topic `imageguard-alerts`.
9. **On failure (after 2 retries):** the event is sent to the **SQS DLQ** so it can be replayed.
10. **GET /images (JWT):** the web app asks for the user’s images.
  - **10a. Invoke:** API Gateway calls the `get-images` Lambda.
  - **10b. Query by userId:** the Lambda queries DynamoDB with the `userId` taken from the token, filters by label, and returns short-lived **presigned GET URLs**.
11. **Logs & Metrics (all Lambdas):** every function reports to **CloudWatch**.
12. **Alarm:** a CloudWatch alarm notifies the administrator through SNS.
Throughout, each Lambda runs under its own **IAM execution role** with least privilege (red dashed line in the diagram).
### 4.3. Security Design (Security Pillar)
| Principle | How ImageGuard applies it |
| --- | --- |
| **Least privilege** | Each Lambda has its own role. `process-image` is limited to `s3:GetObject` on `uploads/*`, the two Rekognition actions `rekognition:DetectModerationLabels` and `rekognition:DetectLabels`, `dynamodb:UpdateItem` on **one** table and `sns:Publish` on **one** topic. `presign-upload` may only use `s3:PutObject` on `uploads/*` and `dynamodb:PutItem`. `get-images` may only use `dynamodb:Query` and `s3:GetObject` |
| **Rekognition limitation** | Rekognition’s image APIs **cannot be restricted to a resource**, so `"Resource": "*"` is unavoidable. To compensate, only **the two needed actions** are allowed, never `rekognition:*` |
| **Nothing public** | Both buckets use **Block Public Access**. Only CloudFront can read the website bucket (OAC). The image bucket is reachable only through short-lived presigned URLs |
| **Sign-in and data isolation** | Every API call needs a Cognito JWT. `userId` always comes from the token’s `sub` claim and **never** from the request, so users cannot see each other’s data |
| **No hard-coded credentials** | Lambdas use IAM roles with temporary credentials. Bucket, table and topic names are **environment variables**. The GitHub repository holds no access keys and includes a `.gitignore` |
| **Encryption** | S3 and DynamoDB encrypt data at rest by default, and all traffic uses HTTPS/TLS |
| **Input checks** | Presigned URLs fix the image `Content-Type`, and the Lambda verifies size (≤ 5 MB) and format before calling the AI |
| **Privacy** | No face recognition and no biometric data are stored |
> **Why are the Lambdas outside a VPC?** Nothing in this design needs a private network: the functions reach S3, DynamoDB, Rekognition and SNS over HTTPS, with access controlled by IAM. A VPC would add NAT Gateway or interface endpoint fees (tens of USD a month) with little security benefit here. The team keeps it as a future option if compliance requires it.
### 4.4. Mapping to the AWS Well-Architected Framework
| Pillar | How it is applied |
| --- | --- |
| Operational Excellence | Logs kept 14 days, a dashboard, alarms and a guide that can be repeated |
| Security | See Section 4.3 |
| Reliability | Event-driven flow, two automatic Lambda retries, a DLQ for failed events and explicit `PENDING/APPROVED/REJECTED/FAILED` states |
| Performance Efficiency | Uploads go directly to S3, Rekognition reads from S3 (the Lambda never downloads images), and DynamoDB is queried by key |
| Cost Optimization | Fully serverless; **`DetectLabels` runs only for images that pass moderation**; S3 Lifecycle, 14-day logs and AWS Budgets |
| Sustainability | No idle resources; compute runs only when an image arrives |
### 4.5. Why Amazon Rekognition
The first plan used **Amazon Bedrock** to process invoices. While testing, the team found its account had a Bedrock quota of 0, and the request to raise it was declined. The team moved to **Amazon Rekognition**, a ready-made AI service that works on the current account and was verified in the console, and reused most of the serverless design. The episode is written up in the Troubleshooting part of the workshop.
### 4.6. Monitoring and Alerting Plan
| Signal | Threshold | Action |
| --- | --- | --- |
| `process-image` Lambda errors | ≥ 1 in 5 minutes | CloudWatch alarm, email through SNS |
| Messages visible in the SQS DLQ | ≥ 1 | Email through SNS, replay the failed event |
| API Gateway 5XX responses | ≥ 5 in 5 minutes | Email through SNS, check the Lambda logs |
| `process-image` duration (p95) | > 8 seconds | Investigate, because the end-to-end target is 10 seconds |
| Custom metric: rejected images | Any increase | Review on the CloudWatch dashboard |
| AWS Budgets | 80% of USD 10 | Email, pause testing and review usage |
### 4.7. Moderation Policy
The proposed default policy uses Rekognition’s top-level moderation categories. The list lives in an environment variable, so the team can change it without editing code.
| Category | Default action |
| --- | --- |
| Explicit | `REJECTED` and email the administrator |
| Violence, Visually Disturbing, Hate Symbols | `REJECTED` and email the administrator |
| Drugs & Tobacco | `REJECTED` and email the administrator |
| Alcohol, Gambling, Rude Gestures | Allowed, but the label is stored for review |
## 5. Plan & Timeline
Internship period: 14/09/2026 – 04/12/2026. The workshop is due on **19/11/2026**, which leaves 15 days before the internship certificate is needed.
### 5.1. Roles
| Member | Main responsibility |
| --- | --- |
| Member A | Team lead, integration, IAM and security |
| Member B | `process-image` Lambda, Rekognition integration, test images and accuracy evaluation |
| Member C | Cognito, API Gateway and the `presign-upload` and `get-images` Lambdas |
| Member D | Static web UI, S3 and CloudFront (OAC) |
| Member E | CloudWatch, SNS, DLQ, cost estimate and clean-up guide |
*Every member should be able to explain the whole architecture, not just their own part.*
### 5.2. Weekly Timeline
| Internship week | Dates | Tasks | Output |
| --- | --- | --- | --- |
| Week 7 | 26/10 – 01/11 | Settle the topic and write proposal v1.0; try Bedrock and find the zero quota | Proposal v1.0, incident notes |
| Week 8 | 02/11 – 08/11 | Ask for a quota increase (declined); switch to Rekognition and write proposal v2.0; create S3, DynamoDB and IAM roles; build `process-image` with its S3 trigger | Proposal v2.0, working image pipeline |
| Week 9 | 09/11 – 15/11 | Cognito, API Gateway and the two API Lambdas; static site and CloudFront; CloudWatch metrics, alarms and dashboard; SNS and DLQ; testing; AWS Pricing Calculator; demo video | End-to-end demo, video, cost table |
| Week 10 | 16/11 – 19/11 | Write the bilingual workshop guide (steps, testing, clean-up), take screenshots, finish the diagram, peer review | **Workshop submitted on 19/11** |
| Week 10–12 | 20/11 – 04/12 | Apply mentor feedback, finish blogs and worklogs, clean up resources | Final report |
## 6. Cost Estimation
**Workload scenario:** a small forum or classified-ad site with about **3,000 images per month** (~100 a day, 1 MB each, about 3% violating) and roughly 30,000 views and searches. Region **us-east-1**, on-demand prices, **before Free Tier and credits**.
| Service | Assumptions | Estimated cost (USD/month) |
| --- | --- | --- |
| Amazon Rekognition | 3,000 `DetectModerationLabels` + ~2,910 `DetectLabels` calls × USD 0.001 per image | 5.91 |
| AWS Lambda | 36,000 invocations, ~3,200 GB-seconds | 0.06 (within the free tier) |
| Amazon API Gateway (HTTP API) | 33,000 requests | 0.03 |
| Amazon DynamoDB (On-demand) | ~6,000 write and 30,000 read units, under 1 GB | 0.01 |
| Amazon S3 | 3 GB (30-day lifecycle), 3,000 PUT, 6,000 GET | 0.09 |
| Amazon CloudWatch | 0.5 GB logs, 3 alarms, 3 custom metrics, 1 dashboard | 0.27 (mostly within the free tier) |
| Amazon CloudFront | ~2 GB transfer, 40,000 requests | 0.00 (Always Free) |
| Amazon Cognito | Under 10,000 MAU | 0.00 |
| Amazon SNS, Amazon SQS | About 100 emails or messages | 0.00 |
| **Total** | | **≈ USD 6.40/month (≈ USD 76/year)** |
During development and the demo (under 300 images), Rekognition costs about USD 0.60 and the account’s credits cover everything.
*AWS Pricing Calculator link:* `[to be added after the estimate is created]`
### Cost Optimization Strategy
- **Fully serverless:** no EC2, NAT Gateway or other always-on resources.
- **Moderate first, label second:** rejected images skip `DetectLabels`, saving one AI call each.
- Rekognition **reads images directly from S3**, so each Lambda run is short (~2 seconds) and needs only 256 MB.
- An **S3 Lifecycle rule** deletes images after 30 days (or archives them to S3 Glacier if they must be kept).
- **CloudWatch Logs are kept 14 days** instead of the default “Never expire”.
- An **HTTP API** instead of a REST API (about 70% cheaper).
- An **AWS Budgets** alert at USD 10, and **clean-up** right after grading.
- Beyond 1 million images per month, Rekognition’s tiered pricing lowers the per-image price automatically.
## 7. Risks & Mitigation
| Risk | Impact | Mitigation |
| --- | --- | --- |
| Rekognition blocks good images or misses bad ones | Frustrated users, or harmful content slips through | Thresholds and blocked categories are environment variables that can be tuned; accuracy is measured on the test set |
| Quotas on new accounts (already hit with Bedrock) | AI calls fail | Rekognition confirmed working on the account; retry with backoff when throttled |
| S3 → Lambda → S3 loop | Runaway cost | The trigger covers only `uploads/`, and the Lambda never writes back to S3 |
| Access keys pushed to GitHub | Account takeover | IAM roles only, a `.gitignore`, and a review before each commit |
| Test images that infringe copyright or are too disturbing | Legal and ethical problems | Use the team’s own or freely licensed photos; test with mild categories (alcohol, tobacco, toy weapons) |
| Unexpected charges | Loss of credits | AWS Budgets and a clean-up checklist |
| Abuse of the upload endpoint (spam, oversized files) | Cost spikes and noisy data | Presigned URLs expire in 5 minutes and fix type and size; API Gateway throttling; per-user limits |
| A user disputes a rejection | Loss of trust | Each `REJECTED` record keeps the reason and confidence; an admin review flow is listed as a future improvement |
| Very little time (~2 weeks) | Missed deadline | Build the MVP first and extras later; every week has a clear deliverable |
| Members still new to AWS | Delays and unanswered mentor questions | Learn while building, hold peer reviews, and have everyone explain the full design |
## 8. Expected Outcomes
- A working web app: sign in, upload an image, and get either a block with an email alert or a labeled, searchable picture.
- Monitoring in place, with a CloudWatch dashboard, alarms and email alerts.
- A bilingual workshop guide that others can follow from scratch, covering testing, troubleshooting and clean-up.
- New skills for the team: event-driven serverless design, least-privilege IAM, AWS AI services, monitoring, cost control, and adapting when a service is quota-limited.
## 9. Future Improvements
- An admin review page and an appeal flow for rejected images.
- Amazon Rekognition Custom Labels for categories specific to the platform.
- AWS WAF in front of CloudFront to block abusive traffic.
- AWS X-Ray tracing to spot slow steps in the pipeline.
- Infrastructure as Code (AWS SAM or CloudFormation) so the whole stack deploys with one command.
- A move to Singapore (ap-southeast-1) for lower latency and local data requirements.
