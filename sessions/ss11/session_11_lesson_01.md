# SESSION 11: REVERSE PROXY VỚI NGINX TRONG MÔI TRƯỜNG PRODUCTION

## LESSON 01: Khái niệm Reverse Proxy và vai trò của Nginx trong kiến trúc Microservices

---

### PHẦN 1. MỤC TIÊU BÀI HỌC
Sau khi hoàn thành bài học này, học viên có khả năng:
* **Phân biệt** được nguyên lý hoạt động của Forward Proxy so với Reverse Proxy.
* **Giải thích** được vai trò của Nginx làm cổng kết nối duy nhất (Single Entry Point) bảo vệ hệ thống QuickBite trên production.
* **Mô tả** được sơ đồ luồng di chuyển của dữ liệu (Traffic Flow) đi từ client ngoài Internet qua Nginx Reverse Proxy trước khi đập vào API Gateway.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ
Hiện tại, cụm container microservices của dự án QuickBite (như `user-service`, `restaurant-service`, `api-gateway`) đã chạy độc lập và an toàn bên trong VPS. Để chạy thử ở local hoặc test nhanh, chúng ta thường map trực tiếp cổng các container ra ngoài host (ví dụ: `8080:8080` cho API Gateway, `8081:8081` cho User Service...).

Tuy nhiên, khi đưa hệ thống lên môi trường Production thực tế, cách làm này bộc lộ các điểm yếu nghiêm trọng:
* **Tấn công diện rộng (Attack Surface):** Việc mở toang quá nhiều cổng mạng ra Internet tạo điều kiện cho hacker thực hiện quét cổng, khai thác trực tiếp lỗ hổng bảo mật của từng service.
* **Quản trị phức tạp:** Client (Web/Mobile) phải ghi nhớ và kết nối trực tiếp đến nhiều cổng khác nhau, gây khó khăn cho việc cấu hình bảo mật SSL/HTTPS và chính sách CORS.
* **Không thể che giấu hạ tầng:** Hacker dễ dàng biết được hệ thống đang có các service nào chạy trên các cổng nào để lên kịch bản tấn công.

Do đó, chúng ta cần một thành phần duy nhất đứng ở "mặt tiền" (cổng 80/443), đại diện cho toàn bộ hệ thống để tiếp nhận request và bẻ hướng định tuyến vào bên trong. Thành phần đó chính là **Nginx Reverse Proxy**.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI

#### 3.1 Forward Proxy vs Reverse Proxy
Để tránh nhầm lẫn, chúng ta phân biệt rõ hai khái niệm proxy phổ biến:

| Tiêu chí | Forward Proxy | Reverse Proxy (Nginx) |
| :--- | :--- | :--- |
| **Vị trí đứng** | Đứng trước Client (đại diện cho Client) | Đứng trước Server (đại diện cho Server) |
| **Mục đích** | Giúp Client đi ra Internet ẩn danh, vượt tường lửa (Ví dụ: VPN, Proxy công ty chặn web đen). | Đón nhận request từ Internet gửi vào các server nội bộ ẩn phía sau. |
| **Nhận biết** | Client biết rõ mình đang qua Proxy. Server đích chỉ thấy IP của Proxy chứ không thấy IP thật của Client. | Client hoàn toàn không biết cấu trúc server phía sau, chỉ nghĩ đang tương tác trực tiếp với Reverse Proxy. |

```text
Forward Proxy (VPN/Company Proxy):
[ Client ] ──► [ Forward Proxy ] ──► [ Internet ] ──► [ Server đích ]

Reverse Proxy (Nginx):
[ Client ] ──► [ Internet ] ──► [ Reverse Proxy ] ──► [ Mạng nội bộ / Localhost ] ──► [ API Gateway/Microservices ]
```

#### 3.2 Vai trò của Nginx làm Reverse Proxy trong Microservices
Nginx (Engine X) là một phần mềm mã nguồn mở rất nhẹ, hiệu năng cực kỳ cao nhờ kiến trúc hướng sự kiện (Event-driven). Khi làm Reverse Proxy cho hệ thống QuickBite, Nginx mang lại các lợi ích vượt trội:
1. **Single Entry Point (Điểm vào duy nhất):** Tường lửa VPS chỉ cần mở duy nhất cổng 80 (HTTP) và 443 (HTTPS) trỏ vào Nginx. Toàn bộ các cổng microservices khác đều có thể được chặn lại hoặc cô lập an toàn ở localhost.
2. **Ẩn giấu hạ tầng (Security Hardening):** Client bên ngoài chỉ giao tiếp với Nginx, hoàn toàn không biết địa chỉ mạng nội bộ, cổng chạy, hay ngôn ngữ phát triển của các service phía sau.
3. **SSL Termination:** Nginx đảm nhận việc giải mã HTTPS (cổng 443) và chuyển tiếp HTTP thường vào mạng nội bộ. Việc này giúp giảm tải tính toán mã hóa cho các container Spring Boot.
4. **Tải tĩnh (Static Content Caching):** Nginx có thể trả trực tiếp các file HTML/CSS/JS của trang quản trị (Admin Dashboard) hay hình ảnh sản phẩm cực kỳ nhanh mà không cần gọi đến Spring Boot.

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (SƠ ĐỒ TRAFFIC FLOW TRÊN VPS)

