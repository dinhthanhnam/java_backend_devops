# SESSION 01: TỔNG QUAN DEVOPS & QUY TRÌNH CI/CD

## LESSON 01: Tổng quan DevOps và hạn chế của triển khai thủ công

---

### 1. Mục tiêu bài học

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Phân tích** được quy trình bàn giao phần mềm truyền thống từ môi trường local lên máy chủ.
* **Chỉ ra và đánh giá** được các hạn chế cốt lõi và rủi ro của phương thức triển khai thủ công.
* **Giải thích** được định nghĩa, mục tiêu của DevOps và vị trí của nó trong vòng đời phát triển phần mềm (SDLC).
* **Vận hành** các câu lệnh cơ bản để đóng gói ứng dụng Spring Boot và quản lý tiến trình bằng Systemd trên server Linux.

---

### 2. Triển khai phần mềm là gì và sự "ngây thơ" của lập trình viên

Trước nay, khi học lập trình, bạn chỉ mới học về **phát triển** (Development) - tức là viết code, mở IDE, nhấn nút "Run" hoặc "Debug", rồi mở trình duyệt gõ `localhost:8080`. Mọi thứ thật lung linh và hoàn hảo trong chiếc "hộp cát" (sandbox) cá nhân của bạn.

Nhưng hãy tỉnh táo lại: **Không ai trả tiền cho một ứng dụng chỉ chạy được trên localhost của bạn.**

**Triển khai phần mềm (Deployment)** là quá trình đưa ứng dụng từ máy tính của bạn đến máy chủ thực tế (Production Server) để người dùng cuối có thể truy cập, sử dụng và đem lại doanh thu cho doanh nghiệp. Một lập trình viên giỏi không chỉ biết viết code chạy được, mà phải hiểu cách đưa sản phẩm đó ra thế giới thực. Nếu bạn nghĩ code chạy mượt trên máy cá nhân đã là xong việc, bạn vẫn còn cực kỳ "ngây thơ" và "khờ khạo" về quy trình làm phần mềm thực tế.

> [!TIP]
> **Image Prompt gợi ý:**
> A split-screen illustration. On the left: a relaxed developer in a cozy, bright room looking at a glowing monitor displaying a simple green "localhost:8080" web page in a clean sandbox. On the right: a dark, complex data center with stormy server racks, flashing warning lights, and massive traffic lines, representing the harsh reality of the production environment. Modern tech art style, neon accents.

---

### 3. Vấn đề thực tế: Kịch bản "ăn hành" của QuickBite

Hãy tưởng tượng hệ thống QuickBite có 4 dịch vụ độc lập: `user-service`, `restaurant-service`, `order-service`, và `notification-service`. Để chạy thử trên môi trường thực tế, chúng ta được cấp một số máy chủ ảo (VPS) cấu hình cực kỳ tối giản (1 vCPU, 1GB RAM) nhằm tiết kiệm chi phí.

Nếu bạn chọn con đường **triển khai thủ công** - tự mình gõ lệnh, tự copy file, tự cài đặt hệ thống - bạn sẽ lập tức rơi vào những kịch bản "dở khóc dở cười" dưới đây:

#### Kịch bản 1: Vòng lặp build-deploy vô tận chỉ vì... một dấu phẩy
Bạn phát hiện một lỗi chính tả nhỏ trên API hoặc chỉ muốn sửa đổi một dấu phẩy trong dòng log của `order-service`. 
Quy trình thủ công bắt đầu:
1. Chạy lệnh đóng gói tại máy local (`./gradlew clean bootJar`) -> chờ 3 phút.
2. Dùng lệnh `scp` đẩy file JAR (dung lượng ~50MB) lên server thông qua đường truyền internet công cộng -> chờ 5 phút.
3. SSH vào server, gõ lệnh reload và restart dịch vụ -> mất 2 phút.
Tổng cộng bạn tốn **10 phút** cuộc đời chỉ để sửa một dấu phẩy! Nếu phải sửa 10 lần một ngày, bạn sẽ không còn thời gian để làm việc gì khác.

> [!TIP]
> **Image Prompt gợi ý:**
> An anxious developer trapped inside a giant circular loop line. Along the loop, there are icons representing: 1. Editing a tiny comma in code, 2. A slow upload progress bar (SCP), 3. An SSH terminal prompt, 4. A server rebooting. The developer looks exhausted, holding his head. Minimalist vector style, contrasting colors (red and dark grey).

#### Kịch bản 2: "Râu ông này cắm cằm bà kia" (Environment Drift)
Khi triển khai thủ công, bạn phải tự cấu hình các file `application.yml` hoặc nạp biến môi trường bằng tay.
Một ngày đẹp trời, bạn deploy `order-service` lên server chạy thật (Production) nhưng sơ ý lấy nhầm file cấu hình của máy cá nhân (chứa URL database trỏ về `localhost` hoặc database test của Dev). 
* **Hậu quả:** Ứng dụng chạy trên server thật nhưng cố gắng kết nối và đọc/ghi dữ liệu vào database test ở máy của bạn. Dữ liệu của khách hàng thật bị xáo trộn, thông tin bảo mật bị rò rỉ, và tệ nhất là database dev của bạn bị phá hủy hoàn toàn bởi lượng request thực tế từ người dùng.

