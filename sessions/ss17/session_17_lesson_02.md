# SESSION 17: LOGGING TẬP TRUNG VỚI EFK STACK

## LESSON 02: Triển khai EFK Stack bằng Docker Compose

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, học viên có khả năng:

* **Tổ chức và triển khai** được ngăn xếp EFK độc lập bằng Docker Compose trên máy chủ Ubuntu Server.
* **Cấu hình** được Elasticsearch, Fluentd và Kibana cùng tham gia vào Docker network `quickbite-net`.
* **Xây dựng** được Custom Dockerfile cho Fluentd để cài đặt plugin kết nối `fluent-plugin-elasticsearch`.
* **Thiết lập** hệ thống Persistent Volume cho dữ liệu index của Elasticsearch và bộ đệm (buffer) của Fluentd.
* **Kiểm tra trạng thái sẵn sàng (Healthcheck)** của các container và thiết lập kết nối an toàn tới Kibana qua SSH Tunnel.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ

Ba thành phần Elasticsearch, Fluentd và Kibana thuộc về lớp hạ tầng giám sát (Observability Infrastructure) và có vòng đời vận hành hoàn toàn độc lập với các microservices nghiệp vụ của QuickBite. Nếu gộp chung toàn bộ vào tệp `docker-compose.yml` tại thư mục gốc, tệp cấu hình sẽ bị phình to, phức tạp và gây khó khăn khi cần nâng cấp hoặc khởi động lại riêng cụm logging.

Tương tự như cách QuickBite đã tách riêng ngăn xếp Prometheus/Grafana, cụm EFK được tổ chức thành một Docker Compose project độc lập trong thư mục `efk/`:

```text
~/quickbite/
├── docker-compose.yml             # Quản lý 4 microservices nghiệp vụ
├── prometheus_grafana/            # Quản lý hạ tầng Metrics
└── efk/                           # Quản lý hạ tầng Logging tập trung
    ├── docker-compose.yml
    └── fluentd/
        ├── Dockerfile
        └── fluent.conf
```

Mặc dù nằm ở hai Compose project riêng biệt, các container vẫn có thể giao tiếp thông suốt với nhau thông qua mạng ngoài dùng chung `quickbite-net`.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI

#### 3.1 Yêu cầu cấu hình Kernel Host cho Elasticsearch

Elasticsearch sử dụng công nghệ `mmapfs` để ánh xạ các chỉ mục lưu trữ vào bộ nhớ ảo. Mặc định trên Ubuntu, giới hạn số lượng vùng ánh xạ `vm.max_map_count` quá thấp khiến Elasticsearch bị lỗi khi khởi động.

Thiết lập cấu hình vĩnh viễn trên Ubuntu host:

```bash
# Ghi cấu hình kernel và áp dụng ngay lập tức
echo 'vm.max_map_count=262144' | sudo tee /etc/sysctl.d/99-elasticsearch.conf
sudo sysctl --system

# Xác nhận giá trị đã được cập nhật thành công
sysctl vm.max_map_count
```

*Kết quả đầu ra hợp lệ:*
```text
vm.max_map_count = 262144
```

#### 3.2 Khóa phiên bản và Thiết lập Persistent Volumes

* **Đồng bộ phiên bản:** Elasticsearch và Kibana phải sử dụng cùng phiên bản. Bài lab này cố định ở `8.15.0` để các thành phần tương thích và tái lập được; đây không phải khẳng định về phiên bản mới nhất.
* **Lưu trữ dữ liệu bền vững (Persistent Volumes):**

| Tên Volume | Vị trí gắn trong Container | Mục đích kỹ thuật |
| :--- | :--- | :--- |
| `elasticsearch-data` | `/usr/share/elasticsearch/data` | Lưu trữ toàn bộ chỉ mục log; bảo toàn dữ liệu khi container bị xóa hoặc nâng cấp. |
| `fluentd-buffer` | `/fluentd/buffer` | Lưu trữ các tệp đệm log tạm thời trên đĩa khi Elasticsearch gặp sự cố hoặc quá tải. |

#### 3.3 Xây dựng Custom Fluentd Image

Image Fluentd chính thức trên Docker Hub chỉ chứa các plugin cơ bản. Để gửi dữ liệu sang Elasticsearch, ta cần tạo một Custom Dockerfile dựa trên phiên bản Debian:

```dockerfile
# Sử dụng base image Debian chính thức hỗ trợ jemalloc tối ưu bộ nhớ
FROM fluent/fluentd:v1.19.3-debian-2.2

USER root
# Cài đặt plugin output Elasticsearch và chuẩn bị thư mục buffer
RUN gem install fluent-plugin-elasticsearch --no-document \
    && mkdir -p /fluentd/buffer \
    && chown -R fluent:fluent /fluentd
USER fluent
```

#### 3.4 Cấu hình chi tiết `fluent.conf`

Tạo tệp cấu hình `~/quickbite/efk/fluentd/fluent.conf`:

```conf
# 1. Tiếp nhận log từ Docker logging driver qua giao thức forward
<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>

# 2. Bóc tách trường 'log' từ chuỗi text sang các trường JSON độc lập
<filter quickbite.**>
  @type parser
  key_name log
  reserve_data true
  <parse>
    @type json
  </parse>
</filter>

# 3. Bổ sung các nhãn metadata môi trường chung
<filter quickbite.**>
  @type record_transformer
  <record>
    environment production
    collector fluentd
  </record>
</filter>

# 4. Điều hướng và đẩy dữ liệu vào Elasticsearch
<match quickbite.**>
  @type elasticsearch
  host elasticsearch
  port 9200
  logstash_format true
  logstash_prefix quickbite
  include_tag_key true
  tag_key fluentd_tag

  # Cấu hình bộ đệm dạng file chống mất dữ liệu khi mất kết nối
  <buffer>
    @type file
    path /fluentd/buffer/quickbite
    flush_mode interval
    flush_interval 5s
    retry_forever true
    chunk_limit_size 8m
    total_limit_size 512m
  </buffer>
</match>
```

#### 3.5 Tệp `docker-compose.yml` hoàn chỉnh cho cụm EFK

Tạo tệp `~/quickbite/efk/docker-compose.yml`:

```yaml
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.15.0
    container_name: quickbite-elasticsearch
    environment:
      discovery.type: single-node
      xpack.security.enabled: "false"
      xpack.license.self_generated.type: basic
      ES_JAVA_OPTS: "-Xms1g -Xmx1g"
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data
    ports:
      - "127.0.0.1:9200:9200"
    healthcheck:
      test: ["CMD-SHELL", "curl -fsS http://localhost:9200 >/dev/null"]
      interval: 10s
      timeout: 5s
      retries: 20
    networks:
      - quickbite-net

  fluentd:
    build:
      context: ./fluentd
    container_name: quickbite-fluentd
    volumes:
      - ./fluentd/fluent.conf:/fluentd/etc/fluent.conf:ro
      - fluentd-buffer:/fluentd/buffer
    environment:
      FLUENTD_CONF: fluent.conf
    ports:
      - "127.0.0.1:24224:24224"
      - "127.0.0.1:24224:24224/udp"
    depends_on:
      elasticsearch:
        condition: service_healthy
    networks:
      - quickbite-net

  kibana:
    image: docker.elastic.co/kibana/kibana:8.15.0
    container_name: quickbite-kibana
    environment:
      ELASTICSEARCH_HOSTS: '["http://elasticsearch:9200"]'
    ports:
      - "127.0.0.1:5601:5601"
    depends_on:
      elasticsearch:
        condition: service_healthy
    networks:
      - quickbite-net

volumes:
  elasticsearch-data:
  fluentd-buffer:

networks:
  quickbite-net:
    external: true
```

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (KHỞI ĐỘNG VÀ KIỂM TRA CỤM EFK)

#### 4.1 Tạo cấu trúc thư mục và khởi chạy Stack

1. Thiết lập cấu trúc thư mục làm việc:
   ```bash
   cd ~/quickbite
   mkdir -p efk/fluentd
   cd efk
   ```

2. Đặt 3 file cấu hình đã chuẩn bị vào đúng vị trí:
   - `docker-compose.yml` (tại thư mục `efk/`)
   - `fluentd/Dockerfile`
   - `fluentd/fluent.conf`

3. Tiến hành build image và khởi chạy toàn bộ cụm dịch vụ trong nền:
   ```bash
   docker compose up -d --build
   docker compose ps
   ```

*Kết quả đầu ra mong đợi:*
```text
NAME                     IMAGE                                    STATUS
quickbite-elasticsearch  elasticsearch:8.15.0                    running (healthy)
quickbite-fluentd        efk-fluentd                              running
quickbite-kibana         kibana:8.15.0                           running
```

#### 4.2 Kiểm tra trạng thái Elasticsearch API qua Loopback

Kiểm tra trực tiếp từ máy chủ Ubuntu để xác nhận Elasticsearch đã sẵn sàng:

```bash
# Kiểm tra thông tin chung của node
curl http://127.0.0.1:9200

# Kiểm tra trạng thái sức khỏe của cụm (Cluster Health)
curl http://127.0.0.1:9200/_cat/health?v
```

Cụm hoạt động bình thường khi cột `status` hiển thị trạng thái `green` hoặc `yellow` (trạng thái bình thường của mô hình single-node).

#### 4.3 Truy cập giao diện Kibana an toàn qua SSH Tunnel

Để bảo đảm an toàn dữ liệu, cổng `5601` của Kibana chỉ được bind vào `127.0.0.1` trên máy chủ. Kỹ sư vận hành truy cập từ máy cá nhân bằng cơ chế SSH Port Forwarding:

1. Trên máy tính cá nhân, mở terminal và chạy lệnh:
   ```bash
   ssh -L 5601:127.0.0.1:5601 <deployer>@<vps-public-ip>
   ```

2. Mở trình duyệt web và truy cập địa chỉ:
   ```text
   http://127.0.0.1:5601
   ```

Giao diện trang chủ của Kibana sẽ hiển thị sẵn sàng cho việc tạo Data View và phân tích dữ liệu log.

---

### PHẦN 5. TỔNG KẾT

* **Kiến trúc phân tách:** EFK được vận hành như một Compose project độc lập, giúp bảo trì linh hoạt và không làm ảnh hưởng đến mã nguồn microservices của QuickBite.
* **Persistent Volumes:** Bắt buộc sử dụng named volumes cho `elasticsearch-data` và `fluentd-buffer` để bảo toàn chỉ mục log và chống mất dữ liệu khi có sự cố mạng.
* **Cơ chế Buffer của Fluentd:** Lưu trữ đệm dạng file trên đĩa giúp Fluentd tự động lưu giữ và retry liên tục cho tới khi Elasticsearch phục hồi.
* **Bảo mật truy cập:** Tất cả các cổng quản trị (`9200`, `5601`) và cổng collector (`24224`) chỉ mở ở phạm vi loopback của máy chủ.

---

### PHẦN 6. CÂU HỎI TỰ LUẬN ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)

**Câu hỏi:** Tại sao `fluentd-buffer` cần được cấu hình dưới dạng Named Volume gắn với thư mục đĩa thay vì chỉ lưu tạm trên RAM của container?

**Gợi ý trả lời:**

Nếu lưu buffer trên RAM (memory buffer), khi container Fluentd bị khởi động lại hoặc máy chủ gặp sự cố mất điện, toàn bộ dữ liệu log đang chờ gửi sẽ bị xóa sạch. Cấu hình buffer dạng file (`@type file`) kết hợp với Named Volume giúp dữ liệu log được lưu an toàn xuống đĩa cứng vật lý của máy chủ VPS, cho phép Fluentd tiếp tục gửi lại toàn bộ dữ liệu ngay khi hệ thống hoạt động trở lại.

#### Câu 2 (Phân tích mạng)

**Câu hỏi:** Tại sao Fluentd trong tệp `fluent.conf` có thể kết nối với Elasticsearch bằng địa chỉ `host elasticsearch`, trong khi Docker Logging Driver trên host lại phải dùng `127.0.0.1:24224`?

**Gợi ý trả lời:**

* Fluentd và Elasticsearch đều là các container cùng tham gia mạng ngoài `quickbite-net`, do đó chúng có thể sử dụng Docker DNS nội bộ để phân giải tên service `elasticsearch`.
* Docker Logging Driver được thực thi bởi tiến trình `dockerd` trên hệ điều hành host bên ngoài container network, vì vậy nó bắt buộc phải kết nối tới cổng đã được publish trên địa chỉ loopback của máy chủ (`127.0.0.1:24224`).

#### Câu 3 (Xử lý sự cố)

**Câu hỏi:** Container `quickbite-elasticsearch` liên tục bị thoát (exited) ngay sau khi khởi động và log thông báo lỗi liên quan đến `max virtual memory areas vm.max_map_count [65530] is too low`. Hãy nêu nguyên nhân và cách khắc phục.

**Gợi ý trả lời:**

* **Nguyên nhân:** Nhân Linux (Kernel) của hệ điều hành host đang giới hạn số lượng vùng nhớ ảo tối đa ở mức mặc định (65530), không đáp ứng yêu cầu tối thiểu của Elasticsearch (262144).
* **Khắc phục:** Chạy lệnh `echo 'vm.max_map_count=262144' | sudo tee /etc/sysctl.d/99-elasticsearch.conf` và áp dụng bằng `sudo sysctl --system`, sau đó khởi động lại cụm EFK.

---

### PHẦN 7. BÀI TẬP THỰC HÀNH

1. Tạo thư mục `~/quickbite/efk` và khởi tạo đầy đủ 3 tệp cấu hình: `docker-compose.yml`, `fluentd/Dockerfile`, `fluentd/fluent.conf`.
2. Kiểm tra và xác nhận tham số `vm.max_map_count` trên máy chủ Ubuntu đạt giá trị tối thiểu `262144`.
3. Khởi chạy cụm EFK bằng Docker Compose và kiểm tra trạng thái sức khỏe của Elasticsearch đạt `healthy`.
4. Thiết lập SSH Tunnel từ máy cá nhân và truy cập thành công vào giao diện web của Kibana tại cổng `5601`.

---

### KẾT QUẢ MONG ĐỢI

Sau khi hoàn thành bài học, học viên làm chủ quy trình triển khai hoàn chỉnh cụm EFK Stack trên máy chủ Ubuntu bằng Docker Compose, thiết lập đầy đủ cơ chế lưu trữ bền vững và bảo đảm an toàn truy cập cho hệ thống giám sát.
