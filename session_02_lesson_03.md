# SESSION 02: GIỚI THIỆU DOCKER

## LESSON 03: Cài đặt Docker và kiểm tra môi trường

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Giải thích** được kiến trúc Client-Server của Docker Engine (Docker CLI, REST API và Docker Daemon).
* **Triển khai cài đặt** thành công Docker Desktop trên Windows/macOS và Docker Engine gốc trên Linux Ubuntu đúng quy chuẩn kỹ thuật.
* **Sử dụng** các lệnh kiểm tra trạng thái (`docker version`, `docker info`) và khởi chạy container thử nghiệm đầu tiên (`docker run hello-world`).
* **Khắc phục** được lỗi kết nối cơ bản giữa Docker Client và Docker Daemon.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (THIẾT LẬP MÔI TRƯỜNG DOCKER ĐỒNG BỘ)

Ở Lesson 2, chúng ta đã nắm được lý thuyết nền tảng về bộ ba Docker Image, Container và Registry. Nhưng mọi lý thuyết sẽ là vô nghĩa nếu bạn không thể tự mình gõ những dòng lệnh đầu tiên để kiểm tra môi trường.

Để bắt tay vào đóng gói `user-service` và `order-service` của hệ thống QuickBite, việc đầu tiên bạn cần làm là cài đặt Docker lên máy tính cá nhân. Lúc này, bạn sẽ đối mặt với các vấn đề:
* Làm thế nào để cài đặt Docker đúng chuẩn?
* Tại sao giao diện dòng lệnh của Docker trên Windows, macOS và Linux lại giống hệt nhau, nhưng cơ chế hoạt động bên dưới của chúng lại rất khác biệt?
* Nếu bạn dùng Windows và cài đặt Docker Desktop sai cách (không bật ảo hóa WSL 2), Docker sẽ chạy cực kỳ chậm, ngốn RAM kinh hoàng và khiến máy tính của bạn bị đơ cứng.

> [!TIP]
> **Image Prompt gợi ý:**
> A neat technical infographic diagram. In the center is the Docker Engine. On the left: a Windows laptop icon showing "WSL2 enabled". In the middle: a macOS laptop icon showing "Apple Silicon native setup". On the right: a Linux Rack Server icon showing "Native Docker Engine". Arrows connect all of them to a central, glowing blue whale logo of Docker. Clean, modern, flat vector art.

*Bài học này sẽ hướng dẫn bạn làm chủ kiến trúc Docker Engine và thiết lập môi trường Sandbox chuẩn xác trên máy tính cá nhân.*

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (KIẾN TRÚC DOCKER ENGINE & QUY TRÌNH CÀI ĐẶT)

#### 3.1 Kiến trúc Client - Server của Docker Engine
Docker Engine hoạt động theo mô hình Client - Server (Khách - Chủ) gồm 3 thành phần chính:

```text
 ┌────────────────────────┐         REST API         ┌────────────────────────┐
 │   Docker CLI (Client)  │ ───────────────────────► │ Docker Daemon (Server) │
 │ (Người dùng gõ lệnh)   │ ◄─────────────────────── │ (dockerd - Chạy ngầm)  │
 └────────────────────────┘                          └────────────────────────┘
```

1. **Docker CLI (Client):** Là công cụ dòng lệnh (executable `docker`) giúp người dùng tương tác. Khi bạn gõ `docker run` hay `docker build`, lệnh này không trực tiếp chạy container mà nó chỉ đóng vai trò gửi yêu cầu đi.
2. **REST API:** Cầu nối định nghĩa các chuẩn interface để Docker CLI có thể giao tiếp và gửi chỉ thị cho Server.
3. **Docker Daemon (Server - tên tiến trình là `dockerd`):** Đây mới là **"trái tim"** thực sự của Docker. Nó là một dịch vụ chạy ngầm trên máy chủ, chịu trách nhiệm lắng nghe yêu cầu từ API và trực tiếp thực hiện việc quản lý, tải Image, khởi tạo Container, chia mạng (Network) và gắn ổ đĩa (Volume).

