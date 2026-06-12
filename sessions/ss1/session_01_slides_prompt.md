# PROMPT CHO GAMMA: TỔNG QUAN DEVOPS & QUY TRÌNH CI/CD (SESSION 1)

## 1. CONTEXT & ROLE
* **Role:** DevOps Architect kiêm Giảng viên cao cấp. Giọng điệu trực diện, thực chiến, giảng giải rõ ràng từ bản chất, sử dụng các thuật ngữ chuyên ngành quen thuộc ("ăn hành", "nút thắt cổ chai", "sập server", "lệch cấu hình").
* **Target Audience:** Kỹ sư phần mềm, học viên đang phát triển dự án Spring Boot Microservices (hệ thống QuickBite).
* **Objective:** Giải thích cặn kẽ bản chất kỹ thuật, cung cấp sẵn các câu lệnh, cấu hình và sơ đồ luồng chi tiết để người học hiểu và áp dụng được ngay vào thực tế.

---

## 2. DESIGN & VISUAL SPECS
* **Theme:** Dark Mode chủ đạo (nền `#121214`), màu nhấn Neon Emerald Green (trạng thái thành công/hệ thống chạy tốt) và Neon Red/Orange (cảnh báo lỗi hệ thống/nỗi đau).
* **Typography:** Font chữ hiện đại (*Inter*, *Outfit*).
* **Layouts:** Tự động điều phối bố cục phù hợp (Timeline cho các bước Pipeline, bảng so sánh 2-3 cột cho các môi trường, Grid cards cho phân lớp kiến trúc mạng, Code blocks cho cấu hình YAML và Shell script).

---

## 3. PARTITION INSTRUCTIONS
* **Số lượng slide khuyến nghị:** 15 - 20 slides.
* **Nguyên tắc phân bổ nội dung:**
  * **LESSON 01 (Mở đầu & Dẫn dắt):** Tập trung sâu sắc vào việc khơi gợi động lực học tập. Trình bày chi tiết bối cảnh, sự dịch chuyển tư duy và mô tả cụ thể 5 kịch bản thất bại thực tế của quy trình triển khai thủ công.
  * **Từ LESSON 02 đến LESSON 06 (Giải pháp & Ví dụ thực tế):** Đi thẳng vào nội dung chính, định nghĩa chi tiết, sơ đồ luồng dữ liệu, bảng so sánh và các khối lệnh/cấu hình có thể copy-paste được ngay.
  * **Sử dụng văn bản rõ ràng:** Cung cấp đầy đủ các đoạn mô tả chi tiết, giải thích ý nghĩa từng dòng code và tham số lệnh để Gamma chỉ việc hiển thị trực tiếp lên slide mà không cần tự suy diễn thêm text.

---

## 4. HIERARCHICAL OUTLINE (DÀN Ý CHI TIẾT)

### LESSON 01: Tổng quan DevOps và hạn chế của triển khai thủ công

#### Slide 1: Sự dịch chuyển tư duy của một Kỹ sư phần mềm
* **Tư duy cũ (Local-centric - "Chạy ngon trên localhost"):** 
  * Chỉ quan tâm viết code trên máy cá nhân, nhấn nút "Run" trên IDE là xong việc.
  * Coi việc cài đặt hệ điều hành, cấu hình mạng và triển khai server là việc của người khác.
* **Tư duy mới (Production-ready - "Sẵn sàng chạy thật"):**
  * Hiểu rằng không ai trả tiền cho một ứng dụng chỉ chạy trên localhost.
  * Code viết ra phải được đóng gói chuẩn hóa, tự động kiểm thử, tự động deploy và liên tục giám sát lỗi theo thời gian thực trên Production.
  * Chịu trách nhiệm về vòng đời vận hành của dòng code mình viết ra.

#### Slide 2: Nỗi đau triển khai thủ công - Kịch bản 1 & 2 (Dự án QuickBite)
* **Bối cảnh:** Triển khai dịch vụ `user-service` trên một VPS cấu hình yếu (1 vCPU, 1GB RAM).
* **Kịch bản 1: Vòng lặp sửa lỗi vô tận chỉ vì một dấu phẩy:**
  * Mỗi lần sửa một lỗi chính tả nhỏ -> Phải gõ lệnh build JAR tại local (tốn 3 phút) -> Dùng lệnh `scp` đẩy file JAR ~50MB lên server qua internet (tốn 5 phút) -> SSH vào server khởi chạy lại dịch vụ (tốn 2 phút).
  * *Hậu quả:* Tốn 10 phút cuộc đời chỉ để cập nhật một dấu phẩy! Sửa 10 lần mất nguyên một buổi sáng.
