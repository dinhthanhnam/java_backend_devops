# Kịch bản Thuyết trình - Session 05: Multi-services & API Gateway

---

## Lesson 1: Kiến trúc đa dịch vụ và thiết kế thực thể trong hệ thống phân tán QuickBite

### 1. Phần lý thuyết

**(Mở đầu - Khơi gợi bối cảnh thực tế)**
Xin chào các bạn học viên! Ở buổi học trước, chúng ta đã cùng nhau làm quen với Docker Compose và dựng thành công một container database PostgreSQL độc lập. Nhưng trong thực tế phát triển các ứng dụng quy mô lớn như nền tảng giao đồ ăn QuickBite, chúng ta không chỉ có một ứng dụng duy nhất, mà hệ thống sẽ được chia nhỏ thành nhiều dịch vụ backend độc lập, bao gồm: `user-service`, `restaurant-service`, `order-service` và `notification-service`.

Thế thì, khi chuyển từ mô hình Monolith (đơn thể) sang kiến trúc đa dịch vụ (Multi-services Architecture), thay đổi lớn nhất và nhức nhối nhất nằm ở đâu? Đó chính là ở **tổ chức cơ sở dữ liệu**.

**[Slide 1: Nguyên tắc cô lập dữ liệu (Database-per-Service) & Hạn chế của liên kết JPA]**
Trong một hệ thống Monolith truyền thống, tất cả các bảng dữ liệu đều nằm chung trong một cơ sở dữ liệu logic. Việc lấy thông tin người dùng từ đơn hàng cực kỳ đơn giản: các bạn chỉ cần khai báo một mối quan hệ `@ManyToOne User` trong Hibernate, hoặc viết một câu lệnh SQL `JOIN` là xong.

Tuy nhiên, trong kiến trúc Microservices, mỗi dịch vụ sở hữu riêng một cơ sở dữ liệu logic (mẫu thiết kế **Database-per-Service**). 
- `user-service` nắm giữ `quickbite_user_db`.
- `restaurant-service` nắm giữ `quickbite_restaurant_db`.
- `order-service` nắm giữ `quickbite_order_db`.

Do nằm trên các cơ sở dữ liệu độc lập, thậm chí chạy trên các máy chủ vật lý khác nhau, chúng ta **tuyệt đối không thể** thiết lập quan hệ khóa ngoại vật lý (Foreign Key) giữa các bảng nằm ở hai database khác nhau. Các annotation JPA quen thuộc như `@OneToMany` hay `@ManyToMany` xuyên database cũng hoàn toàn bất lực. Mọi giao tiếp và truy vấn dữ liệu chéo lúc này bắt buộc phải đi qua giao diện mạng HTTP REST API.

**[Slide 2: Liên kết lỏng qua ID (Soft References) và Giao tiếp REST API qua FeignClient]**
Để giải quyết bài toán tham chiếu dữ liệu giữa các dịch vụ độc lập mà không dùng khóa ngoại vật lý, chúng ta áp dụng cơ chế **Soft References** (Liên kết lỏng qua ID):
- Bảng `orders` sẽ chỉ lưu các trường ID dạng số nguyên `Long` (như `customer_id`, `restaurant_id`) đại diện cho thực thể ở dịch vụ đối tác.
- Khi cần kiểm tra thông tin đối tác *tại thời điểm xử lý nghiệp vụ tức thì* (ví dụ: `order-service` cần kiểm tra ví của `customer_id` có đủ tiền không, hay `restaurant_id` có đang mở cửa không), `order-service` sẽ dùng **Spring Cloud OpenFeign** để gửi request HTTP REST API trực tiếp sang dịch vụ đối tác.

**[Slide 3: Bài toán bảo toàn lịch sử giao dịch và Mẫu thiết kế Snapshot (Snapshot Pattern)]**
Thế thì câu hỏi đặt ra là: Nếu chúng ta chỉ lưu mỗi `customer_id`, `restaurant_id` hay `menu_item_id` (Soft Reference), rồi mỗi khi khách xem lại lịch sử đơn hàng cũ lại dùng FeignClient gọi API sang `restaurant-service` lấy tên món và giá tiền về hiển thị thì có ổn không?

**(Nhấn mạnh - Phân tích rủi ro tài chính thực tế)**
Câu trả lời là: **Hoàn toàn không ổn đối với dữ liệu lịch sử tài chính!**
Các bạn hãy hình dung: Hôm nay khách đặt món "Trà sữa trân châu" giá **30.000đ**. Tuần sau, nhà hàng đổi tên món thành "Trà sữa đặc biệt" và tăng giá lên **40.000đ**.
Nếu chúng ta chỉ lưu `menu_item_id` rồi truy vấn động từ `restaurant-service`, hóa đơn cũ của khách hàng sẽ bị hiển thị sai thành 40.000đ. Khách hàng sẽ khiếu nại ngay vì giá trị lịch sử của giao dịch đã bị thay đổi!

