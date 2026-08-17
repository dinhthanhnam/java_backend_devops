# QUIZ LESSON 01: KHÁC BIỆT LOGGING GIỮA DEVELOPMENT VÀ PRODUCTION

## Q1

Sự khác biệt cốt lõi về mục đích sử dụng log giữa môi trường Development và môi trường Production là gì?

[A]
Development phục vụ debug mã nguồn và kiểm tra nhanh tính năng; Production phục vụ giám sát vận hành, điều tra sự cố và kiểm toán hệ thống.
[EXP]
Đúng. Ở Development lập trình viên cần xem chi tiết để sửa code; ở Production log trở thành dữ liệu vận hành để theo dõi sức khỏe hệ thống và xử lý lỗi người dùng thực.
[B]
Development chỉ ghi nhận các lỗi nghiêm trọng ERROR; Production chỉ ghi nhận các thông tin nghiệp vụ bình thường INFO.
[EXP]
Sai. Ngược lại, Development thường bật cả DEBUG và TRACE để kiểm tra chi tiết, còn Production duy trì mức cần thiết để tối ưu hiệu năng.
[C]
Development bắt buộc phải lưu trữ log vào cơ sở dữ liệu tập trung; Production chỉ cần hiển thị log tạm thời trên cửa sổ terminal.
[EXP]
Sai. Production mới là môi trường cần hệ thống lưu trữ tập trung lâu dài; Development thường chỉ cần quan sát trực tiếp trên terminal cục bộ.
[D]
Development ưu tiên định dạng JSON chuẩn hóa; Production ưu tiên định dạng văn bản tự do có màu sắc để người đọc quan sát nhanh.
[EXP]
Sai. Production yêu cầu định dạng có cấu trúc (JSON) để máy móc và công cụ tự động parse/index; Development mới chuộng text màu dễ đọc bằng mắt.

@correct: A
@point: 20

## Q2

Trong kiến trúc container hóa với Docker, tại sao ứng dụng Spring Boot nên ghi log ra `stdout` và `stderr` thay vì tự quản lý ghi vào file đĩa cục bộ bên trong container?

[A]
Vì ghi log ra file bên trong container sẽ khiến ứng dụng Java bị lỗi xung đột quyền truy cập hệ điều hành.
[EXP]
Sai. Ứng dụng Java hoàn toàn có thể ghi file nếu có quyền, nhưng đây không phải là nguyên nhân kiến trúc.
[B]
Vì ghi log ra `stdout`/`stderr` giúp tăng tốc độ xử lý các phép toán tính toán CPU bên trong ứng dụng Spring Boot.
[EXP]
Sai. Việc ghi log ra console không giúp tăng tốc độ xử lý CPU của các thuật toán nghiệp vụ.
[C]
Vì giúp tách biệt trách nhiệm: ứng dụng chỉ tạo log, còn Docker Engine và các công cụ thu gom sẽ đảm nhận việc lưu trữ và điều hướng.
[EXP]
Đúng. Ứng dụng không nên tự quản lý file log; Docker Engine sẽ tiếp nhận luồng `stdout`/`stderr` và chuyển tiếp cho các logging driver an toàn.
[D]
Vì Docker container không hỗ trợ tính năng tạo mới và lưu trữ tệp tin trên hệ thống tệp nội bộ.
[EXP]
Sai. Docker container có writable layer và hỗ trợ tạo file bình thường, nhưng dữ liệu sẽ mất khi container bị xóa nếu không gắn volume.

@correct: C
@point: 20

## Q3

Khi rà soát log của một microservice trên môi trường Production, dòng thông điệp log nào dưới đây vi phạm nghiêm trọng quy tắc an toàn thông tin?

[A]
`WARN [order-service] Order rejected: orderId=ORD-1024, reason=RESTAURANT_CLOSED`
[EXP]
Dòng log này an toàn vì chỉ chứa ID đơn hàng và lý do nghiệp vụ, không chứa thông tin nhạy cảm.
[B]
`INFO [user-service] Login success: username=nam, password=Rikkei@123, token=eyJhbGciOi...`
[EXP]
Đúng. Dòng log này vi phạm nghiêm trọng vì để lộ mật khẩu ở dạng plain text và chứa JWT access token bí mật của người dùng.
[C]
`INFO [order-service] Payment succeeded: orderId=ORD-1024, customerId=USR-500, amount=150000`
[EXP]
Dòng log này chuẩn hóa, chứa các định danh nghiệp vụ phục vụ việc tra cứu và đối soát dữ liệu giao dịch.
[D]
`ERROR [restaurant-service] Database connection timeout: host=db-restaurant, port=3306`
[EXP]
Dòng log này an toàn, chỉ chứa thông tin kỹ thuật hạ tầng phục vụ điều tra lỗi kết nối mạng.

@correct: B
@point: 20

## Q4

Kỹ sư vận hành muốn theo dõi các dòng log của container `quickbite-order` phát sinh trong 10 phút gần nhất kèm mốc thời gian chi tiết của từng dòng thì cần thực thi câu lệnh Docker Compose nào?

[A]
`docker compose logs -f --tail 10 quickbite-order`
[EXP]
Sai. Cờ `--tail 10` chỉ lấy 10 dòng log cuối cùng chứ không lọc theo khoảng thời gian 10 phút, và thiếu cờ `-t` để hiện timestamp.
[B]
`docker compose logs --time 10m quickbite-order`
[EXP]
Sai. Docker Compose không hỗ trợ cờ `--time`, cú pháp đúng để chỉ định mốc thời gian là `--since`.
[C]
`docker compose logs -t --last 10m quickbite-order`
[EXP]
Sai. Cờ `--last` không tồn tại trong tập lệnh của Docker Compose logs.
[D]
`docker compose logs -t --since 10m quickbite-order`
[EXP]
Đúng. Cờ `-t` (timestamps) hiển thị mốc thời gian chi tiết của từng sự kiện và `--since 10m` lọc các log sinh ra trong 10 phút qua.

@correct: D
@point: 20

## Q5

Tại sao việc chỉ xem log console cục bộ của một service đơn lẻ trên máy Development không đủ đáp ứng nhu cầu điều tra sự cố trên hệ thống QuickBite Production?

[A]
Vì Production gồm nhiều microservices chạy phân tán trên VPS, một request đi qua nhiều container độc lập nên cần liên kết và quan sát log đa dịch vụ.
[EXP]
Đúng. Khi đặt món lỗi, sự cố có thể phát sinh tại order-service, restaurant-service hoặc user-service; log trên máy local không phản ánh được dữ liệu trên VPS.
[B]
Vì các container chạy trên VPS bị khóa hoàn toàn tính năng xuất log ra màn hình console.
[EXP]
Sai. Container trên VPS vẫn xuất log ra console bình thường và được Docker Engine thu thập lại.
[C]
Vì môi trường Production bắt buộc phải chạy trực tiếp bằng lệnh `./gradlew bootRun` trên máy chủ.
[EXP]
Sai. Trên Production các service được đóng gói thành container Docker và điều khiển qua Docker Compose/Kubernetes.
[D]
Vì máy chủ VPS tự động mã hóa toàn bộ nội dung text của log khiến con người không thể đọc được.
[EXP]
Sai. Log trên VPS không bị mã hóa mặc định, nó ở dạng JSON hoặc plain text do ứng dụng cấu hình.

@correct: A
@point: 20