#### 3.2 Hướng dẫn thiết lập môi trường trên các hệ điều hành

##### A. Trên Windows (Sử dụng Docker Desktop)
* **Yêu cầu bắt buộc:** Bật tính năng ảo hóa phần cứng (Hardware Virtualization) trong BIOS của máy tính.
* **Cơ chế:** Vì Docker yêu cầu nhân Linux để chạy container, Docker Desktop trên Windows sử dụng **WSL 2 (Windows Subsystem for Linux)** làm lớp ảo hóa nhân Linux siêu nhẹ (chỉ tốn vài trăm MB RAM thay vì dùng các máy ảo nặng nề).
* **Các bước:** Tải file cài đặt từ trang chủ, tích hợp tùy chọn sử dụng WSL 2 trong quá trình cài đặt.

##### B. Trên macOS (Sử dụng Docker Desktop)
* **Lưu ý đặc biệt:** macOS chia thành 2 dòng chip: Intel và Apple Silicon (M1, M2, M3). Bạn phải tải đúng phiên bản cài đặt tương thích với chip máy Mac của mình.
* **Cơ chế:** macOS không có nhân Linux, Docker Desktop sẽ tự động chạy một máy ảo Linux siêu nhỏ ở chế độ ẩn để làm môi trường chạy cho các container.

##### C. Trên Linux Ubuntu (Sử dụng Docker Engine gốc)
* **Lưu ý thực chiến:** Trên server chạy thật (Production/Staging), chúng ta **không cài Docker Desktop** vì nó có giao diện đồ họa nặng nề. Chúng ta chỉ cài đặt bản phân phối **Docker Engine gốc** dạng CLI để tiết kiệm tối đa RAM và CPU.
* **Lệnh cài đặt nhanh qua Repository chính thức:**
```bash
# Cập nhật và cài đặt thư viện hỗ trợ HTTPS
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg

# Thêm GPG key chính thức của Docker
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Cấu hình apt repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Cài đặt Docker Engine
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```

---

### PHẦN 4. THỰC HÀNH: KIỂM TRA MÔI TRƯỜNG & CONTAINER ĐẦU TIÊN

Sau khi cài đặt xong, bạn mở Terminal lên và chạy chuỗi lệnh chẩn đoán sau:

#### Lệnh 1: Kiểm tra kết nối Client - Daemon
```bash
docker version
```
* **Kết quả mong đợi:** In ra thông tin phiên bản của cả **Client** (Dòng lệnh) và **Server** (Docker Daemon). Điều này chứng tỏ Client đã giao tiếp thành công với Daemon qua REST API.

#### Lệnh 2: Kiểm tra tài nguyên hệ thống Docker
```bash
docker info
```
* **Kết quả mong đợi:** Hiển thị chi tiết số lượng container đang chạy, số lượng image đang có ở local, Driver lưu trữ (Storage Driver), và tài nguyên CPU/RAM mà Docker đang được phép sử dụng từ máy host.

#### Lệnh 3: Khởi chạy container đầu tiên
```bash
docker run hello-world
```
* **Luồng chạy nội bộ của lệnh:**
  1. Docker Client gửi yêu cầu chạy image `hello-world` tới Docker Daemon.
  2. Docker Daemon kiểm tra ở bộ nhớ local xem đã có image `hello-world` chưa.
  3. Vì là máy mới tinh chưa có image, Daemon tự động kết nối lên **Docker Hub** để tải (pull) image `hello-world` về máy.
  4. Daemon giải nén image và khởi chạy container. Container in ra dòng chữ chào mừng và tự động dừng lại.

---

### PHẦN 5. HIỂU LẦM THƯỜNG GẶP & LƯU Ý BẢO MẬT (DOCKER CLI VS DAEMON)

