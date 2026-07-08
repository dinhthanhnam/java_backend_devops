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
    │ App (User)  │ App (Restaurant)│   │ App (User)  │ App (Restaurant)│
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

* **Vấn đề:** Trước khi tiến hành đóng gói `user-service` hay `restaurant-service` thành container, máy tính của lập trình viên hoặc máy chủ phải được thiết lập môi trường Docker chạy ổn định, đồng bộ về phiên bản engine.

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

## DOCKER COMPOSE VÀ DOCKERFILE

---

### LESSON 01: Dockerfile và cách đóng gói ứng dụng Spring Boot thực tế

#### 1. Mục tiêu bài học

* **Giải thích** được ý nghĩa của Dockerfile như là "nguồn sự thật duy nhất" (Source of Truth) cho đặc tả môi trường chạy.
* **Biên soạn** đúng cấu trúc Dockerfile để đóng gói ứng dụng Java Spring Boot từ file JAR.
* **Phân biệt** bản chất hoạt động của chỉ thị `ENTRYPOINT` và `CMD` (tiến trình Foreground PID 1 vs Background qua shell, cơ chế nhận tín hiệu tắt máy an toàn `SIGTERM`).

#### 2. Bối cảnh hệ thống

* **Trạng thái:** STATE 1 — Containerization (Đóng gói ứng dụng đơn lẻ).
* **Vấn đề:** Để chạy `user-service` hoặc `restaurant-service` thủ công bằng lệnh `docker run`, ta phải truyền quá nhiều tham số phức tạp (port, volume, JRE, working dir, run jar command). Cấu hình này rất dễ sai sót và bị phân tán. Dockerfile ra đời làm bản thiết kế chuẩn hóa và nhất quán để tự động build thành image.

#### 3. Nội dung trọng tâm

* **Các chỉ thị cơ bản trong Dockerfile:** `FROM` (chọn base image JRE Alpine gọn nhẹ), `WORKDIR` (thư mục làm việc), `COPY` (sao chép file JAR từ máy host), `EXPOSE` (khai báo cổng), và `ENTRYPOINT`/`CMD` (lệnh khởi chạy).
* **Cơ chế tắt máy an toàn (Graceful Shutdown):** Giải thích lý do sử dụng `ENTRYPOINT ["java", "-jar", "app.jar"]` ở dạng mảng (Exec Form) để Java chạy trực tiếp dưới PID 1, giúp nhận tín hiệu `SIGTERM` và đóng kết nối an toàn, thay vì dùng `CMD` dạng shell khiến tiến trình shell chiếm PID 1 và Java bị tắt cưỡng bức bằng `SIGKILL`.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Tạo Dockerfile cho `user-service`, thực hiện build image và chạy container.
* **Command:**
```bash
# 1. Đứng tại thư mục chứa source code user-service có file JAR đã build
# Viết Dockerfile:
# FROM eclipse-temurin:17-jre-alpine
# WORKDIR /app
# COPY build/libs/user-service.jar app.jar
# EXPOSE 8081
# ENTRYPOINT ["java", "-jar", "app.jar"]

# 2. Thực hiện đóng gói image
docker build -t quickbite-user-service:v1 .

# 3. Khởi chạy container từ image tự đóng gói
docker run -d -p 8081:8081 --name user-app quickbite-user-service:v1
```
* **Output mong đợi:** Build image thành công dung lượng ~100MB. Container khởi chạy và log Spring Boot hiển thị mượt mà.

#### 5. Điểm cần nhấn mạnh

* Bắt buộc sử dụng JRE Alpine thay vì JDK đầy đủ trên Production để tối ưu dung lượng và bảo mật (giảm bề mặt tấn công).
* Khai báo Exec Form (`[...]`) trong `ENTRYPOINT` để ứng dụng nhận trực tiếp tín hiệu hệ thống.

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Nghĩ rằng viết chỉ thị `EXPOSE 8081` trong Dockerfile là Docker sẽ tự động mở cổng ra máy host vật lý.
* **Đính chính:** `EXPOSE` chỉ mang tính chất tài liệu hóa. Bạn bắt buộc phải dùng tham số `-p 8081:8081` khi chạy container để ánh xạ cổng ra ngoài.

---

### LESSON 02: Docker Compose và khái niệm hệ thống nhiều container

#### 1. Mục tiêu bài học

* **Giải thích** được định nghĩa, vai trò và sự cần thiết của hệ thống nhiều container (Multi-container System) trong kiến trúc Microservices.
* **Hiểu bản chất** tại sao Docker Compose giải quyết các hạn chế của việc quản lý thủ công nhiều container.
* **Phân tích** quy trình 3 bước hoạt động nền tảng: Đóng gói (Dockerfile) -> Khai báo (docker-compose.yml) -> Vận hành (docker compose CLI).

#### 2. Bối cảnh hệ thống

* **Trạng thái:** STATE 2 — Multi-container Local System.
* **Vấn đề:** Khi ứng dụng QuickBite phình to gồm `user-service`, `restaurant-service` và `quickbite-db`, việc gõ lệnh `docker run` thủ công cho từng container, tra cứu địa chỉ IP động qua `docker inspect` và cứng cấu hình IP kết nối sẽ dẫn tới sự đổ vỡ khi IP thay đổi (IP Drift). Hệ thống nhiều container đòi hỏi cơ chế quản trị cấu hình tập trung.

#### 3. Nội dung trọng tâm

* **Khái niệm hệ thống nhiều container:** Giải thích sự cần thiết của việc tách biệt container (đa dạng công nghệ, scale độc lập, cô lập lỗi).
* **Docker Compose là gì:** Công cụ định nghĩa và chạy hệ thống đa container theo phong cách mô tả (Declarative YAML) thay vì mệnh lệnh CLI thủ công.
* **Quy trình 3 bước:** Định nghĩa môi trường chạy (Dockerfile) -> Mô tả cấu trúc hệ thống (docker-compose.yml) -> Khởi chạy toàn bộ bằng một lệnh duy nhất (`docker compose up`).

#### 4. Demo và thực hành

* **Mục tiêu demo:** Cài đặt kiểm tra Docker Compose, viết một file compose tối giản chạy thử container Java tester để kiểm tra hoạt động.
* **Command:**
```bash
# 1. Kiểm tra phiên bản Compose
docker compose version

# 2. Tạo file docker-compose.yml mẫu tối giản:
# version: '3.8'
# services:
#   java-tester:
#     image: eclipse-temurin:17-jre-alpine
#     command: java -version

# 3. Khởi chạy và dọn dẹp
docker compose up
docker compose down
```
* **Output mong đợi:** In ra phiên bản Compose, container chạy in thông tin version Java rồi tự dừng và được dọn dẹp sạch sẽ.

#### 5. Điểm cần nhấn mạnh

* Docker Compose gom tất cả tài nguyên (container, network, volume) vào một stack tập trung giúp dọn dẹp triệt để, không để lại rác hệ thống.

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Docker Compose là giải pháp tối ưu để triển khai và tự động co giãn (Auto Scaling) trên môi trường Production lớn nhiều server.
* **Đính chính:** Docker Compose chỉ phục vụ môi trường local, staging hoặc production nhỏ chạy trên single host. Khi hệ thống lớn và phức tạp, cần chuyển sang Kubernetes hoặc Docker Swarm.

---

### LESSON 03: Cấu trúc file docker-compose.yml (services, image, build)

#### 1. Mục tiêu bài học

* **Đọc và giải thích** được cấu trúc định dạng của tệp `docker-compose.yml` (các từ khóa chính: `services`, `image`, `build`, `depends_on`, `entrypoint`).
* **Vận dụng thuộc tính `build`** để tự động build image từ Dockerfile local khi up compose.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** STATE 2 — Multi-container Local System.
* **Vấn đề:** Làm sao để vừa khởi chạy một database có sẵn trên registry (`postgres`), vừa tự build và chạy mã nguồn Java local của `user-service` chỉ bằng một lệnh duy nhất mà không cần chạy `docker build` thủ công trước?

#### 3. Nội dung trọng tâm

* **Cấu trúc YAML:** Định dạng phân cấp bằng thụt lề khoảng trắng, phân biệt các khóa cốt lõi của file Compose.
* **Thuộc tính build:** Khai báo context đường dẫn tới thư mục chứa code và Dockerfile.
* **Depends_on & Entrypoint:** Quản lý thứ tự khởi chạy (DB chạy trước backend) và ghi đè lệnh chạy mặc định.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Viết file `docker-compose.yml` tích hợp `quickbite-db` (postgres) và backend `quickbite-user` tự build từ Dockerfile.
* **Command:**
```bash
# 1. Chuẩn bị cấu trúc thư mục gồm docker-compose.yml và thư mục con user-service (chứa Dockerfile và file JAR)
# 2. Khởi chạy kèm tham số build bắt buộc
docker compose up -d --build
```
* **Output mong đợi:** Compose tự động truy cập vào thư mục backend, build image từ Dockerfile, gắn thẻ và khởi động đồng thời cả 2 container database và backend.

#### 5. Điểm cần nhấn mạnh

* Thay đổi mã nguồn Java đòi hỏi phải biên dịch ra file JAR mới trên máy host và chạy lệnh kèm cờ `--build` thì Compose mới đóng gói lại image mới.

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Trong một service, nếu khai báo cả từ khóa `image` và `build` sẽ gây lỗi cú pháp.
* **Đính chính:** Hoàn toàn có thể. Khi đó Compose sẽ build từ Dockerfile và tự động đặt tên image theo nhãn khai báo ở `image`.

---

### LESSON 04: Biến môi trường và cấu hình port

#### 1. Mục tiêu bài học

* **Cấu hình** ánh xạ cổng (`ports`) để mở cổng truy cập từ bên ngoài vào container.
* **Truyền biến môi trường** (`environment`) để cấu hình động cho ứng dụng Spring Boot lúc chạy.
* **Bảo mật** thông tin nhạy cảm thông qua file ngoại vi `.env` và cơ chế nội suy biến.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** STATE 2 — Multi-container Local System.
* **Vấn đề:** Nếu ta viết cứng mật khẩu quản trị database và các cấu hình cổng vào trực tiếp trong `docker-compose.yml` rồi push lên Git, dữ liệu nhạy cảm sẽ bị rò rỉ. Hơn nữa, việc chuyển đổi cổng và cấu hình giữa các môi trường (Dev, Staging, Prod) sẽ đòi hỏi sửa đổi file YAML liên tục.

#### 3. Nội dung trọng tâm

* **Port Mapping:** Phân biệt cổng ngoài máy host và cổng trong container.
* **Nội suy biến (Interpolation):** Đọc giá trị từ file `.env` bằng cú pháp `${VARIABLE}`.
* **Quy chuẩn bảo mật:** Khóa file `.env` chứa mật khẩu thật bằng `.gitignore`, chỉ push file `.env.example` làm mẫu lên Git.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Thiết lập file `.env` để nạp cổng, mật khẩu database và nạp cấu hình qua lệnh config.
* **Command:**
```bash
# 1. Viết file .env: DB_PASSWORD=secret_pass ...
# 2. Khai báo các biến nội suy trong docker-compose.yml
# 3. Chạy lệnh kiểm thử nội suy biến
docker compose config
```
* **Output mong đợi:** Lệnh config in ra toàn bộ cấu hình YAML hoàn chỉnh với các giá trị biến môi trường thực tế đã được điền chính xác.

#### 5. Điểm cần nhấn mạnh

* File `.env` mặc định được Compose tự động nhận diện nếu nằm cùng thư mục chạy lệnh.

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Từ khóa `ports` và `expose` có chức năng giống nhau.
* **Đính chính:** `ports` mở cổng ra ngoài máy host vật lý. `expose` chỉ khai báo cổng giao tiếp nội bộ giữa các container trong mạng ảo, máy host không thể kết nối trực tiếp.

---

### LESSON 05: Volume và network trong Docker Compose

#### 1. Mục tiêu bài học

* **Cấu hình Named Volume** để lưu trữ dữ liệu database bền vững, tránh mất dữ liệu khi xóa container.
* **Ứng dụng Service Discovery** để kết nối microservices với database qua tên service thay vì IP cứng.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** STATE 2 — Multi-container Local System.
* **Vấn đề:** Mặc định hệ thống file container là tạm thời, chạy `docker compose down` sẽ xóa sạch dữ liệu trong database Postgres. Đồng thời, nếu database restart đổi IP, backend sẽ mất kết nối. Ta cần giải pháp lưu trữ bền vững và mạng ảo tự phân giải DNS.

#### 3. Nội dung trọng tâm

* **Named Volume:** Khai báo volume toàn cục và mount vào `/var/lib/postgresql/data` của Postgres.
* **User-defined Bridge Network:** Cơ chế mạng ảo mặc định của Compose, tự kích hoạt DNS nội bộ giúp phân giải tên service thành IP động.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Cấu hình volume và network trong file compose, chạy thử tạo dữ liệu, stop/down cụm và up lại để chứng minh dữ liệu còn nguyên vẹn.
* **Command:**
```bash
# 1. Khai báo volumes và networks trong file compose
# 2. Up cụm, tạo dữ liệu demo
# 3. Down cụm và Up lại
docker compose down
docker compose up -d
```
* **Output mong đợi:** Dữ liệu cũ trong database vẫn tồn tại nguyên vẹn sau khi dựng lại container.

#### 5. Điểm cần nhấn mạnh

* DNS nội bộ của mạng bridge mặc định của Compose cho phép gọi thẳng tên service (ví dụ: `quickbite-db:5432`) làm host kết nối.

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Lệnh `docker compose down` sẽ xóa toàn bộ dữ liệu trong Named Volume.
* **Đính chính:** Lệnh này chỉ xóa container và mạng. Dữ liệu trong volume vật lý trên host vẫn được giữ an toàn 100%. Chỉ mất dữ liệu nếu cố tình thêm cờ `-v` (`docker compose down -v`).

---

### LESSON 06: Quản lý vòng đời hệ thống với Docker Compose

#### 1. Mục tiêu bài học

* **Làm chủ** toàn diện các câu lệnh quản lý vòng đời hệ thống (`up`, `down`, `stop`, `start`).
* **Chẩn đoán lỗi** nhanh chóng thông qua xem danh sách (`ps`), log tập trung có màu sắc (`logs -f`) và tương tác trực tiếp (`exec`).

#### 2. Bối cảnh hệ thống

* **Trạng thái:** STATE 2 — Multi-container Local System.
* **Vấn đề:** Khi up cả cụm lên, có thể backend lỗi kết nối database và crash ngay lập tức. Để chẩn đoán, ta không thể chạy logs thủ công cho từng container rời rạc mà cần luồng log tổng hợp đồng bộ dòng thời gian của toàn bộ cụm.

#### 3. Nội dung trọng tâm

* **Nhóm lệnh điều khiển:** Phân biệt `stop` (tạm dừng giữ container) vs `down` (xóa sạch container).
* **Nhóm lệnh chẩn đoán:** Sử dụng `logs -f` có màu phân biệt giữa các service, sử dụng `exec` để kiểm tra kết nối nội bộ.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Thực hành chẩn đoán lỗi cụm container QuickBite bằng logs, ps và pg_isready.
* **Command:**
```bash
# 1. Khởi chạy ngầm
docker compose up -d
# 2. Xem logs tổng hợp thời gian thực
docker compose logs -f --tail=50
# 3. Thực thi kiểm tra trạng thái database nội bộ
docker compose exec quickbite-db pg_isready -U postgres
```
* **Output mong đợi:** Thấy logs của các container hiển thị phân biệt màu sắc, lệnh pg_isready báo database sẵn sàng kết nối.

#### 5. Điểm cần nhấn mạnh

* Nhấn Ctrl+C để thoát chế độ logs không làm ảnh hưởng tới trạng thái đang chạy của container.

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Lệnh `docker compose stop` cũng giải phóng hoàn toàn bộ nhớ RAM và dọn sạch container khỏi hệ thống.
* **Đính chính:** Lệnh `stop` chỉ dừng tiến trình, container vẫn tồn tại và chiếm tài nguyên lưu trữ cấu hình tạm. Bắt buộc dùng `down` để dọn dẹp sạch sẽ tài nguyên.

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

