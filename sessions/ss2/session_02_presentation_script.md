# Kịch bản Thuyết trình - Session 02: Giới thiệu Docker

---

## Lesson 1: Khái niệm Container và so sánh Docker với máy ảo (VM)

### 1. Phần lý thuyết

**(Mở đầu - Khơi gợi bối cảnh thực tế)**
Xin chào các bạn học viên! Ở buổi học trước, chúng ta đã làm quen với triết lý DevOps và quy trình CI/CD tự động rồi. Thế thì hôm nay, thầy sẽ cùng các bạn đi giải quyết một bài toán cực kỳ phổ biến trong thực tế khi vận hành hệ thống microservices. Đó là bài toán: **Xung đột phiên bản Java Runtime**.

Chúng ta hãy cùng xét một bối cảnh cụ thể của dự án QuickBite: Hiện tại hệ thống đang cần chạy đồng thời 2 dịch vụ độc lập trên cùng một máy chủ VPS.
1. Dịch vụ thứ nhất là `user-service`, do sử dụng một số thư viện bảo mật cũ chưa tương thích nên bắt buộc phải chạy trên **Java 17**.
2. Dịch vụ thứ hai là `restaurant-service`, do cần sử dụng tính năng Virtual Threads để tối ưu hiệu năng nên yêu cầu phải chạy trên **Java 21**.

Để triển khai cả 2 dịch vụ này, nếu làm theo những cách truyền thống trước đây, các bạn sẽ phải đối mặt với 2 phương án và những hạn chế đi kèm:
- **Phương án thứ nhất là Triển khai trực tiếp (Native Run):** Nghĩa là các bạn phải cài đặt song song cả JDK 17 và JDK 21 trên cùng một hệ điều hành của server. Sau đó, chúng ta phải cấu hình thủ công biến `JAVA_HOME` riêng cho từng tiến trình khi khởi chạy. Cách làm này rất dễ xảy ra lỗi. Chỉ cần một cấu hình sai lệch nhỏ, hoặc khi hệ điều hành tự động cập nhật, ứng dụng sẽ bị crash ngay lập tức do lỗi không tương thích phiên bản Java.
- **Phương án thứ hai là Sử dụng Máy ảo (Virtual Machine):** Chúng ta sẽ tạo ra 2 máy ảo độc lập, máy thứ nhất chạy Java 17 cho `user-service`, máy thứ hai chạy Java 21 cho `restaurant-service`. Cách này giúp cô lập môi trường rất tốt, nhưng lại cực kỳ lãng phí tài nguyên phần cứng. Mỗi máy ảo sẽ phải tự duy trì một hệ điều hành khách (Guest OS) riêng, làm tiêu tốn khoảng 1 đến 2 Gigabyte RAM chỉ để chạy hệ điều hành nền, trong khi tài nguyên của VPS thì có giới hạn.

**[Slide: Phép ẩn dụ Biệt thự vs Căn hộ]**
Để dễ hình dung nhất về sự khác biệt giữa Máy ảo (VM) và Container (Docker), thầy có một phép ẩn dụ trực quan thế này:
- **Máy ảo (VM) giống như "Biệt thự đơn lập":** Mỗi biệt thự được xây dựng trên một nền móng riêng biệt, có hệ thống tường bao, điện nước và mái nhà độc lập (tương ứng với **Guest OS** riêng biệt). Tính cô lập và bảo mật rất cao nhưng chi phí tài nguyên cực kỳ đắt đỏ.
- **Docker Container giống như "Căn hộ chung cư":** Các căn hộ nằm chung trong một tòa nhà, chia sẻ chung nền móng và hạ tầng đường nước chính (tương ứng với việc **dùng chung nhân Host OS - Shared Kernel**). Bên trong căn hộ `user-service`, ta cấu hình JRE 17. Căn hộ `restaurant-service` cấu hình JRE 21. Runtime của mỗi căn hộ hoàn toàn độc lập, tiết kiệm tài nguyên CPU/RAM tối đa và thời gian khởi tạo cực nhanh.

