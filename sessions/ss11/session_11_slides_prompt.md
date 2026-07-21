# HƯỚNG DẪN CẤU TRÚC VÀ NỘI DUNG SLIDE (HSSF) - SESSION 11
## CHỦ ĐỀ: REVERSE PROXY VỚI NGINX TRONG MÔI TRƯỜNG PRODUCTION

Tài liệu này cung cấp thiết kế chi tiết từng slide theo chuẩn **HSSF (HTML Slide System Framework)** của Rikkei Education. Mỗi slide được định nghĩa rõ ràng về nhãn (`data-hssf-label`), cấu trúc layout component, nội dung hiển thị (tiếng Việt), các đoạn mã (code) và nội dung thuyết trình (Speaker Notes).

---

## I. BẢN ĐỒ SLIDE (SLIDE MAP)

| # | `data-hssf-label` | Chủ đề Slide (Focus) | HSSF Components chính |
|---|-------------------|----------------------|-----------------------|
| 1 | Title | Tiêu đề chính | `hssf-slide--title`, `hssf-title-block` |
| 2 | Agenda | Lộ trình buổi học | `hssf-header`, `hssf-agenda` |
| 3 | Sec-01 | Phân đoạn 01: Khái niệm & Luồng mạng | `hssf-slide--section`, `hssf-section-block` |
| 4 | Pain-Expose | Vấn đề mở cổng trực tiếp | `hssf-compare`, `hssf-callout--danger` |
| 5 | Concept-Proxy | Phân biệt Forward vs Reverse Proxy | `hssf-compare`, `hssf-list` |
| 6 | Diagram-Traffic | Sơ đồ luồng dữ liệu (Traffic Flow) | `hssf-diagram`, `hssf-flow`, `hssf-flow__node` |
| 7 | Sec-02 | Phân đoạn 02: Cấu trúc File Config | `hssf-slide--section`, `hssf-section-block` |
| 8 | Config-Inclusion | Nguyên tắc nạp file tại `conf.d/` | `hssf-callout--info`, `hssf-list` |
| 9 | Code-Blocks | Khối `server` và `location` | `hssf-columns`, `hssf-code` |
| 10| Table-Priority | Quy tắc ưu tiên so khớp Location | `hssf-heading`, `hssf-table--striped` |
| 11| Sec-03 | Phân đoạn 03: Proxy API Gateway | `hssf-slide--section`, `hssf-section-block` |
| 12| Steps-Proxy | Cấu hình `proxy_pass` & headers | `hssf-steps`, `hssf-code`, `hssf-columns` |
| 13| Steps-Reload | Quy trình kiểm tra & Nạp nóng | `hssf-timeline` + fragments |
| 14| Sec-04 | Phân đoạn 04: Domain & SSL/HTTPS | `hssf-slide--section`, `hssf-section-block` |
| 15| Concept-SSL | Tầm quan trọng của SSL & Certbot | `hssf-columns`, `hssf-card` |
| 16| Steps-Certbot | Từng bước cài Certbot xin SSL | `hssf-steps`, `hssf-code` |
| 17| SSL-Termination | Cơ chế SSL Termination & Redirect 301 | `hssf-compare`, `hssf-callout--tip` |
| 18| Summary | Tổng kết bài học | `hssf-list`, `hssf-stat` |
| 19| End | Kết thúc slide | `hssf-brand-end` |

---

## II. CHI TIẾT CẤU TRÚC VÀ NỘI DUNG TỪNG SLIDE

### SLIDE 1: Title
* **HSSF Classes:** `hssf-slide hssf-slide--title`
* **Layout / Components:**
  * `hssf-title-block` có `hssf-accent--bar-left`
  * Eyebrow: `SESSION 11: DEVOPS IN ACTION`
  * Title: `Reverse Proxy với Nginx trên Production`
  * Meta: `Rikkei Academy - Bộ môn DevOps`
* **Speaker Notes:** Chào các bạn học viên. Hôm nay chúng ta sẽ bước sang Session 11, học cách thiết lập cổng bảo vệ biên Nginx Reverse Proxy cho cụm microservices QuickBite trên Production.

