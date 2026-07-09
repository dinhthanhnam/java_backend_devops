# SESSION 11: REVERSE PROXY VỚI NGINX TRONG MÔI TRƯỜNG PRODUCTION

## LESSON 03: Triển khai Nginx làm Reverse Proxy chuyển tiếp yêu cầu đến API Gateway

---

### PHẦN 1. MỤC TIÊU BÀI HỌC
Sau khi hoàn thành bài học này, học viên có khả năng:
* **Cấu hình** thành công chỉ thị `proxy_pass` trong tệp cấu hình con `/etc/nginx/conf.d/quickbite.conf` để chuyển hướng traffic từ cổng 80 vào cổng 8080 của API Gateway.
* **Thiết lập** các HTTP Headers (`Host`, `X-Real-IP`, `X-Forwarded-For`, `X-Forwarded-Proto`) để bảo toàn thông tin của request ban đầu.
* **Áp dụng** cơ chế nạp nóng cấu hình (Zero-Downtime Hot Reload) để cập nhật cấu hình Nginx mà không làm gián đoạn dịch vụ của khách hàng.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ
Trong kiến trúc của dự án QuickBite, cổng vào trung tâm điều phối toàn bộ các microservices là dịch vụ API Gateway (`api-gateway`). Trên môi trường Production, container `api-gateway` chạy trong Docker và map cổng ra ngoài host:
```yaml
ports:
  - "8080:8080"
```
Lúc này, API Gateway lắng nghe trực tiếp trên cổng `8080` của máy chủ VPS. Chúng ta không muốn người dùng khi đặt đồ ăn phải gõ thêm cổng `:8080` trên đường dẫn URL (nhìn rất thiếu chuyên nghiệp). Hơn thế nữa, việc mở cổng `8080` trực tiếp ra Internet là một nguy cơ bảo mật. 

Chúng ta cần đặt Nginx ở cổng `80` (cổng HTTP mặc định của thế giới Web), tiếp nhận toàn bộ các request gửi tới và chuyển tiếp (proxy) chúng vào cổng `8080` của API Gateway trên localhost (`127.0.0.1`). Việc này vừa giúp che giấu cổng thực tế, vừa giúp hệ thống có một cổng vào chuẩn hóa duy nhất.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI

#### 3.1 Chỉ thị `proxy_pass`
Chỉ thị `proxy_pass` là cốt lõi của tính năng Reverse Proxy trong Nginx. Nó hướng dẫn Nginx chuyển tiếp request nhận được ở `location` hiện tại tới một máy chủ đích khác (Upstream Server).
* Cú pháp:
  ```nginx
  location /api {
      proxy_pass http://127.0.0.1:8080;
  }
  ```
* *Cơ chế hoạt động:* Khi Client gửi request tới `http://<vps_ip>/api/v1/users`, Nginx khớp với `location /api`, sau đó thay thế phần host bằng `http://127.0.0.1:8080` và gửi tiếp request tới API Gateway.

#### 3.2 Bảo toàn thông tin Request qua HTTP Headers
Khi Nginx đứng làm trung gian nhận request rồi gửi lại một request mới tới API Gateway, địa chỉ IP của client gửi tới API Gateway lúc này sẽ bị đổi thành địa chỉ IP nội bộ của Nginx (`127.0.0.1`). Điều này khiến ứng dụng Spring Boot phía sau không thể nhận diện được IP thực tế của khách hàng (ví dụ: phục vụ ghi log, chặn spam IP, hoặc định vị địa lý).

Để giải quyết vấn đề này, Nginx cung cấp chỉ thị `proxy_set_header` để đính kèm thêm các tiêu đề HTTP (Headers) trước khi chuyển tiếp request:
* `proxy_set_header Host $host`: Bảo toàn tên miền ban đầu của request.
* `proxy_set_header X-Real-IP $remote_addr`: Lưu IP thực tế của Client kết nối tới Nginx.
* `proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for`: Ghi vết chuỗi IP mà request đã đi qua (hữu ích khi đi qua nhiều proxy).
* `proxy_set_header X-Forwarded-Proto $scheme`: Lưu giao thức ban đầu (HTTP hay HTTPS).

