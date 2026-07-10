# SESSION 13: GIÁM SÁT HỆ THỐNG VỚI PROMETHEUS

## LESSON 02: Tích hợp Spring Boot Actuator và cấu hình xuất chỉ số (Metrics Exposure)

---

### PHẦN 1. MỤC TIÊU BÀI HỌC
Sau khi hoàn thành bài học này, học viên có khả năng:
* **Tích hợp** thành công các thư viện Spring Boot Actuator và Micrometer Prometheus Registry vào mã nguồn dự án Spring Boot.
* **Cấu hình** chọn lọc các endpoints giám sát thông qua tệp tin `application.yml`.
* **Áp dụng** nhãn ứng dụng tự động (Common Tags) vào toàn bộ metrics của dịch vụ để phân biệt nguồn gốc dữ liệu.
* **Giải thích** được các nguyên tắc bảo mật thông tin khi phơi bày (expose) dữ liệu Actuator trên môi trường Production.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ
Mỗi microservice viết bằng Java Spring Boot (như `user-service`, `order-service`) thực chất là một tiến trình chạy ngầm phức tạp trong máy ảo Java Virtual Machine (JVM). Khi ứng dụng có dấu hiệu chạy chậm, lập trình viên cần biết các thông số kỹ thuật nội bộ của JVM:
* Lượng RAM cấp phát cho vùng nhớ Heap (Heap Memory) đã dùng bao nhiêu byte?
* Số lượng Thread đang hoạt động đồng thời (Live Threads) là bao nhiêu?
* Tiến trình Garbage Collection (GC) dọn rác RAM hoạt động mất bao lâu và tần suất thế nào?

Tuy nhiên, các thông tin này mặc định nằm ẩn sâu bên trong JVM. Nếu chúng ta tự viết code Java để định kỳ đo đạc và in ra log file thì sẽ cực kỳ tốn công và làm log phình to rất nhanh. Chúng ta cần một giải pháp có sẵn để tự động đo đạc toàn bộ các thông số này, đồng thời xuất (expose) chúng ra dưới dạng một endpoint web chuẩn hóa để máy chủ Prometheus có thể truy cập kéo dữ liệu về.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI

#### 3.1 Spring Boot Actuator và Endpoints
* **Spring Boot Actuator:** Là một module chính thống của Spring Boot, cung cấp các tính năng sẵn dùng (Out-of-the-box) giúp kiểm tra và tương tác với ứng dụng đang chạy.
* **Endpoints:** Actuator cung cấp các endpoint HTTP để hiển thị thông tin. Một số endpoint phổ biến:
  * `/actuator/health`: Trạng thái sức khỏe ứng dụng (UP/DOWN).
  * `/actuator/info`: Thông tin chung của ứng dụng (phiên bản, tác giả).
  * `/actuator/metrics`: Danh sách các chỉ số thô được đo đạc.

#### 3.2 Bộ chuyển đổi Micrometer Prometheus Registry
Mặc dù Actuator thu thập được rất nhiều chỉ số qua endpoint `/actuator/metrics`, nhưng dữ liệu trả về ở đây có định dạng JSON phân tầng phức tạp của Spring Boot. Prometheus Server hoàn toàn không thể đọc và hiểu được cấu trúc JSON này.

Để giải quyết, chúng ta sử dụng thư viện **Micrometer** kết hợp với **Prometheus Registry**. Micrometer đóng vai trò là một lớp trừu tượng đo đạc (giống như SLF4J đối với logging). Khi tích hợp Prometheus Registry, Micrometer sẽ tự động dịch chuyển toàn bộ dữ liệu chỉ số thu thập được từ Actuator sang định dạng văn bản thô (Plain text) theo đúng cú pháp dòng lệnh của Prometheus và xuất ra tại endpoint chuyên biệt:
```text
/actuator/prometheus
```

#### 3.3 Nguyên lý bảo mật Endpoint giám sát
Các endpoint của Actuator (đặc biệt là metrics và prometheus) chứa các thông số cực kỳ chi tiết về cấu trúc hệ thống, cổng chạy, tên class, tài nguyên RAM/CPU. Nếu hacker tiếp cận được các thông tin này, họ có thể phân tích sơ đồ hệ thống để lên kịch bản tấn công khai thác.

Do đó, trên môi trường Production, chúng ta áp dụng các nguyên tắc bảo mật nghiêm ngặt:
1. **Chỉ mở các endpoint cần thiết:** Tuyệt đối không dùng cấu hình mở toang hoác `include: "*"` mà chỉ mở cụ thể `health` và `prometheus`.
2. **Cô lập mạng:** Chỉ cho phép truy cập các cổng microservices nội bộ (như `8081`, `8083`) từ mạng ảo của Docker. Không cấu hình proxy chuyển tiếp các cổng này ra ngoài qua Nginx. Chỉ duy nhất container Prometheus nằm chung mạng Docker mới có quyền kéo dữ liệu từ các endpoint này.

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (CẤU HÌNH XUẤT METRICS CHO ORDER-SERVICE)

Học viên thực hiện cấu hình mã nguồn và tệp thiết lập cho dịch vụ `order-service` để phơi bày dữ liệu metrics sang dạng Prometheus.

#### 4.1 Cấu hình Dependencies trong tệp `build.gradle`
Thêm 2 thư viện Actuator và Micrometer Prometheus vào khối dependencies của dự án `order-service`:
```groovy
dependencies {
    // ... các dependencies khác giữ nguyên ...
    
    # 1. Thư viện Actuator thu thập thông số JVM
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    
    # 2. Thư viện dịch metrics sang định dạng Prometheus
    implementation 'io.micrometer:micrometer-registry-prometheus'
}
```
*(Sau khi thêm, nhớ chạy `./gradlew build` hoặc Refresh Gradle project trên IDE để tải thư viện về).*

