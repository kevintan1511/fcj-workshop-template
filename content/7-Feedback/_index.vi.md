---
title: "Chia sẻ, đóng góp ý kiến"
date: 2026-09-14
weight: 7
chapter: false
pre: " <b> 7. </b> "
---

# Chia sẻ, đóng góp ý kiến

Trong suốt 3 tháng thực tập tại **Công ty TNHH Amazon Web Services Việt Nam** trong khuôn khổ chương trình **First Cloud AI Journey (FCAJ)** từ **14/09/2026** đến **14/12/2026**, em đã có một hành trình học tập và trải nghiệm thực tế đầy ý nghĩa. Dưới đây là những chia sẻ, đánh giá trung thực và đóng góp ý kiến từ góc nhìn cá nhân của một sinh viên chuyên ngành Trí tuệ nhân tạo (Đại học FPT).

---

### Đánh giá chung về chương trình

**1. Môi trường làm việc**
* Văn phòng AWS Việt Nam tại Tầng 26 Tòa nhà Bitexco mang lại một không gian làm việc hiện đại, chuyên nghiệp và truyền cảm hứng rất lớn với góc nhìn toàn cảnh thành phố.
* Việc được cấp tài khoản AWS thực tế cùng ngân sách thực hành (AWS Credits) giúp sinh viên xóa bỏ khoảng cách giữa lý thuyết giảng đường và triển khai thực chiến trên môi trường đám mây doanh nghiệp.

**2. Sự hỗ trợ từ đội ngũ Mentor và Ban tổ chức**
* Các anh mentor (anh Lữ Hoàn Thiện, anh Nguyễn Gia Hưng) luôn nhiệt tình, tâm huyết và có chiều sâu chuyên môn cao.
* Điều em tâm đắc nhất ở các mentor là phương pháp hướng dẫn mang tính định hướng: không đưa ra đáp án có sẵn ngay lập tức, mà đặt câu hỏi gợi mở để sinh viên tự đọc tài liệu kiến trúc (AWS Architecture Center), tra cứu mã lỗi và kiểm tra log trên CloudWatch để rèn luyện kỹ năng phân tích nguyên nhân gốc rễ (Root Cause Analysis).

**3. Sự phù hợp giữa công việc thực tập và chuyên ngành**
* Là sinh viên chuyên ngành Trí tuệ nhân tạo (AI), trước đây em chủ yếu nghiên cứu thuật toán và huấn luyện mô hình trên môi trường máy tính cá nhân hoặc Google Colab.
* Tham gia chương trình FCAJ giúp em nhận thức rõ ràng rằng trong doanh nghiệp, mô hình AI chỉ thực sự tạo ra giá trị khi được tích hợp vào một hạ tầng đám mây hoàn chỉnh: luồng dữ liệu tự động với S3 Event Notifications, kiến trúc xử lý Serverless với AWS Lambda, lưu trữ phân tán an toàn trên DynamoDB và bảo mật qua IAM/Presigned URL.

**4. Cơ hội học hỏi và phát triển kỹ năng**
* Em học được cách tư duy như một kỹ sư giải pháp đám mây (Solution Architect): thiết kế hệ thống bám sát 6 trụ cột của AWS Well-Architected Framework, ước tính chi phí vận hành bằng AWS Pricing Calculator, thiết lập cơ chế cảnh báo ngân sách tự động (AWS Budgets) và biên soạn tài liệu kỹ thuật chuẩn mực song ngữ.
* Dự án Workshop cuối kỳ (**ImageGuard**) là trải nghiệm thực chiến quý giá nhất, giúp em tự tay xử lý các bài toán kỹ thuật từ bước thiết kế sơ đồ, cấu hình phân quyền cho đến gỡ lỗi và kiểm thử toàn diện.

