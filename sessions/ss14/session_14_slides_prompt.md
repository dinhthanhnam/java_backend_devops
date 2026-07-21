# HƯỚNG DẪN CẤU TRÚC VÀ NỘI DUNG SLIDE (HSSF) - SESSION 14
## CHỦ ĐỀ: TẠO DASHBOARD VỚI GRAFANA

Tài liệu này cung cấp thiết kế chi tiết từng slide theo chuẩn **HSSF (HTML Slide System Framework)** của Rikkei Education. Mỗi slide được định nghĩa rõ ràng về nhãn (`data-hssf-label`), cấu trúc layout component, nội dung hiển thị (tiếng Việt), các đoạn mã (code) và nội dung thuyết trình (Speaker Notes).

---

## I. BẢN ĐỒ SLIDE (SLIDE MAP)

| # | `data-hssf-label` | Chủ đề Slide (Focus) | HSSF Components chính |
|---|-------------------|----------------------|-----------------------|
| 1 | Title | Tiêu đề chính | `hssf-slide--title`, `hssf-title-block` |
| 2 | Agenda | Lộ trình buổi học | `hssf-header`, `hssf-agenda` |
| 3 | Sec-01 | Phân đoạn 01: Giới thiệu Grafana | `hssf-slide--section`, `hssf-section-block` |
| 4 | Pain-Prometheus | Sự hạn chế của Prometheus UI | `hssf-compare`, `hssf-callout--danger` |
| 5 | Grafana-Concept | Grafana là gì & cơ chế bổ trợ | `hssf-grid`, `hssf-card` |
| 6 | Diagram-Architecture | Sơ đồ luồng dữ liệu 3 tầng | `hssf-columns--2`, `hssf-flow` |
| 7 | Code-Compose-Grafana | Cài đặt Grafana bằng Docker Compose | `hssf-code`, `hssf-callout--warning` |
| 8 | Sec-02 | Phân đoạn 02: Kết nối Prometheus | `hssf-slide--section`, `hssf-section-block` |
| 9 | Add-DataSource | Thêm Prometheus làm Data Source | `hssf-columns`, `hssf-steps`, `hssf-callout--info` |
| 10 | Docker-DNS-Data | Sử dụng DNS nội bộ thay vì localhost | `hssf-compare`, `hssf-callout--tip` |
| 11 | Sec-03 | Phân đoạn 03: Giám sát VPS & JVM | `hssf-slide--section`, `hssf-section-block` |
| 12 | Import-Dashboard | Cách import Dashboard cộng đồng | `hssf-steps`, `hssf-code` |
| 13 | Monitor-VPS | Dashboard giám sát VPS (ID 1860) | `hssf-columns`, `hssf-defs`, `hssf-callout--info` |
| 14 | Monitor-JVM | Dashboard giám sát JVM (ID 4701) | `hssf-columns`, `hssf-defs`, `hssf-callout--success` |
| 15 | Sec-04 | Phân đoạn 04: Cấu hình Alerting | `hssf-slide--section`, `hssf-section-block` |
| 16 | Alert-Flow | Luồng hoạt động của Alerting | `hssf-flow`, `hssf-flow__node` |
| 17 | Config-Alert | Thiết lập Alert Rule & Contact Point | `hssf-steps`, `hssf-list`, `hssf-callout--danger` |
| 18 | Summary | Tổng kết bài học | `hssf-header`, `hssf-list`, `hssf-stat` |
| 19 | End | Kết thúc slide | `hssf-brand-end` |

---

## II. CHI TIẾT CẤU TRÚC VÀ NỘI DUNG TỪNG SLIDE

### SLIDE 1: Title
* **HSSF Classes:** `hssf-slide hssf-slide--title`
* **Layout / Components:**
  * `hssf-title-block` có `hssf-accent--bar-left`
  * Eyebrow: `SESSION 14 · DEVOPS IN ACTION`
  * Title: `Tạo Dashboard Giám Sát với Grafana`
  * Meta: `Rikkei Academy · Bộ môn DevOps`
* **Speaker Notes:** Chào các bạn học viên. Hôm nay chúng ta sẽ tiếp tục Session 14, tìm hiểu cách kết nối Grafana với Prometheus để trực quan hóa dữ liệu hạ tầng VPS và ứng dụng Java Spring Boot thành các Dashboard chuyên nghiệp, đồng thời cấu hình cảnh báo sự cố tự động.

---

