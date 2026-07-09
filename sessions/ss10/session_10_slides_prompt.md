# PROMPT CHO GAMMA: TRIỂN KHAI HỆ THỐNG LÊN VPS (SESSION 10)

## 1. CONTEXT & ROLE
* **Role:** DevOps Architect kiêm Giảng viên cao cấp. Giọng điệu chuyên nghiệp, thực tế, đi thẳng vào bản chất kỹ thuật, cấu hình dòng lệnh và thực tiễn vận hành hệ thống.
* **Target Audience:** Kỹ sư phần mềm, học viên đang học cách đưa hệ thống Spring Boot Microservices (dự án QuickBite) từ môi trường phát triển cục bộ lên hạ tầng đám mây thực tế.
* **Objective:** Hướng dẫn từng bước thiết lập kết nối VPS an toàn qua Bitvise, cấu hình SSH Hardening để chống tấn công Brute-force, cài đặt Docker Engine và thiết lập tường lửa UFW, nạp khóa xác thực SSH cá nhân lên GitHub để clone code an toàn và khởi chạy toàn bộ cụm microservices thông qua tệp cấu hình môi trường `.env`.

---

## 2. PARTITION INSTRUCTIONS
* **Số lượng slide khuyến nghị:** 15 - 18 slides.
* **Nguyên tắc phân bổ nội dung:**
  * **LESSON 01 (Thiết lập kết nối):** Giới thiệu VPS, so sánh hạ tầng, kết nối qua Terminal, chỉ ra các Pain Points, hướng dẫn sử dụng Bitvise SSH Client lưu profile `.tlp` và nâng cấp hệ thống (`apt update && apt upgrade`).
  * **LESSON 02 (Bảo mật SSH):** Tấn công Brute-force, SSH Hardening, tạo user `deployer` với đặc quyền `sudo`, sinh khóa Ed25519, tắt root/password đăng nhập, lỗi StrictModes do phân quyền sai.
  * **LESSON 03 (Tường lửa & Docker):** Cài đặt Docker Engine từ repo chính thức, phân quyền group `docker`, cấu hình UFW (Default Deny Incoming), lỗi cấm cổng SSH khi bật UFW, và hiện tượng Docker bypass UFW qua iptables.
  * **LESSON 04 (Vận hành Microservices):** Phân biệt 2 loại SSH Key, nạp SSH Key vào tài khoản GitHub, clone Private Repo, thiết lập `.env` cho Production, chạy Docker Compose Up và các lệnh giám sát (`ps`, `logs`, `stats`).
* **Tính sư phạm và kỹ thuật:** Trình bày rõ ràng các câu lệnh, đường dẫn tệp tin cấu hình và giải thích các tham số cốt lõi.

---

## 3. HIERARCHICAL OUTLINE (DÀN Ý CHI TIẾT) & SPEAKER NOTES

### LESSON 01: Khởi tạo VPS và thiết lập kết nối cơ bản qua Bitvise Client

#### Slide 1: Khái niệm VPS và lựa chọn hạ tầng triển khai
* **Khái niệm:** Virtual Private Server (VPS) là máy chủ ảo hoạt động độc lập, được phân chia tài nguyên từ máy chủ vật lý bằng công nghệ ảo hóa (KVM, VMware).
* **So sánh hạ tầng:**
  * *VPS:* Chi phí cố định rẻ, có toàn quyền quản trị root hệ thống, tài nguyên vật lý cố định trên một node.
  * *Cloud Server (AWS EC2, GCE):* Khả năng tự động co giãn (Auto-scaling) mạnh mẽ, tính sẵn sàng cao, trả tiền linh hoạt theo giờ/phút.
  * *PaaS (Heroku, Render):* Dễ dùng nhưng chi phí scale rất đắt và bị giới hạn quyền truy cập sâu vào OS.
