# SESSION 05: MULTI-SERVICES & API GATEWAY

## LESSON 02: Quy trình chạy Spring Boot Service cùng cơ sở dữ liệu bằng Docker Compose

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Kế thừa kiến trúc** cơ sở dữ liệu tách biệt dùng chung đã thiết lập từ Session 4 để chạy nhiều backend service đồng thời.
* **Cấu hình biến môi trường tiền tố (`USER_`, `RESTAURANT_`)** trong file `.env` chung để tránh xung đột cấu hình.
* **Biên soạn tệp Docker Compose** để khởi chạy đồng thời nhiều backend service (`user-service` và `restaurant-service`) kết nối tới cùng một container database qua mạng ngoài (`external: true`).

---

### PHẦN 2. TỔNG QUAN HỆ THỐNG DỰ ÁN (VISUALIZATION)

Trong học phần này, mã nguồn chi tiết của 4 dịch vụ Spring Boot đã được Tech Lead chuẩn bị sẵn. Sinh viên không cần viết mã nguồn Java từ đầu, nhiệm vụ của bạn là clone dự án về máy local và cấu hình hệ thống chạy đa container bằng Docker Compose.

Dưới đây là sơ đồ luồng hoạt động và liên kết cổng mạng của hệ thống QuickBite:

```text
                          ┌────────────────────────┐
                          │   Client / Browser     │
                          └───────────┬────────────┘
                                      │ (Cổng 80 - Nginx)
                                      ▼
                          ┌────────────────────────┐
                          │   API Gateway Layer    │
                          └───────────┬────────────┘
                                      │ (Định tuyến)
             ┌────────────────────────┼────────────────────────┐
             ▼ (Cổng 8081)            ▼ (Cổng 8082)            ▼ (Cổng 8083)
     ┌───────────────┐        ┌───────────────┐        ┌───────────────┐
     │ user-service  │        │restaurant-svc │        │ order-service │
     │ (Quản lý User)│        │(Quản lý Quán) │        │ (Quản lý Đơn) │
     └───────┬───────┘        └───────┬───────┘        └───────┬───────┘
             │                        │                        │
             └────────────────────────┼────────────────────────┘
                                      │ (Kết nối qua mạng ảo quickbite-net)
                                      ▼
                          ┌────────────────────────┐
                          │      quickbite-db      │
                          │   (Postgres ngầm)      │
                          └────────────────────────┘
```

Để tối ưu hóa tài nguyên cho máy tính cá nhân của bạn, chúng ta kế thừa container PostgreSQL chạy độc lập đã thiết lập ở **Session 4 Lesson 1** để chạy chung các database logic riêng cho từng dịch vụ.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (KẾ THỪA CƠ SỞ DỮ LIỆU DÙNG CHUNG & ĐA DỊCH VỤ)

#### 3.1 Cơ chế nạp biến môi trường động cho nhiều dịch vụ
Như đã phân tích ở **Session 4 Lesson 4**, việc sử dụng biến môi trường giúp Spring Boot thay đổi tham số cấu hình linh hoạt. Trong kiến trúc nhiều dịch vụ (Multi-services), ta cần lưu ý:
* Mỗi dịch vụ (ví dụ: `user-service`, `restaurant-service`) kết nối tới một database logic riêng biệt (như `quickbite_user_db` và `quickbite_restaurant_db`).
* Để tránh việc các key cấu hình bị trùng lặp và ghi đè lẫn nhau trong tệp `.env` dùng chung của dự án, các biến môi trường riêng của từng service phải được tiền tố hóa bằng tên service (ví dụ: `USER_DB_NAME` và `RESTAURANT_DB_NAME`), sau đó ánh xạ vào các biến môi trường Spring Boot yêu cầu trong file `docker-compose.yml`.

#### 3.2 Khởi tạo và Phân quyền Database logic
Cơ chế tự động chạy file `init-db.sql` của Postgres đã được học ở **Session 4 Lesson 1 (Mục 5.2)**. File `init-db.sql` chạy duy nhất một lần khi dựng database stack và đã khởi tạo sẵn database cùng tài khoản sở hữu cho tất cả các dịch vụ:
* `quickbite_user_db` (cho `user-service`).
* `quickbite_restaurant_db` (cho `restaurant-service`).
Vì vậy, khi khởi chạy các dịch vụ backend Spring Boot mới, ta không cần cấu hình lại hay chạy lại Postgres container nữa mà chỉ cần cắm các container backend vào mạng ảo dùng chung.

