# SESSION 16: LOGGING TRONG SPRING BOOT MÔI TRƯỜNG PRODUCTION

## LESSON 03: Cấu hình Logback trong Spring Boot

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, học viên có khả năng:

* **Giải thích** được vai trò và cơ chế hoạt động của Logback trong hệ sinh thái Spring Boot.
* **Đọc hiểu và phân tích** các thành phần cốt lõi trong tệp cấu hình `logback-spring.xml`.
* **Cấu hình** thành thạo Console Appender, Encoder và Root Logger cho các microservices trong QuickBite.
* **Điều chỉnh linh hoạt** log level thông qua cấu hình và biến môi trường mà không cần thay đổi mã nguồn Java.
* **Kiểm tra và kiểm soát** luồng log của ứng dụng khi triển khai thực tế bên trong Docker container.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ

Trong mã nguồn các service của **QuickBite**, lập trình viên thường xuyên sử dụng các câu lệnh `log.info()`, `log.warn()` hoặc `log.error()`. Tuy nhiên, các câu lệnh này chỉ chịu trách nhiệm sinh nội dung, chứ không tự quyết định thông điệp sẽ được ghi ra đâu (console hay file), theo định dạng nào (plain text hay JSON) và những mức độ nghiêm trọng nào được phép xuất hiện.

Nếu thiếu cấu hình chuẩn hóa, mỗi service sẽ hoạt động một cách tự phát:
* Service này in log ra console, service khác tự ghi vào file trong container dẫn tới nguy cơ mất dữ liệu.
* Service này bật mức `DEBUG` gây tràn đĩa, service khác chỉ ghi `ERROR` khiến việc truy vết trở nên bất khả thi.

Khi xảy ra sự cố trên cụm máy chủ VPS, sự thiếu đồng bộ này sẽ khiến việc tổng hợp và phân tích dữ liệu bị tê liệt. **Logback** đóng vai trò là engine trung tâm tiếp nhận sự kiện từ SLF4J để xử lý định dạng và điều hướng luồng xuất log:

```text
Lời gọi log trong Java (SLF4J)
        ↓
    Logback Engine
        ↓
 Appender & Encoder (Định dạng & Lọc)
        ↓
 Luồng stdout / stderr của Docker Container
```

---

### PHẦN 3. KIẾN THỨC CỐT LÕI

#### 3.1 Tệp cấu hình `logback-spring.xml` trong Spring Boot

Spring Boot sử dụng Logback làm logging framework mặc định thông qua starter `spring-boot-starter-logging`. Cấu hình logging được đặt trong tệp `logback-spring.xml` tại thư mục `src/main/resources`:

```text
order-service/src/main/resources/logback-spring.xml
user-service/src/main/resources/logback-spring.xml
restaurant-service/src/main/resources/logback-spring.xml
notification-service/src/main/resources/logback-spring.xml
```

> **Nguyên tắc đặt tên:** Luôn sử dụng tên `logback-spring.xml` thay vì `logback.xml` tiêu chuẩn. Tiền tố `-spring` cho phép Logback tận dụng đầy đủ các tính năng nâng cao của Spring như: đọc biến môi trường từ `application.yml`, hỗ trợ cấu hình theo Spring Profile (`<springProfile>`) và kế thừa các thiết lập mặc định của framework.

#### 3.2 Các thành phần cốt lõi của cấu hình Logback

Cấu hình chuẩn của `order-service` được thiết lập như sau:

```xml
<configuration>
    <property name="service.name" value="order-service"/>

    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="co.elastic.logging.logback.EcsEncoder">
            <serviceName>${service.name}</serviceName>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE" />
    </root>
</configuration>
```

##### Bảng phân tích chi tiết các thẻ cấu hình:

| Thẻ cấu hình | Chức năng kỹ thuật | Ý nghĩa trong QuickBite |
| :--- | :--- | :--- |
| `<property>` | Khai báo biến tái sử dụng trong file cấu hình. | Định nghĩa tên định danh dịch vụ (`service.name`). |
| `<appender>` | Xác định đích đến của dữ liệu log (Console, File, Socket). | `ConsoleAppender` chuyển hướng toàn bộ log ra luồng `stdout`/`stderr`. |
| `<encoder>` | Định dạng cấu trúc dữ liệu của từng dòng log. | Chuyển đổi log event thành định dạng JSON chuẩn ECS. |
| `<root>` | Logger gốc áp dụng cho toàn bộ ứng dụng. | Thiết lập mức chặn tối thiểu là `INFO` (bỏ qua `TRACE`, `DEBUG`). |
| `<appender-ref>` | Liên kết Root Logger với Appender cụ thể. | Gắn cấu hình Console Appender vào luồng xuất log chính. |

> **Lưu ý:** Mỗi service bắt buộc phải khai báo giá trị `service.name` duy nhất và chính xác để phục vụ việc phân biệt nguồn log khi chạy đồng thời nhiều container.

#### 3.3 Phân biệt Root Logger và Package Logger

Root Logger là bộ lọc toàn cục. Khi Root Logger đặt ở mức `INFO`, toàn bộ các log `DEBUG` và `TRACE` từ mã nguồn ứng dụng lẫn các thư viện phụ thuộc (Spring, Hibernate, Tomcat) đều bị triệt tiêu.

Khi cần điều tra sự cố ở một module cụ thể mà không muốn làm ngập hệ thống bởi log của các thư viện bên thứ ba, ta có thể cấu hình **Package Logger** riêng biệt:

```yaml
# Cấu hình trong application.yml
logging:
  level:
    root: INFO
    com.quickbite.order_service: DEBUG
```

Cấu hình này giữ cho toàn bộ hệ thống hoạt động ở mức `INFO`, nhưng riêng package `com.quickbite.order_service` sẽ xuất thêm log `DEBUG` để phục vụ chẩn đoán.

#### 3.4 Quản lý Log Level linh hoạt bằng biến môi trường

Trên môi trường Production với Docker, việc thay đổi cấu hình không được phép làm gián đoạn hoặc yêu cầu build lại mã nguồn. Spring Boot hỗ trợ ánh xạ cấu hình từ biến môi trường:

```yaml
logging:
  level:
    root: ${LOG_LEVEL_ROOT:INFO}
    com.quickbite.order_service: ${LOG_LEVEL_ORDER_SERVICE:INFO}
```

Khi triển khai với Docker Compose, ta có thể ghi đè log level trực tiếp:

```yaml
services:
  quickbite-order:
    environment:
      LOG_LEVEL_ROOT: INFO
      LOG_LEVEL_ORDER_SERVICE: DEBUG
```

#### 3.5 Nguyên tắc Console Logging trong Container

Trong kiến trúc container, `ConsoleAppender` đưa toàn bộ log ra `stdout`/`stderr`. Docker Engine sẽ tự động thu gom các luồng này và phân phối tới các công cụ điều tra qua lệnh `docker compose logs`.

Việc cấu hình ghi file trực tiếp bên trong container là không cần thiết, làm tăng chi phí quản lý volume, phân quyền ghi đĩa và tiềm ẩn rủi ro mất dữ liệu khi container bị tái tạo.

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (CẤU HÌNH VÀ KIỂM TRA LOGBACK)

Phần thực hành tập trung vào việc đối chiếu cấu hình Logback giữa các service QuickBite và kiểm chứng cơ chế điều khiển log level trên môi trường Docker.

#### 4.1 Rà soát cấu hình Logback của OrderService

Kiểm tra cấu trúc tệp `logback-spring.xml` của `order-service` để đảm bảo tuân thủ tiêu chuẩn:

```xml
<configuration>
    <!-- 1. Tên service định danh -->
    <property name="service.name" value="order-service"/>

    <!-- 2. Console Appender với ECS JSON Encoder -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="co.elastic.logging.logback.EcsEncoder">
            <serviceName>${service.name}</serviceName>
        </encoder>
    </appender>

    <!-- 3. Mức log mặc định của ứng dụng -->
    <root level="INFO">
        <appender-ref ref="CONSOLE" />
    </root>
</configuration>
```

