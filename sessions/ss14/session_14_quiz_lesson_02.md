# QUIZ LESSON 02: KẾT NỐI GRAFANA VỚI PROMETHEUS

## Q1

Khi thêm nguồn dữ liệu Prometheus (Data Source) trong giao diện cấu hình của Grafana, trường thông tin nào bắt buộc phải thiết lập chính xác?

[A]
Địa chỉ thư mục đĩa cứng chứa volume `prometheus-data` của host VPS.
[EXP]
Grafana không truy cập trực tiếp vào volume đĩa cứng của Prometheus, nó giao tiếp qua cổng HTTP API.
[B]
Đường dẫn HTTP URL trỏ tới API endpoint của máy chủ Prometheus Server.
[EXP]
Đúng. Grafana cần URL của Prometheus Server (ví dụ: http://quickbite-prometheus:9090) để gửi yêu cầu truy vấn dữ liệu.
[C]
Đường dẫn dẫn đến file cấu hình `prometheus.yml` trên hệ thống.
[EXP]
Grafana không đọc trực tiếp file cấu hình của Prometheus Server.
[D]
Mật khẩu tài khoản admin mặc định của máy chủ cơ sở dữ liệu MySQL.
[EXP]
MySQL không liên quan đến việc kết nối giữa Grafana và nguồn dữ liệu Prometheus.

@correct: B
@point: 20

## Q2

Khi hai container Grafana và Prometheus chạy chung mạng nội bộ Docker (quickbite-net), cấu hình URL kết nối nào dưới đây là ĐÚNG và an toàn nhất?

[A]
`http://localhost:9090`
[EXP]
Sai. Cấu hình này trỏ về chính container Grafana, dẫn đến lỗi Connection Refused do Prometheus không chạy chung container với Grafana.
[B]
`http://quickbite-prometheus:9090`
[EXP]
Đúng. Sử dụng tên dịch vụ (service name) khai báo trong file docker-compose giúp Docker DNS phân giải chính xác địa chỉ IP nội bộ trong mạng ảo.
[C]
`http://127.0.0.1:3000`
[EXP]
Sai. 3000 là cổng chạy của Grafana, không phải của Prometheus Server.
[D]
`http://public_ip_vps:9090`
[EXP]
Sai về mặt bảo mật. Sử dụng IP public của VPS yêu cầu phải mở cổng 9090 ra Internet, tạo nguy cơ mất an toàn thông tin rất cao.

@correct: B
@point: 20

## Q3

Tại sao việc cấu hình URL kết nối tới Prometheus bằng `http://127.0.0.1:9090` bên trong giao diện Grafana lại báo lỗi Connection Refused?

[A]
Vì Prometheus Server chỉ chấp nhận kết nối qua cổng bảo mật HTTPS.
[EXP]
Prometheus Server mặc định sử dụng giao thức HTTP thông thường.
[B]
Vì địa chỉ `127.0.0.1` được phân giải cục bộ là chính bản thân container Grafana, nơi không có tiến trình Prometheus nào đang chạy.
[EXP]
Đúng. Mỗi container có một loopback interface riêng biệt, do đó 127.0.0.1 trỏ về chính nó chứ không trỏ về máy host hay container khác.
[C]
Vì Prometheus Server đã chủ động chặn toàn bộ các kết nối đi từ IP localhost.
[EXP]
Prometheus không chặn IP localhost, lỗi xảy ra do cơ chế định tuyến mạng ảo của container.
[D]
Vì Grafana không được cấp quyền đọc tệp tin cơ sở dữ liệu TSDB của Prometheus.
[EXP]
Kết nối qua HTTP API không phụ thuộc vào quyền đọc file đĩa cứng của Grafana.

@correct: B
@point: 20

## Q4

Nút kiểm tra Save & Test trong phần cấu hình Data Source của Grafana thực hiện hành vi kiểm chứng nào?

[A]
Thực hiện ghi thử một tệp dữ liệu kiểm tra vào volume lưu trữ của Prometheus.
[EXP]
Nguồn dữ liệu Prometheus là Read-Only đối với Grafana, Grafana không có quyền ghi dữ liệu vào TSDB.
[B]
Gửi một request truy cập API kiểm tra kết nối tới Prometheus URL để xác nhận Prometheus phản hồi thành công.
[EXP]
Đúng. Save & Test gửi truy vấn thử tới Prometheus để kiểm tra xem URL có thông mạng và trả về dữ liệu hay không.
[C]
Tự động quét toàn bộ microservices trong cụm Docker Compose để tìm cổng metrics.
[EXP]
Hành vi này là Service Discovery của Prometheus Server, không phải của Grafana.
[D]
Yêu cầu hệ điều hành host VPS khởi động lại container Prometheus để cập nhật IP mới.
[EXP]
Grafana không thể ra lệnh khởi động lại container Docker từ xa qua nút bấm này.

@correct: B
@point: 20

## Q5

Khi cấu hình kết nối giữa Grafana và Prometheus thành công, thông báo màu xanh nào sẽ hiển thị ở cuối trang cấu hình?

[A]
`Connected successfully to Database`
[EXP]
Đây không phải là nội dung thông báo chuẩn của Grafana dành cho nguồn dữ liệu Prometheus.
[B]
`Successfully queried the Prometheus API.`
[EXP]
Đúng. Grafana sẽ hiển thị thông báo này để xác nhận việc kết nối và truy vấn thử API của Prometheus đã hoàn tất tốt đẹp.
[C]
`Prometheus container is up and running`
[EXP]
Grafana chỉ kiểm tra kết nối API, không trực tiếp truy cập vào Docker Daemon để đọc trạng thái container.
[D]
`All microservices metrics loaded successfully`
[EXP]
Thông báo này không chính xác, Grafana không đếm số lượng metrics microservice khi lưu cấu hình.

@correct: B
@point: 20
