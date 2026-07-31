# PROMPT CHO GAMMA: ĐÓNG GÓI DOCKER IMAGE & ĐẨY LÊN REGISTRY (SESSION 8)

## 1. CONTEXT & ROLE
* **Role:** DevOps Architect kiêm Giảng viên cao cấp. Giọng điệu chuyên nghiệp, trực diện, đi thẳng vào bản chất kỹ thuật và thực tiễn vận hành hệ thống.
* **Target Audience:** Kỹ sư phần mềm, học viên đang phát triển dự án Spring Boot Microservices (hệ thống QuickBite).
* **Objective:** Giải thích quy trình tự động hóa đóng gói Docker image trong luồng CI/CD, kỹ thuật tối ưu hóa Multi-stage build, quản lý phiên bản và vận hành GitHub Container Registry từ local đến môi trường tự động hóa CI/CD.

---

## 2. PARTITION INSTRUCTIONS
* **Số lượng slide khuyến nghị:** 15 - 20 slides.
* **Nguyên tắc phân bổ nội dung:**
  * **LESSON 01 (Mở đầu & Nền tảng):** Chuyển đổi từ đóng gói thủ công sang tự động hóa CI/CD. Phân tích mô hình DinD vs DooD.
  * **LESSON 02 (Tối ưu hóa):** Phân tích kỹ thuật Multi-stage build, tối ưu hóa kích thước image và vấn đề cache dependencies của Gradle.
  * **LESSON 03 (Đẩy Image từ Local):** Quy trình tương tác thủ công với GitHub Container Registry từ local terminal để hiểu bản chất xác thực và gắn tag.
  * **LESSON 04 (Kéo & Chạy trong CI/CD):** Tự động hóa kéo image từ Registry về Runner sử dụng Predefined Variables và verify ứng dụng.
  * **LESSON 05 (Thực hành tổng hợp):** Kịch bản thực hành từng bước cho dịch vụ `user-service`.
  * **Tính chuyên nghiệp:** Loại bỏ hoàn toàn các từ ngữ suồng sã, giật gân. Tập trung giải thích thuật ngữ kỹ thuật chính xác.

---

## 3. HIERARCHICAL OUTLINE (DÀN Ý CHI TIẾT)

### LESSON 01: Quy trình Build Docker image trong luồng CI/CD

#### Slide 1: Chuyển dịch từ Build JAR sang tự động đóng gói Image trên CI
* **Sự kế thừa từ bài học trước:**
  * Ở Session 7, chúng ta đã cấu hình thành công quy trình tự động biên dịch ứng dụng Spring Boot ra file JAR trên hệ thống CI.
* **Đặt vấn đề chuyển đổi:**
  * Khi file JAR thành phẩm đã được tự động biên dịch sạch trên CI Server, việc tiếp tục đóng gói Docker Image ở local một cách thủ công sẽ gây ngắt quãng quy trình.
  * Tận dụng hạ tầng CI để tự động hóa khâu đóng gói Docker Image từ chính file JAR vừa biên dịch.
* **Lợi ích đạt được:**
  * Triệt tiêu hoàn toàn thao tác gõ lệnh `docker build` thủ công của lập trình viên.
  * Đảm bảo tính đồng nhất tuyệt đối của Docker Image dựa trên bản build sạch từ CI.

#### Slide 2: Kiến trúc Workflow 2 Job cơ bản
* Quy trình chuyển tiếp mã nguồn sang sản phẩm đóng gói:
```text
[ Git Push ] ──► [ Job: build_jar ] ──(Lưu JAR artifacts)──► [ Job: build_image ]
                                                                       │ (docker build)
                                                                       ▼
                                                              [ Single-stage Image ]
```
* **Job 1: `build_jar`**
  * Sử dụng JDK 17 cài đặt thông qua action.
  * Thực thi biên dịch mã nguồn Java ra file JAR và lưu lại bằng `actions/upload-artifact`.
* **Job 2: `build_image`**
  * Chạy tuần tự sau Job 1 (thông qua `needs: [build_jar]`).
  * Tải file JAR artifacts từ Job 1 về bằng `actions/download-artifact` và thực hiện lệnh `docker build`.

