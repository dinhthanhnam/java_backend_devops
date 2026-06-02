# CONTENT DEVELOPMENT GUIDELINE

## Học phần DevOps thực hành cho Spring Boot Microservices

### 1. Mục tiêu học phần

Học phần được xây dựng nhằm giúp sinh viên tiếp cận DevOps theo hướng thực hành, tập trung vào các kỹ năng triển khai và vận hành hệ thống backend trong môi trường thực tế.

Sau khi hoàn thành học phần, sinh viên phải có khả năng:

- Hiểu được vai trò của DevOps trong quy trình phát triển phần mềm hiện đại.

- Hiểu được quy trình CI/CD từ lúc lập trình đến khi triển khai hệ thống.

- Sử dụng Docker để đóng gói ứng dụng.

- Xây dựng và vận hành hệ thống nhiều container bằng Docker Compose.

- Triển khai hệ thống Spring Boot Microservices lên VPS.

- Thiết lập quy trình CI/CD cơ bản bằng GitLab CI/CD.

- Cấu hình Reverse Proxy bằng Nginx.

- Thiết lập hệ thống giám sát bằng Prometheus và Grafana.

- Thu thập và phân tích log tập trung bằng EFK Stack.

- Hiểu được các vấn đề phổ biến khi vận hành hệ thống production và cách xử lý cơ bản.

Học phần không hướng đến đào tạo DevOps Engineer chuyên sâu. Mục tiêu là giúp sinh viên backend có khả năng tự triển khai, giám sát và vận hành hệ thống của mình trong các dự án thực tế quy mô nhỏ và trung bình.

---

# 2. Triết lý xây dựng nội dung

Toàn bộ học phần phải tuân thủ các nguyên tắc sau.

## 2.1 Thực hành là trung tâm

Mọi kiến thức được đưa vào bài học phải phục vụ cho một thao tác thực tế.

Không trình bày lý thuyết chỉ vì đó là kiến thức nền.

Mỗi khái niệm cần trả lời được tối thiểu các câu hỏi:

- Khái niệm này giải quyết vấn đề gì?

- Nếu không có nó sẽ xảy ra chuyện gì?

- Sinh viên sẽ sử dụng nó ở đâu trong hệ thống thực tế?

- Khi nào nên sử dụng và khi nào không nên sử dụng?

Nếu không trả lời được các câu hỏi trên thì không nên đưa nội dung đó vào học phần.

---

## 2.2 Học thông qua hệ thống xuyên suốt

Toàn bộ học phần phải xoay quanh một hệ thống duy nhất mang tên:

QuickBite – Food Delivery Microservices Platform

QuickBite là một nền tảng đặt món ăn trực tuyến được xây dựng theo kiến trúc Microservices nhằm phục vụ mục tiêu học tập của học phần.

Hệ thống bao gồm các thành phần chính:

- API Gateway

- User Service

- Restaurant Service

- Order Service

- Notification Service

- Postgres Database

Trong các giai đoạn nâng cao của học phần có thể bổ sung:

- Prometheus

- Grafana

- Elasticsearch

- Fluentd

- Kibana

- Nginx

Toàn bộ Session trong học phần phải sử dụng chính hệ thống này làm ví dụ minh họa.

Không thay đổi sang các hệ thống khác như:

- Book Management

- Student Management

- Inventory Management

- Employee Management

trong quá trình xây dựng nội dung.

Ví dụ:

Session Docker:  
Docker hóa Order Service.

Session Docker Compose:  
Triển khai Order Service cùng Postgresql.

Session API Gateway:  
Định tuyến request từ Gateway tới các Service.

Session CI/CD:  
Tự động build và push image cho hệ thống QuickBite.

Session VPS:  
Triển khai QuickBite lên môi trường thực tế.

Session Monitoring:  
Giám sát các Service đang vận hành.

Session Logging:  
Thu thập và phân tích log từ toàn bộ hệ thống.

Mục tiêu là giúp sinh viên cảm nhận được quá trình phát triển, triển khai và vận hành một sản phẩm hoàn chỉnh thay vì học các công nghệ riêng lẻ.

---

## 2.3 Ngữ cảnh nghiệp vụ chuẩn của học phần

Mọi ví dụ, sơ đồ, tình huống, bài thực hành và câu hỏi đánh giá nên ưu tiên sử dụng ngữ cảnh của hệ thống QuickBite.

