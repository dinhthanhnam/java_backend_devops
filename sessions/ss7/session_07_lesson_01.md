# SESSION 07: TỰ ĐỘNG HÓA QUY TRÌNH CI/CD VỚI GITHUB ACTIONS

## LESSON 01: Tổng quan GitHub Actions và Kiến trúc Runner

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, học viên có khả năng:
* **Giải thích** được định nghĩa, vai trò và vị trí của GitHub Actions trong quy trình phát triển và vận hành DevOps.
* **Phân tích** kiến trúc phân tán giữa GitHub và GitHub Actions Runner trong việc nhận diện và điều phối các công việc (jobs).
* **Phân biệt** được các loại Runner (GitHub-hosted runner, Self-hosted runner) và đặc thù tài nguyên của chúng.
* **Thực hành cấu hình và đăng ký** thành công một Self-hosted runner cục bộ thông qua script chuẩn của GitHub.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (HẠN CHẾ CỦA VIỆC KIỂM TRA & BUILD THỦ CÔNG)

Hệ thống QuickBite hoạt động theo kiến trúc đa dịch vụ (microservices). Mỗi khi một lập trình viên cập nhật mã nguồn (ví dụ: sửa đổi logic trong `user-service`), quy trình thông thường bao gồm:
1. Chạy unit test cục bộ để xác minh tính toàn vẹn của mã nguồn.
2. Biên dịch mã nguồn thành tệp tin JAR (ví dụ: chạy `./gradlew bootJar`).
3. Đẩy mã nguồn lên hệ thống quản lý phiên bản Git.

* **Thách thức kỹ thuật:**
  * **Thiếu tính tự động hóa và nhất quán:** Lập trình viên có thể bỏ qua bước 1 và 2, trực tiếp đẩy mã nguồn lên Git. Hệ quả là mã nguồn lỗi hoặc không biên dịch được vẫn được hợp nhất vào kho lưu trữ chung, gây gián đoạn quy trình của toàn đội.
  * **Sự khác biệt về môi trường:** Mã nguồn có thể hoạt động ổn định trên máy của lập trình viên A (sử dụng JDK 17) nhưng khi biên dịch ở máy của lập trình viên B (sử dụng JDK 21) lại phát sinh lỗi do xung đột phiên bản trình biên dịch (compiler).

Để giải quyết vấn đề này, quy trình phát triển phần mềm hiện đại yêu cầu thiết lập một hệ thống kiểm thử và biên dịch tự động, tập trung hóa trên một máy chủ độc lập. Đó là lý do hệ thống **GitHub Actions** được triển khai.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (KIẾN TRÚC GITHUB ACTIONS)

#### 3.1 Khái niệm CI/CD
* **CI (Continuous Integration - Tích hợp liên tục):** Quy trình tự động hóa việc hợp nhất mã nguồn mới từ các lập trình viên. Mỗi khi có mã nguồn mới được đẩy lên Git, hệ thống sẽ tự động chạy các bước kiểm tra cú pháp, biên dịch và chạy kiểm thử để xác minh chất lượng.
* **CD (Continuous Delivery / Deployment - Chuyển giao/Triển khai liên tục):** Quy trình tự động hóa các bước tiếp theo như đóng gói Docker image, đẩy lên registry và triển khai sản phẩm lên máy chủ vận hành (Production).

#### 3.2 Kiến trúc điều phối của GitHub Actions
Quy trình CI/CD của GitHub Actions hoạt động dựa trên sự phối hợp của hai thành phần độc lập:

```text
  [ GitHub Server (Web UI/Git Repo) ]
                 │
      (Giao tiếp REST API / Polling)
                 │
                 ▼
  [ GitHub Actions Runner (Máy chủ thực thi) ] ── (Chạy Jobs) ──► [ Môi trường Host OS ]
```

* **GitHub Server:** Nơi lưu trữ mã nguồn, hiển thị giao diện quản lý và điều phối các luồng sự kiện (workflow). GitHub Server không trực tiếp thực thi các lệnh biên dịch hay kiểm thử.
* **GitHub Actions Runner:** Ứng dụng độc lập đóng vai trò tác nhân thực thi. Runner liên tục kết nối với GitHub Server (thông qua cơ chế long-polling) để nhận các công việc (jobs) đang chờ xử lý, tải mã nguồn, thực thi kịch bản và gửi log trả về cho hệ thống trung tâm.