**[Slide: So sánh kỹ thuật trực diện Máy ảo (VM) vs Docker Container]**
Nhìn vào sơ đồ kiến trúc phần cứng, chúng ta thấy rõ điểm mấu chốt kỹ thuật:
- **Cơ chế ảo hóa:** VM ảo hóa ở **tầng phần cứng** (Phần mềm Hypervisor tạo ra CPU/RAM ảo). Container ảo hóa ở **tầng hệ điều hành** (Chạy trên OS host qua Docker Engine).
- **Hệ điều hành:** VM mang theo một **Guest OS** đầy đủ nặng hàng GB. Container **không có Guest OS**, dùng chung nhân (Kernel) của Host OS.
- **Tốc độ boot:** VM mất **hàng phút** để khởi động Guest OS. Container boot **mili-giây** vì chỉ khởi động tiến trình.
- **Tài nguyên:** VM khóa cứng tài nguyên tĩnh (cấp 2GB RAM là mất luôn 2GB RAM). Container tiêu hao tài nguyên động theo thực tế sử dụng.

**[Slide: Bản chất kỹ thuật của Container - Namespaces & Cgroups]**
Thế thì container thực chất là cái gì? Bản chất của nó chỉ là một tiến trình (Process) Linux bình thường, nhưng được nhân Linux bao bọc bởi 2 công cụ cô lập:
1. **Namespaces (Phân vùng cô lập):** Giống như việc lắp cửa khóa và tường ngăn giữa các căn hộ chung cư. Nó tạo ra một không gian ảo cô lập cho tiến trình (Mạng, File System, PID, User). Ví dụ: Nhờ `NET Namespace`, các container đều có thể chạy độc lập cùng lắng nghe cổng nội bộ `8080` mà không bị xung đột cổng mạng trên máy host.
2. **Cgroups (Control Groups - Giới hạn tài nguyên):** Giống như công tơ điện và đồng hồ nước giới hạn mức tiêu thụ tối đa. Nó giới hạn lượng CPU, RAM tối đa mà container được phép dùng, tránh trường hợp một dịch vụ bị treo làm nghẽn toàn bộ hệ thống.

*(Nhấn mạnh - Rủi ro bảo mật)*: Các bạn cần lưu ý, do dùng chung nhân hệ điều hành (Shared Kernel), nếu xảy ra lỗi bảo mật nghiêm trọng ở nhân Linux, hacker có thể leo thang đặc quyền để vượt ngục container (Container Escape) chiếm quyền máy host. Vì vậy, trên Production, các container thường được chạy bên trong các máy ảo (VM) độc lập để đảm bảo độ cô lập bảo mật phần cứng tốt nhất.

---

## Lesson 2: Docker image, container và registry

### 1. Phần lý thuyết

**[Slide: Vấn đề lệch cấu hình (Environment Drift)]**
Chào các bạn, chào mừng các bạn đến với nội dung số 2 của bài 2 này: Docker image, Docker container, Image Registry

Ở bài trước thầy đã giải thích các bạn về tính huống chạy song song các phiên bản java bằng công nghệ container rồi.
Chúng ta sẽ quay về với một kịch bản đời thường hơn, mà có thể các bạn đã từng gặp phải, hoặc ít nhiều cũng đã từng nghe, đó là câu chuyện "code chạy trên máy em mà, code chạy trên máy tao mà, máy của mày bị làm sao ý".

Câu chuyện này, tóm gọn lại nó là vấn đề lệch môi trường, nhưng liệu các bạn có bao giờ nghĩ rằng, mình có thể đóng gói toàn bộ môi trường để mang đi không, đỡ được bao nhiêu lỗi.

Nghe thì ngớ ngẩn, nhưng thực tế là làm được, nó chính là xoay quanh công nghệ container hoá, và các khái niệm Docker Image, Docker Container, hay Docker Registry.

Và chúng ta sẽ đi tìm hiểu, phân biệt 3 khái niệm này, vì sao nó lại tách ra 3 cái, và nó làm những việc gì.

