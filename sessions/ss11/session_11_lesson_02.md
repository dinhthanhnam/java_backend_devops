# SESSION 11: REVERSE PROXY VỚI NGINX TRONG MÔI TRƯỜNG PRODUCTION

## LESSON 02: Cấu trúc tệp cấu hình Nginx và các khối khai báo chính (Configuration Blocks)

---

### PHẦN 1. MỤC TIÊU BÀI HỌC
Sau khi hoàn thành bài học này, học viên có khả năng:
* **Mô tả** được cơ chế hoạt động của tệp cấu hình chính `nginx.conf` và cách Nginx tự động nạp các tệp cấu hình con từ thư mục `/etc/nginx/conf.d/`.
* **Sử dụng thành thạo** các khối khai báo `server`, `location` và các chỉ thị phổ biến bên trong để thiết lập máy chủ ảo.
* **Áp dụng** các quy tắc so khớp đường dẫn URI (Prefix, Exact, Regular Expression) trong khối `location`.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ
Khi bắt tay vào cấu hình Nginx, nhiều lập trình viên có thói quen chỉnh sửa trực tiếp tệp tin cấu hình gốc `/etc/nginx/nginx.conf`. Việc này cực kỳ nguy hiểm trong môi trường Production vì:
* Dễ làm hỏng cấu hình toàn cục của hệ thống (như số lượng luồng xử lý, giới hạn file mở, logs hệ thống).
* Khó quản lý khi hệ thống chạy nhiều dự án khác nhau (mọi cấu hình chồng chéo trong một file khổng lồ).
* Gây gián đoạn toàn bộ dịch vụ nếu gõ sai một ký tự nhỏ trong file gốc.

Chuẩn vận hành thực tế là: **Không bao giờ chỉnh sửa trực tiếp `nginx.conf` gốc**. Tệp gốc này đã được thiết kế sẵn để tự động nạp (import) mọi tệp tin có đuôi `.conf` trong thư mục `/etc/nginx/conf.d/` qua dòng lệnh:
```nginx
include /etc/nginx/conf.d/*.conf;
```
Do đó, chúng ta chỉ cần tạo mới các file cấu hình độc lập (như `quickbite.conf`) đặt vào thư mục `/etc/nginx/conf.d/`. Để làm được điều này, chúng ta cần nắm vững cú pháp viết các block `server` và `location` độc lập.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI

#### 3.1 Cơ chế nạp cấu hình và phân cấp trong Nginx
Cấu trúc cấu hình Nginx được thiết kế theo hình cây phân cấp nghiêm ngặt:

1. **Main Context:** Tệp `/etc/nginx/nginx.conf` gốc. Chứa các thiết lập hệ thống toàn cục (`worker_processes`, `pid`).
2. **Khối `events`:** Nằm trong Main Context, định cấu hình hiệu năng mạng (`worker_connections`).
3. **Khối `http`:** Bao bọc toàn bộ cấu hình web. Bên trong khối `http` của file gốc sẽ có dòng `include /etc/nginx/conf.d/*.conf;`.
4. **Khối `server`:** Nằm bên trong file cấu hình con (như `/etc/nginx/conf.d/quickbite.conf`). Đại diện cho một Virtual Host (máy chủ ảo).
5. **Khối `location`:** Nằm trong `server`, dùng để định cấu hình chi tiết cho từng URI cụ thể.

> [!NOTE]
> Do các file cấu hình con trong `/etc/nginx/conf.d/` được tự động import thẳng vào bên trong khối `http` của file gốc, bạn **không được phép** khai báo các block `http { ... }` hay `events { ... }` trong các file con này.

#### 3.2 Khối khai báo `server` (Virtual Host)
Khối `server` xác định cách thức Nginx tiếp nhận request dựa trên Port và Domain. Các chỉ thị cốt lõi bao gồm:
* `listen`: Cổng lắng nghe (ví dụ: `listen 80;` cho HTTP, `listen 443 ssl;` cho HTTPS).
* `server_name`: Tên miền hoặc địa chỉ IP mà khối server này phục vụ (ví dụ: `api.quickbite.com` hoặc `_` để khớp với mọi request đổ vào IP VPS).
* `access_log` & `error_log`: Đường dẫn ghi nhật ký truy cập và lỗi riêng cho server ảo này.

