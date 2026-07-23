# QUIZ LESSON 05: CẤU HÌNH CẢNH BÁO CƠ BẢN KHI SERVICE GẶP SỰ CỐ

## Q1

Trong cơ chế Alerting của Grafana, thuật ngữ Contact Point đại diện cho thành phần cấu hình nào?

[A]
Công thức toán học PromQL dùng để tính toán ngưỡng cảnh báo lỗi.
[EXP]
Đây là Alert Rule, không phải Contact Point.
[B]
Kênh liên lạc nhận thông tin cảnh báo sự cố (ví dụ: Telegram Webhook, Slack Channel, Email).
[EXP]
Đúng. Contact Point quy định nơi gửi tin nhắn thông báo khi có sự cố xảy ra.
[C]
Endpoint HTTP phơi bày metrics thô của Spring Boot Actuator.
[EXP]
Đây là Target xuất metrics, không phải nơi nhận tin nhắn cảnh báo.
[D]
Kho lưu trữ dữ liệu chuỗi thời gian TSDB của Prometheus Server.
[EXP]
Đây là cơ sở dữ liệu lưu trữ metrics, không liên quan đến cấu hình gửi tin nhắn.

@correct: B
@point: 20

## Q2

Trạng thái Pending của một Alert Rule trong Grafana biểu thị ý nghĩa kỹ thuật nào?

[A]
Chỉ số đã vượt ngưỡng cảnh báo, hệ thống đang chờ hết thời gian trễ cấu hình trước khi chính thức phát tin nhắn cảnh báo.
[EXP]
Đúng. Trạng thái Pending xuất hiện khi điều kiện lỗi được thỏa mãn nhưng chưa duy trì đủ lâu (Pending duration) để chuyển sang trạng thái Firing chính thức.
[B]
Alert Rule đang gặp lỗi cú pháp PromQL nên không thể tiến hành đánh giá dữ liệu.
[EXP]
Nếu lỗi cú pháp, Alert Rule sẽ chuyển sang trạng thái Error chứ không phải Pending.
[C]
Hệ thống đã gửi thành công tin nhắn cảnh báo sự cố đến kênh Telegram của DevOps.
[EXP]
Gửi tin nhắn thành công xảy ra khi Alert Rule ở trạng thái Firing.
[D]
Ứng dụng microservice đích đã hoạt động bình thường trở lại và lỗi đã được tự động khắc phục.
[EXP]
Khi hệ thống bình thường trở lại, Alert Rule sẽ chuyển về trạng thái Normal/Resolved.

@correct: A
@point: 20

## Q3

Tại sao các kỹ sư DevOps được khuyên tuyệt đối không đặt thời gian trễ (Pending duration) quá ngắn (ví dụ dưới 10 giây) cho các Alert Rule đo lường CPU Usage?

[A]
Vì làm như vậy sẽ khiến Prometheus Server không kịp kéo dữ liệu metrics.
[EXP]
Pending duration ở Grafana không ảnh hưởng đến chu kỳ kéo metrics của Prometheus Server.
[B]
Để tránh các cảnh báo giả (Flapping Alerts) khi CPU chỉ bị tăng đột biến trong vài giây ngắn ngủi do các tiến trình khởi động hoặc nạp cấu hình tức thời.
[EXP]
Đúng. Đặt thời gian trễ giúp lọc bỏ các cảnh báo nhiễu khi hệ thống tự cân bằng lại tải trong thời gian ngắn.
[C]
Để giảm bớt dung lượng bộ nhớ đệm RAM mà Grafana tiêu thụ khi gửi tin nhắn.
[EXP]
Thời gian trễ không liên quan đến việc tiết kiệm dung lượng RAM của Grafana.
[D]
Vì Telegram Webhook tự động khóa kết nối nếu nhận được cảnh báo có thời gian trễ dưới 10 giây.
[EXP]
Telegram API giới hạn tần suất gửi tin (Rate limit) chứ không khóa kết nối dựa trên cấu hình thời gian trễ của Grafana.

@correct: B
@point: 20

## Q4

Để cấu hình Alert Rule cảnh báo ngay lập tức khi dịch vụ order-service bị sập nguồn (không phản hồi kết nối), câu lệnh PromQL nào dưới đây là chuẩn xác nhất?

[A]
`up{application="order-service"} == 0`
[EXP]
Đúng. Chỉ số `up` trả về 1 khi target hoạt động bình thường và trả về 0 khi target ngắt kết nối hoặc sập nguồn.
[B]
`jvm_memory_used_bytes{application="order-service"} == 0`
[EXP]
Sai. Khi JVM cạn RAM và sập, metrics jvm sẽ biến mất hoàn toàn chứ không trả về giá trị bằng 0.
[C]
`http_server_requests_seconds_count == 0`
[EXP]
Sai. Chỉ số này chỉ đo số lượng request xử lý, không đại diện trực tiếp cho trạng thái sống/chết (UP/DOWN) của container.
[D]
`node_cpu_seconds_total{mode="idle"} == 100`
[EXP]
Sai. Đây là chỉ số đo thời gian CPU nhàn rỗi của máy VPS, không liên quan đến trạng thái sống chết của container order-service.

@correct: A
@point: 20

## Q5

Khi nhận được tin nhắn cảnh báo sự cố từ Grafana Alert gửi đến Telegram, thông tin nào dưới đây thường KHÔNG giúp kỹ sư DevOps nhanh chóng định vị lỗi?

[A]
Tên microservice đang gặp sự cố (nhãn application).
[EXP]
Đây là thông tin tối quan trọng để DevOps biết cụ thể container nào đang bị sập.
[B]
Ngưỡng giá trị vi phạm thực tế tại thời điểm cảnh báo kích hoạt.
[EXP]
Giúp DevOps đánh giá được mức độ nghiêm trọng của sự cố (ví dụ CPU đạt 98%).
[C]
Tổng dung lượng ổ cứng tối đa mà VPS có thể nâng cấp trong tương lai.
[EXP]
Đúng. Thông tin nâng cấp phần cứng trong tương lai không thuộc chỉ số thời gian thực và không giúp xử lý sự cố hiện tại.
[D]
Trạng thái của Alert Rule (Firing - đang bị lỗi hoặc Resolved - đã khắc phục xong).
[EXP]
Giúp DevOps nhận biết lỗi đang tiếp diễn hay hệ thống đã tự động phục hồi.

@correct: C
@point: 20