* **Vai trò:** Quản lý tài khoản, thông tin định danh, ví tiền (UserWallet) và các địa chỉ nhận hàng (UserAddress). Một User không thể đặt hàng nếu không có địa chỉ giao và không thể thanh toán nếu không có ví (mô phỏng).
* **Sơ đồ lớp thực thể (Entity Class):**
```java
@Entity @Table(name = "users")
public class User {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String password;
    private String fullName;
    @Enumerated(EnumType.STRING)
    private Role role; // CUSTOMER, DRIVER, MERCHANT

    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
    private List<UserAddress> addresses; // Phục vụ cho việc chọn địa chỉ Ship

    @OneToOne(mappedBy = "user", cascade = CascadeType.ALL)
    private UserWallet wallet; // Phục vụ trừ tiền khi đặt đơn
}

@Entity @Table(name = "user_addresses")
public class UserAddress {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String label; // Nhà riêng, Công ty
    private String detailAddress;
    private boolean isDefault;
    
    @ManyToOne @JoinColumn(name = "user_id")
    private User user;
}

@Entity @Table(name = "user_wallets")
public class UserWallet {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private BigDecimal balance; // Số dư tài khoản mô phỏng
    
    @OneToOne @JoinColumn(name = "user_id")
    private User user;
}
```

##### B. Restaurant Service (`restaurant-service`)

* **Vai trò:** Quản lý thông tin nhà hàng, trạng thái đóng/mở cửa, phân chia danh mục (MenuCategory) và các món ăn (MenuItem) tương ứng.
* **Sơ đồ lớp thực thể (Entity Class):**
```java
@Entity @Table(name = "restaurants")
public class Restaurant {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private Long ownerId; // Tham chiếu sang User (Role MERCHANT)
    private boolean isOpen;

    @OneToMany(mappedBy = "restaurant", cascade = CascadeType.ALL)
    private List<MenuCategory> categories; // Cơm, Nước uống, Tráng miệng
}

@Entity @Table(name = "menu_categories")
public class MenuCategory {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @ManyToOne @JoinColumn(name = "restaurant_id")
    private Restaurant restaurant;

    @OneToMany(mappedBy = "category", cascade = CascadeType.ALL)
    private List<MenuItem> items;
}

@Entity @Table(name = "menu_items")
public class MenuItem {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private BigDecimal basePrice;
    private boolean isAvailable;

    @ManyToOne @JoinColumn(name = "category_id")
    private MenuCategory category;
}
```

##### C. Order Service (`order-service`)

* **Vai trò:** Trọng tâm xử lý luồng nghiệp vụ. Tiếp nhận yêu cầu đặt món, lưu vết lịch sử trạng thái (OrderStatusHistory) để tracking, và gán thông tin phân bổ Tài xế (driverId).
* **Sơ đồ lớp thực thể (Entity Class):**
```java
@Entity @Table(name = "orders")
public class Order {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private Long customerId;
    private Long restaurantId;
    private Long driverId; // Sẽ được cập nhật khi có Tài xế nhận đơn
    private Long deliveryAddressId; // Tham chiếu sang Địa chỉ của User

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> items;

    private BigDecimal itemsPrice; // Tiền món ăn
    private BigDecimal shippingFee; // Phí ship tính riêng
    private BigDecimal totalPrice;  // Tổng tiền khách trả

    @Enumerated(EnumType.STRING)
    private OrderStatus status; 

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderStatusHistory> statusHistories; // Lưu vết dòng thời gian (Timeline)
}

@Entity @Table(name = "order_items")
public class OrderItem {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private Long menuItemId; 
    private String itemName; // Snapshot tên món tại thời điểm mua (Tránh Merchant đổi tên món làm sai lệch lịch sử)
    private Integer quantity;
    private BigDecimal price;

    @ManyToOne @JoinColumn(name = "order_id")
    private Order order;
}

@Entity @Table(name = "order_status_history")
public class OrderStatusHistory {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Enumerated(EnumType.STRING)
    private OrderStatus status; // Lưu lại mốc thời gian chuyển trạng thái
    private LocalDateTime changedAt;
    private String note; // Lý do hủy đơn, lý do từ chối...

    @ManyToOne @JoinColumn(name = "order_id")
    private Order order;
}
```

##### D. Notification Service (`notification-service`)

* **Vai trò:** Tiếp nhận sự kiện thay đổi trạng thái đơn hàng để gửi thông báo, phân biệt rõ các kênh (In-app, Email, SMS) và lưu vết trạng thái gửi (deliveryStatus).
* **Sơ đồ lớp thực thể (Entity Class):**
```java
@Entity @Table(name = "notifications")
public class Notification {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private Long userId; // Người nhận
    private String title;
    private String content;
    
    @Enumerated(EnumType.STRING)
    private NotificationType type; // IN_APP, EMAIL, SMS
    
    @Enumerated(EnumType.STRING)
    private DeliveryStatus deliveryStatus; // PENDING, SENT, FAILED
    
    private LocalDateTime createdAt;
}
```

#### 3. Vận hành "Dây chuyền" của các Thực thể mới

Khi sinh viên code, họ sẽ thấy các service tương tác với nhau chặt chẽ theo từng bước logic của một platform giao đồ ăn thực tế, kể cả khi chỉ gọi API thuần túy (Feign Client / WebClient):

1. **Khách hàng chọn món & bấm Đặt đơn:** Order Service tạo Order (status: `PENDING`) -> Đồng thời tạo một bản ghi vào `OrderStatusHistory` (lưu mốc thời gian tạo).
2. **Trừ tiền thanh toán:** Order Service gọi sang User Service -> Kiểm tra ví tiền (`UserWallet`) xem đủ số dư không. Nếu đủ -> Trừ tiền (Mô phỏng thanh toán) -> Cập nhật Order thành `ACCEPTED`.
3. **Thông báo nhà hàng:** Order Service gọi sang Restaurant Service -> Đẩy thông báo cho Merchant chuẩn bị món ăn.
4. **Điều phối tài xế:** Order Service kích hoạt luồng "Điều phối" (Mô phỏng gán tài xế) -> Tìm User có role = `DRIVER` -> Gán `driverId` vào Order -> Cập nhật status thành `SHIPPING` + Thêm một dòng History mới.
5. **Đẩy thông báo trạng thái:** Ở mỗi bước chuyển trạng thái (`PENDING` -> `ACCEPTED` -> `SHIPPING`), Order Service đều gửi tín hiệu (HTTP Post) sang Notification Service -> Notification Service tạo bản ghi, phân loại (`IN_APP`/`EMAIL`) rồi chuyển `deliveryStatus` thành `SENT`.

#### 4. Ý nghĩa thiết kế: Giải quyết triệt để bệnh "Underengineer"

* **Nghiệp vụ thực tế hơn:** Việc tích hợp phí ship (`shippingFee`), lịch sử trạng thái (`OrderStatusHistory`), ví tiền (`UserWallet`) buộc sinh viên phải tư duy hệ thống nghiêm túc, không thể làm kiểu hời hợt.
* **Bài toán Data Integrity (Toàn vẹn dữ liệu):** Khi `Order Service` đã trừ tiền ở `User Service`, nhưng đến bước báo sang `Restaurant Service` thì nhà hàng lại bấm "Từ chối" vì hết món. Sinh viên sẽ phải tự viết code Logic để gọi lại `User Service` cộng lại tiền vào `UserWallet` (Mô phỏng luồng hoàn tiền - Rollback bằng code).
* **Giải quyết bài toán Snapshot:** Nhìn vào `OrderItem`, việc lưu `itemName` trực tiếp thay vì chỉ lưu ID bắt buộc sinh viên phải suy nghĩ về tư duy thiết kế hệ thống thương mại điện tử: *Nếu ngày mai nhà hàng đổi tên món từ "Phở" thành "Phở đặc biệt", đơn hàng hôm nay in ra hóa đơn vẫn phải giữ đúng tên "Phở" tại thời điểm mua.*

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

1. **Bước 1 (Tiếp nhận & Tạo đơn):** Khách hàng chọn món & bấm Đặt đơn. Request đi qua Gateway chuyển tới `order-service` (Port 8083). `order-service` tạo Order mới (trạng thái `status = PENDING`) và lưu vết timeline vào `OrderStatusHistory`.
2. **Bước 2 (Xác thực thanh toán):** `order-service` gọi API sang `user-service` (Port 8081) để kiểm tra số dư ví (`UserWallet`). Nếu tài khoản đủ tiền, hệ thống trừ tiền (mô phỏng thanh toán) và cập nhật đơn hàng thành `ACCEPTED`.
3. **Bước 3 (Chuẩn bị món):** `order-service` gọi sang `restaurant-service` (Port 8082) để gửi thông báo cho Merchant chuẩn bị món ăn.
4. **Bước 4 (Điều phối & Giao hàng):** `order-service` kích hoạt luồng điều phối tìm Tài xế (`User` với `Role = DRIVER`). Khi tài xế nhận đơn, gán `driverId` vào đơn hàng và cập nhật trạng thái thành `SHIPPING` (kèm theo một bản ghi lịch sử trạng thái mới).
5. **Bước 5 (Thông báo trạng thái):** Ở mỗi bước chuyển trạng thái (`PENDING` -> `ACCEPTED` -> `SHIPPING`), `order-service` gửi tín hiệu (HTTP Post) sang `notification-service` (Port 8084) để tạo và gửi thông báo (`IN_APP` hoặc `EMAIL`) tới người dùng tương ứng.

#### Điểm cần nhấn mạnh cho giảng viên (Pedagogical Tips):

* **Tính độc lập dữ liệu:** Nhấn mạnh rằng dù chạy chung một container PostgreSQL vật lý để tiết kiệm RAM khi làm demo tại lớp, các service kết nối vào 4 database hoàn toàn khác biệt bằng tài khoản được phân quyền riêng. `order-service` tuyệt đối không được phép thực hiện câu lệnh SQL Join sang bảng `users` hay `restaurants` mà bắt buộc phải đi qua giao tiếp mạng (REST API) sử dụng cơ chế Service Discovery của Docker Network.
* **Tách biệt cấu hình:** Nhờ cách thiết kế đặt giá trị mặc định dạng `${BIẾN:Giá_trị_mặc_định}`, mã nguồn ứng dụng có thể chạy mượt mà ở máy local của sinh viên khi làm bài tập (`localhost`) và tự động chuyển hướng mượt mà sang gọi tên container (`quickbite-db`) khi đóng gói chạy Docker mà không cần sửa đổi bất kỳ dòng code Java nào.
* **Bài toán Toàn vẹn & Rollback:** Hãy chỉ cho sinh viên thấy tầm quan trọng của việc xử lý lỗi (ví dụ: hoàn tiền vào ví nếu nhà hàng hết món và hủy đơn) để rèn luyện tư duy lập trình hệ thống phân tán.

---

*Giảng viên lưu ý: Việc nắm chắc chi tiết cấu trúc Entity, Port, và cách liên kết DNS của từng service tại Session 05 là bước đệm bắt buộc để sinh viên bước sang **Session 06**, nơi các em sẽ tự tay viết file `docker-compose.yml` để khởi chạy toàn bộ 4 dịch vụ này chỉ bằng một câu lệnh duy nhất.*

---

# SESSION 07

## TỰ ĐỘNG HÓA BIÊN DỊCH VỚI GITLAB CI

---

### LESSON 01: Tổng quan GitLab CI/CD và Kiến trúc GitLab Runner

#### 1. Mục tiêu bài học

* **Giải thích** được định nghĩa, vai trò và vị trí của GitLab CI/CD trong quy trình phát triển và vận hành DevOps.
* **Phân tích** kiến trúc Client-Server giữa GitLab Server và GitLab Runner trong việc nhận diện và điều phối các công việc (jobs).
* **Phân biệt** được các loại Runner (Shared Runner, Specific Runner) và các loại Executor phổ biến (Docker, Shell).

#### 2. Bối cảnh hệ thống

* **Trạng thái:** STATE 2 — Multi-Container Local System (Các dịch vụ QuickBite đã đóng gói và chạy ổn định bằng Docker Compose trên môi trường local).
* **Vấn đề:** Mỗi khi mã nguồn cập nhật, lập trình viên vẫn phải kiểm thử và đóng gói thủ công trên máy cá nhân trước khi phân phối. Quy trình này dễ xảy ra sai sót. Cần một hệ thống tự động hóa tập trung để giải phóng sức người và đảm bảo tính nhất quán trên các môi trường. Ở bước này, chúng ta tận dụng GitLab Shared Runner trực tuyến để tối ưu hóa thời gian triển khai, đồng thời học viên vẫn được hướng dẫn cách thiết lập Runner tự host để hiểu bản chất kỹ thuật.

#### 3. Nội dung trọng tâm

* **Khái niệm CI/CD:** Continuous Integration (Tích hợp liên tục - liên tục hợp nhất, chạy test tự động khi có code mới) và Continuous Delivery/Deployment (Chuyển giao/Triển khai liên tục).
* **Kiến trúc GitLab CI/CD:**
  * *GitLab Server:* Nơi lưu trữ mã nguồn, quản lý giao diện trực quan và điều phối các luồng chạy pipeline.
  * *GitLab Runner:* Một dịch vụ/máy ảo độc lập (Executor) trực tiếp thực thi các câu lệnh được định nghĩa trong pipeline.
* **Phân loại Runner:** Shared Runner (dùng chung cho toàn hệ thống do GitLab cung cấp) và Specific Runner (dành riêng cho dự án cụ thể tự host).
* **Executor:** Cần phân biệt Docker Executor (mỗi job chạy trong 1 container cô lập hoàn toàn, sạch sẽ) và Shell Executor (chạy trực tiếp lệnh trên hệ điều hành máy host của Runner).

#### 4. Demo và thực hành

* **Mục tiêu demo:** Cài đặt và đăng ký một GitLab Runner cục bộ (Local Specific Runner) sử dụng Docker Compose và Docker Executor cho dự án QuickBite.
* **Cấu hình Docker Compose (`docker-compose.yml`):**
```yaml
version: '3.8'

services:
  gitlab-runner:
    image: gitlab/gitlab-runner:latest
    container_name: local-gitlab-runner
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - gitlab-runner-config:/etc/gitlab-runner

volumes:
  gitlab-runner-config:
```
* **Command:**
```bash
# 1. Khởi chạy GitLab Runner bằng Docker Compose
docker compose up -d

# 2. Đăng ký Runner vào dự án GitLab bằng Authentication Token mới (lấy từ Settings -> CI/CD -> Runners)
docker compose exec gitlab-runner gitlab-runner register \
  --non-interactive \
  --url "https://gitlab.com/" \
  --token "GLRT-YOUR_NEW_AUTHENTICATION_TOKEN_HERE" \
  --executor "docker" \
  --docker-image "alpine:latest" \
  --description "Local Runner for QuickBite Project" \
  --tag-list "docker,quickbite" \
  --run-untagged="true" \
  --locked="false"
```
* **Output mong đợi:** Đăng ký thành công. Trên giao diện quản lý GitLab (Settings -> CI/CD -> Runners), hiển thị một Runner mới ở trạng thái hoạt động (màu xanh lá cây) với tag `docker,quickbite`.

#### 5. Lưu ý và lỗi sai thường gặp

* **Quên mount Docker socket:** Khi thiết lập file `docker-compose.yml` cho tự host Runner, sinh viên thường quên mount đường dẫn `/var/run/docker.sock:/var/run/docker.sock`. Điều này khiến Docker Executor hoàn toàn mất quyền khởi tạo container phụ cho các job và làm sập pipeline.
* **Sử dụng sai Token đăng ký:** Nhiều sinh viên nhầm lẫn sử dụng mã Registration Token cũ (đã bị GitLab khai tử) thay vì tạo Authentication Token mới có tiền tố `GLRT-` trên giao diện GitLab, dẫn đến lỗi xác thực đăng ký thất bại.

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Cho rằng GitLab Server trực tiếp thực thi các câu lệnh biên dịch và kiểm thử được định nghĩa trong file cấu hình.
* **Đính chính:** GitLab Server chỉ đóng vai trò điều hướng và hiển thị giao diện. Việc thực thi câu lệnh hoàn toàn do GitLab Runner đảm nhận. Nếu không cấu hình Runner, pipeline sẽ bị kẹt ở trạng thái `pending` vô hạn.

