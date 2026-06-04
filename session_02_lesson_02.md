# SESSION 02: GIỚI THIỆU DOCKER

## LESSON 02: Docker image, container và registry

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Định nghĩa và làm rõ** mối quan hệ mật thiết giữa ba khái niệm nền tảng: Docker Image, Docker Container và Docker Registry.
* **Giải thích** được cơ chế phân lớp (Layered Architecture) và tính bất biến (Immutability) của Docker Image.
* **Mô tả** được quy trình vòng đời của một ứng dụng từ lúc đóng gói thành Image đến khi phân phối và khởi chạy thực tế.
* **Tra cứu** và kiểm chứng thông tin qua các nguồn tài liệu chính thống của Docker.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (LÀM SAO ĐỂ CHUYỂN GIAO MÔI TRƯỜNG MÀ KHÔNG BỊ LỆCH PHA?)

Ở Lesson 1, chúng ta đã hiểu lý do vì sao Container ra đời để giải quyết bài toán cô lập môi trường chạy (như chạy song song `user-service` Java 17 và `order-service` Java 21) mà không làm lãng phí RAM của server như Máy ảo (VM). 

Nhưng một câu hỏi thực tế lớn hơn xuất hiện: **Làm thế nào để chúng ta đóng gói toàn bộ môi trường chạy phức tạp đó và chuyển giao nó từ máy cá nhân lên server Staging/Production một cách đồng nhất?**

Nếu chúng ta vẫn làm theo cách cũ ở Session 1:
1. Bạn sửa code `user-service` ở máy local.
2. Bạn compile ra file JAR rồi dùng `scp` đẩy file JAR trần trụi lên server.
3. Trên server Staging, bạn lại phải tự cài đặt JDK 17, tự cấu hình các biến môi trường múi giờ bằng tay.

* **Nỗi đau chưa chấm dứt:** Quy trình này vẫn để lộ lỗ hổng lớn về **lệch pha môi trường (Environment Drift)**. Chỉ cần phiên bản Java trên server Staging bị cấu hình sai, hoặc múi giờ hệ thống của server lệch so với máy của bạn, ứng dụng sẽ crash ngay lập tức. Câu nói kinh điển *"Ơ kìa, code chạy ngon trên máy em mà!"* (It works on my machine!) lại tiếp tục vang lên.

*Để giải quyết triệt để vấn đề chuyển giao, Docker không chỉ chạy các tiến trình cô lập, mà nó cung cấp một giải pháp đóng gói toàn diện. Chúng ta cần một công cụ để **đóng băng** (freeze) toàn bộ môi trường và phân phối nó tự động. Đó chính là lúc bạn cần làm quen với bộ ba thực thể: **Docker Image** (Bản đóng băng), **Docker Container** (Tiến trình giải nén chạy thực tế), và **Docker Registry** (Nhà kho phân phối).*

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (IMAGE, CONTAINER & REGISTRY)

Hệ sinh thái Docker xoay quanh ba khái niệm cốt lõi đại diện cho ba giai đoạn của vòng đời ứng dụng:

```text
  [ Docker Registry (Kho lưu trữ) ] ── Pull (Tải về) ─► [ Docker Image (Bản thiết kế) ]
                                                                 │
                                                            Run (Khởi chạy)
                                                                 │
                                                                 ▼
                                                    [ Docker Container (Thực thể) ]
```

#### 3.1 Docker Image (Bản thiết kế bất biến - Tương đương Class trong OOP)
* **Định nghĩa:** Docker Image là một khuôn mẫu đóng gói ở trạng thái chỉ đọc (Read-only). Nó chứa toàn bộ mã nguồn `user-service`, runtime Java (JRE), các biến môi trường cấu hình mặc định, và hệ điều hành Linux tối giản cần thiết để chạy ứng dụng.
* **Tính chất cốt lõi:**
  - **Bất biến (Immutable):** Một khi Image đã được build thành công, không ai có thể sửa đổi nội dung bên trong nó. Muốn sửa cấu hình mặc định hoặc code, bạn bắt buộc phải build một Image mới.
  - **Cấu trúc phân lớp (Layers):** Docker Image được cấu tạo từ nhiều lớp file system xếp chồng lên nhau. 
    - Ví dụ: Lớp đáy là OS Ubuntu tối giản, lớp tiếp theo là JDK 17, lớp trên cùng là code `user-service.jar`. 
    - Cơ chế này giúp tái sử dụng tài nguyên cực kỳ tốt. Nếu bạn build dịch vụ `order-service` chạy Java 17, Docker sẽ tái sử dụng lại các layer OS và JDK 17 có sẵn ở máy, chỉ tải thêm layer chứa file JAR mới của `order-service`.