#### Kịch bản 3: Lệch pha môi trường chạy (Java Runtime Mismatch)
Ở máy cá nhân, bạn cài Java 21 để viết code Spring Boot, sử dụng các tính năng thời thượng như Virtual Threads.
Khi lên server Linux, bạn cài đặt Java bằng lệnh mặc định của hệ điều hành (`apt install default-jdk`) mà không kiểm tra kỹ. Server thực tế chỉ được cài Java 17 hoặc thậm chí Java 11.
* **Hậu quả:** Khi khởi chạy file JAR trên server, hệ thống ném ra một lỗi kinh điển:
  `java.lang.UnsupportedClassVersionError: Has been compiled by a more recent version of the Java Runtime`.
  Ứng dụng sập ngay lập tức. Bạn hoang mang không biết gỡ Java cũ và cài đặt cấu hình `JAVA_HOME` trên giao diện dòng lệnh (CLI) của Linux như thế nào.

#### Kịch bản 4: Hỗn loạn phiên bản đóng gói (File Naming Chaos)
Không có quy trình quản lý tự động, bạn tự copy file lên server và bắt đầu đặt tên file một cách vô tội vạ để backup:
`order-service-0.0.1.jar`, `order-service-final.jar`, `order-service-final-v2.jar`, `order-service-fixed-bug.jar`...
Sau một tuần, khi hệ thống gặp lỗi nghiêm trọng và cần quay lại phiên bản cũ ổn định (Rollback), cả đội nhìn vào thư mục server và hoàn toàn bất lực: Không ai biết file nào thực sự đang chạy, file nào tương ứng với commit nào trên Git, và file nào an toàn để khôi phục.

#### Kịch bản 5: "Mù" thông tin khi có sự cố (Log Blindness)
Ứng dụng sập ngầm ở nền sau (background) của server. Khách hàng bắt đầu khiếu nại vì không đặt được món ăn. 
Vì triển khai thủ công và không có hệ thống ghi log tập trung, bạn phải SSH vào server, lò dò gõ lệnh tìm file log. Bạn mở ra một file log thô nặng vài GB, chữ chạy hoa mắt và không biết tìm từ khóa nào để chẩn đoán lỗi. Trong lúc bạn đang "mù thông tin", hệ thống tiếp tục chết và doanh nghiệp đang mất tiền theo từng giây.

---

### 4. Kiến thức cốt lõi

Triển khai thủ công bộc lộ rõ những hạn chế chí mạng về **Tốc độ**, **Độ ổn định**, và **Bảo mật**. Để giải quyết triệt để, chúng ta cần chuyển đổi tư duy sang **DevOps**.

#### 4.1 Quy trình bàn giao truyền thống vs DevOps
Quy trình truyền thống gồm các bước thao tác bằng tay tuần tự:

```text
┌──────────────┐      ┌────────────────┐      ┌─────────────────────┐      ┌─────────────┐
│  Mã nguồn   │ ──►  │ Build Artifact │ ──►  │ Deliver (Copy file) │ ──►  │ Run Systemd │
│ (Local Code) │      │  (File JAR)    │      │    (Local -> Srv)   │      │  (Restart)  │
└──────────────┘      └────────────────┘      └─────────────────────┘      └─────────────┘
```

* **Build Artifact:** Lập trình viên tự chạy gradle/maven để đóng gói file JAR/WAR.
* **Deliver:** Chuyển file đóng gói qua mạng (SCP, FTP) lên server.
* **Configure:** Thiết lập thủ công các biến môi trường, database url.
* **Execute:** Khởi chạy tiến trình và quản lý bằng Systemd hoặc script khởi động.

**DevOps** (kết hợp của **Dev**elopment và **Op**eration**s**) sinh ra để tự động hóa toàn bộ chuỗi này. DevOps biến các bước thủ công rời rạc thành một chuỗi tự động hóa khép kín (CI/CD Pipeline), giúp mã nguồn từ Git tự động đi thẳng lên máy chủ chạy thật một cách an toàn mà không cần con người trực tiếp can thiệp.

> [!TIP]
> **Image Prompt gợi ý:**
> A modern architectural diagram showing a continuous infinity loop of DevOps (Plan, Code, Build, Test, Release, Deploy, Operate, Monitor). The loop is clean, glowing in blue and purple lights, with arrows indicating automation flowing smoothly without human friction. Minimalist high-tech dashboard style.

---

### 5. Thực hành: Trải nghiệm Triển khai Thủ công trên Server

Để hiểu DevOps quý giá thế nào, trước tiên bạn phải tự mình trải nghiệm cảm giác "ăn hành" khi triển khai thủ công dịch vụ `order-service` trên server Ubuntu.

#### 5.1 Cấu hình Systemd dịch vụ trên Server
Trên server, ta cấu hình file dịch vụ Systemd tại đường dẫn `/etc/systemd/system/quickbite-order.service` để quản lý tiến trình chạy ngầm:

