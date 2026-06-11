# SESSION 04: DOCKER COMPOSE CƠ BẢN

## LESSON 01: Docker Compose và khái niệm hệ thống nhiều container

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Giải thích** được định nghĩa, vai trò và sự cần thiết của hệ thống nhiều container (Multi-container System) trong kiến trúc Microservices.
* **Hiểu bản chất** tại sao Docker Compose là công cụ tối ưu để thay thế quy trình quản trị, cấu hình thủ công bằng các câu lệnh `docker run` đơn lẻ.
* **Phân tích** được quy trình 3 bước hoạt động nền tảng của Docker Compose: Đóng gói (Dockerfile) -> Khai báo (docker-compose.yml) -> Vận hành (docker compose CLI).
* **Kiểm tra và xác thực** cài đặt Docker Compose trên môi trường local (WSL 2 / Linux).

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (NỖI ĐAU CỦA VIỆC QUẢN LÝ THỦ CÔNG)

Hãy tưởng tượng bạn đang tiếp quản dự án QuickBite ở môi trường phát triển local. Hệ thống lúc này đã phình to ra 3 thành phần cốt lõi:
1. **`quickbite-db` (Database):** Container chạy PostgreSQL 15 để lưu trữ dữ liệu.
2. **`user-service` (Dịch vụ người dùng):** Ứng dụng Spring Boot chạy Java 17 cần kết nối tới `quickbite-db`.
3. **`restaurant-service` (Dịch vụ nhà hàng):** Ứng dụng Spring Boot chạy Java 21 cần kết nối tới `quickbite-db`.

Để dựng môi trường chạy thử nghiệm toàn bộ hệ thống này, bạn phải mở terminal lên và gõ tuần tự các câu lệnh sau:
```bash
# 1. Khởi chạy Database
docker run -d --name quickbite-db -e POSTGRES_PASSWORD=secret postgres:15-alpine

# 2. Bạn phải tìm địa chỉ IP nội bộ của container database vừa chạy
docker inspect quickbite-db
#Ví dụ tìm được địa chỉ: 172.17.0.2

# 3. Khởi chạy User Service, truyền cứng địa chỉ IP 172.17.0.2 của DB vào biến kết nối
docker run -d --name quickbite-user -p 8081:8081 -v /path/to/user-service/build/libs:/app -w /app -e SPRING_DATASOURCE_URL=jdbc:postgresql://172.17.0.2:5432/postgres -e SPRING_DATASOURCE_USERNAME=postgres -e SPRING_DATASOURCE_PASSWORD=secret eclipse-temurin:17-jre-alpine java -jar user-service.jar

# 4. Khởi chạy Restaurant Service, cũng truyền cứng địa chỉ IP 172.17.0.2 của DB vào biến kết nối
docker run -d --name quickbite-restaurant -p 8082:8082 -v /path/to/restaurant-service/build/libs:/app -w /app -e SPRING_DATASOURCE_URL=jdbc:postgresql://172.17.0.2:5432/postgres -e SPRING_DATASOURCE_USERNAME=postgres -e SPRING_DATASOURCE_PASSWORD=secret eclipse-temurin:21-jre-alpine java -jar restaurant-service.jar
```

* **Nỗi đau bắt đầu:** 
  * **Tra cứu IP thủ công phiền phức:** Bạn phải gõ lệnh `docker inspect` rất dài và phức tạp chỉ để lấy địa chỉ IP động của container database.
  * **IP biến động (IP Drift):** Nếu container `quickbite-db` gặp lỗi tự khởi động lại và nhận một IP mới (ví dụ: `172.17.0.3`), cả hai backend sẽ lập tức bị mất kết nối và crash. Bạn lại phải đi inspect lấy IP mới rồi gõ lại các câu lệnh run backend từ đầu.
  * **Sai lệch cú pháp:** Chỉ cần gõ sai một ký tự trong đường dẫn volume hoặc cổng, cả cụm sẽ lỗi. Việc dừng dọn dẹp cũng bắt buộc bạn gõ thủ công `docker stop` và `docker rm` từng container một.