Do đó, chúng ta áp dụng giải pháp kết hợp hoàn hảo giữa **Soft Reference** và **Snapshot Pattern**:
- **Soft Reference (Lưu ID):** Định danh đối tác để thực hiện các truy vấn nghiệp vụ tức thời qua FeignClient (kiểm tra số dư, kiểm tra trạng thái mở cửa).
- **Snapshot Pattern (Chụp ảnh dữ liệu):** Tại thời điểm bấm đặt đơn, `order-service` sẽ "chụp ảnh" và lưu cứng các thông tin tài chính/lịch sử (`customer_name`, `merchant_name`, `item_name`, `price`) trực tiếp vào DB của `order-service`. Dù sau này nhà hàng có đổi tên hay tăng giá món ăn bao nhiêu lần, hóa đơn lịch sử trong `order-service` vẫn được bảo toàn chính xác 100%.

---

### 2. Phần thực hành (Demo)

**[Live Demo: Thiết kế cấu trúc thực thể và khai báo Feign Client]**
Bây giờ, thầy sẽ hướng dẫn các bạn quan sát cấu trúc bảng của `order-service` và cách định nghĩa một Feign Client để giao tiếp liên dịch vụ.

*(Thao tác trên IDE/Terminal)*:
- *Bước 1: Quan sát cấu trúc bảng `orders` và `order_items` lưu trữ trường thông tin Snapshot:*
  ```sql
  -- Bảng orders trong quickbite_order_db
  CREATE TABLE orders (
      id BIGSERIAL PRIMARY KEY,
      customer_id BIGINT NOT NULL,          -- Soft Reference sang user-service
      customer_name VARCHAR(100) NOT NULL,  -- Snapshot: Tên khách lúc đặt
      restaurant_id BIGINT NOT NULL,        -- Soft Reference sang restaurant-service
      merchant_name VARCHAR(100) NOT NULL,  -- Snapshot: Tên quán lúc đặt
      total_price DECIMAL(12,2) NOT NULL,
      status VARCHAR(20) NOT NULL
  );

  -- Bảng order_items lưu Snapshot giá món ăn
  CREATE TABLE order_items (
      id BIGSERIAL PRIMARY KEY,
      order_id BIGINT REFERENCES orders(id),
      menu_item_id BIGINT NOT NULL,        -- Soft Reference sang menu_items
      item_name VARCHAR(100) NOT NULL,     -- Snapshot: Tên món lúc đặt
      price DECIMAL(10,2) NOT NULL,        -- Snapshot: Giá món lúc đặt
      quantity INT NOT NULL
  );
  ```
- *Bước 2: Khai báo giao diện FeignClient trong Spring Boot của `order-service` để gọi API lấy thông tin người dùng:*
  ```java
  @FeignClient(name = "user-service", url = "http://user-service:8081")
  public interface UserServiceClient {

      @GetMapping("/api/v1/users/{id}")
      UserDTO getUserById(@PathVariable("id") Long id);
  }
  ```
  *(Giải thích): FeignClient giúp chúng ta gọi API sang `user-service` đơn giản như việc gọi một hàm Java nội bộ mà không cần tự viết mã HTTP client phức tạp.*

---

## Lesson 2: Quy trình chạy Spring Boot Service cùng cơ sở dữ liệu bằng Docker Compose

### 1. Phần lý thuyết

**(Khơi gợi vấn đề thực tế)**
Ở Lesson 1, chúng ta đã hiểu về tư duy thiết kế dữ liệu đa dịch vụ. Bây giờ, bài toán đặt ra cho DevOps và Backend Developer là: Làm thế nào để đóng gói và khởi chạy đồng thời nhiều microservices Java Spring Boot cùng một container PostgreSQL trên môi trường phát triển (Local/Staging)?

Nếu chúng ta gõ lệnh `docker run` thủ công cho từng dịch vụ: `user-service` (Java 17 - Cổng 8081), `restaurant-service` (Java 21 - Cổng 8082)... thì việc quản lý cổng, mạng ảo và biến môi trường sẽ cực kỳ rối rắm và dễ xảy ra xung đột.

**[Slide 4: Sơ đồ tổng quan hệ thống QuickBite & Chuỗi kết nối mạng]**
Để giải quyết vấn đề này, chúng ta kế thừa container PostgreSQL (`quickbite-db`) đã được dựng ngầm từ Session 4. Container database này đã được chạy script `init-db.sql` để tạo sẵn 4 database logic riêng biệt: `quickbite_user_db`, `quickbite_restaurant_db`, `quickbite_order_db`, và `quickbite_notification_db`.

Nhiệm vụ của chúng ta trong file `docker-compose.yml` của dự án microservices là:
1. Đóng gói các dịch vụ backend thành các container độc lập.
2. Cắm tất cả các container backend này vào cùng một mạng ảo ngoài tên là `quickbite-net` (mạng chung chứa container database đang chạy ngầm).