#### Slide 3: Giải pháp chạy Docker trong CI/CD: DinD vs DooD (Docker Socket Sharing)
* Để chạy được lệnh `docker build` bên trong Runner, có hai mô hình kiến trúc chính:
* **Docker-in-Docker (DinD):**
  * Khởi tạo container phụ Docker Daemon bên trong Runner.
  * *Nhược điểm:* Tốn thời gian khởi tạo, yêu cầu quyền privileged, không tận dụng được cache image của máy host.
* **Docker-outside-of-Docker (DooD) / Chia sẻ Docker Socket:**
  * Chia sẻ trực tiếp Docker socket vật lý của máy host cho Runner.
  * *Lợi ích:* Chuyển tiếp lệnh thực thi trực tiếp tới Docker daemon của máy host, tận dụng bộ nhớ cache layer có sẵn để tối ưu hóa tốc độ build, loại bỏ khởi tạo dịch vụ phụ.

#### Slide 4: Thực hành cấu hình Workflow 2 Job mẫu
* Cấu hình tệp tin `.github/workflows/ci.yml` sử dụng Local Runner và DooD:
```yaml
name: Build Image 2 Job
on: [push]
jobs:
  build_jar:
    runs-on: [self-hosted, quickbite]
    steps:
      - uses: actions/checkout@v5
      - run: chmod +x ./gradlew && ./gradlew bootJar
      - uses: actions/upload-artifact@v4
        with:
          name: app-jar
          path: build/libs/*.jar
  build_image:
    runs-on: [self-hosted, quickbite]
    needs: [build_jar]
    steps:
      - uses: actions/checkout@v5
      - uses: actions/download-artifact@v4
        with:
          name: app-jar
          path: build/libs/
      - run: docker build -t user-service:latest .
```

---

### LESSON 02: Tối ưu hóa Dockerfile cho Production (Multi-stage build)

#### Slide 5: Điểm nghẽn của Workflow 2 Job truyền thống
* Mặc dù giải quyết được vấn đề tự động hóa, quy trình 2 Job bộc lộ 3 nhược điểm lớn:
  1. *Phụ thuộc băng thông:* Phải truyền tải file JAR nặng nề làm artifacts qua lại giữa lưu trữ của GitHub và Runner.
  2. *Quy trình build không khép kín (Non-self-contained):* Dockerfile đơn tầng không tự biên dịch được. Lập trình viên không thể chạy lệnh `docker build` trực tiếp ở local nếu thiếu file JAR biên dịch sẵn.
  3. *Xung đột phiên bản:* Rủi ro lệch phiên bản JRE/JDK giữa lúc compile trên host và lúc chạy trong container.
* **Giải pháp:** Đưa toàn bộ bước biên dịch vào bên trong Dockerfile thông qua kỹ thuật **Multi-stage build**.

#### Slide 6: Khái niệm và Sơ đồ luồng Multi-stage Build
* Sử dụng nhiều chỉ thị `FROM` trong cùng một Dockerfile để chia quy trình thành các giai đoạn (stages) độc lập:
```text
[ Dockerfile Build ]
     │
     ├─► [ Stage 1: AS builder ] ──► (Tải Gradle, JDK, biên dịch JAR)
     │                                         │
     │                                 COPY --from=builder
     │                                         ▼
     └─► [ Stage 2: AS runner ]  ──► (Nhận JAR, dùng JRE Alpine) ──► [ Production Image ]
```
* **Stage 1 (Builder stage):** Sử dụng base image JDK đầy đủ để biên dịch mã nguồn.
* **Stage 2 (Runtime stage):** Sử dụng base image JRE siêu gọn nhẹ, sao chép file JAR thương phẩm từ Stage 1 sang (`COPY --from=builder`).
* *Kết quả:* Loại bỏ toàn bộ công cụ compile nặng nề và mã nguồn thô Java khỏi image cuối cùng, tối ưu dung lượng và bảo mật.

#### Slide 7: Thực hành viết Dockerfile Multi-stage & Tinh gọn Workflow
* Cấu hình tệp `Dockerfile` của dịch vụ `user-service`:
```dockerfile
FROM eclipse-temurin:17-jdk-alpine AS builder
WORKDIR /app
COPY . .
RUN chmod +x ./gradlew && ./gradlew bootJar --no-daemon

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /app/build/libs/*.jar app.jar
EXPOSE 8081
ENTRYPOINT ["java", "-jar", "app.jar"]
```
* Tệp `.github/workflows/ci.yml` được tinh gọn chỉ còn 1 Job:
```yaml
name: Multi-stage Build
on: [push]
jobs:
  build_image:
    runs-on: [self-hosted, quickbite]
    steps:
      - uses: actions/checkout@v5
      - run: docker build -t user-service:latest .
```

