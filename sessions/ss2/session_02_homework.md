# SESSION 02: BÀI TẬP VỀ NHÀ (HOMEWORK)
## Đề tài: Thiết lập Container, Cô lập Môi trường & Script tự động hóa staging

Tài liệu này chứa 6 bài tập thực hành dành cho Session 2 của hệ thống QuickBite. Bạn cần hoàn thành đầy đủ các bài tập và nộp bài theo hướng dẫn ở cuối mỗi bài. Kết quả được đánh giá theo tiêu chí Đạt (Pass) hoặc Không đạt (Fail).

---

### BÀI TẬP 1 (Mức độ Khá): Khởi chạy và chẩn đoán container database PostgreSQL

#### 1. Mục tiêu mong muốn đạt được
* **Khởi chạy thành công** một container dịch vụ chạy ngầm sử dụng cờ detached mode (`-d`).
* **Cấu hình chính xác** ánh xạ cổng (`-p`) và đặt tên định danh (`--name`) cho container database.

#### 2. Mô tả yêu cầu
Bạn cần chuẩn bị cơ sở dữ liệu PostgreSQL cho dịch vụ `user-service` hoạt động trong container:
1. Hãy khởi chạy một container database PostgreSQL phiên bản `15-alpine` từ Docker Hub.
2. Thiết lập cấu hình khởi chạy thỏa mãn các yêu cầu:
   * Chạy ngầm dưới nền (Detached mode).
   * Đặt tên container rõ ràng là `quickbite-db`.
   * Ánh xạ cổng (Port mapping) từ cổng `5432` của máy host vào cổng `5432` nội bộ của container.
   * Truyền mật khẩu quản trị database thông qua biến môi trường `POSTGRES_PASSWORD=secret`.

#### 3. Kiểm thử và kết quả mong muốn
1. Gõ lệnh liệt kê các container đang hoạt động:
   ```bash
   docker ps
   ```
2. **Kết quả mong đợi:** Xuất hiện một dòng thông tin container tên `quickbite-db`, trạng thái `Up`, cổng `0.0.0.0:5432->5432/tcp`.
3. Gõ lệnh kiểm tra danh sách image đang lưu ở local:
   ```bash
   docker images
   ```
4. **Kết quả mong đợi:** Hiển thị image `postgres` với tag `15-alpine`.

#### 4. Hướng dẫn nộp bài
* Chụp ảnh màn hình terminal chạy lệnh `docker ps` hiển thị rõ container `quickbite-db` đang hoạt động.
* Lưu hình ảnh vào thư mục Git cá nhân của bạn tại: `/homework/session_02/exercise_01/db_status.png`.

---

### BÀI TẬP 2 (Mức độ Khá): Kết nối dịch vụ `user-service` từ máy host vào Database trong container

#### 1. Mục tiêu mong muốn đạt được
* **Thiết lập kết nối liên môi trường** (Cross-environment connection) từ ứng dụng chạy trực tiếp trên máy host vào dịch vụ chạy trong Docker container.
* **Cấu hình động** thông số kết nối thông qua biến môi trường.

#### 2. Mô tả yêu cầu
Bây giờ, chúng ta sẽ kết nối dịch vụ `user-service` (đang chạy trực tiếp ở máy local/host) vào database `quickbite-db` vừa dựng ở Bài 1:
1. Khai báo các thông số kết nối PostgreSQL vào biến môi trường hệ thống (hoặc nạp qua file cấu hình chạy ứng dụng):
   * URL kết nối trỏ về: `jdbc:postgresql://localhost:5432/postgres` (hoặc tên database mặc định).
   * Tài khoản: `postgres`, mật khẩu: `secret`.
2. Khởi chạy ứng dụng `user-service` bằng Gradle wrapper (`./gradlew bootRun`) hoặc chạy trực tiếp file JAR đã build.

