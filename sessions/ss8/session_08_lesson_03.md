# SESSION 08: ĐÓNG GÓI DOCKER IMAGE & ĐẨY LÊN REGISTRY

## LESSON 03: Phiên bản hóa và Đẩy Docker Image lên Registry từ Local

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, học viên có khả năng:
* **Giải thích** được vai trò và cơ chế hoạt động của GitLab Container Registry.
* **Áp dụng** các quy tắc đặt tên và gắn tag phiên bản (tagging) cho Docker image.
* **Thực hiện** đăng nhập bảo mật và đẩy (push) Docker image từ local lên Registry trực tuyến thành công.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (GIẢI QUYẾT BOTTLENECK RUNTIME TRONG CI)

Trong các bài học trước (State 1 & 2), chúng ta tự động hóa việc build Docker image trực tiếp trên GitLab Runner. Tuy nhiên, việc này phát sinh một **pain point** lớn trong thực tế vận hành:
* Mỗi lượt commit code, GitLab Runner phải khởi chạy từ đầu, tải dependencies Gradle và build image. Tiến trình này tốn khoảng **5 phút** runtime.
* Nếu dự án có nhiều microservices và deploy liên tục, thời gian chờ đợi sẽ rất lớn và chi phí tài nguyên (số phút chạy Runner) sẽ bị cạn kiệt nhanh chóng.

**Giải pháp tối ưu chi phí và thời gian (State 3: Build & Push từ Local):**
* Chuyển quy trình build và versioning về máy local cá nhân của lập trình viên. Máy local tận dụng toàn bộ bộ nhớ cache dependencies Gradle (`~/.gradle`) và cache layer của Docker Desktop nên tốc độ build chỉ mất vài giây.
* Đóng gói xong, lập trình viên đặt nhãn phiên bản và đẩy (push) image trực tiếp lên kho lưu trữ tập trung (**GitLab Container Registry**). Pipeline CI/CD sau đó chỉ cần kéo image này về chạy mà không phải build lại.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI

#### 3.1 GitLab Container Registry
GitLab Container Registry là kho lưu trữ Docker image riêng tư (private) đi kèm với từng dự án GitLab.
* Địa chỉ máy chủ lưu trữ mặc định: `registry.gitlab.com`.
* Đường dẫn lưu image của dự án có dạng: `registry.gitlab.com/<namespace>/user-service` (với `<namespace>` là username hoặc group của học viên trên GitLab).
* Quản lý trực quan: Xem danh sách image đã đẩy lên tại giao diện Web của GitLab -> **Deploy** -> **Container Registry**. Sau khi đăng nhập thành công bằng lệnh `docker login registry.gitlab.com`, học viên có thể truy cập trực tiếp đường dẫn `https://gitlab.com/<namespace>/user-service/container_registry` trên trình duyệt để xem trong kho chứa những gì.

#### 3.2 Quy tắc đặt nhãn (Tagging) và Phiên bản hóa (Versioning)
Để đẩy được image lên Registry, tên của image nguồn ở máy local bắt buộc phải khớp chính xác với đường dẫn Registry được GitLab phân quyền:
```bash
docker tag <local_image_name> registry.gitlab.com/<namespace>/<project_name>:<version_tag>
```
* **Version tag:** Phân bản image rõ ràng theo phiên bản phát hành (ví dụ: `1.0.0`, `1.1.0-RC1`) để phục vụ quản lý phiên bản và rollback khi có lỗi, tránh dùng nhãn tĩnh `latest`.

#### 3.3 Đăng nhập bảo mật từ Local bằng Access Token
Vì GitLab Registry là private, máy local cần đăng nhập để xác thực quyền ghi (push). Để đảm bảo an toàn thông tin:
* **Tuyệt đối không sử dụng mật khẩu tài khoản GitLab chính** để đăng nhập từ terminal.
* Sử dụng **Personal Access Token (PAT)** hoặc **Project Deploy Token** được cấp quyền giới hạn (`write_registry`, `read_registry`).

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (BUILD VÀ PUSH IMAGE TỪ LOCAL)

Học viên thực hiện đóng gói, gắn tag phiên bản và đẩy image của microservice `user-service` từ máy phát triển cá nhân lên GitLab Container Registry.

#### 4.1 Tạo Personal Access Token trên GitLab
1. Trên giao diện Web của GitLab, click chọn ảnh đại diện ở góc trên bên trái -> **Preferences** -> **Access Tokens**.
2. Nhấp chọn **Add new token**.
3. Điền tên token (ví dụ: `local-docker-access`), chọn Scopes: `write_registry` và `read_registry`.
4. Nhấp **Create personal access token** và sao chép mã token hiển thị (mã này chỉ xuất hiện 1 lần duy nhất).

#### 4.2 Thực thi các lệnh build và đẩy image tại Local Terminal
Mở terminal tại máy cá nhân, di chuyển vào thư mục gốc của dự án `user-service` và thực hiện chuỗi lệnh:

```bash
# 1. Đăng nhập an toàn vào Registry sử dụng Personal Access Token
# Thay <gitlab_username> bằng tên đăng nhập và nhập token vừa tạo khi terminal hỏi mật khẩu (Password)
docker login registry.gitlab.com -u <gitlab_username>

# 2. Biên dịch JAR nhanh chóng tại local (nhờ tận dụng cache dependencies Gradle có sẵn)
./gradlew bootJar

# 3. Build Docker image tại local sử dụng Dockerfile đơn tầng (đã chuẩn bị ở Lesson 01)
docker build -t user-service:1.0.0 .

# 4. Gắn tag Registry cho image (thay thế <namespace> bằng username hoặc group của bạn trên GitLab)
docker tag user-service:1.0.0 registry.gitlab.com/<namespace>/user-service:1.0.0

# 5. Đẩy image lên GitLab Container Registry trực tuyến
docker push registry.gitlab.com/<namespace>/user-service:1.0.0
```

> [!IMPORTANT]
> **Không kiểm tra trạng thái trong after_script:**
> Việc kiểm chứng trạng thái đẩy image được thực hiện trực tiếp thông qua log output thành công của lệnh `docker push` tại terminal local và xác minh trên giao diện Web UI của GitLab.

#### 4.3 Kết quả mong đợi
Terminal in ra log tiến trình tải các layer lên mạng internet và kết thúc bằng thông báo thành công. Khi truy cập vào dự án GitLab trên trình duyệt, mục **Deploy** -> **Container Registry** hiển thị image `user-service` đi kèm tag `1.0.0` hoạt động chính xác.

---

### PHẦN 5. LƯU Ý, LỖI SAI VÀ HIỂU LẦM THƯỜNG GẶP (DÀNH CHO HỌC VIÊN)

* **Lỗi từ chối phân quyền (`denied: requested access to the resource is denied`):** Lỗi xảy ra do học viên gõ sai cấu trúc đường dẫn Registry (ví dụ sai username hoặc sai tên dự án), hoặc quên đăng nhập `docker login` thành công trước đó.
  *Khắc phục:* Kiểm tra kỹ địa chỉ registry của dự án trên giao diện GitLab Web UI và thực hiện đăng nhập lại bằng Personal Access Token có đủ quyền.
* **Lộ lọt mật khẩu cá nhân:** Sinh viên gõ mật khẩu tài khoản chính của GitLab trực tiếp vào terminal hoặc lưu trong các file script tự động.
  *Khắc phục:* Luôn dùng Personal Access Token hoặc Deploy Token để đăng nhập trên terminal, đảm bảo tính bảo mật tối đa.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG

1. **Tài liệu hướng dẫn GitLab Container Registry:**
   * [GitLab Container Registry | GitLab Documentation](https://docs.gitlab.com/ee/user/packages/container_registry/)
2. **Quản lý Personal Access Tokens trên GitLab:**
   * [Personal access tokens | GitLab Documentation](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao việc sử dụng Personal Access Token lại an toàn hơn so với việc sử dụng mật khẩu tài khoản GitLab chính để đăng nhập Docker Registry từ máy local?
* *Gợi ý:* Vì Personal Access Token có thể được cấu hình giới hạn quyền (chỉ cho phép đọc/ghi registry) và có thời hạn sử dụng. Nếu token bị lộ, kẻ xấu cũng không thể chiếm quyền kiểm soát toàn bộ tài khoản GitLab hoặc sửa đổi mã nguồn dự án. Hơn nữa, lập trình viên có thể dễ dàng thu hồi (revoke) token bất kỳ lúc nào mà không cần đổi mật khẩu chính.

#### Câu 2 (Phân tích so sánh)
Sự khác biệt giữa lệnh `docker build -t` và lệnh `docker tag` trong chuỗi lệnh thực hành trên máy local là gì?
* *Gợi ý:* Lệnh `docker build -t` trực tiếp biên dịch mã nguồn và đóng gói để tạo ra một Docker image mới lưu cục bộ dưới máy. Lệnh `docker tag` chỉ tạo thêm một bí danh (alias/pointer) trỏ đến image đã có sẵn đó, định dạng lại tên theo đúng địa chỉ của Registry trực tuyến để chuẩn bị cho lệnh push.

#### Câu 3 (Cấu hình hệ thống)
Nếu đường dẫn Registry của dự án học viên là `registry.gitlab.com/nguyena/java-project` và học viên muốn đẩy image dịch vụ `order-service` với phiên bản `2.1.0`, câu lệnh gắn tag và push tương ứng sẽ là gì?
* *Gợi ý:*
  ```bash
  docker tag order-service:latest registry.gitlab.com/nguyena/java-project/order-service:2.1.0
  docker push registry.gitlab.com/nguyena/java-project/order-service:2.1.0
  ```
