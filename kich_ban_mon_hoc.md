# SESSION 01

## TỔNG QUAN DEVOPS & QUY TRÌNH CI/CD

---

### LESSON 01: Tổng quan DevOps và hạn chế của triển khai thủ công

#### 1. Mục tiêu bài học

* **Phân tích** được các bước trong quy trình bàn giao phần mềm truyền thống và vị trí của DevOps trong vòng đời phát triển.
* **Chỉ ra** được các hạn chế về mặt tốc độ và rủi ro sai sót con người khi thực hiện cập nhật ứng dụng thủ công.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** STATE 0 — Local Application. Các dịch vụ của QuickBite (`user-service`, `restaurant-service`, `order-service`, `notification-service`) chạy độc lập, cấu hình trực tiếp trên môi trường local hoặc server phát triển.

* **Vấn đề:** Mỗi lần có bản cập nhật logic tính toán từ đội phát triển, việc bàn giao (deploy) lên server thử nghiệm yêu cầu các thao tác thủ công nối tiếp nhau: build file, truyền tải file, và điều khiển dịch vụ hệ thống. Quy trình này làm chậm tốc độ phản hồi tính năng của hệ thống QuickBite.

#### 3. Nội dung trọng tâm

* **Quy trình triển khai ứng dụng truyền thống:** Luồng di chuyển của mã nguồn từ máy local -> Build Artifact (JAR) -> Chuyển file lên Server -> Khởi chạy/Restart dịch vụ bằng Systemd.
* **Khái niệm DevOps:** Sự kết hợp văn hóa, tư duy tự động hóa giữa nhóm Phát triển (Dev) và Vận hành (Ops) nhằm rút ngắn thời gian phát hành tính năng nhưng vẫn giữ vững độ ổn định của QuickBite.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Minh họa chuỗi thao tác cập nhật một tính năng nhỏ trong `order-service` lên server bằng phương thức thủ công thông qua Systemd.

* **File liên quan:** `order-service-0.0.1.jar`, `quickbite-order.service` (File cấu hình Systemd trên server).
* **Command:**
```bash
# Máy Local: Build ra file thực thi
./gradlew bootJar

# Máy Local: Copy file JAR lên server 
scp build/libs/order-service-0.0.1.jar user@server_ip:/opt/quickbite/

# Trên Server: Restart dịch vụ hệ thống quản lý ứng dụng
ssh user@server_ip "sudo systemctl restart quickbite-order"
```

* **Kết quả mong đợi:** Ứng dụng cập nhật thành công, tuy nhiên người vận hành phải trực tiếp can thiệp bằng dòng lệnh ở từng bước.

#### 5. Điểm cần nhấn mạnh
* Systemd (`systemctl`) giải quyết tốt bài toán quản lý vòng đời ứng dụng trên một server (tự động start cùng OS, tự restart khi crash) nhưng không giải quyết được bài toán tự động hóa quy trình từ lúc code thay đổi trên Git đến khi lên server.
* DevOps không thay thế các công cụ quản trị hệ thống mà tích hợp chúng vào một chuỗi tự động hóa khép kín.

#### 6. Hiểu lầm thường gặp
* **Hiểu sai:** Cho rằng DevOps là thay thế hoàn toàn con người bằng công cụ.
* **Đính chính:** Công cụ chỉ là phương tiện thực thi; bản chất DevOps là tối ưu hóa quy trình làm việc và sự cộng tác giữa các phòng ban.

---

### LESSON 02: Khái niệm CI/CD (quy trình build, test, deploy)

#### 1. Mục tiêu bài học
* **Phân biệt** được các giai đoạn và ranh giới trách nhiệm của Continuous Integration (CI), Continuous Delivery (CD), và Continuous Deployment (CD).
* **Xác định** được các điều kiện cần để một pipeline tự động chuyển trạng thái mã nguồn thành sản phẩm chạy được.

#### 2. Bối cảnh hệ thống
* **Trạng thái:** STATE 0 — Local Application.
* **Vấn đề:** Khi nhiều thành viên cùng tích hợp mã nguồn vào các service như `restaurant-service` hay `user-service`, việc xung đột code (conflict) hoặc lỗi logic phá vỡ tính năng cũ chỉ được phát hiện muộn vào cuối phân đoạn phát triển, gây tốn chi phí sửa chữa.

#### 3. Nội dung trọng tâm
* **Continuous Integration (CI):** Cơ chế tự động kích hoạt quá trình kiểm tra (Build, Compile, Run Unit Test) ngay khi có thay đổi mã nguồn trên Git repository.
* **Continuous Delivery & Continuous Deployment (CD):** 
    *   *Delivery:* Tự động đóng gói và sẵn sàng triển khai, kích hoạt môi trường đích bằng một thao tác phê duyệt (nhấn nút).
    *   *Deployment:* Tự động hóa hoàn toàn luồng phát hành từ kho code thẳng lên môi trường chạy thực tế mà không cần can thiệp thủ công.

#### 4. Demo và thực hành
* **Mục tiêu demo:** Sơ đồ hóa luồng kiểm soát chất lượng tự động (Pipeline Logic) của hệ thống QuickBite từ Git đến Server.
* **Sơ đồ kiến trúc luồng (Pipeline Flow):**
```text
[Developer Git Push] ──► [Git Repository] ──(Webhook Trigger)──► [CI/CD Engine]
                                                                      │
      ┌───────────────────────┬───────────────────────┐                ▼
      ▼                       ▼                       ▼          [Deployment]
[Stage: Compile] ──► [Stage: Unit Test] ──► [Stage: Package] ──►  (Systemd/
(Check Syntax)       (Verify Logic)         (Create JAR)          Container)
```

* **Kết quả mong đợi:** Sinh viên hình dung được thứ tự thực thi nghiêm ngặt của một đường ống CI/CD: bất kỳ bước nào thất bại (ví dụ: Test sập), toàn bộ quy trình sẽ dừng lại ngay lập tức để bảo vệ hệ thống.

#### 5. Điểm cần nhấn mạnh

* Tự động hóa triển khai (CD) không có giá trị nếu thiếu bước kiểm soát chất lượng tự động (CI). Unit Test chính là lưới an toàn cốt lõi của CI.
* Mục tiêu tối thượng của CI/CD là phát hiện lỗi sớm nhất có thể (Fail-fast).

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Viết script tự động copy file JAR lên server khi có code mới là đã có CI/CD hoàn chỉnh.
* **Đính chính:** Đó chỉ là tự động hóa thao tác chuyển file (Deployment). Nếu bỏ qua bước Compile tập trung và Test tự động, quy trình đó chưa có thành phần "CI".

