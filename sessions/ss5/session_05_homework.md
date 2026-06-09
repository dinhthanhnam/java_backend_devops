# SESSION 05: BÀI TẬP VỀ NHÀ (HOMEWORK)
## Đề tài: Triển khai Hệ thống Đa dịch vụ & API Gateway

Tài liệu này chứa 6 bài tập thực hành dành cho Session 5 của hệ thống QuickBite. Bạn cần hoàn thành đầy đủ các bài tập và nộp bài theo đúng hướng dẫn. Kết quả bài làm được đánh giá theo cơ chế Đạt (Pass) hoặc Không đạt (Fail).

---

### BÀI TẬP 1 (Mức độ Khá): Viết Dockerfile đóng gói đồng thời 4 dịch vụ Spring Boot của QuickBite

#### 1. Mục tiêu mong muốn đạt được
* **Biên soạn thành công cấu trúc Dockerfile** tối ưu hóa cho từng microservice.
* **Tự đóng gói đồng thời** 4 dịch vụ Spring Boot độc lập (`user-service`, `restaurant-service`, `order-service`, `notification-service`) thành các Docker Images.

#### 2. Mô tả yêu cầu
Bạn hãy đóng vai trò là kỹ sư DevOps viết file đóng gói Docker cho cả 4 dịch vụ của QuickBite:
1. Tạo 4 tệp tin đặt tên chính xác là `Dockerfile` (không có phần mở rộng) đặt tại thư mục gốc của từng thư mục dự án tương ứng:
   * `/user-service/Dockerfile`
   * `/restaurant-service/Dockerfile`
   * `/order-service/Dockerfile`
   * `/notification-service/Dockerfile`
2. Các file `Dockerfile` cần tuân thủ đặc tả:
   * Sử dụng base image JRE 17 gọn nhẹ (`eclipse-temurin:17-jre-alpine`).
   * Đặt thư mục làm việc mặc định trong container là `/app`.
   * Sao chép file JAR đã biên dịch của dịch vụ đó vào container dưới tên `app.jar`.
   * Khai báo lệnh khởi chạy `java -jar app.jar` bằng Exec Form.
3. Thực hiện chạy lệnh build image cho cả 4 dịch vụ với các tên tag tương ứng:
   * `quickbite-user-service:latest`
   * `quickbite-restaurant-service:latest`
   * `quickbite-order-service:latest`
   * `quickbite-notification-service:latest`

#### 3. Kiểm thử và kết quả mong muốn
1. Di chuyển vào từng thư mục dịch vụ, build file JAR và đóng gói image:
   ```bash
   # Ví dụ đối với user-service
   cd user-service
   ./gradlew bootJar
   docker build -t quickbite-user-service:latest .
   ```
   *(Thực hiện tương tự cho 3 dịch vụ còn lại).*
2. Gõ lệnh liệt kê danh sách image để kiểm tra:
   ```bash
   docker images | grep quickbite
   ```
3. **Kết quả mong đợi:** Cả 4 Docker Images của 4 dịch vụ xuất hiện đầy đủ trong danh sách với tên và tag chính xác.

#### 4. Hướng dẫn nộp bài
* Đẩy 4 tệp tin `Dockerfile` của 4 dịch vụ lên Git tại các thư mục tương ứng của dự án.
* Chụp màn hình terminal chạy lệnh `docker images | grep quickbite` hiển thị 4 image đã build. Lưu ảnh vào thư mục cá nhân tại: `/homework/session_05/exercise_01/build_images.png`.

---

### BÀI TẬP 2 (Mức độ Khá): Thiết lập Docker Compose khởi chạy toàn bộ 4 dịch vụ và database dùng chung mạng

#### 2. Mô tả yêu cầu
Tạo hạ tầng điều phối tập trung cho toàn bộ hệ thống QuickBite:
1. Tạo file đặt tên chính xác là `docker-compose.yml` đặt tại thư mục gốc của dự án (`/quickbite-infra/docker-compose.yml`).
2. Khai báo 5 dịch vụ bao gồm: `quickbite-db` (Postgres database), `quickbite-user`, `quickbite-restaurant`, `quickbite-order`, và `quickbite-notification`.
3. Cấu hình cho tất cả các container tham gia vào chung một mạng Bridge tùy biến mang tên `quickbite-net`.
4. Cấu hình biến môi trường kết nối database cơ bản cho các service.
5. Để phục vụ kiểm thử, hãy map các cổng của tất cả các dịch vụ backend ra ngoài máy host (ví dụ: `8081:8081`, `8082:8082`, `8083:8083`, `8084:8084`).

