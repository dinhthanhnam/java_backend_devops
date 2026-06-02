# SESSION 01: TỔNG QUAN DEVOPS & QUY TRÌNH CI/CD

## LESSON 01: Tổng quan DevOps và hạn chế của triển khai thủ công

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, sinh viên có khả năng:
* **Phân tích** được quy trình bàn giao phần mềm truyền thống từ máy local của lập trình viên lên môi trường máy chủ chạy thật.
* **Chỉ ra và đánh giá** được các hạn chế cốt lõi (tốc độ, độ ổn định, rủi ro lỗi con người) của phương thức triển khai thủ công.
* **Giải thích** được định nghĩa, mục tiêu của DevOps và vị trí của nó trong vòng đời phát triển phần mềm (SDLC).
* **Vận hành** được các câu lệnh cơ bản để đóng gói ứng dụng Spring Boot và quản lý tiến trình bằng Systemd trên server Linux.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (THE QUICKBITE SCENARIO)

Trong quá trình phát triển (development), chúng ta thường làm việc trong một môi trường lý tưởng: Máy tính cá nhân dư dả tài nguyên (16GB - 32GB RAM, CPU nhiều nhân), hệ điều hành có giao diện đồ họa (GUI) trực quan. Chúng ta mở 4 tab IDE để phát triển và chạy 4 chương trình Spring Boot của hệ thống QuickBite cùng lúc, mọi thứ diễn ra rất mượt mà chỉ bằng một nút bấm "Run" hoặc "Debug".

Nhưng có bao giờ bạn tự đặt câu hỏi:
* Liệu chúng ta đã thực sự hiểu cách chạy một ứng dụng Java từ file JAR ở môi trường thực tế diễn ra như thế nào chưa?
* Khi đã biết lệnh chạy một file Java bằng dòng lệnh `java -jar app.jar`, liệu nó đã giải quyết được trọn vẹn bài toán triển khai hệ thống hay chưa?

Câu trả lời chắc chắn là **Chưa**. Môi trường triển khai (Production/Staging Server) hoàn toàn khác biệt so với môi trường phát triển của bạn. Đó là một thế giới không có giao diện đồ họa, tài nguyên bị giới hạn nghiêm ngặt và yêu cầu tính sẵn sàng liên tục.

#### Tình huống thực tế với hệ thống QuickBite
Hãy tưởng tượng hệ thống QuickBite lúc này có 4 dịch vụ độc lập (`user-service`, `restaurant-service`, `order-service`, `notification-service`). Để tiết kiệm chi phí, đội ngũ quyết định triển khai theo cách truyền thống: mỗi service chạy trên một con máy chủ ảo (VPS hoặc EC2) cực kỳ nhỏ với cấu hình tối thiểu (1 vCPU và 1GB RAM).

Quy trình triển khai thủ công này lập tức vấp phải những vấn đề "điên đầu" sau:
1. **Server quá yếu, không thể tự build:** Với 1GB RAM, VPS không thể tự chạy các lệnh biên dịch nặng nề như `./gradlew build`. Chỉ cần chạy build, server sẽ bị tràn RAM và sập ngay lập tức (Out of Memory). Do đó, bạn bắt buộc phải chạy build trên máy cá nhân, sau đó copy file JAR nặng hàng chục MB lên server.
2. **Quy trình cập nhật thủ công cồng kềnh:** Để cập nhật một tính năng mới, bạn phải:
   - Xóa file build cũ trên server (đồng nghĩa với việc phiên bản cũ mất đi hoàn toàn, nếu phiên bản mới bị lỗi thì không còn đường lui).
   - Đặt file JAR mới vào đúng vị trí.
   - Khởi động lại dịch vụ bằng lệnh Systemd (dù việc quản lý tiến trình bằng Systemd tạm ổn, nhưng mọi thao tác gõ lệnh vẫn phải làm bằng tay).
3. **Bài toán kết nối mạng (Networking) nhức óc:** 4 dịch vụ này nằm trên 4 VPS khác nhau. Để chúng có thể giao tiếp với nhau, bạn phải tìm đúng địa chỉ IP của từng service để khai báo cấu hình kết nối (đấy là còn giả định rằng các VPS có IP tĩnh và bạn chỉ cần cấu hình một lần). Mỗi lần một VPS bị thay đổi IP hoặc thay đổi cấu hình, bạn sẽ phải vào từng server còn lại để cập nhật cấu hình bằng tay.

