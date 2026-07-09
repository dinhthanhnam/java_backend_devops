# Kịch bản Thuyết trình - Session 10: Triển khai hệ thống lên VPS

---

## Lesson 1: Giới thiệu VPS và thiết lập kết nối cơ bản qua Bitvise Client

### 1. Phần lý thuyết

**(Mở đầu - Khơi gợi bối cảnh thực tế)**
Chào các bạn! Trong suốt chặng đường từ Session 1 đến giờ, chúng ta đã cùng nhau xây dựng và đóng gói các dịch vụ của QuickBite như `user-service`, `restaurant-service` hay database chạy rất mượt mà trên máy tính cá nhân. Nhưng thực tế đi làm, khách hàng hay người dùng không thể truy cập vào máy tính cá nhân của bạn để đặt đồ ăn được. Chúng ta bắt buộc phải đưa hệ thống lên một máy chủ có địa chỉ IP công khai, hoạt động liên tục 24/7 trên môi trường Internet. Và giải pháp phổ biến, tiết kiệm nhất chính là **Virtual Private Server (VPS)**.

**[Slide 1: Khái niệm VPS và lựa chọn hạ tầng triển khai]**
Hiểu một cách đơn giản, VPS là một máy chủ ảo được phân tách từ máy chủ vật lý bằng công nghệ ảo hóa (như KVM). Nó chạy một hệ điều hành độc lập và cung cấp cho chúng ta quyền quản trị tối cao `root`. 
* Khi so sánh với **Cloud Server** (như AWS EC2), VPS thường rẻ hơn, chi phí cố định theo tháng và tài nguyên phần cứng cố định. Còn Cloud Server thì mạnh mẽ hơn ở khả năng tự động co giãn (Auto-scaling) nhưng cấu hình và quản trị phức tạp hơn rất nhiều.
* Với hệ thống Microservices QuickBite, thầy khuyến nghị các bạn thuê một VPS chạy hệ điều hành **Ubuntu Server 24.04 LTS** (phiên bản hỗ trợ lâu dài cực kỳ ổn định), cấu hình tối thiểu là **2 vCPUs và 2GB RAM** để các container không bị thiếu tài nguyên khi chạy đồng thời.

**[Slide 2: Kết nối VPS bằng Terminal truyền thống & các Pain Points]**
Thông thường, khi mua VPS xong, nhà cung cấp sẽ gửi cho bạn một địa chỉ IP công khai và mật khẩu root. Ở máy local (Windows/macOS), bạn có thể mở CMD hoặc PowerShell lên và gõ:
```bash
ssh root@<vps_public_ip>
```
Sau đó hệ thống sẽ yêu cầu xác nhận vân tay khóa bảo mật (Host key fingerprint) lần đầu tiên, bạn chỉ cần gõ `yes` và nhập mật khẩu root là đăng nhập được vào màn hình đen CLI của Ubuntu.

Tuy nhiên, nếu cứ dùng Terminal mặc định này để quản lý hệ thống lâu dài, bạn sẽ gặp phải những **Pain Points** rất khó chịu:
1. Mỗi lần mở máy làm việc là phải tìm lại địa chỉ IP, gõ lại lệnh SSH thủ công.
2. Phải nhớ và gõ mật khẩu root phức tạp liên tục, cực kỳ mất thời gian.
3. Không có giao diện trực quan để truyền tải tệp tin (như file `.env` hay download log). Bạn sẽ phải dùng các lệnh CLI rất phức tạp như `scp` hoặc `rsync`.

**[Slide 3: Giải pháp: Lưu Session trên Bitvise SSH Client]**
Để giải quyết tất cả các pain points trên, chúng ta sử dụng công cụ **Bitvise SSH Client** trên Windows. 
* Tại tab **Login**, chúng ta điền IP của VPS, cổng `22`, user `root` và mật khẩu.
* Đặc biệt, các bạn hãy nhấn nút **Save Profile As** ở menu bên trái để xuất cấu hình thành tệp tin có phần mở rộng là **`.tlp`** (ví dụ: `quickbite_root.tlp`). Lần sau làm việc, chỉ cần nạp lại file `.tlp` này là kết nối thành công chỉ bằng một click chuột.
* Sau khi đăng nhập thành công, Bitvise không tự động mở các cửa sổ mà cung cấp hai biểu tượng trực quan ở menu bên trái để bạn tự mở khi cần: **New terminal console** (để gõ lệnh CLI) và **New SFTP window** (hiển thị giao diện 2 cột giúp bạn kéo thả truyền file giữa máy local và VPS cực kỳ nhanh chóng).

---

