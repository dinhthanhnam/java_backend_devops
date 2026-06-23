# SESSION 07: TỰ ĐỘNG HÓA BIÊN DỊCH VỚI GITLAB CI

## LESSON 04: Build ứng dụng Spring Boot bằng Maven/Gradle trong CI

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Cấu hình** được môi trường biên dịch (JDK, Gradle Wrapper) phù hợp với phiên bản Java của từng microservice.
* **Biên soạn** thành công pipeline hoàn chỉnh thực hiện: Cấp quyền Gradle Wrapper -> Biên dịch ra file JAR thương mại (`bootJar`) và lưu vào artifacts một cách tối giản.
* **Tối ưu hóa** hiệu năng pipeline chạy trên Shared Runner bằng cách loại bỏ các cấu hình cache rườm rà và bỏ qua bước test để pipeline chạy nhanh nhất.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (THÁCH THỨC VỀ HIỆU NĂNG VÀ TÀI NGUYÊN PIPELINE)

Trong quy trình phát triển thực hành tại lớp học hoặc các dự án nhỏ, chúng ta tận dụng tài nguyên **Shared Runner trực tuyến miễn phí** của GitLab. Tuy nhiên, Shared Runner có những giới hạn:
* Tài nguyên CPU/RAM được cấp phát hạn chế.
* Không duy trì bộ nhớ đệm (cache) cục bộ ổn định như máy chủ Runner tự cấu hình (self-hosted). Mỗi khi chạy, Runner có thể là một máy ảo hoàn toàn khác nhau.
* Nếu chạy test tự động khi build, Spring Boot sẽ phải khởi chạy toàn bộ cấu trúc ứng dụng (Spring Context) lên RAM của Runner, làm tốn thời gian và dễ gây lỗi timeout hoặc thiếu RAM.

Để tối ưu hóa thời gian chạy pipeline xuống dưới **1.5 phút**, giúp sinh viên có phản hồi kết quả nhanh nhất sau khi push code, chúng ta cần thiết kế một **pipeline CI tinh giản tối đa**: bỏ qua bước kiểm thử tự động, tắt cấu hình cache rườm rà và đi thẳng vào mục tiêu build ra file JAR thành phẩm.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (CẤU HÌNH PIPELINE CI TINH GIẢN)

#### 3.1 Chọn Image JDK phù hợp
Ứng dụng Spring Boot yêu cầu môi trường JDK đầy đủ để có thể biên dịch mã nguồn Java. Chúng ta sử dụng image chính thức từ Eclipse Temurin đi kèm hệ điều hành Alpine siêu nhẹ để tối ưu dung lượng:
* Java 17 (dành cho `user-service`, `order-service`): `eclipse-temurin:17-jdk-alpine`
* Java 21 (dành cho `restaurant-service`): `eclipse-temurin:21-jdk-alpine`

#### 3.2 Tái sử dụng Gradle Wrapper (`gradlew`)
Chúng ta không cần cài đặt Gradle lên hệ điều hành của Runner. Thay vào đó, tận dụng tệp tin thực thi Gradle Wrapper (`gradlew` và thư mục `gradle/wrapper/`) đã được đính kèm sẵn trong mã nguồn dự án:
* **`chmod +x ./gradlew`:** Cấp quyền thực thi cho file wrapper trước khi chạy (bắt buộc trên Linux/Docker Runner).
* **`./gradlew bootJar`:** Câu lệnh yêu cầu Gradle biên dịch và đóng gói ứng dụng Spring Boot thành file JAR thực thi. Khác với lệnh `build` chạy đầy đủ các tác vụ kiểm thử (tests), lệnh `bootJar` mặc định không phụ thuộc vào tác vụ `test`, giúp quy trình CI diễn ra nhanh nhất mà không cần chạy các bài kiểm thử.

#### 3.3 Tận dụng Fallback của application.yml
Khi build JAR, Spring Boot có thể cần kiểm tra cấu hình kết nối. Nhờ cách viết cấu hình thông minh sử dụng fallback trong `application.yml` ở Session 5:
```yaml
url: jdbc:postgresql://${DB_HOST:localhost}:${DB_PORT:5432}/${DB_NAME:quickbite_user}
```
Nếu không nạp biến môi trường hệ thống, ứng dụng sẽ tự động sử dụng giá trị mặc định (như `localhost`). Nhờ đó, quá trình biên dịch và đóng gói file JAR diễn ra mượt mà mà không đòi hỏi phải có một cơ sở dữ liệu PostgreSQL thực tế đang chạy lúc build.

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (BIÊN SOẠN FILE CI CHO USER-SERVICE)

---

#### 4.1 Biên soạn tệp cấu hình `.gitlab-ci.yml`
Tạo file `.gitlab-ci.yml` tại thư mục gốc của dự án `user-service` với cấu hình tinh giản sau:

```yaml
image: eclipse-temurin:17-jdk-alpine

stages:
  - build

build_executable_jar:
  stage: build
  tags:
    - quickbite
  script:
    # Cấp quyền thực thi cho Gradle Wrapper có sẵn trong dự án
    - chmod +x ./gradlew
    # Biên dịch và đóng gói file JAR (tác vụ bootJar mặc định không chạy kiểm thử)
    - ./gradlew bootJar
  artifacts:
    paths:
      - build/libs/*.jar
    expire_in: 3 days
```

> [!TIP]
> **Đồng bộ cho restaurant-service:**
> Đối với dịch vụ `restaurant-service` chạy Java 21, giảng viên chỉ cần hướng dẫn sinh viên sửa dòng cấu hình image đầu tiên thành:
> `image: eclipse-temurin:21-jdk-alpine`
> Toàn bộ các dòng cấu hình phía sau được giữ nguyên vẹn.

#### 4.2 Kết quả mong đợi
Khi push code lên GitLab, pipeline sẽ khởi chạy và hoàn thành chỉ trong khoảng **1 đến 1.5 phút**. Tệp tin sản phẩm `user-service.jar` được sinh ra tại đường dẫn `build/libs/` và được lưu giữ an toàn trên hệ thống lưu trữ artifact của GitLab.

---

### PHẦN 5. LƯU Ý, LỖI SAI VÀ HIỂU LẦM THƯỜNG GẶP

* **Quên cấp quyền chạy cho Gradle Wrapper:** Rất nhiều sinh viên quên không khai báo lệnh `chmod +x ./gradlew` trước khi gọi lệnh build, dẫn đến việc Runner sập ngay lập tức với lỗi `Permission Denied` trên môi trường Linux.
* **Sử dụng sai phiên bản JDK nền:** Dùng image JDK 17 cho các dịch vụ viết trên Java 21 (như `restaurant-service`) hoặc ngược lại sẽ dẫn đến các lỗi biên dịch của Java Compiler hoặc lỗi phiên bản class chạy ứng dụng (`UnsupportedClassVersionError`). Cần sửa khóa `image` cho đồng nhất với phiên bản Java của dự án.
* **Phân biệt tác vụ bootJar và build:** Học viên dễ nhầm lẫn rằng tác vụ `bootJar` cũng sẽ chạy test giống như tác vụ `build`. Cần nhớ rằng `bootJar` chỉ làm nhiệm vụ biên dịch và đóng gói, không phụ thuộc vào `test`. Trong các dự án thực tế của doanh nghiệp, chúng ta thường sử dụng tác vụ `build` để bắt buộc chạy toàn bộ các ca kiểm thử trước khi đóng gói sản phẩm, hoặc thiết lập stage test riêng biệt để kiểm soát chất lượng.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG (XÁC MINH KIẾN THỨC)

1. **Đóng gói ứng dụng Spring Boot với Gradle:**
   * [Spring Boot Gradle Plugin Reference Guide](https://docs.spring.io/spring-boot/docs/current/gradle-plugin/reference/htmlsingle/)
2. **Các tác vụ trong Gradle:**
   * [Gradle Java Library Plugin Reference](https://docs.gradle.org/current/userguide/java_library_plugin.html)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao chúng ta sử dụng tệp tin `./gradlew` (Gradle Wrapper) có sẵn trong dự án thay vì chạy lệnh cài đặt Gradle trực tiếp trên môi trường của Runner?
* *Gợi ý:* Sử dụng Gradle Wrapper giúp đảm bảo tính nhất quán về mặt phiên bản Gradle được sử dụng để build dự án trên mọi môi trường (local, CI/CD, Production) mà không phụ thuộc vào việc máy chủ chạy có cài sẵn Gradle hay không và cài phiên bản nào.

#### Câu 2 (Phân tích so sánh)
Sự khác biệt về hành vi chạy kiểm thử (unit test) giữa câu lệnh `./gradlew bootJar` và câu lệnh `./gradlew build` là gì?
* *Gợi ý:* Lệnh `./gradlew bootJar` chỉ tập trung biên dịch mã nguồn và đóng gói file JAR thực thi mà không kích hoạt các tác vụ chạy unit test. Ngược lại, lệnh `./gradlew build` là một lifecycle task đầy đủ, sẽ tự động kích hoạt tác vụ chạy kiểm thử (`test`) và chỉ tiến hành đóng gói khi tất cả các kiểm thử đều vượt qua thành công.

#### Câu 3 (Cấu hình hệ thống)
Để đóng gói một microservice viết bằng Java 21 (ví dụ: `restaurant-service`), lập trình viên cần thay đổi thông số nào trong file cấu hình `.gitlab-ci.yml` mẫu của `user-service`?
* *Gợi ý:* Chỉ cần thay đổi giá trị của từ khóa `image` từ `eclipse-temurin:17-jdk-alpine` (Java 17) thành `eclipse-temurin:21-jdk-alpine` (Java 21) để Runner khởi tạo môi trường biên dịch tương thích với phiên bản Java của mã nguồn.