#### 3. Kiểm thử và kết quả mong muốn
1. Xem log console khởi động của ứng dụng `user-service`.
2. **Kết quả mong đợi:** Ứng dụng khởi chạy thành công mà không gặp lỗi kết nối database (`Connection refused`). Log hiển thị dòng chữ khởi tạo kết nối HikariCP thành công tới PostgreSQL.

#### 4. Hướng dẫn nộp bài
* Chụp ảnh màn hình console log của `user-service` hiển thị rõ các dòng log kết nối database thành công.
* Lưu hình ảnh vào thư mục Git: `/homework/session_02/exercise_02/user_service_db_connect.png`.

---

### BÀI TẬP 3 (Mức độ Khá): Truy cập và tương tác nội bộ với database PostgreSQL (`docker exec`)

#### 1. Mục tiêu mong muốn đạt được
* **Sử dụng thành thạo** lệnh `docker exec -it` để truy cập vào shell bên trong một container đang hoạt động.
* **Vận hành công cụ dòng lệnh** của database để truy vấn và kiểm tra trạng thái dữ liệu nội bộ.

#### 2. Mô tả yêu cầu
Anh Tech Lead yêu cầu bạn kiểm tra danh sách các cơ sở dữ liệu hiện có trực tiếp từ bên trong container `quickbite-db`:
1. Sử dụng lệnh `docker exec` kết hợp cờ tương tác để mở shell `sh` truy cập vào container `quickbite-db`.
2. Đăng nhập vào trình quản trị `psql` của PostgreSQL dưới tư cách user `postgres`.
3. Chạy lệnh liệt kê danh sách database đang có trong hệ thống Postgres.

#### 3. Kiểm thử và kết quả mong muốn
1. Chạy lệnh đăng nhập và truy vấn bên trong container:
   ```bash
   psql -U postgres
   \l
   ```
2. **Kết quả mong đợi:** Danh sách các database mặc định (`postgres`, `template0`...) hiển thị ra dưới dạng bảng.
3. Gõ `\q` và `exit` để quay lại terminal máy host. Đảm bảo container `quickbite-db` vẫn hoạt động bình thường (`Up`) qua lệnh `docker ps`.

#### 4. Hướng dẫn nộp bài
* Chụp ảnh màn hình terminal hiển thị bảng danh sách các database (`\l`) khi đang ở trong shell container.
* Lưu hình ảnh vào thư mục Git: `/homework/session_02/exercise_03/psql_inspect.png`.

---

### BÀI TẬP 4 (Mức độ Khá): Giám sát log thời gian thực của container (`docker logs -f`)

#### 1. Mục tiêu mong muốn đạt được
* **Ứng dụng cơ chế trích xuất log** của Docker để theo dõi hoạt động và phát hiện lỗi kết nối.
* **Sử dụng cờ `-f`** để stream log thời gian thực (real-time).

#### 2. Mô tả yêu cầu
Bạn cần giám sát xem database `quickbite-db` có thực sự nhận được yêu cầu kết nối từ `user-service` hay không:
1. Mở một cửa sổ Terminal mới, gõ lệnh theo dõi log liên tục (follow log) của container `quickbite-db`.
2. Khởi chạy ứng dụng `user-service` hoặc gọi một API đăng ký/đăng nhập để kích hoạt dòng kết nối tới database.
3. Quan sát các dòng log mới xuất hiện trong Terminal giám sát.

#### 3. Kiểm thử và kết quả mong muốn
1. Lệnh chạy giám sát:
   ```bash
   docker logs -f quickbite-db
   ```
2. **Kết quả mong đợi:** Khi `user-service` khởi động hoặc gọi API, Terminal giám sát lập tức in thêm các dòng log kết nối dạng: `connection received` hoặc `database system is ready to accept connections`.

#### 4. Hướng dẫn nộp bài
* Chụp ảnh màn hình Terminal đang stream log hiển thị rõ các dòng log kết nối được ghi nhận từ ứng dụng.
* Lưu hình ảnh vào thư mục Git: `/homework/session_02/exercise_04/postgres_logs.png`.

---

