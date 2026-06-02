# LEARNING ARCHITECTURE SPECIFICATION

## QuickBite – Food Delivery Microservices Platform

---

# 1. Mục tiêu của Learning Architecture

Learning Architecture Specification định nghĩa:

- Trạng thái kiến trúc hệ thống QuickBite theo từng giai đoạn học tập

- Các artifact kỹ thuật được tạo ra ở mỗi session

- Quan hệ phụ thuộc giữa các session

- Sự tiến hóa của hệ thống từ local → containerized → distributed → production → monitored → observable

Mục tiêu cốt lõi:

Sinh viên phải luôn trả lời được:

- Hệ thống hiện tại đang chạy như thế nào?

- Service nào đã được triển khai?

- Thành phần nào đang thiếu?

- Sau session tiếp theo hệ thống sẽ thay đổi ra sao?

---

# 2. Tổng quan kiến trúc mục tiêu (Final State)

Ở cuối khóa học, QuickBite phải đạt kiến trúc sau:

## 2.1 Logical Architecture

Client → Nginx Reverse Proxy → API Gateway → Microservices

Microservices:

- user-service

- restaurant-service

- order-service

- notification-service

Data layer:

- Posgresql (persistent volume)

Observability:

- Prometheus (metrics)

- Grafana (dashboard)

- EFK Stack (logs)

CI/CD:

- GitLab CI/CD

- Docker Registry

Deployment:

- VPS (Docker Compose)

---

## 2.2 Runtime Architecture

- Tất cả service chạy trong Docker containers

- Giao tiếp qua Docker network nội bộ

- Expose ra ngoài thông qua Nginx

- Gateway là entry point chính của hệ thống

---

# 3. Nguyên tắc tiến hóa hệ thống

Hệ thống không được xây dựng “một lần”, mà phải tiến hóa theo từng phase:

## Phase 1 – Local Development

- Chạy Spring Boot service trực tiếp

- Posgres local hoặc Docker đơn lẻ

- Chưa có gateway

- Chưa có CI/CD

Mục tiêu: hiểu service đơn lẻ

---

## Phase 2 – Containerization

- Mỗi service được Dockerize

- Postgres chạy bằng container

- Introduce Dockerfile

Mục tiêu: đóng gói ứng dụng

---

## Phase 3 – Multi-container System

- Docker Compose quản lý toàn bộ hệ thống

- Services giao tiếp qua network nội bộ

- Introduce service-to-service communication

Mục tiêu: hệ thống nhiều service

---

## Phase 4 – API Gateway Layer

- Thêm Spring Cloud Gateway

- Client không gọi trực tiếp service

- Routing theo path

Mục tiêu: kiến trúc microservices đúng nghĩa

---

## Phase 5 – CI/CD Automation

- GitLab CI build project

- Build Docker image

- Push image lên registry

- Pipeline theo stage

Mục tiêu: tự động hóa build & deploy artifact

---

## Phase 6 – Production Deployment

- Deploy toàn bộ hệ thống lên VPS

- Docker Compose production mode

- Environment separation

Mục tiêu: hệ thống chạy thật ngoài server

---

## Phase 7 – Reverse Proxy Layer

- Nginx làm reverse proxy

- Domain mapping

- HTTPS (Let’s Encrypt)

Mục tiêu: production-ready entry layer

---

## Phase 8 – Observability Layer

- Prometheus thu metrics

- Grafana dashboard

- Node Exporter cho VPS

Mục tiêu: quan sát hệ thống runtime

---

## Phase 9 – Logging & Debugging

- EFK Stack

- Centralized logging

- TraceId correlation across services

Mục tiêu: debug distributed system

---

# 4. Artifact Map theo Session

Mỗi session phải tạo ra ít nhất một artifact rõ ràng.

## 4.1 Artifact Definition

Artifact là một trong các dạng:

- Source code (Spring Boot service)

- Dockerfile

- docker-compose.yml

- CI pipeline (.gitlab-ci.yml)

- Nginx config

- Monitoring config (Prometheus/Grafana)

- Logging config (Logback/EFK)

- Deployment script (VPS setup)

---

## 4.2 Mapping ví dụ

### Session 02 – Docker Basics

Artifact:

- Dockerfile cho user-service

---

### Session 04 – Docker Compose

Artifact:

- docker-compose.yml (user-service + postgresql)

---

### Session 05 – Microservices Communication

Artifact:

- Updated compose file (multi-service)

- Network configuration

---

### Session 07 – CI/CD Introduction

Artifact:

- .gitlab-ci.yml (build stage)

---

### Session 08 – CI/CD Automation

Artifact:

- Full pipeline (build → test → dockerize → push)

---

### Session 10 – VPS Deployment

Artifact:

- VPS deployment guide

- production docker-compose.yml

---

### Session 11 – Nginx Reverse Proxy

Artifact:

- nginx.conf production config

---

### Session 13 – Prometheus

Artifact:

- prometheus.yml

- metrics endpoint config

---

### Session 14 – Grafana

Artifact:

- dashboard config

---

### Session 17 – Logging System

Artifact:

- logback.xml (JSON logs)

- fluentd config

- kibana index pattern

---

# 5. Dependency giữa các Session

Một số session phụ thuộc trực tiếp:

- Docker Compose phụ thuộc Docker Basics

- Gateway phụ thuộc Multi-container System

- CI/CD phụ thuộc Dockerization

- VPS Deployment phụ thuộc CI/CD

- Monitoring phụ thuộc VPS Deployment

- Logging phụ thuộc Distributed System hoàn chỉnh

Không được dạy Monitoring hoặc Logging trước khi hệ thống chạy được trên VPS.

---

# 6. Trạng thái hệ thống theo thời gian

## State 0 – Chưa có hệ thống

Chỉ có Spring Boot service chạy local.

---

## State 1 – Containerized Service

1 service chạy bằng Docker.

---

## State 2 – Multi-container Local System

Multiple services + database.

---

## State 3 – Microservices Architecture

Gateway + multiple services + network communication.

---

## State 4 – CI/CD Enabled System

Pipeline tự động build & push image.

---

## State 5 – Production Deployment

Hệ thống chạy trên VPS.

---

## State 6 – Observable System

Monitoring + Logging + Metrics đầy đủ.

---

# 7. Quy tắc bắt buộc cho toàn bộ Learning Design

- Không session nào được tồn tại độc lập

- Không được introduce công nghệ không phục vụ state hiện tại

- Mọi công nghệ phải gắn với một vấn đề cụ thể trong QuickBite

- Mỗi session phải nâng cấp hệ thống lên một state cao hơn

- Không có session nào chỉ mang tính “giới thiệu lý thuyết”

---

# 8. Kết luận

Learning Architecture của QuickBite không phải là tập hợp bài học.

Nó là một tiến trình:

Local System → Container → Microservices → CI/CD → Production → Observability

Mỗi session là một bước tiến hóa có thể đo được của hệ thống thực tế.
