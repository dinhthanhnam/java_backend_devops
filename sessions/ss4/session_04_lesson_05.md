# SESSION 04: DOCKER COMPOSE CƠ BẢN

## LESSON 05: Quản lý vòng đời hệ thống với Docker Compose

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Làm chủ** toàn diện các câu lệnh quản lý vòng đời cụm container thông qua Docker Compose CLI (`up`, `down`, `stop`, `start`).
* **Thực hiện chẩn đoán lỗi** nhanh chóng bằng cách kiểm tra danh sách container (`ps`) và stream log tổng hợp (`logs -f`) của toàn bộ hệ thống.
* **Tương tác trực tiếp** với các container trong cụm bằng câu lệnh thực thi dòng lệnh (`exec`).
* **Phân biệt** được phạm vi ảnh hưởng của việc xóa tài nguyên (`down`) và dừng tài nguyên (`stop`).

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (MA TRẬN LỖI KHI KHỞI CHẠY ĐỒNG THỜI)

Hãy tưởng tượng bạn vừa thực hiện lệnh khởi chạy toàn bộ hệ thống QuickBite bằng Docker Compose:
```bash
docker compose up -d
```
Màn hình terminal in ra trạng thái cả `quickbite-db` và `quickbite-user` đều báo `Started` màu xanh lá cây. 

Tuy nhiên, khi bạn mở trình duyệt hoặc Postman gửi API request tới cổng `8081` của `quickbite-user`, hệ thống báo lỗi `500 Internal Server Error` hoặc lỗi không phản hồi. Bạn rơi vào ma trận hoang mang:
* *Database đã thực sự sẵn sàng nhận kết nối chưa?*
* *Hay database đang chạy lỗi khởi động nên sập?*
* *Hay mã nguồn Java của backend bị ném ngoại lệ (Exception) lúc kết nối?*

Nếu làm theo cách cũ, bạn phải mở 2 terminal mới chạy 2 lệnh `docker logs` riêng biệt cho từng container để so khớp dòng thời gian lỗi. Việc này cực kỳ mất thời gian và khiến bạn bối rối không biết nguyên nhân gốc rễ bắt đầu từ đâu.

*Để chẩn đoán nhanh và vận hành hệ thống trơn tru, bạn cần làm chủ bộ câu lệnh quản lý vòng đời và chẩn đoán của Docker Compose CLI.*

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (CÁC LỆNH ĐIỀU KHIỂN DOCKER COMPOSE CLI CHỦ CHỐT)

Docker Compose CLI cung cấp các công cụ tương tác trực tiếp với toàn bộ "stack" (ngăn xếp) container của bạn:

```text
                 [ File cấu hình: docker-compose.yml ]
                                   │
               ┌───────────────────┼───────────────────┐
               ▼                   ▼                   ▼
      [ Khởi tạo & Chạy ]     [ Giám sát & Check ]    [ Dừng & Dọn dẹp ]
      - compose up            - compose ps            - compose stop
      - compose start         - compose logs          - compose down
```

#### 3.1 Nhóm lệnh khởi tạo và dừng
* `docker compose up -d`: Khởi chạy toàn bộ hệ thống chạy ngầm dưới nền (Detached mode). Lệnh này sẽ tự động tạo mạng, volume, build image (nếu cấu hình build) và khởi chạy container.
* `docker compose stop`: Tạm dừng hoạt động của các container (trạng thái container chuyển thành `Exited`), nhưng **không xóa bỏ** container. Dữ liệu ghi tạm trong container vẫn được giữ nguyên.
* `docker compose start`: Khởi động lại các container đã bị dừng bởi lệnh `stop`.
* `docker compose down`: Dừng và **xóa bỏ hoàn toàn** các container và mạng ảo của dự án. Lệnh này giúp dọn sạch RAM và CPU của máy.

#### 3.2 Nhóm lệnh chẩn đoán và tương tác
* `docker compose ps`: Liệt kê danh sách các container thuộc dự án hiện hành kèm theo ID, trạng thái hoạt động (Up/Exit), và cổng ánh xạ mạng.
* `docker compose logs -f`: Stream log tổng hợp của **tất cả** các container trong cụm thời gian thực.
  * *Điểm ưu việt:* Docker Compose sẽ tự động phân tách log của từng dịch vụ bằng các màu sắc khác nhau và chèn thêm tiền tố tên dịch vụ ở đầu dòng (ví dụ: `quickbite-user-1  |`, `quickbite-db-1    |`). Điều này giúp bạn dễ dàng theo dõi trình tự lỗi diễn ra giữa các dịch vụ.
