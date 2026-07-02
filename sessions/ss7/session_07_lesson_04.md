# SESSION 07: TỰ ĐỘNG HÓA QUY TRÌNH CI/CD VỚI GITHUB ACTIONS

## LESSON 04: Ứng dụng CI/CD: Biên dịch tự động Spring Boot với Gradle

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, học viên có khả năng:
* **Cấu hình** được môi trường biên dịch (JDK) thông qua Action chuẩn `actions/setup-java` phù hợp với phiên bản Java của từng microservice.
* **Biên soạn** thành công quy trình đóng gói hoàn chỉnh: Tải mã nguồn -> Cấu hình Java -> Cấp quyền Gradle Wrapper -> Biên dịch ra file JAR thương mại (`bootJar`) một cách tối giản.
* **Lưu trữ** tệp tin thành phẩm bằng cách sử dụng Action `actions/upload-artifact` để có thể tải xuống từ giao diện GitHub.
* **Tối ưu hóa** hiệu năng luồng chạy bằng cách tinh giản các bước kiểm thử rườm rà.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (THÁCH THỨC VỀ HIỆU NĂNG VÀ TÀI NGUYÊN)

Trong quy trình thực hành học tập, chúng ta tận dụng tài nguyên **GitHub-hosted runner miễn phí** được cấp phát chung cho các dự án mã nguồn mở hoặc dự án cá nhân nhỏ. Tuy nhiên, chúng có những giới hạn về mặt hệ thống:
* Máy ảo chỉ được cấp phát cấu hình RAM/CPU có giới hạn (thường là 2 vCPU và 7GB RAM).
* Môi trường sạch hoàn toàn ở mỗi lượt chạy, đồng nghĩa với việc không duy trì bộ đệm thư viện (cache) nếu không được thiết lập tường minh.
* Nếu kích hoạt toàn bộ quy trình kiểm thử tự động cùng lúc, Spring Boot sẽ phải nạp toàn bộ cấu trúc ứng dụng (Spring Context) lên RAM của Runner, làm tăng đáng kể thời gian thực thi và rủi ro hết tài nguyên cục bộ.

Để tối ưu hóa thời gian trả về kết quả (thường chỉ nên nằm dưới mức **1.5 phút** cho các tác vụ biên dịch), chúng ta cần thiết kế một **luồng công việc (workflow) tinh giản tối đa**: tạm thời bỏ qua bước kiểm thử tự động phức tạp và tiến thẳng vào mục tiêu cốt lõi là biên dịch ra tệp tin JAR thành phẩm.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (CẤU HÌNH PIPELINE CI TINH GIẢN)

#### 3.1 Cấu hình JDK thông minh
Ứng dụng Spring Boot yêu cầu môi trường JDK đầy đủ để biên dịch mã nguồn Java. Thay vì thiết lập hệ điều hành theo cách thủ công, GitHub cung cấp Action `actions/setup-java`:
* Hỗ trợ cài đặt nhanh các phiên bản: Java 17 (dành cho `user-service`), Java 21 (dành cho `restaurant-service`).
* Chỉ định nhà phân phối bằng tham số `distribution: 'temurin'` (tương đương với bộ Eclipse Temurin JDK tối ưu hóa).

#### 3.2 Tái sử dụng Gradle Wrapper (`gradlew`)
Hệ thống không yêu cầu cài đặt cài đặt Gradle vào hệ điều hành của máy chủ Runner. Chúng ta sẽ tận dụng tệp tin thực thi Gradle Wrapper (`gradlew` và thư mục `gradle/wrapper/`) đã được nhà phát triển tích hợp sẵn trong mã nguồn gốc:
* **`chmod +x ./gradlew`:** Cấp quyền thực thi cho tệp tin (execute bit) trước khi khởi chạy (điều kiện bắt buộc trên môi trường Linux).
* **`./gradlew bootJar`:** Câu lệnh yêu cầu công cụ Gradle biên dịch và đóng gói ứng dụng Spring Boot thành tệp JAR thực thi. Tác vụ `bootJar` có đặc điểm là mặc định không kích hoạt các quy trình kiểm thử (tests), đảm bảo hệ thống phản hồi nhanh nhất.

#### 3.3 Tận dụng Fallback của application.yml
Trong quá trình đóng gói JAR, khuôn khổ Spring Boot có thể sẽ kiểm thử kết nối cơ sở dữ liệu để xác minh tính hợp lệ. Phương pháp thiết lập dự phòng (fallback) trong `application.yml` ở Session 5 sẽ phát huy tác dụng:
```yaml
url: jdbc:postgresql://${DB_HOST:localhost}:${DB_PORT:5432}/${DB_NAME:quickbite_user}
```
Nhờ cơ chế này, quá trình biên dịch sẽ sử dụng giá trị mặc định (`localhost`) một cách mượt mà mà không đòi hỏi phải thiết lập một cơ sở dữ liệu PostgreSQL thực tế đang hoạt động đồng thời trên Runner.

#### 3.4 Lưu trữ Artifacts
Khác với kiến trúc máy chủ nguyên khối thông thường, tệp JAR sau khi được sinh ra sẽ nằm trên không gian ảo của Runner và sẽ biến mất khi phiên chạy kết thúc. Để giữ lại thành quả, cần dùng `actions/upload-artifact@v4` để đóng gói tệp tin đó và tải lên không gian lưu trữ của GitHub.

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (BIÊN SOẠN FILE CI CHO USER-SERVICE)