### SLIDE 2: Agenda
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Nội dung bài học", Subtitle: "4 phần trọng tâm về trực quan hóa dữ liệu")
  * `hssf-agenda`:
    * Lộ trình 1: Giới thiệu Grafana, vai trò và cài đặt bằng Docker Compose
    * Lộ trình 2: Kết nối Grafana với nguồn dữ liệu Prometheus (Data Source)
    * Lộ trình 3: Nhập (Import) các dashboard giám sát VPS và JVM Spring Boot
    * Lộ trình 4: Thiết lập cơ chế cảnh báo (Alerting) tự động khi có sự cố
* **Speaker Notes:** Đây là 4 nội dung chính. Chúng ta sẽ đi từ việc cài đặt cơ bản, liên kết cơ sở dữ liệu, dựng giao diện theo dõi sức khỏe hệ thống và thiết lập hệ thống tự động gửi tin nhắn báo lỗi.

---

### SLIDE 3: Sec-01 (Divider)
* **HSSF Classes:** `hssf-slide hssf-slide--section`
* **Layout / Components:**
  * `hssf-section-block`
  * `hssf-footer--light`
  * Big Number: `01`
  * Title: `Grafana và Vai trò Trực quan hóa Hệ thống`
* **Speaker Notes:** Phần 1: Tìm hiểu lý do tại sao Prometheus UI mặc định chưa đủ tốt và Grafana giúp chúng ta như thế nào.

---

### SLIDE 4: Pain-Prometheus (Hạn chế của Prometheus UI)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Hạn chế của Prometheus UI", Subtitle: "Tại sao cần thêm một tầng trực quan hóa?")
  * `hssf-compare` chia 2 cột:
    * Cột Trái (Prometheus UI thô sơ):
      * Chỉ hiển thị biểu đồ đơn lẻ cho từng câu lệnh truy vấn PromQL.
      * Không thể lưu trữ, gộp nhóm nhiều đồ thị thành một bảng điều khiển (Dashboard).
      * Không hỗ trợ phân quyền người dùng (RBAC), không có màn hình login.
    * Cột Phải (Grafana chuyên nghiệp):
      * Tổng hợp hàng chục biểu đồ trên một màn hình duy nhất.
      * Dashboard lưu trữ động, dễ dàng chia sẻ và xem thời gian thực.
      * Hệ thống bảo mật đăng nhập và phân quyền user cực kỳ chặt chẽ.
  * `hssf-callout--danger` ở chân slide: "Để lộ giao diện thô Prometheus ra ngoài là lỗ hổng bảo mật. Grafana cung cấp cổng bảo mật đăng nhập."
* **Speaker Notes:** Prometheus rất mạnh ở backend nhưng giao diện của nó chỉ để debug PromQL thô sơ. Để giám sát chuyên nghiệp, phân quyền an toàn, ta cần Grafana.

---

### SLIDE 5: Grafana-Concept (Khái niệm Grafana)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Grafana là gì?", Subtitle: "Tầng trực quan hóa dữ liệu hàng đầu")
  * `hssf-grid--2` chứa 4 card `hssf-card--outline`:
    * Card 1: Icon `🎨` | Title: Visualization | Body: Hỗ trợ đa dạng biểu đồ: Graph, Gauge, Bar chart, Heatmap.
    * Card 2: Icon `🔌` | Title: Data Sources | Body: Không tự lưu metrics. Kết nối trực tiếp tới Prometheus, InfluxDB, MySQL...
    * Card 3: Icon `👥` | Title: Phân quyền (RBAC) | Body: Hỗ trợ tạo tài khoản Admin, Editor, Viewer để giới hạn quyền truy cập.
    * Card 4: Icon `🔔` | Title: Cảnh báo (Alerting) | Body: Tự động gửi cảnh báo qua Telegram, Slack, Email khi metrics vượt ngưỡng.
* **Speaker Notes:** Grafana là bộ não hiển thị. Nó không chứa dữ liệu metrics, mà chỉ kéo dữ liệu từ các Data Source khác về vẽ đồ thị theo yêu cầu.

---

### SLIDE 6: Diagram-Architecture (Sơ đồ luồng dữ liệu)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Luồng Dữ liệu Giám sát", Subtitle: "Kiến trúc 3 tầng tiêu chuẩn")
  * `hssf-columns hssf-columns--2 hssf-columns--loose hssf-fill hssf-columns--center`:
    * Cột 1 (Luồng dọc): `hssf-flow hssf-flow--col`
      * Node 1 (Outline): `Targets (JVM / VPS)` (sub: Expose metrics)
      * Edge (labeled: HTTP GET / Pull) →
      * Node 2 (Soft): `Prometheus (Port 9090)` (sub: TSDB Backend)
      * Edge (labeled: PromQL Query) →
      * Node 3 (Primary): `Grafana (Port 3000)` (sub: Visualization Frontend)
    * Cột 2: `hssf-stack`
      * Khối giải thích:
        * 1. **Tầng Thu thập:** Spring Boot Actuator và Node Exporter hiển thị cổng metrics dạng text.
        * 2. **Tầng Lưu trữ:** Prometheus quét và ghi dữ liệu chuỗi thời gian vào đĩa cứng.
        * 3. **Tầng Hiển thị:** Grafana truy vấn PromQL liên tục để hiển thị lên màn hình DevOps.
