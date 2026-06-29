# PROMPT CHO GAMMA: MULTI-SERVICES & API GATEWAY (SESSION 5)

## 1. CONTEXT & ROLE
* **Role:** DevOps Architect kiêm Giảng viên cao cấp. Giọng điệu chuyên nghiệp, trực diện, đi thẳng vào bản chất kỹ thuật và thực tiễn vận hành hệ thống.
* **Target Audience:** Kỹ sư phần mềm, học viên đang phát triển dự án Spring Boot Microservices (hệ thống QuickBite).
* **Objective:** Giải thích kiến trúc đa dịch vụ, cơ chế cô lập dữ liệu (Database-per-service), giao tiếp liên dịch vụ qua Docker Network, vai trò và cấu hình định tuyến của API Gateway, và luồng giao dịch nghiệp vụ end-to-end cùng kỹ thuật xử lý giao dịch bù (Compensating Transaction) bảo vệ tính nhất quán dữ liệu.

---

## 2. PARTITION INSTRUCTIONS
* **Số lượng slide khuyến nghị:** 15 - 20 slides.
* **Nguyên tắc phân bổ nội dung:**
  * **LESSON 01 (Thiết kế thực thể):** Kiến trúc đa dịch vụ, Database-per-service, Snapshot Pattern, và liên kết lỏng qua ID.
  * **LESSON 02 & 03 (Khởi chạy & Giao tiếp):** Docker Compose đa dịch vụ, nạp biến môi trường tiền tố, Service Discovery qua DNS nội bộ, và Spring Cloud OpenFeign.
  * **LESSON 04 & 05 (API Gateway & Định tuyến):** Nỗi đau kết nối trực tiếp, khái niệm API Gateway, phân biệt với Reverse Proxy, và cấu hình Route/Predicate/Filter của Spring Cloud Gateway.
  * **LESSON 06 (Luồng chạy & Giao dịch bù):** Vòng đời 6 bước của đơn đặt hàng, Saga Pattern, giao dịch bù (Compensating Transaction), và truy vết nhật ký đan xen (Interlaced Logs).
  * **Độ thoáng đãng:** Trình bày ngắn gọn, súc tích, đi thẳng vào bản chất kỹ thuật. Loại bỏ hoàn toàn các câu từ suồng sã.

---

## 3. HIERARCHICAL OUTLINE (DÀN Ý CHI TIẾT)

### LESSON 01: Kiến trúc đa dịch vụ và thiết kế thực thể trong hệ thống phân tán QuickBite

#### Slide 1: Sự chuyển dịch từ Monolith sang Kiến trúc Đa dịch vụ (Multi-services)
* **Bối cảnh dự án QuickBite:** Chia tách mã nguồn đơn thể thành 4 microservices chạy độc lập:
  * `user-service`: Quản lý tài khoản, ví tiền và địa chỉ.
  * `restaurant-service`: Quản lý thực đơn và nhà hàng.
  * `order-service`: Quản lý giỏ hàng và đơn hàng.
  * `notification-service`: Quản lý thông báo.
* **Nguyên tắc cô lập dữ liệu (Database-per-service):**
  * Mỗi microservice sở hữu một cơ sở dữ liệu logic riêng biệt (ví dụ: `quickbite_user_db`, `quickbite_restaurant_db`).
  * Triệt tiêu hoàn toàn các liên kết khóa ngoại vật lý và các câu lệnh SQL JOIN trực tiếp xuyên database.
  * Mọi giao tiếp dữ liệu bắt buộc phải thực hiện qua giao diện mạng (REST API).

#### Slide 2: Thiết kế liên kết lỏng (Soft References) trong JPA
* **Thách thức:** JPA không hỗ trợ liên kết `@ManyToOne` hay `@ManyToMany` xuyên cơ sở dữ liệu vật lý.
* **Giải pháp: Liên kết lỏng qua ID (Soft References)**
  * Thay vì ánh xạ thực thể Hibernate trực tiếp (ví dụ: trường `User customer` trong thực thể `Order`), ta chỉ lưu trữ khóa chính của thực thể đối tác dưới dạng một trường số nguyên thông thường (`Long customerId`).
  * Khi cần hiển thị thông tin chi tiết, dịch vụ sẽ thực hiện truy vấn HTTP (qua Feign Client) sang dịch vụ quản lý thực thể tương ứng để lấy dữ liệu.

