# SESSION 10: TRIỂN KHAI HỆ THỐNG LÊN VPS (VIRTUAL PRIVATE SERVER)

## LESSON 04: Đồng bộ cấu hình qua Git bằng SSH Key và vận hành cụm Microservices

---

### 1. Phân biệt hai loại SSH Key trong quy trình triển khai

Trong quy trình triển khai hạ tầng DevOps chuyên nghiệp, bạn sẽ làm việc với hai loại kết nối SSH độc lập. Lập trình viên thường bị nhầm lẫn giữa hai loại khóa này:

1. **SSH Key 1 (Local máy cá nhân -> VPS):**
   * *Nơi tạo:* Sinh ra trên máy tính cá nhân của bạn.
   * *Nơi đặt Public Key:* VPS (`/home/deployer/.ssh/authorized_keys`).
   * *Mục đích:* Xác thực cho phép bạn đăng nhập từ máy tính cá nhân vào quản trị VPS.
2. **SSH Key 2 (VPS -> GitHub/GitLab):**
   * *Nơi tạo:* Sinh ra trực tiếp trên máy chủ VPS bằng tài khoản `deployer`.
   * *Nơi đặt Public Key:* Cài đặt trong mục **SSH and GPG keys** trên tài khoản cá nhân GitHub/GitLab của bạn.
   * *Mục đích:* Cấp quyền cho VPS truy cập, kéo mã nguồn (Git Clone/Pull) từ các kho chứa Git riêng tư (Private Repository) mà tài khoản của bạn được cấp quyền về máy chủ.

```text
  [ Máy Local ] ──(SSH Key 1)──► [ VPS (deployer) ] ──(SSH Key 2)──► [ GitHub (Private Repo) ]
```

---

### 2. Thực hành thiết lập kết nối Git và vận hành Microservices

#### Bước 1: Sinh khóa SSH Key trên VPS để kết nối GitHub
Tại terminal của VPS (đang đăng nhập bằng user `deployer`), chạy lệnh tạo khóa mới:

```bash
ssh-keygen -t ed25519 -C "vps_to_github_key"
```
*(Nhấn `Enter` liên tục để lưu mặc định tại `/home/deployer/.ssh/id_ed25519` và không cài passphrase).*

#### Bước 2: Lấy khóa công khai và cấu hình SSH Key trên GitHub
Hiển thị nội dung khóa công khai vừa sinh ra trên VPS:

```bash
cat ~/.ssh/id_ed25519.pub
```

Sao chép toàn bộ chuỗi ký tự hiển thị trên màn hình (bắt đầu bằng `ssh-ed25519 ...`).

1. Đăng nhập vào trang web GitHub, click vào ảnh đại diện ở góc trên cùng bên phải và chọn **Settings**.
2. Ở thanh menu bên trái, tìm và chọn **SSH and GPG keys** -> Nhấn **New SSH key**.
3. Điền thông tin:
   * **Title:** Nhập tên gợi nhớ để phân biệt (Ví dụ: `quickbite_vps_deployer`).
   * **Key type:** Giữ nguyên `Authentication Key`.
   * **Key:** Dán toàn bộ nội dung khóa công khai đã sao chép ở trên vào đây.
4. Nhấn **Add SSH key**.


#### Bước 3: Kiểm tra kết nối từ VPS đến GitHub
Quay lại terminal của VPS, chạy lệnh kiểm tra kết nối:

```bash
ssh -T git@github.com
```

Nếu nhận lời chào mừng từ GitHub như dưới đây, kết nối đã thành công:
```text
Hi your_username/quickbite-base! You've successfully authenticated, but GitHub does not provide shell access.
```

#### Bước 4: Clone dự án về VPS
Tạo thư mục dự án và clone mã nguồn về:

```bash
mkdir -p ~/projects && cd ~/projects

# Thực hiện clone dự án sử dụng giao thức SSH
git clone git@github.com:your_username/quickbite-base.git
```
*(Thay thế `your_username/quickbite-base.git` bằng link SSH Repository thực tế của bạn).*

#### Bước 5: Cấu hình tệp tin môi trường `.env`
Di chuyển vào thư mục hạ tầng dự án vừa clone về (chứa tệp tin `docker-compose.yml`):

```bash
cd ~/projects/quickbite-base
```

Tạo mới và chỉnh sửa file `.env` bằng trình soạn thảo `nano`:
```bash
nano .env
```

Khai báo các biến cấu hình phù hợp cho môi trường Production thực tế trên VPS:
```text
# Cấu hình Database
DB_HOST=quickbite-db
DB_PORT=5432
DB_NAME=quickbite
DB_USERNAME=postgres
DB_PASSWORD=MySecurePassword123!

# Cấu hình cổng chạy ứng dụng Backend
USER_SERVICE_PORT=8081
RESTAURANT_SERVICE_PORT=8082
ORDER_SERVICE_PORT=8083
NOTIFICATION_SERVICE_PORT=8084

# Cấu hình bảo mật JWT Token
JWT_SECRET_KEY=zX8$9bQ!Wd2@mL7#pKs5*tVnE&cR3_yUeI1oP0aSdfGhJkl
```
*(Nhấn `Ctrl + O` -> `Enter` để lưu, và `Ctrl + X` để thoát).*

#### Bước 6: Khởi chạy cụm Microservices bằng Docker Compose
Tiến hành tải các image cần thiết và khởi chạy toàn bộ các dịch vụ ở chế độ chạy ngầm (detached mode):

```bash
docker compose up -d
```

#### Bước 7: Kiểm tra và giám sát các dịch vụ
Sau khi khởi chạy, sử dụng các câu lệnh sau để theo dõi sức khỏe hệ thống:

```bash
# 1. Kiểm tra danh sách container và trạng thái sức khoẻ (healthy)
docker compose ps

# 2. Xem logs hoạt động thời gian thực của các container
docker compose logs -f

# 3. Giám sát lượng CPU, RAM tiêu thụ thực tế của từng container trên VPS
docker stats
```

---

### 3. Nguyên tắc bảo mật tệp cấu hình `.env`

> [!IMPORTANT]
> Tệp `.env` chứa các thông số nhạy cảm và bảo mật cao nhất của dự án chạy thực tế (như mật khẩu cơ sở dữ liệu, khoá bí mật JWT token).
> * **Không bao giờ được đẩy file `.env` lên Git Repository.**
> * Đảm bảo file `.env` đã được khai báo bên trong tệp `.gitignore` ở thư mục gốc của dự án.
> * Mỗi môi trường (Local, Staging, Production) sẽ tự viết và quản lý một file `.env` vật lý riêng biệt tại máy chủ đó.

---

### 4. Hiểu lầm thường gặp

* **Hiểu sai:** Nghĩ rằng chỉ cần cài đặt công cụ Git trên VPS là có thể chạy lệnh `git clone` bất kỳ dự án nào từ GitHub về máy chủ mà không cần cấu hình khóa xác thực.
* **Đính chính:** Đối với các kho chứa mã nguồn để ở chế độ riêng tư (Private Repository), máy chủ GitHub sẽ từ chối mọi yêu cầu truy cập ẩn danh. SSH Key cung cấp một kênh xác thực tự động và an toàn cao mà không yêu cầu bạn phải gõ tài khoản/mật khẩu GitHub trực tiếp dưới dạng plain text trên máy chủ VPS.

---

### 5. Câu hỏi tự luận đánh giá nhanh

#### Câu 1 (Hiểu bản chất)
Phân biệt sự khác nhau về vai trò và vị trí lưu trữ của SSH Key dùng đăng nhập VPS (Local -> VPS) và SSH Key dùng để clone code từ GitHub (VPS -> GitHub). Sẽ có nguy cơ bảo mật gì nếu bạn sao chép Private Key của máy cá nhân lên VPS?
* *Gợi ý:* 
  * *SSH Key đăng nhập (Local -> VPS):* Private Key nằm ở máy local, Public Key nằm trên VPS. Dùng để đăng nhập quản trị VPS.
  * *SSH Key clone code (VPS -> GitHub):* Private Key nằm trên VPS, Public Key cấu hình trên **Settings -> SSH and GPG keys** của GitHub. Dùng để VPS có quyền kéo code từ Git về.
  * *Nguy cơ:* Nếu bạn đưa Private Key cá nhân (dùng để đăng nhập VPS) lên lưu trữ trên VPS, khi máy chủ VPS bị hacker xâm nhập hoặc bị lộ tệp tin cấu hình, kẻ xấu có thể lấy được khóa này và dùng nó để đăng nhập ngược trở lại vào các hệ thống khác của bạn (hoặc máy local nếu mở cổng), vi phạm nguyên tắc cô lập bảo mật.

#### Câu 2 (Phân tích)
Tại sao tệp tin `.env` bắt buộc phải nằm trong `.gitignore`? Nếu một thành viên trong nhóm vô tình push tệp tin `.env` của môi trường Production lên một public repository trên GitHub, bạn cần phải xử lý tình huống này thế nào để bảo mật lại hệ thống?
* *Gợi ý:* Tệp `.env` chứa toàn bộ mật khẩu cơ sở dữ liệu, khoá bí mật JWT kết nối và các token dịch vụ của môi trường chạy thật. Nếu bị đưa lên Git công khai, bất kỳ ai cũng có thể đọc được và hack vào hệ thống. Trong trường hợp bị lộ, bạn bắt buộc phải:
  1. Lập tức đổi toàn bộ các mật khẩu database, khoá bí mật JWT và token dịch vụ đã khai báo trong file `.env` đó trên máy chủ VPS.
  2. Sử dụng các công cụ như `git-filter-repo` hoặc BFG Repo-Cleaner để xóa sạch lịch sử commit chứa file `.env` này trên GitHub, hoặc xóa và tạo lại repo mới nếu cần thiết.

#### Câu 3 (Nâng cao)
Khi chạy lệnh giám sát `docker stats` trên VPS, bạn nhận thấy lượng RAM tiêu thụ của container `order-service` tăng liên tục theo thời gian và không có dấu hiệu giảm xuống ngay cả khi không có request nào đổ vào hệ thống, sau đó container bị crash đột ngột với mã Exit Code `137`. Hãy phân tích hiện tượng này, cho biết Exit Code `137` có ý nghĩa gì và đề xuất phương pháp chẩn đoán nguyên nhân lỗi trong mã nguồn Java Spring Boot.
* *Gợi ý:* 
  * *Phân tích hiện tượng:* Container bị rò rỉ bộ nhớ (Memory Leak) trong JVM, dẫn đến tràn bộ nhớ Heap.
  * *Exit Code 137:* Có ý nghĩa là tiến trình trong container đã bị nhân Linux (Kernel) cưỡng bức tắt bằng tín hiệu `SIGKILL` (do công cụ Out-Of-Memory Killer thực thi để cứu hệ điều hành không bị sập khi cạn kiệt RAM vật lý của VPS).
  * *Phương pháp chẩn đoán:* Bật cấu hình JVM option sinh dump file khi tràn bộ nhớ (`-XX:+HeapDumpOnOutOfMemoryError`), sử dụng các công cụ như Eclipse Memory Analyzer (MAT) hoặc VisualVM để phân tích file dump đó nhằm tìm ra các đối tượng Java nào đang chiếm bộ nhớ lớn mà không được Garbage Collector (GC) giải phóng.