#### Slide 8: Pain Point mới - Vấn đề Cache Dependencies trong CI/CD
* **Nguyên nhân:**
  * Do tiến trình biên dịch diễn ra bên trong container builder tạm thời của lệnh `docker build`, Gradle không thể truy cập thư mục `.gradle` của máy host.
  * Mỗi lần chạy workflow, Gradle bắt buộc phải tải lại toàn bộ thư viện dependencies từ Maven Central.
* **Hệ quả:**
  * Kéo dài thời gian chạy (runtime) của job lên tới **4-5 phút**.
  * Tiêu tốn băng thông mạng và tài nguyên hệ thống đáng kể.
* *Định hướng:* Có thể tối ưu hóa bước này bằng cách tận dụng cache layer của Docker (sẽ học ở các chuyên đề nâng cao về cache).

---

### LESSON 03: Phiên bản hóa và Đẩy Docker Image lên Registry từ Local

#### Slide 9: Kiến trúc GitHub Container Registry
* **Định nghĩa:** GitHub Container Registry (GHCR) là kho lưu trữ Docker image nằm trong GitHub Packages đi kèm dự án.
* **Cấu trúc đường dẫn Registry:**
  `ghcr.io/<repository_namespace>/<project_name>:<version_tag>`
* **Quản trị trực quan:** Kiểm tra danh sách image đã đẩy lên thông qua tab **Packages** trên Profile GitHub.

#### Slide 10: Quy tắc đặt nhãn (Tagging) và Phiên bản hóa (Versioning)
* **Tầm quan trọng của Version Tag:**
  * Tránh sử dụng nhãn tĩnh `latest` cho môi trường Production vì dễ gây ghi đè ngoài ý muốn và không thể rollback khi lỗi.
  * Áp dụng quy tắc đánh phiên bản Semantic Versioning (ví dụ: `1.0.0`, `1.1.0`).
* **Lệnh gắn tag của Docker CLI:**
  ```bash
  docker tag <local_image> ghcr.io/<repository_namespace>/<project_name>:<version_tag>
  ```
  Lệnh này tạo ra một alias (bí danh) trỏ tới image nguồn, phục vụ cho việc định tuyến đẩy lên đúng kho chứa của dự án.

#### Slide 11: Xác thực bảo mật bằng Personal Access Token (PAT)
* Do kho lưu trữ Registry yêu cầu quyền truy cập, Docker CLI cần xác thực quyền trước khi ghi.
* **Nguyên tắc bảo mật:**
  * *Tuyệt đối không* sử dụng mật khẩu tài khoản chính của GitHub để đăng nhập.
  * Sử dụng **Personal Access Token (PAT)** được cấp quyền giới hạn: tích chọn quyền `write:packages` và `read:packages`.
* **Lệnh xác thực an toàn:**
  ```bash
  docker login ghcr.io -u <github_username_cua_ban>
  ```
  Nhập mã token PAT khi terminal yêu cầu cung cấp Password.

#### Slide 12: Quy trình Build & Push thủ công từ Local Terminal
* Chuỗi lệnh thực thi tại terminal máy phát triển cá nhân:
```bash
# 1. Đăng nhập hệ thống
docker login ghcr.io -u <username>
# 2. Biên dịch nhanh (tận dụng cache local)
./gradlew bootJar
# 3. Build Docker image
docker build -t user-service:1.0.0 .
# 4. Gắn nhãn Registry tương thích
docker tag user-service:1.0.0 ghcr.io/<username>/user-service:1.0.0
# 5. Đẩy image lên cloud
docker push ghcr.io/<username>/user-service:1.0.0
```
* **Lưu ý bảo mật:** Quy trình build và push từ máy local là một **anti-pattern** trong Production thực tế do dễ gây mất nhất quán mã nguồn. Đây chỉ là bài thực hành sư phạm để sinh viên nắm vững nguyên lý hoạt động.

---

