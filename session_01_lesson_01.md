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

Hãy tưởng tượng hệ thống QuickBite của chúng ta đang bắt đầu triển khai với dịch vụ đầu tiên là `user-service` (quản lý người dùng). Để chạy thử trên môi trường thực tế, chúng ta được cấp một máy chủ ảo (VPS) cấu hình cực kỳ tối giản (1 vCPU, 1GB RAM) nhằm tiết kiệm chi phí.

Nếu bạn chọn con đường **triển khai thủ công** - tự mình gõ lệnh, tự copy file, tự cài đặt hệ thống - bạn sẽ lập tức rơi vào những kịch bản "dở khóc dở cười" dưới đây:

#### Kịch bản 1: Vòng lặp build-deploy vô tận chỉ vì... một dấu phẩy
Bạn phát hiện một lỗi chính tả nhỏ trên API hoặc chỉ muốn sửa đổi một dấu phẩy trong dòng log của `user-service`. 
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
Một ngày đẹp trời, bạn deploy `user-service` lên server chạy thật (Production) nhưng sơ ý lấy nhầm file cấu hình của máy cá nhân (chứa URL database trỏ về `localhost` hoặc database test của Dev). 
* **Hậu quả:** Ứng dụng chạy trên server thật nhưng cố gắng kết nối và đọc/ghi dữ liệu vào database test ở máy của bạn. Dữ liệu của khách hàng thật bị xáo trộn, thông tin bảo mật bị rò rỉ, và tệ nhất là database dev của bạn bị phá hủy hoàn toàn bởi lượng request thực tế từ người dùng.

#### Kịch bản 3: Lệch pha môi trường chạy (Java Runtime Mismatch)
Ở máy cá nhân, bạn cài Java 21 để viết code Spring Boot, sử dụng các tính năng thời thượng như Virtual Threads.
Khi lên server Linux, bạn cài đặt Java bằng lệnh mặc định của hệ điều hành (`apt install default-jdk`) mà không kiểm tra kỹ. Server thực tế chỉ được cài Java 17 hoặc thậm chí Java 11.
* **Hậu quả:** Khi khởi chạy file JAR trên server, hệ thống ném ra một lỗi kinh điển:
  `java.lang.UnsupportedClassVersionError: Has been compiled by a more recent version of the Java Runtime`.
  Ứng dụng sập ngay lập tức. Bạn hoang mang không biết gỡ Java cũ và cài đặt cấu hình `JAVA_HOME` trên giao diện dòng lệnh (CLI) của Linux như thế nào.

#### Kịch bản 4: Hỗn loạn phiên bản đóng gói (File Naming Chaos)
Không có quy trình quản lý tự động, bạn tự copy file lên server và bắt đầu đặt tên file một cách vô tội vạ để backup:
`user-service-0.0.1.jar`, `user-service-final.jar`, `user-service-final-v2.jar`, `user-service-fixed-bug.jar`...
Sau một tuần, khi hệ thống gặp lỗi nghiêm trọng và cần quay lại phiên bản cũ ổn định (Rollback), cả đội nhìn vào thư mục server và hoàn toàn bất lực: Không ai biết file nào thực sự đang chạy, file nào tương ứng với commit nào trên Git, và file nào an toàn để khôi phục.

#### Kịch bản 5: "Mù" thông tin khi có sự cố (Log Blindness)
Ứng dụng sập ngầm ở nền sau (background) của server. Khách hàng bắt đầu khiếu nại vì không đặt được món ăn. 
Vì triển khai thủ công và không có hệ thống ghi log tập trung, bạn phải SSH vào server, lò dò gõ lệnh tìm file log. Bạn mở ra một file log thô nặng vài GB, chữ chạy hoa mắt và không biết tìm từ khóa nào để chẩn đoán lỗi. Trong lúc bạn đang "mù thông tin", hệ thống tiếp tục chết và doanh nghiệp đang mất tiền theo từng giây.

---

### 4. Kiến thức cốt lõi

Triển khai thủ công bộc lộ rõ những hạn chế chí mạng về **Tốc độ**, **Độ ổn định**, và **Bảo mật**. Để giải quyết triệt để, chúng ta cần phân rã quy trình truyền thống và tiếp cận tư duy **DevOps**.

