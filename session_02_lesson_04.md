# SESSION 02: GIỚI THIỆU DOCKER

## LESSON 04: Các lệnh Docker cơ bản trong vòng đời container

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Làm chủ** các câu lệnh quản lý vòng đời của một container: Tạo mới, khởi chạy, dừng lại, và xóa bỏ.
* **Cấu hình thành thạo** các tham số cờ mạng (`-d`, `-p`, `--name`) để chạy ngầm và mở cổng truy cập container từ máy host.
* **Phân biệt** sự khác biệt bản chất giữa trạng thái dừng hoạt động (Stopped) và trạng thái bị xóa bỏ hoàn toàn (Destroyed) của container.
* **Tra cứu** các tham số chạy container nâng cao thông qua tài liệu chính thống của Docker.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (CHẠY DATABASE TRONG CONTAINER NHƯ THẾ NÀO?)

Ở Lesson 3, chúng ta đã cài đặt thành công Docker Engine và khởi chạy container kiểm thử đầu tiên (`hello-world`). Tuy nhiên, container `hello-world` chỉ in ra một dòng chào mừng rồi tự động dừng lại lập tức.

Trong thực tế phát triển dự án QuickBite, các dịch vụ như Database PostgreSQL hay `user-service` cần chạy ngầm liên tục, sẵn sàng tiếp nhận các kết nối mạng. Giả sử bạn cần thiết lập một database PostgreSQL làm kho lưu trữ dữ liệu tạm thời cho dịch vụ `user-service`:
* Nếu bạn cài đặt PostgreSQL trực tiếp lên máy host theo cách truyền thống (tải file cài đặt về chạy), máy tính của bạn sẽ rất nhanh bị nặng do các tiến trình chạy ngầm của hệ thống, và cực kỳ dễ xung đột cổng mặc định `5432` nếu bạn đã lỡ cài một bản Postgres cũ trước đó.
* Bạn quyết định đưa PostgreSQL vào chạy trong container của Docker. Lúc này, bạn phải giải quyết được 3 vấn đề:
  1. Làm sao để database chạy ngầm độc lập bên dưới mà không chiếm quyền điều khiển và "khóa cứng" cửa sổ Terminal hiện tại của bạn?
  2. Làm sao để ứng dụng Spring Boot `user-service` đang chạy ở máy host có thể chọc được vào cổng `5432` nằm sâu bên trong vùng mạng cô lập của container?
  3. Làm sao để đặt một cái tên dễ nhớ (như `quickbite-db`) thay vì để Docker tự đặt tên ngẫu nhiên dài dòng khó quản lý?

> [!TIP]
> **Image Prompt gợi ý:**
> A structural flow diagram of a container's lifecycle. Four boxes represent the states: 1. "Created" (grey), 2. "Running" (glowing green, showing a database icon inside), 3. "Stopped" (orange, showing inactive process), 4. "Destroyed" (red, showing container dissolving). Arrows represent transition commands like `docker run`, `docker stop`, and `docker rm`. Modern flat technical vector style.

*Bài học này sẽ hướng dẫn bạn làm chủ bộ ba cờ tham số sống còn của Docker để thiết lập và kiểm soát vòng đời của container một cách chuyên nghiệp.*

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (VÒNG ĐỜI CONTAINER & CÁC THAM SỐ CẤU HÌNH)

#### 3.1 Vòng đời của một Container
Một Docker Container trải qua các trạng thái tuần tự sau trong vòng đời của nó:

```text
 ┌───────────┐   docker run   ┌───────────┐   docker stop   ┌───────────┐   docker rm   ┌───────────┐
 │  Created  │ ─────────────► │  Running  │ ──────────────► │  Stopped  │ ────────────► │ Destroyed │
 │ (Đã tạo)  │                │ (Đang chạy│                 │ (Đã dừng) │               │ (Đã xóa)  │
 └───────────┘                └───────────┘                 └───────────┘               └───────────┘
```

* **Created (Đã tạo):** Container được khởi tạo từ Image, các lớp cấu hình đã sẵn sàng nhưng tiến trình chính bên trong chưa thực sự chạy.
* **Running (Đang hoạt động):** Tiến trình chính của ứng dụng bên trong container được kích hoạt (ví dụ: PostgreSQL đang lắng nghe cổng mạng). Container tiêu thụ tài nguyên RAM/CPU của máy host.
* **Stopped / Exited (Đã dừng):** Tiến trình chính bên trong container bị tắt (do lệnh tắt hoặc do ứng dụng bị lỗi crash). Lúc này, container không còn tiêu thụ RAM và CPU nữa, nhưng toàn bộ dữ liệu ghi trên lớp ghi Writable Layer vẫn còn nguyên trên ổ đĩa của máy host.
* **Destroyed (Đã xóa):** Toàn bộ container cùng lớp ghi tạm thời Writable Layer bị xóa bỏ hoàn toàn khỏi hệ thống, giải phóng hoàn toàn dung lượng ổ cứng.

