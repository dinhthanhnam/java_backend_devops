# Kịch bản Thuyết trình - Session 02: Giới thiệu Docker

---

## Lesson 1: Khái niệm Container và so sánh Docker với máy ảo (VM)

### 1. Phần lý thuyết

**(Mở đầu - Khơi gợi bối cảnh thực tế)**
Xin chào các bạn học viên! Ở buổi học trước, chúng ta đã làm quen với triết lý DevOps và quy trình CI/CD tự động. Thế thì hôm nay, thầy sẽ đưa các bạn đi giải quyết một bài toán cực kỳ đau đầu trong thế giới microservices thực tế. Đó là bài toán: **Xung đột phiên bản Runtime**.

Hãy tưởng tượng, hệ thống QuickBite của chúng ta chạy thử trên môi trường Staging hiện tại đang có 2 dịch vụ độc lập:
1. `user-service` được viết từ lâu, đang chạy ổn định trên **Java 17** do vướng một số thư viện Spring Security và JWT cũ chưa tương thích với bản mới.
2. `restaurant-service` được viết sau, sử dụng các tính năng mới của **Java 21** như Virtual Threads để tối ưu hóa hiệu năng chịu tải.

Nhiệm vụ của chúng ta là phải triển khai đồng thời cả 2 dịch vụ này lên cùng một máy chủ VPS. Lúc này, nếu làm theo phương pháp truyền thống, các bạn sẽ đối mặt với 2 phương án đầy "đau đớn":
- **Phương án 1 là Triển khai trực tiếp (Native Run):** Các bạn phải cài song song cả JDK 17 và JDK 21 trên cùng một hệ điều hành. Sau đó phải cấu hình thủ công các biến môi trường `JAVA_HOME` riêng biệt cho từng tiến trình, rồi gọi dịch vụ bằng các đường dẫn tuyệt đối cực kỳ phức tạp. Chỉ cần một đợt cập nhật hệ điều hành hoặc cấu hình nhầm biến môi trường, một trong hai dịch vụ sẽ bị lỗi crash phiên bản Class (`UnsupportedClassVersionError`) ngay lập tức.
- **Phương án 2 là Sử dụng Máy ảo (Virtual Machine - VM):** Để cô lập hoàn toàn môi trường, các bạn tạo ra 2 máy ảo riêng biệt (VM 1 chạy JDK 17 để chạy `user-service`, VM 2 chạy JDK 21 để chạy `restaurant-service`). Phương án này giải quyết được xung đột Java, nhưng lại cực kỳ lãng phí. Các bạn phải mất thêm khoảng 2GB RAM chỉ để chạy 2 Hệ điều hành khách (Guest OS) nền. Máy chủ VPS của các bạn sẽ nhanh chóng cạn kiệt tài nguyên.

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
Thế thì sau khi giải quyết bài toán chạy song song các phiên bản Java trên một server, chúng ta lại đối mặt với một vấn đề lớn hơn: **Làm thế nào để chúng ta đóng gói toàn bộ môi trường chạy phức tạp đó và chuyển giao nó từ máy cá nhân lên server một cách đồng nhất?**

Nếu chúng ta chỉ copy file JAR trần trụi lên server bằng lệnh `scp`, chúng ta vẫn bị dính lỗi lệch cấu hình hệ thống (như sai phiên bản Java của OS server, hoặc lệch múi giờ hệ điều hành), khiến ứng dụng sập hoặc log sai lệch giờ. Câu nói kinh điển *"Ơ kìa, code chạy ngon trên máy em mà!"* (It works on my machine!) lại tiếp tục vang lên.

Để giải quyết triệt để vấn đề này, Docker cung cấp bộ ba thực thể: **Docker Image**, **Docker Container**, và **Docker Registry**.

- **Docker Image (Bản thiết kế bất biến - Class trong OOP):** Là một khuôn mẫu đóng gói ở trạng thái chỉ đọc (Read-only) chứa toàn bộ mã nguồn, runtime Java (JRE), biến môi trường mặc định và OS tối giản để chạy ứng dụng.
  - **Tính bất biến (Immutable):** Image đã build thành công không thể chỉnh sửa. Mọi thay đổi yêu cầu build một Image mới.
  - **Cấu trúc phân lớp (Layers):** Gồm nhiều lớp file system xếp chồng lên nhau (Ubuntu -> JRE -> code file JAR). Docker tự động tái sử dụng các layer có sẵn giữa các image để tiết kiệm dung lượng đĩa.
  - **Build từ Dockerfile:** Quy trình build Image được định nghĩa tự động qua mã nguồn cấu hình `Dockerfile`.
- **Docker Container (Thực thể sống động - Object trong OOP):** Là một thực thể tiến trình được khởi chạy từ Docker Image, hoạt động trên RAM và CPU. Khi khởi chạy, Docker Engine phủ thêm một lớp ghi tạm thời **Writable Layer (Container Layer)** lên trên Image chỉ đọc. Mọi dữ liệu phát sinh (log, file tạm) chỉ ghi vào lớp này và sẽ mất khi xóa container (`docker rm`), bảo toàn Image gốc bên dưới.
- **Docker Registry (Nhà kho lưu trữ):** Kho lưu trữ tập trung dùng để quản lý và chia sẻ các Docker Image. Gồm Public Registry (như Docker Hub chứa image mẫu của Postgres, OpenJDK) và Private Registry của doanh nghiệp để lưu trữ bảo mật mã nguồn đóng.