**[Slide 5: Chiến lược quản lý cấu hình bằng tiền tố trong file .env]**
Khi có nhiều dịch vụ cùng chạy trong một file Compose, một sai lầm rất phổ biến của lập trình viên là đặt tên biến môi trường trùng nhau trong file `.env` (ví dụ: `DB_NAME=quickbite_user_db`).
- Nếu đặt trùng tên, biến môi trường của dịch vụ này sẽ ghi đè lên dịch vụ khác.
- Để khắc phục, chúng ta áp dụng quy tắc **Tiền tố hóa (Prefixing)** cho từng dịch vụ: `USER_DB_NAME`, `RESTAURANT_DB_NAME`, `ORDER_DB_NAME`... Sau đó, trong file `docker-compose.yml`, chúng ta ánh xạ các biến này vào đúng biến môi trường mà ứng dụng Spring Boot yêu cầu.

---

### 2. Phần thực hành (Demo)

**[Live Demo: Thao tác cấu hình file .env và docker-compose.yml khởi chạy 2 microservices]**
Thầy sẽ hướng dẫn các bạn từng bước biên soạn file cấu hình để khởi chạy đồng thời `user-service` và `restaurant-service`.

*(Thao tác trên Terminal)*:
- *Bước 1: Chuẩn bị tệp `.env` chứa biến môi trường tiền tố hóa tại thư mục gốc `/quickbite-project/.env`:*
  ```env
  # Database chung
  DB_HOST=quickbite-db
  DB_PORT=5432

  # User Service
  USER_DB_NAME=quickbite_user_db
  USER_DB_USERNAME=quickbite_user
  USER_DB_PASSWORD=quickbite_user
  USER_SERVER_PORT=8081

  # Restaurant Service
  RESTAURANT_DB_NAME=quickbite_restaurant_db
  RESTAURANT_DB_USERNAME=quickbite_restaurant
  RESTAURANT_DB_PASSWORD=quickbite_restaurant
  RESTAURANT_SERVER_PORT=8082
  ```

- *Bước 2: Viết file `/quickbite-project/docker-compose.yml` để khởi chạy đồng thời các service:*
  ```yaml
  version: '3.8'

  services:
    user-service:
      build:
        context: ./user-service
      container_name: user-service
      ports:
        - "${USER_SERVER_PORT}:${USER_SERVER_PORT}"
      environment:
        - DB_HOST=${DB_HOST}
        - DB_PORT=${DB_PORT}
        - DB_NAME=${USER_DB_NAME}
        - DB_USERNAME=${USER_DB_USERNAME}
        - DB_PASSWORD=${USER_DB_PASSWORD}
        - SERVER_PORT=${USER_SERVER_PORT}
      networks:
        - quickbite-net

    restaurant-service:
      build:
        context: ./restaurant-service
      container_name: restaurant-service
      ports:
        - "${RESTAURANT_SERVER_PORT}:${RESTAURANT_SERVER_PORT}"
      environment:
        - DB_HOST=${DB_HOST}
        - DB_PORT=${DB_PORT}
        - DB_NAME=${RESTAURANT_DB_NAME}
        - DB_USERNAME=${RESTAURANT_DB_USERNAME}
        - DB_PASSWORD=${RESTAURANT_DB_PASSWORD}
        - SERVER_PORT=${RESTAURANT_SERVER_PORT}
      networks:
        - quickbite-net

  networks:
    quickbite-net:
      external: true
  ```
  *(Giải thích): Cờ `external: true` giúp Compose hiểu rằng mạng `quickbite-net` đã được tạo từ trước bởi stack database và không tự tạo mạng mới.*

- *Bước 3: Thực hiện build file JAR và khởi chạy cụm dịch vụ:*
  ```bash
  # Biên dịch file JAR ở máy host
  cd user-service && ./gradlew bootJar
  cd ../restaurant-service && ./gradlew bootJar
  cd ..

  # Khởi chạy cụm dịch vụ Compose
  docker compose up -d --build
  ```

- *Bước 4: Kiểm tra log để đảm bảo 2 dịch vụ kết nối database thành công:*
  ```bash
  docker compose logs -f user-service restaurant-service
  ```

---

## Lesson 3: Giao tiếp liên dịch vụ qua Docker Network trong kiến trúc phân tán

### 1. Phần lý thuyết

**(Mở đầu - Đặt vấn đề rủi ro kết nối)**
Ở Lesson 2, cả hai dịch vụ `user-service` và `restaurant-service` đã khởi chạy thành công. Bây giờ, giả sử `user-service` cần gửi một HTTP request sang `restaurant-service` để kiểm tra trạng thái đóng/mở cửa của quán trước khi duyệt tài khoản.

Thế thì câu hỏi đặt ra là: Trong file cấu hình của `user-service`, chúng ta phải điền URL kết nối tới `restaurant-service` như thế nào?

