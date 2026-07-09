# PROMPT CHO GAMMA: REVERSE PROXY VỚI NGINX TRONG PRODUCTION (SESSION 11)

## 1. CONTEXT & ROLE
* **Role:** DevOps Architect kiêm Giảng viên cao cấp. Giọng điệu chuyên nghiệp, đi thẳng vào bản chất kỹ thuật, thực tiễn bảo mật và vận hành hệ thống.
* **Target Audience:** Kỹ sư phần mềm, học viên đang triển khai dự án Spring Boot Microservices (hệ thống QuickBite) lên môi trường Cloud thực tế.
* **Objective:** Giải thích bản chất Reverse Proxy, hướng dẫn cấu hình Nginx thông qua thư mục `conf.d/` (giữ nguyên tệp gốc `nginx.conf`), thiết lập proxy chuyển tiếp lưu lượng và bảo toàn header đến API Gateway, cấu hình tên miền và SSL/HTTPS tự động bằng Certbot Let's Encrypt.

---

## 2. PARTITION INSTRUCTIONS
* **Số lượng slide khuyến nghị:** 15 - 17 slides.
* **Nguyên tắc phân bổ nội dung:**
  * **LESSON 01 (Tổng quan & Khái niệm):** Reverse Proxy, phân biệt Forward vs Reverse Proxy, vai trò bảo vệ biên (Edge Proxy) trong cụm microservices.
  * **LESSON 02 (Cú pháp cấu hình):** Cơ chế nạp cấu hình tự động (không sửa tệp gốc `nginx.conf`), cú pháp khối `server`, khối `location` và các quy tắc so khớp (Prefix, Exact, Regex).
  * **LESSON 03 (Thực hành Proxy API Gateway):** Cấu hình `proxy_pass` chuyển tiếp tới API Gateway map trên host, thiết lập các HTTP headers bảo toàn thông tin, và kỹ thuật Zero-Downtime Hot Reload.
  * **LESSON 04 (Domain & SSL HTTPS):** Cấu hình DNS A Record, cài đặt Certbot, tự động xin chứng chỉ Let's Encrypt, redirect 301 HTTP to HTTPS, và nguyên lý hoạt động của SSL Termination.

---

## 3. HIERARCHICAL OUTLINE (DÀN Ý CHI TIẾT) & SPEAKER NOTES

### LESSON 01: Khái niệm Reverse Proxy và vai trò của Nginx trong kiến trúc Microservices

#### Slide 1: Bối cảnh - Hạn chế của việc Expose trực tiếp cổng microservices
* **Hiện trạng hệ thống:** Cụm container microservices (Database, Backend, API Gateway) chạy độc lập trên VPS.
* **Rủi ro khi mở cổng trực tiếp (ví dụ: `8080`, `8081`...) ra ngoài Internet:**
  * *Diện tấn công rộng (Attack Surface):* Tạo điều kiện cho hacker quét cổng và khai thác lỗ hổng trực tiếp.
  * *Lộ thông tin hạ tầng:* Người dùng bên ngoài biết rõ từng dịch vụ chạy trên cổng nào, bằng công nghệ gì.
  * *Khó cấu hình bảo mật:* Phải thiết lập SSL/HTTPS phức tạp trên từng container Spring Boot riêng lẻ.
* **Giải pháp:** Sử dụng **Nginx làm Reverse Proxy** đứng ở "mặt tiền" của VPS, chỉ mở duy nhất cổng 80/443 ra ngoài.
* **Speaker Notes:** *Chào các bạn. Khi đưa dự án QuickBite lên VPS, việc mở toang các cổng 8080 hay 8081 ra ngoài Internet là một sai lầm bảo mật cực kỳ nghiêm trọng. Chúng ta cần một giải pháp bảo mật biên để chặn đứng việc quét cổng này. Đó là lúc chúng ta cần Nginx làm Reverse Proxy để che giấu hạ tầng phía sau.*