#### 3. Kiểm thử và kết quả mong muốn
1. Khởi chạy hệ thống bằng Docker Compose:
   ```bash
   docker compose up -d
   ```
2. Kiểm tra danh sách container hoạt động:
   ```bash
   docker compose ps
   ```
3. **Kết quả mong đợi:** Cả 5 container đều đang chạy ở trạng thái `Up`, các cổng tương ứng được mở ra host máy chủ.

#### 4. Hướng dẫn nộp bài
* Đẩy file `docker-compose.yml` phiên bản Bài 2 lên Git tại thư mục gốc `/docker-compose.yml`.
* Chụp ảnh màn hình chạy lệnh `docker compose ps` hiển thị đầy đủ cụm 5 container đang chạy. Lưu hình ảnh tại: `/homework/session_05/exercise_02/compose_up.png`.

---

### BÀI TẬP 3 (Mức độ Khá): Cấu hình biến môi trường kết nối database riêng biệt cho từng service

#### 1. Mục tiêu mong muốn đạt được
* **Áp dụng nguyên tắc tách biệt dữ liệu** (Database-per-service) trong Docker Compose.
* **Cấu hình chính xác biến môi trường kết nối** để định hướng mỗi dịch vụ vào một database logic độc lập.

#### 2. Mô tả yêu cầu
Để không xảy ra tình trạng các dịch vụ ghi chung dữ liệu vào một database mặc định, hãy cập nhật cấu hình:
1. Sử dụng một container `quickbite-db` duy nhất, nhưng nạp biến môi trường tên database riêng biệt cho từng dịch vụ backend trong file `docker-compose.yml`:
   * `quickbite-user` kết nối tới database logic: `quickbite_user`
   * `quickbite-restaurant` kết nối tới database logic: `quickbite_restaurant`
   * `quickbite-order` kết nối tới database logic: `quickbite_order`
   * `quickbite-notification` kết nối tới database logic: `quickbite_notification`
2. Cấu hình biến `DB_HOST` của tất cả các dịch vụ backend nhận giá trị là tên container `quickbite-db` để phân giải qua mạng Docker Network.
3. Đặt cấu hình `depends_on` cho 4 service backend trỏ tới `quickbite-db`.

#### 3. Kiểm thử và kết quả mong muốn
1. Thực hiện chạy lệnh kiểm tra cấu hình nội suy của compose:
   ```bash
   docker compose config
   ```
2. **Kết quả mong đợi:** Terminal in ra thông số cấu hình hoàn chỉnh của 5 service, đảm bảo các trường `environment` chứa đúng tên database tương ứng của từng dịch vụ và không có lỗi cú pháp.

#### 4. Hướng dẫn nộp bài
* Cập nhật file `docker-compose.yml` (chứa cấu hình biến môi trường DB) lên Git.
* Chụp ảnh màn hình kết quả chạy lệnh `docker compose config`. Lưu tại: `/homework/session_05/exercise_03/env_config.png`.

---

### BÀI TẬP 4 (Mức độ Khá): Tự động khởi tạo database schema logic qua Spring Data JPA và Hibernate DDL

#### 1. Mục tiêu mong muốn đạt được
* **Ứng dụng thành công cơ chế khởi tạo tự động** `/docker-entrypoint-initdb.d/` của Postgres container.
* **Kích hoạt cơ chế Hibernate DDL-Auto** để tự động tạo cấu trúc bảng cho 4 database logic khi ứng dụng khởi chạy.