#### 3.3 Khối khai báo `location` và Quy tắc so khớp đường dẫn (URI Matching)
Khối `location` bẻ hướng xử lý dữ liệu dựa trên đường dẫn URI. Cú pháp cơ bản:
```nginx
location [modifier] uri {
    # Chỉ thị xử lý
}
```
Các `modifier` (ký tự bổ trợ) quyết định độ ưu tiên so khớp đường dẫn:

1. **Khớp chính xác (`=`) - Độ ưu tiên cao nhất:**
   * Cú pháp: `location = /favicon.ico`
   * Chỉ khớp khi URI giống hệt 100%. Tốc độ xử lý nhanh nhất.
2. **Khớp tiền tố ưu tiên (`^~`) - Độ ưu tiên thứ hai:**
   * Cú pháp: `location ^~ /assets/`
   * Nếu khớp tiền tố này, Nginx dừng tìm kiếm và áp dụng luôn cấu hình, không kiểm tra các Regex location khác.
3. **Khớp Regex phân biệt hoa thường (`~`) hoặc không phân biệt hoa thường (`~*`):**
   * *Regex phân biệt hoa thường (`~`):* `location ~ \.(jpg|jpeg|png|gif)$` (Chỉ khớp `.jpg`, không khớp `.JPG`).
   * *Regex KHÔNG phân biệt hoa thường (`~*`):* `location ~* \.json$` (Khớp cả `.json` và `.JSON`).
   * Dùng để xử lý các tệp tin tĩnh hoặc các mẫu đường dẫn động.
4. **Khớp tiền tố mặc định (Không dùng ký tự modifier) - Độ ưu tiên thấp nhất:**
   * Cú pháp: `location /api`
   * Khớp với bất kỳ đường dẫn nào bắt đầu bằng `/api` (ví dụ: `/api/v1/users`, `/api/test`).

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (THIẾT LẬP FILE CONFIG CON CHO DỰ ÁN QUICKBITE)

Chúng ta thực hiện tạo một file cấu hình custom hoàn toàn mới nằm trong thư mục `/etc/nginx/conf.d/` để phục vụ riêng dự án QuickBite.

*(Các bước cấu hình giả định trên VPS)*:
1. Tạo file cấu hình con có tên `quickbite.conf` trong thư mục `/etc/nginx/conf.d/`:
   ```bash
   sudo nano /etc/nginx/conf.d/quickbite.conf
   ```
2. Soạn thảo nội dung file chỉ chứa khối `server` độc lập (tuyệt đối không bao bọc bởi `http`):
   ```nginx
   server {
      # Lắng nghe ở cổng 80 (HTTP tiêu chuẩn)
      listen 80;
      
      # Nhận diện mọi request truy cập qua IP của VPS
      server_name _;

      # Phân tách file log riêng để dễ theo dõi và debug
      access_log /var/log/nginx/quickbite_access.log;
      error_log /var/log/nginx/quickbite_error.log;

      # Location khớp tiền tố mặc định cho trang chủ
      location / {
         root /usr/share/nginx/html;
         index index.html index.htm;
      }

      # Location khớp chính xác trang lỗi
      error_page 500 502 503 504 /50x.html;
      location = /50x.html {
         root /usr/share/nginx/html;
      }
   }
   ```
3. Lưu file và kiểm tra cú pháp cấu hình Nginx xem có bị lỗi dấu chấm phẩy `;` hay đặt sai khối không:
   ```bash
   sudo nginx -t
   ```
   *Kết quả mong đợi:* Hệ thống báo `syntax is ok` và `test is successful`.

---

