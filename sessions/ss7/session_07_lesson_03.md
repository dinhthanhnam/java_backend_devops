# SESSION 07: TỰ ĐỘNG HÓA BIÊN DỊCH VỚI GITLAB CI

## LESSON 03: Cơ chế hoạt động của Pipeline và Phân tách Stages

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Phân tích** được luồng hoạt động tuần tự (Sequential) của các `stages` và song song (Parallel) của các `jobs` trong một pipeline.
* **Giải thích** được cơ chế hoạt động độc lập và cô lập (Isolation) của từng job chạy trên Docker Executor.
* **Vận dụng** phân tách cấu trúc stages để quản lý tiến trình tự động hóa.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (ĐIỀU PHỐI VÀ QUẢN LÝ TIẾN TRÌNH PIPELINE)

Để thiết lập quy trình CI/CD hiệu quả cho hệ thống Microservices QuickBite, việc hiểu rõ cách các công việc (jobs) được kích hoạt và điều phối là vô cùng quan trọng.
* Các công việc kiểm tra cú pháp và build code có chạy đồng thời được không?
* Làm sao đảm bảo bước in báo cáo chỉ chạy sau khi việc kiểm tra đã hoàn thành?

Thay vì lý thuyết suông, việc trực quan hóa luồng chạy thông qua một kịch bản giả lập có 2 stage khác nhau sẽ giúp bạn thấy tận mắt cách hệ thống kiểm soát thứ tự thực thi và cô lập tài nguyên.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (LUỒNG PIPELINE & CÔ LẬP JOB)

#### 3.1 Luồng hoạt động của Stages và Jobs
Một Pipeline là tập hợp của nhiều Giai đoạn (Stages), mỗi Stage lại chứa một hoặc nhiều Công việc (Jobs) cụ thể:
* **Quy trình chạy Stage tuần tự:** Các stage chạy tuần tự từ trên xuống dưới theo khai báo tại khối `stages`. Nếu một job trong stage trước bị lỗi (Failed), toàn bộ pipeline sẽ dừng lại ngay lập tức, các stage chạy sau sẽ bị hủy bỏ (skipped).
* **Quy trình chạy Job song song:** Các job thuộc cùng một stage sẽ được phân phối và chạy song song (nếu hệ thống có đủ máy Runner trống), giúp rút ngắn tối đa thời gian chờ đợi.

#### 3.2 Cơ chế cô lập Job (Job Isolation)
Đây là đặc tính an toàn cốt lõi khi sử dụng Docker Executor:
1. Runner nhận job -> Khởi tạo một container mới hoàn toàn sạch sẽ từ image cấu hình.
2. Tải mã nguồn mới nhất từ Git về container đó.
3. Thực thi kịch bản lệnh trong `script`.
4. Hủy bỏ container ngay khi hoàn thành (mọi file tạm phát sinh trong lúc chạy sẽ bị xóa sạch khỏi đĩa cứng).

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (THIẾT LẬP PIPELINE 2 GIAI ĐOẠN GIẢ LẬP)

Chúng ta sẽ thiết lập một kịch bản cấu hình có 2 stages: `info` (chứa các job chạy song song) và `print` (chạy tuần tự phía sau).

#### 4.1 Biên soạn tệp cấu hình `.gitlab-ci.yml`
Tạo file `.gitlab-ci.yml` tại thư mục gốc của dự án `user-service` với cấu hình sau:

```yaml
stages:
  - info
  - print

job_info_1:
  stage: info
  tags:
    - quickbite
  script:
    - echo "Bắt đầu thu thập thông tin hệ thống..."
    - uname -a

job_info_2:
  stage: info
  tags:
    - quickbite
  script:
    - echo "Bắt đầu kiểm tra môi trường..."
    - whoami

job_print:
  stage: print
  tags:
    - quickbite
  script:
    - echo "In ra kết quả cuối cùng từ pipeline..."
```

#### 4.2 Kết quả mong đợi
* Khi push code lên GitLab, pipeline sẽ hiển thị trực quan 2 cột tương ứng với 2 stages.
* Cột `info` chứa 2 jobs `job_info_1` và `job_info_2` chạy đồng thời (song song).
* Cột `print` chứa job `job_print` ở trạng thái chờ đợi. Chỉ khi cả 2 jobs ở stage `info` chuyển sang màu xanh (Passed), job `job_print` mới được kích hoạt chạy tiếp theo.

---

### PHẦN 5. LƯU Ý, LỖI SAI VÀ HIỂU LẦM THƯỜNG GẶP

* **Cấu hình sai thứ tự stage:** Gán job vào một stage không tồn tại trong danh sách `stages` toàn cục hoặc ghi sai chính tả tên stage làm pipeline báo lỗi cấu hình không hợp lệ (`invalid configuration`).
* **Đồng bộ hóa trạng thái job:** Lầm tưởng các job song song có thể can thiệp dữ liệu lẫn nhau trong quá trình chạy. Thực tế, chúng chạy trên các container độc lập hoàn toàn, không thể chia sẻ tài nguyên nếu không sử dụng các cơ chế truyền dẫn chuyên biệt.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG (XÁC MINH KIẾN THỨC)

1. **Luồng hoạt động của GitLab Pipelines:**
   * [GitLab Pipelines | GitLab Documentation](https://docs.gitlab.com/ee/ci/pipelines/)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Điều gì xảy ra với các stage tiếp theo của pipeline nếu một job ở stage đầu tiên bị lỗi (Failed)?
* *Gợi ý:* Toàn bộ các stage và job chạy phía sau sẽ bị hủy bỏ (skipped) ngay lập tức để bảo vệ hệ thống khỏi việc sử dụng hoặc triển khai các sản phẩm bị lỗi.

#### Câu 2 (Phân tích so sánh)
Các job trong cùng một stage có thể chạy song song trên các máy Runner khác nhau không?
* *Gợi ý:* Có, nếu hệ thống có nhiều Runner hoạt động đồng thời, GitLab Server sẽ tự động phân phối các job song song tới các máy Runner trống để tối ưu hiệu năng.

#### Câu 3 (Cấu hình hệ thống)
Làm thế nào để pipeline CI/CD nhận diện được thứ tự thực thi của các job?
* *Gợi ý:* Dựa vào danh sách khai báo toàn cục của khóa `stages` và thuộc tính `stage` được chỉ định bên trong từng job.