---

### LESSON 02: Cấu trúc và Cú pháp file `.gitlab-ci.yml`

#### 1. Mục tiêu bài học

* **Nắm vững** cú pháp YAML và quy tắc thụt lề bắt buộc trong file cấu hình `.gitlab-ci.yml`.
* **Khai báo và cấu trúc** thành công một file cấu hình cơ bản gồm các thành phần cốt lõi: `stages`, `image`, `variables`, và các định nghĩa `job`.
* **Sử dụng** được các từ khóa kiểm soát luồng chạy của job như `only`, `except`, `rules`, `before_script`, `after_script`.
* **Áp dụng** cơ chế `tags` để điều phối job chạy trên đúng môi trường mong muốn.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** GitLab Runner đã sẵn sàng nhận lệnh (sử dụng Shared Runner hoặc Local Runner đã đăng ký).
* **Vấn đề:** Cần khai báo các bước kiểm thử, biên dịch cho GitLab Runner hiểu thông qua một "kịch bản" chuẩn hóa. File `.gitlab-ci.yml` là đặc tả kịch bản đó, được lưu trữ trực tiếp cùng mã nguồn để áp dụng cơ chế "Infrastructure as Code".
* **Lưu ý cấu trúc thư mục:**
  > *Mỗi service (`user-service`, `restaurant-service`...) hiện tại đã là một Git Repository độc lập. File `.gitlab-ci.yml` dưới đây phải được đặt ngay tại thư mục gốc của chính service đó (ví dụ: `user-service/.gitlab-ci.yml`), không đặt ở thư mục cha.*

#### 3. Nội dung trọng tâm

* **Cú pháp YAML:** Cấu trúc key-value, phân cấp thụt lề bằng khoảng trắng (spaces), tuyệt đối không được dùng phím Tab.
* **Cấu trúc file `.gitlab-ci.yml`:**
  * `image`: Docker image mặc định làm môi trường chạy cho các job (ví dụ: `eclipse-temurin:17-jdk-alpine`).
  * `stages`: Định nghĩa danh sách các giai đoạn của quy trình và thứ tự thực thi của chúng (ví dụ: `compile`, `test`, `package`).
  * `variables`: Định nghĩa các biến môi trường dùng chung trong pipeline.
  * `job`: Khối lệnh thực thi độc lập. Mỗi job bắt buộc phải thuộc về một stage (`stage: compile`) và chứa kịch bản dòng lệnh (`script`).
* **Sử dụng tags:** Chỉ định tag của Runner cụ thể được phép nhận job (ví dụ: `tags: [quickbite]`). Nếu không khai báo, job có thể bị các Runner dùng chung khác giành quyền thực thi (hiện tượng cướp job), dẫn đến sai khác môi trường.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Tạo file `.gitlab-ci.yml` tối giản đầu tiên tại thư mục gốc của dự án `user-service` có khai báo tags để in ra thông tin môi trường.
* **Cấu hình hệ thống (`user-service/.gitlab-ci.yml`):**
```yaml
image: eclipse-temurin:17-jdk-alpine

variables:
  PROJECT_NAME: "QuickBite-User-Service"

stages:
  - info

print_env_job:
  stage: info
  tags:
    - quickbite # Chỉ định Runner local tránh hiện tượng bị cướp job
  before_script:
    - echo "Chuẩn bị in thông tin môi trường cho dự án ${PROJECT_NAME}..."
  script:
    - java -version
    - echo "Tên dự án hiện tại là ${PROJECT_NAME}"
    - echo "Nhánh Git đang chạy pipeline là ${CI_COMMIT_BRANCH}"
  after_script:
    - echo "Hoàn thành job in thông tin môi trường."
```
* **Output mong đợi:** Khi push code lên nhánh chính, pipeline tự động kích hoạt. Job `print_env_job` hoàn thành ở trạng thái Passed, log hiển thị chi tiết thông tin phiên bản Java 17 của image Eclipse Temurin và giá trị các biến môi trường nội suy.

#### 5. Lưu ý và lỗi sai thường gặp

* **Đặt sai vị trí file cấu hình:** Sinh viên có thói quen viết file `.gitlab-ci.yml` và đặt ở thư mục tổng của dự án QuickBite thay vì đặt ở thư mục gốc của từng repository của service độc lập (như `user-service/.gitlab-ci.yml`). Điều này khiến GitLab Server không thể phát hiện file cấu hình của từng service độc lập.
* **Thụt lề YAML sai quy chuẩn:** YAML bắt buộc thụt lề bằng khoảng trắng (space). Lỗi cực kỳ phổ biến của sinh viên là sử dụng phím Tab để thụt lề hoặc thụt lề không đồng đều giữa các thuộc tính cùng cấp, dẫn đến lỗi phân tích cú pháp YAML.

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Nghĩ rằng các biến môi trường bắt đầu bằng `$CI_` (như `$CI_COMMIT_BRANCH`) phải được khai báo trong phần `variables` thì mới sử dụng được.
* **Đính chính:** Đây là các **Predefined Variables (Biến môi trường định sẵn)** do GitLab tự động nạp vào mọi container chạy job để cung cấp thông tin về commit, tác giả, branch, merge request... Lập trình viên chỉ việc gọi ra sử dụng trực tiếp.

---

### LESSON 03: Cơ chế hoạt động của Pipeline và Phân tách Stages

#### 1. Mục tiêu bài học

* **Phân tích** được luồng hoạt động tuần tự (Sequential) của các `stages` và song song (Parallel) của các `jobs` trong một pipeline.
* **Giải thích** được cơ chế hoạt động độc lập và cô lập (Isolation) của từng job chạy trên Docker Executor.
* **Vận dụng** phân tách cấu trúc stages để quản lý tiến trình tự động hóa.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** Lập trình viên đã biết cách viết file `.gitlab-ci.yml` cơ bản.
* **Vấn đề:** Để nắm rõ nguyên lý hoạt động của CI/CD, chúng ta cần trực quan hóa cách các stages và các jobs chạy như thế nào trong thực tế. Việc thiết lập các công việc giả lập giúp dễ dàng quan sát luồng thực thi song song và tuần tự mà không mất thời gian biên dịch mã nguồn thật.

#### 3. Nội dung trọng tâm

* **Stages vs Jobs:** Các job thuộc cùng một stage sẽ chạy song song (nếu hệ thống có đủ Runner), trong khi các stage sẽ chạy tuần tự. Nếu một job trong stage trước thất bại, toàn bộ các stage sau sẽ bị hủy bỏ (skipped).
* **Sự cô lập của Job (Job Isolation):** Mỗi job khi bắt đầu sẽ clone mã nguồn mới nhất về một container mới tinh, sạch sẽ. Khi job kết thúc, container bị hủy, mọi file phát sinh sẽ bị xóa sạch.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Viết pipeline có 2 stage giả lập (`info` và `print`) với 2 job chạy song song trong stage đầu tiên để kiểm chứng tiến trình chạy.
* **Cấu hình hệ thống (`user-service/.gitlab-ci.yml`):**
```yaml
stages:
  - info
  - print

job_info_1:
  stage: info
  tags:
    - quickbite
  script:
    - echo "Bắt đầu thu thập thông tin hệ thống..."
    - uname -a

job_info_2:
  stage: info
  tags:
    - quickbite
  script:
    - echo "Bắt đầu kiểm tra môi trường..."
    - whoami

job_print:
  stage: print
  tags:
    - quickbite
  script:
    - echo "In ra kết quả cuối cùng từ pipeline..."
```
* **Output mong đợi:** Khi pipeline chạy, cả hai job `job_info_1` và `job_info_2` xuất hiện đồng thời và chạy song song. Sau khi cả hai job này hoàn thành thành công, job `job_print` ở stage `print` mới được kích hoạt chạy tuần tự tiếp theo.

#### 5. Lưu ý và lỗi sai thường gặp

* **Cấu hình sai thứ tự stage:** Gán job vào một stage không tồn tại trong danh sách `stages` toàn cục hoặc ghi sai chính tả tên stage làm pipeline báo lỗi cấu hình không hợp lệ (`invalid configuration`).
* **Đồng bộ hóa trạng thái job:** Lầm tưởng các job song song có thể can thiệp dữ liệu lẫn nhau trong quá trình chạy. Thực tế, chúng chạy trên các container độc lập hoàn toàn.

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Nghĩ rằng các job trong cùng một stage luôn chạy cùng một lúc trên một máy duy nhất.
* **Đính chính:** Các job song song được phân phối độc lập. Chúng có thể chạy trên các máy Runner khác nhau nếu hệ thống có cấu hình multi-runner, đảm bảo tính phân tán và tối ưu tài nguyên.

---

### LESSON 04: Build ứng dụng Spring Boot bằng Maven/Gradle trong CI

#### 1. Mục tiêu bài học

* **Cấu hình** được môi trường biên dịch (JDK, Gradle Wrapper) phù hợp với phiên bản Java của microservice.
* **Biên soạn** thành công pipeline hoàn chỉnh thực hiện: Cấp quyền Gradle Wrapper -> Biên dịch ra file JAR thương mại (`bootJar`) và lưu vào artifacts một cách tối giản.
* **Tối ưu hóa** hiệu năng pipeline chạy trên Shared Runner bằng cách loại bỏ các cấu hình cache rườm rà và bỏ qua bước test để pipeline chạy nhanh nhất.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** Các microservices QuickBite chạy Java 17 (`user-service`, `order-service`) và Java 21 (`restaurant-service`) cần được tự động hóa quy trình build.
* **Vấn đề:** Để tối ưu hóa thời gian chạy trên Shared Runner của GitLab và đạt tốc độ thực thi nhanh nhất, chúng ta cần tinh giản tối đa các bước trung gian không cần thiết, tập trung thẳng vào mục tiêu build ra file JAR thành phẩm. Đồng thời, do tính chất cô lập của container, file JAR sinh ra sẽ bị mất khi job kết thúc trừ khi ta dùng cơ chế `artifacts` để lưu trữ nó tạm thời trên GitLab Server.

#### 3. Nội dung trọng tâm

* **Base Image phù hợp:** Lựa chọn các image chứa JDK đầy đủ (như `eclipse-temurin:17-jdk-alpine`) làm môi trường chạy các lệnh Gradle/Maven.
* **Tối giản hóa quy trình CI:** Sử dụng tác vụ `bootJar` để biên dịch và đóng gói nhanh chóng (tác vụ này mặc định không chạy các bài kiểm thử unit test) để đạt hiệu năng thực thi tối đa.
* **Cơ chế Artifacts:** Khai báo `artifacts` để đóng gói file JAR sinh ra và đẩy lên GitLab Server lưu giữ tạm thời.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Xây dựng pipeline CI tinh giản hoàn chỉnh cho dịch vụ `user-service` thực hiện đóng gói và lưu giữ file JAR Spring Boot.
* **Cấu hình hệ thống (`user-service/.gitlab-ci.yml`):**
```yaml
image: eclipse-temurin:17-jdk-alpine

stages:
  - build

build_executable_jar:
  stage: build
  tags:
    - quickbite
  script:
    # Cấp quyền thực thi cho Gradle Wrapper có sẵn trong dự án
    - chmod +x ./gradlew
    # Biên dịch và đóng gói file JAR (tác vụ bootJar mặc định không chạy kiểm thử)
    - ./gradlew bootJar
  artifacts:
    paths:
      - build/libs/*.jar
    expire_in: 3 days
```
* **Output mong đợi:** Pipeline chạy thành công. Job `build_executable_jar` sinh ra file `user-service.jar` và lưu vào mục Artifacts trên hệ thống GitLab Server.

#### 5. Lưu ý và lỗi sai thường gặp

* **Quên cấp quyền chạy cho Gradle Wrapper:** Không khai báo lệnh `chmod +x ./gradlew` trước khi thực thi build, dẫn đến lỗi `Permission Denied` trên môi trường Linux của Runner.
* **Quên khai báo artifacts để chuyển giao file:** Nghĩ rằng file JAR được compile ở job trước mặc định sẽ tồn tại ở job sau. Bắt buộc phải khai báo từ khóa `artifacts` kèm theo đường dẫn tệp tin để lưu trữ.

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Cho rằng thuộc tính `cache` và `artifacts` có chức năng giống nhau và có thể thay thế cho nhau.
* **Đính chính:** `artifacts` dùng để chuyển giao các file kết quả đầu ra (như file JAR) giữa các stage; `cache` dùng để lưu trữ các thư mục thư viện phụ thuộc (`.gradle/caches`) để tăng tốc độ tải cho các lần chạy sau.

---

### LESSON 05: Phân tích log pipeline và xử lý lỗi build

#### 1. Mục tiêu bài học

* **Định vị và mở** giao diện Log Console của Job trên GitLab Web UI.
* **Chẩn đoán** nguyên nhân gây sập pipeline thông qua phương pháp đọc log từ dưới lên.
* **Xử lý** ba kịch bản lỗi thực tế kế thừa từ pipeline build của Lesson 04: lỗi phân quyền (`Permission Denied`), lỗi biên dịch (`Compilation Failed`), và lỗi kiểm thử thất bại (`Test FAILED`).

#### 2. Bối cảnh hệ thống

* **Trạng thái:** Kế thừa cấu hình build dự án `user-service` từ Lesson 04.
* **Vấn đề:** Khi pipeline báo đỏ (Failed), hệ thống là một hộp đen nếu sinh viên không biết cách truy cập và phân tích log console. Cần các kịch bản lỗi thực tế để giảng viên demo trực tiếp trên GitLab, giúp sinh viên nhận diện lỗi và biết cách sửa đổi mã nguồn/cấu hình.

#### 3. Nội dung trọng tâm

* **Kỹ năng đọc log console:** Phân tích mã lỗi thoát (`exit code`), xác định câu lệnh gây lỗi và tra cứu chi tiết thông tin từ compiler/test framework.
* **Kịch bản lỗi 1 (Permission Denied):** Lỗi sập do thiếu lệnh cấp quyền thực thi cho Gradle Wrapper.
* **Kịch bản lỗi 2 (Compilation Failed):** Lỗi trình biên dịch do sai cú pháp hoặc sai tên class/interface trong mã nguồn Java.
* **Kịch bản lỗi 3 (Test FAILED):** Lỗi kiểm thử khi thay đổi lệnh build để kích hoạt các ca kiểm thử và gặp assert thất bại.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Giảng viên thao tác trực tiếp trên GitLab, chạy pipeline lỗi để sinh viên thực hành đọc log chẩn đoán tại chỗ.
* **Kịch bản 1 (Permission Denied):**
  * *Tạo lỗi:* Xóa dòng `- chmod +x ./gradlew` trong `.gitlab-ci.yml`.
  * *Log sập:* `/bin/sh: eval: line 135: ./gradlew: Permission denied` và `exit code 1`.
  * *Cách xử lý:* Thêm lại lệnh `chmod +x ./gradlew` trước lệnh build.
* **Kịch bản 2 (Compilation Failed):**
  * *Tạo lỗi:* Viết sai cú pháp Java trong `UserService.java` (ví dụ sửa thành `UserReposotory`).
  * *Log sập:* `Task :compileJava FAILED` và chỉ ra dòng code lỗi chính tả.
  * *Cách xử lý:* Sửa đúng cú pháp ở local và push lại.
* **Kịch bản 3 (Test FAILED):**
  * *Tạo lỗi:* Thay đổi câu lệnh trong file cấu hình `.gitlab-ci.yml` từ `./gradlew bootJar` sang `./gradlew build` để chạy test, kết hợp sửa đổi assert của một Unit Test ở local để nó fail.
  * *Log sập:* `Task :test FAILED` kèm chi tiết `AssertionError` của testcase cụ thể.
  * *Cách xử lý:* Sửa lại logic code hoặc testcase cho đúng và push lại.

#### 5. Lưu ý và lỗi sai thường gặp