* **Kịch bản 2: Lệch pha cấu hình (Environment Drift) gây sập Database:**
  * Triển khai thủ công yêu cầu copy cấu hình bằng tay. Sơ ý để nguyên file `application.yml` chứa thông số database của máy dev local khi chạy trên server Production thật.
  * *Hậu quả:* Ứng dụng chạy trên Production nhưng lại kết nối và ghi đè dữ liệu vào database test ở máy dev cá nhân. Dữ liệu của khách hàng thật bị xáo trộn và rò rỉ.

#### Slide 3: Nỗi đau triển khai thủ công - Kịch bản 3, 4 & 5
* **Kịch bản 3: Lệch phiên bản chạy (Java Runtime Mismatch):**
  * Máy local dùng Java 21 để viết các tính năng mới. Lên server chạy lệnh cài đặt mặc định của hệ điều hành, cài nhầm Java 17/11.
  * *Hậu quả:* Khi chạy file JAR, JVM ném lỗi `UnsupportedClassVersionError` và sập ngay lập tức vì không tương thích phiên bản.
* **Kịch bản 4: Hỗn loạn phiên bản đóng gói:**
  * Đặt tên file backup thủ công trên server: `user-service-0.0.1.jar`, `user-service-final.jar`, `final-v2.jar`, `fixed-bug.jar`...
  * *Hậu quả:* Khi hệ thống gặp lỗi nghiêm trọng và cần quay lại phiên bản cũ ổn định (Rollback), cả đội nhìn vào thư mục server và bất lực vì không biết file nào an toàn để chạy lại.
* **Kịch bản 5: "Mù" thông tin khi có sự cố (Log Blindness):**
  * Ứng dụng chạy ngầm bị sập đột ngột. Không có log tập trung, người vận hành phải SSH vào server, mở file log thô nặng hàng GB để tìm lỗi.
  * *Hậu quả:* Hệ thống tiếp tục chết, doanh nghiệp mất tiền theo từng giây trong lúc dev mò mẫm file log.

#### Slide 4: Quy trình bàn giao truyền thống & Bức tường ngăn cách
* **Luồng di chuyển của file trong quy trình truyền thống:**
  ```text
  [Mã nguồn Local] ──(Biên dịch tay)──► [File JAR] ──(scp/ftp)──► [Server Staging/Prod] ──(Gõ lệnh chạy)──► [Dịch vụ chạy ngầm]
  ```
* **Bức tường đổ lỗi (Wall of Confusion) giữa Dev và Ops:**
  * *Đội Phát triển (Dev):* Muốn cập nhật tính năng mới thật nhanh để kịp tiến độ -> Dễ làm mất ổn định hệ thống.
  * *Đội Vận hành (Ops):* Muốn hệ thống luôn chạy ổn định, ít thay đổi -> Ngại cập nhật phiên bản mới.
  * *Hậu quả:* Khi có lỗi xảy ra, Dev đổ tại hệ điều hành của server cấu hình sai, Ops đổ tại code của Dev viết lỗi. Quy trình bàn giao bị đứt gãy.

#### Slide 5: Bản chất DevOps và 3 trụ cột đối với Lập trình viên Java
* **DevOps là gì?**
  * DevOps = **Development (Phát triển) + Operations (Vận hành)**.
  * Không chỉ là một chức danh công việc, mà là sự giao thoa giữa **Văn hóa (Culture)**, **Quy trình (Process)** và **Công cụ (Tools)** nhằm tự động hóa vòng đời phát hành phần mềm.
* **3 Yêu cầu đối với lập trình viên backend:**
  1. *Am hiểu hệ thống thực tế:* Hiểu cách ứng dụng Spring Boot kết nối database, chạy ngầm trên nền Linux và cách cấu hình ghi nhận log.
  2. *Hiểu sâu về Java Runtime (JVM):* Biết cách cấu hình bộ nhớ JVM (Heap size) để tránh bị Linux Kernel giết tiến trình vì lỗi cạn kiệt RAM (`Out of Memory` - OOM Killer) trên VPS yếu.
  3. *Sản phẩm cốt lõi - Đường ống tự động (CI/CD Pipeline):* Thiết lập dây chuyền tự động hóa thay thế hoàn toàn việc gõ lệnh thủ công.
