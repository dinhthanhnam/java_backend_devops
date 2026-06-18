# PROMPT CHO GAMMA: TỔNG QUAN DEVOPS & QUY TRÌNH CI/CD (SESSION 1)

## 1. CONTEXT & ROLE
* **Role:** DevOps Architect kiêm Giảng viên cao cấp. Giọng điệu chuyên nghiệp, trực diện, đi thẳng vào bản chất kỹ thuật và thực tiễn vận hành hệ thống.
* **Target Audience:** Kỹ sư phần mềm, học viên đang phát triển dự án Spring Boot Microservices (hệ thống QuickBite).
* **Objective:** Dịch chuyển tư duy từ triển khai thủ công tại local sang tự động hóa vận hành trên Production với độ tin cậy và tính ổn định cao nhất.

---

## 2. PARTITION INSTRUCTIONS
* **Số lượng slide khuyến nghị:** 12 - 16 slides.
* **Nguyên tắc phân bổ nội dung:**
  * **LESSON 01 (Mở đầu & Dẫn dắt):** Phân tích bối cảnh, so sánh tư duy hệ thống và chỉ ra các trường hợp lỗi triển khai thực tế trong quy trình thủ công. Dẫn dắt sang các bài học kỹ thuật tiếp theo.
  * **Từ LESSON 02 đến LESSON 06 (Giải pháp kỹ thuật):** Đi thẳng vào định nghĩa, sơ đồ luồng hoạt động, cấu hình và lệnh thực hành thực tế.
  * **Độ thoáng đãng:** Một slide chỉ trình bày một thông điệp hoặc khái niệm cốt lõi. Không nhồi nhét chữ.
  * **Độ cô đọng cao:** Sử dụng các câu văn ngắn gọn, súc tích, loại bỏ các từ ngữ suồng sã, giật gân (như "ăn hành", "intern").

---

## 3. HIERARCHICAL OUTLINE (DÀN Ý CHI TIẾT)

### LESSON 01: Tổng quan DevOps và hạn chế của triển khai thủ công

#### Slide 1: Sự chuyển dịch tư duy vận hành hệ thống
* **Tư duy hướng Local (Local-centric):**
  * Tập trung duy nhất vào việc phát triển mã nguồn chạy được trên môi trường cá nhân (localhost).
  * Thiếu sự quan tâm đến cách đóng gói, cấu hình máy chủ, bảo mật và khả năng mở rộng.
* **Tư duy hướng Production (Production-ready):**
  * Mã nguồn viết ra phải được đóng gói chuẩn hóa, tự động kiểm thử và sẵn sàng triển khai lên Production.
  * Đảm bảo khả năng giám sát lỗi thời gian thực và khôi phục hệ thống tự động.

#### Slide 2: Các lỗi triển khai thủ công điển hình (Dự án QuickBite)
* Triển khai dịch vụ `user-service` thủ công trên máy chủ ảo (VPS) gặp các hạn chế sau:
  1. *Quy trình cập nhật kéo dài:* Sửa đổi nhỏ -> Build file JAR cục bộ -> Truyền file qua SCP -> SSH vào máy chủ để khởi chạy lại. Thời gian thực hiện trung bình 10 phút mỗi lần.
  2. *Sai lệch cấu hình (Environment Drift):* Sử dụng nhầm file cấu hình database dev/test trên máy chủ Production, dẫn đến việc làm sai lệch dữ liệu của hệ thống đang vận hành thực tế.

#### Slide 3: Các lỗi triển khai thủ công về Runtime và Quản lý phiên bản
* 3. *Bất đồng bộ phiên bản Java (Java Runtime Mismatch):* Sử dụng Java 21 ở máy cá nhân nhưng máy chủ chỉ cài đặt Java 17/11, gây ra lỗi `UnsupportedClassVersionError` khi chạy file JAR.
* 4. *Hỗn loạn phiên bản đóng gói:* Đặt tên file backup thủ công (`user-service-0.0.1.jar`, `user-service-final.jar`, `fixed-bug.jar`), gây khó khăn và mất an toàn khi cần khôi phục phiên bản cũ (Rollback).
* 5. *Thiếu thông tin giám sát (Log Blindness):* Ứng dụng sập không rõ nguyên nhân; không có log tập trung, buộc phải tìm lỗi thủ công trong các file log thô dung lượng lớn.