#### 3.3 Phân loại GitHub Actions Runner
* **GitHub-hosted runner:** Các máy ảo (VM) do chính GitHub quản lý và cung cấp sẵn (chạy Linux, Windows hoặc macOS). Chúng sạch sẽ, được khởi tạo mới hoàn toàn cho mỗi công việc, nhưng có giới hạn về cấu hình phần cứng và thời gian thực thi miễn phí.
* **Self-hosted runner:** Máy chủ vật lý hoặc máy ảo do người dùng tự cài đặt ứng dụng Runner và đăng ký vào kho lưu trữ (repository) cụ thể. Loại này cung cấp quyền kiểm soát hoàn toàn về phần cứng, bộ nhớ đệm (cache) và không bị tính phí thời gian chạy bởi GitHub.

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (CÀI ĐẶT SELF-HOSTED RUNNER BẰNG DOCKER COMPOSE)

Mặc dù việc cấu hình Script trực tiếp trên máy chủ là tiêu chuẩn chung, việc triển khai **Self-hosted runner thông qua Docker Compose** mang lại ưu điểm vượt trội về tính cô lập (isolation) và quản lý vòng đời (lifecycle management), duy trì môi trường hệ thống tương tự như kiến trúc của GitLab Runner.

#### 4.1 Lựa chọn cấp độ quản lý Runner: Repo-level vs Org-level
Hệ thống cho phép cấu hình Runner ở hai cấp độ khác nhau. Việc nắm rõ sự khác biệt giúp tối ưu hóa tài nguyên phần cứng:

| Tiêu chí | Cấp độ Repository (Repo-level) | Cấp độ Tổ chức (Org-level) |
| :--- | :--- | :--- |
| **Phạm vi hoạt động** | Chỉ nhận công việc từ 1 repository duy nhất. | Nhận công việc từ **tất cả** các repository nằm trong Organization đó. |
| **Bảo mật & Phân quyền** | Quản trị độc lập trong nội bộ dự án. | Quản lý tập trung bằng **Runner Groups** (Quy định repo nào được phép dùng). |
| **Tối ưu tài nguyên** | Dễ lãng phí nếu dự án không có luồng build liên tục. | Tối ưu cực tốt (Khuyên dùng cho kiến trúc Microservices như QuickBite). |

#### 4.2 Lấy Token xác thực trên GitHub
Tùy thuộc vào cấp độ Runner được chọn, vị trí lấy mã định danh bảo mật (Token) sẽ khác nhau:

* **Đối với Repo-level:**
  1. Truy cập vào kho lưu trữ tại: `https://github.com/<REPO_OWNER>/<REPO_NAME>`
  2. Chọn **Settings** -> **Actions** -> **Runners**.
  3. Nhấp vào nút **New self-hosted runner** và sao chép chuỗi Token.

* **Đối với Org-level:**
  1. Truy cập vào giao diện quản trị tổ chức tại: `https://github.com/organizations/<ORG_NAME>/settings/profile`
  2. Tại thanh điều hướng bên trái, chọn **Actions** -> **Runners**.
  3. Nhấp vào nút **New runner** -> **New self-hosted runner** và sao chép chuỗi Token.

#### 4.3 Khởi chạy dịch vụ Runner bằng Docker Compose
Tạo tệp tin `docker-compose.yml` tại thư mục cấu hình trên máy chủ để quản lý dịch vụ Runner. Dưới đây là cấu hình hỗ trợ cả hai phương pháp:

```yaml
services:
  action-runner:
    image: myoung34/github-runner:latest
    container_name: action-runner
    environment:
      # KHAI BÁO CẤP ĐỘ QUẢN LÝ (CHỌN 1 TRONG 2 CÁCH DƯỚI ĐÂY)
      # Cách 1 (Dành cho Repo-level):
      # REPO_URL: "https://github.com/<REPO_OWNER>/<REPO_NAME>"
      
      # Cách 2 (Dành cho Org-level):
      RUNNER_SCOPE: "org"
      ORG_NAME: "backend-fullskill-devops"
      
      # CẤU HÌNH ĐỊNH DANH
      RUNNER_NAME: "quickbite-runner-01"
      RUNNER_TOKEN: "YOUR_SECURE_TOKEN_HERE" # Lấy từ bước 4.2 tương ứng
      LABELS: "self-hosted,quickbite,backend,ubuntu"
      EPHEMERAL: "true"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    restart: always
```

