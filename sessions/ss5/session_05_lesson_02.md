# SESSION 05: MULTI-SERVICES & API GATEWAY

## LESSON 02: Quy trình chạy Spring Boot Service cùng cơ sở dữ liệu bằng Docker Compose

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Cấu hình** tệp cấu hình `application.yml` động trong Spring Boot sử dụng cơ chế nạp biến môi trường ngoại vi để tránh viết cứng thông tin kết nối.
* **Tối ưu hóa tài nguyên** bằng cách triển khai nhiều cơ sở dữ liệu logic (Database Partitioning) trên duy nhất một container PostgreSQL vật lý chạy trong Docker Compose.
* **Tự động hóa khởi tạo cơ sở dữ liệu và phân quyền người dùng** khi container Postgres khởi động thông qua việc mount file script SQL vào thư mục đặc biệt `/docker-entrypoint-initdb.d/`.
* **Biên soạn tệp Docker Compose** để khởi chạy đồng thời nhiều backend service phụ thuộc chung vào một container cơ sở dữ liệu (`depends_on`).

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
                                      │ (Kết nối qua dải IP ảo nội bộ)
                                      ▼
                          ┌────────────────────────┐
                          │      quickbite-db      │
                          │      (PostgreSQL)      │
                          └────────────────────────┘
```

Để tối ưu hóa tài nguyên cho máy tính cá nhân của bạn, thay vì khởi chạy 4 container database PostgreSQL độc lập, chúng ta chỉ cần chạy **duy nhất 1 container PostgreSQL** vật lý. Sau đó, chúng ta sẽ tạo các cơ sở dữ liệu logic riêng cho từng dịch vụ và phân quyền sở hữu tương ứng.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (NỘI SUY BIẾN VÀ KHỞI TẠO DB TỰ ĐỘNG)

#### 3.1 Cấu hình biến môi trường động trong Spring Boot
Spring Boot hỗ trợ cú pháp `${TÊN_BIẾN:Giá_trị_mặc_định}` trong file `application.yml`:
* Ví dụ: `url: jdbc:postgresql://${DB_HOST:localhost}:${DB_PORT:5432}/${DB_NAME:quickbite_user_db}`
* Khi chạy trực tiếp ở IDE, Spring Boot sử dụng giá trị mặc định là `localhost`.
* Khi chạy trong Docker, Docker Compose sẽ nạp các biến môi trường như `DB_HOST=quickbite-db`, ứng dụng sẽ tự động định tuyến đến container database mà không cần chỉnh sửa mã nguồn.

#### 3.2 Tự động tạo cơ sở dữ liệu và phân quyền sở hữu
Image PostgreSQL chính thức hỗ trợ thư mục đặc biệt `/docker-entrypoint-initdb.d/`. Khi container database khởi động lần đầu tiên, nó sẽ quét thư mục này và tự động thực thi các tệp tin script `.sql` hoặc `.sh` được mount vào đây.
Chúng ta sẽ viết một tệp tin SQL để tạo người dùng (users) và cơ sở dữ liệu (databases) riêng cho từng dịch vụ:
* `quickbite_user`: User sở hữu database `quickbite_user_db` (dùng cho `user-service`).
* `quickbite_restaurant`: User sở hữu database `quickbite_restaurant_db` (dùng cho `restaurant-service`).

---

### PHẦN 4. HƯỚNG DẪN THỰC HÀNH TRIỂN KHAI

Trong phần này, chúng ta sẽ thực hành cấu hình và khởi chạy đồng thời **2 dịch vụ: `user-service` (Java 17) và `restaurant-service` (Java 21)** kết nối chung tới container database `quickbite-db`.

#### 4.1 Biên soạn File script SQL khởi tạo cơ sở dữ liệu
Tạo thư mục `init-scripts/` nằm cạnh file compose. Tạo tệp tin `init-db.sql` bên trong chứa các lệnh SQL để tạo người dùng và cơ sở dữ liệu:

```sql
-- init-scripts/init-db.sql

-- Tạo User và Database cho user-service
CREATE USER quickbite_user WITH PASSWORD 'quickbite_user';
CREATE DATABASE quickbite_user_db OWNER quickbite_user;

-- Tạo User và Database cho restaurant-service
CREATE USER quickbite_restaurant WITH PASSWORD 'quickbite_restaurant';
CREATE DATABASE quickbite_restaurant_db OWNER quickbite_restaurant;
```

#### 4.2 Cấu hình tệp `application.yml` động cho các Dịch vụ

* **Dịch vụ `user-service` (Cổng 8081):**
```yaml
server:
  port: 8081
spring:
  datasource:
    url: jdbc:postgresql://${DB_HOST:localhost}:${DB_PORT:5432}/${DB_NAME:quickbite_user_db}
    username: ${DB_USERNAME:quickbite_user}
    password: ${DB_PASSWORD:quickbite_user}
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```

* **Dịch vụ `restaurant-service` (Cổng 8082):**
```yaml
server:
  port: 8082
spring:
  datasource:
    url: jdbc:postgresql://${DB_HOST:localhost}:${DB_PORT:5432}/${DB_NAME:quickbite_restaurant_db}
    username: ${DB_USERNAME:quickbite_restaurant}
    password: ${DB_PASSWORD:quickbite_restaurant}
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```