#### Slide 4: Quy trình bàn giao truyền thống & Bức tường ngăn cách (Wall of Confusion)
* **Quy trình truyền thống:**
  `Mã nguồn Local ──► Build JAR ──► Truyền file (SCP/FTP) ──► Cấu hình ──► SSH Khởi chạy`
* **Bức tường ngăn cách (Wall of Confusion):**
  * Đội phát triển (Dev) ưu tiên tốc độ phát hành tính năng mới.
  * Đội vận hành (Ops) ưu tiên tính ổn định của hệ thống.
  * Thiếu sự tự động hóa dẫn đến việc đổ lỗi lẫn nhau khi xảy ra sự cố.

#### Slide 5: Định nghĩa DevOps & 3 Trụ cột cho Lập trình viên Backend
* **DevOps:** Sự kết hợp giữa Phát triển (Development) và Vận hành (Operations) thông qua cải tiến Văn hóa, Quy trình và Công cụ tự động hóa.
* **3 Yêu cầu đối với lập trình viên Java Backend:**
  1. *Am hiểu hạ tầng thực tế:* Quản lý tiến trình chạy ngầm trên Linux, tối ưu hóa kết nối cơ sở dữ liệu.
  2. *Hiểu sâu runtime (JVM):* Cấu hình tham số RAM (Heap size) phù hợp với tài nguyên máy chủ, tránh lỗi cạn bộ nhớ và bị tiến trình OOM Killer chấm dứt hoạt động.
  3. *Đường ống tự động (Automated Pipeline):* Tự động hóa toàn bộ chu kỳ tích hợp và triển khai.
* **Dẫn nhập chuyển tiếp:** *"Để tự động hóa hoàn toàn quy trình này và loại bỏ các lỗi thao tác thủ công, chúng ta bắt đầu với quy trình CI/CD..."*

### LESSON 02: Khái niệm CI/CD (quy trình build, test, deploy)

#### Slide 6: Khái niệm CI/CD và Tự động hóa quy trình
* **Vấn đề:** Kiểm tra chất lượng mã nguồn thủ công gây mất thời gian và dễ lọt lỗi cú pháp hoặc logic lên nhánh chính.
* **Giải pháp:** Tự động kích hoạt quy trình kiểm tra qua Git Webhook mỗi khi lập trình viên thực hiện `git push`.
* **Phân biệt 3 thành phần chính:**
  * **CI (Continuous Integration - Tích hợp liên tục):** Tự động biên dịch (Compile) và chạy kiểm thử tự động (Unit Test) ngay khi có code mới. Áp dụng triết lý *Fail-fast* (Thất bại sớm) để ngăn chặn mã nguồn lỗi đi sâu vào hệ thống.
  * **Continuous Delivery (Chuyển giao liên tục):** Tự động đóng gói (JAR/Docker Image) sau khi kiểm thử thành công. Việc triển khai lên Production yêu cầu phê duyệt thủ công (Manual Approval).
  * **Continuous Deployment (Triển khai liên tục):** Tự động hóa hoàn toàn; mã nguồn vượt qua kiểm thử sẽ tự động cập nhật trực tiếp lên Production mà không cần phê duyệt.

#### Slide 7: Sơ đồ luồng hoạt động của Pipeline CI/CD
* Bất kỳ bước nào lỗi, toàn bộ pipeline dừng lại ngay lập tức để bảo vệ hệ thống:
  ```text
  [Git Push] ──► [Compile (Biên dịch)] ──(Lỗi)──► [DỪNG & BÁO LỖI]
                       │ (Thành công)
                       ▼
                 [Unit Test (Kiểm thử)] ──(Lỗi)──► [DỪNG & BÁO LỖI]
                       │ (Thành công)
                       ▼
                 [Package (Đóng gói)] ──► [Deploy (Triển khai)]
  ```
* **Lưu ý:** Tự động copy file mà không qua biên dịch tập trung và chạy Unit Test chỉ là CD (Triển khai), hoàn toàn thiếu cấu phần CI (Kiểm soát chất lượng).

### LESSON 03: Mô hình môi trường Dev, Staging và Production

