# SESSION 05: MULTI-SERVICES & API GATEWAY

## LESSON 01: Kiến trúc đa dịch vụ và thiết kế thực thể JPA trong hệ thống QuickBite

---

### PHẦN 1. MỤC TIÊU BÀI HỌC

Sau khi hoàn thành bài học này, bạn sẽ có khả năng:
* **Giải thích** được kiến trúc đa dịch vụ (Multi-services Architecture) của nền tảng QuickBite và vai trò riêng biệt của từng dịch vụ.
* **Hiểu bản chất** cấu trúc các lớp thực thể JPA (Entities) của 4 dịch vụ cốt lõi: `user-service`, `restaurant-service`, `order-service`, và `notification-service`.
* **Phân tích và thiết kế** được giải pháp bảo toàn tính lịch sử của dữ liệu thông qua cơ chế Snapshot (lưu thông tin món ăn trực tiếp vào chi tiết đơn hàng thay vì chỉ trỏ ID).
* **Nắm rõ nguyên tắc cô lập dữ liệu** (Database-per-service), hiểu lý do không tồn tại khóa ngoại vật lý giữa các cơ sở dữ liệu của các dịch vụ khác nhau.

---

### PHẦN 2. VẤN ĐỀ THỰC TẾ (NỖI ĐAU CỦA THIẾT KẾ HỆ THỐNG THIẾU THỰC TẾ)

Ở Session 4, chúng ta đã khởi chạy thử nghiệm một dịch vụ đơn lẻ (`user-service`) kết nối tới cơ sở dữ liệu (`quickbite-db`). Tuy nhiên, trong thực tế kinh doanh của một nền tảng đặt đồ ăn trực tuyến như QuickBite, nghiệp vụ diễn ra phức tạp hơn nhiều. Hệ thống cần quản lý thông tin khách hàng, nhà hàng, thực đơn, ví tiền, tài xế, đơn hàng và gửi thông báo.

Khi hệ thống phình to, việc thiết kế cơ sở dữ liệu và lớp thực thể thường gặp hai vấn đề chí mạng sau:

1. **Lỗi "Underengineer" - Thiết kế thiếu tính lịch sử (Mất dấu Snapshot):**
   * *Tình huống:* Trong bảng chi tiết đơn hàng (`order_items`), lập trình viên chỉ lưu trường `menu_item_id` để tham chiếu tới bảng thực đơn của nhà hàng nhằm tiết kiệm dung lượng.
   * *Nỗi đau:* Hôm nay khách hàng đặt món "Phở bò" giá 50.000đ. Ngày mai, nhà hàng đổi tên món thành "Phở đặc biệt" và tăng giá lên 70.000đ. Khi khách hàng xem lại lịch sử đơn hàng của ngày hôm qua, hóa đơn sẽ hiển thị là "Phở đặc biệt" với giá 70.000đ! Điều này gây ra sự mâu thuẫn tài chính nghiêm trọng cho cả kế toán lẫn trải nghiệm người dùng.
2. **Sự phụ thuộc chéo dữ liệu (Cross-Database Join):**
   * *Tình huống:* Trong kiến trúc phân tán, mỗi dịch vụ sở hữu cơ sở dữ liệu riêng (Database-per-service). Dịch vụ đơn hàng (`order-service`) chạy trên DB `quickbite_order`, trong khi dịch vụ người dùng (`user-service`) chạy trên DB `quickbite_user`. 
   * *Nỗi đau:* Nếu lập trình viên cố tình tạo liên kết khóa ngoại JPA như `@ManyToOne User user` hay viết câu lệnh SQL JOIN chéo giữa bảng `orders` và bảng `users`, ứng dụng sẽ crash ngay lập tức vì hai bảng này nằm ở hai máy chủ/cơ sở dữ liệu vật lý hoàn toàn cô lập.

*Để giải quyết triệt để các vấn đề này, chúng ta cần một thiết kế thực thể JPA chuẩn hóa cho kiến trúc đa dịch vụ, áp dụng mẫu thiết kế Snapshot và tuân thủ nguyên tắc cô lập dữ liệu.*

---

### PHẦN 3. KIẾN THỨC CỐT LÕI (MÔ HÌNH THIẾT KẾ ĐỐI TƯỢNG TRONG HỆ THỐNG PHÂN TÁN)

