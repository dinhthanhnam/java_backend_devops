# Kịch bản Thuyết trình - Session 11: Reverse Proxy với Nginx trong môi trường Production

---

## Lesson 1: Khái niệm Reverse Proxy và vai trò của Nginx trong kiến trúc Microservices

### 1. Phần lý thuyết

**(Mở đầu - Khơi gợi bối cảnh thực tế)**
Chào các bạn! Ở buổi học trước, chúng ta đã cấu hình thành công VPS và khởi chạy các dịch vụ của dự án QuickBite. Thế nhưng, hiện trạng hệ thống lúc này là gì? Các dịch vụ của chúng ta đang chạy ngầm trong container Docker, và để kiểm tra, chúng ta đã map các cổng như `8080` (API Gateway) hay `8081` (User Service) ra ngoài máy host VPS. 

Các bạn hãy tưởng tượng, một hệ thống chạy Production thực tế mà mở toang hoác hàng loạt cổng như vậy ra Internet thì điều gì sẽ xảy ra? Các con bot tự động trên mạng sẽ liên tục dò quét IP của bạn, tìm kiếm xem cổng nào đang mở để khai thác lỗ hổng bảo mật. Web hay Mobile app của chúng ta cũng sẽ phải gọi đến các cổng khác nhau rất lộn xộn. Chưa kể là việc cấu hình HTTPS bảo mật trên từng container Spring Boot riêng lẻ cực kỳ phức tạp và tốn tài nguyên CPU. Để giải quyết triệt để vấn đề này, chúng ta cần một chiếc "khiên bảo vệ" đứng ở ngay biên mạng của VPS để đón nhận toàn bộ request. Và đó chính là **Nginx Reverse Proxy**.

**[Slide 1: Khái niệm VPS và lựa chọn hạ tầng triển khai]**
Trước tiên, chúng ta cần phân biệt rõ hai khái niệm: **Forward Proxy** và **Reverse Proxy**.
* **Forward Proxy (Proxy xuôi):** Là proxy đại diện cho phía Client. Ví dụ khi các bạn dùng VPN để vượt tường lửa hay ẩn danh IP của mình khi lướt web. Server đích chỉ nhìn thấy IP của Forward Proxy chứ không thấy IP thật của bạn.
* **Reverse Proxy (Proxy ngược):** Ngược lại hoàn toàn, đây là proxy đại diện cho phía máy chủ. Nó đứng trước cụm máy chủ backend của chúng ta. Khi Client ngoài Internet gửi request vào, họ chỉ biết và giao tiếp duy nhất với IP/Port của Reverse Proxy. Client hoàn toàn không biết phía sau có bao nhiêu server, chạy ở cổng nào hay viết bằng ngôn ngữ gì. Nginx chính là một Reverse Proxy cực kỳ mạnh mẽ như vậy.

**[Slide 2: Vai trò của Nginx làm Reverse Proxy]**
Khi đặt Nginx làm Reverse Proxy đứng ở biên hệ thống, chúng ta đạt được 4 lợi ích lớn:
1. **Single Entry Point (Điểm vào duy nhất):** Chúng ta đóng toàn bộ các cổng microservices trên tường lửa VPS, chỉ mở duy nhất cổng 80 (HTTP) và 443 (HTTPS) trỏ vào Nginx.
2. **Ẩn giấu hạ tầng (Security Hardening):** Che giấu hoàn toàn IP nội bộ và các cổng thực tế của container.
3. **SSL Termination (Giải mã SSL tại biên):** Nginx chịu trách nhiệm bắt tay mã hóa HTTPS nặng nề với Client, sau đó chuyển tiếp HTTP thường vào mạng nội bộ của VPS để giảm tải CPU giải mã cho Spring Boot.
4. **Tối ưu file tĩnh (Static Content Caching):** Nginx đọc ghi file cực nhanh nên có thể trả trực tiếp giao diện Frontend Admin Dashboard mà không cần gọi vào ứng dụng Spring Boot.

---

