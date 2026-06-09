# SESSION 05: MULTI-SERVICES & API GATEWAY

## LESSON 04: API Gateway và điểm truy cập tập trung trong hệ thống Microservices

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Giải thích** được định nghĩa, vai trò và sự cần thiết của API Gateway trong kiến trúc Microservices.
* **Phân tích** được các bất cập chí mạng của việc cho Client (Mobile, Web) giao tiếp trực tiếp với nhiều microservices (quản lý port, CORS, phân tán mã nguồn bảo mật).
* **Hiểu bản chất** cơ chế bảo vệ của API Gateway đóng vai trò làm lá chắn an ninh, che giấu cấu trúc mạng nội bộ của hệ thống.
* **Phân biệt** được sự khác nhau giữa API Gateway cấp độ ứng dụng (Application-level Gateway) và Reverse Proxy cấp độ hạ tầng (như Nginx).

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (NỖI ĐAU CỦA VIỆC SỬ DỤNG ENDPOINT PHÂN TÁN)

Trong các bài học trước, chúng ta đã khởi chạy thành công 4 dịch vụ Spring Boot nội bộ của QuickBite cùng chạy trên một Docker Network:
1. `user-service` (cổng 8081)
2. `restaurant-service` (cổng 8082)
3. `order-service` (cổng 8083)
4. `notification-service` (cổng 8084)

Lúc này, nhóm phát triển Frontend bắt đầu tích hợp mã nguồn ứng dụng di động đặt món ăn của khách hàng. Họ ngay lập tức đụng phải ba nỗi đau cực kỳ lớn:

1. **Ma trận quản lý Endpoint phía Client:**
   * *Nỗi đau:* Để hiển thị thông tin trang chủ đặt món, ứng dụng di động phải gửi đồng thời: 1 request lấy thông tin ví tới `http://localhost:8081/users/me`, 1 request lấy danh sách món ăn tới `http://localhost:8082/restaurants/active`, và 1 request tạo đơn tới `http://localhost:8083/orders`. Client phải quản lý quá nhiều URL và port. Khi chuyển từ môi trường thử nghiệm sang Production thực tế, việc cấu hình và thay đổi hàng loạt địa chỉ IP/Domain của từng cổng dịch vụ trở thành một cơn ác mộng.
2. **Cơn ác mộng CORS (Cross-Origin Resource Sharing):**
   * *Nỗi đau:* Trình duyệt web chặn các request gọi chéo cổng. Để frontend chạy được, lập trình viên bắt buộc phải chèn cấu hình cho phép CORS trên toàn bộ 4 service độc lập. Chỉ cần thiếu sót cấu hình ở 1 service, tính năng đó sẽ bị block ngay trên trình duyệt của người dùng.
3. **Trùng lặp mã nguồn Bảo mật (Redundant Auth Logic):**
   * *Nỗi đau:* Hệ thống yêu cầu khách hàng phải đăng nhập mới được xem ví và đặt đơn. Do đó, mã nguồn giải mã Token JWT, kiểm tra quyền truy cập (Role validation) buộc phải viết lặp đi lặp lại ở cả `user-service` và `order-service`. Nếu mai sau Tech Lead muốn đổi thuật toán giải mã token, cả đội sẽ phải sửa code, build và deploy lại tất cả các microservices.

*Để giải quyết các vấn đề trên, kiến trúc Microservices hiện đại giới thiệu một thành phần trung gian mang tên **API Gateway**.*

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (API GATEWAY LÀ GÌ?)

#### 3.1 Khái niệm API Gateway
**API Gateway** là một thành phần máy chủ đóng vai trò là **điểm truy cập tập trung duy nhất (Single Entry Point)** cho mọi yêu cầu gọi API từ phía Client (Web, Mobile, Third-party) đi vào hệ thống.

Thay vì kết nối trực tiếp đến từng dịch vụ riêng lẻ, Client chỉ cần biết và gửi toàn bộ yêu cầu tới địa chỉ duy nhất của API Gateway. API Gateway sẽ phân tích yêu cầu, thực hiện các nghiệp vụ kiểm tra và điều hướng (routing) gói tin đến dịch vụ nội bộ thích hợp.