Luồng nghiệp vụ mặc định của học phần:

Khách hàng đặt món ăn

↓

API Gateway tiếp nhận request

↓

Order Service tạo đơn hàng

↓

Restaurant Service kiểm tra trạng thái nhà hàng hoặc món ăn

↓

Notification Service gửi thông báo kết quả

Ngữ cảnh này được sử dụng xuyên suốt khi giải thích:

- Docker

- Docker Compose

- Network

- API Gateway

- Reverse Proxy

- CI/CD

- Monitoring

- Logging

- TraceId

Ví dụ:

Khi giải thích Docker Network:

Order Service cần giao tiếp với Restaurant Service.

Khi giải thích API Gateway:

Client chỉ gọi Gateway thay vì gọi trực tiếp từng Service.

Khi giải thích TraceId:

Một TraceId đi qua Gateway → Order Service → Restaurant Service → Notification Service.

Khi giải thích Monitoring:

Theo dõi số lượng đơn hàng được tạo mỗi phút.

Khi giải thích Logging:

Theo dõi toàn bộ vòng đời của một yêu cầu đặt món.

Việc duy trì ngữ cảnh nghiệp vụ thống nhất giúp người học dễ dàng liên kết các kiến thức giữa các Session và hiểu rõ hơn vai trò của từng công nghệ trong hệ thống thực tế.

---

## 2.4 Ví dụ phải chạy được

Mọi ví dụ code xuất hiện trong học phần phải đáp ứng ít nhất một trong các yêu cầu sau:

- Có thể chạy trực tiếp.

- Có thể copy và sử dụng ngay.

- Có thể kiểm chứng kết quả.

Không sử dụng pseudo-code nếu có thể thay thế bằng ví dụ thực tế.

Ưu tiên:

- Dockerfile hoàn chỉnh.

- docker-compose.yml hoàn chỉnh.

- Nginx configuration hoàn chỉnh.

- GitLab CI pipeline hoàn chỉnh.

- Spring Boot configuration hoàn chỉnh.

---

## 2.5 Đơn giản hóa nhưng không đơn giản hóa quá mức

Mục tiêu của học phần là giúp sinh viên hiểu bản chất và áp dụng được.

Không cố gắng mô phỏng toàn bộ hệ thống production cấp doanh nghiệp.

Không đưa vào các chủ đề nâng cao nếu chúng không phục vụ trực tiếp cho mục tiêu học phần.

Ví dụ:

Nên học:

- Docker

- Docker Compose

- GitLab CI/CD

- VPS

- Nginx

- Prometheus

- Grafana

- EFK

Không bắt buộc:

- Kubernetes

- Service Mesh

- ArgoCD

- Terraform

- AWS ECS

- AWS EKS

Trừ khi có module nâng cao riêng.

---

# 3. Định nghĩa chuẩn Session

Một Session đại diện cho một chủ đề lớn.

Mỗi Session thường kéo dài từ 4 đến 6 Lesson.

Mỗi Session phải có:

- Mục tiêu rõ ràng.

- Chủ đề xuyên suốt.

- Kết quả đầu ra cụ thể.

Ví dụ:

Session Docker Compose

Kết quả đầu ra:

Sinh viên có thể triển khai một hệ thống nhiều container bằng Docker Compose.

Không được định nghĩa Session theo dạng:

"Học một số kiến thức về Docker Compose"

Vì không xác định được kết quả đầu ra.

---

# 4. Định nghĩa chuẩn Lesson

Lesson là đơn vị nhỏ nhất của nội dung.

Mỗi Lesson tương ứng với một video E-learning.

Một Lesson chỉ tập trung vào một mục tiêu học tập duy nhất.

Không nhồi nhét nhiều khái niệm trong cùng một Lesson.

Ví dụ không tốt:

Docker Compose, Network và Volume.

Ví dụ tốt:

Docker Compose và vai trò trong hệ thống nhiều container.

Volume trong Docker Compose.

Network trong Docker Compose.

---

# 5. Định hướng cấu trúc Lesson

