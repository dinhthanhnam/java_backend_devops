# SESSION 08: ĐÓNG GÓI DOCKER IMAGE & ĐẨY LÊN REGISTRY

## LESSON 04: Sử dụng Docker Image từ Registry trong Luồng CI/CD

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, học viên có khả năng:
* **Cấu hình** thành công file `.github/workflows/ci.yml` để tự động xác thực và kéo (pull) image từ GitHub Container Registry (GHCR) về máy Runner.
* **Tận dụng** các biến môi trường ẩn định sẵn (Predefined Variables) của GitHub Actions để bảo vệ thông tin xác thực.
* **Giải thích** được cơ chế tối ưu hiệu năng và thời gian chạy (runtime) của luồng công việc khi chuyển đổi từ build sang pull image.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (KỊCH BẢN KÉO IMAGE TỪ REGISTRY TRONG CI/CD)

Sau khi thực hành đẩy thành công Docker image lên GitHub Container Registry từ local ở Lesson 03 (State 3), câu hỏi đặt ra là: Làm thế nào để luồng CI/CD tích hợp và sử dụng image này?

**Vai trò của kịch bản thực hành (State 4: Pull & Run trong CI/CD):**
* Giúp học viên làm quen với lệnh kéo image (`docker pull`) và khởi chạy container verify tự động từ bên trong script của Runner.
* Hiểu cách xác thực tự động với GitHub Container Registry sử dụng các biến bảo mật mặc định do GitHub cung cấp sẵn mà không cần cấu hình tài khoản cá nhân.

> [!IMPORTANT]
> **Lưu ý về quy trình thực tế:**
> Việc tách biệt biên dịch ở local và kéo chạy ở CI/CD là phương án giả lập phục vụ mục đích sư phạm để học viên tiếp cận dần với khái niệm tích hợp Registry. Trong luồng vận hành thực tế của doanh nghiệp, để đảm bảo tính an toàn thông tin và tính nhất quán của mã nguồn, luồng CI/CD sẽ chịu trách nhiệm tự động hóa toàn bộ: Biên dịch mã nguồn -> Đóng gói và Push image -> Kéo về chạy thử nghiệm và triển khai (Deploy).

---

### PHẦN 3. KIẾN THỨC CỐT LÕI

#### 3.1 Sử dụng Predefined Variables để tự động đăng nhập trong CI/CD
Khi chạy trong môi trường Runner, chúng ta không được và không cần cấu hình thủ công Personal Access Token cá nhân vào file YAML. GitHub Actions tự động cấp phát một token tạm thời (temporary token) chỉ có hiệu lực trong lúc job chạy và nạp sẵn vào các biến ngữ cảnh (context variables):
* **`ghcr.io`:** Địa chỉ máy chủ Docker Registry của GitHub.
* **`${{ github.actor }}`:** Tên đăng nhập (username) của người đã kích hoạt luồng workflow.
* **`${{ secrets.GITHUB_TOKEN }}`:** Token đăng nhập tạm thời do hệ thống tự động sinh ra, có hiệu lực ngay trong lúc job hoàn thành.
* **`ghcr.io/${{ github.repository }}`:** Đường dẫn lưu trữ image tự động nhận diện theo tên kho chứa của dự án (dạng `ghcr.io/username/project-name`).

#### 3.2 Lệnh xác thực và kéo image trong Script
Trong script của job, Runner thực thi các câu lệnh sau để chuẩn bị môi trường chạy:
```bash
# Đăng nhập tự động bằng tài khoản tạm thời
docker login ghcr.io -u ${{ github.actor }} -p ${{ secrets.GITHUB_TOKEN }}

# Kéo image từ registry dự án về Runner
docker pull ghcr.io/${{ github.repository }}:<version_tag>
```

#### 3.3 Cơ chế Ẩn biến bảo mật (Masking Variables) của GitHub
Giá trị thực của các biến bảo mật hệ thống như `${{ secrets.GITHUB_TOKEN }}` được GitHub tự động kiểm duyệt và ẩn đi. Nếu có bất kỳ dòng log nào cố tình in giá trị này ra, hệ thống sẽ tự động chuyển đổi thành chuỗi `***` trên log console để đảm bảo an toàn tuyệt đối.

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (CẤU HÌNH PULL VÀ RUN IMAGE TRONG PIPELINE)

Học viên tiến hành cập nhật file cấu hình `.github/workflows/ci.yml` của dự án `user-service` để kéo image phiên bản `1.0.0` (đã đẩy lên Registry từ local ở Lesson 03) về chạy verify trong luồng CI/CD.

#### 4.1 Biên soạn tệp cấu hình Workflow `.github/workflows/ci.yml`
Cập nhật file cấu hình `ci.yml` tại dự án `user-service`:

```yaml
name: Kiểm thử Docker Image từ Registry

on:
  push:
    branches:
      - main

jobs:
  test_image:
    runs-on: [self-hosted, quickbite]
    # Yêu cầu quyền truy cập packages cho GITHUB_TOKEN
    permissions:
      packages: read
      contents: read
    steps:
      - name: Lấy mã nguồn
        uses: actions/checkout@v5

      - name: Kiểm thử Image
        run: |
          # 1. Đăng nhập tự động vào Registry sử dụng tài khoản tạm thời của job
          docker login ghcr.io -u ${{ github.actor }} -p ${{ secrets.GITHUB_TOKEN }}

          # 2. Kéo trực tiếp image phiên bản 1.0.0 đã được push từ local về Runner
          # Lưu ý: GitHub tự động chuyển đổi repository name thành chữ thường để phù hợp với Docker
          IMAGE_TAG=$(echo "ghcr.io/${{ github.repository }}:1.0.0" | tr '[:upper:]' '[:lower:]')
          docker pull $IMAGE_TAG

          # 3. Khởi chạy thử container từ image vừa kéo về để verify ứng dụng hoạt động ổn định
          docker run -d --name verify_user_service -p 8081:8081 $IMAGE_TAG

          # 4. Chờ 5 giây để Spring Boot hoàn tất quá trình khởi động
          sleep 5

          # 5. Kiểm tra danh sách container để xác nhận ứng dụng đang chạy ở trạng thái UP (exit code = 0)
          docker ps

          # 6. Dọn dẹp môi trường (dừng và xóa container verify) trực tiếp trong script chính
          docker rm -f verify_user_service
```

