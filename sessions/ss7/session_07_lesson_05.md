# SESSION 07: CI/CD CƠ BẢN VỚI GITLAB

## LESSON 05: Phân tích log pipeline và xử lý lỗi build

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Đọc hiểu và phân tích** cấu trúc log xuất ra từ giao diện điều khiển (Console Output) của GitLab CI.
* **Xác định và sửa đổi nhanh** các lỗi build thực tế phổ biến (lỗi biên dịch cú pháp Java, lỗi phân quyền file thực thi Gradle).
* **Tư duy chẩn đoán lỗi** hệ thống dựa trên dấu vết log thay vì phỏng đoán nguyên nhân một cách mơ hồ.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (NỖI ĐAU CỦA VIỆC PIPELINE BÁO ĐỎ)

Khi bạn thiết lập quy trình CI/CD tự động, việc pipeline bị báo lỗi đỏ (Failed) là điều xảy ra thường xuyên trong quá trình phát triển.
* **Nỗi đau thực tế:**
  Khác với môi trường local chạy trên IDE (như IntelliJ IDEA hay Eclipse) có giao diện đồ họa hỗ trợ chỉ ra tận nơi dòng code bị lỗi, môi trường CI hoàn toàn là một hộp đen. Khi có lỗi xảy ra, container chạy job sẽ sụp đổ và bạn chỉ nhận về một luồng log văn bản thuần túy (console log) kéo dài hàng trăm dòng. 

Nếu không có kỹ năng đọc và phân tích log một cách khoa học, lập trình viên sẽ rơi vào tình trạng loay hoay thử nghiệm sửa code một cách ngẫu nhiên, mất hàng giờ đồng hồ đẩy code lên xuống chỉ để cầu may pipeline chuyển sang màu xanh.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (CÁC LỖI BUILD PHỔ BIẾN TRONG PIPELINE TINH GIẢN)

Do chúng ta đã chuyển đổi sang cấu hình build tối giản (chạy lệnh `./gradlew bootJar -x test` và sử dụng cấu hình mặc định có sẵn trong `application.yml`), các lỗi phức tạp liên quan đến kết nối cơ sở dữ liệu vật lý hoặc lỗi tràn bộ nhớ (Out of Memory - OOM) khi boot Spring Boot context đã được loại bỏ hoàn toàn. 

Sinh viên giờ đây sẽ chỉ tập trung xử lý các lỗi build cơ bản nhưng cực kỳ phổ biến sau:

#### 3.1 Lỗi quên cấp quyền file thực thi (`Permission Denied`)
* **Dấu hiệu nhận biết:** Log console báo lỗi ngay khi bắt đầu thực thi lệnh chạy Gradle Wrapper:
  ```text
  $ ./gradlew bootJar -x test
  /bin/sh: eval: line 135: ./gradlew: Permission denied
  ERROR: Job failed: exit code 1
  ```
* **Bản chất lỗi:** Tệp tin `./gradlew` đẩy lên Git bị mất quyền thực thi (`execute`) trên môi trường Linux của Runner.
* **Cách khắc phục:** Thêm lệnh cấp quyền `chmod +x ./gradlew` ngay trước câu lệnh thực thi build trong file `.gitlab-ci.yml`.

#### 3.2 Lỗi biên dịch cú pháp Java (`Java Compilation Errors`)
* **Dấu hiệu nhận biết:** Log của Gradle báo lỗi `Compilation failed` đi kèm chi tiết lỗi của compiler:
  ```text
  > Task :compileJava FAILED
  /builds/quickbite/user-service/src/main/java/com/quickbite/user/service/UserService.java:24: error: cannot find symbol
          private UserReposotory userRepository;
                  ^
    symbol:   class UserReposotory
    location: class UserService
  1 error
  ```
* **Bản chất lỗi:** Có lỗi chính tả, thiếu import thư viện, hoặc sai tên class/interface trong mã nguồn Java.
* **Cách khắc phục:** Đọc kỹ dòng log để xác định đường dẫn tệp tin, dòng bị lỗi (dòng 24) và sửa lỗi cú pháp trực tiếp trong mã nguồn Java dưới local trước khi push lại code.

#### 3.3 Lỗi sai lệch phiên bản JDK/JRE (`Java Version Mismatch`)
* **Dấu hiệu nhận biết:** Log compiler báo lỗi không tương thích Class Version:
  ```text
  Class file has wrong version 65.0, should be 61.0
  ```
* **Bản chất lỗi:** Mã nguồn dự án khai báo sử dụng Java 21 (class version 65.0) nhưng image chạy CI của Runner lại cấu hình sử dụng JDK 17 (class version 61.0).
* **Cách khắc phục:** Cập nhật lại từ khóa `image` trong file cấu hình `.gitlab-ci.yml` cho khớp với phiên bản Java của dự án.

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (ĐỌC LOG PHÂN TÍCH LỖI THỰC TẾ)

Dưới đây là một ví dụ minh họa về luồng console log khi job bị failed trên giao diện GitLab và cách tiếp cận xử lý:

#### 4.1 Log lỗi sập pipeline trên GitLab console
```text
Running with gitlab-runner 16.5.0 (85d304a5)
  on docker-auto-scale 7a21b3c9
Preparing the "docker" executor
Using Docker executor with image eclipse-temurin:17-jdk-alpine ...
Pulling docker image eclipse-temurin:17-jdk-alpine ...
Using docker image sha256:d8b2... for eclipse-temurin:17-jdk-alpine ...
Preparing environment
Checking out 5a3c1e2d as main...
Skipping Git submodules setup
Downloading artifacts
Installing dependencies
$ ./gradlew bootJar -x test
/bin/sh: eval: line 135: ./gradlew: Permission denied
Cleaning up project directory and file based variables
ERROR: Job failed: exit code 1
```

#### 4.2 Quy trình chẩn đoán và khắc phục
1. **Tìm điểm sập (Failure point):** Cuộn xuống cuối cùng của log để xem dòng báo lỗi có tiền tố `ERROR:` hoặc dòng chứa mã lỗi `exit code`. Ở đây lỗi là: `Job failed: exit code 1`.
2. **Xác định câu lệnh gây lỗi:** Nhìn ngay lên phía trên dòng ERROR để xem câu lệnh nào vừa được thực thi và sinh lỗi. Ta thấy: `$ ./gradlew bootJar -x test` sinh ra log `Permission denied`.
3. **Thực hiện sửa lỗi:** Cập nhật file `.gitlab-ci.yml` mẫu để bổ sung lệnh cấp quyền chạy:
```yaml
build_executable_jar:
  stage: build
  script:
    - chmod +x ./gradlew
    - ./gradlew bootJar -x test
```

---

### PHẦN 5. LƯU Ý, LỖI SAI VÀ HIỂU LẦM THƯỜNG GẶP

* **Phỏng đoán nguyên nhân mơ hồ khi sập build:** Sinh viên khi thấy pipeline báo đỏ thường có thói quen thay đổi code Java ngẫu nhiên mà không đọc log console. Hãy luôn cuộn xuống cuối log để tìm dòng FAILED/ERROR, xác định câu lệnh gây lỗi, sau đó xem chi tiết thông báo lỗi của Java Compiler để sửa code chính xác dưới local trước khi push lại.
* **Bỏ qua các mã lỗi Exit Code:** Bỏ qua các mã exit code của hệ điều hành Linux (như exit code 1 đại diện cho lỗi chung hoặc permission, exit code 127 đại diện cho lệnh không tồn tại). Việc đọc và hiểu exit code giúp khoanh vùng nguyên nhân lỗi rất nhanh.
* **Hiểu lầm rằng mọi lỗi đỏ đều do code Java sai:** Nhiều sinh viên lầm tưởng chỉ cần code Java compile được ở máy local thì pipeline không thể lỗi. Thực tế, lỗi đỏ thường phát sinh do cấu hình file YAML sai thụt lề, phân quyền của Runner (Permission Denied) hoặc sai lệch phiên bản môi trường build (JDK).

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG (XÁC MINH KIẾN THỨC)

1. **Khắc phục lỗi trong GitLab CI/CD:**
   * [GitLab CI/CD Troubleshooting | GitLab Documentation](https://docs.gitlab.com/ee/ci/troubleshooting.html)
2. **Quy chuẩn mã lỗi exit code của tiến trình Linux:**
   * [Bash Standard Exit Codes Reference](https://tldp.org/LDP/abs/html/exitcodes.html)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao lỗi `Permission denied` thường chỉ xuất hiện khi chạy build trên Runner (Linux) mà không xuất hiện khi chạy ở máy local của sinh viên (sử dụng Windows)?
* *Gợi ý:* Vì hệ điều hành Windows không quản lý chặt chẽ thuộc tính quyền thực thi (execute bit) của file như các hệ điều hành nhân Unix (Linux/macOS). Khi file được đẩy lên Git từ máy Windows, thuộc tính quyền thực thi của file `./gradlew` có thể bị mất, khiến hệ điều hành Linux của Runner chặn không cho phép khởi chạy tiến trình.

#### Câu 2 (Phân tích so sánh)
Lầm tưởng phổ biến nào của lập trình viên khi sửa lỗi đỏ của pipeline mà không thông qua việc đọc console log?
* *Gợi ý:* Lập trình viên thường lầm tưởng mọi lỗi đỏ của job CI đều do lỗi logic viết sai trong code Java, từ đó tập trung sửa code Java vô ích trong khi thực tế lỗi có thể nằm ở cấu hình môi trường của file YAML hoặc phân quyền chạy của Gradle Wrapper.

#### Câu 3 (Cấu hình hệ thống)
Nếu log của job build báo lỗi `UnsupportedClassVersionError: class has been compiled by a more recent version of the Java Runtime`, nguyên nhân do đâu và cách xử lý thế nào?
* *Gợi ý:* Nguyên nhân là do dự án được phát triển và yêu cầu chạy trên phiên bản Java cao hơn (ví dụ Java 21) so với phiên bản JDK của image đang chạy trên Runner (ví dụ image đang dùng JDK 17). Cách xử lý là nâng cấp phiên bản image JDK trong file `.gitlab-ci.yml` cho tương thích (ví dụ dùng `eclipse-temurin:21-jdk-alpine`).