Lesson cần được thiết kế mạch lạc, tập trung vào giải quyết vấn đề thực tế, khuyến khích sự linh hoạt trong cách đặt tên đề mục và bố cục bài viết thay vì ép buộc một khuôn mẫu cứng nhắc (ví dụ: không nhất thiết phải chia thành đúng "Phần 1" đến "Phần 6"). 

Tuy nhiên, nội dung bài học nên đảm bảo truyền tải đầy đủ các khối logic sau:

* **Mục tiêu học tập:** Giúp người học định hình rõ ràng kết quả đầu ra (làm được gì, hiểu được gì sau bài học).
* **Đặt vấn đề thực tế:** Xuất phát từ khó khăn, tình huống thực tế của hệ thống QuickBite để người học hiểu lý do công nghệ đó tồn tại.
* **Kiến thức cốt lõi:** Trình bày ngắn gọn, súc tích các khái niệm/lý thuyết phục vụ trực tiếp cho việc giải quyết vấn đề.
* **Thực hành minh họa (Demo):** Cung cấp các bước thực thi chi tiết (cấu hình, câu lệnh, mã nguồn) để kiểm chứng lý thuyết.
* **Tổng kết & Đánh giá:** Đúc rút lại bài học và đưa ra các câu hỏi xử lý tình huống hoặc đọc hiểu để kiểm tra tư duy thực chiến.

---

# 6. Quy tắc đặt tên Lesson

Tên bài học phải mang tính học thuật và mô tả đúng nội dung.

Không sử dụng tiêu đề mang tính hội thoại hoặc blog.

Không sử dụng các tiêu đề dạng:

- Docker là gì?

- VPS là gì?

- Monitoring là gì?

Ưu tiên cấu trúc:

[Khái niệm] + [Vai trò hoặc ngữ cảnh sử dụng]

Ví dụ:

Docker và Virtual Machine trong triển khai ứng dụng.

Docker Image, Container và Registry.

Monitoring và Metrics trong vận hành hệ thống.

Reverse Proxy trong kiến trúc Microservices.

CI/CD trong quy trình phát triển phần mềm.

Tiêu đề phải giúp người học hình dung được nội dung trước khi mở bài học.

---

# 7. Quy tắc sử dụng mã nguồn

Code không được xuất hiện chỉ để minh họa cú pháp.

Code phải phục vụ một mục tiêu học tập cụ thể.

Mỗi đoạn code cần giải thích:

- Đang giải quyết vấn đề gì.

- Thành phần nào quan trọng.

- Kết quả sau khi chạy là gì.

Ưu tiên các ví dụ nhỏ nhưng hoàn chỉnh.

Không trình bày các đoạn code quá dài nếu không cần thiết.

---

# 8. Quy tắc sử dụng hình ảnh và sơ đồ

Mọi sơ đồ phải phục vụ việc giải thích hệ thống.

Không tạo sơ đồ chỉ để trang trí.

Ưu tiên:

- Luồng request.

- Kiến trúc hệ thống.

- Pipeline CI/CD.

- Luồng log.

- Luồng monitoring.

Sơ đồ phải đơn giản, dễ đọc và bám sát ví dụ đang được sử dụng trong học phần.

---

# 9. Quy tắc xây dựng câu hỏi đánh giá

Mỗi Lesson cần tối thiểu ba câu hỏi.

Câu 1:  
Kiểm tra khả năng hiểu bản chất.

Câu 2:  
Kiểm tra khả năng đọc và dự đoán kết quả.

Câu 3:  
Kiểm tra khả năng xử lý tình huống thực tế.

Hạn chế các câu hỏi chỉ yêu cầu ghi nhớ định nghĩa.

---

# 10. Quy tắc dành cho AI khi tạo nội dung

AI phải ưu tiên tính thực tiễn hơn tính hàn lâm.

AI không được viết nội dung theo phong cách sách giáo khoa thuần túy.

AI phải luôn:

- Bắt đầu từ vấn đề thực tế.

- Giải thích lý do tồn tại của công nghệ.

- Liên hệ với hệ thống xuyên suốt của học phần.

- Cung cấp ví dụ có thể chạy được nếu phù hợp.

- Ưu tiên minh họa bằng cấu hình, lệnh hoặc mã nguồn thực tế.

- Kết thúc bằng khả năng áp dụng thực tế của người học.

AI không được tạo nội dung chỉ tập trung vào định nghĩa hoặc lịch sử phát triển của công nghệ.