#### 4.1 Quy trình bàn giao truyền thống (Traditional Software Delivery Flow)
Quy trình truyền thống gồm các bước thao tác bằng tay tuần tự:

```text
┌──────────────┐      ┌────────────────┐      ┌─────────────────────┐      ┌─────────────┐
│  Mã nguồn    │ ──►  │ Build Artifact │ ──►  │ Deliver (Copy file) │ ──►  │ Run Systemd │
│ (Local Code) │      │  (File JAR)    │      │    (Local -> Srv)   │      │  (Restart)  │
└──────────────┘      └────────────────┘      └─────────────────────┘      └─────────────┘
```

* **Build Artifact (Đóng gói):** Lập trình viên tự chạy Gradle hoặc Maven ở máy local để đóng gói mã nguồn thành file JAR/WAR.
* **Deliver (Truyền tải):** Chuyển file đóng gói qua mạng internet bằng các giao thức thủ công như SCP hoặc FTP lên máy chủ.
* **Configure (Cấu hình):** Thiết lập thủ công các thông số kết nối Database, cổng mạng và biến môi trường ngay trên hệ điều hành của máy chủ.
* **Execute (Khởi chạy):** Khởi chạy tiến trình Java và quản lý việc chạy ngầm hoặc tự động khởi động lại bằng Systemd hoặc các script tự viết.

Trong mô hình này, tồn tại một **"Bức tường ngăn cách" (Wall of Confusion)** giữa đội Phát triển (Dev) và Vận hành (Ops). Dev chỉ tập trung viết code rồi "ném" file JAR qua bức tường cho Ops chạy. Khi hệ thống lỗi, Dev đổ cho Ops cấu hình server sai, Ops đổ cho Dev viết code lỗi. Quy trình bàn giao đứt gãy và thủ công này chính là nguyên nhân gây ra sự chậm trễ và bất ổn định.

#### 4.2 DevOps không chỉ là một công việc (DevOps is not just a job)
**DevOps** (kết hợp của **Dev**elopment và **Op**eration**s**) sinh ra để phá bỏ bức tường ngăn cách đó. Tuy nhiên, bạn cần hiểu đúng: **DevOps không đơn thuần là một vị trí công việc, một phòng ban mới hay một checklist nhiệm vụ được giao.** 

Bản chất của DevOps đối với một lập trình viên là **sự am hiểu tổng quan về hệ thống thực tế**, **sự hiểu biết sâu sắc về chính ngôn ngữ lập trình bạn sử dụng (Java)** và đích đến cuối cùng là xây dựng được **Đường ống tự động hóa (Automated Pipeline)**:

1. **Am hiểu hệ thống thực tế:** Bạn phải hiểu rõ ứng dụng của mình "sống" như thế nào ngoài đời thực. Nó kết nối đến cơ sở dữ liệu (Database) như thế nào? Tiến trình Java chạy ngầm trên Linux ra sao? Khi hệ thống gặp sự cố, làm sao để cấu hình ghi nhận log và giám sát tài nguyên nhằm chẩn đoán lỗi nhanh nhất?
2. **Hiểu biết sâu sắc về ngôn ngữ phát triển (Java/Spring Boot):** Đối với lập trình viên Java, DevOps đòi hỏi bạn phải làm chủ được cách Java vận hành ở môi trường thực tế:
   - Hiểu rõ cơ chế quản lý bộ nhớ của **JVM (Java Virtual Machine)**: Heap size, Metaspace hoạt động thế nào trên môi trường container và các VPS cấu hình yếu (ví dụ: cách cấu hình RAM cho JVM để tránh bị Linux Kernel giết tiến trình vì lỗi `Out of Memory` - OOM Killer).
   - Hiểu rõ Spring Boot đọc và ghi đè cấu hình thế nào (`application.yml` nạp cấu hình động từ biến môi trường của hệ điều hành).
   - Hiểu rõ các tối ưu hóa ở tầng mã nguồn (ví dụ: cách tối ưu hóa kết nối Database Pool) ảnh hưởng trực tiếp đến hiệu năng của server 1 vCPU và 1GB RAM như thế nào.