### BÀI TẬP 5 (Mức độ Giỏi): Khởi chạy dịch vụ `user-service` thực tế trong container JRE bằng cơ chế Mount thư mục (Volume Mounting)

#### 1. Mục tiêu mong muốn đạt được
* **Chạy ứng dụng Spring Boot thực tế** trong môi trường container sử dụng image JRE chính thức từ Docker Hub mà không cần viết Dockerfile.
* **Vận dụng cơ chế Volume Mounting (`-v`)** để chia sẻ file thực thi từ máy host vào bên trong container.
* **Cấu hình môi trường mạng và database** thông qua biến môi trường (`-e`) và cơ chế định tuyến ngược về host.

#### 2. Mô tả yêu cầu
Bạn cần triển khai dịch vụ `user-service` thực tế chạy trong một container JRE mà chưa cần viết Dockerfile:
1. Thực hiện biên dịch (compile) ứng dụng `user-service` trên máy host thành file JAR.
   * *Gợi ý:* Chạy lệnh `./gradlew bootJar` hoặc `./gradlew build -x test` ở thư mục dự án `user-service`. Xác định tên file JAR được sinh ra trong thư mục `build/libs` (thường là `user-service-0.0.1-SNAPSHOT.jar` hoặc tương tự).
2. Viết câu lệnh `docker run` khởi chạy container từ image JRE 17 chính thức `eclipse-temurin:17-jre-alpine` thỏa mãn các tiêu chí sau:
   * Khởi chạy ngầm dưới nền (Detached mode) và đặt tên container là `quickbite-user`.
   * Ánh xạ cổng `-p 8081:8081` để máy host hoặc trình duyệt có thể gọi API vào container.
   * Sử dụng cờ `-v` để mount thư mục chứa file JAR trên máy host (sử dụng đường dẫn tuyệt đối, ví dụ: `C:/Users/Nathan/backend_java_devops/java_backend_devops/user-service/build/libs`) vào thư mục `/app` trong container.
   * Đặt thư mục làm việc mặc định của container là `/app` bằng cờ `-w /app`.
   * Sử dụng cờ `--add-host=host.docker.internal:host-gateway` để container có thể phân giải tên miền `host.docker.internal` trỏ về IP của máy host (nơi database `quickbite-db` đã mở cổng `5432` ở Bài tập 1).
   * Truyền các cấu hình kết nối database thông qua biến môi trường Spring Boot:
     - `-e SPRING_DATASOURCE_URL=jdbc:postgresql://host.docker.internal:5432/postgres` (hoặc tên db của bạn).
     - `-e SPRING_DATASOURCE_USERNAME=postgres`
     - `-e SPRING_DATASOURCE_PASSWORD=secret`
   * Câu lệnh thực thi cuối container là: `java -jar [tên_file_jar_của_bạn].jar` (ví dụ: `java -jar user-service-0.0.1-SNAPSHOT.jar`).

#### 3. Kiểm thử và kết quả mong muốn
1. Khởi chạy container và xem log:
   ```bash
   docker logs quickbite-user
   ```
2. **Kết quả mong đợi:** Ứng dụng Spring Boot khởi chạy thành công bên trong container, log hiển thị HikariCP kết nối thành công tới database Postgres qua `host.docker.internal:5432` mà không gặp lỗi kết nối.
3. Mở trình duyệt truy cập: `http://localhost:8081/actuator/health` (hoặc endpoint API có sẵn của dịch vụ).
4. **Kết quả mong đợi:** Phản hồi trạng thái hoạt động thành công (ví dụ: `{"status":"UP"}`).

#### 4. Hướng dẫn nộp bài
* Viết câu lệnh `docker run` hoàn chỉnh của bạn vào file script `run_user_service.sh` và nộp lên Git tại: `/homework/session_02/exercise_05/run_user_service.sh`.
* Chụp màn hình terminal hiển thị log Spring Boot khởi động thành công của container `quickbite-user` lưu thành `user_service_container.png` trong thư mục trên.

---

### BÀI TẬP 6 (Mức độ Xuất sắc): Kịch bản shell script dọn dẹp và khởi tạo môi trường Staging tự động (`init-staging.sh`)

