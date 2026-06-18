# PROMPT CHO GAMMA: DOCKER COMPOSE VÀ DOCKERFILE (SESSION 4)

## 1. CONTEXT & ROLE
* **Role:** DevOps Architect kiêm Giảng viên cao cấp. Giọng điệu chuyên nghiệp, trực diện, đi thẳng vào bản chất kỹ thuật và thực tiễn vận hành hệ thống.
* **Target Audience:** Kỹ sư phần mềm, học viên đang phát triển dự án Spring Boot Microservices (hệ thống QuickBite).
* **Objective:** Giải thích cặn kẽ quy trình đóng gói ứng dụng với Dockerfile và điều phối hệ thống nhiều container với Docker Compose, cung cấp sẵn các câu lệnh, cấu hình và sơ đồ luồng chi tiết để người học hiểu và áp dụng được ngay.

---

## 2. PARTITION INSTRUCTIONS
* **Số lượng slide khuyến nghị:** 15 - 20 slides.
* **Nguyên tắc phân bổ nội dung:**
  * **LESSON 01 (Mở đầu & Dẫn dắt):** Phân tích bối cảnh, sự bất tiện khi vận hành các lệnh khởi chạy thủ công, và giới thiệu Dockerfile như "nguồn sự thật duy nhất" (Source of Truth). Dẫn dắt sang các bài học tiếp theo.
  * **Từ LESSON 02 đến LESSON 06 (Giải pháp kỹ thuật):** Đi thẳng vào định nghĩa, sơ đồ luồng hoạt động, cấu hình YAML và lệnh thực hành thực tế.
  * **Độ thoáng đãng:** Một slide chỉ trình bày một thông điệp hoặc khái niệm cốt lõi. Không nhồi nhét chữ.
  * **Độ cô đọng cao:** Sử dụng các câu văn ngắn gọn, súc tích, loại bỏ các từ ngữ suồng sã, giật gân (như "ăn hành", "intern").

---

## 3. HIERARCHICAL OUTLINE (DÀN Ý CHI TIẾT)

### LESSON 01: Dockerfile và cách đóng gói ứng dụng Spring Boot thực tế

#### Slide 1: Vấn đề thực tế - Sự phân tán cấu hình trong vận hành thủ công
* **Bối cảnh dự án QuickBite:** Triển khai 3 dịch vụ cơ bản ở local:
  * Database: container PostgreSQL 15.
  * Dịch vụ `user-service` chạy Java 17.
  * Dịch vụ `restaurant-service` chạy Java 21.
* **Các lệnh chạy thủ công dài dòng, dễ sai sót:**
  * Khởi chạy Database: `docker run -d --name quickbite-db -e POSTGRES_PASSWORD=secret postgres:15-alpine`
  * Tìm IP động của DB: `docker inspect quickbite-db` (Ví dụ: `172.17.0.2`).
  * Khởi chạy `user-service`, truyền IP và mount thư mục JAR:
    `docker run -d --name user-service -p 8081:8081 -v /path/to/jar:/app -e DB_HOST=172.17.0.2 eclipse-temurin:17-jre-alpine java -jar user-service.jar`
* **Hạn chế:**
  * Khó theo dõi phiên bản Java và cổng kết nối của từng dịch vụ.
  * Lệnh gõ quá dài, dễ sai lệch tham số cổng, tên biến môi trường hoặc đường dẫn volume.
  * Địa chỉ IP động dễ thay đổi khi container restart, gây đứt gãy kết nối.
* **Giải pháp:** Sử dụng **`Dockerfile`** để gom toàn bộ đặc tả môi trường chạy vào một tệp cấu hình duy nhất nằm trong mã nguồn dịch vụ.

#### Slide 2: Cấu trúc Dockerfile cơ bản cho ứng dụng Spring Boot
* Một tệp `Dockerfile` là tập hợp các chỉ thị được thực thi tuần tự để xây dựng một Docker Image tĩnh:
  ```dockerfile
  # 1. Định nghĩa Base Image (JRE Alpine siêu nhẹ)
  FROM eclipse-temurin:17-jre-alpine
  # 2. Thiết lập thư mục làm việc mặc định trong container
  WORKDIR /app
  # 3. Sao chép file JAR đã build từ host vào container
  COPY build/libs/user-service.jar app.jar
  # 4. Khai báo cổng lắng nghe nội bộ (chỉ mang ý nghĩa tài liệu hóa)
  EXPOSE 8081
  # 5. Lệnh khởi chạy tiến trình chính
  ENTRYPOINT ["java", "-jar", "app.jar"]
  ```