#### 3.1 Thiết kế Cơ sở dữ liệu riêng biệt (Database-per-Service Pattern)
Trong kiến trúc Microservices, mỗi dịch vụ phải sở hữu hoàn toàn dữ liệu của nó để đảm bảo tính độc lập và khả năng mở rộng. 
* **`user-service`** quản lý dữ liệu về người dùng (`users`), địa chỉ giao hàng (`user_addresses`) và ví tiền giả lập (`user_wallets`).
* **`restaurant-service`** quản lý dữ liệu về nhà hàng (`restaurants`), danh mục thực đơn (`menu_categories`) và món ăn (`menu_items`).
* **`order-service`** quản lý dữ liệu về đơn hàng (`orders`), chi tiết món ăn trong đơn (`order_items`) và lịch sử chuyển trạng thái đơn hàng (`order_status_history`).
* **`notification-service`** quản lý lịch sử gửi thông báo (`notifications`).

```text
  ┌─────────────────┐      ┌────────────────────────┐      ┌─────────────────┐      ┌────────────────────────┐
  │  user-service   │      │   restaurant-service   │      │  order-service  │      │  notification-service  │
  └────────┬────────┘      └───────────┬────────────┘      └────────┬────────┘      └───────────┬────────────┘
           ▼                           ▼                            ▼                           ▼
 ┌───────────────────┐       ┌────────────────────┐       ┌───────────────────┐       ┌────────────────────┐
 │  DB:              │       │  DB:               │       │  DB:              │       │  DB:               │
 │  quickbite_user   │       │  quickbite_rest    │       │  quickbite_order  │       │  quickbite_notif   │
 └───────────────────┘       └────────────────────┘       └───────────────────┘       └────────────────────┘
```

#### 3.2 Cơ chế Snapshot trong thiết kế thực thể thương mại điện tử
Khi khách hàng mua một món ăn, các thông tin mang tính thời điểm như **tên món ăn** và **giá tiền** phải được "chụp lại" (snapshot) và lưu trực tiếp vào thực thể `OrderItem`. 
* Chúng ta chỉ giữ trường ID (`menuItemId`) như một mã tham chiếu tham khảo.
* Tên món (`itemName`) và giá tiền (`price`) được sao chép cứng vào bảng `order_items` tại thời điểm tạo đơn hàng. Kể từ đó, mọi thay đổi về thực đơn của nhà hàng sẽ không làm sai lệch hóa đơn đã thanh toán.

#### 3.3 Ràng buộc lỏng qua ID (Soft References)
Thay vì sử dụng các mối quan hệ JPA truyền thống như `@ManyToOne` trỏ đến một Entity của dịch vụ khác, các thực thể trong hệ thống Microservices liên kết với nhau qua các ID dạng số nguyên (`Long`).
* Ví dụ: Trong thực thể `Order` của `order-service`, chúng ta lưu `customerId` và `restaurantId` dưới dạng trường `Long` thông thường, thay vì `@ManyToOne User user` hay `@ManyToOne Restaurant restaurant`.

---

### PHẦN 4. DEMO VÀ THỰC HÀNH: THIẾT KẾ CÁC LỚP THỰC THỂ JPA CHO QUICKBITE

Dưới đây là mã nguồn Java Spring Boot chi tiết định nghĩa cấu trúc Entity cho 4 dịch vụ cốt lõi của QuickBite. Tất cả đều sử dụng Spring Data JPA để ánh xạ xuống PostgreSQL.

#### 4.1 Thực thể dịch vụ Người dùng (`user-service`)
Dịch vụ này quản lý tài khoản, ví tiền và các địa chỉ nhận hàng của người dùng. Một User không thể đặt hàng nếu thiếu địa chỉ giao và ví tiền để thanh toán.

```java
// User.java
package com.quickbite.user.entity;

import jakarta.persistence.*;
import java.util.List;

@Entity 
@Table(name = "users")
public class User {
    @Id 
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String username;
    private String password;
    private String fullName;

    @Enumerated(EnumType.STRING)
    private Role role; // CUSTOMER, DRIVER, MERCHANT

    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
    private List<UserAddress> addresses; // Danh sách địa chỉ nhận hàng

    @OneToOne(mappedBy = "user", cascade = CascadeType.ALL)
    private UserWallet wallet; // Ví tiền điện tử giả lập
    
    // Getters, Setters, Constructors
}

// UserAddress.java
package com.quickbite.user.entity;

import jakarta.persistence.*;

@Entity 
@Table(name = "user_addresses")
public class UserAddress {
    @Id 
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String label; // Ví dụ: Nhà riêng, Văn phòng
    private String detailAddress;
    private boolean isDefault;
    
    @ManyToOne 
    @JoinColumn(name = "user_id")
    private User user;
    
    // Getters, Setters, Constructors
}

// UserWallet.java
package com.quickbite.user.entity;

import jakarta.persistence.*;
import java.math.BigDecimal;

@Entity 
@Table(name = "user_wallets")
public class UserWallet {
    @Id 
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private BigDecimal balance; // Số dư ví tiền
    
    @OneToOne 
    @JoinColumn(name = "user_id")
    private User user;
    
    // Getters, Setters, Constructors
}
```