**[Slide 7: Sai lầm khi dùng localhost & Cơ chế Service Discovery qua DNS Docker]**
Nhiều bạn theo thói quen sẽ điền `http://localhost:8082`. 
**(Nhấn mạnh)**: Đây là một sai lầm rất kinh điển! Ký tự `localhost` bên trong một container chỉ trỏ về **chính không gian cô lập của container đó**, hoàn toàn không trỏ sang container khác hay máy host vật lý. Request gọi tới `localhost:8082` sẽ thất bại ngay lập tức.

Thế còn việc dùng địa chỉ IP nội bộ của container (ví dụ `172.18.0.4`) thì sao? 
IP của container mang tính chất động (ephemeral). Mỗi lần các bạn restart container hoặc build lại code, Docker Engine sẽ thu hồi IP cũ và cấp IP mới. Viết cứng IP trong cấu hình sẽ làm sập hệ thống mỗi khi cập nhật.

Giải pháp chuẩn xác ở đây là dựa vào cơ chế **Service Discovery** (Phát hiện dịch vụ tự động) của Docker Network:
- Khi các container tham gia vào cùng một mạng ảo (User-defined Bridge Network), Docker bật sẵn một trình DNS Server nội bộ.
- Các container có thể gọi nhau bằng **Tên container/dịch vụ** (ví dụ: `http://restaurant-service:8082`).
- Trình DNS nội bộ sẽ tự động dịch tên `restaurant-service` thành IP thực tế hiện tại của container đó.

**[Slide 8: Chiến lược bảo mật cổng mạng (Ports vs Expose)]**
Khi các container đã có thể giao tiếp nội bộ thông qua tên miền DNS, việc mở tất cả các cổng (`8081`, `8082`) ra ngoài máy host vật lý bằng thuộc tính `ports` sẽ tạo ra kẽ hở bảo mật lớn (kẻ xấu có thể quét cổng và truy cập trực tiếp vào backend nội bộ).

Vì vậy, chúng ta áp dụng nguyên tắc đóng kín an ninh:
- Bỏ thuộc tính `ports` đối với các dịch vụ nội bộ.
- Thay bằng `expose: - "8082"`. Cấu hình này cho phép cổng `8082` thông suốt bên trong mạng ảo Compose cho các container gọi nhau, nhưng hoàn toàn ngăn chặn truy cập từ internet ngoài máy host.

---

### 2. Phần thực hành (Demo)

**[Live Demo: Đóng cổng mạng nội bộ và kiểm chứng DNS bằng lệnh Ping]**
Thầy sẽ hướng dẫn các bạn cập nhật file `docker-compose.yml` để đóng port `restaurant-service` và thực hiện kiểm chứng phân giải DNS nội bộ.

*(Thao tác trên Terminal)*:
- *Bước 1: Cập nhật tệp `docker-compose.yml` đóng port ra ngoài host cho `restaurant-service`:*
  ```yaml
  restaurant-service:
    build:
      context: ./restaurant-service
    container_name: restaurant-service
    # Thay 'ports' bằng 'expose' để bảo mật cổng nội bộ
    expose:
      - "8082"
    environment:
      - DB_HOST=${DB_HOST}
      - DB_PORT=${DB_PORT}
      - DB_NAME=${RESTAURANT_DB_NAME}
      - DB_USERNAME=${RESTAURANT_DB_USERNAME}
      - DB_PASSWORD=${RESTAURANT_DB_PASSWORD}
      - SERVER_PORT=${RESTAURANT_SERVER_PORT}
    networks:
      - quickbite-net
  ```

- *Bước 2: Tải lại cấu hình Compose:*
  ```bash
  docker compose up -d
  ```

- *Bước 3: Thực thi lệnh ping kiểm tra phân giải DNS từ `user-service` sang `restaurant-service`:*
  ```bash
  docker compose exec user-service ping -c 3 restaurant-service
  ```
  *(Kết quả mong đợi): Terminal hiển thị gói tin ping gửi thành công tới địa chỉ IP ảo (ví dụ: `PING restaurant-service (172.18.0.4)`), chứng minh DNS nội bộ của Docker đã phân giải tên container thành công.*

---

## Lesson 4: API Gateway và điểm truy cập tập trung trong hệ thống Microservices

### 1. Phần lý thuyết

**(Khơi gợi bối cảnh thực tế)**
Chúng ta đã khởi chạy 4 microservices backend cho QuickBite: `user-service` (8081), `restaurant-service` (8082), `order-service` (8083), và `notification-service` (8084).

Bây giờ, nhóm phát triển ứng dụng di động (Frontend) bắt đầu tích hợp API. Họ lập tức tìm gặp đội Backend và phàn nàn về 3 "nỗi đau" cực kỳ nhức nhối:

**[Slide 10: Ba nỗi đau khi cho Client gọi trực tiếp Microservices]**
1. **Ma trận quản lý Endpoint:** Để hiển thị một màn hình đặt hàng, ứng dụng Mobile phải gọi tới 3 URL và port khác nhau: `http://localhost:8081/users/me` lấy thông tin ví, `http://localhost:8082/restaurants/active` lấy món ăn, và `http://localhost:8083/orders` để tạo đơn. Phía Client phải quản lý quá nhiều port. Khi đổi môi trường Production, việc thay đổi hàng loạt domain/IP trở thành cơn ác mộng.
2. **Cơn ác mộng CORS (Cross-Origin Resource Sharing):** Trình duyệt web chặn các request gọi chéo cổng. Đội backend phải vào từng microservice trong số 4 dịch vụ để thêm cấu hình CORS. Chỉ cần sót 1 service, tính năng sẽ bị lỗi ngay trên máy khách hàng.
3. **Trùng lặp mã nguồn Bảo mật (Auth Logic):** Logic xác thực Token JWT và phân quyền người dùng buộc phải viết đi viết lại ở tất cả 4 microservices. Khi Tech Lead muốn đổi thuật toán giải mã Token, cả đội phải sửa code, build và deploy lại toàn bộ các microservices.

**[Slide 11: Định nghĩa API Gateway & Các chức năng cốt lõi]**
Để giải quyết triệt để 3 nỗi đau trên, kiến trúc Microservices hiện đại đưa vào một thành phần cửa ngõ gọi là **API Gateway**.

**API Gateway** là một máy chủ đóng vai trò là **điểm truy cập tập trung duy nhất (Single Entry Point)** cho mọi yêu cầu API từ Client đi vào hệ thống. Client từ nay chỉ cần biết duy nhất một địa chỉ gốc (ví dụ cổng `8080` của Gateway).

Các chức năng cốt lõi của API Gateway bao gồm:
- **Định tuyến động (Dynamic Routing):** Nhận request từ Client, kiểm tra đường dẫn URI và điều hướng gói tin ngầm tới đúng microservice phía sau.
- **Bảo mật tập trung (Centralized Auth & CORS):** Giải mã Token JWT và xử lý CORS ngay tại cửa ngõ. Request không hợp lệ sẽ bị chặn đứng ngay lập tức với mã lỗi `401 Unauthorized`, không cho phép đi sâu vào làm phiền các backend service.
- **Lá chắn an toàn (Security Shield):** Cho phép đóng kín hoàn toàn cổng của tất cả các microservices nội bộ (`8081`, `8082`, `8083`), chỉ mở duy nhất cổng `8080` của API Gateway ra bên ngoài internet.

**[Slide 12: Phân biệt API Gateway và Reverse Proxy Nginx]**
Thầy muốn lưu ý một câu hỏi phỏng vấn rất hay gặp: "Nginx và Spring Cloud Gateway khác nhau thế nào?"
- **Nginx:** Là Reverse Proxy cấp hạ tầng (TCP/HTTP thô), chạy bằng C, tối ưu cực cao cho việc phân phối file tĩnh Frontend, cấu hình SSL/HTTPS và nén dữ liệu.
- **Spring Cloud Gateway:** Là API Gateway cấp ứng dụng viết bằng Java, can thiệp sâu vào logic ứng dụng. Nó tích hợp chặt chẽ với hệ sinh thái Spring, cho phép viết các bộ lọc Filter bằng Java để soi xét Header, Body request và phân quyền nghiệp vụ phức tạp.
- **Mô hình chuẩn doanh nghiệp:** Nginx đứng ngoài cùng đón HTTPS -> chuyển tiếp API vào cho Spring Cloud Gateway xử lý logic định tuyến và bảo mật.

---

### 2. Phần thực hành (Demo)

**[Live Demo: Sơ đồ điều hướng và thiết lập kiến trúc TARGET cho QuickBite]**
Bây giờ, chúng ta sẽ xem xét đặc tả kiến trúc mục tiêu (STATE 3) của QuickBite khi có API Gateway.

```text
                            ┌────────────────────────┐
                            │   Client App / Mobile  │
                            └───────────┬────────────┘
                                        │ (Gọi cổng duy nhất: 8080)
                                        ▼
                            ┌────────────────────────┐
                            │  Spring Cloud Gateway  │
                            └───────────┬────────────┘
                                        │ (Điều hướng mạng nội bộ)
           ┌────────────────────────────┼────────────────────────────┐
           ▼ (/api/v1/users/**)         ▼ (/api/v1/restaurants/**)   ▼ (/api/v1/orders/**)
   ┌───────────────┐            ┌───────────────┐            ┌───────────────┐
   │ user-service  │            │restaurant-svc │            │ order-service │
   │  (Port 8081)  │            │  (Port 8082)  │            │  (Port 8083)  │
   └───────────────┘            └───────────────┘            └───────────────┘
```