* **Giải thích các chỉ thị chính:**
  * `FROM`: Điểm khởi đầu, lấy một image nền có sẵn làm môi trường gốc.
  * `WORKDIR`: Tạo và di chuyển vào thư mục làm việc bên trong container.
  * `COPY`: Sao chép file thực thi từ máy host vật lý vào hệ thống file của container.
  * `EXPOSE`: Khai báo cổng mạng mà container sẽ lắng nghe khi chạy.
  * `ENTRYPOINT`: Định nghĩa lệnh mặc định được thực thi khi container khởi động.

#### Slide 3: Phân biệt ENTRYPOINT vs CMD và Cơ chế Graceful Shutdown
* **ENTRYPOINT (Exec Form - Khuyên dùng):**
  * Khai báo dưới dạng mảng: `ENTRYPOINT ["java", "-jar", "app.jar"]`.
  * Tiến trình Java được gán trực tiếp làm tiến trình gốc **PID 1**.
  * Nhận trực tiếp tín hiệu `SIGTERM` khi chạy `docker stop` để thực hiện **Graceful Shutdown** (đóng kết nối database an toàn, hoàn thành các request dở dang rồi tắt).
* **CMD (Shell Form):**
  * Khai báo dạng chuỗi: `CMD java -jar app.jar`.
  * Lệnh chạy qua trình shell trung gian `/bin/sh -c`, khiến shell chiếm quyền PID 1, còn Java chạy dưới dạng tiến trình con.
  * Trình shell không chuyển tiếp tín hiệu `SIGTERM` cho Java. Sau 10 giây timeout chờ đợi, Docker Engine phát lệnh cưỡng bức `SIGKILL` tiêu diệt tiến trình Java lập tức, gây mất an toàn dữ liệu.

#### Slide 4: Quy trình đóng gói và Khởi chạy thử nghiệm tại máy Local
* Thực hiện lần lượt 4 bước build và chạy:
```bash
# 1. Đóng gói mã nguồn Java thành file JAR bằng Gradle
./gradlew bootJar
# 2. Thực hiện build Docker Image từ Dockerfile (dấu chấm ở cuối là build context)
docker build -t quickbite-user-service:v1 .
# 3. Khởi chạy Container từ Image tự build
docker run -d -p 8081:8081 --name user-service quickbite-user-service:v1
# 4. Kiểm tra nhật ký log để xác nhận Spring Boot khởi chạy thành công
docker logs -f user-service
```
* **Hiểu lầm thường gặp (Expose vs Publish):**
  * `EXPOSE 8081` trong Dockerfile chỉ mang ý nghĩa tài liệu hóa cổng kết nối mặc định của ứng dụng, không tự động mở cổng ra máy host vật lý.
  * Để kết nối được từ trình duyệt máy host, bắt buộc phải truyền tham số `-p 8081:8081` (Publish Port) khi chạy lệnh `docker run`.

---

### LESSON 02: Docker Compose và khái niệm hệ thống nhiều container

#### Slide 5: Hệ thống nhiều container (Multi-container System) trong Microservices
* **Định nghĩa:** Mô hình vận hành trong đó mỗi dịch vụ được đóng gói và hoạt động trong một container độc lập (Database, User Service, Restaurant Service, Gateway).
* **Tại sao cần chia tách thành các container riêng biệt?**
  1. *Đa dạng công nghệ (Technology Diversity):* Dịch vụ A chạy Java 17, dịch vụ B chạy Java 21, Database viết bằng C. Tách biệt giúp tránh phình to image và xung đột thư viện runtime.
  2. *Mở rộng độc lập (Independent Scaling):* Dễ dàng scale up số lượng container của các dịch vụ chịu tải cao mà không ảnh hưởng đến database.
  3. *Cô lập lỗi (Fault Isolation):* Một dịch vụ bị crash không kéo sập toàn bộ hệ thống.
* **Thách thức:** Việc điều phối kết nối mạng và quản lý vòng đời thủ công cho hàng chục container rất phức tạp.

#### Slide 6: Docker Compose và Quy trình 3 bước làm việc tiêu chuẩn
* **Docker Compose là gì?**
  * Công cụ định nghĩa và chạy các ứng dụng Docker đa container.
  * Thay thế phong cách Mệnh lệnh CLI (`docker run` đơn lẻ) bằng phong cách **Mô tả trạng thái (Declarative)** thông qua tệp cấu hình duy nhất `docker-compose.yml` (định dạng YAML).
