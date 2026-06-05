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

### BÀI TẬP 5 (Mức độ Giỏi): Khởi chạy đồng thời các dịch vụ Java khác phiên bản JDK không xung đột (Chỉ cung cấp gợi ý)

#### 1. Mục tiêu mong muốn đạt được
* **Chứng minh năng lực cô lập môi trường** (Runtime Isolation) của công nghệ Container mà không cần cài đặt nhiều bộ JDK lên hệ điều hành máy host.
* **Xâu chuỗi lệnh** chạy container để chạy song song nhiều runtime JDK khác nhau.

#### 2. Mô tả yêu cầu
Bạn được giao nhiệm vụ chạy thử nghiệm phiên bản Java của hai dịch vụ: `user-service` (yêu cầu chạy trên **Java 17**) và `order-service` (yêu cầu chạy trên **Java 21**) trên cùng một máy chủ VPS.
1. Hãy tìm cách sử dụng hai image JRE chính thức từ Docker Hub: `eclipse-temurin:17-jre-alpine` và `eclipse-temurin:21-jre-alpine` để khởi chạy hai container song song.
2. Đặt tên container tương ứng là `quickbite-user` và `quickbite-order`.
3. Viết một câu lệnh duy nhất cho mỗi container để khi chạy, container in ra phiên bản Java bên trong (`java -version`) rồi tự động xóa bỏ container ngay lập tức để tiết kiệm bộ nhớ (sử dụng cờ tự dọn dẹp `--rm`).
* *Gợi ý (Hints):* Cú pháp lệnh có dạng `docker run --rm --name [tên] [image] java -version`.

#### 3. Kiểm thử và kết quả mong muốn
* Khi chạy lệnh của container `quickbite-user`: màn hình console in ra thông tin Java version `"17.x.x"`.
* Khi chạy lệnh của container `quickbite-order`: màn hình console in ra thông tin Java version `"21.x.x"`.
* Sau khi chạy xong, lệnh `docker ps -a` xác nhận không còn hai container này tồn tại (đã được tự động xóa sạch).

#### 4. Hướng dẫn nộp bài
* Viết 2 câu lệnh bạn sử dụng vào file script `run-jdk-check.sh`. Đẩy file lên Git tại: `/homework/session_02/exercise_05/run-jdk-check.sh`.
* Chụp màn hình terminal hiển thị kết quả in ra phiên bản Java của cả 2 container lưu thành `jdk_isolation.png` trong thư mục trên.

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
