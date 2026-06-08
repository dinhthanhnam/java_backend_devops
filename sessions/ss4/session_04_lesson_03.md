# SESSION 04: DOCKER COMPOSE CƠ BẢN

## LESSON 03: Biến môi trường và cấu hình port

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Cấu hình chính xác** ánh xạ cổng (`ports`) để mở cổng kết nối từ máy host vào trong container.
* **Vận dụng** cơ chế truyền biến môi trường (`environment`) vào container để thay đổi cấu hình runtime của ứng dụng Spring Boot linh hoạt.
* **Thiết lập và quản lý bảo mật** thông số cấu hình nhạy cảm (như mật khẩu database, token API) bằng tệp cấu hình ngoại vi `.env`.
* **Sử dụng lệnh kiểm tra** cấu hình Compose (`docker compose config`) trước khi khởi chạy hệ thống.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (HIỂM HỌA LỘ MẬT KHẨU TRÊN GIT)

Hãy tưởng tượng bạn đang viết tệp `docker-compose.yml` để triển khai hệ thống QuickBite. Bạn điền trực tiếp mật khẩu quản trị cơ sở dữ liệu vào file cấu hình:
```yaml
services:
  quickbite-db:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: "Super-Secret-DB-Password-123456"
```
Bạn commit file này và push lên kho lưu trữ mã nguồn chung của công ty (GitLab/GitHub). 

* **Hiểm họa xảy ra:** 
  * Sáng hôm sau, Tech Lead và đội an ninh thông tin (Security Team) phát hiện ra thông tin nhạy cảm bị công khai. Bất kỳ ai có quyền xem code đều đọc được mật khẩu database. Đây là lỗ hổng bảo mật nghiêm trọng trong DevOps gọi là **Hardcoded Credentials Leak**.
  * Hơn thế nữa, khi bạn mang file Compose này sang deploy ở môi trường khác (như Staging hay Production), bạn lại phải sửa trực tiếp file `docker-compose.yml` để thay đổi mật khẩu và cổng kết nối của từng môi trường. Việc này vi phạm hoàn toàn nguyên lý **"Build một lần, chạy mọi nơi"** (Build once, run anywhere).

*Để giải quyết triệt để vấn đề bảo mật và tính linh hoạt cấu hình, chúng ta cần phân tách mã nguồn và thông tin cấu hình nhạy cảm thông qua **Biến môi trường** và file **`.env`**.*

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (PORTS VS EXPOSE, BIẾN MÔI TRƯỜNG & TỆP .ENV)

#### 3.1 Cấu hình Port (`ports`)
Cú pháp ánh xạ cổng trong file Compose sử dụng định dạng:
```yaml
ports:
  - "HOST_PORT:CONTAINER_PORT"
```
* **Ý nghĩa:** Docker Engine sẽ lắng nghe cổng `HOST_PORT` trên máy host vật lý và chuyển hướng (NAT) toàn bộ lưu lượng truy cập vào cổng `CONTAINER_PORT` nội bộ của container.
* **Ví dụ:** `- "8081:8081"` mở cổng 8081 ra máy host để bạn truy cập được API Spring Boot qua `http://localhost:8081`.

#### 3.2 Biến môi trường (`environment`)
Thay vì viết cứng cấu hình trong file Java properties, Spring Boot cho phép nạp đè cấu hình thông qua các biến môi trường. Trong file Compose, ta truyền các biến này qua từ khóa `environment`:
```yaml
services:
  quickbite-user:
    image: eclipse-temurin:17-jre-alpine
    environment:
      - SPRING_DATASOURCE_USERNAME=postgres
      - SPRING_DATASOURCE_PASSWORD=secret
```

#### 3.3 Tách biệt cấu hình bảo mật bằng file `.env`
Để tránh lộ thông tin nhạy cảm trên Git, Docker Compose tự động tìm kiếm một file ẩn tên là `.env` nằm cùng thư mục với file `docker-compose.yml`. 

1. **Cú pháp trong file `.env`** (chứa các cặp Key=Value thông thường):
```env
DB_PASSWORD=Super-Secret-DB-Password-123456
USER_PORT=8081
```
2. **Cơ chế nội suy biến (Interpolation) trong `docker-compose.yml`:**
   Chúng ta sử dụng cú pháp `${TÊN_BIẾN}` để gọi giá trị từ file `.env` nạp vào file Compose:
```yaml
services:
  quickbite-db:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}

  quickbite-user:
    build: ./user-service
    ports:
      - "${USER_PORT}:8081"
```
3. **Quy tắc bảo mật quan trọng:**
   * File `.env` chứa mật khẩu thực tế **tuyệt đối không được commit lên Git** (bắt buộc phải khai báo trong file `.gitignore`).
   * Thay vào đó, bạn chỉ push file mẫu đặt tên là `.env.example` (chứa các key trống không có mật khẩu thật, ví dụ: `DB_PASSWORD=`) để làm tài liệu hướng dẫn cho các lập trình viên khác tự điền mật khẩu của riêng họ khi clone code về chạy.

