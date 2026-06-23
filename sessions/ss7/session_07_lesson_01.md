# SESSION 07: CI/CD CƠ BẢN VỚI GITLAB

## LESSON 01: Tổng quan GitLab CI/CD và GitLab Runner

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Giải thích** được định nghĩa, vai trò và vị trí của GitLab CI/CD trong quy trình phát triển và vận hành DevOps.
* **Phân tích** kiến trúc Client-Server giữa GitLab Server và GitLab Runner trong việc nhận diện và điều phối các công việc (jobs).
* **Phân biệt** được các loại Runner (Shared Runner, Specific Runner) và các loại Executor phổ biến (Docker, Shell).
* **Thực hành cấu hình và đăng ký** thành công một GitLab Runner tự host cục bộ qua Docker Compose.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (NỖI ĐAU CỦA VIỆC KIỂM TRA & BUILD THỦ CÔNG)

Hãy tưởng tượng nhóm phát triển của bạn đang tiếp quản hệ thống QuickBite chạy đa dịch vụ. Mỗi khi một lập trình viên cập nhật mã nguồn (ví dụ: sửa logic ví tiền trong `user-service`), quy trình thông thường sẽ là:
1. Chạy unit test ở máy local để kiểm tra xem có làm hỏng tính năng cũ không.
2. Biên dịch mã nguồn ra file JAR (ví dụ: chạy `./gradlew bootJar`).
3. Đẩy code lên Git.

* **Nỗi đau bắt đầu:**
  * **Thiếu tính tự động hóa và nhất quán:** Lập trình viên có thể quên chạy bước 1 và 2 mà trực tiếp push code lên Git. Kết quả là mã nguồn lỗi hoặc không biên dịch được vẫn được đưa lên kho lưu trữ chung, làm ảnh hưởng đến cả đội ngũ.
  * **Sự khác biệt về môi trường:** Code chạy tốt trên máy của lập trình viên A (sử dụng JDK 17 cài sẵn) nhưng khi biên dịch ở máy lập trình viên B hoặc server (sử dụng JDK 21) thì sập do xung đột phiên bản compiler.

Để giải quyết bài toán này, quy trình phát triển phần mềm hiện đại yêu cầu thiết lập một hệ thống kiểm tra và biên dịch tự động, tập trung hóa trên máy chủ trung gian. Đó chính là lý do công cụ **GitLab CI/CD** và **GitLab Runner** được đưa vào sử dụng.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (KIẾN TRÚC GITLAB CI/CD)

#### 3.1 Khái niệm CI/CD
* **CI (Continuous Integration - Tích hợp liên tục):** Quy trình tự động hóa việc tích hợp mã nguồn mới từ nhiều lập trình viên. Mỗi khi có code mới được push lên Git, hệ thống sẽ tự động chạy các bước kiểm tra cú pháp, biên dịch và chạy test để đảm bảo chất lượng code.
* **CD (Continuous Delivery / Deployment - Chuyển giao/Triển khai liên tục):** Quy trình tự động hóa các bước tiếp theo như đóng gói Docker image, đẩy lên registry và tự động deploy sản phẩm mới lên máy chủ Production.

#### 3.2 Kiến trúc Client-Server của GitLab CI/CD
Quy trình CI/CD của GitLab hoạt động dựa trên sự phối hợp của hai thành phần độc lập:

```text
  [ GitLab Server (Web UI/Git Repo) ]
                │
     (Giao tiếp REST API / Polling)
                │
                ▼
  [ GitLab Runner (Máy chủ thực thi) ] ── (Chạy Jobs) ──► [ Docker / Host OS ]
```

* **GitLab Server:** Nơi lưu trữ mã nguồn, quản lý cấu hình, hiển thị giao diện trực quan và điều phối các luồng chạy pipeline. GitLab Server không trực tiếp chạy lệnh build hay test.
* **GitLab Runner:** Một dịch vụ/máy ảo độc lập đóng vai trò là tác nhân thực thi. Runner liên tục gửi yêu cầu lên GitLab Server (polling) để nhận các công việc (jobs) đang chờ xử lý, tải mã nguồn về, thực hiện các lệnh build rồi trả kết quả ngược lại cho Server hiển thị.