* **Quy trình 3 bước làm việc tiêu chuẩn:**
  1. *Đóng gói (Define):* Viết `Dockerfile` định nghĩa môi trường chạy cho từng dịch vụ.
  2. *Khai báo (Declare):* Viết tệp `docker-compose.yml` khai báo các service, cổng mạng, volume lưu trữ, mạng ảo nội bộ và các tham số môi trường.
  3. *Vận hành (Run):* Chạy lệnh duy nhất `docker compose up` để tự động hóa toàn bộ chu kỳ khởi tạo hệ thống.

#### Slide 7: Thực hành kiểm tra cài đặt Docker Compose
* **Kiểm tra phiên bản đang hoạt động:**
  ```bash
  docker compose version
  ```
  *(Từ phiên bản v2, Docker Compose được viết bằng Go và chạy trực tiếp dưới dạng plugin của Docker CLI).*
* **Bổ sung plugin nếu gặp lỗi thiếu lệnh:**
  ```bash
  sudo apt-get update
  sudo apt-get install -y docker-buildx-plugin docker-compose-plugin
  ```
* **Khởi chạy file Compose tối giản để kiểm tra:**
  * Nội dung tệp `docker-compose.yml`:
    ```yaml
    version: '3.8'
    services:
       java-tester:
          image: eclipse-temurin:17-jre-alpine
          command: java -version
    ```
  * Lệnh khởi chạy và dọn dẹp:
    `docker compose up` (Tải image, khởi tạo container và in version) -> `docker compose down` (Dọn dẹp container).

---

### LESSON 03: Cấu trúc file docker-compose.yml (services, image, build)

#### Slide 8: Các từ khóa cơ bản trong cấu trúc docker-compose.yml
* Cấu trúc tệp Compose viết bằng định dạng YAML (thụt lề bằng khoảng trắng, cấm dùng phím Tab):
  * `version`: Phiên bản định dạng của file Compose (khuyên dùng `'3.8'`).
  * `services`: Khối định nghĩa danh sách các container (services) trong cụm.
  * `image`: Tên image được kéo về từ Registry (ví dụ: `postgres:15-alpine`).
  * `build`: Cấu hình tự động đóng gói image từ `Dockerfile` nội bộ:
    * `context`: Thư mục chứa mã nguồn và tệp `Dockerfile`.
    * `dockerfile`: Tên tệp Dockerfile cấu hình (mặc định là `Dockerfile`).
* **Ví dụ khai báo build:**
  ```yaml
  services:
    quickbite-user:
      build:
        context: ./user-service
      ports:
        - "8081:8081"
  ```

#### Slide 9: Chiến lược tách biệt Database chạy độc lập cho dự án QuickBite
* Trong kiến trúc microservices thực tế, Database luôn được chạy độc lập thay vì gom chung tệp Compose với Application vì 4 lý do:
  1. *Tối ưu chu kỳ phát triển (Development Loop):* Backend rebuild/restart liên tục. Tách biệt giúp database hoạt động ổn định không bị khởi động lại ngắt quãng.
  2. *Tiết kiệm tài nguyên:* Chạy một container database chung chứa nhiều database con cho từng service thay vì chạy mỗi service một container database riêng.
  3. *Bảo vệ dữ liệu (Stateless vs Stateful):* Tránh việc vô tình xóa sạch dữ liệu database khi dọn dẹp backend bằng lệnh `docker compose down -v`.
  4. *Tiêu chuẩn Production:* Sẵn sàng cho việc cấu hình kết nối tới các dịch vụ Cloud Database (AWS RDS, Cloud SQL) sau này.

#### Slide 10: Thực hành cấu hình Database chạy độc lập và tự khởi tạo dữ liệu
* PostgreSQL hỗ trợ tự động quét và chạy các script SQL nằm trong thư mục `/docker-entrypoint-initdb.d/` trong container khi khởi chạy lần đầu.
* **File kịch bản khởi tạo SQL (`init-db.sql`):**
  ```sql
  CREATE USER quickbite_user WITH PASSWORD 'quickbite_user';
  CREATE DATABASE quickbite_user_db OWNER quickbite_user;
  GRANT ALL PRIVILEGES ON DATABASE quickbite_user_db TO quickbite_user;
  ```
* **File cấu hình `docker-compose.yml` cho database chung:**
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
        POSTGRES_PASSWORD: secret_password
      volumes:
        - db-data:/var/lib/postgresql/data
        - ./init-db.sql:/docker-entrypoint-initdb.d/init-db.sql
      networks:
        - quickbite-net
      restart: always
  volumes:
    db-data:
  networks:
    quickbite-net:
      name: quickbite-net
      driver: bridge
  ```
* *Lệnh khởi chạy giữ chạy ngầm lâu dài:* `docker compose up -d` (chạy tại thư mục `quickbite-database`).

#### Slide 11: Thực hành tự động build và start dịch vụ backend
* File cấu hình backend `docker-compose.yml` (đặt tại thư mục `quickbite-project/`):
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
      external: true  # Sử dụng mạng ảo chung do database đã tạo trước đó
  ```