3. **Sản phẩm cốt lõi của DevOps - Đường ống tự động (Automated Pipeline):** 
   Một trong những thành quả thực tế và quan trọng nhất mà DevOps mang lại là thiết lập được các **đường ống tự động hóa (CI/CD Pipeline)**. Đây giống như một dây chuyền lắp ráp tự động trong nhà máy, thay thế hoàn toàn cho các thao tác thủ công cồng kềnh:
   
   * **Git Commit & Push (Kích hoạt):** Lập trình viên chỉ cần đẩy code mới lên Git repository. Sự kiện này sẽ tự động kích hoạt (trigger) toàn bộ đường ống chạy mà không cần con người can thiệp.
   * **Build & Unit Test (Kiểm soát chất lượng):** Đường ống tự động kéo code về, biên dịch ra file thực thi và chạy toàn bộ các bài kiểm thử tự động (Unit Test). Nếu phát hiện lỗi logic, đường ống sẽ dừng lại ngay lập tức và báo còi (Fail-fast) cho lập trình viên sửa.
   * **Package & Deliver (Đóng gói & Phân phối):** Nếu code vượt qua vòng kiểm thử, hệ thống tự động đóng gói ứng dụng thành file JAR (hoặc Docker Image) và đẩy lên kho lưu trữ tập trung.
   * **Execute Server Script (Triển khai & Khởi chạy):** Đường ống tự động chạy các script đã được thiết lập sẵn trên server từ xa để dọn dẹp file cũ, đặt file JAR mới vào vị trí và ra lệnh cho Systemd restart ứng dụng một cách trơn tru.

DevOps là sự dịch chuyển văn hóa và tư duy: Lập trình viên không chỉ biết viết code chạy được trên localhost, mà phải am hiểu cách dòng code đó được đóng gói, chạy qua đường ống kiểm soát tự động và vận hành ổn định trên máy chủ thực tế.

> [!TIP]
> **Image Prompt gợi ý:**
> A modern architectural diagram showing a continuous infinity loop of DevOps (Plan, Code, Build, Test, Release, Deploy, Operate, Monitor). The loop is clean, glowing in blue and purple lights, with arrows indicating automation flowing smoothly without human friction. Minimalist high-tech dashboard style.

---

### 5. Thực hành: Trải nghiệm Triển khai Thủ công trên Server

Để hiểu DevOps quý giá thế nào, trước tiên bạn phải tự mình trải nghiệm cảm giác "ăn hành" khi triển khai thủ công dịch vụ `user-service` trên server Ubuntu.

> [!IMPORTANT]
> **Lưu ý về Môi trường thực hành (Giả định mặc định):**
> Ở buổi học đầu tiên này, mục tiêu lớn nhất của chúng ta là thấu hiểu **luồng di chuyển của file** và **nỗi đau của các thao tác thủ công**, chứ không phải cấu hình hạ tầng. 
> * Hãy **giả định mặc định** rằng Staging Server (Linux) đã được đội vận hành (Ops) dựng sẵn, cài sẵn Java và cấp cho bạn thông tin kết nối SSH. Bạn tạm thời sử dụng các lệnh `scp`, `ssh` như một "hộp đen" có sẵn.
> * Toàn bộ quy trình tự mua VPS, thiết lập Linux, cấu hình SSH Key và Firewall từ con số 0 sẽ được hướng dẫn rất chi tiết tại **Session 10 (Triển khai hệ thống lên VPS)**.

#### 5.1 Cấu hình chạy dịch vụ trên máy chủ (Linux Systemd vs Windows Server Service)
Trong thực tế phát triển phần mềm, Linux (như Ubuntu/Debian) là hệ điều hành máy chủ tiêu chuẩn của DevOps do tính gọn nhẹ và bảo mật. Tuy nhiên, nếu doanh nghiệp của bạn bắt buộc phải sử dụng **Windows Server**, quy trình khởi chạy tiến trình sẽ rất khác:
* **Trên Windows Server:** Bạn không có Systemd. Thay vào đó, bạn sẽ phải dùng công cụ bên thứ ba như **NSSM (Non-Sucking Service Manager)** để "bọc" lệnh `java -jar` thành một **Windows Service** chạy ngầm, hoặc cấu hình chạy qua **Task Scheduler**.
* **Trên Linux Server (Hệ điều hành chuẩn của khóa học):** Chúng ta sử dụng **Systemd** - bộ quản lý hệ thống mặc định của Linux. Ta sẽ tạo một file cấu hình dịch vụ tại đường dẫn `/etc/systemd/system/quickbite-user.service` trên server để quản lý tiến trình chạy ngầm của `user-service`:

```ini
[Unit]
Description=QuickBite User Service
After=syslog.target network.target

[Service]
User=quickbite
Type=simple
WorkingDirectory=/opt/quickbite
ExecStart=/usr/bin/java -jar /opt/quickbite/user-service-0.0.1.jar
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

#### 5.2 Các bước cập nhật thủ công từ Local lên Server

**Bước 1: Đóng gói ứng dụng tại máy Local**
Di chuyển vào thư mục code và build file JAR:
```bash
cd c:/Users/Nathan/java_backend_devops/user-service
./gradlew clean bootJar
```
*Kết quả:* File `user-service-0.0.1.jar` được tạo tại `build/libs/`.

**Bước 2: Truyền file JAR lên Server qua lệnh SCP**
```bash
# Đẩy file JAR lên thư mục tạm /tmp trên server (IP giả định: 10.0.1.15)
scp build/libs/user-service-0.0.1.jar user@10.0.1.15:/tmp/
```

**Bước 3: SSH vào Server để di chuyển file và cấp quyền**
```bash
# Kết nối tới server
ssh user@10.0.1.15

# Di chuyển file từ thư mục tạm sang thư mục ứng dụng chính thức
sudo mv /tmp/user-service-0.0.1.jar /opt/quickbite/

# Cấp quyền sở hữu cho user chạy dịch vụ
sudo chown quickbite:quickbite /opt/quickbite/user-service-0.0.1.jar
```

**Bước 4: Khởi động lại dịch vụ bằng Systemd**
```bash
# Reload cấu hình Systemd
sudo systemctl daemon-reload

# Restart dịch vụ để tải file JAR mới
sudo systemctl restart quickbite-user

# Kiểm tra trạng thái hoạt động
sudo systemctl status quickbite-user
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

#### Câu 2 (Đọc và phán đoán ý nghĩa cấu hình)
Dưới đây là một đoạn trích từ file cấu hình Systemd mẫu đã học trong bài:
```ini
[Service]
WorkingDirectory=/opt/quickbite
ExecStart=/usr/bin/java -jar /opt/quickbite/user-service-0.0.1.jar
Restart=always
RestartSec=10
```
Dựa vào các từ khóa tiếng Anh có sẵn, hãy phán đoán xem các dòng cấu hình trên có ý nghĩa gì đối với ứng dụng Java của chúng ta khi chạy trên server.
* *Gợi ý:*
  - `WorkingDirectory=/opt/quickbite`: Chỉ định thư mục làm việc chính thức của ứng dụng trên máy chủ Linux.
  - `ExecStart=...`: Lệnh thực thi để khởi chạy ứng dụng (ở đây là lệnh chạy file Java JAR quen thuộc `java -jar`).
  - `Restart=always`: Cấu hình cho phép Systemd tự động khởi động lại ứng dụng nếu nó bị crash hoặc dừng đột ngột.
  - `RestartSec=10`: Thời gian chờ 10 giây trước khi Systemd tiến hành chạy lại ứng dụng sau khi sập.

#### Câu 3 (Xử lý tình huống thực tế)
Một lập trình viên vừa hoàn thành sửa code cho tính năng đăng ký người dùng của `user-service` ở máy local và muốn cập nhật lên server Staging. Theo đúng quy trình triển khai thủ công đã học, lập trình viên này cần thực hiện các hành động sau:
* (a) Kết nối SSH vào server Linux để reload cấu hình và restart dịch vụ.
* (b) Sử dụng lệnh SCP để truyền tải file JAR mới từ máy local lên server Staging.
* (c) Chạy lệnh biên dịch và đóng gói mã nguồn thành file JAR tại máy local.

Hãy sắp xếp thứ tự thực hiện đúng của các hành động trên để hoàn tất việc cập nhật.
* *Gợi ý:* Thứ tự đúng là **(c) -> (b) -> (a)**. (Đóng gói tại local trước -> Truyền file đóng gói lên server -> Kết nối vào server để kích hoạt chạy file mới).
