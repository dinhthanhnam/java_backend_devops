# SESSION 03: BÀI TẬP VỀ NHÀ TỔNG HỢP (INTEGRATED HOMEWORK)
## Đề tài: Quản trị Hệ điều hành & Triển khai Độc lập Đa dịch vụ bằng Docker CLI

Tài liệu này chứa 6 bài tập thực hành tổng hợp tích hợp toàn bộ kiến thức về Linux, Shell Scripting từ Session 1 và Docker Basics từ Session 2. Bạn cần hoàn thành đầy đủ và nộp bài theo đúng cấu trúc Git quy định. Kết quả được đánh giá theo tiêu chí Đạt (Pass) hoặc Không đạt (Fail).

---

### BÀI TẬP 1 (Mức độ Khá): Thiết lập môi trường Linux & Phân quyền bảo mật cho Staging

#### 1. Mục tiêu mong muốn đạt được
* **Thành thạo quản lý người dùng và nhóm** trong Linux.
* **Áp dụng phân quyền chặt chẽ** (Ownership & Permissions) để bảo vệ mã nguồn và file thực thi trên server Staging theo tiêu chuẩn bảo mật (không chạy bằng user `root`).

#### 2. Mô tả yêu cầu
Bạn hãy đóng vai trò quản trị viên hệ thống (SysAdmin) thiết lập thư mục và quyền chạy cho ứng dụng QuickBite trên Staging server (môi trường Ubuntu Linux):
1. Tạo một nhóm người dùng hệ thống mới tên là `quickbite`.
2. Tạo một tài khoản hệ thống mới tên là `quickbite` thuộc nhóm `quickbite` (không cấp quyền đăng nhập shell trực tiếp và không tạo home directory để tăng tính an toàn, sử dụng tùy chọn `-r` và shell `/bin/false`).
3. Tạo thư mục làm việc chính của dự án tại `/opt/quickbite/`.
4. Thay đổi quyền sở hữu (Ownership) của thư mục `/opt/quickbite/` thuộc về user `quickbite` và group `quickbite`.
5. Cấp quyền truy cập (Permissions) cho thư mục này ở mức `750` (`rwxr-x---`) để đảm bảo chỉ có chủ sở hữu và thành viên trong nhóm mới được phép đọc và thực thi dữ liệu bên trong.

#### 3. Kiểm thử và kết quả mong muốn
1. Kiểm tra sự tồn tại của user và group bằng cách đọc file `/etc/passwd` và `/etc/group` hoặc chạy lệnh:
   ```bash
   id quickbite
   ```
   **Kết quả mong đợi:** Hiển thị rõ user `quickbite` thuộc nhóm `quickbite`.
2. Kiểm tra chi tiết phân quyền của thư mục vừa tạo:
   ```bash
   ls -ld /opt/quickbite
   ```
   **Kết quả mong đợi:** In ra quyền truy cập dạng `drwxr-x---` với chủ sở hữu là `quickbite` và nhóm sở hữu là `quickbite`.

#### 4. Hướng dẫn nộp bài
* Lưu lại các lệnh Linux bạn đã sử dụng để tạo và cấu hình quyền vào file văn bản tại thư mục Git cá nhân: `/homework/session_03/exercise_01/setup_env.sh`.
* Chụp màn hình terminal chạy lệnh kiểm tra `id quickbite` và `ls -ld /opt/quickbite`. Lưu hình ảnh vào: `/homework/session_03/exercise_01/permission_evidence.png`.

---

### BÀI TẬP 2 (Mức độ Khá): Triển khai và kiểm tra cơ sở dữ liệu PostgreSQL Container

#### 1. Mục tiêu mong muốn đạt được
* **Khởi chạy thành công** container dịch vụ cơ sở dữ liệu từ Docker Hub.
* **Sử dụng công cụ chẩn đoán** đi kèm để kiểm tra trạng thái sẵn sàng kết nối của database.

#### 2. Mô tả yêu cầu
Khởi chạy hạ tầng dữ liệu cho QuickBite bằng cách chạy container PostgreSQL 15:
1. Tải image `postgres:15-alpine` từ Docker Hub về máy.
2. Khởi chạy một container đặt tên chính xác là `quickbite-db` ở chế độ chạy ngầm (detached mode `-d`).
3. Ánh xạ cổng mặc định `-p 5432:5432` ra ngoài máy host.
4. Thiết lập mật khẩu quản trị database là `secret` thông qua biến môi trường `POSTGRES_PASSWORD`.