#### Slide 2: Phân biệt Forward Proxy vs Reverse Proxy
* **Forward Proxy (Proxy xuôi):**
  * Đứng trước Client, đại diện cho Client gửi request ra ngoài Internet (ví dụ: VPN, Proxy nội bộ công ty).
  * *Mục đích:* Vượt tường lửa, ẩn danh IP của Client.
* **Reverse Proxy (Proxy ngược):**
  * Đứng trước Server, đại diện cho cụm Server nhận request từ bên ngoài gửi vào.
  * *Mục đích:* Định tuyến, bảo mật, cân bằng tải. Client không hề biết cấu trúc của Server phía sau.
* **Speaker Notes:** *Hãy nhớ quy tắc đơn giản: Forward Proxy đại diện cho người dùng để đi ra ngoài Internet, còn Reverse Proxy đại diện cho máy chủ để đón nhận yêu cầu từ Internet gửi vào. Client chỉ giao tiếp duy nhất với Reverse Proxy mà thôi.*

#### Slide 3: Sơ đồ luồng di chuyển dữ liệu (Traffic Flow) qua Reverse Proxy
* Sơ đồ kết nối mạng an toàn từ Client đến Backend:
```text
[ Client (Browser) ] ──(HTTP cổng 80)──► [ Nginx Reverse Proxy ]
                                                  │ (proxy_pass chuyển tiếp)
                                                  ▼
                                      [ API Gateway (Localhost:8080) ]
                                                  │
                                       (Mạng nội bộ Docker)
                                                  ▼
                                       [ Microservices (8081-8084) ]
```
* Nginx nhận mọi request ở cổng mặc định 80/443 của VPS.
* Nginx phân tích đường dẫn và chuyển tiếp nội bộ tới API Gateway.
* **Speaker Notes:** *Khi Client gọi API, gói tin sẽ đập vào cổng 80 của Nginx. Nginx sẽ bẻ hướng chuyển tiếp gói tin này vào cổng 8080 của API Gateway trên localhost. API Gateway tiếp tục điều phối tới các service con thông qua mạng nội bộ Docker. Lớp phòng thủ này giúp giữ cho các microservices luôn an toàn.*

---

### LESSON 02: Cấu trúc tệp cấu hình Nginx và các khối khai báo chính (Configuration Blocks)

#### Slide 4: Chuẩn vận hành: Không sửa đổi tệp tin gốc `nginx.conf`
* **Nguyên tắc Production:** Giữ nguyên bản tệp tin cấu hình gốc `/etc/nginx/nginx.conf`.
* **Cơ chế nạp cấu hình tự động (Inclusion):**
  * Trong khối `http` của tệp gốc có dòng lệnh: `include /etc/nginx/conf.d/*.conf;`
  * Nginx tự động import toàn bộ các file `.conf` trong thư mục `/etc/nginx/conf.d/`.
* **Cú pháp file cấu hình con (ví dụ: `quickbite.conf`):**
  * Chỉ chứa các block `server` và `location` độc lập.
  * *Tuyệt đối không* bao bọc bởi block `http` hay `events` để tránh lỗi cú pháp lồng nhau.
* **Speaker Notes:** *Một nguyên tắc DevOps quan trọng là không bao giờ sửa trực tiếp file cấu hình hệ thống `nginx.conf`. Chúng ta sẽ tạo riêng một file cấu hình dự án trong thư mục `conf.d/`. File này sẽ tự động được Nginx nạp vào hệ thống.*

#### Slide 5: Cấu trúc khối khai báo `server` (Virtual Host)
* Định nghĩa một máy chủ ảo xử lý các request:
  ```nginx
  server {
      listen 80;
      server_name _;

      access_log /var/log/nginx/quickbite_access.log;
      error_log /var/log/nginx/quickbite_error.log;
      
      # Location blocks...
  }
  ```
  * `listen`: Cổng mạng lắng nghe (cổng 80).
  * `server_name`: Tên miền khớp request (kí tự `_` khớp với mọi request đổ vào IP VPS).
  * `access_log` & `error_log`: Nhật ký truy cập và log lỗi riêng biệt để debug.
