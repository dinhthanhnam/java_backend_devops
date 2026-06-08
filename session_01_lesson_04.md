# SESSION 01: TỔNG QUAN DEVOPS & QUY TRÌNH CI/CD

## LESSON 04: Kiến trúc triển khai hệ thống Microservices và luồng đi của dữ liệu

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau bài học này, bạn sẽ có khả năng:
* **Phác thảo** được sơ đồ kiến trúc triển khai tổng thể của hệ thống QuickBite ở trạng thái mục tiêu (gồm Nginx, API Gateway, các Microservices và Database).
* **Giải thích** được vai trò và sự khác biệt giữa lớp Nginx Reverse Proxy và API Gateway trong mô hình triển khai.
* **Phân tích** được luồng đi bảo mật của dữ liệu từ thiết bị Client đến cơ sở dữ liệu nội bộ.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (SỰ NGUY HIỂM KHI GIAO TIẾP TRỰC TIẾP)

Khi phát triển và chạy thử dịch vụ `user-service` ở local, ứng dụng client (Web/Mobile) gọi trực tiếp đến IP và cổng của ứng dụng đó (ví dụ: `http://localhost:8080` hoặc IP của server Staging `http://10.0.1.15:8080`).

Cách làm trực diện này chỉ hoạt động khi bạn có duy nhất một dịch vụ đơn lẻ. Nhưng khi dự án QuickBite phát triển lên, chúng ta sẽ có nhiều dịch vụ chạy đồng thời. Nếu bắt client phải tự nhớ và gọi trực tiếp IP của từng dịch vụ, chúng ta sẽ lập tức đối mặt với những thảm họa thiết kế sau:
1. **Lỗ hổng bảo mật chết người:** Việc phơi bày trực tiếp địa chỉ IP và cổng dịch vụ của backend ra internet sẽ giúp hacker dễ dàng quét cổng, phát hiện lỗi hệ điều hành và tấn công trực tiếp vào máy chủ chạy code.
2. **Quản lý HTTPS phức tạp:** Việc cấu hình chứng chỉ bảo mật SSL (HTTPS) cho hàng chục cổng mạng khác nhau của từng dịch vụ đơn lẻ cực kỳ tốn thời gian và khó quản lý thời hạn chứng chỉ.
3. **Mất tính linh hoạt:** Mỗi lần máy chủ backend thay đổi địa chỉ IP hoặc chuyển dịch cổng chạy (ví dụ từ `8080` sang `8081`), lập trình viên sẽ phải sửa code của ứng dụng Mobile/Web phía khách hàng và bắt người dùng cập nhật phiên bản mới thì mới kết nối lại được.

---

### PHẦN 3. KIẾN TRÚC TRIỂN KHAI TIÊU CHUẨN CỦA QUICKBITE (FINAL STATE)

Để giải quyết triệt để các vấn đề trên, kiến trúc triển khai Microservices của QuickBite được thiết kế chia lớp chặt chẽ để bảo vệ dòng dữ liệu:

```text
                               [ Client (Web / Mobile) ]
                                           │
                                           ▼ (HTTPS: Port 443 - Public Internet)
                               ┌───────────────────────┐
                               │  Nginx Reverse Proxy  │ 
                               └─────┬───────────┬─────┘
         (web.quickbite.com)         │           │         (api.quickbite.com)
         ┌───────────────────────────┘           └───────────────────────────┐
         ▼                                                                   ▼
┌──────────────────┐                                               ┌────────────────────────┐
│  Frontend Static │                                               │       API Gateway      │
│  (HTML/CSS/JS)   │                                               └────────────┬───────────┘
└──────────────────┘                                                            │ (Private Internal Network)
                                                              ┌─────────────────┼──────────────────┐
                                                              ▼                 ▼                  ▼
                                                        ┌──────────────┐   ┌──────────────┐   ┌──────────┐
                                                        │ user-service │   │ order-service│   │   ...    │
                                                        └──────┬───────┘   └────┬─────────┘   └────┬─────┘
                                                               │                │                  │
                                                               └────────────────┼──────────────────┘
                                                                                ▼
                                                                         [PostgreSQL DB]
```

> **Lưu ý:** Trong các Session đầu tiên (Session 1 & 2), chúng ta chỉ tập trung hoàn toàn vào việc triển khai mảnh ghép cơ bản nhất là **`user-service`** và **PostgreSQL DB**. Các thành phần Nginx, API Gateway, Frontend và các service khác sẽ lần lượt được tích hợp ở các Session sau để hoàn thiện sơ đồ này.