### 2. Phần thực hành (Demo)

**[Live Demo: Kết nối Terminal đầu tiên và cập nhật hệ thống]**
Bây giờ, thầy sẽ thao tác trực tiếp các bước kết nối đầu tiên và cập nhật hệ điều hành sạch trên VPS.

*(Thao tác trên Bitvise Client và Terminal)*:
- *Bước 1: Điền thông tin Host, Port, Username, Password vào tab Login của Bitvise.*
- *Bước 2: Bấm Save Profile As để lưu lại file `quickbite_root.tlp` ra màn hình Desktop.*
- *Bước 3: Bấm Login để kết nối. Chấp nhận Host Key khi được hỏi.*
- *Bước 4: Nhấp chọn nút **New terminal console** ở menu bên trái để mở cửa sổ dòng lệnh đen.*
- *Bước 5: Trên terminal của VPS, chạy lệnh cập nhật danh sách gói và tiến hành nâng cấp toàn bộ hệ thống lên phiên bản mới nhất:*
  ```bash
  sudo apt update && sudo apt upgrade -y
  ```
  *(Giải thích): Quá trình này sẽ tải về các bản vá lỗi bảo mật của Ubuntu 24.04 LTS. Nếu hệ thống hiển thị màn hình chọn cấu hình Kernel, các bạn cứ nhấn `Enter` để chọn mặc định nhé.*

---

## Lesson 2: Tạo User mới và bảo mật VPS bằng cơ chế khóa SSH (SSH Hardening)

### 1. Phần lý thuyết

**[Slide 4: Nguy cơ tấn công dò mật khẩu & Nguyên lý hoạt động của SSH Key]**
Các bạn thân mến, một khi VPS có IP public và mở cổng 22 ra Internet, các hệ thống bot tự động sẽ liên tục thực hiện hàng triệu lượt dò mật khẩu (tấn công Brute-force) mỗi ngày. Nếu chúng ta để cấu hình đăng nhập root bằng mật khẩu mặc định, nguy cơ VPS bị chiếm quyền điều khiển là cực kỳ cao.

Vì vậy, chúng ta cần thực hiện **SSH Hardening** (Bảo mật dịch vụ SSH) theo 2 nguyên tắc cốt lõi:
1. **Đặc quyền tối thiểu (Least Privilege):** Tạo một người dùng thường tên là `deployer`, chỉ khi nào làm tác vụ admin mới dùng lệnh `sudo`. Tuyệt đối cấm tài khoản root đăng nhập trực tiếp từ xa.
2. **Xác thực bằng SSH Key:** Loại bỏ đăng nhập bằng mật khẩu. Chỉ cho phép các máy tính local có giữ **Private Key** (Khóa riêng tư) khớp với **Public Key** (Khóa công khai) lưu trên VPS (`~/.ssh/authorized_keys`) mới được quyền kết nối.

---

### 2. Phần thực hành (Demo)

**[Live Demo: Tạo User, sinh khóa SSH và vô hiệu hóa password]**
Thầy sẽ tiến hành cấu hình bảo mật SSH từng bước một. Các bạn hãy quan sát kỹ.

*(Thao tác trên Terminal VPS và PowerShell máy local)*:
- *Bước 1: Trên VPS (quyền root), tạo user `deployer` và đưa vào nhóm quản trị `sudo`:*
  ```bash
  adduser deployer
  usermod -aG sudo deployer
  ```
- *Bước 2: Mở PowerShell trên máy local, sinh cặp khóa bảo mật cao Ed25519:*
  ```powershell
  ssh-keygen -t ed25519 -C "deployer_vps_key"
  ```
  *(Giải thích): Nhấn Enter lưu mặc định tại thư mục `.ssh/id_ed25519`.*
- *Bước 3: Sao chép khóa công khai từ máy local lên VPS:*
  ```powershell
  ssh-copy-id -i ~/.ssh/id_ed25519.pub deployer@<vps_public_ip>
  ```
- *Bước 4: Mở một cửa sổ PowerShell local khác, test thử kết nối bằng SSH Key xem có cần gõ mật khẩu không:*
  ```powershell
  ssh deployer@<vps_public_ip>
  ```
- *Bước 5: Khi đã kết nối thành công không cần mật khẩu, quay lại terminal VPS, mở file cấu hình SSH:*
  ```bash
  sudo nano /etc/ssh/sshd_config
  ```
  Tìm và sửa đổi 3 cấu hình sau:
  ```text
  PermitRootLogin no
  PasswordAuthentication no
  PubkeyAuthentication yes
  ```
- *Bước 6: Khởi động lại dịch vụ SSH:*
  ```bash
  sudo systemctl restart ssh
  ```

