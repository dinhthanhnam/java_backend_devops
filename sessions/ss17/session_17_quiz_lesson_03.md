# QUIZ LESSON 03: GỬI LOG TỪ SPRING BOOT ĐẾN FLUENTD

## Q1

Tùy chọn `fluentd-async: "true"` trong khối cấu hình logging của Docker Compose giải quyết vấn đề kỹ thuật nào khi khởi động container?

[A]
Cho phép container ứng dụng khởi động bình thường (non-blocking) ngay cả khi Fluentd tạm thời chưa sẵn sàng hoặc có độ trễ kết nối ban đầu.
[EXP]
Đúng. Chế độ bất đồng bộ giúp ứng dụng không bị treo khi khởi động; Docker Daemon sẽ tự động đệm log trong bộ nhớ và kết nối lại trong nền.
[B]
Tự động kích hoạt cơ chế nén dữ liệu log theo chuẩn nén bất đồng bộ để tiết kiệm băng thông.
[EXP]
Sai. Tùy chọn này không có chức năng nén dữ liệu.
[C]
Yêu cầu Fluentd chỉ xử lý các log có mức độ nghiêm trọng `ERROR` và bỏ qua các mức khác.
[EXP]
Sai. Tham số này không can thiệp vào mức độ log level của ứng dụng.
[D]
Chuyển hướng toàn bộ các log xuất ra từ `stderr` sang một máy chủ dự phòng khác.
[EXP]
Sai. Cả `stdout` và `stderr` đều được chuyển tiếp tới địa chỉ Fluentd đã cấu hình.

@correct: A
@point: 20

## Q2

Trong khối cấu hình `<filter quickbite.**>` của Fluentd, tại sao tham số `reserve_data true` lại mang tính bắt buộc khi giải mã trường `log`?

[A]
Để yêu cầu Fluentd tự động lưu trữ một bản sao dự phòng của log vào thư mục `/tmp` của máy chủ.
[EXP]
Sai. Tham số này không tạo file dự phòng trong thư mục `/tmp`.
[B]
Để ngăn chặn việc các trường dữ liệu có giá trị null bị đẩy vào Elasticsearch.
[EXP]
Sai. Tham số này không dùng để lọc bỏ giá trị null.
[C]
Để giữ lại toàn bộ các metadata do Docker cung cấp (`container_name`, `container_id`, `source`) khi bóc tách chuỗi JSON sang các trường dữ liệu mới.
[EXP]
Đúng. Nếu thiếu `reserve_data true`, parser JSON sẽ xóa sạch metadata của container và chỉ giữ lại các trường được giải mã từ Spring Boot.
[D]
Để khóa quyền ghi của tất cả các container khác vào tệp đệm buffer của Fluentd.
[EXP]
Sai. Tham số này là thuộc tính của Parser Filter, không liên quan đến phân quyền file buffer.

@correct: C
@point: 20

## Q3

Khi cập nhật khối cấu hình `logging` trong tệp `docker-compose.yml` của các microservices, tại sao lệnh `docker compose restart` không có tác dụng mà bắt buộc phải chạy `docker compose up -d --force-recreate`?

[A]
Vì lệnh `restart` chỉ có tác dụng đối với các container sử dụng image mặc định từ Docker Hub.
[EXP]
Sai. Lệnh `restart` áp dụng cho mọi loại container nhưng không thay đổi cấu hình khởi tạo của Docker daemon.
[B]
Vì Logging Driver là cấu hình thuộc tính tĩnh được gắn cố định khi tạo container; lệnh `restart` không thay đổi cấu hình hạ tầng này mà cần phải tạo mới container.
[EXP]
Đúng. Docker chỉ nạp Logging Driver khi container được tạo mới (`create`), do đó bắt buộc phải dùng `--force-recreate` để áp dụng driver mới.
[C]
Vì Docker Compose tự động khóa file cấu hình `docker-compose.yml` ở chế độ chỉ đọc (Read-Only) khi container đang chạy.
[EXP]
Sai. Docker Compose không khóa tệp tin cấu hình.
[D]
Vì lệnh `restart` sẽ tự động chuyển logging driver về trạng thái mặc định `none`.
[EXP]
Sai. Lệnh `restart` giữ nguyên toàn bộ cấu hình container hiện có.

@correct: B
@point: 20

## Q4

Nhãn cấu hình `tag: "quickbite.order"` trong tệp `docker-compose.yml` đóng vai trò gì khi dữ liệu log được chuyển tiếp tới Fluentd Collector?

[A]
Dùng làm tên đăng nhập tài khoản quản trị khi Fluentd kết nối sang Elasticsearch.
[EXP]
Sai. Tài khoản xác thực được khai báo riêng trong cấu hình Elasticsearch plugin, không dùng tag.
[B]
Quy định thời gian sống tối đa (TTL) của dòng log trong kho lưu trữ dữ liệu.
[EXP]
Sai. Thời gian lưu trữ dữ liệu do chính sách quản lý chỉ mục (ILM) của Elasticsearch quyết định.
[C]
Dùng để ghi đè tên của Java package phát sinh sự kiện log trong Spring Boot.
[EXP]
Sai. Tên Java package được quản lý trong trường `log.logger` của chuẩn ECS.
[D]
Đóng vai trò là định danh định tuyến giúp Fluentd nhận diện và phân luồng dữ liệu theo mẫu khớp `<filter quickbite.**>` và `<match quickbite.**>`.
[EXP]
Đúng. Nhãn tag giúp Fluentd xác định chính xác quy tắc lọc (parser) và đích đến (Elasticsearch output) dành riêng cho hệ thống QuickBite.

@correct: D
@point: 20

## Q5

Docker Logging Driver `fluentd` đóng gói nội dung log xuất ra từ `stdout` của ứng dụng Spring Boot vào trường nào trong cấu trúc gói tin chuyển tiếp sang Fluentd?

[A]
Đóng gói toàn bộ chuỗi JSON gốc của Spring Boot vào trường `log` dưới dạng String.
[EXP]
Đúng. Docker driver bọc dòng text console của container vào trường `log` và gửi kèm các metadata container sang Fluentd.
[B]
Tự động phân tách từng trường của Spring Boot thành các tham số HTTP Header riêng biệt.
[EXP]
Sai. Docker driver giao tiếp qua giao thức forward socket, không chuyển đổi thành HTTP Header.
[C]
Chỉ gửi thông tin `container_id` và loại bỏ hoàn toàn nội dung thông điệp nghiệp vụ.
[EXP]
Sai. Nội dung log xuất ra console luôn được bảo toàn trọn vẹn trong trường `log`.
[D]
Mã hóa toàn bộ nội dung thành chuỗi Base64 và lưu vào trường `payload`.
[EXP]
Sai. Docker logging driver không mã hóa Base64 cho trường nội dung log.

@correct: A
@point: 20
