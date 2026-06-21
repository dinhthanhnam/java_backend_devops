# SESSION 05: MULTI-SERVICES & API GATEWAY

## LESSON 01: Kiến trúc đa dịch vụ và thiết kế thực thể trong hệ thống phân tán QuickBite

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Giải thích** được cấu trúc đa dịch vụ (Multi-services Architecture) của nền tảng QuickBite và vai trò của từng thành phần.
* **Hiểu bản chất** cấu trúc dữ liệu và thực thể của 4 dịch vụ cốt lõi: `user-service`, `restaurant-service`, `order-service` và `notification-service`.
* **Phân tích và thiết kế** được giải pháp bảo toàn dữ liệu lịch sử thông qua cơ chế Snapshot (lưu thông tin tên khách hàng, tên cửa hàng, tên món ăn và giá bán trực tiếp tại thời điểm đặt đơn thay vì chỉ liên kết ID).
* **Nắm rõ nguyên tắc cô lập dữ liệu** (Database-per-service), hiểu lý do không thể sử dụng các quan hệ liên kết trực tiếp ở mức database (như `@OneToMany` hay `@ManyToMany` xuyên cơ sở dữ liệu) giữa các dịch vụ.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (THIẾT LẬP HỆ THỐNG PHÂN TÁN VÀ LƯU TRỮ LỊCH SỬ)

Khi chuyển đổi từ một hệ thống chạy đơn bản (Monolith) sang kiến trúc đa dịch vụ chạy độc lập, chúng ta gặp những thay đổi lớn về cách tổ chức cơ sở dữ liệu:

1. **Ràng buộc dữ liệu trong kiến trúc Database-per-Service:**
   * Trong hệ thống monolithic truyền thống, tất cả các bảng đều nằm chung một cơ sở dữ liệu. Việc lấy thông tin người dùng từ đơn hàng rất đơn giản bằng cách định nghĩa khóa ngoại hoặc liên kết `@ManyToOne User` trong Hibernate.
   * Tuy nhiên, khi chuyển sang microservices, `user-service` quản lý database `quickbite_user_db`, còn `restaurant-service` quản lý `quickbite_restaurant_db`. Do nằm ở các database logic khác nhau (thậm chí có thể chạy trên các server vật lý khác nhau), chúng ta không thể thiết lập quan hệ khóa ngoại vật lý và cũng không thể thực hiện các câu lệnh SQL JOIN trực tiếp hay các liên kết JPA truyền thống. Ví dụ: `user-service` không thể truy cập thông tin của `restaurant-service` thông qua mối quan hệ `@OneToMany restaurants`. Mọi giao tiếp và truy vấn chéo bắt buộc phải thực hiện qua giao diện mạng (REST API) sử dụng các HTTP Client như **FeignClient**.

2. **Bài toán bảo toàn lịch sử dữ liệu (Snapshot Pattern):**
   * Giả sử trong bảng chi tiết đơn hàng, lập trình viên chỉ lưu `menuItemId` để tham chiếu tới món ăn trong thực đơn của `restaurant-service`.
   * Thực tế vận hành cho thấy: Hôm nay khách hàng đặt món "Trà sữa trân châu" với giá 30.000đ. Ngày mai, nhà hàng cập nhật giá món ăn này lên 40.000đ hoặc đổi tên món thành "Trà sữa đặc biệt". Nếu hệ thống chỉ lưu ID và truy cập động từ `restaurant-service`, khi khách hàng xem lại lịch sử đơn hàng cũ, hóa đơn sẽ hiển thị sai lệch thông tin và giá tiền so với thời điểm họ mua.
   * Để giải quyết việc này, hệ thống cần chụp lại trạng thái dữ liệu (Snapshot) tại thời điểm giao dịch phát sinh. Các thông tin động dễ thay đổi như tên khách hàng (`customerName`), tên nhà hàng (`merchantName`), tên món ăn (`itemName`), và giá món (`price`) cần được sao chép và lưu cứng trực tiếp vào đơn hàng.

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (MÔ HÌNH DỮ LIỆU TRONG HỆ THỐNG PHÂN TÁN)

#### 3.1 Mô hình Database-per-Service
Mỗi dịch vụ sở hữu cơ sở dữ liệu logic riêng để đảm bảo tính cô lập và tự chủ:

```text
  [ user-service ]           [ restaurant-service ]         [ order-service ]         [ notification-service ]
         │                            │                            │                             │
  (DB: quickbite_user_db)     (DB: quickbite_restaurant_db)  (DB: quickbite_order_db)      (DB: quickbite_notification_db)
```

#### 3.2 Cơ chế Snapshot trong lưu trữ giao dịch
Khi một đơn hàng được tạo ra, dữ liệu từ các dịch vụ khác sẽ được chụp lại và lưu trực tiếp vào database của `order-service`:

```text
[Thời điểm đặt đơn]
- Lấy tên người dùng từ user-service        ──► Lưu vào orders.customer_name (Snapshot)
- Lấy tên nhà hàng từ restaurant-service    ──► Lưu vào orders.merchant_name (Snapshot)
- Lấy tên món ăn và giá từ thực đơn         ──► Lưu vào order_items.item_name & order_items.price (Snapshot)
```

#### 3.3 Liên kết lỏng qua ID (Soft References) và Giao tiếp REST API
Để liên kết thông tin giữa các dịch vụ, các thực thể không dùng liên kết JPA trực tiếp (như `@ManyToOne`) mà chỉ lưu trữ khóa chính của thực thể đối tác dưới dạng một trường số nguyên thông thường (`Long`).
* Ví dụ: Thực thể `Order` lưu trường `customerId` (kiểu `Long`) thay vì đối tượng `User`.
* Khi cần lấy thông tin chi tiết của người dùng, dịch vụ sẽ gọi API sang `user-service` thông qua một **FeignClient** (công cụ khai báo REST client trực quan của Spring Cloud):

```java
// Mã giả Feign Client trong Spring Boot
@FeignClient(name = "user-service")
public interface UserServiceClient {
    @GetMapping("/users/{id}")
    UserDTO getUserById(@PathVariable("id") Long id);
}
```

---

### PHẦN 4. THIẾT KẾ CẤU TRÚC BẢNG DỮ LIỆU (ASCII VISUALIZATION)

Dưới đây là thiết kế cấu trúc các bảng dữ liệu logic của hệ thống QuickBite, thể hiện rõ các khóa chính (PK), khóa ngoại nội bộ (FK) và các trường tham chiếu lỏng sang dịch vụ khác (Soft Ref):

#### 4.1 Cơ sở dữ liệu: `quickbite_user_db` (Dịch vụ `user-service`)

* **Bảng `users` (Quản lý tài khoản và vai trò):**
  
  | Tên cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
  | :--- | :--- | :--- | :--- |
  | `id` | Bigint | PK, Auto Increment | Khóa chính |
  | `username` | Varchar(50) | Unique, Not Null | Tên đăng nhập |
  | `password` | Varchar(100)| Not Null | Mật khẩu mã hóa |
  | `full_name`| Varchar(100)| Not Null | Họ và tên hiển thị |
  | `role` | Varchar(20) | Not Null | Vai trò (CUSTOMER, DRIVER, MERCHANT) |

* **Bảng `user_addresses` (Danh sách địa chỉ nhận hàng):**
  
  | Tên cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
  | :--- | :--- | :--- | :--- |
  | `id` | Bigint | PK, Auto Increment | Khóa chính |
  | `label` | Varchar(50) | Not Null | Nhãn địa chỉ (Nhà riêng, Công ty) |
  | `detail_address` | Varchar(255) | Not Null | Địa chỉ chi tiết |
  | `is_default` | Boolean | Not Null | Địa chỉ mặc định |
  | `user_id` | Bigint | FK -> `users(id)` | Liên kết tới tài khoản sở hữu |

* **Bảng `user_wallets` (Ví tiền giả lập để thanh toán):**
  
  | Tên cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
  | :--- | :--- | :--- | :--- |
  | `id` | Bigint | PK, Auto Increment | Khóa chính |
  | `balance` | Decimal(12,2)| Not Null | Số dư tài khoản |
  | `user_id` | Bigint | FK -> `users(id)`, Unique | Liên kết 1-1 tới tài khoản sở hữu |

---

#### 4.2 Cơ sở dữ liệu: `quickbite_restaurant_db` (Dịch vụ `restaurant-service`)