> [!WARNING]
> **(Cảnh báo cực kỳ quan trọng)**:
> Tuyệt đối không được tắt cửa sổ terminal cũ đi khi bạn chưa mở cửa sổ terminal mới và kết nối thử thành công bằng user `deployer`. Nếu cấu hình lỗi và bạn ngắt kết nối cũ, bạn sẽ bị khóa quyền truy cập VPS vĩnh viễn!
> Ngoài ra, thư mục `~/.ssh` trên VPS phải được phân quyền đúng `700` và file `authorized_keys` là `600`. Nếu để phân quyền quá rộng (như `777`), cơ chế StrictModes của SSH Daemon sẽ chặn đăng nhập để đảm bảo an toàn.

---

## Lesson 3: Thiết lập Tường lửa UFW và Cài đặt Docker trên Ubuntu Server

### 1. Phần lý thuyết

**[Slide 5: Cài đặt Docker Engine chuẩn Enterprise]**
Để chạy các container Backend và Database của QuickBite trên VPS Ubuntu 24.04 LTS, chúng ta cần cài đặt Docker Engine. Thầy lưu ý là chúng ta cấm dùng lệnh cài đặt mặc định `docker.io` vì nó chứa phiên bản cũ không được cập nhật vá lỗi nhanh nhất. Chúng ta sẽ liên kết tới repository chính thức của Docker để cài phiên bản chuẩn doanh nghiệp mới nhất.

**[Slide 6: Phân quyền Docker CLI và lệnh `newgrp`]**
Sau khi cài đặt, mặc định chỉ có user `root` hoặc các user chạy lệnh kèm `sudo` mới có quyền giao tiếp với Docker socket `/var/run/docker.sock`. Việc lạm dụng lệnh `sudo docker` rất dễ gây lỗi phân quyền file và không an toàn. Do đó, chúng ta đưa user `deployer` vào nhóm hệ thống tên là `docker`. Sau đó, ta dùng lệnh `newgrp docker` để cập nhật quyền nhóm ngay lập tức cho session hiện hành mà không cần phải thực hiện log out rồi login lại.

**[Slide 7: Tường lửa UFW và Lỗ hổng Docker Bypass iptables]**
Để bảo vệ hệ thống khỏi các đợt quét cổng lạ từ Internet, chúng ta sử dụng tường lửa **UFW**. Quy tắc của chúng ta là: Chặn mặc định các kết nối đi vào (Default Deny Incoming), cho phép kết nối đi ra thoải mái (Default Allow Outgoing) để tải thư viện, và chỉ mở 3 cổng duy nhất là **22 (SSH)**, **80 (HTTP)**, và **443 (HTTPS)**.

*Thầy có một lưu ý kỹ thuật rất nâng cao ở đây:* Khi bạn khởi chạy một container Docker và sử dụng cờ map cổng ra máy host (ví dụ `-p 8080:8080`), mặc dù bạn dùng UFW để chặn cổng này (`sudo ufw deny 8080/tcp`), client bên ngoài **vẫn có thể kết nối được bình thường**!
* Nguyên nhân là do Docker Engine khi khởi động sẽ tự động can thiệp và chèn các luật định tuyến mạng (`iptables rules`) trực tiếp vào nhân Kernel Linux. Các luật của Docker được ưu tiên xử lý trước các chuỗi kiểm tra của UFW, do đó gói tin được đẩy thẳng vào container trước khi UFW kịp ngăn chặn.
* Do đó, để an toàn, các bạn tuyệt đối không map các cổng database nội bộ ra host (không dùng `-p 5432:5432`), mà chỉ để các container tự giao tiếp nội bộ trong mạng ảo Docker Network.

---

### 2. Phần thực hành (Demo)

**[Live Demo: Cài đặt Docker Engine và cấu hình Tường lửa UFW]**
Bây giờ thầy sẽ chạy các lệnh cài đặt Docker và bật tường lửa trên VPS.

*(Thao tác trên Terminal VPS)*:
- *Bước 1: Nạp GPG key và thêm nguồn cài đặt Docker chính thức:*
  ```bash
  sudo apt-get update
  sudo apt-get install ca-certificates curl gnupg -y
  sudo install -m 0755 -d /etc/apt/keyrings
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
  sudo chmod a+r /etc/apt/keyrings/docker.gpg
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  ```
- *Bước 2: Tiến hành cài đặt Docker Engine:*
  ```bash
  sudo apt-get update
  sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
  ```
- *Bước 3: Phân quyền chạy docker không dùng sudo:*
  ```bash
  sudo usermod -aG docker deployer
  newgrp docker
  ```