Đây chính là lúc bạn nhận ra: **DevOps không phải là tên của một vị trí công việc hay chức danh.** DevOps là hiểu rõ kiến trúc hệ thống, có cái nhìn tổng quan toàn diện, và gắn liền quá trình phát triển (Dev) với quá trình triển khai (Ops) để loại bỏ hoàn toàn câu nói quen thuộc nhưng đầy vô trách nhiệm: *"Nhưng nó chạy bình thường trên máy của em mà!"*

---

### PHẦN 3. KIẾN THỨC CỐT LÕI

#### 3.1 Quy trình triển khai ứng dụng truyền thống (Traditional Software Delivery Flow)
Trong môi trường truyền thống, quy trình đưa mã nguồn từ máy local lên server vận hành qua 4 giai đoạn chính:

```text
┌──────────────┐      ┌────────────────┐      ┌─────────────────────┐      ┌─────────────┐
│  Mã nguồn   │ ──►  │ Build Artifact │ ──►  │ Deliver (Copy file) │ ──►  │ Run Systemd │
│ (Local Code) │      │  (File JAR)    │      │    (Local -> Srv)   │      │  (Restart)  │
└──────────────┘      └────────────────┘      └─────────────────────┘      └─────────────┘
```

1. **Build Artifact (Đóng gói):** Lập trình viên biên dịch mã nguồn Java và đóng gói thành file lưu trữ chạy được (JAR/WAR đối với Spring Boot).
2. **Deliver (Bàn giao/Truyền tải):** Chuyển file đóng gói đó qua mạng internet từ máy tính cá nhân lên ổ đĩa của máy chủ.
3. **Configure (Cấu hình):** Thiết lập các biến môi trường, kết nối database và cấp quyền chạy file trên hệ điều hành máy chủ.
4. **Execute (Khởi chạy):** Quản lý tiến trình (Process Management) của ứng dụng bằng các bộ quản lý dịch vụ của hệ điều hành (như Systemd trên Linux).

#### 3.2 Hạn chế của triển khai thủ công
* **Tốc độ chậm (Low Velocity):** Thời gian chờ build, truyền tải và thao tác bằng tay chiếm phần lớn thời gian phát hành.
* **Rủi ro sai sót con người (Human Errors):** Mọi thao tác gõ dòng lệnh trực tiếp (CLI) đều có thể dẫn đến sai sót. Chỉ cần quên cập nhật một biến môi trường, ứng dụng sẽ chạy sai logic.
* **Môi trường không đồng nhất (Environmental Drift):** Cấu hình máy local của dev khác với Staging, khác với Production. Dẫn đến hiện tượng kinh điển: *"Code chạy tốt trên máy của em, nhưng lên server thì sập!"*
* **Khó khôi phục (Hard to Rollback):** Khi phiên bản mới gặp sự cố nghiêm trọng, việc quay về phiên bản cũ (Rollback) đòi hỏi phải lặp lại các bước thủ công tương tự, làm tăng thời gian sập hệ thống (Downtime).

#### 3.3 Khái niệm DevOps và Vòng đời Phát triển
**DevOps** là sự kết hợp từ viết tắt của **Dev**elopment (Phát triển phần mềm) và **Op**eration**s** (Vận hành hệ thống).

```text
                  VÒNG ĐỜI DEVOPS (VÒNG LẶP VÔ CỰC)
                  
        ┌───────────────────────────────────────────┐
        │     Plan  ──►  Code  ──►  Build  ──►  Test│  (DEV)
        ▼                                           │
      Deploy  ◄──  Release  ◄──  Monitor  ◄──Operate│  (OPS)
        └───────────────────────────────────────────┘
```

* **Bản chất:** DevOps là sự kết hợp của văn hóa làm việc, quy trình cộng tác và các công cụ tự động hóa nhằm rút ngắn thời gian phát hành tính năng (Time-to-market) mà vẫn đảm bảo độ ổn định cao nhất của hệ thống.
* **Vị trí của DevOps:** DevOps lấp đầy khoảng trống đứt gãy thông tin giữa nhóm Dev (muốn thay đổi nhanh) và nhóm Ops (muốn hệ thống ổn định, ngại thay đổi). Nó biến các bước thủ công riêng lẻ thành một chuỗi tự động hóa khép kín.

---

### PHẦN 4. MINH HỌA HOẶC DEMO (THỰC HÀNH TRIỂN KHAI THỦ CÔNG)

Chúng ta sẽ thực hiện việc cập nhật thủ công logic API của `order-service` trên hệ thống QuickBite từ máy local lên máy chủ thử nghiệm chạy hệ điều hành Linux Ubuntu.