* **Bảng `restaurants` (Thông tin cửa hàng):**
  
  | Tên cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
  | :--- | :--- | :--- | :--- |
  | `id` | Bigint | PK, Auto Increment | Khóa chính |
  | `name` | Varchar(100)| Not Null | Tên nhà hàng |
  | `owner_id` | Bigint | Soft Ref -> `users(id)` | ID của chủ cửa hàng (không tạo FK vật lý) |
  | `is_open` | Boolean | Not Null | Trạng thái đóng/mở cửa |

* **Bảng `menu_categories` (Danh mục thực đơn):**

  | Tên cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
  | :--- | :--- | :--- | :--- |
  | `id` | Bigint | PK, Auto Increment | Khóa chính |
  | `name` | Varchar(100)| Not Null | Tên danh mục (Cơm, Nước uống...) |
  | `restaurant_id`| Bigint | FK -> `restaurants(id)`| Liên kết tới nhà hàng sở hữu danh mục |

* **Bảng `menu_items` (Danh sách món ăn trong thực đơn):**
  
  | Tên cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
  | :--- | :--- | :--- | :--- |
  | `id` | Bigint | PK, Auto Increment | Khóa chính |
  | `name` | Varchar(100)| Not Null | Tên món ăn |
  | `base_price`| Decimal(10,2)| Not Null | Giá bán mặc định |
  | `is_available` | Boolean | Not Null | Trạng thái còn hàng/hết hàng |
  | `category_id`| Bigint | FK -> `menu_categories(id)`| Liên kết tới danh mục món ăn |

---

#### 4.3 Cơ sở dữ liệu: `quickbite_order_db` (Dịch vụ `order-service`)

* **Bảng `orders` (Quản lý đơn hàng tổng quan):**
  
  | Tên cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
  | :--- | :--- | :--- | :--- |
  | `id` | Bigint | PK, Auto Increment | Khóa chính |
  | `customer_id`| Bigint | Soft Ref -> `users(id)` | ID khách đặt (không tạo FK vật lý) |
  | `customer_name`| Varchar(100)| Not Null | **Snapshot:** Họ tên khách hàng lúc đặt đơn |
  | `restaurant_id`| Bigint | Soft Ref -> `restaurants(id)`| ID nhà hàng (không tạo FK vật lý) |
  | `merchant_name`| Varchar(100)| Not Null | **Snapshot:** Tên cửa hàng lúc đặt đơn |
  | `driver_id` | Bigint | Soft Ref -> `users(id)` | ID tài xế nhận giao (cho phép Null) |
  | `delivery_address_id`| Bigint | Soft Ref -> `user_addresses(id)`| Địa chỉ nhận hàng (không tạo FK vật lý) |
  | `items_price`| Decimal(12,2)| Not Null | **Snapshot:** Tổng tiền món ăn tại thời điểm mua |
  | `shipping_fee`| Decimal(12,2)| Not Null | **Snapshot:** Phí ship tại thời điểm mua |
  | `total_price`| Decimal(12,2)| Not Null | Tổng giá trị đơn hàng khách trả |
  | `status` | Varchar(20) | Not Null | Trạng thái (PENDING, ACCEPTED, SHIPPING, DELIVERED) |

* **Bảng `order_items` (Chi tiết món ăn trong đơn hàng):**
  
  | Tên cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
  | :--- | :--- | :--- | :--- |
  | `id` | Bigint | PK, Auto Increment | Khóa chính |
  | `order_id` | Bigint | FK -> `orders(id)` | Liên kết tới đơn hàng tổng |
  | `menu_item_id`| Bigint | Soft Ref -> `menu_items(id)`| ID món ăn gốc |
  | `item_name` | Varchar(100)| Not Null | **Snapshot:** Tên món ăn tại thời điểm mua |
  | `quantity` | Integer | Not Null | Số lượng đặt mua |
  | `price` | Decimal(10,2)| Not Null | **Snapshot:** Đơn giá bán tại thời điểm mua |

* **Bảng `order_status_history` (Lưu vết lịch sử trạng thái đơn hàng):**

  | Tên cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
  | :--- | :--- | :--- | :--- |
  | `id` | Bigint | PK, Auto Increment | Khóa chính |
  | `status` | Varchar(20) | Not Null | Trạng thái chuyển dịch |
  | `changed_at`| Timestamp | Not Null | Thời điểm chuyển trạng thái |
  | `note` | Varchar(255) | Null | Ghi chú (lý do hủy, từ chối...) |
  | `order_id` | Bigint | FK -> `orders(id)` | Liên kết tới đơn hàng tương ứng |

---