* **Sửa code mò mẫm không đọc log:** Học viên thường sửa code ngẫu nhiên khi thấy pipeline báo đỏ thay vì đọc log console để định vị vị trí và nguyên nhân lỗi chính xác.
* **Bỏ qua mã lỗi Exit Code:** Không phân biệt các mã exit code của hệ điều hành (exit code 1 do lỗi biên dịch/phân quyền, exit code 127 do gõ sai lệnh).

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Lầm tưởng mọi lỗi đỏ của pipeline luôn do lỗi cú pháp Java trong mã nguồn.
* **Đính chính:** Lỗi đỏ có thể phát sinh từ cấu hình YAML sai thụt lề, phân quyền của Runner (Permission Denied) hoặc sai phiên bản JDK nền của Docker image.

---

# SESSION 08

## ĐÓNG GÓI DOCKER IMAGE & ĐẨY LÊN REGISTRY

---

### LESSON 01: Quy trình Build Docker image trong pipeline CI/CD

#### 1. Mục tiêu bài học

* **Giải thích** được thách thức khi build Docker image bên trong môi trường chạy của Runner (cô lập).
* **Áp dụng** giải pháp Docker-in-Docker (DinD) để chuẩn bị môi trường chạy các câu lệnh Docker.
* **Cấu hình** thành công Docker service phụ trợ trong file CI/CD.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** File JAR Spring Boot đã được tạo ra từ Session 07 và sẵn sàng để đóng gói thành container.
* **Vấn đề:** Để tuân thủ kiến trúc Microservices Containerization, mỗi service cần có Docker image riêng. Tuy nhiên, Runner bản chất đang chạy trong một Docker container cô lập. Để chạy lệnh `docker build` tạo ra image mới, ta cần một cơ chế cho phép container Runner giao tiếp với một Docker Daemon.

#### 3. Nội dung trọng tâm

* **Khái niệm Docker-in-Docker (DinD):** Giải pháp chạy một Docker daemon phụ bên trong container Runner, cung cấp môi trường độc lập hoàn toàn để Runner gọi lệnh `docker build`, `docker login`, `docker push`.
* **Cấu hình `services`:** Sử dụng từ khóa `services: [docker:dind]` trong file `.gitlab-ci.yml` để kích hoạt container phụ trợ chạy daemon Docker song song với job chính.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Cấu hình file `.gitlab-ci.yml` dùng image `docker` và dịch vụ `dind` để kiểm tra môi trường Docker hoạt động trong CI.
* **Cấu hình hệ thống (`user-service/.gitlab-ci.yml`):**
```yaml
image: docker:latest

services:
  - docker:dind

stages:
  - test_docker

check_docker_job:
  stage: test_docker
  tags:
    - quickbite
  script:
    - docker info
```
* **Output mong đợi:** Job chạy thành công và in ra thông số cấu hình Docker Engine đang hoạt động bên trong Runner.

#### 5. Lưu ý và lỗi sai thường gặp

* **Thiếu khai báo services:** Quên khai báo `services: [docker:dind]` khiến job chính báo lỗi không thể kết nối tới Docker daemon (`Cannot connect to the Docker daemon`).
* **Phiên bản Docker không đồng nhất:** Dùng phiên bản image `docker` và service `docker:dind` lệch nhau quá nhiều gây ra xung đột giao thức giao tiếp.

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Cho rằng chỉ cần Runner chạy trên máy có cài Docker là mọi job mặc định có thể gọi lệnh `docker` mà không cần cấu hình thêm gì.
* **Đính chính:** Runner chạy bằng Docker Executor cô lập hoàn toàn môi trường của nó với máy host. Bắt buộc phải khai báo dịch vụ `docker:dind` để tự tạo ra Docker Engine nội bộ cho job đó chạy.

---

### LESSON 02: Tối ưu hóa Dockerfile cho Production (Multi-stage build)

#### 1. Mục tiêu bài học

* **Phân tích** được nhược điểm về kích thước và bảo mật của các Dockerfile truyền thống.
* **Ứng dụng** kỹ thuật Multi-stage build để chia quy trình đóng gói thành nhiều giai đoạn.
* **Viết** được Dockerfile tối ưu kế thừa trực tiếp file JAR từ artifacts.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** File JAR đã được biên dịch sẵn ở Session 07.
* **Vấn đề:** Nếu viết Dockerfile chứa đầy đủ cả công cụ compile mã nguồn (JDK) và mã nguồn thô, dung lượng image cuối cùng sẽ rất lớn (hơn 500MB). Cần tối ưu để image production chỉ chứa file JAR thực thi và môi trường chạy tối thiểu (JRE Alpine).

#### 3. Nội dung trọng tâm

* **Multi-stage build:** Sử dụng nhiều câu lệnh `FROM` trong cùng một Dockerfile để copy sản phẩm từ stage này sang stage khác, loại bỏ các công cụ build thừa thãi.
* **JRE Alpine Image:** Sử dụng base image JRE Alpine gọn nhẹ (chỉ khoảng 50MB-100MB) để tối ưu kích thước truyền tải và giảm bề mặt tấn công bảo mật.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Viết Dockerfile tối ưu 2 giai đoạn kế thừa file JAR từ artifacts để đóng gói ứng dụng Spring Boot.
* **Cấu hình hệ thống (`user-service/Dockerfile`):**
```dockerfile
# Sử dụng JRE gọn nhẹ cho môi trường chạy Production
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
# Sao chép trực tiếp file JAR đã được build từ artifacts của CI
COPY build/libs/*.jar app.jar
EXPOSE 8081
ENTRYPOINT ["java", "-jar", "app.jar"]
```
* **Output mong đợi:** Dockerfile hoạt động trơn tru. Khi build image, dung lượng image thu gọn tối đa và không chứa mã nguồn thô.

#### 5. Lưu ý và lỗi sai thường gặp

* **Copy sai đường dẫn file JAR:** Đường dẫn copy trong `COPY` không khớp với vị trí lưu trữ file JAR của artifacts dẫn đến lỗi build image thất bại.
* **Nhầm lẫn phiên bản JRE:** Chọn JRE 17 cho ứng dụng yêu cầu JRE 21 gây ra lỗi `UnsupportedClassVersionError` khi chạy container.

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Lầm tưởng Dockerfile tối ưu cho CI/CD vẫn phải copy toàn bộ mã nguồn thô Java vào rồi chạy lệnh `./gradlew build` bên trong Dockerfile.
* **Đính chính:** Vì quá trình biên dịch (compile/build JAR) đã được hoàn tất trước đó ở stage build của CI, Dockerfile lúc này chỉ làm nhiệm vụ đóng gói file JAR đã có sẵn. Việc này giúp tiết kiệm thời gian build image rất nhiều.

---

### LESSON 03: Tự động hóa Build Docker Image trong GitLab CI

#### 1. Mục tiêu bài học

* **Khai báo** thành công stage build image trong file cấu hình `.gitlab-ci.yml`.
* **Sử dụng** cơ chế phụ thuộc `needs` hoặc `dependencies` để lấy file JAR artifacts sang job build image.
* **Áp dụng** tag phiên bản tự động bằng biến môi trường.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** Dockerfile và file `.gitlab-ci.yml` đã được viết riêng lẻ.
* **Vấn đề:** Cần kết hợp hai thành phần này lại để quy trình đóng gói diễn ra hoàn toàn tự động ngay khi lập trình viên thực hiện commit code lên GitLab.

#### 3. Nội dung trọng tâm
* **Stage build Docker Image:** Khai báo stage tiếp theo trong chuỗi pipeline chạy sau stage compile/build JAR.
* **Tagging Image:** Sử dụng biến môi trường định sẵn của commit hash như `$CI_COMMIT_SHORT_SHA` để tự động gán nhãn phiên bản cho image, tăng tính vết (Traceability).

#### 4. Demo và thực hành

* **Mục tiêu demo:** Viết pipeline hoàn chỉnh gồm 2 stage: build JAR Spring Boot và build Docker Image kế thừa file JAR.
* **Cấu hình hệ thống (`user-service/.gitlab-ci.yml`):**
```yaml
image: eclipse-temurin:17-jdk-alpine

stages:
  - build_jar
  - build_image

variables:
  IMAGE_NAME: "user-service"

build_jar_job:
  stage: build_jar
  tags:
    - quickbite
  script:
    - chmod +x ./gradlew
    - ./gradlew bootJar
  artifacts:
    paths:
      - build/libs/*.jar
    expire_in: 1 hour

build_docker_image_job:
  stage: build_image
  image: docker:latest
  services:
    - docker:dind
  tags:
    - quickbite
  dependencies:
    - build_jar_job # Tải artifacts từ job trước về workspace
  script:
    - docker build -t ${IMAGE_NAME}:${CI_COMMIT_SHORT_SHA} .
```
* **Output mong đợi:** Pipeline chạy thành công qua cả 2 giai đoạn. Job thứ hai tự động tải file JAR về workspace và chạy lệnh `docker build` tạo ra image thành công.

#### 5. Lưu ý và lỗi sai thường gặp

* **Quên cấu hình dependencies/needs:** Khiến job build image khởi chạy mà không có file JAR artifacts trong thư mục làm việc, dẫn đến lệnh `COPY` trong Dockerfile báo lỗi không tìm thấy file.
* **Không chỉ định Runner phù hợp cho job dind:** Cần đảm bảo job build image chạy trên Runner có cấu hình Docker-in-Docker.

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Cho rằng mọi file sinh ra ở job thứ nhất mặc định luôn xuất hiện ở job thứ hai mà không cần khai báo thuộc tính `dependencies`.
* **Đính chính:** Môi trường chạy của từng job là hoàn toàn cô lập. Mọi dữ liệu chuyển giao bắt buộc phải đi qua cơ chế `artifacts` và được job sau khai báo kế thừa thông qua `dependencies` hoặc `needs`.

---

### LESSON 04: Cấu hình Xác thực (Authentication) Docker Registry trong CI/CD

#### 1. Mục tiêu bài học

* **Giải thích** được cơ chế hoạt động và vai trò của Docker Registry (GitLab Container Registry).
* **Ứng dụng** an toàn các biến ẩn bảo mật (Masked Variables) để bảo vệ tài khoản đăng nhập.
* **Thực hiện** lệnh đăng nhập an toàn vào registry trong pipeline.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** Image được build thành công trên máy Runner nhưng chưa thể gửi đi đâu.
* **Vấn đề:** Để lưu trữ tập trung, image cần được đẩy lên Registry. Tuy nhiên, lệnh `docker login` yêu cầu thông tin xác thực. Nếu ta hardcode tài khoản mật khẩu trực tiếp vào file cấu hình thì sẽ vi phạm nghiêm trọng quy chuẩn an toàn thông tin doanh nghiệp.

#### 3. Nội dung trọng tâm

* **GitLab Container Registry:** Kho lưu trữ Docker Image tích hợp sẵn đi kèm với từng dự án GitLab.
* **Predefined Security Variables:** GitLab cung cấp sẵn các biến môi trường tạm thời cho mỗi lượt chạy pipeline (`$CI_REGISTRY_USER`, `$CI_REGISTRY_PASSWORD`, `$CI_REGISTRY`) giúp job đăng nhập trực tiếp mà không cần cấu hình tài khoản cá nhân.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Thêm bước đăng nhập vào Registry tích hợp trong job build image.
* **Cấu hình hệ thống (`user-service/.gitlab-ci.yml`):**
```yaml
build_docker_image_job:
  stage: build_image
  image: docker:latest
  services:
    - docker:dind
  tags:
    - quickbite
  dependencies:
    - build_jar_job
  script:
    # Sử dụng các biến ẩn định sẵn để đăng nhập an toàn
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA .
```
* **Output mong đợi:** Log của pipeline hiển thị quá trình đăng nhập thành công (`Login Succeeded`) mà không làm hiển thị thông tin tài khoản mật khẩu thật của lập trình viên.

#### 5. Lưu ý và lỗi sai thường gặp

* **Ghi đè hoặc xóa nhầm biến hệ thống:** Vô tình khai báo biến trùng tên trong mục cài đặt làm mất giá trị biến mặc định của GitLab.
* **Container Registry bị tắt ở dự án:** Lỗi đăng nhập thất bại do tính năng lưu trữ image chưa được bật trong phần cấu hình dự án của GitLab.

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Lầm tưởng phải tự tạo ra tài khoản deploy token và đưa vào file cấu hình thì mới có quyền đăng nhập registry.
* **Đính chính:** GitLab CI tự động cấp phát một token đăng nhập tạm thời thông qua biến `$CI_REGISTRY_PASSWORD` chỉ có hiệu lực trong suốt thời gian chạy của chính job đó, đảm bảo an toàn tuyệt đối.

---

### LESSON 05: Đẩy Image lên Registry và Phân tách Pipeline tổng thể

#### 1. Mục tiêu bài học

* **Biên soạn** câu lệnh thực hiện đẩy (push) image lên registry trực tuyến thành công.
* **Phân tích** sơ đồ tổng thể quy trình vận hành tự động hóa (luồng end-to-end) của một microservice.
* **Đánh giá** kết quả đóng gói trên giao diện Web UI của GitLab.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** Image đã được build và đăng nhập registry thành công.
* **Vấn đề:** Image hiện tại vẫn đang nằm cục bộ trong container dind tạm thời của Runner và sẽ bị xóa khi kết thúc job. Cần đẩy image này lên kho lưu trữ trực tuyến để bất kỳ máy chủ nào khác (như VPS Production) cũng có thể kéo về sử dụng.

#### 3. Nội dung trọng tâm

* **Lệnh `docker push`:** Lệnh đồng bộ hóa Docker Image từ môi trường cục bộ lên Registry trực tuyến.
* **Luồng chạy tổng thể:** Push code -> Chạy job build JAR -> Đóng gói image -> Đăng nhập -> Push Image -> Quản lý phiên bản trên GitLab Web.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Hoàn thiện pipeline CI đóng gói và push image hoàn chỉnh cho `user-service`.
* **Cấu hình hệ thống (`user-service/.gitlab-ci.yml`):**
```yaml
image: eclipse-temurin:17-jdk-alpine

stages:
  - build_jar
  - build_image

variables:
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA

build_jar_job:
  stage: build_jar
  tags:
    - quickbite
  script:
    - chmod +x ./gradlew
    - ./gradlew bootJar
  artifacts:
    paths:
      - build/libs/*.jar
    expire_in: 1 hour

build_docker_image_job:
  stage: build_image
  image: docker:latest
  services:
    - docker:dind
  tags:
    - quickbite
  dependencies:
    - build_jar_job
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $IMAGE_TAG .
    - docker push $IMAGE_TAG
```
* **Output mong đợi:** Pipeline chạy thành công qua cả 2 stage. Trên giao diện Web của dự án GitLab (mục Deploy -> Container Registry), xuất hiện một Docker Image mới đi kèm tag tương ứng với mã hash commit.

#### 5. Lưu ý và lỗi sai thường gặp

* **Quên gắn thẻ tag của Registry:** Khai báo sai tên image (không bắt đầu bằng địa chỉ Registry như `$CI_REGISTRY_IMAGE`) khiến lệnh push báo lỗi không được phân quyền lưu trữ.
* **Dung lượng image quá lớn:** Không tối ưu Dockerfile khiến quá trình push diễn ra cực kỳ lâu và dễ bị ngắt kết nối giữa chừng do timeout.

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Nghĩ rằng chỉ cần chạy lệnh `docker build` là đủ, image tự động bay lên trời mà không cần chạy lệnh `docker push`.
* **Đính chính:** Image được build xong chỉ nằm ở bộ nhớ đệm cục bộ của container dind. Lệnh `docker push` là bắt buộc để tải image lên kho lưu trữ internet.
# SESSION 10

## TRIỂN KHAI HỆ THỐNG LÊN VPS (VIRTUAL PRIVATE SERVER)

---

### LESSON 01: Giới thiệu VPS và thiết lập kết nối cơ bản qua Bitvise Client

#### 1. Mục tiêu bài học