---

### SLIDE 2: Agenda
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Nội dung bài học", Subtitle: "4 phần trọng tâm vận hành hệ thống")
  * `hssf-agenda`:
    * Lộ trình 1: Khái niệm & vai trò của Reverse Proxy trong Microservices
    * Lộ trình 2: Cú pháp tệp cấu hình Nginx và độ ưu tiên so khớp Location
    * Lộ trình 3: Thực hành cấu hình Proxy tới API Gateway & Zero-Downtime Reload
    * Lộ trình 4: Trỏ Domain và tự động hóa mã hóa SSL/HTTPS với Certbot
* **Speaker Notes:** Đây là lộ trình 4 phần chúng ta sẽ đi qua. Từ lý thuyết luồng đi của gói tin, đến thực hành cấu hình, và cuối cùng là bảo mật HTTPS.

---

### SLIDE 3: Sec-01 (Divider)
* **HSSF Classes:** `hssf-slide hssf-slide--section`
* **Layout / Components:**
  * `hssf-section-block`
  * `hssf-footer--light`
  * Big Number: `01`
  * Title: `Reverse Proxy & Luồng Giao Thức Mạng`
* **Speaker Notes:** Chúng ta sẽ bắt đầu phần 1, tìm hiểu lý do tại sao không được mở cổng trực tiếp và sự khác nhau giữa Proxy xuôi và ngược.

---

### SLIDE 4: Pain-Expose (Vấn đề & Giải pháp)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-compare` chia 2 cột:
    * Cột Trái (Cons - Không có Reverse Proxy):
      * Đỏ/Danger: Mở toang các cổng `8080`, `8081` ra Internet.
      * Hacker dò quét port (Port scanning) và tấn công trực diện vào Spring Boot.
      * Phải cài đặt cấu hình SSL/HTTPS cho từng service độc lập.
    * Cột Phải (Pros - Có Nginx Reverse Proxy):
      * Xanh/Success: Cô lập toàn bộ cổng backend trong mạng nội bộ.
      * Chỉ mở cổng `80`/`443` ở biên.
      * Tập trung giải mã SSL tại Nginx (SSL Termination).
  * `hssf-callout--danger` ở chân slide: "Expose trực tiếp cổng ứng dụng Spring Boot ra Internet là một lỗ hổng bảo mật mức nghiêm trọng."
* **Speaker Notes:** Trên Production, việc mở trực tiếp cổng 8080 tạo diện tấn công lớn. Giải pháp là đặt Nginx đứng trước che chở cho toàn bộ backend phía sau.

---

### SLIDE 5: Concept-Proxy (Phân biệt Forward vs Reverse Proxy)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-compare` chia 2 cột:
    * Cột 1: `Forward Proxy (Proxy Xuôi)`
      * Đứng trước: Client (Người dùng).
      * Đại diện cho: Client gửi request ra ngoài.
      * Mục đích: Vượt tường lửa, ẩn danh IP Client (Ví dụ: VPN 1.1.1.1, Proxy công ty).
    * Cột 2: `Reverse Proxy (Proxy Ngược)`
      * Đứng trước: Server (Hệ thống).
      * Đại diện cho: Cụm server nhận request đổ vào.
      * Mục đích: Định tuyến, cân bằng tải, bảo mật biên (Ví dụ: Nginx).
* **Speaker Notes:** Forward proxy bảo vệ người dùng, còn Reverse proxy bảo vệ máy chủ. Client gọi API chỉ thấy duy nhất IP của Nginx.

---

### SLIDE 6: Diagram-Traffic (Sơ đồ luồng mạng)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Sơ đồ Traffic Flow", Subtitle: "Từ Internet đến Microservices")
  * `hssf-diagram` chứa `hssf-flow` với các node:
    * Node 1: `[ Client ]` -> Arrow -> Node 2 (Primary): `[ Nginx (Port 80/443) ]` (Edge Firewall UFW)
    * Node 2 -> Arrow (proxy_pass) -> Node 3 (Soft): `[ API Gateway (8080) ]` (Localhost)
    * Node 3 -> Connector -> Node 4: `[ user-service:8081 ]` & Node 5: `[ order-service:8083 ]` (Mạng nội bộ Docker)