### 2. Phần thực hành (Demo)

**[Live Demo: Sơ đồ luồng di chuyển dữ liệu (Traffic Flow) trên VPS]**
Bây giờ, chúng ta cùng phân tích sơ đồ dòng dữ liệu đi vào VPS khi có Nginx Reverse Proxy.

*(Thao tác vẽ bảng hoặc trình bày sơ đồ kiến trúc)*:
- *Bước 1: Client gửi một request API lấy thông tin nhà hàng: `http://<vps_public_ip>/api/v1/restaurants`.*
- *Bước 2: Gói tin đập vào cổng 80 của VPS và được tiếp nhận bởi Nginx Reverse Proxy.*
- *Bước 3: Nginx đọc file cấu hình, nhận thấy request này bắt đầu bằng tiền tố `/api`. Nó lập tức chuyển hướng request này vào cổng `8080` của API Gateway đang lắng nghe trên localhost (`127.0.0.1:8080`).*
- *Bước 4: API Gateway tiếp nhận, thực hiện định tuyến nội bộ đến container `restaurant-service` ở cổng `8082` trong mạng Docker để xử lý và trả ngược lại kết quả qua Nginx về cho Client.*

---

## Lesson 2: Cấu trúc tệp cấu hình Nginx và các khối khai báo chính (Configuration Blocks)

### 1. Phần lý thuyết

**[Slide 3: Chuẩn vận hành: Không sửa đổi tệp tin gốc nginx.conf]**
Chào các bạn. Khi làm việc với Nginx trên môi trường Production, thầy có một lưu ý sống còn: **Không bao giờ được chỉnh sửa trực tiếp tệp tin cấu hình gốc `/etc/nginx/nginx.conf`**. 

Tại sao lại như vậy? Tệp `nginx.conf` gốc quản lý các thiết lập hệ thống toàn cục của Nginx cấp thấp (như số lượng Worker Processes xử lý đồng thời, đường dẫn file PID, logs hệ thống). Nếu bạn sửa trực tiếp file này, rủi ro làm hỏng cấu hình hệ thống là rất cao. Thay vào đó, tệp gốc này đã được khai báo sẵn chỉ thị nạp tự động trong khối `http`:
```nginx
include /etc/nginx/conf.d/*.conf;
```
Điều này có nghĩa là Nginx sẽ tự động quét và import toàn bộ các tệp tin có đuôi `.conf` nằm trong thư mục `/etc/nginx/conf.d/`. Nhiệm vụ của chúng ta là chỉ viết các file cấu hình con (như `quickbite.conf`) đặt vào thư mục này để quản lý độc lập.

**[Slide 4: Cấu trúc khối cấu hình custom trong conf.d/]**
Vì các tệp con được Nginx import trực tiếp vào bên trong khối `http` của tệp gốc, nên cấu trúc file con của chúng ta cực kỳ đơn giản:
* **Không cần** khai báo các khối hệ thống bao ngoài như `events { ... }` hay `http { ... }`.
* **Khai báo trực tiếp** khối `server` (Virtual Host) để định nghĩa máy chủ ảo lắng nghe trên một cổng (`listen 80;`) và nhận dạng tên miền hoặc IP cụ thể (`server_name;`).
* **Khai báo các khối `location`** bên trong khối `server` để định nghĩa quy tắc xử lý cho từng đường dẫn URI.