#### Slide 3: Bảo toàn lịch sử dữ liệu với Snapshot Pattern
* **Vấn đề thực tế:** Nếu chỉ lưu ID của món ăn (`menuItemId`) và nhà hàng (`merchantId`) trong bảng Đơn hàng. Khi nhà hàng thay đổi giá món ăn hoặc đổi tên quán, lịch sử các hóa đơn cũ hiển thị cho khách hàng sẽ bị sai lệch.
* **Giải pháp: Thiết kế Snapshot**
  * Tại thời điểm phát sinh giao dịch (đặt đơn), hệ thống chụp lại trạng thái tĩnh của dữ liệu và lưu cứng vào bảng Đơn hàng.
  * Các trường như tên khách hàng (`customer_name`), tên nhà hàng (`merchant_name`), tên món ăn (`item_name`), và giá bán (`price`) được sao chép trực tiếp vào database của `order-service` thay vì tham chiếu động.

---

### LESSON 02: Quy trình chạy Spring Boot Service cùng cơ sở dữ liệu bằng Docker Compose

#### Slide 4: Mô hình phân bổ cổng mạng và kết nối Database dùng chung
* **Mô hình triển khai tối ưu tài nguyên:**
  * Triển khai một container PostgreSQL (`quickbite-db`) chung cho toàn hệ thống.
  * Container PostgreSQL chứa nhiều cơ sở dữ liệu logic độc lập cho từng dịch vụ.
  * Các dịch vụ Spring Boot kết nối tới database tương ứng thông qua mạng ảo chung `quickbite-net`.
* **Tránh xung đột cấu hình bằng tiền tố biến môi trường:**
  * Sử dụng tệp `.env` chung để định nghĩa tham số.
  * Tiền tố hóa các biến cho từng dịch vụ (ví dụ: `USER_DB_NAME`, `RESTAURANT_DB_NAME`) để tránh ghi đè cấu hình Spring Boot khi nạp chung file.

#### Slide 5: Thực hành cấu hình Docker Compose chạy đa dịch vụ
* Trích đoạn cấu hình tệp `docker-compose.yml` điều phối 2 dịch vụ kết nối mạng ngoài:
```yaml
services:
  user-service:
    build: ./user-service
    ports: [- "8081:8081"]
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://quickbite-db:5432/${USER_DB_NAME}
      - SPRING_DATASOURCE_USERNAME=${USER_DB_USERNAME}
      - SPRING_DATASOURCE_PASSWORD=${USER_DB_PASSWORD}
    networks: [- quickbite-net]

  restaurant-service:
    build: ./restaurant-service
    ports: [- "8082:8082"]
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://quickbite-db:5432/${RESTAURANT_DB_NAME}
      - SPRING_DATASOURCE_USERNAME=${RESTAURANT_DB_USERNAME}
      - SPRING_DATASOURCE_PASSWORD=${RESTAURANT_DB_PASSWORD}
    networks: [- quickbite-net]

networks:
  quickbite-net:
    external: true
```

---

### LESSON 03: Giao tiếp liên dịch vụ qua Docker Network trong kiến trúc phân tán

#### Slide 6: Docker Network Bridge và Cơ chế Service Discovery
* **Docker Embedded DNS Server:**
  * Khi các container hoạt động chung trong một mạng bridge ảo, Docker tự động kích hoạt DNS server nội bộ.
  * Trình DNS này phân giải tên dịch vụ khai báo trong Compose (ví dụ: `http://restaurant-service:8082`) thành địa chỉ IP động hiện tại của container tương ứng.
  * Triệt tiêu việc ghi cứng (hardcode) IP tĩnh của container, đảm bảo kết nối hoạt động ổn định khi restart hệ thống.

#### Slide 7: Khai báo giao tiếp đồng bộ bằng Spring Cloud OpenFeign
* **Khái niệm:** OpenFeign là thư viện khai báo REST client trực quan của Spring Cloud, cho phép gọi API của dịch vụ khác như các hàm Java thông thường.
* **Mã nguồn khai báo Feign Client trong `user-service` gọi sang `restaurant-service`:**
```java
@FeignClient(name = "restaurant-service", url = "http://restaurant-service:8082")
public interface RestaurantServiceClient {
    @GetMapping("/api/v1/restaurants/{id}/status")
    Boolean getRestaurantStatus(@PathVariable("id") Long id);
}
```
* *Cơ chế:* URL sử dụng tên dịch vụ `restaurant-service` làm host. DNS của Docker sẽ xử lý phân giải IP ảo tại thời điểm runtime.