* **Speaker Notes:** Request từ Client gửi đến cổng 80 của Nginx. Nginx dùng proxy_pass đẩy tới API Gateway chạy ở localhost:8080. Từ đây, API Gateway điều phối qua mạng nội bộ Docker.

---

### SLIDE 7: Sec-02 (Divider)
* **HSSF Classes:** `hssf-slide hssf-slide--section`
* **Layout / Components:**
  * `hssf-section-block`
  * Big Number: `02`
  * Title: `Cấu trúc Tệp tin Cấu hình Nginx`
* **Speaker Notes:** Chúng ta bước sang phần 2, tìm hiểu cách viết file cấu hình Nginx đúng tiêu chuẩn công nghiệp.

---

### SLIDE 8: Config-Inclusion (Quy trình nạp file)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-callout--info` (Title: "Nguyên tắc Vận hành Production", Body: "Tuyệt đối không chỉnh sửa tệp tin gốc nginx.conf của hệ điều hành.")
  * `hssf-list` giải thích cơ chế inclusion:
    * Nginx tự động import cấu hình bằng chỉ thị `include /etc/nginx/conf.d/*.conf;` bên trong khối `http` gốc.
    * Kỹ sư chỉ viết tệp cấu hình riêng (ví dụ: `quickbite.conf`) đặt vào thư mục `/etc/nginx/conf.d/`.
    * Tệp cấu hình con chỉ chứa các khối `server { ... }`, tuyệt đối không khai báo lặp lại khối `http { ... }`.
* **Speaker Notes:** Để quản lý mã nguồn sạch sẽ, chúng ta giữ nguyên file nginx.conf gốc và viết file cấu hình dự án của mình vào thư mục `conf.d/`.

---

### SLIDE 9: Code-Blocks (Cấu trúc Server & Location)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-columns--1-1` chia đôi màn hình:
    * Cột 1: Giải thích các chỉ thị:
      * `listen 80`: Lắng nghe cổng HTTP.
      * `server_name`: Tên miền của website.
      * `location`: Khối xử lý đường dẫn URI.
    * Cột 2: `hssf-code` chứa cú pháp:
      ```nginx
      server {
          listen 80;
          server_name api.quickbite.com;

          location / {
              root /usr/share/nginx/html;
              index index.html;
          }
      }
      ```
* **Speaker Notes:** Khối `server` định nghĩa một Virtual Host (máy chủ ảo). Bên trong nó chứa các khối `location` nhỏ hơn để phân loại request dựa theo đường dẫn URL.

---

### SLIDE 10: Table-Priority (Quy tắc so khớp location)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-heading` (Title: "Quy tắc so khớp Location", Subtitle: "Độ ưu tiên xử lý của Nginx")
  * `hssf-table--striped`:
    | Ký tự Modifier | Loại so khớp | Độ ưu tiên | Ví dụ cấu hình |
    | :---: | :--- | :---: | :--- |
    | `=` | Khớp chính xác (Exact Match) | **1 (Cao nhất)** | `location = /favicon.ico` |
    | `^~` | Khớp tiền tố ưu tiên (Không quét Regex) | **2** | `location ^~ /assets/` |
    | `~` | Khớp Regex (Phân biệt chữ hoa/thường) | **3** | `location ~ \.(jpg|png)$` |
    | `~*` | Khớp Regex (Không phân biệt chữ hoa/thường) | **3** | `location ~* \.json$` |
    | *(Không có)* | Khớp tiền tố mặc định (Prefix Match) | **4 (Thấp nhất)** | `location /api/` |
* **Speaker Notes:** Khi có nhiều block location cùng khớp một URI, Nginx sẽ áp dụng thứ tự ưu tiên: Dấu `=` đứng đầu, tiếp theo là `^~`, sau đó là Regex và cuối cùng là so khớp tiền tố mặc định.

