# PROMPT CHO GAMMA: TỔNG QUAN DEVOPS & QUY TRÌNH CI/CD (SESSION 1)

Sao chép toàn bộ nội dung trong khối block câu lệnh (code block) dưới đây để dán trực tiếp vào Gamma App (chọn tính năng **Text to Presentation** và dùng **Dark Theme**).

---

```text
[BỐI CẢNH & VAI TRÒ GIẢNG DẠY]
Bạn là một DevOps Architect kiêm Giảng viên kỹ thuật cao cấp, có kinh nghiệm thực chiến dày dặn tại các hệ thống lớn. Hãy tạo một bài thuyết trình slide chuyên sâu, thực chiến và lôi cuốn cho các lập trình viên/intern mới bắt đầu bước vào thế giới hạ tầng tự động. 
Giọng điệu: Trực diện, mạnh mẽ, thực tế (sử dụng các thuật ngữ như "ăn hành", "tech lead", "lệch cấu hình", "sập kết nối"). Tránh tối đa các định nghĩa lý thuyết suông.

[QUY CHUẨN THIẾT KẾ & VISUAL]
- Giao diện: Thiết lập Dark Mode chủ đạo (màu nền xám sẫm `#121214`). Sử dụng màu xanh Neon Emerald Green để đánh dấu các trạng thái thành công, tối ưu hóa và màu đỏ/cam Neon để đánh dấu các trạng thái cảnh báo, sự cố sập hệ thống.
- Bố cục: Đa dạng hóa linh hoạt giữa các dạng bố cục: 2 cột so sánh trực diện, dòng chảy timeline ngang, thẻ thông tin dạng lưới (Grid cards) và các khối sơ đồ khối (ASCII diagrams).

[CHỈ THỊ PHÂN TÁCH SLIDE ĐỘNG - QUAN TRỌNG]
Dựa trên dàn ý chi tiết dưới đây, bạn hãy tự động phân tích và chia nhỏ nội dung thành số lượng slide phù hợp (khuyến nghị từ 12-18 slides). 
Tuyệt đối không được nhồi nhét nhiều chủ đề khác nhau vào cùng một slide. Hãy tuân thủ nguyên tắc: "Một slide chỉ giải quyết một thông điệp/khái niệm cốt lõi" để đảm bảo slide thoáng đãng, dễ đọc, chuyên nghiệp. Tự động thiết kế layout phù hợp nhất cho từng slide dựa trên cấu trúc thông tin (so sánh, timeline, sơ đồ).

---

# DÀN Ý CHI TIẾT BÀI HỌC: TỔNG QUAN DEVOPS & PIPELINE CI/CD

## CHỦ ĐỀ 1: GIỚI THIỆU KHÓA HỌC & ĐỘNG LỰC HẠ TỰ ĐỘNG
### 1. Khóa học "DevOps thực chiến với Microservices"
* **Bối cảnh thực tế:** Chúng ta sẽ thực hành trực tiếp trên hệ thống QuickBite (nền tảng đặt đồ ăn trực tuyến) gồm các dịch vụ backend Java Spring Boot (`user-service`, `order-service`, `api-gateway`) chạy trên cơ sở dữ liệu PostgreSQL.
* **Mục tiêu khóa học:** Giúp sinh viên chuyển dịch từ tư duy "chạy code local" sang tư duy "tự động hóa vận hành trên Production" với tỷ lệ uptime cao nhất.

### 2. Sự chuyển dịch tư duy của một kỹ sư
* **Tư duy cũ (Local-centric):** Chỉ quan tâm code chạy ngon trên máy cá nhân, không quan tâm cách deploy và cấu hình server.
* **Tư duy mới (Production-ready):** Code viết ra phải được đóng gói chuẩn hóa, tự động kiểm thử, tự động deploy và giám sát lỗi theo thời gian thực.

---

## CHỦ ĐỀ 2: NỖI ĐAU TRIỂN KHAI THỦ CÔNG & SỰ RA ĐỜI CỦA DEVOPS
### 1. Quy trình deploy thủ công truyền thống
* **Các bước thực hiện:**
  - Lập trình viên chạy lệnh compile ra file JAR trên máy local.
  - Sử dụng công cụ FTP hoặc lệnh `scp` đẩy file JAR thô lên máy chủ VPS.
  - Sử dụng SSH kết nối vào VPS, gõ lệnh `nohup java -jar user-service.jar &` để chạy ngầm tiến trình.
