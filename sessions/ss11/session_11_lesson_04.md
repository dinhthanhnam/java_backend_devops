# SESSION 11: REVERSE PROXY VỚI NGINX TRONG MÔI TRƯỜNG PRODUCTION

## LESSON 04: Cấu hình tên miền (Domain) và mã hóa bảo mật SSL/HTTPS với Let's Encrypt

---

### PHẦN 1. MỤC TIÊU BÀI HỌC
Sau khi hoàn thành bài học này, học viên có khả năng:
* **Cấu hình liên kết** bản ghi tên miền (DNS A Record) trỏ về địa chỉ IP công khai của VPS.
* **Sử dụng** công cụ Certbot để xin và cài đặt tự động chứng chỉ SSL miễn phí từ tổ chức phát hành **Let's Encrypt**.
* **Cấu hình tự động chuyển hướng** (301 Redirect) toàn bộ lưu lượng HTTP thường (cổng 80) lên giao thức HTTPS mã hóa (cổng 443).
* **Giải thích** được cơ chế hoạt động của SSL Termination tại Nginx Reverse Proxy.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ
Khi gọi API của dự án QuickBite thông qua địa chỉ IP thô (`http://103.82.20.15/api/...`), chúng ta đối mặt với 2 vấn đề lớn:
1. **Thiếu chuyên nghiệp:** Khách hàng không bao giờ muốn ghi nhớ một dãy số IP dài dòng, khó nhớ khi sử dụng dịch vụ.
2. **Nguy cơ mất an toàn thông tin:** HTTP truyền tin dưới dạng văn bản thuần túy (Plain text). Bất kỳ ai nằm trên đường truyền mạng (như admin Wi-Fi công cộng, nhà cung cấp dịch vụ mạng) đều có thể nghe lén và bắt gọn gói tin chứa mật khẩu hoặc JWT Token của người dùng.

Để giải quyết triệt để, hệ thống bắt buộc phải sử dụng tên miền chính thức (như `api.quickbite.com`) và kích hoạt giao thức bảo mật **HTTPS** (mã hóa dữ liệu truyền tải trên đường truyền). Việc này đòi hỏi chúng ta phải cấu hình tên miền và xin chứng chỉ bảo mật SSL từ một tổ chức uy tín.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI

#### 3.1 Bản ghi DNS A (Address Record)
DNS (Domain Name System) hoạt động như một danh bạ điện thoại của Internet. Để chuyển đổi tên miền dạng chữ sang địa chỉ IP vật lý của máy chủ VPS, chúng ta cần cấu hình bản ghi **A Record** trên trang quản trị tên miền:
* *Tên bản ghi (Host):* `api` (hoặc `@` cho tên miền gốc).
* *Loại bản ghi:* `A`
* *Giá trị (Value):* Điền IP công khai của VPS.

#### 3.2 Let's Encrypt và Certbot
* **Let's Encrypt:** Là một tổ chức phát hành chứng chỉ bảo mật (Certificate Authority - CA) phi lợi nhuận, cung cấp chứng chỉ SSL/TLS miễn phí, tự động hóa cao và được tin cậy bởi mọi trình duyệt trên toàn thế giới.
* **Certbot:** Là một công cụ dòng lệnh (Client) chạy trên máy chủ VPS, có nhiệm vụ tự động xác thực quyền sở hữu tên miền với Let's Encrypt, tự động tải xuống chứng chỉ SSL và tự động cấu hình tích hợp nó vào tệp cấu hình của Nginx.

#### 3.3 Khái niệm SSL Termination (Giải mã SSL tại biên)
Trong kiến trúc Microservices, quá trình mã hóa và giải mã SSL tiêu tốn khá nhiều tài nguyên tính toán của CPU. Để tối ưu hóa hiệu năng, người ta áp dụng mô hình **SSL Termination**:

```text
               (MÃ HÓA HTTPS - Cổng 443)                  (HTTP THƯỜNG - Cổng 8080)
[ Client (Browser) ] ──────────────► [ Nginx Reverse Proxy ] ──────────────► [ API Gateway / Backend ]
                                  (Giải mã SSL & Đọc request)
```

