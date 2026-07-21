# SESSION 14: TẠO DASHBOARD VỚI GRAFANA

## LESSON 05: Cấu hình cảnh báo cơ bản khi service gặp sự cố

---

### PHẦN 1. MỤC TIÊU BÀI HỌC
Sau khi hoàn thành bài học này, học viên có khả năng:
* **Hiểu rõ** cơ chế hoạt động của hệ thống cảnh báo (Alerting) trong Grafana.
* **Cấu hình** điểm nhận cảnh báo (Contact Point) qua Telegram hoặc Discord Webhook.
* **Thiết lập** quy tắc cảnh báo (Alert Rule) tự động kích hoạt khi có container microservice bị sập hoặc CPU của VPS vượt ngưỡng an toàn.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ
Sở hữu các Dashboard giám sát đẹp mắt mới chỉ giúp chúng ta giải quyết phần ngọn khi có sự cố. Chúng ta không thể phân công kỹ sư ngồi nhìn vào màn hình Grafana 24/7 để chờ xem khi nào biểu đồ chuyển sang màu đỏ.
* Thực tế, khi có một microservice (ví dụ `order-service`) bị sập vào lúc 2 giờ sáng, khách hàng không thể thanh toán được.
* Đến 8 giờ sáng, khi khách hàng phàn nàn và gửi ticket báo lỗi, đội ngũ kỹ thuật mới tá hỏa đi kiểm tra và khởi động lại dịch vụ.

Để hệ thống vận hành tự động và phản ứng tức thì, chúng ta cần cấu hình Grafana tự động giám sát các chỉ số ngầm. Ngay khi phát hiện bất thường, Grafana phải chủ động bắn tin nhắn cảnh báo trực tiếp vào nhóm chat (Telegram/Discord) của đội ngũ DevOps để xử lý ngay lập tức.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI

#### 3.1 Luồng xử lý Cảnh báo (Alerting Workflow) của Grafana
Hệ thống Alerting trong Grafana hoạt động qua 3 bước khép kín:
1. **Alert Rule (Quy tắc):** Định nghĩa điều kiện lỗi dựa trên câu lệnh PromQL (ví dụ: `up == 0` trong 2 phút).
2. **Contact Point (Kênh liên lạc):** Định nghĩa nơi sẽ gửi thông tin cảnh báo (Telegram Bot Token, Discord Webhook URL, Email, Slack...).
3. **Notification Policy (Chính sách):** Khớp các nhãn (Labels) của Alert Rule với Contact Point để phân phối tin nhắn đến đúng nhóm kỹ thuật phụ trách.

#### 3.2 Các trạng thái của một Alert Rule
* **Normal (Hoạt động bình thường):** Chỉ số nằm trong ngưỡng an toàn.
* **Pending (Chờ kích hoạt):** Chỉ số đã vượt ngưỡng lỗi nhưng chưa đủ thời gian kiểm thử (ví dụ: CPU > 90% nhưng mới duy trì được 1 phút, trong khi cấu hình yêu cầu phải kéo dài 5 phút). Trạng thái này giúp lọc bớt các cảnh báo ảo (false positive).
* **Firing (Kích hoạt cảnh báo):** Chỉ số vượt ngưỡng vượt quá thời gian cấu hình. Grafana chính thức phát tin nhắn cảnh báo tới Contact Point.
* **Resolved (Đã khắc phục):** Chỉ số quay lại ngưỡng an toàn, Grafana tự động gửi tin nhắn báo hệ thống đã phục hồi ổn định.

---

### PHẦN 4. DEMO VÀ THỰC HÀNH (THIẾT LẬP CẢNH BÁO TELEGRAM/DISCORD)

Học viên thực hiện cấu hình cảnh báo tự động khi microservice bị sập (`DOWN`).

#### 4.1 Cấu hình Contact Point (Kênh nhận tin nhắn)
Trong bài thực hành này, chúng ta sử dụng **Discord Webhook** (hoặc **Telegram Bot**):
1. Trên Discord, tạo một channel `#system-alerts`, vào cài đặt channel -> **Integrations** -> **Webhooks** -> Tạo Webhook mới và sao chép URL (Webhook URL).
2. Trên Grafana, vào menu bên trái -> **Alerting** -> **Contact points**.
3. Nhấn **+ Add contact point**.
4. Thiết lập cấu hình:
   * **Name:** `Discord-Alerts`
   * **Integration:** Chọn `Discord` (hoặc `Telegram`).
   * **Webhook URL:** Dán URL vừa sao chép từ Discord vào.
