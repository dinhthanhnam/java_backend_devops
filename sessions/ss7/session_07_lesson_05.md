# SESSION 07: TỰ ĐỘNG HÓA BIÊN DỊCH VỚI GITLAB CI

## LESSON 05: Thực hành phân tích log và chẩn đoán lỗi build trên GitLab CI

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, học viên có khả năng:
* **Xác định vị trí** và truy cập giao diện Log Console của Job trên GitLab Web UI.
* **Chẩn đoán nguyên nhân** lỗi của pipeline bằng phương pháp phân tích log từ dưới lên.
* **Khắc phục trực tiếp** các lỗi thực tế kế thừa từ kịch bản build của Lesson 04 bao gồm: lỗi phân quyền thực thi (`Permission Denied`), lỗi biên dịch mã nguồn Java (`Compilation Failed`), và lỗi kiểm thử thất bại (`Test FAILED`).

---

### PHẦN 2. THIẾT LẬP MÔI TRƯỜNG THỰC HÀNH VÀ DEMO TRỰC TIẾP

Bài học này được thiết kế để giảng viên thực hiện demo trực tiếp và hướng dẫn học viên chẩn đoán lỗi trên giao diện GitLab. Không tập trung vào lý thuyết trừu tượng, bài học đi thẳng vào việc tạo ra các tình huống lỗi thực tế dựa trên cấu hình build Spring Boot tối giản từ Lesson 04.

#### 2.1 File cấu hình baseline chuẩn (từ Lesson 04)
Học viên sử dụng dự án `user-service` với file cấu hình `.gitlab-ci.yml` chuẩn ban đầu:
```yaml
image: eclipse-temurin:17-jdk-alpine

stages:
  - build

build_executable_jar:
  stage: build
  tags:
    - quickbite
  script:
    - chmod +x ./gradlew
    - ./gradlew bootJar
  artifacts:
    paths:
      - build/libs/*.jar
    expire_in: 3 days
```

---

### PHẦN 3. KỊCH BẢN LỖI 1: LỖI PHÂN QUYỀN WRAPPER (PERMISSION DENIED)

#### 3.1 Cách tạo lỗi demo
1. Giảng viên chỉnh sửa file `.gitlab-ci.yml`, loại bỏ lệnh cấp quyền `chmod +x ./gradlew`:
```yaml
build_executable_jar:
  stage: build
  tags:
    - quickbite
  script:
    - ./gradlew bootJar
```
2. Commit và push thay đổi lên GitLab repository để kích hoạt pipeline.

#### 3.2 Nhận diện lỗi trên GitLab Log Console
Khi pipeline báo trạng thái thất bại (Failed), giảng viên hướng dẫn học viên mở log của job `build_executable_jar` và quan sát phần console output:
```text
$ ./gradlew bootJar
/bin/sh: eval: line 135: ./gradlew: Permission denied
Cleaning up project directory and file based variables
ERROR: Job failed: exit code 1
```

#### 3.3 Giải thích nguyên nhân và cách khắc phục
* **Ý nghĩa lỗi:** Tệp thực thi `./gradlew` thiếu thuộc tính quyền thực thi (execute bit) trên hệ điều hành Linux của Runner. Lỗi này thường xuất hiện khi dự án được phát triển hoặc chỉnh sửa trên Windows (hệ điều hành không quản lý chặt chẽ thuộc tính execute bit của tệp tin), dẫn đến việc Git không lưu trữ thuộc tính này khi đẩy lên repository.
* **Cách khắc phục:** Khai báo lệnh `chmod +x ./gradlew` trước lệnh biên dịch để cấp quyền thực thi cho wrapper trên môi trường Linux của Runner.

---

### PHẦN 4. KỊCH BẢN LỖI 2: LỖI BIÊN DỊCH MÃ NGUỒN (COMPILATION FAILED)

#### 4.1 Cách tạo lỗi demo
1. Giảng viên cố tình đưa một lỗi cú pháp vào mã nguồn Java của dịch vụ `user-service`. Ví dụ, trong file `src/main/java/com/quickbite/user/service/UserService.java`, thay đổi tên repository thành một tên không tồn tại hoặc xóa một dấu chấm phẩy `;` ở cuối dòng:
   ```java
   // Lỗi cố ý: Viết sai tên class UserRepository
   private UserReposotory userRepository;
   ```