```text
  TRƯỚC KHI CÓ API GATEWAY                       SAU KHI CÓ API GATEWAY
                                              
      ┌──────────────┐                            ┌──────────────┐
      │ Client App   │                            │ Client App   │
      └─┬───┬───┬────┘                            └──────┬───────┘
        │   │   │                                        │ (Gọi cổng duy nhất: 8080)
        │   │   └─► user-service (8081)                  ▼
        │   └─────► restaurant-service (8082)     ┌──────────────┐
        └─────────► order-service (8083)          │ API Gateway  │
                                                  └─┬────┬────┬──┘
                                                    │    │    │ (Điều hướng nội bộ)
                                                    │    │    └─► user-service
                                                    │    └──────► restaurant-service
                                                    └───────────► order-service
```

#### 3.2 Các chức năng cốt lõi của API Gateway
1. **Định tuyến động (Dynamic Routing):** Phân tích URI của request để chuyển tiếp gói tin (ví dụ: request bắt đầu bằng `/api/v1/users/**` sẽ được chuyển hướng ngầm tới `user-service`).
2. **Xử lý tập trung các vấn đề chéo (Cross-cutting Concerns):**
   * *Xác thực và phân quyền (Authentication/Authorization):* Giải mã và xác thực Token JWT ngay tại cửa ngõ Gateway. Nếu token hợp lệ, Gateway mới cho phép đi tiếp; nếu không, trả về ngay lỗi `401 Unauthorized`, giúp giảm tải việc xử lý bảo mật cho các service phía sau.
   * *Quản lý CORS tập trung:* Cấu hình CORS duy nhất tại API Gateway, Client sẽ không bao giờ bị lỗi chặn chéo nguồn từ trình duyệt.
   * *Giới hạn tần suất gọi (Rate Limiting):* Ngăn chặn các cuộc tấn công DDoS bằng cách giới hạn số lượng request tối đa một địa chỉ IP hoặc một User được gọi trong một phút.
3. **Lá chắn an toàn (Security Shield):** Gateway đóng vai trò làm proxy bảo vệ. Chúng ta hoàn toàn có thể đóng kín tất cả các port của các microservices nội bộ (`8081`, `8082`...) không cho mở ra internet, chỉ mở duy nhất cổng của API Gateway (`8080`). Không ai có thể chọc phá trực tiếp vào backend của bạn.

---

### PHẦN 4. ĐẶC TẢ KIẾN TRÚC MỤC TIÊU CHO QUICKBITE

Trong các bài tiếp theo, chúng ta sẽ đưa thành phần **Spring Cloud Gateway** vào cụm Docker Compose để nâng cấp kiến trúc QuickBite lên **STATE 3** (Microservices Architecture thực thụ).

* **Cổng truy cập duy nhất của Client:** `8080` (Cổng của Spring Cloud Gateway).
* **Định tuyến đường dẫn ngầm định:**
  * Client gọi `http://localhost:8080/api/v1/users/**` ──► Chuyển tới `http://quickbite-user:8081/api/v1/users/**`
  * Client gọi `http://localhost:8080/api/v1/restaurants/**` ──► Chuyển tới `http://quickbite-restaurant:8082/api/v1/restaurants/**`
  * Client gọi `http://localhost:8080/api/v1/orders/**` ──► Chuyển tới `http://quickbite-order:8083/api/v1/orders/**`
  * Client gọi `http://localhost:8080/api/v1/notifications/**` ──► Chuyển tới `http://quickbite-notification:8084/api/v1/notifications/**`

---

### PHẦN 5. HIỂU LẦM THƯỜNG GẶP (API GATEWAY VS REVERSE PROXY NGINX)