> [!IMPORTANT]
> **Tránh dọn dẹp môi trường ở một bước độc lập mà không có cơ chế bắt lỗi:**
> Trong GitHub Actions, nếu một step thất bại, các step sau mặc định sẽ bị bỏ qua. Nếu bạn muốn step dọn dẹp `docker rm -f` luôn chạy dù test thành công hay thất bại, hãy cấu hình `if: always()` ở step dọn dẹp hoặc gộp chung vào 1 đoạn `run` duy nhất như ví dụ trên (mặc dù thực tế nếu lệnh `docker ps` thất bại thì lệnh sau cũng không chạy, cần thêm `|| true`).

#### 4.2 Kết quả mong đợi
Khi push file cấu hình lên GitHub, job `test_image` sẽ được thực thi. Mở log console, chúng ta sẽ thấy Runner đăng nhập thành công, kéo image về cực kỳ nhanh, chạy container verify ổn định và kết thúc job thành công. Tổng thời gian chạy (runtime) của job sẽ hiển thị cực kỳ ngắn (thường chỉ khoảng 10-15 giây).

---

### PHẦN 5. LƯU Ý, LỖI SAI VÀ HIỂU LẦM THƯỜNG GẶP (DÀNH CHO HỌC VIÊN)

* **Quên cấp quyền (permissions) cho GITHUB_TOKEN:** Lỗi `denied: unauthenticated` khi cố gắng pull từ GHCR trong luồng workflow.
  *Khắc phục:* Phải khai báo `permissions: packages: read` để token tạm thời có quyền kéo image từ registry.
* **Lỗi tên kho chứa có chữ viết hoa (invalid reference format):** Biến `${{ github.repository }}` giữ nguyên chữ viết hoa nếu tên kho chứa của bạn có viết hoa (ví dụ: `Nathan/My-Project`). Tuy nhiên, Docker image không cho phép sử dụng ký tự viết hoa.
  *Khắc phục:* Sử dụng lệnh shell `tr '[:upper:]' '[:lower:]'` để chuyển đổi toàn bộ thành chữ thường trước khi sử dụng với câu lệnh docker.
* **Lỗi không tìm thấy tag image (`manifest for ... not found`):** Xảy ra khi tag phiên bản khai báo trong file cấu hình CI/CD (ví dụ: `1.0.0`) không khớp với tag image thực tế đã được push lên Registry ở local trước đó.
  *Khắc phục:* Đảm bảo tag image được pull khớp chính xác với tag đã đẩy lên registry ở bài thực hành trước.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG

1. **Xác thực với GitHub Container Registry:**
   * [Authenticating to the Container registry | GitHub Documentation](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry#authenticating-to-the-container-registry)
2. **Sử dụng GITHUB_TOKEN trong workflow:**
   * [Automatic token authentication | GitHub Documentation](https://docs.github.com/en/actions/security-guides/automatic-token-authentication)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao thời gian chạy (runtime) của Runner ở bài học này lại rút ngắn đáng kể (từ 5 phút xuống còn khoảng 10 giây) so với việc build image trực tiếp ở Lesson 01 và Lesson 02?
* *Gợi ý:* Vì Runner không phải thực hiện các tác vụ biên dịch mã nguồn Java nặng nề và tải hàng trăm MB dependencies từ Maven Central nữa. Lệnh `docker pull` chỉ làm nhiệm vụ tải trực tiếp các layer của image đã được đóng gói sẵn từ Registry về, giúp tiết kiệm tối đa thời gian biên dịch và tài nguyên mạng.

#### Câu 2 (Phân tích so sánh)
Mức độ bảo mật của biến `${{ secrets.GITHUB_TOKEN }}` được bảo vệ như thế nào trên hệ thống GitHub Actions so với việc lập trình viên tự định nghĩa biến mật khẩu cá nhân?
* *Gợi ý:* Biến `${{ secrets.GITHUB_TOKEN }}` được hệ thống sinh ra tự động dưới dạng token tạm thời và chỉ tồn tại trong vòng đời của job đang chạy; khi job kết thúc, token lập tức bị hủy bỏ. Đối với các biến tự định nghĩa tĩnh, chúng tồn tại vô hạn thời hạn và dễ bị lộ lọt nếu bị hacker khai thác.

#### Câu 3 (Cấu hình hệ thống)
Nếu ứng dụng Spring Boot cần cấu hình thêm thông số kết nối database (ví dụ profile `dev`), học viên cần bổ sung cấu hình gì vào lệnh `docker run` trong script của file `.github/workflows/ci.yml`?
* *Gợi ý:* Học viên cần bổ sung tham số biến môi trường bằng cờ `-e` vào lệnh `docker run`, ví dụ:
  ```bash
  docker run -d --name verify_user_service -e SPRING_PROFILES_ACTIVE=dev -p 8081:8081 $IMAGE_TAG
  ```