* **Khởi chạy và tự động rebuild:**
  ```bash
  docker compose up --build
  ```
  *Lưu ý:* Khi code Java thay đổi, Docker Compose không tự phát hiện để rebuild. Bạn bắt buộc phải build lại file JAR mới trên máy host và chạy lệnh kèm cờ `--build` để ép hệ thống đóng gói lại image mới.

---

### LESSON 04: Biến môi trường và cấu hình port

#### Slide 12: Cấu hình Port Mapping và rủi ro lộ mật khẩu trên Git
* **Cú pháp Port Mapping (`ports`):**
  * Ánh xạ cổng: `- "HOST_PORT:CONTAINER_PORT"`.
  * Docker Engine lắng nghe cổng `HOST_PORT` trên máy host vật lý và NAT traffic vào cổng `CONTAINER_PORT` bên trong container.
* **Lỗ hổng bảo mật rò rỉ thông tin (Credentials Leak):**
  * Ghi trực tiếp mật khẩu database và các khóa bí mật (JWT secret key) vào file `docker-compose.yml` rồi push lên Git là lỗi bảo mật nghiêm trọng.
  * Gây khó khăn khi triển khai trên các môi trường khác nhau (Dev, Staging, Prod), vi phạm nguyên lý "Build once, run anywhere".

#### Slide 13: Giải pháp tách biệt cấu hình bảo mật bằng file `.env`
* **1. Tạo tệp cấu hình ẩn `.env` (Đặt cùng cấp với file `docker-compose.yml`):**
  ```env
  DB_HOST=quickbite-db
  DB_PORT=5432
  USER_DB_NAME=quickbite_user_db
  USER_DB_USERNAME=quickbite_user
  USER_DB_PASSWORD=quickbite_user
  USER_SERVER_PORT=8081
  ```
  *(Thêm tệp `.env` vào file `.gitignore` để tránh bị đẩy lên Git).*
* **2. Khai báo biến nội suy `${...}` trong `docker-compose.yml`:**
  ```yaml
  services:
    quickbite-user:
      build: ./user-service
      ports:
        - "${USER_SERVER_PORT}:${USER_SERVER_PORT}"
      environment:
        - DB_HOST=${DB_HOST}
        - DB_PORT=${DB_PORT}
        - DB_NAME=${USER_DB_NAME}
        - DB_USERNAME=${USER_DB_USERNAME}
        - DB_PASSWORD=${USER_DB_PASSWORD}
        - SERVER_PORT=${USER_SERVER_PORT}
  ```
* **3. Lệnh kiểm tra và nội suy biến:**
  ```bash
  docker compose config
  ```
  *Cơ chế:* Đọc file `.env`, thay thế các biến `${...}` thành giá trị thực tế và in ra cấu hình hoàn chỉnh để kiểm tra cú pháp trước khi chạy.

---

### LESSON 05: Volume và network trong Docker Compose

#### Slide 14: Docker Volumes - Cơ chế lưu trữ dữ liệu bền vững
* Mặc định, hệ thống file của container là tạm thời (Ephemeral), dữ liệu sẽ mất khi container bị xóa.
* **Named Volumes (Volume định danh do Docker tự quản lý):**
  * Cấu hình khai báo:
    ```yaml
    services:
      quickbite-db:
        image: postgres:15-alpine
        volumes:
          - db-data:/var/lib/postgresql/data
    volumes:
      db-data:
    ```
  * *Bản chất:* Docker tự động quản lý thư mục ẩn trên host (Linux: `/var/lib/docker/volumes/...`). Bảo mật, hiệu năng đọc ghi cao, độc lập với cấu trúc hệ điều hành host. Dùng cho dữ liệu động như Database.
* **Bind Mounts (Liên kết trực tiếp thư mục từ máy host):**
  * Cấu hình khai báo: `- ./init-db.sql:/docker-entrypoint-initdb.d/init-db.sql`.
  * *Bản chất:* Ánh xạ trực tiếp tệp tin từ thư mục máy host vào container. Phù hợp cho tệp cấu hình tĩnh hoặc code hot-reload. Phụ thuộc cấu trúc thư mục của máy host.

