# QUIZ LESSON 04: TRIỂN KHAI NODE EXPORTER GIÁM SÁT TÀI NGUYÊN VPS

## Q1

Vai trò cốt lõi của Node Exporter trong hệ thống giám sát Prometheus tập trung là gì?

[A]
Thu thập các số liệu về Heap Memory, GC pause và số lượng Thread nội bộ của máy ảo Java.
[EXP]
Đây là vai trò của Spring Boot Actuator, không phải của Node Exporter.
[B]
Đo đạc và hiển thị các chỉ số liên quan đến tài nguyên vật lý và hệ điều hành của máy chủ host VPS (như CPU load, Memory usage, Disk I/O, Network Traffic).
[EXP]
Đúng. Node Exporter được cài đặt để thu thập các metrics cấp hệ điều hành (OS) và phần cứng của máy chủ VPS.
[C]
Đóng vai trò là cơ sở dữ liệu chuỗi thời gian để lưu trữ toàn bộ các metrics thu thập được.
[EXP]
Đây là vai trò của Prometheus Server (TSDB), Node Exporter không lưu trữ dữ liệu lâu dài.
[D]
Tự động gửi cảnh báo qua ứng dụng Telegram khi phát hiện Microservice bị sập nguồn.
[EXP]
Đây là vai trò của Alertmanager hoặc Grafana Alerting, Node Exporter chỉ xuất metrics thô.

@correct: B
@point: 20

## Q2

Tại sao khi triển khai Node Exporter dưới dạng container Docker, chúng ta bắt buộc phải mount thư mục `/proc` và `/sys` của máy host VPS kèm theo cờ `:ro` (Read-Only)?

[A]
Để Node Exporter có quyền sửa đổi cấu hình phần cứng của VPS host trực tiếp từ bên trong container.
[EXP]
Node Exporter chỉ có nhiệm vụ đọc/giám sát chỉ số hệ thống, không được phép chỉnh sửa tài nguyên máy host.
[B]
Để hạn chế tốc độ đọc ghi đĩa cứng của container Node Exporter, tránh làm VPS bị nghẽn đĩa.
[EXP]
Cờ `:ro` không dùng để giới hạn băng thông I/O của đĩa.
[C]
Để đảm bảo an toàn bảo mật, cho phép Node Exporter đọc dữ liệu hệ thống ảo để lấy metrics nhưng tuyệt đối không có quyền can thiệp, chỉnh sửa hay phá hoại hệ điều hành host.
[EXP]
Đúng. Chế độ Read-Only `:ro` đảm bảo an toàn tuyệt đối cho hệ điều hành host VPS trước container Node Exporter.
[D]
Để Docker tự động mở cổng mạng 9100 ra Internet mà không cần khai báo ports.
[EXP]
Mount volume không tự động mở hoặc map cổng mạng của container.

@correct: C
@point: 20

## Q3

Khi hệ thống xảy ra sự cố sập container Java Spring Boot do lỗi cạn kiệt bộ nhớ RAM máy ảo (Out Of Memory - Exit Code 137), chỉ số nào từ Spring Boot Actuator sẽ giúp bạn nhận diện sự cố này trước lúc ứng dụng sập?

[A]
Chỉ số dung lượng đĩa cứng SSD trống được đo từ file hệ thống ảo.
[EXP]
Đĩa cứng trống không biểu thị tình trạng cạn kiệt bộ nhớ RAM (Heap Memory) của ứng dụng Java.
[B]
Chỉ số `jvm_memory_used_bytes` tăng kịch trần tiến sát giới hạn tối đa `jvm_memory_max_bytes` và thời gian dọn rác GC pauses tăng vọt.
[EXP]
Đúng. Khi cạn kiệt Heap RAM, bộ nhớ sử dụng của JVM sẽ chạm mốc tối đa, đồng thời Java liên tục chạy tiến trình dọn rác (GC) để cứu vãn tài nguyên khiến thời gian GC pause tăng cao.
[C]
Chỉ số băng thông mạng card eth0 vượt ngưỡng giới hạn cho phép.
[EXP]
Băng thông mạng không phản ánh trực tiếp trạng thái bộ nhớ Heap RAM của máy ảo Java.
[D]
Chỉ số đo lường tải CPU của hệ điều hành VPS host đạt mức 100%.
[EXP]
CPU load của host VPS là chỉ số vĩ mô, không trực tiếp phản ánh vùng nhớ Heap bên trong container JVM.

@correct: B
@point: 20

## Q4

Nhận định nào dưới đây mô tả chính xác sự khác biệt về đối tượng cài đặt giữa Spring Boot Actuator và Node Exporter?

[A]
Actuator được tích hợp trực tiếp vào mã nguồn của từng microservice; Node Exporter được cài đặt 1 instance duy nhất trên mỗi máy chủ VPS.
[EXP]
Đúng. Actuator là thư viện Java nằm trong code ứng dụng, còn Node Exporter là một agent độc lập chạy trực tiếp trên hệ điều hành VPS (hoặc container riêng) để giám sát máy chủ đó.
[B]
Actuator chạy như một container độc lập trên VPS; Node Exporter được nhúng trực tiếp vào mã nguồn Java Spring Boot.
[EXP]
Nhận định này bị đảo ngược vị trí giữa hai công cụ.
[C]
Cả Actuator và Node Exporter đều phải nhúng chung vào trong cùng một dự án mã nguồn Java.
[EXP]
Node Exporter là phần mềm viết bằng Go, chạy độc lập, không thể nhúng vào dự án Java Spring Boot.
[D]
Actuator chỉ chạy được trên môi trường Windows; Node Exporter chỉ chạy được trên môi trường Linux.
[EXP]
Cả hai công cụ đều có khả năng chạy đa nền tảng (Windows, Linux, macOS...).

@correct: A
@point: 20

## Q5

Tham số cấu hình `--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($|/)` của Node Exporter mang lại lợi ích gì cho việc lưu trữ và truy vấn metrics của Prometheus?

[A]
Kích hoạt tính năng nén dữ liệu metrics dạng text để giảm thiểu băng thông mạng truyền tải.
[EXP]
Tham số này không có chức năng nén dữ liệu.
[B]
Loại trừ các hệ thống tệp tin ảo của Linux (vốn không chiếm ổ cứng thực tế) khỏi danh sách đo đạc, giúp loại bỏ các metrics rác làm phình to TSDB và làm đơn giản câu lệnh PromQL.
[EXP]
Đúng. Loại trừ các phân vùng ảo như `/proc`, `/sys`, `/dev`... giúp Prometheus chỉ tập trung quét dung lượng ổ cứng vật lý thực tế (SSD) của VPS.
[C]
Cho phép Node Exporter tự động phân giải DNS của Prometheus Server trong mạng Docker.
[EXP]
Tham số này không cấu hình DNS hay cơ chế phân giải tên miền.
[D]
Bắt buộc Node Exporter chỉ được xuất metrics khi VPS bị đầy đĩa cứng 100%.
[EXP]
Node Exporter xuất metrics liên tục theo chu kỳ scrape, không phụ thuộc vào trạng thái đầy đĩa.

@correct: B
@point: 20
