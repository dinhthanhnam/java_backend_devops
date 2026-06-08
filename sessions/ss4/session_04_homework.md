# SESSION 04: BÀI TẬP VỀ NHÀ (HOMEWORK)
## Đề tài: Đóng gói Ứng dụng & Triển khai Đa Container bằng Docker Compose

Tài liệu này chứa 6 bài tập thực hành dành cho Session 4 của hệ thống QuickBite. Bạn cần hoàn thành đầy đủ các bài tập và nộp bài theo hướng dẫn ở cuối mỗi bài. Kết quả được đánh giá theo tiêu chí Đạt (Pass) hoặc Không đạt (Fail).

---

### BÀI TẬP 1 (Mức độ Khá): Viết Dockerfile cơ bản để đóng gói Spring Boot `user-service`

#### 1. Mục tiêu mong muốn đạt được
* **Biên soạn cấu trúc Dockerfile chuẩn** cho ứng dụng Java Spring Boot.
* **Tự build và đóng gói thành công** một Docker Image từ mã nguồn local.

#### 2. Mô tả yêu cầu
Bạn hãy đóng vai trò kỹ sư DevOps viết file đóng gói cho dịch vụ `user-service` của QuickBite:
1. Tạo một file đặt tên chính xác là `Dockerfile` (không có phần mở rộng đuôi) nằm tại thư mục gốc của project `/user-service/`.
2. Định nghĩa các chỉ thị trong file để:
   * Sử dụng base image JRE 17 gọn nhẹ của Eclipse Temurin (`eclipse-temurin:17-jre-alpine`).
   * Đặt thư mục làm việc mặc định trong container là `/app`.
   * Sao chép file JAR đã build từ máy host (`build/libs/user-service.jar` hoặc tên file SNAPSHOT tương ứng của bạn) vào container dưới tên `app.jar`.
   * Chạy câu lệnh `java -jar app.jar` bằng cú pháp mảng (Exec Form) để làm tiến trình gốc (PID 1).
3. Thực thi lệnh build image đặt tên (tag) là `quickbite-user-service:test` trực tiếp bằng Docker CLI.

#### 3. Kiểm thử và kết quả mong muốn
1. Biên dịch file JAR trên host và build Docker image:
   ```bash
   # (Đứng tại thư mục user-service)
   ./gradlew bootJar
   docker build -t quickbite-user-service:test .
   ```
2. Kiểm tra danh sách image:
   ```bash
   docker images
   ```
3. **Kết quả mong đợi:** Xuất hiện image `quickbite-user-service` với tag `test`, kích thước nhỏ gọn (khoảng 100MB - 150MB).

#### 4. Hướng dẫn nộp bài
* Đẩy file `Dockerfile` của bạn lên Git tại thư mục dự án của bạn: `/user-service/Dockerfile`.
* Chụp màn hình terminal chạy lệnh `docker images` hiển thị rõ image `quickbite-user-service:test`. Lưu hình ảnh vào thư mục Git cá nhân tại: `/homework/session_04/exercise_01/build_success.png`.

---

### BÀI TẬP 2 (Mức độ Khá): Thiết lập Docker Compose khởi chạy database PostgreSQL

#### 1. Mục tiêu mong muốn đạt được
* **Biên soạn đúng cú pháp YAML** cho tệp tin `docker-compose.yml`.
* **Khai báo và khởi chạy dịch vụ** Postgres thông qua Docker Compose CLI.

#### 2. Mô tả yêu cầu
Bắt đầu dựng hạ tầng tập trung cho QuickBite bằng cách tạo file Compose:
1. Tạo file đặt tên chính xác là `docker-compose.yml` nằm ở thư mục gốc của toàn bộ dự án (`/quickbite-project/docker-compose.yml`).
2. Khai báo dịch vụ cơ sở dữ liệu với cấu hình:
   * Tên service trong compose: `quickbite-db`.
   * Sử dụng image `postgres:15-alpine`.
   * Ánh xạ cổng `-p 5432:5432` ra ngoài máy host.
   * Truyền mật khẩu database `secret` qua biến môi trường `POSTGRES_PASSWORD`.

#### 3. Kiểm thử và kết quả mong muốn
1. Khởi chạy dịch vụ Compose ở chế độ chạy ngầm:
   ```bash
   docker compose up -d
   ```
2. Kiểm tra trạng thái:
   ```bash
   docker compose ps
   ```
3. **Kết quả mong đợi:** Container chạy thành công ở trạng thái `Up`, cổng `5432` được mở ra host.
4. Chạy lệnh dừng hệ thống:
   ```bash
   docker compose down
   ```

#### 4. Hướng dẫn nộp bài
* Đẩy file `docker-compose.yml` (phiên bản Bài 2) lên Git tại thư mục gốc: `/docker-compose.yml`.
* Chụp ảnh màn hình terminal chạy lệnh `docker compose ps` hiển thị rõ database đang chạy. Lưu hình ảnh tại: `/homework/session_04/exercise_02/compose_db.png`.