#### 3.3 Zero-Downtime Hot Reload
Nginx nổi tiếng với khả năng cập nhật cấu hình mà không làm gián đoạn bất kỳ request nào đang xử lý.
* Lệnh thực thi: `sudo systemctl reload nginx` (hoặc `nginx -s reload`).
* *Cơ chế:* Tiến trình cha (Master process) sẽ đọc file cấu hình mới. Nếu cấu hình hợp lệ, nó sẽ khởi tạo các tiến trình con mới (Worker processes) chạy với cấu hình mới để tiếp nhận các request mới. Đồng thời, nó ra lệnh cho các Worker cũ xử lý nốt các request dở dang rồi tự hủy một cách nhẹ nhàng.

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (CẤU HÌNH PROXY SANG API GATEWAY TRÊN VPS)

Học viên thực hiện cấu hình Reverse Proxy cho dự án QuickBite trên VPS theo các bước sau:

#### 4.1 Biên soạn tệp cấu hình con `/etc/nginx/conf.d/quickbite.conf`
1. Đăng nhập vào VPS bằng Bitvise SSH Client, mở Terminal và tạo file cấu hình con:
   ```bash
   sudo nano /etc/nginx/conf.d/quickbite.conf
   ```
2. Nhập nội dung cấu hình sau (chuyển tiếp toàn bộ request bắt đầu bằng `/api` vào API Gateway đang chạy ở cổng 8080 của máy host):
   ```nginx
   server {
       listen 80;
       server_name _;

       # Ghi log riêng cho QuickBite
       access_log /var/log/nginx/quickbite_access.log;
       error_log /var/log/nginx/quickbite_error.log;

       # Cấu hình chuyển tiếp request tới API Gateway
       location /api {
           proxy_pass http://127.0.0.1:8080;
           
           # Đính kèm HTTP Headers để bảo toàn thông tin request gốc
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```
3. Nhấn `Ctrl + O`, `Enter` để lưu file và `Ctrl + X` để thoát nano.

#### 4.2 Kiểm tra cú pháp và kích hoạt cấu hình
1. Luôn chạy lệnh kiểm tra cú pháp trước khi chạy reload:
   ```bash
   sudo nginx -t
   ```
   *Đảm bảo kết quả trả về:* `nginx: configuration file /etc/nginx/nginx.conf test is successful`.
2. Reload dịch vụ Nginx để áp dụng cấu hình mới:
   ```bash
   sudo systemctl reload nginx
   ```

#### 4.3 Kiểm chứng kết nối từ Client local
1. Mở Postman hoặc terminal máy cá nhân của bạn.
2. Gửi request HTTP GET tới IP VPS công khai ở cổng 80 (không cần gõ cổng 8080 nữa):
   ```bash
   curl http://<vps_public_ip>/api/v1/restaurants
   ```
3. *Kết quả mong đợi:* Nhận về mã trạng thái `200 OK` cùng dữ liệu JSON danh sách nhà hàng từ dịch vụ backend.
4. Trên VPS, kiểm tra file access log của Nginx để xem log request vừa thực hiện:
   ```bash
   tail -n 10 /var/log/nginx/quickbite_access.log
   ```

---

### PHẦN 5. CÂU HỎI TỰ LUẬN ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
* **Câu hỏi:** Giải thích ý nghĩa và sự cần thiết của dòng cấu hình `proxy_set_header X-Real-IP $remote_addr;` khi thiết lập Reverse Proxy trong Nginx.
* **Gợi ý trả lời:**
  * Dòng cấu hình này có ý nghĩa đính kèm thêm một HTTP header tên là `X-Real-IP` chứa giá trị IP thật của client gửi tới (`$remote_addr`) vào request trước khi Nginx chuyển tiếp nó tới API Gateway.
  * Sự cần thiết: Nếu không có dòng này, request đến API Gateway sẽ mang IP nguồn là IP của Nginx (`127.0.0.1`). Điều này khiến ứng dụng backend không thể biết được địa chỉ IP thực tế của khách hàng ngoài Internet để phục vụ các tác vụ như bảo mật, rate limit theo IP, ghi log vết truy cập hoặc thống kê địa lý.

