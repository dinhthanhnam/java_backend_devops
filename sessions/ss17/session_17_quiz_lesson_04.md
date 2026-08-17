# QUIZ LESSON 04: PHÂN TÍCH VÀ DEBUG LOG TRONG KIBANA BẰNG TRACEID

## Q1

Tại sao trường `@timestamp` bắt buộc phải được chọn làm Time Field khi thiết lập Data View cho tập chỉ mục `quickbite-*` trong giao diện Kibana?

[A]
Để yêu cầu Kibana tự động xóa các dòng log cũ hơn 30 ngày khỏi ổ đĩa cứng của Elasticsearch.
[EXP]
Sai. Việc xóa dữ liệu cũ do chính sách ILM (Index Lifecycle Management) của Elasticsearch quản lý.
[B]
Vì `@timestamp` là mốc thời gian chuẩn do ứng dụng Spring Boot sinh ra, giúp Kibana đồng bộ chính xác bộ lọc thời gian và phân bố dữ liệu trên biểu đồ histogram.
[EXP]
Đúng. Kibana cần Time Field để sắp xếp sự kiện theo tiến trình thời gian thực và áp dụng các bộ lọc như *Last 15 minutes*.
[C]
Để kích hoạt tính năng gửi cảnh báo tự động qua Email khi có lỗi phát sinh.
[EXP]
Sai. Tính năng Alerting được cấu hình riêng trong phần Kibana Alerts & Rules, không phụ thuộc vào tên Time Field.
[D]
Vì Elasticsearch chỉ cho phép tìm kiếm dữ liệu khi chỉ mục có trường mang tên `@timestamp`.
[EXP]
Sai. Elasticsearch tìm kiếm được trên mọi trường, nhưng Kibana cần Time Field để trực quan hóa theo chuỗi thời gian.

@correct: B
@point: 20

## Q2

Câu truy vấn KQL (Kibana Query Language) nào dưới đây dùng để lọc chính xác tất cả các dòng log có mức `ERROR` phát sinh từ microservice `order-service` trong thanh tìm kiếm của Discover?

[A]
`service = "order-service" && level = "ERROR"`
[EXP]
Sai. Cú pháp KQL chuẩn sử dụng dấu hai chấm `:` thay vì dấu bằng `=`, và dùng từ khóa `and` thay vì `&&`.
[B]
`SELECT * FROM logs WHERE service = "order-service" AND level = "ERROR"`
[EXP]
Sai. Đây là cú pháp câu lệnh SQL truyền thống, không phải cú pháp của Kibana Query Language (KQL).
[C]
`service.name : "order-service" and log.level : "ERROR"`
[EXP]
Đúng. Tên trường ECS chuẩn là `service.name` và `log.level`, kết hợp toán tử điều kiện `and` của KQL.
[D]
`tags: "order-service" OR severity: "ERROR"`
[EXP]
Sai. Tên trường không đúng theo chuẩn ECS và toán tử `OR` sẽ lấy tất cả log của order-service lẫn log ERROR của các service khác.

@correct: C
@point: 20

## Q3

Khi lần theo một chuỗi log theo `trace.id` trong Kibana Discover, tại sao dòng log `ERROR` xuất hiện ở cuối chuỗi thường KHÔNG PHẢI là nguyên nhân gốc rễ (Root Cause) của sự cố?

[A]
Vì trong chuỗi gọi liên dịch vụ, lỗi phát sinh đầu tiên từ service phụ thuộc tầng dưới, còn service tầng trên chỉ ghi nhận log lỗi tổng thể do nhận phản hồi thất bại từ tầng dưới.
[EXP]
Đúng. Ví dụ `user-service` bị lỗi database ném exception trước, khiến `order-service` nhận mã lỗi 500 và ghi log ERROR sau cùng; nguyên nhân gốc rễ nằm ở sự kiện lỗi đầu tiên.
[B]
Vì Kibana tự động đảo ngược thứ tự các dòng log bị lỗi nghiêm trọng xuống dưới cùng của danh sách.
[EXP]
Sai. Thứ tự hiển thị hoàn toàn do người dùng chọn sắp xếp theo cột `@timestamp`.
[C]
Vì Elasticsearch luôn ưu tiên ghi nhận các dòng log của `order-service` chậm hơn các service khác 5 giây.
[EXP]
Sai. Độ trễ ghi nhận chỉ phụ thuộc vào thời điểm gửi của collector, không có quy tắc làm chậm 5 giây.
[D]
Vì dòng log cuối cùng luôn là thông điệp tự sinh của hệ thống Docker logging driver.
[EXP]
Sai. Dòng log cuối cùng là log nghiệp vụ do ứng dụng Spring Boot tạo ra.

@correct: A
@point: 20

## Q4

Khi thực hiện điều tra sự cố của một request cụ thể bằng câu truy vấn `trace.id : "..."` trong Discover, thao tác sắp xếp nào dưới đây giúp kỹ sư theo dõi diễn biến nghiệp vụ một cách trực quan và logic nhất?

[A]
Sắp xếp theo trường `log.level` từ A đến Z.
[EXP]
Sai. Sắp xếp theo log level sẽ làm xáo trộn trình tự thời gian của các bước xử lý nghiệp vụ.
[B]
Sắp xếp theo độ dài của thông điệp `message` từ ngắn nhất đến dài nhất.
[EXP]
Sai. Độ dài thông điệp không có ý nghĩa trong việc tái hiện luồng nghiệp vụ.
[C]
Sắp xếp ngẫu nhiên để quan sát tổng quan các container tham gia vào hệ thống.
[EXP]
Sai. Debug cần trình tự thời gian chính xác, không dùng sắp xếp ngẫu nhiên.
[D]
Sắp xếp cột `@timestamp` theo chiều tăng dần (Oldest first / từ cũ đến mới).
[EXP]
Đúng. Sắp xếp từ cũ đến mới giúp tái hiện chính xác từng bước xử lý theo dòng thời gian thực từ lúc nhận request đến khi kết thúc.

@correct: D
@point: 20

## Q5

Khi kiểm tra log trên Kibana, kỹ sư phát hiện dòng log sau: `WARN [order-service] Order rejected: restaurantId=REST-001, reason=RESTAURANT_CLOSED`. Dòng log này thuộc nhóm phân loại sự cố nào?

[A]
Lỗi sập hạ tầng hệ thống (System Outage) cần can thiệp khởi động lại toàn bộ máy chủ VPS.
[EXP]
Sai. Đây không phải lỗi hạ tầng hệ điều hành hay phần cứng.
[B]
Lỗi nghiệp vụ có kiểm soát (Business Flow) đã được ứng dụng lường trước và phản hồi mã lỗi phù hợp cho client.
[EXP]
Đúng. Nhà hàng tạm đóng là tình huống nghiệp vụ thông thường, ứng dụng xử lý từ chối đơn có kiểm soát và ghi nhận mức WARN.
[C]
Lỗi mất kết nối mạng nghiêm trọng giữa Docker daemon và Fluentd Collector.
[EXP]
Sai. Nếu mất kết nối collector thì log đã không thể xuất hiện trên giao diện Kibana.
[D]
Lỗi xung đột khóa chính bên trong bảng cơ sở dữ liệu MySQL của `order-service`.
[EXP]
Sai. Lỗi cơ sở dữ liệu sẽ sinh ra ngoại lệ `SQLException` và được ghi ở mức `ERROR`.

@correct: B
@point: 20
