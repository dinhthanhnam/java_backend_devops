# SESSION 14: TẠO DASHBOARD VỚI GRAFANA

## LESSON 02: Kết nối Grafana với Prometheus

---

### PHẦN 1. MỤC TIÊU BÀI HỌC
Sau khi hoàn thành bài học này, học viên có khả năng:
* **Cấu hình kết nối** thành công Prometheus làm nguồn dữ liệu (Data Source) bên trong giao diện quản trị của Grafana.
* **Giải thích** được cơ chế phân giải tên miền nội bộ của Docker Network khi kết nối hai container này.
* **Sử dụng** chức năng "Explore" của Grafana để viết và chạy thử nghiệm các câu lệnh truy vấn PromQL thô.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ
Sau khi cài đặt xong Grafana ở Lesson 01, chúng ta mới chỉ có một giao diện hiển thị rỗng, chưa có bất kỳ dữ liệu nào. 
Khi cố gắng cấu hình kết nối từ Grafana sang Prometheus:
* Nhiều học viên điền địa chỉ URL kết nối là `http://localhost:9090`. Kết quả là Grafana báo lỗi `Connection refused` (Từ chối kết nối).
* Một số học viên dùng IP Public của VPS `http://<vps_public_ip>:9090` nhưng cũng bị lỗi vì trước đó tường lửa UFW đã chặn cổng 9090 không cho bên ngoài truy cập.

Để kết nối chính xác và bảo mật, chúng ta cần tận dụng mạng ảo nội bộ của Docker. Bài học này sẽ hướng dẫn chi tiết cách thiết lập kết nối chuẩn Production giữa hai dịch vụ này.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI

#### 3.1 Docker Network DNS resolution
Do container Grafana (`quickbite-grafana`) và container Prometheus (`quickbite-prometheus`) chạy chung mạng ảo `quickbite-net`, chúng có khả năng giao tiếp với nhau bằng tên container thay vì IP vật lý:
* Nếu chúng ta điền `localhost` hoặc `127.0.0.1` vào cấu hình Grafana, container Grafana sẽ tự hiểu là đang gọi đến chính nó (cổng 9090 của container Grafana, vốn không tồn tại dịch vụ nào).
* Địa chỉ kết nối chính xác phải là:
  ```text
  http://quickbite-prometheus:9090
  ```
  Nhân Docker DNS sẽ tự dịch chuyển tên `quickbite-prometheus` thành IP nội bộ tương ứng trong mạng ảo để kết nối an toàn.

#### 3.2 Khái niệm Data Source trong Grafana
* **Data Source:** Là các bộ kết nối (connectors) tích hợp sẵn trong Grafana để giao tiếp với các hệ thống cơ sở dữ liệu bên ngoài.
* Với Prometheus, Grafana sẽ dùng giao thức HTTP REST API để gửi các câu lệnh PromQL, nhận về dữ liệu dạng JSON chuỗi thời gian và vẽ biểu đồ.

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (KẾT NỐI PROMETHEUS DATA SOURCE)

Học viên thực hiện kết nối Prometheus vào Grafana và chạy thử truy vấn thô.

#### 4.1 Các bước cấu hình trên giao diện Grafana
1. Đăng nhập vào Grafana (`http://<vps_public_ip>:3000`).
2. Ở thanh menu bên trái, nhấn vào biểu tượng bánh răng **Connections** -> Chọn **Data sources**.
3. Nhấn nút **Add data source**.
4. Chọn nguồn dữ liệu là **Prometheus** từ danh sách hiển thị.
5. Cấu hình các thông số cơ bản:
   * **Name:** `Prometheus` (Để mặc định làm nguồn dữ liệu chính - Default).
   * **Connection HTTP URL:** `http://quickbite-prometheus:9090`
6. Kéo xuống cuối trang và nhấn nút **Save & test**.
   * *Kết quả mong đợi:* Xuất hiện thông báo màu xanh lá cây: **"Data source is working"** (Nguồn dữ liệu đang hoạt động tốt).

```text
┌────────────────────────────────────────────────────────┐
│ Connection                                             │
│ URL: [ http://quickbite-prometheus:9090              ] │
└────────────────────────────────────────────────────────┘
```

#### 4.2 Thử nghiệm truy vấn PromQL trong tab Explore
1. Nhấn vào biểu tượng la bàn **Explore** trên thanh công cụ bên trái.
2. Đảm bảo nguồn dữ liệu được chọn ở góc trên bên trái là **Prometheus**.
3. **Chuyển sang chế độ gõ câu lệnh (Code Mode):**
   * Mặc định Grafana (từ phiên bản 9+) sẽ ở chế độ **Builder** (chọn qua selectbox).
   * Nhấn nút **Code** ở góc trên bên phải thanh công cụ truy vấn (nằm bên cạnh nút *Builder*) để chuyển sang chế độ tự do gõ PromQL.
4. Nhập một câu lệnh PromQL của Backend Java (đã cấu hình quét từ Session 13):
   * Để xem dung lượng RAM mà các ứng dụng Java Spring Boot đang sử dụng, gõ:
     ```promql
     jvm_memory_used_bytes
     ```
   * Hoặc chọn các metric khác có sẵn như: `up`, `process_cpu_usage`, `http_server_requests_seconds_count`.
5. Nhấn nút **Run query** (hoặc tổ hợp phím `Shift + Enter`).
6. *Kết quả mong đợi:* Biểu đồ chuỗi thời gian hiển thị biến động RAM của các ứng dụng Java (user-service, order-service,...) được vẽ thành công ngay phía dưới.

---

### PHẦN 5. CÂU HỎI TỰ LUẬN ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
* **Câu hỏi:** Giải thích tại sao cấu hình URL kết nối đến Prometheus trong Grafana là `http://localhost:9090` lại dẫn đến lỗi kết nối, mặc dù cả Prometheus và Grafana đều đang chạy trên cùng một máy chủ VPS vật lý?
* **Gợi ý trả lời:**
  * Bởi vì hai dịch vụ này đang chạy dưới dạng các container Docker riêng biệt. Theo cơ chế cô lập mạng của container, từ khóa `localhost` hoặc địa chỉ `127.0.0.1` viết bên trong container nào thì sẽ chỉ trỏ đến chính container đó (Loopback Interface của container).
  * Khi cấu hình URL là `http://localhost:9090` trong giao diện Grafana, container Grafana sẽ tự gửi request đến cổng 9090 của chính nó chứ không thể đi xuyên qua container Prometheus, dẫn đến lỗi kết nối.

#### Câu 2 (Phân tích lỗi)
* **Câu hỏi:** Giả sử khi nhấn nút **Save & test**, Grafana trả về lỗi `HTTP Error Bad Gateway (502)` hoặc `Context Deadline Exceeded`. Hãy đề xuất các bước kiểm tra xử lý sự cố.
* **Gợi ý trả lời:**
  * Bước 1: Kiểm tra xem container Prometheus có đang chạy thực tế hay không bằng lệnh `docker compose ps`.
  * Bước 2: Kiểm tra xem cả hai container `quickbite-grafana` và `quickbite-prometheus` có được đặt chung trong một mạng ảo Docker network không (đọc log docker compose hoặc dùng lệnh `docker network inspect <network_name>`).
  * Bước 3: Kiểm tra xem tên container Prometheus khai báo trong URL cấu hình có khớp chính xác với `container_name` trong tệp `docker-compose.yml` hay không (ở đây là `quickbite-prometheus`).