#### Slide 8: Bản đồ 3 Môi trường & Nguyên tắc "Build Once, Run Anywhere"
* Hệ thống QuickBite được phân chia thành 3 môi trường độc lập:
  * **Dev (Development):** Môi trường máy local của lập trình viên. Dữ liệu giả lập, độ ổn định thấp, phục vụ debug.
  * **Staging (UAT):** Môi trường thử nghiệm độc lập, giống Production 99%. Dữ liệu mô phỏng, phục vụ nghiệm thu và kiểm thử tích hợp.
  * **Prod (Production):** Môi trường chạy thật phục vụ khách hàng. Yêu cầu bảo mật tuyệt đối và độ ổn định cao (uptime 99.99%).
* **Nguyên tắc "Build Once, Run Anywhere":**
  * File JAR chỉ được biên dịch một lần duy nhất từ máy chủ CI. 
  * Sử dụng đúng file JAR này chạy trên mọi môi trường. Sự khác biệt được nạp thông qua các tham số cấu hình động tại thời điểm khởi chạy.

#### Slide 9: Thực hành quản lý cấu hình bằng Biến môi trường
* **Vấn đề:** Hardcode tài khoản database trong `application.yml` gây lộ thông tin bảo mật trên Git và buộc phải build lại file JAR khi thay đổi môi trường.
* **Cấu hình động trong `application.yml` (Twelve-Factor App):**
  ```yaml
  spring:
    datasource:
      url: ${QUICKBITE_DB_URL:jdbc:postgresql://localhost:5432/quickbite_user}
      username: ${QUICKBITE_DB_USER:postgres}
      password: ${QUICKBITE_DB_PASS:password123}
  ```
  *Giải thích:* Ứng dụng nạp giá trị từ biến môi trường hệ thống. Nếu không tìm thấy biến, cấu hình mặc định (sau dấu hai chấm) sẽ được sử dụng cho môi trường local.
* **Lệnh khởi chạy nạp cấu hình động trên Staging Server:**
  ```bash
  export QUICKBITE_DB_URL=jdbc:postgresql://10.0.1.20:5432/quickbite_user_staging
  export QUICKBITE_DB_USER=staging_admin
  export QUICKBITE_DB_PASS=StagingSecurePass@2026
  java -jar user-service-0.0.1.jar
  ```
* **Nguyên tắc bảo mật:** Không lưu trữ mật khẩu Production trên Git. Mật khẩu chỉ được phép tồn tại dưới dạng biến môi trường ở runtime của server Production.

### LESSON 04: Kiến trúc triển khai hệ thống Microservices và luồng đi của dữ liệu

#### Slide 10: Sơ đồ kiến trúc triển khai QuickBite phân lớp
* **Vấn đề:** Client kết nối trực tiếp đến IP và cổng dịch vụ backend gây lộ cổng nội bộ ra ngoài internet, khó quản lý HTTPS và thiếu linh hoạt khi di chuyển dịch vụ.
* **Kiến trúc phân lớp an toàn:**
  ```text
  [ Client (Web/Mobile) ] ──(Port 443 - Public)──► [ Nginx Reverse Proxy ]
                                                          │
                                                          ▼ (Port 8080 - Routing)
                                                   [ API Gateway ]
                                                          │
                                         ┌────────────────┴────────────────┐
                                         ▼ (Private Network)               ▼ (Private Network)
                                  [ user-service ]                  [ order-service ]
                                         │                                 │
                                         ▼ (Port 5432)                     ▼ (Port 5432)
                                 [ user_db (Postgres) ]            [ order_db (Postgres) ]
  ```

#### Slide 11: Vai trò các thành phần trong kiến trúc triển khai
* **Nginx (Edge Layer):** Chốt chặn ngoài cùng. Nhận request cổng 80/443, trả file tĩnh Frontend (`web.quickbite.com`), thực hiện giải mã SSL (SSL Termination) và chuyển tiếp request API về API Gateway.
* **API Gateway (Spring Cloud Gateway):** Định tuyến động request theo Path (ví dụ: `/api/v1/users` -> `user-service`), xử lý bộ lọc bảo mật xác thực (Authentication) và giới hạn lượt gọi (Rate Limiting).
* **Lớp dịch vụ & Lớp dữ liệu nội bộ:** Nằm trong mạng nội bộ (Private Network), ẩn hoàn toàn khỏi internet. Áp dụng nguyên lý *Database-per-service* để cô lập dữ liệu.

### LESSON 05: Hệ điều hành Linux và vai trò trong triển khai hệ thống