#### 4.2 Thực thể dịch vụ Nhà hàng (`restaurant-service`)
Quản lý thông tin nhà hàng, danh mục thực đơn và chi tiết món ăn của từng nhà hàng.

```java
// Restaurant.java
package com.quickbite.restaurant.entity;

import jakarta.persistence.*;
import java.util.List;

@Entity 
@Table(name = "restaurants")
public class Restaurant {
    @Id 
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private Long ownerId; // Chỉ lưu ID tham chiếu sang User (Role: MERCHANT) ở user-service
    private boolean isOpen;

    @OneToMany(mappedBy = "restaurant", cascade = CascadeType.ALL)
    private List<MenuCategory> categories; // Phân mục: Đồ ăn, Nước uống...
    
    // Getters, Setters, Constructors
}

// MenuItem.java
package com.quickbite.restaurant.entity;

import jakarta.persistence.*;
import java.math.BigDecimal;

@Entity 
@Table(name = "menu_items")
public class MenuItem {
    @Id 
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private BigDecimal basePrice;
    private boolean isAvailable;

    @ManyToOne 
    @JoinColumn(name = "category_id")
    private MenuCategory category;
    
    // Getters, Setters, Constructors
}
```

#### 4.3 Thực thể dịch vụ Đơn hàng (`order-service`)
Đóng vai trò trung tâm xử lý nghiệp vụ. Lưu lịch sử đơn hàng, chi tiết Snapshot món ăn và gán tài xế.

```java
// Order.java
package com.quickbite.order.entity;

import jakarta.persistence.*;
import java.math.BigDecimal;
import java.util.List;

@Entity 
@Table(name = "orders")
public class Order {
    @Id 
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private Long customerId;        // Liên kết lỏng sang User ở user-service
    private Long restaurantId;      // Liên kết lỏng sang Restaurant ở restaurant-service
    private Long driverId;          // Sẽ được cập nhật khi có Tài xế nhận đơn
    private Long deliveryAddressId; // Lưu ID địa chỉ giao hàng của User

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> items;

    private BigDecimal itemsPrice;  // Tổng tiền các món
    private BigDecimal shippingFee;  // Phí giao hàng
    private BigDecimal totalPrice;   // Tổng tiền khách phải trả (itemsPrice + shippingFee)

    @Enumerated(EnumType.STRING)
    private OrderStatus status; // PENDING, ACCEPTED, SHIPPING, DELIVERED, CANCELLED

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderStatusHistory> statusHistories; // Lưu dòng vết thời gian (Timeline)
    
    // Getters, Setters, Constructors
}

// OrderItem.java (Áp dụng kỹ thuật Snapshot để lưu thông tin tại thời điểm mua)
package com.quickbite.order.entity;

import jakarta.persistence.*;
import java.math.BigDecimal;

@Entity 
@Table(name = "order_items")
public class OrderItem {
    @Id 
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private Long menuItemId;    // Chỉ lưu ID tham chiếu tham khảo
    private String itemName;    // SNAPSHOT: Sao chép cứng tên món ăn tại thời điểm mua
    private Integer quantity;   // Số lượng món đặt
    private BigDecimal price;   // SNAPSHOT: Sao chép cứng giá món ăn tại thời điểm mua

    @ManyToOne 
    @JoinColumn(name = "order_id")
    private Order order;
    
    // Getters, Setters, Constructors
}
```

#### 4.4 Thực thể dịch vụ Thông báo (`notification-service`)
Dịch vụ này tiếp nhận sự kiện thay đổi trạng thái đơn hàng để gửi email, SMS hoặc thông báo in-app đến người dùng.

```java
// Notification.java
package com.quickbite.notification.entity;

import jakarta.persistence.*;
import java.time.LocalDateTime;

@Entity 
@Table(name = "notifications")
public class Notification {
    @Id 
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private Long userId; // Liên kết lỏng tới người nhận ở user-service
    private String title;
    private String content;
    
    @Enumerated(EnumType.STRING)
    private NotificationType type; // IN_APP, EMAIL, SMS
    
    @Enumerated(EnumType.STRING)
    private DeliveryStatus deliveryStatus; // PENDING, SENT, FAILED
    
    private LocalDateTime createdAt;
    
    // Getters, Setters, Constructors
}
```

---

