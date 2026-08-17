# QUIZ LESSON 04: STRUCTURED LOGGING VỚI ĐỊNH DẠNG JSON

## Q1

Ưu thế vượt trội lớn nhất của Structured Logging (dạng JSON) so với Unstructured Logging (dạng văn bản thô) trong hệ thống microservices là gì?

[A]
Dữ liệu được tổ chức thành các trường khóa-giá trị cố định (`key-value`), giúp các công cụ tự động lập chỉ mục, tìm kiếm và lọc dữ liệu chính xác mà không cần dùng regex phức tạp.
[EXP]
Đúng. Structured logging cho phép máy tính và công cụ giám sát hiểu trực tiếp cấu trúc từng trường (`service.name`, `log.level`, `trace.id`) với hiệu năng cao.
[B]
Dạng JSON giúp dung lượng của tệp log giảm đi 90% so với log dạng văn bản truyền thống.
[EXP]
Sai. Thực tế JSON log thường có dung lượng lớn hơn text thô một chút do phải chứa thêm tên của các trường dữ liệu.
[C]
Dạng JSON tự động mã hóa toàn bộ dữ liệu log giúp ngăn chặn hoàn toàn việc rò rỉ mật khẩu người dùng.
[EXP]
Sai. JSON là định dạng cấu trúc dữ liệu thuần túy, không có khả năng tự động che giấu hoặc mã hóa thông tin nhạy cảm nếu lập trình viên ghi vào.
[D]
Dạng JSON cho phép ứng dụng Spring Boot không cần thông qua thư viện Logback mà ghi trực tiếp vào đĩa cứng.
[EXP]
Sai. Logback vẫn là engine sinh log và sử dụng JSON Encoder để định dạng dữ liệu đầu ra.

@correct: A
@point: 20

## Q2

Trong chuẩn Elastic Common Schema (ECS) được tạo bởi `logback-ecs-encoder`, trường `@timestamp` đại diện cho thông tin gì?

[A]
Thời điểm container Docker được tạo mới trên máy chủ VPS.
[EXP]
Sai. `@timestamp` không phản ánh thời gian tạo container mà là thời gian sinh sự kiện log.
[B]
Khoảng thời gian (tính bằng mili-giây) mà hàm xử lý Java đã thực thi.
[EXP]
Sai. Khoảng thời gian thực thi thường được lưu trong các trường đo lường (metrics) hoặc `event.duration`.
[C]
Mốc thời gian chính xác khi log event được sinh ra bên trong ứng dụng Spring Boot theo chuẩn quốc tế ISO-8601 UTC.
[EXP]
Đúng. `@timestamp` lưu thời điểm xuất hiện log event (ví dụ `2026-08-14T10:15:22.456Z`) để đồng bộ dòng thời gian trên toàn hệ thống.
[D]
Thời điểm hệ điều hành Linux thực hiện xoay vòng tệp tin log (Log Rotation).
[EXP]
Sai. Thời điểm xoay vòng tệp do daemon của hệ thống quản lý, không nằm trong trường `@timestamp` của ECS.

@correct: C
@point: 20

## Q3

Khi trích xuất JSON log bằng lệnh `docker compose logs` để xử lý qua công cụ phân tích `jq`, tại sao cờ `--no-log-prefix` lại mang tính bắt buộc?

[A]
Để yêu cầu Docker Compose nén dữ liệu log theo chuẩn GZIP trước khi gửi tới `jq`.
[EXP]
Sai. Cờ này không có chức năng nén dữ liệu.
[B]
Để loại bỏ tiền tố tên container ở đầu dòng (ví dụ `quickbite-order-1 | `), giúp mỗi dòng đầu ra là một JSON object hoàn chỉnh mà `jq` có thể parse được.
[EXP]
Đúng. Tiền tố mặc định của Compose làm hỏng cấu trúc chuỗi JSON, khiến `jq` báo lỗi `parse error` và không đọc được dữ liệu.
[C]
Để tăng tốc độ đọc log từ đĩa cứng lên gấp hai lần.
[EXP]
Sai. Cờ này chỉ định dạng lại chuỗi hiển thị đầu ra, không can thiệp vào tốc độ đọc đĩa.
[D]
Để tự động lọc bỏ toàn bộ các dòng log có mức độ nghiêm trọng thấp hơn `WARN`.
[EXP]
Sai. Cờ `--no-log-prefix` không lọc log theo level.

@correct: B
@point: 20

## Q4

Câu lệnh kết hợp nào dưới đây sử dụng `jq` để lọc chính xác tất cả các dòng log có mức `ERROR` từ container `quickbite-order`?

[A]
`docker compose logs quickbite-order | grep "ERROR" | jq .`
[EXP]
Sai. Lệnh này thiếu cờ `--no-log-prefix` khiến `jq` bị lỗi parse, và `grep` không phân biệt được chữ ERROR nằm ở message hay trường log.level.
[B]
`docker compose logs --no-log-prefix quickbite-order | jq -c 'filter(.level == "ERROR")'`
[EXP]
Sai. Trong cú pháp của `jq`, hàm lọc điều kiện là `select()`, và tên trường ECS chuẩn là `log.level` chứ không phải `level`.
[C]
`docker compose logs --tail 100 quickbite-order | jq 'find("log.level=ERROR")'`
[EXP]
Sai. `find` không phải là cú pháp lọc dữ liệu hợp lệ trong công cụ dòng lệnh `jq`.
[D]
`docker compose logs --no-log-prefix --tail 100 quickbite-order | jq -c 'select(.["log.level"] == "ERROR")'`
[EXP]
Đúng. Cờ `--no-log-prefix` tạo JSON hợp lệ, và `select(.["log.level"] == "ERROR")` lọc chính xác giá trị của trường mức độ nghiêm trọng.

@correct: D
@point: 20

## Q5

Một lập trình viên cho rằng vì JSON log hỗ trợ cấu trúc phong phú nên muốn đưa toàn bộ đối tượng `UserLoginRequest` vào log để tiện kiểm tra. Đánh giá nào dưới đây là ĐÚNG về mặt kỹ thuật và an toàn thông tin?

[A]
Không được phép, vì đối tượng chứa mật khẩu ở dạng plain text; việc chuyển sang định dạng JSON không làm dữ liệu nhạy cảm trở nên an toàn hơn.
[EXP]
Đúng. Mật khẩu và token tuyệt đối không được ghi ra log dưới bất kỳ định dạng nào (kể cả JSON có cấu trúc).
[B]
Hoàn toàn được khuyến khích, vì JSON log sẽ tự động mã hóa trường mật khẩu thành các dấu sao `***`.
[EXP]
Sai. JSON Encoder chỉ chuyển đổi thuộc tính Java thành chuỗi JSON thô, không tự động nhận biết để che giấu mật khẩu.
[C]
Được phép nếu sử dụng chuẩn ECS, vì chuẩn ECS chỉ lưu trữ các trường dữ liệu công khai.
[EXP]
Sai. Nếu đưa đối tượng chứa password vào thì ECS vẫn sẽ in ra bình thường nếu không được lập trình viên chủ động lọc bỏ.
[D]
Không được phép, vì dung lượng của đối tượng `UserLoginRequest` sẽ làm ứng dụng Java bị lỗi tràn bộ nhớ heap (OutOfMemoryError).
[EXP]
Sai. Dung lượng 1 object đăng nhập rất nhỏ không gây OutOfMemory, nhưng vi phạm nghiêm trọng về mặt bảo mật an toàn thông tin.

@correct: A
@point: 20
