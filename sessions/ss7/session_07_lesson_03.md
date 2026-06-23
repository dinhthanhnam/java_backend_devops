# SESSION 07: CI/CD CƠ BẢN VỚI GITLAB

## LESSON 03: Cách hoạt động của pipeline CI/CD

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Phân tích** được luồng hoạt động tuần tự (Sequential) của các `stages` và song song (Parallel) của các `jobs` trong một pipeline.
* **Giải thích** được cơ chế hoạt động độc lập và cô lập (Isolation) của từng job chạy trên Docker Executor.
* **Vận dụng** thuộc tính `artifacts` để thu thập và chuyển giao sản phẩm đầu ra (như tệp tin báo cáo, file thực thi) qua các stage kế tiếp.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (NỖI ĐAU CỦA VIỆC MẤT FILE GIỮA CÁC GIAI ĐOẠN)

Trong quy trình phát triển thực tế, các giai đoạn (stages) của pipeline có mối quan hệ phụ thuộc chặt chẽ:
* Giai đoạn 1: Biên dịch và tạo file JAR thành phẩm (`user-service.jar`).
* Giai đoạn 2: Đóng gói file JAR đó vào Dockerfile để build thành Docker Image.

* **Nỗi đau xảy ra:**
  Do cơ chế hoạt động cô lập hoàn toàn của Docker Executor, mỗi job sẽ khởi chạy một Docker container mới tinh và sạch sẽ, sau đó tự hủy container đó khi job kết thúc. Kết quả là file JAR được sinh ra ở Giai đoạn 1 sẽ **biến mất vĩnh viễn** khi Giai đoạn 1 kết thúc. Khi Giai đoạn 2 khởi chạy, container mới hoàn toàn không tìm thấy file JAR nào để thực hiện build Docker Image.

Để giải quyết vấn đề chuyển giao dữ liệu/sản phẩm giữa các môi trường cô lập, GitLab CI/CD cung cấp cơ chế chuyển giao tài nguyên trung gian thông qua thuộc tính **`artifacts`**.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (LUỒNG CHẠY PIPELINE & CƠ CHẾ ARTIFACTS)

#### 3.1 Luồng hoạt động của Stages và Jobs
Một Pipeline là tập hợp của nhiều Giai đoạn (Stages), mỗi Stage lại chứa một hoặc nhiều Công việc (Jobs) cụ thể:
* **Quy trình chạy Stage tuần tự:** Các stage chạy tuần tự từ trên xuống dưới theo khai báo tại khối `stages`. Nếu một job trong stage trước bị lỗi (Failed), toàn bộ pipeline sẽ dừng lại ngay lập tức, các stage chạy sau sẽ bị hủy bỏ (skipped).
* **Quy trình chạy Job song song:** Nếu dự án của bạn có nhiều Runner hoạt động đồng thời, các job thuộc cùng một stage sẽ được phân phối chạy song song để tiết kiệm thời gian.

#### 3.2 Cơ chế cô lập Job (Job Isolation)
Đây là đặc tính an toàn cốt lõi khi sử dụng Docker Executor:
1. Runner nhận job -> Dựng container mới từ image cấu hình.
2. Clone mã nguồn mới nhất từ Git vào container.
3. Thực thi kịch bản lệnh trong `script`.
4. Hủy bỏ container (mọi file tạm phát sinh trong lúc chạy sẽ bị xóa sạch khỏi đĩa cứng).

#### 3.3 Cơ chế hoạt động của Artifacts
`**artifacts**` là cơ chế cho phép GitLab Runner thu thập các file/thư mục được sinh ra từ một job, đóng gói lại và đẩy ngược lên lưu trữ tạm thời trên máy chủ GitLab Server:

```text
[ Stage 1: Job build_jar ] ──► (Tạo file app.jar) ──(Upload Artifact)──► [ GitLab Server ]
                                                                                │
                                                                         (Tải xuống Artifact)
                                                                                │
[ Stage 2: Job build_image] ◄───────────────────────────────────────────────────┘
```

* Khi job chạy sau ở stage tiếp theo khởi chạy, Runner sẽ tự động tải các file artifact từ GitLab Server về thư mục làm việc để job chạy sau có thể kế thừa và tiếp tục xử lý.
* Sử dụng thuộc tính `expire_in` để giới hạn thời gian lưu trữ artifact trên server (ví dụ: `2 days` hoặc `1 week`), sau thời gian này GitLab sẽ tự động xóa file để giải phóng dung lượng ổ cứng.

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (THIẾT LẬP ARTIFACTS GIỮA CÁC STAGES)

