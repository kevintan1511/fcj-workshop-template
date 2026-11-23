---
title: "Hướng dẫn triển khai từng bước"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

# Hướng dẫn triển khai từng bước

Full source code: [static/code/imageguard](https://github.com/Pham-Nhut-An/fcj-workshop/tree/main/static/code/imageguard). Replace `<ACCOUNT_ID>` with your Account ID (12 digits, no dashes).
## Step 1 – Foundation resources
### 1.1. Image bucket
**S3 → Create bucket** → `imageguard-images-<ACCOUNT_ID>` · ACLs disabled · **Block all public access** · Versioning disabled · **SSE-S3**.
- **Management → Lifecycle rule** `delete-uploads-30d`: prefix `uploads/`, expire after **30 days**.
- **Permissions → Bucket policy**: deny any request without HTTPS (`aws:SecureTransport = false`).
> 📷 **Screenshot placeholder:** *S3 Block public access and bucket policy*

### 1.2. DynamoDB
**Create table** `imageguard-images` · Partition key `userId` (String) · Sort key `imageId` (String) · **On-demand**.
### 1.3. SNS and SQS
- SNS **Standard** topic `imageguard-alerts`, **encryption off** (see [Troubleshooting](#s5-5)), Email subscription → click **Confirm subscription** in the email.
- SQS **Standard** `imageguard-process-dlq`, **14-day** retention, default SSE-SQS.
### 1.4. IAM role for `process-image`
**IAM → Roles → Create role** → AWS service **Lambda** → **attach no managed policy** → `imageguard-process-image-role` → **Create inline policy** (JSON):
```bash
{
  "Version": "2012-10-17",
  "Statement": [
    {"Sid": "ReadUploadedImages", "Effect": "Allow", "Action": "s3:GetObject", "Resource": "arn:aws:s3:::imageguard-images-<ACCOUNT_ID>/uploads/*"},
    {"Sid": "AnalyzeImagesWithRekognition", "Effect": "Allow", "Action": ["rekognition:DetectModerationLabels", "rekognition:DetectLabels"], "Resource": "*"},
    {"Sid": "UpdateImageResults", "Effect": "Allow", "Action": "dynamodb:UpdateItem", "Resource": "arn:aws:dynamodb:us-east-1:<ACCOUNT_ID>:table/imageguard-images"},
    {"Sid": "SendModerationAlerts", "Effect": "Allow", "Action": "sns:Publish", "Resource": "arn:aws:sns:us-east-1:<ACCOUNT_ID>:imageguard-alerts"},
    {"Sid": "SendFailedEventsToDLQ", "Effect": "Allow", "Action": "sqs:SendMessage", "Resource": "arn:aws:sqs:us-east-1:<ACCOUNT_ID>:imageguard-process-dlq"},
    {"Sid": "WriteOwnLogsOnly", "Effect": "Allow", "Action": ["logs:CreateLogGroup", "logs:CreateLogStream", "logs:PutLogEvents"], "Resource": ["arn:aws:logs:us-east-1:<ACCOUNT_ID>:log-group:/aws/lambda/imageguard-process-image", "arn:aws:logs:us-east-1:<ACCOUNT_ID>:log-group:/aws/lambda/imageguard-process-image:*"]}
  ]
}
```

The Review page must list **6 services**. If it says *“The policy failed legacy parsing”*, check the Account ID (see [Testing and Troubleshooting](#s5-5)).
> 📷 **Screenshot placeholder:** *The role has a single inline policy*

## Step 2 – `process-image` Lambda and S3 Event
### 2.1. Create the function
**Lambda → Create function → Author from scratch** · name `imageguard-process-image` · **Python 3.12** · **Additional settings**: ARM64 **on**, Custom execution role **on** → `imageguard-process-image-role`, Function URL **off**.
> 📷 **Screenshot placeholder:** *Creating the process-image Lambda*

Paste `process_image/lambda_function.py` → **Deploy**. Memory **256 MB**, timeout **30 seconds**.
| Environment variable | Value |
| --- | --- |
| `TABLE_NAME` | `imageguard-images` |
| `BUCKET_NAME` | `imageguard-images-<ACCOUNT_ID>` |
| `TOPIC_ARN` | `arn:aws:sns:us-east-1:<ACCOUNT_ID>:imageguard-alerts` |
| `BLOCKED_CATEGORIES` | `Explicit,Violence,Visually Disturbing,Drugs & Tobacco,Hate Symbols` |
| `MIN_MODERATION_CONFIDENCE` | `60` |
| `MAX_LABELS` · `MIN_LABEL_CONFIDENCE` | `10` · `80` |
### 2.2. Key ideas in the code
```bash
# Exponential-backoff retries for throttling / 5xx
BOTO_CONFIG = Config(retries={"max_attempts": 5, "mode": "standard"})

# Accept only well-formed keys -> blocks unknown keys and path traversal
KEY_PATTERN = re.compile(r"^uploads/([A-Za-z0-9-]{1,64})/([A-Za-z0-9-]{1,64})\\.(jpg|jpeg|png)$")

# Conditional UpdateItem: the item must exist and still be PENDING
ConditionExpression="attribute_exists(#uid) AND #st = :pending"

# Publish to SNS only after the DB update succeeded
if blocked and updated:
    publish_alert(user_id, image_id, key, blocked)
```

Metrics use the **Embedded Metric Format**: one JSON log line with an `_aws` key, and CloudWatch creates `ImagesProcessed`, `ImagesApproved`, `ImagesRejected`, `ProcessingLatency`, `AlertPublishFailed` (namespace `ImageGuard`). No `cloudwatch:PutMetricData` permission is needed.
### 2.3. Retries, DLQ and trigger
- **Configuration → Asynchronous invocation**: maximum age 1 hour, retry attempts **2**.
- **Destinations → On failure** → SQS `imageguard-process-dlq`.
- **Add trigger → S3**: image bucket, event **PUT**, prefix `uploads/`, suffix `.jpg`.
> 📷 **Screenshot placeholder:** *Function overview: S3 → Lambda → SQS*

## Step 3 – Cognito, API Gateway, static website
Step 3 can be done in the Console in the order below, or by running [`setup.sh`](https://github.com/Pham-Nhut-An/fcj-workshop/tree/main/static/code/imageguard/scripts/setup.sh) in **CloudShell**, which creates everything with the AWS CLI (about 3 minutes).
> 📷 **Screenshot placeholder:** *Output of setup.sh in CloudShell*

### 3.1. Two API Lambdas
Create 2 dedicated roles (least privilege), then 2 Python 3.12 arm64 functions, 128 MB, 10 seconds:
| Lambda | Inline policy permissions | Environment variables |
| --- | --- | --- |
| `imageguard-presign-upload` | `s3:PutObject` (`uploads/*`), `dynamodb:PutItem`, logs | `BUCKET_NAME`, `TABLE_NAME`, `URL_EXPIRES_SECONDS=300` |
| `imageguard-get-images` | `dynamodb:Query`, `s3:GetObject` (`uploads/*`), logs | `BUCKET_NAME`, `TABLE_NAME`, `URL_EXPIRES_SECONDS=300`, `MAX_ITEMS=50` |
`presign-upload` reads `userId` from `requestContext.authorizer.jwt.claims.sub`, generates `imageId` (UUID), creates the `PENDING` item, then signs a presigned PUT URL. The URL is signed with the Lambda’s own permissions, so the role needs `s3:PutObject`.
### 3.2. Static website + CloudFront
1. Bucket `imageguard-web-<ACCOUNT_ID>` (private, SSE-S3); upload `index.html`, `app.js`, `config.js`.
2. **CloudFront → Create distribution**: S3 web bucket origin with **OAC**, Redirect HTTP to HTTPS, CachingOptimized, response headers **SecurityHeadersPolicy**, default root object `index.html`, WAF off.
3. Bucket policy allows only `cloudfront.amazonaws.com` with `AWS:SourceArn` = the distribution ARN.
> 📷 **Screenshot placeholder:** *ImageGuard served through CloudFront*

### 3.3. Cognito
User pool `imageguard-users`, **email** sign-in, self sign-up enabled. App client **SPA without a client secret**: Authorization code grant, scopes `openid email`, callback and sign-out URL = `https://<CLOUDFRONT_DOMAIN>/`. Create a domain (Managed login).
> 📷 **Screenshot placeholder:** *Cognito Managed Login page*

### 3.4. API Gateway HTTP API
- Routes `POST /upload-url` → `presign-upload`, `GET /images` → `get-images`, stage `$default` with auto-deploy.
- **JWT Authorizer** `cognito-jwt`: issuer `https://cognito-idp.us-east-1.amazonaws.com/<USER_POOL_ID>`, audience `<APP_CLIENT_ID>`, attached to both routes.
- **CORS**: only origin `https://<CLOUDFRONT_DOMAIN>`, headers `authorization, content-type`, methods `GET, POST, OPTIONS`.
- **Throttling** on the default route: rate 10, burst 20.
### 3.5. Wire everything together
- CORS on the image bucket: allow `PUT, GET` from the CloudFront domain.
- Fill in `config.js` (Cognito domain, client ID, redirect URI, API URL), upload it again, create a CloudFront invalidation for `/*`.
## Step 4 – Monitoring
Run [`step4.sh`](https://github.com/Pham-Nhut-An/fcj-workshop/tree/main/static/code/imageguard/scripts/step4.sh) in CloudShell, or create these in the Console:
| Alarm | Metric | Threshold (5 minutes) |
| --- | --- | --- |
| `imageguard-{process-image,presign-upload,get-images}-errors` | AWS/Lambda Errors | ≥ 1 |
| `imageguard-process-image-throttles` | AWS/Lambda Throttles | ≥ 1 |
| `imageguard-dlq-not-empty` | AWS/SQS ApproximateNumberOfMessagesVisible | ≥ 1 |
| `imageguard-alert-publish-failed` | ImageGuard AlertPublishFailed | ≥ 1 |
| `imageguard-api-5xx` | AWS/ApiGateway 5xx | ≥ 5 |
Every alarm notifies the `imageguard-alerts` SNS topic on both **ALARM** and **OK**. The `imageguard` dashboard shows APPROVED/REJECTED counts, processing time, Lambda, API Gateway, the DLQ and alarm status. Log retention is **14 days** for all 3 Lambdas.
> 📷 **Screenshot placeholder:** *imageguard CloudWatch dashboard*