#### 4.1 Cấu hình Systemd trên Server (`/etc/systemd/system/quickbite-order.service`)
Để quản lý tiến trình chạy ngầm của ứng dụng Spring Boot và tự khởi động lại nếu ứng dụng bị crash, trên server đã được cấu hình dịch vụ Systemd như sau:

```ini
[Unit]
Description=QuickBite Order Service
After=syslog.target network.target

[Service]
User=quickbite
Type=simple
WorkingDirectory=/opt/quickbite
ExecStart=/usr/bin/java -jar /opt/quickbite/order-service-0.0.1.jar
SuccessExitStatus=143
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

#### 4.2 Các lệnh thực hiện cập nhật thủ công

**Bước 1: Đóng gói mã nguồn tại máy Local**
Sử dụng công cụ Gradle của Spring Boot để biên dịch và đóng gói ứng dụng thành file JAR:
```bash
# Di chuyển vào thư mục của order-service trên máy local
cd c:/Users/Nathan/backend_java_devops/java_backend_devops/order-service

# Tiến hành dọn dẹp thư mục build cũ và đóng gói file JAR mới
./gradlew clean bootJar
```
*Kết quả:* Một file `order-service-0.0.1.jar` được tạo ra trong thư mục `build/libs/`.

**Bước 2: Truyền file JAR mới lên Server qua mạng bằng lệnh SCP**
```bash
# Đẩy file JAR từ local lên thư mục tạm /tmp trên server (IP giả định: 10.0.1.15)
scp build/libs/order-service-0.0.1.jar user@10.0.1.15:/tmp/
```
*(Yêu cầu người vận hành phải nhập mật khẩu SSH của server hoặc đã cài đặt sẵn SSH Key).*

**Bước 3: Kết nối SSH vào Server và chuẩn bị dịch vụ**
```bash
# SSH vào máy chủ Linux
ssh user@10.0.1.15

# [Tại Server Terminal]
# Di chuyển file JAR mới từ thư mục tạm /tmp vào thư mục hoạt động chính thức
sudo mv /tmp/order-service-0.0.1.jar /opt/quickbite/

# Đảm bảo file JAR thuộc sở hữu của user chạy service để bảo mật
sudo chown quickbite:quickbite /opt/quickbite/order-service-0.0.1.jar
```

**Bước 4: Khởi động lại dịch vụ bằng Systemd**
```bash
# Yêu cầu Systemd reload lại cấu hình nếu có thay đổi
sudo systemctl daemon-reload

# Restart dịch vụ order-service để dừng JAR cũ, khởi chạy JAR mới
sudo systemctl restart quickbite-order