#### 3.3 Phân loại GitLab Runner
* **Shared Runner (Runner dùng chung):** Do hệ thống GitLab cung cấp sẵn và chia sẻ chung cho toàn bộ cộng đồng/doanh nghiệp. Shared Runner rất tiện dụng nhưng có nhược điểm là thời gian chờ hàng đợi (queue) lâu và tài nguyên giới hạn.
* **Specific Runner (Runner riêng biệt):** Do bạn tự cài đặt và đăng ký riêng cho một dự án cụ thể. Runner này hoạt động độc lập và chỉ nhận các jobs thuộc dự án được phân quyền, đảm bảo tốc độ tối đa.

#### 3.4 Khái niệm Executor trong Runner
Khi đăng ký Runner, bạn phải cấu hình **Executor** - công nghệ dùng để tạo môi trường chạy dòng lệnh:
* **Shell Executor:** Runner chạy trực tiếp các lệnh trên hệ điều hành máy host của nó. Phương pháp này dễ cấu hình nhưng thiếu tính cô lập (file sinh ra của job này có thể làm ảnh hưởng đến job khác).
* **Docker Executor (Khuyên dùng):** Mỗi job chạy trong một container Docker hoàn toàn sạch sẽ và độc lập (cô lập hoàn toàn). Khi job kết thúc, container tự động bị hủy, đảm bảo không để lại rác hệ thống.

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (CÀI ĐẶT VÀ ĐĂNG KÝ LOCAL RUNNER QUA DOCKER COMPOSE)

Dù trong chương trình học QuickBite, chúng ta định hướng tận dụng Shared Runner của GitLab để tối giản hóa tài nguyên, việc thực hành tự host một Local Specific Runner trên máy cá nhân giúp sinh viên hiểu rõ bản chất hoạt động.

#### 4.1 Khởi chạy dịch vụ GitLab Runner bằng Docker Compose
Tạo tệp tin `docker-compose.yml` tại thư mục làm việc để quản lý dịch vụ Runner:

```yaml
version: '3.8'

services:
  gitlab-runner:
    image: gitlab/gitlab-runner:latest
    container_name: local-gitlab-runner
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - gitlab-runner-config:/etc/gitlab-runner

volumes:
  gitlab-runner-config:
```

Khởi chạy container chạy ngầm bằng lệnh:
```bash
docker compose up -d
```

* **Giải thích thuộc tính volumes:**
  * `/var/run/docker.sock:/var/run/docker.sock`: Mount Docker socket của máy host vào container Runner. Việc này cho phép Runner (chạy bên trong container) có quyền điều khiển Docker Engine của host để khởi tạo các container phụ phục vụ cho quá trình build (Docker-in-Docker).
  * `gitlab-runner-config:/etc/gitlab-runner`: Named volume dùng để lưu trữ cấu hình bền vững của Runner sau khi đăng ký, tránh mất cấu hình khi container bị khởi động lại hoặc recreate.

#### 4.2 Đăng ký Runner vào GitLab bằng Authentication Token mới
Truy cập vào giao diện GitLab dự án của bạn, chọn **Settings** -> **CI/CD** -> **Runners**, bấm tạo Runner mới để lấy mã token có tiền tố là `GLRT-`. Sau đó chạy lệnh đăng ký trực tiếp thông qua Docker Compose:

```bash
docker compose exec gitlab-runner gitlab-runner register \
  --non-interactive \
  --url "https://gitlab.com/" \
  --token "GLRT-YOUR_NEW_AUTHENTICATION_TOKEN_HERE" \
  --executor "docker" \
  --docker-image "alpine:latest" \
  --description "Local Runner for QuickBite Project" \
  --tag-list "docker,quickbite" \
  --run-untagged="true" \
  --locked="false"
```