5. Nhấn nút **Test** để gửi thử một tin nhắn mẫu. Kiểm tra channel Discord xem có nhận được tin nhắn test của Grafana hay không.
6. Nhấn **Save contact point**.

#### 4.2 Tạo Alert Rule giám sát trạng thái Microservices
Chúng ta thiết lập quy tắc: "Cảnh báo ngay nếu có bất kỳ service nào có trạng thái `DOWN` quá 2 phút".

1. Vào **Alerting** -> **Alert rules** -> Nhấn **+ Create rule**.
2. Thiết lập bước **1. Define query and alert condition**:
   * Chọn Data Source là `Prometheus`.
   * Nhập câu lệnh PromQL kiểm tra trạng thái hoạt động:
     ```promql
     up == 0
     ```
3. Thiết lập bước **2. Set evaluation behavior**:
   * **Folder:** Tạo một thư mục tên là `QuickBite Alerts`.
   * **Evaluation group:** Tạo group mới chạy định kỳ mỗi `1m` (1 phút).
   * **Pending period (For):** Điền `2m` (Nếu service sập liên tục quá 2 phút mới kích hoạt cảnh báo).
4. Thiết lập bước **3. Configure notifications**:
   * Chọn Contact Point mặc định là `Discord-Alerts` đã cấu hình ở trên.
5. Nhấn **Save and exit** ở góc trên bên phải.

#### 4.3 Kiểm chứng cảnh báo (Simulate Incident)
1. Truy cập vào VPS và chủ động tắt thử container `order-service` để tạo sự cố giả lập:
   ```bash
   docker compose stop order-service
   ```
2. Đợi khoảng 1 phút, vào tab **Alert rules** trên Grafana sẽ thấy rule chuyển sang trạng thái màu vàng **Pending**.
3. Tiếp tục đợi thêm 2 phút, rule chuyển sang màu đỏ **Firing**.
4. *Kết quả mong đợi:* Nhóm Discord `#system-alerts` nhận được tin nhắn cảnh báo chi tiết có dạng:
   `[Firing] mock-service-monitor up == 0 ... order-service:8083 is down`.
5. Khởi động lại container:
   ```bash
   docker compose start order-service
   ```
6. Đợi 1-2 phút, Discord sẽ nhận được tin nhắn màu xanh báo trạng thái **[Resolved]**.

---

### PHẦN 5. CÂU HỎI TỰ LUẬN ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
* **Câu hỏi:** Tham số **Pending period (For)** trong cấu hình Alert Rule có vai trò gì? Điều gì xảy ra nếu chúng ta thiết lập thời gian này bằng `0` (cảnh báo lập tức)?
* **Gợi ý trả lời:**
  * Vai trò: Tham số này đóng vai trò là một bộ lọc thời gian trễ để xác nhận xem sự cố có thực sự xảy ra lâu dài hay không, tránh phát các cảnh báo ảo khi chỉ số chỉ bị biến động đột biến trong vài giây (ví dụ CPU vọt lên 95% rồi hạ xuống ngay lập tức, hoặc service khởi động mất 15 giây).
  * Nếu đặt bằng `0`: Hệ thống sẽ bắn cảnh báo lập tức ngay khi có một request quét metrics bị rớt gói tin hoặc trong lúc deploy container đang khởi động lại. Việc này làm hòm thư cảnh báo bị rác (Alert Fatigue), khiến đội ngũ DevOps bị chai lỳ và bỏ qua các cảnh báo thực sự nguy hiểm.

#### Câu 2 (Thiết kế hệ thống)
* **Câu hỏi:** Trong môi trường Production thực tế, tại sao người ta khuyên không nên cấu hình máy chủ Grafana Alerting chạy chung một cụm VPS vật lý với các microservices ứng dụng cần giám sát?
* **Gợi ý trả lời:**
  * Để đảm bảo tính sẵn sàng cao của hệ thống cảnh báo (High Availability). Nếu VPS vật lý bị sập nguồn hoàn toàn, hoặc ổ cứng bị hỏng vật lý, toàn bộ cụm microservices sẽ chết, đồng thời container Grafana Alerting chạy trên đó cũng chết theo.
  * Hậu quả là hệ thống đã sập hoàn toàn nhưng đội ngũ DevOps không nhận được bất kỳ tin nhắn cảnh báo nào vì máy phát cảnh báo đã chết cùng nạn nhân. Do đó, máy chủ giám sát (Prometheus/Grafana) nên được triển khai độc lập trên một VPS riêng biệt so với máy chạy Product Backend.