**[Slide: Khái niệm Docker Image - Bản thiết kế bất biến]**
- **Docker Image (Tương đương Class trong OOP):** Là một khuôn mẫu đóng gói ở trạng thái chỉ đọc (Read-only) chứa toàn bộ mã nguồn, runtime Java (JRE), biến môi trường cấu hình mặc định và OS tối giản để chạy ứng dụng.
- **Tính chất cốt lõi:**
  - **Tính bất biến (Immutable):** Image đã build thành công không thể chỉnh sửa. Mọi thay đổi yêu cầu build một Image mới.
  - **Cấu trúc phân lớp (Layers):** Docker Image được cấu tạo từ nhiều lớp file system xếp chồng lên nhau (ví dụ: Lớp đáy là OS Ubuntu tối giản, lớp tiếp theo là JDK 17, lớp trên cùng là code file JAR). 
  - **Kiến thức nâng cao - Layer Caching & Cơ chế OverlayFS:** Docker sử dụng hệ thống file xếp lớp (như OverlayFS hoặc UnionFS). Nhờ vậy, khi build dịch vụ `restaurant-service` dùng chung base Java 17 với `user-service`, Docker sẽ tái sử dụng lại các layer OS và JDK 17 có sẵn ở máy mà không cần tải hay lưu trữ lại. Hơn nữa, khi triển khai bản cập nhật, nếu các bạn chỉ thay đổi code ứng dụng, Docker chỉ tạo và tải lên lớp trên cùng chứa file JAR mới (~5KB), còn 100MB+ của các lớp OS và JRE bên dưới sẽ được giữ nguyên từ cache. Nhờ đó, tốc độ triển khai ứng dụng chỉ tính bằng giây thay vì vài chục phút.

**[Slide: Khái niệm Docker Container - Thực thể chạy thực tế]**
- **Docker Container (Tương đương Object trong OOP):** Là một thực thể tiến trình sống động được khởi chạy từ Docker Image, hoạt động trên RAM và CPU.
- **Kiến thức nâng cao - Writable Layer & Cơ chế Copy-on-Write (CoW):** 
  Khi khởi chạy container, Docker Engine sẽ phủ thêm một lớp ghi tạm thời gọi là **Writable Layer** lên trên Image chỉ đọc. Mọi thao tác như ghi log, tạo file tạm của ứng dụng chỉ được lưu ở lớp ghi tạm này. 
  Đặc biệt, nếu ứng dụng cần sửa đổi một file hệ thống có sẵn trong Image, Docker sẽ copy file đó từ Image chỉ đọc lên lớp Writable Layer rồi mới chỉnh sửa (gọi là Copy-on-Write). Cơ chế này giúp hàng trăm container chạy chung một Image mà không sợ ghi đè hay làm hỏng dữ liệu của nhau. 
  *(Các bạn lưu ý)*: Khi container bị xóa (`docker rm`), lớp Writable Layer này sẽ bị hủy bỏ hoàn toàn. Dữ liệu phát sinh bên trong sẽ mất sạch, và Image gốc bên dưới vẫn toàn vẹn.

**[Slide: Khái niệm Docker Registry - Nhà kho phân phối]**
- **Docker Registry:** Là kho lưu trữ tập trung dùng để quản lý và chia sẻ các Docker Image. Gồm Public Registry (như Docker Hub chứa image mẫu của Postgres, OpenJDK) và Private Registry của doanh nghiệp (như AWS ECR, GitLab Registry) để bảo mật mã nguồn.
- **Kiến thức nâng cao - Registry Tag vs Digest:** Thông thường, chúng ta hay gọi Image qua các "tag" (ví dụ: `17-jre-alpine`). Tuy nhiên, các tag này có thể bị ghi đè (người khác build đè bản mới lên cùng tag đó). Để đảm bảo tính bảo mật và bất biến tuyệt đối trên Production, các kỹ sư DevOps thường gọi Image bằng chuỗi mã hóa **Digest (mã SHA256)** duy nhất của Image đó để tránh tình trạng deploy nhầm phiên bản bị thay đổi.