**[Slide 5: Khối location và Quy tắc so khớp đường dẫn (URI Matching)]**
Một kiến thức cực kỳ quan trọng mà các bạn đi phỏng vấn rất hay gặp là **Độ ưu tiên so khớp location** trong Nginx. Khi có nhiều block location cùng khai báo, Nginx sẽ chọn khối nào? Độ ưu tiên được tính từ cao xuống thấp như sau:
1. **Khớp chính xác (dấu `=`):** Ví dụ `location = /favicon.ico`. Chỉ khớp khi URI giống hệt 100%. Nginx sẽ chọn ngay khối này và dừng tìm kiếm.
2. **Khớp tiền tố ưu tiên (ký tự `^~`):** Ví dụ `location ^~ /assets/`. Nếu khớp tiền tố này, Nginx sẽ dừng quét các khối sử dụng biểu thức chính quy (Regex).
3. **Khớp Regex phân biệt hoa thường (`~`) hoặc không phân biệt hoa thường (`~*`):** Ví dụ `location ~ \.(jpg|png)$` (chỉ khớp chữ thường) và `location ~* \.json$` (khớp cả `.json` và `.JSON`).
4. **Khớp tiền tố mặc định (không có modifier):** Ví dụ `location /api`. Khớp với bất kỳ đường dẫn nào bắt đầu bằng `/api`. Đây là khối có độ ưu tiên thấp nhất.

---

### 2. Phần thực hành (Demo)

**[Live Demo: Viết file cấu hình con quickbite.conf mẫu]**
Bây giờ, thầy sẽ tạo thử một file cấu hình custom đặt trong thư mục `conf.d/` để các bạn thấy rõ cấu trúc.

*(Thao tác trên Terminal VPS)*:
- *Bước 1: Di chuyển vào thư mục `/etc/nginx/conf.d/` và tạo file cấu hình mới:*
  ```bash
  sudo nano /etc/nginx/conf.d/quickbite.conf
  ```
- *Bước 2: Viết một khối server độc lập (không khai báo http bao ngoài):*
  ```nginx
  server {
      listen 80;
      server_name _; # Khớp với mọi request đổ vào IP VPS

      access_log /var/log/nginx/quickbite_access.log;
      error_log /var/log/nginx/quickbite_error.log;

      location / {
          root /usr/share/nginx/html;
          index index.html index.htm;
      }
  }
  ```
- *Bước 3: Nhấn Ctrl + O để lưu và Ctrl + X để thoát.*
- *Bước 4: Luôn chạy lệnh kiểm tra lỗi cú pháp:*
  ```bash
  sudo nginx -t
  ```
  *(Giải thích): Lệnh này giúp quét toàn bộ file config con xem có bị thiếu dấu `;` hay sai khối khai báo không.*

---

## Lesson 3: Triển khai Nginx làm Reverse Proxy chuyển tiếp yêu cầu đến API Gateway

### 1. Phần lý thuyết

**[Slide 6: Chuyển tiếp request bằng proxy_pass]**
Chào các bạn. Trong bài học này, chúng ta sẽ thực hiện cấu hình Reverse Proxy thực tế để bẻ hướng toàn bộ request của Client vào dịch vụ API Gateway.
* Như đã phân tích, API Gateway của QuickBite chạy trong container Docker và đang map cổng `8080:8080` ra ngoài host VPS.
* Để chuyển tiếp request, trong khối `location /api` của Nginx, chúng ta sử dụng chỉ thị **`proxy_pass`** trỏ tới cổng 8080 của API Gateway trên localhost:
  ```nginx
  proxy_pass http://127.0.0.1:8080;
  ```
* Bằng cách này, chúng ta chỉ cần mở cổng 80 của Nginx ra ngoài Internet, còn cổng 8080 của API Gateway sẽ được tường lửa UFW chặn lại, ngăn chặn mọi kết nối trực tiếp không an toàn.

**[Slide 7: Bảo toàn thông tin Client qua HTTP Headers]**
Có một vấn đề kỹ thuật rất sắc sảo ở đây: Khi request đi qua Nginx Reverse Proxy, Nginx sẽ tạo ra một request mới để gửi tới API Gateway. Lúc này, IP nguồn của request gửi tới API Gateway sẽ bị đổi thành địa chỉ IP nội bộ của Nginx là `127.0.0.1`. 
* Làm thế nào để API Gateway và các microservices Spring Boot phía sau biết được địa chỉ IP thực tế của khách hàng (phục vụ ghi log, chặn spam)?
* Giải pháp là chúng ta sử dụng chỉ thị `proxy_set_header` để đính kèm thêm các header gốc trước khi chuyển tiếp:
  * `proxy_set_header Host $host;` (Bảo toàn host name).
  * `proxy_set_header X-Real-IP $remote_addr;` (Lưu IP thật của Client).
  * `proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;` (Ghi vết chuỗi IP proxy).
  * `proxy_set_header X-Forwarded-Proto $scheme;` (Lưu giao thức gốc HTTP/HTTPS).