#### 3.1 Lớp biên giới (Edge Layer): Nginx và API Gateway
Đây là hai chốt chặn duy nhất mở cửa đón dữ liệu từ internet vào hệ thống:
* **Nginx (Reverse Proxy & Web Server):** Là tấm khiên đứng ở rìa ngoài cùng của hệ thống. Nó tiếp nhận request trực tiếp từ internet qua cổng tiêu chuẩn `80` (HTTP) hoặc `443` (HTTPS). 
  * *Nhiệm vụ:* Nginx sẽ kiểm tra tên miền truy cập để điều hướng:
    - **Điều hướng Frontend (`web.quickbite.com`):** Nginx đóng vai trò là một Web Server lưu trữ các file tĩnh (HTML, CSS, JS) của ứng dụng Frontend. Khi người dùng truy cập giao diện, Nginx ngay lập tức trả về các file này cực kỳ nhanh chóng mà không cần đi qua bất kỳ logic backend nào. *(Lưu ý: Môn học này chúng ta tập trung thực chiến DevOps, nên phần Frontend này chỉ mang tính giới thiệu kiến trúc, thực tế chúng ta không triển khai code FE).*
    - **Điều hướng API Backend (`api.quickbite.com`):** Nginx giải mã HTTPS (SSL Termination) và chuyển tiếp (proxy) request đó đến API Gateway ở phía sau.
* **API Gateway (Spring Cloud Gateway):** Đứng ngay sau Nginx. 
  * *Nhiệm vụ:* Gateway đọc đường dẫn (Path) của request để định tuyến đến đúng service bên trong (ví dụ: request có path bắt đầu bằng `/api/v1/users` sẽ được định tuyến về `user-service`). Đồng thời, Gateway xử lý các bộ lọc bảo mật chung như xác thực người dùng (Authentication) trước khi cho phép request đi sâu vào hệ thống.

#### 3.2 Lớp dịch vụ nội bộ (Internal Service Layer)
* Các dịch vụ nghiệp vụ như `user-service` (và các service phát triển thêm sau này) nằm hoàn toàn trong **mạng nội bộ (Private Network)**. 
* Các server này "mù" với thế giới bên ngoài: internet không thể kết nối trực tiếp đến chúng và ngược lại. Chúng chỉ giao tiếp với API Gateway và giao tiếp nội bộ với nhau.

#### 3.3 Lớp dữ liệu (Persistence Layer)
* Cơ sở dữ liệu PostgreSQL cũng nằm sâu trong vùng an toàn của mạng nội bộ.
* *Nguyên lý Database-per-service:* Về mặt thiết kế logic, các service không được phép truy cập chéo database của nhau. Tuy nhiên, về mặt vật lý, để tiết kiệm tài nguyên hạ tầng, các database độc lập này có thể chạy chung trên một cụm máy chủ PostgreSQL duy nhất (chia làm các cơ sở dữ liệu riêng biệt).

---

### PHẦN 4. ĐIỂM CẦN NHẤN MẠNH VÀ HIỂU LẦM THƯỜNG GẶP

* **Điểm cần nhấn mạnh:** Kiến trúc triển khai Microservices yêu cầu tư duy phân rã hạ tầng. Mỗi dịch vụ phải có khả năng khởi chạy, dừng lại hoặc nâng cấp một cách hoàn toàn độc lập mà không làm ảnh hưởng hay kéo sập các dịch vụ còn lại trong hệ thống.
* **Hiềm lầm thường gặp:** Nghĩ rằng mỗi microservice bắt buộc phải sở hữu một máy chủ cơ sở dữ liệu vật lý riêng biệt hoàn toàn.
* **Đính chính:** Việc tách biệt database là yêu cầu về mặt logic (Database-per-service) để tránh tranh chấp dữ liệu. Ở quy mô vừa và nhỏ, chúng ta hoàn toàn có thể chạy chung các database này trên một máy chủ PostgreSQL duy nhất để tối ưu hóa chi phí vận hành.

---

### PHẦN 5. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao hệ thống của chúng ta cần cả **Nginx** ở ngoài cùng và **API Gateway** ở phía sau mà không gộp chung làm một? Hãy nêu nhiệm vụ đặc trưng của từng thành phần.
* *Gợi ý:* Nginx và API Gateway đảm nhận nhiệm vụ ở các tầng khác nhau. Nginx chuyên xử lý các tác vụ ở tầng hạ tầng mạng và tối ưu hóa hiệu năng (Reverse Proxy, SSL Termination, phân tải tĩnh). API Gateway (như Spring Cloud Gateway) chạy ở tầng ứng dụng Java, chuyên xử lý các logic nghiệp vụ phần mềm (định tuyến động theo đường dẫn ứng dụng, bộ lọc xác thực Auth Filter, Rate Limiting tầng ứng dụng). Sự kết hợp này mang lại cả hiệu năng cực cao và tính linh hoạt cho hệ thống.

#### Câu 2 (Đọc sơ đồ và phán đoán)
Dựa vào sơ đồ kiến trúc triển khai của QuickBite, những thành phần nào được phép mở cổng công khai tiếp nhận kết nối từ Internet (Public Network) và những thành phần nào bắt buộc phải nằm ẩn hoàn toàn trong mạng nội bộ (Private Network)?
* *Gợi ý:* Chỉ có **Nginx** được phép mở cổng tiếp nhận kết nối trực tiếp từ Internet (cổng 80/443). Tất cả các thành phần còn lại như **API Gateway**, **`user-service`** và **PostgreSQL Database** bắt buộc phải nằm trong vùng mạng nội bộ để đảm bảo an toàn, tránh bị hacker scan cổng và tấn công trực diện.