**5. Văn hóa doanh nghiệp và tinh thần đồng đội**
* Văn hóa cởi mở, tôn trọng sự sáng tạo và chủ động của sinh viên.
* Tinh thần phối hợp trong nhóm **AITeamFPT** rất gắn kết. Khi đề tài ban đầu gặp khó khăn về hạn ngạch quota của mô hình AI, cả nhóm đã cùng nhau thảo luận, phân tích và nhanh chóng chuyển đổi sang Amazon Rekognition một cách nhịp nhàng.
* Các sự kiện công nghệ cộng đồng (Buildrathon Kickoff, AWS Cloud & AI Day Watch Party, Cloud Mastery 1) đã giúp em mở rộng mạng lưới quan hệ và giao lưu học hỏi với nhiều kỹ sư giỏi trong ngành.

**6. Chính sách và quyền lợi thực tập**
* Sinh viên được tạo điều kiện sử dụng không gian làm việc tại văn phòng AWS, được tặng voucher kích hoạt 12 tháng học tập trên nền tảng AWS Skill Builder Premium, hỗ trợ credit thực hành và tham gia miễn phí các hội thảo công nghệ cao cấp.

---

### Trải nghiệm và câu hỏi thêm

* **Điều em hài lòng và tâm đắc nhất:**
  Trực tiếp thấy hệ thống **ImageGuard** của nhóm vận hành trơn tru từ giao diện người dùng: người dùng tải ảnh lên qua web, hệ thống tự động kích hoạt luồng kiểm duyệt nội dung không phù hợp, phân loại nhãn và gửi thông báo cảnh báo tức thì về email qua Amazon SNS mà không cần phải quản trị bất kỳ máy chủ vật lý nào.

* **Những điểm đề xuất cải thiện cho các khóa thực tập sau:**
  * *Tích hợp bài lab AI sớm hơn:* Trong giai đoạn đầu, lộ trình tập trung khá nhiều vào hạ tầng mạng (VPC) và máy chủ ảo (EC2). Ban tổ chức có thể lồng ghép các bài lab nhỏ về gọi API dịch vụ AI qua Lambda ngay từ tuần 3–4 để sinh viên ngành AI sớm bắt nhịp và duy trì sự hào hứng với chuyên môn.
  * *Cảnh báo và hướng dẫn về Service Quotas:* Đối với các tài khoản mới, hạn ngạch của một số dịch vụ AI tạo sinh nâng cao (như Bedrock) thường bằng 0 và cần thời gian xét duyệt. Chương trình nên lưu ý trước điều này hoặc chuẩn bị sẵn kịch bản dự phòng để các nhóm không bị gián đoạn tiến độ.
  * *Tổ chức phiên giải đáp kỹ thuật định kỳ (Office Hours):* Đề xuất có khoảng 1 buổi/tuần (khoảng 45–60 phút trực tuyến) để các anh mentor hỗ trợ gỡ rối nhanh các vấn đề thường gặp về phân quyền IAM hay CORS policy.

* **Bạn có sẵn lòng giới thiệu chương trình cho bạn bè không? Vì sao?**
  * **Chắc chắn có (100%):** Em đã và đang giới thiệu chương trình FCAJ cho các bạn sinh viên khóa dưới tại trường Đại học FPT. Đây là chương trình thực tập chất lượng hàng đầu hiện nay, giúp sinh viên không chỉ có kiến thức vững chắc về Cloud/AI mà còn định hình tác phong làm việc chuyên nghiệp chuẩn quốc tế.

---

### Đề xuất và định hướng tương lai

* **Định hướng cá nhân:** Ngay sau khi hoàn thành kỳ thực tập, em sẽ tận dụng tài khoản Skill Builder Premium được cấp để ôn tập và thi chứng chỉ quốc tế **AWS Certified Cloud Practitioner** và **AWS Certified AI Practitioner** nhằm củng cố lộ trình nghề nghiệp.
* **Lời cảm ơn:** Em xin gửi lời cảm ơn chân thành và sâu sắc nhất đến Ban quản lý chương trình First Cloud AI Journey, các anh mentor tại AWS Việt Nam và quý Thầy Cô trường Đại học FPT đã tạo điều kiện tối đa cho em trong suốt 3 tháng thực tập vừa qua.