---

### BÀI TẬP 3 (Mức độ Khá): Liên kết backend tự build và database qua Service Discovery trong Compose

#### 1. Mục tiêu mong muốn đạt được
* **Ứng dụng thuộc tính `build`** để Compose tự động build image từ Dockerfile.
* **Cấu hình liên kết mạng** cho phép các container tự tìm thấy nhau qua DNS bằng tên service.

#### 2. Mô tả yêu cầu
Bây giờ, hãy bổ sung dịch vụ `user-service` vào trong file `docker-compose.yml` của Bài 2:
1. Khai báo thêm service backend đặt tên là `quickbite-user`.
2. Sử dụng key `build` để chỉ định Context trỏ tới thư mục `./user-service`.
3. Ánh xạ cổng `8081:8081` ra ngoài máy host.
4. Cấu hình biến môi trường kết nối database cho Spring Boot. **Lưu ý quan trọng:** Không sử dụng IP, hãy cấu hình Spring URL kết nối sử dụng chính tên service database: `jdbc:postgresql://quickbite-db:5432/postgres`.
5. Đặt cấu hình `depends_on` để đảm bảo container database khởi chạy trước backend.

#### 3. Kiểm thử và kết quả mong muốn
1. Chạy lệnh build và khởi động:
   ```bash
   docker compose up -d --build
   ```
2. Theo dõi log khởi động của backend:
   ```bash
   docker compose logs -f quickbite-user
   ```
3. **Kết quả mong đợi:** Log Spring Boot khởi chạy thành công, HikariCP kết nối thành công tới Postgres thông qua DNS phân giải từ tên service `quickbite-db`. Không có lỗi kết nối.
4. Truy cập `http://localhost:8081/actuator/health` hiển thị thành công trạng thái `UP`.

#### 4. Hướng dẫn nộp bài
* Đập nhật file `docker-compose.yml` của bạn chứa cả 2 service và nộp lên Git.
* Chụp ảnh màn hình console logs của `quickbite-user` báo kết nối database thành công. Lưu hình ảnh tại: `/homework/session_04/exercise_03/integration_logs.png`.

---

### BÀI TẬP 4 (Mức độ Khá): Khai báo Named Volume bảo vệ bền vững dữ liệu Database

#### 1. Mục tiêu mong muốn đạt được
* **Khai báo và mount Named Volume** chính xác trong file Compose.
* **Xác minh khả năng bảo toàn dữ liệu** của cơ sở dữ liệu qua các chu trình tắt/dựng container.

#### 2. Mô tả yêu cầu
Tránh mất mát dữ liệu database khi thực hiện lệnh dọn dẹp hệ thống:
1. Cập nhật file `docker-compose.yml` khai báo một Named Volume toàn cục ở cuối file đặt tên là `quickbite-db-volume`.
2. Mount volume này vào thư mục lưu trữ dữ liệu `/var/lib/postgresql/data` của service `quickbite-db`.
3. Khởi chạy lại hệ thống.

#### 3. Kiểm thử và kết quả mong muốn
1. Khởi chạy: `docker compose up -d`.
2. Đăng nhập vào database qua container và tạo thử một bảng mẫu:
   ```bash
   docker compose exec quickbite-db psql -U postgres -c "CREATE TABLE test_homework(id serial PRIMARY KEY, name VARCHAR(50));"
   ```
3. Dừng và xóa sạch container:
   ```bash
   docker compose down
   ```
4. Bật lại cụm container:
   ```bash
   docker compose up -d
   ```
5. Kiểm tra lại sự tồn tại của bảng vừa tạo:
   ```bash
   docker compose exec quickbite-db psql -U postgres -c "SELECT * FROM test_homework;"
   ```
6. **Kết quả mong đợi:** Lệnh SQL thực thi thành công mà không báo lỗi thiếu bảng (`relation "test_homework" does not exist`). Chứng tỏ dữ liệu đã được bảo toàn nguyên vẹn.

#### 4. Hướng dẫn nộp bài
* Cập nhật tệp `docker-compose.yml` chứa cấu hình volume và nộp lên Git.
* Chụp màn hình terminal chạy lệnh SELECT kiểm tra bảng thành công sau khi restart. Lưu tại: `/homework/session_04/exercise_04/volume_verify.png`.

---

### BÀI TẬP 5 (Mức độ Giỏi): Phân tách cấu hình bảo mật ra file biến môi trường ngoại vi `.env`

#### 1. Mục tiêu mong muốn đạt được
* **Tách biệt dữ liệu nhạy cảm** ra khỏi file định nghĩa hạ tầng Compose.
* **Áp dụng cơ chế nội suy biến** của Docker Compose.

