# SESSION 05: MULTI-SERVICES & API GATEWAY

## LESSON 05: Spring Cloud Gateway và định tuyến yêu cầu trong kiến trúc đa dịch vụ

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Biên soạn** chính xác tệp cấu hình YAML định nghĩa định tuyến của Spring Cloud Gateway.
* **Phân biệt và cấu hình** thành công các thành phần cốt lõi của một Route: `ID`, `URI`, `Predicates` (điều kiện lọc) và `Filters` (bộ lọc thay đổi request/response).
* **Tích hợp** container dịch vụ API Gateway vào hệ thống `docker-compose.yml` chung.
* **Thực hiện đóng kín** cổng dịch vụ của các microservices nội bộ và mở duy nhất cổng gateway để bảo mật hệ thống.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (NỖI ĐAU CỦA VIỆC ĐỊNH TUYẾN THỦ CÔNG BẰNG CODE)

Khi quyết định đặt một API Gateway làm cửa ngõ cho 4 dịch vụ QuickBite, nếu chúng ta tự viết một ứng dụng Spring Boot thông thường sử dụng `RestTemplate` trong Controller để nhận và chuyển tiếp thủ công từng API:
1. *Nỗi đau viết mã lặp (Boilerplate code):* Bạn phải viết hàng tá các class Controller chỉ để nhận request rồi gửi tiếp đi. Khi một microservice phía sau thêm mới 10 API, bạn lại phải sửa code của Gateway để bổ sung các Endpoint tương ứng và deploy lại.
2. *Nỗi đau thay đổi đường dẫn (Path rewriting):* Giả sử Client gọi tới Gateway bằng đường dẫn `/api/v1/orders/123` nhưng dịch vụ nội bộ `order-service` lại được code để nhận đường dẫn dạng `/orders/123` (bỏ tiền tố `/api/v1`). Việc xử lý cắt chuỗi URL thủ công bằng code Java cho hàng trăm API sẽ dẫn đến rủi ro sai cú pháp và cực kỳ khó quản lý.

*Spring Cloud Gateway ra đời để giải quyết nỗi đau này bằng cách cung cấp một bộ khung cấu hình Declarative (khai báo YAML) cực kỳ mạnh mẽ, giúp cấu hình toàn bộ các tuyến định tuyến và xử lý URL chỉ trong vài dòng cấu hình mà không cần viết một dòng code Java nào.*

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (CÁC THÀNH PHẦN CỦA ROUTE TRONG SPRING CLOUD GATEWAY)

Mô hình định tuyến của Spring Cloud Gateway hoạt động dựa trên ba thành phần nền tảng:

```text
  Client Request ──► [ Predicate (Khớp Path?) ] ──► [ Filter (Sửa Header/Path) ] ──► Downstream Service
```

1. **Route (Tuyến đường):** Là đối tượng định tuyến cơ bản của gateway. Nó bao gồm một ID duy nhất, một URI đích của dịch vụ phía sau, một tập hợp các điều kiện lọc (Predicates) và các bộ lọc (Filters).
2. **Predicate (Điều kiện khớp):** Là một hàm điều kiện logic giúp Gateway xác định xem request hiện tại có khớp với Route này hay không. 
   * Ví dụ: `- Path=/api/v1/users/**` có nghĩa là "nếu đường dẫn request bắt đầu bằng `/api/v1/users/` thì sẽ áp dụng Route này".
3. **Filter (Bộ lọc xử lý):** Là các bộ tiền xử lý (pre-filter) và hậu xử lý (post-filter) cho phép sửa đổi request hoặc response trước và sau khi gửi đến dịch vụ đích.
   * Ví dụ: `StripPrefix=2` (cắt bỏ 2 thành phần đầu tiên của path trước khi chuyển tiếp), hoặc `AddRequestHeader` (thêm Header định danh nguồn gửi request).

---

### PHẦN 4. DEMO THỰC TẾ: CẤU HÌNH SPRING CLOUD GATEWAY VÀ TÍCH HỢP DOCKER COMPOSE

#### 4.1 Biên soạn tệp cấu hình `application.yml` cho Gateway
Dưới đây là cấu hình hoàn chỉnh của dịch vụ Gateway (`gateway-service`), định tuyến các request dựa trên path pattern đến 4 container Spring Boot nội bộ của QuickBite:

