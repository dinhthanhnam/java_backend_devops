# SESSION 08: ĐÓNG GÓI DOCKER IMAGE & ĐẨY LÊN REGISTRY

## LESSON 02: Tối ưu hóa Dockerfile cho Production (Multi-stage build)

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, học viên có khả năng:
* **Phân tích** được hạn chế của việc phân chia quy trình build thành nhiều stage độc lập trong workflow CI/CD và tính ưu việt của Multi-stage build.
* **Ứng dụng** kỹ thuật Multi-stage build để thực hiện biên dịch mã nguồn Java và đóng gói sản phẩm hoàn toàn bên trong Dockerfile.
* **Tinh giản** cấu hình workflow CI/CD về mức tối giản nhất.
* **Đánh giá** được tác động của việc thiếu cache dependencies đến thời gian chạy (runtime) của Runner.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (SỰ PHỨC TẠP CỦA WORKFLOW 2 JOB VÀ QUY TRÌNH BUILD KHÔNG KHÉP KÍN)

Ở bài học trước (State 1), chúng ta đã tự động hóa việc build Docker image bằng workflow CI/CD gồm 2 jobs độc lập (`build_jar` và `build_image`). Tuy nhiên, quy trình này bộc lộ một số hạn chế trong thực tế:
1. **Cấu hình workflow phức tạp và tốn tài nguyên:** Việc phân tách thành nhiều job yêu cầu cấu hình truyền tải file JAR thông qua Action `actions/upload-artifact@v4` ở job trước và `actions/download-artifact@v4` ở job sau. Tiến trình này tiêu tốn dung lượng lưu trữ tạm thời trên GitHub và tăng băng thông truyền tải mạng giữa Runner và server.
2. **Quy trình build không khép kín (Self-contained):** File `Dockerfile` đơn tầng không thể tự build độc lập nếu thiếu file JAR được biên dịch sẵn từ trước. Lập trình viên không thể chạy một lệnh `docker build` duy nhất tại máy local để đóng gói hoàn chỉnh ứng dụng từ mã nguồn thô mà bắt buộc phải chạy lệnh biên dịch `./gradlew bootJar` trên môi trường phát triển trước.
3. **Rủi ro sai lệch phiên bản môi trường cục bộ:** Nếu lập trình viên build JAR ở máy local bằng một phiên bản JDK khác (ví dụ JDK 21) rồi dùng Dockerfile đơn tầng đóng gói với base image JRE 17, ứng dụng có thể gặp lỗi runtime do không tương thích phiên bản bytecode.

Để chuẩn hóa quy trình build khép kín, độc lập với các công cụ cài đặt ngoài container, chúng ta cần đưa bước biên dịch mã nguồn vào bên trong chính Dockerfile thông qua kỹ thuật **Multi-stage build** (State 2: Multi-stage Dockerfile).

---

### PHẦN 3. KIẾN THỨC CỐT LÕI

#### 3.1 Khái niệm Multi-stage Build trong Dockerfile
Multi-stage build cho phép sử dụng nhiều câu lệnh `FROM` trong cùng một tệp tin Dockerfile để chia quy trình đóng gói thành nhiều giai đoạn (stages) có nhiệm vụ riêng biệt:
* **Stage 1 (Builder stage):** Sử dụng một base image JDK đầy đủ (như `eclipse-temurin:17-jdk-alpine AS builder`), sao chép toàn bộ mã nguồn Java vào và thực thi lệnh biên dịch `./gradlew bootJar --no-daemon`.
* **Stage 2 (Runtime stage):** Sử dụng một base image Java Runtime Environment (JRE) siêu gọn nhẹ (như `eclipse-temurin:17-jre-alpine`). Thực hiện sao chép file JAR thương phẩm từ Stage 1 sang (`COPY --from=builder`).
* **Kết quả:** Toàn bộ công cụ JDK nặng nề và mã nguồn thô Java sẽ bị bỏ lại ở Stage 1, giúp image cuối cùng có kích thước tối thiểu và có độ bảo mật cao nhất (không lộ mã nguồn thô).

