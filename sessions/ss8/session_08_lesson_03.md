# SESSION 08: ĐÓNG GÓI DOCKER IMAGE & ĐẨY LÊN REGISTRY

## LESSON 03: Phiên bản hóa và Đẩy Docker Image lên Registry từ Local

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, học viên có khả năng:
* **Giải thích** được vai trò và cơ chế hoạt động của GitHub Container Registry (ghcr.io).
* **Áp dụng** các quy tắc đặt tên và gắn tag phiên bản (tagging) cho Docker image.
* **Thực hiện** đăng nhập bảo mật và đẩy (push) Docker image từ local lên Registry trực tuyến thành công.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (BÀI THỰC HÀNH SƯ PHẠM VỚI DOCKER REGISTRY)

Trong các bài học trước (State 1 & 2), chúng ta tự động hóa việc build Docker image trực tiếp trên GitHub Actions Runner. Tuy nhiên, để làm quen với các khái niệm cốt lõi của **GitHub Container Registry (GHCR)**, học viên cần nắm vững các thao tác thủ công từ môi trường phát triển local.

**Mục đích của kịch bản thực hành (State 3: Build & Push từ Local):**
* **Mô phỏng từng bước (Step-by-step Learning):** Tách biệt tiến trình đóng gói và đẩy image để trực quan hóa cách thức xác thực, gắn nhãn (tag) phiên bản và truyền tải dữ liệu bằng Docker CLI.
* **Tối ưu hóa thời gian thực hành:** Tận dụng bộ nhớ cache dependencies Gradle (`~/.gradle`) và Docker layer có sẵn tại máy local giúp đẩy nhanh tiến độ làm bài thực hành.

> [!WARNING]
> **Không áp dụng trong môi trường Production thực tế:**
> Quy trình build và push image thủ công từ máy local của lập trình viên là một **anti-pattern** (phản hoa văn) và vi phạm nghiêm trọng nguyên tắc bảo mật. Máy local không được kiểm soát an toàn như Runner, đồng thời quy trình này gây mất tính nhất quán của mã nguồn (image được push lên có thể chứa code chưa được commit trên Git). Trong thực tế, toàn bộ tiến trình này bắt buộc phải chạy tự động hóa hoàn toàn trên luồng CI/CD.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI

#### 3.1 GitHub Container Registry (ghcr.io)
GitHub Container Registry (GHCR) là kho lưu trữ Docker image nằm trong hệ sinh thái GitHub Packages đi kèm với từng tài khoản hoặc tổ chức GitHub.
* Địa chỉ máy chủ lưu trữ mặc định: `ghcr.io`.
* Đường dẫn lưu image của dự án có dạng: `ghcr.io/<github_username>/user-service` (với `<github_username>` là username của học viên trên GitHub).
* Quản lý trực quan: Xem danh sách image đã đẩy lên tại trang Profile GitHub của bạn -> tab **Packages**.

#### 3.2 Quy tắc đặt nhãn (Tagging) và Phiên bản hóa (Versioning)
Để đẩy được image lên Registry, tên của image nguồn ở máy local bắt buộc phải khớp chính xác với đường dẫn Registry được GitHub phân quyền:
```bash
docker tag <local_image_name> ghcr.io/<github_username>/<project_name>:<version_tag>
```
* **Version tag:** Phân bản image rõ ràng theo phiên bản phát hành (ví dụ: `1.0.0`, `1.1.0-RC1`) để phục vụ quản lý phiên bản và rollback khi có lỗi, tránh dùng nhãn tĩnh `latest`.

#### 3.3 Đăng nhập bảo mật từ Local bằng Access Token
Vì bạn muốn ghi (push) image lên GHCR, máy local cần đăng nhập để xác thực quyền ghi. Để đảm bảo an toàn thông tin:
* **Tuyệt đối không sử dụng mật khẩu tài khoản GitHub chính** để đăng nhập từ terminal.
* Sử dụng **Personal Access Token (PAT)** được cấp quyền giới hạn (`write:packages`, `read:packages`).

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (BUILD VÀ PUSH IMAGE TỪ LOCAL)

Học viên thực hiện đóng gói, gắn tag phiên bản và đẩy image của microservice `user-service` từ máy phát triển cá nhân lên GitHub Container Registry.

#### 4.1 Tạo Personal Access Token trên GitHub
1. Trên giao diện Web của GitHub, click chọn ảnh đại diện ở góc trên bên phải -> **Settings** -> **Developer settings** (dưới cùng bên trái).
2. Nhấp chọn **Personal access tokens** -> **Tokens (classic)** -> **Generate new token (classic)**.
3. Điền tên token (Note: `local-docker-access`), tích chọn các Scopes: `write:packages` và `read:packages`.
4. Nhấp **Generate token** và sao chép mã token bắt đầu bằng `ghp_` hiển thị (mã này chỉ xuất hiện 1 lần duy nhất).