---

### PHẦN 4. HƯỚNG DẪN THỰC HÀNH TRIỂN KHAI

Chúng ta sẽ thực hành cấu hình và khởi chạy đồng thời **2 dịch vụ: `user-service` (Java 17 - Cổng 8081) và `restaurant-service` (Java 21 - Cổng 8082)** kết nối chung tới container database `quickbite-db` đã chạy ngầm từ trước thông qua mạng ảo `quickbite-net`.

#### 4.1 Chuẩn bị cấu trúc thư mục
Hãy đảm bảo dự án của bạn có cấu trúc thư mục như sau:
```text
Thư mục làm việc/
├── quickbite-database/              # Đã thiết lập ở Session 4 (đang chạy ngầm)
│   ├── docker-compose.yml           
│   └── init-db.sql                  
│
└── quickbite-project/               # Dự án microservices chính
    ├── docker-compose.yml           # Khởi chạy user-service và restaurant-service
    ├── .env                         # Lưu trữ cấu hình biến môi trường
    ├── user-service/
    │   ├── Dockerfile
    │   └── build/libs/user-service.jar
    └── restaurant-service/
        ├── Dockerfile
        └── build/libs/restaurant-service.jar
```

#### 4.2 Cấu hình tệp `.env` dùng chung cho các dịch vụ
Tạo file `.env` đặt tại `/quickbite-project/.env` để quản lý các biến môi trường cho cả 2 dịch vụ:
```env
# Database Common Configuration (Kết nối tới database đang chạy ngầm)
DB_HOST=quickbite-db
DB_PORT=5432

# User Service Configuration
USER_DB_NAME=quickbite_user_db
USER_DB_USERNAME=quickbite_user
USER_DB_PASSWORD=quickbite_user
USER_SERVER_PORT=8081

# Restaurant Service Configuration
RESTAURANT_DB_NAME=quickbite_restaurant_db
RESTAURANT_DB_USERNAME=quickbite_restaurant
RESTAURANT_DB_PASSWORD=quickbite_restaurant
RESTAURANT_SERVER_PORT=8082
```

#### 4.3 Cấu hình tệp `application.yml` động cho các Dịch vụ
* **Dịch vụ `user-service` (`./user-service/src/main/resources/application.yml`):**
```yaml
server:
  port: ${SERVER_PORT:8081}
spring:
  datasource:
    url: jdbc:postgresql://${DB_HOST:localhost}:${DB_PORT:5432}/${DB_NAME:quickbite_user_db}
    username: ${DB_USERNAME:quickbite_user}
    password: ${DB_PASSWORD:quickbite_user}
  jpa:
    hibernate:
      ddl-auto: update
```

* **Dịch vụ `restaurant-service` (`./restaurant-service/src/main/resources/application.yml`):**
```yaml
server:
  port: ${SERVER_PORT:8082}
spring:
  datasource:
    url: jdbc:postgresql://${DB_HOST:localhost}:${DB_PORT:5432}/${DB_NAME:quickbite_restaurant_db}
    username: ${DB_USERNAME:quickbite_restaurant}
    password: ${DB_PASSWORD:quickbite_restaurant}
  jpa:
    hibernate:
      ddl-auto: update
```

#### 4.4 Khai báo cấu hình `docker-compose.yml`
Tạo tệp `/quickbite-project/docker-compose.yml` định nghĩa các dịch vụ backend:
```yaml
version: '3.8'

services:
  quickbite-user:
    build:
      context: ./user-service
    container_name: quickbite-user
    ports:
      - "${USER_SERVER_PORT}:${USER_SERVER_PORT}"
    environment:
      - DB_HOST=${DB_HOST}
      - DB_PORT=${DB_PORT}
      - DB_NAME=${USER_DB_NAME}
      - DB_USERNAME=${USER_DB_USERNAME}
      - DB_PASSWORD=${USER_DB_PASSWORD}
      - SERVER_PORT=${USER_SERVER_PORT}
    networks:
      - quickbite-net

  quickbite-restaurant:
    build:
      context: ./restaurant-service
    container_name: quickbite-restaurant
    ports:
      - "${RESTAURANT_SERVER_PORT}:${RESTAURANT_SERVER_PORT}"
    environment:
      - DB_HOST=${DB_HOST}
      - DB_PORT=${DB_PORT}
      - DB_NAME=${RESTAURANT_DB_NAME}
      - DB_USERNAME=${RESTAURANT_DB_USERNAME}
      - DB_PASSWORD=${RESTAURANT_DB_PASSWORD}
      - SERVER_PORT=${RESTAURANT_SERVER_PORT}
    networks:
      - quickbite-net

networks:
  quickbite-net:
    external: true  # Tham chiếu tới mạng ảo chung có sẵn của database stack
```

