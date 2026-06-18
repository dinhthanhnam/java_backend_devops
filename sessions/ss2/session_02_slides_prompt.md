# PROMPT CHO GAMMA: GIỚI THIỆU DOCKER (SESSION 2)

## 1. CONTEXT & ROLE
* **Role:** DevOps Architect kiêm Giảng viên cao cấp. Giọng điệu chuyên nghiệp, trực diện, đi thẳng vào bản chất kỹ thuật và thực tiễn vận hành hệ thống.
* **Target Audience:** Kỹ sư phần mềm, học viên đang phát triển dự án Spring Boot Microservices (hệ thống QuickBite).
* **Objective:** Giải thích cặn kẽ bản chất kỹ thuật của container, cung cấp sẵn các câu lệnh, cấu hình và sơ đồ luồng chi tiết để người học hiểu và áp dụng được ngay vào thực tế.

---

## 2. PARTITION INSTRUCTIONS
* **Số lượng slide khuyến nghị:** 12 - 16 slides.
* **Nguyên tắc phân bổ nội dung:**
  * **LESSON 01 (Mở đầu & Dẫn dắt):** Phân tích bối cảnh bất đồng bộ phiên bản Java Runtime, so sánh kiến trúc giữa máy ảo (VM) và Container bằng phép ẩn dụ (Biệt thự vs Căn hộ), giải thích Namespaces và Cgroups.
  * **Từ LESSON 02 đến LESSON 05 (Giải pháp kỹ thuật):** Đi thẳng vào định nghĩa, sơ đồ luồng hoạt động, cấu hình và lệnh thực hành thực tế.
  * **Độ thoáng đãng:** Một slide chỉ trình bày một thông điệp hoặc khái niệm cốt lõi. Không nhồi nhét chữ.
  * **Độ cô đọng cao:** Sử dụng các câu văn ngắn gọn, súc tích, loại bỏ các từ ngữ suồng sã, giật gân (như "ăn hành", "intern").

---

## 3. HIERARCHICAL OUTLINE (DÀN Ý CHI TIẾT)

### LESSON 01: Khái niệm container và so sánh Docker với máy ảo

#### Slide 1: Vấn đề thực tế - Xung đột phiên bản Runtime của Microservices
* **Bối cảnh dự án QuickBite:** Triển khai đồng thời 2 dịch vụ độc lập lên cùng một máy chủ VPS:
  * Dịch vụ 1: `user-service` yêu cầu chạy trên **Java 17**.
  * Dịch vụ 2: `restaurant-service` sử dụng Virtual Threads, yêu cầu chạy trên **Java 21**.
* **Các hạn chế của giải pháp truyền thống:**
  * *Triển khai trực tiếp (Native Run):* Phải cài đặt song song cả JDK 17 và JDK 21 trên cùng một hệ điều hành. Phải quản lý biến `JAVA_HOME` và đường dẫn chạy tiến trình thủ công, dễ dẫn đến lỗi crash `UnsupportedClassVersionError` khi cấu hình sai lệch.
  * *Sử dụng Máy ảo (Virtual Machine - VM):* Tạo các máy ảo riêng biệt để cô lập runtime. Giải pháp này gây lãng phí lớn tài nguyên phần cứng vì mỗi máy ảo phải duy trì một Hệ điều hành khách (Guest OS) riêng biệt (~1-2GB RAM mỗi VM).

#### Slide 2: Phép ẩn dụ trực quan - Biệt thự đơn lập vs Căn hộ chung cư
* **Máy ảo (VM) giống như "Biệt thự đơn lập":**
  * Mỗi biệt thự được xây dựng trên một nền móng riêng biệt, có hệ thống tường bao, điện nước và mái nhà độc lập (tương ứng với **Guest OS** riêng biệt).
  * *Đặc tính:* Tính cô lập và bảo mật phần cứng rất cao nhưng chi phí tài nguyên cực kỳ đắt đỏ.
