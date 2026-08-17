# QUIZ LESSON 01: LOGGING TẬP TRUNG TRONG HỆ THỐNG MICROSERVICES VÀ TỔNG QUAN EFK

## Q1

Trong ngăn xếp EFK, thành phần nào chịu trách nhiệm tiếp nhận luồng log từ Docker, bóc tách chuỗi JSON, bổ sung metadata container và đệm dữ liệu trước khi chuyển tiếp?

[A]
Kibana
[EXP]
Sai. Kibana là giao diện trực quan hóa dữ liệu và cung cấp công cụ tìm kiếm cho kỹ sư vận hành, không trực tiếp thu gom log.
[B]
Fluentd
[EXP]
Đúng. Fluentd đóng vai trò là Collector/Shipper: nhận log qua giao thức forward, parse JSON, đính kèm metadata và đẩy sang Elasticsearch.
[C]
Elasticsearch
[EXP]
Sai. Elasticsearch là kho lưu trữ và lập chỉ mục (index) phân tán, không trực tiếp lắng nghe luồng log từ Docker daemon.
[D]
Spring Boot Actuator
[EXP]
Sai. Spring Boot Actuator là module đo lường chỉ số (metrics) và kiểm tra sức khỏe của ứng dụng Java, không thuộc ngăn xếp EFK.

@correct: B
@point: 20

## Q2

Tại sao ứng dụng Spring Boot trong hệ thống QuickBite KHÔNG NÊN cấu hình để tự gửi trực tiếp log qua HTTP API sang Elasticsearch?

[A]
Vì Elasticsearch chỉ hỗ trợ nhận dữ liệu thông qua giao thức kết nối cơ sở dữ liệu JDBC của Java.
[EXP]
Sai. Elasticsearch hỗ trợ đầy đủ RESTful HTTP API để nhận dữ liệu JSON.
[B]
Vì việc gửi trực tiếp làm ứng dụng phụ thuộc chặt chẽ vào kho log, gây tăng độ trễ xử lý nghiệp vụ và có nguy cơ gián đoạn nếu Elasticsearch gặp sự cố.
[EXP]
Đúng. Mô hình chuẩn là ứng dụng chỉ ghi log ra console (`stdout`), việc thu gom và chuyển tiếp có buffer an toàn thuộc về lớp hạ tầng của Docker và Fluentd.
[C]
Vì Spring Boot không hỗ trợ các thư viện định dạng log theo chuẩn JSON.
[EXP]
Sai. Spring Boot hỗ trợ mạnh mẽ Logback cùng các thư viện như `logback-ecs-encoder` để sinh log JSON.
[D]
Vì Docker container sẽ tự động chặn toàn bộ các kết nối HTTP hướng ngoại của ứng dụng Spring Boot.
[EXP]
Sai. Docker container cho phép kết nối mạng bình thường nếu cấu hình network đúng.

@correct: B
@point: 20

## Q3

Khi cấu hình Docker Logging Driver cho các microservices, tại sao địa chỉ của Collector bắt buộc phải khai báo là `fluentd-address: "127.0.0.1:24224"` thay vì `fluentd-address: "fluentd:24224"`?

[A]
Vì Docker Logging Driver được thực thi bởi tiến trình Docker Daemon trên hệ điều hành host, nơi không sử dụng cơ chế Docker DNS nội bộ của container network.
[EXP]
Đúng. Tiến trình Docker daemon chạy trên host bên ngoài container network nên không thể phân giải tên miền `fluentd`, bắt buộc phải trỏ vào IP loopback của host.
[B]
Vì cổng 24224 của Fluentd chỉ chấp nhận kết nối từ các địa chỉ IP thuộc lớp mạng công cộng (Public IP).
[EXP]
Sai. Cổng 24224 nhận kết nối từ loopback hoặc mạng nội bộ.
[C]
Vì tên miền `fluentd` đã được hệ điều hành Ubuntu gán cố định cho dịch vụ DNS máy chủ.
[EXP]
Sai. Ubuntu không có tên miền mặc định là `fluentd`; tên miền này chỉ tồn tại bên trong mạng ảo của Docker.
[D]
Vì Fluentd từ chối tiếp nhận các gói tin log nếu không có chứng chỉ bảo mật TLS khi dùng tên miền.
[EXP]
Sai. Lỗi kết nối xuất phát từ cơ chế định tuyến mạng của Docker daemon, không liên quan đến TLS.

@correct: A
@point: 20

## Q4

Rủi ro bảo mật lớn nhất khi mở công khai cổng `9200` (Elasticsearch API) và cổng `5601` (Kibana) ra mạng Internet trên máy chủ VPS là gì?

[A]
Khiến ứng dụng Spring Boot bị mất kết nối tới cơ sở dữ liệu MySQL nội bộ.
[EXP]
Sai. Cổng 9200 và 5601 không ảnh hưởng trực tiếp tới kết nối database của ứng dụng.
[B]
Làm tăng gấp đôi chi phí băng thông đường truyền mạng nội bộ giữa các container trong mạng `quickbite-net`.
[EXP]
Sai. Tấn công mạng nhắm vào dữ liệu và quyền truy cập chứ không chỉ là chi phí băng thông.
[C]
Kẻ tấn công có thể trích xuất toàn bộ dữ liệu log nhạy cảm của hệ thống, thao túng chỉ mục hoặc chiếm quyền điều khiển bảng giám sát.
[EXP]
Đúng. Elasticsearch chứa toàn bộ nhật ký nghiệp vụ và lỗi hệ thống, mở công khai sẽ làm lộ dữ liệu và tạo điều kiện cho các cuộc tấn công phá hoại.
[D]
Khiến Docker daemon tự động dừng hoạt động của tất cả các microservices nghiệp vụ.
[EXP]
Sai. Docker daemon không tự tắt container khi có cổng bị mở ra Internet.

@correct: C
@point: 20

## Q5

Trong các phương án dưới đây, phương án nào thể hiện đúng nhất các hạn chế của việc chỉ kiểm tra log thủ công từng container bằng lệnh `docker compose logs` trên môi trường Production?

[A]
Log bị phân mảnh theo từng container, có nguy cơ mất lịch sử khi container bị tạo lại và không thể lọc trực quan đa chiều theo `trace.id` trên toàn hệ thống.
[EXP]
Đúng. Đây là ba hạn chế lớn nhất được nêu trong bài học, thúc đẩy sự cần thiết phải triển khai giải pháp logging tập trung EFK.
[B]
Lệnh `docker compose logs` làm tiêu tốn 100% dung lượng RAM của máy chủ VPS mỗi khi thực thi.
[EXP]
Sai. Lệnh này chỉ đọc tệp log của container, không làm cạn kiệt toàn bộ RAM của máy chủ.
[C]
Docker Compose không hỗ trợ tính năng xuất log theo thời gian thực với cờ theo dõi `-f`.
[EXP]
Sai. Docker Compose hỗ trợ đầy đủ cờ `-f` để live stream log.
[D]
Lệnh `docker compose logs` chỉ hiển thị được các dòng log ở mức `ERROR` và tự động ẩn các log `INFO`.
[EXP]
Sai. Lệnh này hiển thị toàn bộ luồng `stdout`/`stderr` của container mà không tự ý lọc bỏ mức độ nào.

@correct: A
@point: 20