```yaml
# gateway-service/src/main/resources/application.yml
server:
  port: 8080

spring:
  cloud:
    gateway:
      routes:
        # 1. Tuyến đường định hướng tới user-service
        - id: user_route
          uri: http://${USER_SVC_HOST:localhost}:${USER_SVC_PORT:8081}
          predicates:
            - Path=/api/v1/users/**

        # 2. Tuyến đường định hướng tới restaurant-service
        - id: restaurant_route
          uri: http://${RESTAURANT_SVC_HOST:localhost}:${RESTAURANT_SVC_PORT:8082}
          predicates:
            - Path=/api/v1/restaurants/**

        # 3. Tuyến đường định hướng tới order-service
        - id: order_route
          uri: http://${ORDER_SVC_HOST:localhost}:${ORDER_SVC_PORT:8083}
          predicates:
            - Path=/api/v1/orders/**

        # 4. Tuyến đường định hướng tới notification-service
        - id: notification_route
          uri: http://${NOTIFICATION_SVC_HOST:localhost}:${NOTIFICATION_SVC_PORT:8084}
          predicates:
            - Path=/api/v1/notifications/**
```

*Giải thích cấu hình:*
* Các biến `${USER_SVC_HOST:localhost}` giúp linh hoạt trỏ về `localhost` khi dev ở máy cá nhân hoặc trỏ về tên container khi chạy trong Docker.
* Ký tự `**` đại diện cho mọi đường dẫn con ở phía sau (Wildcard).

#### 4.2 Cấu hình Tích hợp trong `docker-compose.yml`
Hãy cập nhật tệp `docker-compose.yml` của toàn bộ hệ thống để nhúng Gateway vào mạng nội bộ. Chúng ta sẽ xóa bỏ cấu hình `ports` của 4 dịch vụ backend cũ, chỉ mở duy nhất cổng `8080` của Gateway ra ngoài máy host vật lý.

```yaml
version: '3.8'

services:
  quickbite-db:
    image: postgres:15-alpine
    container_name: quickbite-db
    environment:
      - POSTGRES_PASSWORD=secret
    volumes:
      - ./init-scripts/init-db.sql:/docker-entrypoint-initdb.d/init-db.sql
    networks:
      - quickbite-net

  # Các Dịch vụ Backend (Chỉ dùng expose cổng nội bộ)
  quickbite-user:
    build: context: ./user-service
    container_name: quickbite-user
    expose:
      - "8081"
    environment:
      - DB_HOST=quickbite-db
      - DB_NAME=quickbite_user
    depends_on:
      - quickbite-db
    networks:
      - quickbite-net

  quickbite-restaurant:
    build: context: ./restaurant-service
    container_name: quickbite-restaurant
    expose:
      - "8082"
    environment:
      - DB_HOST=quickbite-db
      - DB_NAME=quickbite_restaurant
    depends_on:
      - quickbite-db
    networks:
      - quickbite-net

  quickbite-order:
    build: context: ./order-service
    container_name: quickbite-order
    expose:
      - "8083"
    environment:
      - DB_HOST=quickbite-db
      - DB_NAME=quickbite_order
      - RESTAURANT_SVC_HOST=quickbite-restaurant
      - RESTAURANT_SVC_PORT=8082
    depends_on:
      - quickbite-db
      - quickbite-restaurant
    networks:
      - quickbite-net

  quickbite-notification:
    build: context: ./notification-service
    container_name: quickbite-notification
    expose:
      - "8084"
    environment:
      - DB_HOST=quickbite-db
      - DB_NAME=quickbite_notification
    depends_on:
      - quickbite-db
    networks:
      - quickbite-net

  # API Gateway (Cổng ngoài duy nhất của toàn bộ hệ thống)
  quickbite-gateway:
    build:
      context: ./gateway-service
    container_name: quickbite-gateway
    ports:
      - "8080:8080" # Ánh xạ cổng 8080 ra ngoài máy host vật lý
    environment:
      - USER_SVC_HOST=quickbite-user
      - USER_SVC_PORT=8081
      - RESTAURANT_SVC_HOST=quickbite-restaurant
      - RESTAURANT_SVC_PORT=8082
      - ORDER_SVC_HOST=quickbite-order
      - ORDER_SVC_PORT=8083
      - NOTIFICATION_SVC_HOST=quickbite-notification
      - NOTIFICATION_SVC_PORT=8084
    depends_on:
      - quickbite-user
      - quickbite-restaurant
      - quickbite-order
      - quickbite-notification
    networks:
      - quickbite-net

networks:
  quickbite-net:
    driver: bridge
```