---

### PHẦN 4. THỰC HÀNH: SỬ DỤNG FILE .ENV ĐỂ CẤU HÌNH HỆ THỐNG DOCKER COMPOSE

Hãy thực hành cấu hình bảo mật hệ thống QuickBite local:

1. Tạo file `.env` nằm ở thư mục dự án của bạn:
   ```env
   # Cấu hình Database
   DB_USER=postgres
   DB_PASSWORD=quickbite_db_secret_pass
   DB_PORT=5432

   # Cấu hình User Service
   USER_SERVICE_PORT=8081
   ```
2. Tạo file `.gitignore` để khóa file `.env` lại không cho đẩy lên Git:
   ```text
   .env
   /build/
   ```
3. Viết file `docker-compose.yml` sử dụng các biến nội suy:
```yaml
version: '3.8'

services:
  quickbite-db:
    image: postgres:15-alpine
    ports:
      - "${DB_PORT}:5432"
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}

  quickbite-user:
    build:
      context: ./user-service
    ports:
      - "${USER_SERVICE_PORT}:8081"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://quickbite-db:5432/postgres
      SPRING_DATASOURCE_USERNAME: ${DB_USER}
      SPRING_DATASOURCE_PASSWORD: ${DB_PASSWORD}
```
4. Chạy lệnh kiểm tra tính hợp lệ và hiển thị cấu hình sau khi nội suy biến:
```bash
docker compose config
```
5. **Kết quả mong đợi:** Lệnh `config` sẽ đọc file `.env`, thay thế toàn bộ ký tự `${...}` thành giá trị thực tế và in ra cấu hình hoàn chỉnh. Bạn sẽ thấy rõ mật khẩu `quickbite_db_secret_pass` và các cổng đã được điền chính xác.
6. Tiến hành khởi chạy cụm dịch vụ:
```bash
docker compose up -d
```

---

### PHẦN 5. HIỂU LẦM THƯỜNG GẶP (PORTS VS EXPOSE TRONG COMPOSE)

* **Hiểu sai:** Từ khóa `ports` và `expose` đều dùng để mở cổng dịch vụ của container ra ngoài máy host vật lý.
* **Đính chính:** Hoàn toàn khác nhau.
  * `ports`: Ánh xạ cổng ra ngoài máy host vật lý, cho phép bất kỳ ai từ bên ngoài (hoặc trình duyệt máy host) kết nối vào container.
  * `expose`: Chỉ mang tính chất khai báo cổng hoạt động nội bộ. Các container khác kết nối chung mạng Docker có thể gọi vào cổng này, nhưng **máy host vật lý hoàn toàn không thể kết nối trực tiếp** được vào cổng này.
  * **Lời khuyên bảo mật:** Đối với database PostgreSQL nội bộ (`quickbite-db`), chúng ta chỉ cần dùng `expose: - "5432"` (hoặc không cần khai báo nếu các container dùng chung mạng tùy biến), tuyệt đối không dùng `ports` để tránh nguy cơ hacker quét và tấn công trực tiếp vào database của bạn từ bên ngoài internet.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG (XÁC MINH KIẾN THỨC)

1. **Cách sử dụng biến môi trường trong Docker Compose:**
   * [Environment variables in Docker Compose - Docker Docs](https://docs.docker.com/compose/environment-variables/)
2. **Đặc tả cấu hình thuộc tính Ports và Expose:**
   * [Compose file ports reference - Docker Docs](https://docs.docker.com/compose/compose-file/05-services/#ports)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao khi chạy lệnh `docker compose config` chúng ta không cần phải truyền đường dẫn file `.env` mà Docker Compose vẫn tự động tìm thấy và nạp được dữ liệu?
* *Gợi ý:* Vì theo thiết kế mặc định, Docker Compose CLI sẽ tự động tìm kiếm một tệp tin có tên chính xác là `.env` nằm trong cùng thư mục nơi lệnh `docker compose` được thực thi. Nếu muốn sử dụng một file env có tên khác (ví dụ: `.env.production`), bạn phải sử dụng cờ truyền file tường minh là `docker compose --env-file .env.production config`.

#### Câu 2 (Xử lý tình huống)
Nếu bạn thay đổi giá trị cổng `USER_SERVICE_PORT=8082` trong file `.env` khi cụm container `quickbite-user` đang hoạt động, container có tự động chuyển đổi sang lắng nghe tại cổng `8082` ngay lập tức hay không? Bạn cần chạy lệnh gì để nhận cấu hình mới?
* *Gợi ý:* Container không tự động nhận cổng mới ngay lập tức vì cấu hình port mapping được thiết lập tĩnh khi khởi tạo container. Để áp dụng cấu hình mới, bạn cần chạy lại lệnh: `docker compose up -d`. Docker Compose sẽ tự động phân tích phát hiện cấu hình file `.env` đã thay đổi, tự động dừng container cũ và tạo mới container `quickbite-user` chạy ở cổng `8082` mà không cần restart các container khác không thay đổi cấu hình.
