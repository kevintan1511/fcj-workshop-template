# Báo cáo thực tập First Cloud Journey - Nguyễn Ngọc Tân
ImageGuard – Kiểm duyệt & Gắn nhãn ảnh tự động trên AWS Serverless
ImageGuard là ứng dụng web serverless theo kiến trúc hướng sự kiện (event-driven) giúp nền tảng rao vặt, diễn đàn hoặc cộng đồng tự động kiểm duyệt nội dung và gắn nhãn phân loại ảnh do người dùng tải lên bằng Amazon Rekognition.

Ảnh hợp lệ (APPROVED): tự động gắn tối đa 10 nhãn mô tả nội dung (ví dụ Laptop, Beer, Bicycle) để tìm kiếm nhanh theo nhãn.
Ảnh vi phạm (REJECTED): chặn hiển thị trên giao diện và gửi email cảnh báo cho quản trị viên qua Amazon SNS (chỉ kèm userId, imageId, s3Key và danh mục vi phạm — không đính kèm link ảnh).
Giao diện Dark Mode AI Security Console & Batch Upload: hỗ trợ chọn/kéo-thả tối đa 10 ảnh JPG/PNG mỗi lượt, tải song song (concurrency = 3) trực tiếp lên S3 kèm thanh tiến độ từng ảnh, lọc theo nhãn/trạng thái đồng bộ URL.
Kiến trúc hệ thống
Sơ đồ kiến trúc ImageGuard

Luồng xử lý chính
Giao diện tĩnh: Người dùng truy cập web qua Amazon CloudFront (HTTPS + Security Headers), đọc từ bucket private imageguard-web-<ACCOUNT_ID> qua Origin Access Control (OAC).
Xác thực: Đăng nhập qua Amazon Cognito Managed Login (OAuth 2.0 Authorization Code + PKCE).
Upload trực tiếp lên S3: Client gọi POST /upload-url (được xác thực bằng Cognito JWT Authorizer trên API Gateway HTTP API). Lambda presign-upload ghi bản ghi PENDING vào DynamoDB và trả về Presigned PUT URL (hết hạn sau 5 phút). Trình duyệt upload ảnh thẳng lên bucket private imageguard-images-<ACCOUNT_ID>.
Kiểm duyệt & Gắn nhãn bất đồng bộ: Sự kiện s3:ObjectCreated (prefix uploads/) kích hoạt Lambda process-image:
Gọi rekognition:DetectModerationLabels trước.
Nếu vi phạm danh mục cấm (Explicit, Violence, Visually Disturbing, Drugs & Tobacco, Hate Symbols với độ tin cậy 
≥
60
): cập nhật DynamoDB REJECTED có điều kiện (attribute_exists(userId) AND status = PENDING) rồi mới gửi cảnh báo qua Amazon SNS (bỏ qua DetectLabels để tiết kiệm chi phí).
Nếu hợp lệ: gọi rekognition:DetectLabels (tối đa 10 nhãn, độ tin cậy 
≥
80
) và cập nhật APPROVED.
Phát metric theo CloudWatch Embedded Metric Format (EMF) (ImagesProcessed, ImagesApproved, ImagesRejected, ProcessingLatency, AlertPublishFailed).
Retry bất đồng bộ 2 lần; nếu vẫn lỗi thì đẩy sự kiện vào SQS Dead-Letter Queue (imageguard-process-dlq).
Tra cứu, Xuất bản & Bảng tin công khai (Kho của tôi ↔ Bảng tin công khai):
GET /images?label=...&status=...: Lambda get-images truy vấn theo userId (claims.sub), lọc theo nhãn/trạng thái, trả thống kê summary (gồm cả số ảnh đã xuất bản) và chỉ ký Presigned GET URL cho ảnh APPROVED.
POST /images/{imageId}/publish: Lambda publish-image khóa điều kiện attribute_exists(userId) AND status = APPROVED — chỉ ảnh đã duyệt mới được phép xuất bản lên sparse GSI PublicFeedIndex (hoặc thu hồi bất cứ lúc nào).
GET /feed?label=...: Lambda get-feed đọc PublicFeedIndex (feedScope = "PUBLIC"), trả ảnh công khai kèm nhãn và tên tác giả đã che email (authorLabel).
Cấu trúc thư mục
AWS_product/
├── backend/
│   ├── presign_upload/lambda_function.py   # POST /upload-url (JPG/PNG -> Presigned PUT URL)
│   ├── process_image/lambda_function.py    # S3 trigger -> Rekognition -> DynamoDB + SNS + EMF
│   ├── get_images/lambda_function.py       # GET /images[?label=...&status=...]
│   ├── publish_image/lambda_function.py    # POST /images/{imageId}/publish (Publish Gate)
│   └── get_feed/lambda_function.py         # GET /feed[?label=...] (Public Feed via GSI)
├── frontend/
│   ├── index.html                          # Giao diện Dark Mode AI Security Console (Batch Upload + Gallery)
│   ├── styles.css                          # CSS dark theme responsive thuần
│   ├── app.js                              # Cognito PKCE + Batch Upload S3 (XHR progress) + API client
│   ├── config.js                           # Cấu hình runtime kết nối Cognito & API Gateway
│   └── config.example.js                   # Mẫu file cấu hình
├── scripts/
│   ├── step1-2-foundation.sh               # Tạo S3, DynamoDB, SNS, SQS DLQ, IAM & process-image
│   ├── step3-api-web.sh                    # Tạo API Lambdas, CloudFront, Cognito, API Gateway & upload web
│   ├── step4-monitoring.sh                 # Tạo 7 CloudWatch Alarms & Dashboard `imageguard`
│   ├── deploy-all.sh                       # Chạy tuần tự Step 1 → 4
│   ├── batch_test.sh                       # Chạy batch test 10 ảnh thực tế qua S3 + Rekognition + DynamoDB
│   └── cleanup.sh                          # Xóa toàn bộ tài nguyên AWS theo đúng thứ tự phụ thuộc
├── tests/
│   ├── mock_aws.py                         # Mock boto3/botocore để test offline
│   ├── test_presign_upload.py
│   ├── test_process_image.py
│   └── test_get_images.py
├── docs/
│   ├── imageguard-architecture.drawio
│   └── imageguard-architecture.drawio.png
├── template.yaml                           # AWS SAM Infrastructure-as-Code đầy đủ
├── samconfig.toml.example                  # Mẫu cấu hình SAM CLI
├── requirements-dev.txt
├── .gitignore
└── README.md
Hướng dẫn chạy & Triển khai
1. Xem trước giao diện cục bộ (Preview UI)
cd frontend
python -m http.server 8080
Mở http://localhost:8080/ trên trình duyệt để xem bố cục Dark Mode và kiểm thử màn hình chính. Khi bấm Đăng nhập, giao diện sẽ nhắc cập nhật thông số AWS thật vào frontend/config.js trước khi kết nối Cognito/Rekognition.

