# QUIZ LESSON 01: GRAFANA VÀ VAI TRÒ TRONG GIÁM SÁT HỆ THỐNG

## Q1

Mục đích cốt lõi của việc tích hợp Grafana vào hệ thống giám sát Microservices là gì?

[A]
Để lưu trữ và quản lý cơ sở dữ liệu chuỗi thời gian (TSDB) thay thế cho Prometheus Server.
[EXP]
Grafana không tự lưu trữ dữ liệu metrics, cơ sở dữ liệu TSDB vẫn do Prometheus Server đảm nhiệm.
[B]
Đóng vai trò là tầng hiển thị (Visualization Layer) giúp truy vấn dữ liệu từ Prometheus và vẽ các biểu đồ trực quan động.
[EXP]
Đúng. Grafana là công cụ frontend chuyên trực quan hóa dữ liệu từ các nguồn khác nhau.
[C]
Để chủ động gửi yêu cầu HTTP GET kéo dữ liệu trực tiếp từ các container microservices.
[EXP]
Việc kéo dữ liệu thô (Scraping) từ container microservices là nhiệm vụ của Prometheus, không phải của Grafana.
[D]
Để tự động tối ưu hóa dung lượng RAM và dọn rác GC cho máy chủ VPS.
[EXP]
Grafana chỉ hiển thị chỉ số, không thể can thiệp hệ thống để tối ưu RAM hay GC cho VPS.

@correct: B
@point: 20

## Q2

Khi so sánh giữa giao diện mặc định của Prometheus ở cổng 9090 và giao diện của Grafana ở cổng 3000, nhận định nào dưới đây phản ánh đúng hạn chế của Prometheus UI?

[A]
Prometheus UI không hỗ trợ viết các câu lệnh truy vấn PromQL để lấy metrics.
[EXP]
Prometheus UI hoàn toàn hỗ trợ viết và chạy thử câu lệnh PromQL.
[B]
Prometheus UI không có cơ chế đăng nhập bảo mật mặc định và không thể lưu nhiều đồ thị thành một bảng điều khiển hoàn chỉnh.
[EXP]
Đúng. Prometheus UI không có login, không có phân quyền và chỉ hiển thị biểu đồ đơn lẻ cho một câu truy vấn tại một thời điểm.
[C]
Prometheus UI tiêu tốn băng thông mạng gấp nhiều lần so với Grafana.
[EXP]
Prometheus UI rất thô sơ nên thực tế tiêu tốn ít tài nguyên và băng thông hơn so với giao diện đồ họa nặng của Grafana.
[D]
Prometheus UI chỉ hiển thị được metrics của ứng dụng Windows, không hiển thị được Linux.
[EXP]
Prometheus UI hiển thị bất kỳ metrics nào lưu trong TSDB, không phụ thuộc vào hệ điều hành.

@correct: B
@point: 20

## Q3

Trong kiến trúc giám sát 3 phân lớp tiêu chuẩn (Target -> Prometheus -> Grafana), luồng truyền tải dữ liệu khi kỹ sư DevOps xem Dashboard trên Grafana diễn ra như thế nào?

[A]
Grafana chủ động pull metrics trực tiếp từ Target → Lưu vào RAM của Grafana → Hiển thị lên trình duyệt.
[EXP]
Grafana không pull metrics trực tiếp từ Target và không lưu trữ dữ liệu này.
[B]
Target đẩy metrics về Grafana → Grafana đẩy tiếp sang lưu trữ ở Prometheus.
[EXP]
Cả Target và Grafana đều không đẩy dữ liệu theo hướng này.
[C]
Grafana gửi câu lệnh PromQL đến Prometheus API → Prometheus lấy dữ liệu từ đĩa cứng trả về → Grafana vẽ đồ thị.
[EXP]
Đúng. Grafana đóng vai trò client truy vấn dữ liệu từ Prometheus Server qua HTTP API của Prometheus.
[D]
Prometheus tự động gửi cảnh báo đẩy (Push) dữ liệu ảnh chụp biểu đồ sang Grafana mỗi 15 giây.
[EXP]
Prometheus không tự động đẩy ảnh biểu đồ sang Grafana.

@correct: C
@point: 20

## Q4

Khi cấu hình container Grafana chạy bằng Docker Compose, tại sao việc thiết lập volume mount cho thư mục `/var/lib/grafana` là bắt buộc?

[A]
Để lưu trữ toàn bộ lịch sử dữ liệu metrics chuỗi thời gian cào được từ microservices.
[EXP]
Dữ liệu metrics được lưu tại Prometheus volume, không lưu ở Grafana.
[B]
Để lưu giữ các cấu hình kết nối nguồn dữ liệu (Data Source), tài khoản người dùng và thiết kế Dashboard khi container restart.
[EXP]
Đúng. Thư mục này chứa cơ sở dữ liệu sqlite nội bộ của Grafana lưu trữ toàn bộ cấu hình hoạt động của ứng dụng.
[C]
Để kích hoạt tính năng gửi cảnh báo qua Webhook Telegram.
[EXP]
Tính năng Alerting gửi qua mạng HTTP Webhook, không phụ thuộc vào việc mount volume này.
[D]
Để Docker Engine tự động đổi mật khẩu tài khoản admin mặc định khi phát hiện sự cố bảo mật.
[EXP]
Docker Engine không tự động quản lý hay thay đổi mật khẩu của Grafana.

@correct: B
@point: 20

## Q5

Bảo mật của cổng map Grafana (3000) ra ngoài host VPS khác gì so với cổng map Prometheus (9090) ra ngoài máy host trên môi trường Production?

[A]
Cổng 3000 an toàn hơn vì Grafana tích hợp sẵn hệ thống đăng nhập và phân quyền tài khoản (RBAC) bắt buộc.
[EXP]
Đúng. Hacker tiếp cận cổng 3000 của Grafana vẫn phải đi qua màn hình xác thực đăng nhập, trong khi cổng 9090 của Prometheus mặc định mở tự do không có mật khẩu.
[B]
Cổng 9090 an toàn hơn vì Prometheus tự động mã hóa SSL/TLS cho mọi kết nối đi vào.
[EXP]
Prometheus mặc định không tự động bật SSL/TLS, nó kết nối bằng HTTP thông thường.
[C]
Không có sự khác biệt, cả hai cổng đều bắt buộc phải chặn hoàn toàn bằng tường lửa VPS.
[EXP]
Thực tế cổng 3000 của Grafana được phép mở ra ngoài Internet để DevOps truy cập giao diện, còn cổng 9090 của Prometheus bắt buộc phải chặn.
[D]
Cổng 3000 kém an toàn hơn vì nó sử dụng giao thức truyền tin Push-based dễ bị tấn công DDoS.
[EXP]
Grafana sử dụng HTTP thông thường, không liên quan đến giao thức Push-based dễ bị tấn công hơn.

@correct: A
@point: 20