* **Hiểu lầm thường gặp:** API Gateway (như Spring Cloud Gateway) hoàn toàn có thể thay thế cho Nginx, hoặc Nginx chính là API Gateway duy nhất cần cài đặt.
* **Sự thật:** 
  * **Nginx** là một phần mềm Reverse Proxy và Web Server cực kỳ mạnh mẽ ở lớp dưới hạ tầng (tầng mạng TCP/UDP và HTTP thô). Nó xử lý rất nhanh việc cấu hình SSL/HTTPS, nén dữ liệu, và phân phối các file tĩnh (HTML/CSS/JS) của Frontend.
  * **Spring Cloud Gateway** là một ứng dụng viết bằng Java chạy trên nền tảng Spring. Nó hiểu sâu về logic ứng dụng và tích hợp chặt chẽ với hệ sinh thái Java (như Eureka Service Discovery, Spring Security, Spring Reactive). Nó cho phép viết các bộ lọc Filter bằng code Java để can thiệp sâu vào Header, Body của request, kiểm tra quyền hạn nghiệp vụ phức tạp.
  * **Mô hình chuẩn doanh nghiệp:** Kết hợp cả hai. Nginx đứng ở ngoài cùng tiếp nhận HTTPS, phân phối file tĩnh của Frontend, sau đó chuyển tiếp các request API vào cho Spring Cloud Gateway ở bên trong xử lý logic định tuyến ứng dụng và bảo mật.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG (XÁC MINH KIẾN THỨC)

1. **Mẫu thiết kế API Gateway trong Microservices:**
   * [API Gateway Pattern - Microservices.io](https://docs.google.com/url?q=https://microservices.io/patterns/apigateway.html)
2. **Tổng quan về Spring Cloud Gateway:**
   * [Spring Cloud Gateway Official Documentation](https://docs.google.com/url?q=https://spring.io/projects/spring-cloud-gateway)
3. **Phân biệt API Gateway và Reverse Proxy:**
   * [API Gateway vs. Reverse Proxy - Kong HQ](https://docs.google.com/url?q=https://konghq.com/blog/learning-center/api-gateway-vs-reverse-proxy)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao việc áp dụng API Gateway lại giúp lập trình viên Frontend giảm thiểu được thời gian cấu hình và cập nhật code khi chuyển đổi môi trường chạy ứng dụng?
* *Gợi ý:* Vì Frontend chỉ cần lưu trữ và gọi tới duy nhất một địa chỉ gốc (Base URL) của API Gateway (ví dụ: `http://api.quickbite.com`). Việc điều hướng request đến server nào, cổng nào ở phía sau hoàn toàn do cấu hình định tuyến bên trong API Gateway quản lý, Frontend không cần phải thay đổi hay biết về các cổng nội bộ của backend.

#### Câu 2 (Đọc và dự đoán)
Nếu chúng ta chặn hoàn toàn việc mở cổng (ports) ra ngoài máy host đối với `user-service` (cổng 8081) và chỉ mở cổng của API Gateway (`8080`), điều gì xảy ra nếu Client từ ngoài internet cố gắng gửi request trực tiếp tới địa chỉ `http://[IP_SERVER]:8081/api/v1/users`?
* *Gợi ý:* Kết nối sẽ bị từ chối lập tức (Connection Refused). Do cổng 8081 không được map ra ngoài máy host vật lý, hệ điều hành của máy host sẽ không tiếp nhận và không chuyển tiếp gói tin vào container của `user-service`.

#### Câu 3 (Xử lý tình huống)
Tech Lead yêu cầu bạn bổ sung tính năng chặn các request từ các IP có hành vi spam gọi API tạo đơn hàng liên tục (quá 20 lần/giây) để bảo vệ database. Bạn sẽ triển khai logic này ở từng microservice hay triển khai tập trung tại API Gateway? Tại sao?
* *Gợi ý:* Nên triển khai tập trung tại API Gateway (sử dụng tính năng Rate Limiting Filter). Việc này giúp ngăn chặn và lọc bỏ các request xấu ngay tại cửa ngõ của hệ thống, bảo vệ tài nguyên tính toán và băng thông cho các microservices phía sau, tránh việc các microservices phải xử lý request rồi mới ném lỗi làm quá tải CPU.
