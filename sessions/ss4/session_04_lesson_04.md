# SESSION 04: DOCKER COMPOSE CƠ BẢN

## LESSON 04: Volume và network trong Docker Compose

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Cấu hình Named Volume** trong Docker Compose để lưu trữ dữ liệu bền vững cho cơ sở dữ liệu PostgreSQL, tránh mất dữ liệu khi tắt container.
* **Giải thích** được cơ chế tạo mạng mặc định và mạng tùy biến của Docker Compose.
* **Vận dụng cơ chế Service Discovery** (Phát hiện dịch vụ tự động) để kết nối các microservices với database thông qua tên dịch vụ thay vì địa chỉ IP cứng.
* **Kiểm chứng** tính toàn vẹn của dữ liệu và mạng sau các chu trình xóa/dựng cụm container.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (NỖI ĐAU MẤT DỮ LIỆU & IP BỊ THAY ĐỔI)

Hãy tưởng tượng bạn đang chạy thử nghiệm hệ thống QuickBite trên local bằng Docker Compose. 
1. **Nỗi đau mất sạch dữ liệu:** Bạn khởi chạy database bằng `docker compose up -d`, tạo tài khoản, đăng ký món ăn demo rất mất thời gian. Cuối buổi chiều, trước khi tắt máy ra về, bạn gõ lệnh dọn dẹp hệ thống: `docker compose down`. Sáng hôm sau, bạn bật lại hệ thống bằng `docker compose up -d`. Bạn bàng hoàng nhận ra toàn bộ tài khoản và món ăn đã biến mất không dấu vết, database trở lại trạng thái trống rỗng như lúc mới cài.
2. **Nỗi đau IP biến động:** Dịch vụ `user-service` cần gọi tới database. Nếu bạn điền cứng địa chỉ IP của container database (ví dụ: `172.19.0.2`), kết nối sẽ lỗi ngay lập tức khi database restart và được cấp IP mới (ví dụ: `172.19.0.3`).

*Hai vấn đề nghiêm trọng này sẽ được giải quyết triệt để thông qua hai cơ chế cốt lõi của Docker Compose: **Volumes** (Lưu trữ bền vững) và **Networks** (Mạng ảo nội bộ & DNS).*

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (DOCKER VOLUMES & DOCKER NETWORKS TRONG COMPOSE)

#### 3.1 Docker Volumes trong Compose (Named Volume)
Mặc định, hệ thống file bên trong container là tạm thời (Ephemereal). Mọi dữ liệu ghi vào đó sẽ bị xóa sạch khi container bị hủy bỏ.
Để lưu trữ dữ liệu bền vững (Persistence), chúng ta sử dụng **Named Volume** (Volume có tên định danh):
* **Cú pháp khai báo:** Chúng ta khai báo khối `volumes:` toàn cục ở cuối file Compose, sau đó mount volume đó vào thư mục lưu trữ dữ liệu của database bên trong container.
* **Đường dẫn lưu dữ liệu PostgreSQL:** PostgreSQL lưu toàn bộ database tại thư mục `/var/lib/postgresql/data` trong container.
```yaml
services:
  quickbite-db:
    image: postgres:15-alpine
    volumes:
      - db-data:/var/lib/postgresql/data  # Mount Named Volume vào thư mục dữ liệu

volumes:
  db-data:  # Khai báo Named Volume toàn cục
```
* **Cơ chế hoạt động:** Docker Engine sẽ tạo một thư mục quản lý riêng trên ổ cứng máy host và liên kết nó với thư mục dữ liệu của Postgres. Khi container bị xóa đi, thư mục trên máy host vẫn an toàn nguyên vẹn. Khi container mới khởi chạy và mount vào volume này, toàn bộ dữ liệu cũ sẽ lập tức xuất hiện trở lại.

#### 3.2 Docker Networks trong Compose & Service Discovery
Mặc định, khi bạn khởi chạy một tệp Compose, Docker Compose sẽ tự động làm 3 việc:
1. Tạo ra một mạng bridge riêng biệt dành cho dự án (thường đặt tên là `[tên_thư_mục_dự_án]_default`).
2. Tự động đưa tất cả các container (services) được khai báo trong file Compose tham gia vào mạng chung này.
3. Kích hoạt dịch vụ **DNS nội bộ** của Docker.

##### Cơ chế Service Discovery (Phát hiện dịch vụ)
Nhờ DNS nội bộ hoạt động trong mạng chung, các container có thể gọi nhau trực tiếp bằng **Tên dịch vụ (Service Name)** được khai báo trong file Compose (như `quickbite-db`, `quickbite-user`) thay vì sử dụng địa chỉ IP nội bộ không ổn định.

```text
  [ user-service (Service Name) ]
                │
                ▼ (Kết nối qua URL: jdbc:postgresql://quickbite-db:5432/postgres)
  [ DNS nội bộ của Docker Compose ] (Tự dịch "quickbite-db" -IP 172.19.0.3)
                │
                ▼
  [ quickbite-db (Service Name) ]
```

[!TIP]
**So sánh trực quan: Nếu làm bằng lệnh Docker thủ công thì sao?**
Để dễ hình dung những gì Docker Compose đang âm thầm tự động hóa dưới nền, đây là cách bạn phải gõ thủ công bằng các lệnh CLI đơn lẻ:
```bash
# Bước 1: Tự tạo một mạng bridge tùy biến bằng tay
docker network create --driver bridge quickbite-net

# Bước 2: Khởi chạy container database và ném vào mạng ảo đó
docker run -d --name quickbite-db --network quickbite-net -e POSTGRES_PASSWORD=secret_password postgres:15-alpine

# Bước 3: Khởi chạy container backend kết nối vào chung mạng ảo để DNS nội bộ nhận diện được tên
docker run -d --name quickbite-user --network quickbite-net -p 8081:8081 -v /path/to/libs:/app -w /app eclipse-temurin:17-jre-alpine java -jar user-service.jar
```
* **Kết luận:** Thay vì phải gõ và quản lý 3 lệnh rời rạc cùng dải tham số phức tạp ở trên, Docker Compose chỉ đơn giản là đọc file `docker-compose.yml` rồi tự chạy đống lệnh này thay cho bạn.