2. Đảm bảo file `.gitlab-ci.yml` vẫn giữ lệnh cấp quyền `chmod +x ./gradlew`.
3. Commit và push mã nguồn lên GitLab để kích hoạt pipeline.

#### 4.2 Nhận diện lỗi trên GitLab Log Console
Mở log của job thất bại và định vị khu vực báo lỗi của Java Compiler:
```text
$ ./gradlew bootJar
> Task :compileJava FAILED
/builds/quickbite/user-service/src/main/java/com/quickbite/user/service/UserService.java:24: error: cannot find symbol
        private UserReposotory userRepository;
                ^
  symbol:   class UserReposotory
  location: class UserService
1 error
```

#### 4.3 Giải thích nguyên nhân và cách khắc phục
* **Ý nghĩa lỗi:** Trình biên dịch Java (compiler) không thể dịch mã nguồn thành bytecode do vi phạm quy tắc cú pháp hoặc không tìm thấy các tham chiếu lớp/giao diện. Log console chỉ ra chính xác đường dẫn tệp tin, dòng (dòng 24) và ký tự gây lỗi.
* **Cách khắc phục:** Học viên cần mở đúng tệp tin và dòng được chỉ định trong log, sửa lại mã nguồn đúng cú pháp, chạy thử lệnh `./gradlew compileJava` dưới local để xác nhận biên dịch thành công trước khi commit và push lại.

---

### PHẦN 5. KỊCH BẢN LỖI 3: LỖI KIỂM THỬ THẤT BẠI (TEST FAILED)

#### 5.1 Cách tạo lỗi demo
1. Giảng viên điều chỉnh file cấu hình `.gitlab-ci.yml` để kích hoạt việc chạy Unit Test bằng cách thay thế tác vụ `bootJar` (mặc định không chạy test) thành tác vụ `build` (chạy đầy đủ các bước kiểm thử):
```yaml
build_executable_jar:
  stage: build
  tags:
    - quickbite
  script:
    - chmod +x ./gradlew
    - ./gradlew build # Thay bootJar bằng build để kích hoạt chạy test
```
2. Chỉnh sửa logic của một lớp kiểm thử trong thư mục `src/test/java/...` để cố tình làm kiểm thử thất bại (ví dụ: thay đổi giá trị mong đợi trong assertion `assertEquals(200, status)` thành `assertEquals(500, status)` hoặc ném ra ngoại lệ).
3. Commit và push thay đổi lên GitLab để kích hoạt pipeline.

#### 5.2 Nhận diện lỗi trên GitLab Log Console
Mở log của job thất bại và quan sát phần kết quả thực thi kiểm thử:
```text
> Task :test

UserServiceApplicationTests > testCreateUser() FAILED
    org.opentest4j.AssertionFailedError at UserServiceApplicationTests.java:17

2 tests completed, 1 failed

> Task :test FAILED
```

#### 5.3 Giải thích nguyên nhân và cách khắc phục
* **Ý nghĩa lỗi:** Mã nguồn Java biên dịch thành công, nhưng logic chạy thử nghiệm không khớp với thiết kế kiểm thử (Assertion thất bại). Gradle phát hiện kiểm thử thất bại và dừng tiến trình đóng gói để tránh xuất bản một sản phẩm lỗi.
* **Cách khắc phục:** 
  1. Xác định kiểm thử bị lỗi thông qua log console.
  2. Sửa lại mã nguồn logic hoặc assertion trong tệp kiểm thử cho chính xác.
  3. Chạy thử nghiệm dưới máy local bằng lệnh `./gradlew test` để đảm bảo tất cả kiểm thử đều vượt qua (passed) trước khi thực hiện push code.

---

### PHẦN 6. QUY TRÌNH CHẨN ĐOÁN LỖI PIPELINE TRÊN GITLAB (DÀNH CHO HỌC VIÊN)