* **Speaker Notes:** Luồng dữ liệu chạy từ Target qua cơ chế Pull của Prometheus, rồi từ Prometheus truyền lên Grafana thông qua các câu truy vấn PromQL.

---

### SLIDE 7: Code-Compose-Grafana (Docker Compose cài đặt)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Cấu hình Docker Compose cho Grafana", Subtitle: "Thêm dịch vụ vào file docker-compose.yml hạ tầng")
  * `hssf-code` (docker-compose.yml / yaml):
    ```yaml
    grafana:
      image: grafana/grafana:10.0.3
      container_name: quickbite-grafana
      ports:
        - "3000:3000"
      volumes:
        - grafana-data:/var/lib/grafana
      networks:
        - quickbite-net
      environment:
        - GF_SECURITY_ADMIN_PASSWORD=admin_secure_pass
      restart: unless-stopped
    ```
  * `hssf-callout--warning`: "Mount volume `grafana-data` là bắt buộc để lưu trữ tài khoản và thiết kế các Dashboard khi container restart."
* **Speaker Notes:** Khai báo dịch vụ Grafana chạy ở cổng 3000. Đặt mật khẩu admin mặc định qua biến môi trường và mount thư mục dữ liệu ra VPS host.

---

### SLIDE 8: Sec-02 (Divider)
* **HSSF Classes:** `hssf-slide hssf-slide--section`
* **Layout / Components:**
  * `hssf-section-block`
  * Big Number: `02`
  * Title: `Kết nối Grafana với Nguồn dữ liệu Prometheus`
* **Speaker Notes:** Phần 2: Cấu hình liên kết nguồn dữ liệu giữa Grafana và Prometheus bằng DNS mạng nội bộ Docker.

---

### SLIDE 9: Add-DataSource (Thêm Data Source)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Liên kết nguồn dữ liệu", Subtitle: "Thiết lập Prometheus Data Source")
  * `hssf-columns--1-1`:
    * Cột 1: `hssf-steps`
      * Step 1: Đăng nhập Grafana → Vào **Connections** → **Data sources**.
      * Step 2: Bấm **Add data source** → Chọn **Prometheus**.
      * Step 3: Nhập URL kết nối nội bộ của Prometheus.
      * Step 4: Bấm **Save & test** → Báo xanh *"Successfully queried..."* là thành công.
    * Cột 2: `hssf-stack`
      * Cấu hình URL kết nối chuẩn:
        * Tên URL: `http://quickbite-prometheus:9090`
      * `hssf-callout--info`: "Không cần mở cổng 9090 ra ngoài Internet. Grafana và Prometheus nói chuyện hoàn toàn trong mạng ảo `quickbite-net`."
* **Speaker Notes:** Để kết nối, ta vào cài đặt Data Source, chọn Prometheus và điền URL nội bộ của Docker Network.

---

### SLIDE 10: Docker-DNS-Data (Lưu ý về URL kết nối)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Tại sao không dùng localhost?", Subtitle: "Nguyên lý định tuyến mạng nội bộ Docker")
  * `hssf-compare`:
    * Cột Trái (Sai — Dùng localhost/127.0.0.1):
      * URL cấu hình: `http://127.0.0.1:9090`
      * Lỗi: Thất bại (Connection Refused).
      * Nguyên nhân: `127.0.0.1` bên trong container Grafana trỏ về chính container Grafana, chứ không phải máy host hay container Prometheus.
    * Cột Phải (Đúng — Dùng Docker DNS):
      * URL cấu hình: `http://quickbite-prometheus:9090`
      * Kết quả: Thành công.
      * Nguyên nhân: Docker Network tự động dịch chuyển tên container thành IP nội bộ chính xác.
  * `hssf-callout--tip` ở chân slide: "Hãy luôn dùng tên container (service name) để thiết lập kết nối giữa các dịch vụ chạy chung docker-compose."
* **Speaker Notes:** Đây là lỗi kinh điển của fresher. Cần hiểu rõ 127.0.0.1 trong container là cục bộ của container đó. Phải gọi bằng tên container Prometheus.