* **Giải thích tham số:**
  * `docker compose exec gitlab-runner`: Thực thi lệnh trực tiếp trong container `gitlab-runner` đang chạy.
  * `--token`: Mã xác thực của Runner mới.
  * `--executor "docker"`: Định nghĩa môi trường chạy là Docker container.
  * `--docker-image "alpine:latest"`: Docker image mặc định sử dụng nếu job trong file cấu hình không khai báo image cụ thể.
  * `--tag-list "docker,quickbite"`: Gắn thẻ tag cho Runner để phân phối job phù hợp.

* **Kết quả mong đợi:** Lệnh báo đăng ký thành công. Trên giao diện GitLab xuất hiện một Runner mới màu xanh lá cây sẵn sàng nhận job.

---

### PHẦN 5. LƯU Ý, LỖI SAI VÀ HIỂU LẦM THƯỜNG GẶP

* **Lỗi quên mount Docker Socket:** Sinh viên khi tự host Runner thường quên hoặc cố tình bỏ qua dòng volumes `/var/run/docker.sock`. Điều này làm Runner bị mất quyền giao tiếp với Docker Engine của máy host, dẫn đến lỗi sập toàn bộ các job sử dụng Docker Executor.
* **Sử dụng sai token đăng ký cũ:** GitLab phiên bản mới yêu cầu sử dụng Authentication Token (có tiền tố `GLRT-`) để đăng ký Runner. Nhiều sinh viên vẫn sử dụng Registration Token cũ của dự án, dẫn đến lỗi xác thực và không thể kích hoạt được Runner.
* **Hiểu lầm về vai trò của GitLab Server:** Nhiều sinh viên lầm tưởng GitLab Server trực tiếp biên dịch và kiểm thử mã nguồn. Thực tế, GitLab Server chỉ điều phối và hiển thị kết quả; việc thực thi hoàn toàn do GitLab Runner đảm nhận. Nếu không cấu hình Runner, pipeline sẽ bị kẹt ở trạng thái `pending` vô hạn.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG (XÁC MINH KIẾN THỨC)

1. **Tổng quan về GitLab CI/CD:**
   * [GitLab CI/CD Official Documentation](https://docs.gitlab.com/ee/ci/)
2. **Hướng dẫn cài đặt và đăng ký GitLab Runner:**
   * [GitLab Runner Registration Guide](https://docs.gitlab.com/ee/runner/register/)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao GitLab Server không trực tiếp thực thi các dòng lệnh trong pipeline (như chạy test hay compile Java) mà bắt buộc phải thông qua GitLab Runner?
* *Gợi ý:* Để đảm bảo hiệu năng và tính cô lập. GitLab Server chỉ quản lý giao diện, database và điều hướng. Việc thực thi job rất tốn tài nguyên CPU/RAM, do đó phải đẩy việc này sang các máy chủ Runner độc lập nhằm tránh làm sập máy chủ quản lý trung tâm khi có hàng nghìn lập trình viên cùng push code.

#### Câu 2 (Phân tích so sánh)
Sự khác biệt cốt lõi về môi trường chạy giữa Docker Executor và Shell Executor của GitLab Runner là gì?
* *Gợi ý:* Docker Executor tạo ra một container hoàn toàn sạch sẽ, độc lập cho mỗi job và tự hủy container đó khi job kết thúc, đảm bảo không để lại rác hoặc gây xung đột môi trường. Shell Executor chạy trực tiếp trên hệ điều hành của máy host chứa Runner, làm các job dùng chung môi trường và dễ gây lỗi xung đột file/thư viện.

#### Câu 3 (Cấu hình hệ thống)
Khi cài đặt GitLab Runner bằng Docker, tham số mount `-v /var/run/docker.sock:/var/run/docker.sock` có vai trò gì?
* *Gợi ý:* Cho phép container GitLab Runner giao tiếp trực tiếp với Docker Daemon của máy host vật lý. Nhờ đó, Runner có thể gửi lệnh tạo, khởi chạy hoặc xóa các container Docker phụ phục vụ cho việc thực thi job của pipeline.