#### 3.2 Bộ ba cờ tham số sống còn (`-d`, `-p`, `--name`)
Để đưa một dịch vụ như PostgreSQL vào vận hành ổn định trong container, lệnh `docker run` cung cấp 3 tham số quan trọng:

1. **Chạy ngầm dưới nền - Detached Mode (`-d`):**
   * Mặc định, khi bạn chạy container, Docker sẽ gắn dòng dữ liệu ra (STDOUT) vào Terminal của bạn. Nếu bạn tắt Terminal hoặc gõ `Ctrl+C`, container sẽ sập.
   * Thêm cờ `-d` sẽ báo cho Docker Daemon chạy container độc lập dưới nền hệ thống, trả lại quyền nhập lệnh ngay lập tức cho Terminal của bạn.
2. **Ánh xạ cổng mạng - Port Mapping (`-p host_port:container_port`):**
   * Container chạy trong một phân vùng mạng cô lập. Nếu không cấu hình cờ này, không ai ở bên ngoài máy host có thể kết nối tới ứng dụng bên trong container.
   * Ví dụ: `-p 5432:5432` nghĩa là: *"Hễ có ai gửi request tới cổng `5432` của máy host, hãy chuyển tiếp toàn bộ dữ liệu đó vào cổng `5432` bên trong container"*.
3. **Đặt tên định danh - Naming (`--name`):**
   * Nếu không đặt tên, Docker sẽ tự sinh ra một tên ngẫu nhiên (ví dụ: `epic_spitzer`). Việc này cực kỳ khó cho các script tự động hóa.
   * Thêm `--name quickbite-db` giúp bạn dễ dàng chỉ định container này trong các câu lệnh quản trị tiếp theo.

---

### PHẦN 4. DEMO THỰC HÀNH: QUẢN LÝ VÒNG ĐỜI CONTAINER `quickbite-db`

Hãy mở Terminal trên môi trường Sandbox của bạn và thực hiện đầy đủ các bước quản lý vòng đời của database PostgreSQL dưới đây:

#### Bước 1: Khởi chạy container database chạy ngầm
Chúng ta sử dụng image PostgreSQL bản rút gọn (`alpine`) và truyền mật khẩu database thông qua biến môi trường `-e POSTGRES_PASSWORD`:
```bash
docker run -d --name quickbite-db -p 5432:5432 -e POSTGRES_PASSWORD=secret postgres:15-alpine
```
* **Kết quả:** Terminal trả về một chuỗi ký tự dài (đây chính là Container ID đầy đủ) và lập tức cho phép bạn gõ lệnh tiếp theo.

#### Bước 2: Liệt kê các container đang hoạt động
```bash
docker ps
```
* **Kết quả mong đợi:** Danh sách in ra một dòng chứa container tên là `quickbite-db`, đang chạy (`Up...`), và ánh xạ cổng `0.0.0.0:5432->5432/tcp`.

#### Bước 3: Dừng hoạt động của container database
Khi không cần chạy thử nghiệm database nữa, ta hạ lệnh dừng:
```bash
docker stop quickbite-db
```
* **Kết quả:** Tiến trình PostgreSQL bị tắt an toàn. Lệnh `docker ps` lúc này sẽ không còn hiển thị container này nữa.

#### Bước 4: Kiểm tra danh sách tất cả các container (kể cả đã tắt)
```bash
docker ps -a
```
* **Kết quả mong đợi:** Container `quickbite-db` xuất hiện lại trong danh sách nhưng cột Status ghi là `Exited (0) ...` (Đã dừng an toàn). Dữ liệu cấu hình của database vẫn được lưu trên đĩa cứng máy host.

#### Bước 5: Xóa bỏ hoàn toàn container khỏi hệ thống
Để dọn dẹp dung lượng ổ đĩa và giải phóng hoàn toàn tài nguyên:
```bash
docker rm quickbite-db
```
* **Kết quả:** Chạy lại `docker ps -a` sẽ thấy container `quickbite-db` đã biến mất hoàn toàn. Lúc này, bạn có thể tự do tạo một container mới trùng tên `quickbite-db` mà không bị lỗi xung đột.

---

### PHẦN 5. HIỂU LẦM THƯỜNG GẶP & RÀNG BUỘC KHI XÓA CONTAINER (STATUS BLOCK)