```text
[ Dockerfile Build ]
     │
     ├─► [ Stage 1: AS builder ] ──► (Tải Gradle, JDK, biên dịch JAR)
     │                                         │
     │                                 COPY --from=builder
     │                                         ▼
     └─► [ Stage 2: AS runner ]  ──► (Nhận JAR, dùng JRE Alpine) ──► [ Production Image ]
```

#### 3.2 Tinh giản Workflow CI/CD tương ứng
Khi toàn bộ tiến trình build đã được Dockerfile quản lý, workflow CI/CD không cần thực hiện job biên dịch riêng biệt và truyền tải artifacts qua GitHub server nữa. File `.github/workflows/ci.yml` được tinh gọn tối đa: Chỉ cần khai báo duy nhất job `build_image` và chạy lệnh `docker build`.

#### 3.3 Pain Point mới: Vấn đề Runtime trong CI/CD
Mặc dù Multi-stage build chuẩn hóa môi trường cực kỳ tốt, nó lại tạo ra một vấn đề kỹ thuật nghiêm trọng trong CI/CD:
* Do tiến trình biên dịch mã nguồn diễn ra bên trong container builder tạm thời của lệnh `docker build`, Gradle **không thể tái sử dụng cache dependencies** từ thư mục `.gradle` của máy host do không được cấu hình mount volume cache.
* Khi chạy lệnh `./gradlew bootJar` bên trong Dockerfile, Gradle bắt buộc phải tải lại toàn bộ dependencies và plugins từ Maven Central.
* Việc này kéo dài thời gian chạy (runtime) của Runner lên tới **5 phút**, tiêu tốn băng thông mạng và gây lãng phí tài nguyên vận hành CI/CD cực kỳ lớn.

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (MULTI-STAGE BUILD TRONG DOCKERFILE)

Học viên sửa đổi cấu hình của dự án `user-service` để chuyển quy trình biên dịch vào bên trong Dockerfile và rút gọn file CI/CD.

#### 4.1 Biên soạn tệp tin `Dockerfile`
Cập nhật `user-service/Dockerfile` thành cấu trúc Multi-stage:
```dockerfile
# Stage 1: Giai đoạn biên dịch (Builder stage)
FROM eclipse-temurin:17-jdk-alpine AS builder
WORKDIR /app

# Sao chép toàn bộ mã nguồn vào container (kết hợp .dockerignore để loại bỏ thư mục build/ và .gradle/ local)
COPY . .

# Cấp quyền thực thi và tiến hành biên dịch JAR (sử dụng --no-daemon để tránh treo máy trong container)
RUN chmod +x ./gradlew && ./gradlew bootJar --no-daemon

# Stage 2: Giai đoạn chạy ứng dụng (Runtime stage)
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

# Sao chép file JAR đã biên dịch từ Stage 1 (builder) sang Stage 2
COPY --from=builder /app/build/libs/*.jar app.jar

# Khai báo cổng mạng ứng dụng và lệnh khởi chạy
EXPOSE 8081
ENTRYPOINT ["java", "-jar", "app.jar"]
```

#### 4.2 Biên soạn tệp cấu hình `.github/workflows/ci.yml`
Rút gọn file cấu hình `user-service/.github/workflows/ci.yml` chỉ còn 1 job (không cần DinD do dùng Local Runner gắn socket):
```yaml
name: Build Docker Image Multi-stage

on:
  push:
    branches:
      - main

jobs:
  build_image:
    runs-on: [self-hosted, quickbite]
    steps:
      - name: Lấy mã nguồn
        uses: actions/checkout@v5

      - name: Đóng gói Docker Image
        # Build Docker image trực tiếp, toàn bộ việc compile được thực hiện bên trong Dockerfile
        run: docker build -t user-service:latest .
```

> [!IMPORTANT]
> **Không kiểm tra trạng thái trong các bước sau run build:**
> Việc kiểm tra phải chạy trực tiếp trong khối script để nếu lệnh build sập, job CI/CD sẽ lập tức dừng lại và thông báo Failed.

