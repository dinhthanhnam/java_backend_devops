Session 01    Tổng quan DevOps & Quy trình CI/CD    

Lesson 01    Tổng quan DevOps và hạn chế của triển khai thủ công
Lesson 02    Khái niệm CI/CD (quy trình build, test, deploy)
Lesson 03    Mô hình môi trường Dev, Staging và Production
Lesson 04    Kiến trúc triển khai hệ thống Microservices
Session 02    "Giới thiệu Docker"    

Lesson 01    Khái niệm container và so sánh Docker với máy ảo
Lesson 02    Docker image, container và registry
Lesson 03    Cài đặt Docker và kiểm tra môi trường
Lesson 04    Các lệnh Docker cơ bản trong vòng đời container
Lesson 05    Kiểm tra log và truy cập container (logs, exec)
Session 03   Hệ thống kiến thức Session 01, 02

Session 04    Docker Compose và Dockerfile    

Lesson 01    Dockerfile và cách đóng gói ứng dụng Spring Boot thực tế
Lesson 02    Docker Compose và khái niệm hệ thống nhiều container
Lesson 03    Cấu trúc file docker-compose.yml (services, image, build)
Lesson 04    Biến môi trường (environment) và cấu hình port
Lesson 05    Volume và network trong Docker Compose
Lesson 06    Quản lý vòng đời hệ thống với Docker Compose

Session 05    Multi-container System & API Gateway   

Lesson 01    Kiến trúc hệ thống nhiều service trong microservices
Lesson 02    Chạy Spring Boot service cùng database bằng Docker Compose
Lesson 03    Giao tiếp giữa các service qua Docker network
Lesson 04    Vai trò của API Gateway trong hệ thống microservices
Lesson 05    Định tuyến request với Spring Cloud Gateway
Lesson 06    Luồng request end-to-end (client → gateway → service)

Session 06    Hệ thống kiến thức Session 03,04        

Session 07    Tự động hóa biên dịch với GitLab CI    

Lesson 01    Tổng quan CI/CD với GitHub Actions và Kiến trúc Runner
Lesson 02    Xây dựng cấu trúc Workflow CI/CD cơ bản
Lesson 03    Cơ chế điều phối và phân tách Jobs trong luồng CI/CD
Lesson 04    Ứng dụng CI/CD: Biên dịch tự động Spring Boot với Gradle
Lesson 05    Chẩn đoán sự cố và quản trị log hệ thống CI/CD

Session 08    Đóng gói Docker Image & Đẩy lên Registry  

Lesson 01    Quy trình Build Docker image trong pipeline CI/CD
Lesson 02    Tối ưu hóa Dockerfile cho Production (Multi-stage build)
Lesson 03    Phiên bản hóa và đẩy Docker image lên Registry từ Local
Lesson 04    Sử dụng Docker Image từ Registry trong Luồng CI/CD
Lesson 05    Kịch bản Thực hành Tổng hợp với user-service

Session 09    Hệ thống kiến thức Session 07,08        

Session 10    Triển khai hệ thống lên VPS    

Lesson 01    Khởi tạo VPS và thiết lập kết nối cơ bản qua Bitvise Client
Lesson 02    Tạo User mới và bảo mật VPS bằng cơ chế khóa SSH (SSH Hardening)
Lesson 03    Cài đặt nhanh Docker & Docker Compose và thiết lập tường lửa UFW
Lesson 04    Đồng bộ dự án qua Git bằng SSH Key và vận hành cụm Microservices

Session 11    Reverse proxy với Nginx trong môi trường production    

Lesson 01    Khái niệm Reverse Proxy và vai trò của Nginx trong kiến trúc Microservices
Lesson 02    Cấu trúc tệp cấu hình Nginx và các khối khai báo chính
Lesson 03    Triển khai Nginx làm Reverse Proxy chuyển tiếp yêu cầu đến API Gateway
Lesson 04    Cấu hình tên miền (Domain) và mã hóa bảo mật SSL/HTTPS với Let's Encrypt

Session 12    Hệ thống kiến thức Session 10,11

Session 13    Giám sát hệ thống với Prometheus    

Lesson 01    Khái niệm giám sát hệ thống và giới thiệu Prometheus
Lesson 02    Tích hợp Spring Boot Actuator và cấu hình xuất chỉ số
Lesson 03    Triển khai Prometheus bằng Docker Compose
Lesson 04    Triển khai Node Exporter giám sát tài nguyên VPS

Session 14    Tạo dashboard với Grafana

Lesson 01    Grafana và vai trò trong giám sát hệ thống
Lesson 02    Kết nối Grafana với Prometheus
Lesson 03    Tạo dashboard giám sát VPS
Lesson 04    Tạo dashboard giám sát Spring Boot service
Lesson 05    Cấu hình cảnh báo cơ bản khi service gặp sự cố
Session 15    Hệ thống kiến thức Session 10,11        

Session 16    Logging trong Spring Boot môi trường production    

Lesson 01    Khác biệt logging giữa development và production
Lesson 02    Log level và quy ước ghi log backend
Lesson 03    Cấu hình Logback trong Spring Boot
Lesson 04    Structured logging với định dạng JSON
Lesson 05    MDC và traceId trong luồng request phân tán

Session 17    Logging tập trung với EFK Stack

Lesson 01    Logging tập trung trong hệ thống microservices
Lesson 02    Tổng quan Elasticsearch, Fluentd và Kibana
Lesson 03    Triển khai EFK Stack bằng Docker Compose
Lesson 04    Gửi log từ Spring Boot đến Fluentd
Lesson 05    Phân tích log trong Kibana
Lesson 06    Debug hệ thống phân tán bằng traceId