* **Phân biệt** được mô hình máy chủ ảo VPS với các dịch vụ điện toán đám mây (Cloud Services) và mô hình triển khai truyền thống.
* **Xác định** tiêu chuẩn lựa chọn cấu hình phần cứng phù hợp cho việc vận hành hệ thống Microservices.
* **Thực hiện kết nối** và quản lý thông tin phiên làm việc trên VPS bằng Bitvise SSH Client.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** Dịch chuyển môi trường triển khai từ máy phát triển cục bộ (Local Development) sang hạ tầng máy chủ ảo độc lập (Production Cloud Infrastructure).
* **Vấn đề:** Để đưa ứng dụng QuickBite tiếp cận người dùng thực tế, hệ thống không thể chạy mãi trên máy tính cá nhân của lập trình viên. Việc thiết lập một máy chủ Linux độc lập (headless server) là bắt buộc. Tuy nhiên, việc ghi nhớ thủ công thông tin kết nối, địa chỉ IP và mật khẩu root của VPS để đăng nhập liên tục qua terminal rất bất tiện và dễ xảy ra sai sót trong quá trình thao tác quản trị.

#### 3. Nội dung trọng tâm

* **Khái niệm VPS:** Bản chất của máy chủ ảo được phân tách bằng công nghệ ảo hóa từ một máy chủ vật lý, chạy hệ điều hành độc lập (ở đây sử dụng Ubuntu Server LTS) và quản trị hoàn toàn qua giao diện dòng lệnh (CLI).
* **So sánh hạ tầng:** So sánh chi tiết giữa VPS truyền thống với Cloud Services (như AWS EC2, Google Compute Engine) và dịch vụ Container Serverless/PaaS về chi phí, khả năng kiểm soát hệ thống và mức độ phức tạp trong vận hành.
* **Nguyên tắc chọn cấu hình:** Đánh giá các thông số CPU, RAM, dung lượng SSD dựa trên số lượng container của hệ thống QuickBite nhằm tối ưu hóa chi phí và đảm bảo hiệu năng.
* **Công cụ quản trị Bitvise SSH Client:** Giới thiệu tính năng lưu trữ thông tin phiên (Session Profiles), quản lý khóa bảo mật, tích hợp SFTP trực quan để hỗ trợ thao tác vận hành từ máy local chạy hệ điều hành Windows.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Hướng dẫn mua VPS theo tháng, thực hiện kết nối cơ bản qua Terminal mặc định và thiết lập lưu phiên đăng nhập trên Bitvise SSH Client.
* **Các bước thực hiện:**
  1. Hướng dẫn các bước lựa chọn gói dịch vụ VPS (khuyến nghị hệ điều hành Ubuntu Server 22.04 LTS, tối thiểu 2 vCPU và 2GB RAM để vận hành cụm Microservices).
  2. Sử dụng Terminal/PowerShell tại máy local để thực hiện kết nối SSH đầu tiên bằng tài khoản root:
     ```bash
     ssh root@vps_public_ip
     ```
  3. Cập nhật cơ sở dữ liệu gói phần mềm và nâng cấp hệ điều hành VPS lên phiên bản mới nhất:
     ```bash
     sudo apt update && sudo apt upgrade -y
     ```
  4. Cấu hình Bitvise SSH Client trên máy local: Nhập `Host` (IP của VPS), `Port` (22), `Username` (root), thiết lập phương thức xác thực bằng mật khẩu để đăng nhập ban đầu, sau đó lưu lại cấu hình dưới dạng tệp tin profile (.bjp) để kết nối nhanh trong các phiên làm việc sau mà không cần nhập lại mật khẩu.

* **Kết quả mong đợi:** Người học làm chủ giao diện CLI của VPS, kết nối ổn định thông qua Bitvise SSH Client và chuẩn bị sẵn sàng hệ điều hành sạch cho các bước tiếp theo.

#### 5. Điểm cần nhấn mạnh
* Việc nâng cấp hệ thống bằng lệnh `apt upgrade` ngay sau khi nhận VPS là bước bắt buộc để vá các lỗ hổng bảo mật của phiên bản phân phối cũ từ nhà cung cấp dịch vụ.
* Bitvise SSH Client hỗ trợ mở đồng thời cửa sổ Terminal (Terminal Console) và trình quản lý tệp tin đồ họa (SFTP window), giúp giảm tải thao tác lệnh khi quản trị tệp tin trên máy chủ.

#### 6. Hiểu lầm thường gặp
* **Hiểu sai:** Nghĩ rằng VPS và Cloud Server (như AWS EC2) là một hoặc chỉ khác nhau ở tên gọi.
* **Đính chính:** VPS thường có tài nguyên cố định trên một node vật lý và khó mở rộng linh hoạt. Cloud Server hoạt động trên nền tảng điện toán đám mây phân tán với khả năng tự động co giãn (Auto Scaling), tính sẵn sàng cao (High Availability) và thanh toán linh hoạt theo giờ/phút sử dụng thực tế.

---

### LESSON 02: Tạo User mới và bảo mật VPS bằng cơ chế khóa SSH (SSH Hardening)

#### 1. Mục tiêu bài học

* **Giải thích** cơ chế tấn công dò mã (Brute-force) nhắm vào cổng SSH mặc định của máy chủ.
* **Thiết lập** tài khoản người dùng quản trị mới với quyền hạn hạn chế và phân quyền thực thi quản trị tối cao qua `sudo`.
* **Cấu hình tối ưu bảo mật SSH (SSH Hardening)** để vô hiệu hóa hoàn toàn phương thức xác thực bằng mật khẩu và tài khoản root.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** VPS đã được khởi tạo và kết nối thành công với tài khoản root sử dụng xác thực mật khẩu qua cổng 22.
* **Vấn đề:** Khi một máy chủ có IP công khai (Public IP) xuất hiện trên Internet, các hệ thống botnet tự động sẽ liên tục quét cổng 22 và thực hiện hàng triệu lượt thử đăng nhập bằng mật khẩu (Brute-force) mỗi ngày. Nếu quản trị viên giữ nguyên tài khoản root mặc định và cho phép đăng nhập bằng mật khẩu, máy chủ QuickBite sẽ đối mặt với nguy cơ bị xâm nhập rất cao.

#### 3. Nội dung trọng tâm

* **Nguy cơ Brute-force mật khẩu:** Cơ chế hoạt động của các cuộc tấn công dò quét tự động và lý do tại sao phương thức xác thực bằng mật khẩu thông thường không đủ an toàn cho môi trường Production.
* **Nguyên tắc đặc quyền tối thiểu (Least Privilege):** Hạn chế tối đa việc sử dụng tài khoản root cho các hoạt động vận hành thường nhật. Thay vào đó, tạo một người dùng quản trị riêng biệt và chỉ nâng quyền khi cần thiết.
* **Cơ chế xác thực khóa SSH (Asymmetric Cryptography):** Sử dụng cặp khóa mã hóa bất đối xứng gồm Private Key (nằm trên máy local cá nhân của quản trị viên) và Public Key (đặt trong thư mục được cấu hình trên VPS) để xác thực danh tính mà không cần truyền mật khẩu qua mạng.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Tạo tài khoản user mới, thiết lập xác thực bằng SSH Key và khóa cấu hình đăng nhập root cùng mật khẩu truyền thống.
* **Các bước thực hiện:**
  1. Trên VPS, tạo người dùng mới (ví dụ đặt tên là `deployer`) và gán vào nhóm quản trị `sudo`:
     ```bash
     sudo adduser deployer
     sudo usermod -aG sudo deployer
     ```
  2. Tại máy local (PowerShell/Terminal), sinh cặp khóa SSH bảo mật cao sử dụng thuật toán Ed25519:
     ```bash
     ssh-keygen -t ed25519 -C "deployer_vps_key"
     ```
     *(Lưu ý: Tệp `id_ed25519` là khóa bí mật cá nhân, tệp `id_ed25519.pub` chứa khóa công khai).*
  3. Đẩy khóa công khai (Public Key) từ máy local lên VPS và lưu vào danh sách xác thực của user `deployer`:
     ```bash
     ssh-copy-id -i ~/.ssh/id_ed25519.pub deployer@vps_public_ip
     ```
  4. Tiến hành kết nối thử từ máy local bằng user `deployer` với SSH Key (không nhập mật khẩu) để kiểm tra tính chính xác.
  5. Thực hiện bảo mật dịch vụ SSH (SSH Hardening) bằng cách chỉnh sửa tệp cấu hình daemon SSH trên VPS:
     ```bash
     sudo nano /etc/ssh/sshd_config
     ```
     Tìm và sửa đổi các chỉ thị cấu hình sau:
     ```text
     PermitRootLogin no
     PasswordAuthentication no
     PubkeyAuthentication yes
     ```
  6. Khởi động lại dịch vụ SSH daemon để áp dụng cấu hình mới:
     ```bash
     sudo systemctl restart ssh
     ```

* **Kết quả mong đợi:** Không thể sử dụng tài khoản `root` để SSH vào VPS. Thử kết nối SSH bằng mật khẩu thông thường sẽ bị hệ thống từ chối ngay lập tức. Chỉ duy nhất máy local nắm giữ Private Key tương ứng mới có quyền truy cập VPS dưới tên người dùng `deployer`.

#### 5. Điểm cần nhấn mạnh
* Tuyệt đối không được tắt phiên kết nối SSH hiện tại trước khi kiểm tra thành công kết nối SSH mới bằng SSH Key trên một cửa sổ Terminal/Bitvise khác. Sai sót trong cấu hình có thể dẫn đến việc bị khóa quyền truy cập máy chủ vĩnh viễn.
* File Private Key sinh ra trên máy local phải được lưu trữ an toàn, không chia sẻ cho bất kỳ ai và không được đưa lên kho chứa mã nguồn công khai (Git Repository).

#### 6. Hiểu lầm thường gặp
* **Hiểu sai:** Cho rằng việc đặt mật khẩu tài khoản cực kỳ phức tạp (dài, nhiều ký tự đặc biệt) là đã đủ an toàn trước tấn công Brute-force mà không cần cấu hình SSH Key.
* **Đính chính:** Tấn công Brute-force có thể làm cạn kiệt tài nguyên mạng và băng thông của VPS do số lượng yêu cầu kết nối quá lớn. Tắt hoàn toàn tính năng xác thực mật khẩu giúp triệt tiêu nguy cơ này ngay tại bước bắt tay kết nối của giao thức SSH.

---

### LESSON 03: Thiết lập Tường lửa UFW và Cài đặt Docker trên Ubuntu Server

#### 1. Mục tiêu bài học

* **Cài đặt sạch (Clean Install)** Docker Engine và Docker Compose Plugin chính thức từ kho lưu trữ của Docker trên môi trường dòng lệnh Linux.
* **Cấu hình phân quyền** cho phép người dùng chạy các lệnh Docker CLI trực tiếp mà không cần tiền tố `sudo`.
* **Thiết lập tường lửa UFW** nhằm kiểm soát chặt chẽ toàn bộ lưu lượng mạng đi vào hệ thống, chỉ mở các cổng dịch vụ tiêu chuẩn.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** VPS đã được bảo mật kết nối SSH bằng tài khoản `deployer`. Hệ điều hành chưa được cài đặt Docker để chạy container và chưa có hệ thống lọc lưu lượng mạng tầng Host.
* **Vấn đề:** Để chạy các dịch vụ Spring Boot của QuickBite dưới dạng container, ta cần cài đặt môi trường Docker Engine ổn định và đúng chuẩn LTS. Đồng thời, việc chạy các lệnh quản trị Docker với tiền tố `sudo` liên tục tạo ra các tệp tin có quyền root không mong muốn và tiềm ẩn rủi ro bảo mật hệ thống. Bên bên đó, hệ thống cần được bao bọc bởi một lớp tường lửa ở cấp độ máy chủ để ngăn chặn mọi truy cập trái phép từ Internet vào các cổng nội bộ.

#### 3. Nội dung trọng tâm

* **Cài đặt Docker chuẩn Enterprise:** Tránh sử dụng các gói cài đặt mặc định của Ubuntu (`docker.io` hoặc `snap`) vốn thường cũ và thiếu ổn định. Thay vào đó, thiết lập liên kết trực tiếp tới Docker Repository chính thức để nhận các bản vá bảo mật mới nhất.
* **Cơ chế Docker Socket & Docker Group:** Hiểu lý do tại sao lệnh Docker yêu cầu quyền root và cách thức nhóm bảo mật `docker` giúp xử lý vấn đề phân quyền.
* **Tường lửa UFW (Uncomplicated Firewall):** Nguyên lý hoạt động lọc gói tin tầng host và cách cấu hình chính sách mặc định chặn toàn bộ kết nối đi vào (Default Deny), chỉ mở các cổng kết nối dịch vụ thiết yếu (22 cho SSH quản trị, 80 cho HTTP và 443 cho HTTPS sau này).

#### 4. Demo và thực hành

* **Mục tiêu demo:** Cài đặt Docker Engine và Docker Compose, thiết lập phân quyền user nhóm docker, cấu hình tường lửa UFW chỉ cho phép các cổng 22, 80, 443.
* **Các bước thực hiện:**
  1. Gỡ bỏ hoàn toàn các gói cài đặt cũ hoặc không chính thức (nếu có):
     ```bash
     for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
     ```
  2. Nạp khóa GPG chính thức và thêm Docker Repository vào cơ sở dữ liệu `apt`:
     ```bash
     sudo apt-get update
     sudo apt-get install ca-certificates curl gnupg -y
     sudo install -m 0755 -d /etc/apt/keyrings
     curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
     sudo chmod a+r /etc/apt/keyrings/docker.gpg
     echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
     ```
  3. Cài đặt các gói thành phần cốt lõi của Docker:
     ```bash
     sudo apt-get update
     sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
     ```
  4. Tạo nhóm người dùng `docker` và đưa tài khoản `deployer` vào nhóm để chạy lệnh không cần `sudo`:
     ```bash
     sudo usermod -aG docker deployer
     newgrp docker
     ```
  5. Cấu hình tường lửa UFW bảo vệ máy chủ:
     ```bash
     sudo ufw default deny incoming  # Chặn mọi kết nối đi vào mặc định
     sudo ufw default allow outgoing # Cho phép máy chủ kết nối ra Internet tự do
     sudo ufw allow 22/tcp           # Mở cổng kết nối SSH quản trị
     sudo ufw allow 80/tcp           # Mở cổng dịch vụ HTTP
     sudo ufw allow 443/tcp          # Mở cổng dịch vụ HTTPS
     sudo ufw enable                 # Kích hoạt UFW hoạt động
     ```
  6. Kiểm tra lại trạng thái hoạt động của các dịch vụ:
     ```bash
     docker version
     sudo ufw status verbose
     ```

* **Kết quả mong đợi:** Lệnh `docker version` chạy thành công bằng user `deployer` mà không báo lỗi phân quyền socket. Tường lửa UFW hiển thị trạng thái `active` với danh sách các cổng được mở duy nhất là 22, 80, 443.

#### 5. Điểm cần nhấn mạnh
* Phải thực hiện mở cổng SSH bằng lệnh `sudo ufw allow 22/tcp` **TRƯỚC KHI** chạy lệnh kích hoạt tường lửa `sudo ufw enable`. Việc bỏ qua bước này sẽ dẫn đến việc kết nối SSH hiện tại bị ngắt và người dùng không thể kết nối lại vào máy chủ.
* Lệnh `newgrp docker` giúp cập nhật quyền hạn nhóm cho session terminal hiện tại ngay lập tức mà không cần thực hiện logout rồi login lại.

#### 6. Hiểu lầm thường gặp
* **Hiểu sai:** Nghĩ rằng tường lửa UFW chặn cổng 5432 (PostgreSQL) sẽ làm cho ứng dụng Spring Boot chạy trong container Docker không kết nối được tới database PostgreSQL chạy trong container khác.
* **Đính chính:** Các container Docker giao tiếp nội bộ thông qua card mạng ảo `bridge network` được quản lý bởi Docker Engine. Tường lửa UFW cài đặt trên hệ điều hành host chỉ lọc lưu lượng mạng đi qua các card mạng vật lý kết nối ra Internet. Do đó, giao tiếp nội bộ giữa các container hoàn toàn không bị ảnh hưởng.

---

### LESSON 04: Đồng bộ cấu hình qua Git bằng SSH Key và vận hành cụm Microservices

#### 1. Mục tiêu bài học