---

### SLIDE 11: Sec-03 (Divider)
* **HSSF Classes:** `hssf-slide hssf-slide--section`
* **Layout / Components:**
  * `hssf-section-block`
  * Big Number: `03`
  * Title: `Thực hành cấu hình Proxy tới API Gateway`
* **Speaker Notes:** Phần 3, chúng ta sẽ bắt tay cấu hình chuyển tiếp request tới API Gateway và giải quyết bài toán mất IP thật của Client.

---

### SLIDE 12: Steps-Proxy (Cấu hình proxy_pass và Headers)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-columns--1-1` chia đôi màn hình:
    * Cột 1: `hssf-steps`
      * Step 1: Khai báo proxy_pass tới IP:Port đích.
      * Step 2: Ghi nhận Host name gốc của Client.
      * Step 3: Truyền IP thật của Client xuống Backend.
    * Cột 2: `hssf-code`
      ```nginx
      location /api/ {
          proxy_pass http://127.0.0.1:8080/;
          
          # Bảo toàn thông tin Client
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
      }
      ```
* **Speaker Notes:** Chỉ dùng `proxy_pass` là chưa đủ, backend sẽ thấy IP nguồn là của Nginx (`127.0.0.1`). Chúng ta cần khai báo thêm các header `X-Real-IP` và `X-Forwarded-For` để truyền IP thật của client xuống backend Spring Boot.

---

### SLIDE 13: Steps-Reload (Quy trình nạp nóng Zero-Downtime)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Quy trình DevOps chuẩn khi đổi cấu hình", Subtitle: "Không gây gián đoạn hệ thống (Zero-Downtime)")
  * `hssf-timeline`:
    * Item 1: `Bước 1: Lưu file cấu hình` (Cấu hình xong tệp `.conf` trong `/etc/nginx/conf.d/`).
    * Item 2: `Bước 2: Kiểm tra lỗi cú pháp` (Chạy lệnh `sudo nginx -t`. Nếu báo `syntax is ok`, chuyển sang bước 3).
    * Item 3: `Bước 3: Nạp nóng cấu hình` (Chạy lệnh `sudo systemctl reload nginx`).
  * `hssf-callout--warning` ở chân slide: "Tuyệt đối không dùng lệnh restart trên môi trường Production vì sẽ làm ngắt quãng các kết nối hiện tại của khách hàng."
* **Speaker Notes:** Đây là quy trình bắt buộc trên Production: Kiểm tra cú pháp trước bằng `nginx -t`, sau đó nạp nóng bằng lệnh `reload`. Các request đang xử lý của khách hàng sẽ được giữ nguyên mà không bị ngắt quãng.

---

### SLIDE 14: Sec-04 (Divider)
* **HSSF Classes:** `hssf-slide hssf-slide--section`
* **Layout / Components:**
  * `hssf-section-block`
  * Big Number: `04`
  * Title: `Mã hóa bảo mật SSL/HTTPS & Let's Encrypt`
* **Speaker Notes:** Chúng ta bước vào phần cuối, thực hiện trỏ tên miền và cài đặt chứng chỉ SSL tự động để kích hoạt HTTPS.

---

### SLIDE 15: Concept-SSL (Bảo mật HTTPS)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-columns--2` chia làm 2 card:
    * Card 1 (hssf-card --outline):
      * Icon: `🔒`
      * Title: `Tại sao cần HTTPS?`
      * Body: HTTP truyền tin dạng Plaintext dễ bị nghe lén (Man-in-the-middle). HTTPS mã hóa toàn bộ dữ liệu (JWT Token, Password, DB details) trên đường truyền.
    * Card 2 (hssf-card --outline):
      * Icon: `🤖`
      * Title: `Certbot & Let's Encrypt`
      * Body: Cung cấp chứng chỉ SSL miễn phí tự động. Certbot tự động xác thực tên miền qua DNS và ghi đè cấu hình SSL an toàn vào file Nginx.