* **Speaker Notes:** *Khối `server` đại diện cho máy chủ ảo. Ở đây chúng ta cấu hình Nginx lắng nghe ở cổng 80 và nhận diện mọi request đổ vào IP của VPS thông qua chỉ thị server_name gán bằng dấu gạch dưới.*

#### Slide 6: Khối khai báo `location` và Độ ưu tiên so khớp đường dẫn (URI Matching)
* Khối `location` xử lý request dựa trên đường dẫn URI.
* **Các modifier (ký tự bổ trợ) quyết định độ ưu tiên:**
  1. **Khớp chính xác (`=`) [Độ ưu tiên 1]:** `location = /favicon.ico`
  2. **Khớp tiền tố ưu tiên (`^~`) [Độ ưu tiên 2]:** `location ^~ /assets/` (Dừng quét Regex).
  3. **Khớp Regex phân biệt hoa thường (`~`) hoặc không phân biệt (`~*`) [Độ ưu tiên 3]:**
     * `~ \.(jpg|png)$` (Chỉ khớp chữ thường).
     * `~* \.json$` (Khớp cả `.json` và `.JSON`).
  4. **Khớp tiền tố mặc định (không modifier) [Độ ưu tiên 4]:** `location /api`
* **Speaker Notes:** *Khối `location` là nơi chúng ta quyết định bẻ hướng request. Nginx có quy tắc ưu tiên so khớp rất chặt chẽ: Khớp chính xác bằng dấu bằng có độ ưu tiên cao nhất, tiếp theo là khớp tiền tố ưu tiên, sau đó đến các khối Regex phân biệt hoa thường hoặc không phân biệt hoa thường, và thấp nhất là khớp tiền tố mặc định.*

---

### LESSON 03: Triển khai Nginx làm Reverse Proxy chuyển tiếp yêu cầu đến API Gateway

#### Slide 7: Chuyển tiếp request bằng chỉ thị `proxy_pass`
* **Cú pháp cấu hình:**
  ```nginx
  location /api {
      proxy_pass http://127.0.0.1:8080;
  }
  ```
* **Cơ chế hoạt động:** Nginx tiếp nhận request trên cổng 80, thay thế phần host bằng `http://127.0.0.1:8080` (API Gateway đang lắng nghe trên localhost) rồi gửi tiếp đi.
* **Bảo mật cổng:** Chặn hoàn toàn cổng 8080 trên tường lửa UFW, chỉ cho phép giao tiếp nội bộ từ Nginx (localhost) tới API Gateway.
* **Speaker Notes:** *Để chuyển tiếp request, ta dùng chỉ thị proxy_pass chỉ định đích đến là cổng 8080 của API Gateway trên localhost. Nhờ vậy, chúng ta có thể đóng cổng 8080 trên tường lửa UFW để ngăn chặn kết nối trực tiếp từ bên ngoài.*

#### Slide 8: Bảo toàn thông tin Client qua HTTP Headers
* Khi chuyển tiếp qua proxy, request mới sẽ mang IP nguồn là IP của Nginx.
* **Cấu hình bổ sung các chỉ thị bảo toàn headers:**
  ```nginx
  proxy_set_header Host $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto $scheme;
  ```
  * `X-Real-IP`: Chứa IP thật của Client để backend Spring Boot phục vụ các logic bảo mật, ghi vết.
* **Speaker Notes:** *Một lỗi phổ biến là backend Spring Boot chỉ thấy IP 127.0.0.1 của Nginx. Chúng ta giải quyết bằng cách dùng proxy_set_header để chuyển tiếp IP thực của client ($remote_addr) cho backend xử lý.*

