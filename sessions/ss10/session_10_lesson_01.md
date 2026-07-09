# SESSION 10: TRIỂN KHAI HỆ THỐNG LÊN VPS (VIRTUAL PRIVATE SERVER)

## LESSON 01: Giới thiệu VPS và thiết lập kết nối cơ bản qua Bitvise Client

---

### 1. Khái niệm VPS và lựa chọn cấu hình hạ tầng

#### 1.1 Virtual Private Server (VPS) là gì?
**Virtual Private Server (VPS)** là một máy chủ ảo được phân tách từ một máy chủ vật lý vật lý thông qua công nghệ ảo hóa (như KVM, VMware). Mỗi VPS chạy hệ điều hành độc lập (ở đây chúng ta sử dụng **Ubuntu Server 24.04 LTS**), có toàn quyền quản trị cao nhất (`root`) và tài nguyên phần cứng (CPU, RAM, SSD) được cam kết riêng biệt.

#### 1.2 So sánh VPS với Cloud Server và PaaS/Serverless

| Tiêu chí | VPS truyền thống | Cloud Server (AWS EC2, GCE) | PaaS / Serverless (Heroku, Render) |
| :--- | :--- | :--- | :--- |
| **Bản chất** | Một máy chủ ảo cố định trên một máy vật lý | Cụm máy chủ ảo phân tán trên hạ tầng Cloud lớn | Môi trường chạy ứng dụng được quản lý sẵn |
| **Khả năng co giãn** | Khó nâng cấp nóng, tài nguyên cố định | Auto-scaling tự động tăng giảm theo tải thực tế | Tự động hoàn toàn, chỉ cần deploy mã nguồn |
| **Mức độ kiểm soát** | Toàn quyền (Root OS, Network, Kernel) | Toàn quyền kiểm soát ảo hóa sâu rộng | Bị giới hạn cấu hình hệ điều hành & cổng mạng |
| **Chi phí** | Rất rẻ, trả cố định theo tháng | Đắt hơn, tính theo giờ/phút sử dụng | Rất đắt khi scale, miễn phí/rẻ lúc thử nghiệm |
| **Độ phức tạp vận hành** | Trung bình (cần tự cài OS, bảo mật) | Cao (nhiều dịch vụ đi kèm IAM, VPC) | Thấp (chỉ tập trung vào code) |

#### 1.3 Nguyên tắc lựa chọn cấu hình cho hệ thống QuickBite Microservices
Hệ thống QuickBite bao gồm 4 container Spring Boot (`user-service`, `restaurant-service`, `order-service`, `notification-service`) cùng hệ quản trị cơ sở dữ liệu (PostgreSQL/MySQL).
* **Khuyến nghị cấu hình tối thiểu:** **2 vCPUs và 2GB RAM** (nếu tốt hơn nên là 4GB RAM).
* **Ổ cứng:** Tối thiểu **20GB - 30GB SSD** để chứa hệ điều hành, docker images và tệp tin logs.

---

### 2. Kết nối VPS bằng Terminal truyền thống

Sau khi mua VPS từ các nhà cung cấp (Clouding, Vietnix, Linode, DigitalOcean...), bạn sẽ được cung cấp:
* **IP công khai (Public IP)** (Ví dụ: `103.82.20.15`)
* **Tài khoản mặc định:** `root`
* **Mật khẩu root** hoặc **SSH Key** ban đầu.

Để thực hiện kết nối đầu tiên bằng công cụ dòng lệnh mặc định (Command Prompt hoặc PowerShell trên Windows, Terminal trên macOS/Linux), ta gõ:

```bash
ssh root@<vps_public_ip>
# Ví dụ: ssh root@103.82.20.15
```

Hệ thống sẽ hỏi bạn có tin tưởng dấu vân tay khóa (Host key fingerprint) của server này không. Nhập `yes`, sau đó nhập mật khẩu root được cấp.

