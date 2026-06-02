# QUICKBITE LEARNING EVOLUTION MAP

## DevOps Practical Curriculum State Machine

---

# 1. Mục tiêu tài liệu

Tài liệu này mô tả sự tiến hóa của hệ thống QuickBite trong suốt học phần DevOps.

Mỗi session không đứng độc lập, mà là một bước nâng cấp hệ thống.

Tài liệu này dùng để:

- Định nghĩa trạng thái hệ thống theo từng giai đoạn học

- Xác định đầu ra kỹ thuật sau mỗi session

- Đảm bảo tính liên tục trong quá trình giảng dạy

- Làm context chuẩn cho AI sinh nội dung bài học

---

# 2. Nguyên tắc hệ thống

- Hệ thống luôn bắt đầu từ local machine

- Mỗi session phải nâng cấp hệ thống lên một trạng thái rõ ràng

- Không có session nào chỉ mang tính lý thuyết thuần túy

- Tất cả ví dụ đều xoay quanh cùng một hệ thống QuickBite

- Không giới thiệu hệ thống mới trong quá trình học

---

# 3. Trạng thái tổng thể của hệ thống

Hệ thống QuickBite tiến hóa theo 8 trạng thái chính:

---

## STATE 0 – Local Application

Trạng thái ban đầu:

- Spring Boot services chạy trực tiếp trên máy local

- Postgresql local hoặc embedded

- Chưa có Docker

- Chưa có microservices thật sự

Mục tiêu:  
Hiểu từng service hoạt động độc lập như thế nào.

---

## STATE 1 – Containerization

- Đóng gói từng service bằng Docker

- Tạo Dockerfile cho Spring Boot application

- Run service trong container đơn lẻ

Kết quả:  
Mỗi service chạy độc lập trong container.

---

## STATE 2 – Multi-container System

- Sử dụng Docker Compose

- Chạy nhiều service cùng MySQL

- Thiết lập Docker network nội bộ

Kết quả:  
Hệ thống nhiều service chạy cùng nhau trên local.

---

## STATE 3 – Microservices Communication Layer

- Service-to-service communication

- Giới thiệu API Gateway

- Routing request qua gateway

Kết quả:  
Client không gọi trực tiếp service nữa.

---

## STATE 4 – CI/CD Automation

- GitLab CI pipeline

- Build & test application

- Docker image build automation

- Push image lên registry

Kết quả:  
Hệ thống build tự động hóa hoàn toàn.

---

## STATE 5 – Production Deployment

- Deploy toàn bộ hệ thống lên VPS

- Docker Compose production setup

- Environment configuration

Kết quả:  
Hệ thống chạy trên server thật.

---

## STATE 6 – Reverse Proxy Layer

- Nginx làm entry point

- Domain routing

- HTTPS configuration

Kết quả:  
Hệ thống có lớp truy cập production chuẩn.

---

## STATE 7 – Observability (Monitoring)

- Prometheus thu thập metrics

- Grafana dashboard

- Node Exporter cho VPS

Kết quả:  
Hệ thống có khả năng quan sát hiệu năng.

---

## STATE 8 – Logging & Debugging Layer

- Structured logging (JSON)

- MDC / traceId

- EFK stack (Elasticsearch, Fluentd, Kibana)

Kết quả:  
Có khả năng trace toàn bộ request end-to-end.

---

# 4. Mapping Session → System State

Session không độc lập, mà gắn với state transition:

| Session | Transition              |
| ------- | ----------------------- |
| S01     | Awareness (0)           |
| S02     | 0 → 1                   |
| S03     | Review                  |
| S04     | 1 → 2                   |
| S05     | 2 → 3                   |
| S07     | 3 → 4                   |
| S08     | 4                       |
| S10     | 4 → 5                   |
| S11     | 5 → 6                   |
| S13     | 6 → 7                   |
| S14     | 7                       |
| S16     | 7 → 8                   |
| S17     | 8                       |
| S18     | Full integration review |

---

# 5. Artifact Evolution Model

Mỗi state tạo ra artifact cụ thể:

---

## STATE 1

- Dockerfile per service

---

## STATE 2

- docker-compose.yml (multi-service)

---

## STATE 3

- Gateway config

- Service communication config

---

## STATE 4

- .gitlab-ci.yml pipeline

- Docker build automation

---

## STATE 5

- VPS deployment scripts

- production compose file

---

## STATE 6

- nginx.conf

---

## STATE 7

- prometheus.yml

- grafana dashboards

---

## STATE 8

- logback.xml (JSON logs)

- fluentd config

- kibana config

---

# 6. Quy tắc bắt buộc

- Không session nào được skip state

- Không introduce công nghệ ngoài state hiện tại

- Mỗi state phải có artifact rõ ràng

- Mọi ví dụ phải quay về QuickBite

- Không thay đổi system model trong quá trình học

---

# 7. Kết luận

QuickBite Learning Evolution Model biến toàn bộ học phần thành:

Một hệ thống đang được xây dựng, triển khai và vận hành theo từng giai đoạn.

Sinh viên không học công nghệ rời rạc.

Sinh viên đang “build một production system từng bước một”.