**[Slide: Quy trình phân phối Image qua dòng lệnh (Ví dụ minh họa)]**
Sau khi chúng ta cài đặt Docker ở bài học tiếp theo, quy trình đóng gói và kéo image về sử dụng sẽ hoạt động qua các câu lệnh cơ bản sau:
1. **`docker pull [tên_image]`**: Lệnh yêu cầu Docker Daemon kết nối lên Registry (Docker Hub) để tải Image về máy cục bộ.
   *Ví dụ:* `docker pull eclipse-temurin:17-jre-alpine`
2. **`docker images`**: Lệnh liệt kê danh sách toàn bộ các Image đã được tải về và lưu trữ trên ổ cứng của server.
3. **`docker run --rm [tên_image] [câu_lệnh]`**: Khởi tạo một container từ Image, thực thi câu lệnh bên trong, và tự động xóa bỏ container ngay khi chạy xong (`--rm`) để giải phóng tài nguyên.
   *Ví dụ:* `docker run --rm eclipse-temurin:17-jre-alpine java -version` (Dùng để kiểm tra nhanh phiên bản Java mà không cần giữ lại container).

---

## Lesson 3: Cài đặt Docker và kiểm tra môi trường

### 1. Phần lý thuyết

**[Slide: Kiến trúc Client - Server của Docker Engine]**
Thế thì trước khi bắt tay vào cài đặt, chúng ta cần hiểu Docker Engine hoạt động theo mô hình Client - Server gồm 3 thành phần chính:
1. **Docker CLI (Client):** Công cụ dòng lệnh (executable `docker`) tiếp nhận lệnh của người dùng và chuyển tiếp REST API request đi.
2. **REST API:** Giao thức chuẩn hóa giúp CLI tương tác với Server.
3. **Docker Daemon (Server - tiến trình `dockerd`):** Dịch vụ chạy ngầm trên máy chủ, chịu trách nhiệm quản lý tài nguyên, tải Image, khởi tạo Container, Network và Volume.

*(Nhấn mạnh - Về việc sử dụng Docker Desktop)*:
Thầy lưu ý là Docker Desktop hoàn toàn không phù hợp cho việc thực hành DevOps chuyên sâu. Nó ngốn nhiều tài nguyên và che giấu cơ chế socket bên dưới. Chúng ta bắt buộc cài đặt Docker Engine gốc trực tiếp trong shell Linux (WSL hoặc VM).

### 2. Phần thực hành (Demo)

**[Live Demo: Quy trình cài đặt Docker Engine gốc trên Ubuntu]**
Bây giờ, thầy sẽ hướng dẫn các bạn cài đặt Docker Engine gốc trực tiếp trên môi trường Ubuntu Linux (ở đây thầy demo trên máy WSL 2 Ubuntu).

*(Hướng dẫn thao tác trên Terminal)*:
- *Bước 1: Cài đặt các thư viện HTTPS hỗ trợ tải gói an toàn:*
  ```bash
  sudo apt-get update
  sudo apt-get install -y ca-certificates curl gnupg
  ```
- *Bước 2: Tạo thư mục keyring và tải GPG key chính thức của Docker về để xác thực gói:*
  ```bash
  sudo install -m 0755 -d /etc/apt/keyrings
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
  ```
- *Bước 3: Cấu hình apt repository trỏ về nguồn Docker chính thức:*
  ```bash
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  ```
- *Bước 4: Cập nhật danh sách gói và thực hiện cài đặt Docker Engine:*
  ```bash
  sudo apt-get update
  sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
  ```

**[Live Demo: Cấu hình sau cài đặt (Post-installation)]**
*Sau khi cài xong, nếu các bạn chạy lệnh `docker ps` ngay sẽ bị lỗi phân quyền socket. Thầy sẽ hướng dẫn các bạn cấu hình chạy Docker không cần gõ `sudo`:*
- *Thêm user hiện tại vào nhóm `docker`:*
  ```bash
  sudo groupadd docker
  sudo usermod -aG docker $USER
  ```
- *Nạp lại cấu hình nhóm ngay lập tức:*
  ```bash
  newgrp docker
  ```

