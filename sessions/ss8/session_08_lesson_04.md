# SESSION 08: ĐÓNG GÓI DOCKER IMAGE & ĐẨY LÊN REGISTRY

## LESSON 04: Sử dụng Docker Image từ Registry trong Pipeline CI/CD

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, học viên có khả năng:
* **Cấu hình** thành công file `.gitlab-ci.yml` để tự động xác thực và kéo (pull) image từ GitLab Container Registry về máy Runner.
* **Tận dụng** các biến môi trường ẩn định sẵn (Predefined Variables) của GitLab để bảo vệ thông tin xác thực.
* **Giải thích** được cơ chế tối ưu hiệu năng và thời gian chạy (runtime) của pipeline khi chuyển đổi từ build sang pull image.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (TỐI ƯU HÓA RUNTIME PIPELINE TRONG DỰ ÁN THỰC TẾ)

Sau khi đẩy thành công Docker image lên GitLab Container Registry từ local ở Lesson 03 (State 3), câu hỏi đặt ra là: Làm thế nào để pipeline CI/CD tích hợp và sử dụng image này?
* Trong thực tế, các doanh nghiệp hoặc đội ngũ phát triển nhỏ thường tối ưu hóa chi phí vận hành bằng cách **loại bỏ bước build Docker image phức tạp trong CI/CD**.
* Thay vì để máy Runner chạy tốn 5 phút tải code và compile, pipeline CI/CD sẽ chuyển sang luồng tối giản: Tự động kéo trực tiếp image đã được lập trình viên build và đẩy lên trước đó để thực hiện các job kiểm thử tự động, verify tính ổn định, hoặc kích hoạt deploy.
* Nhờ cơ chế này, thời gian chạy (runtime) của GitLab Runner sẽ giảm từ 5 phút xuống chỉ còn **khoảng 10 giây**, giúp tiết kiệm tài nguyên hệ thống và đẩy nhanh tiến độ bàn giao sản phẩm (State 4: Pull & Run trong CI/CD).

---

### PHẦN 3. KIẾN THỨC CỐT LÕI

#### 3.1 Sử dụng Predefined Variables để tự động đăng nhập trong CI/CD
Khi chạy trong môi trường Runner, chúng ta không được và không cần cấu hình thủ công Personal Access Token cá nhân vào file YAML. GitLab CI/CD tự động cấp phát một token tạm thời (temporary token) chỉ có hiệu lực trong lúc job chạy và nạp sẵn vào các biến môi trường định sẵn:
* **`$CI_REGISTRY`:** Địa chỉ máy chủ Docker Registry của GitLab (mặc định: `registry.gitlab.com`).
* **`$CI_REGISTRY_USER`:** Tên đăng nhập tạm thời do GitLab tự động sinh ra cho lượt chạy pipeline.
* **`$CI_REGISTRY_PASSWORD`:** Token đăng nhập tạm thời tương ứng với tài khoản trên, tự động hết hiệu lực ngay khi job hoàn thành.
* **`$CI_REGISTRY_IMAGE`:** Đường dẫn lưu trữ image mặc định của dự án (dạng `registry.gitlab.com/username/project-name`).

#### 3.2 Lệnh xác thực và kéo image trong Script
Trong script của job, Runner thực thi các câu lệnh sau để chuẩn bị môi trường chạy:
```bash
# Đăng nhập tự động bằng tài khoản tạm thời
docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY

# Kéo image từ registry dự án về Runner
docker pull $CI_REGISTRY_IMAGE:<version_tag>
```

#### 3.3 Cơ chế Ẩn biến bảo mật (Masking Variables) của GitLab
Giá trị thực của các biến bảo mật hệ thống như `$CI_REGISTRY_PASSWORD` được GitLab tự động kiểm duyệt và ẩn đi. Nếu có bất kỳ dòng log nào cố tình in giá trị này ra, hệ thống sẽ tự động chuyển đổi thành chuỗi `[MASKED]` trên log console để đảm bảo an toàn tuyệt đối.

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (CẤU HÌNH PULL VÀ RUN IMAGE TRONG PIPELINE)

Học viên tiến hành cập nhật file cấu hình `.gitlab-ci.yml` của dự án `user-service` để kéo image phiên bản `1.0.0` (đã đẩy lên Registry từ local ở Lesson 03) về chạy verify trong pipeline CI/CD.

#### 4.1 Biên soạn tệp cấu hình `.gitlab-ci.yml`
Cập nhật file cấu hình `.gitlab-ci.yml` tại thư mục gốc của dự án `user-service` (`user-service/.gitlab-ci.yml`):