#### 4.3 Biên soạn các tệp `Dockerfile` đóng gói dịch vụ

* **Dockerfile cho `user-service` (`./user-service/Dockerfile`):**
```dockerfile
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY build/libs/user-service.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

* **Dockerfile cho `restaurant-service` (`./restaurant-service/Dockerfile`):**
```dockerfile
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY build/libs/restaurant-service.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

#### 4.4 Khai báo Cấu hình tệp `docker-compose.yml`
Tạo tệp `docker-compose.yml` ở thư mục gốc của dự án chứa các service sau:

```yaml
version: '3.8'

services:
  quickbite-db:
    image: postgres:15-alpine
    container_name: quickbite-db
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_PASSWORD=secret
    volumes:
      # Mount script khởi tạo database và phân quyền
      - ./init-scripts/init-db.sql:/docker-entrypoint-initdb.d/init-db.sql
      - quickbite-db-data:/var/lib/postgresql/data
    networks:
      - quickbite-net

  quickbite-user:
    build:
      context: ./user-service
    container_name: quickbite-user
    ports:
      - "8081:8081"
    environment:
      - DB_HOST=quickbite-db
      - DB_PORT=5432
      - DB_NAME=quickbite_user_db
      - DB_USERNAME=quickbite_user
      - DB_PASSWORD=quickbite_user
    depends_on:
      - quickbite-db
    networks:
      - quickbite-net

  quickbite-restaurant:
    build:
      context: ./restaurant-service
    container_name: quickbite-restaurant
    ports:
      - "8082:8082"
    environment:
      - DB_HOST=quickbite-db
      - DB_PORT=5432
      - DB_NAME=quickbite_restaurant_db
      - DB_USERNAME=quickbite_restaurant
      - DB_PASSWORD=quickbite_restaurant
    depends_on:
      - quickbite-db
    networks:
      - quickbite-net

volumes:
  quickbite-db-data:

networks:
  quickbite-net:
    driver: bridge
```

#### 4.5 Khởi chạy và kiểm tra hệ thống
1. Biên dịch các tệp JAR trên máy host:
   ```bash
   # Đứng tại thư mục gốc của từng service
   cd user-service && ./gradlew bootJar
   cd ../restaurant-service && ./gradlew bootJar
   ```
2. Khởi chạy toàn bộ hệ thống bằng Docker Compose:
   ```bash
   # Đứng tại thư mục gốc chứa file docker-compose.yml
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
* **Sự thật:** Cấu hình `ddl-auto` chỉ có tác dụng tạo hoặc cập nhật **các bảng (tables), chỉ mục, và mối quan hệ** bên trong một cơ sở dữ liệu logic **đã được khởi tạo thành công**. Nếu cơ sở dữ liệu `quickbite_user_db` chưa được tạo trước trên Postgres server, driver JDBC sẽ ném lỗi kết nối và ứng dụng sẽ dừng lại. Đó là lý do tại sao chúng ta bắt buộc phải có file script `init-db.sql` chạy ngầm từ phía Postgres trước.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG (XÁC MINH KIẾN THỨC)

1. **Tổng quan về khởi tạo PostgreSQL container:**
   * [PostgreSQL Docker Hub Official Documentation](https://hub.docker.com/_/postgres)
2. **Cú pháp nạp biến môi trường của Docker Compose:**
   * [Environment variables in Docker Compose - Docker Docs](https://docs.docker.com/compose/environment-variables/)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao trong file `docker-compose.yml` ở Phần 4, chúng ta khai báo thuộc tính `depends_on: - quickbite-db` cho hai service backend, nhưng đôi khi backend khởi động vẫn gặp lỗi kết nối tới database lúc mới bật?
* *Gợi ý:* `depends_on` chỉ đảm bảo container `quickbite-db` **bắt đầu khởi chạy** trước backend, chứ nó không biết khi nào dịch vụ PostgreSQL bên trong container thực sự sẵn sàng nhận kết nối (thường Postgres mất từ 2 đến 5 giây để khởi động nhân hệ thống và chạy script init). Để xử lý triệt để, hệ thống cần cơ chế tự động thử lại (Retry) của Spring Boot hoặc viết script chờ.

#### Câu 2 (Xử lý tình huống)
Nếu bạn thay đổi cổng mặc định của ứng dụng `user-service` trong file `application.yml` từ `8081` thành `8085`. Bạn sẽ phải thay đổi cấu hình cổng như thế nào trong tệp `docker-compose.yml` để người dùng bên ngoài máy host vẫn truy cập được dịch vụ qua cổng `8081`?
* *Gợi ý:* Bạn cần thay đổi ánh xạ trong thuộc tính `ports` của `quickbite-user` thành `"8081:8085"`. Lúc này, cổng bên ngoài máy host (bên trái) giữ nguyên là `8081`, còn cổng container nội bộ (bên phải) được chuyển hướng chính xác đến cổng `8085` của ứng dụng Spring Boot.