#### 4.5 Khởi chạy và kiểm tra hệ thống
1. Biên dịch các tệp JAR trên máy host (đã học ở Session 2):
   ```bash
   cd user-service && ./gradlew bootJar
   cd ../restaurant-service && ./gradlew bootJar
   ```
2. Khởi chạy toàn bộ hệ thống bằng Docker Compose:
   ```bash
   # Đứng tại thư mục gốc /quickbite-project/ chứa file docker-compose.yml
   docker compose up -d --build
   ```
3. Xem logs khởi động của các dịch vụ để xác nhận kết nối thành công:
   ```bash
   docker compose logs -f quickbite-user
   docker compose logs -f quickbite-restaurant
   ```

---

### PHẦN 5. HIỂU LẦM THƯỜNG GẶP (CƠ CHẾ CẬP NHẬT DATABASE CỦA HIBERNATE)

* **Hiểu lầm thường gặp:** Khi cấu hình `ddl-auto: update`, Hibernate/JPA sẽ tự động kết nối và tạo cơ sở dữ liệu vật lý (như `quickbite_user_db`) trên Postgres server nếu database đó chưa tồn tại.
* **Sự thật:** Cấu hình `ddl-auto` chỉ có tác dụng tạo hoặc cập nhật **các bảng (tables), chỉ mục, và mối quan hệ** bên trong một cơ sở dữ liệu logic **đã được khởi tạo thành công**. Nếu cơ sở dữ liệu `quickbite_user_db` chưa được tạo trước trên Postgres server, driver JDBC sẽ ném lỗi kết nối và ứng dụng sẽ dừng lại. Đó là lý do tại sao chúng ta bắt buộc phải có file script `init-db.sql` chạy ngầm từ phía Postgres trước (đã chạy ở Session 4).

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG (XÁC MINH KIẾN THỨC)

1. **Tổng quan về khởi tạo PostgreSQL container:**
   * [PostgreSQL Docker Hub Official Documentation](https://hub.docker.com/_/postgres)
2. **Cú pháp nạp biến môi trường của Docker Compose:**
   * [Environment variables in Docker Compose - Docker Docs](https://docs.docker.com/compose/environment-variables/)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao trong file `docker-compose.yml` ở Phần 4, chúng ta không sử dụng thuộc tính `depends_on: - quickbite-db` cho hai service backend như các bài học trước?
* *Gợi ý:* Vì container database hiện nay đang chạy ở một stack độc lập bên ngoài (`quickbite-database/`). Từ khóa `depends_on` chỉ có tác dụng thiết lập thứ tự khởi chạy giữa các service nằm trong cùng một tệp `docker-compose.yml`. Đối với các tài nguyên bên ngoài, backend cần sử dụng cơ chế tự động kết nối lại (retry) hoặc kiểm tra sức khỏe của database trước khi chạy.

#### Câu 2 (Xử lý tình huống)
Nếu bạn thay đổi cổng mặc định của ứng dụng `user-service` trong file `application.yml` từ `8081` thành `8085`. Bạn sẽ phải thay đổi cấu hình cổng như thế nào trong tệp `.env` để người dùng bên ngoài máy host vẫn truy cập được dịch vụ qua cổng `8081`?
* *Gợi ý:* Bạn thay đổi `USER_SERVER_PORT=8081` trong file `.env`? Không, nếu cổng ứng dụng đổi thành `8085`, bạn cần ánh xạ cổng của service `quickbite-user` trong file `docker-compose.yml` từ `"${USER_SERVER_PORT}:${USER_SERVER_PORT}"` thành `"8081:8085"`.