#### 2. Mô tả yêu cầu
Tránh lỗi crash kết nối do database chưa được tạo trước:
1. Tạo thư mục `init-scripts` nằm cạnh file `docker-compose.yml`. Tạo tệp `init-db.sql` bên trong chứa các lệnh SQL để tạo 4 database logic: `quickbite_user`, `quickbite_restaurant`, `quickbite_order`, và `quickbite_notification`.
2. Thực hiện mount tệp `init-db.sql` vật lý trên máy host vào thư mục `/docker-entrypoint-initdb.d/init-db.sql` bên trong container `quickbite-db`.
3. Đảm bảo cấu hình biến `ddl-auto` của cả 4 ứng dụng Spring Boot được thiết lập là `update` (thông qua biến môi trường hoặc file application.yml).
4. Khởi chạy lại hệ thống từ đầu (yêu cầu chạy `docker compose down -v` để xóa sạch volumes cũ trước khi khởi động lại).

#### 3. Kiểm thử và kết quả mong muốn
1. Chạy lệnh tắt và dọn dẹp cũ: `docker compose down -v`.
2. Khởi chạy lại hệ thống: `docker compose up -d --build`.
3. Sử dụng công cụ `psql` bên trong container database để kiểm tra xem các bảng của `user-service` đã tự động được Hibernate sinh ra trong database `quickbite_user` hay chưa:
   ```bash
   docker compose exec quickbite-db psql -U postgres -d quickbite_user -c "\dt"
   ```
4. **Kết quả mong đợi:** Terminal in ra danh sách các bảng gồm `users`, `user_addresses`, và `user_wallets` đã được tạo thành công trong database `quickbite_user`.

#### 4. Hướng dẫn nộp bài
* Đẩy file `init-scripts/init-db.sql` và file `docker-compose.yml` cập nhật lên Git.
* Chụp màn hình terminal chạy lệnh hiển thị danh sách bảng của database `quickbite_user` ở Phần 3. Lưu tại: `/homework/session_05/exercise_04/table_generation.png`.

---

### BÀI TẬP 5 (Mức độ Giỏi): Cấu hình định tuyến và tích hợp Spring Cloud Gateway vào mạng cụm container

#### 1. Mục tiêu mong muốn đạt được
* **Xây dựng cấu hình Route** định tuyến động cho Spring Cloud Gateway bằng file YAML.
* **Thực hiện đóng kín hệ thống** (đóng các cổng của microservices nội bộ và mở duy nhất cổng gateway) để bảo vệ hạ tầng.

#### 2. Mô tả yêu cầu
Nâng cấp kiến trúc hệ thống lên STATE 3 bằng cách tích hợp API Gateway làm cửa ngõ duy nhất:
1. Viết file cấu hình `application.yml` cho dự án `gateway-service` (cổng chạy 8080) định nghĩa 4 routes định tuyến request dựa vào các path pattern:
   * `/api/v1/users/**` ──► `http://quickbite-user:8081`
   * `/api/v1/restaurants/**` ──► `http://quickbite-restaurant:8082`
   * `/api/v1/orders/**` ──► `http://quickbite-order:8083`
   * `/api/v1/notifications/**` ──► `http://quickbite-notification:8084`
2. Cập nhật `docker-compose.yml`, khai báo dịch vụ `quickbite-gateway` và map cổng `"8080:8080"` ra ngoài máy host.
3. **Thay đổi bảo mật quan trọng:** Hãy xóa bỏ cấu hình `ports` (ví dụ: `8081:8081`) của 4 dịch vụ backend cũ, thay thế bằng từ khóa `expose` để chỉ cho phép các cổng này hoạt động nội bộ trong mạng `quickbite-net`.

#### 3. Kiểm thử và kết quả mong muốn
1. Khởi chạy lại toàn bộ cụm: `docker compose up -d --build`.
2. Kiểm thử gửi API request qua cổng Gateway 8080 bằng Postman hoặc Curl:
   ```bash
   curl -i http://localhost:8080/api/v1/users/actuator/health
   ```
3. **Kết quả mong đợi:** Request qua cổng 8080 trả về status `200 OK` và thông tin sức khỏe của `user-service`.
4. Thử gọi trực tiếp tới cổng 8081:
   ```bash
   curl -i http://localhost:8081/api/v1/users/actuator/health
   ```
5. **Kết quả mong đợi:** Kết nối thất bại (Connection refused hoặc Timeout) vì cổng 8081 đã được đóng kín khỏi máy host vật lý.

