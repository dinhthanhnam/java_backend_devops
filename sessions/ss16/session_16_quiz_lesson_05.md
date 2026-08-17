# QUIZ LESSON 05: MDC VÀ TRACEID TRONG LUỒNG REQUEST PHÂN TÁN

## Q1

Trong kiến trúc microservices gồm nhiều service gọi nối tiếp nhau, vai trò quan trọng nhất của mã định danh `traceId` là gì?

[A]
Dùng để xác thực chữ ký điện tử và phân quyền người dùng khi truy cập vào các API nội bộ.
[EXP]
Sai. Việc xác thực và phân quyền là nhiệm vụ của JWT Token hoặc API Key, không phải của traceId.
[B]
Xâu chuỗi và liên kết toàn bộ các sự kiện log phát sinh độc lập tại nhiều microservices khác nhau thuộc về cùng một luồng request của khách hàng.
[EXP]
Đúng. `traceId` là mã định danh xuyên suốt giúp kỹ sư vận hành gom nhóm và tái hiện toàn bộ hành trình của một giao dịch qua các container.
[C]
Tự động tối ưu hóa đường truyền mạng giúp giảm độ trễ khi gọi HTTP giữa các container.
[EXP]
Sai. `traceId` chỉ là dữ liệu định danh phục vụ giám sát và điều tra, không can thiệp vào hiệu năng mạng.
[D]
Dùng làm khóa chính (Primary Key) trong các bảng của cơ sở dữ liệu quan hệ MySQL.
[EXP]
Sai. Khóa chính trong database là các ID nghiệp vụ (như `order_id`, `user_id`), không nên dùng `traceId` làm khóa chính bảng.

@correct: B
@point: 20

## Q2

MDC (Mapped Diagnostic Context) trong thư viện SLF4J hoạt động dựa trên cơ chế kỹ thuật nào để lưu trữ dữ liệu ngữ cảnh log theo từng luồng request?

[A]
`ThreadLocal` - lưu trữ dữ liệu ngữ cảnh gắn liền riêng biệt với luồng (thread) đang thực thi request hiện tại.
[EXP]
Đúng. MDC sử dụng `ThreadLocal` giúp dữ liệu được truy cập toàn cục trong cùng một luồng mà không cần phải truyền tham số qua từng hàm Java.
[B]
`ServletContext` - chia sẻ dữ liệu chung cho toàn bộ các người dùng trong ứng dụng web.
[EXP]
Sai. `ServletContext` là vùng nhớ chung toàn ứng dụng; nếu lưu traceId vào đây sẽ khiến các request của người dùng khác nhau bị đè dữ liệu lên nhau.
[C]
`HttpSession` - lưu trữ trạng thái người dùng tạm thời trên bộ nhớ RAM của máy chủ.
[EXP]
Sai. Trong kiến trúc microservices không trạng thái (stateless), hệ thống không sử dụng HttpSession để lưu traceId.
[D]
`Static Heap Variable` - biến tĩnh được nạp sẵn vào bộ nhớ heap khi ứng dụng khởi động.
[EXP]
Sai. Biến static thông thường không an toàn trong môi trường đa luồng (multi-threaded) và sẽ gây xung đột dữ liệu giữa các request.

@correct: A
@point: 20

## Q3

Tại sao trong `TraceIdFilter`, lệnh dọn dẹp `MDC.remove("trace.id")` bắt buộc phải được đặt bên trong khối lệnh `finally`?

[A]
Để kích hoạt tiến trình Garbage Collection giải phóng bộ nhớ heap của máy ảo Java ngay lập tức.
[EXP]
Sai. `MDC.remove()` chỉ xóa entry trong map của ThreadLocal, không trực tiếp ra lệnh cho Garbage Collector.
[B]
Để ngăn chặn trình duyệt web của khách hàng tự động gửi lại request bị lỗi.
[EXP]
Sai. Khối `finally` của filter chạy ở backend và không điều khiển hành vi của trình duyệt web.
[C]
Vì các web server như Tomcat sử dụng Thread Pool để tái sử dụng luồng; nếu không xóa, luồng đó khi nhận request mới sẽ bị gán nhầm `traceId` của request cũ.
[EXP]
Đúng. Xóa MDC trong `finally` đảm bảo luồng luôn được làm sạch (kể cả khi có Exception), tránh hiện tượng ô nhiễm ngữ cảnh log (context pollution).
[D]
Để mã hóa thông điệp log trước khi gửi sang hệ thống thu thập tập trung.
[EXP]
Sai. Khối lệnh này không liên quan đến việc mã hóa dữ liệu.

@correct: C
@point: 20

## Q4

Thành phần nào trong `order-service` chịu trách nhiệm đọc `trace.id` từ MDC và đính kèm vào header HTTP `X-Trace-Id` khi gọi sang `restaurant-service` qua OpenFeign?

[A]
`TraceIdFilter`
[EXP]
Sai. `TraceIdFilter` chỉ chịu trách nhiệm tiếp nhận request đến (inbound) ở tầng Servlet, không can thiệp vào request gửi đi của Feign Client.
[B]
`Logback Appender`
[EXP]
Sai. Appender chỉ xuất log ra console/file, không tham gia vào luồng gửi HTTP request giữa các service.
[C]
`EcsEncoder`
[EXP]
Sai. Encoder chỉ có nhiệm vụ định dạng log event sang chuỗi JSON.
[D]
`FeignRequestInterceptor` (triển khai giao diện `feign.RequestInterceptor`)
[EXP]
Đúng. Interceptor chặn các request gửi đi (outbound) của Feign, lấy `trace.id` từ MDC và thêm vào header `X-Trace-Id` để lan truyền sang service đích.

@correct: D
@point: 20

## Q5

Nếu một request từ Client gửi tới API của `order-service` mà KHÔNG có header `X-Trace-Id`, `TraceIdFilter` sẽ xử lý như thế nào theo chuẩn thiết kế của QuickBite?

[A]
Từ chối request ngay lập tức và trả về mã lỗi HTTP 400 Bad Request cho Client.
[EXP]
Sai. Thiếu header trace không phải là lỗi nghiệp vụ; service vẫn tiếp nhận và xử lý bình thường.
[B]
Tự động sinh ra một chuỗi UUID ngẫu nhiên mới, nạp vào MDC của luồng và đính kèm vào response header trả về cho Client.
[EXP]
Đúng. Nếu request chưa có mã trace, service đầu vào sẽ đóng vai trò khởi tạo UUID mới để đảm bảo luồng luôn có mã định danh truy vết.
[C]
Bỏ qua việc ghi log và không khởi tạo bất kỳ dữ liệu nào trong MDC.
[EXP]
Sai. Mọi request đều cần có traceId để phục vụ việc giám sát và điều tra sự cố.
[D]
Sử dụng địa chỉ IP của Client để làm giá trị thay thế cho `trace.id`.
[EXP]
Sai. Địa chỉ IP không thể dùng làm traceId vì nhiều request từ cùng một IP sẽ bị trùng lặp mã định danh.

@correct: B
@point: 20
