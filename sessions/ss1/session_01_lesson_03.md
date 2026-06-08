# SESSION 01: TỔNG QUAN DEVOPS & QUY TRÌNH CI/CD

## LESSON 03: Mô hình môi trường Dev, Staging và Production

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau bài học này, bạn sẽ có khả năng:
* **Phân biệt** được mục đích sử dụng, đặc tính dữ liệu và tính chất của 3 môi trường: Development (Dev), Staging, và Production (Prod).
* **Áp dụng** nguyên tắc tách biệt cấu hình khỏi mã nguồn bằng biến môi trường (Environment Variables).
* **Cấu hình** ứng dụng Spring Boot đọc thông số database động để triển khai trên các môi trường khác nhau mà không cần build lại file JAR.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (NGUY CƠ LỘ THÔNG TIN VÀ SẬP DATABASE)

Hiện tại, dịch vụ `user-service` của QuickBite cần kết nối đến cơ sở dữ liệu PostgreSQL để lưu thông tin tài khoản. Trong quá trình phát triển ở local, lập trình viên đang cấu hình cứng (hardcode) tài khoản trong file `user-service/src/main/resources/application.yml` như sau:

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/quickbite_user
    username: postgres
    password: super-secret-password-123
```

Khi chuẩn bị đưa ứng dụng này lên chạy thử trên máy chủ Staging và chạy thật trên Production, quy trình thủ công này lập tức gặp các vấn đề nghiêm trọng:
1. **Lộ lọt thông tin bảo mật:** Kể cả khi dự án sử dụng Private Repository (kho lưu trữ riêng tư) trên GitHub/GitLab, rủi ro rò rỉ thông tin vẫn cực kỳ cao. Kho code private của dự án thường được phân quyền rộng rãi: từ các lập trình viên, các bạn QA/QC (đôi khi không chuyên sâu về tech) cho đến các bạn thực tập sinh (intern) thời vụ - những người có thể chỉ làm việc vài tháng rồi chuyển đi. Việc để mật khẩu database Production thật hiển thị công khai trên Git đồng nghĩa với việc bạn trao chìa khóa hệ thống cho tất cả những người này.
2. **Khó khăn khi chuyển đổi môi trường:** Mỗi môi trường có một IP database và mật khẩu riêng. Nếu cấu hình cứng, mỗi lần chuyển từ Dev sang Staging hay Prod, bạn lại phải mở code ra sửa file cấu hình, chạy lệnh build lại file JAR mới. Quy trình này rất cồng kềnh và dễ gây nhầm lẫn phiên bản.

---

### PHẦN 3. MÔ HÌNH 3 MÔI TRƯỜNG TIÊU CHUẨN

Để quản lý sản phẩm an toàn, dự án QuickBite được chia làm 3 môi trường hoạt động độc lập:

| Môi trường | Đối tượng sử dụng | Đặc điểm dữ liệu | Yêu cầu độ ổn định |
| :--- | :--- | :--- | :--- |
| **Dev (Development)** | Lập trình viên | Dữ liệu giả lập, tự tạo | Thấp (lỗi có thể sửa ngay) |
| **Staging (UAT/Thử nghiệm)** | Tester & Khách hàng duyệt | Mô phỏng dữ liệu thật (không nhạy cảm) | Trung bình - Cao (giống 99% Prod) |
| **Prod (Production/Chạy thật)** | Người dùng cuối | Dữ liệu thật, nhạy cảm | Tuyệt đối (Không được sập) |

> **Nguyên tắc vàng của DevOps:** **Build once, run anywhere (Build một lần, chạy mọi nơi).**
> Một file JAR duy nhất sau khi được build thành công từ máy chủ CI phải chạy được trên cả Dev, Staging và Prod mà không được phép biên dịch lại. Sự khác biệt giữa các môi trường chỉ là các tham số cấu hình được nạp vào lúc khởi chạy.

---

### PHẦN 4. GIẢI PHÁP: QUẢN LÝ CẤU HÌNH BẰNG BIẾN MÔI TRƯỜNG

Áp dụng triết lý của phương pháp luận **Twelve-Factor App** (tách biệt hoàn toàn cấu hình khỏi mã nguồn), chúng ta sẽ cấu hình ứng dụng Spring Boot đọc thông số kết nối Database động từ biến môi trường (Environment Variables) của hệ điều hành.

#### 4.1 Cấu hình động trong file `application.yml` của `user-service`
Thay vì viết cứng mật khẩu, ta sử dụng cú pháp placeholder của Spring Boot: `${TÊN_BIẾN:Giá_trị_mặc_định}`:

```yaml
spring:
  datasource:
    url: ${QUICKBITE_DB_URL:jdbc:postgresql://localhost:5432/quickbite_user}
    username: ${QUICKBITE_DB_USER:postgres}
    password: ${QUICKBITE_DB_PASS:password123}
```
* **Giải thích:** Khi khởi chạy, Spring Boot sẽ tìm biến môi trường có tên `QUICKBITE_DB_URL`. Nếu tìm thấy, nó sẽ nạp giá trị đó; nếu không tìm thấy, nó sẽ tự động dùng giá trị mặc định phía sau dấu hai chấm (`jdbc:postgresql://localhost:5432/quickbite_user`) để chạy ở máy local của dev.

#### 4.2 Lệnh khởi chạy trên server Staging bằng cách nạp biến môi trường
Khi deploy lên server Staging, người vận hành chỉ cần export các biến môi trường tương ứng của Staging trên terminal của server trước khi chạy ứng dụng:

```bash
# 1. Nạp các cấu hình của môi trường Staging vào hệ điều hành
export QUICKBITE_DB_URL=jdbc:postgresql://10.0.1.20:5432/quickbite_user_staging
export QUICKBITE_DB_USER=staging_admin
export QUICKBITE_DB_PASS=StagingSecurePass@2026

# 2. Khởi chạy file JAR duy nhất
java -jar user-service-0.0.1.jar
```
*Kết quả:* Ứng dụng `user-service` tự động kết nối vào database của Staging mà không cần sửa bất kỳ dòng code nào. Mật khẩu database thật hoàn toàn nằm trên server, không bị lộ trên Git.

---

### PHẦN 5. HIỂU LẦM TAI HẠI

* **Hiểu sai:** Dự án của tôi dùng Private Repository (kho lưu trữ riêng tư) trên GitHub/GitLab, nên tôi có thể tạo các file cấu hình cứng chứa mật khẩu thật (như `application-prod.yml`) rồi commit lên Git, khi chạy chỉ cần gõ `--spring.profiles.active=prod` là xong.
* **Đính chính:** Đây là tư duy bảo mật cực kỳ sai lầm. Như đã phân tích ở Phần 2, kho code private của bạn vẫn có rất nhiều người truy cập được (từ các lập trình viên khác, QA/QC cho đến cộng tác viên và thực tập sinh thời vụ). 
  Hãy luôn ghi nhớ một châm ngôn bảo mật bất di bất dịch: **"Bí mật chỉ thực sự là bí mật khi người giữ bí mật... bị câm."** Việc để mật khẩu Production tồn tại trên Git (dù là private repo) đồng nghĩa với việc bạn tự ý tiết lộ thông tin quan trọng nhất của hệ thống. Mật khẩu Production chỉ được phép tồn tại duy nhất ở runtime trên server Production (dưới dạng các biến môi trường được nạp bảo mật lúc chạy).

---

### PHẦN 6. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao DevOps yêu cầu một file JAR duy nhất của `user-service` phải chạy được trên tất cả các môi trường mà không được biên dịch lại cho từng môi trường?
* *Gợi ý:* Để đảm bảo tính đồng nhất của phiên bản (Consistency). Nếu bạn build lại một file JAR mới cho Production, không ai đảm bảo được trong lúc build lại không có dòng code mới nào bị lọt vào hoặc phát sinh lỗi do môi trường biên dịch khác nhau. File JAR chạy trên Prod phải chính xác là file JAR đã được test thành công trên Staging.

#### Câu 2 (Đọc hiểu cú pháp cấu hình)
Giả sử file `application.yml` cấu hình dòng password như sau: `password: ${QUICKBITE_DB_PASS}` (không có dấu hai chấm và mật khẩu mặc định). Chuyện gì xảy ra nếu lập trình viên chạy lệnh `java -jar user-service-0.0.1.jar` ở máy local mà không export biến `QUICKBITE_DB_PASS`?
* *Gợi ý:* Ứng dụng Spring Boot sẽ bị lỗi khởi chạy lập tức (Application run failed). Spring Boot không thể phân giải (resolve) placeholder `${QUICKBITE_DB_PASS}` do không tìm thấy biến môi trường và cũng không có giá trị mặc định để thay thế, dẫn đến lỗi cấu hình kết nối database.
