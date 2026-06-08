# SESSION 04: DOCKER COMPOSE CƠ BẢN

## LESSON 02: Cấu trúc file docker-compose.yml (services, image, build)

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

#### 3.2 Dockerfile cơ bản cho Spring Boot
Để Docker Compose có thể build một image nội bộ cho dịch vụ Spring Boot của bạn, chúng ta phải chuẩn bị một file tên là `Dockerfile` (không có phần mở rộng đuôi) nằm tại thư mục gốc của project backend (ví dụ: `/user-service/Dockerfile`):

```dockerfile
# Bước 1: Chọn image JRE Java 17 gọn nhẹ
FROM eclipse-temurin:17-jre-alpine

# Bước 2: Đặt thư mục làm việc mặc định trong container
WORKDIR /app

# Bước 3: Sao chép file JAR đã build từ máy host vào container
COPY build/libs/user-service.jar app.jar

# Bước 4: Khai báo lệnh chạy ứng dụng khi khởi động container
ENTRYPOINT ["java", "-jar", "app.jar"]
```

#### 3.3 Khai báo thuộc tính `build` trong Compose
Thay vì chỉ định cứng một image có sẵn trên mạng thông qua key `image`, chúng ta sử dụng key `build` trỏ tới Dockerfile cục bộ:
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
  quickbite-db:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: secret

  quickbite-user:
    build:
      context: ./user-service
    ports:
      - "8081:8081"
```
3. Chạy lệnh build và start cụm dịch vụ đồng thời:
```bash
docker compose up --build
```
4. **Kết quả mong đợi:** Docker Compose tự động truy cập vào thư mục `./user-service`, thực thi lệnh build đóng gói file JAR thành image tĩnh, đặt tên image tạm thời, sau đó khởi chạy container database và backend cùng một lúc.

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