*Quy tắc định tuyến ngầm:*
- Client gọi `http://localhost:8080/api/v1/users/1` ──► Gateway chuyển tiếp ngầm tới `http://user-service:8081/api/v1/users/1`.
- Client gọi `http://localhost:8080/api/v1/restaurants/active` ──► Gateway chuyển tiếp ngầm tới `http://restaurant-service:8082/api/v1/restaurants/active`.

---

## Lesson 5: Spring Cloud Gateway và định tuyến yêu cầu trong kiến trúc đa dịch vụ

### 1. Phần lý thuyết

**(Đặt vấn đề)**
Ở Lesson 4, chúng ta đã nắm được lý thuyết về API Gateway. Bây giờ, làm thế nào để cấu hình dịch vụ **Spring Cloud Gateway** trong dự án Java Spring Boot?

Nếu tự viết mã Java Controller để nhận và chuyển tiếp request thủ công, chúng ta sẽ dính phải nỗi đau lặp code. Spring Cloud Gateway giải quyết vấn đề này bằng cơ chế cấu hình **Declarative (Khai báo YAML)** cực kỳ ngắn gọn và mạnh mẽ.

**[Slide 13: Ba thành phần cốt lõi của một Route trong Spring Cloud Gateway]**
Mỗi tuyến đường (Route) trong Spring Cloud Gateway được cấu hình dựa trên 3 thành phần chính:
1. **Route ID:** Định danh duy nhất cho tuyến đường (ví dụ: `user_route`).
2. **URI:** Địa chỉ của microservice đích nhận gói tin (ví dụ: `http://user-service:8081`).
3. **Predicates (Điều kiện khớp):** Tập hợp các hàm điều kiện logic để Gateway quyết định request nào thuộc về Route này. Phổ biến nhất là `- Path=/api/v1/users/**` (mọi đường dẫn bắt đầu bằng `/api/v1/users/`).
4. **Filters (Bộ lọc):** Cho phép can thiệp chỉnh sửa request/response trước và sau khi gửi tới microservice (ví dụ: thêm Header, cắt bớt tiền tố URI `StripPrefix`).

**[Slide 15: Cảnh báo rủi ro về Blocking IO trong WebFlux]**
**(Nhấn mạnh - Kiến thức chuyên sâu)**
Spring Cloud Gateway được phát triển trên nền tảng **Spring WebFlux** và máy chủ không đồng bộ **Netty** (Non-blocking IO).
- Netty chỉ dùng một số lượng rất ít luồng (Event Loop Threads) để xử lý đồng thời hàng chục ngàn request.
- Nếu các bạn tự viết Filter tùy biến mà vô tình chèn các lệnh gây khóa luồng (Blocking IO) như `Thread.sleep()` hoặc gọi các thư viện JDBC/Hibernate truyền thống, luồng Event Loop sẽ bị khóa cứng.
- Hậu quả là toàn bộ API Gateway sẽ bị ngưng trệ, khiến **toàn bộ hệ thống sập hoàn toàn**! Do đó, mọi xử lý tùy biến trên Gateway bắt buộc phải tuân thủ triết lý Lập trình Phản ứng (Reactive Non-blocking).

---

### 2. Phần thực hành (Demo)

**[Live Demo: Biên soạn tệp application.yml cho Gateway và tích hợp Docker Compose 5 Services]**
Bây giờ, chúng ta sẽ thực hành cấu hình file `application.yml` cho `gateway-service` và cập nhật file `docker-compose.yml` để hoàn thiện cụm 5 container microservices.

*(Thao tác trên IDE/Terminal)*:
- *Bước 1: Tạo file cấu hình `application.yml` cho `gateway-service`:*
  ```yaml
  server:
    port: 8080

  spring:
    cloud:
      gateway:
        routes:
          - id: user_route
            uri: http://${USER_SVC_HOST:localhost}:${USER_SVC_PORT:8081}
            predicates:
              - Path=/api/v1/users/**

          - id: restaurant_route
            uri: http://${RESTAURANT_SVC_HOST:localhost}:${RESTAURANT_SVC_PORT:8082}
            predicates:
              - Path=/api/v1/restaurants/**

          - id: order_route
            uri: http://${ORDER_SVC_HOST:localhost}:${ORDER_SVC_PORT:8083}
            predicates:
              - Path=/api/v1/orders/**

          - id: notification_route
            uri: http://${NOTIFICATION_SVC_HOST:localhost}:${NOTIFICATION_SVC_PORT:8084}
            predicates:
              - Path=/api/v1/notifications/**
  ```

- *Bước 2: Cập nhật tệp `.env` bổ sung cấu hình host cho Gateway:*
  ```env
  GATEWAY_SERVER_PORT=8080
  USER_SVC_HOST=user-service
  USER_SVC_PORT=8081
  RESTAURANT_SVC_HOST=restaurant-service
  RESTAURANT_SVC_PORT=8082
  ORDER_SVC_HOST=order-service
  ORDER_SVC_PORT=8083
  NOTIFICATION_SVC_HOST=notification-service
  NOTIFICATION_SVC_PORT=8084
  ```