#### Slide 15: Docker Networks & Cơ chế Service Discovery
* Khi khởi chạy tệp Compose, hệ thống tự động tạo một mạng ảo Bridge cho cụm và bật sẵn dịch vụ **DNS nội bộ**.
* **Cơ chế Service Discovery (Phát hiện dịch vụ tự động):**
  * Các container kết nối chung mạng ảo có thể giao tiếp với nhau bằng **Tên dịch vụ (Service Name)** khai báo trong file Compose (ví dụ: `quickbite-db`) thay vì sử dụng địa chỉ IP nội bộ không ổn định.
  * Tên dịch vụ tự động được phân giải thành IP động tương ứng nhờ DNS nội bộ.
* **Tham chiếu mạng ngoài (`external: true`):**
  * Cấu hình khai báo:
    ```yaml
    networks:
      quickbite-net:
        external: true
    ```
  * *Bản chất:* Docker Compose không tự tạo mạng mới mà đi kết nối các container vào mạng `quickbite-net` đã có sẵn. Mạng này tồn tại độc lập với vòng đời của cụm container.

#### Slide 16: Thực hành kiểm chứng tính toàn vẹn dữ liệu
* Quy trình kiểm tra dữ liệu qua các chu trình tắt/mở hệ thống:
  ```bash
  # 1. Tạo dữ liệu thử nghiệm trong container database đang chạy
  docker compose exec quickbite-db psql -U postgres -d quickbite_user_db -c "CREATE TABLE test_volume (note VARCHAR(50)); INSERT INTO test_volume VALUES ('Du lieu an toan!');"
  # 2. Xóa bỏ cụm container của dự án (cả database và backend)
  docker compose down
  # 3. Khởi chạy lại cụm container database
  docker compose up -d
  # 4. Truy vấn lại bảng dữ liệu để xác minh
  docker compose exec quickbite-db psql -U postgres -d quickbite_user_db -c "SELECT * FROM test_volume;"
  ```
* **Kết quả mong đợi:** Dữ liệu vẫn tồn tại nguyên vẹn nhờ cơ chế Named Volume.
* **Mẹo phân biệt:** Mạng mặc định của Docker CLI (`default bridge`) không hỗ trợ DNS nội bộ. Mạng mặc định của Docker Compose thực chất là một mạng `User-defined Bridge Network` được bật sẵn DNS nội bộ giúp tự động phân giải tên container.

---

### LESSON 06: Quản lý vòng đời hệ thống với Docker Compose

#### Slide 17: Các lệnh điều khiển Docker Compose CLI chủ chốt
* Các công cụ tương tác trực tiếp với toàn bộ ngăn xếp container của dự án:
  * **Khởi tạo và chạy:**
    * `docker compose up -d`: Khởi chạy toàn bộ cụm dịch vụ chạy ngầm dưới nền.
    * `docker compose start`: Kích hoạt lại các container đã bị dừng.
  * **Dừng và dọn dẹp:**
    * `docker compose stop`: Tạm dừng hoạt động các container (trạng thái Exited), giải phóng CPU/RAM nhưng giữ nguyên container và dữ liệu tạm thời trên đĩa.
    * `docker compose down`: Dừng và xóa bỏ hoàn toàn các container, mạng ảo của dự án để giải phóng tài nguyên.
  * **Lưu ý rủi ro:** Lệnh `docker compose down` giữ an toàn cho dữ liệu trong Named Volume. Dữ liệu chỉ bị xóa sạch khi bạn truyền thêm cờ xóa volume: `docker compose down -v` (hoặc `--volumes`).

#### Slide 18: Nhóm lệnh chẩn đoán trạng thái hệ thống
* **Liệt kê trạng thái container:**
  ```bash
  docker compose ps
  ```
  Hiển thị danh sách container, trạng thái (Up/Exit), và cổng ánh xạ mạng thực tế.
* **Xem nhật ký log tổng hợp của toàn cụm:**
  ```bash
  docker compose logs -f --tail=50
  ```
  *Ưu việt:* Stream log của tất cả container cùng lúc, tự động phân tách màu sắc và chèn tiền tố tên dịch vụ ở đầu dòng giúp dễ dàng theo dõi trình tự lỗi.
* **Kiểm tra trạng thái sẵn sàng của Database độc lập:**
  ```bash
  docker exec -it quickbite-db pg_isready -U postgres
  ```
  Console trả ra dòng chữ `/var/run/postgresql:5432 - accepting connections` báo hiệu DB đã sẵn sàng nhận kết nối từ backend.
* **Chạy câu lệnh SQL trực tiếp từ host để test kết nối:**
  ```bash
  docker compose exec quickbite-db psql -U postgres -c "SELECT 1;"
  ```