```ini
[Unit]
Description=QuickBite Order Service
After=syslog.target network.target

[Service]
User=quickbite
Type=simple
WorkingDirectory=/opt/quickbite
ExecStart=/usr/bin/java -jar /opt/quickbite/order-service-0.0.1.jar
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

#### 5.2 Các bước cập nhật thủ công từ Local lên Server

**Bước 1: Đóng gói ứng dụng tại máy Local**
Di chuyển vào thư mục code và build file JAR:
```bash
cd c:/Users/Nathan/java_backend_devops/order-service
./gradlew clean bootJar
```
*Kết quả:* File `order-service-0.0.1.jar` được tạo tại `build/libs/`.

**Bước 2: Truyền file JAR lên Server qua lệnh SCP**
```bash
# Đẩy file JAR lên thư mục tạm /tmp trên server (IP giả định: 10.0.1.15)
scp build/libs/order-service-0.0.1.jar user@10.0.1.15:/tmp/
```

**Bước 3: SSH vào Server để di chuyển file và cấp quyền**
```bash
# Kết nối tới server
ssh user@10.0.1.15

# Di chuyển file từ thư mục tạm sang thư mục ứng dụng chính thức
sudo mv /tmp/order-service-0.0.1.jar /opt/quickbite/

# Cấp quyền sở hữu cho user chạy dịch vụ
sudo chown quickbite:quickbite /opt/quickbite/order-service-0.0.1.jar
```

**Bước 4: Khởi động lại dịch vụ bằng Systemd**
```bash
# Reload cấu hình Systemd
sudo systemctl daemon-reload

# Restart dịch vụ để tải file JAR mới
sudo systemctl restart quickbite-order

# Kiểm tra trạng thái hoạt động
sudo systemctl status quickbite-order
```

Nếu màn hình hiển thị trạng thái `active (running)`, ứng dụng đã chạy. Tuy nhiên, nếu bạn sửa đổi tiếp code, bạn sẽ phải lặp lại toàn bộ 4 bước gõ tay này từ đầu!

---

### 6. Tổng kết

* **Triển khai thủ công** là nguồn cơn của sự chậm trễ, sai lệch môi trường (Environment Drift) và rủi ro lỗi con người cực kỳ lớn.
* **Systemd** chỉ giúp quản lý tiến trình cục bộ trên một server (tự restart khi crash) chứ không giải quyết bài toán tự động đưa code từ máy lập trình viên lên môi trường thực tế.
* **Hướng đi tiếp theo:** Chúng ta sẽ nghiên cứu **CI/CD** và **Docker** để giải thoát bản thân khỏi đống lệnh gõ tay nhàm chán này, tự động hóa hoàn toàn quy trình đóng gói và triển khai.

---

### 7. Câu hỏi đánh giá thực chiến

#### Câu 1 (Hiểu bản chất)
Tại sao việc cấu hình Systemd tự khởi động lại ứng dụng (`Restart=always`) trên server vẫn không được coi là đã áp dụng DevOps? Hãy chỉ ra điểm khác biệt cốt lõi.
* *Gợi ý:* Systemd chỉ là công cụ quản lý tiến trình tĩnh tại một server local. Nó không tự động tích hợp mã nguồn từ Git, không tự test chất lượng, và không tự động hóa quy trình bàn giao. DevOps là một chuỗi tự động hóa và cộng tác khép kín từ khâu code đến vận hành thực tế.

#### Câu 2 (Dự đoán lỗi cấu hình)
Nếu thư mục `/opt/quickbite` trên server thuộc sở hữu của user `root` với quyền hạn `750` (`rwxr-x---`). Chuyện gì xảy ra nếu bạn chạy dịch vụ bằng Systemd cấu hình dòng lệnh `User=quickbite`?
* *Gợi ý:* Dịch vụ sẽ thất bại (`failed` / `exit-code`). Vì tiến trình Java được khởi chạy dưới danh nghĩa user `quickbite`, nhưng user này không có quyền đọc/thực thi trong thư mục `/opt/quickbite` thuộc sở hữu của `root` với quyền hạn hạn chế.

#### Câu 3 (Xử lý sự cố thực tế)
Bạn vừa restart `order-service` thủ công trên server. Lệnh `systemctl status` báo trạng thái dịch vụ là `active (running)`. Tuy nhiên, khi gọi API từ trình duyệt thì nhận được lỗi `502 Bad Gateway`. 
Hãy đưa ra **3 câu lệnh chẩn đoán nhanh** ngay trên server để tìm nguyên nhân gốc rễ.
* *Gợi ý:*
  1. Xem log thời gian thực của ứng dụng để kiểm tra lỗi crash ngầm hoặc lỗi kết nối DB: `journalctl -u quickbite-order.service -n 100 -f`
  2. Kiểm tra xem port 8081 của service đã được lắng nghe chưa: `sudo ss -tulpn | grep 8081`
  3. Kiểm tra xem server có thể gọi nội bộ tới service được không: `curl -I http://localhost:8081/api/v1/orders/version`