* **Docker Container giống như "Căn hộ chung cư":**
  * Các căn hộ nằm chung trong một tòa nhà, chia sẻ chung nền móng và hạ tầng đường nước chính (tương ứng với việc **dùng chung nhân Host OS - Shared Kernel**).
  * Bên trong căn hộ `user-service`, ta cấu hình JRE 17. Căn hộ `restaurant-service` cấu hình JRE 21. Runtime của mỗi căn hộ hoàn toàn độc lập.
  * *Đặc tính:* Tiết kiệm tài nguyên CPU/RAM tối đa và thời gian khởi tạo cực nhanh.

#### Slide 3: So sánh kỹ thuật trực diện Máy ảo (VM) vs Docker Container
* Bảng so sánh các tiêu chí kỹ thuật cốt lõi:
  * *Cơ chế ảo hóa:* VM ảo hóa ở **tầng phần cứng** (Hypervisor tạo CPU/RAM ảo). Container ảo hóa ở **tầng hệ điều hành** (Chạy trên OS host qua Docker Engine).
  * *Hệ điều hành:* VM mang theo một **Guest OS** đầy đủ. Container **không có Guest OS**, dùng chung nhân (Kernel) của Host OS.
  * *Môi trường Java:* VM chạy JDK riêng trên hệ điều hành khách. Container đóng gói kèm một phiên bản JDK riêng biệt chỉ chạy trong container đó.
  * *Tốc độ boot:* VM mất **hàng phút** (khởi động OS). Container boot **mili-giây** (khởi động tiến trình).
  * *Tài nguyên:* VM **khóa cứng tài nguyên tĩnh**. Container **tiêu hao tài nguyên động** theo thực tế sử dụng.

#### Slide 4: Sơ đồ kiến trúc phần cứng chi tiết
* So sánh sơ đồ kiến trúc phần cứng của Máy ảo và Docker Container:
  ```text
          MÔ HÌNH MÁY ẢO (VM)                 MÔ HÌNH DOCKER CONTAINER
    ┌───────────────────────────────┐   ┌───────────────────────────────┐
    │ user-service  │restaurant-svc │   │ user-service  │restaurant-svc │
    │   (Java 17)   │   (Java 21)   │   │   (Java 17)   │   (Java 21)   │
    ├─────────────┬─┴───────────────┤   ├─────────────┬─┴───────────────┤
    │ Thư viện    │ Thư viện        │   │ Thư viện    │ Thư viện        │
    ├─────────────┼─────────────────┤   ├───────────────────────────────┤
    │ Guest OS 1  │ Guest OS 2      │   │         Docker Engine         │
    ├─────────────┴─────────────────┤   ├───────────────────────────────┤
    │          Hypervisor           │   │            Host OS            │
    ├───────────────────────────────┤   ├───────────────────────────────┤
    │       Infrastructure          │   │       Infrastructure          │
    └───────────────────────────────┘   └───────────────────────────────┘
  ```
* *Điểm mấu chốt:* Container thay thế lớp Guest OS và Hypervisor bằng **Docker Engine** gọn nhẹ. Tiến trình chạy trực tiếp trên Host OS giúp đạt hiệu năng tiệm cận dịch vụ chạy trực tiếp (Native).

#### Slide 5: Bản chất kỹ thuật của Container - Namespaces & Cgroups
* **Container thực chất chỉ là một tiến trình (Process) Linux bình thường**, được cô lập bởi 2 tính năng của nhân Linux:
  1. **Namespaces (Phân vùng cô lập):**
     * Tạo môi trường tách biệt cho tiến trình (Mạng, File System, PID, User).
     * Ví dụ: `NET Namespace` cho phép các container chạy độc lập cùng lắng nghe cổng nội bộ `8080` mà không bị xung đột cổng mạng trên máy host.
  2. **Cgroups (Control Groups - Giới hạn tài nguyên):**
     * Giới hạn lượng CPU, RAM, I/O tối đa mà container được phép sử dụng, ngăn chặn một dịch vụ bị lỗi treo làm nghẽn toàn bộ hệ thống.
