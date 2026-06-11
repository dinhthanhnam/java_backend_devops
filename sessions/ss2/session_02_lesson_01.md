# SESSION 02: GIỚI THIỆU DOCKER

## LESSON 01: Khái niệm container và so sánh Docker với máy ảo

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Phân tích** được sự khác biệt về mặt kiến trúc phần cứng và hiệu năng giữa Container (Docker) và Máy ảo (Virtual Machine - VM).
* **Giải thích** được cơ chế chia sẻ nhân (Shared Kernel) của Docker giúp tối ưu hóa tài nguyên phần cứng cho hệ thống.
* **Mô tả** được vai trò của hai tính năng nhân Linux (`Namespaces` và `Cgroups`) trong việc cô lập tiến trình của container.
* **Tra cứu** và kiểm chứng kiến trúc qua các nguồn tài liệu chính thống của Docker và Linux.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (XUNG ĐỘT RUNTIME GIỮA CÁC MICROSERVICES)

Hãy tưởng tượng bạn đang quản lý môi trường chạy thử (Staging) cho hệ thống QuickBite trên một VPS. Lúc này, hệ thống đã phát triển lên 2 dịch vụ độc lập:
1. **`user-service` (Dịch vụ quản lý người dùng):** Được phát triển từ trước, đang chạy ổn định trên **Java 17** (do vướng một số thư viện Spring Security và JWT cũ chưa tương thích với bản mới).
2. **`restaurant-service` (Dịch vụ nhà hàng):** Được phát triển sau, sử dụng các tính năng mới của **Java 21** (như Virtual Threads để tối ưu hiệu năng).

Nhiệm vụ của bạn là phải triển khai đồng thời cả 2 dịch vụ này lên cùng một máy chủ VPS. Bạn sẽ lập tức đối mặt với hai lựa chọn đầy "đau đớn":

* **Phương án 1: Triển khai trực tiếp trên hệ điều hành máy chủ (Native Run)**
  * Bạn phải cài đặt cùng lúc JDK 17 và JDK 21 trên cùng một server.
  * Bạn sẽ phải cấu hình thủ công các biến môi trường `JAVA_HOME` riêng biệt cho từng tiến trình, gọi dịch vụ bằng các đường dẫn tuyệt đối phức tạp. Chỉ cần một đợt cập nhật hệ điều hành hoặc cấu hình nhầm biến môi trường, một trong hai dịch vụ sẽ bị lỗi crash phiên bản Class (`UnsupportedClassVersionError`).
* **Phương án 2: Sử dụng Máy ảo (Virtual Machine - VM)**
  * Để cô lập hoàn toàn môi trường, bạn tạo ra 2 máy ảo riêng biệt (VM 1 chạy JDK 17 để chạy `user-service`, VM 2 chạy JDK 21 để chạy `restaurant-service`).
  * Việc này giải quyết được xung đột Java, nhưng cực kỳ lãng phí: Bạn phải mất thêm khoảng 2GB RAM chỉ để chạy 2 Hệ điều hành khách (Guest OS) nền. Máy chủ VPS của bạn sẽ nhanh chóng cạn kiệt tài nguyên.

*Công nghệ **Container (Docker)** ra đời để giải quyết triệt để bài toán này: Cho phép cô lập hoàn toàn môi trường runtime Java của từng dịch vụ mà không hề gây lãng phí tài nguyên hệ thống.*

---

### PHẦN 3. HÌNH ẢNH ẨN DỤ DỄ HIỂU: BIỆT THỰ ĐƠN LẬP VS CĂN HỘ CHUNG CƯ

Để dễ hình dung nhất về sự khác biệt giữa Máy ảo (VM) và Container (Docker), hãy so sánh chúng với hai mô hình nhà ở thực tế:

```text
       MÔ HÌNH MÁY ẢO (VM)                 MÔ HÌNH DOCKER CONTAINER
  (Biệt thự đơn lập riêng móng)          (Căn hộ chung cư chung móng)
        
     ┌───────────────────────┐              ┌───────────────────────┐
     │     user-service      │              │     user-service      │
     │       (Java 17)       │              │       (Java 17)       │
     ├───────────────────────┤              ├───────────────────────┤
     │       Guest OS        │              │  restaurant-service   │
     │    (Hệ điều hành 1)   │              │       (Java 21)       │
     ├───────────────────────┤              ├───────────────────────┤
     │       Móng riêng      │              │                       │
     └───────────────────────┘              │     Móng dùng chung   │
     ┌───────────────────────┐              │    (Shared Kernel /   │
     │  restaurant-service   │              │        Host OS)       │
     │       (Java 21)       │              │                       │
     ├───────────────────────┤              │                       │
     │       Guest OS        │              │                       │
     │    (Hệ điều hành 2)   │              │                       │
     ├───────────────────────┤              │                       │
     │       Móng riêng      │              │                       │
     └───────────────────────┘              └───────────────────────┘
```

#### 3.1 Máy ảo (VM) giống như "Biệt thự đơn lập"
* Mỗi biệt thự được xây dựng trên một **nền móng riêng biệt**, có hệ thống điện nước, tường bao và mái nhà độc lập (tương ứng với **Guest OS**).
* **Ứng dụng:** Bạn có thể thiết kế nhà thứ nhất (`user-service`) theo phong cách cổ điển (JDK 17) và nhà thứ hai (`restaurant-service`) theo phong cách hiện đại (JDK 21) mà không sợ ảnh hưởng đến nhau.
* **Nhược điểm:** Chi phí xây dựng cực kỳ đắt đỏ, tốn diện tích đất (tốn RAM và dung lượng ổ cứng cho Guest OS).

#### 3.2 Container (Docker) giống như "Căn hộ chung cư"
* Các căn hộ nằm chung trong một tòa nhà, **chia sẻ chung nền móng, khung cột chịu lực và đường ống nước chính** của tòa nhà (tương ứng với **Shared Kernel - dùng chung nhân Host OS**).
* **Ứng dụng:** Bên trong căn hộ của `user-service`, bạn trang trí nội thất kiểu cổ điển (chứa JDK 17), căn hộ của `restaurant-service` trang trí kiểu hiện đại (chứa JDK 21). Môi trường runtime bên trong mỗi căn hộ hoàn toàn độc lập và không hề xung đột với nhau.
* **Ưu điểm:** Xây dựng cực nhanh, chi phí rất rẻ, tiết kiệm diện tích tối đa (tiêu hao CPU/RAM tối giản vì không cần xây móng riêng).

---

### PHẦN 4. KIẾN THỨC CỐT LÕI (MÁY ẢO VS CONTAINER)

#### 4.1 Bảng so sánh kỹ thuật trực diện

| Tiêu chí | Máy ảo (Virtual Machine - VM) | Container (Docker Container) |
| :--- | :--- | :--- |
| **Cơ chế ảo hóa** | Ảo hóa ở **tầng phần cứng** (Phần mềm Hypervisor tạo ra CPU ảo, RAM ảo). | Ảo hóa ở **tầng hệ điều hành** (Chạy trực tiếp trên OS máy chủ thông qua Docker Engine). |
| **Hệ điều hành** | Mỗi VM mang theo một **Guest OS** đầy đủ nặng hàng GB. | **Không có Guest OS**, dùng chung nhân (Kernel) của Host OS. |
| **Môi trường Java** | Mỗi VM bắt buộc phải cấu hình và chạy JDK riêng trên hệ điều hành khách đó. | Mỗi Container đóng gói kèm một phiên bản JDK riêng biệt (chỉ chạy trong container đó). |
| **Tốc độ boot** | **Chậm (hàng phút)** vì phải chờ Guest OS khởi động lại từ đầu. | **Gần như tức thì (mili-giây)** vì chỉ khởi động tiến trình dịch vụ. |
| **Sử dụng tài nguyên** | **Khóa cứng tài nguyên tĩnh** (Cấp 2GB RAM là mất luôn 2GB RAM cho máy chủ). | **Tiêu hao tài nguyên động** theo nhu cầu thực tế của dịch vụ. |