Khi pipeline của dự án báo trạng thái Failed (màu đỏ), học viên cần thực hiện quy trình chẩn đoán có hệ thống sau:
1. **Truy cập giao diện log:** Chọn **Build** -> **Jobs** trên thanh menu trái của GitLab, click vào job bị lỗi.
2. **Đọc log từ dưới lên:**
   * Tìm dòng cuối cùng hiển thị mã lỗi thoát (ví dụ: `ERROR: Job failed: exit code 1`).
   * Di chuyển ngược lên phía trên để xác định câu lệnh gây lỗi (dòng bắt đầu bằng ký tự `$`).
   * Đọc chi tiết thông tin lỗi nằm giữa câu lệnh gây lỗi và mã lỗi thoát để xác định chính xác nguyên nhân (lỗi compiler chỉ rõ dòng code, lỗi test chỉ rõ tên kiểm thử thất bại).
3. **Khắc phục và xác minh:** Thực hiện sửa đổi tương ứng, chạy kiểm tra cục bộ (local) trước khi push code lên Git.

---

### PHẦN 7. LƯU Ý, LỖI SAI VÀ HIỂU LẦM THƯỜNG GẶP

* **Chỉnh sửa mã nguồn không có cơ sở khi sập build:** Học viên thường tự suy đoán nguyên nhân và sửa đổi mã nguồn ngẫu nhiên khi thấy pipeline báo đỏ. Hãy luôn tuân thủ việc đọc log để định vị chính xác vị trí lỗi trước khi sửa.
* **Bỏ qua mã lỗi Exit Code:** Không chú ý đến các mã lỗi do hệ thống trả về (ví dụ: `exit code 1` thường do lỗi biên dịch hoặc phân quyền, `exit code 127` do gõ sai tên lệnh thực thi).
* **Mặc định mọi lỗi đỏ đều do code Java:** Nhiều lỗi thực tế xuất phát từ cấu hình YAML sai định dạng thụt lề, sai phiên bản JDK nền của Docker image, hoặc thiếu lệnh phân quyền chạy Gradle Wrapper.

---

### PHẦN 8. TÀI LIỆU THAM KHẢO CHÍNH THỐNG

1. **Khắc phục lỗi trong GitLab CI/CD:**
   * [GitLab CI/CD Troubleshooting | GitLab Documentation](https://docs.gitlab.com/ee/ci/troubleshooting.html)
2. **Quy chuẩn mã lỗi exit code của tiến trình Linux:**
   * [Bash Exit Codes Reference](https://tldp.org/LDP/abs/html/exitcodes.html)

---

### PHẦN 9. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao lỗi `Permission denied` thường chỉ xuất hiện khi chạy build trên Runner (Linux) mà không xuất hiện khi chạy ở máy local của học viên sử dụng hệ điều hành Windows?
* *Gợi ý:* Vì hệ điều hành Windows không quản lý chặt chẽ thuộc tính quyền thực thi (execute bit) của file như các hệ điều hành nhân Unix (Linux/macOS). Khi file được đẩy lên Git từ máy Windows, thuộc tính quyền thực thi của file `./gradlew` có thể bị mất, khiến hệ điều hành Linux của Runner chặn không cho phép khởi chạy tiến trình.

#### Câu 2 (Phân tích so sánh)
Mục đích của việc thiết lập pipeline dừng lại ngay lập tức khi phát hiện lỗi Unit Test thất bại (Kịch bản 3) là gì?
* *Gợi ý:* Áp dụng nguyên lý **Fail-fast (Thất bại sớm)**. Việc dừng pipeline ngay khi test thất bại giúp ngăn chặn đóng gói sản phẩm bị lỗi và tránh triển khai phiên bản lỗi này lên các môi trường tiếp theo.

#### Câu 3 (Chẩn đoán lỗi)
Nếu log của job build báo lỗi `UnsupportedClassVersionError: class has been compiled by a more recent version of the Java Runtime`, nguyên nhân do đâu và cách xử lý thế nào?
* *Gợi ý:* Nguyên nhân do dự án được thiết lập chạy trên phiên bản Java cao hơn (ví dụ Java 21) so với phiên bản JDK của Docker image đang cấu hình cho Runner (ví dụ image đang dùng JDK 17). Cách xử lý là nâng cấp phiên bản image JDK trong file `.gitlab-ci.yml` cho tương thích (ví dụ dùng `eclipse-temurin:21-jdk-alpine`).