* **Rủi ro bảo mật (Shared Kernel):**
  * Do dùng chung nhân hệ điều hành (Shared Kernel), nếu xảy ra lỗi bảo mật nghiêm trọng ở nhân Linux, hacker có thể leo thang đặc quyền để vượt ngục container (Container Escape) chiếm quyền máy host.
  * *Lời khuyên thực chiến:* Trên Production, các container thường được chạy bên trong các máy ảo (VM) độc lập để đảm bảo độ cô lập bảo mật phần cứng tốt nhất.
* **Dẫn nhập chuyển tiếp:** *"Để đóng gói ứng dụng cùng toàn bộ runtime cần thiết và phân phối nó không lỗi lệch pha cấu hình, chúng ta cần tìm hiểu bộ ba: Image, Container và Registry..."*

---

### LESSON 02: Docker image, container và registry

#### Slide 6: Vấn đề lệch cấu hình (Environment Drift) & Định nghĩa Docker Image
* **Vấn đề thực tế:** Việc chỉ copy file JAR lên server Staging dễ gây ra lỗi lệch cấu hình hệ thống (như sai phiên bản Java của OS server, hoặc lệch múi giờ hệ điều hành), khiến ứng dụng sập hoặc log sai lệch giờ.
* **Docker Image (Bản thiết kế bất biến - Class trong OOP):**
  * Là khuôn mẫu chỉ đọc (Read-only) chứa toàn bộ mã nguồn, runtime Java (JRE), biến môi trường mặc định và OS tối giản để chạy ứng dụng.
  * **Tính bất biến (Immutable):** Image đã build thành công không thể chỉnh sửa. Mọi thay đổi yêu cầu build một Image mới.
  * **Cấu trúc phân lớp (Layers):** Gồm nhiều lớp file system xếp chồng lên nhau (Ubuntu -> JRE -> code file JAR). Docker tự động tái sử dụng các layer có sẵn giữa các image để tiết kiệm dung lượng đĩa.
  * **Build từ Dockerfile:** Quy trình build Image được định nghĩa tự động qua mã nguồn cấu hình `Dockerfile`.

#### Slide 7: Định nghĩa Docker Container, Registry và Quy trình phân phối
* **Docker Container (Thực thể sống động - Object trong OOP):**
  * Là một thực thể tiến trình được khởi chạy từ Docker Image, hoạt động trên RAM và CPU.
  * *Cơ chế Writable Layer:* Khi khởi chạy, Docker Engine phủ thêm một lớp ghi tạm thời **Writable Layer (Container Layer)** lên trên Image chỉ đọc. Mọi dữ liệu phát sinh (log, file tạm) chỉ ghi vào lớp này và sẽ mất khi xóa container (`docker rm`), bảo toàn Image gốc bên dưới.
* **Docker Registry (Nhà kho lưu trữ):**
  * Kho lưu trữ tập trung dùng để quản lý và chia sẻ các Docker Image.
    * *Public Registry:* Docker Hub (chứa image mẫu của Postgres, OpenJDK, Nginx).
    * *Private Registry:* Kho riêng tư của doanh nghiệp (AWS ECR, GitLab Registry) để lưu trữ bảo mật mã nguồn đóng của dự án QuickBite.
* **Sơ đồ phân phối ứng dụng:**
  ```text
   [Local Dev] ──(Build Image)──► [Push] ──► [Registry] ──► [Pull] ──► [Server Staging/Prod] ──► [Run Container]
  ```

#### Slide 8: Các lệnh thực hành minh họa vòng đời mẫu
* Thực hiện tải và kiểm tra phiên bản Java chuẩn hóa qua container:
```bash
# 1. Tải (pull) image JRE 17 từ Docker Hub
docker pull eclipse-temurin:17-jre-alpine

# 2. Liệt kê các Image hiện có cục bộ
docker images

# 3. Khởi chạy container kiểm tra phiên bản Java và tự động hủy sau khi chạy xong
docker run --rm eclipse-temurin:17-jre-alpine java -version
```
* **Lưu ý quan trọng:** Sửa đổi cấu hình trực tiếp bên trong container đang chạy chỉ lưu tạm ở Writable Layer và sẽ biến mất khi container bị xóa. Muốn thay đổi vĩnh viễn phải sửa file cấu hình ở ngoài máy host rồi tiến hành build lại Image mới.