* **Khuyến nghị QuickBite:** Ubuntu Server 24.04 LTS, tối thiểu 2 vCPUs, 2GB-4GB RAM, ổ cứng 30GB SSD.
* **Speaker Notes:** *Chào các bạn. Hôm nay chúng ta sẽ đưa dự án QuickBite lên đám mây thực tế. Bước đầu tiên là hiểu về VPS. Khác với Local hay các PaaS như Render bị giới hạn cổng mạng, VPS cho chúng ta quyền root tối cao trên hệ điều hành Ubuntu 24.04 LTS để có toàn quyền điều phối các container Backend và Database. Với dự án của chúng ta, các bạn hãy chuẩn bị một VPS tối thiểu 2 vCPUs và 2GB RAM để chạy mượt cụm container nhé.*

#### Slide 2: Kết nối VPS bằng Terminal truyền thống & các Pain Points
* **Thực hiện kết nối:**
  ```bash
  ssh root@<vps_public_ip>
  ```
  Nhập `yes` để xác nhận Host key fingerprint lần đầu và nhập mật khẩu root được cấp.
* **Các Pain Points (Bất tiện) khi dùng Terminal truyền thống:**
  1. *Quản lý phiên kém:* Phải ghi nhớ IP/port và gõ lại lệnh SSH thủ công mỗi khi mở máy.
  2. *Nhập mật khẩu lặp đi lặp lại:* Dễ gõ nhầm và tốn thời gian.
  3. *Không có giao diện quản lý file:* Khi cần nạp file `.env` hoặc xem log hệ thống, việc dùng lệnh `scp` hoặc chỉnh sửa qua `nano` / `vi` rất chậm chạp.
  4. *Rủi ro mất kết nối:* Rớt mạng đột ngột có thể làm treo hoặc ngắt các lệnh đang thực thi dở.
* **Speaker Notes:** *Thông thường, chúng ta kết nối qua SSH bằng lệnh `ssh root@IP`. Tuy nhiên, khi làm việc thực tế, việc nhớ IP, gõ lại mật khẩu phức tạp liên tục rất mệt mỏi. Hơn thế nữa, việc truyền tệp tin qua lệnh SCP thuần túy rất bất tiện vì không có giao diện trực quan. Chúng ta cần một giải pháp tốt hơn.*

#### Slide 3: Thiết lập kết nối lưu Profile trên Bitvise SSH Client
* **Cấu hình tab Login:**
  * *Host:* Địa chỉ IP công khai của VPS.
  * *Port:* `22` (mặc định).
  * *Username / Password:* Tài khoản root và mật khẩu tương ứng.
* **Lưu Profile tiện lợi:**
  * Nhấn nút **Save Profile As** ở bảng bên trái để xuất cấu hình ra tệp tin `.tlp` (Ví dụ: `quickbite_root.tlp`). Lần sau chỉ cần nạp lại file `.tlp` để kết nối lập tức.
* **Giao diện làm việc Bitvise (Menu bên trái):**
  * *New terminal console:* Mở màn hình dòng lệnh CLI của VPS.
  * *New SFTP window:* Mở trình duyệt tệp tin đồ họa 2 cột để kéo thả file dễ dàng giữa máy local và VPS.
* **Speaker Notes:** *Bitvise SSH Client là một công cụ đắc lực trên Windows. Chúng ta lưu lại cấu hình dưới dạng file `.tlp`. Sau đó, ta chỉ cần bấm "New terminal console" để gõ lệnh và "New SFTP window" để duyệt tệp tin theo kiểu kéo thả trực quan. Điều này giúp đẩy nhanh tốc độ vận hành và giảm thiểu sai sót gõ nhầm IP.*

#### Slide 4: Thực hành: Cập nhật hệ thống VPS đầu tiên
* **Lệnh thực thi bắt buộc:**
  ```bash
  sudo apt update && sudo apt upgrade -y
  ```