---

### LESSON 03: Mô hình môi trường Dev, Staging và Production

#### 1. Mục tiêu bài học

* **Phân loại** được mục đích, đặc tính dữ liệu và đối tượng sử dụng của các môi trường: Development, Staging, và Production.
* **Áp dụng** nguyên tắc quản lý cấu hình bằng biến môi trường (Environment Variables) để tách biệt thông tin nhạy cảm khỏi mã nguồn.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** STATE 0 — Local Application.

* **Vấn đề:** Hiện tại thông tin kết nối PostgreSQL (URL, Username, Password) của `user-service` đang được khai báo cố định (hardcode) trong file `application.yml`. Khi chuyển mã nguồn giữa môi trường kiểm thử của nội bộ và môi trường chạy thật của khách hàng, việc phải chỉnh sửa lại file này gây rủi ro lộ lọt thông tin bảo mật và sai lệch phiên bản.

#### 3. Nội dung trọng tâm

* **Mô hình 3 môi trường tiêu chuẩn:**
* *Development (Dev):* Môi trường nội bộ của lập trình viên, dữ liệu giả lập, thay đổi liên tục.
* *Staging (UAT):* Môi trường mô phỏng Production để kiểm thử tích hợp cuối cùng.
* *Production (Prod):* Môi trường chịu tải thực tế, phục vụ người dùng cuối của QuickBite, yêu cầu bảo mật và toàn vẹn dữ liệu tối đa.

* **Nguyên tắc Twelve-Factor App (Phần Configuration):** Mã nguồn cấu hình một lần, chạy ở mọi nơi nhờ nạp tham số thay đổi qua biến môi trường (Environment Variables) khi ứng dụng khởi chạy.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Cấu hình ứng dụng Spring Boot đọc thông số kết nối Database động từ hệ điều hành và minh họa cách ghi đè cấu hình runtime.

* **Cấu hình hệ thống (`user-service/src/main/resources/application.yml`):**
```yaml
spring:
  datasource:
    url: ${QUICKBITE_DB_URL:jdbc:postgresql://localhost:5432/quickbite_user}
    username: ${QUICKBITE_DB_USER:postgres}
    password: ${QUICKBITE_DB_PASS:password123}
```

* **Command:**
```bash
# Chạy mô phỏng trên môi trường Staging bằng cách nạp biến môi trường
export QUICKBITE_DB_URL=jdbc:postgresql://10.0.1.20:5432/quickbite_user_staging
export QUICKBITE_DB_USER=staging_admin
export QUICKBITE_DB_PASS=StagingSecurePass@2026

java -jar user-service.jar
```

* **Kết quả mong đợi:** Ứng dụng Spring Boot khởi chạy thành công và kết nối chính xác vào database của Staging mà không cần thay đổi bất kỳ dòng mã nguồn nào trong file JAR.

#### 5. Điểm cần nhấn mạnh

* Một Artifact duy nhất (file JAR) phải chạy được trên tất cả các môi trường mà không cần rebuild lại.

* Thông tin xác thực (Credentials) của môi trường Production tuyệt đối không được cam kết (commit) vào Git repository dưới bất kỳ hình thức nào.

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Tạo nhiều file cấu hình cứng như `application-dev.yml`, `application-staging.yml`, `application-prod.yml` chứa sẵn mật khẩu rồi chuyển đổi bằng flag `--spring.profiles.active`.
* **Lỗi triển khai:** Cách làm này vi phạm nghiêm trọng an toàn thông tin vì thông tin tài khoản Prod bị lộ trên Git và bất kỳ ai có quyền tiếp cận mã nguồn đều thấy được mật khẩu hệ thống thật.

---

### LESSON 04: Kiến trúc triển khai hệ thống Microservices

#### 1. Mục tiêu bài học

* **Phác thảo** được sơ đồ kiến trúc triển khai tổng thể của hệ thống QuickBite ở trạng thái mục tiêu.

* **Giải thích** được vai trò của API Gateway trong việc làm điểm phân phối duy nhất cho luồng dữ liệu microservices.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** STATE 0 — Local Application.

* **Vấn đề:** Khi ứng dụng phía máy khách (Frontend Web/Mobile) cần thực hiện các chức năng như lấy thông tin nhà hàng rồi tiến hành đặt món, nó phải gửi request đến hai IP hoặc Port khác nhau của `restaurant-service` và `order-service`. Việc này gây khó khăn cho việc quản lý bảo mật, phân tải và cấu hình tên miền ở phía client.

#### 3. Nội dung trọng tâm

* **Thành phần kiến trúc triển khai QuickBite:**
* *Edge Layer:* API Gateway — Điểm tiếp nhận request duy nhất từ bên ngoài.
* *Internal Service Layer:* Các dịch vụ nghiệp vụ biệt lập nằm sau vùng mạng an toàn (`user-service`, `restaurant-service`, `order-service`, `notification-service`).
* *Persistence Layer:* Hệ quản trị cơ sở dữ liệu PostgreSQL chịu trách nhiệm lưu trữ cho từng service.

* **Luồng đi của dữ liệu nghiệp vụ chuẩn:** Client -> API Gateway -> Định tuyến dựa trên Path `/api/v1/orders` đến `order-service` -> Giao tiếp nội bộ đến `restaurant-service` để kiểm tra trạng thái -> `notification-service` bắn thông báo.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Minh họa trực quan sơ đồ khối tương tác vật lý của hệ thống QuickBite để chuẩn bị tư duy cho việc đóng gói và chia mạng nội bộ ở các session kế tiếp.

* **Sơ đồ kiến trúc hệ thống (Logical & Network Deployment Architecture):**
```text
[ Client Application (Web / Mobile) ]
                  │
                  ▼ (Public Network: Port 80 / 443)
          ┌───────────────┐
          │  API Gateway  │
          └───────┬───────┘
                  │ (Private Internal Network)
     ┌────────────┼────────────┬────────────┐
     ▼            ▼            ▼            ▼
[user-svc]   [order-svc]  [rest-svc]   [notif-svc]
     │            │            │
     └────────────┼────────────┘
                  ▼
           [PostgreSQL DB]
```

