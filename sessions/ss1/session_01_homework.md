# SESSION 01: BÀI TẬP VỀ NHÀ (HOMEWORK)
## Đề tài: Khởi tạo môi trường Linux Sandbox, Thao tác CLI & Tự động hóa Deploy cơ bản

Tài liệu này chứa 6 bài tập thực hành dành cho Session 1 của hệ thống QuickBite. Bạn cần hoàn thành đầy đủ các bài tập và nộp bài theo hướng dẫn ở cuối mỗi bài.

---

### BÀI TẬP 1 (Mức độ Khá): Khởi dựng Sandbox Linux (WSL cho Windows / VM cho macOS)

#### 1. Mục tiêu mong muốn đạt được
* **Khởi tạo và vận hành** thành công một môi trường Linux độc lập (Sandbox) trên máy tính cá nhân để phục vụ thực hành xuyên suốt khóa học.
* **Sử dụng kết nối Terminal** để giao tiếp với hệ điều hành Linux.

#### 2. Mô tả yêu cầu
Vì khóa học của chúng ta hướng đến triển khai thực chiến trên Linux nhưng hiện tại chưa học đến bài cấu hình VPS Cloud, bạn cần tự chuẩn bị một "hộp cát" (Sandbox) Linux chạy hệ điều hành Ubuntu Server (hoặc Ubuntu LTS) ngay trên máy tính cá nhân của mình:
* **Đối với sinh viên sử dụng hệ điều hành Windows:**
  * Kích hoạt tính năng và cài đặt **WSL 2 (Windows Subsystem for Linux)** chạy bản phân phối Ubuntu (khuyên dùng Ubuntu 22.04 LTS hoặc 24.04 LTS).
* **Đối với sinh viên sử dụng hệ điều hành macOS:**
  * Bạn bắt buộc phải cài đặt một **Máy ảo (Virtual Machine - VM)** chạy Ubuntu Server (có thể sử dụng các phần mềm miễn phí như UTM cho Mac Silicon, hoặc VirtualBox / VMware cho Mac Intel).
  * *Lưu ý:* Mặc dù các kỹ thuật nâng cao về Máy ảo (VM) sẽ được dạy chi tiết ở các session sau, nhưng ở thời điểm hiện tại, đây là cách duy nhất để các bạn sử dụng macOS có một môi trường Linux chuẩn hóa đồng bộ với khóa học.

#### 3. Kiểm thử và kết quả mong muốn
1. Mở cửa sổ Terminal của WSL (trên Windows) hoặc khởi động VM và kết nối SSH vào VM (trên macOS).
2. Chạy lệnh kiểm tra thông tin hệ điều hành:
   ```bash
   uname -a
   cat /etc/os-release
   ```
3. **Kết quả mong đợi:** Terminal trả về thông tin hệ điều hành Linux Ubuntu kèm theo phiên bản kernel chi tiết.

#### 4. Hướng dẫn nộp bài
* Chụp ảnh màn hình cửa sổ Terminal Linux đang chạy lệnh `cat /etc/os-release` hiển thị rõ thông tin phiên bản Ubuntu trên máy của bạn.
* Lưu hình ảnh vào thư mục Git cá nhân của bạn tại: `/homework/session_01/exercise_01/sandbox_setup.png`.

---

### BÀI TẬP 2 (Mức độ Khá): Làm quen CLI Linux: Thao tác file và kiểm tra User/Group

#### 1. Mục tiêu mong muốn đạt được
* **Thực thi nhuần nhuyễn** các lệnh duyệt thư mục và xem chi tiết thuộc tính tập tin (`ls -la`).
* **Truy vấn danh sách người dùng và nhóm** trực tiếp từ các file cấu hình hệ thống Linux (`getent`, `cat`).

#### 2. Mô tả yêu cầu
Hãy làm quen với giao diện dòng lệnh (CLI) bằng cách thực hiện các thao tác quản trị thư mục và người dùng sau trên môi trường Linux Sandbox bạn vừa cài đặt:
1. Di chuyển vào thư mục home (`cd ~`), tạo một thư mục trống tên là `quickbite-test` bằng lệnh `mkdir`.
2. Tạo một file rỗng tên là `readme.txt` và viết một vài nội dung vào bên trong.
3. Đứng tại thư mục đó, thực hiện liệt kê chi tiết danh sách file (bao gồm cả file ẩn) kèm phân quyền của chúng.
4. Chạy lệnh hiển thị danh sách tất cả các tài khoản người dùng (users) đang tồn tại trên hệ thống.
5. Chạy lệnh hiển thị danh sách tất cả các nhóm người dùng (groups) đang tồn tại trên hệ thống.