2. Triển khai lên AWS – Cách A: AWS SAM (khuyên dùng)
Yêu cầu: đã cài AWS CLI và AWS SAM CLI, đã cấu hình credentials (aws configure).

cp samconfig.toml.example samconfig.toml
# Chỉnh AdminEmail trong samconfig.toml thành email nhận cảnh báo của bạn
sam build
sam deploy --guided   # hoặc: sam deploy
Sau khi deploy xong, lấy các giá trị từ Outputs của stack (WebUrl, ApiUrl, CognitoDomain, UserPoolClientId, WebBucketName, CloudFrontDistributionId), cập nhật vào frontend/config.js, rồi upload thư mục frontend/ lên WebBucketName:

aws s3 sync frontend/ s3://<WebBucketName>/ --exclude "config.example.js" --delete
aws cloudfront create-invalidation --distribution-id <CloudFrontDistributionId> --paths "/*"
3. Triển khai lên AWS – Cách B: Bash Scripts trong AWS CloudShell
Mở AWS CloudShell tại Region us-east-1, clone/upload thư mục dự án và chạy:

export ADMIN_EMAIL="your-email@example.com"
bash scripts/deploy-all.sh
Script tự động thực hiện toàn bộ Step 1 → Step 4, tự ghi frontend/config.js với thông số thật, upload web lên CloudFront và in thông tin truy cập ra imageguard-outputs.txt. Nhớ mở hộp thư ADMIN_EMAIL để bấm Confirm subscription cho SNS.

Kiểm thử (Unit Tests)
Toàn bộ unit test chạy 100% offline không cần AWS credentials hay cài đặt thêm thư viện ngoài:

python -m unittest discover -s tests -v
Nếu đã cài pytest (pip install -r requirements-dev.txt):

pytest -v
Để kiểm thử độ chính xác thực tế của Amazon Rekognition sau khi đã deploy lên AWS (chạy trong AWS CloudShell):

bash scripts/batch_test.sh
Bảo mật & Well-Architected Framework
IAM Least Privilege: Mỗi Lambda có 1 IAM Role riêng, chỉ cấp đúng action và resource ARN cần dùng (ví dụ presign-upload chỉ có s3:PutObject trên uploads/* và dynamodb:PutItem; get-images chỉ có dynamodb:Query và s3:GetObject).
Không public bucket: Cả imageguard-images-* và imageguard-web-* đều bật Block Public Access và ép buộc HTTPS (aws:SecureTransport).
Cô lập dữ liệu người dùng: userId luôn trích xuất từ JWT claims.sub đã được API Gateway xác thực; client không thể giả mạo userId.
Idempotency & Chống upload ngoài luồng: process-image chỉ xử lý các item đã tồn tại ở trạng thái PENDING, tránh gửi email lặp khi S3 gửi trùng event hoặc khi upload trực tiếp không qua API.
Tối ưu chi phí: 100% serverless (arm64, HTTP API, DynamoDB On-demand), chỉ gọi DetectLabels khi ảnh vượt qua kiểm duyệt, S3 Lifecycle tự xóa sau 30 ngày, CloudWatch Logs giữ 14 ngày (~8–9 USD/tháng với tải 3.000 ảnh/tháng chưa trừ Free Tier).
Dọn dẹp tài nguyên (Clean-up)
Nếu triển khai bằng Bash Scripts:
bash scripts/cleanup.sh
Nếu triển khai bằng AWS SAM: làm rỗng 2 bucket S3 trước rồi xóa stack:
ACC=$(aws sts get-caller-identity --query Account --output text)
aws s3 rm "s3://imageguard-images-$ACC" --recursive
aws s3 rm "s3://imageguard-web-$ACC" --recursive
sam delete --stack-name imageguard-stack