**[Live Demo: Kiểm tra môi trường]**
- *Giờ thầy kiểm tra kết nối Client - Daemon bằng lệnh:*
  ```bash
  docker version
  ```
  *Các bạn thấy thông tin hiển thị đầy đủ của cả Client và Server Daemon.*
- *Xem thông tin tài nguyên hệ thống:*
  ```bash
  docker info
  ```
- *Khởi chạy container đầu tiên để xác minh luồng hoạt động:*
  ```bash
  docker run hello-world
  ```
  *Docker Daemon sẽ tự động pull image `hello-world` từ Docker Hub về nếu máy local chưa có, sau đó khởi chạy container và in ra màn hình thông điệp chào mừng.*

---

## Lesson 4: Các lệnh Docker cơ bản trong vòng đời container

### 1. Phần lý thuyết

**[Slide: Câu chuyện thực tế - Kịch bản "ăn hành" của Intern khi chạy Database]**
Chúng ta sẽ đặt một bài toán như sau: *"Làm sao để chạy một container PostgreSQL làm database tạm thời, container này phải có thể chạy được dưới nền (không chiếm shell) và không bị vô tình ngắt tắt bằng CTR C, và làm sao cho ứng dụng `user-service` đang chạy ở máy host lại có thể kết nối vào database trong container để ghi dữ liệu"*.

Dĩ nhiên rồi, để chạy được container, lệnh đầu tiên chúng ta cần biết là `docker run`, cùng với **4 tham số quan trọng sau đây**:
- **`-d` (Detached mode):** Chạy container dưới nền (background), giải phóng terminal.
- **`-p host_port:container_port` (Port Mapping):** Ánh xạ cổng mạng máy host vào cổng container. Ví dụ: `-p 5432:5432` giúp chuyển hướng yêu cầu truy cập cổng `5432` của máy host vào database trong container.
- **`-e KEY=VALUE` (Environment Variable):** Truyền biến môi trường vào container (ví dụ: `-e POSTGRES_PASSWORD=secret` để đặt mật khẩu).
- **`--name`:** Đặt tên định danh dễ quản lý (ví dụ: `quickbite-db`).

**[Slide: Vòng đời của Container & Vấn đề lưu trữ dữ liệu]**
Tóm lại, tiến trình của container trải qua 4 trạng thái chính:
`Created (Đã tạo) ──► Running (Đang chạy) ──► Stopped (Đã dừng) ──► Destroyed (Đã xóa vĩnh viễn)`
- Lệnh `docker stop` chỉ tạm dừng tiến trình, dữ liệu tạm thời vẫn còn trên ổ đĩa.
- Tuy nhiên, khi dùng `docker rm` để xóa hẳn container, toàn bộ dữ liệu ghi trên lớp Writable Layer của container đó (ví dụ các bảng dữ liệu database đã tạo) sẽ biến mất hoàn toàn. 

Để giải quyết vấn đề lưu trữ bền vững (Data Persistence), chúng ta sử dụng **Docker Volume** (cụ thể là **Named Volume - Volume có tên**). Nó là một phân vùng ổ đĩa do Docker tự động tạo và quản lý độc lập ngoài vòng đời của container trên máy host.
- Cú pháp gắn volume khi chạy container: `-v tên_volume:đường_dẫn_trong_container`
- Đối với PostgreSQL, thư mục chứa dữ liệu mặc định bên trong container là `/var/lib/postgresql/data`. Do đó, chúng ta sẽ truyền tham số: `-v quickbite-db-data:/var/lib/postgresql/data`.
- Khi container bị xóa, phân vùng `quickbite-db-data` trên máy host vẫn nguyên vẹn. Khi các bạn tạo container mới và gắn lại volume này, dữ liệu cũ sẽ tự động được phục hồi đầy đủ.

### 2. Phần thực hành (Demo)

**[Live Demo: Vòng đời hoạt động và Sự cố mất dữ liệu khi thiếu Volume]**
Bây giờ, thầy sẽ demo trực tiếp cho các bạn thấy sự nguy hiểm của việc không mount volume, và cách dùng Named Volume để bảo vệ dữ liệu database của QuickBite qua các câu lệnh psql thực tế.

