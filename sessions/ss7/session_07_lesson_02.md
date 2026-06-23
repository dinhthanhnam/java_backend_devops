# SESSION 07: TỰ ĐỘNG HÓA BIÊN DỊCH VỚI GITLAB CI

## LESSON 02: Cấu trúc và Cú pháp file .gitlab-ci.yml

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Nắm vững** cú pháp YAML và quy tắc thụt lề bắt buộc trong file cấu hình `.gitlab-ci.yml`.
* **Khai báo và cấu trúc** thành công một file cấu hình cơ bản gồm các thành phần cốt lõi: `stages`, `image`, `variables`, và các định nghĩa `job`.
* **Sử dụng** được các từ khóa kiểm soát luồng chạy của job như `only`, `rules`, `before_script`, `after_script`.
* **Áp dụng** cơ chế `tags` để điều phối job chạy trên đúng môi trường mong muốn.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (THÁCH THỨC TRONG THIẾT LẬP KỊCH BẢN TỰ ĐỘNG HÓA)

Khi đã cài đặt và đăng ký GitLab Runner thành công, Runner vẫn chưa thể tự động biết được nó cần thực thi những hành động nào cho mã nguồn của bạn.
* Cần chạy lệnh gì để biên dịch dự án Spring Boot?
* Dùng phiên bản JDK nào để build?
* Có cần in ra log thông báo khi hoàn thành không?

Để cung cấp hướng dẫn chạy cho Runner, chúng ta phải viết một file kịch bản cấu hình lưu trữ trực tiếp cùng mã nguồn theo nguyên lý **Infrastructure as Code (Hạ tầng như mã nguồn)**. Định dạng file cấu hình được GitLab sử dụng là YAML, đặt tên tệp tin cố định là `.gitlab-ci.yml`.

> [!IMPORTANT]
> **Lưu ý đặc biệt về cấu trúc thư mục:**
> Mỗi service (`user-service`, `restaurant-service`...) hiện tại đã là một Git Repository độc lập. File `.gitlab-ci.yml` dưới đây phải được đặt ngay tại thư mục gốc của chính service đó (ví dụ: `user-service/.gitlab-ci.yml`), tuyệt đối không đặt ở thư mục cha của dự án.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (CÚ PHÁP VÀ CÁC TỪ KHÓA NỀN TẢNG)

#### 3.1 Cú pháp YAML cơ bản
YAML là định dạng dữ liệu có cấu trúc phân cấp bằng **thụt lề khoảng trắng (spaces)**:
* Không sử dụng phím Tab để thụt lề; bắt buộc sử dụng phím cách (phổ biến nhất là 2 khoảng trắng).
* Định nghĩa dữ liệu theo cặp `key: value`. Lưu ý bắt buộc phải có một khoảng trắng (space) sau dấu hai chấm `:`.
* Các phần tử của một danh sách (array) được bắt đầu bằng dấu gạch ngang `-` và một khoảng trắng.

#### 3.2 Các từ khóa cấu hình toàn cục (Global Keywords)
* **`image`:** Định nghĩa Docker image làm môi trường chạy mặc định cho các job (ví dụ: `eclipse-temurin:17-jdk-alpine`). Mọi câu lệnh trong job sẽ được thực thi bên trong container khởi tạo từ image này.
* **`stages`:** Khai báo danh sách các giai đoạn lớn của pipeline và xác định thứ tự chạy của chúng từ trên xuống dưới (ví dụ: `info`, `compile`, `test`).
* **`variables`:** Nơi định nghĩa các biến môi trường dùng chung trong toàn bộ pipeline.

#### 3.3 Định nghĩa một Job cụ thể
Một Job là đơn vị thực thi dòng lệnh nhỏ nhất trong pipeline. Cấu trúc khai báo một job gồm:
* **Tên job:** Do lập trình viên tự đặt (ví dụ: `print_env_job`).
* **`stage`:** Chỉ định job này thuộc giai đoạn nào trong danh sách `stages` đã khai báo toàn cục.
* **`script`:** Khối lệnh chứa các câu lệnh shell được thực thi tuần tự trong container. Đây là thuộc tính bắt buộc của mọi job.

#### 3.4 Các từ khóa kiểm soát luồng chạy của Job
* **`before_script`:** Chạy một loạt các câu lệnh phụ trước khi các câu lệnh trong `script` chính bắt đầu.
* **`after_script`:** Chạy các câu lệnh dọn dẹp hoặc in log sau khi các câu lệnh trong `script` chính đã hoàn thành (dù job chạy thành công hay thất bại).
* **`rules` / `only`:** Điều kiện để quyết định xem job có được kích hoạt hay không (ví dụ: `only: [main]` chỉ chạy job khi code được push lên nhánh `main`).

#### 3.5 Sử dụng tags để điều phối Job
* **`tags`:** Dùng để chỉ định cụ thể những GitLab Runner nào được phép nhận thực thi job này.
* Khi đăng ký Local Runner với tag `quickbite`, nếu job không khai báo `tags: [quickbite]`, GitLab Server có thể phân phối job này cho bất kỳ Runner dùng chung (Shared Runner) nào khác của hệ thống đang hoạt động. Điều này dẫn đến hiện tượng "cướp job" và gây sập pipeline do môi trường Shared Runner không tương thích. Bằng cách gán tag rõ ràng, chúng ta đảm bảo job chỉ chạy trên Runner local đã định cấu hình.

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (TẠO FILE .GITLAB-CI.YML ĐẦU TIÊN)