### 2. Phần thực hành (Demo)

**[Live Demo: Vòng đời tải và chạy Image]**
Bây giờ, thầy sẽ minh họa cho các bạn thấy cách tải một bản phân phối Java JRE chính thức từ Docker Hub về máy cá nhân và chạy nó.

*(Hướng dẫn thao tác trên Terminal)*:
- *Đầu tiên, thầy tải (pull) image JRE 17 gọn nhẹ (dùng alpine linux) từ Docker Hub về:*
  ```bash
  docker pull eclipse-temurin:17-jre-alpine
  ```
- *Tiếp theo, thầy kiểm tra danh sách các Image đang lưu ở bộ nhớ máy local bằng lệnh:*
  ```bash
  docker images
  ```
  *Các bạn thấy dòng `eclipse-temurin` với tag `17-jre-alpine` hiển thị với kích thước cực kỳ nhẹ, chỉ khoảng 100MB.*
- *Bây giờ, thầy sẽ khởi chạy thử một container kiểm tra phiên bản Java bên trong:*
  ```bash
  docker run --rm eclipse-temurin:17-jre-alpine java -version
  ```
  *Hệ thống hiển thị phiên bản Java 17 thành công và tự động xóa container ngay khi chạy xong nhờ cờ `--rm`. Rất sạch sẽ và an toàn.*

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
Thầy kể cho các bạn nghe một câu chuyện thực tế tại dự án QuickBite thế này:
Anh Tech Lead giao cho một bạn Intern một nhiệm vụ: *"Chạy một container PostgreSQL để làm database tạm thời, sao cho ứng dụng `user-service` đang chạy ở máy host kết nối vào được để ghi dữ liệu"*.

Bạn Intern gõ lệnh run mặc định và lập tức rơi vào hai rắc rối:
1. **Bị khóa cứng Terminal:** Log khởi động của Postgres chiếm trọn màn hình. Bạn ấy bấm `Ctrl+C` để thoát ra, thế là container database cũng bị tắt ngóm luôn.
2. **Không thể kết nối:** Sau khi chạy lại được, database báo đã khởi động xong. Tuy nhiên, ứng dụng `user-service` ở máy host kết nối vào cổng `5432` thì liên tục báo lỗi `Connection Refused`.

Để giải quyết triệt để vấn đề này, chúng ta cần làm chủ **4 tham số sống còn khi chạy Container**:
- **`-d` (Detached mode):** Chạy container dưới nền (background), giải phóng terminal.
- **`-p host_port:container_port` (Port Mapping):** Ánh xạ cổng mạng máy host vào cổng container. Ví dụ: `-p 5432:5432` giúp chuyển hướng yêu cầu truy cập cổng `5432` của máy host vào database trong container.
- **`-e KEY=VALUE` (Environment Variable):** Truyền biến môi trường vào container (ví dụ: `-e POSTGRES_PASSWORD=secret` để đặt mật khẩu).
- **`--name`:** Đặt tên định danh dễ quản lý (ví dụ: `quickbite-db`).

**[Slide: Vòng đời của Container]**
Các bạn lưu ý, tiến trình của container trải qua 4 trạng thái chính:
`Created (Đã tạo) ──► Running (Đang chạy) ──► Stopped (Đã dừng) ──► Destroyed (Đã xóa vĩnh viễn)`
Lưu ý quan trọng: Lệnh `docker stop` chỉ tạm dừng tiến trình hệ thống, giống như bạn tắt một ứng dụng trên máy. Toàn bộ dữ liệu ghi trên lớp Writable Layer của container (ví dụ: các bảng dữ liệu database đã tạo) vẫn nằm an toàn trên ổ đĩa cho đến khi bạn chạy lệnh `docker rm` để xóa bỏ hoàn toàn container khỏi hệ thống.

### 2. Phần thực hành (Demo)

**[Live Demo: Vòng đời hoạt động của container PostgreSQL]**
Bây giờ, thầy sẽ demo lần lượt quy trình quản lý một container database `quickbite-db` bằng dòng lệnh.

*(Hướng dẫn thao tác trên Terminal)*:
- *Bước 1: Khởi chạy container PostgreSQL chạy ngầm, mở cổng 5432, đặt tên rõ ràng:*
  ```bash
  docker run -d --name quickbite-db -p 5432:5432 -e POSTGRES_PASSWORD=secret postgres:15-alpine
  ```
- *Bước 2: Kiểm tra danh sách container đang hoạt động:*
  ```bash
  docker ps
  ```
  *Tiến trình Postgres đang chạy ở chế độ nền và cổng 5432 đã được ánh xạ thành công.*
- *Bước 3: Dừng container database:*
  ```bash
  docker stop quickbite-db
  ```
- *Bước 4: Liệt kê tất cả các container, bao gồm cả container đã dừng:*
  ```bash
  docker ps -a
  ```
  *Trạng thái lúc này hiển thị Exited (0) nhưng cấu hình và dữ liệu vẫn còn đó.*
- *Bước 5: Khởi động lại container cũ đã dừng mà không cần tạo mới:*
  ```bash
  docker start quickbite-db
  ```
- *Bước 6: Dừng và xóa bỏ hoàn toàn container khỏi hệ thống:*
  ```bash
  docker stop quickbite-db
  docker rm quickbite-db
  ```
  *Lúc này container mới thực sự bị xóa sạch khỏi đĩa máy host.*

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
