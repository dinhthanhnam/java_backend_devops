# SESSION 14: TẠO DASHBOARD VỚI GRAFANA

## LESSON 04: Tạo dashboard giám sát Spring Boot service

---

### PHẦN 1. MỤC TIÊU BÀI HỌC
Sau khi hoàn thành bài học này, học viên có khả năng:
* **Import** thành công Dashboard chuyên dụng cho JVM và Spring Boot (ID 4701 hoặc 11378) vào Grafana.
* **Đọc hiểu** các thông số cơ bản về bộ nhớ JVM (Heap, Non-Heap) và tiến trình dọn rác Garbage Collection (GC).
* **Giám sát** được hiệu năng các yêu cầu HTTP (Throughput, Latency) gửi tới các REST API của Spring Boot.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ
Khi vận hành cụm Microservices Spring Boot trên Production, chúng ta thường gặp phải các lỗi ngầm rất khó phát hiện từ bên ngoài:
* Ứng dụng đột ngột phản hồi chậm chạp mà không có lỗi cụ thể, CPU tăng nhẹ, nghi ngờ do máy ảo Java chạy GC liên tục để giải phóng bộ nhớ.
* Có hiện tượng rò rỉ bộ nhớ (Memory Leak), RAM tăng liên tục qua các ngày và cuối cùng container bị sập đột ngột (OOM Exit 137).
* Lượng request đổ về API tăng đột biến nhưng không rõ API nào đang chịu tải lớn nhất và tốc độ phản hồi trung bình là bao nhiêu mili-giây.

Dữ liệu thô từ Spring Boot Actuator kết hợp với Grafana Dashboard sẽ phơi bày toàn bộ "nội tạng" của máy ảo JVM, giúp chúng ta chủ động tìm ra nguyên nhân gốc rễ của các lỗi hiệu năng này.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI

#### 3.1 Lựa chọn Dashboard JVM quốc dân
* Với dữ liệu đo đạc bằng thư viện Micrometer từ Spring Boot Actuator, mã ID Dashboard phổ biến và đầy đủ nhất là **`4701`** (JVM (Micrometer)) hoặc **`11378`**.
* Dashboard này tự động phân tích hàng trăm chỉ số của JVM để vẽ thành các khu vực riêng biệt: Bộ nhớ JVM, Trạng thái Thread, Tiến trình Garbage Collection, và Thống kê HTTP Requests.

#### 3.2 Các chỉ số JVM quan trọng cần theo dõi

1. **JVM Memory (Bộ nhớ Heap):**
   * Chỉ số: `jvm_memory_used_bytes{area="heap"}` và `jvm_memory_max_bytes{area="heap"}`.
   * Ý nghĩa: Hiển thị lượng RAM thực tế đang bị chiếm dụng bởi các đối tượng Java (Heap Space) so với ngưỡng tối đa được cấp phát (Xmx). Nếu lượng dùng tiến sát Max liên tục, hệ thống sắp bị tràn bộ nhớ (OOM).
2. **Garbage Collection (GC Pauses):**
   * Chỉ số: `rate(jvm_gc_pause_seconds_sum[5m])`.
   * Ý nghĩa: Đo đạc thời gian ứng dụng bị dừng hoàn toàn (Stop-the-world) để dọn rác. Nếu tổng thời gian GC pauses quá cao, ứng dụng sẽ bị giật lag nghiêm trọng.
3. **Trạng thái luồng (Thread States):**
   * Chỉ số: `jvm_threads_states_threads`.
   * Ý nghĩa: Theo dõi số lượng luồng đang chạy (Runnable), đang chờ (Waiting) hoặc bị khóa (Blocked). Nếu số lượng thread Blocked tăng đột biến, hệ thống đang gặp hiện tượng nghẽn cổ chai (Deadlock hoặc nghẽn DB Connection Pool).

#### 3.3 Đo lường Hiệu năng HTTP Request
* **Throughput (Lưu lượng):**
  ```promql
  sum(rate(http_server_requests_seconds_count[5m])) by (uri, status)
  ```
  *Ý nghĩa:* Tính số lượng request HTTP/giây đổ về từng endpoint, phân chia theo mã trạng thái HTTP (200, 404, 500) để biết API nào đang bị lỗi nhiều nhất.