---

### SLIDE 11: Sec-03 (Divider)
* **HSSF Classes:** `hssf-slide hssf-slide--section`
* **Layout / Components:**
  * `hssf-section-block`
  * Big Number: `03`
  * Title: `Giám sát tài nguyên VPS & Ứng dụng JVM Spring Boot`
* **Speaker Notes:** Phần 3: Cách import các template Dashboard nổi tiếng để giám sát VPS và ứng dụng Spring Boot chỉ trong vài giây.

---

### SLIDE 12: Import-Dashboard (Cách Import Dashboard)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Tận dụng Dashboard cộng đồng", Subtitle: "Tiết kiệm thời gian thiết kế biểu đồ")
  * `hssf-steps`:
    * Step 1: Truy cập thư viện mã nguồn tại **grafana.com/dashboards**.
    * Step 2: Tìm kiếm và copy **ID** của dashboard cần sử dụng.
    * Step 3: Vào giao diện Grafana → Chọn **Dashboards** → Bấm **New** → Chọn **Import**.
    * Step 4: Dán ID vào ô → Chọn nguồn dữ liệu **Prometheus** → Bấm **Import**.
  * `hssf-code` (Cú pháp truy vấn PromQL mẫu để test ở Explore):
    ```promql
    # Kiểm tra tỷ lệ CPU sử dụng trung bình của VPS
    100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
    ```
* **Speaker Notes:** Thay vì tự vẽ tay từng biểu đồ, ta có thể import các dashboard mẫu từ cộng đồng bằng mã ID, sau đó chọn nguồn dữ liệu Prometheus là xong.

---

### SLIDE 13: Monitor-VPS (Giám sát VPS - ID 1860)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Dashboard Giám sát VPS", Subtitle: "Sử dụng mẫu Node Exporter Full - ID: 1860")
  * `hssf-columns--1-1`:
    * Cột 1: `hssf-defs` (Các chỉ số theo dõi cốt lõi)
      * CPU Usage: Tỷ lệ CPU đang xử lý của máy host
      * RAM Usage: Dung lượng bộ nhớ vật lý đã dùng
      * Disk Space: Dung lượng ổ cứng SSD còn trống
      * Network Traffic: Băng thông mạng tải lên/tải xuống
    * Cột 2: `hssf-stack`
      * Mô tả giao diện hiển thị:
        * Biểu đồ hình cung (Gauge) hiển thị phần trăm sử dụng.
        * Biểu đồ đường (Graph) theo dõi biến động tải theo thời gian thực.
  * `hssf-callout--info`: "Mẫu ID 1860 giúp DevOps phát hiện ngay khi VPS bị đầy đĩa SSD (Disk Full) hoặc nghẽn băng thông mạng."
* **Speaker Notes:** Dashboard Node Exporter Full có ID 1860 cho chúng ta toàn bộ thông số phần cứng của VPS. Giúp phát hiện nhanh các nguy cơ cạn kiệt RAM/Disk.

---

### SLIDE 14: Monitor-JVM (Giám sát JVM - ID 4701)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Dashboard Giám sát Java JVM", Subtitle: "Sử dụng mẫu JVM (Micrometer) - ID: 4701")
  * `hssf-columns--1-1`:
    * Cột 1: `hssf-defs` (Các chỉ số theo dõi Spring Boot)
      * JVM Heap: Vùng nhớ cấp phát động cho đối tượng Java
      * GC Rate & Time: Tần suất dọn rác bộ nhớ của Java
      * Active Threads: Số luồng đang thực thi đồng thời
      * HTTP Latency: Thời gian xử lý phản hồi các API
    * Cột 2: `hssf-stack`
      * Các điểm cần chú ý:
        * Lọc riêng biệt từng service qua nhãn `application`.
        * Phát hiện rò rỉ bộ nhớ (Memory Leak) qua xu hướng tăng Heap.
  * `hssf-callout--success`: "JVM dashboard là công cụ đắc lực để debug lỗi Out Of Memory (OOM) của Java Spring Boot trên Production."
* **Speaker Notes:** Dashboard JVM Micrometer ID 4701 giúp xem chi tiết bộ nhớ Heap và GC. Nếu thấy Heap liên tục tăng không giảm, đó là dấu hiệu rò rỉ bộ nhớ.

---

### SLIDE 15: Sec-04 (Divider)
* **HSSF Classes:** `hssf-slide hssf-slide--section`
* **Layout / Components:**
  * `hssf-section-block`
  * Big Number: `04`
  * Title: `Cấu hình Cảnh báo Tự động (Alerting)`