* **Hiểu lầm thường gặp:** Khi muốn xóa nhanh một container đang chạy, tôi chỉ cần gõ lệnh `docker rm [tên_container]` là xong.
* **Sự thật:** Docker Daemon sẽ chặn đứng hành động này và ném ra lỗi: 
  `Error response from daemon: You cannot remove a running container ... Stop the container before attempting removal`.
  - **Lý do:** Docker bảo vệ ứng dụng tránh việc bị mất dữ liệu bất ngờ khi tiến trình vẫn đang ghi dữ liệu vào ổ đĩa. 
  - **Cách xử lý đúng:** Bắt buộc phải gõ `docker stop` trước để tiến trình kết thúc an toàn, sau đó mới gõ `docker rm`. Chỉ trong trường hợp khẩn cấp hoặc viết script tự động hóa, bạn mới dùng cờ cưỡng chế `-f` (`docker rm -f [tên_container]`).

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG (XÁC MINH KIẾN THỨC)

Để kiểm chứng cơ chế quản lý vòng đời và các cờ tham số của container, bạn có thể tham khảo trực tiếp các tài liệu uy tín từ Docker:
1. **Đặc tả chi tiết các tham số của lệnh chạy container (`docker run`):**
   * [Docker Run Reference Guide - Docker Docs](https://docs.docker.com/reference/cli/docker/container/run/)
2. **Hướng dẫn cơ bản về vòng đời và hoạt động của Container:**
   * [Containers Basics - Docker Docs](https://docs.docker.com/get-started/docker-concepts/the-basics/#what-is-a-container)
3. **Cơ chế hoạt động của mạng và ánh xạ cổng trong Docker:**
   * [Docker Networking and Ports - Docker Docs](https://docs.docker.com/engine/network/)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Giải thích sự khác biệt lớn nhất về tài nguyên (RAM, CPU, Ổ đĩa) của container `quickbite-db` giữa hai trạng thái: sau khi chạy lệnh `docker stop` và sau khi chạy lệnh `docker rm`.
* *Gợi ý:*
  - Sau lệnh `docker stop`: Container giải phóng hoàn toàn tài nguyên CPU và RAM của máy host. Tuy nhiên, dữ liệu trên lớp ghi Writable Layer và file cấu hình của container vẫn bị chiếm dụng trên dung lượng ổ cứng (Ổ đĩa chưa được giải phóng).
  - Sau lệnh `docker rm`: Toàn bộ dữ liệu của container trên ổ cứng bị xóa bỏ sạch sẽ, ổ đĩa của máy host được giải phóng hoàn toàn.

#### Câu 2 (Đọc hiểu và dự đoán)
Nếu bạn khởi chạy database bằng câu lệnh sau:
`docker run -d --name quickbite-db postgres:15-alpine`
(Bạn quên không khai báo cờ `-p 5432:5432`). Hỏi ứng dụng Spring Boot `user-service` đang khởi chạy trực tiếp trên máy local (ngoài máy host) có thể kết nối được tới database PostgreSQL này để ghi dữ liệu hay không? Tại sao?
* *Gợi ý:* Hoàn toàn không kết nối được. Mặc dù database bên trong container vẫn khởi chạy thành công và lắng nghe cổng `5432` nội bộ, nhưng do thiếu cờ Port Mapping (`-p`), Docker Engine không mở cổng chuyển tiếp dữ liệu từ máy host vào container. Card mạng của máy host không thể tìm thấy đường dẫn kết nối vào bên trong container.

#### Câu 3 (Xử lý tình huống thực tế)
Bạn gõ lệnh khởi chạy container PostgreSQL để làm database và nhận được thông báo lỗi đỏ từ hệ thống:
`Conflict. The container name "/quickbite-db" is already in use by container "[chuỗi_ID]". You have to remove (or rename) that container to be able to reuse that name.`
Hãy nêu nguyên nhân gây ra lỗi này và trình bày 2 cách xử lý tình huống trên. Cách nào là an toàn nhất?
* *Gợi ý:* 
  - Nguyên nhân: Hệ thống đang tồn tại một container cũ (có thể đang chạy hoặc đã stop nhưng chưa xóa) trùng tên là `quickbite-db`.
  - Cách 1 (Nhanh nhưng rủi ro): Xóa container cũ đi bằng lệnh `docker rm quickbite-db` (nếu container đã dừng) rồi chạy lại lệnh. Cách này sẽ làm mất toàn bộ dữ liệu của container cũ đó.
  - Cách 2 (An toàn nhất): Đổi tên container mới khi khởi chạy thành một tên khác (ví dụ: `--name quickbite-db-v2`) hoặc kiểm tra xem container cũ có đang chứa dữ liệu quan trọng không trước khi quyết định xóa nó.
