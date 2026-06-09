# SESSION 05: MULTI-SERVICES & API GATEWAY

## LESSON 02: Quy trình chạy Spring Boot Service cùng cơ sở dữ liệu bằng Docker Compose

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Cấu hình** thành công tệp cấu hình `application.yml` động trong Spring Boot sử dụng cơ chế nạp biến môi trường ngoại vi.
* **Tối ưu hóa tài nguyên** bằng cách triển khai nhiều cơ sở dữ liệu logic (Database Partitioning) trên duy nhất một container PostgreSQL vật lý chạy trong Docker Compose.
* **Xây dựng và ánh xạ** biến môi trường kết nối cơ sở dữ liệu chính xác cho các dịch vụ trong cụm container.
* **Tự động hóa khởi tạo cơ sở dữ liệu** khi container Postgres khởi động thông qua thư mục đặc biệt `/docker-entrypoint-initdb.d/`.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (NỖI ĐAU CỦA VIỆC TIÊU TỐN TÀI NGUYÊN VÀ HARDCODE CẤU HÌNH)

Hãy tưởng tượng bạn đang xây dựng môi trường chạy thử nghiệm local cho 4 dịch vụ Spring Boot của QuickBite. Nếu mỗi dịch vụ sử dụng một hệ quản trị cơ sở dữ liệu độc lập chạy trong 4 container PostgreSQL riêng biệt, bạn sẽ phải đối mặt với hai vấn đề lớn:

1. **Khủng hoảng tài nguyên (RAM/CPU):**
   * *Nỗi đau:* Mỗi container PostgreSQL trống tiêu tốn trung bình từ 150MB đến 200MB RAM. Việc khởi chạy 4 container database độc lập sẽ lãng phí gần 1GB RAM trên máy tính của bạn trước khi kịp khởi động bất kỳ dịch vụ Spring Boot nào. Đối với máy tính cá nhân của sinh viên có cấu hình trung bình, đây là một gánh nặng cực kỳ lớn làm chậm trễ quá trình debug.
2. **Nỗi đau Hardcode cấu hình:**
   * *Nỗi đau:* Lập trình viên viết cứng URL kết nối cơ sở dữ liệu là `jdbc:postgresql://localhost:5432/quickbite_user` trong file `application.yml` khi code ở máy local. Khi đóng gói chạy trong Docker Network, ứng dụng sẽ sập ngay vì địa chỉ `localhost` lúc này trỏ về chính bên trong container của ứng dụng chứ không phải container database. Việc phải liên tục sửa code và rebuild file JAR mỗi khi đổi môi trường triển khai đi ngược lại triết lý DevOps.

*Để giải quyết bài toán này, chúng ta sẽ cấu hình Spring Boot nhận biến môi trường động để linh hoạt chuyển đổi môi trường, đồng thời gom 4 database logic vào chạy trên duy nhất 1 container PostgreSQL duy nhất nhưng vẫn đảm bảo nguyên tắc cô lập dữ liệu.*

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (NỘI SUY BIẾN VÀ KHỞI TẠO CƠ SỞ DỮ LIỆU)

#### 3.1 Cơ chế nội suy biến môi trường của Spring Boot
Spring Boot hỗ trợ viết cấu hình dạng `${TÊN_BIẾN:Giá_trị_mặc_định}` trong file `application.yml`.
* Ví dụ: `url: jdbc:postgresql://${DB_HOST:localhost}:${DB_PORT:5432}/${DB_NAME:quickbite_user}`
* *Cách hoạt động:* 
  * Khi chạy trực tiếp ứng dụng trên IDE (IntelliJ), nếu không khai báo biến môi trường nào, Spring Boot sẽ tự lấy giá trị mặc định sau dấu hai chấm để chạy (`localhost`, `5432`).
  * Khi đóng gói ứng dụng chạy trong Docker Compose, chúng ta chỉ việc truyền các biến môi trường như `DB_HOST=quickbite-db`, ứng dụng sẽ tự động phân giải URL kết nối sang tên container của database mà không cần thay đổi bất kỳ dòng code nào.

#### 3.2 Khởi tạo đa cơ sở dữ liệu trong Container PostgreSQL
Mặc định, image PostgreSQL của Docker chỉ khởi tạo duy nhất một database được khai báo qua biến `POSTGRES_DB` lúc dựng container. 
* Để tạo thêm nhiều database khác (phục vụ 4 services của QuickBite), chúng ta có thể sử dụng cơ chế `/docker-entrypoint-initdb.d/` của Postgres.
* Bất kỳ file script nào có phần mở rộng `.sql` hoặc `.sh` được đưa vào thư mục `/docker-entrypoint-initdb.d/` bên trong container sẽ tự động được Postgres thực thi ngay trong lần đầu tiên container khởi tạo.

---

### PHẦN 4. DEMO VÀ THỰC HÀNH: THIẾT LẬP CẤU HÌNH VÀ KHỞI CHẠY HỆ THỐNG

#### 4.1 Cấu hình file `application.yml` động cho các Dịch vụ

Hãy cập nhật tệp `application.yml` cho từng dịch vụ như sau:

##### A. Đối với `user-service` (cổng chạy 8081):
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