```yaml
image: docker:latest

stages:
  - test_image

test_docker_image_job:
  stage: test_image
  tags:
    - quickbite
  script:
    # 1. Đăng nhập tự động vào Registry sử dụng tài khoản tạm thời của job
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY

    # 2. Kéo trực tiếp image phiên bản 1.0.0 đã được push từ local về Runner
    - docker pull $CI_REGISTRY_IMAGE:1.0.0

    # 3. Khởi chạy thử container từ image vừa kéo về để verify ứng dụng hoạt động ổn định
    - docker run -d --name verify_user_service -p 8081:8081 $CI_REGISTRY_IMAGE:1.0.0

    # 4. Chờ 5 giây để Spring Boot hoàn tất quá trình khởi động
    - sleep 5

    # 5. Kiểm tra danh sách container để xác nhận ứng dụng đang chạy ở trạng thái UP (exit code = 0)
    - docker ps

    # 6. Dọn dẹp môi trường (dừng và xóa container verify) trực tiếp trong script chính
    - docker rm -f verify_user_service
```

> [!IMPORTANT]
> **Tránh sử dụng after_script để dọn dẹp container:**
> Sinh viên thường có thói quen đưa các câu lệnh dọn dẹp môi trường như `docker rm -f` vào `after_script`. Điều này là không nên vì `after_script` chạy trong shell độc lập và lỗi phát sinh trong đó không được tính vào kết quả Passed/Failed của job. Hãy viết tất cả các bước (bao gồm cả dọn dẹp) vào khối `script` chính.

#### 4.2 Kết quả mong đợi
Khi push file cấu hình lên GitLab, job `test_docker_image_job` sẽ được thực thi. Mở log console, chúng ta sẽ thấy Runner đăng nhập thành công, kéo image về cực kỳ nhanh, chạy container verify ổn định và kết thúc job thành công. Tổng thời gian chạy (runtime) của job sẽ hiển thị cực kỳ ngắn (thường chỉ khoảng 10-15 giây).

---

### PHẦN 5. LƯU Ý, LỖI SAI VÀ HIỂU LẦM THƯỜNG GẶP (DÀNH CHO HỌC VIÊN)

* **Gõ nhầm tên biến bảo mật của GitLab:** Sinh viên thường gõ sai ký tự viết hoa/thường hoặc thiếu chữ cái (ví dụ gõ thành `$CI_REGISTRY_PASS` thay vì `$CI_REGISTRY_PASSWORD`). Lỗi này khiến tham số truyền vào lệnh login bị trống và báo lỗi đăng nhập thất bại.
  *Khắc phục:* Luôn đối chiếu và sử dụng chính xác tên các biến mặc định của GitLab.
* **Lỗi không tìm thấy tag image (`manifest for ... not found`):** Xảy ra khi tag phiên bản khai báo trong file cấu hình CI/CD (ví dụ: `1.0.0`) không khớp với tag image thực tế đã được push lên Registry ở local trước đó.
  *Khắc phục:* Đảm bảo tag image được pull khớp chính xác với tag đã đẩy lên registry ở bài thực hành trước.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG

1. **Các biến môi trường định sẵn của GitLab CI/CD:**
   * [Predefined variables reference | GitLab Documentation](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html)
2. **Đăng nhập Docker Registry bằng dòng lệnh:**
   * [docker login CLI reference | Docker Documentation](https://docs.docker.com/engine/reference/commandline/login/)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao thời gian chạy (runtime) của Runner ở bài học này lại rút ngắn đáng kể (từ 5 phút xuống còn khoảng 10 giây) so với việc build image trực tiếp ở Lesson 01 và Lesson 02?
* *Gợi ý:* Vì Runner không phải thực hiện các tác vụ biên dịch mã nguồn Java nặng nề và tải hàng trăm MB dependencies từ Maven Central nữa. Lệnh `docker pull` chỉ làm nhiệm vụ tải trực tiếp các layer của image đã được đóng gói sẵn từ Registry về, giúp tiết kiệm tối đa thời gian biên dịch và tài nguyên mạng.

#### Câu 2 (Phân tích so sánh)
Mức độ bảo mật của biến `$CI_REGISTRY_PASSWORD` được bảo vệ như thế nào trên hệ thống GitLab CI/CD so với việc lập trình viên tự định nghĩa biến mật khẩu cá nhân?
* *Gợi ý:* Biến `$CI_REGISTRY_PASSWORD` được hệ thống sinh ra tự động dưới dạng token tạm thời và chỉ tồn tại trong vòng đời của job đang chạy; khi job kết thúc, token lập tức bị hủy bỏ. Đối với các biến tự định nghĩa tĩnh, chúng tồn tại vô hạn thời hạn và dễ bị lộ lọt nếu không được cấu hình chế độ Masked cẩn thận.

#### Câu 3 (Cấu hình hệ thống)
Nếu ứng dụng Spring Boot cần cấu hình thêm thông số kết nối database (ví dụ profile `dev`), học viên cần bổ sung cấu hình gì vào lệnh `docker run` trong script của file `.gitlab-ci.yml`?
* *Gợi ý:* Học viên cần bổ sung tham số biến môi trường bằng cờ `-e` vào lệnh `docker run`, ví dụ:
  ```bash
  docker run -d --name verify_user_service -e SPRING_PROFILES_ACTIVE=dev -p 8081:8081 $CI_REGISTRY_IMAGE:1.0.0
  ```