# Kiểm tra xem dịch vụ đã khởi chạy thành công hay chưa
sudo systemctl status quickbite-order
```
*Kết quả mong đợi trên màn hình:* Dịch vụ có trạng thái `active (running)`.

```text
● quickbite-order.service - QuickBite Order Service
     Loaded: loaded (/etc/systemd/system/quickbite-order.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2026-06-02 13:50:00 UTC; 12s ago
   Main PID: 12453 (java)
      Tasks: 28 (limit: 2340)
     Memory: 312.4M
```

**Bước 5: Kiểm tra kết quả cập nhật API**
```bash
# Gọi API lấy phiên bản từ máy host hoặc từ client ngoài internet
curl http://10.0.1.15:8081/api/v1/orders/version
```
*Phản hồi trả về:* `{"version": "v1.1-ship-discount", "status": "active"}`. Tính năng mới đã chính thức hoạt động!

---

### PHẦN 5. TỔNG KẾT

* **Quy trình truyền thống:** Luôn yêu cầu thao tác tuần tự từng bước bằng tay (Build local -> Copy file -> SSH -> Command shell -> Restart).
* **Điểm yếu cốt lõi:** Tốc độ bàn giao cực kỳ chậm, rủi ro sai sót con người rất lớn, và không có tính nhất quán về môi trường chạy.
* **Systemd vs DevOps:** 
  * **Systemd** giải quyết rất tốt bài toán *quản lý vòng đời ứng dụng trên một máy chủ độc lập* (tự khởi chạy cùng OS, tự restart khi crash ứng dụng).
  * Tuy nhiên, **Systemd không thể tự động hóa quy trình** từ khi lập trình viên đẩy code lên Git cho đến khi code đó chạy thành công trên máy chủ. Đó là lý do tại sao chúng ta cần đến các giải pháp DevOps.
* **Định hướng tiếp theo:** Để giải quyết triệt để vấn đề này, bài học sau chúng ta sẽ nghiên cứu mô hình **CI/CD** - chìa khóa vàng giúp tự động hóa toàn bộ 5 bước thủ công trên mà không cần con người can thiệp trực tiếp.

---

### PHẦN 6. CÂU HỎI ĐÁNH GIÁ (ASSESSMENT & SITUATIONAL QUESTIONS)

#### Câu 1 (Kiểm tra khả năng hiểu bản chất)
Trong quy trình cập nhật ứng dụng QuickBite lên server, tại sao việc sử dụng các công cụ quản lý dịch vụ như Systemd (`systemctl restart`) lại không được xem là đã áp dụng quy trình DevOps? Điểm khác biệt mấu chốt ở đây là gì?

*Gợi ý trả lời:* Systemd chỉ là công cụ quản lý tiến trình cục bộ trên một server. Nó không có khả năng tự động hóa việc đồng bộ mã nguồn từ Git, không tự chạy test chất lượng, không đóng gói và không tự truyền tải file. DevOps đòi hỏi một chuỗi tự động hóa liên tục từ khâu code đến khâu monitor (CI/CD Pipeline), chứ không chỉ dừng lại ở lệnh chạy dịch vụ.

#### Câu 2 (Kiểm tra khả năng đọc hiểu & dự đoán kết quả cấu hình)
Cho file cấu hình Systemd của dịch vụ `quickbite-order.service` như sau:
```ini
[Service]
WorkingDirectory=/opt/quickbite
ExecStart=/usr/bin/java -jar /opt/quickbite/order-service-0.0.1.jar
Restart=always
```
Nếu trên server, thư mục `/opt/quickbite` đang thuộc quyền sở hữu của user `root` với quyền hạn truy cập `drwxr-x--- (750)`. 
Chuyện gì sẽ xảy ra nếu lập trình viên thực hiện lệnh chạy service bằng lệnh: `sudo systemctl start quickbite-order` nhưng trong file service khai báo thêm dòng `User=quickbite`? Dịch vụ có khởi chạy thành công không? Giải thích tại sao.

*Gợi ý trả lời:* Dịch vụ sẽ thất bại (`failed` / `exit-code`). Vì cấu hình chỉ định dịch vụ chạy dưới danh nghĩa user `quickbite`. Tuy nhiên thư mục làm việc `/opt/quickbite` lại thuộc sở hữu của `root` và phân quyền `750` (chỉ root và group sở hữu có quyền đọc/thực thi, các user khác như `quickbite` không có quyền truy cập). Khi đó, tiến trình Java chạy bằng user `quickbite` không thể truy cập vào thư mục để đọc file JAR và ghi log.

#### Câu 3 (Kiểm tra khả năng xử lý tình huống thực tế)
Bạn vừa thực hiện triển khai thủ công phiên bản cập nhật của `order-service` lên server. Sau khi restart service bằng Systemd, bạn gõ lệnh kiểm tra:
```bash
sudo systemctl status quickbite-order
```
Hệ thống hiển thị trạng thái `active (running)`. Tuy nhiên, khi bạn truy cập vào ứng dụng qua API Gateway hoặc gọi trực tiếp qua trình duyệt thì nhận được lỗi `502 Bad Gateway` hoặc `Connection Refused`. 

Hãy nêu **3 bước chẩn đoán nhanh** bằng dòng lệnh trên server để xác định nguyên nhân gốc rễ của sự cố này.

*Gợi ý trả lời:*
1. **Bước 1 (Kiểm tra log ứng dụng):** Xem log Spring Boot khởi chạy thực tế bên trong xem có bị crash ngầm hoặc lỗi kết nối database hay không (vì Systemd có thể báo active khi tiến trình java mới khởi động nhưng sau đó java bị văng exception):
   ```bash
   journalctl -u quickbite-order.service -n 100 -f
   ```
2. **Bước 2 (Kiểm tra cổng mạng):** Xác nhận ứng dụng Spring Boot đã lắng nghe (bind) thành công cổng 8081 trên server hay chưa:
   ```bash
   sudo ss -tulpn | grep 8081
   # hoặc
   sudo netstat -tulpn | grep 8081
   ```
3. **Bước 3 (Kiểm tra giao tiếp cục bộ):** Kiểm tra xem lỗi ở bản thân ứng dụng hay do firewall/Gateway chặn bằng cách gọi trực tiếp API ngay tại local của server:
   ```bash
   curl -I http://localhost:8081/api/v1/orders/version
   ```
