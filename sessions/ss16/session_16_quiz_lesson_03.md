# QUIZ LESSON 03: CẤU HÌNH LOGBACK TRONG SPRING BOOT

## Q1

Trong ứng dụng Spring Boot, tại sao tệp cấu hình logging nên được đặt tên là `logback-spring.xml` thay vì `logback.xml` tiêu chuẩn?

[A]
Vì tiền tố `-spring` cho phép Logback tích hợp sâu với Spring Environment, hỗ trợ đọc biến từ `application.yml` và phân chia cấu hình theo Spring Profiles.
[EXP]
Đúng. Đặt tên `logback-spring.xml` giúp Spring Boot toàn quyền kiểm soát quá trình khởi tạo Logback và tận dụng các thẻ mở rộng như `<springProfile>`.
[B]
Vì nếu đặt tên là `logback.xml` thì ứng dụng Spring Boot sẽ tự động vô hiệu hóa toàn bộ cơ chế logging.
[EXP]
Sai. Tên `logback.xml` vẫn hoạt động nhưng được nạp trước khi Spring Environment sẵn sàng nên không đọc được biến từ `application.yml`.
[C]
Vì tên `logback-spring.xml` là bắt buộc để Docker Engine có thể nhận diện và kích hoạt driver `json-file`.
[EXP]
Sai. Docker Engine không can thiệp vào cách đặt tên file cấu hình nội bộ của Java application.
[D]
Vì định dạng XML của `logback.xml` không tương thích với bộ giải mã JSON của thư viện ECS.
[EXP]
Sai. Cả hai tên tệp đều sử dụng cú pháp XML chuẩn của Logback.

@correct: A
@point: 20

## Q2

Thẻ khai báo `<property name="service.name" value="order-service"/>` trong tệp `logback-spring.xml` đóng vai trò kỹ thuật gì trong hệ thống microservices QuickBite?

[A]
Chỉ định tên tệp tin vật lý mà Logback sẽ tạo ra trên ổ đĩa cứng của máy chủ VPS.
[EXP]
Sai. Thẻ này định nghĩa một biến chuỗi, không trực tiếp tạo ra tệp tin vật lý.
[B]
Dùng để đăng ký tên dịch vụ với máy chủ Eureka Server trong hệ thống.
[EXP]
Sai. Đăng ký Eureka sử dụng thuộc tính `spring.application.name`, không liên quan đến thẻ property của Logback.
[C]
Định nghĩa tên định danh dịch vụ để ECS Encoder chèn trường `service.name` vào mỗi dòng log, giúp phân biệt nguồn gốc log giữa các container.
[EXP]
Đúng. Giá trị này được encoder sử dụng để gắn nhãn nguồn gốc phát sinh log của từng microservice khi thu thập tập trung.
[D]
Thiết lập quyền truy cập cho người dùng quản trị khi đăng nhập vào giao diện web của service.
[EXP]
Sai. Thuộc tính này hoàn toàn không có chức năng phân quyền bảo mật.

@correct: C
@point: 20

## Q3

Khi quan sát cấu hình trong tệp `application.yml` dưới đây, các lời gọi `log.debug()` trong package `com.quickbite.order_service` và trong framework `org.hibernate` sẽ được xuất ra như thế nào?

```yaml
logging:
  level:
    root: INFO
    com.quickbite.order_service: DEBUG
```

[A]
Cả `com.quickbite.order_service` và `org.hibernate` đều xuất được các dòng log ở mức `DEBUG`.
[EXP]
Sai. `org.hibernate` chịu sự quản lý của root logger (mức INFO) nên log DEBUG của nó sẽ bị chặn.
[B]
`com.quickbite.order_service` xuất được log `DEBUG`, trong khi `org.hibernate` chỉ xuất các log từ mức `INFO` trở lên.
[EXP]
Đúng. Package logger ghi đè mức chi tiết lên DEBUG cho riêng mã nguồn nghiệp vụ, trong khi các thư viện khác giữ nguyên mức INFO của root.
[C]
Tất cả các dòng log `DEBUG` đều bị chặn do root logger đang khóa ở mức `INFO`.
[EXP]
Sai. Package logger có độ ưu tiên cụ thể hơn và ghi đè lên thiết lập chung của root logger.
[D]
Ứng dụng sẽ báo lỗi xung đột cấu hình và không thể khởi động được.
[EXP]
Sai. Đây là cú pháp cấu hình phân tầng chuẩn mực và rất phổ biến trong Spring Boot.

@correct: B
@point: 20

## Q4

Giải pháp nào dưới đây là tối ưu nhất để thay đổi log level của một microservice đang chạy bằng Docker Compose trên Production sang `DEBUG` mà KHÔNG cần sửa mã nguồn Java và KHÔNG cần build lại Docker image?

[A]
Truy cập trực tiếp vào container bằng `docker exec` và chỉnh sửa file `logback-spring.xml` trong thư mục JAR.
[EXP]
Sai. Không thể sửa trực tiếp file resource bên trong tệp JAR đã đóng gói khi container đang chạy.
[B]
Xóa container hiện tại và chạy trực tiếp lệnh `./gradlew bootRun --debug` trên terminal của máy chủ VPS.
[EXP]
Sai. Trên Production các service bắt buộc phải chạy trong môi trường container hóa để đảm bảo tính nhất quán.
[C]
Thêm tham số `-Dlogging.level.root=DEBUG` trực tiếp vào mã nguồn Java trong file `OrderServiceApplication.java`.
[EXP]
Sai. Cách làm này đòi hỏi phải sửa code, commit Git và build lại toàn bộ Docker image.
[D]
Cấu hình biến môi trường trong file `docker-compose.yml` (ví dụ `LOG_LEVEL_ROOT: DEBUG`) tương ứng với tham chiếu trong `application.yml` và chạy `docker compose up -d`.
[EXP]
Đúng. Sử dụng biến môi trường giúp ghi đè cấu hình tức thì và tái tạo container chỉ trong vài giây mà không cần build lại mã nguồn.

@correct: D
@point: 20

## Q5

Nếu một kỹ sư cấu hình nhầm Root Logger trong `logback-spring.xml` ở mức `<root level="WARN">`, dòng log nào dưới đây trong mã nguồn sẽ BỊ ẨN và không xuất hiện trên console của container?

[A]
`log.error("Database connection lost to host db-order:3306", exception);`
[EXP]
Log ERROR có mức nghiêm trọng cao hơn WARN nên vẫn được xuất ra bình thường.
[B]
`log.warn("Menu item is currently out of stock: itemId={}", itemId);`
[EXP]
Log WARN có mức độ ngang bằng với root level nên vẫn được hiển thị bình thường.
[C]
`log.info("Order created successfully: orderId={}, totalAmount={}", orderId, amount);`
[EXP]
Đúng. Log INFO có mức nghiêm trọng thấp hơn WARN nên sẽ bị bộ lọc của Root Logger loại bỏ hoàn toàn.
[D]
`log.error("Payment transaction timeout for orderId={}", orderId);`
[EXP]
Log ERROR có mức nghiêm trọng cao nhất nên luôn luôn được ghi nhận.

@correct: C
@point: 20