* **Hậu quả thực tế (Intern "ăn hành"):**
  - Chỉ cần gõ sai một cổng port mạng, VPS sẽ báo lỗi port collision.
  - Quên cấu hình biến môi trường database, ứng dụng sập ngay lập tức.
  - Bấm nhầm tổ hợp phím `Ctrl + C` làm tắt ngóm dịch vụ.
  - Khi code lỗi, không biết cách rollback phiên bản cũ, gây downtime hệ thống kéo dài.

### 2. DevOps là gì? Phá vỡ bức tường ngăn cách
* **Bức tường đổ lỗi (The Wall of Confusion):**
  - Đội phát triển (Development): Muốn cập nhật tính năng mới thật nhanh -> Dễ gây mất ổn định hệ thống.
  - Đội vận hành (Operations): Muốn hệ thống luôn ổn định, ít thay đổi -> Ngại cập nhật bản mới.
  - Hậu quả: Khi có lỗi, Dev đổ tại hệ điều hành của VPS, Ops đổ tại code của Dev viết lỗi.
* **Định nghĩa DevOps:** 
  - DevOps = **Development + Operations**.
  - Là sự giao thoa giữa **Văn hóa (Culture)**, **Quy trình (Process)** và **Công cụ (Tools)**.
  - Mục tiêu: Hợp nhất hai đội ngũ, tự động hóa toàn bộ chu kỳ phát hành phần mềm để tăng tốc độ bàn giao và đảm bảo độ tin cậy tuyệt đối.

---

## CHỦ ĐỀ 3: QUY TRÌNH CI/CD & CƠ CHẾ FAIL-FAST
### 1. Khái niệm CI/CD và vòng đời tự động hóa
* **CI (Continuous Integration - Tích hợp liên tục):**
  - Quy trình tự động kích hoạt ngay khi dev push code lên Git repository.
  - Hệ thống CI Engine tự động chạy: Biên dịch (Compile) -> Chạy kiểm thử (Unit Test) -> Đóng gói (Package) thành file JAR/Docker Image.
* **CD (Continuous Delivery - Bàn giao liên tục):**
  - Đảm bảo sản phẩm (artifact) sau khi đóng gói luôn ở trạng thái sẵn sàng để phát hành lên staging/production bất cứ lúc nào.
* **Continuous Deployment (Triển khai liên tục):**
  - Tự động hóa 100% việc lấy bản đóng gói mới nhất và cập nhật thẳng lên server production mà không cần con người can thiệp thủ công.

### 2. Cơ chế Fail-fast (Thất bại sớm) trong Pipeline
* **Triết lý Fail-fast:** Phát hiện lỗi càng sớm càng tốt để ngăn chặn code lỗi đi xa hơn vào hệ thống.
* **Sơ đồ luồng hoạt động của Pipeline:**
  ```text
  [ CI/CD Engine ]
        │
        ▼
  [ Stage 1: Biên dịch (Compile) ] ─────────(Thất bại)──► [ DỪNG & BÁO LỖI ]
        │ (Thành công)
        ▼
  [ Stage 2: Kiểm thử (Unit Test) ] ────────(Thất bại)──► [ DỪNG & BÁO LỖI ]
        │ (Thành công)
        ▼
  [ Stage 3: Đóng gói (Package) ] ──────────(Thất bại)──► [ DỪNG & BÁO LỖI ]
        │ (Thành công)
        ▼
  [ Stage 4: Tự động Deploy (CD) ] ───────► [ HỆ THỐNG HOẠT ĐỘNG NGON LÀNH ]
  ```
* **Ý nghĩa:** Nếu bất kỳ bước nào (ví dụ: Unit Test) bị fail, toàn bộ pipeline sẽ bị ngắt lập tức (Abort), không cho phép chuyển sang bước Deploy. Lập trình viên nhận báo động đỏ để sửa code ngay.

---

## CHỦ ĐỀ 4: BẢN ĐỒ MÔI TRƯỜNG DỰ ÁN DEV - STAGING - PRODUCTION
### 1. So sánh ba môi trường hệ thống tiêu chuẩn
* **Môi trường Dev (Development):**
  - Máy local của lập trình viên.
  - Đặc điểm: Thay đổi code liên tục, debug trực tiếp, dữ liệu test giả lập, độ cô lập thấp.