#### 3. Kiểm thử và kết quả mong muốn
1. Để kiểm tra danh sách file chi tiết, bạn chạy:
   ```bash
   cd ~/quickbite-test
   ls -la
   # Kết quả mong đợi: Hiển thị dòng thông tin file readme.txt có định dạng quyền hạn như -rw-r--r--
   ```
2. Để hiển thị danh sách user, chạy lệnh đọc file cấu hình passwd:
   ```bash
   getent passwd
   # Hoặc: cat /etc/passwd
   # Kết quả mong đợi: Danh sách các dòng user hệ thống hiển thị ra console
   ```
3. Để hiển thị danh sách group, chạy lệnh đọc file cấu hình group:
   ```bash
   getent group
   # Hoặc: cat /etc/group
   # Kết quả mong đợi: Danh sách các nhóm người dùng hiện ra console
   ```

#### 4. Hướng dẫn nộp bài
* Chụp ảnh màn hình terminal chạy lệnh `ls -la` trong thư mục `quickbite-test` lưu thành `ls_la.png`.
* Chụp ảnh màn hình terminal chạy lệnh `getent passwd` hiển thị danh sách người dùng lưu thành `users_list.png`.
* Lưu toàn bộ các ảnh chụp trên vào thư mục Git: `/homework/session_01/exercise_02/`.

---

### BÀI TẬP 3 (Mức độ Khá): Script cài đặt ban đầu (`initial-script.sh`) và tạo user/group `quickbite` tự động

#### 1. Mục tiêu mong muốn đạt được
* **Viết shell script** tự động hóa việc cài đặt các gói phần mềm và runtime JDK 17.
* **Cấu hình tạo người dùng hệ thống** bằng dòng lệnh bảo mật (`groupadd`, `useradd`).

#### 2. Mô tả yêu cầu
Hãy viết một shell script có tên `initial-script.sh` chạy trên Linux để thiết lập môi trường chạy ban đầu cho QuickBite. Script phải thực hiện tự động các bước sau:
1. Cập nhật hệ thống: `sudo apt-get update && sudo apt-get upgrade -y`.
2. Cài đặt các gói phần mềm bắt buộc: `openjdk-17-jdk`, `git`, `curl`.
3. Kiểm tra xem nhóm (group) `quickbite` đã tồn tại chưa, nếu chưa thì chạy lệnh tạo nhóm `quickbite`.
4. Tạo tài khoản người dùng hệ thống (system user) tên là `quickbite` thuộc nhóm `quickbite` với hai ràng buộc bảo mật cực kỳ quan trọng:
   * Không tự động tạo thư mục home cá nhân (dùng cờ `-r`).
   * Khóa không cho phép đăng nhập trực tiếp (đặt login shell là `/bin/false`).

#### 3. Kiểm thử và kết quả mong muốn
1. Cấp quyền thực thi và chạy script:
   ```bash
   chmod +x initial-script.sh
   sudo ./initial-script.sh
   ```
2. Kiểm tra xem user và group `quickbite` đã được tạo thành công chưa:
   ```bash
   id quickbite
   # Kết quả mong đợi: Hiển thị thông tin UID và GID thuộc nhóm quickbite
   
   getent passwd | grep quickbite
   # Kết quả mong đợi: Dòng cấu hình của quickbite kết thúc bằng /bin/false
   ```

#### 4. Hướng dẫn nộp bài
* Đẩy file script `initial-script.sh` lên Git cá nhân tại: `/homework/session_01/exercise_03/initial-script.sh`.
* Chụp ảnh màn hình chạy lệnh `id quickbite` hiển thị kết quả thành công, lưu thành `user_verify.png` trong thư mục trên.

---

### BÀI TẬP 4 (Mức độ Khá): Thiết lập thư mục ứng dụng và phân quyền sở hữu tối giản

#### 1. Mục tiêu mong muốn đạt được
* **Ứng dụng nguyên tắc phân quyền tối giản** (Least Privilege) để bảo vệ mã nguồn ứng dụng Java Spring Boot.
* **Sử dụng thành thạo** lệnh đổi quyền sở hữu (`chown`) và phân quyền truy cập (`chmod`).