---

### PHẦN 4. THỰC HÀNH: KHAI BÁO VOLUME VÀ NETWORK LIÊN KẾT ĐA CONTAINER

Hãy cập nhật tệp `docker-compose.yml` để hoàn thiện hạ tầng mạng và lưu trữ bền vững cho QuickBite:

1. Viết tệp `docker-compose.yml` hoàn chỉnh:
```yaml
version: '3.8'

services:
  quickbite-db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: secret_password
    volumes:
      - quickbite-db-volume:/var/lib/postgresql/data
    networks:
      - quickbite-net

  quickbite-user:
    build:
      context: ./user-service
    ports:
      - "8081:8081"
    environment:
      # Sử dụng tên service "quickbite-db" làm host kết nối database
      SPRING_DATASOURCE_URL: jdbc:postgresql://quickbite-db:5432/postgres
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: secret_password
    networks:
      - quickbite-net
    depends_on:
      - quickbite-db

volumes:
  quickbite-db-volume: # Khai báo Named Volume lưu trữ database

networks:
  quickbite-net:
    driver: bridge # Khai báo mạng ảo bridge tùy biến cho dự án
```
2. Khởi chạy hệ thống:
```bash
docker compose up -d
```
3. Kiểm tra xem volume và network đã được tạo thành công chưa:
```bash
docker volume ls
# Kết quả mong đợi: Hiển thị volume tên [thư_mục_dự_án]_quickbite-db-volume

docker network ls
# Kết quả mong đợi: Hiển thị mạng tên [thư_mục_dự_án]_quickbite-net
```
4. **Kiểm tra độ bền vững dữ liệu:**
   * Truy cập vào database qua `docker exec` tạo thử một bảng dữ liệu hoặc chạy ứng dụng đăng ký người dùng.
   * Chạy lệnh xóa cụm dịch vụ: `docker compose down`.
   * Khởi chạy lại: `docker compose up -d`.
   * Truy cập lại database để kiểm tra. **Kết quả mong đợi:** Toàn bộ dữ liệu bạn đã tạo trước đó vẫn còn nguyên vẹn 100%.

---

### PHẦN 5. HIỂU LẦM THƯỜNG GẶP (MẠNG MẶC ĐỊNH CỦA DOCKER COMPOSE)

* **Hiểu sai:** Mạng mặc định tự động tạo ra khi chạy Docker Compose hoạt động giống hệt mạng mặc định (`default bridge`) của Docker CLI khi chạy lệnh `docker run` đơn lẻ.
* **Đính chính:** **Hoàn toàn khác nhau về bản chất.**
  * Mạng mặc định của Docker CLI (`default bridge`) **không hỗ trợ tính năng tự động phân giải tên container (DNS nội bộ)**. Container chỉ có thể kết nối với nhau qua IP.
  * Mạng mặc định do Docker Compose tự động tạo ra cho dự án thực chất là một mạng **User-defined Bridge Network**. Mạng này tự động bật sẵn dịch vụ DNS nội bộ, cho phép các dịch vụ gọi nhau bằng tên service một cách hoàn toàn tự động và ổn định. Do đó, bạn không cần phải tự tạo mạng thủ công như ở các bài học trước.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG (XÁC MINH KIẾN THỨC)

1. **Quản lý mạng trong Docker Compose:**
   * [Networking in Docker Compose - Docker Docs](https://docs.docker.com/compose/networking/)
2. **Quản lý dữ liệu bằng Volume trong Docker Compose:**
   * [Volumes in Docker Compose - Docker Docs](https://docs.docker.com/compose/compose-file/07-volumes/)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Named Volume lưu trữ dữ liệu thực tế ở đâu trên máy host vật lý chạy hệ điều hành Linux?
* *Gợi ý:* Trên các hệ điều hành Linux, Docker Engine quản lý toàn bộ các Named Volumes tại thư mục mặc định `/var/lib/docker/volumes/[tên_volume]/_data`. Khi container hoạt động và ghi dữ liệu, Docker sẽ ánh xạ các hoạt động ghi file trực tiếp xuống thư mục vật lý này của máy host.

#### Câu 2 (Xử lý tình huống)
Nếu bạn chạy hai dự án Microservices hoàn toàn khác nhau trên cùng một máy host vật lý (dự án A nằm ở thư mục `quickbite-dev` và dự án B nằm ở thư mục `hotel-booking`), các container của dự án B có thể gọi được các container của dự án A bằng tên service của chúng được không? Tại sao?
* *Gợi ý:* Không được. Mặc định, mỗi tệp Docker Compose khi khởi chạy sẽ tạo ra một mạng bridge cô lập riêng biệt dựa trên tên của thư mục dự án (ví dụ mạng `quickbite-dev_default` và mạng `hotel-booking_default`). Do các container nằm ở hai mạng ảo cô lập khác nhau, DNS của mạng B không thể nhìn thấy và phân giải được tên miền của các container thuộc mạng A. Đây là cơ chế bảo mật cô lập tuyệt vời giúp tránh nhiễu cấu hình giữa các dự án chạy chung một server.