*Để giải phóng lập trình viên khỏi gánh nặng gõ lệnh và kiểm soát IP thủ công, Docker cung cấp một công cụ quản lý cấu hình tập trung vô cùng mạnh mẽ: **Docker Compose**.*

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (DOCKER COMPOSE LÀ GÌ?)

#### 3.1 Khái niệm
**Docker Compose** là một công cụ dùng để định nghĩa và chạy các ứng dụng Docker đa container (Multi-container Docker Applications). 

Thay vì quản lý các container bằng các câu lệnh động CLI đơn lẻ (Imperative Style - Phong cách Mệnh lệnh), Docker Compose chuyển sang phong cách **Declarative (Mô tả trạng thái)**: Bạn chỉ cần viết tất cả cấu hình mong muốn (tên container, port, network, volume, biến môi trường) vào một tệp tin cấu hình duy nhất có tên là `docker-compose.yml` (dạng ngôn ngữ YAML). Docker Compose sẽ đọc tệp tin này và tự động dựng, quản lý toàn bộ hệ thống giúp bạn.

```text
  [ Cấu hình tập trung: docker-compose.yml ]
                     │
         Khởi chạy (docker compose up)
                     │
     ┌───────────────┼───────────────┐
     ▼               ▼               ▼
[ Container 1 ] [ Container 2 ] [ Container 3 ]
 (quickbite-db)  (user-service) (restaurant-service)
```

#### 3.2 Quy trình 3 bước làm việc tiêu chuẩn với Docker Compose
Quy trình phát triển và vận hành ứng dụng bằng Docker Compose được tóm tắt qua 3 bước cốt lõi:

1. **Đóng gói (Define):** Viết `Dockerfile` cho từng service của bạn để định nghĩa môi trường chạy tĩnh (ví dụ: Spring Boot code + JDK).
2. **Khai báo (Declare):** Viết tệp tin `docker-compose.yml` khai báo các service, các port cần expose, volume lưu dữ liệu, mạng kết nối và thứ tự ưu tiên khởi chạy.
3. **Vận hành (Run):** Chỉ cần chạy một câu lệnh duy nhất: `docker compose up`. Docker Compose sẽ tự động kéo các image cần thiết, build các image nội bộ, tạo mạng và khởi chạy toàn bộ các dịch vụ đồng thời.

---

### PHẦN 4. THỰC HÀNH CƠ BẢN: KIỂM TRA DOCKER COMPOSE VÀ UP THỬ CONTAINER

#### 4.1 Kiểm tra phiên bản Docker Compose
Hiện nay, Docker Compose v2 đã được tích hợp trực tiếp làm plugin của Docker CLI. Do đó, bạn không còn phải dùng lệnh `docker-compose` (có dấu gạch ngang của v1 cũ) nữa mà chuyển sang sử dụng trực tiếp lệnh:

```bash
docker compose version
```
* **Kết quả mong đợi:** Terminal trả về phiên bản Docker Compose đang hoạt động (ví dụ: `Docker Compose version v2.20.2` hoặc mới hơn).
* **Xử lý sự cố (Nếu gặp lỗi "docker: 'compose' is not a docker command" hoặc báo thiếu command):**
  Điều này có nghĩa là bạn mới chỉ cài đặt lõi Docker Engine cơ bản mà quên chưa cài đặt các plugin mở rộng đi kèm. Hãy mở terminal trên WSL 2 / Linux Ubuntu và chạy lệnh sau để bổ sung đầy đủ:
```bash
sudo apt-get update
sudo apt-get install -y docker-buildx-plugin docker-compose-plugin
```
  *(Sau khi cài đặt thành công, hãy gõ lại lệnh `docker compose version` để kiểm chứng).*