Tiêu chí đánh giá cấu hình:
1. Đã khai báo dependency `co.elastic.logging:logback-ecs-encoder` trong `build.gradle`.
2. Sử dụng `ConsoleAppender` chuẩn thay vì FileAppender.
3. Thuộc tính `service.name` phản ánh chính xác tên microservice.
4. Mức root logger được khóa ở `INFO` để bảo đảm an toàn hiệu năng.

#### 4.2 Tính nhất quán về `service.name` giữa các Microservices

Mỗi service trong hệ thống QuickBite cần có tệp cấu hình độc lập với tên dịch vụ tương ứng:

| Service | Đường dẫn cấu hình | Giá trị `service.name` bắt buộc |
| :--- | :--- | :--- |
| `order-service` | `order-service/src/main/resources/logback-spring.xml` | `order-service` |
| `restaurant-service` | `restaurant-service/src/main/resources/logback-spring.xml` | `restaurant-service` |
| `user-service` | `user-service/src/main/resources/logback-spring.xml` | `user-service` |
| `notification-service` | `notification-service/src/main/resources/logback-spring.xml` | `notification-service` |

> **Cảnh báo sai lầm:** Sao chép file cấu hình giữa các dự án mà quên đổi giá trị `service.name` sẽ dẫn đến việc toàn bộ log bị gán nhầm nguồn gốc, gây sai lệch hoàn toàn dữ liệu điều tra trên Production.

#### 4.3 Quan sát output log từ Console Appender

Khởi chạy cụm dịch vụ QuickBite và kiểm tra luồng log được xuất ra từ container:

```bash
cd ~/quickbite
docker compose ps
docker compose logs --tail 30 quickbite-order
```

Nếu container đang chạy và xuất hiện các dòng log, điều này chứng minh `ConsoleAppender` đã ghi dữ liệu thành công ra `stdout` và được Docker tiếp nhận.

Nếu không quan sát được log, tiến hành kiểm tra theo các bước:
1. Container có đang trong trạng thái `running` không?
2. Tệp cấu hình có đặt đúng tên `logback-spring.xml` trong thư mục `src/main/resources` không?
3. Cấu hình root logger có vô tình bị đặt lên mức `ERROR` khiến các log `INFO` bị lọc bỏ không?

#### 4.4 Thực nghiệm điều chỉnh mức log DEBUG có kiểm soát

Khi cần điều tra chuyên sâu, cấu hình bổ sung biến môi trường trong `docker-compose.yml` để nâng mức log cho riêng `order-service`:

```yaml
services:
  quickbite-order:
    environment:
      - LOG_LEVEL_ROOT=INFO
      - LOG_LEVEL_ORDER_SERVICE=DEBUG
```

Khởi động lại service và kiểm tra log:

```bash
docker compose up -d quickbite-order
docker compose logs -f quickbite-order
```

Sau khi hoàn tất việc chẩn đoán lỗi, bắt buộc phải đưa cấu hình về mức `INFO` để tránh lãng phí tài nguyên lưu trữ của máy chủ.

---

### PHẦN 5. TỔNG KẾT

* **Vai trò của Logback:** Logback là động cơ thực thi nhận log từ SLF4J, áp dụng quy tắc lọc log level, định dạng dữ liệu và điều hướng xuất log ra đích đến.
* **Cấu hình chuẩn trong Container:** Sử dụng `logback-spring.xml`, xuất log qua `ConsoleAppender` ra `stdout` và đặt root logger ở mức `INFO`.
* **Định danh nguồn gốc:** Thuộc tính `service.name` là thông tin bắt buộc để phân biệt nguồn phát log giữa các microservices.
* **Tối ưu hóa chẩn đoán:** Sử dụng package logger và biến môi trường để bật mức `DEBUG` cục bộ khi cần thiết mà không phải sửa mã nguồn hay làm ảnh hưởng tới toàn bộ hệ thống.

---

### PHẦN 6. CÂU HỎI TỰ LUẬN ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)