#### 4. Hướng dẫn nộp bài
* Đẩy file cấu hình định tuyến `/gateway-service/src/main/resources/application.yml` và file `docker-compose.yml` hoàn chỉnh lên Git.
* Chụp ảnh màn hình kết quả chạy hai lệnh curl (qua cổng 8080 thành công và cổng 8081 thất bại). Lưu ảnh tại: `/homework/session_05/exercise_05/gateway_routing.png`.

---

### BÀI TẬP 6 (Mức độ Xuất sắc): Viết kịch bản Shell Script vận hành tự động và kiểm thử luồng tích hợp Đơn hàng qua Gateway (`quickbite-integration-test.sh`)

#### 1. Mục tiêu mong muốn đạt được
* **Tự động hóa hoàn toàn** quy trình build, start cụm microservices, kiểm tra sức khỏe và chạy thử tích hợp.
* **Sử dụng cú pháp lập trình Bash Script** để tự động kiểm thử API (Smoke Test) và tự động dọn dẹp hệ thống.

#### 2. Mô tả yêu cầu
Bạn hãy đóng vai trò kỹ sư DevOps viết script tự động hóa `quickbite-integration-test.sh` thực hiện chuỗi hành động sau:
1. Script chấp nhận hai tham số đầu vào: `./quickbite-integration-test.sh [run | clean]`.
2. **Khi chạy với tham số `run`:**
   * Tự động khởi chạy cụm dịch vụ: `docker compose up -d --build`.
   * Viết vòng lặp (loop) kiểm tra tình trạng database PostgreSQL: `docker compose exec quickbite-db pg_isready -U postgres` liên tục mỗi 2 giây, tối đa 5 lần. Nếu quá thời gian (timeout 10 giây) mà database chưa chạy, in thông báo lỗi màu đỏ, dừng cụm và thoát với code `1`.
   * Chờ thêm 5 giây để các dịch vụ Spring Boot hoàn tất khởi động.
   * Gửi một request GET qua Gateway để kiểm tra sức khỏe của `user-service`: `http://localhost:8080/api/v1/users/actuator/health`.
   * Nếu Gateway trả về status `200 OK`, in ra màn hình thông báo lớn màu xanh lá: `"CỔNG GATEWAY HOẠT ĐỘNG ỔN ĐỊNH. HỆ THỐNG LIÊN THÔNG THÀNH CÔNG!"` và thoát với code `0`.
   * Nếu request sập hoặc trả về mã lỗi khác, in thông báo lỗi màu đỏ kèm theo 30 dòng log cuối của cụm container, tự động chạy `docker compose down -v` dọn dẹp và thoát với code `1`.
3. **Khi chạy với tham số `clean`:**
   * Tự động dừng cụm container và xóa bỏ hoàn toàn dữ liệu volumes: `docker compose down -v`.

#### 3. Kiểm thử và kết quả mong muốn
1. Cấp quyền thực thi và chạy kịch bản khởi động:
   ```bash
   chmod +x quickbite-integration-test.sh
   ./quickbite-integration-test.sh run
   ```
2. **Kết quả mong đợi:** Hệ thống dựng lên, script in ra các dòng kiểm tra trạng thái database, thực hiện gọi curl kiểm định thành công và kết thúc bằng dòng chữ xanh lá thông báo liên thông thành công.
3. Thử sửa sai tên service database của `quickbite-user` trong file compose để kích hoạt lỗi kết nối và chạy lại script:
   * **Kết quả mong đợi:** Script phát hiện ra lỗi (do healthcheck của user-service không trả về 200), in logs debug màu đỏ, tự động tắt cụm và dọn dẹp sạch sẽ tài nguyên.

#### 4. Hướng dẫn nộp bài
* Đẩy file kịch bản điều khiển `quickbite-integration-test.sh` lên Git tại thư mục bài tập: `/homework/session_05/exercise_06/quickbite-integration-test.sh`.
* Chụp ảnh màn hình console chạy script thành công và hiển thị dòng chữ màu xanh lá. Lưu hình ảnh tại: `/homework/session_05/exercise_06/integration_success.png`.
