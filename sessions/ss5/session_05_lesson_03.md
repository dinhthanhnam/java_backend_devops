# SESSION 05: MULTI-SERVICES & API GATEWAY

## LESSON 03: Giao tiếp liên dịch vụ qua Docker Network trong kiến trúc phân tán

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Giải thích** được cơ chế phân giải tên miền tự động (Service Discovery) qua DNS server nội bộ của Docker Network.
* **Hiểu bản chất** của mạng mặc định (Default Network) được tạo tự động bởi Docker Compose.
* **Cấu hình** cấu trúc mã nguồn giao tiếp đồng bộ REST API giữa các Spring Boot container bằng **Spring Cloud OpenFeign**.
* **Đóng cổng mạng** của các service nội bộ để bảo mật hệ thống, chỉ cho phép giao tiếp nội bộ trong mạng Compose.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (KẾT NỐI MẠNG GIỮA CÁC SERVICE TRÊN STAGING/PRODUCTION)

Khi chạy hệ thống QuickBite ở Lesson 2, cả hai dịch vụ `user-service` và `restaurant-service` đều hoạt động độc lập và chỉ kết nối tới cơ sở dữ liệu. Tuy nhiên, trong thực tế, các dịch vụ này cần gọi lẫn nhau để kiểm tra thông tin và xử lý nghiệp vụ:
* Ví dụ: Khi khách hàng đặt đơn hàng hoặc thực hiện giao dịch, hệ thống cần gửi request kiểm tra trạng thái hoạt động của nhà hàng hoặc xác thực thông tin tài khoản người dùng.

Để hai dịch vụ container có thể giao tiếp với nhau qua giao thức HTTP, chúng ta cần giải quyết hai vấn đề:
1. **Quản lý địa chỉ IP động:** Khi một container khởi động lại hoặc được cập nhật code, Docker Engine sẽ thu hồi IP cũ và cấp cho nó một địa chỉ IP mới. Nếu viết cứng IP nội bộ của container này vào cấu hình của container kia, kết nối sẽ bị gián đoạn mỗi khi restart hệ thống.
2. **Bảo mật cổng mạng (Ports Exposing):** Không nên mở toàn bộ các cổng mạng (`8081`, `8082`) ra ngoài máy host vật lý. Việc mở cổng tự do sẽ tăng rủi ro bảo mật (tin tặc có thể quét cổng và gọi trực tiếp tới API nội bộ). Chúng ta chỉ nên expose các cổng này trong mạng ảo nội bộ để các container gọi nhau, và đóng hoàn toàn truy cập từ internet.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (DOCKER COMPOSE DEFAULT NETWORK & OPENFEIGN)

#### 3.1 Mạng mặc định của Docker Compose (Default Network)
Khi khởi chạy một file `docker-compose.yml`, Docker Compose không sử dụng mạng bridge mặc định của Docker CLI. Thay vào đó, nó sẽ **tự động tạo ra một mạng ảo riêng biệt** dành cho toàn bộ các dịch vụ được định nghĩa trong file đó.
* Mạng này tương đương với một **User-defined Bridge Network**.
* Mạng này được kích hoạt sẵn bộ phân giải tên miền nội bộ (**Embedded DNS Server** chạy tại IP ảo `127.0.0.11`).
* Nhờ có DNS Server nội bộ, tất cả các container trong cùng file compose có thể tự phân giải tên gọi của nhau thông qua thuộc tính `container_name` hoặc tên `service` (ví dụ: gọi tới `http://quickbite-restaurant:8082`).

```text
  [ user-service container ]
             │
         Gửi request tới: http://quickbite-restaurant:8082
             │
             ▼
    [ Embedded DNS Server ]  ── Phân giải tên miền ──► IP ảo hiện tại: 172.20.0.3
             │
             └───────────────────── Gửi gói tin HTTP ─────────────────────┐
                                                                          ▼
                                                            [ restaurant-svc container ]
```

#### 3.2 Khái niệm Spring Cloud OpenFeign
Trong Spring Boot, thay vì sử dụng RestTemplate viết code thủ công, lập trình viên thường dùng **OpenFeign** để thực hiện các cuộc gọi REST API. OpenFeign giúp lập trình viên viết REST client dưới dạng các Interface rất trực quan. Cú pháp gọi sẽ sử dụng trực tiếp tên container làm tên host trong URL.

---

### PHẦN 4. HƯỚNG DẪN THỰC HÀNH CẤU HÌNH VÀ KIỂM TRA

#### 4.1 Khai báo mã giả Feign Client gọi liên dịch vụ
Dưới đây là mã giả (pseudocode) mô tả cách cấu hình Feign Client trong `user-service` để gọi sang API của `restaurant-service` lấy trạng thái đóng/mở cửa của nhà hàng:

```java
// RestaurantClient.java
package com.quickbite.user.client;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

// "quickbite-restaurant" là tên container/service trong file docker-compose.yml
@FeignClient(name = "quickbite-restaurant", url = "http://quickbite-restaurant:8082")
public interface RestaurantClient {

    @GetMapping("/restaurants/{id}/status")
    boolean checkRestaurantStatus(@PathVariable("id") Long id);
}
```

#### 4.2 Cập nhật docker-compose.yml đóng cổng mạng ra ngoài
Vì các container đã giao tiếp nội bộ trong mạng ảo mặc định, dịch vụ `restaurant-service` không cần mở cổng `8082` ra máy host vật lý nữa. Chúng ta sẽ loại bỏ từ khóa `ports` và chỉ sử dụng `expose` (hoặc để trống, vì mặc định các cổng bên trong mạng Compose mặc định đều thông suốt với nhau):

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
      - quickbite-db-data:/var/lib/postgresql/data

  quickbite-user:
    build:
      context: ./user-service
    container_name: quickbite-user
    # Mở port 8081 ra ngoài host để Client/Browser có thể gọi vào kiểm thử
    ports:
      - "8081:8081"
    environment:
      - DB_HOST=quickbite-db
      - DB_PORT=5432
      - DB_NAME=quickbite_user_db
      - DB_USERNAME=quickbite_user
      - DB_PASSWORD=quickbite_user
    depends_on:
      - quickbite-db

  quickbite-restaurant:
    build:
      context: ./restaurant-service
    container_name: quickbite-restaurant
    # KHÔNG dùng ports mở ra host nữa, chỉ expose nội bộ trong mạng Compose
    expose:
      - "8082"
    environment:
      - DB_HOST=quickbite-db
      - DB_PORT=5432
      - DB_NAME=quickbite_restaurant_db
      - DB_USERNAME=quickbite_restaurant
      - DB_PASSWORD=quickbite_restaurant
    depends_on:
      - quickbite-db

volumes:
  quickbite-db-data:
```

#### 4.3 Thực hiện kiểm chứng phân giải tên miền
1. Khởi chạy hệ thống Compose:
   ```bash
   docker compose up -d
   ```
2. Thực thi lệnh ping từ container `quickbite-user` sang container `quickbite-restaurant` bằng chính tên container để kiểm tra DNS hoạt động:
   ```bash
   docker compose exec quickbite-user ping -c 3 quickbite-restaurant
   ```
3. **Kết quả mong đợi:** Lệnh ping thực hiện thành công, hiển thị rõ DNS nội bộ đã phân giải tên miền `quickbite-restaurant` thành IP ảo của nó trong mạng (ví dụ: `172.18.0.4`).

---

### PHẦN 5. HIỂU LẦM THƯỜNG GẶP (localhost TRONG CONTAINER)

* **Hiểu lầm thường gặp:** Khi hai container chạy chung trên một máy chủ vật lý, chúng ta có thể cấu hình URL kết nối là `http://localhost:8082` để gọi sang nhau.
* **Sự thật:** Không được. Từ khóa `localhost` viết bên trong mã nguồn chạy trong container sẽ trỏ thẳng về **chính không gian cô lập của container đó**, không phải máy host vật lý. Để kết nối, ta bắt buộc phải sử dụng tên của container đích (`quickbite-restaurant`) làm host trong URL kết nối nhờ cơ chế DNS nội bộ của Docker Network.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG (XÁC MINH KIẾN THỨC)

1. **Mạng mặc định trong Docker Compose:**
   * [Docker Compose default network - Docker Docs](https://docs.docker.com/compose/networking/)
2. **Cách phân giải DNS nội bộ của Docker:**
   * [Docker DNS Service Discovery - Docker Docs](https://docs.docker.com/network/#user-defined-bridge-networks)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao khi chạy các container đơn lẻ bằng lệnh `docker run` thông thường mà không khai báo mạng cụ thể, chúng không thể gọi nhau bằng tên container, nhưng khi khởi chạy bằng tệp Compose thì chúng lại gọi nhau được?
* *Gợi ý:* Lệnh `docker run` thông thường đưa các container vào mạng `default bridge` của Docker CLI, nơi DNS nội bộ bị tắt và không hỗ trợ phân giải tên container. Docker Compose tự động tạo một mạng ảo riêng biệt (tương đương User-defined Bridge Network) cho cụm dịch vụ, nơi DNS được kích hoạt sẵn để tự động phân giải tên container.

#### Câu 2 (Xử lý tình huống)
Nếu bạn thay đổi thuộc tính `container_name: quickbite-restaurant` thành `container_name: quickbite-restaurant-v2` trong file compose, bạn có cần cập nhật cấu hình URL kết nối Feign Client ở các dịch vụ khác gọi tới nó hay không?
* *Gợi ý:* Có. Bởi vì DNS nội bộ phân giải tên miền dựa trên chính tên container được định nghĩa. Khi đổi tên container, các cuộc gọi cũ đến tên cũ sẽ gặp lỗi không tìm thấy host (`UnknownHostException`).
