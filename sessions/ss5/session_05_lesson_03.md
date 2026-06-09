# SESSION 05: MULTI-SERVICES & API GATEWAY

## LESSON 03: Giao tiếp liên dịch vụ qua Docker Network trong kiến trúc phân tán

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Giải thích** được cơ chế phân giải tên miền tự động (Service Discovery) qua DNS server nội bộ của Docker Network.
* **Cấu hình và triển khai** mã nguồn giao tiếp đồng bộ REST API giữa các Spring Boot container bằng `RestTemplate`.
* **Kiểm chứng** sự hoạt động ổn định của kết nối liên container thông qua việc thay đổi IP của container đích nhưng giữ nguyên tên miền gọi.
* **Phân tích** các rủi ro bảo mật và hiệu năng khi cho phép các dịch vụ gọi nhau trực tiếp qua địa chỉ IP máy host thay vì mạng nội bộ.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (NỖI ĐAU CỦA VIỆC PHỤ THUỘC IP VÀ MỞ PORT TỰ DO)

Trong luồng nghiệp vụ của QuickBite, khi khách hàng đặt đơn hàng mới, `order-service` (cổng 8083) không tự ý tạo đơn mà bắt buộc phải thực hiện 2 nhiệm vụ xác thực:
1. Gọi sang `user-service` (cổng 8081) để kiểm tra xem ví tiền khách hàng có đủ số dư thanh toán hay không.
2. Gọi sang `restaurant-service` (cổng 8082) để kiểm tra xem nhà hàng đó đang mở cửa hay đóng cửa, và các món ăn trong đơn hàng có đúng giá không.

Nếu triển khai các container này chạy độc lập, lập trình viên thường đối mặt với hai vấn đề lớn:

1. **IP Drift (Sự biến động địa chỉ IP):**
   * *Nỗi đau:* Lập trình viên lấy IP nội bộ của container `restaurant-service` (ví dụ: `http://172.20.0.5:8082`) và điền trực tiếp vào cấu hình của `order-service`. Khi container `restaurant-service` được cập nhật code mới và restart, Docker Engine sẽ thu hồi IP cũ và cấp một IP mới (ví dụ: `172.20.0.9`). Dịch vụ đặt đơn sẽ ngay lập tức bị sập hoàn toàn do lỗi kết nối timeout.
2. **Mở cổng hệ thống quá đà (Over-exposing ports):**
   * *Nỗi đau:* Để các dịch vụ gọi nhau qua IP máy host, lập trình viên cấu hình map tất cả các cổng `8081`, `8082`, `8083`, `8084` ra ngoài máy host. Điều này tạo điều kiện cho các kẻ tấn công mạng có thể bypass (vượt qua) các lớp bảo vệ để chọc phá hoặc giả lập request gọi trực tiếp tới các API thanh toán của `user-service` từ internet.

*Giải pháp chuẩn chỉnh của DevOps là gom tất cả các container này vào chung một mạng Bridge tùy biến của Docker, cấu hình cho chúng gọi nhau qua tên miền nội bộ và đóng các cổng dịch vụ không cần thiết ra ngoài máy host.*

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (DOCKER DNS VÀ GIAO TIẾP LIÊN CONTAINER)

#### 3.1 Cơ chế Docker Embedded DNS Server
Khi chúng ta khởi chạy cụm container tham gia vào một **User-defined Bridge Network** (Mạng cầu tự định nghĩa), Docker Engine sẽ kích hoạt một máy chủ DNS nội bộ chạy ngầm tại địa chỉ IP đặc biệt `127.0.0.11`.
* Khi container `quickbite-order` gửi request tới địa chỉ `http://quickbite-restaurant:8082`, DNS Server nội bộ sẽ tự động tra cứu bảng ánh xạ tên container sang dải IP hiện hành của nó.
* Nhờ đó, bất kể IP của container đích có bị thay đổi sau mỗi lần restart, DNS nội bộ sẽ tự cập nhật bản ghi để định tuyến request đi đúng hướng một cách trong suốt đối với mã nguồn Java.

```text
 [ order-service container ]
          │
      Gửi request tới: http://quickbite-restaurant:8082
          │
          ▼
 [ Docker DNS (127.0.0.11) ] ── Phân giải tên miền ──► IP hiện tại: 172.20.0.9
          │
          └───────────────────── Gửi gói tin HTTP ─────────────────────┐
                                                                       ▼
                                                          [ restaurant-service container ]
```

