# QUIZ LESSON 02: LOG LEVEL VÀ QUY ƯỚC GHI LOG BACKEND

## Q1

Trong hệ sinh thái Spring Boot và SLF4J, thứ tự sắp xếp chuẩn của các mức log theo chiều tăng dần về mức độ nghiêm trọng là gì?

[A]
`DEBUG → TRACE → INFO → ERROR → WARN`
[EXP]
Sai. TRACE là mức chi tiết nhất thấp hơn DEBUG, và WARN có độ ưu tiên thấp hơn ERROR.
[B]
`TRACE → DEBUG → INFO → WARN → ERROR`
[EXP]
Đúng. Thứ tự tăng dần từ chi tiết nhất (TRACE) đến sự cố nghiêm trọng nhất (ERROR).
[C]
`INFO → DEBUG → TRACE → WARN → ERROR`
[EXP]
Sai. INFO là mức thông tin nghiệp vụ cao hơn TRACE và DEBUG.
[D]
`TRACE → INFO → DEBUG → ERROR → WARN`
[EXP]
Sai. DEBUG nằm giữa TRACE và INFO, và WARN đứng trước ERROR.

@correct: B
@point: 20

## Q2

Trong quy trình đặt món của QuickBite, khi phát hiện nhà hàng đối tác đang tạm thời đóng cửa nên đơn hàng không thể tiếp tục, service nên ghi nhận sự kiện này ở mức log nào là phù hợp nhất?

[A]
`ERROR`
[EXP]
Sai. Nhà hàng đóng cửa là tình huống nghiệp vụ bình thường đã được lường trước, không phải lỗi kỹ thuật sập hệ thống.
[B]
`DEBUG`
[EXP]
Sai. Mức DEBUG thường bị tắt trên Production nên đội ngũ vận hành sẽ không thể theo dõi được tỷ lệ đơn bị từ chối do nhà hàng đóng.
[C]
`WARN`
[EXP]
Đúng. WARN dành cho các biến cố bất thường cần lưu ý nhưng ứng dụng đã kiểm soát được và có kịch bản phản hồi phù hợp.
[D]
`TRACE`
[EXP]
Sai. TRACE chỉ dùng để theo dõi từng bước thuật toán nội bộ trong môi trường phát triển cục bộ.

@correct: C
@point: 20

## Q3

Đoạn mã Java nào dưới đây tuân thủ đúng quy ước parameterized logging của SLF4J và đảm bảo ghi lại đầy đủ stacktrace khi xảy ra ngoại lệ?

[A]
`log.error("Payment processing failed: orderId={}", savedOrder.getId(), exception);`
[EXP]
Đúng. Sử dụng placeholder `{}` để định dạng chuỗi và truyền đối tượng exception ở tham số cuối cùng để Logback tự động in stacktrace.
[B]
`log.error("Payment processing failed for Order ID " + savedOrder.getId() + ": " + exception.getMessage());`
[EXP]
Sai. Dùng phép nối chuỗi (`+`) làm tốn bộ nhớ và chỉ lấy được message ngắn mà mất toàn bộ dấu vết stacktrace của exception.
[C]
`log.error("Payment processing failed: orderId={}, error={}", savedOrder.getId(), exception.toString());`
[EXP]
Sai. Truyền `exception.toString()` vào placeholder sẽ chỉ in một dòng mô tả ngoại lệ mà không xuất được cây stacktrace.
[D]
`log.error(String.format("Payment processing failed: orderId=%s, ex=%s", savedOrder.getId(), exception));`
[EXP]
Sai. `String.format` thực hiện nối chuỗi ngay lập tức và không tận dụng được cơ chế in stacktrace tự động của SLF4J.

@correct: A
@point: 20

## Q4

Tại sao không nên cấu hình ghi log mọi lỗi validation dữ liệu từ người dùng (HTTP 400 Bad Request) ở mức `WARN` hoặc `ERROR` trên môi trường Production?

[A]
Vì Spring Boot sẽ tự động dừng hoạt động của web server khi có quá nhiều log ERROR xuất hiện.
[EXP]
Sai. Spring Boot không tự động tắt máy chủ chỉ vì số lượng log ERROR tăng lên.
[B]
Vì các thư viện bảo mật sẽ chặn toàn bộ các request tiếp theo nếu phát hiện log validation bị ghi nhận.
[EXP]
Sai. Logback không tự động can thiệp vào quy trình chặn request của các bộ lọc bảo mật.
[C]
Vì ghi log validation sẽ làm thay đổi mã trạng thái HTTP trả về cho trình duyệt của khách hàng.
[EXP]
Sai. Hành vi ghi log hoàn toàn độc lập với mã phản hồi HTTP response trả về cho client.
[D]
Vì lỗi validation xảy ra rất thường xuyên từ phía người dùng, việc ghi log mức cao sẽ gây nhiễu loạn và che khuất các lỗi hệ thống nghiêm trọng thực sự.
[EXP]
Đúng. Người dùng nhập sai dữ liệu là hành vi phổ biến; ghi nhận mức WARN/ERROR sẽ làm tràn ngập màn hình giám sát và gây cảnh báo giả.

@correct: D
@point: 20

## Q5

Khi thực hiện ghi log sau khi phát thông báo thành công trong `NotificationService`, phương án nào dưới đây vừa tuân thủ tiêu chuẩn an toàn thông tin vừa hỗ trợ tốt cho việc điều tra sự cố?

[A]
`log.info("Notification sent: user={}, content={}", request.getUserId(), request.getContent());`
[EXP]
Sai. Trường `content` có thể chứa nội dung nhạy cảm như mã OTP giao dịch, số dư tài khoản hoặc thông tin riêng tư của khách hàng.
[B]
`log.info("Notification sent: notificationId={}, userId={}, type={}", notification.getId(), notification.getUserId(), notification.getType());`
[EXP]
Đúng. Chỉ ghi nhận các khóa định danh nghiệp vụ và loại thông báo, đảm bảo an toàn bảo mật và đủ thông tin tra cứu khi cần.
[C]
`log.info("Notification sent successfully: payload={}", request.toString());`
[EXP]
Sai. In toàn bộ đối tượng request payload sẽ làm rò rỉ toàn bộ thông tin chi tiết của thông báo ra log.
[D]
`log.info("Notification sent");*/`
[EXP]
Sai. Thông điệp quá chung chung, không có ID định danh khiến kỹ sư vận hành không thể tra cứu sự kiện thuộc về người dùng nào.

@correct: B
@point: 20