*   **Kết quả mong đợi:** Sinh viên hiểu rõ nguyên lý: Chỉ có API Gateway mở cửa ra ngoài Internet, các service còn lại được ẩn hoàn toàn phía sau mạng nội bộ.

#### 5. Điểm cần nhấn mạnh
*   Kiến trúc triển khai Microservices yêu cầu tư duy phân rã hạ tầng: Mỗi cấu phần phải có khả năng khởi chạy, dừng lại và nâng cấp độc lập mà không kéo sập các thành phần khác.
*   Giao tiếp giữa các service nội bộ (Internal RPC/HTTP) cần được tối ưu tốc độ và bảo mật, tránh đi vòng qua internet công cộng.

#### 6. Hiểu lầm thường gặp
*   **Hiểu sai:** Nghĩ rằng mỗi service trong hệ thống microservices bắt buộc phải sở hữu một máy chủ cơ sở dữ liệu vật lý riêng biệt hoàn toàn.
*   **Đính chính:** Về mặt kiến trúc logic, dữ liệu phải độc lập (Database-per-service), không truy cập chéo database của nhau. Nhưng ở mặt triển khai hạ tầng, các schema/database độc lập này hoàn toàn có thể chạy chung trên một cụm máy chủ PostgreSQL duy nhất để tối ưu chi phí vận hành ở quy mô vừa và nhỏ.

---

# SESSION 02
## GIỚI THIỆU DOCKER

---

### LESSON 01: Khái niệm container và so sánh Docker với máy ảo

#### 1. Mục tiêu bài học
*   **Phân tích** được sự khác biệt về mặt kiến trúc phần cứng và hiệu năng giữa Container (Docker) và Máy ảo (Virtual Machine - VM).
*   **Giải thích** được cơ chế chia sẻ nhân (Shared Kernel) của Docker giúp tối ưu hóa tài nguyên phần cứng cho hệ thống nhiều dịch vụ như QuickBite.

#### 2. Bối cảnh hệ thống
*   **Trạng thái:** Chuyển dịch từ STATE 0 (Local Application) sang STATE 1 (Containerization).
*   **Vấn đề:** Khi cần triển khai đồng thời 4 dịch vụ Spring Boot của QuickBite cùng hệ quản trị database lên máy chủ thử nghiệm, nếu dùng giải pháp Máy ảo truyền thống (VM) tạo ra 5 máy ảo riêng biệt, hệ thống sẽ bị lãng phí một lượng lớn tài nguyên phần cứng (CPU, RAM) chỉ để chạy 5 hệ điều hành khách (Guest OS) độc lập, dẫn đến việc quá tải RAM của server.

#### 3. Nội dung trọng tâm
*   **Kiến trúc Máy ảo (Virtual Machine):** Chạy trên lớp Hypervisor, mỗi VM mang theo một Guest OS hoàn chỉnh đầy đủ thư viện nhân -> Khởi động chậm (hàng phút), dung lượng lớn (hàng GB), tiêu tốn RAM cố định.
*   **Kiến trúc Container (Docker):** Chạy trực tiếp trên Docker Engine, chia sẻ chung nhân hệ điều hành của máy chủ (Host OS) -> Khởi động tức thì (hàng mili-giây), dung lượng cực nhẹ (hàng MB), tiêu thụ tài nguyên theo nhu cầu thực tế của ứng dụng.

#### 4. Demo và thực hành
*   **Mục tiêu demo:** Minh họa trực quan cấu trúc tầng kiến trúc giữa VM và Docker bằng sơ đồ so sánh trực diện.
*   **Sơ đồ so sánh kiến trúc phần cứng:**

```text
    MÔ HÌNH MÁY ẢO (VM)                 MÔ HÌNH DOCKER CONTAINER
    ┌───────────────────────────────┐   ┌───────────────────────────────┐
    │ App (User)  │ App (Order)     │   │ App (User)  │ App (Order)     │
    ├─────────────┼─────────────────┤   ├─────────────┼─────────────────┤
    │ Thư viện    │ Thư viện        │   │ Thư viện    │ Thư viện        │
    ├─────────────┼─────────────────┤   ├───────────────────────────────┤
    │ Guest OS    │ Guest OS        │   │         Docker Engine         │
    ├─────────────┴─────────────────┤   ├───────────────────────────────┤
    │          Hypervisor           │   │            Host OS            │
    ├───────────────────────────────┤   ├───────────────────────────────┤
    │       Infrastructure          │   │       Infrastructure          │
    └───────────────────────────────┘   └───────────────────────────────┘
```

* **Kết quả mong đợi:** Sinh viên hiểu rõ tại sao Docker lại là giải pháp tối ưu cho kiến trúc Microservices nơi số lượng dịch vụ nhỏ cần chạy đồng thời rất lớn.

#### 5. Điểm cần nhấn mạnh

* Container không phải là một Máy ảo thu nhỏ. Container thực chất là một tiến trình (Process) của Linux được cô lập bằng các tính năng của Nhân (Kernel) bao gồm `Namespaces` và `Cgroups`.
* Vì chia sẻ chung nhân Host OS, Docker Container chạy trên Linux có hiệu năng tiệm cận ứng dụng chạy trực tiếp (Native), vượt trội hoàn toàn so với VM.

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Container bảo mật tuyệt đối và cô lập mạnh mẽ hơn Máy ảo.
* **Đính chính:** Vì dùng chung nhân (Shared Kernel) với Host OS, nếu một container chiếm được quyền can thiệp vào nhân hệ điều hành, nó có thể ảnh hưởng đến máy chủ và các container khác. VM với lớp Guest OS riêng biệt vẫn có mức độ cô lập bảo mật tầng sâu cao hơn.

---

### LESSON 02: Docker image, container và registry

#### 1. Mục tiêu bài học

* **Định nghĩa** và làm rõ mối quan hệ mật thiết giữa ba khái niệm nền tảng: Docker Image, Docker Container và Docker Registry.
* **Mô tả** được quy trình vòng đời của một ứng dụng từ lúc đóng gói thành Image đến khi phân phối và chạy thực tế.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** STATE 1 — Containerization.

* **Vấn đề:** Đội phát triển hoàn thành tính năng của `restaurant-service` và xuất ra file JAR. Tuy nhiên khi bàn giao, môi trường chạy của mỗi máy chủ khác nhau (thiếu thư viện, sai múi giờ, phiên bản Java không đồng nhất), dẫn tới việc ứng dụng hoạt động không ổn định, không giống nhau giữa các máy.