* **Giải thích** mối quan hệ và cơ chế xác thực giữa Git/GitHub với các cặp khóa SSH Key bảo mật.
* **Phân biệt** rõ ràng vai trò của hai loại khóa SSH Key hoạt động trong quy trình triển khai: SSH Key của máy local truy cập VPS và SSH Key của VPS truy cập Git Repository.
* **Thao tác** thiết lập cấu hình SSH Key trên tài khoản Git (GitHub/GitLab) và clone các dự án private an toàn về máy chủ.
* **Vận hành và giám sát** cụm container Microservices của QuickBite sử dụng tệp tin cấu hình môi trường `.env` chuẩn Production.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** VPS đã được bảo mật bằng tường lửa, cài đặt sẵn Docker và phân quyền user `deployer`.
* **Vấn đề:** Để khởi chạy hệ thống QuickBite, ta cần nạp các mã nguồn cấu hình triển khai bao gồm tệp `docker-compose.yml`, các mã nguồn backend để build nếu cần, và cấu hình môi trường `.env`. Việc truyền tải thủ công các tệp tin cấu hình này từ máy local lên VPS qua SCP là phản khuôn mẫu trong DevOps chuyên nghiệp, gây khó khăn cho việc quản lý phiên bản và cập nhật mã nguồn. Giải pháp tối ưu là sử dụng hệ thống quản lý mã nguồn Git để đồng bộ trực tiếp từ Repository về VPS. Tuy nhiên, các Repository lưu trữ mã nguồn của doanh nghiệp thường ở dạng riêng tư (Private Repository), yêu cầu cơ chế xác thực an toàn bằng khóa SSH Key được thiết lập trên VPS.

#### 3. Nội dung trọng tâm

* **Git và cơ chế xác thực qua SSH:** Sử dụng giao thức SSH để xác thực tài khoản Git mà không cần điền username/password khi thao tác với kho lưu trữ mã nguồn từ máy chủ VPS CLI.
* **Phân biệt rõ ràng vai trò của 2 cặp khóa SSH:**
  1. **SSH Key 1 (Local -> VPS):** Sinh ra ở máy cá nhân (Local). Khóa Public được đặt trên VPS để cho phép quản trị viên đăng nhập vào quản trị hệ thống.
  2. **SSH Key 2 (VPS -> GitHub/GitLab):** Sinh ra trực tiếp trên máy chủ VPS. Khóa Public được cấu hình trên GitHub/GitLab (dưới dạng Deploy Key của repository hoặc SSH Key cá nhân) để cho phép VPS có quyền đọc và clone mã nguồn về.
* **Ứng dụng cấu hình môi trường Production:** Nạp các tham số bảo mật (mật khẩu database, token bí mật, đường dẫn endpoint) qua tệp `.env` đã thiết kế ở Session 08 để tách biệt hoàn toàn mã nguồn triển khai và thông số cấu hình nhạy cảm.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Cấu hình SSH Key trên VPS để kết nối GitHub, clone các dự án `quickbite-base`, `quickbite-user`, `quickbite-restaurant`, thiết lập tệp `.env` và khởi chạy toàn bộ cụm Microservices bằng Docker Compose.
* **Các bước thực hiện:**
  1. Trên VPS (đăng nhập bằng user `deployer`), sinh cặp khóa SSH mới dùng để kết nối với kho Git:
     ```bash
     ssh-keygen -t ed25519 -C "vps_to_github_key"
     ```
  2. Đọc và sao chép nội dung của khóa công khai vừa tạo:
     ```bash
     cat ~/.ssh/id_ed25519.pub
     ```
  3. Truy cập vào giao diện web của GitHub/GitLab, thêm nội dung khóa công khai này vào mục **Deploy Keys** của các repository dự án (`quickbite-base`, `quickbite-user`, `quickbite-restaurant`) hoặc mục **SSH Keys** của tài khoản nhà phát triển.
  4. Thực hiện kiểm tra kết nối SSH giữa VPS và GitHub:
     ```bash
     ssh -T git@github.com
     ```
  5. Clone các dự án về thư mục làm việc trên VPS:
     ```bash
     mkdir -p ~/projects && cd ~/projects
     git clone git@github.com:your_username/quickbite-base.git
     git clone git@github.com:your_username/quickbite-user.git
     git clone git@github.com:your_username/quickbite-restaurant.git
     ```
  6. Di chuyển vào thư mục hạ tầng triển khai (`quickbite-base` chứa file `docker-compose.yml`). Tạo và hiệu chỉnh tệp tin cấu hình môi trường `.env`:
     ```bash
     nano .env
     ```
     *(Khai báo các biến môi trường cấu hình cơ sở dữ liệu, cổng chạy, thông số xác thực bí mật tương tự như đã học ở Bài học 08 dành cho môi trường Production).*
  7. Khởi chạy toàn bộ cụm Microservices ở chế độ chạy ngầm (detached mode):
     ```bash
     docker compose up -d
     ```
  8. Kiểm tra trạng thái và giám sát hoạt động của hệ thống trên VPS:
     ```bash
     docker compose ps
     docker compose logs -f
     docker stats
     ```

* **Kết quả mong đợi:** Các dự án được clone thành công từ Git thông qua xác thực khóa SSH. Lệnh `docker compose ps` hiển thị toàn bộ các container dịch vụ ở trạng thái `Up (healthy)`.

#### 5. Điểm cần nhấn mạnh
* Việc phân biệt rõ vai trò của 2 cặp khóa SSH giúp sinh viên tránh được sai sót nghiêm trọng là sao chép Private Key của máy local lên VPS hoặc ngược lại.
* File `.env` chứa các thông tin tuyệt mật của môi trường chạy thực tế (Production), do đó tệp tin này bắt buộc phải được đưa vào danh sách `.gitignore` để không bao giờ bị đẩy ngược lại lên kho chứa mã nguồn công khai.

#### 6. Hiểu lầm thường gặp
* **Hiểu sai:** Nghĩ rằng chỉ cần cài đặt Git là có thể clone bất kỳ Repository nào về VPS mà không cần thiết lập SSH Key hay tài khoản.
* **Đính chính:** Với các Repository để ở chế độ riêng tư (Private), máy chủ VPS không có thông tin xác thực danh tính sẽ bị Git Server từ chối truy cập. Cơ chế SSH Key cung cấp một kênh xác thực bảo mật và tự động hóa cao mà không cần lưu trữ mật khẩu trực tiếp dưới dạng văn bản thuần túy (plain text) trên máy chủ.

---# SESSION 11

## REVERSE PROXY VỚI NGINX TRONG MÔI TRƯỜNG PRODUCTION

---

### LESSON 01: Khái niệm Reverse Proxy và vai trò của Nginx trong kiến trúc Microservices

#### 1. Mục tiêu bài học

* **Phân biệt** được cơ chế hoạt động giữa Forward Proxy và Reverse Proxy.
* **Giải thích** được tại sao Nginx đóng vai trò là chiếc "khiên bảo vệ" kiêm cổng định tuyến lưu lượng (Routing) tối thượng cho hệ thống QuickBite trước khi dòng tiền/dữ liệu đổ vào các service.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** STATE 3 — Production Cloud Infrastructure.
* **Vấn đề:** Hiện tại, 4 dịch vụ của QuickBite đang ẩn mình an toàn sau mạng nội bộ của VPS. Khách hàng sử dụng ứng dụng Mobile/Web ở ngoài Internet không thể kết nối tới hệ thống vì các cổng service đã bị khóa. Chúng ta không thể mở toang các cổng `8081`-`8084` ra ngoài vì vi phạm bảo mật. Hệ thống cần một thành phần duy nhất đứng ở "mặt tiền" (cổng 80), tiếp nhận mọi request của khách hàng và tự động phân phối (định tuyến) vào đúng vị trí của từng service bên trong.

#### 3. Nội dung trọng tâm

* **Forward Proxy vs Reverse Proxy:**
* *Forward Proxy:* Đứng trước Client, đại diện cho Client để đi ra Internet (ví dụ: VPN giúp ẩn danh IP người dùng).
* *Reverse Proxy:* Đứng trước Server, đại diện cho Server để đón nhận request từ Internet gửi vào. Client hoàn toàn không biết cấu trúc server phía sau.

* **Lợi ích khi đặt Nginx làm Reverse Proxy cho QuickBite:**
* *Single Entry Point:* Chỉ cần mở duy nhất cổng 80/443 trên IP của VPS.
* *Security:* Ẩn giấu hoàn toàn địa chỉ mạng, cổng chạy và kiến trúc microservices bên dưới.
* *Tải tĩnh (Static Content):* Nginx có thể trực tiếp gánh tải phần Frontend (HTML/CSS/JS) cực kỳ nhanh, giải phóng tài nguyên cho Spring Boot chỉ tập trung xử lý API nghiệp vụ.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Sơ đồ hóa kiến trúc dòng dữ liệu đi qua Nginx Reverse Proxy trên môi trường VPS thực tế.
* **Luồng di chuyển của dữ liệu (Traffic Flow):**
* Khách hàng gọi API lấy thông tin ví dụ: `http://vps_public_ip/api/v1/users`
* Request đập vào cổng 80 của VPS -> Nginx tiếp nhận.
* Nginx đọc cấu hình (Context Path `/api/v1/users`) -> Hiểu rằng cần đẩy request này vào mạng nội bộ Docker tới địa chỉ `http://user-service:8081/api/v1/users`.

---

### LESSON 02: Cấu hình Nginx định tuyến (Routing) dòng dữ liệu Microservices

#### 1. Mục tiêu bài học

* **Viết file cấu hình `nginx.conf**` để định tuyến chính xác dòng dữ liệu dựa trên đường dẫn Context Path `/api/v1/...`.
* **Tích hợp Nginx vào cụm Docker Compose** để tận dụng cơ chế Service Discovery, gọi các dịch vụ backend bằng tên container.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** STATE 3 — Production Cloud Infrastructure.
* **Vấn đề:** Tiến hành cấu hình thực tế cho Nginx trên VPS để kết nối toàn bộ hạ tầng 4 dịch vụ đơn lẻ thành một thể thống nhất, giúp Client bên ngoài có thể thực hiện trọn vẹn luồng nghiệp vụ: Đăng nhập -> Xem nhà hàng -> Tạo đơn hàng.

#### 3. Nội dung trọng tâm

* **Cấu trúc khối cấu hình Nginx (`server`, `location`):**
* Khối `server`: Khai báo cổng lắng nghe (`listen 80`) và tên miền/IP của máy chủ (`server_name`).
* Khối `location`: Khai báo quy tắc khớp chuỗi đường dẫn (Path Matching) và sử dụng chỉ thị `proxy_pass` để bẻ hướng luồng dữ liệu sang container tương ứng.

* **Xử lý HTTP Headers:** Khi Nginx chuyển tiếp request, nó cần nạp thêm các thông tin tiêu đề (`X-Real-IP`, `X-Forwarded-For`) để ứng dụng Spring Boot phía sau biết được IP thực sự của khách hàng là gì, thay vì chỉ thấy IP của Nginx.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Viết file cấu hình `quickbite.conf`, nhúng container Nginx vào file `docker-compose.yml` hạ tầng và kiểm chứng kết nối API End-to-End từ máy cá nhân qua VPS.
* **Cấu hình hệ thống (Tệp cấu hình Nginx `./quickbite-infra/nginx/quickbite.conf`):**

```nginx
server {
    listen 80;
    server_name _; # Nhận mọi request đổ vào IP của VPS

    # Cấu hình log để tiện debug
    access_log /var/log/nginx/quickbite_access.log;
    error_log /var/log/nginx/quickbite_error.log;

    # Khai báo các tham số header chung khi chuyển tiếp request
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    # 1. Định tuyến luồng User Service
    location /api/v1/users {
        proxy_pass http://user-service:8081;
    }

    # 2. Định tuyến luồng Restaurant Service
    location /api/v1/restaurants {
        proxy_pass http://restaurant-service:8082;
    }

    # 3. Định tuyến luồng Order Service
    location /api/v1/orders {
        proxy_pass http://order-service:8083;
    }

    # 4. Định tuyến luồng Notification Service
    location /api/v1/notifications {
        proxy_pass http://notification-service:8084;
    }

    # Cấu hình trang lỗi mặc định nếu gọi sai endpoint
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }
}
```

* **Cập nhật tệp `docker-compose.yml` (Bổ sung thêm mảnh ghép Nginx):**

```yaml
version: '3.8'

services:
  # ... 4 services (user, restaurant, order, notification) và quickbite-db giữ nguyên cấu hình mạng ...

  nginx-gateway:
    image: nginx:1.25-alpine
    container_name: quickbite-nginx-gateway
    ports:
      - "80:80" # Mở duy nhất cổng 80 của VPS nối vào cổng 80 container Nginx
    volumes:
      # Gắn file cấu hình từ VPS vào trong container Nginx
      - ./nginx/quickbite.conf:/etc/nginx/conf.d/default.conf
      # Gắn thư mục để lưu log ra ngoài VPS phục vụ việc xem log
      - ./logs/nginx:/var/log/nginx
    networks:
      - quickbite-net
    depends_on:
      - user-service
      - restaurant-service
      - order-service
      - notification-service

networks:
  quickbite-net:
    driver: bridge

```

* **Lệnh chạy thực tế và kiểm chứng (Từ máy cá nhân của Sinh viên):**
```bash
# Trên VPS: Khởi động lại cụm docker compose để nhận cấu hình Nginx mới
docker compose up -d --remove-orphans

# Từ máy cá nhân ở nhà: Dùng Postman hoặc cURL gọi thử API thông qua IP của VPS trên cổng 80
curl http://vps_public_ip/api/v1/restaurants
```

* **Kết quả mong đợi:** Khách hàng nhận về mã trạng thái `200 OK` cùng chuỗi dữ liệu JSON danh sách nhà hàng. Khi gõ lệnh `tail -f ./logs/nginx/quickbite_access.log` trên VPS, giảng viên sẽ thấy dòng log ghi nhận request vừa thực hiện thành công.

#### 5. Điểm cần nhấn mạnh cho giảng viên (Pedagogical Tips)

* **Sức mạnh của DNS Docker:** Hãy chỉ cho sinh viên thấy trong file cấu hình Nginx, chuỗi định tuyến được viết là `proxy_pass http://user-service:8081`. Nginx (vốn là một công cụ độc lập của bên thứ ba) có thể hiểu được chữ `user-service` là gì nhờ việc nó được đặt chung mạng `quickbite-net` với các service khác và thừa hưởng DNS nội bộ của Docker Engine.
* **Kiểm tra cú pháp cấu hình Nginx:** Hướng dẫn sinh viên thói quen: Mỗi lần sửa đổi file cấu hình `.conf` của Nginx, trước khi restart container, nên dùng câu lệnh `docker exec quickbite-nginx-gateway nginx -t` để hệ thống tự động kiểm tra xem có bị gõ thiếu dấu chấm phẩy (`;`) hay sai cú pháp ở dòng nào không. Điều này giúp tránh làm sập cổng Gateway đang chạy trên production.

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Nghĩ rằng mỗi lần sửa file `quickbite.conf` là phải chạy lệnh `docker compose restart nginx-gateway` làm gián đoạn cổng kết nối của người dùng.
* **Đính chính:** Nginx hỗ trợ cơ chế nạp lại cấu hình mà không cần dừng tiến trình (Zero-Downtime Hot Reload). Thay vì restart container, giảng viên hướng dẫn sinh viên gõ câu lệnh: `docker exec quickbite-nginx-gateway nginx -s reload`. Hệ thống sẽ lập tức cập nhật luật định tuyến mới mà không làm rớt bất kỳ request nào của khách hàng đang đặt đồ ăn.

---

*Giảng viên lưu ý: Hoàn thành Session 11, sinh viên đã chính thức thiết lập xong một hệ thống **Production-ready** thu nhỏ cho QuickBite trên môi trường Cloud thực tế. Sự kết hợp giữa tường lửa VPS, mạng cô lập Docker Network và cổng định tuyến duy nhất Nginx Reverse Proxy tạo nên một kiến trúc hạ tầng phòng thủ chiều sâu chuẩn chỉnh của DevOps.*

---

# SESSION 13

## GIÁM SÁT HỆ THỐNG (MONITORING) VỚI PROMETHEUS

---

### LESSON 01: Kiến trúc Prometheus và Mô hình Thu thập dữ liệu (Pull-based)