* **Môi trường Staging (Pre-production):**
  - Máy chủ VPS độc lập, mô phỏng giống 99% hệ thống thật.
  - Đặc điểm: Dùng để chạy thử nghiệm tích hợp (Integration Test) và nghiệm thu tính năng (UAT) trước khi release.
* **Môi trường Production (Hệ thống thật):**
  - Máy chủ chạy dịch vụ thực tế phục vụ khách hàng.
  - Đặc điểm: Yêu cầu bảo mật tuyệt đối, uptime 99.99%, chịu tải cao, giám sát 24/7.

### 2. Nỗi đau lệch pha cấu hình (Environment Drift)
* **Bản chất:** Code chạy mượt mà trên máy Dev nhưng đưa lên Staging hoặc Production lại crash.
* **Nguyên nhân:** Khác biệt phiên bản Java (Dev chạy Java 21, Server chạy Java 17), cấu hình múi giờ hệ điều hành khác nhau, thiếu file cấu hình hoặc biến môi trường ở server.
* **Giải pháp DevOps:** Sử dụng công nghệ đóng gói Container (Docker) để đóng băng mã nguồn cùng toàn bộ runtime cần thiết vào một khối bất biến. Chạy trên Dev thế nào thì lên Production sẽ chạy y hệt như thế.

---

## CHỦ ĐỀ 5: KIẾN TRÚC MẠNG VÀ TƯƠNG TÁC VẬT LÝ QUICKBITE
### 1. Kiến trúc phân rã dịch vụ Microservices
* Hệ thống QuickBite được thiết kế chia nhỏ thành các thành phần chạy độc lập để dễ dàng bảo trì và scale:
  - **API Gateway:** Điểm đón request tập trung của hệ thống.
  - **User Service:** Quản lý thông tin tài khoản, ví tiền khách hàng.
  - **Order Service:** Quản lý quy trình tạo và xử lý đơn đặt món.
* **Quy tắc cô lập dữ liệu:** Mỗi service sở hữu một database Postgres riêng biệt. Không service nào được phép chọc trực tiếp vào database của service khác mà bắt buộc phải gọi qua API của nhau.

### 2. Sơ đồ tương tác mạng vật lý của hệ thống QuickBite
* Sơ đồ dòng chảy dữ liệu thực tế và các cổng port hoạt động:
  ```text
  [ Client / Trình duyệt ]
            │ (Cổng 80 - HTTP)
            ▼
  [ Nginx Reverse Proxy ]
            │ (Cổng 8080 - Routing)
            ▼
  [ API Gateway ]
       ├──────────────────────────────┐
       ▼ (Cổng 8081 - API)            ▼ (Cổng 8082 - API)
  [ user-service ]               [ order-service ]
       │                              │
       ▼ (Cổng 5432)                  ▼ (Cổng 5432)
  [ user_db (Postgres) ]         [ order_db (Postgres) ]
  ```
* **Nguyên lý bảo mật phân lớp:**
  - Chỉ duy nhất cổng `80` (Nginx) được mở ra ngoài internet để đón người dùng.
  - Các cổng API nội bộ (`8080`, `8081`, `8082`) và database (`5432`) được chạy cô lập trong mạng nội bộ, ngăn chặn hoàn toàn tin tặc quét cổng tấn công trực diện.

---

## CHỦ ĐỀ 6: LỘ TRÌNH THỰC HÀNH HẠ TẦNG SESSION 1
### Các nhiệm vụ thực hành bắt buộc cho sinh viên:
1. **Dựng môi trường Sandbox:** Cài đặt thành công hệ điều hành Linux Ubuntu (hoặc kích hoạt WSL 2 Ubuntu trên Windows) để làm quen với giao diện dòng lệnh (CLI).
2. **Quản lý quyền & Bảo mật hệ thống:** Thực hành tạo nhóm người dùng (`groupadd`), thêm tài khoản dịch vụ (`useradd -r -s /bin/false`) và thiết lập phân quyền thư mục chặt chẽ (`chmod`, `chown`).
3. **Viết Shell Script tự động hóa:** 
   - Viết kịch bản kiểm tra xung đột cổng mạng vật lý trước khi deploy.
   * Tạo script tự động tải code mới, build file JAR bằng Gradle, kiểm tra lỗi và khởi chạy ngầm ứng dụng backend.
```