#### 3. Nội dung trọng tâm

* **Docker Image:** Một khuôn mẫu đóng gói (Template) ở trạng thái chỉ đọc (Read-only), chứa toàn bộ mã nguồn, runtime (JDK), thư viện hệ thống và cấu hình cần thiết để ứng dụng hoạt động. Được ví như "Bản thiết kế" hoặc đĩa CD cài đặt game.
* **Docker Container:** Một thực thể sống (Instance) được khởi tạo từ Docker Image. Là môi trường thực thi cô lập có trạng thái đọc-ghi (Read-Write layer). Được ví như "Ngôi nhà" được xây từ bản thiết kế.
* **Docker Registry:** Kho lưu trữ tập trung dùng để quản lý và phân phối các Docker Image (Ví dụ: Docker Hub, GitLab Container Registry).

#### 4. Demo và thực hành

* **Mục tiêu demo:** Minh họa luồng tương tác ba thực thể bằng mô hình kéo-thả và khởi chạy một image cơ bản.

* **Sơ đồ luồng tương tác khái niệm:**
```text
[ Docker Registry (Docker Hub) ]
             │
             ▼ (docker pull)
     [ Docker Image ] (Read-only Template trên đĩa)
             │
             ▼ (docker run)
   ┌───────────────────┐
   │ Docker Container  │ (Biến Image thành Tiến trình chạy thực tế)
   └───────────────────┘
```
*   **Lệnh thực hiện minh họa vòng đời mẫu (Command):**
```bash
# Pull một image chính thức từ Registry về máy local
docker pull eclipse-temurin:17-jre-alpine

# Kiểm tra danh sách các Image đang có ở local máy
docker images
```

* **Kết quả mong đợi:** Sinh viên phân biệt rõ trạng thái tĩnh trên ổ đĩa (Image) và trạng thái động trong bộ nhớ RAM (Container).

#### 5. Điểm cần nhấn mạnh

* Mối quan hệ Image - Container tương đương với mối quan hệ Class - Object trong lập trình hướng đối tượng (OOP). Từ một Image có thể sinh ra hàng trăm Container chạy song song.
* Docker Image được cấu tạo từ nhiều lớp (Layers) xếp chồng lên nhau, giúp tối ưu dung lượng lưu trữ thông qua cơ chế tái sử dụng layer cũ.

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Sửa đổi dữ liệu hoặc file cấu hình bên trong một Container đang chạy thì Docker Image sinh ra nó cũng sẽ tự động thay đổi theo.
* **Đính chính:** Image là bất biến (Immutable). Mọi thay đổi khi Container đang chạy chỉ được ghi lên một lớp tạm thời (Container Layer / Writable Layer) trên đỉnh và sẽ mất đi hoàn toàn nếu Container đó bị xóa, không hề tác động vào Image gốc.

---

### LESSON 03: Cài đặt Docker và kiểm tra môi trường

#### 1. Mục tiêu bài học

* **Triển khai cài đặt** thành công Docker Engine và Docker Desktop trên môi trường hệ điều hành cá nhân đúng kỹ thuật.
* **Sử dụng** các lệnh kiểm tra hệ thống cơ bản để xác nhận Docker Daemon hoạt động bình thường.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** STATE 1 — Containerization.

* **Vấn đề:** Trước khi tiến hành đóng gói `notification-service` hay `order-service` thành container, máy tính của lập trình viên hoặc máy chủ phải được thiết lập môi trường Docker chạy ổn định, đồng bộ về phiên bản engine.

#### 3. Nội dung trọng tâm

* **Thành phần kiến trúc Docker Engine:** Docker Daemon (Dịch vụ chạy ngầm quản lý tài nguyên), Docker CLI (Bộ công cụ dòng lệnh tương tác với người dùng), và REST API.
* **Cài đặt thực tế:** Quy trình thiết lập Docker Desktop trên Windows/macOS (sử dụng WSL2 hoặc ảo hóa phần cứng) và cài đặt Docker Engine gốc trên môi trường Linux Ubuntu.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Thực hiện kiểm tra tính sẵn sàng của Docker Engine và chạy thử nghiệm container đầu tiên để xác nhận môi trường hoạt động thông suốt.

* **Command:**
```bash
# Kiểm tra phiên bản hiển thị của Docker Client và Server
docker version

# Kiểm tra thông tin chi tiết về tài nguyên hệ thống Docker đang quản lý
docker info

# Chạy thử nghiệm container kiểm tra môi trường tiêu chuẩn
docker run hello-world
```

* **Output mong đợi từ hệ thống:**
```text
Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

#### 5. Điểm cần nhấn mạnh

* Khi chạy lệnh `docker version`, cần đảm bảo cả hai thành phần `Client` và `Server (Engine)` đều phản hồi thông tin thành công. Nếu Server báo lỗi kết nối, nghĩa là Docker Daemon chưa được khởi động.
* Trên hệ điều hành Linux, cần cấu hình thêm bước phân quyền cho User hiện hành thuộc group `docker` để tránh việc phải gõ lệnh `sudo` trước mỗi câu lệnh Docker.

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Nghĩ rằng Docker Desktop trên Windows chạy độc lập mà không cần bất kỳ công cụ hỗ trợ ảo hóa nào.
* **Đính chính:** Bản chất Docker chạy trên nhân Linux. Trên môi trường Windows, Docker Desktop bắt buộc phải dựa vào kiến trúc ảo hóa của WSL2 (Windows Subsystem for Linux) hoặc Hyper-V để tạo ra một nhân Linux siêu nhẹ chạy ngầm bên dưới.

---

### LESSON 04: Các lệnh Docker cơ bản trong vòng đời container

#### 1. Mục tiêu bài học

* **Làm chủ** các câu lệnh quản lý vòng đời của một container: Tạo mới, khởi chạy, tạm dừng, dừng lại và xóa bỏ.
* **Áp dụng** các tham số cấu hình port (Port Mapping) và đặt tên để container dịch vụ có thể truy cập được từ máy host.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** STATE 1 — Containerization.

* **Vấn đề:** Lập trình viên muốn chạy thử một container PostgreSQL làm database tạm thời cho hệ thống QuickBite ở môi trường local. Yêu cầu đặt ra là phải kiểm soát được trạng thái chạy của database này, đặt tên tường minh và ánh xạ port chính xác để ứng dụng Spring Boot từ máy host kết nối vào được.

#### 3. Nội dung trọng tâm

* **Vòng đời Container:** Created -> Running -> Exited / Stopped -> Destroyed.
* **Các tham số cốt lõi khi chạy Container:**
* `-d` (Detached mode): Chạy ngầm container dưới nền, trả lại quyền điều khiển terminal cho người dùng.
* `-p` (Port mapping): Ánh xạ cổng theo quy tắc `Cổng_Máy_Host:Cổng_Trong_Container`.
* `--name`: Đặt tên định danh duy nhất cho container thay vì để Docker tự sinh tên ngẫu nhiên.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Khởi chạy, quản lý trạng thái và dọn dẹp một container PostgreSQL dùng làm database đích cho QuickBite.

* **Command:**
```bash
# 1. Khởi chạy một container Postgres, chạy ngầm, map port 5432, đặt tên rõ ràng
docker run -d --name quickbite-db -p 5432:5432 -e POSTGRES_PASSWORD=secret postgres:15-alpine

