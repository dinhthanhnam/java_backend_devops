# HƯỚNG DẪN CẤU TRÚC VÀ NỘI DUNG SLIDE (HSSF) - SESSION 13
## CHỦ ĐỀ: GIÁM SÁT HỆ THỐNG VỚI PROMETHEUS

Tài liệu này cung cấp thiết kế chi tiết từng slide theo chuẩn **HSSF (HTML Slide System Framework)** của Rikkei Education. Mỗi slide được định nghĩa rõ ràng về nhãn (`data-hssf-label`), cấu trúc layout component, nội dung hiển thị (tiếng Việt), các đoạn mã (code) và nội dung thuyết trình (Speaker Notes).

---

## I. BẢN ĐỒ SLIDE (SLIDE MAP)

| # | `data-hssf-label` | Chủ đề Slide (Focus) | HSSF Components chính |
|---|-------------------|----------------------|-----------------------|
| 1 | Title | Tiêu đề chính | `hssf-slide--title`, `hssf-title-block` |
| 2 | Agenda | Lộ trình buổi học | `hssf-header`, `hssf-agenda` |
| 3 | Sec-01 | Phân đoạn 01: Monitoring & Metrics | `hssf-slide--section`, `hssf-section-block` |
| 4 | Pain-Blackbox | Vấn đề "Hộp đen" trên Production | `hssf-compare`, `hssf-callout--danger` |
| 5 | Concept-Metrics | 4 loại Metrics cơ bản | `hssf-grid`, `hssf-card` |
| 6 | Pull-Push | So sánh Pull-based vs Push-based | `hssf-compare`, `hssf-list` |
| 7 | Diagram-Scrape | Sơ đồ kiến trúc Prometheus scraping | `hssf-flow`, `hssf-flow__node` |
| 8 | Sec-02 | Phân đoạn 02: Spring Boot Actuator | `hssf-slide--section`, `hssf-section-block` |
| 9 | Actuator-Role | Vai trò Actuator & Micrometer | `hssf-columns`, `hssf-defs`, `hssf-callout--info` |
| 10 | Code-Gradle | Cấu hình dependencies & application.yml | `hssf-columns`, `hssf-code` |
| 11 | Security-Endpoint | Nguyên tắc bảo mật endpoint Actuator | `hssf-compare`, `hssf-callout--danger` |
| 12 | Sec-03 | Phân đoạn 03: Triển khai Prometheus | `hssf-slide--section`, `hssf-section-block` |
| 13 | Config-Prom | Cấu trúc file prometheus.yml | `hssf-columns`, `hssf-code`, `hssf-defs` |
| 14 | Code-Compose | Docker Compose cho Prometheus | `hssf-code`, `hssf-callout--warning` |
| 15 | Sec-04 | Phân đoạn 04: Node Exporter | `hssf-slide--section`, `hssf-section-block` |
| 16 | What-NodeExp | Node Exporter là gì & cơ chế mount | `hssf-columns`, `hssf-list`, `hssf-callout--info` |
| 17 | Table-Compare | So sánh Node Exporter vs JVM Actuator | `hssf-heading`, `hssf-table--striped` |
| 18 | Code-NodeExp | Docker Compose & prometheus.yml cho Node Exporter | `hssf-columns`, `hssf-code` |
| 19 | Summary | Tổng kết bài học | `hssf-header`, `hssf-list` |
| 20 | End | Kết thúc slide | `hssf-brand-end` |

---

## II. CHI TIẾT CẤU TRÚC VÀ NỘI DUNG TỪNG SLIDE

### SLIDE 1: Title
* **HSSF Classes:** `hssf-slide hssf-slide--title`
* **Layout / Components:**
  * `hssf-title-block` có `hssf-accent--bar-left`
  * Eyebrow: `SESSION 13 · DEVOPS IN ACTION`
  * Title: `Giám sát hệ thống với Prometheus`
  * Meta: `Rikkei Academy · Bộ môn DevOps`
* **Speaker Notes:** Chào các bạn học viên. Hôm nay chúng ta sẽ bước sang Session 13, học cách xây dựng hệ thống giám sát tập trung bằng Prometheus cho cụm microservices QuickBite trên Production.