*(Hướng dẫn thao tác trên Terminal)*:
- *Bước 1: Khởi chạy container database KHÔNG cấu hình volume:*
  ```bash
  docker run -d --name quickbite-db -p 5432:5432 -e POSTGRES_PASSWORD=secret postgres:15-alpine
  ```
- *Bước 2: Kết nối vào database bằng công cụ psql bên trong container để tạo dữ liệu giả lập:*
  ```bash
  docker exec -it quickbite-db psql -U postgres
  ```
  *Bên trong môi trường psql, gõ các lệnh SQL:*
  ```sql
  CREATE DATABASE quickbite_user;
  \c quickbite_user;
  CREATE TABLE users (id SERIAL PRIMARY KEY, name VARCHAR(50));
  INSERT INTO users (name) VALUES ('Nguyen Van A');
  SELECT * FROM users;
  \q
  ```
- *Bước 3: Dừng và xóa container này đi:*
  ```bash
  docker stop quickbite-db
  docker rm quickbite-db
  ```
- *Bước 4: Khởi chạy lại một container mới (vẫn KHÔNG cấu hình volume):*
  ```bash
  docker run -d --name quickbite-db -p 5432:5432 -e POSTGRES_PASSWORD=secret postgres:15-alpine
  ```
  *Thử kết nối vào psql để kiểm tra lại database `quickbite_user`:*
  ```bash
  docker exec -it quickbite-db psql -U postgres
  ```
  *Bên trong psql, các bạn gõ lệnh kiểm tra:*
  ```sql
  \l
  ```
  *(Giải thích): Các bạn thấy danh sách database hoàn toàn trống rỗng, không hề có `quickbite_user` vì container trước đã bị xóa sạch lớp Writable Layer. Thoát ra ngoài bằng lệnh:*
  ```sql
  \q
  ```
  *Dọn dẹp container lỗi này:*
  ```bash
  docker stop quickbite-db
  docker rm quickbite-db
  ```

- *Bước 5: Khởi chạy container mới tinh kèm theo Named Volume:*
  ```bash
  docker run -d --name quickbite-db -p 5432:5432 -e POSTGRES_PASSWORD=secret -v quickbite-db-data:/var/lib/postgresql/data postgres:15-alpine
  ```
- *Bước 6: Kết nối lại và tạo dữ liệu như ở Bước 2:*
  ```bash
  docker exec -it quickbite-db psql -U postgres
  ```
  *Chạy lại các lệnh SQL:*
  ```sql
  CREATE DATABASE quickbite_user;
  \c quickbite_user;
  CREATE TABLE users (id SERIAL PRIMARY KEY, name VARCHAR(50));
  INSERT INTO users (name) VALUES ('Nguyen Van A');
  \q
  ```
- *Bước 7: Tiến hành dừng và xóa container để kiểm tra tính bền vững:*
  ```bash
  docker stop quickbite-db
  docker rm quickbite-db
  ```
- *Bước 8: Khởi chạy một container mới tinh nhưng gắn chung Named Volume `quickbite-db-data` cũ vào:*
  ```bash
  docker run -d --name quickbite-db -p 5432:5432 -e POSTGRES_PASSWORD=secret -v quickbite-db-data:/var/lib/postgresql/data postgres:15-alpine
  ```
- *Bước 9: Kết nối psql vào thẳng database `quickbite_user` để kiểm chứng dữ liệu:*
  ```bash
  docker exec -it quickbite-db psql -U postgres -d quickbite_user
  ```
  *Chạy lệnh kiểm tra bảng:*
  ```sql
  SELECT * FROM users;
  ```
  *(Kết quả mong đợi): Màn hình hiển thị chính xác dòng dữ liệu 'Nguyen Van A' của chúng ta. Dữ liệu đã được bảo vệ thành công! Thoát psql bằng lệnh:*
  ```sql
  \q
  ```


---

## Lesson 5: Kiểm tra log và truy cập container (logs, exec)

### 1. Phần lý thuyết