```text
The authenticity of host '103.82.20.15 (103.82.20.15)' can't be established.
ED25519 key fingerprint is SHA256:7u/b8H8v...
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
root@103.82.20.15's password:
```

#### 2.1 Các Pain Points (Hạn chế) khi quản trị qua Terminal thông thường
Mặc dù Terminal mặc định rất nhẹ, nhưng việc vận hành lâu dài trên môi trường Production bằng Terminal thuần túy sẽ gặp các khó khăn sau:
1. **Quản lý phiên (Session) kém:** Mỗi lần mở máy làm việc, bạn phải tìm lại địa chỉ IP, cổng SSH (nếu đã đổi) và gõ lại lệnh SSH thủ công.
2. **Nhập mật khẩu lặp đi lặp lại:** Gõ mật khẩu dài, phức tạp bằng tay dễ sai sót và mất thời gian.
3. **Thiếu trình quản lý file trực quan:** Khi cần xem cấu hình, nạp file `.env`, tải file log hoặc upload các file tài nguyên, bạn phải dùng các dòng lệnh phức tạp (`scp`, `rsync`) hoặc các editor thô sơ trên terminal như `nano`, `vi`.
4. **Mất kết nối đột ngột:** Nếu mạng chập chờn, phiên SSH bị đứt, câu lệnh đang chạy dở sẽ bị ngắt quãng.

---

### 3. Giải pháp: Sử dụng Bitvise SSH Client

Để giải quyết các pain points trên, chúng ta sử dụng **Bitvise SSH Client** (dành cho hệ điều hành Windows). Công cụ này tích hợp cả terminal dòng lệnh và trình quản lý tệp tin đồ họa bảo mật (SFTP) trong cùng một giao diện trực quan.

#### 3.1 Cấu hình lưu Session đăng nhập trên Bitvise SSH Client
1. Tải và cài đặt **Bitvise SSH Client** từ trang chủ.
2. Mở ứng dụng, tại tab **Login**, điền các thông tin sau:
   * **Host:** Nhập IP công khai của VPS (Ví dụ: `103.82.20.15`).
   * **Port:** `22` (Cổng SSH mặc định).
   * **Username:** `root`.
   * **Initial Method:** Chọn `password`.
   * **Password:** Nhập mật khẩu root của VPS.
3. Nhấn nút **Save Profile As** ở bảng bên trái để lưu lại cấu hình dưới dạng tệp tin profile (Ví dụ: `quickbite_vps_root.tlp`). Lần sau, bạn chỉ cần nạp lại file `.tlp` này là có thể kết nối ngay lập tức mà không cần nhập lại IP hay mật khẩu.
4. Nhấn **Log in** để thực hiện kết nối.

#### 3.2 Giao diện làm việc của Bitvise
Khi kết nối thành công, Bitvise không tự động hiển thị cửa sổ dòng lệnh hay truyền file. Thay vào đó, bạn sẽ thấy các tùy chọn/biểu tượng nằm ở thanh menu bên trái:
* **New terminal console:** Bấm vào đây để mở cửa sổ Terminal dòng lệnh Linux giúp bạn gõ lệnh trực tiếp trên VPS.
* **New SFTP window:** Bấm vào đây để mở trình duyệt tệp tin đồ họa bảo mật. Giao diện hiển thị hai cột: cột bên trái là thư mục local của máy bạn, cột bên phải là thư mục trên VPS. Việc kéo thả tệp tin qua lại giữa hai cột giúp truyền tải dữ liệu rất nhanh chóng và tiện lợi.

---

### 4. Thực hành: Cập nhật hệ thống VPS đầu tiên

Ngay sau khi kết nối thành công với quyền `root`, công việc bắt buộc đầu tiên để đảm bảo hệ thống an toàn và ổn định là cập nhật danh sách gói phần mềm và vá các lỗ hổng bảo mật của hệ điều hành.

Gõ lệnh sau vào cửa sổ Terminal:

```bash
sudo apt update && sudo apt upgrade -y
```

* **`apt update`**: Tải về danh sách các gói phần mềm mới nhất từ kho lưu trữ của Ubuntu.
* **`apt upgrade -y`**: So sánh danh sách đó với các phần mềm hiện tại trên VPS và tiến hành cài đặt phiên bản mới nhất. Tham số `-y` tự động đồng ý nâng cấp mà không cần hỏi lại.

---

### 5. Những lưu ý quan trọng

* **Nâng cấp nhân hệ điều hành (Kernel):** Quá trình nâng cấp hệ thống có thể mất từ 2 - 5 phút tùy tốc độ mạng của VPS. Đôi khi hệ thống sẽ hiển thị màn hình thông báo cấu hình dịch vụ hoặc nâng cấp Kernel. Bạn chỉ cần nhấn `Enter` để chọn các tùy chọn mặc định của hệ thống.
* **Keep connection alive:** Trên Bitvise, cấu hình gửi tín hiệu giữ kết nối tại tab `Session` -> tick `Keep alive` để tránh bị tự động log out khi không tương tác trong thời gian dài.

---

### 6. Câu hỏi tự luận đánh giá nhanh

#### Câu 1 (Hiểu bản chất)
Tại sao chúng ta nên ưu tiên lựa chọn hệ điều hành Ubuntu Server phiên bản LTS (Long Term Support) như **24.04 LTS** để triển khai các dịch vụ Microservices thay vì các phiên bản non-LTS mới hơn?
* *Gợi ý:* Các phiên bản LTS được cam kết hỗ trợ và vá lỗi bảo mật lâu dài (5 năm), mang lại sự ổn định và nhất quán cao cho hệ thống Production. Ngược lại, các bản non-LTS có vòng đời rất ngắn (9 tháng) và dễ gặp lỗi tương thích phần mềm, không phù hợp cho môi trường vận hành ổn định lâu dài.

#### Câu 2 (Phân tích)
Hãy phân tích các bất tiện (Pain Points) chính khi sử dụng Terminal mặc định của hệ điều hành (như CMD/PowerShell) để quản trị VPS và chỉ ra cách Bitvise SSH Client giải quyết các nhược điểm này thông qua tính năng lưu profile `.tlp` và các nút chức năng trên giao diện.
* *Gợi ý:* Terminal mặc định bắt buộc người dùng nhớ IP/port và nhập mật khẩu thủ công mỗi lần kết nối, rất dễ nhầm lẫn; đồng thời không có sẵn giao diện trực quan để quản lý tập tin. Bitvise cho phép nạp profile `.tlp` đã lưu để kết nối nhanh bằng 1-click; đồng thời cung cấp nút **New SFTP window** hiển thị giao diện 2 cột giúp kéo thả file trực quan và nút **New terminal console** độc lập để gõ lệnh thuận tiện.

#### Câu 3 (Nâng cao)
Một máy chủ VPS Ubuntu 24.04 LTS mới tinh vừa được nhà cung cấp khởi tạo và bàn giao. Nếu bạn không chạy lệnh `sudo apt update && sudo apt upgrade -y` ngay mà trực tiếp thực hiện cài đặt Docker và chạy ứng dụng Spring Boot, hãy phân tích những rủi ro về mặt bảo mật và lỗi cài đặt hệ thống có thể xảy ra.
* *Gợi ý:* Hệ điều hành được đóng gói sẵn của nhà cung cấp VPS có thể đã được lưu trữ từ lâu và chứa các gói thư viện cũ có lỗ hổng bảo mật nghiêm trọng chưa được vá. Nếu trực tiếp cài đặt Docker, bạn có thể tải về các dependencies không tương thích hoặc gặp lỗi xung đột thư viện hệ thống khiến Docker Engine chạy không ổn định hoặc các ứng dụng Spring Boot không thể kết nối mạng bên ngoài do DNS hoặc thư viện SSL của OS bị lỗi thời.