* **Giải thích hoạt động:**
  * `apt update`: Tải về danh sách gói và phiên bản mới nhất từ kho lưu trữ của Ubuntu về máy chủ.
  * `apt upgrade -y`: Tiến hành tải xuống và nâng cấp các phần mềm hiện có trên VPS lên bản mới nhất. Tham số `-y` giúp tự động đồng ý cài đặt.
* **Lưu ý:** Nếu hệ thống yêu cầu chọn nâng cấp nhân Kernel, hãy nhấn `Enter` để đồng ý với các thiết lập mặc định của Ubuntu.
* **Speaker Notes:** *Ngay sau khi kết nối root thành công, việc đầu tiên cần làm là cập nhật hệ thống. Lệnh `apt update` giúp máy cập nhật chỉ mục, còn `apt upgrade` thực hiện tải và cài đặt bản vá bảo mật mới nhất cho các phần mềm có sẵn của OS. Các bạn nhớ chạy lệnh này để vá các lỗ hổng bảo mật trước khi cài đặt thêm bất kỳ ứng dụng nào.*

---

### LESSON 02: Tạo User mới và bảo mật VPS bằng cơ chế khóa SSH (SSH Hardening)

#### Slide 5: Nguy cơ tấn công dò mật khẩu & Nguyên lý hoạt động của SSH Key
* **Tấn công Brute-force:** Các bot tự động liên tục quét cổng 22 trên Internet để dò mật khẩu root.
* **Nguyên tắc Least Privilege:** Không chạy tác vụ thông thường bằng tài khoản root. Tạo user thường và nâng quyền thông qua `sudo` khi cần thiết.
* **Cơ chế SSH Key (Mã hóa bất đối xứng):**
  * *Private Key:* Nằm trên máy local của bạn, phải giữ tuyệt mật.
  * *Public Key:* Nằm trên VPS tại thư mục `~/.ssh/authorized_keys`, dùng để đối chiếu xác thực.
* **Speaker Notes:** *VPS có IP public sẽ bị các botnet tấn công quét cổng 22 liên tục. Nếu ta dùng mật khẩu yếu hoặc để root đăng nhập trực tiếp, hệ thống sẽ rất dễ bị hack. Do đó, chúng ta sẽ áp dụng SSH Key - cơ chế mã hóa bất đối xứng. Public Key được đưa lên VPS, còn Private Key được giữ an toàn trên máy của bạn.*

#### Slide 6: Thực hành: Tạo User mới và Sinh khóa SSH Key Ed25519
* **Tạo người dùng và gán quyền quản trị trên VPS:**
  ```bash
  sudo adduser deployer
  sudo usermod -aG sudo deployer
  ```
* **Sinh cặp khóa SSH trên máy Local (PowerShell):**
  ```powershell
  ssh-keygen -t ed25519 -C "deployer_vps_key"
  ```
  *Lưu ý:* Lưu mặc định tại `~/.ssh/id_ed25519`.
* **Đẩy Public Key từ máy Local lên VPS:**
  ```powershell
  ssh-copy-id -i ~/.ssh/id_ed25519.pub deployer@<vps_public_ip>
  ```
* **Speaker Notes:** *Chúng ta tạo ra user `deployer` và đưa vào nhóm `sudo` để có quyền quản trị. Ở máy local, ta dùng lệnh `ssh-keygen` với thuật toán Ed25519 để tạo cặp khóa. Sau đó, dùng `ssh-copy-id` để đẩy Public Key lên VPS để hoàn tất bước đăng ký khóa.*

#### Slide 7: Thực hành: Cấu hình SSH Hardening trên VPS
* **Chỉnh sửa cấu hình SSH Daemon:**
  ```bash
  sudo nano /etc/ssh/sshd_config
  ```
* **Các tham số cấu hình bảo mật bắt buộc:**
  ```text
  PermitRootLogin no
  PasswordAuthentication no
  PubkeyAuthentication yes
  ```
* **Nạp lại cấu hình dịch vụ:**
  ```bash
  sudo systemctl restart ssh
  ```