#### 4.4 Cơ sở dữ liệu: `quickbite_notification_db` (Dịch vụ `notification-service`)

* **Bảng `notifications` (Danh sách thông báo):**

  | Tên cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
  | :--- | :--- | :--- | :--- |
  | `id` | Bigint | PK, Auto Increment | Khóa chính |
  | `user_id` | Bigint | Soft Ref -> `users(id)` | ID người nhận (không tạo FK vật lý) |
  | `title` | Varchar(100)| Not Null | Tiêu đề thông báo |
  | `content` | Varchar(255)| Not Null | Nội dung chi tiết thông báo |
  | `type` | Varchar(20) | Not Null | Kênh gửi (IN_APP, EMAIL, SMS) |
  | `delivery_status`| Varchar(20)| Not Null | Trạng thái gửi (PENDING, SENT, FAILED) |
  | `created_at`| Timestamp | Not Null | Thời điểm tạo thông báo |

---

### PHẦN 5. HIỂU LẦM THƯỜNG GẶP (FOREIGN KEY TRONG MICROSERVICES)

* **Hiểu lầm thường gặp:** Có thể duy trì quan hệ khóa ngoại vật lý giữa bảng `orders` (trong database `quickbite_order_db`) và bảng `users` (trong database `quickbite_user_db`) nếu ta trỏ đúng địa chỉ kết nối IP.
* **Sự thật:** Không thể tạo được. Khóa ngoại vật lý chỉ có thể được thiết lập khi hai bảng nằm trong cùng một cơ sở dữ liệu logic hoạt động trên cùng một công cụ lưu trữ. Khi đã phân tách thành Database-per-service, các cơ sở dữ liệu hoạt động hoàn toàn độc lập. Việc liên kết lỏng qua trường dữ liệu ID (`Long`) là giải pháp bắt buộc, đi kèm với việc sử dụng giao tiếp API (như Spring Cloud OpenFeign) để truy vấn thông tin khi cần thiết.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG (XÁC MINH KIẾN THỨC)

1. **Mẫu thiết kế Database-per-Service:**
   * [Database per Service Pattern - Microservices.io](https://microservices.io/patterns/data/database-per-service.html)
2. **Khai báo giao tiếp mạng trong Spring với OpenFeign:**
   * [Spring Cloud OpenFeign Official Documentation](https://spring.io/projects/spring-cloud-openfeign)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao trong thiết kế thực thể đơn hàng (`OrderItem`), chúng ta cần sao chép các trường thông tin như `item_name` và `price` trực tiếp từ thực đơn nhà hàng thay vì chỉ lưu `menuItemId` rồi truy vấn động từ `restaurant-service` mỗi khi khách xem lại lịch sử đơn hàng?
* *Gợi ý:* Để áp dụng mẫu thiết kế Snapshot, bảo toàn tính lịch sử tài chính của giao dịch. Giá cả và tên món ăn có thể thay đổi trong tương lai, nhưng hóa đơn cũ của khách hàng thì bắt buộc phải giữ nguyên giá trị tại thời điểm giao dịch phát sinh.

#### Câu 2 (Đọc và dự đoán)
Nếu nhà hàng thay đổi tên từ "Phở 10 Lý Quốc Sư" thành "Phở Lý Quốc Sư - Chi nhánh Cầu Giấy", đơn hàng đã hoàn thành trước đó của khách hàng khi truy vấn lại sẽ hiển thị tên nhà hàng nào? Tại sao?
* *Gợi ý:* Sẽ hiển thị tên cũ là "Phở 10 Lý Quốc Sư". Bởi vì tại thời điểm đặt đơn, tên nhà hàng đã được chụp lại và lưu trực tiếp vào trường `merchant_name` của bảng `orders`.

#### Câu 3 (Xử lý tình huống)
Khi dịch vụ `order-service` cần thực hiện nghiệp vụ kiểm tra thông tin ví của người dùng trước khi duyệt đơn hàng, lập trình viên sẽ làm thế nào khi không thể viết câu lệnh SQL JOIN trực tiếp tới bảng `users` hay `user_wallets`?
* *Gợi ý:* Dịch vụ `order-service` sẽ sử dụng FeignClient để gửi một HTTP request gọi API được expose bởi `user-service` (ví dụ: `GET /users/{id}/wallet`), nhận kết quả số dư tài khoản về để xử lý nghiệp vụ trên ứng dụng.