* **Dẫn nhập chuyển tiếp:** *"Để giải quyết triệt để 5 kịch bản sập nguồn trên, chúng ta bắt đầu hành trình tự động hóa với quy trình CI/CD..."*

---

### LESSON 02: Khái niệm CI/CD (quy trình build, test, deploy)

#### Slide 6: Vấn đề thực tế & Định nghĩa CI vs CD
* **Vấn đề thực tế (Kịch bản Intern và Leader):**
  * Lập trình viên thực tập (Intern) đẩy code mới lên Git. Anh Tech Lead phải tải code về máy cá nhân chạy build và kiểm thử thủ công để đảm bảo code không lỗi cú pháp trước khi merge. Việc này biến Tech Lead thành một "intern cao cấp" làm việc vặt.
* **Giải pháp: Tự động hóa qua Git Webhook**
  * Ngay khi intern gõ `git push`, hệ thống tự động kích hoạt máy chủ độc lập chạy build và Unit Test. Nếu báo đỏ (lỗi), intern tự sửa. Nếu báo xanh, Tech Lead mới tiến hành review code.
* **Phân biệt ranh giới CI và CD:**
  * **CI (Continuous Integration - Tích hợp liên tục):** Tự động biên dịch (**Compile**) và chạy kiểm thử tự động (**Unit Test**) mỗi khi push code. Sử dụng triết lý *Fail-fast* (Thất bại sớm) để phát hiện và chặn lỗi logic ngay lập tức.
  * **Continuous Delivery (Chuyển giao liên tục):** Sau khi CI thành công, code tự động đóng gói thành file JAR/Docker Image sẵn sàng deploy. Việc cập nhật lên Production cần một thao tác bấm nút duyệt thủ công (Manual Approval).
  * **Continuous Deployment (Triển khai liên tục):** Tự động hóa 100%. Code sau khi qua bước CI sẽ tự chạy script deploy thẳng lên Production chạy thật mà không cần con người phê duyệt.

#### Slide 7: Sơ đồ luồng hoạt động của Pipeline CI/CD tiêu chuẩn
* Bất kỳ bước nào thất bại, toàn bộ đường ống sẽ dừng lại ngay lập tức (Abort) để bảo vệ server:
  ```text
  [Dev Git Push] ──► [Git Repository] ──(Webhook)──► [CI/CD Engine]
                                                           │
        ┌──────────────────────────────────────────────────┘
        ▼
  [Stage 1: Compile (Biên dịch)] ─────────(Thất bại)──► [DỪNG PIPELINE & BÁO LỖI]
        │ (Thành công)
        ▼
  [Stage 2: Unit Test (Kiểm thử)] ────────(Thất bại)──► [DỪNG PIPELINE & BÁO LỖI]
        │ (Thành công)
        ▼
  [Stage 3: Package (Đóng gói JAR)] ──────(Thất bại)──► [DỪNG PIPELINE & BÁO LỖI]
        │ (Thành công)
        ▼
  [Stage 4: CD (Tự động Deploy)] ─────────► [HỆ THỐNG CẬP NHẬT HOẠT ĐỘNG ỔN ĐỊNH]
  ```
* **Hiểu lầm kinh điển:** Viết script tự copy file JAR lên server mỗi khi code thay đổi không phải là CI/CD hoàn chỉnh. Nó thiếu hoàn toàn cấu phần **CI (Kiểm soát chất lượng bằng Unit Test)**, dễ dẫn đến việc tự động đẩy code lỗi lên làm sập server.

---

### LESSON 03: Mô hình môi trường Dev, Staging và Production

#### Slide 8: Bản đồ 3 Môi trường và Nguyên tắc "Build Once, Run Anywhere"
* Dự án QuickBite được chia làm 3 môi trường hoạt động độc lập nhằm kiểm soát rủi ro phát hành:
  * **Dev (Development):** Môi trường local của dev. Dữ liệu giả lập tự tạo, yêu cầu ổn định thấp, phục vụ viết code và debug trực tiếp.
  * **Staging (Pre-production):** Máy chủ VPS độc lập mô phỏng giống Prod 99%. Dữ liệu mô phỏng giống thật nhưng không nhạy cảm, phục vụ chạy Integration Test và nghiệm thu tính năng (UAT).
  * **Prod (Production):** Máy chủ chạy thật phục vụ người dùng cuối. Dữ liệu thật nhạy cảm, yêu cầu bảo mật tuyệt đối và uptime 99.99%.
