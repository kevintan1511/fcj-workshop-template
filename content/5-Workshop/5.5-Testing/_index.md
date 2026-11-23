---
title: "Testing and Troubleshooting"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

# Testing and Troubleshooting

## Functional tests
| Test | How | Result |
| --- | --- | --- |
| Valid image | Upload a beer image via the website | `APPROVED`, labels Alcohol · Beer · Bottle…, shown in the gallery |
| Violating image | Upload the wooden toy gun | `REJECTED`, `blockedCategories = Violence`, the admin gets an email |
| Real user | Check DynamoDB | `userId` is the Cognito `sub` UUID, not an email |
| Idempotency | Overwrite the same key | Log `item_missing_or_already_processed`, status unchanged |
| Unknown key | Upload a file with no PENDING item | No new item, no email |
| DLQ | Set a wrong `TABLE_NAME`, invoke asynchronously | After 2 retries the DLQ holds a `RetriesExhausted` message, `approximateInvokeCount: 3` |
> 📷 **Screenshot placeholder:** *Gallery: APPROVED and REJECTED images*

**False positive:** a wooden toy gun is detected as *Violence/Weapons* with ~97% confidence. Improvements: raise `MIN_MODERATION_CONFIDENCE` for specific categories, or add an admin review step for `REJECTED` images.
The DLQ test also proves **least privilege**: the error in the DLQ is `AccessDeniedException … not authorized to perform: dynamodb:UpdateItem on resource: …table/khong-ton-tai`. IAM blocks it immediately because the role can only write to `imageguard-images`.
> 📷 **Screenshot placeholder:** *DLQ message: RetriesExhausted*

## Security tests
| Test | Expected | Actual |
| --- | --- | --- |
| `GET /images` without a token | 401 Unauthorized | ✅ `{"message":"Unauthorized"}` |
| Open a web-bucket file directly | 403 AccessDenied | ✅ only CloudFront can read it (OAC) |
| Open an image Object URL directly | 403 AccessDenied | ✅ |
| Open a presigned URL after 6 minutes | Request has expired | ✅ |
| `http://` | Redirects to `https://` | ✅ |
| Response headers | HSTS, X-Frame-Options, X-Content-Type-Options, Referrer-Policy | ✅ |
| Sign-in | URL contains `code_challenge_method=S256` (PKCE) | ✅ |
> 📷 **Screenshot placeholder:** *API without a token returns 401*

> 📷 **Screenshot placeholder:** *Direct S3 access is denied*

The site is served from CloudFront’s **SGN (Ho Chi Minh City)** edge, so Vietnamese users get fast loads even though the backend runs in us-east-1.
## Troubleshooting
### 1. Inline policy: “The policy failed legacy parsing”
**Cause:** a stray `>` after the Account ID in the DynamoDB ARN (`…:123456789012>:table/…`), so the Review page listed only 5 of 6 services. **Fix:** the Account ID must be 12 digits with no dashes and no `<` `>`; use Find & Replace on `<ACCOUNT_ID>` before pasting.
### 2. Alert email arrived but the item stayed PENDING
**Cause:** at upload time the `img-002` item did not match, so the conditional `UpdateItem` was rejected; the first version of the code **published to SNS before** writing the DB, so the email still went out. **Lesson:** that design let “unknown” uploads spam the admin. **Fixed:** publish to SNS only after the DB stores `REJECTED`; if SNS fails, log an ERROR and emit `AlertPublishFailed` so an alarm catches it.
> 📷 **Screenshot placeholder:** *Email arrived while the item was still PENDING*

### 3. Alarms sent no email
**Symptom:** the alarm went to ALARM but no email arrived; the alarm history said *“Failed to execute action”*. **Cause:** the SNS topic was encrypted with the AWS managed key `alias/aws/sns`. CloudWatch Alarms cannot use that key (Lambda can, so REJECTED emails still arrived). **Fix:** disable topic encryption → history shows *“Successfully executed action”*. If encryption is mandatory, use a customer managed KMS key whose key policy allows `cloudwatch.amazonaws.com` (~USD 1/month). The email only contains `userId`/`imageId`, no image or image link.
> 📷 **Screenshot placeholder:** *Failed to execute action*

> 📷 **Screenshot placeholder:** *Successfully executed action after the fix*

### 4. The Console differs from older docs
- New Lambda console: the role is under **Custom settings → Additional settings → Custom execution role**.
- New CloudWatch console: **Log groups** is now **Logs → Log Management**; Logs Insights is **Log Analytics**.
- A log stream shows only the last few lines: click **Load more** to see the `image_processed` line.
### 5. Amazon Bedrock was not available
The original idea used Amazon Nova Lite on Bedrock, but every team account had a quota of 0 and the increase request was denied. The team switched to **Amazon Rekognition**: no quota request needed, a ready-made moderation API, and per-image pricing that is easy to estimate.
