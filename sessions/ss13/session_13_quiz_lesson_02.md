# QUIZ LESSON 02: TÍCH HỢP SPRING BOOT ACTUATOR VÀ CẤU HÌNH XUẤT CHỈ SỐ

## Q1

Thư viện nào dưới đây bắt buộc phải bổ sung kèm theo Spring Boot Actuator để xuất metrics tương thích với cú pháp của Prometheus?

[A]
`implementation 'io.micrometer:micrometer-registry-prometheus'`
[EXP]
Đúng. Thư viện này đóng vai trò chuyển dịch dữ liệu JSON phức tạp của Actuator sang văn bản thô theo chuẩn cú pháp dòng lệnh của Prometheus.
[B]
`implementation 'org.springframework.boot:spring-boot-starter-web'`
[EXP]
Đây là thư viện Spring Web để tạo RESTful API, không liên quan đến việc định dạng lại metrics cho Prometheus.
[C]
`implementation 'org.springframework.boot:spring-boot-starter-security'`
[EXP]
Đây là thư viện bảo mật Spring Security, dùng để bảo vệ ứng dụng chứ không định dạng metrics.
[D]
`implementation 'io.prometheus:simpleclient_hotspot'`
[EXP]
Thư viện này thuộc Prometheus client Java thuần, không tích hợp tự động vào cơ chế Micrometer của Spring Boot.

@correct: A
@point: 20

## Q2

Sau khi tích hợp Actuator và Micrometer Prometheus Registry thành công, endpoint HTTP mặc định nào sẽ chứa dữ liệu văn bản thô để Prometheus Server gọi vào kéo dữ liệu?

[A]
`/actuator/metrics`
[EXP]
Endpoint này chứa danh sách các chỉ số thô định dạng JSON của Spring Boot mà Prometheus không đọc hiểu trực tiếp được.
[B]
`/actuator/prometheus`
[EXP]
Đúng. Đây là endpoint chuyên biệt chứa dữ liệu text thô tương thích hoàn toàn với Prometheus Server.
[C]
`/prometheus/metrics`
[EXP]
Đường dẫn này không chính xác theo cấu trúc mặc định của Spring Boot Actuator.
[D]
`/actuator/health`
[EXP]
Endpoint này chỉ hiển thị trạng thái sức khỏe tổng quát (UP/DOWN) của ứng dụng, không chứa dữ liệu metrics chi tiết.

@correct: B
@point: 20

## Q3

Khi đọc tệp cấu hình `application.yml` dưới đây của một microservice, nhãn (label) chung nào sẽ tự động được gán vào toàn bộ các metrics do ứng dụng sinh ra?

```yaml
management:
  metrics:
    tags:
      application: ${spring.application.name}
```

[A]
Nhãn `spring.application.name` có giá trị là `application`
[EXP]
Không đúng. Tên nhãn là `application`, còn giá trị của nó sẽ được phân giải từ biến môi trường `${spring.application.name}`.
[B]
Nhãn `application` có giá trị là tên của microservice đó
[EXP]
Đúng. Khai báo này đính kèm một cặp nhãn khóa-giá trị là `application="tên_dịch_vụ"` vào tất cả metrics giúp phân biệt nguồn gốc dữ liệu microservices.
[C]
Nhãn `management.metrics.tags` có giá trị là `application`
[EXP]
Không đúng. Đây là đường dẫn phân cấp trong file YAML, không phải tên nhãn của metric.
[D]
Nhãn `name` có giá trị là `spring`
[EXP]
Không đúng cấu trúc khai báo nhãn trong tệp application.yml.

@correct: B
@point: 20

## Q4

Tại sao việc gắn nhãn (label) định danh ứng dụng thông qua cấu hình `management.metrics.tags` lại cực kỳ quan trọng trong hệ thống microservices chạy song song nhiều service?

[A]
Để kích hoạt tính năng mã hóa dữ liệu trên đường truyền HTTP cho an toàn.
[EXP]
Nhãn metrics không hỗ trợ mã hóa dữ liệu.
[B]
Để các metrics trùng tên (như `jvm_memory_used_bytes`) từ các container khác nhau không bị gộp chung làm một, giúp vẽ biểu đồ riêng biệt cho từng service.
[EXP]
Đúng. Nhãn giúp phân tách rõ ràng metrics của các dịch vụ khác nhau (user-service, order-service...) khi Prometheus Server kéo dữ liệu về kho lưu trữ tập trung.
[C]
Để giảm thiểu dung lượng RAM tiêu thụ khi container Java khởi chạy.
[EXP]
Việc đính kèm nhãn thực chất làm tăng nhẹ dung lượng dữ liệu truyền tải, không giúp giảm dung lượng RAM.
[D]
Để Prometheus Server tự động phát hiện ra cổng chạy HTTP nội bộ của microservices.
[EXP]
Việc phát hiện cổng chạy do file cấu hình prometheus.yml quyết định, nhãn metrics không tự động điều khiển Prometheus.

@correct: B
@point: 20

## Q5

Một DevOps fresher cấu hình cho phép expose các endpoints trên môi trường Production như sau. Cấu hình này dẫn đến rủi ro bảo mật nghiêm trọng nào?

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

[A]
Làm chậm tốc độ phản hồi của toàn bộ các API nghiệp vụ của hệ thống QuickBite.
[EXP]
Việc mở các endpoint giám sát chỉ tốn tài nguyên khi có request truy cập vào chính nó, không trực tiếp làm chậm API nghiệp vụ.
[B]
Khiến ứng dụng Spring Boot không thể khởi động được và ném ra ngoại lệ NullPointerException.
[EXP]
Cú pháp này hoàn toàn hợp lệ, ứng dụng vẫn khởi động bình thường.
[C]
Bị rò rỉ dữ liệu nhạy cảm qua `/actuator/env` (mật khẩu DB, JWT secret) và hacker có thể tắt ứng dụng từ xa qua `/actuator/shutdown`.
[EXP]
Đúng. include: "*" mở toang tất cả các cổng nhạy cảm nội bộ ra ngoài Internet, tạo kịch bản rò rỉ thông tin cấu hình và phá hoại hệ thống từ xa.
[D]
Khiến cho Prometheus Server không thể quét được endpoint `/actuator/prometheus`.
[EXP]
Ngược lại, Prometheus vẫn quét được bình thường nhưng hệ thống phải đối mặt với rủi ro bảo mật nghiêm trọng.

@correct: C
@point: 20