**Câu hỏi:** `ConsoleAppender` có vai trò và ý nghĩa như thế nào khi ứng dụng Spring Boot chạy trong Docker container?

**Gợi ý trả lời:**

`ConsoleAppender` điều hướng toàn bộ dữ liệu log ra luồng đầu ra tiêu chuẩn (`stdout`/`stderr`) của tiến trình Java. Khi chạy trong Docker, Docker Engine sẽ trực tiếp bắt các luồng này và lưu trữ dưới dạng container logs, cho phép kỹ sư vận hành sử dụng lệnh `docker compose logs` để theo dõi tập trung mà không cần can thiệp vào bên trong hệ thống tệp của container.

#### Câu 2 (Phân tích)

**Câu hỏi:** Nếu Root Logger được cấu hình ở mức `WARN`, điều gì sẽ xảy ra với các lời gọi `log.info()`, `log.warn()` và `log.error()` trong mã nguồn?

**Gợi ý trả lời:**

* `log.info()` sẽ bị bỏ qua và không xuất hiện trong log vì có độ ưu tiên thấp hơn mức `WARN`.
* `log.warn()` và `log.error()` vẫn được ghi nhận bình thường vì có độ ưu tiên bằng hoặc cao hơn `WARN`.
* Hậu quả: Việc đặt root logger ở mức `WARN` sẽ làm mất các thông tin mốc nghiệp vụ quan trọng (như tạo đơn thành công, xử lý thanh toán) vốn được ghi ở mức `INFO`.

#### Câu 3 (Xử lý tình huống)

**Câu hỏi:** Khi cần điều tra lỗi trong `order-service`, vì sao nên ưu tiên bật `DEBUG` cho riêng package `com.quickbite.order_service` thay vì bật `DEBUG` cho Root Logger?

**Gợi ý trả lời:**

Bật `DEBUG` cho Root Logger sẽ kích hoạt log chi tiết của toàn bộ các framework bên thứ ba (Spring Framework, Hibernate SQL, Tomcat Engine), sinh ra hàng chục nghìn dòng log rác gây nghẽn I/O và làm trôi mất thông tin cần tìm. Bật `DEBUG` cho riêng package của service giúp khu trú chính xác luồng nghiệp vụ cần điều tra mà vẫn giữ cho hệ thống hoạt động ổn định.

#### Câu 4 (Thực hành)

**Câu hỏi:** Trình bày phương pháp thay đổi log level của một service đang chạy trong Docker Compose mà không cần sửa mã nguồn và không cần build lại Docker image.

**Gợi ý trả lời:**

Khai báo log level trong `application.yml` dưới dạng tham chiếu biến môi trường (ví dụ: `logging.level.root=${LOG_LEVEL_ROOT:INFO}`). Sau đó, trong file `docker-compose.yml`, truyền giá trị mong muốn qua mục `environment` của service (ví dụ: `LOG_LEVEL_ROOT: DEBUG`) và chạy lệnh `docker compose up -d <service-name>` để áp dụng ngay lập tức.

---

### PHẦN 7. BÀI TẬP THỰC HÀNH

1. Mở và kiểm tra tệp `logback-spring.xml` của cả bốn service trong QuickBite, ghi nhận giá trị của `service.name`, loại appender và root level.
2. Mô tả chi tiết hành trình của một thông điệp log từ khi được gọi bởi `log.info()` trong Java cho đến khi hiển thị trên màn hình qua lệnh `docker compose logs`.
3. Thực hiện cấu hình chuyển đổi mức log của package `com.quickbite.order_service` sang `DEBUG` thông qua biến môi trường của Docker Compose và kiểm tra kết quả.
4. Phân tích các rủi ro hệ thống nếu hai service khác nhau cùng khai báo chung một giá trị `service.name`.

---

### KẾT QUẢ MONG ĐỢI

Sau khi hoàn thành bài học, học viên có thể đọc hiểu, thiết lập và tùy chỉnh cấu hình Logback cho các service Spring Boot, kiểm soát hiệu quả định dạng và mức độ chi tiết của log trên môi trường Docker Production.