#### 4.2 Thực thi các lệnh build và đẩy image tại Local Terminal
Mở terminal tại máy cá nhân, di chuyển vào thư mục gốc của dự án `user-service` và thực hiện chuỗi lệnh:

```bash
# 1. Đăng nhập an toàn vào Registry sử dụng Personal Access Token
# Thay <github_username> bằng tên đăng nhập và nhập token vừa tạo khi terminal hỏi mật khẩu (Password)
docker login ghcr.io -u <github_username>

# 2. Biên dịch JAR nhanh chóng tại local (nhờ tận dụng cache dependencies Gradle có sẵn)
./gradlew bootJar

# 3. Build Docker image tại local sử dụng Dockerfile đơn tầng (đã chuẩn bị ở Lesson 01)
docker build -t user-service:1.0.0 .

# 4. Gắn tag Registry cho image (thay thế <github_username> bằng username của bạn)
docker tag user-service:1.0.0 ghcr.io/<github_username>/user-service:1.0.0

# 5. Đẩy image lên GitHub Container Registry trực tuyến
docker push ghcr.io/<github_username>/user-service:1.0.0
```

> [!IMPORTANT]
> **Kiểm tra trạng thái:**
> Việc kiểm chứng trạng thái đẩy image được thực hiện trực tiếp thông qua log output thành công của lệnh `docker push` tại terminal local và xác minh trên giao diện Web UI của GitHub (tab Packages).

#### 4.3 Kết quả mong đợi
Terminal in ra log tiến trình tải các layer lên mạng internet và kết thúc bằng thông báo thành công. Khi truy cập vào Profile GitHub cá nhân trên trình duyệt, mục **Packages** hiển thị image `user-service` đi kèm tag `1.0.0` hoạt động chính xác. (Lưu ý: Mặc định package sẽ ở chế độ Private, bạn có thể vào Package Settings để đổi sang Public nếu cần tải xuống không cần xác thực).

---

### PHẦN 5. LƯU Ý, LỖI SAI VÀ HIỂU LẦM THƯỜNG GẶP (DÀNH CHO HỌC VIÊN)

* **Lỗi từ chối phân quyền (`denied: requested access to the resource is denied`):** Lỗi xảy ra do học viên gõ sai cấu trúc đường dẫn Registry (ví dụ sai username hoặc sai tên dự án), hoặc quên đăng nhập `docker login` thành công trước đó, hoặc PAT bị thiếu quyền `write:packages`.
  *Khắc phục:* Kiểm tra kỹ địa chỉ registry và tạo lại/kiểm tra lại PAT có đủ quyền. Đăng nhập lại bằng `docker login ghcr.io`.
* **Lộ lọt mật khẩu cá nhân:** Sinh viên gõ mật khẩu tài khoản chính của GitHub trực tiếp vào terminal hoặc lưu trong các file script tự động.
  *Khắc phục:* Luôn dùng Personal Access Token (PAT) để đăng nhập trên terminal, đảm bảo tính bảo mật tối đa.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG

1. **Tài liệu hướng dẫn GitHub Container Registry:**
   * [Working with the Container registry | GitHub Documentation](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
2. **Quản lý Personal Access Tokens trên GitHub:**
   * [Managing your personal access tokens | GitHub Documentation](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao việc sử dụng Personal Access Token lại an toàn hơn so với việc sử dụng mật khẩu tài khoản GitHub chính để đăng nhập Docker Registry từ máy local?
* *Gợi ý:* Vì Personal Access Token có thể được cấu hình giới hạn quyền (chỉ cho phép đọc/ghi packages) và có thời hạn sử dụng. Nếu token bị lộ, kẻ xấu cũng không thể chiếm quyền kiểm soát toàn bộ tài khoản GitHub hoặc sửa đổi mã nguồn dự án. Hơn nữa, lập trình viên có thể dễ dàng thu hồi (revoke) token bất kỳ lúc nào mà không cần đổi mật khẩu chính.

#### Câu 2 (Phân tích so sánh)
Sự khác biệt giữa lệnh `docker build -t` và lệnh `docker tag` trong chuỗi lệnh thực hành trên máy local là gì?
* *Gợi ý:* Lệnh `docker build -t` trực tiếp biên dịch mã nguồn và đóng gói để tạo ra một Docker image mới lưu cục bộ dưới máy. Lệnh `docker tag` chỉ tạo thêm một bí danh (alias/pointer) trỏ đến image đã có sẵn đó, định dạng lại tên theo đúng địa chỉ của Registry trực tuyến (`ghcr.io`) để chuẩn bị cho lệnh push.

#### Câu 3 (Cấu hình hệ thống)
Nếu username GitHub của học viên là `nguyena` và học viên muốn đẩy image dịch vụ `order-service` với phiên bản `2.1.0`, câu lệnh gắn tag và push tương ứng sẽ là gì?
* *Gợi ý:*
  ```bash
  docker tag order-service:latest ghcr.io/nguyena/order-service:2.1.0
  docker push ghcr.io/nguyena/order-service:2.1.0
  ```