#### 3. Kiểm thử và kết quả mong muốn
1. Xem danh sách các container đang hoạt động:
   ```bash
   docker ps
   ```
   **Kết quả mong đợi:** Container `quickbite-db` ở trạng thái `Up`, cổng `5432` được mở.
2. Sử dụng công cụ chẩn đoán `pg_isready` tích hợp sẵn trong container PostgreSQL để kiểm tra xem database đã sẵn sàng tiếp nhận kết nối mạng chưa:
   ```bash
   docker exec quickbite-db pg_isready -U postgres
   ```
   **Kết quả mong đợi:** Console in ra dòng chữ: `/var/run/postgresql:5432 - accepting connections`.

#### 4. Hướng dẫn nộp bài
* Lưu câu lệnh khởi chạy container Docker của bạn vào file script tại: `/homework/session_03/exercise_02/deploy_db.sh`.
* Chụp màn hình terminal in ra kết quả của lệnh `docker exec quickbite-db pg_isready -U postgres`. Lưu hình ảnh vào: `/homework/session_03/exercise_02/db_ready.png`.

---

### BÀI TẬP 3 (Mức độ Khá): Triển khai thủ công Spring Boot `user-service` kết nối Database qua Docker IP

#### 1. Mục tiêu mong muốn đạt được
* **Biên dịch mã nguồn** Java Spring Boot thành tệp JAR thực thi.
* **Triển khai ứng dụng Java** chạy cô lập bên trong container bằng cách mount file JAR và truyền động biến kết nối database thủ công qua IP.

#### 2. Mô tả yêu cầu
Bạn hãy đóng vai trò kỹ sư triển khai chạy dịch vụ `user-service` (Java 17, cổng hoạt động 8081) kết nối tới database `quickbite-db` đã tạo ở Bài tập 2:
1. Biên dịch mã nguồn của dịch vụ `user-service` trên máy local thành file thực thi `user-service.jar` (nằm trong thư mục `build/libs/`).
2. Do chưa học về Docker Network, bạn cần tìm địa chỉ IP nội bộ của container `quickbite-db` đang chạy bằng lệnh chẩn đoán của Docker CLI.
3. Khởi chạy một container backend đặt tên là `quickbite-user` sử dụng base image JRE 17 gọn nhẹ (`eclipse-temurin:17-jre-alpine`).
4. Sử dụng cơ chế **Volume Mounting** để mount thư mục chứa file JAR đã build trên host vào thư mục `/app` trong container.
5. Thiết lập cổng ánh xạ ra host `-p 8081:8081`.
6. Truyền địa chỉ IP của container database vừa tìm được vào biến môi trường kết nối database của Spring Boot:
   * `SPRING_DATASOURCE_URL=jdbc:postgresql://<DB_IP_INTERNAL>:5432/postgres`
   * `SPRING_DATASOURCE_USERNAME=postgres`
   * `SPRING_DATASOURCE_PASSWORD=secret`
7. Khởi chạy ứng dụng bằng lệnh: `java -jar /app/user-service.jar`.

#### 3. Kiểm thử và kết quả mong muốn
1. Tìm IP của container DB:
   ```bash
   docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' quickbite-db
   # Ví dụ trả về: 172.17.0.2
   ```
2. Khởi chạy container backend (truyền IP vừa tìm được vào lệnh):
   ```bash
   docker run -d --name quickbite-user -p 8081:8081 -v /absolute/path/to/user-service/build/libs:/app -w /app -e SPRING_DATASOURCE_URL=jdbc:postgresql://172.17.0.2:5432/postgres -e SPRING_DATASOURCE_USERNAME=postgres -e SPRING_DATASOURCE_PASSWORD=secret eclipse-temurin:17-jre-alpine java -jar user-service.jar
   ```
3. Xem logs khởi động của container backend:
   ```bash
   docker logs quickbite-user
   ```
   **Kết quả mong đợi:** Log Spring Boot khởi chạy thành công, HikariCP kết nối thành công tới database PostgreSQL mà không gặp bất cứ lỗi kết nối nào.

#### 4. Hướng dẫn nộp bài
* Lưu lại lệnh chạy container `quickbite-user` của bạn vào file tại: `/homework/session_03/exercise_03/deploy_user.sh`.
* Chụp màn hình logs khởi động thành công của `quickbite-user` kết nối database. Lưu hình ảnh vào: `/homework/session_03/exercise_03/user_connected.png`.

---

### BÀI TẬP 4 (Mức độ Khá): Giám sát trạng thái hoạt động & Xử lý sự cố logs hệ thống