#### 3.2 Giao tiếp đồng bộ qua REST API trong Spring Boot
Trong Java Spring Boot, chúng ta có thể thực hiện các cuộc gọi API đồng bộ dễ dàng bằng công cụ `RestTemplate`. Cấu hình URL endpoint gọi liên dịch vụ sẽ được nạp thông qua các biến môi trường động đã thiết lập từ Lesson 2.

---

### PHẦN 4. DEMO VÀ THỰC HÀNH: TÍCH HỢP GIAO TIẾP VÀ DỌN DẸP PORT

#### 4.1 Triển khai mã nguồn gọi liên dịch vụ bằng `RestTemplate`

Dưới đây là cách dịch vụ `order-service` cấu hình và gọi sang `restaurant-service` để kiểm tra trạng thái hoạt động của nhà hàng trước khi tạo đơn hàng.

##### Bước 1: Khai báo cấu hình `RestTemplate` Bean
```java
package com.quickbite.order.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class RestClientConfig {
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

##### Bước 2: Viết lớp dịch vụ xử lý kết nối
```java
package com.quickbite.order.service;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class OrderValidationService {

    private final RestTemplate restTemplate;
    
    // Nạp URL của restaurant-service từ biến môi trường cấu hình ở file application.yml
    @Value("${services.restaurant-service.url}")
    private String restaurantServiceUrl;

    public OrderValidationService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    public boolean checkRestaurantStatus(Long restaurantId) {
        try {
            // Xây dựng endpoint gọi sang restaurant-service
            String endpoint = restaurantServiceUrl + "/restaurants/" + restaurantId + "/status";
            
            // Gọi API đồng bộ, mong đợi kết quả trả về kiểu Boolean
            Boolean isOpen = restTemplate.getForObject(endpoint, Boolean.class);
            
            return isOpen != null && isOpen;
        } catch (Exception e) {
            // Xử lý khi xảy ra sự cố mất kết nối mạng hoặc server sập
            System.err.println("Lỗi kết nối tới restaurant-service: " + e.getMessage());
            return false; 
        }
    }
}
```

#### 4.2 Cấu hình Docker Network đóng các port nội bộ
Cập nhật file `docker-compose.yml` để các dịch vụ giao tiếp hoàn toàn qua mạng nội bộ. Lưu ý: dịch vụ `restaurant-service` và `notification-service` không cần mở port ra ngoài máy host nữa.

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

  # restaurant-service chạy cổng 8082
  quickbite-restaurant:
    build:
      context: ./restaurant-service
    container_name: quickbite-restaurant
    # KHÔNG dùng ports để mở cổng ra máy host nữa, chỉ expose nội bộ
    expose:
      - "8082"
    environment:
      - DB_HOST=quickbite-db
      - DB_NAME=quickbite_restaurant
    depends_on:
      - quickbite-db
    networks:
      - quickbite-net

  # order-service cần mở port 8083 ra host để kiểm thử
  quickbite-order:
    build:
      context: ./order-service
    container_name: quickbite-order
    ports:
      - "8083:8083"
    environment:
      - DB_HOST=quickbite-db
      - DB_NAME=quickbite_order
      # Nạp tên container làm host cho biến kết nối
      - RESTAURANT_SVC_HOST=quickbite-restaurant
      - RESTAURANT_SVC_PORT=8082
    depends_on:
      - quickbite-db
      - quickbite-restaurant
    networks:
      - quickbite-net

networks:
  quickbite-net:
    driver: bridge
```

#### 4.3 Thực hiện kiểm chứng kết nối
1. Khởi động cụm dịch vụ: `docker compose up -d`.
2. Kiểm tra xem `quickbite-order` có thể phân giải tên miền của `quickbite-restaurant` thành công hay không bằng công cụ ping thông qua lệnh exec:
   ```bash
   docker compose exec quickbite-order ping -c 3 quickbite-restaurant
   ```
3. **Kết quả mong đợi:** Lệnh ping trả về kết quả thành công, phân giải được dải IP nội bộ của container `quickbite-restaurant` (ví dụ: `172.20.0.3`) mà không gặp lỗi.