### LESSON 04: Sử dụng Docker Image từ Registry trong Luồng CI/CD

#### Slide 13: Xác thực tự động trong CI/CD bằng Token hệ thống
* Khi chạy job trên Runner, không được khai báo cứng token cá nhân vào tệp cấu hình YAML. GitHub Actions tự động cấp phát tài khoản tạm thời thông qua các biến ngữ cảnh định sẵn:
  * `ghcr.io`: Địa chỉ máy chủ lưu trữ.
  * `${{ github.actor }}`: Tên đăng nhập người đã khởi kích luồng.
  * `${{ secrets.GITHUB_TOKEN }}`: Token xác thực tạm thời tự động sinh (tự hủy khi job kết thúc).
  * `${{ github.repository }}`: Tên repo, ví dụ `username/user-service`.

#### Slide 14: Thực hành cấu hình Workflow Kéo và Verify Image
* Cập nhật file `.github/workflows/ci.yml` (Nhớ bổ sung quyền packages read):
```yaml
name: Test Image
on: [push]
jobs:
  test_image:
    runs-on: [self-hosted, quickbite]
    permissions:
      packages: read
      contents: read
    steps:
      - run: |
          docker login ghcr.io -u ${{ github.actor }} -p ${{ secrets.GITHUB_TOKEN }}
          IMAGE_TAG=$(echo "ghcr.io/${{ github.repository }}:1.0.0" | tr '[:upper:]' '[:lower:]')
          docker pull $IMAGE_TAG
          docker run -d --name verify_service -p 8081:8081 $IMAGE_TAG
          sleep 5
          docker ps
          docker rm -f verify_service || true
```

#### Slide 15: Tối ưu hiệu năng và Bảo mật thông tin trong CI/CD
* **Tối ưu hóa hiệu năng vượt trội:**
  * Job verify chỉ thực hiện các tác vụ mạng `pull` và chạy thử container, không biên dịch hay build image.
  * Thời gian thực thi cực kỳ ngắn (chỉ mất khoảng **10 - 15 giây**).
* **Bảo mật thông tin đăng nhập:**
  * Giá trị của biến `${{ secrets.GITHUB_TOKEN }}` được GitHub tự động kiểm duyệt và ẩn đi dưới dạng chuỗi `***` trên log console để tránh rò rỉ thông tin đăng nhập.

---

### LESSON 05: Kịch bản Thực hành Tổng hợp với user-service

#### Slide 16: Tóm tắt Kịch bản Thực hành Tổng hợp
* Quy trình thực hành toàn diện gồm 4 bước chính:
```mermaid
graph TD
    A[Bước 1: Viết Multi-stage Dockerfile] --> B[Bước 2: Login GHCR bằng Access Token (PAT)]
    B --> C[Bước 3: Tag & Push Image 1.0.0 từ Local]
    C --> D[Bước 4: Cấu hình ci.yml kéo & verify image]
```
* **Mục tiêu thực hành:** Sinh viên tự tay cấu hình, thực hiện đẩy thành công image từ máy cá nhân lên GHCR dự án, và cấu hình workflow GitHub Actions kiểm thử tự động kéo về chạy trên Local Runner.

#### Slide 17: Các lỗi thường gặp và giải pháp xử lý nhanh
* **Lỗi `denied: unauthenticated` hoặc `manifest not found`:**
  * *Nguyên nhân:* Token không đủ quyền `packages: read` hoặc quên gắn đúng tag 1.0.0.
  * *Khắc phục:* Kiểm tra bổ sung block `permissions` trong luồng, kiểm tra lại tab Packages trên GitHub.
* **Quên chuyển tên repo về viết thường (`[:lower:]`):**
  * *Nguyên nhân:* Tên tài khoản hoặc tên repo có ký tự hoa, Docker báo lỗi invalid reference.
  * *Khắc phục:* Dùng bash `tr '[:upper:]' '[:lower:]'` để hạ xuống chữ thường.
* **Lạm dụng chạy container không background (thiếu `-d`):**
  * *Nguyên nhân:* Lệnh `docker run` chiếm luồng, gây treo Runner không hồi kết.
  * *Khắc phục:* Luôn thêm cờ `-d` để container chạy ngầm, kèm theo lệnh `sleep 5` và `docker rm -f`.