Nginx đóng vai trò là điểm kết thúc của kết nối mã hóa (SSL Termination). Nó tiếp nhận request HTTPS từ Internet gửi tới, sử dụng Private Key trên máy chủ để giải mã dữ liệu thô. Sau đó, Nginx chuyển tiếp request dạng HTTP thường trong môi trường nội bộ an toàn (localhost) tới API Gateway. Cách làm này giúp các container backend Java Spring Boot tập trung tối đa tài nguyên cho logic nghiệp vụ thay vì phải giải mã SSL.

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (CẤU HÌNH HTTPS VỚI LET'S ENCRYPT CHO NGINX)

Học viên thực hành cấu hình tên miền và xin chứng chỉ SSL cho dự án QuickBite trên VPS theo các bước sau:

#### 4.1 Cấu hình DNS tên miền và chuẩn bị file cấu hình Nginx
1. Truy cập vào trang quản trị DNS của nhà đăng ký tên miền (như Cloudflare, GoDaddy). Tạo một bản ghi `A` trỏ tên miền (ví dụ: `api.quickbite.com`) về địa chỉ IP của VPS.
2. Trên VPS, chỉnh sửa tệp tin `/etc/nginx/conf.d/quickbite.conf` để cập nhật chỉ thị `server_name` tương ứng với tên miền đã cấu hình:
   ```nginx
   server {
       listen 80;
       server_name api.quickbite.com;  # Điền tên miền của bạn ở đây

       access_log /var/log/nginx/quickbite_access.log;
       error_log /var/log/nginx/quickbite_error.log;

       location /api {
           proxy_pass http://127.0.0.1:8080;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```
3. Chạy `sudo nginx -t` và `sudo systemctl reload nginx` để áp dụng tên miền mới.

#### 4.2 Cài đặt Certbot và plugin Nginx
Trên hệ điều hành Ubuntu 24.04 LTS, chạy chuỗi lệnh sau để cài đặt Certbot:
```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx -y
```

#### 4.3 Khởi chạy quy trình cấp phát chứng chỉ SSL tự động
1. Thực thi lệnh Certbot tương tác trực tiếp với Nginx để xin chứng chỉ bảo mật:
   ```bash
   sudo certbot --nginx -d api.quickbite.com
   ```
2. *Quá trình tương tác:*
   * Certbot sẽ yêu cầu bạn cung cấp Email để nhận cảnh báo khi chứng chỉ sắp hết hạn.
   * Nhấn `A` để đồng ý với điều khoản sử dụng.
   * Nhấn `Y` hoặc `N` để đồng ý chia sẻ thông tin.
   * Certbot sẽ tự động quét file cấu hình của Nginx, xác thực tên miền qua giao thức HTTP-01 challenge.
   * Khi xác thực thành công, Certbot sẽ tự tạo chứng chỉ SSL lưu tại `/etc/letsencrypt/live/` và **tự động sửa đổi** tệp `/etc/nginx/conf.d/quickbite.conf` để thêm các cấu hình SSL cổng 443 và tạo cấu hình redirect tự động từ cổng 80 lên 443.
3. Reload Nginx để kích hoạt cấu hình mới:
   ```bash
   sudo systemctl reload nginx
   ```

#### 4.4 Kiểm tra kết quả hoạt động
1. Sử dụng trình duyệt truy cập địa chỉ: `http://api.quickbite.com/api/v1/users`.
2. Trình duyệt phải tự động chuyển hướng (Redirect 301) sang đường dẫn: `https://api.quickbite.com/api/v1/users` và hiển thị ổ khóa xanh bảo mật.
3. Chạy lệnh kiểm tra tính năng tự động gia hạn của Certbot để đảm bảo chứng chỉ sẽ tự động được cập nhật khi gần hết hạn:
   ```bash
   sudo certbot renew --dry-run
   ```

---

### PHẦN 5. CÂU HỎI TỰ LUẬN ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
* **Câu hỏi:** Giải thích cơ chế hoạt động của mô hình SSL Termination tại Nginx Reverse Proxy. Tại sao mô hình này lại được khuyến nghị sử dụng rộng rãi trong các kiến trúc Microservices thay vì cấu hình SSL trực tiếp trên từng service?
* **Gợi ý trả lời:**
  * Cơ chế hoạt động: Nginx đứng ở biên mạng sẽ tiếp nhận các kết nối HTTPS (cổng 443) từ Client, sử dụng chứng chỉ SSL cấu hình trên máy chủ để giải mã gói tin mã hóa thành dạng văn bản HTTP thô. Sau đó, Nginx chuyển tiếp request đã giải mã này tới các service backend qua cổng HTTP thường (cổng 8080/8081...) trên localhost hoặc mạng nội bộ.
  * Lý do khuyến nghị sử dụng:
    * Giảm tải cho CPU: Tác vụ giải mã mã hóa tiêu tốn nhiều CPU được tập trung xử lý tại Nginx, giúp giải phóng tài nguyên cho các container Spring Boot chỉ tập trung xử lý logic.
    * Quản lý tập trung chứng chỉ: Chỉ cần cấu hình và gia hạn chứng chỉ SSL tại một nơi duy nhất là Nginx thay vì phải quản trị và cài đặt keystore SSL cho hàng chục service Java độc lập.

#### Câu 2 (Phân tích)
* **Câu hỏi:** Sau khi chạy Certbot cấu hình HTTPS thành công cho tên miền `api.quickbite.com`, DevOps kiểm tra file cấu hình `/etc/nginx/conf.d/quickbite.conf` thì thấy có 2 server block khác nhau. Hãy phân tích vai trò của từng server block này trong việc điều hướng traffic.
* **Gợi ý trả lời:**
  * Block 1 (Lắng nghe cổng 80): Lắng nghe các request HTTP thường của Client. Nó sử dụng chỉ thị `return 301 https://$host$request_uri;` để thực hiện chuyển hướng vĩnh viễn (HTTP 301 Redirect) toàn bộ traffic này sang giao thức HTTPS cổng 443.
  * Block 2 (Lắng nghe cổng 443 ssl): Lắng nghe kết nối HTTPS bảo mật. Block này chứa các đường dẫn cấu hình chứng chỉ SSL (`ssl_certificate` và `ssl_certificate_key`) dùng để giải mã kết nối, đồng thời chứa khối `location /api` để thực thi lệnh `proxy_pass` chuyển tiếp dữ liệu đã giải mã vào API Gateway.

#### Câu 3 (Nâng cao)
* **Câu hỏi:** Giả sử bạn cấu hình tên miền `api.quickbite.com` thông qua Cloudflare (chế độ Proxy - đám mây màu cam) trỏ về IP của VPS. Khi chạy lệnh `sudo certbot --nginx -d api.quickbite.com` để xin chứng chỉ SSL Let's Encrypt, hệ thống báo lỗi không thể xác thực (Verification failed). Hãy phân tích nguyên nhân lỗi này và đề xuất giải pháp xử lý.
* **Gợi ý trả lời:**
  * *Nguyên nhân:* Certbot xác thực qua giao thức HTTP-01 challenge. Nó tạo một tệp tạm thời trên VPS và Let's Encrypt Server sẽ gọi HTTP tới tên miền `api.quickbite.com` để kiểm tra sự tồn tại của tệp này. Tuy nhiên, khi bật chế độ Proxy của Cloudflare, Cloudflare sẽ chặn kết nối trực tiếp, đóng vai trò là CDN trung gian và có thể tự động chặn các request xác thực lạ hoặc chuyển hướng sang HTTPS của chính Cloudflare, dẫn đến Let's Encrypt Server không tiếp cận trực tiếp được tệp xác thực trên VPS của bạn.
  * *Giải pháp xử lý:*
    * Giải pháp 1: Tạm thời chuyển đám mây Cloudflare sang chế độ DNS Only (đám mây màu xám) để tắt proxy trung gian, chạy lệnh Certbot hoàn tất để xin chứng chỉ thành công, sau đó bật lại chế độ Proxy.
    * Giải pháp 2: Sử dụng phương thức xác thực DNS Challenge của Let's Encrypt thay vì HTTP-01 challenge. DevOps sử dụng plugin Cloudflare của Certbot (`certbot-dns-cloudflare`) kết hợp với API Token của Cloudflare để Certbot tự động tạo bản ghi TXT xác thực trên DNS, hoàn toàn bỏ qua việc gọi HTTP challenge vào IP VPS.