# 2. Liệt kê các container đang hoạt động để kiểm tra trạng thái
docker ps

# 3. Dừng container database lại khi không dùng nữa
docker stop quickbite-db

# 4. Liệt kê tất cả các container (bao gồm cả container đã dừng)
docker ps -a

# 5. Xóa bỏ hoàn toàn container khỏi hệ thống
docker rm quickbite-db
```

* **Kết quả mong đợi:** Container khởi chạy ngầm thành công, cổng 5432 trên máy host mở ra tiếp nhận kết nối, và được dọn dẹp sạch sẽ sau khi kết thúc phiên làm việc.

#### 5. Điểm cần nhấn mạnh
*   Lệnh `docker stop` chỉ đưa container về trạng thái dừng (Exited) chứ không xóa dữ liệu hay cấu hình của nó. Muốn giải phóng hoàn toàn tài nguyên ổ đĩa, phải dùng lệnh `docker rm`.
*   Nếu không dùng tham số `-p`, container vẫn chạy nhưng hoàn toàn biệt lập, ứng dụng bên ngoài máy host sẽ không cách nào kết nối được vào database bên trong container.

#### 6. Hiểu lầm thường gặp
*   **Hiểu sai:** Chạy lệnh `docker rm` lên một container đang hoạt động (Status: Up) để xóa nhanh nó.
*   **Lỗi triển khai:** Docker sẽ chặn hành động này để bảo vệ dữ liệu tiến trình runtime. Muốn xóa một container đang chạy, bắt buộc phải `docker stop` trước rồi mới `docker rm`, hoặc dùng cờ cưỡng chế nguy hiểm `-f` (`docker rm -f`).

---

### LESSON 05: Kiểm tra log và truy cập container (logs, exec)

#### 1. Mục tiêu bài học
*   **Trích xuất và theo dõi** toàn bộ thông tin log xuất ra từ bên trong container để phục vụ công tác debug, chẩn đoán lỗi hệ thống.
*   **Truy cập vào bên trong** không gian cô lập của một container đang chạy để kiểm tra file hoặc cấu hình hệ điều hành thu nhỏ.

#### 2. Bối cảnh hệ thống
*   **Trạng thái:** STATE 1 — Containerization.
*   **Vấn đề:** Container `quickbite-db` (PostgreSQL) khởi chạy nhưng ứng dụng Spring Boot báo lỗi kết nối không thành công. Người vận hành hệ thống cần phải xem nhật ký hoạt động nội bộ của database hoặc chui trực tiếp vào trong container để kiểm tra xem database `quickbite_user` đã thực sự được khởi tạo hay chưa.

#### 3. Nội dung trọng tâm
*   **Cơ chế Log của Docker:** Gom toàn bộ luồng dữ liệu đầu ra tiêu chuẩn (`STDOUT` / `STDERR`) của ứng dụng bên trong container và chuyển hướng ra ngoài qua lệnh `docker logs`.
*   **Lệnh tương tác trực tiếp (`docker exec`):** Cho phép chạy một câu lệnh mới (ví dụ như khởi chạy một shell terminal `sh` hoặc `bash`) ngay bên trong không gian bị cô lập của một container đang hoạt động.

#### 4. Demo và thực hành
*   **Mục tiêu demo:** Xem log khởi động của Postgres và truy cập vào terminal bên trong container để tương tác với client dòng lệnh `psql`.
*   **Command:**

```bash
# Giả định container quickbite-db đang chạy ở bài trước

# 1. Xem toàn bộ log đã sinh ra từ lúc container khởi động đến hiện tại
docker logs quickbite-db

# 2. Theo dõi log thời gian thực (Real-time stream) giống lệnh tail -f
docker logs -f quickbite-db

# 3. Truy cập vào bên trong container bằng shell, giữ chế độ tương tác (Interactive tty)
docker exec -it quickbite-db sh

