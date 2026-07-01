# SESSION 07: TỰ ĐỘNG HÓA QUY TRÌNH CI/CD VỚI GITHUB ACTIONS

## LESSON 03: Cơ chế điều phối và phân tách Jobs trong luồng CI/CD

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, học viên có khả năng:
* **Phân tích** được luồng hoạt động mặc định chạy song song (Parallel) của các `jobs` trong một Workflow.
* **Vận dụng** từ khóa `needs` để thiết lập cơ chế luồng chạy tuần tự (Sequential) có sự phụ thuộc.
* **Giải thích** được cơ chế hoạt động độc lập và cô lập (Isolation) của từng công việc trên môi trường máy chủ Runner.
* **Thiết lập** phân tách cấu trúc các công việc để quản lý tiến trình tự động hóa hiệu quả.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (ĐIỀU PHỐI VÀ QUẢN LÝ TIẾN TRÌNH WORKFLOW)

Để thiết lập quy trình CI/CD hiệu quả cho hệ thống Microservices QuickBite, việc hiểu rõ cách hệ thống GitHub Actions kích hoạt và điều phối các công việc (jobs) là vô cùng quan trọng.
* Nếu chúng ta cấu hình hai công việc độc lập trong cùng một file, chúng sẽ chạy theo thứ tự nào?
* Làm sao để đảm bảo bước in báo cáo kết quả chỉ được khởi chạy sau khi bước thu thập thông tin đã hoàn tất thành công?

Thay vì lý thuyết suông, việc trực quan hóa luồng chạy thông qua kịch bản giả lập thiết lập mối liên kết phụ thuộc giữa các công việc sẽ giúp học viên thấy tận mắt cách hệ thống kiểm soát thứ tự thực thi và cơ chế cô lập tài nguyên.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (LUỒNG WORKFLOW & CÔ LẬP JOB)

#### 3.1 Luồng hoạt động mặc định của Jobs
Trong GitHub Actions, nếu bạn định nghĩa nhiều công việc (`jobs`) trong cùng một Workflow mà không khai báo thêm điều kiện gì, hành vi mặc định của hệ thống là **phân phối tất cả các công việc chạy song song cùng lúc**. Hệ thống sẽ cố gắng tìm kiếm các Runner có sẵn phù hợp với khai báo `runs-on` để xử lý chúng đồng thời nhằm rút ngắn tổng thời gian.

#### 3.2 Khởi tạo luồng tuần tự bằng từ khóa `needs`
Để thiết lập sự phụ thuộc (công việc A phải xong thì công việc B mới chạy), chúng ta sử dụng từ khóa `needs`.
* Công việc được cấu hình `needs: [job_truoc_do]` sẽ bắt buộc phải chờ đợi (pending) cho đến khi `job_truoc_do` hoàn thành với trạng thái thành công (Success).
* Nếu bất kỳ công việc nào trong danh sách `needs` bị thất bại (Failed), công việc hiện tại sẽ tự động bị bỏ qua (Skipped) để bảo vệ hệ thống khỏi việc tiếp tục sử dụng dữ liệu hoặc mã nguồn bị lỗi.

#### 3.3 Cơ chế cô lập Công việc (Job Isolation)
Mỗi một `job` trong GitHub Actions được coi là một thực thể cô lập hoàn toàn:
1. Runner nhận được yêu cầu chạy job -> Hệ thống cung cấp một môi trường máy ảo hoàn toàn sạch sẽ (đối với GitHub-hosted runner) hoặc chạy trong một tiến trình tác vụ riêng biệt.
2. Công việc bắt buộc phải dùng lệnh `actions/checkout` để tải mã nguồn, nếu không môi trường làm việc sẽ trống trơn.
3. Các công việc chạy độc lập, file cấu hình và dữ liệu sinh ra tại bộ nhớ cục bộ của công việc A sẽ không được tự động chuyển giao hay dùng chung bởi công việc B.

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (THIẾT LẬP WORKFLOW 2 GIAI ĐOẠN GIẢ LẬP)

Chúng ta sẽ thiết lập một kịch bản gồm 3 jobs: `job_info_1` và `job_info_2` chạy song song, sau đó job `job_print` sẽ chạy tuần tự phía sau khi cả hai job thông tin đã hoàn tất.

#### 4.1 Biên soạn tệp cấu hình `.github/workflows/ci.yml`
Tạo hoặc cập nhật file cấu hình tại dự án `user-service` với nội dung sau:

```yaml
name: Parallel and Sequential Workflow

on: [push]

jobs:
  job_info_1:
    runs-on: [self-hosted, quickbite]
    steps:
      - name: Thu thập thông tin OS
        run: |
          echo "Bắt đầu thu thập thông tin hệ thống..."
          uname -a

  job_info_2:
    runs-on: [self-hosted, quickbite]
    steps:
      - name: Thu thập thông tin User
        run: |
          echo "Bắt đầu kiểm tra tài khoản người dùng..."
          whoami

  job_print:
    needs: [job_info_1, job_info_2] # Khai báo phụ thuộc
    runs-on: [self-hosted, quickbite]
    steps:
      - name: In kết quả
        run: |
          echo "In ra kết quả cuối cùng từ workflow..."
          echo "Chỉ chạy sau khi job_info_1 và job_info_2 hoàn thành thành công."
```

#### 4.2 Kết quả mong đợi
* Khi mã nguồn được đẩy lên GitHub, luồng công việc sẽ kích hoạt. Giao diện trực quan của Actions tab sẽ vẽ một sơ đồ dạng lưới (graph).
* Hai nhánh đầu tiên của sơ đồ là `job_info_1` và `job_info_2` hoạt động song song.
* Nút tiếp theo của sơ đồ, được nối bằng đường kẻ biểu diễn luồng phụ thuộc, là `job_print`. Nút này sẽ hiển thị trạng thái đang chờ. Chỉ khi cả 2 nút đầu tiên chuyển sang trạng thái tích xanh (Success), nút `job_print` mới được kích hoạt thực thi.

---

### PHẦN 5. LƯU Ý, LỖI SAI VÀ HIỂU LẦM THƯỜNG GẶP

* **Tham chiếu sai tên công việc phụ thuộc:** Sử dụng tên job không tồn tại trong từ khóa `needs` hoặc viết sai chính tả (ví dụ: khai báo `needs: job_1` nhưng định nghĩa job là `job-1`) sẽ khiến Workflow bị báo lỗi định dạng tĩnh (Static validation error) ngay lập tức.
* **Đồng bộ hóa trạng thái tệp tin (Artifacts):** Học viên lầm tưởng các công việc có thể đọc trực tiếp các tệp tin sinh ra từ công việc chạy trước (như tệp cấu hình sinh ra tự động). Thực tế, do tính cô lập, nếu công việc `job_print` muốn dùng dữ liệu của `job_info_1`, bắt buộc phải sử dụng các hành động chia sẻ dữ liệu trung gian (actions upload/download artifacts) thay vì truy xuất thông qua đường dẫn cục bộ.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG

1. **Điều khiển thứ tự thực thi công việc (Controlling the execution order of jobs):**
   * [Using jobs in a workflow](https://docs.github.com/en/actions/using-jobs/using-jobs-in-a-workflow#defining-prerequisite-jobs)
2. **Cơ chế truyền dữ liệu giữa các công việc:**
   * [Storing workflow data as artifacts](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Hành vi mặc định của GitHub Actions đối với các khối công việc (`jobs`) được khai báo ngang cấp, không có ràng buộc phụ thuộc là gì?
* *Gợi ý:* Hành vi mặc định là thực thi song song toàn bộ các công việc. GitHub Server sẽ cố gắng cung cấp hoặc yêu cầu đủ số lượng môi trường Runner để giải quyết khối lượng công việc này cùng một lúc nhằm tăng tốc độ xử lý tổng thể của chu trình CI/CD.

#### Câu 2 (Phân tích so sánh)
Điều gì sẽ xảy ra với công việc B (được khai báo `needs: [A]`) nếu như công việc A bị lỗi và dừng lại với mã thoát (exit code) khác 0?
* *Gợi ý:* Toàn bộ các công việc có tính chất phụ thuộc đằng sau (ở đây là B) sẽ tự động bị bỏ qua (Skipped) ngay lập tức. Tính năng này đóng vai trò như một chốt chặn bảo mật, bảo vệ hệ thống khỏi việc sử dụng hoặc triển khai các sản phẩm bị lỗi từ các khâu trước.

#### Câu 3 (Cấu hình hệ thống)
Mặc dù bạn đã thiết lập công việc B chạy sau công việc A, nhưng tại sao công việc B vẫn báo lỗi không tìm thấy mã nguồn dự án mặc dù công việc A đã dùng lệnh `actions/checkout` để tải mã nguồn?
* *Gợi ý:* Do đặc tính cô lập công việc (Job Isolation) của Runner, các công việc được chạy trong các không gian hoàn toàn riêng biệt. Lệnh `actions/checkout` chỉ tải mã nguồn vào thư mục làm việc của công việc A. Để công việc B có mã nguồn, bạn bắt buộc phải khai báo lại lệnh `actions/checkout` ở bước thực thi đầu tiên của công việc B.