**[Slide 8: Zero-Downtime Hot Reload]**
Khi chỉnh sửa file cấu hình Nginx trên môi trường đang chạy thực tế, tuyệt đối cấm sử dụng lệnh `systemctl restart nginx`. Nếu bạn restart dịch vụ, Nginx sẽ bị tắt đi trong vài giây, làm rớt toàn bộ kết nối hiện tại của khách hàng đang đặt đồ ăn, và nếu file config bị lỗi cú pháp, Nginx sẽ không bật lại được (gây sập hệ thống).
* Thay vào đó, chúng ta sử dụng cơ chế **Hot Reload**:
  ```bash
  sudo systemctl reload nginx
  ```
* Tiến trình Master của Nginx sẽ đọc cấu hình mới. Nếu hợp lệ, nó sẽ sinh worker mới chạy cấu hình mới và ra lệnh cho các worker cũ xử lý nốt các request dở dang rồi tự hủy. Quy trình này diễn ra tức thời và hoàn toàn không gây downtime.

---

### 2. Phần thực hành (Demo)

**[Live Demo: Cấu hình proxy chuyển tiếp request vào API Gateway]**
Bây giờ, thầy sẽ cập nhật tệp tin `quickbite.conf` trên VPS để thực hiện chuyển tiếp request.

*(Thao tác trên Terminal VPS)*:
- *Bước 1: Mở tệp tin cấu hình con `quickbite.conf`:*
  ```bash
  sudo nano /etc/nginx/conf.d/quickbite.conf
  ```