---

### SLIDE 2: Agenda
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Nội dung bài học", Subtitle: "4 phần trọng tâm giám sát hệ thống")
  * `hssf-agenda`:
    * Lộ trình 1: Khái niệm giám sát, Metrics & kiến trúc Pull-based của Prometheus
    * Lộ trình 2: Tích hợp Spring Boot Actuator xuất metrics cho Prometheus
    * Lộ trình 3: Triển khai Prometheus Server bằng Docker Compose
    * Lộ trình 4: Node Exporter — giám sát tài nguyên phần cứng VPS
* **Speaker Notes:** Đây là lộ trình 4 phần. Từ khái niệm Monitoring, đến tích hợp mã nguồn, triển khai máy chủ giám sát và cuối cùng giám sát hạ tầng vật lý VPS.

---

### SLIDE 3: Sec-01 (Divider)
* **HSSF Classes:** `hssf-slide hssf-slide--section`
* **Layout / Components:**
  * `hssf-section-block`
  * `hssf-footer--light`
  * Big Number: `01`
  * Title: `Monitoring, Metrics & Kiến trúc Prometheus`
* **Speaker Notes:** Phần 1: Tại sao cần giám sát, 4 loại metrics cơ bản và cơ chế Pull-based khiến Prometheus khác biệt.

---

### SLIDE 4: Pain-Blackbox (Vấn đề "Hộp đen")
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Production = Hộp đen?", Subtitle: "Không có metrics = không có tầm nhìn")
  * `hssf-compare` chia 2 cột:
    * Cột Trái (Cons — Không có giám sát):
      * CPU VPS quá tải? — Không biết.
      * RAM JVM cạn kiệt? — Không biết.
      * API nào bị request đổ dồn? — Không biết.
    * Cột Phải (Pros — Có Prometheus):
      * Metrics liên tục theo thời gian (Time-series).
      * Tự động phát hiện target DOWN.
      * Dashboard trực quan + cảnh báo.
  * `hssf-callout--danger` ở chân slide: "Vận hành mà không có Monitoring giống lái xe bịt mắt — chỉ biết sự cố khi đã va chạm."
* **Speaker Notes:** Khi QuickBite lên Production, hệ thống thành hộp đen. order-service phản hồi chậm giờ cao điểm nhưng không có số liệu, chúng ta chỉ đoán mò.

---

### SLIDE 5: Concept-Metrics (4 loại Metrics)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "4 loại Metrics cốt lõi", Subtitle: "Phân loại dữ liệu đo đạc trong Prometheus")
  * `hssf-grid--2` chứa 4 card `hssf-card--outline`:
    * Card 1: Icon `📈` | Title: Counter | Body: Chỉ tăng / reset 0. VD: `http_requests_total`
    * Card 2: Icon `🌡️` | Title: Gauge | Body: Tăng/giảm tự do. VD: `jvm_memory_used_bytes`
    * Card 3: Icon `📊` | Title: Histogram | Body: Phân phối buckets phía server. VD: Latency API
    * Card 4: Icon `🎯` | Title: Summary | Body: Tính quantile phía client. VD: p99 response time
* **Speaker Notes:** Prometheus phân loại dữ liệu số thành 4 nhóm. Counter chỉ tăng, Gauge tăng giảm tự do, Histogram đếm vào các khoảng, Summary tính phân vị trực tiếp.

---

### SLIDE 6: Pull-Push (So sánh cơ chế thu thập)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Pull-based vs Push-based")
  * `hssf-compare` chia 2 cột:
    * Cột Pros: `Pull-based (Prometheus)`
      * Server chủ động kéo dữ liệu định kỳ (HTTP GET).
      * Không bị DDoS từ client tải cao.
      * Dễ phát hiện target DOWN (kéo thất bại = sập).
      * Phù hợp long-running services.
    * Cột Cons: `Push-based (InfluxDB, Jaeger…)`
      * Client đẩy dữ liệu về Server.
      * Phù hợp short-lived jobs / Serverless.
      * Server dễ bị quá tải khi ngàn client gửi dồn.