#### 2. Mô tả yêu cầu
Để đảm bảo an toàn thông tin khi push code lên Git:
1. Tạo một file `.env` nằm cùng thư mục với `docker-compose.yml`.
2. Đưa các thông số nhạy cảm và cổng cấu hình vào file `.env` bao gồm:
   * Tài khoản/mật khẩu database (`DB_USER`, `DB_PASSWORD`).
   * Cổng hoạt động của backend và database (`USER_PORT`, `DB_PORT`).
3. Cập nhật `docker-compose.yml` thay thế các giá trị cứng bằng cú pháp gọi biến nội suy `${...}`.
4. Tạo thêm file `.env.example` mẫu (không điền mật khẩu thật) để làm hướng dẫn.
5. Cập nhật cấu hình loại bỏ cổng database (`ports`) ra máy host, chuyển sang dùng cấu hình bảo mật (chỉ expose nội bộ trong mạng Compose).

#### 3. Kiểm thử và kết quả mong muốn
1. Chạy lệnh config để kiểm tra:
   ```bash
   docker compose config
   ```
2. **Kết quả mong đợi:** File config in ra đầy đủ giá trị thực tế sau khi nội suy, đảm bảo không có lỗi thiếu biến.
3. Chạy `docker compose up -d` và gọi kiểm tra sức khỏe backend thành công.

#### 4. Hướng dẫn nộp bài
* Đẩy file `.env.example` và tệp `docker-compose.yml` lên Git. **Lưu ý:** Không đẩy file `.env` thật (phải thêm `.env` vào file `.gitignore` và verify không xuất hiện trên Git).
* Chụp ảnh màn hình chạy lệnh `docker compose config` in ra cấu hình đầy đủ. Lưu tại: `/homework/session_04/exercise_05/env_config.png`.

---

### BÀI TẬP 6 (Mức độ Xuất sắc): Kịch bản Shell Script vận hành tự động và Smoke Test sức khỏe cụm dịch vụ (`compose-control.sh`)

#### 1. Mục tiêu mong muốn đạt được
* **Tự động hóa hoàn toàn** quy trình khởi động, kiểm tra sức khỏe tích hợp và tắt dọn dẹp hệ thống đa container.
* **Ứng dụng tư duy lập trình Shell Script** nâng cao để chẩn đoán hệ thống và xử lý báo lỗi tự động.

#### 2. Mô tả yêu cầu
Bạn hãy đóng vai trò kỹ sư DevOps viết script `compose-control.sh` để kiểm soát vận hành tự động cụm Docker Compose:
1. Script chấp nhận một tham số đầu vào khi chạy: `./compose-control.sh [start | stop | clean]`.
2. **Khi chạy với tham số `start`:**
   * Tự động khởi chạy cụm dịch vụ: `docker compose up -d --build`.
   * Viết vòng lặp (loop) tạm dừng và chạy kiểm tra trạng thái database: `docker compose exec quickbite-db pg_isready -U postgres` liên tục mỗi 2 giây, tối đa 5 lần thử (timeout 10 giây).
   * Nếu database sẵn sàng trước khi timeout, in thông báo thành công màu xanh lá và tiếp tục gọi thử API Actuator Health của backend (`http://localhost:8081/actuator/health`). Nếu cả hai đều sẵn sàng, in thông báo lớn màu xanh lá: `"HỆ THỐNG QUICKBITE HOẠT ĐỘNG ỔN ĐỊNH!"` và thoát với code `0`.
   * Nếu quá 10 giây database vẫn không sẵn sàng hoặc API backend báo lỗi, in thông báo lỗi màu đỏ kèm theo 20 dòng logs cuối của cụm Compose, sau đó tự động chạy `docker compose down` dọn dẹp và thoát với code `1`.
3. **Khi chạy với tham số `stop`:**
   * Chạy lệnh tạm dừng hệ thống: `docker compose stop`.
4. **Khi chạy với tham số `clean`:**
   * Dọn dẹp sạch sẽ tài nguyên: `docker compose down -v` (xóa cả volumes).

#### 3. Kiểm thử và kết quả mong muốn
1. Chạy lệnh khởi động: `./compose-control.sh start`.
   * **Kết quả mong đợi:** Script chạy, chờ đợi database phản hồi sẵn sàng, kiểm tra API và kết thúc bằng dòng chữ xanh lá thành công.
2. Thử cố tình sửa sai mật khẩu trong file `.env` khiến backend kết nối lỗi và chạy lại script:
   * **Kết quả mong đợi:** Script phát hiện ra lỗi kết nối sau 10 giây, in log lỗi màu đỏ, in logs debug, tự động dọn dẹp container và thoát với code `1`.

#### 4. Hướng dẫn nộp bài
* Đẩy file kịch bản điều khiển `compose-control.sh` lên Git tại: `/homework/session_04/exercise_06/compose-control.sh`.
* Chụp màn hình chạy script thành công in thông báo xanh lá, lưu file thành `control_success.png` trong thư mục trên.