* `docker compose exec [tên_service] [lệnh]`: Chạy một lệnh trực tiếp bên trong một container đang hoạt động của cụm (tương tự `docker exec`).

---

### PHẦN 4. THỰC HÀNH: CHẨN ĐOÁN VÀ ĐIỀU KHIỂN CỤM CONTAINER QUICKBITE

Hãy thực hành quy trình chẩn đoán lỗi tiêu chuẩn của một kỹ sư DevOps:

1. Khởi chạy hệ thống ở chế độ chạy ngầm:
```bash
docker compose up -d
```
2. Kiểm tra danh sách trạng thái của cụm container:
```bash
docker compose ps
```
   * **Kết quả mong đợi:** Màn hình hiển thị bảng danh sách các container, đảm bảo cột `STATUS` hiển thị `Up` cho cả database và backend.
3. Stream log tổng hợp để giám sát quá trình khởi động:
```bash
docker compose logs -f --tail=50
```
   * Quan sát cách các dòng log đan xen nhau. Nhấn `Ctrl + C` để thoát chế độ xem log (lưu ý: việc thoát logs không làm dừng container).
4. Kiểm tra xem database PostgreSQL đã thực sự sẵn sàng nhận kết nối hay chưa bằng công cụ chẩn đoán nội bộ của Postgres:
```bash
docker compose exec quickbite-db pg_isready -U postgres
```
   * **Kết quả mong đợi:** Console trả về thông báo: `/var/run/postgresql:5432 - accepting connections`.
5. Dọn dẹp sạch sẽ tài nguyên hệ thống sau khi làm việc xong:
```bash
docker compose down
```

---

### PHẦN 5. HIỂU LẦM THƯỜNG GẶP (DOCKER COMPOSE DOWN VS STOP)

* **Hiểu sai:** Lệnh `docker compose down` sẽ xóa sạch toàn bộ dữ liệu của tôi trong database.
* **Sự thật:** Không hề.
  * Lệnh `docker compose down` chỉ xóa container và network ảo. Dữ liệu nằm trong **Named Volumes** vật lý trên máy host vẫn được giữ an toàn 100%. Khi bạn gõ `docker compose up -d` trở lại, dữ liệu database cũ sẽ tự động xuất hiện.
  * **Trường hợp mất dữ liệu thật:** Chỉ xảy ra khi bạn cố tình truyền thêm cờ xóa volume: `docker compose down -v` (hoặc `--volumes`). Cờ `-v` sẽ ép Docker xóa sạch cả Named Volumes lưu trữ. Hãy cực kỳ cẩn thận với cờ này trên môi trường Staging/Production!

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG (XÁC MINH KIẾN THỨC)

1. **Tài liệu tra cứu lệnh Docker Compose CLI:**
   * [Docker Compose CLI Command Reference - Docker Docs](https://docs.docker.com/reference/cli/docker/compose/)
2. **Chi tiết về lệnh docker compose down:**
   * [Docker Compose Down command reference - Docker Docs](https://docs.docker.com/reference/cli/docker/compose/down/)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Khi chạy lệnh `docker compose logs -f`, nếu bạn muốn chỉ xem log của duy nhất container backend `quickbite-user` thì gõ lệnh thế nào?
* *Gợi ý:* Bạn chỉ cần truyền thêm tên của service đó vào sau câu lệnh: `docker compose logs -f quickbite-user`. Docker Compose sẽ lọc và chỉ stream log của riêng service này.

#### Câu 2 (Xử lý tình huống)
Tech Lead yêu cầu bạn kiểm thử xem database `quickbite-db` có thể phản hồi truy vấn SQL trực tiếp hay không mà không cần mở cổng database ra ngoài máy host. Bạn làm thế nào?
* *Gợi ý:* Bạn sử dụng lệnh `docker compose exec` để truy cập vào trình quản trị psql trực tiếp bên trong container đang chạy:
  `docker compose exec quickbite-db psql -U postgres -c "SELECT 1;"`
  Lệnh này sẽ thực thi truy vấn SQL nội bộ bên trong container và trả trực tiếp kết quả về terminal máy host của bạn mà không cần expose cổng mạng ra bên ngoài.