#### 1. Mục tiêu mong muốn đạt được
* **Sử dụng thành thạo** các lệnh giám sát thời gian thực (`docker logs -f` và `docker stats`).
* **Truy vấn và chẩn đoán** sức khỏe của dịch vụ thông qua Actuator endpoints.

#### 2. Mô tả yêu cầu
Thực hiện giám sát và kiểm tra hiệu năng hệ thống đang chạy:
1. Theo dõi logs hoạt động liên tục (stream logs) của container `quickbite-user` để bắt các dòng log khi có request truy cập.
2. Kiểm tra mức độ tiêu thụ tài nguyên phần cứng (CPU, RAM, Network I/O) của cả hai container `quickbite-db` và `quickbite-user` đang chạy trên máy.
3. Gửi request kiểm tra sức khỏe hệ thống (Health Check) tới dịch vụ `user-service` thông qua cổng máy host.

#### 3. Kiểm thử và kết quả mong muốn
1. Chạy lệnh theo dõi tài nguyên động:
   ```bash
   docker stats --no-stream
   ```
   **Kết quả mong đợi:** Hiển thị danh sách các container đang chạy kèm tỷ lệ % CPU, dung lượng RAM sử dụng thực tế (user-service JRE container sẽ tiêu hao khoảng 100-200MB RAM, cực kỳ nhỏ so với VM).
2. Gọi API Health Check của Spring Boot Actuator:
   ```bash
   curl http://localhost:8081/actuator/health
   ```
   **Kết quả mong đợi:** Trả về JSON hiển thị trạng thái hoạt động của service: `{"status":"UP"}`.

#### 4. Hướng dẫn nộp bài
* Chụp màn hình terminal chạy lệnh `docker stats` và màn hình in ra kết quả lệnh `curl` gọi API Actuator Health.
* Lưu hình ảnh vào thư mục Git: `/homework/session_03/exercise_04/health_check.png` và `/homework/session_03/exercise_04/stats.png`.

---

### BÀI TẬP 5 (Mức độ Giỏi): Triển khai song song dịch vụ khác Runtime Java để chứng minh tính cô lập

#### 1. Mục tiêu mong muốn đạt được
* **Hiểu sâu sắc** về tính cô lập môi trường chạy (Runtime Isolation) của Docker Container.
* **Tự cấu hình** chạy song song hai dịch vụ Java sử dụng hai phiên bản JRE khác nhau (Java 17 vs Java 21) trên cùng một máy chủ mà không gây xung đột.

#### 2. Mô tả yêu cầu
*Hệ thống QuickBite vừa phát triển thêm dịch vụ `restaurant-service` (Dịch vụ quản lý nhà hàng) sử dụng Java 21. Bạn cần khởi chạy nó song song với `user-service` Java 17 đã chạy ở Bài tập 3.*
1. Biên dịch mã nguồn của dịch vụ `restaurant-service` thành file thực thi `restaurant-service.jar` ở máy local.
2. Khởi chạy một container backend mới đặt tên là `quickbite-restaurant` ở chế độ chạy ngầm.
3. Sử dụng base image JRE 21 gọn nhẹ (`eclipse-temurin:21-jre-alpine`) để cung cấp runtime Java 21 độc lập cho container này.
4. Mount file JAR của `restaurant-service.jar` từ máy host vào thư mục `/app` của container.
5. Ánh xạ cổng port hoạt động ra máy host là `8082:8082` (tránh xung đột với cổng 8081 của `user-service`).
6. Tìm IP của container `quickbite-db` và truyền vào các biến môi trường kết nối tương tự như Bài tập 3.
7. Khởi chạy ứng dụng bên trong container bằng lệnh: `java -jar /app/restaurant-service.jar`.

#### 3. Kiểm thử và kết quả mong muốn
1. Xem danh sách các container đang chạy:
   ```bash
   docker ps
   ```
   **Kết quả mong đợi:** Có 3 container hoạt động: `quickbite-db` (port 5432), `quickbite-user` (port 8081) và `quickbite-restaurant` (port 8082).
2. Kiểm tra phiên bản Java thực tế chạy bên trong mỗi container để xác minh tính cô lập:
   ```bash
   docker exec quickbite-user java -version
   # Kết quả mong đợi: hiển thị phiên bản OpenJDK 17
   
   docker exec quickbite-restaurant java -version
   # Kết quả mong đợi: hiển thị phiên bản OpenJDK 21
   ```

#### 4. Hướng dẫn nộp bài
* Lưu lệnh chạy container `quickbite-restaurant` vào file: `/homework/session_03/exercise_05/deploy_restaurant.sh`.
* Chụp màn hình terminal chạy 2 lệnh `java -version` ở trên hiển thị rõ 2 phiên bản Java khác biệt. Lưu hình ảnh tại: `/homework/session_03/exercise_05/java_isolation.png`.