#### 4.2 Sơ đồ kiến trúc phần cứng chi tiết

Dưới đây là sơ đồ so sánh trực diện kiến trúc phần cứng của Máy ảo và Docker Container:

```text
        MÔ HÌNH MÁY ẢO (VM)                 MÔ HÌNH DOCKER CONTAINER
  ┌───────────────────────────────┐   ┌───────────────────────────────┐
  │ user-service  │restaurant-svc │   │ user-service  │restaurant-svc │
  │   (Java 17)   │   (Java 21)   │   │   (Java 17)   │   (Java 21)   │
  ├─────────────┬─┴───────────────┤   ├─────────────┬─┴───────────────┤
  │ Thư viện    │ Thư viện        │   │ Thư viện    │ Thư viện        │
  ├─────────────┼─────────────────┤   ├───────────────────────────────┤
  │ Guest OS 1  │ Guest OS 2      │   │         Docker Engine         │
  ├─────────────┴─────────────────┤   ├───────────────────────────────┤
  │          Hypervisor           │   │            Host OS            │
  ├───────────────────────────────┤   ├───────────────────────────────┤
  │       Infrastructure          │   │       Infrastructure          │
  └───────────────────────────────┘   └───────────────────────────────┘
```

> [!IMPORTANT]
> **Điểm mấu chốt cần nhớ:**
> Trong mô hình Container, lớp **Guest OS** và **Hypervisor** nặng nề đã được thay thế hoàn toàn bằng **Docker Engine** gọn nhẹ. Các container chạy trực tiếp trên nhân hệ điều hành của Host OS giúp đạt hiệu năng tiệm cận dịch vụ chạy trực tiếp (Native).

#### 4.3 Bản chất kỹ thuật của Container: Namespaces & Cgroups
Bạn cần lưu ý một điểm mấu chốt: **Container thực chất chỉ là một tiến trình (Process) Linux bình thường**, nhưng được nhân Linux bao bọc bởi 2 công cụ cô lập:

1. **Namespaces (Phân vùng cô lập):**
   * *Giống như việc lắp cửa khóa và tường ngăn giữa các căn hộ chung cư.*
   * Namespaces tạo ra một chiếc "kính thực tế ảo" khiến tiến trình container tin rằng nó đang chạy độc lập trên một hệ điều hành riêng. 
   * Ví dụ: Nhờ `NET Namespace`, container chứa `user-service` và container chứa `restaurant-service` đều có thể tự do cấu hình lắng nghe tại cổng nội bộ `8080` mà không hề gây xung đột cổng mạng trên máy host.
2. **Cgroups (Control Groups - Giới hạn tài nguyên):**
   * *Giống như công tơ điện và đồng hồ nước giới hạn mức tiêu thụ tối đa của từng căn hộ.*
   * Cgroups giới hạn lượng CPU, RAM tối đa mà container được phép dùng, tránh trường hợp một dịch vụ bị treo làm ảnh hưởng đến dịch vụ còn lại.

---

### PHẦN 5. HIỂU LẦM THƯỜNG GẶP & RỦI RO BẢO MẬT (SHARED KERNEL)

* **Hiểu lầm:** Docker Container an toàn bảo mật tuyệt đối hơn Máy ảo (VM).
* **Sự thật:** Vì dùng chung nhân hệ điều hành máy chủ (Shared Kernel), nếu một lỗ hổng bảo mật nghiêm trọng xuất hiện trong nhân Linux (Kernel Vulnerability), hacker nằm trong container có thể khai thác để "vượt ngục" (Container Escape) chiếm quyền điều khiển toàn bộ máy chủ vật lý. Trong khi đó, VM có lớp Guest OS riêng biệt nên việc tấn công xuyên qua Hypervisor để sang máy chủ host là cực kỳ khó khăn.