##### B. Đối với `restaurant-service` (cổng chạy 8082):
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

##### C. Đối với `order-service` (cổng chạy 8083):
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

# Cấu hình gọi chéo API sang restaurant-service
services:
  restaurant-service:
    url: http://${RESTAURANT_SVC_HOST:localhost}:${RESTAURANT_SVC_PORT:8082}
```

##### D. Đối với `notification-service` (cổng chạy 8084):
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

#### 4.2 Thiết lập File Script khởi tạo dữ liệu
Tạo một thư mục con tên là `init-scripts` cùng cấp với file compose của bạn. Bên trong đó, tạo file `init-db.sql` có nội dung như sau để tự động tạo 4 database logic:

```sql
-- init-scripts/init-db.sql
CREATE DATABASE quickbite_user;
CREATE DATABASE quickbite_restaurant;
CREATE DATABASE quickbite_order;
CREATE DATABASE quickbite_notification;
```

#### 4.3 Khai báo Cấu hình trong `docker-compose.yml`
Dưới đây là một phần kịch bản `docker-compose.yml` cấu hình cho container database and dịch vụ `user-service` để minh họa cách mount script và nạp biến môi trường:

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
      # Mount script khởi tạo database logic vào thư mục đặc trưng của Postgres
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
      - DB_NAME=quickbite_user
      - DB_USER=postgres
      - DB_PASS=secret
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

---

### PHẦN 5. HIỂU LẦM THƯỜNG GẶP (CƠ CHẾ DDL-AUTO)

* **Hiểu lầm thường gặp:** Khi khai báo `ddl-auto: update` trong cấu hình Spring Boot, Hibernate sẽ tự động kết nối và tạo cơ sở dữ liệu vật lý (như `quickbite_user`) trên máy chủ Postgres nếu cơ sở dữ liệu đó chưa tồn tại.
* **Sự thật:** Cấu hình `ddl-auto` chỉ có tác dụng tạo hoặc cập nhật **các bảng (tables), chỉ mục (indexes), và ràng buộc** bên trong một cơ sở dữ liệu logic **đã được khởi tạo thành công**. Nếu cơ sở dữ liệu logic (`quickbite_user`) chưa hề được tạo trên hệ quản trị cơ sở dữ liệu Postgres, kết nối JDBC sẽ thất bại ngay lập tức và ném ra ngoại lệ `Connection refused` hoặc `Database does not exist`. Đó là lý do tại sao chúng ta bắt buộc phải có file script `init-db.sql` chạy ngầm từ phía Postgres trước.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG (XÁC MINH KIẾN THỨC)

1. **Cách cấu hình động trong Spring Boot:**
   * [Spring Boot Externalized Configuration - Reference Guide](https://docs.google.com/url?q=https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config)
2. **Hướng dẫn khởi tạo PostgreSQL container:**
   * [PostgreSQL Docker Hub Official Documentation](https://docs.google.com/url?q=https://hub.docker.com/_/postgres)
3. **Cú pháp biến môi trường trong Docker Compose:**
   * [Environment variables in Docker Compose - Docker Docs](https://docs.google.com/url?q=https://docs.docker.com/compose/environment-variables/)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Nếu khởi động hệ thống mà bạn quên không mount file `init-db.sql` vào thư mục `/docker-entrypoint-initdb.d/` của Postgres container, điều gì sẽ xảy ra khi container `quickbite-user` khởi động và cố kết nối với database?
* *Gợi ý:* Postgres container sẽ chỉ tạo database mặc định (thường là `postgres`). Khi `quickbite-user` khởi động, JDBC driver sẽ ném ra ngoại lệ không tìm thấy cơ sở dữ liệu logic có tên là `quickbite_user` và ứng dụng Spring Boot sẽ bị crash sập ngay lập tức.

#### Câu 2 (Đọc và dự đoán)
Nhìn vào cấu hình file `application.yml` của `order-service` ở Phần 4:
`url: http://${RESTAURANT_SVC_HOST:localhost}:${RESTAURANT_SVC_PORT:8082}`
If chúng ta chạy dịch vụ `order-service` trực tiếp bằng IDE trên máy tính cá nhân (không chạy qua Docker) và không thiết lập bất kỳ biến môi trường nào, endpoint mà nó gọi sang `restaurant-service` sẽ có giá trị là gì?
* *Gợi ý:* Nó sẽ sử dụng các giá trị mặc định được định nghĩa sau dấu hai chấm, tức là: `http://localhost:8082`.

#### Câu 3 (Xử lý tình huống)
Hệ thống QuickBite đang chạy ổn định trên Docker Compose. Bây giờ bạn muốn đổi mật khẩu truy cập database của toàn bộ hệ thống từ `secret` thành `quickbite2026`. Để làm điều này một cách chuẩn DevOps mà không cần build lại bất kỳ file JAR nào của các service, bạn cần cập nhật cấu hình ở những file nào?
* *Gợi ý:* Bạn cần cập nhật biến mật khẩu trong cấu hình của database Postgres (`POSTGRES_PASSWORD`) và đổi giá trị biến môi trường `DB_PASS` nạp vào tất cả 4 service Spring Boot trực tiếp trong file `docker-compose.yml` (hoặc file `.env` chứa biến ngoại vi).