* **Latency (Độ trễ phản hồi trung bình):**
  ```promql
  sum(rate(http_server_requests_seconds_sum[5m])) by (uri) / sum(rate(http_server_requests_seconds_count[5m])) by (uri)
  ```
  *Ý nghĩa:* Lấy tổng thời gian xử lý request chia cho tổng số lượng request để tính ra thời gian xử lý trung bình của mỗi API.

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (IMPORT DASHBOARD JVM MICROMETER)

Học viên thực hiện tích hợp Dashboard giám sát Spring Boot Service.

#### 4.1 Các bước thực hiện
1. Đăng nhập vào Grafana.
2. Chọn **Dashboards** -> **+ Create New** -> **Import**.
3. Nhập ID:
   ```text
   4701
   ```
   *(Hoặc có thể sử dụng ID `11378` để có giao diện hiện đại hơn).*
4. Nhấn **Load**.
5. Đổi tên thành `QuickBite JVM Monitoring` và chọn Data Source là `Prometheus`.
6. Nhấn nút **Import**.
7. Sử dụng bộ lọc **Application** ở góc trên bên trái Dashboard để chọn xem thông số của từng service riêng lẻ (như `order-service` hoặc `user-service`).

```text
┌────────────────────────────────────────────────────────┐
│ JVM (Micrometer) Dashboard                             │
│  ├── JVM Heap Memory (Used vs Max)                     │
│  ├── Thread States (Runnable, Blocked, Waiting)        │
│  └── HTTP Request Latency by URI                       │
└────────────────────────────────────────────────────────┘
```

---

### PHẦN 5. CÂU HỎI TỰ LUẬN ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
* **Câu hỏi:** Tại sao đồ thị JVM Heap Memory (`jvm_memory_used_bytes`) của một ứng dụng Java hoạt động bình thường luôn có dạng hình răng cưa (lên cao rồi đột ngột hạ xuống thấp liên tục) chứ không phải là một đường thẳng?
* **Gợi ý trả lời:**
  * Do cơ chế hoạt động của bộ dọn rác Garbage Collection (GC) trong Java. Khi ứng dụng hoạt động, các đối tượng mới liên tục được khởi tạo trên bộ nhớ Heap khiến đồ thị đi lên.
  * Đến một thời điểm hoặc ngưỡng dung lượng nhất định, bộ dọn rác GC sẽ được kích hoạt (Minor GC) để dọn dẹp các đối tượng không còn được tham chiếu, giải phóng RAM khiến dung lượng sử dụng giảm mạnh đột ngột, kéo đồ thị đi xuống. Chu kỳ này lặp lại liên tục tạo nên dạng hình răng cưa đặc trưng của JVM.

#### Câu 2 (Phân tích lỗi)
* **Câu hỏi:** Nếu bạn quan sát thấy đồ thị Heap Memory liên tục tăng lên mà không hề giảm xuống (răng cưa hướng thẳng lên trên), đồng thời số lượng GC Pause tăng vọt và ứng dụng phản hồi rất chậm, hiện tượng gì đang xảy ra với ứng dụng Spring Boot và DevOps nên làm gì?
* **Gợi ý trả lời:**
  * Ứng dụng đang gặp lỗi **Rò rỉ bộ nhớ (Memory Leak)** nghiêm trọng dẫn đến nguy cơ sập ứng dụng do tràn bộ nhớ (Out Of Memory Error). Lập trình viên đang giữ tham chiếu đến các đối tượng không dùng nữa khiến GC không thể dọn dẹp.
  * Hướng giải quyết:
    1. Tiến hành dump bộ nhớ JVM (Heap Dump) bằng các công cụ như jcmd hoặc jmap trên VPS để phân tích xem Class nào đang chiếm giữ RAM bất thường.
    2. Cấu hình tạm thời tăng RAM tối đa (`-Xmx`) cho container Java trong Docker Compose để duy trì dịch vụ trong lúc fix code.