#### 4.1 Biên soạn tệp cấu hình `.github/workflows/build.yml`
Tạo file `build.yml` tại thư mục `.github/workflows/` của dự án `user-service` với cấu hình tinh giản sau:

```yaml
name: Build Spring Boot App

on:
  push:
    branches:
      - main

jobs:
  build_executable_jar:
    runs-on: [self-hosted, quickbite] # Sử dụng Self-hosted runner của tổ chức
    steps:
      - name: Checkout mã nguồn
        uses: actions/checkout@v5
        
      - name: Cài đặt môi trường JDK 17
        uses: actions/setup-java@v5
        with:
          java-version: '17'
          distribution: 'temurin'
          
      - name: Cấp quyền thực thi cho Gradle Wrapper
        run: chmod +x ./gradlew
        
      - name: Biên dịch và đóng gói tệp JAR (Bỏ qua tác vụ Test)
        run: ./gradlew bootJar
        
      - name: Lưu trữ sản phẩm (Artifact)
        uses: actions/upload-artifact@v4
        with:
          name: user-service-jar
          path: build/libs/*.jar
          retention-days: 3
```

> [!TIP]
> **Đồng bộ cho restaurant-service:**
> Đối với dịch vụ `restaurant-service` sử dụng nền tảng Java 21, học viên chỉ cần điều chỉnh tham số `java-version: '17'` thành `java-version: '21'` tại bước cài đặt môi trường. Các quy trình còn lại được kế thừa nguyên vẹn.

#### 4.2 Kết quả mong đợi
Khi thực hiện thao tác đẩy mã nguồn (push), quy trình sẽ khởi động và kết thúc trong khoảng **1.5 phút**. Tại tab **Actions**, chọn phiên chạy vừa hoàn tất. Bạn sẽ thấy mục **Artifacts** nằm ở cuối trang, chứa tệp tin `user-service-jar` sẵn sàng để tải xuống.

---

### PHẦN 5. LƯU Ý, LỖI SAI VÀ HIỂU LẦM THƯỜNG GẶP

* **Bỏ quên lệnh cấp quyền cho Gradle Wrapper:** Rất nhiều học viên không khai báo lệnh `chmod +x ./gradlew` trước khối lệnh biên dịch, hậu quả là máy chủ Runner sẽ trả về mã lỗi thoát (Exit code) với thông báo `Permission Denied`.
* **Khai báo không đồng bộ phiên bản JDK:** Lỗi do thiết lập `java-version: '17'` cho các cấu phần sử dụng chuẩn ngôn ngữ Java 21. Lỗi này gây ra thông báo lỗi định dạng lớp không tương thích (`UnsupportedClassVersionError`) từ trình biên dịch.
* **Sai cấu trúc đường dẫn Artifact:** Nếu khai báo tham số `path: target/*.jar` (dành cho Maven) thay vì `build/libs/*.jar` (dành cho Gradle), hệ thống sẽ không tìm thấy tệp tin nào để tải lên và báo cảnh báo `No files were found with the provided path`.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG

1. **Thiết lập Java Action:**
   * [actions/setup-java repository](https://github.com/actions/setup-java)
2. **Quy chuẩn lưu trữ dữ liệu luồng công việc:**
   * [actions/upload-artifact repository](https://github.com/actions/upload-artifact)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao chúng ta sử dụng tệp tin `./gradlew` (Gradle Wrapper) đính kèm sẵn trong mã nguồn thay vì sử dụng hệ thống cài đặt chung để cài Gradle trực tiếp lên hệ điều hành của máy chủ Runner?
* *Gợi ý:* Sử dụng Gradle Wrapper giúp đảm bảo tính nhất quán tuyệt đối về phiên bản Gradle được sử dụng để xây dựng mã nguồn trên mọi hệ thống (từ máy tính cá nhân đến máy chủ CI/CD) mà không phải lo lắng về việc máy chủ chạy phiên bản công cụ khác biệt gây rủi ro kỹ thuật.

#### Câu 2 (Phân tích so sánh)
Sự khác biệt về hành vi chạy kiểm thử (unit test) giữa lệnh `./gradlew bootJar` và lệnh `./gradlew build` là gì?
* *Gợi ý:* Lệnh `./gradlew bootJar` thuần túy thực hiện chức năng biên dịch và đóng gói tệp JAR mà bỏ qua quy trình kiểm thử tự động. Ngược lại, lệnh `./gradlew build` đại diện cho vòng đời xây dựng toàn diện, yêu cầu mọi kiểm thử phải vượt qua (Passed) mới tiến hành bước tạo tệp thành phẩm.

#### Câu 3 (Cấu hình hệ thống)
Mục đích của việc thiết lập tham số `retention-days: 3` trong cấu hình lưu trữ Artifact là gì?
* *Gợi ý:* GitHub cung cấp một dung lượng lưu trữ giới hạn (Quota) cho các luồng công việc. Việc giới hạn thời gian tồn tại (retention) của tệp tin thành 3 ngày giúp hệ thống tự động dọn dẹp các tệp bản dựng cũ không còn giá trị, tối ưu hóa không gian lưu trữ và tránh vượt định mức miễn phí.