# ---- Các lệnh chạy khi đã ở BÊN TRONG container ----
# Thử gọi công cụ psql kiểm tra database nội bộ
psql -U postgres
\l
exit
exit
# ----------------------------------------------------
```

* **Kết quả mong đợi:** Xem được chi tiết log hệ thống; truy cập mượt mà vào shell của container và thoát ra an toàn mà không làm dừng container gốc.

#### 5. Điểm cần nhấn mạnh

* Tham số `-it` trong lệnh `docker exec` viết tắt của `--interactive` (giữ luồng STDIN mở) và `--tty` (cấp một terminal ảo). Nếu thiếu cặp cờ này, bạn không thể gõ tương tác với shell bên trong.
* Khi sử dụng `docker exec`, bạn đang thao tác với tư cách một tiến trình bổ sung. Việc bạn thoát (`exit`) khỏi shell này chỉ kết thúc tiến trình shell đó, container chính (PostgreSQL) vẫn tiếp tục hoạt động bình thường.

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Dùng lệnh `docker run -it postgres sh` để vào kiểm tra một container PostgreSQL đang chạy lỗi.
* **Lỗi triển khai:** Lệnh `docker run` luôn luôn tạo mới một container hoàn toàn khác từ image gốc. Để can thiệp vào một container **đã tồn tại và đang chạy**, lệnh bắt buộc phải sử dụng là `docker exec`.

---

# SESSION 04

## DOCKER NETWORKING & MULTI-CONTAINER ARCHITECTURE

---

### LESSON 01: Khái niệm mạng trong Docker (Bridge, Host, None)

#### 1. Mục tiêu bài học

* **Phân biệt** được cơ chế hoạt động, phạm vi áp dụng và mức độ cô lập của các driver mạng cơ bản trong Docker: `bridge`, `host`, và `none`.
* **Giải thích** được tại sao driver `bridge` tùy biến (Custom Bridge Network) là lựa chọn tối ưu cho kiến trúc phân rã của QuickBite.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** STATE 1 — Containerization (Các container độc lập).
* **Vấn đề:** Mặc định khi khởi chạy, các container dịch vụ của QuickBite (`user-service`, `order-service`) sẽ tự động rơi vào mạng `default bridge`. Trong mạng mặc định này, các container chỉ có thể giao tiếp với nhau bằng địa chỉ IP nội bộ (IP này thay đổi liên tục mỗi khi restart container), không thể phân giải tên gọi (Container Name) của nhau, gây mất ổn định kết nối.

#### 3. Nội dung trọng tâm

* **Mạng Bridge (Mặc định và Tùy biến):** Tạo ra một switch ảo phần mềm trên máy host. Các container kết nối vào switch này sẽ nhận được một dải IP nội bộ riêng.
* *Điểm cốt lõi:* Mạng **User-defined Bridge (Bridge tùy biến)** cung cấp cơ chế tự động phân giải tên miền nội bộ (Automatic Service Discovery) — các container gọi nhau trực tiếp bằng tên container thay vì IP.


* **Mạng Host:** Container chia sẻ hoàn toàn không gian mạng với máy host (không có lớp NAT/cầu nối). Port của container gắn thẳng vào Port của máy host. Tốc độ nhanh nhất nhưng mất đi tính cô lập port.
* **Mạng None:** Khóa toàn bộ stack mạng của container. Container hoàn toàn biệt lập, không thể kết nối ra ngoài và bên ngoài không thể gọi vào.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Tạo một mạng Bridge tùy biến mang tên `quickbite-net`, cho hai container kết nối vào và thực hiện lệnh ping qua lại bằng tên định danh.
* **Command:**
```bash
# 1. Tạo một mạng bridge tùy biến mới
docker network create --driver bridge quickbite-net

# 2. Liệt kê danh sách mạng để xác nhận
docker network ls

# 3. Khởi chạy container alpha tham gia vào mạng vừa tạo
docker run -d --name service-alpha --network quickbite-net alpine sleep 3600

# 4. Khởi chạy container beta tham gia vào mạng và thử ping sang alpha bằng TÊN
docker run --rm --network quickbite-net alpine ping -c 3 service-alpha

```


* **Output mong đợi:** Lệnh ping từ container beta sang tên `service-alpha` trả về kết quả phản hồi thành công cùng dải IP nội bộ được phân giải tự động.

#### 5. Điểm cần nhấn mạnh

* Mạng `default bridge` của Docker không hỗ trợ tính năng tự động phân giải tên container (Service Discovery). Chỉ có mạng bridge do người dùng tự định nghĩa (`User-defined bridge`) mới có tính năng này.
* Đây chính là chìa khóa để 4 dịch vụ Spring Boot của QuickBite tìm thấy nhau một cách ổn định trong môi trường container.

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Nghĩ rằng muốn hai container gọi được cho nhau thì bắt buộc phải mở cổng (expose port) ra ngoài máy host bằng tham số `-p`.
* **Đính chính:** Tham số `-p` chỉ dùng khi muốn mở cổng cho thế giới bên ngoài hoặc máy host truy cập vào container. Nếu hai container nằm chung một mạng Docker, chúng có thể giao tiếp với nhau qua tất cả các cổng nội bộ một cách tự do mà không cần nới lỏng bảo mật ra bên ngoài.

---

### LESSON 02: Liên kết các container (Service Discovery)

#### 1. Mục tiêu bài học

* **Vận dụng** cơ chế Service Discovery của Docker Network để kết nối ứng dụng Spring Boot tới cơ sở dữ liệu PostgreSQL chạy trong container.
* **Kiểm chứng** tính ổn định của kết nối khi IP của container thay đổi nhưng tên container giữ nguyên.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** STATE 1 — Containerization.
* **Vấn đề:** `order-service` cần kết nối tới `quickbite-db` (Postgres). Nếu chúng ta cấu hình URL kết nối bằng IP nội bộ của container Postgres (ví dụ: `11.0.0.5`), hệ thống sẽ lỗi ngay lập tức khi container Postgres bị restart và Docker cấp cho nó một IP mới (ví dụ: `11.0.0.6`).

#### 3. Nội dung trọng tâm

* **Nguyên lý Service Discovery cục bộ:** Docker Engine tích hợp một DNS server nội bộ. Khi container A gọi container B bằng tên (`http://quickbite-db:5432`), DNS nội bộ sẽ tra cứu bảng ánh xạ tên $\rightarrow$ IP hiện hành để định tuyến dòng dữ liệu chính xác.
* **Thực thi liên kết hệ thống:** Gom database và backend vào chung hạ tầng mạng bảo mật, cấu hình chuỗi kết nối (Connection String) bằng tên định danh.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Khởi chạy một container Postgres, sau đó chạy một container công cụ để kiểm tra khả năng phân giải DNS nội bộ của Docker Network.
* **Command:**
```bash
# Đảm bảo mạng quickbite-net đã được tạo từ bài trước

# 1. Khởi chạy container database nằm trong mạng quickbite-net
docker run -d --name quickbite-db --network quickbite-net -e POSTGRES_PASSWORD=secret postgres:15-alpine

# 2. Sử dụng công cụ nslookup bên trong container kiểm tra DNS nội bộ
docker run --rm --network quickbite-net busybox nslookup quickbite-db

```


* **Output mong đợi:** Lệnh `nslookup` trả về chính xác IP nội bộ của container `quickbite-db`.

#### 5. Điểm cần nhấn mạnh