* **Speaker Notes:** Prometheus nổi bật vì cơ chế Pull-based — server giám sát chủ động kéo dữ liệu, không bị phụ thuộc vào client. Phù hợp kiến trúc microservices chạy dài hạn.

---

### SLIDE 7: Diagram-Scrape (Kiến trúc scraping)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Kiến trúc Prometheus Scraping", Subtitle: "Luồng kéo metrics trên Production VPS")
  * `hssf-flow--row` với các node:
    * Node 1 (Primary): `Prometheus Server` (sub: Scrape mỗi 15s)
    * Edge (labeled: HTTP GET) →
    * Node 2 (Soft): `user-service:8081` (sub: /actuator/prometheus)
    * Node 3 (Soft): `order-service:8083` (sub: /actuator/prometheus)
    * Node 4 (Outline): `node-exporter:9100` (sub: /metrics)
  * `hssf-callout--info`: "Tất cả target nằm chung mạng Docker `quickbite-net`. Prometheus gọi qua DNS nội bộ."
* **Speaker Notes:** Prometheus Server định kỳ gửi HTTP GET kéo metrics từ các target. Nhờ chung Docker Network, Prometheus dùng DNS nội bộ gọi tên container thay vì IP cứng.

---

### SLIDE 8: Sec-02 (Divider)
* **HSSF Classes:** `hssf-slide hssf-slide--section`
* **Layout / Components:**
  * `hssf-section-block`
  * Big Number: `02`
  * Title: `Spring Boot Actuator & Micrometer`
* **Speaker Notes:** Phần 2: Cách tích hợp thư viện xuất metrics chuẩn Prometheus vào mã nguồn Spring Boot.

---

### SLIDE 9: Actuator-Role (Vai trò Actuator & Micrometer)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Spring Boot Actuator + Micrometer")
  * `hssf-columns--1-1`:
    * Cột 1: `hssf-defs`
      * Actuator: Module sẵn có cung cấp endpoint `/actuator/health`, `/actuator/metrics`
      * Micrometer: Lớp trung gian dịch metrics → định dạng Prometheus
      * Endpoint chuyên biệt: `/actuator/prometheus` → Plain text
    * Cột 2: `hssf-flow--col` minh họa luồng:
      * JVM Metrics → Actuator → Micrometer → `/actuator/prometheus` → Prometheus Server
  * `hssf-callout--info`: "Micrometer đóng vai trò như SLF4J cho logging — lớp trừu tượng đo đạc trung gian."
* **Speaker Notes:** Actuator thu thập thông số JVM, Micrometer chuyển đổi từ JSON phức tạp sang Plain text chuẩn Prometheus. Thiếu Micrometer, endpoint `/actuator/prometheus` sẽ không tồn tại.

---

### SLIDE 10: Code-Gradle (Cấu hình dependencies)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Cấu hình mã nguồn", Subtitle: "build.gradle & application.yml")
  * `hssf-columns--1-1`:
    * Cột 1: `hssf-code` (build.gradle / groovy)
      ```groovy
      dependencies {
          implementation 'org.springframework.boot:
              spring-boot-starter-actuator'
          implementation 'io.micrometer:
              micrometer-registry-prometheus'
      }
      ```
    * Cột 2: `hssf-code` (application.yml / yaml)
      ```yaml
      management:
        endpoints:
          web:
            exposure:
              include: health, prometheus
        metrics:
          tags:
            application: ${spring.application.name}
      ```
* **Speaker Notes:** Thêm 2 dependencies: Actuator để thu thập, Micrometer Prometheus Registry để dịch format. Trong application.yml chỉ mở 2 endpoint cần thiết và gắn tag application để phân biệt service.

---

