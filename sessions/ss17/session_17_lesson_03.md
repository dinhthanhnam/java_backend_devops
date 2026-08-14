# SESSION 17: LOGGING TẬP TRUNG VỚI EFK STACK

## LESSON 03: Gửi log từ Spring Boot đến Fluentd

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, học viên có khả năng:

* **Cấu hình** được Docker Logging Driver `fluentd` cho từng microservice trong hệ thống QuickBite.
* **Giải thích** được cấu trúc gói tin do Docker Driver gửi tới Fluentd và cơ chế giải mã trường `log` chứa chuỗi JSON gốc.
* **Thiết lập** hệ thống định tuyến nhãn (Tag) chuẩn hóa để phân luồng và nhận diện log của từng service.
* **Kiểm tra và xác nhận** luồng dữ liệu log đi từ ứng dụng Spring Boot qua Fluentd và được ghi vào Elasticsearch index.
* **Nhận diện và xử lý** các lỗi phổ biến khi cấu hình kết nối giữa Docker Daemon và Fluentd Collector.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ

Ứng dụng Spring Boot trong QuickBite đã được cấu hình ghi JSON log chuẩn ECS ra luồng `stdout`. Nếu chỉ sử dụng logging driver mặc định của Docker (`json-file`), các dòng log này chỉ được ghi vào tệp đệm trên máy chủ host để phục vụ lệnh `docker compose logs`. Fluentd Collector hoàn toàn không tự động đọc được dữ liệu này nếu không có cơ chế chuyển giao trực tiếp.

Docker cung cấp sẵn logging driver chuyên dụng `fluentd`. Khi kích hoạt driver này, Docker Daemon sẽ tự động đóng gói mỗi sự kiện log xuất hiện tại `stdout`/`stderr` thành một thông điệp và chuyển tiếp tới Fluentd qua giao thức forward:

```json
{
  "container_id": "9a8b7c6d5e...",
  "container_name": "/quickbite-order",
  "source": "stdout",
  "tag": "quickbite.order",
  "log": "{\"@timestamp\":\"2026-08-14T10:15:22.456Z\",\"service.name\":\"order-service\",\"message\":\"Payment succeeded for Order ID 125\"}"
}
```

Vấn đề kỹ thuật đặt ra: Toàn bộ JSON log của Spring Boot đang bị bọc bên trong trường `log` dưới dạng một chuỗi văn bản (String). Do đó, Fluentd cần một bộ lọc phân tích (Parser Filter) để trích xuất chuỗi này thành các trường dữ liệu độc lập (`service.name`, `log.level`, `trace.id`), giúp Elasticsearch có thể lập chỉ mục và tìm kiếm chi tiết.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI

#### 3.1 Cấu hình Docker Logging Driver `fluentd` trong Docker Compose

Trong tệp `docker-compose.yml`, khối `logging` được định nghĩa ở cấp độ từng service:

```yaml
logging:
  driver: fluentd
  options:
    fluentd-address: "127.0.0.1:24224"
    fluentd-async: "true"
    tag: "quickbite.order"
```

##### Phân tích chi tiết các tham số tùy chọn:

| Tùy chọn (Option) | Giá trị cấu hình | Ý nghĩa và Vai trò vận hành |
| :--- | :--- | :--- |
| **`fluentd-address`** | `127.0.0.1:24224` | Địa chỉ IP loopback và cổng dịch vụ của Fluentd đã được publish trên máy chủ host. |
| **`fluentd-async`** | `"true"` | Chế độ gửi log bất đồng bộ. Cho phép container ứng dụng khởi động bình thường ngay cả khi Fluentd tạm thời chưa sẵn sàng; Docker daemon sẽ tự động đệm log trong bộ nhớ và kết nối lại trong nền. |
| **`tag`** | `quickbite.<service_name>` | Nhãn định danh nguồn phát log, được Fluentd sử dụng làm căn cứ định tuyến và lọc dữ liệu. |