#### 4.2 Cấu hình tệp tin `application.yml` của Service
Chỉnh sửa file cấu hình `src/main/resources/application.yml` của dự án `order-service`:
```yaml
spring:
  application:
    name: order-service

management:
  endpoints:
    web:
      exposure:
        # Chỉ mở hai endpoint health và prometheus ra ngoài HTTP
        include: health, prometheus
  metrics:
    tags:
      # Tự động gắn nhãn "application" chứa tên ứng dụng vào mọi metrics được tạo ra
      application: ${spring.application.name}
```

#### 4.3 Kiểm chứng kết nối tại Local
1. Khởi chạy dự án `order-service` ở máy local của bạn.
2. Mở trình duyệt web hoặc công cụ Postman truy cập vào đường dẫn:
   ```text
   http://localhost:8083/actuator/prometheus
   ```
3. *Kết quả mong đợi:* Nhận về dữ liệu văn bản thô chứa hàng ngàn dòng metrics có định dạng bắt đầu bằng chữ hoặc dấu gạch dưới, kèm theo giá trị số ở cuối:
   ```text
   # HELP jvm_memory_used_bytes The amount of used memory
   # TYPE jvm_memory_used_bytes gauge
   jvm_memory_used_bytes{application="order-service",area="heap",id="G1 Eden Space",} 4.194304E7
   jvm_memory_used_bytes{application="order-service",area="nonheap",id="Metaspace",} 3.456128E7
   ```

---

### PHẦN 5. CÂU HỎI TỰ LUẬN ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
* **Câu hỏi:** Giải thích vai trò của thư viện `Micrometer Prometheus Registry` khi kết hợp với Spring Boot Actuator. Nếu thiếu thư viện này thì Prometheus Server có thể giám sát được ứng dụng Spring Boot không?
* **Gợi ý trả lời:**
  * Vai trò: Micrometer đóng vai trò là thư viện đo đạc trung gian, thu thập metrics từ Actuator và chuyển đổi (định dạng) dữ liệu JSON phức tạp của Spring Boot sang định dạng văn bản thô (Plain text) theo chuẩn cú pháp dòng lệnh của Prometheus.
  * Nếu thiếu thư viện này, Nginx/Prometheus Server không thể giám sát được ứng dụng Spring Boot, vì endpoint `/actuator/prometheus` sẽ không tồn tại (trả về lỗi 404), hoặc nếu dùng endpoint `/actuator/metrics` thì định dạng trả về là JSON mà Prometheus không thể tự đọc hiểu được.

#### Câu 2 (Phân tích)
* **Câu hỏi:** Tại sao chúng ta cần cấu hình thẻ `management.metrics.tags.application: ${spring.application.name}` trong file `application.yml` của các microservices? Hãy phân tích tầm quan trọng của nhãn này khi hệ thống QuickBite chạy song song nhiều service trên Production.
* **Gợi ý trả lời:**
  * Thẻ cấu hình này giúp tự động đính kèm một cặp nhãn (label) khóa-giá trị là `application="tên_dịch_vụ"` vào tất cả các metrics do microservice đó sinh ra.
  * Tầm quan trọng: Trên Production, máy chủ Prometheus sẽ đi quét và kéo metrics từ rất nhiều container dịch vụ chạy song song (như `user-service`, `order-service`, `restaurant-service`). Nếu không có nhãn `application` để phân biệt, các metrics trùng tên (ví dụ: `jvm_memory_used_bytes`) từ tất cả các container sẽ bị gộp chung làm một, khiến chúng ta không thể vẽ biểu đồ RAM/CPU riêng biệt cho từng service và không thể xác định được tiến trình nào đang gặp sự cố quá tải.

#### Câu 3 (Nâng cao)
* **Câu hỏi:** Trên môi trường Production của VPS, một DevOps đã cấu hình cho phép expose toàn bộ các endpoints của Actuator ra ngoài Internet bằng dòng lệnh `management.endpoints.web.exposure.include: "*"` để tiện kiểm tra lỗi từ xa. Hãy phân tích các nguy cơ bảo mật nghiêm trọng của hành động này và đề xuất giải pháp cấu hình an toàn thay thế.
* **Gợi ý trả lời:**
  * *Các nguy cơ bảo mật nghiêm trọng:*
    * Rò rỉ thông tin nhạy cảm: Các endpoint như `/actuator/env` hiển thị toàn bộ các biến môi trường (bao gồm cả mật khẩu DB, khóa bí mật JWT, API key hệ thống dạng thô).
    * Nguy cơ phá hoại hệ thống: Endpoint `/actuator/shutdown` (nếu được bật) cho phép bất kỳ ai gửi request HTTP POST để tắt ứng dụng Spring Boot từ xa.
    * Lộ sơ đồ kiến trúc ứng dụng: Endpoint `/actuator/mappings` hiển thị toàn bộ các đường dẫn URL API, cổng chạy, cấu trúc class và controllers của mã nguồn.
  * *Giải pháp cấu hình an toàn thay thế:*
    1. Chỉ mở các endpoint thật sự cần thiết bằng cách khai báo cụ thể:
       `management.endpoints.web.exposure.include: health, prometheus`
    2. Chặn toàn bộ kết nối từ ngoài Internet tới các cổng của microservices (như `8081`, `8083`) thông qua tường lửa VPS. Các cổng này chỉ để chạy nội bộ trong mạng ảo Docker, chỉ cho phép duy nhất container Prometheus truy cập lấy dữ liệu qua IP nội bộ.
