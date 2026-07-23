# QUIZ LESSON 03: TẠO DASHBOARD GIÁM SÁT VPS

## Q1

Mã ID mặc định phổ biến của Dashboard cộng đồng được khuyên dùng để giám sát tài nguyên phần cứng VPS thông qua Node Exporter là gì?

[A]
4701
[EXP]
ID 4701 dùng để giám sát JVM Micrometer của Spring Boot, không dùng cho Node Exporter.
[B]
1860
[EXP]
Đúng. Dashboard ID 1860 (Node Exporter Full) là giao diện chuẩn hóa và chi tiết nhất để theo dõi CPU, RAM, Disk của máy chủ Linux.
[C]
8081
[EXP]
8081 là cổng chạy HTTP thông thường của microservice, không phải ID của Dashboard Grafana.
[D]
9100
[EXP]
9100 là cổng chạy mặc định của Node Exporter, không phải ID của Dashboard Grafana.

@correct: B
@point: 20

## Q2

Để đo lường tỷ lệ phần trăm CPU sử dụng thực tế của VPS (CPU Usage %) qua dữ liệu của Node Exporter, chúng ta sử dụng nguyên lý tính toán nào?

[A]
Đếm trực tiếp số lượng tiến trình đang chạy trên nhân Linux tại thời điểm hiện tại.
[EXP]
Đếm tiến trình không phản ánh chính xác phần trăm thời gian xử lý của CPU.
[B]
Lấy 100% trừ đi tỷ lệ thời gian CPU ở trạng thái rảnh rỗi (idle).
[EXP]
Đúng. Nguyên lý chuẩn là tính toán tốc độ thay đổi của thời gian rảnh rỗi (`node_cpu_seconds_total{mode="idle"}`) rồi lấy 100 trừ đi để ra tỷ lệ CPU đang bận xử lý.
[C]
Cộng gộp dung lượng bộ nhớ Heap RAM đang sử dụng của các container Java lại.
[EXP]
Dung lượng Heap RAM là chỉ số của RAM, không liên quan đến tỷ lệ sử dụng CPU.
[D]
Đo đạc tốc độ truyền tải gói tin trên card mạng eth0.
[EXP]
Băng thông mạng không thể dùng làm đại diện cho công suất CPU.

@correct: B
@point: 20

## Q3

Câu lệnh PromQL nào dưới đây dùng để tính toán chính xác tỷ lệ phần trăm CPU sử dụng của VPS trong khoảng thời gian 5 phút gần nhất?

[A]
`100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)`
[EXP]
Đúng. Đây là công thức chuẩn trong Node Exporter để tính tỷ lệ CPU hoạt động trung bình dựa trên tốc độ thay đổi của thời gian CPU ở trạng thái nhàn rỗi.
[B]
`sum(node_cpu_seconds_total{mode="active"}[5m]) / 100`
[EXP]
Sai. Cú pháp này không hợp lệ và không thể trả về tỷ lệ phần trăm chính xác của CPU hoạt động.
[C]
`avg(jvm_cpu_usage) * 100`
[EXP]
Sai. Chỉ số này của Spring Boot Actuator dùng để đo CPU của tiến trình JVM cụ thể, không đo được tổng thể phần cứng VPS.
[D]
`node_memory_Active_bytes / node_memory_MemTotal_bytes * 100`
[EXP]
Sai. Công thức này dùng để tính phần trăm RAM sử dụng của VPS, không dùng cho CPU.

@correct: A
@point: 20

## Q4

Tại sao chỉ số dung lượng ổ cứng trống (Disk Space Available) của VPS lại là chỉ số tối quan trọng mà các kỹ sư DevOps bắt buộc phải hiển thị trên Dashboard giám sát?

[A]
Vì nếu đĩa SSD bị đầy 100%, các hệ quản trị cơ sở dữ liệu (Database) sẽ bị lỗi ghi dữ liệu và crash đột ngột, gây dừng dịch vụ.
[EXP]
Đúng. Đầy ổ cứng là lỗi chí mạng thường xuyên xảy ra do log file phình to khiến Database ngưng hoạt động ngay lập tức.
[B]
Vì đĩa cứng đầy sẽ khiến tốc độ CPU giảm đi một nửa để tiết kiệm năng lượng VPS.
[EXP]
Ổ cứng đầy không làm giảm tốc độ xung nhịp CPU của máy chủ.
[C]
Vì Prometheus Server chỉ hoạt động được khi dung lượng ổ cứng trống lớn hơn 50%.
[EXP]
Prometheus vẫn hoạt động bình thường kể cả khi ổ cứng còn dưới 10% dung lượng, chỉ dừng khi đĩa bị đầy hoàn toàn.
[D]
Vì đĩa cứng trống càng nhiều thì container Spring Boot khởi động càng nhanh.
[EXP]
Tốc độ khởi động của Spring Boot phụ thuộc vào RAM, CPU và tốc độ đọc của đĩa, không phụ thuộc trực tiếp vào phần trăm đĩa trống.

@correct: A
@point: 20

## Q5

Khi xem biểu đồ mạng (Network Traffic) từ Node Exporter trên Dashboard, hai chỉ số `rx` và `tx` đại diện cho các thông số kỹ thuật nào?

[A]
rx là lượng RAM đã sử dụng, tx là tổng dung lượng RAM vật lý của VPS.
[EXP]
rx và tx thuộc nhóm chỉ số mạng, không phải RAM.
[B]
rx là băng thông mạng nhận vào (Receive - Download), tx là băng thông mạng gửi đi (Transmit - Upload).
[EXP]
Đúng. rx (Received) là dữ liệu tải xuống và tx (Transmitted) là dữ liệu tải lên qua card mạng của VPS.
[C]
rx là số lượng CPU core đang rảnh, tx là số lượng CPU core đang bận.
[EXP]
rx và tx không liên quan đến số nhân CPU.
[D]
rx là số request HTTP thành công, tx là số request HTTP bị lỗi 5xx.
[EXP]
rx và tx đo băng thông mạng vật lý ở tầng OS, không phân tích mã trạng thái HTTP của ứng dụng.

@correct: B
@point: 20
