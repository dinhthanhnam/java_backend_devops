# QUIZ LESSON 01: KHÁI NIỆM GIÁM SÁT HỆ THỐNG VÀ KIẾN TRÚC PROMETHEUS

## Q1

Trong mô hình giám sát hệ thống, loại chỉ số (Metric Type) nào dưới đây mô tả một giá trị số học chỉ có thể tăng dần theo thời gian hoặc reset về 0 khi ứng dụng khởi động lại?

[A]
Gauge
[EXP]
Gauge là thước đo giá trị tức thời, có thể tăng hoặc giảm tự do theo thời gian (ví dụ: dung lượng RAM sử dụng).
[B]
Counter
[EXP]
Counter là bộ đếm chỉ tăng dần hoặc reset về 0 khi restart (ví dụ: tổng số request HTTP đã xử lý). Đây là định nghĩa chuẩn của Counter.
[C]
Histogram
[EXP]
Histogram dùng để phân phối tần suất của dữ liệu vào các khoảng (buckets) được chia sẵn.
[D]
Summary
[EXP]
Summary dùng để tính toán phân vị trực tiếp ngay tại phía client trước khi trả về.

@correct: B
@point: 20

## Q2

Cơ chế thu thập dữ liệu mặc định của Prometheus hoạt động theo luồng thực thi nào dưới đây?

[A]
Các ứng dụng client chủ động gọi API POST để gửi (Push) metrics định kỳ về Prometheus Server
[EXP]
Đây là cơ chế Push-based của một số hệ thống truyền thống khác, không phải cơ chế mặc định của Prometheus.
[B]
Prometheus Server định kỳ gửi yêu cầu HTTP GET đến các target (Pull) để kéo metrics về lưu trữ
[EXP]
Prometheus hoạt động theo cơ chế Pull-based. Server sẽ chủ động gửi HTTP GET đến các target khai báo sẵn để kéo metrics về lưu trữ.
[C]
Client viết metrics ra một file log nội bộ, sau đó Prometheus Server SSH vào máy chủ để đọc tệp tin
[EXP]
Prometheus không đọc file log qua SSH. Nó kéo dữ liệu qua endpoint HTTP phơi bày từ target.
[D]
Prometheus Server liên kết trực tiếp với Database của ứng dụng để quét các bản ghi thay đổi
[EXP]
Prometheus không kết nối trực tiếp vào cơ sở dữ liệu nghiệp vụ của ứng dụng để quét dữ liệu.

@correct: B
@point: 20

## Q3

Giả sử ứng dụng Spring Boot của bạn có một endpoint phơi bày metrics dạng text sau. Prometheus sẽ đọc hiểu dòng dữ liệu này như thế nào?

```text
http_requests_total{method="POST",handler="/api/v1/orders"} 1250
```

[A]
Tạo ra một metric tên là `POST` có giá trị đếm là 1250 lần.
[EXP]
Tên metric ở đây là `http_requests_total`. `method` chỉ là một nhãn định danh (label) đi kèm.
[B]
Tăng giá trị của metric `http_requests_total` thêm 1250 đơn vị cho tất cả các endpoint.
[EXP]
Giá trị 1250 is giá trị tuyệt đối tại thời điểm scrape, không phải là số lượng đơn vị tăng thêm cho tất cả endpoint.
[C]
Ghi nhận metric `http_requests_total` có nhãn `method="POST"` và nhãn `handler="/api/v1/orders"` đạt giá trị 1250.
[EXP]
Đúng. Dòng dữ liệu biểu thị tên metric `http_requests_total` đi kèm các cặp nhãn định danh (labels) trong dấu ngoặc nhọn `{}` và giá trị đo đạc là 1250.
[D]
Báo lỗi cú pháp vì thiếu mốc thời gian timestamp đi kèm trực tiếp ở cuối dòng.
[EXP]
Prometheus tự động gán timestamp lúc scrape nếu dòng dữ liệu trả về không đính kèm timestamp cụ thể.

@correct: C
@point: 20

## Q4

Khi so sánh mô hình Pull-based của Prometheus với mô hình Push-based truyền thống trong kiến trúc microservices tải cao, nhận định nào dưới đây phản ánh đúng ưu điểm của Pull-based?

[A]
Pull-based giúp client không cần mở bất kỳ cổng HTTP nào, đảm bảo an toàn tuyệt đối.
[EXP]
Ngược lại, Pull-based yêu cầu các client/target phải mở một cổng HTTP để Prometheus Server có thể gọi vào kéo dữ liệu.
[B]
Pull-based giúp bảo vệ Server giám sát không bị quá tải (DDoS) do tần suất quét hoàn toàn do Server quyết định và cấu hình.
[EXP]
Đúng. Trong cơ chế Pull, Server chủ động kiểm soát tần suất cào dữ liệu (scrape interval), tránh tình trạng hàng ngàn container client đồng loạt dồn dập gửi dữ liệu làm sập Server giám sát.
[C]
Pull-based giúp giảm thiểu hoàn toàn tài nguyên CPU tiêu thụ trên máy chủ Prometheus Server.
[EXP]
Cơ chế Pull vẫn tiêu thụ tài nguyên của Prometheus Server để phân tích văn bản thô nhận về từ các target.
[D]
Pull-based là giải pháp tối ưu nhất cho các tác vụ chạy ngắn hạn (Short-lived jobs/Serverless).
[EXP]
Pull-based không tối ưu cho short-lived jobs vì tiến trình có thể bật lên và tắt đi trước khi Prometheus kịp quét (cần Pushgateway bổ trợ).

@correct: B
@point: 20

## Q5

Để giám sát thời gian phản hồi (Latency) của API thanh toán đơn hàng chạy trên cluster gồm nhiều container song song, việc lựa chọn loại metric nào dưới đây giúp tính toán chính xác phân vị (Quantile) như p99 và cho phép cộng gộp (Aggregate) dữ liệu từ nhiều container?

[A]
Summary, vì nó tính toán phân vị trực tiếp trên từng container client trước khi trả về.
[EXP]
Summary tính toán phân vị ở phía client nên không thể cộng gộp (aggregate) các phân vị này lại với nhau một cách chính xác về mặt toán học khi chạy song song nhiều instance.
[B]
Counter, vì chúng ta chỉ cần đếm tổng thời gian phản hồi và chia cho tổng số request.
[EXP]
Counter đơn thuần không tính được phân vị (như 99% request có thời gian xử lý dưới bao nhiêu giây).
[C]
Gauge, vì thời gian phản hồi API liên tục thay đổi tăng giảm theo thời gian.
[EXP]
Gauge chỉ cho biết thời điểm tức thời, không thể phân phối phân vị thời gian phản hồi một cách chính xác cho hàng triệu request.
[D]
Histogram, vì nó lưu trữ dữ liệu dưới dạng các khoảng thời gian (buckets) và đẩy việc tính phân vị về Prometheus Server xử lý.
[EXP]
Đúng. Histogram chỉ đếm số lượng phần tử rơi vào các buckets ở client. Prometheus Server sẽ nhận các số liệu thô này và thực hiện tính toán phân vị tổng hợp từ nhiều container chạy song song một cách chính xác.

@correct: D
@point: 20