### PHẦN 5. CÂU HỎI TỰ LUẬN ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
* **Câu hỏi:** Tại sao khi viết các tệp cấu hình con trong thư mục `/etc/nginx/conf.d/` chúng ta không được phép bọc chúng bên trong khối `http { ... }`?
* **Gợi ý trả lời:**
  * Vì tệp cấu hình chính `/etc/nginx/nginx.conf` đã khai báo sẵn khối `http { ... }` toàn cục, và bên trong khối `http` đó đã có chỉ thị `include /etc/nginx/conf.d/*.conf;`.
  * Khi Nginx hoạt động, nó sẽ tự động chèn nội dung của tất cả các file trong `conf.d/` vào đúng vị trí của dòng lệnh `include` đó. Nếu chúng ta khai báo thêm khối `http` trong file con, cấu hình sẽ bị trùng lặp khối lồng nhau (HTTP lồng HTTP), dẫn đến lỗi cú pháp nghiêm trọng khi khởi động Nginx.

#### Câu 2 (Phân tích)
* **Câu hỏi:** Giả sử ta cấu hình Nginx có 2 location:
  `location /api/v1/ { ... }` và `location ~* \.json$ { ... }`.
  Khi Client gọi một URI là `/api/v1/users.json`, Nginx sẽ chọn khối `location` nào để xử lý request? Hãy phân tích cơ chế ưu tiên của Nginx để giải thích.
* **Gợi ý trả lời:**
  * Nginx sẽ chọn khối `location ~* \.json$ { ... }` (khớp Regular Expression) để xử lý.
  * *Phân tích cơ chế:* Đầu tiên Nginx sẽ tìm kiếm các location khớp tiền tố (prefix matches). Nó tìm thấy `/api/v1/` khớp với URI. Tuy nhiên, vì location này không sử dụng ký tự sửa đổi ưu tiên (`^~` hoặc `=`), Nginx chỉ tạm thời lưu lại kết quả này và tiếp tục quét qua các location sử dụng Regular Expression (`~` hoặc `~*`). Khi quét đến `location ~* \.json$`, nó thấy khớp với URI. Theo quy tắc của Nginx, so khớp Regex có độ ưu tiên cao hơn so khớp tiền tố thông thường, do đó Nginx sẽ chọn khối Regex để thực thi.

#### Câu 3 (Nâng cao)
* **Câu hỏi:** Trong cấu hình Nginx, việc gõ thiếu dấu chấm phẩy `;` ở cuối mỗi dòng chỉ thị là một lỗi rất phổ biến. Tại sao lỗi này lại cực kỳ nguy hiểm trên môi trường Production và làm thế nào để DevOps ngăn chặn triệt để rủi ro làm sập hệ thống khi cập nhật cấu hình?
* **Gợi ý trả lời:**
  * *Mức độ nguy hiểm:* Nginx đọc tệp cấu hình tuần tự. Thiếu dấu `;` làm Nginx hiểu sai ranh giới của các dòng lệnh, gây lỗi cú pháp nghiêm trọng. Nếu DevOps trực tiếp khởi động lại dịch vụ (`systemctl restart nginx`) khi file cấu hình lỗi, tiến trình Nginx cũ sẽ bị tắt và tiến trình mới không thể khởi động lên được, dẫn đến toàn bộ hệ thống (Web/API) bị sập hoàn toàn (Downtime).
  * *Biện pháp ngăn chặn triệt để:*
    1. **Không bao giờ dùng `restart`:** Tuyệt đối không dùng `systemctl restart nginx`. Thay vào đó, hãy luôn sử dụng lệnh nạp nóng `systemctl reload nginx` (hoặc `nginx -s reload`). Nếu cấu hình lỗi, lệnh reload sẽ báo lỗi và tiến trình Nginx cũ vẫn tiếp tục chạy bình thường để phục vụ khách hàng.
    2. **Kiểm tra cú pháp trước khi reload:** Luôn chạy lệnh kiểm tra cú pháp `sudo nginx -t` sau khi sửa cấu hình. Chỉ khi lệnh này trả về kết quả thành công (`syntax is ok`), chúng ta mới tiến hành reload dịch vụ.
