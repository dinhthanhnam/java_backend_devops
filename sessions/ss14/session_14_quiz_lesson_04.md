# QUIZ LESSON 04: TẠO DASHBOARD GIÁM SÁT SPRING BOOT SERVICE

## Q1

Mã ID phổ biến của Dashboard cộng đồng chuyên dùng để giám sát JVM và mã nguồn Spring Boot thông qua Micrometer là gì?

[A]
1860
[EXP]
1860 dùng cho Node Exporter để giám sát VPS, không dùng cho JVM Spring Boot.
[B]
4701
[EXP]
Đúng. Dashboard ID 4701 (JVM Micrometer) cung cấp đầy đủ thông số Heap Memory, Threads, GC và HTTP Latency của Spring Boot.
[C]
9090
[EXP]
9090 là cổng chạy mặc định của Prometheus Server, không phải ID của Dashboard Grafana.
[D]
8083
[EXP]
8083 là cổng chạy HTTP mặc định của order-service, không phải ID của Dashboard Grafana.

@correct: B
@point: 20

## Q2

Khi theo dõi chỉ số Heap Memory của container Spring Boot, xu hướng biến động nào dưới đây là dấu hiệu cảnh báo lỗi rò rỉ bộ nhớ (Memory Leak) đang diễn ra?

[A]
Dung lượng Heap Memory tăng giảm liên tục theo hình răng cưa sau mỗi chu kỳ chạy GC.
[EXP]
Đây là hiện tượng hoàn toàn bình thường khi Garbage Collector dọn dẹp các đối tượng rác định kỳ.
[B]
Dung lượng Heap Memory liên tục tăng dần theo thời gian và không hề giảm xuống sau các đợt dọn rác GC, tiến sát giới hạn Max Heap.
[EXP]
Đúng. Khi bộ nhớ đã sử dụng tăng liên tục không giảm dù GC đã chạy, chứng tỏ các đối tượng không dùng nữa vẫn bị giữ tham chiếu (Memory Leak), chuẩn bị gây sập OOM.
[C]
Dung lượng Heap Memory đột ngột giảm mạnh về 0 ngay khi người dùng đăng xuất khỏi hệ thống.
[EXP]
Đây là hành vi thu hồi bộ nhớ lý tưởng, không phải rò rỉ bộ nhớ.
[D]
Dung lượng Heap Memory luôn duy trì cố định ở mức 10% trong suốt một tuần vận hành.
[EXP]
Bộ nhớ ổn định ở mức thấp chứng tỏ ứng dụng hoạt động an toàn và tối ưu tài nguyên.

@correct: B
@point: 20

## Q3

Chỉ số GC (Garbage Collection) Pauses Time hiển thị thông số kỹ thuật nào của máy ảo Java?

[A]
Tổng thời gian mà ứng dụng dừng toàn bộ các luồng xử lý (Stop-the-world) để thực hiện dọn dác bộ nhớ RAM.
[EXP]
Đúng. Khi GC chạy (đặc biệt là Major GC), nó sẽ tạm dừng ứng dụng Java. Chỉ số này đo lường tổng thời gian dừng đó.
[B]
Thời gian tối đa mà một request HTTP được phép chờ trước khi bị Timeout.
[EXP]
Đây là cấu hình Connection Timeout của Web Server (Tomcat), không liên quan đến GC Pauses.
[C]
Số lượng đối tượng Java được tạo ra trong bộ nhớ đệm mỗi giây.
[EXP]
GC chỉ đếm thời gian dọn dẹp đối tượng rác, không đếm số lượng đối tượng được khởi tạo mới.
[D]
Tỷ lệ CPU mà tiến trình Java tiêu thụ khi thực hiện kết nối cơ sở dữ liệu.
[EXP]
Mức tiêu thụ CPU của kết nối DB được đo riêng biệt, không thuộc phạm vi của GC Pauses Time.

@correct: A
@point: 20

## Q4

Tại sao chúng ta có thể lọc riêng biệt thông số giám sát cho từng Microservice (như user-service, order-service) trên cùng một Dashboard JVM ID 4701?

[A]
Nhờ Grafana tự động đọc tên file JAR của từng microservice đang chạy trên VPS host.
[EXP]
Grafana không có quyền truy cập vào file hệ thống để đọc tên file JAR của Java.
[B]
Nhờ nhãn (label) `application` được tự động gán vào metrics thông qua cấu hình common tags trong tệp application.yml của từng service.
[EXP]
Đúng. Biến lọc ở đầu Dashboard của Grafana hoạt động dựa trên các nhãn định danh chung này để phân tách số liệu.
[C]
Do mỗi microservice bắt buộc phải sử dụng một phiên bản JVM khác nhau hoàn toàn.
[EXP]
Các microservices chạy chung phiên bản Java JRE/JDK vẫn được phân biệt bình thường nhờ nhãn metrics.
[D]
Do Prometheus Server tự động gán tên thư mục chứa mã nguồn vào các metrics thu thập được.
[EXP]
Prometheus chỉ gán các nhãn khai báo trong prometheus.yml hoặc nhận từ target, không biết cấu trúc thư mục mã nguồn.

@correct: B
@point: 20

## Q5

Khi chỉ số HTTP Latency (thời gian phản hồi API) trên Dashboard JVM tăng đột biến vào giờ cao điểm, hành động nào dưới đây là bước phân tích nguyên nhân phù hợp nhất?

[A]
Kiểm tra chỉ số Active Threads và thời gian truy vấn Database (Connection Pool) xem có bị tắc nghẽn hay không.
[EXP]
Đúng. Latency tăng thường do hết Thread xử lý (Active Threads đạt tối đa) hoặc Database phản hồi chậm khiến Thread bị block chờ đợi.
[B]
Xóa container Node Exporter vì Node Exporter đang chiếm dụng hết băng thông của mạng.
[EXP]
Node Exporter tiêu thụ rất ít tài nguyên và băng thông mạng nội bộ, việc xóa nó không giúp ích cho việc sửa lỗi latency.
[C]
Thực hiện cấu hình mở toang hoác tất cả các endpoints của Actuator bằng cấu hình include: "*".
[EXP]
Mở toang endpoint Actuator chỉ gây nguy cơ bảo mật, không giúp sửa lỗi nghẽn hay làm giảm Latency.
[D]
Ngay lập tức khởi động lại máy chủ vật lý VPS mà không cần xem log.
[EXP]
Đây là hành động mù quáng, có thể gây mất mát dữ liệu và không giúp tìm ra nguyên nhân gốc rễ để khắc phục lâu dài.

@correct: A
@point: 20