- *Bước 2: Thay đổi nội dung, thêm khối location `/api` để chuyển tiếp tới API Gateway ở cổng 8080 của host:*
  ```nginx
  server {
      listen 80;
      server_name _;

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
- *Bước 3: Lưu file và kiểm tra cú pháp:*
  ```bash
  sudo nginx -t
  ```
- *Bước 4: Chạy nạp nóng cấu hình không gián đoạn dịch vụ:*
  ```bash
  sudo systemctl reload nginx
  ```
- *Bước 5: Từ máy cá nhân, dùng lệnh cURL để gọi API của VPS qua cổng 80:*
  ```bash
  curl http://<vps_public_ip>/api/v1/restaurants
  ```
  *(Giải thích): Nhận về kết quả JSON thành công từ Spring Boot mà không cần gõ cổng 8080.*

---

## Lesson 4: Cấu hình tên miền (Domain) và mã hóa bảo mật SSL/HTTPS với Let's Encrypt

### 1. Phần lý thuyết

**[Slide 9: Trỏ tên miền và Sự nguy hiểm của HTTP thường]**
Chào các bạn. Để đưa hệ thống QuickBite vào hoạt động chuyên nghiệp, chúng ta không thể bắt người dùng truy cập qua địa chỉ IP thô dạng số được. Chúng ta cần một tên miền (ví dụ: `api.quickbite.com`).
* Bước đầu tiên là truy cập trình quản lý DNS và tạo một bản ghi **A Record** trỏ tên miền phụ `api` về đúng địa chỉ IP công khai của VPS.
* Tiếp theo là vấn đề bảo mật. Giao thức HTTP truyền tin dưới dạng văn bản thô (Plain text). Nếu người dùng đăng nhập bằng mạng Wi-Fi công cộng, hacker có thể dễ dàng nghe lén và chụp lại mật khẩu hoặc JWT token của họ. Do đó, bắt buộc chúng ta phải bọc toàn bộ hệ thống trong giao thức mã hóa bảo mật **HTTPS** (lắng nghe ở cổng 443).

**[Slide 10: Tự động hóa SSL bằng Let's Encrypt & Certbot]**
Để cấu hình HTTPS, chúng ta cần một chứng chỉ số SSL/TLS được cấp phát bởi một tổ chức Certificate Authority (CA) uy tín. Let's Encrypt là một CA phi lợi nhuận cung cấp chứng chỉ SSL hoàn toàn miễn phí.
* Để tự động hóa việc xin cấp phát và cài đặt chứng chỉ này, chúng ta sử dụng công cụ **Certbot** trên Ubuntu.
* Certbot sẽ tự động thực hiện quy trình xác thực quyền sở hữu tên miền của bạn với máy chủ Let's Encrypt, tải xuống chứng chỉ, lưu trữ tại thư mục `/etc/letsencrypt/` và tự động sửa đổi tệp cấu hình `quickbite.conf` của Nginx để chèn thêm các thông số SSL cổng 443.

**[Slide 11: HTTP to HTTPS Redirect & SSL Termination]**
Khi Certbot cấu hình thành công, nó sẽ tự động chèn thêm các luật chuyển hướng:
* **Redirect 301 HTTP to HTTPS:** Bất kỳ ai cố tình truy cập bằng giao thức `http://` cổng 80 sẽ lập tức được Nginx phản hồi mã lỗi `301 Moved Permanently` bắt buộc trình duyệt phải chuyển sang giao thức bảo mật `https://` cổng 443.
* **Cơ chế SSL Termination:** Nginx đóng vai trò là điểm kết thúc của kết nối mã hóa. Nó chịu trách nhiệm giải mã gói tin mã hóa từ Client gửi đến ở biên mạng của VPS. Sau đó, nó chuyển tiếp request dạng HTTP thô (đã giải mã) cho API Gateway trên localhost. Việc này giúp giảm tối đa tải tính toán CPU giải mã cho cụm microservices Java Spring Boot phía sau.
* **Tự động gia hạn:** Chứng chỉ Let's Encrypt có hạn 90 ngày, nhưng Certbot đã tự động thiết lập systemd timer để tự động chạy lệnh gia hạn khi chứng chỉ còn hạn dưới 30 ngày, DevOps hoàn toàn không cần can thiệp thủ công.

---

### 2. Phần thực hành (Demo)

**[Live Demo: Cài đặt Certbot và kích hoạt HTTPS cho QuickBite]**
Bây giờ, thầy sẽ thực hiện trỏ tên miền và chạy Certbot để kích hoạt ổ khóa xanh HTTPS cho VPS.

*(Thao tác cấu hình)*:
- *Bước 1: Trỏ bản ghi A tên miền `api.quickbite.com` về IP VPS.*
- *Bước 2: Cập nhật chỉ thị `server_name` trong file `quickbite.conf` thành tên miền của bạn:*
  ```nginx
  server_name api.quickbite.com;
  ```
- *Bước 3: Chạy lệnh test và reload Nginx.*
- *Bước 4: Cài đặt Certbot và plugin Nginx trên Ubuntu 24.04 LTS:*
  ```bash
  sudo apt update
  sudo apt install certbot python3-certbot-nginx -y
  ```
- *Bước 5: Chạy Certbot để tự động cấu hình SSL cho tên miền:*
  ```bash
  sudo certbot --nginx -d api.quickbite.com
  ```
  *(Giải thích): Nhập Email nhận cảnh báo và chọn các xác nhận đồng ý điều khoản. Certbot sẽ tự động chèn các cấu hình SSL cổng 443 và cấu hình Redirect 301 vào file `quickbite.conf`.*
- *Bước 6: Reload lại Nginx và truy cập thử bằng trình duyệt qua giao thức `https://api.quickbite.com/api/v1/users` để kiểm chứng ổ khóa xanh bảo mật thành công!*

---

*Buổi học Session 11 của chúng ta kết thúc tại đây. Cảm ơn các bạn và chúc các bạn thực hành cấu hình bảo mật Reverse Proxy Nginx thành công!*