- *Bước 4: Cấu hình tường lửa UFW:*
  ```bash
  sudo ufw default deny incoming
  sudo ufw default allow outgoing
  sudo ufw allow 22/tcp
  sudo ufw allow 80/tcp
  sudo ufw allow 443/tcp
  ```
  *(Cảnh báo): Bắt buộc phải allow 22 trước khi enable!*
  ```bash
  sudo ufw enable
  sudo ufw status verbose
  ```

---

## Lesson 4: Đồng bộ cấu hình qua Git bằng SSH Key và vận hành cụm Microservices

### 1. Phần lý thuyết

**[Slide 8: Phân biệt hai loại kết nối SSH Key]**
Để tải toàn bộ file cấu hình của QuickBite (như file `docker-compose.yml`) lên VPS, việc truyền file thủ công qua SFTP là không tối ưu. Cách chuyên nghiệp là sử dụng Git để đồng bộ trực tiếp từ GitHub về VPS.
Lúc này, chúng ta cần phân biệt rõ hai loại SSH Key hoạt động độc lập:
1. **SSH Key 1 (Local -> VPS):** Sinh ở máy local của bạn, Public key nằm trên VPS để bạn đăng nhập SSH vào quản trị máy chủ.
2. **SSH Key 2 (VPS -> GitHub):** Sinh trực tiếp trên VPS bằng tài khoản `deployer`, Public key cấu hình trong mục **Settings -> SSH and GPG keys** của tài khoản GitHub cá nhân để cho phép VPS có quyền truy cập clone các Private Repository của bạn.

**[Slide 9: Nguyên tắc bảo mật tệp tin `.env` trên Production]**
Khi chạy các container bằng Docker Compose, chúng ta sử dụng tệp tin `.env` để nạp các cấu hình nhạy cảm như mật khẩu database, cổng chạy, JWT secret key.
* **Quy tắc vàng:** Không bao giờ được push file `.env` chứa mật khẩu thực tế lên Git. Tệp tin này bắt buộc phải nằm trong `.gitignore`. Mỗi môi trường (Local, Staging, Production) sẽ tự viết và lưu trữ một file `.env` vật lý riêng biệt ngay tại máy chủ đó.

---

### 2. Phần thực hành (Demo)

**[Live Demo: Đồng bộ dự án, cấu hình môi trường và khởi chạy hệ thống]**
Thầy sẽ thực hiện các bước cấu hình SSH Key trên VPS, clone dự án, nạp file `.env` và chạy Docker Compose.

*(Thao tác trên Terminal VPS và giao diện GitHub)*:
- *Bước 1: Trên VPS, sinh khóa SSH mới kết nối với GitHub:*
  ```bash
  ssh-keygen -t ed25519 -C "vps_to_github_key"
  ```
- *Bước 2: Hiển thị và sao chép khóa công khai:*
  ```bash
  cat ~/.ssh/id_ed25519.pub
  ```
- *Bước 3: Truy cập GitHub cá nhân -> Settings -> SSH and GPG keys -> New SSH key -> Dán nội dung và lưu lại.*
- *Bước 4: Kiểm tra kết nối từ VPS tới GitHub:*
  ```bash
  ssh -T git@github.com
  ```
- *Bước 5: Clone dự án hạ tầng về VPS:*
  ```bash
  mkdir -p ~/projects && cd ~/projects
  git clone git@github.com:your_username/quickbite-base.git
  ```
- *Bước 6: Tạo và cấu hình file `.env` vật lý trên VPS:*
  ```bash
  cd quickbite-base
  nano .env
  ```
  *(Giải thích): Điền các tham số mật khẩu thực tế của Production.*
- *Bước 7: Khởi chạy cụm Microservices:*
  ```bash
  docker compose up -d
  ```
- *Bước 8: Thực hiện giám sát và kiểm tra sức khỏe hệ thống:*
  ```bash
  docker compose ps
  docker compose logs -f
  docker stats
  ```
  *(Giải thích): Nếu thấy container báo trạng thái sập với Exit Code 137, các bạn cần biết đó là lỗi OOM (Out Of Memory) do VPS bị cạn RAM vật lý và Kernel đã tự động phát tín hiệu SIGKILL để hạ container cứu hệ thống. Lúc này, chúng ta cần tối ưu lại dung lượng cấp phát RAM cho các tiến trình Java của Spring Boot nhé.*

---

*Bài thuyết trình Session 10 kết thúc ở đây. Chúc các bạn thực hành thiết lập máy chủ VPS an toàn và vận hành thành công cụm microservices QuickBite!*