#### 1. Mục tiêu bài học

* **Giải thích** được thành phần kiến trúc của Prometheus và ưu/nhược điểm của mô hình thu thập chỉ số kiểu kéo (Pull-based Model) so với kiểu đẩy (Push-based Model).
* **Nắm rõ** cấu trúc định dạng dữ liệu chuỗi thời gian (Time-series Data) và cách thức hoạt động của công cụ quét chỉ số (Scraper).

#### 2. Bối cảnh hệ thống

* **Trạng thái:** STATE 3 — Production Cloud Infrastructure (Hệ thống QuickBite đã chạy trên VPS, có Nginx làm Gateway).
* **Vấn đề:** Ứng dụng `order-service` thỉnh thoảng bị treo vào những khung giờ cao điểm (11h30 - 12h30). Đội ngũ vận hành hoàn toàn "mù" thông tin về trạng thái phần cứng của VPS cũng như trạng thái sức khỏe nội bộ của các container. Hệ thống cần một giải pháp giám sát có khả năng tự động đi gom các chỉ số tài nguyên theo chu kỳ thời gian để làm cơ sở chẩn đoán lỗi.

#### 3. Nội dung trọng tâm

* **Mô hình Pull-based cốt lõi:** Thay vì bắt các service của QuickBite phải tốn tài nguyên tự gửi dữ liệu đi, Prometheus sẽ đứng đóng vai trò trung tâm, định kỳ chủ động gửi request HTTP tới các endpoint của các service để "kéo" (pull/scrape) dữ liệu metrics về.
* **Cấu trúc Time-series Database (TSDB):** Dữ liệu giám sát được lưu trữ dưới dạng chuỗi thời gian, định danh bằng tên chỉ số (metric name) và các cặp nhãn khóa-giá trị (labels key-value).
* *Ví dụ:* `http_requests_total{method="POST", handler="/api/v1/orders", status="201"}`

#### 4. Demo và thực hành

* **Mục tiêu demo:** Minh họa trực quan mô hình thu thập dữ liệu dạng Pull của Prometheus đối với hệ thống Microservices QuickBite.
* **Luồng dữ liệu (Data Flow):**
1. Các service (`user-service`, `order-service`,...) mở sẵn một cổng hiển thị dữ liệu thô.
2. Prometheus đọc file cấu hình `prometheus.yml`, lấy danh sách các địa chỉ đích (targets).
3. Đúng chu kỳ (ví dụ 15 giây), Prometheus gửi lệnh kéo dữ liệu và nạp vào kho lưu trữ TSDB nội bộ.

---

### LESSON 02: Cấu hình Prometheus Scrape Metrics từ Spring Boot Actuator

#### 1. Mục tiêu bài học

* **Cấu hình tích hợp** thư viện Spring Boot Actuator và Micrometer Prometheus Registry vào mã nguồn các dịch vụ QuickBite.
* **Viết tệp cấu hình `prometheus.yml**` hoàn chỉnh để khai báo các job quét chỉ số tự động trong mạng nội bộ Docker.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** STATE 3 — Production Cloud Infrastructure.
* **Vấn đề:** Để Prometheus có thể kéo được dữ liệu từ các dịch vụ Spring Boot, bản thân các dịch vụ này phải được cài đặt "cảm biến" để chuyển đổi các thông số kỹ thuật bên trong JVM (như dung lượng bộ nhớ Heap, số lượng Thread) thành định dạng văn bản thô (Plain text) mà Prometheus hiểu được.

#### 3. Nội dung trọng tâm

* **Spring Boot Actuator & Micrometer:** Actuator mở ra các cổng giám sát, còn Micrometer đóng vai trò như một bộ biên dịch (Adapter) chuyển các chỉ số đo đạc nội bộ của Java sang định dạng chuẩn của Prometheus.
* **Cấu hình bảo mật tầng mạng:** Các endpoint hiển thị chỉ số (`/actuator/prometheus`) chứa nhiều thông tin nhạy cảm về hệ thống, do đó chúng chỉ được mở trong mạng nội bộ Docker (`quickbite-net`) để duy nhất container Prometheus truy cập, tuyệt đối không cấu hình qua Nginx để lộ ra Internet.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Cấu hình mã nguồn một service mẫu, tạo file `prometheus.yml` và nhúng container Prometheus vào file `docker-compose.yml` hạ tầng trên VPS.
* **Cấu hình ứng dụng (`order-service/src/main/resources/application.yml`):**
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health, prometheus # Chỉ mở các endpoint cần thiết
  metrics:
    tags:
      application: ${spring.application.name} # Dán nhãn tên ứng dụng vào mọi metric
```

* **Tệp cấu hình của Prometheus (`./quickbite-infra/prometheus/prometheus.yml`):**
```yaml
global:
  scrape_interval: 15s # Định kỳ quét chỉ số mỗi 15 giây
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'quickbite-backend'
    metrics_path: '/actuator/prometheus'
    static_configs:
      # Gọi trực tiếp bằng tên container nhờ DNS nội bộ của Docker Network
      - targets: ['user-service:8081', 'restaurant-service:8082', 'order-service:8083', 'notification-service:8084']
```

* **Cập nhật tệp `docker-compose.yml` trên VPS:**
```yaml
services:
  # ... các dịch vụ backend và nginx giữ nguyên ...

  prometheus:
    image: prom/prometheus:v2.45.0
    container_name: quickbite-prometheus
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      # Gắn ổ đĩa volume để bảo toàn dữ liệu metrics khi container restart
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    networks:
      - quickbite-net
    # Không cần mở port ra ngoài máy host nếu chỉ dùng nội bộ với Grafana

volumes:
  prometheus-data:
    driver: local
```

* **Kết quả mong đợi:** Sau khi chạy `docker compose up -d`, Prometheus khởi chạy thành công. Giảng viên có thể hướng dẫn sinh viên dùng lệnh `docker exec -it quickbite-prometheus wget -qO- http://localhost:9090/api/v1/targets` để kiểm tra trạng thái các target đều báo `UP` (màu xanh).

---

# SESSION 14

## TẠO DASHBOARD VỚI GRAFANA

---

### LESSON 01: Kết nối Data Source và Ngôn ngữ truy vấn PromQL cơ bản

#### 1. Mục tiêu bài học

* **Thực hiện liên kết** Grafana với nguồn dữ liệu (Data Source) Prometheus an toàn trong môi trường mạng cô lập.
* **Sử dụng ngôn ngữ truy vấn PromQL** để lọc, tính toán tốc độ request và tỷ lệ lỗi của hệ thống QuickBite.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** STATE 3 — Production Cloud Infrastructure.
* **Vấn đề:** Dữ liệu đã được lưu trữ vào Prometheus, tuy nhiên giao diện mặc định của Prometheus rất thô sơ, chỉ hỗ trợ xem các biểu đồ đơn lẻ và không thể lưu thành các bảng điều khiển tập trung. Chúng ta cần đưa **Grafana** vào đóng vai trò là "màn hình hiển thị trung tâm" giúp trực quan hóa toàn bộ dữ liệu này.

#### 3. Nội dung trọng tâm

* **Kiến trúc liên kết Grafana - Prometheus:** Grafana đóng vai trò là Client gửi các câu lệnh truy vấn qua REST API tới Prometheus để lấy dữ liệu số liệu về vẽ biểu đồ.
* **Cú pháp ngôn ngữ PromQL (Prometheus Query Language):**
* *Lọc dữ liệu (Instant Vector):* `process_cpu_usage{application="order-service"}`
* *Hàm tính toán theo thời gian (Range Vector & Rate):* Sử dụng hàm `rate(...)` để tính toán tốc độ tăng trưởng của chỉ số trên mỗi giây.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Thêm Grafana vào hệ thống, cấu hình kết nối và thực hành viết 3 câu lệnh PromQL cốt lõi phục vụ giám sát Microservices.
* **Cập nhật tệp `docker-compose.yml` trên VPS:**
```yaml
services:
  # ... các dịch vụ trước giữ nguyên ...

  grafana:
    image: grafana/grafana:10.0.0
    container_name: quickbite-grafana
    ports:
      - "3000:3000" # Mở cổng 3000 để đội ngũ vận hành truy cập vào dashboard
    volumes:
      - grafana-data:/var/lib/grafana
    networks:
      - quickbite-net

volumes:
  prometheus-data:
  grafana-data:
```

* **Các bước thực hiện cấu hình giao diện:**
1. Truy cập `http://vps_public_ip:3000` (Tài khoản mặc định: `admin` / `admin`).
2. Đi tới *Connections -> Data Sources -> Add data source* -> Chọn **Prometheus**.
3. Tại ô URL, điền: `http://prometheus:9090` (Gọi trực tiếp bằng tên container nội bộ, rất bảo mật). Nhấn *Save & Test*.


* **Thực hành viết câu lệnh PromQL trong mục "Explore":**
* *Câu lệnh 1 (Giám sát RAM của Order Service):*
```text
jvm_memory_used_bytes{application="order-service", area="heap"}

```

* *Câu lệnh 2 (Tính số lượng Request/giây đổ vào Restaurant Service):*
```text
rate(http_server_requests_seconds_count{application="restaurant-service"}[5m])
```

* **Kết quả mong đợi:** Grafana trả về các đồ thị đường (Line Chart) mô tả chính xác lượng RAM trồi sụt hoặc lượng request biến động theo thời gian thực của hệ thống QuickBite.

---

### LESSON 02: Thiết kế Dashboard chuyên nghiệp và Thiết lập Cảnh báo (Alerting)

#### 1. Mục tiêu bài học

* **Xây dựng hoàn chỉnh một Dashboard** quản lý tài nguyên JVM cho Microservices bằng cách import các mẫu thiết kế chuẩn công nghiệp (Community Dashboards).
* **Cấu hình quy tắc cảnh báo (Alert Rules)** trên Grafana để tự động phát hiện khi hệ thống gặp sự cố (ví dụ: CPU quá tải hoặc Service bị sập).

#### 2. Bối cảnh hệ thống

* **Trạng thái:** STATE 3 — Production Cloud Infrastructure.
* **Vấn đề:** Lập trình viên hoặc kỹ sư vận hành không thể ngồi nhìn màn hình Grafana 24/7 để chờ xem khi nào lỗi xảy ra. Hệ thống cần được thiết lập một cơ chế tự động: Khi có bất kỳ service nào của QuickBite bị sập (Container chết), hoặc lượng RAM tiêu thụ vượt ngưỡng 90%, Grafana phải ngay lập tức đưa ra cảnh báo trực quan để đội ngũ kỹ thuật kịp thời ứng cứu.

#### 3. Nội dung trọng tâm

* **Tái sử dụng Dashboard có sẵn (Dashboard Import):** Thay vì tự tay cấu hình hàng trăm biểu đồ thủ công, DevOps tận dụng kho thư viện của Grafana Community với mã ID nổi tiếng **ID 4701** (JVM Micrometer Dashboard) để có ngay một màn hình giám sát chuyên nghiệp.
* **Luồng hoạt động của Grafana Alerting:** Đặt ra một ngưỡng giới hạn (Threshold) -> Định kỳ kiểm tra (Evaluation) -> Nếu vượt ngưỡng -> Chuyển trạng thái sang `Firing` (Kích hoạt cảnh báo).

#### 4. Demo và thực hành

* **Mục tiêu demo:** Import Dashboard giám sát JVM và thiết lập một quy tắc cảnh báo khi một dịch vụ bất kỳ của QuickBite bị sập (Trạng thái Target = 0).
* **Thao tác thực hiện từng bước (Step-by-step):**
1. **Import Màn hình giám sát:** Trên giao diện Grafana, nhấn nút *Dashboards* -> *New* -> *Import*. Nhập ID `4701` và nhấn *Load*. Chọn Data Source là `Prometheus` đã tạo ở bài trước -> Nhấn *Import*.
2. **Thiết lập Quy tắc Cảnh báo (Alert Rule):**
* Đi tới mục *Alerting* -> *Alert rules* -> *Create rule*.
* Đặt tên quy tắc: `QuickBite - Service Down Alert`.
* Tại ô câu lệnh PromQL (Mục A), nhập câu lệnh kiểm tra sự sống của container:

```text
up{job="quickbite-backend"} == 0
```

* Tại mục *Set a threshold* (Ngữ cảnh kích hoạt): Chọn giá trị cảnh báo khi biểu thức trên thỏa mãn (tức là có service trả về giá trị 0).
* Tại mục *Set evaluation behavior*: Cấu hình kiểm tra mỗi `1m` (1 phút), nếu lỗi kéo dài liên tục trong vòng `2m` thì chính thức phát tín hiệu báo động.

* **Kết quả mong đợi:** Sinh viên thu được một Dashboard hiển thị toàn diện các thông số từ CPU, bộ nhớ Heap, trạng thái của Garbage Collection (GC) cho đến số lượng Thread đang chạy của 4 dịch vụ QuickBite. Khi giảng viên làm thử demo chạy lệnh `docker stop order-service` trên VPS, sau 2 phút, quy tắc cảnh báo trên Grafana sẽ lập tức chuyển sang màu đỏ rực (Trạng thái: **Firing**), chỉ rõ đích danh `order-service` đang bị sập.

#### 5. Điểm cần nhấn mạnh cho giảng viên (Pedagogical Tips)

* **Tư duy Giám sát Chủ động (Proactive Monitoring):** Hãy nhấn mạnh cho sinh viên hiểu sự khác biệt lớn của một kỹ sư DevOps: Người làm thủ công đợi khách hàng gọi điện chửi bới mới biết hệ thống lỗi; người làm DevOps nhìn vào Dashboard và các tín hiệu Cảnh báo (Alert) để phát hiện và sửa lỗi từ lúc hệ thống mới chỉ có dấu hiệu quá tải, trước khi người dùng kịp nhận ra.
* **Tầm quan trọng của Volumes:** Hãy nhắc nhở sinh viên kiểm tra kỹ phần khai báo `volumes` cho Grafana (`grafana-data`) trong tệp `docker-compose.yml`. Nếu thiếu phần này, tất cả các Dashboard đã mất công cấu hình hoặc các Alert Rule đã tạo sẽ biến mất hoàn toàn nếu container Grafana bị tắt đi hoặc cập nhật phiên bản mới.

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Cho rằng Grafana Alerting sẽ tự động đi sửa lỗi (ví dụ tự bật lại container khi container bị sập).
* **Đính chính:** Grafana chỉ đóng vai trò là hệ thống **Phát hiện và Cảnh báo** (Observability & Notification). Nó giúp gửi thông tin (qua Email, Telegram, Slack, hoặc hiển thị lên màn hình) để thông báo cho kỹ sư vận hành biết chính xác vị trí lỗi, việc xử lý và khắc phục lỗi sau đó vẫn cần sự can thiệp của con người hoặc các công cụ tự động hóa hạ tầng ở tầng cao hơn.

---

*Giảng viên lưu ý: Kết thúc Session 14, sinh viên đã hoàn thành trọn vẹn chuỗi ma trận giám sát đỉnh cao của DevOps. Sự kết hợp giữa Prometheus (Bộ kho lưu trữ chỉ số chuỗi thời gian) và Grafana (Màn hình trực quan hóa và phát lệnh cảnh báo) giúp hệ thống microservices QuickBite đạt đến trạng thái "Trong suốt về mặt vận hành" (Full System Observability).*

---

# SESSION 16

## LOGGING TRONG SPRING BOOT MÔI TRƯỜNG PRODUCTION

---

### LESSON 01: Cấu hình Logback nâng cao và Chiến lược xoay vòng file log (Log Rotation)

#### 1. Mục tiêu bài học

* **Thiết lập cấu hình** tệp `logback-spring.xml` để phân tách luồng ghi log độc lập giữa bảng điều khiển (Console) và tệp tin (File).
* **Áp dụng chiến lược xoay vòng** (Log Rotation) dựa trên kích thước và thời gian để tối ưu dung lượng lưu trữ, tránh làm tràn ổ cứng VPS.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** STATE 3 — Production Cloud Infrastructure (Hệ thống QuickBite chạy trên VPS).
* **Vấn đề:** Mặc định, Spring Boot ghi toàn bộ nhật ký ra bàn điều khiển (Console). Khi chạy dưới dạng container bằng Docker, các dòng log này được Docker Daemon giữ lại trên ổ đĩa. Sau một vài tháng vận hành, lượng request đặt món tăng cao, file log của `order-service` có thể phình to lên đến hàng chục GB, làm cạn kiệt dung lượng đĩa cứng của VPS và kéo sập toàn bộ hệ thống. Chúng ta cần một cơ chế tự động cắt nhỏ file log và xóa bỏ các log quá cũ.

