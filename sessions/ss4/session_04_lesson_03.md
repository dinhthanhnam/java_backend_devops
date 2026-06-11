# SESSION 04: DOCKER COMPOSE CƠ BẢN

## LESSON 03: Cấu trúc file docker-compose.yml (services, image, build)

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Đọc và giải thích** cấu trúc định dạng tệp tin `docker-compose.yml` (các thành phần chính: `version`, `services`, `image`, `build`).
* **Biên soạn** thành thạo một tệp tin `Dockerfile` cơ bản để đóng gói ứng dụng Spring Boot.
* **Vận dụng thuộc tính `build`** trong Docker Compose để tự động hóa quy trình build image từ Dockerfile và khởi chạy container chỉ với một câu lệnh.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (NỖI ĐAU QUÊN BUILD CODE MỚI)

Hãy tưởng tượng bạn đang sửa đổi một tính năng trong mã nguồn của `user-service`. Sau khi sửa code xong, bạn chạy Gradle để compile ra file JAR mới tại `build/libs/user-service.jar`. 

Để cập nhật tính năng mới này chạy trong container trên máy local, bạn phải thực hiện thủ công 3 bước:
```bash
# 1. Tự chạy lệnh build để đóng gói image mới từ Dockerfile
docker build -t quickbite-user:v2 .

# 2. Xóa container cũ đang chạy bản code cũ
docker stop quickbite-user && docker rm quickbite-user

# 3. Chạy container mới từ image vừa build
docker run -d --name quickbite-user -p 8081:8081 quickbite-user:v2
```

* **Vấn đề rủi ro:** 
  * Quy trình này quá rườm rà. Lập trình viên rất dễ quên chạy bước 1 (quên build lại image mới) mà trực tiếp restart lại container. 
  * Kết quả là container khởi chạy thành công nhưng vẫn chạy trên phiên bản code cũ nằm trong image cũ. Bạn sẽ mất hàng giờ để debug mệt mỏi chỉ vì câu hỏi: *"Tại sao em đã sửa code trên máy rồi mà log lỗi của container vẫn hiển thị dòng lỗi cũ?"*

*Để tự động hóa hoàn toàn chuỗi thao tác: "Build code -> Đóng gói Image -> Tạo Container mới", Docker Compose cung cấp cơ chế tự build image thông qua từ khóa `build`.*

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (CẤU TRÚC docker-compose.yml & DOCKERFILE CƠ BẢN)

#### 3.1 Cấu trúc cơ bản của docker-compose.yml
Tệp cấu hình của Docker Compose được viết bằng định dạng YAML (phân biệt các khối dữ liệu bằng thụt lề khoảng trắng, tuyệt đối không dùng phím Tab). Các từ khóa nền tảng bao gồm:
* `version`: Chỉ định phiên bản định dạng của file Compose (phổ biến nhất hiện nay là `'3.8'`).
* `services`: Khối dữ liệu chứa định nghĩa của tất cả các container trong cụm dịch vụ.
* `image`: Tên Docker Image được lấy từ Docker Hub (ví dụ: `postgres:15-alpine`).

#### 3.2 Khai báo thuộc tính `build` trong Compose
Thay vì chỉ định cứng một image có sẵn trên mạng thông qua key `image`, chúng ta sử dụng key `build` trỏ tới Dockerfile mà chúng ta đã định nghĩa trước đó:
* `context`: Thư mục chứa mã nguồn của dịch vụ (nơi Docker tìm kiếm file Dockerfile và file JAR).
* `dockerfile`: Tên file Dockerfile (mặc định là `Dockerfile`, có thể bỏ qua nếu bạn đặt tên file đúng chuẩn).

Ví dụ cấu trúc khai báo dịch vụ tự build trong `docker-compose.yml`:
```yaml
services:
  quickbite-user:
    build:
      context: ./user-service
      dockerfile: Dockerfile
    ports:
      - "8081:8081"
```

---

### PHẦN 5. THIẾT LẬP DATABASE DÙNG CHUNG QUA DOCKER COMPOSE ĐỘC LẬP (CỐ ĐỊNH CHO TOÀN BỘ MÔN HỌC)

Trong hệ thống Microservices QuickBite, các dịch vụ sẽ phát triển dần theo thời gian. Để chuẩn bị môi trường lưu trữ lâu dài và nhất quán cho toàn bộ khóa học, chúng ta sẽ thiết lập **một container PostgreSQL dùng chung duy nhất** chạy độc lập.

#### 5.1 Tại sao tách biệt Database chạy độc lập thay vì Compose chung?

Thay vì khai báo chung Database (`quickbite-db`) và Application (`user-service`) vào cùng một file `docker-compose.yml`, việc tách biệt database ra chạy độc lập mang lại 4 lợi ích lớn:

1. **Tối ưu tốc độ phát triển (Development Loop):** Database khởi chạy rất nặng và chậm. Khi code Java thay đổi, dev cần rebuild và restart container backend liên tục. Tách biệt DB giúp backend có thể tắt/mở thoải mái mà không làm gián đoạn hoặc phải khởi động lại DB.
2. **Tiết kiệm RAM/CPU:** Hệ thống QuickBite gồm 4 microservices. Nếu mỗi service tự up 1 Postgres container riêng, máy local sẽ chạy 4 Postgres song song gây ngốn tài nguyên. Dựng 1 container Postgres chung chứa 4 database con là tối ưu nhất.
3. **An toàn dữ liệu (Stateless vs Stateful):** Backend là Stateless (không lưu trạng thái) nên xóa/dựng rất dễ dàng. Database là Stateful (giữ dữ liệu). Tách biệt giúp tránh việc sơ ý xóa sạch dữ liệu DB khi dọn dẹp backend bằng lệnh `docker compose down -v`.
4. **Chuẩn Production:** Trong thực tế, database luôn chạy độc lập trên các dịch vụ Managed Database (như AWS RDS, Cloud SQL) chứ không bao giờ chạy chung pod/host với container ứng dụng.

#### 5.2 File cấu hình và Script Khởi tạo Cơ sở dữ liệu tự động

Để tự động tạo đầy đủ 4 database biệt lập kèm tài khoản và phân quyền tương ứng cho từng dịch vụ khi database khởi chạy lần đầu, PostgreSQL hỗ trợ cơ chế quét và chạy tự động các script nằm trong thư mục `/docker-entrypoint-initdb.d/` bên trong container.

Hãy tạo thư mục đặt tên là `quickbite-database` nằm ngoài thư mục dự án Java và chuẩn bị 2 file cấu hình sau:

##### 1. File kịch bản khởi tạo SQL (`init-db.sql`)
Tệp này sẽ tạo các user và database biệt lập cho cả 4 dịch vụ, đồng thời tạo cả phiên bản database có hậu tố `_db` và không có hậu tố nhằm đảm bảo tương thích 100% với cấu hình của toàn bộ các Session trong môn học.

```sql
-- 1. Tạo các User riêng cho từng Service
CREATE USER quickbite_user WITH PASSWORD 'quickbite_user';
CREATE USER quickbite_restaurant WITH PASSWORD 'quickbite_restaurant';
CREATE USER quickbite_order WITH PASSWORD 'quickbite_order';
CREATE USER quickbite_notification WITH PASSWORD 'quickbite_notification';

-- 2. Tạo các Database tương ứng (Tạo cả hai dạng tên để tương thích mọi Session)
CREATE DATABASE quickbite_user_db OWNER quickbite_user;
CREATE DATABASE quickbite_user OWNER quickbite_user;

CREATE DATABASE quickbite_restaurant_db OWNER quickbite_restaurant;
CREATE DATABASE quickbite_restaurant OWNER quickbite_restaurant;

CREATE DATABASE quickbite_order_db OWNER quickbite_order;
CREATE DATABASE quickbite_order OWNER quickbite_order;

CREATE DATABASE quickbite_notification_db OWNER quickbite_notification;
CREATE DATABASE quickbite_notification OWNER quickbite_notification;

-- 3. Phân quyền truy cập toàn diện trên từng database cho user tương ứng
GRANT ALL PRIVILEGES ON DATABASE quickbite_user_db TO quickbite_user;
GRANT ALL PRIVILEGES ON DATABASE quickbite_user TO quickbite_user;

GRANT ALL PRIVILEGES ON DATABASE quickbite_restaurant_db TO quickbite_restaurant;
GRANT ALL PRIVILEGES ON DATABASE quickbite_restaurant TO quickbite_restaurant;

GRANT ALL PRIVILEGES ON DATABASE quickbite_order_db TO quickbite_order;
GRANT ALL PRIVILEGES ON DATABASE quickbite_order TO quickbite_order;

GRANT ALL PRIVILEGES ON DATABASE quickbite_notification_db TO quickbite_notification;
GRANT ALL PRIVILEGES ON DATABASE quickbite_notification TO quickbite_notification;
```

##### 2. File cấu hình Docker Compose cho database (`docker-compose.yml`)
Đặt file này cùng thư mục với `init-db.sql`. Chúng ta sẽ mount file SQL vào thư mục tự khởi động của container và đặt tên mạng cố định là `quickbite-net` để các container backend ở các bài học sau có thể kết nối vào mạng này.

```yaml
version: '3.8'

services:
  quickbite-db:
    image: postgres:15-alpine
    container_name: quickbite-db
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: secret_password  # Mật khẩu quản trị tối cao (Superuser)
    volumes:
      - db-data:/var/lib/postgresql/data
      - ./init-db.sql:/docker-entrypoint-initdb.d/init-db.sql  # Mount script khởi tạo tự động
    networks:
      - quickbite-net
    restart: always

volumes:
  db-data:

networks:
  quickbite-net:
    name: quickbite-net  # Đặt tên cố định cho mạng ảo chung
    driver: bridge
```

#### 5.3 Cách khởi chạy và giữ Database chạy ngầm lâu dài

Để chạy database này một lần duy nhất cho toàn bộ môn học, bạn chỉ cần di chuyển vào thư mục `quickbite-database` và chạy lệnh:

```bash
# Khởi chạy database chạy ngầm và giữ trạng thái luôn restart nếu lỗi
docker compose up -d
```