#### Slide 8: Bảo mật cổng mạng với nguyên lý mạng nội bộ (Internal Ports)
* **Mối đe dọa:** Expose trực tiếp toàn bộ các cổng mạng (`8081`, `8082`, `8083`) ra máy host vật lý sẽ mở rộng bề mặt tấn công của hệ thống. Tin tặc có thể quét cổng và gọi trực tiếp bypass qua kiểm tra bảo mật.
* **Giải pháp:**
  * Loại bỏ khai báo `ports` khỏi cấu hình các microservices nội bộ trong file Compose.
  * Các microservices chỉ kết nối chéo với nhau bên trong mạng ảo `quickbite-net`.
  * Chỉ mở duy nhất cổng của API Gateway ra ngoài internet để tiếp nhận request.

---

### LESSON 04: API Gateway và điểm truy cập tập trung trong hệ thống Microservices

#### Slide 9: Nỗi đau của việc Client giao tiếp trực tiếp với nhiều microservices
* **Hạn chế của mô hình kết nối phân tán:**
  1. *Quản lý Endpoint phức tạp:* Client (Mobile, Web) phải lưu và quản lý hàng chục URL và port của các dịch vụ khác nhau.
  2. *Lỗi CORS (Cross-Origin Resource Sharing):* Trình duyệt chặn các request gọi chéo cổng mạng, buộc phải cấu hình cho phép CORS trên tất cả các dịch vụ độc lập.
  3. *Trùng lặp mã nguồn bảo mật (Authentication):* Logic xác thực Token JWT và phân quyền phải được lập trình lặp đi lặp lại ở mọi service.

#### Slide 10: Khái niệm và Vai trò của API Gateway (Single Entry Point)
* **Định nghĩa:** API Gateway là một thành phần máy chủ làm điểm truy cập tập trung duy nhất cho mọi yêu cầu gọi API từ phía Client (Web, Mobile) đi vào hệ thống.
* **Các chức năng cốt lõi:**
  * *Định tuyến động (Dynamic Routing):* Tự động điều hướng request dựa trên URI (ví dụ: `/api/v1/users/**` chuyển hướng ngầm tới `user-service`).
  * *Xác thực và phân quyền tập trung:* Giải mã và xác thực Token JWT ngay tại cửa ngõ, bảo vệ các dịch vụ phía sau.
  * *Quản lý CORS tập trung:* Giải quyết lỗi chặn truy cập chéo nguồn từ trình duyệt tại một điểm duy nhất.
  * *Giới hạn tần suất (Rate Limiting):* Giới hạn số lượng request tối đa trên mỗi IP/User để bảo vệ tài nguyên hệ thống.
  * *Lá chắn an toàn (Security Shield):* Cho phép đóng hoàn toàn các cổng mạng nội bộ của microservices, chỉ mở duy nhất cổng gateway (ví dụ: `8080`).

#### Slide 11: Phân biệt API Gateway vs Reverse Proxy Nginx
* **Nginx (Reverse Proxy & Web Server cấp độ hạ tầng):**
  * Hoạt động ở lớp dưới hạ tầng (tầng mạng TCP/UDP và HTTP thô).
  * Tối ưu cho cấu hình bảo mật SSL/HTTPS, nén dữ liệu, và phân phối nhanh các tệp tin tĩnh (HTML, CSS, JS) của Frontend.
  * Khó can thiệp sâu vào logic nghiệp vụ của các dịch vụ chạy phía sau.
* **Spring Cloud Gateway (API Gateway cấp độ ứng dụng):**
  * Hoạt động trên nền tảng Spring, hiểu sâu logic ứng dụng và tích hợp chặt chẽ với hệ sinh thái Java.
  * Cho phép lập trình các bộ lọc (Filters) bằng code Java để can thiệp sâu vào Header, Body của request, kiểm tra quyền hạn nghiệp vụ phức tạp.
* **Mô hình phối hợp chuẩn doanh nghiệp:**
  * *Nginx* đứng ngoài cùng tiếp nhận HTTPS từ internet và phân phối file tĩnh Frontend.
  * *Nginx* chuyển tiếp request API vào bên trong cho *Spring Cloud Gateway* để xử lý logic định tuyến ứng dụng và bảo mật.

---

### LESSON 05: Spring Cloud Gateway và định tuyến yêu cầu trong kiến trúc đa dịch vụ

#### Slide 12: Cơ chế định tuyến của Spring Cloud Gateway
* Luồng xử lý request đi qua Gateway:
```text
  Client Request ──► [ Predicate (Khớp Path?) ] ──► [ Filter (Sửa Header/Path) ] ──► Downstream Service
```
* **Các cấu phần cốt lõi của một Route:**
  * `Route ID`: Định danh duy nhất cho tuyến đường.
  * `URI`: Địa chỉ đích của microservice nhận request (ví dụ: `http://user-service:8081`).
  * `Predicate`: Điều kiện để khớp request (ví dụ: nếu `Path=/api/v1/users/**`).
  * `Filter`: Bộ lọc tiền/hậu xử lý (ví dụ: `StripPrefix=2` để cắt bỏ tiền tố `/api/v1` trước khi chuyển tiếp).