---

### LESSON 03: Cài đặt Docker và kiểm tra môi trường

#### Slide 9: Kiến trúc Client - Server của Docker Engine
* Docker Engine hoạt động theo mô hình Client - Server (Khách - Chủ) gồm 3 thành phần chính:
  ```text
   ┌────────────────────────┐         REST API         ┌────────────────────────┐
   │   Docker CLI (Client)  │ ───────────────────────► │ Docker Daemon (Server) │
   │ (Người dùng gõ lệnh)   │ ◄─────────────────────── │ (dockerd - Chạy ngầm)  │
   └────────────────────────┘                          └────────────────────────┘
  ```
  1. **Docker CLI (Client):** Công cụ dòng lệnh (executable `docker`) tiếp nhận lệnh của người dùng và chuyển tiếp REST API request đi.
  2. **REST API:** Giao thức chuẩn hóa giúp CLI tương tác với Server.
  3. **Docker Daemon (Server - tiến trình `dockerd`):** Dịch vụ chạy ngầm quản lý tài nguyên, tải Image, khởi tạo Container, Network và Volume.
* **Về việc sử dụng Docker Desktop:**
  * Docker Desktop không phù hợp cho việc thực hành DevOps chuyên sâu do che giấu cơ chế socket bên dưới và tiêu tốn nhiều tài nguyên. Học viên bắt buộc cài đặt Docker Engine gốc trực tiếp trong shell Linux (WSL hoặc VM).

#### Slide 10: Quy trình cài đặt Docker Engine gốc trên Ubuntu Linux
* Lệnh cài đặt Docker Engine tiêu chuẩn cho server Staging/Production và WSL 2/VM Ubuntu:
```bash
# 1. Cài đặt các thư viện HTTPS hỗ trợ tải gói an toàn
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg

# 2. Thêm GPG key chính thức của Docker để xác thực gói
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 3. Cấu hình apt repository trỏ về nguồn Docker chính thức
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 4. Thực hiện cài đặt Docker Engine
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```

#### Slide 11: Cấu hình sau cài đặt (Post-installation) & Kiểm tra môi trường
* **Chạy Docker không cần gõ `sudo`:** Thêm user hiện hành vào nhóm `docker` để có quyền ghi trực tiếp vào socket:
```bash
# 1. Tạo nhóm docker (nếu chưa có)
sudo groupadd docker
# 2. Thêm người dùng hiện tại ($USER) vào nhóm
sudo usermod -aG docker $USER
# 3. Kích hoạt thay đổi nhóm mà không cần logout
newgrp docker # Nạp lại cấu hình nhóm
```
* **Kích hoạt Systemd trên WSL 2 (Windows):** Thêm cấu hình vào file `sudo nano /etc/wsl.conf`:
```ini
[boot]
systemd=true
```
  Sau đó, chạy lệnh `wsl --shutdown` từ PowerShell của Windows để hoàn tất.
* **Khối lệnh kiểm tra môi trường:**
```bash
# 1. Kiểm tra kết nối Client - Daemon
docker version

# 2. Kiểm tra thông tin tài nguyên hệ thống
docker info

# 3. Khởi chạy container đầu tiên để xác minh luồng tải và chạy
docker run hello-world
```

---

### LESSON 04: Các lệnh Docker cơ bản trong vòng đời container

#### Slide 12: Bốn tham số quan trọng khi khởi chạy Container (`docker run`)
* Khởi chạy container database PostgreSQL cần cấu hình các tham số mạng và chạy nền để tránh bị khóa terminal hoặc lỗi kết nối:
  * **`-d` (Detached mode):** Chạy container dưới nền (background), giải phóng terminal của máy host.
  * **`-p host_port:container_port` (Port Mapping):** Ánh xạ cổng máy host vào cổng container (ví dụ: `-p 5432:5432` cho phép bên ngoài kết nối tới Postgres qua cổng 5432 của máy host).
  * **`-e KEY=VALUE` (Environment Variable):** Truyền biến cấu hình vào container (ví dụ: `-e POSTGRES_PASSWORD=secret` để thiết lập mật khẩu DB).
  * **`--name [tên]`:** Đặt tên định danh cho container dễ quản lý.

