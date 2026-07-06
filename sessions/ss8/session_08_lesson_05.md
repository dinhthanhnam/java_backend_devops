# SESSION 08: ĐÓNG GÓI DOCKER IMAGE & ĐẨY LÊN REGISTRY

## LESSON 05: Kịch bản Thực hành Tổng hợp với user-service

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, học viên có khả năng:
* **Thực hành thành thạo** quy trình build, gắn tag phiên bản Docker image của dịch vụ `user-service` tại máy phát triển local.
* **Đăng nhập và đẩy thành công** image lên GitHub Container Registry (ghcr.io) cá nhân.
* **Biên soạn và kiểm thử** hoàn thiện tệp cấu hình `.github/workflows/ci.yml` để kiểm tra container tự động trong luồng CI/CD với thời gian chạy tối ưu.

---

### PHẦN 2. BỐI CẢNH THỰC HÀNH

Bài thực hành tổng hợp này giúp học viên củng cố các kỹ năng tương tác với GitHub Container Registry đã học ở các bài học trước:
* Đóng gói và gắn tag Docker image thủ công tại máy phát triển cá nhân để nắm vững cú pháp lệnh.
* Đẩy image lên Registry cá nhân bằng Access Token (PAT).
* Cấu hình luồng CI/CD để tự động kéo image đó về và khởi chạy verify trên môi trường Runner.

> [!WARNING]
> **Lưu ý quan trọng cho học viên:**
> Kịch bản build/push từ local và pull/verify trên CI/CD chỉ phục vụ mục đích học tập để chia nhỏ độ phức tạp của hệ thống. Trong môi trường doanh nghiệp thực tế, để đảm bảo tính an toàn thông tin và tính nhất quán của mã nguồn, toàn bộ quy trình từ build, push đến deploy bắt buộc phải chạy hoàn toàn tự động trên máy chủ CI/CD, không có sự can thiệp thủ công từ máy local của lập trình viên.

---

### PHẦN 3. KỊCH BẢN THỰC HÀNH CHI TIẾT (STEP-BY-STEP)

Học viên thực hiện đầy đủ các bước sau trực tiếp trên dự án `user-service` đã có sẵn trong workspace.

#### Bước 1: Viết và kiểm tra Dockerfile tối ưu tại Local
1. Tạo tệp tin `Dockerfile` tại thư mục gốc của dự án `user-service` (`user-service/Dockerfile`) sử dụng cấu trúc Multi-stage build đã học ở Lesson 02:
```dockerfile
# Stage 1: Build stage
FROM eclipse-temurin:17-jdk-alpine AS builder
WORKDIR /app

COPY gradlew .
COPY gradle gradle
COPY build.gradle .

# Cấp quyền thực thi và tiến hành biên dịch JAR (sử dụng --no-daemon để tránh treo máy trong container)
RUN chmod +x ./gradlew 

RUN ./gradlew dependencies --no-daemon

COPY src src

RUN ./gradlew bootJar --no-daemon

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
1. Truy cập giao diện Web của GitHub cá nhân -> **Settings** -> **Developer settings** -> **Personal access tokens** -> **Tokens (classic)**.
2. Tạo một Personal Access Token mới đặt tên là `registry-token`, tích chọn scopes: `write:packages` và `read:packages`. Sao chép mã token bắt đầu bằng `ghp_`.
3. Sử dụng terminal tại máy local, chạy lệnh đăng nhập vào Registry của GitHub:
```bash
# 1. Đăng nhập Docker CLI bằng Personal Access Token (PAT) (Dùng username cá nhân của bạn, kể cả repo thuộc org)
docker login ghcr.io -u <github_username_cua_ban>
```
   *Khi hệ thống yêu cầu nhập Password, dán mã Access Token vừa sao chép vào và nhấn Enter.*

#### Bước 3: Gắn tag Registry và đẩy Image lên GitHub
1. Thực hiện gắn tag image khớp chính xác với đường dẫn Registry của dự án (thay thế `<repository_namespace>` bằng username cá nhân hoặc tên Organization sở hữu GitHub):
```bash
docker tag user-service:1.0.0 ghcr.io/<repository_namespace>/user-service:1.0.0
```
2. Thực hiện đẩy image lên kho lưu trữ trực tuyến:
```bash
docker push ghcr.io/<repository_namespace>/user-service:1.0.0
```
3. Truy cập vào trang Profile trên GitHub -> tab **Packages** để xác nhận image `user-service` đã hiển thị tag `1.0.0`.

#### Bước 4: Cấu hình và chạy thử Workflow CI/CD
1. Tạo cấu trúc thư mục `.github/workflows/` và cập nhật nội dung file `ci.yml` tại thư mục gốc của dự án `user-service` để kéo image về và chạy verify tự động:
```yaml
name: Tích hợp Docker Registry

on:
  push:
    branches:
      - main

jobs:
  test_image:
    runs-on: [self-hosted, quickbite]
    permissions:
      packages: read
      contents: read
    steps:
      - name: Kiểm tra Container
        run: |
          # Đăng nhập tự động sử dụng thông tin định sẵn của Runner
          docker login ghcr.io -u ${{ github.actor }} -p ${{ secrets.GITHUB_TOKEN }}
          
          # Kéo image 1.0.0 đã được học viên push từ local về
          # Chuyển đổi tên kho chứa thành chữ thường
          IMAGE_TAG=$(echo "ghcr.io/${{ github.repository }}:1.0.0" | tr '[:upper:]' '[:lower:]')
          docker pull $IMAGE_TAG
          
          # Chạy container để kiểm tra verify ứng dụng
          docker run -d --name verify_app -p 8081:8081 $IMAGE_TAG
          
          # Chờ ứng dụng Spring Boot khởi động
          sleep 5
          
          # Kiểm tra danh sách container hoạt động
          docker ps
          
          # Dọn dẹp môi trường Runner sau khi kiểm tra xong
          docker rm -f verify_app || true