* **Giải thích các tham số cấu hình:**
  * `image`: Sử dụng bản phân phối mã nguồn mở phổ biến nhất để đóng gói GitHub Runner vào Docker.
  * `ORG_RUNNER` & `ORG_NAME` (hoặc `REPO_URL`): Tham số định vị hệ thống đích để Runner kết nối.
  * `RUNNER_TOKEN`: Chuỗi xác thực bảo mật sinh ra từ hệ thống, dùng để thiết lập tin cậy giữa máy chủ và GitHub.
  * `LABELS`: Các nhãn dùng để điều phối luồng công việc (Workflow routing). Hệ thống sẽ định tuyến các công việc yêu cầu `runs-on: [self-hosted, quickbite]` về máy chủ này.
  * `EPHEMERAL: "true"`: Đảm bảo máy chủ tự động khởi tạo lại môi trường sạch sau mỗi lượt chạy (Job), ngăn ngừa xung đột dữ liệu tồn dư.
  * `volumes`: Gắn kết (Mount) Docker Socket để hỗ trợ các thao tác biên dịch ảo hóa bên trong Runner. Thư mục `_work` được gắn kết ra một Volume chuyên biệt để tối ưu hóa chia sẻ mã nguồn.

#### 4.4 Kích hoạt hệ thống
Khởi động container bằng lệnh:
```bash
docker compose up -d
```

* **Kết quả kỳ vọng:** Dịch vụ Runner khởi chạy ẩn dưới dạng nền (Background Service). Trên giao diện GitHub (tương ứng với cấp độ đã chọn), trạng thái của runner `quickbite-runner-01` sẽ chuyển sang `Idle` (màu xanh lá cây), sẵn sàng tiếp nhận công việc.

---

### PHẦN 5. LƯU Ý, LỖI SAI VÀ HIỂU LẦM THƯỜNG GẶP

* **Quên thiết lập Volume Docker Socket:** Nếu quy trình biên dịch của bạn cần thực thi lệnh ảo hoá bên trong GitHub Actions, việc thiếu cấu hình `- /var/run/docker.sock:/var/run/docker.sock` sẽ khiến hệ thống từ chối quyền truy cập (Permission Denied) tới Docker Daemon.
* **Tái sử dụng Token quá hạn:** Token sinh ra từ giao diện GitHub chỉ có hiệu lực giới hạn trong vòng 1 giờ vì lý do bảo mật. Nếu sinh viên lưu lại tệp cấu hình và chạy `docker compose up -d` sau 1 giờ, hệ thống sẽ báo lỗi `Http response code: NotFound` ở log khởi chạy. Cần tạo lại token mới từ giao diện Web và cập nhật biến `RUNNER_TOKEN`.
* **Bỏ qua thuộc tính Ephemeral:** Nếu tắt biến `EPHEMERAL` (đặt thành `false`), container sẽ giữ nguyên dữ liệu mã nguồn và tệp sinh ra sau khi chạy xong. Điều này gây mất tính nhất quán cho các công việc chạy ở lần tiếp theo do xung đột mã nguồn.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG

1. **Tổng quan về GitHub Actions:**
   * [Understanding GitHub Actions](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions)
2. **Hướng dẫn cấu hình Self-hosted runners:**
   * [Hosting your own runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao GitHub Server không trực tiếp biên dịch mã nguồn (như chạy lệnh Java compiler) mà phải phân phối lệnh đó xuống máy chủ GitHub Actions Runner?
* *Gợi ý:* Để đảm bảo hiệu năng và khả năng mở rộng. Việc biên dịch tiêu tốn lượng lớn CPU/RAM. Nếu GitHub trực tiếp biên dịch cho hàng triệu kho lưu trữ cùng lúc, máy chủ trung tâm sẽ quá tải. Cơ chế phân tán xuống các máy Runner giúp chia sẻ gánh nặng tính toán và đảm bảo tính cô lập tài nguyên cho từng dự án.

#### Câu 2 (Phân tích so sánh)
Sự khác biệt cốt lõi về môi trường chạy giữa GitHub-hosted runner và Self-hosted runner là gì?
* *Gợi ý:* GitHub-hosted runner cung cấp một máy ảo nguyên bản, hoàn toàn sạch sẽ cho mỗi lượt chạy, sau đó sẽ bị hủy để đảm bảo không bị xung đột với các phiên chạy trước. Self-hosted runner chạy trên máy do người dùng tự quản lý, trạng thái hệ điều hành (file tạm, thư viện đã tải) được giữ nguyên giữa các lượt chạy, dẫn đến nguy cơ xung đột môi trường nếu không được dọn dẹp kỹ.

#### Câu 3 (Cấu hình hệ thống)
Mã Token định danh khi cấu hình Self-hosted runner có ý nghĩa bảo mật như thế nào?
* *Gợi ý:* Token đóng vai trò xác thực, đảm bảo rằng chỉ có các máy chủ Runner được người quản trị ủy quyền mới có thể kết nối vào dự án và lấy mã nguồn về thực thi, ngăn chặn các máy chủ trái phép tiếp cận dữ liệu hệ thống.