* **Speaker Notes:** *Sau khi kiểm tra kết nối bằng SSH Key thành công, ta chỉnh sửa file `/etc/ssh/sshd_config`. Ở đây, ta cấu hình `PermitRootLogin no` để chặn root đăng nhập trực tiếp, `PasswordAuthentication no` để tắt đăng nhập bằng mật khẩu thông thường. Cuối cùng, chạy `systemctl restart ssh` để áp dụng cấu hình bảo mật mới.*

#### Slide 8: Cảnh báo bảo mật & Cơ chế StrictModes của SSH Daemon
* **Cảnh báo sống còn:**
  * **Tuyệt đối không** tắt session kết nối root hiện tại cho đến khi đã mở một terminal mới và SSH thành công bằng SSH Key của user `deployer`.
* **Cơ chế StrictModes của sshd:**
  * SSH Daemon sẽ từ chối đọc danh sách authorized_keys nếu thư mục `~/.ssh` hoặc file `authorized_keys` bị phân quyền quá rộng (ví dụ chmod `777` cho phép Group hoặc Others ghi đè).
  * *Phân quyền chuẩn:* Thư mục `~/.ssh` là `700`, file `authorized_keys` là `600`.
* **Speaker Notes:** *Hãy nhớ một quy tắc cực kỳ quan trọng: Đừng bao giờ tắt cửa sổ root cũ khi chưa test đăng nhập bằng SSH Key ở cửa sổ mới thành công. Nếu làm sai, bạn sẽ bị khóa quyền truy cập VPS vĩnh viễn. Ngoài ra, SSH Daemon rất khắt khe về quyền hạn tệp tin (StrictModes). Nếu bạn chmod 777 cho thư mục SSH, SSH Daemon sẽ báo lỗi và không cho phép bạn kết nối nữa để ngăn chặn rủi ro các user khác can thiệp.*

---

### LESSON 03: Cài đặt nhanh Docker & Docker Compose và thiết lập tường lửa UFW

#### Slide 9: Quy trình cài đặt Docker Engine chuẩn Enterprise
* **Tránh dùng:** Gói cài đặt mặc định `docker.io` của Ubuntu vì phiên bản cũ và thiếu các cập nhật vá lỗi bảo mật kịp thời.
* **Quy trình cài đặt chính thống từ Docker Repository:**
  1. Gỡ cài đặt các gói cũ:
     ```bash
     for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
     ```
  2. Nạp khóa GPG và thêm Repository chính thức của Docker vào apt.
  3. Cài đặt các gói cốt lõi:
     ```bash
     sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
     ```
* **Speaker Notes:** *Chúng ta cần cài đặt Docker để chạy các microservices dưới dạng container. Ở đây, các bạn tránh dùng lệnh `apt install docker.io` mặc định của Ubuntu. Hãy nạp kho cài đặt chính thức của Docker để đảm bảo nhận được phiên bản mới nhất, an toàn và ổn định chuẩn doanh nghiệp nhé.*

#### Slide 10: Phân quyền thực thi lệnh Docker cho user thường
* **Vấn đề phân quyền:** Lệnh docker giao tiếp qua Unix socket thuộc sở hữu của `root`. Chạy docker với `sudo` liên tục tạo ra file thuộc quyền root và tiềm ẩn lỗ hổng bảo mật.
* **Giải pháp: Nhóm bảo mật `docker`**
  ```bash
  sudo groupadd docker
  sudo usermod -aG docker deployer
  newgrp docker
  ```
* Lệnh `newgrp docker` giúp kích hoạt quyền nhóm mới ngay lập tức cho session hiện tại mà không cần log out và login lại.
* **Speaker Notes:** *Chạy lệnh docker với tiền tố `sudo` liên tục rất bất tiện và có thể gây lỗi phân quyền file. Để xử lý, ta đưa user `deployer` vào nhóm `docker` của hệ thống. Đồng thời chạy `newgrp docker` để cập nhật quyền nhóm cho session terminal hiện tại ngay lập tức mà không cần logout.*

