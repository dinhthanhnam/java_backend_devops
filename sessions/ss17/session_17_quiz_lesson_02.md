# QUIZ LESSON 02: TRIỂN KHAI EFK STACK BẰNG DOCKER COMPOSE

## Q1

Trước khi khởi động container Elasticsearch trên máy chủ Ubuntu Server, tham số cấu hình nhân Linux (Kernel) nào bắt buộc phải được thiết lập giá trị tối thiểu là `262144`?

[A]
`vm.max_map_count`
[EXP]
Đúng. Elasticsearch sử dụng mmapfs để ánh xạ chỉ mục vào bộ nhớ ảo; tham số `vm.max_map_count` phải đạt tối thiểu 262144 để vượt qua bước kiểm tra bootstrap check.
[B]
`net.ipv4.ip_forward`
[EXP]
Sai. Tham số này liên quan đến việc chuyển tiếp gói tin mạng của hệ điều hành, không phải yêu cầu bộ nhớ của Elasticsearch.
[C]
`fs.file-max`
[EXP]
Sai. `fs.file-max` quy định số lượng file descriptor tối đa của toàn hệ thống, không phải tham số mmapfs chính của Elasticsearch.
[D]
`kernel.pid_max`
[EXP]
Sai. Tham số này quy định số lượng Process ID tối đa trên hệ thống Linux.

@correct: A
@point: 20

## Q2

Tại sao trong dự án QuickBite, ta bắt buộc phải xây dựng Custom Dockerfile cho Fluentd thay vì sử dụng trực tiếp image `fluent/fluentd` mặc định từ Docker Hub?

[A]
Vì image mặc định của Fluentd không hỗ trợ khởi chạy trên hệ điều hành Ubuntu Server.
[EXP]
Sai. Image mặc định chạy tốt trên mọi nền tảng hỗ trợ Docker.
[B]
Vì image mặc định tự động giới hạn số lượng dòng log tiếp nhận tối đa là 100 dòng mỗi phút.
[EXP]
Sai. Fluentd không có cơ chế giới hạn cứng số lượng log như vậy.
[C]
Vì image cơ bản chưa được cài sẵn plugin `fluent-plugin-elasticsearch` và cần thiết lập quyền sở hữu cho thư mục đệm `/fluentd/buffer`.
[EXP]
Đúng. Image gốc chỉ có các plugin cơ bản; ta cần cài plugin kết nối Elasticsearch bằng Ruby gem để chuyển tiếp dữ liệu log.
[D]
Vì Docker Compose chỉ cho phép liên kết các service khi chúng được build từ Dockerfile cục bộ.
[EXP]
Sai. Docker Compose hỗ trợ kéo trực tiếp image có sẵn từ Docker Hub qua từ khóa `image`.

@correct: C
@point: 20

## Q3

Trong cấu hình `docker-compose.yml` của cụm EFK, Named Volume `fluentd-buffer` được gắn vào đường dẫn `/fluentd/buffer` nhằm mục đích kỹ thuật gì?

[A]
Để lưu trữ tệp mã nguồn cấu hình `fluent.conf` khi container khởi động.
[EXP]
Sai. Tệp cấu hình `fluent.conf` được mount từ thư mục cục bộ của dự án vào `/fluentd/etc/fluent.conf`.
[B]
Để lưu trữ các tệp đệm log an toàn trên đĩa cứng, giúp Fluentd tự động retry và chống mất dữ liệu khi Elasticsearch tạm thời quá tải hoặc mất kết nối.
[EXP]
Đúng. Cơ chế file buffer kết hợp Named Volume đảm bảo dữ liệu log không bị xóa mất khi container khởi động lại hoặc khi Elasticsearch tạm ngưng.
[C]
Để tăng tốc độ nén dữ liệu log trước khi ghi vào kho lưu trữ của Kibana.
[EXP]
Sai. Thư mục buffer không dùng để gửi dữ liệu trực tiếp sang Kibana mà dùng để gửi sang Elasticsearch.
[D]
Để sao lưu tự động toàn bộ cơ sở dữ liệu MySQL của các microservices QuickBite.
[EXP]
Sai. Volume này chỉ phục vụ riêng cho cơ chế đệm dữ liệu log của tiến trình Fluentd.

@correct: B
@point: 20

## Q4

Phương thức nào dưới đây được khuyến nghị để truy cập an toàn vào giao diện web Kibana (cổng `5601`) trên máy chủ VPS Production?

[A]
Mở cổng `5601` trên Firewall của VPS và truy cập trực tiếp bằng IP Public qua giao thức HTTP.
[EXP]
Sai. Mở công khai cổng 5601 ra Internet mà không có bảo vệ sẽ dẫn tới nguy cơ bị kẻ xấu dò quét và thao túng hệ thống giám sát.
[B]
Truy cập qua mạng nội bộ Docker `quickbite-net` từ trình duyệt của máy cá nhân.
[EXP]
Sai. Trình duyệt máy cá nhân không nằm bên trong mạng ảo nội bộ Docker của máy chủ VPS nên không thể truy cập trực tiếp.
[C]
Đổi cổng mặc định của Kibana từ `5601` sang cổng `80` của dịch vụ web thông thường.
[EXP]
Sai. Đổi cổng không giải quyết được vấn đề xác thực và mã hóa dữ liệu trên đường truyền công cộng.
[D]
Bind cổng `5601` vào `127.0.0.1` trên máy chủ và thiết lập kết nối mã hóa qua SSH Tunnel (`ssh -L 5601:127.0.0.1:5601 ...`).
[EXP]
Đúng. SSH Tunnel mã hóa toàn bộ phiên làm việc và chỉ cho phép người có quyền truy cập SSH vào máy chủ mới xem được giao diện Kibana.

@correct: D
@point: 20

## Q5

Trong tệp cấu hình `fluent.conf`, khối lệnh `<match quickbite.**>` thiết lập hai thông số `logstash_format true` và `logstash_prefix quickbite` nhằm mục đích gì?

[A]
Yêu cầu Fluentd cài đặt thêm phần mềm Logstash vào bên trong container Elasticsearch.
[EXP]
Sai. Logstash không được cài đặt; đây chỉ là tên quy ước định dạng cấu trúc chỉ mục tương thích với chuẩn Logstash.
[B]
Tự động định dạng và tạo các chỉ mục lưu trữ trong Elasticsearch theo từng ngày với tên có tiền tố `quickbite-YYYY.MM.DD`.
[EXP]
Đúng. Cấu hình này giúp chia nhỏ index theo ngày (ví dụ `quickbite-2026.08.14`), thuận tiện cho việc quản lý vòng đời và xoay vòng dữ liệu log.
[C]
Đổi tên định danh của tất cả các microservices thành tên chung là `quickbite`.
[EXP]
Sai. Tên của từng service vẫn được giữ nguyên trong trường `service.name` của bản ghi JSON.
[D]
Chuyển hướng toàn bộ các log có mức `ERROR` sang một tệp tin văn bản riêng biệt trên host.
[EXP]
Sai. Cấu hình này áp dụng cho toàn bộ dữ liệu log được gửi sang Elasticsearch, không lọc riêng mức ERROR.

@correct: B
@point: 20