#### 4.1 Biên soạn tệp cấu hình
Tạo file `.gitlab-ci.yml` tại thư mục gốc của dự án `user-service` với nội dung sau:

```yaml
image: eclipse-temurin:17-jdk-alpine

variables:
  PROJECT_NAME: "QuickBite-User-Service"

stages:
  - info

print_env_job:
  stage: info
  tags:
    - quickbite # Chỉ định Runner local tránh hiện tượng bị cướp job
  before_script:
    - echo "Chuẩn bị in thông tin môi trường cho dự án ${PROJECT_NAME}..."
  script:
    - java -version
    - echo "Tên dự án hiện tại là ${PROJECT_NAME}"
    - echo "Nhánh Git đang chạy pipeline là ${CI_COMMIT_BRANCH}"
  after_script:
    - echo "Hoàn thành job in thông tin môi trường."
```

#### 4.2 Kết quả mong đợi
Khi thực hiện commit và push code lên nhánh chính của Git, GitLab Server sẽ tự động phát hiện tệp tin này và khởi tạo một pipeline. Khi kiểm tra nhật ký log trên giao diện GitLab, job `print_env_job` sẽ chạy thành công (Passed) và hiển thị log:
* In ra phiên bản JDK 17 (Eclipse Temurin).
* In ra giá trị biến nội suy `QuickBite-User-Service`.
* In ra tên nhánh đang chạy pipeline thông qua biến môi trường định sẵn `$CI_COMMIT_BRANCH`.

---

### PHẦN 5. LƯU Ý, LỖI SAI VÀ HIỂU LẦM THƯỜNG GẶP

* **Đặt sai vị trí tệp tin cấu hình:** Sinh viên thường viết cấu hình `.gitlab-ci.yml` ở thư mục tổng (thư mục cha) của dự án thay vì đặt ở thư mục gốc của từng service độc lập (như `user-service/.gitlab-ci.yml`). Điều này khiến GitLab Server không nhận dạng được cấu hình.
* **Sử dụng phím Tab để thụt lề cấu hình:** Cú pháp YAML cấm sử dụng phím Tab. Sinh viên thường quen tay nhấn Tab khiến hệ thống báo lỗi cú pháp (`syntax error`) và từ chối chạy pipeline. Bắt buộc phải sử dụng khoảng trắng để thụt lề.
* **Hiểu lầm về biến môi trường định sẵn:** Nghĩ rằng các biến môi trường bắt đầu bằng `$CI_` (như `$CI_COMMIT_BRANCH`) phải được khai báo trong phần `variables` thì mới sử dụng được. Thực tế, đây là các biến do GitLab tự động nạp sẵn vào container của job, sinh viên có thể gọi sử dụng trực tiếp.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG (XÁC MINH KIẾN THỨC)

1. **Đặc tả từ khóa của GitLab CI/CD:**
   * [GitLab CI/CD YAML syntax reference](https://docs.gitlab.com/ee/ci/yaml/)
2. **Hướng dẫn biến môi trường định sẵn (Predefined Variables):**
   * [Predefined variables reference](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao trong file cấu hình `.gitlab-ci.yml`, việc sử dụng thụt lề bằng phím Tab lại bị cấm và gây ra lỗi biên dịch?
* *Gợi ý:* Vì định dạng YAML quy định nghiêm ngặt việc phân cấp các khối cấu hình bằng khoảng trắng (spaces). Phím Tab có thể được biên dịch với kích thước khoảng trắng khác nhau tùy thuộc vào cấu hình của trình soạn thảo văn bản, gây mất tính nhất quán và phá vỡ cấu trúc phân cấp của tệp tin.

#### Câu 2 (Đọc và dự đoán)
Nếu một lập trình viên khai báo biến môi trường nội bộ trong file cấu hình dạng `${PROJECT_NAME}` nhưng quên không định nghĩa khóa `PROJECT_NAME` trong khối `variables`, chuyện gì sẽ xảy ra?
* *Gợi ý:* Lệnh chạy sẽ không báo lỗi cú pháp YAML nhưng giá trị hiển thị sẽ là rỗng (blank hoặc chuỗi trống) do hệ thống không tìm thấy giá trị của khóa biến đó để nội suy.

#### Câu 3 (Cấu hình hệ thống)
Mục đích của việc sử dụng các biến có tiền tố `$CI_` (như `$CI_COMMIT_BRANCH` hay `$CI_COMMIT_SHORT_SHA`) trong file `.gitlab-ci.yml` là gì?
* *Gợi ý:* Đây là các biến môi trường định sẵn (Predefined Variables) do GitLab CI/CD tự động cung cấp trong mọi job. Chúng giúp lập trình viên truy xuất nhanh các thông tin động của lượt chạy (như tên nhánh Git, mã hash của commit, tên người trigger pipeline...) mà không cần phải tự cấu hình thủ công.