* **Nguyên tắc "Build Once, Run Anywhere":**
  * Chỉ biên dịch mã nguồn ra một file JAR duy nhất một lần trên máy chủ CI. 
  * Sử dụng chính file JAR này chạy trên cả 3 môi trường. Điểm khác biệt duy nhất giữa các môi trường là các tham số cấu hình nạp vào lúc khởi chạy ứng dụng.

#### Slide 9: Thực hành tách biệt cấu hình bằng Biến môi trường (12-Factor App)
* **Vấn đề:** Hardcode mật khẩu DB trong `application.yml` làm lộ bí mật hệ thống trên Git (dù là Private Repo vẫn có hàng chục dev/intern xem được) và phải build lại file JAR khi đổi server.
* **Giải pháp: Cấu hình động trong `application.yml` của Spring Boot:**
  ```yaml
  spring:
    datasource:
      url: ${QUICKBITE_DB_URL:jdbc:postgresql://localhost:5432/quickbite_user}
      username: ${QUICKBITE_DB_USER:postgres}
      password: ${QUICKBITE_DB_PASS:password123}
  ```
  *Giải thích:* Spring Boot sẽ tìm biến môi trường `QUICKBITE_DB_URL` của hệ điều hành. Nếu tìm thấy, nó sẽ nạp giá trị đó. Nếu không thấy, nó tự động dùng giá trị mặc định sau dấu hai chấm (`jdbc:postgresql://localhost:5432/quickbite_user`) để chạy ở local.
* **Lệnh khởi chạy nạp cấu hình Staging động trên server:**
  ```bash
  # 1. Nạp các biến cấu hình của môi trường Staging vào hệ điều hành
  export QUICKBITE_DB_URL=jdbc:postgresql://10.0.1.20:5432/quickbite_user_staging
  export QUICKBITE_DB_USER=staging_admin
  export QUICKBITE_DB_PASS=StagingSecurePass@2026

  # 2. Khởi chạy file JAR duy nhất đã build
  java -jar user-service-0.0.1.jar
  ```

---

### LESSON 04: Kiến trúc triển khai hệ thống Microservices và luồng đi của dữ liệu

#### Slide 10: Vấn đề giao tiếp trực tiếp & Sơ đồ kiến trúc QuickBite phân lớp
* **Vấn đề thực tế:** Client (Web/Mobile) gọi trực tiếp IP và cổng của từng backend service (ví dụ: `http://10.0.1.15:8081`).
  * *Hậu quả:* Phơi bày các cổng nội bộ ra internet (hacker dễ quét cổng tấn công); Việc quản lý chứng chỉ SSL (HTTPS) cho từng cổng phức tạp; Thiếu linh hoạt khi thay đổi IP của server dịch vụ.
* **Sơ đồ dòng chảy dữ liệu an toàn của hệ thống QuickBite:**
  ```text
  [ Client (Web / Mobile) ]
            │
            ▼ (HTTPS: Port 443 - Public Internet)
  [ Nginx Reverse Proxy ]
            │ (Cổng 8080 - Routing nội bộ)
            ▼
  [ API Gateway (Spring Cloud Gateway) ]
        ├──────────────────────────────┐
        ▼ (Private Network: Port 8081)  ▼ (Private Network: Port 8082)
  [ user-service ]               [ order-service ]
        │                              │
        ▼ (Private Network: Port 5432)  ▼ (Private Network: Port 5432)
  [ user_db (PostgreSQL) ]       [ order_db (PostgreSQL) ]
  ```

#### Slide 11: Chi tiết vai trò các thành phần chốt chặn
* **Nginx (Edge Layer - Lớp biên giới):** Tấm khiên ngoài cùng tiếp nhận kết nối qua cổng mạng tiêu chuẩn `80`/`443`.
  * *Nhiệm vụ:* Trả file tĩnh Frontend nhanh chóng (`web.quickbite.com`); Giải mã SSL (SSL Termination) và chuyển tiếp request API (`api.quickbite.com`) về API Gateway ở phía sau.
