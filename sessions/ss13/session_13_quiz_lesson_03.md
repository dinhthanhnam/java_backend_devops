# QUIZ LESSON 03: TRIỂN KHAI PROMETHEUS BẰNG DOCKER COMPOSE THU THẬP CHỈ SỐ BACKEND

## Q1

Trong tệp tin cấu hình `prometheus.yml`, chỉ thị `scrape_interval: 15s` định nghĩa hành vi nào của Prometheus Server?

[A]
Định kỳ lưu trữ dữ liệu từ RAM xuống đĩa cứng mỗi 15 giây một lần.
[EXP]
Chỉ thị này không cấu hình cơ chế flush dữ liệu TSDB xuống đĩa cứng.
[B]
Định kỳ 15 giây gửi yêu cầu HTTP GET một lần đến các target để kéo dữ liệu metrics về.
[EXP]
Đúng. `scrape_interval` xác định chu kỳ quét, cứ sau 15 giây Prometheus Server sẽ chủ động đi lấy metrics từ các target một lần.
[C]
Tự động xóa lịch sử các metrics cũ sau mỗi 15 giây.
[EXP]
Dữ liệu metrics được lưu giữ lâu dài (mặc định 15 ngày), không bị xóa sau 15 giây.
[D]
Ngắt kết nối với các microservice nếu không nhận được phản hồi trong vòng 15 giây.
[EXP]
Đây là thời gian timeout của request, được quy định bởi tham số `scrape_timeout` chứ không phải `scrape_interval`.

@correct: B
@point: 20

## Q2

Tại sao trong cấu hình `targets` của `prometheus.yml`, chúng ta ghi tên container (ví dụ `order-service:8083`) thay vì ghi địa chỉ IP thô của container đó?

[A]
Vì Prometheus không chấp nhận định dạng địa chỉ IP thô dạng số.
[EXP]
Prometheus hoàn toàn chấp nhận IP thô dạng số.
[B]
Vì địa chỉ IP nội bộ của container Docker có thể thay đổi sau mỗi lần container khởi động lại, còn tên container được DNS nội bộ của Docker phân giải ổn định.
[EXP]
Đúng. Docker Engine tự động cập nhật bản ghi DNS cho tên container, giúp Prometheus luôn kết nối chính xác tới container backend bất kể IP của container bị thay đổi.
[C]
Để bắt buộc luồng traffic đi vũ lực ra ngoài Internet trước khi nạp vào Prometheus.
[EXP]
Kết nối thông qua tên container DNS nội bộ được duy trì hoàn toàn cục bộ bên trong mạng Docker Bridge, không đi ra ngoài Internet.
[D]
Để tăng tốc độ nạp dữ liệu metrics của Prometheus lên gấp đôi.
[EXP]
Tên DNS chỉ giúp định tuyến kết nối động ổn định, không trực tiếp tăng tốc độ nạp hay xử lý dữ liệu.

@correct: B
@point: 20

## Q3

Khi cấu hình container Prometheus chạy bằng Docker Compose, tại sao việc thiết lập volume mount `prometheus-data:/prometheus` lại cực kỳ quan trọng trên môi trường Production?

[A]
Để đồng bộ cấu hình tệp tin `prometheus.yml` từ máy local lên máy chủ Cloud.
[EXP]
Việc mount file cấu hình được thực hiện ở một dòng mount khác, volume này dùng để lưu trữ cơ sở dữ liệu.
[B]
Để bảo toàn dữ liệu lịch sử metrics chuỗi thời gian (TSDB) trên VPS host, tránh bị mất sạch thông tin giám sát khi container bị dừng hoặc nâng cấp.
[EXP]
Đúng. Toàn bộ dữ liệu TSDB được lưu trong thư mục `/prometheus` của container. Việc mount volume ra ngoài giúp dữ liệu tồn tại độc lập với vòng đời của container.
[C]
Để giảm bớt dung lượng bộ nhớ đệm RAM mà Prometheus tiêu thụ khi chạy lệnh truy vấn.
[EXP]
Volume mount lưu trữ dữ liệu lên đĩa cứng, không ảnh hưởng trực tiếp tới cách Prometheus quản lý bộ nhớ đệm RAM.
[D]
Để kích hoạt tính năng tự động phát hiện lỗi và gửi cảnh báo tin nhắn qua điện thoại.
[EXP]
Tính năng cảnh báo được điều khiển bởi cấu hình Alertmanager, không liên quan tới việc mount volume lưu trữ dữ liệu.

@correct: B
@point: 20

## Q4

Tại sao các kỹ sư DevOps được khuyến cáo tuyệt đối không map cổng `9090:9090` của container Prometheus ra ngoài máy host VPS trên môi trường Production?

[A]
Vì cổng 9090 sẽ xung đột trực tiếp với cổng chạy HTTP mặc định của Nginx Reverse Proxy.
[EXP]
Nginx mặc định chạy cổng 80/443, không xung đột trực tiếp với cổng 9090.
[B]
Vì giao diện web mặc định của Prometheus không có lớp xác thực bảo mật tài khoản (mật khẩu), hacker có thể truy cập tự do để xem thông số hạ tầng hoặc gửi lệnh xóa dữ liệu.
[EXP]
Đúng. Prometheus mặc định mở toang cửa. Do đó, ta chỉ mở cổng 9090 trong mạng nội bộ Docker để Grafana kết nối an toàn, không map ra ngoài Internet.
[C]
Vì làm như vậy sẽ khiến Prometheus không thể kéo được metrics từ các container Spring Boot.
[EXP]
Map cổng hay không không ảnh hưởng đến việc Prometheus kéo metrics nội bộ từ container Spring Boot.
[D]
Vì Prometheus Server chỉ hoạt động được khi cổng chạy của nó hoàn toàn trùng khớp với cổng microservice.
[EXP]
Prometheus hoạt động độc lập ở cổng của nó và kéo metrics từ bất kỳ cổng nào của target được khai báo.

@correct: B
@point: 20

## Q5

Giả sử bạn chạy lệnh kiểm tra targets từ bên trong container Prometheus và nhận được kết quả JSON hiển thị trạng thái `"health": "down"` kèm lỗi `"Connection refused"`. Đâu là bước debug SAI trong quy trình cô lập lỗi?

[A]
Chạy lệnh `docker compose logs` để kiểm tra xem microservice mục tiêu có đang gặp lỗi crash mã nguồn Java hay không.
[EXP]
Đây là bước debug đúng và cần thiết để xem ứng dụng Java có hoạt động bình thường trên cổng khai báo hay bị crash.
[B]
Chạy lệnh `docker network inspect` để xác nhận container Prometheus và microservice mục tiêu có nằm chung mạng ảo hay không.
[EXP]
Đây là bước debug đúng vì nếu lệch mạng Docker Network, Prometheus Server không thể phân giải DNS của container target.
[C]
Xóa hoàn toàn volume dữ liệu `prometheus-data` để reset lại cơ sở dữ liệu TSDB của Prometheus.
[EXP]
Đúng, đây là bước debug SAI. Việc xóa volume TSDB chỉ làm mất dữ liệu lịch sử đo đạc, không giải quyết được lỗi kết nối mạng ("Connection refused") giữa hai container.
[D]
Dùng lệnh `docker compose ps` kiểm tra xem container microservice mục tiêu có đang ở trạng thái `Up` hay không.
[EXP]
Đây là bước debug đúng để xác nhận tiến trình container target có đang chạy ngầm hay không.

@correct: C
@point: 20