---

### PHẦN 5. HIỂU LẦM THƯỜNG GẶP (GIAO TIẾP QUA PORT MÁY HOST)

* **Hiểu lầm thường gặp:** Khi hai container chạy trên cùng một máy chủ vật lý, chúng nên gọi nhau thông qua IP của máy host hoặc dùng từ khóa `localhost` (Ví dụ: `http://localhost:8082`) để tối ưu tốc độ truyền tải.
* **Sự thật:** 
  * Từ khóa `localhost` viết bên trong mã nguồn chạy trên container sẽ trỏ thẳng về **chính không gian cô lập của container đó**, không phải máy host vật lý.
  * Nếu dùng IP máy host vật lý, gói tin sẽ phải đi qua các lớp định tuyến của card mạng vật lý rồi mới quay ngược trở lại card mạng ảo của Docker. Việc này gây ra độ trễ (overhead) không đáng có và phụ thuộc vào IP của máy host.
  * Khi sử dụng cơ chế DNS nội bộ của Docker Network (giao tiếp trực tiếp bằng tên container), gói tin đi trực tiếp qua hạ tầng mạng ảo Bridge, đạt tốc độ truyền tải cực nhanh và bảo mật tuyệt đối vì dữ liệu không đi ra ngoài máy host.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG (XÁC MINH KIẾN THỨC)

1. **Tổng quan về cơ chế mạng Docker Container:**
   * [Docker Container Networking - Docker Docs](https://docs.google.com/url?q=https://docs.docker.com/config/containers/container-networking/)
2. **Hướng dẫn sử dụng RestTemplate trong Spring Boot:**
   * [Spring RestTemplate Reference - Baeldung](https://docs.google.com/url?q=https://www.baeldung.com/spring-rest-template-list)
3. **Cách phân giải DNS nội bộ của Docker:**
   * [Docker DNS Service Discovery - Docker Docs](https://docs.google.com/url?q=https://docs.docker.com/network/#user-defined-bridge-networks)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao mạng `default bridge` (mạng mặc định khi không khai báo `networks` cụ thể) lại không thể giúp `quickbite-order` gọi sang `quickbite-restaurant` bằng tên container, buộc chúng ta phải tạo một mạng `User-defined Bridge`?
* *Gợi ý:* Docker Engine thiết kế mạng `default bridge` không tích hợp DNS Server nội bộ vì lý do tương thích ngược và bảo mật. Chỉ có mạng do người dùng tự định nghĩa (`User-defined bridge`) mới được kích hoạt dịch vụ DNS nội bộ để phân giải tên container thành IP.

#### Câu 2 (Đọc và dự đoán)
Giả sử bạn chạy lệnh `docker compose stop quickbite-restaurant` để bảo trì dịch vụ nhà hàng. Khi khách hàng bấm tạo đơn hàng mới, điều gì xảy ra ở phía log của container `quickbite-order` khi chạy hàm `checkRestaurantStatus`?
* *Gợi ý:* Hàm `restTemplate.getForObject` sẽ ném ra ngoại lệ `ResourceAccessException` (ví dụ: ConnectException - Connection refused hoặc Host unreachable). Khối catch sẽ hoạt động, in dòng log lỗi kết nối ra console và trả về kết quả `false`, ngăn cản khách tạo đơn hàng trên hệ thống.

#### Câu 3 (Xử lý tình huống)
Container `quickbite-order` của bạn bỗng dưng không thể kết nối tới `quickbite-restaurant` và báo lỗi `UnknownHostException: quickbite-restaurant`. Hãy đưa ra 3 bước chẩn đoán nhanh bằng Docker Compose CLI để xử lý lỗi này.
* *Gợi ý:* 
  1. Chạy `docker compose ps` để kiểm tra xem container `quickbite-restaurant` có đang ở trạng thái hoạt động (`Up`) hay đã bị dừng (`Exit`).
  2. Kiểm tra xem cả hai container có cùng kết nối chung vào một mạng Docker Network hay không bằng lệnh `docker inspect` hoặc kiểm tra file compose.
  3. Dùng lệnh `docker compose exec quickbite-order nslookup quickbite-restaurant` để kiểm tra xem DNS nội bộ có phân giải được IP hay không.