#### 3. Nội dung trọng tâm

* **Logback Framework:** Bộ thư viện ghi log mặc định và mạnh mẽ của Spring Boot, cho phép cấu hình linh hoạt thông qua file XML.
* **Appenders:** Các thành phần định hướng đầu ra của log. Trong production, hệ thống sử dụng kết hợp `ConsoleAppender` (cho Docker thu thập) và `RollingFileAppender` (lưu trữ vật lý an toàn trên VPS).
* **Policy xoay vòng (Rolling Policy):** Cơ chế quy định khi nào file log cũ được đóng gói (ví dụ: sang ngày mới hoặc khi file đạt kích thước 10MB) và giới hạn tổng dung lượng lưu trữ (ví dụ: tối đa giữ lại log của 30 ngày gần nhất).

#### 4. Demo và thực hành

* **Mục tiêu demo:** Cấu hình file `logback-spring.xml` cho `order-service` để tự động hóa việc chia nhỏ, nén file log cũ thành định dạng `.gz`.
* **Cấu hình hệ thống (`order-service/src/main/resources/logback-spring.xml`):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <property name="LOG_PATH" value="/app/logs" />
    <property name="LOG_FILE" value="order-service" />

    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="ROLLING_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_PATH}/${LOG_FILE}.log</file>
        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
            <layout class="ch.qos.logback.classic.PatternLayout">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
            </layout>
        </encoder>

        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_PATH}/archived/${LOG_FILE}-%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <maxHistory>30</maxHistory>
            <totalSizeCap>1GB</totalSizeCap>
        </rollingPolicy>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE" />
        <appender-ref ref="ROLLING_FILE" />
    </root>

    <logger name="org.hibernate.SQL" level="DEBUG" />
</configuration>
```

* **Kết quả mong đợi:** Khi ứng dụng hoạt động và ghi log, trong thư mục `/app/logs` sẽ xuất hiện file `order-service.log`. Khi thực hiện giả lập ghi dữ liệu lớn, hệ thống tự động tạo thư mục `archived/` chứa các file nén dạng `order-service-2026-06-02.0.log.gz`.

---

### LESSON 02: Chuẩn hóa Định dạng Log JSON cho Microservices

#### 1. Mục tiêu bài học

* **Chuyển đổi cấu trúc log** từ dạng văn bản tự do (Plain Text) sang định dạng cấu trúc **JSON** chuẩn hóa.
* **Giải thích** được tầm quan trọng của cấu trúc dữ liệu khóa-giá trị (Key-Value) trong việc giúp các bộ phân tích nhật ký tự động phân tách trường dữ liệu (Parsing).

#### 2. Bối cảnh hệ thống

* **Trạng thái:** STATE 3 — Production Cloud Infrastructure.
* **Vấn đề:** Định dạng dòng log truyền thống (`2026-06-02 INFO [main] c.q.OrderService - Order created successfully`) rất thân thiện với mắt người đọc, nhưng lại là "ác mộng" đối với các hệ thống phân tích tự động. Khi cần tìm kiếm các đơn hàng có `totalPrice > 500000`, máy tính phải quét chuỗi (Regex) cực kỳ chậm. Nếu chuẩn hóa dòng log thành một đối tượng JSON cấu trúc, các công cụ thu thập log có thể lập tức hiểu và đánh chỉ mục các trường dữ liệu một cách chính xác.

#### 3. Nội dung trọng tâm

* **Logstash Logback Encoder:** Thư viện mở rộng cho phép Logback tự động chuyển hóa cấu trúc LogEvent của Java thành một chuỗi JSON thuần túy.
* **Tính cấu trúc (Structured Logging):** Mọi dòng log xuất ra đều có chung các thuộc tính cơ bản như `@timestamp`, `level`, `thread`, `logger_name`, `message`. Lập trình viên có thể chèn thêm các thuộc tính tùy biến thông qua cấu hình `MDC` (Mapped Diagnostic Context).

#### 4. Demo và thực hành

* **Mục tiêu demo:** Cấu hình log định dạng JSON cho `user-service` phục vụ mục đích phân tích tự động ở Session 17.
* **Cấu hình hệ thống:**

* *Bổ sung thư viện (`user-service/build.gradle`):*
```groovy
implementation 'net.logstash.logback:logstash-logback-encoder:7.4'
```

* *Cập nhật Appender trong file `logback-spring.xml`:*

```xml
<appender name="JSON_CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
    <encoder class="net.logstash.logback.encoder.LogstashEncoder">
        <customFields>{"environment":"production","service":"user-service"}</customFields>
    </encoder>
</appender>

<root level="INFO">
    <appender-ref ref="JSON_CONSOLE" />
</root>

```

* **Output mong đợi từ hệ thống (Console Output):** Dòng log biến đổi hoàn toàn thành cấu trúc JSON một dòng duy nhất:
```json
{"@timestamp":"2026-06-02T07:27:00.123+07:00","@version":"1","message":"User login success: admin","logger_name":"com.quickbite.UserService","thread_name":"http-nio-8081-exec-1","level":"INFO","level_value":20000,"environment":"production","service":"user-service"}

```

#### 5. Điểm cần nhấn mạnh cho giảng viên (Pedagogical Tips)

* Nhấn mạnh với sinh viên: Log JSON sinh ra **không phải để cho con người đọc trực tiếp** bằng mắt trên terminal, mà là để làm nguyên liệu đầu vào chuẩn chỉnh cho các hệ thống quản lý log tập trung như EFK Stack xử lý.

---

# SESSION 17

## LOGGING TẬP TRUNG VỚI EFK STACK

---

### LESSON 01: Kiến trúc EFK Stack và Cơ chế thu thập log qua Filebeat

#### 1. Mục tiêu bài học

* **Phân tích** được vai trò và ranh giới trách nhiệm của từng thành phần trong bộ ba **EFK Stack**: Elasticsearch, Fluentd/Filebeat, và Kibana.
* **Cấu hình** thành công công cụ thu thập log siêu nhẹ **Filebeat** để tự động "lắng nghe" và đọc file log từ thư mục chung của các dịch vụ QuickBite trên VPS.

#### 2. Bối cảnh hệ thống

* **Trạng thái:** Chuyển dịch từ STATE 3 (Log phân tán cục bộ trên đĩa cứng container) sang STATE 4 (Centralized Logging Architecture).
* **Vấn đề:** Hệ thống QuickBite có 4 service chạy ẩn sau mạng nội bộ. Khi khách hàng khiếu nại về lỗi thanh toán đơn hàng, kỹ sư không thể liên tục SSH vào server, mò vào từng thư mục `/app/logs` của từng container để xem lỗi. Chúng ta cần một giải pháp thu thập tự động toàn bộ dữ liệu nhật ký này về một trung tâm lưu trữ tập trung để tra cứu dễ dàng.

#### 3. Nội dung trọng tâm

* **Mô hình kiến trúc EFK Stack:**
* **F (Filebeat/Fluentd):** Bộ thu gom dữ liệu (Data Shipper). Trong bài học này, ta dùng **Filebeat** — một Go-agent siêu nhẹ của Elastic, tốn cực ít tài nguyên RAM, đứng trực tiếp tại nơi sinh log để vận chuyển log đi.
* **E (Elasticsearch):** Trái tim của hệ thống, một công cụ tìm kiếm và phân tích phân tán mạnh mẽ (Search & Analytics Engine), chịu trách nhiệm lưu trữ dữ liệu log và đánh chỉ mục (indexing) để tìm kiếm full-text cực nhanh.
* **K (Kibana):** Giao diện quản trị đồ họa (UI), giúp người dùng dễ dàng truy vấn, lọc, tìm kiếm từ khóa và vẽ các biểu đồ phân tích từ log.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Minh họa trực quan luồng di chuyển cấu trúc của dòng nhật ký từ file log của QuickBite qua Filebeat đổ về Elasticsearch.
* **Luồng dữ liệu (Data Pipeline Flow):**
```text
[Spring Boot Apps] ──(Ghi Log)──► [Thư mục logs/ trên VPS]
                                          │
                                          ▼ (Cào dữ liệu)
                                    [ Filebeat Agent ]
                                          │ (Vận chuyển JSON)
                                          ▼
                                   [ Elasticsearch ] ◄──(Truy vấn)── [ Kibana UI ]
```

---

### LESSON 02: Triển khai EFK Stack và Sử dụng Kibana Truy vết Lỗi (Troubleshooting)

#### 1. Mục tiêu bài học

* **Xây dựng kịch bản Docker Compose** khởi chạy trọn vẹn cụm hạ tầng EFK Stack tích hợp chung vào mạng lưới dịch vụ QuickBite.
* **Thực hiện thao tác truy vết** sự cố nghiệp vụ (Troubleshooting) trên giao diện Kibana bằng ngôn ngữ truy vấn KQL (Kibana Query Language).

#### 2. Bối cảnh hệ thống

* **Trạng thái:** STATE 4 — Centralized Logging Architecture.
* **Vấn đề:** Triển khai cài đặt vật lý bộ công cụ EFK trên VPS và hướng dẫn lập trình viên quy trình vận hành thực tế: Tìm kiếm, lọc và khoanh vùng nguyên nhân gây ra lỗi của hệ thống QuickBite thông qua màn hình tổng đài Kibana.

#### 3. Nội dung trọng tâm

* **Cấu hình gắn kết ổ đĩa chung (Log Volume Mounting):** Các container Spring Boot cấu hình ở Session 16 sẽ ghi log ra một thư mục được chia sẻ chung với máy host VPS. Filebeat container sẽ mount tới đúng thư mục này để đọc dữ liệu.
* **Kibana Discover:** Công cụ tìm kiếm tối thượng của Kibana, hỗ trợ lọc log theo khoảng thời gian chuẩn xác đến từng mili-giây, lọc theo `level: "ERROR"` hoặc tìm kiếm từ khóa nghiệp vụ như mã đơn hàng `order_id`.

#### 4. Demo và thực hành

* **Mục tiêu demo:** Viết file cấu hình, bật cụm EFK và thực hành thao tác tìm kiếm dấu vết một lỗi logic giả lập của hệ thống trên giao diện Kibana.
* **Tệp cấu hình thu gom của Agent (`./quickbite-infra/filebeat/filebeat.yml`):**

```yaml
filebeat.inputs:
  - type: log
    enabled: true
    # Đường dẫn trỏ tới các file log thu thập (bên trong container filebeat)
    paths:
      - /var/log/apps/*.log
    # Nhận diện dòng log là JSON để tự động phân rã các trường dữ liệu
    json.keys_under_root: true
    json.overwrite_keys: true

output.elasticsearch:
  hosts: ["http://elasticsearch:9200"] # Đẩy trực tiếp vào Elasticsearch
  index: "quickbite-logs-%{+yyyy.MM.dd}"

# Vô hiệu hóa cấu hình mẫu mặc định để tự quản lý index
setup.ilm.enabled: false
```

* **Bổ sung cấu hình hạ tầng vào `docker-compose.yml` trên VPS:**

```yaml
services:
  # Cấu hình chia sẻ volume cho các service cũ để Filebeat đọc được log
  order-service:
    # ... cấu hình cũ ...
    volumes:
      - ./logs/apps:/app/logs # Ghi log ra thư mục chung của máy host

  # Mảnh ghép 1: Elasticsearch - Nhà kho dữ liệu
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.17.10
    container_name: quickbite-elasticsearch
    environment:
      - discovery.type=single-node # Chạy chế độ một node tiết kiệm RAM
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m" # Giới hạn RAM tối đa 512MB
    volumes:
      - es-data:/usr/share/elasticsearch/data
    networks:
      - quickbite-net

  # Mảnh ghép 2: Filebeat - Người đi cào và vận chuyển log
  filebeat:
    image: docker.elastic.co/beats/filebeat:7.17.10
    container_name: quickbite-filebeat
    user: root # Cần quyền root để đọc file hệ thống
    volumes:
      - ./filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - ./logs/apps:/var/log/apps:ro # Map chung thư mục chứa log của các app
    networks:
      - quickbite-net
    depends_on:
      - elasticsearch

  # Mảnh ghép 3: Kibana - Màn hình giao diện hiển thị
  kibana:
    image: docker.elastic.co/kibana/kibana:7.17.10
    container_name: quickbite-kibana
    ports:
      - "5601:5601" # Mở cổng 5601 để truy cập giao diện web
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    networks:
      - quickbite-net
    depends_on:
      - elasticsearch

volumes:
  es-data:
```

* **Quy trình thực hiện kiểm chứng lỗi:**
1. Chạy lệnh `docker compose up -d` trên VPS để khởi chạy toàn bộ cụm EFK.
2. Đăng nhập vào giao diện Kibana qua đường dẫn: `http://vps_public_ip:5601`.
3. Đi tới *Stack Management -> Index Patterns -> Create index pattern*. Nhập chuỗi `quickbite-logs-*` và nhấn *Create*.
4. Quay lại mục **Discover** ở thanh menu trái.
5. Thực hành tìm kiếm lỗi bằng cách gõ câu lệnh KQL vào thanh tìm kiếm:

```text
service : "user-service" AND level : "ERROR"
```

* **Kết quả mong đợi:** Kibana hiển thị danh sách toàn bộ các log lỗi của `user-service`. Nhờ việc log được chuẩn hóa sang JSON ở Session 16, sinh viên có thể bấm mở rộng dòng log để xem chi tiết trường `message` hoặc `thread_name` được phân tách thành từng cột dữ liệu trực quan rõ ràng.

#### 5. Điểm cần nhấn mạnh cho giảng viên (Pedagogical Tips)

* **Tư duy quản lý tài nguyên (Resource Constraints):** Giải thích rõ lý do cấu hình tham số `"ES_JAVA_OPTS=-Xms512m -Xmx512m"`. Elasticsearch mặc định ngốn rất nhiều RAM để lưu trữ chỉ mục (có thể tự chiếm 2GB - 4GB RAM). Khi chạy demo trên các con VPS cấu hình vừa phải, việc giới hạn RAM này là bắt buộc để ngăn chặn tình trạng VPS bị treo cứng do hết bộ nhớ.
* **Tính tức thời (Real-time Log Streaming):** Hướng dẫn sinh viên bật tính năng tự động làm mới dữ liệu (Auto-refresh) ở góc phải màn hình Kibana (ví dụ đặt 5 giây một lần). Mỗi khi sinh viên thực hiện một request đặt món lỗi trên Postman, dòng log lỗi tương ứng sẽ lập tức "bắn" lên màn hình Kibana gần như ngay tức thì.

#### 6. Hiểu lầm thường gặp

* **Hiểu sai:** Cho rằng Filebeat sẽ đọc trực tiếp dữ liệu từ màn hình Console thông qua lệnh `docker logs` của từng container.
* **Đính chính:** Trong mô hình bài học này, Filebeat đọc log từ **file vật lý nằm trong thư mục chung** (`/var/log/apps/*.log`) do ứng dụng ghi ra nhờ cấu hình `RollingFileAppender` ở Session 16. Đây là giải pháp phân tách độc lập (Decoupling) chuẩn công nghiệp, giúp giảm tải công việc cho Docker Daemon và đảm bảo log không bị mất kể cả khi container ứng dụng bị sập đột ngột.

---

*Giảng viên lưu ý: Kết thúc Session 17, sinh viên dự án QuickBite đã thiết lập thành công hệ thống hạ tầng quản lý nhật ký tập trung chuẩn sản xuất (Production-ready Centralized Logging). Khả năng cấu hình Logback xoay vòng, chuẩn hóa cấu trúc JSON kết hợp sức mạnh tìm kiếm full-text của Elasticsearch giúp nâng cao đáng kể năng lực làm chủ, giám sát và vận hành hệ thống của một kỹ sư DevOps thực thụ.*