- *Bước 3: Cập nhật `docker-compose.yml` hoàn chỉnh đóng port 4 microservices và chỉ mở cổng 8080 cho Gateway:*
  ```yaml
  version: '3.8'

  services:
    user-service:
      build: ./user-service
      container_name: user-service
      expose: ["8081"]
      environment:
        - DB_HOST=${DB_HOST}
        - DB_PORT=${DB_PORT}
        - DB_NAME=${USER_DB_NAME}
        - DB_USERNAME=${USER_DB_USERNAME}
        - DB_PASSWORD=${USER_DB_PASSWORD}
        - SERVER_PORT=${USER_SERVER_PORT}
      networks: [quickbite-net]

    restaurant-service:
      build: ./restaurant-service
      container_name: restaurant-service
      expose: ["8082"]
      environment:
        - DB_HOST=${DB_HOST}
        - DB_PORT=${DB_PORT}
        - DB_NAME=${RESTAURANT_DB_NAME}
        - DB_USERNAME=${RESTAURANT_DB_USERNAME}
        - DB_PASSWORD=${RESTAURANT_DB_PASSWORD}
        - SERVER_PORT=${RESTAURANT_SERVER_PORT}
      networks: [quickbite-net]

    order-service:
      build: ./order-service
      container_name: order-service
      expose: ["8083"]
      environment:
        - DB_HOST=${DB_HOST}
        - DB_PORT=${DB_PORT}
        - DB_NAME=${ORDER_DB_NAME}
        - DB_USERNAME=${ORDER_DB_USERNAME}
        - DB_PASSWORD=${ORDER_DB_PASSWORD}
        - SERVER_PORT=${ORDER_SERVER_PORT}
      networks: [quickbite-net]

    notification-service:
      build: ./notification-service
      container_name: notification-service
      expose: ["8084"]
      environment:
        - DB_HOST=${DB_HOST}
        - DB_PORT=${DB_PORT}
        - DB_NAME=${NOTIFICATION_DB_NAME}
        - DB_USERNAME=${NOTIFICATION_DB_USERNAME}
        - DB_PASSWORD=${NOTIFICATION_DB_PASSWORD}
        - SERVER_PORT=${NOTIFICATION_SERVER_PORT}
      networks: [quickbite-net]

    gateway-service:
      build: ./gateway-service
      container_name: gateway-service
      ports:
        - "${GATEWAY_SERVER_PORT}:${GATEWAY_SERVER_PORT}"
      environment:
        - USER_SVC_HOST=${USER_SVC_HOST}
        - USER_SVC_PORT=${USER_SVC_PORT}
        - RESTAURANT_SVC_HOST=${RESTAURANT_SVC_HOST}
        - RESTAURANT_SVC_PORT=${RESTAURANT_SVC_PORT}
        - ORDER_SVC_HOST=${ORDER_SVC_HOST}
        - ORDER_SVC_PORT=${ORDER_SVC_PORT}
        - NOTIFICATION_SVC_HOST=${NOTIFICATION_SVC_HOST}
        - NOTIFICATION_SVC_PORT=${NOTIFICATION_SVC_PORT}
      networks: [quickbite-net]

  networks:
    quickbite-net:
      external: true
  ```

- *Bước 4: Khởi chạy cụm 5 dịch vụ và kiểm thử API đi qua Gateway:*
  ```bash
  docker compose up -d --build
  
  # Gọi API thử nghiệm qua cổng 8080 của Gateway
  curl http://localhost:8080/api/v1/users/1
  ```

---

## Lesson 6: Luồng yêu cầu đặt hàng end-to-end từ Client qua API Gateway

### 1. Phần lý thuyết

**(Mở đầu - Đặt vấn đề thất thoát tài chính)**
Bây giờ, chúng ta hãy cùng nhau phân tích một bài toán thực tế vô cùng quan trọng: Luồng chạy nghiệp vụ đặt hàng (Order Flow) đi qua toàn bộ 5 container trong hệ thống QuickBite sẽ diễn ra như thế nào? Và điều gì sẽ xảy ra khi một bước ở giữa bị lỗi?

Trong ứng dụng Monolith, việc đảm bảo tính nhất quán dữ liệu rất đơn giản nhờ annotation `@Transactional`. Nếu bước trừ tiền thành công nhưng bước gán tài xế bị lỗi, cơ sở dữ liệu sẽ tự động rollback toàn bộ.

Tuy nhiên, trong Microservices, dữ liệu ví nằm ở `quickbite_user_db`, còn đơn hàng nằm ở `quickbite_order_db`. Không có cơ chế rollback tự động nào của database có thể cứu được khi lỗi xảy ra xuyên suốt nhiều container!