> [!IMPORTANT]
> **Lời khuyên thực chiến:**
> Trong môi trường Staging/Dev, hãy dùng Docker thoải mái để tiết kiệm RAM. Tuy nhiên trên Production, người ta thường thuê các máy ảo (VM) lớn từ Cloud (AWS EC2, Google Compute Engine) để đảm bảo độ cô lập bảo mật phần cứng, sau đó chạy các Docker Container bên trong các VM đó để quản lý dịch vụ linh hoạt.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG (XÁC MINH KIẾN THỨC)

Để kiểm chứng các thông tin kỹ thuật được nêu trong bài học, bạn có thể tham khảo trực tiếp các nguồn tài liệu uy tín dưới đây:
1. **Khái niệm và kiến trúc Container chính thức từ Docker:**
   * [What is a Container? - Docker Official](https://www.docker.com/resources/what-container/)
2. **So sánh chi tiết Container với Máy ảo từ Microsoft:**
   * [Containers vs. Virtual Machines - Microsoft Learn](https://learn.microsoft.com/en-us/virtualization/windowscontainers/about/)
3. **Tài liệu hướng dẫn (Manual Pages) của Linux về cơ chế Namespaces:**
   * [Linux Namespaces man-pages - man7.org](https://man7.org/linux/man-pages/man7/namespaces.7.html)
4. **Tài liệu hướng dẫn của Linux về cơ chế Cgroups:**
   * [Linux Control Groups cgroups man-pages - man7.org](https://man7.org/linux/man-pages/man7/cgroups.7.html)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Dựa vào phép ẩn dụ "Căn hộ chung cư" ở trên, hãy giải thích tại sao một Docker Container có dung lượng ổ cứng rất nhẹ (chỉ từ vài chục MB) so với máy ảo VM (hàng chục GB)?
* *Gợi ý:* Vì container chia sẻ chung nhân hệ điều hành và các thư viện hệ thống của Host OS (chính là phần móng và kết cấu chịu lực của tòa nhà chung cư), nó không cần đóng gói kèm theo hệ điều hành khách (Guest OS). Container chỉ cần mang theo mã nguồn và các thư viện phụ thuộc tối thiểu để chạy dịch vụ.

#### Câu 2 (Đọc hiểu và dự đoán)
Nếu bạn triển khai `user-service` (chạy JDK 17) và `restaurant-service` (chạy JDK 21) trực tiếp trên hệ điều hành host (không dùng Docker và VM), điều gì sẽ xảy ra nếu lập trình viên thay đổi biến môi trường toàn cục `JAVA_HOME` trỏ về thư mục chứa JDK 21?
* *Gợi ý:* `restaurant-service` sẽ hoạt động bình thường, nhưng `user-service` có thể gặp lỗi nếu lệnh khởi chạy của nó phụ thuộc vào biến `JAVA_HOME` toàn cục này để tìm thư viện chạy dịch vụ, dẫn đến sự bất ổn định hoặc crash tiến trình. Khi đóng gói bằng Docker, mỗi container mang sẵn một bản sao JDK độc lập bên trong, do đó sự thay đổi cấu hình môi trường bên ngoài máy host hoàn toàn không ảnh hưởng đến hoạt động của container.

#### Câu 3 (Xử lý tình huống thực tế)
Khi chạy hệ thống Microservices trên Production, tại sao các doanh nghiệp lớn thường kết hợp cả hai giải pháp: Tạo máy ảo (VM) trước, sau đó mới chạy Docker Container bên trong máy ảo đó?
* *Gợi ý:* Sự kết hợp này mang lại "lợi ích kép": lớp Máy ảo (VM) đóng vai trò lá chắn bảo mật vững chắc ở tầng phần cứng (mỗi nhóm dịch vụ nhạy cảm chạy trên các VM độc lập để tránh rủi ro hacker khai thác Kernel chung), trong khi Docker Container chạy bên trong giúp lập trình viên đóng gói, scale-up dịch vụ nhanh chóng và tối ưu hóa mật độ sử dụng tài nguyên của VM đó.