#### Slide 9: Kiểm tra cấu hình và Nạp nóng Zero-Downtime Reload
* **Lỗi cú pháp:** Thiếu dấu `;` hoặc gõ sai chỉ thị sẽ khiến Nginx không thể khởi động lại, gây downtime hệ thống.
* **Quy trình DevOps chuẩn:**
  1. Kiểm tra lỗi cú pháp:
     ```bash
     sudo nginx -t
     ```
  2. Nạp nóng cấu hình (Zero-Downtime Reload):
     ```bash
     sudo systemctl reload nginx
     ```
     Master process sẽ tạo worker mới chạy cấu hình mới, các worker cũ hoàn tất kết nối dở dang rồi tự hủy, hoàn toàn không gây mất kết nối của người dùng.
* **Speaker Notes:** *Trên Production, tuyệt đối tránh dùng lệnh systemctl restart nginx. Hãy dùng nginx -t để test cú pháp trước, sau đó dùng systemctl reload nginx để cập nhật cấu hình mà không làm rơi bất kỳ request nào của khách hàng.*

---

### LESSON 04: Cấu hình tên miền (Domain) và mã hóa bảo mật SSL/HTTPS với Let's Encrypt

#### Slide 10: Trỏ DNS tên miền và mã hóa bảo mật HTTPS
* **Cấu hình DNS:** Tạo bản ghi **A Record** trên trình quản lý DNS trỏ tên miền (ví dụ: `api.quickbite.com`) về địa chỉ IP public của VPS.
* **Bảo mật HTTPS:**
  * HTTP truyền tin dạng văn bản rõ ràng (Plain text), dễ bị nghe lén lấy trộm mật khẩu hoặc Token.
  * HTTPS (cổng 443) mã hóa toàn bộ dữ liệu trên đường truyền để đảm bảo an toàn thông tin.
* **Speaker Notes:** *Để hệ thống chuyên nghiệp, ta trỏ tên miền bằng bản ghi A về IP VPS. Đồng thời, để tránh rò rỉ JWT token hay mật khẩu trên đường truyền, việc thiết lập mã hóa HTTPS cổng 443 là bắt buộc.*

#### Slide 11: Tự động hóa cấp phát SSL bằng Certbot & Let's Encrypt
* **Let's Encrypt:** Nhà cung cấp chứng chỉ SSL/TLS miễn phí và uy tín toàn cầu.
* **Certbot:** Công cụ tự động hóa xác thực tên miền và cài đặt SSL cho Nginx trên Ubuntu.
* **Các bước triển khai:**
  1. Cài đặt Certbot: `sudo apt install certbot python3-certbot-nginx -y`
  2. Chạy xin chứng chỉ: `sudo certbot --nginx -d api.quickbite.com`
* **Speaker Notes:** *Chúng ta sử dụng Certbot để tự động xin và cấu hình chứng chỉ SSL miễn phí từ Let's Encrypt. Certbot sẽ tự động chèn các cấu hình mã hóa vào file quickbite.conf của chúng ta.*

#### Slide 12: Chuyển hướng HTTP to HTTPS & Cơ chế SSL Termination
* **Redirect 301 HTTP to HTTPS:** Certbot tự cấu hình block server cổng 80 trả về mã redirect `301 Moved Permanently` bắt buộc trình duyệt chuyển hướng lên HTTPS cổng 443.
* **SSL Termination:**
  * Nginx giải mã SSL ở biên (SSL Termination), chuyển tiếp HTTP thường vào localhost tới API Gateway.
  * *Lợi ích:* Giải phóng tài nguyên tính toán giải mã cho các container Spring Boot.
  * *Tự động gia hạn:* Certbot tự cấu hình cron job tự động gia hạn trước khi chứng chỉ hết hạn 90 ngày.
* **Speaker Notes:** *Certbot cũng tự động cấu hình redirect 301 từ HTTP lên HTTPS. Với cơ chế SSL Termination, Nginx chịu trách nhiệm giải mã gói tin ở biên và truyền HTTP thường nội bộ đến API Gateway, giúp giảm tải CPU cho các ứng dụng Java.*