> **Lưu ý vận hành:** Tùy chọn `fluentd-async: "true"` giúp hệ thống không bị nghẽn (non-blocking) khi khởi động, nhưng không thay thế cho việc duy trì sự ổn định của Fluentd. Nếu Fluentd bị sập quá lâu, bộ nhớ đệm của Docker daemon sẽ bị đầy và dẫn tới nguy cơ thất thoát dữ liệu log.

#### 3.2 Chuẩn hóa hệ thống Tag cho các Microservices QuickBite

Để Fluentd có thể tự động gom cụm và xử lý theo mẫu `<filter quickbite.**>`, các service cần tuân thủ quy ước đặt tag thống nhất:

| Microservice | Tên Container chuẩn | Tag cấu hình trong Compose |
| :--- | :--- | :--- |
| `order-service` | `quickbite-order` | `quickbite.order` |
| `restaurant-service` | `quickbite-restaurant` | `quickbite.restaurant` |
| `user-service` | `quickbite-user` | `quickbite.user` |
| `notification-service` | `quickbite-notification` | `quickbite.notification` |

#### 3.3 Cơ chế bóc tách JSON bằng Parser Filter trong Fluentd

Để chuyển đổi chuỗi JSON thô trong trường `log` thành các trường dữ liệu có cấu trúc, Fluentd sử dụng cấu hình parser:

```conf
<filter quickbite.**>
  @type parser
  key_name log
  reserve_data true
  <parse>
    @type json
  </parse>
</filter>
```

* **`key_name log`:** Chỉ định trường dữ liệu chứa chuỗi JSON cần giải mã.
* **`reserve_data true`:** Tham số tối quan trọng. Giữ nguyên toàn bộ các metadata do Docker bổ sung (`container_name`, `container_id`, `source`) và hợp nhất cùng với các trường ECS được giải mã từ Spring Boot (`@timestamp`, `service.name`, `trace.id`).

#### 3.4 Tính độc lập tuyệt đối của ứng dụng Spring Boot

Mô hình chuyển tiếp log qua Docker Logging Driver mang lại lợi thế kiến trúc vượt trội:

* Ứng dụng Spring Boot hoàn toàn không cần thêm bất kỳ thư viện Fluentd SDK nào trong `build.gradle`.
* Ứng dụng không cần biết địa chỉ IP, cổng hay cấu hình của Elasticsearch.
* Mã nguồn nghiệp vụ chỉ tương tác với chuẩn SLF4J; việc thu gom, chuyển đổi và lưu trữ được ủy quyền hoàn toàn cho lớp hạ tầng container.

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (KẾT NỐI QUICKBITE VÀO EFK)

Phần thực hành hướng dẫn cấu hình tích hợp logging driver cho cụm microservices QuickBite và kiểm tra luồng dữ liệu log trên Elasticsearch.

#### 4.1 Kiểm tra trạng thái sẵn sàng của Fluentd Collector

Trước khi áp dụng logging driver cho các microservices, Fluentd phải đang chạy và lắng nghe kết nối tại cổng `24224` trên host:

```bash
cd ~/quickbite/efk
docker compose ps
docker compose logs --tail 30 fluentd
```

Xác nhận: Container `quickbite-fluentd` ở trạng thái `running` và đã tải thành công cấu hình `forward` input.

#### 4.2 Cập nhật cấu hình Logging Driver trong `docker-compose.yml`

Mở tệp `~/quickbite/docker-compose.yml` và bổ sung khối `logging` cho từng microservice:

```yaml
# 1. Cấu hình cho quickbite-order
quickbite-order:
  # ... (giữ nguyên các cấu hình build, ports, environment hiện có)
  logging:
    driver: fluentd
    options:
      fluentd-address: "127.0.0.1:24224"
      fluentd-async: "true"
      tag: "quickbite.order"

# 2. Cấu hình cho quickbite-restaurant
quickbite-restaurant:
  logging:
    driver: fluentd
    options:
      fluentd-address: "127.0.0.1:24224"
      fluentd-async: "true"
      tag: "quickbite.restaurant"

# 3. Cấu hình cho quickbite-user
quickbite-user:
  logging:
    driver: fluentd
    options:
      fluentd-address: "127.0.0.1:24224"
      fluentd-async: "true"
      tag: "quickbite.user"

# 4. Cấu hình cho quickbite-notification
quickbite-notification:
  logging:
    driver: fluentd
    options:
      fluentd-address: "127.0.0.1:24224"
      fluentd-async: "true"
      tag: "quickbite.notification"
```

#### 4.3 Tái tạo Container để áp dụng Logging Driver mới

Logging driver là cấu hình tĩnh gắn liền với quá trình khởi tạo container. Do đó, cần sử dụng cờ `--force-recreate` để Docker tạo lại các container với driver mới:

```bash
cd ~/quickbite
docker compose up -d --force-recreate
docker compose ps
```

#### 4.4 Kiểm tra luồng dữ liệu tới Fluentd và Elasticsearch Index

1. Mở cửa sổ theo dõi log của Fluentd trong thời gian thực:
   ```bash
   cd ~/quickbite/efk
   docker compose logs -f fluentd
   ```

2. Gửi một request nghiệp vụ bất kỳ tới QuickBite API hoặc khởi động lại một service để phát sinh log.

3. Kiểm tra chỉ mục (Index) mới tạo trong Elasticsearch:
   ```bash
   curl http://127.0.0.1:9200/_cat/indices?v
   ```

*Kết quả mong đợi:* Xuất hiện index theo ngày dạng `quickbite-YYYY.MM.DD` với số lượng tài liệu (`docs.count`) tăng dần theo số lượng log phát sinh:

```text
health status index                 uuid                   pri rep docs.count docs.deleted store.size
yellow open   quickbite-2026.08.14  8xK9LmPqR1s2TuVwXyZ34A   1   1        142            0    125.4kb
```

#### 4.5 Bảng chẩn đoán và khắc phục sự cố kết nối

| Hiện tượng lỗi | Nguyên nhân gốc rễ | Hướng xử lý |
| :--- | :--- | :--- |
| **Container ứng dụng không thể khởi động** | Fluentd chưa chạy hoặc địa chỉ `fluentd-address` bị sai/chưa mở cổng `24224`. | Kiểm tra `docker compose ps` tại thư mục `efk/`, đảm bảo Fluentd đang chạy và cổng `127.0.0.1:24224` đã được publish. |
| **Fluentd nhận log nhưng Elasticsearch không có index** | Cấu hình `<match>` trong `fluent.conf` sai địa chỉ hoặc Elasticsearch bị lỗi bộ nhớ. | Xem log của Fluentd qua `docker compose logs fluentd` và kiểm tra sức khỏe cụm qua `curl 127.0.0.1:9200/_cat/health?v`. |
| **Log trong Elasticsearch bị hiển thị dạng text thô trong trường `log`** | Parser JSON của Fluentd chưa hoạt động hoặc tag không khớp với pattern `<filter quickbite.**>`. | Kiểm tra lại cú pháp `key_name log` và đảm bảo tag bắt đầu bằng tiền tố `quickbite.`. |

---

### PHẦN 5. TỔNG KẾT

* **Cơ chế hoạt động:** Docker Logging Driver `fluentd` tự động bắt luồng `stdout`/`stderr` và chuyển tiếp trực tiếp sang Fluentd Collector mà không yêu cầu thay đổi mã nguồn Java.
* **Định tuyến nhãn (Tagging):** Khai báo tag theo tiền tố `quickbite.<service>` giúp phân luồng và xử lý đồng bộ toàn bộ microservices.
* **Giải mã dữ liệu:** Parser Filter của Fluentd với cấu hình `reserve_data true` giúp hợp nhất hoàn hảo giữa metadata hạ tầng container và nội dung log nghiệp vụ ECS.
* **Tạo chỉ mục tự động:** Cấu hình Elasticsearch output của Fluentd tự động gom nhóm log vào các index theo ngày (`quickbite-YYYY.MM.DD`).