* **API Gateway (Spring Cloud Gateway):** Đứng sau Nginx, tiếp nhận request nội bộ.
  * *Nhiệm vụ:* Định tuyến động request theo Path (ví dụ: path `/api/v1/users` -> chuyển tiếp tới `user-service`); Xử lý bộ lọc bảo mật chung như xác thực người dùng (Authentication) và giới hạn lượt gọi (Rate Limiting).
* **Lớp dịch vụ nội bộ (Internal Service Layer) & Lớp dữ liệu (Persistence Layer):**
  * Nằm hoàn toàn trong mạng nội bộ (Private Network), ẩn mình khỏi internet để ngăn tin tặc quét cổng tấn công trực diện.
  * *Nguyên tắc cô lập dữ liệu (Database-per-service):* Mỗi service chỉ chọc vào database riêng của mình. Vật lý có thể chạy chung cụm máy chủ PostgreSQL để tiết kiệm chi phí nhưng logic database phải tách biệt.

---

### LESSON 05: Hệ điều hành Linux và vai trò trong triển khai hệ thống

#### Slide 12: Tại sao học Linux & Lệnh khởi tạo hệ thống
* **Tại sao Linux là chuẩn mực của DevOps?** Linux cực kỳ gọn nhẹ, bảo mật cao và vận hành ổn định qua giao diện dòng lệnh (CLI). Đặc biệt, công nghệ đóng gói Container (Docker) hoạt động dựa trên cơ chế chia sẻ chung nhân hệ điều hành (Shared Kernel) và nhân đó bắt buộc là Linux Kernel.
* **Khối lệnh khởi tạo hệ thống ban đầu khi nhận VPS mới:**
  ```bash
  # 1. Cập nhật danh sách gói và nâng cấp hệ thống vá lỗi bảo mật
  sudo apt-get update && sudo apt-get upgrade -y
  
  # 2. Cài đặt Java Development Kit (JDK 17) để chạy Spring Boot
  sudo apt-get install -y openjdk-17-jdk curl git
  ```
* **Mẹo tự động hóa thực chiến:** Tạo file script cài đặt ban đầu `initial-script.sh`:
  ```bash
  #!/bin/bash
  sudo apt-get update && sudo apt-get upgrade -y
  sudo apt-get install -y openjdk-17-jdk curl git
  ```
  Cấp quyền chạy và khởi chạy tự động:
  ```bash
  chmod +x initial-script.sh
  ./initial-script.sh
  ```

#### Slide 13: Thao tác file, thư mục và quản lý tiến trình trên Linux
* **Các lệnh cơ bản bắt buộc phải thuộc:**
  * `pwd`: Xem đường dẫn thư mục hiện tại.
  * `ls -la`: Liệt kê chi tiết toàn bộ file (bao gồm cả file ẩn bắt đầu bằng dấu chấm).
  * `cd [đường_dẫn]`: Di chuyển thư mục (`cd ..` lùi về thư mục cha).
  * `mkdir -p /opt/quickbite/config`: Tạo thư mục (cờ `-p` tạo luôn các thư mục cha nếu chưa có).
  * `cat [tên_file]`: Đọc nhanh nội dung file. `nano [tên_file]`: Soạn thảo chỉnh sửa file cấu hình trực tiếp trên terminal.
  * `cp` (copy file), `mv` (di chuyển/đổi tên file), `rm -rf` (xóa vĩnh viễn thư mục).
* **Quản lý tiến trình ứng dụng Java Spring Boot:**
  * Tìm ID tiến trình (PID) Java đang chạy: `ps -ef | grep java`
  * Dừng khẩn cấp tiến trình khi bị sập ngầm: `kill -9 [PID]`

#### Slide 14: Quản lý Biến môi trường lâu dài & Điều khiển dịch vụ
* **Thiết lập biến môi trường lâu dài vĩnh viễn (User Profile):**
  * Ghi dòng khai báo biến môi trường vào file cấu hình ẩn `.bashrc` nằm ở thư mục Home của user:
    ```bash
    echo "export QUICKBITE_DB_USER=staging_admin" >> ~/.bashrc
    
    # Nạp lại cấu hình ngay lập tức cho terminal hiện tại mà không cần logout
    source ~/.bashrc
    ```
* **Quản lý dịch vụ hệ thống bằng Systemd (`systemctl`):**
  * Khởi chạy/Dừng dịch vụ: `sudo systemctl start/stop quickbite-user`
  * Khởi động lại/Xem trạng thái: `sudo systemctl restart/status quickbite-user`
  * Cho phép tự khởi động cùng OS khi reboot server: `sudo systemctl enable quickbite-user`