Để chứng minh cơ chế truyền dẫn và lưu trữ của `artifacts` trên môi trường container cô lập, chúng ta xây dựng một pipeline 2 giai đoạn tối giản.

#### 4.1 Biên soạn tệp cấu hình `.gitlab-ci.yml`
Tạo file `.gitlab-ci.yml` tại thư mục gốc của dự án `user-service` với cấu hình sau:

```yaml
image: alpine:latest

stages:
  - generate
  - print

create_file_job:
  stage: generate
  script:
    - echo "Tạo tệp cấu hình tạm thời..."
    - echo "PORT=8081" > config.properties
    - echo "DB_HOST=quickbite-db" >> config.properties
  artifacts:
    paths:
      - config.properties
    expire_in: 10 mins

read_file_job:
  stage: print
  script:
    - echo "Đang đọc tệp cấu hình kế thừa từ stage trước..."
    - cat config.properties
```

#### 4.2 Kết quả mong đợi
* Job `create_file_job` hoàn thành, tạo file `config.properties` và upload lên server thành công.
* Job `read_file_job` khởi chạy ở stage tiếp theo, tự động tải file `config.properties` về và in ra đúng nội dung cấu hình mặc dù chạy trên một container hoàn toàn độc lập với container của job trước.
* Trên giao diện Web UI của GitLab tại trang quản lý pipeline, bạn sẽ thấy xuất hiện nút bấm cho phép tải trực tiếp file `config.properties` về máy local để kiểm tra.

---

### PHẦN 5. LƯU Ý, LỖI SAI VÀ HIỂU LẦM THƯỜNG GẶP

* **Quên khai báo artifacts để chuyển giao file:** Sinh viên rất hay mắc lỗi không khai báo `artifacts` vì nghĩ rằng file được tạo ra ở stage trước sẽ mặc định tồn tại ở các stage sau. Hãy luôn nhớ rằng mỗi job là một container hoàn toàn mới và bị hủy ngay khi xong, bắt buộc phải dùng `artifacts` để đẩy file lên server trung gian.
* **Khai báo sai đường dẫn của artifacts:** Cấu hình sai thuộc tính `paths` (ví dụ: file nằm trong `build/libs/*.jar` nhưng ghi nhầm thành `build/*.jar`) sẽ khiến Runner báo lỗi không tìm thấy tệp tin để tải lên và làm sập job.
* **Hiểu lầm giữa Artifacts và Cache:** Nhiều sinh viên cho rằng hai cơ chế này giống nhau. Thực tế, `artifacts` dùng để chuyển giao sản phẩm đầu ra (file JAR) giữa các stage trong một pipeline; còn `cache` dùng để lưu trữ thư mục thư viện phụ thuộc (`.gradle/caches`) để tăng tốc độ tải ở các lần chạy sau, không dùng để chuyển giao sản phẩm.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG (XÁC MINH KIẾN THỨC)

1. **Quản lý Job Artifacts trong GitLab:**
   * [Job artifacts | GitLab Documentation](https://docs.gitlab.com/ee/ci/jobs/job_artifacts.html)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao file thực thi JAR sinh ra ở stage build lại biến mất ở stage deploy nếu chúng ta không khai báo thuộc tính `artifacts`?
* *Gợi ý:* Vì tính chất cô lập của Docker Executor. Mỗi job khi chạy sẽ được khởi tạo trong một container hoàn toàn độc lập và tự hủy container đó kèm toàn bộ hệ thống file tạm thời ngay khi job kết thúc. Không khai báo artifacts đồng nghĩa với việc không lưu trữ kết quả đầu ra lên máy chủ trung tâm.

#### Câu 2 (Phân tích so sánh)
Điều gì xảy ra với các stage tiếp theo của pipeline nếu một job ở stage đầu tiên bị lỗi (Failed)?
* *Gợi ý:* Toàn bộ các stage và job chạy phía sau sẽ bị hủy bỏ (skipped) ngay lập tức để bảo vệ hệ thống khỏi việc sử dụng hoặc triển khai các sản phẩm bị lỗi.

#### Câu 3 (Cấu hình hệ thống)
Tham số `expire_in: 10 mins` trong cấu hình artifacts có vai trò gì và tại sao nó lại cần thiết trên môi trường Production?
* *Gợi ý:* Xác định thời gian hết hạn lưu trữ của file artifact trên máy chủ GitLab Server (ở đây là 10 phút). Việc này rất cần thiết để giải phóng bộ nhớ đĩa cho server, ngăn chặn việc tích tụ các file sản phẩm cũ qua hàng nghìn lượt commit gây tràn ổ cứng.
