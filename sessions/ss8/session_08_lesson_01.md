# SESSION 08: ĐÓNG GÓI DOCKER IMAGE & ĐẨY LÊN REGISTRY

## LESSON 01: Quy trình Build Docker image trong pipeline CI/CD

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, học viên có khả năng:
* **Giải thích** được quy trình tự động hóa đóng gói Docker image trong pipeline CI/CD từ file JAR được biên dịch sẵn.
* **Áp dụng** cơ chế truyền tải artifacts giữa các stage trong GitLab CI.
* **Phân biệt** được mô hình Docker-in-Docker (DinD) và Docker-outside-of-Docker (DooD) chia sẻ Docker socket.
* **Cấu hình** pipeline CI/CD tối giản giao tiếp trực tiếp với Docker Daemon của máy host.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (TỪ THỦ CÔNG SANG TỰ ĐỘNG)

Trong các session trước (State 0), lập trình viên thực hiện quy trình đóng gói hoàn toàn thủ công ở máy phát triển cá nhân:
1. Chạy lệnh `./gradlew bootJar` để biên dịch mã nguồn Java ra file JAR.
2. Viết Dockerfile đơn tầng (Single-stage Dockerfile) để sao chép file JAR từ thư mục local vào container base.
3. Chạy lệnh `docker build` để đóng gói thành Docker image hoàn thiện.

**Hạn chế của quy trình thủ công:**
* Dễ phát sinh sai sót do thao tác của con người.
* Môi trường build ở local không nhất quán (lệch phiên bản Java, hệ điều hành khác biệt).
* Không thể tích hợp vào luồng tích hợp liên tục (CI/CD) của dự án.

Để giải quyết vấn đề này, chúng ta cần tự động hóa toàn bộ quy trình trên trong pipeline CI/CD của GitLab (State 1: CI/CD Build với 2 Stage) sử dụng hệ thống **Local Runner** đã cấu hình ở các bài học trước.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI

#### 3.1 Quy trình Pipeline CI/CD 2 Stage
Để tự động hóa đóng gói, pipeline CI/CD được chia thành 2 giai đoạn (stages) chạy tuần tự:
1. **Stage 1 (`build_jar`):** Biên dịch mã nguồn Java thành file JAR thương mại sử dụng môi trường JDK thích hợp và lưu trữ kết quả đầu ra dưới dạng **artifacts**.
2. **Stage 2 (`build_image`):** Runner khởi chạy container Docker CLI, tải file JAR artifacts về thư mục workspace, và thực thi lệnh `docker build` dựa trên Dockerfile đơn tầng để đóng gói thành image.

```text
[ Git Push ] ──► [ Stage: build_jar ] ──(Lưu JAR artifacts)──► [ Stage: build_image ]
                                                                       │ (docker build)
                                                                       ▼
                                                              [ Single-stage Image ]
```

#### 3.2 Giải pháp chạy Docker trong CI/CD: DinD vs DooD (Docker Socket Sharing)
Để chạy các lệnh như `docker build` bên trong container của Runner, chúng ta có hai mô hình chính:

1. **Docker-in-Docker (DinD):** Chạy một Docker Daemon phụ độc lập bên trong container Runner thông qua từ khóa `services: - docker:dind`.
   * *Đặc điểm:* Thường dùng trên các Shared Runner online của GitLab (như trên gitlab.com).
   * *Nhược điểm:* Tốn thời gian khởi tạo container phụ, không tận dụng được cache image của máy host, yêu cầu quyền privileged.
2. **Docker-outside-of-Docker (DooD) / Chia sẻ Docker Socket (Lựa chọn của giáo trình):**
   * *Đặc điểm:* Mount trực tiếp Docker socket vật lý của máy host (`/var/run/docker.sock`) vào container của job thông qua cấu hình `volumes` của Runner (đã thiết lập khi dựng Local Runner ở Session 07).
   * *Vì sao không cần `docker:dind`?* Vì các lệnh Docker trong container của job sẽ được chuyển tiếp và thực thi trực tiếp trên Docker daemon của chính máy host.
   * *Lợi ích vượt trội:* Loại bỏ hoàn toàn khai báo `services: - docker:dind` giúp file YAML cực kỳ ngắn gọn; đồng thời tận dụng trực tiếp bộ nhớ cache image và layer có sẵn trên máy host để tăng tốc độ build tối đa.

