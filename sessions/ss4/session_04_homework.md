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

### BÀI TẬP 2 (Mức độ Khá): Thiết lập Docker Compose khởi chạy database PostgreSQL dùng chung

#### 1. Mục tiêu mong muốn đạt được
* **Biên soạn đúng cú pháp YAML** cho tệp tin `docker-compose.yml` của database.
* **Tích hợp script SQL tự động** để tạo các user và database biệt lập cho cả 4 dịch vụ trong toàn bộ khóa học.
* **Khai báo và khởi chạy** cơ sở dữ liệu Postgres dùng chung qua Docker Compose.

#### 2. Mô tả yêu cầu
Bạn hãy đóng vai trò kỹ sư DevOps thiết lập hệ quản trị cơ sở dữ liệu trung tâm cho QuickBite:
1. Tạo thư mục đặt tên là `quickbite-database` nằm ngoài thư mục ứng dụng backend.
2. Bên trong thư mục này, tạo file SQL khởi tạo `init-db.sql` có nội dung giống như đã hướng dẫn ở Phần 5.2 của Bài học 1 (tạo user, database và grant privileges cho cả 4 dịch vụ: `user`, `restaurant`, `order`, `notification`).
3. Tạo file `docker-compose.yml` cấu hình chạy PostgreSQL 15-alpine:
   * Tên service: `quickbite-db`.
   * Đặt tên container cố định: `container_name: quickbite-db`.
   * Ánh xạ cổng `5432:5432`.
   * Mount Named Volume `quickbite-db-volume` vào thư mục lưu trữ dữ liệu của Postgres.
   * Mount file `init-db.sql` vào `/docker-entrypoint-initdb.d/init-db.sql`.
   * Khai báo mạng ảo chung là `quickbite-net` với driver `bridge` và đặt tên cố định mạng này là `quickbite-net`.

#### 3. Kiểm thử và kết quả mong muốn
1. Khởi chạy cụm database ở chế độ chạy ngầm:
   ```bash
   docker compose up -d
   ```
2. Kiểm tra trạng thái container, volume, và network:
   ```bash
   docker compose ps
   docker volume ls
   docker network ls
   ```
3. **Kết quả mong đợi:** Container `quickbite-db` chạy thành công (STATUS = Up), volume và mạng ảo `quickbite-net` được khởi tạo thành công.
4. Kiểm tra việc khởi tạo tự động các database con (Ví dụ `quickbite_user_db`):
   ```bash
   docker exec -it quickbite-db psql -U postgres -l
   ```
   *(Console phải hiển thị danh sách các database con đã được tạo thành công như quickbite_user_db, quickbite_restaurant_db, v.v.).*

#### 4. Hướng dẫn nộp bài
* Đẩy thư mục `quickbite-database/` chứa tệp `docker-compose.yml` và `init-db.sql` lên Git.
* Chụp ảnh màn hình danh sách cơ sở dữ liệu hiển thị trên terminal sau khi chạy lệnh kiểm tra database con. Lưu hình ảnh tại: `/homework/session_04/exercise_02/databases_created.png`.

---

### BÀI TẬP 3 (Mức độ Khá): Liên kết backend tự build và database qua mạng ảo dùng chung

#### 1. Mục tiêu mong muốn đạt được
* **Ứng dụng thuộc tính `build`** để Compose tự động build image từ Dockerfile local của backend.
* **Cấu hình liên kết mạng ngoài (`external`)** để kết nối microservice với database đang chạy ngầm trên mạng ảo chung.

#### 2. Mô tả yêu cầu
Bây giờ, hãy tạo tệp Compose để chạy dịch vụ backend `user-service` kết nối tới database dùng chung:
1. Tạo thư mục đặt tên là `quickbite-project`. Di chuyển thư mục `user-service` chứa mã nguồn của bạn vào trong `quickbite-project/`.
2. Tạo file `docker-compose.yml` tại `/quickbite-project/docker-compose.yml`.
3. Khai báo service `quickbite-user` tự động build từ thư mục `./user-service`.
4. Ánh xạ cổng `8081:8081` ra ngoài máy host.
5. Thiết lập biến môi trường kết nối cơ sở dữ liệu cho Spring Boot kết nối tới database `quickbite-db` (cổng `5432`, db name `quickbite_user_db`, user `quickbite_user`, password `quickbite_user`).
6. Khai báo tham gia mạng ảo `quickbite-net` và chỉ định đây là mạng ngoài (`external: true`).