#### Câu 2 (Phân tích)
* **Câu hỏi:** Phân tích sự khác biệt về cơ chế hoạt động và tầm ảnh hưởng đối với kết nối của người dùng giữa hai lệnh: `systemctl restart nginx` và `systemctl reload nginx`. Tại sao DevOps tuyệt đối tránh dùng lệnh `restart` trên môi trường Production?
* **Gợi ý trả lời:**
  * `systemctl restart nginx`: Tắt hoàn toàn tiến trình Nginx cũ ngay lập tức (kill master và các worker processes), sau đó mới khởi động lại tiến trình mới.
    * *Ảnh hưởng:* Gây gián đoạn dịch vụ (Downtime). Toàn bộ kết nối của người dùng đang truyền dữ liệu tại thời điểm đó sẽ bị ngắt đột ngột (gây lỗi timeout, lỗi đứt kết nối). Nếu tệp cấu hình mới bị lỗi cú pháp, Nginx sẽ không thể bật lại được, khiến hệ thống sập hoàn toàn cho đến khi lỗi được sửa.
  * `systemctl reload nginx`: Master process đọc cấu hình mới. Nếu hợp lệ, nó sẽ tạo các worker mới chạy cấu hình mới và ra lệnh cho các worker cũ xử lý nốt các kết nối đang dang dở rồi mới tự hủy.
    * *Ảnh hưởng:* Hoàn toàn không gây downtime (Zero-Downtime). Khách hàng không nhận biết được sự thay đổi. Nếu cấu hình mới bị lỗi cú pháp, lệnh reload sẽ dừng lại và báo lỗi, đồng thời giữ nguyên các worker cũ chạy bình thường để phục vụ khách hàng.

#### Câu 3 (Nâng cao)
* **Câu hỏi:** Một ứng dụng Spring Boot chạy sau Nginx Reverse Proxy cần lấy thông tin giao thức gốc (HTTP hay HTTPS) của Client. Tuy nhiên, lập trình viên phát hiện phương thức `request.getScheme()` trong Java luôn trả về giá trị `http` dù Client truy cập qua đường dẫn `https://api.quickbite.com`. Hãy phân tích nguyên nhân và đề xuất giải pháp cấu hình ở cả phía Nginx và Spring Boot để khắc phục lỗi này.
* **Gợi ý trả lời:**
  * *Nguyên nhân:* Nginx đảm nhận vai trò giải mã SSL (SSL Termination), nó tiếp nhận request HTTPS từ Client, giải mã rồi chuyển tiếp request dạng HTTP thường vào Spring Boot (`http://127.0.0.1:8080`). Do đó, Spring Boot chỉ nhìn thấy request HTTP thường đi vào nên `request.getScheme()` trả về `http` là hoàn toàn đúng theo thực tế nhận được.
  * *Giải pháp khắc phục:*
    1. **Phía Nginx:** Đính kèm header `X-Forwarded-Proto` trong file cấu hình proxy:
       `proxy_set_header X-Forwarded-Proto $scheme;` (nhằm gửi thông tin giao thức gốc cho backend).
    2. **Phía Spring Boot:** Cấu hình ứng dụng để nhận biết và tin tưởng các header dạng `X-Forwarded-*` do proxy gửi sang bằng cách thêm dòng cấu hình sau vào tệp `application.yml` (hoặc `application.properties`):
       ```yaml
       server:
         forward-headers-strategy: framework
       ```
       Khi cấu hình này được kích hoạt, Spring Boot sẽ đọc header `X-Forwarded-Proto` và tự động ánh xạ ngược lại vào phương thức `request.getScheme()`, giúp trả về giá trị `https` chính xác của client ban đầu.