---

### PHẦN 5. HIỂU LẦM THƯỜNG GẶP (CẤU TRÚC BLOCKING VỚI THREAD-PER-REQUEST)

* **Hiểu lầm thường gặp:** Có thể nhúng các thư viện chặn luồng (Blocking IO) truyền thống như Spring Data JPA/Hibernate để trực tiếp truy cập cơ sở dữ liệu hoặc sử dụng `Thread.sleep()` bên trong các Filter tự viết của Spring Cloud Gateway.
* **Sự thật:** 
  * Spring Cloud Gateway được xây dựng trên nền tảng **Spring WebFlux** và máy chủ web không đồng bộ **Netty** (Non-blocking IO). 
  * Cơ chế của Netty sử dụng very ít luồng (Event Loop Threads) để xử lý đồng thời hàng nghìn request. Nếu bạn thực hiện một thao tác chặn luồng (blocking) bên trong Filter, luồng xử lý đó sẽ bị khóa cứng. Khi tất cả các luồng xử lý của Netty bị khóa, API Gateway sẽ bị treo hoàn toàn và không tiếp nhận thêm bất kỳ request nào khác của người dùng, làm sập toàn bộ hệ thống. 
  * Do đó, mọi bộ lọc Filter tùy biến trong Gateway bắt buộc phải tuân thủ triết lý Reactive (Non-blocking).

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG (XÁC MINH KIẾN THỨC)

1. **Hướng dẫn cấu hình Route trong Spring Cloud Gateway:**
   * [Spring Cloud Gateway Routes - Developer Guide](https://docs.google.com/url?q=https://spring.io/projects/spring-cloud-gateway#learn)
2. **Danh sách các Predicate Factories có sẵn:**
   * [Route Predicate Factories - Spring Docs](https://docs.google.com/url?q=https://docs.spring.io/spring-cloud-gateway/reference/spring-cloud-gateway/gatewayfilter-factories.html)
3. **Danh sách các GatewayFilter Factories có sẵn:**
   * [GatewayFilter Factories - Spring Docs](https://docs.google.com/url?q=https://docs.spring.io/spring-cloud-gateway/reference/spring-cloud-gateway/gatewayfilter-factories.html)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Sự khác biệt lớn nhất giữa `expose` và `ports` trong file cấu hình `docker-compose.yml` ở Phân 4 là gì?
* *Gợi ý:* `ports` dùng để mở và ánh xạ cổng từ container ra ngoài máy host vật lý (Client bên ngoài mạng Docker có thể gọi vào). Trong khi đó, `expose` chỉ khai báo cổng hoạt động nội bộ của container nhằm phục vụ truyền thông giữa các container nằm chung một mạng Docker Network, không cho phép bên ngoài máy host chọc trực tiếp vào cổng này.

#### Câu 2 (Đọc và dự đoán)
Giả sử bạn gửi một request GET tới địa chỉ `http://localhost:8080/api/v1/users/1`. Dựa trên tệp cấu hình của Gateway ở Phần 4, yêu cầu này sẽ khớp với Route nào và được chuyển tiếp sang URL nội bộ nào trong mạng Docker?
* *Gợi ý:* Request khớp với `user_route` (do path khớp điều kiện `/api/v1/users/**`). Nó sẽ được chuyển tiếp sang URL nội bộ của container `quickbite-user`: `http://quickbite-user:8081/api/v1/users/1`.

#### Câu 3 (Xử lý tình huống)
Sau khi chạy lệnh `docker compose up -d`, bạn gửi API request tới cổng 8080 của Gateway và nhận về mã lỗi `503 Service Unavailable`. Việc đầu tiên bạn cần kiểm tra là gì?
* *Gợi ý:* Mã lỗi `503 Service Unavailable` chỉ ra rằng Gateway hoạt động bình thường nhưng không thể chuyển tiếp request tới container backend đích vì container đó đang bị tắt (`Exit`) hoặc cấu hình URL đích (`uri` của route) bị sai host/port. Bạn cần chạy lệnh `docker compose ps` để kiểm tra trạng thái hoạt động của các container backend.