**[Slide 16: Vòng đời 6 bước Runtime của một yêu cầu Đặt món]**
Dưới đây là kịch bản tích hợp dòng dữ liệu chạy thực tế qua cụm container QuickBite:
1. **Bước 1 (Tiếp nhận):** Khách hàng bấm Đặt đơn. Request `POST /api/v1/orders` gửi tới cổng `8080` của API Gateway. Gateway điều hướng gói tin tới `order-service` (cổng 8083).
2. **Bước 2 (Tạo đơn tạm):** `order-service` tạo đơn hàng trong DB `quickbite_order_db` với trạng thái `PENDING`.
3. **Bước 3 (Thanh toán ví):** `order-service` gọi FeignClient sang `user-service` (8081) để trừ tiền ví. `user-service` trừ tiền trong DB `quickbite_user_db`.
4. **Bước 4 (Chuẩn bị món):** `order-service` gọi sang `restaurant-service` (8082) để báo quán làm món.
   - *Nếu quán đồng ý:* Chuyển sang Bước 5.
   - *Nếu quán từ chối (hết món):* Kích hoạt **Giao dịch bù (Compensating Transaction)**: `order-service` gọi lại API của `user-service` để **cộng lại tiền vào ví cho khách**, rồi chuyển trạng thái đơn thành `CANCELLED`.
5. **Bước 5 (Điều phối tài xế):** `order-service` gán tài xế và đổi trạng thái đơn thành `SHIPPING`.
6. **Bước 6 (Thông báo):** `order-service` gửi request sang `notification-service` (8084) để gửi tin nhắn/email thông báo cho khách hàng.

**[Slide 17: Saga Pattern & Tính nhất quán cuối cùng (Eventual Consistency)]**
Mô hình xử lý lỗi phân tán bằng cách gọi mã lệnh hoàn tác ở trên được gọi là **Saga Pattern (Orchestration)**.
- Chúng ta chấp nhận từ bỏ tính nhất quán mạnh (Strong Consistency) để đổi lấy hiệu năng cao cho hệ thống.
- Thay vào đó, hệ thống áp dụng **Tính nhất quán cuối cùng (Eventual Consistency)**: Dữ liệu giữa các dịch vụ có thể lệch nhau trong vài giây ngắn ngủi, nhưng cuối cùng thông qua mã bù (Compensating Code), mọi thứ sẽ trở về trạng thái chính xác và đồng bộ.

---

### 2. Phần thực hành (Demo)

**[Live Demo: Truy vết luồng log đan xen (Interlaced Logs) của cụm 5 Services]**
Cuối cùng, thầy sẽ hướng dẫn các bạn cách theo dõi và chẩn đoán luồng chạy thực tế của cả cụm 5 container bằng câu lệnh stream log của Docker Compose.

*(Thao tác trên Terminal)*:
- *Bước 1: Chạy lệnh theo dõi log đan xen toàn cụm dịch vụ:*
  ```bash
  docker compose logs -f
  ```

- *Bước 2: Phân tích luồng log in ra màn hình khi thực hiện thành công một đơn hàng:*
  ```text
  gateway-service-1        | [Gateway] Chuyển tiếp request POST /api/v1/orders tới order-service:8083
  order-service-1          | [Order] Nhận yêu cầu tạo đơn từ Customer ID: 1, Restaurant ID: 2.
  order-service-1          | [Order] Lưu đơn hàng ID: 99 thành công vào DB (Trạng thái: PENDING).
  order-service-1          | [Order] Đang gửi yêu cầu trừ tiền ví (100.000đ) sang user-service...
  user-service-1           | [User] Nhận yêu cầu trừ tiền ví cho User ID: 1. Số dư hiện tại: 150.000đ.
  user-service-1           | [User] Trừ tiền thành công. Số dư mới: 50.000đ.
  order-service-1          | [Order] Kết quả thanh toán: THÀNH CÔNG. Cập nhật đơn thành ACCEPTED.
  order-service-1          | [Order] Đang thông báo thực đơn sang restaurant-service...
  restaurant-service-1     | [Restaurant] Nhận yêu cầu chuẩn bị đơn hàng ID: 99.
  restaurant-service-1     | [Restaurant] Merchant chấp nhận đơn hàng.
  order-service-1          | [Order] Nhà hàng đã chấp nhận. Tiến hành tìm tài xế giao hàng...
  order-service-1          | [Order] Đã gán tài xế ID: 5. Cập nhật trạng thái đơn thành: SHIPPING.
  order-service-1          | [Order] Gửi sự kiện trạng thái đơn hàng sang notification-service...
  notification-service-1   | [Notification] Nhận yêu cầu thông báo đơn hàng ID: 99.
  notification-service-1   | [Notification] Gửi thông báo thành công (Trạng thái: SENT).
  ```

Đó là toàn bộ kiến thức và quy trình xây dựng, cấu hình và vận hành hệ thống Microservices đa dịch vụ kết hợp API Gateway trong Session 05. Cảm ơn các bạn đã lắng nghe và hẹn gặp lại các bạn ở Session tiếp theo!