### PHẦN 5. HIỂU LẦM THƯỜNG GẶP (FOREIGN KEY XUYÊN DATABASE)

* **Hiểu lầm thường gặp:** Có thể định nghĩa khóa ngoại vật lý và ánh xạ JPA (ví dụ: `@ManyToOne`) từ thực thể `Order` (trong database `quickbite_order`) tới thực thể `User` (trong database `quickbite_user`) thông qua một cấu hình JPA nâng cao nào đó.
* **Sự thật:** Không thể. 
  * Cơ chế khóa ngoại vật lý (Foreign Key) chỉ được hỗ trợ bởi hệ quản trị cơ sở dữ liệu khi các bảng nằm trong **cùng một database logic**.
  * Trong kiến trúc Microservices, mỗi dịch vụ chạy trên một cơ sở dữ liệu hoàn toàn độc lập, thậm chí có thể nằm trên các máy chủ cơ sở dữ liệu vật lý khác nhau. Việc cố gắng tạo khóa ngoại chéo database sẽ phá vỡ tính cô lập (Decoupling) và khiến các dịch vụ không thể khởi động hoặc không thể chạy độc lập. 
  * Cách duy nhất là lưu ID tham chiếu lỏng (`Long customerId`) và thực hiện truy vấn thông tin chi tiết qua giao tiếp API (REST, gRPC) hoặc đồng bộ dữ liệu.

---

### PHẦN 6. TÀI LIỆU THAM KHẢO CHÍNH THỐNG (XÁC MINH KIẾN THỨC)

1. **Nguyên lý thiết kế dữ liệu cô lập trong Microservices:**
   * [Database per Service Pattern - Microservices.io](https://docs.google.com/url?q=https://microservices.io/patterns/data/database-per-service.html)
2. **Hướng dẫn ánh xạ quan hệ thực thể với Spring Data JPA:**
   * [Spring Data JPA Reference Documentation](https://docs.google.com/url?q=https://docs.spring.io/spring-data/jpa/docs/current/reference/html/)
3. **Mẫu thiết kế lưu trữ lịch sử đơn hàng (Snapshot Pattern):**
   * [Designing E-Commerce Database Schemas - Designing Data-Intensive Applications](https://docs.google.com/url?q=https://dataintensive.net/)

---

### PHẦN 7. CÂU HỎI ĐÁNH GIÁ NHANH

#### Câu 1 (Hiểu bản chất)
Tại sao trong thực thể `OrderItem` chúng ta bắt buộc phải lưu trữ cả trường `itemName` và `price` trực tiếp thay vì chỉ lưu `menuItemId` và truy vấn động từ `restaurant-service` mỗi khi hiển thị hóa đơn?
* *Gợi ý:* Để đảm bảo tính toàn vẹn lịch sử (Snapshot Pattern). Nếu cửa hàng thay đổi giá món ăn hoặc cập nhật lại thực đơn trong tương lai, hóa đơn của các đơn hàng cũ đã đặt vẫn phải giữ nguyên thông tin tên món và giá cả tại đúng thời điểm khách đặt hàng.

#### Câu 2 (Đọc và dự đoán)
Điều gì sẽ xảy ra nếu lập trình viên định nghĩa thực thể `Order` có cấu hình ánh xạ JPA `@ManyToOne` kết nối trực tiếp đến thực thể `User` khi hai thực thể này được triển khai trên hai dịch vụ có cơ sở dữ liệu vật lý nằm độc lập?
* *Gợi ý:* Ứng dụng sẽ gặp lỗi ngay lập tức khi khởi chạy hoặc khi thực hiện kiểm tra schema (DDL validation). Hibernate/JPA không thể tìm thấy bảng `users` trong cơ sở dữ liệu `quickbite_order` để thiết lập khóa ngoại, dẫn đến lỗi biên dịch cấu hình persistence unit.

#### Câu 3 (Xử lý tình huống)
Để bảo toàn tính cô lập và hiệu năng của hệ thống QuickBite, khi dịch vụ `order-service` cần hiển thị thông tin hóa đơn gồm cả tên đầy đủ của Khách hàng (`fullName`), lập trình viên nên làm thế nào khi thực thể `Order` chỉ lưu trường `customerId`?
* *Gợi ý:* Dịch vụ `order-service` sẽ gửi một HTTP request (ví dụ sử dụng Feign Client hoặc WebClient) tới dịch vụ `user-service` với tham số là `customerId` để truy vấn thông tin chi tiết người dùng (`fullName`) tại thời điểm hiển thị hóa đơn, thay vì tìm cách thực hiện lệnh JOIN cơ sở dữ liệu.