#### Slide 12: Vai trò của Linux & Lệnh khởi tạo hệ thống
* **Lý do sử dụng Linux:** Là hệ điều hành máy chủ tiêu chuẩn của DevOps. Công nghệ Container (Docker) chia sẻ chung nhân hệ điều hành (Shared Kernel) và yêu cầu nhân đó bắt buộc là Linux Kernel.
* **Cài đặt JDK 17 trên Ubuntu Linux:**
  ```bash
  sudo apt-get update && sudo apt-get upgrade -y
  sudo apt-get install -y openjdk-17-jdk curl git
  ```
* **Viết Script cài đặt tự động (`initial-script.sh`):**
  ```bash
  #!/bin/bash
  sudo apt-get update && sudo apt-get upgrade -y
  sudo apt-get install -y openjdk-17-jdk curl git
  ```
  Cấp quyền thực thi và khởi chạy:
  ```bash
  chmod +x initial-script.sh
  ./initial-script.sh
  ```

#### Slide 13: Thao tác file, thư mục và quản lý dịch vụ
* **Các lệnh cơ bản bắt buộc:** `pwd`, `ls -la` (danh sách chi tiết), `cd`, `mkdir -p` (tạo thư mục phân cấp), `cat` (xem file), `nano/vi` (soạn thảo), `cp`, `mv`, `rm -rf`.
* **Cấu hình biến môi trường lâu dài:**
  ```bash
  echo "export QUICKBITE_DB_USER=staging_admin" >> ~/.bashrc
  source ~/.bashrc # Nạp lại cấu hình ngay lập tức
  ```
* **Quản lý tiến trình & Dịch vụ:**
  * Tìm PID tiến trình Java: `ps -ef | grep java` -> Dừng tiến trình: `kill -9 [PID]`.
  * Điều khiển Systemd: `sudo systemctl start/stop/restart/status/enable/disable [service]`.
  * **Kiểm tra và tải lại Nginx:**
    * `sudo nginx -t`: Kiểm tra cú pháp file cấu hình trước khi áp dụng.
    * `sudo nginx -s reload`: Tải lại cấu hình Nginx mà không cần restart máy chủ.

### LESSON 06: Quản lý quyền và các lệnh mạng cơ bản trong Linux

#### Slide 14: Bảo mật hệ thống - Tạo người dùng chuyên dụng
* **Nguyên nhân bảo mật:** Chạy ứng dụng bằng quyền `root` tạo ra lỗ hổng bảo mật nghiêm trọng. Nếu ứng dụng bị khai thác, hacker sẽ có quyền kiểm soát toàn bộ máy chủ.
* **Tạo nhóm và user hệ thống chuyên dụng chạy dịch vụ:**
  ```bash
  sudo groupadd quickbite
  sudo useradd -r -g quickbite -s /bin/false quickbite
  ```
  *Ý nghĩa tham số:*
  * `-r`: Tạo tài khoản hệ thống (system account), không tạo thư mục home.
  * `-s /bin/false`: Vô hiệu hóa shell đăng nhập. Người dùng không thể login hoặc SSH vào hệ thống, ngăn chặn hacker mở shell tương tác.

#### Slide 15: Phân quyền thư mục & Chẩn đoán cổng mạng vật lý
* **Quy trình phân quyền thư mục ứng dụng:**
  ```bash
  sudo mkdir -p /opt/quickbite/user-service
  sudo chown -R quickbite:quickbite /opt/quickbite # Đổi chủ sở hữu
  sudo chmod -R 750 /opt/quickbite # Đọc/ghi/chạy cho owner, đọc/chạy cho group, cấm user khác
  chmod +x gradlew # Cấp quyền chạy script build
  ```
* **Chẩn đoán mạng & Cổng dịch vụ:**
  * `ip addr`: Kiểm tra IP máy chủ.
  * `ping [IP_Server]`: Kiểm tra đường truyền vật lý tới máy chủ khác.
  * `curl -I [URL]`: HTTP request nhanh kiểm tra trạng thái dịch vụ.
* **Xử lý sự cố trùng cổng (Port Conflict - Cổng 8080):**
  * Kiểm tra tiến trình đang chiếm cổng:
    ```bash
    sudo ss -tulpn | grep :8080
    ```
    *Kết quả hiển thị PID (ví dụ PID = 14522).*
  * Giải phóng cổng mạng:
    ```bash
    sudo kill -9 14522
    ```
