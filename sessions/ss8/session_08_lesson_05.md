# SESSION 08: ĐÓNG GÓI DOCKER IMAGE & ĐẨY LÊN REGISTRY

## LESSON 05: Kịch bản Thực hành Tổng hợp với user-service

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, học viên có khả năng:
* **Thực hành thành thạo** quy trình build, gắn tag phiên bản Docker image của dịch vụ `user-service` tại máy phát triển local.
* **Đăng nhập và đẩy thành công** image lên GitLab Container Registry cá nhân.
* **Biên soạn và kiểm thử** hoàn thiện tệp cấu hình `.gitlab-ci.yml` để kiểm tra container tự động trong pipeline CI/CD với thời gian chạy tối ưu.

---

### PHẦN 2. BỐI CẢNH THỰC HÀNH

Bài thực hành này giả lập quy trình làm việc thực tế tại doanh nghiệp:
* Lập trình viên hoàn thành tính năng, đóng gói và tối ưu hóa image trực tiếp tại máy phát triển cá nhân để tận dụng cache local cực nhanh.
* Image sau khi build được đẩy lên GitLab Container Registry để làm kho lưu trữ tập trung.
* Đường ống CI/CD của GitLab được cấu hình tối giản, chỉ làm nhiệm vụ kéo image về, chạy thử nghiệm verify để đảm bảo tính đúng đắn trước khi chuyển sang các bước triển khai tiếp theo.

---

### PHẦN 3. KỊCH BẢN THỰC HÀNH CHI TIẾT (STEP-BY-STEP)

Học viên thực hiện đầy đủ các bước sau trực tiếp trên dự án `user-service` đã có sẵn trong workspace.

#### Bước 1: Viết và kiểm tra Dockerfile tối ưu tại Local
1. Tạo tệp tin `Dockerfile` tại thư mục gốc của dự án `user-service` (`user-service/Dockerfile`) sử dụng cấu trúc Multi-stage build đã học ở Lesson 02:
   ```dockerfile
   # Stage 1: Build stage
   FROM eclipse-temurin:17-jdk-alpine AS builder
   WORKDIR /app
   # Sao chép toàn bộ mã nguồn vào container
   COPY . .
   RUN chmod +x ./gradlew && ./gradlew bootJar --no-daemon

   # Stage 2: Runtime stage
   FROM eclipse-temurin:17-jre-alpine
   WORKDIR /app
   COPY --from=builder /app/build/libs/*.jar app.jar
   EXPOSE 8081
   ENTRYPOINT ["java", "-jar", "app.jar"]
   ```
2. Chạy thử câu lệnh build image tại local terminal để đảm bảo Dockerfile hoạt động không có lỗi:
   ```bash
   docker build -t user-service:1.0.0 .
   ```

#### Bước 2: Khởi tạo Access Token và đăng nhập Registry
1. Truy cập giao diện Web của GitLab cá nhân -> **Preferences** -> **Access Tokens**.
2. Tạo một Personal Access Token mới đặt tên là `registry-token`, chọn thời hạn thích hợp và tích chọn 2 scopes: `write_registry` và `read_registry`. Sao chép mã token được sinh ra.
3. Sử dụng terminal tại máy local, chạy lệnh đăng nhập vào Registry của GitLab:
   ```bash
   docker login registry.gitlab.com -u <gitlab_username>
   ```
   *Khi hệ thống yêu cầu nhập Password, dán mã Access Token vừa sao chép vào và nhấn Enter.*

#### Bước 3: Gắn tag Registry và đẩy Image lên GitLab
1. Thực hiện gắn tag image khớp chính xác với đường dẫn Registry của dự án cá nhân (thay thế `<username>` bằng username GitLab thật của học viên):
   ```bash
   docker tag user-service:1.0.0 registry.gitlab.com/<username>/java_backend_devops/user-service:1.0.0
   ```
2. Thực hiện đẩy image lên kho lưu trữ trực tuyến:
   ```bash
   docker push registry.gitlab.com/<username>/java_backend_devops/user-service:1.0.0
   ```
3. Truy cập vào mục **Deploy** -> **Container Registry** trên GitLab Web UI của dự án để xác nhận image đã hiển thị tag `1.0.0`.

#### Bước 4: Cấu hình và chạy thử Pipeline CI/CD
1. Cập nhật nội dung file `.gitlab-ci.yml` tại thư mục gốc của dự án `user-service` để kéo image về và chạy verify tự động:
   ```yaml
   image: docker:latest

   stages:
     - test_image

   test_docker_image_job:
     stage: test_image
     tags:
       - quickbite
     script:
       # Đăng nhập tự động sử dụng thông tin định sẵn của Runner
       - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
       
       # Kéo image 1.0.0 đã được học viên push từ local về
       - docker pull $CI_REGISTRY_IMAGE:1.0.0
       
       # Chạy container để kiểm tra verify ứng dụng
       - docker run -d --name verify_app -p 8081:8081 $CI_REGISTRY_IMAGE:1.0.0
       
       # Chờ ứng dụng Spring Boot khởi động
       - sleep 5
       
       # Kiểm tra danh sách container hoạt động
       - docker ps
       
       # Dọn dẹp môi trường Runner sau khi kiểm tra xong
       - docker rm -f verify_app
   ```