#### 3. Kiểm thử và kết quả mong muốn
1. Khởi chạy dịch vụ backend kèm build image mới:
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
* Đẩy tệp `/quickbite-project/docker-compose.yml` lên Git.
* Chụp ảnh màn hình console logs của `quickbite-user` báo kết nối database thành công. Lưu hình ảnh tại: `/homework/session_04/exercise_03/integration_logs.png`.

---

### BÀI TẬP 4 (Mức độ Khá): Xác minh khả năng bảo toàn dữ liệu của Database qua chu trình tắt/dựng độc lập

#### 1. Mục tiêu mong muốn đạt được
* **Hiểu bản chất** hoạt động của Named Volume trên container database chạy độc lập.
* **Xác minh dữ liệu được giữ an toàn** ngay cả khi container database bị xóa hoàn toàn.

#### 2. Mô tả yêu cầu
Thực hiện chu trình dọn dẹp và khởi tạo lại database để kiểm tra tính toàn vẹn dữ liệu:
1. Sử dụng lệnh CLI toàn cục để truy cập database `quickbite_user_db` và tạo một bảng mẫu:
   ```bash
   docker exec -it quickbite-db psql -U quickbite_user -d quickbite_user_db -c "CREATE TABLE homework_table (id serial PRIMARY KEY, val VARCHAR(50)); INSERT INTO homework_table (val) VALUES ('Dữ liệu bài tập 4 an toàn!');"
   ```
2. Thực hiện xóa toàn bộ container database (di chuyển vào thư mục `quickbite-database` và chạy lệnh):
   ```bash
   docker compose down
   ```
3. Khởi chạy lại container database:
   ```bash
   docker compose up -d
   ```
4. Thực hiện kiểm tra lại dữ liệu trong bảng `homework_table`.

#### 3. Kiểm thử và kết quả mong muốn
1. Chạy truy vấn SQL từ terminal để lấy dữ liệu từ bảng vừa tạo:
   ```bash
   docker exec -it quickbite-db psql -U quickbite_user -d quickbite_user_db -c "SELECT * FROM homework_table;"
   ```
2. **Kết quả mong đợi:** Lệnh SQL thực thi thành công và hiển thị dòng dữ liệu `'Dữ liệu bài tập 4 an toàn!'` mà không báo lỗi thiếu bảng. Chứng tỏ Named Volume `quickbite-db-volume` hoạt động hoàn hảo.

#### 4. Hướng dẫn nộp bài
* Chụp ảnh màn hình terminal chạy truy vấn SQL thành công hiển thị rõ chuỗi dữ liệu sau khi dựng lại container database. Lưu tại: `/homework/session_04/exercise_04/volume_verify.png`.

---

### BÀI TẬP 5 (Mức độ Giỏi): Phân tách cấu hình bảo mật ra file biến môi trường ngoại vi `.env`

#### 1. Mục tiêu mong muốn đạt được
* **Tách biệt thông tin nhạy cảm** (tài khoản kết nối DB, cổng dịch vụ) ra khỏi mã nguồn hạ tầng YAML của backend.
* **Áp dụng cơ chế nội suy biến** của Docker Compose.
* **Tổ chức biến môi trường khoa học**, tránh xung đột cấu hình giữa các dịch vụ trong hệ thống Microservices bằng cách tiền tố hóa (prefix) các biến riêng của từng service.

#### 2. Mô tả yêu cầu
Để đảm bảo an toàn thông tin khi push file Compose lên Git:
1. Tạo một file `.env` nằm trong thư mục `/quickbite-project/`.
2. Đưa các thông số cấu hình và mật khẩu vào file `.env` bao gồm:
   * Các biến dùng chung cho database: `DB_HOST`, `DB_PORT`.
   * Các biến riêng biệt dành cho `user-service` (được tiền tố hóa bằng `USER_` để tránh xung đột với các service khác sau này): `USER_DB_NAME`, `USER_DB_USERNAME`, `USER_DB_PASSWORD`, `USER_SERVER_PORT`, `USER_JWT_SECRET_KEY`, `USER_JWT_EXPIRED_ACCESS`, `USER_JWT_EXPIRED_REFRESH`.
