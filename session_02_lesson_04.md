# SESSION 02: GIỚI THIỆU DOCKER

## LESSON 04: Các lệnh Docker cơ bản trong vòng đời container

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Sử dụng thành thạo** các lệnh điều khiển vòng đời container: Khởi chạy, dừng, khởi động lại và xóa bỏ.
* **Cấu hình chính xác** cổng mạng (`-p`), chạy ngầm (`-d`), đặt tên container (`--name`) và truyền biến môi trường (`-e`).
* **Phân biệt** được trạng thái dừng hoạt động (Stopped) và xóa bỏ hoàn toàn (Destroyed) của container.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (KỊCH BẢN "ĂN HÀNH" CỦA INTERN KHI CHẠY DATABASE)

Hãy tưởng tượng một tình huống thực tế tại dự án QuickBite:
Anh Tech Lead giao cho một bạn Intern một nhiệm vụ tưởng chừng cực kỳ đơn giản: *"Chạy một container PostgreSQL để làm database tạm thời, sao cho ứng dụng `user-service` đang chạy ở máy host kết nối vào được để ghi dữ liệu"*.

Bạn Intern hí hửng nghĩ thầm: *"Nhiệm vụ này quá dễ! Chỉ cần khởi chạy container lên là xong"*. 
Tuy nhiên, ngay sau khi gõ lệnh run mặc định, bạn Intern lập tức rơi vào hai rắc rối:

1. **Bị khóa cứng Terminal:** Log khởi động của Postgres chiếm trọn màn hình và khóa chặt shell session. Do không thể gõ thêm lệnh nào khác, bạn Intern bấm nhầm tổ hợp phím `Ctrl+C` để thoát ra, và thế là... container database cũng bị tắt ngóm ngay lập tức.
2. **Không thể kết nối:** Sau khi mở cửa sổ Terminal mới và chạy lại thành công, database báo đã khởi động xong. Tuy nhiên, ứng dụng `user-service` ở máy host cố gắng kết nối vào cổng `5432` thì liên tục báo lỗi kết nối thất bại (`Connection Refused`). 

*Để giải cứu bạn Intern, chúng ta cần nắm vững các tham số cờ (flags) để vừa chạy ngầm container vừa mở cổng kết nối Card mạng ra bên ngoài.*

---

### PHẦN 3. KIẾN THỨC CỐT LÕI

#### 3.1 Bốn tham số sống còn khi chạy Container
Khi thực hiện lệnh `docker run`, bạn cần làm chủ 4 tham số quan trọng sau:
* **`-d` (Detached mode):** Chạy container dưới nền (background). Giải phóng Terminal để bạn tiếp tục nhập các câu lệnh khác. Container sẽ chạy độc lập với trạng thái đóng/mở của cửa sổ Terminal.
* **`-p host_port:container_port` (Port Mapping):** Ánh xạ cổng mạng của máy host vào cổng mạng nội bộ của container. Ví dụ: `-p 5432:5432` giúp chuyển hướng mọi yêu cầu truy cập cổng `5432` của máy host vào cổng `5432` của PostgreSQL trong container.
* **`-e KEY=VALUE` (Environment Variable):** Truyền biến môi trường vào bên trong container. Các tiến trình (như Database Postgres) sẽ đọc các biến này để cấu hình thông số hoạt động khi khởi chạy (ví dụ: `-e POSTGRES_PASSWORD=secret` để đặt mật khẩu quản trị).
* **`--name [tên_định_danh]`:** Đặt tên tường minh cho container (ví dụ: `quickbite-db`) để dễ dàng quản lý thay vì để Docker tự sinh tên ngẫu nhiên.