#### 4.2 Viết và khởi chạy tệp Compose tối giản
Hãy tạo một thư mục trống và thực hành viết file compose đầu tiên để kiểm tra hoạt động:

1. Tạo file `docker-compose.yml`:
```yaml
version: '3.8'

services:
   java-tester:
      image: eclipse-temurin:17-jre-alpine
      command: java -version
```
2. Khởi chạy file Compose bằng lệnh:
```bash
docker compose up
```
3. **Kết quả mong đợi:** Docker Compose sẽ tự động kéo image `eclipse-temurin:17-jre-alpine` (nếu chưa có ở local), khởi chạy một container, in thông tin phiên bản Java 17 ra console rồi tự động dừng lại.
4. Dọn dẹp tài nguyên vừa tạo:
   ```bash
   docker compose down
   ```

---

### PHẦN 5. HIỂU LẦM THƯỜNG GẶP (DOCKER COMPOSE VS KUBERNETES)

* **Hiểu lầm thường gặp:** Docker Compose có thể dùng để deploy và scale ứng dụng Microservices lớn trên môi trường Production thực tế với hàng trăm máy chủ vật lý.
* **Sự thật:** Docker Compose chỉ được thiết kế cho môi trường **phát triển local (Development), Staging hoặc triển khai Production quy mô nhỏ chạy trên một máy chủ duy nhất (Single Host)**. 
  * Docker Compose không thể tự động giám sát sức khỏe container và khởi động lại trên máy chủ khác khi máy chủ vật lý bị sập phần cứng (lỗi Node).
  * Khi hệ thống mở rộng lên hàng trăm server, cần tính năng tự động co giãn (Auto Scaling), tự phục hồi (Self-healing) và định tuyến giao thông phức tạp, chúng ta phải chuyển sang sử dụng các bộ điều phối (Orchestrators) như **Docker Swarm** hoặc **Kubernetes (K8s)**.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG (XÁC MINH KIẾN THỨC)

1. **Tổng quan về Docker Compose từ trang chủ:**
   * [Docker Compose Overview - Docker Docs](https://docs.google.com/url?q=https://docs.docker.com/compose/)
2. **Hướng dẫn cài đặt Docker Compose trên các môi trường:**
   * [Install Docker Compose - Docker Docs](https://docs.docker.com/compose/install/)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao từ phiên bản Docker Compose v2, câu lệnh CLI lại chuyển từ `docker-compose` (gạch ngang) sang `docker compose` (khoảng trắng)?
* *Gợi ý:* Docker Compose v1 trước đây được viết bằng ngôn ngữ Python dưới dạng một công cụ độc lập tách biệt với Docker CLI chính. Kể từ phiên bản v2, Docker Compose đã được viết lại bằng ngôn ngữ Go và tích hợp trực tiếp làm một plugin của Docker CLI. Do đó, cú pháp chuyển thành lệnh con `docker compose` để đồng bộ hóa và tối ưu hóa hiệu năng giao tiếp.

#### Câu 2 (So sánh triết lý)
So sánh sự khác nhau về tính bền vững và dọn dẹp tài nguyên giữa việc chạy các container bằng file Bash Script chứa các lệnh `docker run` và chạy bằng file `docker-compose.yml` kết hợp lệnh `docker compose down`.
* *Gợi ý:* Chạy bằng Bash script chỉ đơn giản là thực thi tuần tự các câu lệnh đơn lẻ, Docker không hề biết các container đó có mối liên kết với nhau. Khi muốn tắt đi, bạn phải tự tìm kiếm và viết lệnh stop/rm từng container một cách thủ công. Với Docker Compose, tất cả tài nguyên (container, network, volume) được định nghĩa chung trong một "stack" (ngăn xếp). Lệnh `docker compose down` sẽ tự động phân tích và dọn dẹp sạch sẽ toàn bộ các tài nguyên liên quan trong stack đó một cách an toàn và triệt để, không để lại rác trên hệ thống.