#### Slide 13: Thực hành cấu hình YAML định tuyến tập trung
* Biên soạn tệp cấu hình `application.yml` cho `gateway-service`:
```yaml
server:
  port: 8080
spring:
  cloud:
    gateway:
      routes:
        - id: user_route
          uri: http://user-service:8081
          predicates:
            - Path=/api/v1/users/**
          filters:
            - StripPrefix=2
        - id: order_route
          uri: http://order-service:8083
          predicates:
            - Path=/api/v1/orders/**
          filters:
            - StripPrefix=2
```

#### Slide 14: Tích hợp Gateway vào Docker Compose
* Khai báo container Gateway làm cửa ngõ duy nhất mở cổng ra ngoài máy host vật lý:
```yaml
services:
  gateway-service:
    build: ./gateway-service
    ports:
      - "8080:8080" # Mở duy nhất cổng gateway
    networks:
      - quickbite-net

  user-service:
    build: ./user-service
    # Không khai báo ports để giấu cổng 8081 khỏi máy host
    networks:
      - quickbite-net
```

---

### LESSON 06: Luồng yêu cầu đặt hàng end-to-end từ Client qua API Gateway

#### Slide 15: Vòng đời 6 bước của một đơn đặt hàng (End-to-End Runtime Flow)
* Quy trình tích hợp dòng dữ liệu giữa các container:
```text
  Client App ──(1. POST /api/v1/orders)──► [ API Gateway ] ──► [ order-service ]
                                                                      │
   ┌──────────────────────────────────────────────────────────────────┴───────────────┐
   ▼ (2. Tạo Order PENDING)                                                           ▼ (5. Điều phối DRIVER)
 [ DB: quickbite_order ]                                                             [ user-service ]
   │                                                                                  │
   ├─(3. API: Trừ tiền)──► [ user-service ] ──► [ DB: quickbite_user ]                ├─(6. HTTP POST)──► [ notification-service ]
   │                                                                                  │
   └─(4. API: Báo món)──► [ restaurant-service ] ──► [ DB: quickbite_restaurant ]     ▼
                                                                                    [ DB: quickbite_notification ]
```
1. Client gửi yêu cầu tới Gateway `8080`, Gateway chuyển tiếp tới `order-service` `8083`.
2. `order-service` tạo đơn hàng trạng thái `PENDING` trong database của nó.
3. `order-service` gọi sang `user-service` kiểm tra số dư và trừ tiền ví.
4. `order-service` gọi sang `restaurant-service` báo nhà hàng chuẩn bị món.
5. `order-service` gọi sang `user-service` để điều phối tài xế.
6. `user-service` gọi sang `notification-service` gửi thông báo cho khách hàng.

#### Slide 16: Nhất quán dữ liệu phân tán với Saga Pattern và Giao dịch bù
* **Vấn đề thực tế:** Nếu ví tiền của người dùng đã bị trừ ở bước 3, nhưng đến bước 4 nhà hàng từ chối nhận đơn do hết nguyên liệu. Do hai database nằm ở hai microservices khác nhau, không có cơ chế rollback mặc định nào có thể hoàn lại tiền cho người dùng.
* **Giải pháp: Giao dịch bù (Compensating Transaction)**
  * Lập trình viên phải viết mã xử lý lỗi (catch exception) trong luồng đặt hàng của `order-service`.
  * Khi nhà hàng từ chối, `order-service` cập nhật trạng thái đơn thành `CANCELLED`, đồng thời gửi một request HTTP POST (giao dịch bù) sang `user-service` yêu cầu cộng lại tiền (hoàn tiền) vào ví của người dùng.

#### Slide 17: Kỹ thuật truy vết lỗi bằng Nhật ký đan xen (Interlaced Logs)
* **Thách thức:** Khi xảy ra lỗi giao dịch phân tán, việc mở log của từng container riêng lẻ rất khó để tái hiện lại trình tự lỗi.
* **Giải pháp: Stream log tổng hợp**
  * Sử dụng CLI để stream log đồng thời của toàn bộ cụm dịch vụ:
    ```bash
    docker compose logs -f --tail=100
    ```
  * Docker Compose tự động phân tách màu sắc log của từng container và chèn tiền tố tên dịch vụ ở đầu dòng (ví dụ: `gateway-service | ...`, `order-service | ...`), giúp kỹ sư DevOps dễ dàng truy vết đường đi của request từ lúc đi vào Gateway cho đến khi gặp lỗi tại các microservice phía sau.