### SLIDE 11: Security-Endpoint (Bảo mật Actuator)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Bảo mật Endpoint Actuator trên Production")
  * `hssf-compare`:
    * Cột Cons (SAI — Mở toang):
      * `include: "*"` → Expose hết endpoint ra Internet.
      * `/actuator/env` → Lộ mật khẩu DB, JWT Secret.
      * `/actuator/shutdown` → Tắt ứng dụng từ xa.
    * Cột Pros (ĐÚNG — Chọn lọc):
      * Chỉ mở `health, prometheus`.
      * Chặn port 8081/8083 bằng UFW.
      * Chỉ Prometheus container nội bộ mới kéo được.
  * `hssf-callout--danger`: "Expose `include: \"*\"` trên Production = Phơi bày toàn bộ sơ đồ kiến trúc cho hacker."
* **Speaker Notes:** Endpoint Actuator chứa thông tin nhạy cảm. Trên Production, chỉ mở health và prometheus, chặn cổng microservice từ Internet.

---

### SLIDE 12: Sec-03 (Divider)
* **HSSF Classes:** `hssf-slide hssf-slide--section`
* **Layout / Components:**
  * `hssf-section-block`
  * Big Number: `03`
  * Title: `Triển khai Prometheus bằng Docker Compose`
* **Speaker Notes:** Phần 3: Hands-on cấu hình prometheus.yml và docker-compose để chạy Prometheus trên VPS.

---

### SLIDE 13: Config-Prom (File prometheus.yml)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Cấu trúc prometheus.yml")
  * `hssf-columns--1-2`:
    * Cột 1: `hssf-defs` giải thích từ khóa:
      * `scrape_interval`: Chu kỳ kéo metrics (15s)
      * `job_name`: Tên nhóm target
      * `metrics_path`: Đường dẫn endpoint
      * `targets`: Danh sách DNS:Port
    * Cột 2: `hssf-code` (prometheus.yml / yaml)
      ```yaml
      global:
        scrape_interval: 15s

      scrape_configs:
        - job_name: 'quickbite-backend'
          metrics_path: '/actuator/prometheus'
          static_configs:
            - targets:
                - 'user-service:8081'
                - 'order-service:8083'
      ```
* **Speaker Notes:** File prometheus.yml là bản đồ chỉ đường. scrape_interval quyết định tần suất kéo, targets dùng tên DNS container thay vì IP cứng.

---

### SLIDE 14: Code-Compose (Docker Compose Prometheus)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Docker Compose cho Prometheus", Subtitle: "Volume + Network, không expose port 9090")
  * `hssf-code` (docker-compose.yml / yaml):
    ```yaml
    prometheus:
      image: prom/prometheus:v2.45.0
      container_name: quickbite-prometheus
      volumes:
        - ./prometheus/prometheus.yml:
            /etc/prometheus/prometheus.yml
        - prometheus-data:/prometheus
      command:
        - '--config.file=/etc/prometheus/prometheus.yml'
        - '--storage.tsdb.path=/prometheus'
      networks:
        - quickbite-net
      # KHÔNG expose 9090 ra ngoài host
    ```
  * `hssf-callout--warning`: "Giao diện web Prometheus KHÔNG có authentication. Expose port 9090 = bất kỳ ai cũng truy cập và xóa dữ liệu."
* **Speaker Notes:** Mount volume prometheus-data để bảo toàn dữ liệu TSDB khi container restart. Không map port 9090 ra host để bảo mật.

---

### SLIDE 15: Sec-04 (Divider)
* **HSSF Classes:** `hssf-slide hssf-slide--section`
* **Layout / Components:**
  * `hssf-section-block`
  * Big Number: `04`
  * Title: `Node Exporter — Giám sát phần cứng VPS`
* **Speaker Notes:** Phần 4: Node Exporter — agent đo đạc CPU, RAM, Disk, Network của hệ điều hành máy host.

---

### SLIDE 16: What-NodeExp (Vai trò & cơ chế mount)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Node Exporter là gì?", Subtitle: "Agent giám sát cấp hệ điều hành")
  * `hssf-columns--1-1`:
    * Cột 1: `hssf-list`
      * Agent chính thức của Prometheus cho Linux
      * Đo: CPU load, RAM, Disk I/O, Network
      * Lắng nghe cổng `9100`, endpoint `/metrics`
    * Cột 2: `hssf-list` (Cơ chế mount Read-Only)
      * `/proc → /host/proc:ro` — Thông tin tiến trình & RAM
      * `/sys → /host/sys:ro` — Thiết bị phần cứng kernel
      * `/ → /rootfs:ro` — Dung lượng ổ đĩa
  * `hssf-callout--info`: "Bắt buộc mount `:ro` (Read-Only). Container chỉ đọc thông số, không can thiệp hệ thống host."