#### Slide 11: Thiết lập Tường lửa UFW (Uncomplicated Firewall)
* **Nguyên tắc "Default Deny":** Chặn mọi kết nối đi vào mặc định, chỉ mở các cổng dịch vụ thiết yếu.
* **Kịch bản cấu hình tường lửa:**
  ```bash
  sudo ufw default deny incoming
  sudo ufw default allow outgoing
  sudo ufw allow 22/tcp
  sudo ufw allow 80/tcp
  sudo ufw allow 443/tcp
  sudo ufw enable
  ```
* *Cảnh báo:* Bắt buộc chạy lệnh cho phép cổng `22/tcp` trước khi chạy `ufw enable` để tránh bị khóa truy cập SSH.
* **Speaker Notes:** *Để bảo vệ VPS khỏi các đợt scan cổng tự động, chúng ta dùng tường lửa UFW. Ta thiết lập chặn mặc định đi vào và mở các cổng 22 (SSH), 80 (HTTP) và 443 (HTTPS). Nhắc lại một lần nữa: Phải allow cổng 22 trước khi gõ ufw enable để tránh bị ngắt kết nối SSH nhé.*

#### Slide 12: Hiện tượng Docker bypass UFW Firewall qua iptables
* **Xung đột hoạt động:** Khi bạn chạy container bằng Docker và map cổng ra ngoài máy host (ví dụ `-p 8080:8080`), lệnh cấm cổng của UFW (`sudo ufw deny 8080/tcp`) sẽ **không có hiệu lực**!
* **Nguyên nhân kỹ thuật:**
  * Docker tự động can thiệp trực tiếp và sửa đổi cấu hình định tuyến cấp thấp (`iptables rules`) của hệ điều hành Kernel để thực hiện chuyển tiếp gói tin (NAT).
  * Chuỗi xử lý của Docker được thực thi trước chuỗi kiểm tra gói tin của UFW, do đó gói tin được đẩy thẳng vào container trước khi UFW kịp lọc chặn.
* **Khắc phục:** Viết rule chặn thủ công trong chuỗi `DOCKER-USER` của iptables hoặc cấu hình không cho Docker tự ý sửa iptables.
* **Speaker Notes:** *Có một lỗi bảo mật rất phổ biến mà nhiều DevOps hay gặp: Nghĩ rằng UFW đã cấm cổng 8080 thì bên ngoài không vào được container. Nhưng thực tế Docker tự động can thiệp trực tiếp vào iptables của Kernel và bypass qua các rule của UFW. Hãy luôn lưu ý điểm này để cấu hình bảo mật đúng cách, tránh để lộ các cổng database nội bộ ra ngoài.*

---

### LESSON 04: Đồng bộ dự án qua Git bằng SSH Key và vận hành cụm Microservices

#### Slide 13: Phân biệt hai loại kết nối SSH Key trong quy trình triển khai
* **SSH Key 1 (Local -> VPS):**
  * *Private Key:* Nằm trên máy local của bạn.
  * *Public Key:* Đặt tại file `~/.ssh/authorized_keys` của VPS.
  * *Mục đích:* Dùng để xác thực kết nối đăng nhập của quản trị viên vào VPS.
* **SSH Key 2 (VPS -> GitHub):**
  * *Private Key:* Sinh trực tiếp trên VPS bằng tài khoản `deployer`.
  * *Public Key:* Cấu hình tại **Settings -> SSH and GPG keys** của tài khoản GitHub cá nhân.
  * *Mục đích:* Cấp quyền cho VPS có thể kéo mã nguồn (Git Clone/Pull) từ Private Repository của bạn về máy chủ.