#### 1. Mục tiêu mong muốn đạt được
* **Tự động hóa hoàn toàn** (End-to-end Automation) quy trình dọn dẹp tài nguyên cũ và khởi tạo môi trường database sạch sẽ.
* **Áp dụng tư duy lập trình Shell Script** để kiểm tra xung đột cổng mạng hệ thống và kiểm tra độ sẵn sàng của dịch vụ (Health Check/Smoke Test).

#### 2. Mô tả yêu cầu
Bạn hãy đóng vai trò là một kỹ sư DevOps thiết lập script `init-staging.sh` để tự động hóa quy trình làm sạch và dựng môi trường Staging cho database của QuickBite. Kịch bản script cần thực hiện các bước sau:
1. **Bước 1: Làm sạch container cũ:**
   * Script tự động kiểm tra xem trên hệ thống có container nào (dù đang chạy hay đã dừng) mang tên `quickbite-db` hay chưa.
   * Nếu có, thực hiện dừng (`docker stop`) và xóa bỏ hoàn toàn (`docker rm`) container đó để tránh lỗi xung đột tên. In ra thông báo đã dọn dẹp container cũ.
2. **Bước 2: Kiểm tra xung đột cổng mạng vật lý:**
   * Trước khi chạy container mới, script phải kiểm tra xem cổng `5432` của máy host có đang bị chiếm dụng bởi một tiến trình PostgreSQL cài trực tiếp (native) trên hệ điều hành hay không (gợi ý: sử dụng lệnh `ss` hoặc `netstat` kết hợp `grep`).
   * Nếu cổng `5432` bị chiếm dụng bởi tiến trình ngoài Docker, in thông báo lỗi màu đỏ (sử dụng mã màu ANSI) và dừng script lập tức với exit code `1`.
3. **Bước 3: Khởi tạo database mới:**
   * Khởi chạy một container PostgreSQL mới từ image `postgres:15-alpine` chạy ngầm, tên `quickbite-db`, map port `5432:5432` và mật khẩu `secret`.
4. **Bước 4: Kiểm thử độ sẵn sàng (Smoke Test):**
   * Script tạm dừng (sleep) 5 giây để chờ PostgreSQL khởi động tiến trình.
   * Tự động chạy lệnh kiểm tra trạng thái database không cần tương tác: `docker exec quickbite-db pg_isready -U postgres` để xem database đã sẵn sàng tiếp nhận kết nối chưa.
   * Nếu database phản hồi sẵn sàng (exit code của lệnh `pg_isready` trả về `0`), in thông báo thành công màu xanh lá cây: `"DATABASE STAGING KHỞI TẠO THÀNH CÔNG!"` và thoát với exit code `0`.
   * Nếu database bị lỗi khởi động (exit code khác `0`), in thông báo lỗi đỏ và in 20 dòng log cuối cùng của container bằng lệnh `docker logs` để hỗ trợ debug.

#### 3. Kiểm thử và kết quả mong muốn
1. Chạy thử script: `sudo ./init-staging.sh`.
   * **Kết quả mong đợi:** Script dọn dẹp container cũ (nếu có), kiểm tra cổng mạng, khởi chạy container mới và in ra dòng thông báo màu xanh lá báo thành công ở cuối.
2. Cố tình chạy một dịch vụ native chiếm cổng `5432` trên máy host (hoặc sửa port map của container mới sang cổng đang bị chiếm) rồi chạy lại script:
   * **Kết quả mong đợi:** Script lập tức báo lỗi đỏ ở Bước 2 và dừng hoạt động, không thực hiện các bước sau.

#### 4. Hướng dẫn nộp bài
* Đẩy file kịch bản thông minh `init-staging.sh` lên Git cá nhân tại: `/homework/session_02/exercise_06/init-staging.sh`.
* Chụp màn hình terminal chạy script khởi tạo thành công in màu xanh lá, lưu file thành `staging_success.png` trong thư mục trên.