#### 2. Mô tả yêu cầu
Chúng ta cần chuẩn bị thư mục chứa file JAR của `user-service` sao cho chỉ có người dùng được cấp quyền chạy dịch vụ mới thao tác được, ngăn chặn hoàn toàn người dùng bên ngoài đọc trộm file cấu hình (chứa DB password):
1. Tạo thư mục làm việc của dự án tại `/opt/quickbite/user-service`.
2. Sử dụng lệnh `chown` để thay đổi chủ sở hữu của thư mục `/opt/quickbite` (và toàn bộ thư mục con bên trong) về cho user `quickbite` và group `quickbite`.
3. Đặt quyền hạn truy cập tập tin cho thư mục `/opt/quickbite` là `750`. Hãy giải thích ngắn gọn ý nghĩa của mã số phân quyền `750` này.

#### 3. Kiểm thử và kết quả mong muốn
1. Chạy lệnh kiểm tra thuộc tính và quyền hạn thư mục:
   ```bash
   ls -la /opt
   ```
2. **Kết quả mong đợi:** Thư mục `quickbite` hiển thị quyền hạn dạng `drwxr-x---` và có cột chủ sở hữu/nhóm là `quickbite quickbite`.

#### 4. Hướng dẫn nộp bài
* Viết các câu lệnh Linux bạn sử dụng trong bài này vào file script `setup-dirs.sh`. Đẩy file lên Git cá nhân tại: `/homework/session_01/exercise_04/setup-dirs.sh`.
* Chụp màn hình terminal chạy lệnh `ls -la /opt` hiển thị rõ phân quyền thư mục `quickbite` lưu thành `permissions.png` trong thư mục trên.

---

### BÀI TẬP 5 (Mức độ Giỏi): Xử lý trùng cổng 8080 và chẩn đoán kết nối mạng

#### 1. Mục tiêu mong muốn đạt được
* **Phát hiện và dừng tiến trình** chiếm dụng cổng mạng để giải quyết sự cố crash ứng dụng lúc khởi động.
* **Chẩn đoán kết nối dịch vụ** ở xa thông qua các dòng lệnh mạng của Linux (`ping`, `curl`).

#### 2. Mô tả yêu cầu
* **Tình huống 1:** Ứng dụng `user-service` của bạn khi khởi chạy báo lỗi trùng cổng kết nối `8080` (`Address already in use`). Bạn cần tìm ra Process ID (PID) của tiến trình cũ đang chiếm giữ cổng này và ra lệnh tắt nó đi để giải phóng cổng.
* **Tình huống 2:** Ứng dụng `user-service` báo lỗi không kết nối được tới Database PostgreSQL ở địa chỉ IP `10.0.1.20`. Hãy chạy các lệnh kiểm tra xem đường truyền vật lý tới máy chủ Database kia có thông suốt hay không, và cổng kết nối database `5432` tại IP đó có phản hồi kết nối hay không.
* *Gợi ý (Hints):* Sử dụng lệnh `ss -tulpn` kết hợp `grep` để lọc cổng. Sử dụng lệnh `kill -9` để tắt tiến trình. Sử dụng `ping` để kiểm tra kết nối vật lý và `curl -v` hoặc `nc` để thăm dò trạng thái mở của cổng kết nối DB.

#### 3. Kiểm thử và kết quả mong muốn
1. Kết quả chạy lệnh tìm kiếm tiến trình chiếm cổng:
   ```bash
   sudo ss -tulpn | grep :8080
   # Kết quả mong đợi: Hiển thị dòng trạng thái chứa thông tin dạng "users:(("java",pid=xxxx,fd=yy))"
   ```
2. Sau khi chạy lệnh giải phóng cổng mạng, chạy lại lệnh kiểm tra cổng `8080` không còn kết quả nào được in ra.

#### 4. Hướng dẫn nộp bài
* Viết một file ghi chép ngắn dạng Markdown tên là `network-diagnose.md` ghi lại toàn bộ các câu lệnh bạn sử dụng kèm giải thích mục đích từng lệnh đối với 2 tình huống trên.
* Đẩy file báo cáo này lên Git tại: `/homework/session_01/exercise_05/network-diagnose.md`.
* Chụp ảnh màn hình terminal hiển thị dòng thông tin PID chiếm dụng cổng 8080 trước khi tắt lưu thành `port_fix.png` trong thư mục trên.

---