* **Hiểu lầm:** Tôi gõ lệnh chạy container trong cửa sổ Terminal. Khi tôi tắt cửa sổ Terminal đó đi hoặc tắt máy Terminal, container đang chạy ngầm cũng sẽ bị dừng lại theo.
* **Sự thật:** Hoàn toàn sai. Docker CLI chỉ là một "khách gửi thư". Khi bạn chạy container (nhất là ở chế độ chạy ngầm `-d`), yêu cầu đã được gửi đến **Docker Daemon (Server)** chạy độc lập dưới nền của hệ thống. Container sẽ tiếp tục chạy bình thường cho dù bạn tắt Terminal hay tắt kết nối SSH, trừ khi bạn chủ động tắt tiến trình Docker Daemon hoặc tắt máy tính vật lý.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG (XÁC MINH KIẾN THỨC)

Để kiểm chứng quy trình cài đặt chuẩn và kiến trúc của Docker Engine, bạn có thể tham khảo trực tiếp các liên kết chính thức sau từ Docker Docs:
1. **Kiến trúc chi tiết của Docker Engine:**
   * [Docker Engine Architecture - Docker Docs](https://docs.docker.com/get-started/overview/#docker-architecture)
2. **Hướng dẫn cài đặt Docker Desktop trên Windows (WSL 2):**
   * [Docker Desktop for Windows Install Guide - Docker Docs](https://docs.docker.com/desktop/install/windows-install/)
3. **Hướng dẫn cài đặt Docker Desktop trên macOS:**
   * [Docker Desktop for Mac Install Guide - Docker Docs](https://docs.docker.com/desktop/install/mac-install/)
4. **Hướng dẫn cài đặt Docker Engine gốc trên Ubuntu Server:**
   * [Install Docker Engine on Ubuntu - Docker Docs](https://docs.docker.com/engine/install/ubuntu/)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Khi gõ lệnh `docker ps` trên Terminal, bạn nhận được thông báo lỗi sau:
`Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?`
Hãy giải thích nguyên nhân gây ra lỗi này và xác định thành phần nào trong kiến trúc Docker Engine đang gặp sự cố.
* *Gợi ý:* Lỗi này chứng tỏ Docker Client (`docker` CLI) vẫn hoạt động bình thường, nhưng nó không thể kết nối tới Docker Daemon (`dockerd` - Server) thông qua socket nội bộ. Nguyên nhân là do dịch vụ Docker Daemon đang bị tắt (chưa được start) hoặc bạn chạy lệnh mà không có quyền quản trị (cần thêm `sudo` trên Linux).

#### Câu 2 (Đọc hiểu và dự đoán)
Giải thích tại sao khi chạy lệnh `docker run hello-world` lần thứ hai, màn hình console không còn xuất hiện dòng chữ:
`Unable to find image 'hello-world:latest' locally`?
* *Gợi ý:* Vì ở lần chạy đầu tiên, Docker Daemon đã tải image `hello-world` từ Docker Hub về lưu trữ cục bộ ở ổ đĩa của máy host. Ở lần chạy thứ hai, Daemon phát hiện image đã có sẵn ở local nên lập tức khởi chạy container luôn mà không cần kết nối mạng để pull lại nữa.

#### Câu 3 (Xử lý tình huống thực tế)
Khi bạn được giao nhiệm vụ triển khai dịch vụ `user-service` lên một server Ubuntu Linux trên môi trường Cloud (như AWS EC2), bạn nên lựa chọn cài đặt bản **Docker Desktop** hay bản **Docker Engine** gốc? Tại sao?
* *Gợi ý:* Bạn bắt buộc phải chọn bản **Docker Engine** gốc. Bởi vì server Cloud chạy hệ điều hành Linux dạng minimal (chỉ có giao diện CLI dòng lệnh, không có giao diện đồ họa GUI). Docker Desktop yêu cầu tài nguyên lớn để duy trì giao diện đồ họa và môi trường máy ảo, việc cài đặt nó lên Cloud Server sẽ gây lãng phí tài nguyên CPU/RAM vô ích và làm giảm hiệu năng chạy ứng dụng.