#### 3.2 Docker Container (Thực thể sống động - Tương đương Object trong OOP)
* **Định nghĩa:** Docker Container là một thực thể sống (Instance) được khởi tạo từ Docker Image. Nó chính là tiến trình ứng dụng Java thực sự đang chạy trên RAM và CPU của máy chủ.
* **Cơ chế hoạt động:** Khi bạn ra lệnh chạy một container từ Image, Docker Engine sẽ lấy Image đó (trạng thái chỉ đọc) và phủ lên trên cùng một lớp ghi tạm thời gọi là **Writable Layer (hoặc Container Layer)**.
* **Tính chất:**
  - Mọi thao tác tạo file, ghi log, ghi cấu hình khi container đang chạy đều ghi vào lớp Writable Layer này.
  - Nếu bạn xóa Container, lớp Writable Layer này sẽ bị hủy bỏ hoàn toàn. Image gốc nằm bên dưới vẫn toàn vẹn và sạch sẽ.
  - Từ **một** Image duy nhất, bạn có thể khởi chạy **hàng chục** Container `user-service` chạy song song trên các cổng mạng khác nhau.

#### 3.3 Docker Registry (Nhà kho chứa hàng)
* **Định nghĩa:** Là kho lưu trữ tập trung dùng để quản lý và chia sẻ các Docker Image.
* **Các loại Registry:**
  - **Public Registry:** Kho công cộng như **Docker Hub** (mặc định), nơi chứa hàng triệu image chính thức của các công nghệ (PostgreSQL, OpenJDK, Nginx, v.v.).
  - **Private Registry:** Kho lưu trữ riêng tư của doanh nghiệp (như GitLab Container Registry, AWS ECR) để lưu trữ bảo mật mã nguồn đóng của dự án QuickBite.

---

### PHẦN 4. SƠ ĐỒ VÒNG ĐỜI PHÁT HÀNH ỨNG DỤNG KHÉP KÍN

Quy trình phát hành ứng dụng chuẩn DevOps bằng Docker diễn ra như sau:

```text
 1. Local Dev (Máy cá nhân)         2. Docker Hub / Registry            3. Staging/Prod Server
 ┌──────────────────────┐           ┌──────────────────────┐           ┌──────────────────────┐
 │ Viết code Java       │           │                      │           │                      │
 │      ▼               │           │                      │           │                      │
 │ Build Docker Image   │─Push───►  │ Lưu trữ Image        │─Pull───►  │ Tải Image về         │
 │ (Chứa code + JDK 17) │           │ (user-service:v1)    │           │ (user-service:v1)    │
 └──────────────────────┘           └──────────────────────┘           │      ▼               │
                                                                       │ Chạy Container       │
                                                                       │ (Java 17 chuẩn hóa)  │
                                                                       └──────────────────────┘
```

#### Các lệnh thực hành minh họa vòng đời mẫu:
Để tải một bản phân phối Java JRE chính thức từ Docker Hub về máy cá nhân, bạn mở Terminal và chạy:

```bash
# 1. Tải (pull) image JRE 17 gọn nhẹ (dùng alpine linux) từ Docker Hub
docker pull eclipse-temurin:17-jre-alpine

# 2. Xem danh sách các Image đang lưu ở bộ nhớ máy local
docker images
# Output mong đợi: Hiển thị dòng eclipse-temurin với tag 17-jre-alpine, kích thước chỉ khoảng ~100MB

# 3. Khởi chạy thử một container kiểm tra phiên bản Java bên trong
docker run --rm eclipse-temurin:17-jre-alpine java -version
# Output mong đợi: Hiển thị phiên bản java "17.x.x" thành công và tự động xóa container khi chạy xong (--rm)
```

---