### BÀI TẬP 6 (Mức độ Xuất sắc): Xây dựng kịch bản script deploy tự động (`deploy-local.sh`) chạy file JAR Spring Boot dưới dạng dịch vụ ngầm

#### 1. Mục tiêu mong muốn đạt được
* **Tự động hóa hoàn toàn** chuỗi công việc deploy thủ công thành kịch bản "Một chạm" (One-click deployment).
* **Ứng dụng triết lý kiểm soát lỗi Fail-fast** của DevOps vào shell script để dừng quy trình triển khai ngay khi có bước bị lỗi.

#### 2. Mô tả yêu cầu
Hãy giải phóng lập trình viên khỏi quy trình deploy thủ công nhàm chán bằng cách viết một shell script tự động hóa toàn diện có tên `deploy-local.sh`. 
Script chạy cục bộ trên môi trường Linux Sandbox và phải thực hiện tuần tự các bước thông minh sau:
1. **Bước 1: Biên dịch mã nguồn (Build & Test):**
   * Di chuyển vào thư mục chứa code và chạy lệnh đóng gói file JAR: `./gradlew clean bootJar`.
   * *Yêu cầu Fail-fast:* Kiểm tra exit code (`$?`) của lệnh build. Nếu build thất bại (lỗi code Java hoặc kiểm thử Unit Test thất bại), script phải dừng lập tức và in thông báo lỗi màu đỏ. Tuyệt đối không thực thi các bước sau.
2. **Bước 2: Chuẩn bị hạ tầng thư mục:**
   * Tự động kiểm tra xem thư mục `/opt/quickbite/user-service` đã tồn tại chưa, nếu chưa thì tạo mới và phân quyền ownership cho user/group `quickbite`.
3. **Bước 3: Sao chép ứng dụng:**
   * Dừng tiến trình hoặc dịch vụ Systemd `quickbite-user` cũ (nếu có).
   * Sao chép file JAR mới build vào `/opt/quickbite/user-service-0.0.1.jar` và thay đổi chủ sở hữu file JAR về cho user và group `quickbite`.
4. **Bước 4: Khởi động dịch vụ chạy ngầm:**
   * Thực hiện khởi chạy file JAR dưới dạng chạy ngầm (chạy ngầm an toàn bằng cách dùng Systemd service `quickbite-user` hoặc chạy nền thông qua lệnh `nohup java -jar ... > /dev/null 2>&1 &` dưới danh nghĩa user `quickbite`).
5. **Bước 5: Smoke Test:**
   * Cho script tạm nghỉ (sleep) `5` giây để ứng dụng Spring Boot hoàn tất quá trình khởi tạo tiến trình JVM nền.
   * Chạy lệnh kiểm tra xem cổng `8080` đã được mở và ứng dụng Java Spring Boot đã lắng nghe kết nối thành công chưa.
   * Nếu cổng 8080 mở, in thông báo thành công màu xanh lá: `"DEPLOYS DỊCH VỤ THÀNH CÔNG!"`.
   * Nếu cổng 8080 đóng (dịch vụ sập ngầm lúc chạy JVM), in thông báo lỗi và trích xuất hiển thị nhanh 30 dòng nhật ký lỗi cuối cùng từ file log của ứng dụng để lập trình viên dễ dàng debug.

#### 3. Kiểm thử và kết quả mong muốn
1. Chạy thử script bằng lệnh: `sudo ./deploy-local.sh`.
   * **Kết quả mong đợi:** Script chạy tuần tự, in ra thông báo rõ ràng cho từng bước bằng màu sắc trực quan (sử dụng ANSI color codes trong bash script) và in ra thông báo màu xanh lá báo thành công ở cuối.
2. Cố tình sửa sai cú pháp trong mã nguồn Java của `user-service` và chạy lại script:
   * **Kết quả mong đợi:** Script phải báo lỗi đỏ rực và dừng ngay lập tức ở Bước 1.

#### 4. Hướng dẫn nộp bài
* Đẩy file script thông minh `deploy-local.sh` lên Git cá nhân tại: `/homework/session_01/exercise_06/deploy-local.sh`.
* Chụp màn hình terminal chạy deploy thành công in màu xanh lá, lưu file thành `deploy_success.png` trong thư mục trên.
* Chụp màn hình script tự động dừng khi phát hiện build lỗi ở Bước 1, lưu file thành `deploy_fail_fast.png` trong thư mục trên.