---

### BÀI TẬP 6 (Mức độ Xuất sắc): Kịch bản Shell Script vận hành tự động và Smoke Test sức khỏe cụm dịch vụ (`run-all-staging.sh`)

#### 1. Mục tiêu mong muốn đạt được
* **Tự động hóa hoàn toàn** quy trình dọn dẹp, kiểm tra xung đột phần cứng, khởi chạy cụm dịch vụ và chạy thử kiểm tra liên kết (Smoke Test).
* **Ứng dụng tư duy lập trình Shell Script** để chẩn đoán hệ thống, tự động phân tích lấy IP động của container và xử lý lỗi tự động.

#### 2. Mô tả yêu cầu
Bạn hãy đóng vai trò kỹ sư DevOps viết một script đặt tên là `run-all-staging.sh` để tự động hóa toàn bộ quy trình vận hành thủ công ở các bài tập trên:
1. **Dọn dẹp hệ thống cũ:** Tự động kiểm tra và xóa bỏ các container `quickbite-db`, `quickbite-user`, `quickbite-restaurant` nếu chúng đang tồn tại trên máy để giải phóng tài nguyên.
2. **Kiểm tra xung đột cổng trên Host:** Trước khi khởi chạy, script phải tự động kiểm tra xem các cổng mạng vật lý `5432`, `8081`, `8082` trên máy host có đang bị tiến trình khác chiếm dụng hay không. Nếu có, in thông báo lỗi màu đỏ và dừng script ngay lập tức.
3. **Khởi chạy Database:** Khởi chạy container `quickbite-db` bằng Docker CLI ở cổng `5432`.
4. **Đợi Database sẵn sàng:** Viết vòng lặp (loop) chờ đợi và chạy kiểm tra trạng thái database qua lệnh `pg_isready` mỗi 2 giây, tối đa 5 lần thử (timeout 10 giây). Nếu quá 10 giây database vẫn không sẵn sàng, in lỗi và thoát.
5. **Trích xuất IP động của Database:** Dùng lệnh `docker inspect` kết hợp lọc chuỗi để lấy chính xác IP nội bộ của container `quickbite-db` vừa chạy và gán vào một biến shell (ví dụ: `DB_IP`).
6. **Khởi chạy hai backend service:** Khởi chạy container `quickbite-user` (Java 17, cổng 8081) và `quickbite-restaurant` (Java 21, cổng 8082), truyền biến `DB_IP` vào các tham số môi trường kết nối.
7. **Smoke Test tích hợp:** Chờ 5 giây cho các backend khởi động xong, sau đó dùng `curl` gọi thử API Actuator Health của cả `user-service` (port 8081) và `restaurant-service` (port 8082).
   * Nếu cả hai API đều trả về trạng thái `UP`, in ra thông báo lớn có màu xanh lá: `"CỤM DỊCH VỤ STAGING HOẠT ĐỘNG ỔN ĐỊNH!"` và thoát với code `0`.
   * Nếu một trong hai dịch vụ bị lỗi hoặc không thể kết nối, in thông báo lỗi màu đỏ, in ra 30 dòng log cuối của service bị lỗi để hỗ trợ debug, chạy lệnh xóa các container vừa tạo để dọn dẹp hệ thống và thoát với code `1`.

#### 3. Kiểm thử và kết quả mong muốn
1. Chạy thử kịch bản tự động hóa trên server Staging:
   ```bash
   chmod +x run-all-staging.sh
   ./run-all-staging.sh
   ```
   **Kết quả mong đợi:** Toàn bộ tiến trình diễn ra tự động từ xóa container cũ, kiểm tra port, dựng database, tự lấy IP truyền vào các backend, và kết thúc bằng dòng thông báo xanh lá thành công.
2. Thử giả lập lỗi: Cố tình chạy một ứng dụng chiếm cổng `8081` trên máy host, hoặc truyền sai mật khẩu db kết nối trong script để xem script có tự động phát hiện lỗi, in log debug và dừng dọn dẹp hay không.

#### 4. Hướng dẫn nộp bài
* Đẩy file kịch bản tự động hóa của bạn lên Git tại đường dẫn: `/homework/session_03/exercise_06/run-all-staging.sh`.
* Chụp ảnh màn hình chạy script thành công in ra thông báo màu xanh lá. Lưu hình ảnh vào: `/homework/session_03/exercise_06/staging_success.png`.