3. Cập nhật file `/quickbite-project/docker-compose.yml` thay thế các giá trị cứng bằng cú pháp gọi biến nội suy `${...}` và ánh xạ chúng vào các biến môi trường mà container Java yêu cầu (ví dụ: `DB_NAME=${USER_DB_NAME}`).
4. Tạo thêm file `.env.example` mẫu (không điền mật khẩu thật) đặt trong thư mục dự án.
5. Cập nhật cấu hình loại bỏ cổng database (`ports`) ra ngoài host ở file `.env` (nếu có), đảm bảo database chỉ kết nối nội bộ.

#### 3. Kiểm thử và kết quả mong muốn
1. Đứng tại thư mục `/quickbite-project/` chạy lệnh kiểm tra cấu hình:
   ```bash
   docker compose config
   ```
2. **Kết quả mong đợi:** Cấu hình YAML được hiển thị đầy đủ giá trị thực tế sau khi nội suy biến, không có lỗi cú pháp hoặc thiếu biến.
3. Chạy `docker compose up -d` và gọi API kiểm tra sức khỏe backend thành công.

#### 4. Hướng dẫn nộp bài
* Đẩy file `.env.example` và tệp `docker-compose.yml` (phiên bản Bài 5) lên Git tại thư mục `/quickbite-project/`. **Bắt buộc:** Thêm `.env` vào file `.gitignore` để không bị push lên Git.
* Chụp ảnh màn hình chạy lệnh `docker compose config` hiển thị rõ cấu hình nạp biến thành công. Lưu tại: `/homework/session_04/exercise_05/env_config.png`.

---

### BÀI TẬP 6 (Mức độ Xuất sắc): Kịch bản Shell Script vận hành tự động và Smoke Test sức khỏe dịch vụ (`compose-control.sh`)

#### 1. Mục tiêu mong muốn đạt được
* **Tự động hóa hoàn toàn** quy trình khởi động, kiểm tra sức khỏe liên kết đa container (database chạy ngầm và backend chạy động).
* **Áp dụng tư duy lập trình Shell Script** để chẩn đoán hệ thống và xử lý dọn dẹp tự động khi có lỗi xảy ra.

#### 2. Mô tả yêu cầu
Bạn hãy đóng vai trò kỹ sư DevOps viết script `compose-control.sh` đặt tại `/quickbite-project/` để kiểm soát vận hành tự động:
1. Script chấp nhận một tham số đầu vào khi chạy: `./compose-control.sh [start | stop | clean]`.
2. **Khi chạy với tham số `start`:**
   * Tự động khởi chạy dịch vụ backend: `docker compose up -d --build`.
   * Viết vòng lặp (loop) kiểm tra trạng thái của container database `quickbite-db` bằng câu lệnh toàn cục: `docker exec quickbite-db pg_isready -U postgres` liên tục mỗi 2 giây, tối đa 5 lần thử (timeout 10 giây).
   * Nếu database sẵn sàng trước khi timeout, in thông báo thành công màu xanh lá và tiếp tục gọi thử API Actuator Health của backend (`http://localhost:8081/actuator/health`). Nếu cả hai đều sẵn sàng, in thông báo lớn màu xanh lá: `"HỆ THỐNG QUICKBITE HOẠT ĐỘNG ỔN ĐỊNH!"` và thoát với code `0`.
   * Nếu quá 10 giây database vẫn không sẵn sàng hoặc API backend báo lỗi, in thông báo lỗi màu đỏ kèm theo 20 dòng logs cuối của cụm backend, sau đó tự động chạy `docker compose down` dọn dẹp backend và thoát với code `1`.
3. **Khi chạy với tham số `stop`:**
   * Chạy lệnh tạm dừng backend: `docker compose stop`.
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