### PHẦN 5. HIỂU LẦM THƯỜNG GẶP & ĐIỂM CẦN NHẤN MẠNH

* **Hiểu lầm thường gặp:** Khi đang chạy container `user-service`, nếu tôi chui vào trong container sửa đổi một file cấu hình hoặc tạo thêm một thư mục, thì Docker Image sinh ra container đó cũng tự động được cập nhật theo.
* **Sự thật:** Docker Image là **bất biến (Immutable)**. Mọi sửa đổi của bạn chỉ tồn tại trên lớp Writable Layer của riêng container đó. Khi container bị xóa (`docker rm`), toàn bộ file sửa đổi sẽ biến mất mãi mãi. Để lưu giữ các thay đổi đó, bạn bắt buộc phải sửa file cấu hình ở ngoài máy host rồi build lại một Image mới.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG (XÁC MINH KIẾN THỨC)

Để kiểm chứng các thông tin kỹ thuật và đi sâu tìm hiểu cơ chế hoạt động của Docker, bạn hãy truy cập các tài liệu chính thức từ Docker Documentation:
1. **Kiến trúc và các khái niệm nền tảng của Docker:**
   * [Docker Architecture and Concepts - Docker Docs](https://docs.docker.com/get-started/docker-concepts/the-basics/)
2. **Cách hoạt động của hệ thống file phân lớp và Writable Layer:**
   * [How Images and Containers work with storage drivers - Docker Docs](https://docs.docker.com/storage/storagedriver/)
3. **Hướng dẫn sử dụng Docker Registry và Docker Hub:**
   * [Docker Registry Official Guide - Docker Docs](https://docs.docker.com/dependencies/)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Dựa trên kiến trúc Class và Object của lập trình hướng đối tượng (OOP), hãy so sánh tương quan giữa Docker Image và Docker Container.
* *Gợi ý:* Docker Image tương tự như một Class (Lớp) trong OOP - nó là bản thiết kế tĩnh, định nghĩa sẵn cấu trúc, biến môi trường và runtime hệ thống. Docker Container tương tự như một Object (Đối tượng) - nó là thực thể sống động được khởi tạo trong bộ nhớ từ bản thiết kế đó, có thể chạy, dừng và tương tác thực tế. Từ một Class (Image) ta có thể khởi tạo hàng ngàn Object (Container) độc lập.

#### Câu 2 (Đọc hiểu và dự đoán)
Một lập trình viên khởi chạy một container PostgreSQL làm database cho `user-service`, sau đó ghi vào đó 100 thông tin khách hàng mới. Nếu lập trình viên chạy lệnh xóa container này đi (`docker rm`) và chạy lệnh tạo một container PostgreSQL mới từ image ban đầu, 100 thông tin khách hàng đó có còn tồn tại không? Tại sao?
* *Gợi ý:* 100 thông tin khách hàng đó sẽ bị mất hoàn toàn. Bởi vì dữ liệu database được ghi vào lớp Writable Layer tạm thời của container đó. Khi container bị xóa, lớp này bị xóa theo. Để lưu trữ dữ liệu bền vững, chúng ta cần sử dụng cơ chế liên kết dữ liệu với máy host (sẽ học ở bài học về Docker Volume).

#### Câu 3 (Xử lý tình huống thực tế)
Dịch vụ `user-service` deploy lên server Staging gặp sự cố hiển thị sai múi giờ của log (bị chậm mất 7 tiếng do server Staging dùng giờ UTC, còn máy local của lập trình viên dùng giờ Việt Nam GMT+7). Theo bạn, lập trình viên nên xử lý lỗi này bằng cách cấu hình múi giờ trực tiếp trên OS của server Staging, hay cấu hình thiết lập múi giờ ngay trong quá trình build Docker Image? Tại sao?
* *Gợi ý:* Nên cấu hình múi giờ ngay trong quá trình build Docker Image (hoặc thông qua biến môi trường khi chạy container). Vì điều này đảm bảo tính "Build once, run anywhere" của Docker. Nếu cấu hình trên OS của server, khi chuyển sang deploy ở server Production khác ta lại phải cấu hình lại bằng tay, vi phạm nguyên lý tự động hóa và dễ xảy ra sai sót.