* **Speaker Notes:** *Trong thực tế, bạn sẽ làm việc với 2 cặp khóa SSH khác nhau. Khóa thứ nhất giúp bạn đăng nhập từ máy local vào VPS. Khóa thứ hai do VPS sinh ra và cấu hình lên GitHub để VPS có quyền kéo code từ các repo riêng tư về chạy. Các bạn chú ý phân biệt rõ để tránh copy nhầm khóa riêng tư lên máy chủ.*

#### Slide 14: Thực hành: Cấu hình SSH Key VPS kết nối GitHub
* **Sinh khóa SSH trên VPS:**
  ```bash
  ssh-keygen -t ed25519 -C "vps_to_github_key"
  ```
* **Lấy khóa công khai và dán lên GitHub:**
  * Chạy lệnh hiển thị khóa: `cat ~/.ssh/id_ed25519.pub`
  * Đăng nhập GitHub -> **Settings** -> **SSH and GPG keys** -> **New SSH key** -> Dán nội dung và lưu lại.
* **Kiểm tra kết nối thành công:**
  ```bash
  ssh -T git@github.com
  ```
* **Speaker Notes:** *Để VPS kết nối được với GitHub, ta chạy lệnh `ssh-keygen` trên VPS. Ta copy nội dung khóa công khai hiển thị ra màn hình và dán vào mục SSH and GPG Keys trong Settings tài khoản GitHub của mình. Cuối cùng, chạy `ssh -T git@github.com` để kiểm tra bắt tay thành công.*

#### Slide 15: Thực hành: Clone dự án và cấu hình tệp tin môi trường `.env`
* **Tải mã nguồn dự án hạ tầng về VPS:**
  ```bash
  mkdir -p ~/projects && cd ~/projects
  git clone git@github.com:your_username/quickbite-base.git
  ```
* **Tạo file môi trường Production `.env`:**
  ```bash
  cd ~/projects/quickbite-base
  nano .env
  ```
* **Nguyên tắc bảo mật tệp `.env`:**
  * **Không bao giờ** push file `.env` chứa mật khẩu thực tế lên Git.
  * Phải khai báo `.env` vào file `.gitignore` của dự án.
* **Speaker Notes:** *Sau khi clone dự án về, ta di chuyển vào thư mục và tạo file `.env` chứa các biến môi trường cấu hình thực tế cho Database, cổng chạy ứng dụng và JWT key bảo mật. Hãy ghi nhớ quy tắc bảo mật tối quan trọng: File `.env` chứa các mật khẩu thực tế tuyệt đối không được đưa lên Git.*

#### Slide 16: Vận hành và Giám sát cụm Microservices trên Production
* **Khởi chạy hệ thống ở chế độ chạy ngầm (Detached mode):**
  ```bash
  docker compose up -d
  ```
* **Giám sát sức khỏe hệ thống qua các dòng lệnh:**
  * *Kiểm tra trạng thái container:* `docker compose ps`
  * *Theo dõi nhật ký log thời gian thực:* `docker compose logs -f`
  * *Giám sát CPU/RAM tiêu thụ của từng container:* `docker stats`
* **Xử lý sự cố Exit Code 137:** Chỉ ra lỗi container bị kill do cạn kiệt RAM vật lý (Out Of Memory). Cần kiểm tra log và cấu hình lại dung lượng Heap Memory của JVM phù hợp.
* **Speaker Notes:** *Cuối cùng, ta chạy cụm microservices bằng lệnh `docker compose up -d`. Để kiểm tra hệ thống hoạt động ổn định, các bạn sử dụng các lệnh giám sát như `docker compose ps` để xem trạng thái container, `docker compose logs -f` để theo dõi log và `docker stats` để kiểm tra tài nguyên CPU/RAM tiêu thụ. Nếu gặp lỗi container bị sập với Exit Code 137, đó là dấu hiệu VPS bị cạn RAM và OOM Killer đã can thiệp tắt ứng dụng, khi đó ta cần tối ưu lại lượng RAM cấp phát cho JVM nhé. Chúc các bạn thực hành thành công!*