* **Speaker Notes:** Phần cuối: Học cách cấu hình Grafana tự động phát hiện lỗi và gửi tin nhắn cảnh báo qua Telegram/Slack.

---

### SLIDE 16: Alert-Flow (Luồng hoạt động Alerting)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Luồng hoạt động của Alerting", Subtitle: "Từ lúc xảy ra sự cố đến khi thông báo tới DevOps")
  * `hssf-flow hssf-flow--row hssf-flow--loose` (Bọc flex để căn chỉnh thẳng hàng):
    * Node 1: `1. Quét chỉ số` (sub: Prometheus thu thập CPU > 90%)
    * Edge →
    * Node 2 (Soft): `2. Đánh giá Rule` (sub: Grafana chạy query thấy vi phạm > 5 phút)
    * Edge →
    * Node 3 (Primary): `3. Trạng thái FIRING` (sub: Kích hoạt gửi tin nhắn cảnh báo)
    * Edge →
    * Node 4: `4. Nhận thông báo` (sub: Telegram/Slack của DevOps)
  * `hssf-callout--info`: "Alerting giúp DevOps không cần túc trực nhìn màn hình 24/7. Hệ thống sẽ chủ động kêu cứu khi gặp lỗi."
* **Speaker Notes:** Đây là luồng cảnh báo. Khi chỉ số vượt ngưỡng, Grafana chuyển rule sang trạng thái FIRING và bắn tin nhắn tới kênh liên lạc đã cấu hình.

---

### SLIDE 17: Config-Alert (Cấu hình Alert trên Grafana)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Thiết lập cảnh báo", Subtitle: "Từng bước cài đặt Alerting")
  * `hssf-columns--1-2`:
    * Cột 1: `hssf-steps`
      * Step 1: Tạo **Contact Point** (Liên kết webhook Telegram/Slack).
      * Step 2: Tạo **Alert Rule** dựa trên truy vấn PromQL.
      * Step 3: Định nghĩa ngưỡng (ví dụ: `up == 0` hoặc CPU > 90%).
      * Step 4: Cấu hình thời gian trễ (Pending duration: 2 phút).
    * Cột 2: `hssf-stack`
      * Cú pháp PromQL cảnh báo microservice sập:
        * `up{application="order-service"} == 0`
      * `hssf-callout--danger`: "Tránh đặt thời gian trễ quá ngắn để hạn chế cảnh báo giả (Flapping Alerts) khi hệ thống chỉ quá tải tức thời trong vài giây."
* **Speaker Notes:** Chúng ta tạo một Contact Point (ví dụ Webhook Telegram), viết câu lệnh check target `up == 0` (sập service) và đặt thời gian chờ 2 phút để tránh báo động giả.

---

### SLIDE 18: Summary
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Tổng kết Session 14", Subtitle: "Những lưu ý quan trọng khi triển khai Grafana")
  * `hssf-list`:
    * Grafana là tầng hiển thị (Visualization), kết nối Prometheus qua mạng nội bộ Docker.
    * URL kết nối bắt buộc dùng DNS: `http://quickbite-prometheus:9090`.
    * Mount volume `grafana-data` để giữ an toàn dữ liệu dashboards và tài khoản.
    * Sử dụng dashboard cộng đồng: ID 1860 (VPS) và ID 4701 (JVM).
    * Thiết lập Alerting qua Telegram/Slack để phản ứng nhanh khi có sự cố.
  * `hssf-cluster hssf-cluster--center`:
    <div class="hssf-stat">
      <span class="hssf-stat__value">3000</span>
      <span class="hssf-stat__label">Port mặc định</span>
    </div>
    <div class="hssf-stat">
      <span class="hssf-stat__value">1860 &amp; 4701</span>
      <span class="hssf-stat__label">Mã Dashboard phổ biến</span>
    </div>
* **Speaker Notes:** Tóm lại, Grafana hoàn thiện bức tranh giám sát. Nhớ mount volume, dùng DNS nội bộ và thiết lập cảnh báo tự động để tối ưu vận hành.

---

### SLIDE 19: End
* **HSSF Classes:** `hssf-slide hssf-slide--section`
* **Layout / Components:**
  * `hssf-brand-end`
    * Kicker: `DEVOPS IN ACTION`
    * Title: `Chúc các bạn thực hành thành công!`
    * Org: `Rikkei Academy - Rikkei Education`
  * `hssf-footer--light hssf-footer--nopage`
* **Speaker Notes:** Cảm ơn các bạn. Chúng ta bắt đầu phần bài tập Lab: cài đặt Grafana, kết nối Prometheus và import dashboard giám sát microservices.