**[Slide: Câu chuyện lỗi Database của Intern]**
Quay lại dự án QuickBite, ứng dụng `user-service` báo lỗi không kết nối được tới database. Anh Tech Lead bảo bạn Intern: *"Em hãy vào kiểm tra xem database `quickbite_user` đã được tạo tự động bên trong container `quickbite-db` chưa, và kiểm tra xem có log lỗi gì trong Postgres không"*.

Bạn Intern loay hoay gõ lệnh:
```bash
docker run -it postgres:15-alpine sh
```
Bạn ấy chui vào shell, nhưng tìm mãi không thấy database nào tên là `quickbite_user`. 
Lý do là bạn ấy đã dùng lệnh `docker run`, lệnh này luôn tạo ra một container mới tinh, trống rỗng từ image gốc, chứ không hề can thiệp vào container database `quickbite-db` đang chạy lỗi kia.

Để chẩn đoán đúng tiến trình đang chạy mà không làm gián đoạn dịch vụ, chúng ta cần sử dụng hai lệnh chẩn đoán tối quan trọng: `docker logs` và `docker exec`.

- **Cơ chế Logs của Docker (`docker logs`):** Docker Daemon tự động thu thập toàn bộ luồng đầu ra tiêu chuẩn (`STDOUT`) và báo lỗi (`STDERR`) phát ra từ tiến trình chính bên trong container và lưu lại.
- **Lệnh tương tác trực tiếp (`docker exec`):** Cho phép bạn chạy một tiến trình mới (như một shell terminal `sh` hoặc `bash`) ngay bên trong phân vùng cô lập của một container **đang chạy**.
  - Cú pháp bắt buộc để mở Terminal tương tác: `docker exec -it [tên_container] [lệnh_shell]`.
  - Cờ `-i` (interactive) để giữ luồng nhập dữ liệu luôn mở để gõ phím tương tác.
  - Cờ `-t` (tty) cấp một Terminal ảo hiển thị giao diện dòng lệnh.

*(Nhấn mạnh)*: Gõ lệnh `exit` để thoát khỏi shell của `docker exec` chỉ tắt tiến trình shell con đó, tiến trình PostgreSQL chính của container vẫn chạy bình thường, hoàn toàn không làm sập container.

### 2. Phần thực hành (Demo)

**[Live Demo: Kiểm tra logs và kết nối Database bên trong Container]**
Thầy sẽ khởi chạy lại container `quickbite-db` và thực hiện các bước chẩn đoán lỗi trực tiếp cho các bạn xem.

*(Hướng dẫn thao tác trên Terminal)*:
- *Đầu tiên, thầy chạy lại container postgres:*
  ```bash
  docker run -d --name quickbite-db -p 5432:5432 -e POSTGRES_PASSWORD=secret postgres:15-alpine
  ```
- *Bước 1: Xem toàn bộ log khởi động của database PostgreSQL:*
  ```bash
  docker logs quickbite-db
  ```
- *Bước 2: Theo dõi log thời gian thực (Follow) để giám sát các truy vấn mạng:*
  ```bash
  docker logs -f quickbite-db
  ```
  *(Thầy gõ Ctrl+C để thoát khỏi chế độ theo dõi).*
- *Bước 3: Sử dụng `docker exec` truy cập vào shell bên trong container database:*
  ```bash
  docker exec -it quickbite-db sh
  ```
- *Bước 4: Gọi công cụ `psql` để truy cập database với user `postgres`:*
  ```bash
  psql -U postgres
  ```
- *Bước 5: Liệt kê danh sách các database đang tồn tại bằng lệnh:*
  ```bash
  \l
  ```
- *Bước 6: Thoát khỏi công cụ `psql` và thoát khỏi shell container quay về máy host:*
  ```bash
  \q
  exit
  ```
  *Tiến trình Postgres chính của container vẫn đang hoạt động ổn định ở chế độ nền.*

Đó là toàn bộ những kỹ năng kiểm tra và tương tác cốt lõi nhất của Docker giúp các bạn chẩn đoán và vận hành hệ thống một cách trơn tru. Cảm ơn các bạn và hẹn gặp lại ở Session tiếp theo!