#### 4.3 Kết quả mong đợi
Pipeline chạy thành công chỉ với duy nhất một job `build_image`. Tuy nhiên, thời gian chạy của job kéo dài khoảng vài phút do tiến trình tải dependencies của Gradle bên trong container diễn ra rất lâu.

---

### PHẦN 5. LƯU Ý, LỖI SAI VÀ HIỂU LẦM THƯỜNG GẶP (DÀNH CHO HỌC VIÊN)

* **Không cấu hình `.dockerignore` khi sử dụng `COPY . .`:** Khi dùng câu lệnh `COPY . .`, Docker sẽ sao chép toàn bộ thư mục dự án ở máy local vào container, bao gồm cả các thư mục chứa cache và tệp tin biên dịch tạm thời dưới máy local như `build/` và `.gradle/`. Việc này làm tăng dung lượng build context không cần thiết và có thể gây lỗi hoặc xung đột tệp tin biên dịch.
  *Khắc phục:* Học viên bắt buộc phải tạo tệp tin `.dockerignore` ở thư mục gốc của dự án và khai báo loại bỏ các thư mục này:
```text
build/
.gradle/
.git/
```
* **Thời gian build trong CI/CD quá lâu:** Học viên lo lắng khi thấy thời gian chạy workflow tăng vọt từ vài chục giây lên 4-5 phút. Đây là hành vi bình thường của cơ chế Multi-stage build khi chạy trong CI/CD. Dù đã chia sẻ Docker socket của máy host để thực thi lệnh build image nhanh hơn, tiến trình chạy Gradle bên trong container builder (`FROM eclipse-temurin:17-jdk-alpine AS builder`) hoàn toàn tách biệt và không thể truy cập thư mục cache Gradle (`~/.gradle`) của máy host, dẫn đến việc phải tải lại toàn bộ dependencies từ Maven Central ở mỗi lượt build.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG

1. **Hướng dẫn Multi-stage build của Docker:**
   * [Use multi-stage builds | Docker Documentation](https://docs.docker.com/build/building/multi-stage/)
2. **Quản lý Cache trong Gradle:**
   * [Gradle Build Environment Cache | Gradle Documentation](https://docs.gradle.org/current/userguide/build_environment.html)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao kích thước của Docker image cuối cùng được tạo ra bằng Multi-stage build (ở Lesson 02) lại nhỏ gọn và bảo mật hơn rất nhiều so với image chứa đầy đủ bộ cài đặt JDK và mã nguồn Java?
* *Gợi ý:* Vì Multi-stage build chỉ sao chép file JAR thương phẩm duy nhất từ stage biên dịch sang stage runtime JRE Alpine (dung lượng nhẹ dưới 100MB). Toàn bộ mã nguồn Java thô và bộ JDK nặng nề phục vụ biên dịch đều bị bỏ lại ở stage builder trước đó, giúp giảm dung lượng image và loại bỏ hoàn toàn mã nguồn thô khỏi môi trường production.

#### Câu 2 (Phân tích so sánh)
Tại sao thời gian build image bằng Multi-stage build trên máy phát triển local cá nhân thường nhanh hơn rất nhiều so với khi chạy trong job CI/CD của GitHub Actions Runner?
* *Gợi ý:* Vì máy local cá nhân của lập trình viên có lưu trữ sẵn bộ nhớ đệm (cache) Gradle dependencies ở thư mục `~/.gradle` của máy host và cache các layer của Docker. Trong khi đó, trên môi trường CI/CD của Runner, tiến trình Gradle biên dịch chạy bên trong container builder cô lập và không được ánh xạ thư mục cache Gradle của máy host, dẫn đến việc phải tải lại toàn bộ dependencies từ internet ở mỗi lần build.

#### Câu 3 (Cấu hình hệ thống)
Trong Dockerfile đa tầng trên, lệnh `COPY --from=builder /app/build/libs/*.jar app.jar` đóng vai trò gì?
* *Gợi ý:* Lệnh này giúp sao chép file JAR đã được build thành công từ thư mục làm việc `/app/build/libs/` của stage 1 (được đặt tên là `builder`) sang thư mục làm việc của stage 2, giúp tách biệt hoàn toàn môi trường biên dịch và chạy thực tế.