#### 3.3 Dockerfile đơn tầng (Single-stage)
Dockerfile được sử dụng ở bài học này là Dockerfile đơn tầng chuẩn hóa (tương tự như Session 04), thực hiện sao chép file JAR đã có sẵn trong thư mục làm việc vào container:
```dockerfile
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY build/libs/*.jar app.jar
EXPOSE 8081
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (CẤU HÌNH PIPELINE CI/CD VỚI LOCAL RUNNER)

Học viên thực hiện cấu hình tệp tin `.gitlab-ci.yml` và `Dockerfile` tại thư mục gốc của dự án `user-service` để tự động hóa quy trình đóng gói.

#### 4.1 Biên soạn tệp tin `Dockerfile`
Tạo file `Dockerfile` tại thư mục gốc của dự án `user-service` (`user-service/Dockerfile`):
```dockerfile
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY build/libs/*.jar app.jar
EXPOSE 8081
ENTRYPOINT ["java", "-jar", "app.jar"]
```

#### 4.2 Biên soạn tệp cấu hình `.gitlab-ci.yml`
Cập nhật file `.gitlab-ci.yml` tại thư mục gốc của dự án `user-service` (`user-service/.gitlab-ci.yml`):
```yaml
image: eclipse-temurin:17-jdk-alpine

stages:
  - build_jar
  - build_image

variables:
  IMAGE_NAME: "user-service"

# Job biên dịch code ra file JAR
build_jar_job:
  stage: build_jar
  tags:
    - quickbite
  script:
    - chmod +x ./gradlew
    - ./gradlew bootJar
  artifacts:
    paths:
      - build/libs/*.jar
    expire_in: 1 hour

# Job build Docker image sử dụng file JAR artifacts từ stage trước
# Lưu ý: Không cần khai báo services: - docker:dind do dùng Local Runner gắn socket
build_docker_image_job:
  stage: build_image
  image: docker:latest
  tags:
    - quickbite
  dependencies:
    - build_jar_job
  script:
    # Biên dịch Docker image từ Dockerfile đơn tầng trực tiếp trên daemon của máy host
    - docker build -t ${IMAGE_NAME}:latest .
```

> [!IMPORTANT]
> **Không kiểm tra môi trường bằng after_script:**
> Tất cả các câu lệnh kiểm tra (như `docker info` hay `docker images`) phải được thực thi trực tiếp trong khối `script` chính. Việc đưa các lệnh này vào `after_script` là sai nguyên tắc vì nếu các kiểm tra trong `after_script` thất bại, job vẫn được báo trạng thái thành công (Passed), gây lọt lỗi trong pipeline.

#### 4.3 Kết quả mong đợi
Khi push cấu hình lên GitLab, pipeline sẽ khởi chạy tuần tự qua 2 stage. Log của job `build_docker_image_job` hiển thị quá trình copy file JAR thành công và hoàn tất lệnh build image.

---

### PHẦN 5. LƯU Ý, LỖI SAI VÀ HIỂU LẦM THƯỜNG GẶP (DÀNH CHO HỌC VIÊN)

* **Thiếu cấu hình chia sẻ Docker Socket:** Học viên khi tự host Runner thường quên cấu hình socket vật lý `/var/run/docker.sock` trong volumes của dịch vụ Runner (Docker Compose hoặc lệnh đăng ký). Lỗi này khiến job CI/CD báo lỗi:
  ```text
  Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
  ```
  *Khắc phục:* Đảm bảo cấu hình volumes của Runner chứa khai báo `"/var/run/docker.sock:/var/run/docker.sock"`.
* **Hiểu lầm về việc bắt buộc phải dùng `services`:** Sinh viên đọc tài liệu trực tuyến của GitLab thấy luôn yêu cầu khai báo `services: - docker:dind` nên tự ý thêm vào. Với thiết kế sử dụng Local Runner của giáo trình, việc khai báo dind là dư thừa và làm chậm thời gian chạy do phải khởi tạo thêm container phụ.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG

1. **Sử dụng Docker build trong GitLab CI/CD:**
   * [Run Docker commands in GitLab CI/CD | GitLab Documentation](https://docs.gitlab.com/ee/ci/docker/using_docker_build.html)
2. **Cấu hình chia sẻ Docker Socket cho Runner:**
   * [Binding Docker socket for GitLab Runner | GitLab Documentation](https://docs.gitlab.com/ee/ci/docker/using_docker_build.html#use-docker-socket-binding)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao ở Stage 2 (`build_docker_image_job`), chúng ta bắt buộc phải cấu hình thuộc tính `dependencies` trỏ đến `build_jar_job`?
* *Gợi ý:* Vì mỗi job chạy trên một container độc lập và sạch sẽ. Khi job 1 kết thúc, container biên dịch sẽ bị hủy và file JAR sẽ biến mất. Nhờ thuộc tính `dependencies`, GitLab Runner sẽ tải file JAR artifacts đã được lưu trữ trên GitLab Server về workspace của container ở job 2 trước khi chạy script.

#### Câu 2 (Phân tích so sánh)
Tại sao thiết kế sử dụng Local Runner với cơ chế chia sẻ Docker Socket (DooD) lại tối ưu hơn việc dùng dịch vụ Docker-in-Docker (DinD) trong bài học này?
* *Gợi ý:* Vì cơ chế chia sẻ Docker Socket cho phép container của job giao tiếp trực tiếp với Docker daemon của máy host. Nhờ đó, job có thể tái sử dụng trực tiếp các image đã tải và các layer đã build lưu trên máy host, không tốn thời gian tải lại và khởi tạo một daemon phụ độc lập như DinD.

#### Câu 3 (Cấu hình hệ thống)
Nếu cấu hình Local Runner không mount Docker socket, làm thế nào để job CI/CD có thể build được Docker image?
* *Gợi ý:* Cần phải đổi sang sử dụng mô hình Docker-in-Docker (DinD) bằng cách khai báo `services: - docker:dind` trong file `.gitlab-ci.yml`, đồng thời cấu hình Runner ở chế độ `privileged = true` trong file `config.toml` của nó.