#### 3.2 Vòng đời của Container
Tiến trình của container trải qua 4 trạng thái chính:
```text
[Created] (Đã tạo) ──► [Running] (Đang chạy) ──► [Stopped] (Đã dừng) ──► [Destroyed] (Đã xóa)
```
* **Created (Đã tạo):** Container được khởi tạo từ Image, các lớp cấu hình đã sẵn sàng nhưng tiến trình chính bên trong chưa thực sự chạy.
* **Running (Đang hoạt động):** Tiến trình chính của ứng dụng bên trong container được kích hoạt (ví dụ: PostgreSQL đang lắng nghe cổng mạng). Container tiêu thụ tài nguyên RAM/CPU của máy host.
* **Stopped (Exited):** Tiến trình bên trong container đã tắt. Container giải phóng hoàn toàn CPU và RAM, nhưng dữ liệu ghi tạm thời và file cấu hình vẫn được lưu trên ổ đĩa của máy host.
* **Destroyed:** Container bị xóa bỏ hoàn toàn khỏi hệ thống, giải phóng bộ nhớ ổ đĩa.

---

### PHẦN 4. KHỐI LỆNH THỰC HÀNH CỐT LÕI

Hãy mở Terminal và thực hiện lần lượt các lệnh quản lý container `quickbite-db` (dùng image PostgreSQL tối giản `alpine`):

```bash
# 1. Khởi chạy container PostgreSQL chạy ngầm, mở cổng 5432, đặt tên rõ ràng
docker run -d --name quickbite-db -p 5432:5432 -e POSTGRES_PASSWORD=secret postgres:15-alpine

# 2. Liệt kê các container đang hoạt động để kiểm tra trạng thái
docker ps
# Output hiển thị rõ trạng thái Up và thông số Port mapping.

# 3. Dừng container database
docker stop quickbite-db

# 4. Liệt kê tất cả các container (bao gồm cả container đã dừng)
docker ps -a
# Status lúc này hiển thị Exited (0).

# 5. Khởi động lại container cũ đã dừng mà không cần tạo mới
docker start quickbite-db

# 6. Xóa bỏ hoàn toàn container khỏi hệ thống
# Lưu ý: Phải stop container trước khi rm, hoặc dùng cờ -f để xóa cưỡng chế
docker stop quickbite-db
docker rm quickbite-db
```

---

### PHẦN 5. HIỂU LẦM THƯỜNG GẶP

* **Hiểu sai:** Lệnh `docker stop` sẽ xóa sạch dữ liệu và container.
* **Đính chính:** Lệnh `docker stop` chỉ tạm dừng tiến trình hệ thống giống như việc bạn tắt một ứng dụng trên máy. Toàn bộ dữ liệu ghi trên lớp Writable Layer của container (ví dụ: các bảng dữ liệu database đã tạo) vẫn nằm an toàn trên ổ đĩa cho đến khi bạn chạy lệnh `docker rm`.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG

* [Docker Run Reference - Docker Docs](https://docs.docker.com/reference/cli/docker/container/run/)
* [Docker Container CLI commands - Docker Docs](https://docs.docker.com/reference/cli/docker/container/)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1
Nếu khởi chạy container bằng lệnh: `docker run -d --name quickbite-db postgres:15-alpine` (quên cờ `-p 5432:5432`), ứng dụng Spring Boot chạy trực tiếp từ máy local có kết nối vào Database được không? Tại sao?
* *Gợi ý:* Không kết nối được. Thiếu `-p`, Docker sẽ không ánh xạ cổng mạng từ máy host vào container. Ứng dụng ngoài máy host không thể nhìn thấy cổng `5432` của database.

#### Câu 2
Làm thế nào để xóa một container đang ở trạng thái chạy (`Up`) mà không cần chạy lệnh `docker stop` trước? Lệnh đó có rủi ro gì?
* *Gợi ý:* Sử dụng lệnh xóa cưỡng chế: `docker rm -f [tên_container]`. Rủi ro của lệnh này là có thể gây mất mát dữ liệu hoặc hỏng file do tiến trình bên trong container bị tắt đột ngột (tương đương `kill -9`) khi đang ghi dữ liệu.