Trong kịch bản triển khai thực tế của Session 11, luồng di chuyển dữ liệu (Traffic Flow) từ Client ở ngoài Internet đi vào hệ thống QuickBite được thực hiện tuần tự như sau:

```text
                                  MÔI TRƯỜNG VPS PRODUCTION
                       ┌───────────────────────────────────────────────┐
                       │ Tường lửa UFW (Chỉ cho phép cổng 80, 443)     │
                       │                                               │
[ Client (Postman) ] ──┼───► [ Nginx Reverse Proxy (Cổng 80) ]         │
                       │                 │                             │
                       │        (proxy_pass chuyển tiếp)               │
                       │                 ▼                             │
                       │     [ API Gateway (Localhost:8080) ]          │
                       │                 │                             │
                       │                 ├─► [ user-service:8081 ]     │
                       │                 └─► [ restaurant-service:8082]│
                       └───────────────────────────────────────────────┘
```

1. **Client gửi yêu cầu:** Client dùng Postman gửi request lấy danh sách món ăn: `http://<vps_public_ip>/api/v1/restaurants`. Request này đập vào cổng 80 của VPS và được tiếp nhận bởi Nginx.
2. **Nginx bẻ hướng định tuyến:** Nginx đọc cấu hình, phát hiện tiền tố `/api`. Nó lập tức chuyển hướng request này tới cổng của API Gateway đang lắng nghe trên localhost: `http://127.0.0.1:8080/api/v1/restaurants`.
3. **API Gateway điều phối:** API Gateway tiếp nhận request từ Nginx, giải mã và thực hiện định tuyến nội bộ đến `restaurant-service` ở cổng `8082` để lấy dữ liệu trả về cho Nginx, rồi Nginx phản hồi lại cho Client.

---

### PHẦN 5. CÂU HỎI TỰ LUẬN ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
* **Câu hỏi:** Phân biệt sự khác nhau cơ bản giữa Forward Proxy và Reverse Proxy. Cho ví dụ thực tế về từng loại.
* **Gợi ý trả lời:** 
  * Forward Proxy đại diện cho Client (đứng trước Client), giúp Client ẩn danh hoặc vượt tường lửa để truy cập Internet (Ví dụ: VPN 1.1.1.1, proxy công ty kiểm soát lượt truy cập của nhân viên).
  * Reverse Proxy đại diện cho Server (đứng trước cụm Server), đón nhận request từ Internet gửi vào và phân phối tới các server thích hợp phía sau mà Client không hề biết cấu trúc của Server (Ví dụ: Nginx đứng trước API Gateway của hệ thống QuickBite).

#### Câu 2 (Phân tích)
* **Câu hỏi:** Tại sao khi đưa hệ thống Microservices lên môi trường Production, việc mở trực tiếp các cổng `8080`, `8081`, `8082` ra ngoài Internet cho Client kết nối trực tiếp lại bị coi là một lỗ hổng bảo mật nghiêm trọng?
* **Gợi ý trả lời:**
  * Tạo ra diện tấn công (Attack Surface) rộng: Hacker có thể dò quét cổng (port scanning) và khai thác trực tiếp lỗ hổng trên từng service riêng lẻ mà không đi qua các bộ lọc bảo mật tập trung.
  * Lộ sơ đồ kiến trúc hệ thống: Client bên ngoài biết rõ từng dịch vụ nằm ở cổng nào, công nghệ chạy là gì.
  * Khó quản lý chứng chỉ SSL/HTTPS: Phải cấu hình mã hóa SSL trên từng Spring Boot service đơn lẻ, gây tốn tài nguyên xử lý mã hóa của CPU của từng container và phức tạp khi quản trị chứng chỉ.

#### Câu 3 (Nâng cao)
* **Câu hỏi:** Trong kiến trúc hệ thống, khi đã sử dụng Spring Cloud Gateway làm API Gateway để định tuyến và bảo mật ứng dụng, tại sao người ta vẫn bắt buộc phải dựng thêm Nginx làm Reverse Proxy đứng ở trước API Gateway? Nginx và API Gateway bổ trợ cho nhau như thế nào?
* **Gợi ý trả lời:**
  * Nginx đóng vai trò là Edge Reverse Proxy (ở biên mạng): Đảm nhận nhiệm vụ giải mã SSL/HTTPS (SSL Termination), xử lý nén Gzip, cache các tệp tin tĩnh (Frontend HTML/JS) và bảo vệ cổng ở tầng mạng của hệ điều hành VPS (chỉ mở 80/443).
  * API Gateway (Spring Cloud Gateway) đứng ở tầng ứng dụng phía sau: Chỉ nhận request HTTP thường đã được giải mã từ Nginx, tập trung xử lý nghiệp vụ Microservices như xác thực Token JWT, giải mã Request Body, định tuyến động (Dynamic Routing) theo Service Discovery và xử lý logic kết hợp dữ liệu (Request/Response Transformation).
  * Sự kết hợp này giúp phân tách rõ ràng trách nhiệm: Nginx tối ưu hóa hiệu năng truyền tải mạng và bảo mật biên, còn API Gateway tối ưu hóa logic điều phối microservices.