```bash
docker run -d --name quickbite-db -p 5432:5432 -e POSTGRES_PASSWORD=secret postgres:15-alpine
```

#### Slide 13: Vòng đời của Container & Lệnh điều khiển
* **Sơ đồ vòng đời của container:**
  ```text
  [Created] (Đã tạo cấu hình) ──► [Running] (Đang hoạt động) ──► [Stopped] (Đã tắt tiến trình) ──► [Destroyed] (Xóa khỏi đĩa)
  ```
* **Các câu lệnh quản lý container thực tế:**
```bash
# 1. Tạo và chạy container PostgreSQL dưới nền, ánh xạ cổng 5432
docker run -d --name quickbite-db -p 5432:5432 -e POSTGRES_PASSWORD=secret postgres:15-alpine

# 2. Kiểm tra danh sách container đang chạy
docker ps

# 3. Tắt container (tạm dừng tiến trình chính)
docker stop quickbite-db

# 4. Kiểm tra danh sách tất cả container (gồm cả đã dừng)
docker ps -a
# Status: Exited (0)

# 5. Khởi động lại container cũ đã dừng
docker start quickbite-db

# 6. Dừng và xóa vĩnh viễn container khỏi đĩa máy host
docker stop quickbite-db
docker rm quickbite-db
```
* **Lưu ý:** Lệnh `docker stop` chỉ tạm dừng tiến trình, dữ liệu tạm thời vẫn được giữ lại. Dữ liệu chỉ bị xóa sạch khi dùng lệnh `docker rm`.

---

### LESSON 05: Kiểm tra log và truy cập container (logs, exec)

#### Slide 14: Cơ chế quản lý Logs của Docker (`docker logs`)
* **Bản chất cơ chế:** Docker Daemon tự động thu thập và lưu trữ toàn bộ luồng đầu ra tiêu chuẩn (`STDOUT`) và báo lỗi (`STDERR`) phát ra từ tiến trình chính bên trong container.
* Lập trình viên có thể truy xuất log để chẩn đoán lỗi khởi chạy của database hoặc ứng dụng mà không cần mở file log trực tiếp trên máy host:
  * **Xem toàn bộ log lịch sử:**
```bash
docker logs quickbite-db
# Output ví dụ:
# [POSTGRES] ready for connections
# [INFO] Database initialized...
```
  * **Theo dõi log liên tục thời gian thực (Follow):**
```bash
docker logs -f quickbite-db
# Nhấn tổ hợp phím Ctrl+C để dừng
# Log sẽ tự động cập nhật khi
# có request mới đến container...

```

#### Slide 15: Truy cập container đang chạy (`docker exec`) & Debug Database
* **Lệnh `docker exec`:**
  * Cho phép chạy một tiến trình con phụ (như shell `sh` hoặc `bash`) song song bên trong phân vùng cô lập của container **đang hoạt động**.
  * Cú pháp mở terminal tương tác:
```bash
docker exec -it [tên_container] [lệnh_shell]
```
    *Ý nghĩa cờ `-it`:* `-i` giữ luồng nhập dữ liệu (`STDIN`) luôn mở; `-t` cấp Terminal ảo để hiển thị shell tương tác.
* **Khối lệnh thực hành kết nối kiểm tra database bên trong container:**
```bash
# 1. Mở shell tương tác vào container database đang hoạt động
docker exec -it quickbite-db sh

# ---- LỆNH TRONG TERMINAL CONTAINER ----
# Kết nối vào database Postgres bằng công cụ psql
psql -U postgres

# Liệt kê danh sách database để kiểm tra sự tồn tại
\l

# Thoát psql và thoát khỏi shell container quay về máy host
\q
exit
# ---------------------------------------
```
* **Điểm cần lưu ý:** Lệnh `exit` trong shell của `docker exec` chỉ dừng tiến trình con shell vừa được tạo, hoàn toàn không làm gián đoạn tiến trình PostgreSQL chính của container.