---

### PHẦN 6. CÂU HỎI TỰ LUẬN ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)

**Câu hỏi:** Phân tích ý nghĩa của tùy chọn `fluentd-async: "true"` trong cấu hình Docker Compose. Tại sao không nên coi đây là giải pháp để bỏ qua việc theo dõi Fluentd?

**Gợi ý trả lời:**

* `fluentd-async: "true"` kích hoạt chế độ gửi bất đồng bộ, giúp container ứng dụng không bị chặn (blocked) khi khởi động nếu Fluentd tạm thời chưa sẵn sàng hoặc có độ trễ kết nối.
* Tuy nhiên, Docker Daemon chỉ lưu trữ log trong một bộ đệm vòng tròn (ring buffer) có dung lượng hữu hạn trong bộ nhớ. Nếu Fluentd bị sập kéo dài hoặc mất kết nối vĩnh viễn, bộ đệm này sẽ bị tràn và các dòng log tiếp theo sẽ bị bỏ rơi (dropped). Do đó, Fluentd vẫn bắt buộc phải được giám sát và duy trì tính sẵn sàng liên tục.

#### Câu 2 (Phân tích cấu hình)

**Câu hỏi:** Tại sao tham số `reserve_data true` trong cấu hình `<filter quickbite.**>` của Fluentd là bắt buộc? Nếu bỏ tham số này, điều gì sẽ xảy ra?

**Gợi ý trả lời:**

Mặc định khi parser JSON giải mã trường `log`, nó sẽ thay thế toàn bộ bản ghi gốc bằng các trường mới được parse ra. Tham số `reserve_data true` chỉ định cho Fluentd phải giữ lại toàn bộ các trường metadata ban đầu do Docker cung cấp (`container_name`, `container_id`, `source`). Nếu bỏ tham số này, bản ghi đẩy vào Elasticsearch sẽ chỉ có các trường của Spring Boot và mất sạch toàn bộ thông tin định danh container của Docker.

#### Câu 3 (Thao tác vận hành)

**Câu hỏi:** Khi chỉnh sửa khối `logging` trong tệp `docker-compose.yml` của các microservices, tại sao lệnh `docker compose restart` không đủ để áp dụng thay đổi mà bắt buộc phải chạy `docker compose up -d --force-recreate`?

**Gợi ý trả lời:**

Logging driver là cấu hình thuộc về thuộc tính khởi tạo container (`HostConfig.LogConfig`). Lệnh `restart` chỉ khởi động lại tiến trình bên trong container hiện có mà không làm thay đổi cấu hình hạ tầng container. Để thay đổi logging driver, bắt buộc phải hủy bỏ container cũ và tạo mới container bằng lệnh `docker compose up -d --force-recreate`.

---

### PHẦN 7. BÀI TẬP THỰC HÀNH

1. Khởi động cụm EFK và kiểm tra kết nối tại cổng `127.0.0.1:24224` trên máy chủ host.
2. Bổ sung cấu hình logging driver với các tag tương ứng cho cả 4 microservices trong tệp `docker-compose.yml`.
3. Thực hiện tái tạo container bằng lệnh `docker compose up -d --force-recreate` và theo dõi quá trình gửi log qua `docker compose logs -f fluentd`.
4. Truy vấn danh sách chỉ mục trong Elasticsearch bằng `curl http://127.0.0.1:9200/_cat/indices?v` và xác nhận index `quickbite-*` đã được tạo thành công.

---

### KẾT QUẢ MONG ĐỢI

Sau khi hoàn thành bài học, học viên cấu hình thành thạo Docker Logging Driver để chuyển tiếp toàn bộ dữ liệu log từ các container Spring Boot sang Fluentd Collector, giải mã thành công dữ liệu có cấu trúc và ghi nhận bền vững vào các index của Elasticsearch.