* **Speaker Notes:** Container mặc định bị cô lập Namespace, không thấy tài nguyên host. Node Exporter giải quyết bằng cách mount /proc /sys ở chế độ chỉ đọc.

---

### SLIDE 17: Table-Compare (Node Exporter vs JVM Actuator)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-heading` (Kicker: "So sánh", Title: "Node Exporter vs Spring Boot Actuator")
  * `hssf-table--striped`:
    | Tiêu chí | Node Exporter | Actuator |
    | :--- | :--- | :--- |
    | Phạm vi | Hệ điều hành VPS | Máy ảo JVM |
    | Chỉ số | CPU, RAM vật lý, Disk, Network | Heap, GC Pauses, HTTP Requests |
    | Cài đặt | 1 instance / VPS | Tích hợp vào từng microservice |
    | Endpoint | `:9100/metrics` | `:PORT/actuator/prometheus` |
* **Speaker Notes:** Node Exporter là cái nhìn vĩ mô (hạ tầng), Actuator là cái nhìn vi mô (ứng dụng). Cần kết hợp cả hai để có bức tranh toàn cảnh.

---

### SLIDE 18: Code-NodeExp (Docker Compose Node Exporter)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Tích hợp Node Exporter")
  * `hssf-columns--1-1`:
    * Cột 1: `hssf-code` (docker-compose.yml / yaml)
      ```yaml
      node-exporter:
        image: prom/node-exporter:v1.6.0
        volumes:
          - /proc:/host/proc:ro
          - /sys:/host/sys:ro
          - /:/rootfs:ro
        command:
          - '--path.procfs=/host/proc'
          - '--path.sysfs=/host/sys'
        networks:
          - quickbite-net
      ```
    * Cột 2: `hssf-code` (prometheus.yml / yaml)
      ```yaml
      - job_name: 'vps-hardware'
        metrics_path: '/metrics'
        static_configs:
          - targets:
              - 'node-exporter:9100'
      ```
* **Speaker Notes:** Thêm service node-exporter vào docker-compose, mount 3 thư mục hệ thống Read-Only. Bổ sung job mới trong prometheus.yml để kéo metrics phần cứng.

---

### SLIDE 19: Summary
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Tổng kết Session 13", Subtitle: "Hệ thống giám sát Prometheus hoàn chỉnh")
  * `hssf-list`:
    * Prometheus Pull-based — chủ động kéo metrics, phát hiện target DOWN.
    * Actuator + Micrometer → endpoint `/actuator/prometheus` chuẩn hóa.
    * prometheus.yml dùng DNS nội bộ Docker, không dùng IP cứng.
    * Node Exporter mount Read-Only — giám sát CPU/RAM/Disk VPS.
    * Không expose port 9090 / không mở `include: "*"` trên Production.
* **Speaker Notes:** Tóm lại, chúng ta đã xây dựng hệ thống giám sát hoàn chỉnh: Prometheus kéo metrics từ Actuator và Node Exporter, tất cả cô lập trong Docker Network.

---

### SLIDE 20: End
* **HSSF Classes:** `hssf-slide hssf-slide--section`
* **Layout / Components:**
  * `hssf-brand-end`
    * Kicker: `DEVOPS IN ACTION`
    * Title: `HỌC VIỆN ĐÀO TẠO LẬP TRÌNH CHẤT LƯỢNG NHẬT BẢN`
    * Org: `Rikkei Education`
  * `hssf-footer--light hssf-footer--nopage`
* **Speaker Notes:** Cảm ơn các bạn. Bài tiếp theo chúng ta sẽ học cách trực quan hóa toàn bộ metrics này bằng Grafana Dashboard.