* Tên container (`--name quickbite-db`) đóng vai trò như một Domain Name (Tên miền) nội bộ. Do đó, việc đặt tên container chuẩn hóa, tường minh là cực kỳ quan trọng trong DevOps.

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Cho rằng tính năng Service Discovery này hoạt động xuyên suốt giữa các máy chủ vật lý khác nhau một cách mặc định.
* **Đính chính:** Cơ chế DNS nội bộ của driver `bridge` chỉ có hiệu lực trên phạm vi **một máy chủ vật lý độc lập (Single Host)**. Khi hệ thống mở rộng ra nhiều máy chủ, chúng ta cần các giải pháp mạng nâng cao hơn như mạng Overlay (Docker Swarm) hoặc Kubernetes.

---

# SESSION 05

## TỔNG QUAN VỀ MULTI-SERVICES & THIẾT KẾ ĐỐI TƯỢNG HỆ THỐNG QUICKBITE

---

### LESSON 01: Chi tiết Kiến trúc Đóng gói & Thiết kế Thực thể (Entity) các Dịch vụ

#### 1. Mục tiêu bài học

* **Nắm trọn vẹn cấu trúc lớp dữ liệu** và mô hình thiết kế đối tượng (Object Design) của 4 core services thuộc hệ thống QuickBite.
* **Phân tích mối quan hệ** logic giữa các thực thể và cơ chế ánh xạ biến môi trường (Environment Variables) đặc trưng cho từng service để chuẩn bị cấu hình chạy tự động.

#### 2. Tổng quan Thiết kế Đối tượng (Object & Entity Design) của Hệ thống

Giảng viên cần nắm chắc bản đồ thiết kế thực thể của 4 services dưới đây để hướng dẫn sinh viên cách hệ thống lưu trữ dữ liệu và phối hợp nghiệp vụ. Toàn bộ các service đều sử dụng **Spring Data JPA** để ánh xạ xuống cơ sở dữ liệu PostgreSQL.

##### A. User Service (`user-service`)

* **Vai trò:** Quản lý tài khoản, thông tin định danh và phân quyền người dùng (Khách hàng, Tài xế, Chủ cửa hàng).
* **Sơ đồ lớp thực thể (Entity Class):**
```java
@Entity @Table(name = "users")
public class User {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(unique = true, nullable = false)
    private String username;
    private String password; // Hash BCrypt
    private String fullName;
    private String email;
    private String phone;
    @Enumerated(EnumType.STRING)
    private Role role; // CUSTOMER, DRIVER, MERCHANT
}
```

##### B. Restaurant Service (`restaurant-service`)

* **Vai trò:** Quản lý thông tin nhà hàng, trạng thái đóng/mở cửa và thực đơn món ăn (Menu Items).
* **Sơ đồ lớp thực thể (Entity Class):**
```java
@Entity @Table(name = "restaurants")
public class Restaurant {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String address;
    private boolean isActive; // Trạng thái hoạt động

    @OneToMany(mappedBy = "restaurant", cascade = CascadeType.ALL)
    private List<MenuItem> menuItems;
}

@Entity @Table(name = "menu_items")
public class MenuItem {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private BigDecimal price;
    private boolean isAvailable;

    @ManyToOne @JoinColumn(name = "restaurant_id")
    private Restaurant restaurant;
}
```

##### C. Order Service (`order-service`)

* **Vai trò:** Trọng tâm xử lý luồng nghiệp vụ. Tiếp nhận yêu cầu đặt món, tính toán tổng tiền, lưu trạng thái đơn hàng và phối hợp điều hướng luồng thông tin.
* **Sơ đồ lớp thực thể (Entity Class):**
```java
@Entity @Table(name = "orders")
public class Order {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private Long customerId;   // Tham chiếu từ user-service (Loosely Coupled)
    private Long restaurantId; // Tham chiếu từ restaurant-service

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> items;

    private BigDecimal totalPrice;
    @Enumerated(EnumType.STRING)
    private OrderStatus status; // PENDING, ACCEPTED, SHIPPING, DELIVERED, CANCELLED
    private LocalDateTime createdAt;
}

@Entity @Table(name = "order_items")
public class OrderItem {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private Long menuItemId; // Tham chiếu món ăn từ restaurant-service
    private Integer quantity;
    private BigDecimal price;

    @ManyToOne @JoinColumn(name = "order_id")
    private Order order;
}
```

##### D. Notification Service (`notification-service`)

* **Vai trò:** Tiếp nhận sự kiện thay đổi trạng thái đơn hàng để gửi thông báo (Mô phỏng gửi Email/SMS/Push Notification) tới người dùng cuối.
* **Sơ đồ lớp thực thể (Entity Class):**
```java
@Entity @Table(name = "notifications")
public class Notification {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private Long userId;       // Người nhận thông báo
    private String title;
    private String message;
    private boolean isRead;
    private LocalDateTime sentAt;
}
```

---

### LESSON 02: Cấu hình Khởi chạy Môi trường (Configuration Specs) cho Giảng viên

Để chuẩn bị cho việc viết kịch bản Docker Compose ở các bước tiếp theo, toàn bộ cấu hình hệ thống của QuickBite đã được chuẩn hóa thông qua các biến môi trường. Dưới đây là bảng đặc tả cấu hình chi tiết của từng service để giảng viên dễ dàng nắm bắt tổng quan hoặc thiết lập môi trường chạy demo.

#### 1. Cấu hình Cơ sở dữ liệu tập trung (PostgreSQL Database State)

Để tiết kiệm tài nguyên hạ tầng, hệ thống sử dụng một container PostgreSQL duy nhất làm nền tảng lưu trữ bền vững (Persistence Layer), chia làm 4 database biệt lập để đảm bảo nguyên tắc kiến trúc microservices.

* **Tên Container:** `quickbite-db`
* **Mạng tham gia:** `quickbite-net`
* **Cổng nội bộ:** `5432`
* **Thông tin truy cập nội bộ:**
* Database 1: `quickbite_user` (Dành cho `user-service`)
* Database 2: `quickbite_restaurant` (Dành cho `restaurant-service`)
* Database 3: `quickbite_order` (Dành cho `order-service`)
* Database 4: `quickbite_notification` (Dành cho `notification-service`)

---

#### 2. Chi tiết Biến môi trường và File cấu hình của từng Service

##### A. Thông số triển khai `user-service`

* **Cổng chạy mặc định:** `8081`
* **File cấu hình mẫu (`application.yml`):**
```yaml
server:
  port: 8081
spring:
  datasource:
    url: jdbc:postgresql://${DB_HOST:localhost}:${DB_PORT:5432}/${DB_NAME:quickbite_user}
    username: ${DB_USER:postgres}
    password: ${DB_PASS:secret}
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true

```