* **Lệnh điều khiển máy chủ Nginx:**
  * `sudo nginx -t`: **(Cực kỳ quan trọng)** Kiểm tra lỗi cú pháp file cấu hình Nginx xem có sai sót gì không trước khi áp dụng thực tế.
  * `sudo nginx -s reload`: Tải lại cấu hình mới mà không cần restart máy chủ, đảm bảo không ngắt quãng kết nối của khách hàng đang truy cập.

---

### LESSON 06: Quản lý quyền và các lệnh mạng cơ bản trong Linux

#### Slide 15: Tạo người dùng hệ thống chuyên dụng & Bảo mật phân quyền
* **Mối nguy hiểm khi chạy app bằng quyền `root`:** User `root` có quyền tối cao can thiệp vào toàn bộ hệ điều hành. Nếu hacker khai thác được lỗ hổng bảo mật trong code của ứng dụng Java, hacker sẽ chiếm quyền kiểm soát dưới danh nghĩa user `root` và có quyền xóa sạch dữ liệu hoặc chiếm luôn máy chủ vật lý.
* **Tạo nhóm và user hệ thống chuyên dụng bảo mật:**
  ```bash
  # 1. Tạo nhóm quickbite
  sudo groupadd quickbite
  
  # 2. Tạo tài khoản hệ thống chuyên dụng chạy ứng dụng
  sudo useradd -r -g quickbite -s /bin/false quickbite
  ```
  *Ý nghĩa tham số:*
  * `-r`: Đánh dấu là system account, không tạo thư mục home, dùng riêng chạy background service.
  * `-s /bin/false`: Vô hiệu hóa login shell. **Không ai có thể đăng nhập hoặc dùng SSH kết nối vào server bằng user này**, ngăn chặn hacker chiếm quyền shell tương tác của server nếu ứng dụng bị tấn công.

#### Slide 16: Thực hành phân quyền thư mục ứng dụng
* Thiết lập thư mục an toàn chạy dịch vụ `user-service`:
  ```bash
  # 1. Tạo thư mục chứa ứng dụng
  sudo mkdir -p /opt/quickbite/user-service
  
  # 2. Đổi chủ sở hữu toàn bộ thư mục sang user và group quickbite
  sudo chown -R quickbite:quickbite /opt/quickbite
  
  # 3. Phân quyền truy cập thư mục tối giản và chặt chẽ nhất
  sudo chmod -R 750 /opt/quickbite
  ```
  *Giải thích quyền 750:*
  * **Chủ sở hữu (quickbite user):** Đầy đủ quyền Đọc, Ghi, Thực thi (7 -> `rwx`).
  * **Nhóm sở hữu (quickbite group):** Có quyền Đọc và Thực thi để chạy file (5 -> `r-x`).
  * **Những người dùng khác (others):** Không có bất kỳ quyền truy cập nào (0 -> `---`), bảo vệ tuyệt đối các file chứa thông tin nhạy cảm.
  * *Cấp quyền chạy script build:* `chmod +x gradlew` (cho phép chạy build script).

#### Slide 17: Chẩn đoán lỗi mạng & Xử lý xung đột cổng mạng vật lý
* **Các lệnh chẩn đoán mạng cơ bản:**
  * `ip addr`: Kiểm tra địa chỉ IP hiện tại của server Linux.
  * `ping 10.0.1.20`: Kiểm tra đường truyền vật lý tới máy chủ database xem có bị đứt kết nối không.
  * `curl -I https://api.quickbite.com`: Gửi HTTP request nhanh kiểm tra dịch vụ backend có phản hồi không.
* **Xử lý sự cố trùng cổng mạng (Ví dụ: cổng 8080 đang bị chiếm dụng):**
  * Khi khởi chạy Spring Boot báo lỗi: `Web server failed to start. Port 8080 was already in use.`
  * Lệnh kiểm tra tiến trình đang chiếm cổng:
    ```bash
    sudo ss -tulpn | grep :8080
    ```
    *Kết quả hiển thị:*
    `tcp LISTEN 0 100 *:8080 *:* users:(("java",pid=14522,fd=15))`
  * Gõ lệnh kill tiến trình cũ để giải phóng cổng 8080:
    ```bash
    sudo kill -9 14522
    ```