Từ thời điểm này, database PostgreSQL của bạn đã sẵn sàng chạy ngầm trong máy tính. Bạn có thể tự do phát triển, build lại, tắt/mở các container ứng dụng Spring Boot ở các bài học tiếp theo mà không cần phải chạm vào hay khởi động lại database nữa!

---

### PHẦN 4. THỰC HÀNH: KHỞI TẠO FILE COMPOSE VỚI DOCKERFILE VÀ KEY BUILD

Hãy tạo cấu trúc thư mục dự án QuickBite local và khởi chạy tự động build:

1. Thiết lập cấu trúc thư mục mẫu:
   ```text
   quickbite-project/
   ├── docker-compose.yml
   └── user-service/
       ├── Dockerfile
       └── build/
           └── libs/
               └── user-service.jar  (File JAR đã được build sẵn từ Gradle)
   ```
2. Viết nội dung file `docker-compose.yml` tại thư mục gốc:
```yaml
version: '3.8'

services:
  quickbite-user:
    build:
      context: ./user-service
    ports:
      - "8081:8081"
    networks:
      - quickbite-net

networks:
  quickbite-net:
    external: true  # Sử dụng mạng ảo chung đã được khởi tạo bởi database ở Bài 1
```
3. Chạy lệnh build và start dịch vụ:
```bash
docker compose up --build
```
4. **Kết quả mong đợi:** Docker Compose tự động truy cập vào thư mục `./user-service`, thực thi lệnh build đóng gói file JAR thành image tĩnh, đặt tên image tạm thời, sau đó khởi chạy container backend. Do container backend tham gia chung vào mạng ảo `quickbite-net` với database `quickbite-db` đang chạy ngầm từ trước, nó có thể phân giải tên miền và kết nối tới database một cách ổn định.

---

### PHẦN 5. HIỂU LẦM THƯỜNG GẶP (KẾT HỢP IMAGE VS BUILD TRONG SERVICE)

* **Hiểu lầm:** Trong một service của file Compose, chúng ta chỉ được phép khai báo một trong hai từ khóa: hoặc là `image`, hoặc là `build`. Nếu khai báo cả hai sẽ gây lỗi cú pháp.
* **Sự thật:** Hoàn toàn có thể khai báo cả hai. 
  * Ví dụ:
```yaml
quickbite-user:
  build: ./user-service
  image: quickbite-user-service:v1.0.0
```
  * **Cơ chế hoạt động:** Khi bạn chạy `docker compose up --build`, Docker Compose sẽ build image từ thư mục `./user-service` theo hướng dẫn Dockerfile, nhưng thay vì đặt tên ngẫu nhiên, nó sẽ tự động gán nhãn tên image đó là `quickbite-user-service:v1.0.0`. Kỹ thuật này rất quan trọng để tag tên image tự build trước khi đẩy (push) lên kho lưu trữ Docker Registry.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG (XÁC MINH KIẾN THỨC)

1. **Đặc tả cấu trúc file Docker Compose (Services section):**
   * [Compose file services reference - Docker Docs](https://docs.docker.com/compose/compose-file/05-services/)
2. **Chi tiết về cấu hình build trong Compose:**
   * [Compose Build definition - Docker Docs](https://docs.docker.com/compose/compose-file/build/)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Trong file Dockerfile của Spring Boot, tại sao ta nên sử dụng chỉ thị `ENTRYPOINT ["java", "-jar", "app.jar"]` thay vì chỉ thị `CMD java -jar app.jar`?
* *Gợi ý:* Sử dụng cú pháp dạng mảng (Exec Form) như `ENTRYPOINT ["java", "-jar", "app.jar"]` giúp tiến trình Java chạy trực tiếp dưới dạng PID 1 (tiến trình gốc của container). Điều này cho phép container nhận trực tiếp các tín hiệu hệ thống (như SIGTERM khi chạy lệnh `docker stop`) để tắt ứng dụng một cách êm ái (Graceful Shutdown). Trong khi đó, dùng CMD dạng chuỗi (Shell Form) sẽ khởi chạy tiến trình qua một shell `/bin/sh -c`, khiến Java chạy dưới dạng tiến trình con của shell và không thể nhận được tín hiệu tắt của hệ thống, dẫn đến việc tắt cưỡng bức và mất dữ liệu.

#### Câu 2 (Xử lý tình huống)
Nếu bạn thay đổi mã nguồn Java trong `user-service`, sau đó chỉ chạy lệnh `docker compose up` mà không dùng cờ `--build`, Docker Compose có tự động phát hiện mã nguồn Java thay đổi để build lại image mới hay không? Tại sao?
* *Gợi ý:* Không. Docker Compose chỉ tự động build lại image nếu nó phát hiện thư mục context bị thiếu image tương ứng, hoặc bản thân file `Dockerfile` có sự thay đổi. Nó không có khả năng tự chui vào mã nguồn Java kiểm tra xem code có thay đổi hay không. Do đó, mỗi khi cập nhật code Java, bạn bắt buộc phải build lại file JAR mới trên máy host và chạy lệnh kèm cờ build: `docker compose up --build` để ép hệ thống đóng gói lại image mới.