```
2. Thực hiện push file cấu hình lên GitHub để kích hoạt pipeline tự động.
3. Theo dõi log console của job trên tab **Actions** của GitHub, kiểm tra xem job có hoàn thành thành công và xác nhận tổng thời gian chạy (runtime) cực kỳ nhanh (thường dưới 30 giây).

---

### PHẦN 4. YÊU CẦU BÁO CÁO MINH CHỨNG (DÀNH CHO HỌC VIÊN)

Học viên sau khi hoàn thành thực hành cần chụp và đính kèm 3 hình ảnh minh chứng sau vào bài báo cáo:
1. **Minh chứng 1:** Ảnh chụp màn hình Terminal local hiển thị quá trình build và đẩy (`docker push`) image thành công lên registry.
2. **Minh chứng 2:** Ảnh chụp giao diện tab **Packages** trên giao diện GitHub Web hiển thị rõ tag `1.0.0` của image `user-service`.
3. **Minh chứng 3:** Ảnh chụp giao diện log thực thi của GitHub Actions hiển thị log thành công của job `test_image` đi kèm thông số thời gian chạy (runtime) tối ưu (dưới 30 giây).

---

### PHẦN 5. LƯU Ý LỖI VÀ HIỂU LẦM THƯỜNG GẶP (DÀNH CHO HỌC VIÊN)

* **Pipeline báo lỗi không tìm thấy image (Image not found):** Xảy ra khi sinh viên push cấu hình file `ci.yml` lên GitHub kích hoạt workflow trước khi chạy các lệnh build và push image từ local lên Registry. Do Registry trống rỗng, lệnh `docker pull` của Runner sẽ thất bại ngay lập tức.
  *Khắc phục:* Bắt buộc phải hoàn tất việc build và push image từ local thành công lên Registry trước khi đẩy file workflow CI/CD.
* **Lỗi `denied: unauthenticated` khi chạy workflow:** Xảy ra do học viên quên bổ sung phần phân quyền `permissions: packages: read` cho luồng làm việc khiến token mặc định không được truy cập Registry.
  *Khắc phục:* Kiểm tra file YAML và bổ sung permission.
* **Ứng dụng bị sập âm thầm (Silent Crash):** Lệnh `docker run` chạy thành công và tạo container, nhưng ứng dụng Spring Boot bên trong bị crash ngay lập tức do thiếu biến cấu hình hoặc xung đột cổng. Lệnh `docker ps` chạy sau đó sẽ không hiển thị container hoạt động, nhưng script shell có thể không báo đỏ do lệnh kết thúc bằng `docker ps` thành công.
  *Khắc phục:* Kiểm tra log của container cục bộ tại local bằng lệnh `docker logs <container_name>` để debug lỗi trước khi đẩy lên registry.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG

1. **Lệnh chạy Container của Docker CLI:**
   * [docker run command reference | Docker Documentation](https://docs.docker.com/engine/reference/commandline/run/)
2. **Kiểm tra Logs Workflow của GitHub Actions:**
   * [Using workflow run logs | GitHub Documentation](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/using-workflow-run-logs)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao trong kịch bản thực hành tổng hợp này, việc dọn dẹp container (`docker rm -f verify_app`) lại được thực hiện chung một khối `run` với các lệnh khác thay vì tách thành một `step` dọn dẹp độc lập ở cuối?
* *Gợi ý:* Vì trong GitHub Actions mặc định nếu một lệnh `run` hoặc `step` gặp lỗi (exit code khác 0), các lệnh phía sau sẽ bị hủy bỏ (skip). Để chạy dọn dẹp dù cho test fail hay pass, chúng ta có thể nối lệnh dọn dẹp bằng `|| true` hoặc tách thành step riêng nhưng phải khai báo thuộc tính `if: always()`. Việc gộp lại giúp code ngắn gọn và dễ theo dõi đối với mục đích học thuật.

#### Câu 2 (Phân tích so sánh)
Nếu thay đổi phiên bản code của ứng dụng và muốn chạy thử nghiệm bản build mới, học viên cần thực hiện lại các bước nào trong kịch bản thực hành trên?
* *Gợi ý:* Cần thực hiện lại: (1) Biên dịch JAR và build image tại local với tag phiên bản mới (ví dụ `1.0.1`), (2) Gắn tag Registry tương ứng `1.0.1` và push lên GHCR, (3) Cập nhật file `.github/workflows/ci.yml` sửa tag từ `1.0.0` thành `1.0.1` và push file cấu hình lên GitHub để chạy test.

#### Câu 3 (Cấu hình hệ thống)
Tại sao lệnh `docker run` trong script của job sử dụng tham số `-d` và `-p 8081:8081`?
* *Gợi ý:* Tham số `-d` (detached mode) giúp container chạy ngầm dưới nền, giải phóng shell để Runner có thể tiếp tục thực thi các dòng lệnh tiếp theo (`sleep`, `docker ps`). Tham số `-p 8081:8081` ánh xạ cổng mạng để ứng dụng container có thể giao tiếp trên cổng 8081 của Runner phục vụ việc verify.
