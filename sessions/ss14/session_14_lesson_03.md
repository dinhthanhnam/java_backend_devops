# SESSION 14: TẠO DASHBOARD VỚI GRAFANA

## LESSON 03: Tạo dashboard giám sát VPS

---

### PHẦN 1. MỤC TIÊU BÀI HỌC
Sau khi hoàn thành bài học này, học viên có khả năng:
* **Import** (nhập) thành công các Dashboard có sẵn từ cộng đồng Grafana để giám sát hạ tầng.
* **Giải thích** được cấu trúc và các loại Panel hiển thị chính trong Grafana (Graph, Stat, Gauge).
* **Đọc hiểu và giải thích** các câu lệnh PromQL cơ bản dùng để đo đạc CPU, RAM và Disk của máy chủ VPS từ dữ liệu Node Exporter.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ
Khi bắt đầu tự thiết kế một bảng điều khiển (Dashboard) giám sát VPS, DevOps thường gặp bế tắc:
* Không biết nên biểu diễn thông số CPU, RAM, Disk dưới dạng biểu đồ nào cho dễ nhìn (hình tròn, cột hay đồ thị đường thẳng).
* Việc viết các câu lệnh PromQL phức tạp để tính tỷ lệ phần trăm CPU sử dụng thực tế (loại bỏ thời gian CPU rảnh rỗi - idle) rất khó khăn đối với người mới bắt đầu.
* Việc tự vẽ thủ công từng ô chỉ số (Panel) tốn rất nhiều thời gian và công sức.

Để giải quyết vấn đề này, Grafana cung cấp một kho tàng Dashboard chia sẻ từ cộng đồng (Grafana Community Dashboards). Chúng ta chỉ cần import ID của Dashboard mẫu chuẩn thế giới và tùy chỉnh lại cho khớp với hệ thống của mình.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI

#### 3.1 Cơ chế Import Dashboard bằng ID
* Cộng đồng Grafana chia sẻ các thiết kế Dashboard dưới dạng các tệp cấu hình JSON. Mỗi Dashboard được cấp một mã số ID định danh duy nhất trên trang chủ `grafana.com/grafana/dashboards`.
* Dashboard quốc dân để giám sát phần cứng Linux sử dụng Node Exporter có ID nổi tiếng là **`1860`** (Node Exporter Full).
* Bằng cách nhập ID này, Grafana sẽ tự động tải tệp cấu hình JSON về, khởi tạo toàn bộ các đồ thị chuyên nghiệp chỉ trong vài giây.

#### 3.2 Các câu lệnh PromQL cốt lõi giám sát VPS

1. **Tính phần trăm CPU đang sử dụng:**
   ```promql
   100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
   ```
   *Giải thích:* Lấy 100% trừ đi tỷ lệ CPU ở trạng thái rảnh rỗi (`mode="idle"`) trung bình trong 5 phút gần nhất để ra lượng CPU thực sự đang làm việc.
2. **Tính tỷ lệ % bộ nhớ RAM đã sử dụng:**
   ```promql
   (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100
   ```
   *Giải thích:* Lấy tổng RAM trừ đi RAM khả dụng để ra lượng RAM đã dùng, sau đó chia cho tổng dung lượng RAM để tính tỷ lệ phần trăm.
3. **Tính phần trăm ổ đĩa cứng SSD đã sử dụng (phân vùng `/`):**
   ```promql
   100 - ((node_filesystem_avail_bytes{mountpoint="/"} * 100) / node_filesystem_size_bytes{mountpoint="/"})
   ```

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (IMPORT DASHBOARD NODE EXPORTER FULL)

Học viên thực hiện tích hợp Dashboard giám sát VPS vào hệ thống.

#### 4.1 Các bước Import Dashboard
1. Truy cập giao diện Grafana.
2. Nhấn vào biểu tượng **Dashboards** (ô vuông) ở menu bên trái -> Chọn **+ Create New** -> Chọn **Import**.
3. Tại ô **Import via grafana.com**, nhập mã ID:
   ```text
   1860
   ```
4. Nhấn nút **Load**.
5. Giao diện cấu hình Dashboard xuất hiện:
   * **Name:** Có thể đổi tên thành `QuickBite VPS Monitoring`.
   * **Folder:** Chọn `General`.
   * **Prometheus:** Chọn nguồn dữ liệu `Prometheus` đã kết nối ở Lesson 02.
6. Nhấn nút **Import**.
7. *Kết quả mong đợi:* Hệ thống lập tức chuyển hướng sang một giao diện Dashboard cực kỳ chuyên nghiệp hiển thị chi tiết biểu đồ CPU, RAM, dung lượng đĩa, tốc độ đọc/ghi đĩa SSD và băng thông card mạng của VPS.

```text
┌────────────────────────────────────────────────────────┐
│ Import Dashboard (ID: 1860)                            │
│  ├── CPU Usage (Graph / Line Chart)                    │
│  ├── Memory Usage (Gauge / Circular)                   │
│  └── Disk Usage (Stat / Color indicator)               │
└────────────────────────────────────────────────────────┘
```

---

### PHẦN 5. CÂU HỎI TỰ LUẬN ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
* **Câu hỏi:** Trong câu lệnh PromQL tính dung lượng đĩa SSD đã sử dụng ở Phần 3, tại sao chúng ta bắt buộc phải có điều kiện lọc `{mountpoint="/"}`? Nếu bỏ bộ lọc này đi thì biểu đồ sẽ hiển thị sai lệch như thế nào?
* **Gợi ý trả lời:**
  * Hệ điều hành Linux quản lý đĩa cứng theo các điểm gắn kết (mount points). Điểm gắn kết `/` đại diện cho ổ đĩa cứng thực tế chính chứa hệ điều hành và mã nguồn dự án của VPS.
  * Nếu bỏ bộ lọc `{mountpoint="/"}` đi, Prometheus sẽ tính toán cả các hệ thống tệp ảo của nhân Linux (như `/sys`, `/proc`, `/boot`) hoặc các mount point tạm thời của Docker, dẫn đến việc Grafana vẽ ra nhiều đường đĩa cứng ảo không có giá trị thực tế, gây rối loạn thông tin giám sát.

#### Câu 2 (Phân tích)
* **Câu hỏi:** Phân biệt mục đích sử dụng của 3 loại Panel hiển thị chính trên Dashboard: **Graph (Time-series)**, **Stat**, và **Gauge**. Cho ví dụ cụ thể nên áp dụng loại nào cho chỉ số nào của VPS.
* **Gợi ý trả lời:**
  * **Graph (Time-series):** Dùng để biểu diễn sự biến động của dữ liệu theo thời gian dưới dạng biểu đồ đường. Phù hợp cho chỉ số CPU Load hoặc Network Traffic để theo dõi xu hướng cao điểm.
  * **Stat:** Hiển thị một con số cụ thể kèm màu nền cảnh báo (ví dụ xanh/đỏ). Phù hợp hiển thị thời gian máy chủ đã chạy liên tục (Uptime) hoặc trạng thái tắt/mở của node.
  * **Gauge:** Biểu diễn dữ liệu dưới dạng đồng hồ đo (hình tròn hoặc thanh trượt) hiển thị mức độ phần trăm. Phù hợp nhất cho việc hiển thị dung lượng RAM hoặc Disk đã sử dụng hiện tại (ví dụ RAM đang dùng ở mức 80%).