* **Speaker Notes:** Trên môi trường Internet công cộng, việc truyền Token qua HTTP thường vô cùng nguy hiểm. Chúng ta sẽ dùng Certbot để tự động cài đặt chứng chỉ SSL miễn phí từ Let's Encrypt.

---

### SLIDE 16: Steps-Certbot (Cài đặt thực tế)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-columns--1-1` chia đôi màn hình:
    * Cột 1: `hssf-steps`
      * Step 1: Tạo bản ghi A trỏ tên miền về IP VPS.
      * Step 2: Cài đặt Certbot và plugin Nginx.
      * Step 3: Chạy Certbot xin và cài SSL.
    * Cột 2: `hssf-code`
      ```bash
      # 1. Cài đặt Certbot trên Ubuntu
      sudo apt update
      sudo apt install certbot python3-certbot-nginx -y

      # 2. Chạy xin và tự cấu hình SSL cho tên miền
      sudo certbot --nginx -d api.quickbite.com
      ```
* **Speaker Notes:** Đầu tiên bạn phải chắc chắn tên miền đã được trỏ về IP VPS của bạn. Sau đó chạy lệnh certbot với cờ `--nginx`, Certbot sẽ tự tìm file cấu hình có `server_name` tương ứng và chèn code cấu hình HTTPS cổng 443 vào đó.

---

### SLIDE 17: SSL-Termination (Cơ chế SSL Termination & Redirect)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-compare` chia 2 cột:
    * Cột 1: `SSL Termination (Giải mã tại biên)`
      * Client gửi request HTTPS (cổng 443) tới Nginx.
      * Nginx giải mã SSL thành HTTP thường.
      * Nginx chuyển tiếp HTTP thường tới API Gateway (localhost:8080).
      * *Lợi ích:* Giải phóng CPU cho các container Java Spring Boot.
    * Cột 2: `Redirect 301 (Chuyển hướng bắt buộc)`
      * Người dùng gõ `http://api.quickbite.com`.
      * Nginx trả về mã lỗi `301 Moved Permanently`.
      * Trình duyệt tự động chuyển hướng lên `https://api.quickbite.com`.
  * `hssf-callout--tip` ở cuối: "Certbot cũng tự động cấu hình một cron job để tự gia hạn chứng chỉ Let's Encrypt mỗi khi gần hết hạn 90 ngày."
* **Speaker Notes:** Cơ chế giải mã SSL tại Nginx được gọi là SSL Termination. Việc này giúp các microservices Spring Boot không cần tốn năng lực xử lý CPU để giải mã HTTPS, mà chỉ cần nhận gói tin HTTP thường trong mạng nội bộ an toàn.

---

### SLIDE 18: Summary
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Tổng kết Session 11", Subtitle: "Những lưu ý quan trọng khi triển khai Nginx")
  * `hssf-list`:
    * Luôn cô lập cổng Microservices, chỉ mở cổng 80/443 ra ngoài qua Nginx.
    * Tận dụng folder `conf.d/` để nạp cấu hình tự động.
    * Luôn kiểm tra cú pháp bằng `nginx -t` trước khi chạy `systemctl reload nginx`.
    * Cấu hình đầy đủ các header `X-Real-IP` để backend nhận biết được IP thật của Client.
* **Speaker Notes:** Tóm lại, Nginx là bức tường bảo vệ biên cực kỳ quan trọng cho hệ thống. Hãy ghi nhớ quy trình reload không downtime và cấu hình bảo toàn IP thật của client.

---

### SLIDE 19: End
* **HSSF Classes:** `hssf-slide hssf-slide--section`
* **Layout / Components:**
  * `hssf-brand-end`
    * Kicker: `DEVOPS IN ACTION`
    * Title: `Cảm ơn các bạn đã lắng nghe!`
    * Org: `Rikkei Academy - Rikkei Education`
  * `hssf-footer--light hssf-footer--nopage`
* **Speaker Notes:** Cảm ơn các bạn. Bây giờ chúng ta sẽ bắt đầu phần làm bài tập thực hành Lab cấu hình Nginx Reverse Proxy trên VPS.