2. Thực hiện push file cấu hình lên GitLab để kích hoạt pipeline tự động.
3. Theo dõi log console của job trên GitLab Web UI, kiểm tra xem job có hoàn thành thành công và xác nhận tổng thời gian chạy (runtime) cực kỳ nhanh (thường dưới 30 giây).

---

### PHẦN 4. YÊU CẦU BÁO CÁO MINH CHỨNG (DÀNH CHO HỌC VIÊN)

Học viên sau khi hoàn thành thực hành cần chụp và đính kèm 3 hình ảnh minh chứng sau vào bài báo cáo:
1. **Minh chứng 1:** Ảnh chụp màn hình Terminal local hiển thị quá trình build và đẩy (`docker push`) image thành công lên registry.
2. **Minh chứng 2:** Ảnh chụp giao diện **Container Registry** trên giao diện GitLab Web UI hiển thị rõ tag `1.0.0` của image `user-service`.
3. **Minh chứng 3:** Ảnh chụp giao diện **Pipeline Log** trên GitLab hiển thị log thực thi thành công của job `test_docker_image_job` đi kèm thông số thời gian chạy (runtime) tối ưu (dưới 30 giây).

---

### PHẦN 5. LƯU Ý LỖI VÀ HIỂU LẦM THƯỜNG GẶP (DÀNH CHO HỌC VIÊN)

* **Pipeline báo lỗi không tìm thấy image (Image not found):** Xảy ra khi sinh viên push cấu hình file `.gitlab-ci.yml` lên GitLab kích hoạt pipeline trước khi chạy các lệnh build và push image từ local lên Registry. Do Registry trống rỗng, lệnh `docker pull` của Runner sẽ thất bại ngay lập tức.
  *Khắc phục:* Bắt buộc phải hoàn tất việc build và push image từ local thành công lên Registry trước khi chạy pipeline CI/CD.
* **Không cấu hình `.dockerignore` dẫn đến build context quá lớn:** Khi thực hiện lệnh `docker build` ở local với `COPY . .`, nếu học viên không cấu hình `.dockerignore`, Docker sẽ sao chép toàn bộ thư mục `build/` và `.gradle/` từ máy local vào container, làm chậm tiến trình build và có thể gây lỗi hoặc xung đột tệp tin biên dịch cục bộ.
  *Khắc phục:* Học viên bắt buộc phải tạo tệp tin `.dockerignore` tại thư mục gốc của dự án và khai báo loại bỏ `build/`, `.gradle/`, `.git/` trước khi chạy build.
* **Ứng dụng bị sập âm thầm (Silent Crash):** Lệnh `docker run` chạy thành công và tạo container, nhưng ứng dụng Spring Boot bên trong bị crash ngay lập tức do thiếu biến cấu hình hoặc xung đột cổng. Lệnh `docker ps` chạy sau đó sẽ không hiển thị container hoạt động.
  *Khắc phục:* Kiểm tra log của container cục bộ tại local bằng lệnh `docker logs <container_name>` để debug lỗi trước khi đẩy lên registry.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG

1. **Lệnh chạy Container của Docker CLI:**
   * [docker run command reference | Docker Documentation](https://docs.docker.com/engine/reference/commandline/run/)
2. **Tài liệu hướng dẫn Pipeline Troubleshooting của GitLab:**
   * [Troubleshooting GitLab CI/CD | GitLab Documentation](https://docs.gitlab.com/ee/ci/troubleshooting.html)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao trong kịch bản thực hành tổng hợp này, việc dọn dẹp container (`docker rm -f verify_app`) lại được thực hiện ở cuối khối `script` thay vì đưa vào `after_script` như thói quen thông thường?
* *Gợi ý:* Vì nếu đưa vào `after_script` (tiến trình shell độc lập), mã lỗi thoát của các lệnh dọn dẹp sẽ không được GitLab dùng để đánh giá trạng thái Passed/Failed của job. Việc viết trực tiếp vào `script` giúp Runner kiểm soát và phát hiện lỗi dọn dẹp chính xác hơn, bảo vệ tính toàn vẹn của pipeline.

#### Câu 2 (Phân tích so sánh)
Nếu thay đổi phiên bản code của ứng dụng và muốn chạy thử nghiệm bản build mới, học viên cần thực hiện lại các bước nào trong kịch bản thực hành trên?
* *Gợi ý:* Cần thực hiện lại: (1) Biên dịch JAR và build image tại local với tag phiên bản mới (ví dụ `1.0.1`), (2) Gắn tag Registry tương ứng `1.0.1` và push lên Registry, (3) Cập nhật file `.gitlab-ci.yml` sửa tag từ `1.0.0` thành `1.0.1` và push file cấu hình lên GitLab để chạy test.

#### Câu 3 (Cấu hình hệ thống)
Tại sao lệnh `docker run` trong script của job sử dụng tham số `-d` và `-p 8081:8081`?
* *Gợi ý:* Tham số `-d` (detached mode) giúp container chạy ngầm dưới nền, giải phóng shell để Runner có thể tiếp tục thực thi các dòng lệnh tiếp theo (`sleep`, `docker ps`). Tham số `-p 8081:8081` ánh xạ cổng mạng để ứng dụng container có thể giao tiếp trên cổng 8081 của Runner phục vụ việc verify.