* **Biến môi trường cần nạp khi chạy container:**
* `DB_HOST=quickbite-db` (Gọi trực tiếp tên container database qua Docker Network)
* `DB_PORT=5432`
* `DB_NAME=quickbite_user`
* `DB_USER=postgres`
* `DB_PASS=secret`

##### B. Thông số triển khai `restaurant-service`

* **Cổng chạy mặc định:** `8082`
* **File cấu hình mẫu (`application.yml`):**
```yaml
server:
  port: 8082
spring:
  datasource:
    url: jdbc:postgresql://${DB_HOST:localhost}:${DB_PORT:5432}/${DB_NAME:quickbite_restaurant}
    username: ${DB_USER:postgres}
    password: ${DB_PASS:secret}
  jpa:
    hibernate:
      ddl-auto: update

```

* **Biến môi trường cần nạp khi chạy container:**
* `DB_HOST=quickbite-db`
* `DB_PORT=5432`
* `DB_NAME=quickbite_restaurant`
* `DB_USER=postgres`
* `DB_PASS=secret`

##### C. Thông số triển khai `order-service`

* **Cổng chạy mặc định:** `8083`
* **Mối liên kết nghiệp vụ nội bộ:** `order-service` cần gọi API sang `restaurant-service` để kiểm tra thực đơn hợp lệ trước khi tạo đơn hàng. Cấu hình này sử dụng Spring RestTemplate hoặc OpenFeign.
* **File cấu hình mẫu (`application.yml`):**
```yaml
server:
  port: 8083
spring:
  datasource:
    url: jdbc:postgresql://${DB_HOST:localhost}:${DB_PORT:5432}/${DB_NAME:quickbite_order}
    username: ${DB_USER:postgres}
    password: ${DB_PASS:secret}
  jpa:
    hibernate:
      ddl-auto: update

# Cấu hình endpoint gọi dịch vụ khác bằng DNS Docker Network
services:
  restaurant-service:
    url: http://${RESTAURANT_SVC_HOST:localhost}:${RESTAURANT_SVC_PORT:8082}
```

* **Biến môi trường cần nạp khi chạy container:**
* `DB_HOST=quickbite-db`
* `DB_PORT=5432`
* `DB_NAME=quickbite_order`
* `DB_USER=postgres`
* `DB_PASS=secret`
* `RESTAURANT_SVC_HOST=restaurant-service` (Tên container của Restaurant Service)
* `RESTAURANT_SVC_PORT=8082`

##### D. Thông số triển khai `notification-service`

* **Cổng chạy mặc định:** `8084`
* **File cấu hình mẫu (`application.yml`):**
```yaml
server:
  port: 8084
spring:
  datasource:
    url: jdbc:postgresql://${DB_HOST:localhost}:${DB_PORT:5432}/${DB_NAME:quickbite_notification}
    username: ${DB_USER:postgres}
    password: ${DB_PASS:secret}
  jpa:
    hibernate:
      ddl-auto: update
```

* **Biến môi trường cần nạp khi chạy container:**
* `DB_HOST=quickbite-db`
* `DB_PORT=5432`
* `DB_NAME=quickbite_notification`
* `DB_USER=postgres`
* `DB_PASS=secret`
---

### LESSON 03: Phân tích luồng tương tác thực tế (Runtime Integration Scenario)

Để minh họa cách các thực thể và cấu hình trên phối hợp nhịp nhàng khi toàn bộ hệ thống lên container, giảng viên có thể sử dụng kịch bản nghiệp vụ sau để giải thích luồng dữ liệu end-to-end cho sinh viên:

1. **Bước 1 (Tiếp nhận):** Khách hàng gửi lệnh đặt món ăn. Request đi qua Gateway (sẽ cấu hình ở session sau) chuyển tới `order-service` (Port 8083).
2. **Bước 2 (Kiểm tra dữ liệu chéo qua DNS):** `order-service` sử dụng chuỗi URL cấu hình `http://restaurant-service:8082/api/v1/restaurants/...` để kiểm tra xem món ăn đó có còn hàng (trường `isAvailable` của `MenuItem` entity bằng `true`) hay không.
3. **Bước 3 (Ghi nhận trạng thái):** Nếu hợp lệ, `order-service` tạo bản ghi mới vào bảng `orders` của database `quickbite_order` với trạng thái `status = PENDING`.
4. **Bước 4 (Kích hoạt thông báo):** Ngay sau khi đơn hàng được lưu, một tín hiệu truyền tải thông tin (HTTP post hoặc Message Event) được gửi tới `http://notification-service:8084`. Dịch vụ này lập tức tạo một bản ghi `Notification` lưu vào database `quickbite_notification` để chuẩn bị đẩy thông báo xác nhận đơn hàng thành công về máy khách.

#### Điểm cần nhấn mạnh cho giảng viên (Pedagogical Tips):

* **Tính độc lập dữ liệu:** Nhấn mạnh rằng dù chạy chung một container PostgreSQL vật lý để tiết kiệm RAM khi làm demo tại lớp, các service kết nối vào 4 database hoàn toàn khác biệt bằng tài khoản được phân quyền riêng. `order-service` tuyệt đối không được phép thực hiện câu lệnh SQL Join sang bảng `users` hay `restaurants` mà bắt buộc phải đi qua giao tiếp mạng (REST API) sử dụng cơ chế Service Discovery của Docker Network.
* **Tách biệt cấu hình:** Nhờ cách thiết kế đặt giá trị mặc định dạng `${BIẾN:Giá_trị_mặc_định}`, mã nguồn ứng dụng có thể chạy mượt mà ở máy local của sinh viên khi làm bài tập (`localhost`) và tự động chuyển hướng mượt mà sang gọi tên container (`quickbite-db`) khi đóng gói chạy Docker mà không cần sửa đổi bất kỳ dòng code Java nào.

---

*Giảng viên lưu ý: Việc nắm chắc chi tiết cấu trúc Entity, Port, và cách liên kết DNS của từng service tại Session 05 là bước đệm bắt buộc để sinh viên bước sang **Session 06**, nơi các em sẽ tự tay viết file `docker-compose.yml` để khởi chạy toàn bộ 4 dịch vụ này chỉ bằng một câu lệnh duy nhất.*