# Kịch bản Thuyết trình - Session 04: Docker Compose & Dockerfile

---

## Lesson 1: Dockerfile và cách đóng gói ứng dụng Spring Boot thực tế

### 1. Phần lý thuyết

**(Mở đầu - Khơi gợi bối cảnh thực tế)**
Xin chào các bạn học viên! Ở buổi học trước, chúng ta đã thoải mái sử dụng lệnh `docker run` để khởi chạy container database PostgreSQL. Sở dĩ chúng ta làm việc đó rất dễ dàng là vì PostgreSQL đã được đóng gói sẵn thành một image hoàn chỉnh trên Docker Hub, các bạn chỉ việc kéo về chạy mà không cần thay đổi bất kỳ thành phần nào bên trong cả.

Thế nhưng, câu hỏi đặt ra là: Đối với các dịch vụ do chính chúng ta viết code ra, ví dụ như `user-service`, thì sao? Rõ ràng là chưa có một image nào của `user-service` tồn tại trên Docker Hub cả. Cái chúng ta mong muốn ở đây là một image nền (Base Image) chứa môi trường chạy JRE hoặc JDK sạch, còn code Spring Boot thì nằm ở máy host.

**[Slide 1: Vấn đề thực tế - Sự phân tán cấu hình trong vận hành thủ công]**
Về mặt kỹ thuật, chúng ta vẫn có thể sử dụng các tham số của lệnh `docker run` trỏ tới image JRE trắng, rồi mount file JAR từ máy host vào bên trong container để khởi chạy. Thế nhưng, cách làm này không hề tối ưu và cực kỳ dễ xảy ra sai sót khi chúng ta phải cấu hình thủ công các thông số hệ thống.

Chưa kể đến là vấn đề kết nối mạng. Khi dịch vụ backend của chúng ta đã được đưa vào chạy bên trong container, nó sẽ không thể hiểu được khái niệm `localhost` của máy host đại diện cho database nữa. Việc port mapping cho database từ trước đến nay cũng không còn hiệu quả cho việc kết nối trực tiếp giữa hai container với nhau. Lúc này, chúng ta buộc phải chạy một lệnh inspect thủ công để tìm ra địa chỉ IP nội bộ chính xác của container database trong mạng Bridge mặc định của Docker để nạp vào backend.

Việc gõ các lệnh chạy thủ công này quá dài dòng và phức tạp. Để giải quyết triệt để tất cả các vấn đề trên, chúng ta sử dụng **Dockerfile**. Đây được coi là "nguồn sự thật duy nhất" (Source of Truth) dùng để đóng gói toàn bộ đặc tả môi trường chạy (Hệ điều hành nền, Java Runtime, file JAR, cổng chạy, lệnh khởi động) vào một tệp cấu hình duy nhất đặt ngay trong mã nguồn dịch vụ. Bất kỳ ai khi có mã nguồn này đều có thể tự động build ra một Docker Image hoạt động nhất quán mà không cần cấu hình thủ công.

**[Slide 2: Cấu trúc Dockerfile cơ bản cho ứng dụng Spring Boot]**
Một tệp Dockerfile là tập hợp các chỉ thị được thực thi tuần tự từ trên xuống để build ra một Docker Image.
Các chỉ thị cơ bản bao gồm:
- **`FROM`**: Khai báo image nền. Ở đây chúng ta sử dụng JRE Alpine siêu nhẹ để giảm dung lượng file build.
- **`WORKDIR`**: Thiết lập thư mục làm việc mặc định bên trong container.
- **`COPY`**: Sao chép file JAR đã build từ máy host vào bên trong container.
- **`EXPOSE`**: Khai báo cổng lắng nghe nội bộ của ứng dụng (mang ý nghĩa tài liệu hóa).
- **`ENTRYPOINT`**: Lệnh chạy mặc định khi container được khởi tạo.

**[Slide 3: Phân biệt ENTRYPOINT vs CMD và Cơ chế Graceful Shutdown]**
Thầy muốn các bạn lưu ý một kiến thức rất sắc sảo ở đây: Đó là sự khác biệt giữa `ENTRYPOINT` viết dưới dạng mảng (Exec Form) và `CMD` viết dưới dạng chuỗi (Shell Form) liên quan đến việc tắt ứng dụng an toàn.
- **ENTRYPOINT (Exec Form - Khuyên dùng):** Cú pháp dạng mảng `ENTRYPOINT ["java", "-jar", "app.jar"]`. Khi chạy, tiến trình Java sẽ được gán trực tiếp làm tiến trình gốc **PID 1** của container. Khi chúng ta gõ lệnh tắt container (`docker stop`), tín hiệu tắt hệ thống nhẹ nhàng `SIGTERM` sẽ gửi trực tiếp đến Java. Spring Boot nhận diện được tín hiệu này để từ từ đóng các connection pool database, hoàn tất các request dở dang rồi mới tắt. Đây gọi là **Graceful Shutdown** (tắt máy an toàn).
- **CMD (Shell Form):** Cú pháp dạng chuỗi `CMD java -jar app.jar`. Docker sẽ chạy nó thông qua một trình shell trung gian `/bin/sh -c`. Lúc này, trình shell chiếm quyền PID 1, còn Java chỉ là tiến trình con. Khi tắt container, trình shell nhận `SIGTERM` nhưng không chuyển tiếp xuống tiến trình Java. Sau 10 giây chờ đợi không phản hồi, Docker Engine buộc phải phát lệnh cưỡng bức `SIGKILL` tiêu diệt tiến trình Java ngay lập tức, rất dễ gây lỗi dữ liệu dở dang.

### 2. Phần thực hành (Demo)

**[Live Demo: Đóng gói và chạy thử ứng dụng Spring Boot]**
Bây giờ, thầy sẽ thực hiện tuần tự các bước build và đóng gói dịch vụ `user-service`.

*(Thao tác trên Terminal)*:
- *Bước 1: Chúng ta di chuyển vào thư mục dự án `user-service` và build file JAR bằng Gradle:*
  ```bash
  cd user-service
  ./gradlew bootJar
  ```
- *Bước 2: Tạo một file tên là `Dockerfile` (không có đuôi mở rộng) nằm tại thư mục gốc của project:*
  ```dockerfile
  FROM eclipse-temurin:17-jre-alpine
  WORKDIR /app
  COPY build/libs/user-service.jar app.jar
  EXPOSE 8081
  ENTRYPOINT ["java", "-jar", "app.jar"]
  ```
- *Bước 3: Thực hiện build Docker Image từ file Dockerfile vừa viết:*
  ```bash
  docker build -t quickbite-user-service:v1 .
  ```
  *(Giải thích): Dấu chấm ở cuối đại diện cho thư mục hiện hành làm Build Context.*
- *Bước 4: Kiểm tra xem Image đã được tạo thành công chưa:*
  ```bash
  docker images
  ```
- *Bước 5: Khởi chạy container từ image tự build:*
  ```bash
  docker run -d -p 8081:8081 --name user-service quickbite-user-service:v1
  ```
  *(Giải thích): Cờ `EXPOSE 8081` trong Dockerfile chỉ mang ý nghĩa tài liệu hóa. Để kết nối được từ máy host, các bạn bắt buộc phải truyền cờ `-p 8081:8081` khi run.*
- *Bước 6: Kiểm tra logs để đảm bảo Spring Boot khởi chạy thành công:*
  ```bash
  docker logs -f user-service
  ```

---

## Lesson 2: Docker Compose và khái niệm hệ thống nhiều container

### 1. Phần lý thuyết

**[Slide 5: Hệ thống nhiều container trong Microservices]**
Thế thì khi hệ thống QuickBite phình to ra với nhiều dịch vụ (như Database, User Service, Restaurant Service, Gateway), mỗi dịch vụ sẽ được đóng gói và chạy trong một container độc lập. 

Tại sao chúng ta phải chia tách thành nhiều container riêng biệt như vậy?
- **Đa dạng công nghệ:** Dịch vụ này chạy Java 17, dịch vụ kia chạy Java 21, Database viết bằng C. Tách biệt giúp tránh phình to image và xung đột runtime.
- **Mở rộng độc lập (Scaling):** Các bạn có thể chạy 3-4 container cho dịch vụ chịu tải cao như Order Service mà không cần phải nhân bản thêm container database.
- **Cô lập lỗi:** Một dịch vụ bị crash không thể kéo sập toàn bộ hệ thống.

Tuy nhiên, thách thức lớn ở đây là làm sao để điều phối kết nối mạng và quản lý vòng đời cho cả chục container này? Nếu cứ dùng lệnh CLI `docker run` thủ công thì sẽ vô cùng phức tạp và dễ nhầm lẫn.

**[Slide 6: Định nghĩa Docker Compose và Quy trình 3 bước]**
Để giải quyết bài toán điều phối, Docker cung cấp công cụ **Docker Compose**.
Thay vì gõ từng lệnh CLI riêng lẻ (phong cách Mệnh lệnh), Docker Compose cho phép chúng ta mô tả toàn bộ trạng thái hệ thống gồm nhiều container trong một tệp cấu hình duy nhất tên là `docker-compose.yml` (định dạng YAML).

Quy trình làm việc tiêu chuẩn gồm 3 bước:
1. **Đóng gói (Define):** Viết `Dockerfile` cho từng dịch vụ.
2. **Khai báo (Declare):** Viết file `docker-compose.yml` để khai báo các dịch vụ, cổng kết nối, mạng ảo nội bộ và các biến môi trường.
3. **Vận hành (Run):** Chạy duy nhất lệnh `docker compose up` để tự động hóa toàn bộ chu kỳ khởi tạo hệ thống.

### 2. Phần thực hành (Demo)

**[Live Demo: Kiểm tra cài đặt và chạy file Compose tối giản]**
Thầy sẽ hướng dẫn các bạn kiểm tra công cụ Docker Compose trên máy ảo Linux.

*(Thao tác trên Terminal)*:
- *Bước 1: Kiểm tra phiên bản Docker Compose:*
  ```bash
  docker compose version
  ```
  *(Giải thích): Ở các phiên bản mới, Docker Compose chạy trực tiếp dưới dạng plugin của Docker CLI nên không cần viết dấu gạch ngang ở giữa.*
- *Bước 2: Nếu hệ thống báo thiếu lệnh, các bạn tiến hành cài đặt plugin:*
  ```bash
  sudo apt-get update
  sudo apt-get install -y docker-compose-plugin
  ```
- *Bước 3: Tạo một file `docker-compose.yml` tối giản để chạy thử nghiệm:*
  ```yaml
  version: '3.8'
  services:
    java-tester:
      image: eclipse-temurin:17-jre-alpine
      command: java -version
  ```
- *Bước 4: Khởi chạy file Compose:*
  ```bash
  docker compose up
  ```
  *Docker Compose sẽ tự động tải image, tạo container, in ra phiên bản Java rồi dừng lại.*
- *Bước 5: Dọn dẹp container vừa tạo:*
  ```bash
  docker compose down
  ```

---

## Lesson 3: Cấu trúc file docker-compose.yml (services, image, build)

### 1. Phần lý thuyết

**[Slide 8: Cấu trúc cơ bản của tệp docker-compose.yml]**
Tệp cấu hình của Docker Compose được viết bằng định dạng YAML. Các bạn lưu ý là YAML phân biệt cấu trúc bằng thụt lề khoảng trắng (indentation), tuyệt đối không được dùng phím Tab trên bàn phím.
Các từ khóa quan trọng bao gồm:
- **`version`**: Phiên bản định dạng của file Compose (chúng ta dùng bản phổ biến `'3.8'`).
- **`services`**: Khối định nghĩa danh sách các container chạy trong cụm.
- **`image`**: Chỉ định image có sẵn được kéo về từ Registry.
- **`build`**: Cấu hình tự động đóng gói image từ mã nguồn cục bộ. Nó gồm `context` (thư mục chứa Dockerfile và code) và `dockerfile` (tên file Dockerfile).

**[Slide 9: Chiến lược tách biệt Database chạy độc lập]**
Thầy muốn nhấn mạnh một thiết kế rất quan trọng trong dự án QuickBite và thực tế doanh nghiệp: **Database phải được chạy độc lập hoàn toàn khỏi Application Compose**. Chúng ta không gộp chung cấu hình Database và Code Spring Boot vào chung một file Compose vì 4 lý do:
1. **Chu kỳ phát triển (Development Loop):** Database khởi chạy rất nặng. Khi lập trình viên sửa code Spring Boot, chúng ta phải build và restart backend liên tục. Tách biệt giúp database hoạt động ổn định, không bị khởi động lại ngắt quãng theo backend.
2. **Tiết kiệm tài nguyên:** Thay vì mỗi microservice chạy một container database riêng gây tốn RAM, chúng ta dựng 1 container database dùng chung duy nhất chứa nhiều database con bên trong.
3. **An toàn dữ liệu:** Tránh việc vô tình xóa sạch dữ liệu DB khi các bạn dọn dẹp backend bằng lệnh `docker compose down -v`.
4. **Tiêu chuẩn Production:** Giúp chúng ta sẵn sàng kết nối tới các dịch vụ Cloud Database (như AWS RDS) sau này mà không làm ảnh hưởng đến cấu hình container ứng dụng.

### 2. Phần thực hành (Demo)

**[Live Demo: Cấu hình Database dùng chung khởi tạo tự động]**
Bây giờ, chúng ta sẽ thực hành thiết lập một container PostgreSQL độc lập và tự động khởi tạo 4 database con cho hệ thống QuickBite.

*(Thao tác trên Terminal)*:
- *Bước 1: Tạo thư mục `quickbite-database` nằm độc lập ngoài dự án Java:*
  ```bash
  mkdir quickbite-database
  cd quickbite-database
  ```
- *Bước 2: Chuẩn bị file script SQL tự động khởi tạo database (`init-db.sql`):*
  *Chúng ta sẽ viết script tạo user và database riêng cho từng microservice (User, Restaurant, Order, Notification) để phân tách quyền sở hữu.*
  *(Tạo file init-db.sql với nội dung tạo database).*
- *Bước 3: Viết file cấu hình `docker-compose.yml` cho database:*
  ```yaml
  version: '3.8'
  services:
    quickbite-db:
      image: postgres:15-alpine
      container_name: quickbite-db
      ports:
        - "5432:5432"
      environment:
        POSTGRES_USER: postgres
        POSTGRES_PASSWORD: secret_password
      volumes:
        - db-data:/var/lib/postgresql/data
        - ./init-db.sql:/docker-entrypoint-initdb.d/init-db.sql
      networks:
        - quickbite-net
      restart: always
  volumes:
    db-data:
  networks:
    quickbite-net:
      name: quickbite-net
      driver: bridge
  ```
  *(Giải thích): Script SQL được mount vào thư mục tự chạy `/docker-entrypoint-initdb.d/` của Postgres. Mạng ảo được đặt tên cố định là `quickbite-net`.*
- *Bước 4: Khởi chạy database chạy ngầm lâu dài:*
  ```bash
  docker compose up -d
  ```

**[Live Demo: Cấu hình tự động build và chạy dịch vụ Backend]**
Tiếp theo, thầy sẽ cấu hình tự build dịch vụ `user-service` và kết nối vào mạng của database vừa chạy.

*(Thao tác trên Terminal)*:
- *Bước 5: Tạo file `docker-compose.yml` tại thư mục gốc của dự án backend:*
  ```yaml
  version: '3.8'
  services:
    quickbite-user:
      build:
        context: ./user-service
      ports:
        - "8081:8081"
      networks:
        - quickbite-net
  networks:
    quickbite-net:
      external: true
  ```
  *(Giải thích): Cờ `external: true` khai báo rằng mạng ảo `quickbite-net` đã được tạo từ trước bởi container database, Compose không tự tạo mạng mới.*
- *Bước 6: Khởi chạy và tự động build ứng dụng:*
  ```bash
  docker compose up --build
  ```
  *(Lưu ý thực chiến): Khi các bạn thay đổi code Java, Docker Compose không tự phát hiện để rebuild. Các bạn bắt buộc phải build lại file JAR mới trên máy host trước, rồi chạy lệnh kèm cờ `--build` thì hệ thống mới cập nhật code mới.*

---

## Lesson 4: Biến môi trường và cấu hình port

### 1. Phần lý thuyết

**[Slide 12: Cú pháp Port Mapping và Lỗ hổng bảo mật rò rỉ thông tin]**
Khi chạy container, cú pháp `ports` dạng `- "HOST_PORT:CONTAINER_PORT"` giúp Docker NAT traffic từ ngoài máy host vào trong container.

Tuy nhiên, có một lỗi bảo mật cực kỳ nghiêm trọng mà các bạn rất hay mắc phải: Đó là ghi trực tiếp mật khẩu database, token mật của ứng dụng vào thẳng file `docker-compose.yml` rồi đẩy lên Git repository.
- Hành động này khiến bất kỳ ai có quyền truy cập Git đều có thể đọc được mật khẩu hệ thống thật.
- Đồng thời làm cứng cấu hình, vi phạm nguyên tắc "Build once, run anywhere" khi cần chuyển đổi giữa các môi trường Dev, Staging và Production.

**[Slide 13: Giải pháp tách biệt cấu hình bằng file .env]**
Để giải quyết triệt để lỗi này, chúng ta sử dụng tệp tin môi trường ẩn mang tên `.env` đặt cùng cấp với file `docker-compose.yml`.
- Các thông số bảo mật, mật khẩu sẽ được khai báo trong file `.env`.
- Chúng ta thêm file `.env` này vào `.gitignore` để ngăn không cho đẩy lên Git.
- Trong file `docker-compose.yml`, các bạn sử dụng cú pháp nội suy `${TEN_BIEN}` để gọi động các biến này từ file `.env` ra khi khởi chạy.

### 2. Phần thực hành (Demo)

**[Live Demo: Cấu hình biến môi trường qua file .env]**
Bây giờ, thầy sẽ hướng dẫn các bạn tách toàn bộ cấu hình nhạy cảm của `user-service` ra file môi trường ngoài.

*(Thao tác trên Terminal)*:
- *Bước 1: Tạo file `.env` cùng cấp với file `docker-compose.yml`:*
  ```env
  DB_HOST=quickbite-db
  DB_PORT=5432
  USER_DB_NAME=quickbite_user_db
  USER_DB_USERNAME=quickbite_user
  USER_DB_PASSWORD=quickbite_user
  USER_SERVER_PORT=8081
  ```
- *Bước 2: Thay đổi nội dung `docker-compose.yml` để nạp biến môi trường động:*
  ```yaml
  version: '3.8'
  services:
    quickbite-user:
      build: ./user-service
      ports:
        - "${USER_SERVER_PORT}:${USER_SERVER_PORT}"
      environment:
        - DB_HOST=${DB_HOST}
        - DB_PORT=${DB_PORT}
        - DB_NAME=${USER_DB_NAME}
        - DB_USERNAME=${USER_DB_USERNAME}
        - DB_PASSWORD=${USER_DB_PASSWORD}
        - SERVER_PORT=${USER_SERVER_PORT}
  ```
- *Bước 3: Kiểm tra xem Docker Compose đã nhận diện và thay thế biến chính xác chưa:*
  ```bash
  docker compose config
  ```
  *(Giải thích): Lệnh `config` sẽ đọc file `.env`, tự động thay thế các biến `${...}` thành giá trị thực tế và in ra tệp cấu hình hoàn chỉnh để chúng ta kiểm tra lỗi cú pháp trước khi chạy thực tế.*

---

## Lesson 5: Volume và network trong Docker Compose

### 1. Phần lý thuyết

**[Slide 14: Docker Volumes - Cơ chế lưu trữ dữ liệu bền vững]**
Như các bạn đã biết, hệ thống file của container là tạm thời. Dữ liệu sẽ biến mất hoàn toàn khi container bị xóa bỏ. Trong Docker Compose, chúng ta quản lý dữ liệu qua 2 cơ chế chính:
- **Named Volumes (Volume định danh):** Được định nghĩa ở khối `volumes` ở cuối file Compose. Docker tự động quản lý thư mục lưu trữ ẩn này trên máy host. Cơ chế này đem lại hiệu năng đọc ghi cao, độc lập với cấu trúc thư mục của hệ điều hành host, chuyên dùng cho dữ liệu động của Database.
- **Bind Mounts (Liên kết trực tiếp thư mục):** Ánh xạ trực tiếp một file hoặc thư mục cụ thể từ máy host vào container. Cơ chế này phụ thuộc vào cấu trúc thư mục của máy host, phù hợp để gắn các file cấu hình tĩnh hoặc phục vụ quá trình hot-reload code.

**[Slide 15: Docker Networks & Cơ chế Service Discovery]**
Khi khởi chạy một tệp Compose, Docker tự động tạo ra một mạng ảo Bridge cho cụm container và bật sẵn một dịch vụ DNS nội bộ.
Một tính năng cực kỳ quan trọng ở đây chính là **Service Discovery (Phát hiện dịch vụ tự động)**:
- Các container kết nối chung mạng ảo có thể giao tiếp với nhau bằng **Tên dịch vụ (Service Name)** khai báo trong file Compose (ví dụ: `quickbite-db`) thay vì sử dụng địa chỉ IP nội bộ không ổn định.
- Tên dịch vụ tự động được phân giải thành IP động tương ứng nhờ DNS nội bộ của Docker.

*(Nhấn mạnh)*: Mạng mặc định của Docker CLI (`default bridge`) không hề hỗ trợ DNS nội bộ. Chỉ có mạng ảo do người dùng tự định nghĩa (User-defined Bridge Network) hoặc mạng mặc định của Docker Compose mới được kích hoạt sẵn tính năng này.

### 2. Phần thực hành (Demo)

**[Live Demo: Kiểm chứng tính toàn vẹn dữ liệu qua Volume]**
Bây giờ, thầy sẽ hướng dẫn các bạn kiểm tra xem dữ liệu trong Postgres có thực sự được bảo vệ vĩnh viễn nhờ Named Volume khi tắt cụm dịch vụ hay không.

*(Thao tác trên Terminal)*:
- *Bước 1: Tạo dữ liệu thử nghiệm trong container database đang chạy bằng cách gửi lệnh truy vấn từ host:*
  ```bash
  docker compose exec quickbite-db psql -U postgres -d quickbite_user_db -c "CREATE TABLE test_volume (note VARCHAR(50)); INSERT INTO test_volume VALUES ('Du lieu an toan!');"
  ```
- *Bước 2: Tiến hành xóa bỏ toàn bộ cụm container bằng lệnh:*
  ```bash
  docker compose down
  ```
  *(Giải thích): Lệnh này dừng và xóa sạch các container, nhưng Named Volume `db-data` vẫn được giữ an toàn trên máy host.*
- *Bước 3: Khởi chạy lại cụm container:*
  ```bash
  docker compose up -d
  ```
- *Bước 4: Truy vấn lại bảng dữ liệu để xác minh:*
  ```bash
  docker compose exec quickbite-db psql -U postgres -d quickbite_user_db -c "SELECT * FROM test_volume;"
  ```
  *(Kết quả mong đợi): Dòng chữ 'Du lieu an toan!' hiển thị thành công. Dữ liệu hoàn toàn được bảo toàn nguyên vẹn.*

---

## Lesson 6: Quản lý vòng đời hệ thống với Docker Compose

### 1. Phần lý thuyết

**[Slide 17: Các lệnh điều khiển Docker Compose CLI chủ chốt]**
Khi vận hành dự án thực tế, các bạn sẽ phải thường xuyên quản lý chu kỳ hoạt động của hệ thống thông qua các câu lệnh Docker Compose CLI:
- **Khởi chạy hệ thống:**
  - `docker compose up -d`: Khởi chạy toàn bộ cụm dịch vụ chạy ngầm dưới nền.
  - `docker compose start`: Kích hoạt lại các container đã bị dừng (trạng thái Stopped).
- **Dừng và dọn dẹp:**
  - `docker compose stop`: Tạm dừng các container, giải phóng RAM và CPU nhưng giữ nguyên dữ liệu tạm trên đĩa.
  - `docker compose down`: Dừng và xóa bỏ hoàn toàn các container và mạng ảo của dự án để giải phóng tài nguyên.
  - *(Lưu ý rủi ro)*: Lệnh `docker compose down` mặc định giữ an toàn cho dữ liệu trong Named Volume. Dữ liệu chỉ bị xóa sạch khi các bạn truyền thêm cờ xóa volume: `docker compose down -v`.

### 2. Phần thực hành (Demo)

**[Live Demo: Các lệnh chẩn đoán hệ thống thực chiến]**
Cuối cùng, thầy sẽ hướng dẫn các bạn sử dụng các lệnh chẩn đoán trạng thái hoạt động của cụm container.

*(Thao tác trên Terminal)*:
- *Bước 1: Liệt kê trạng thái và cổng kết nối của các container đang hoạt động:*
  ```bash
  docker compose ps
  ```
- *Bước 2: Xem logs tổng hợp của toàn bộ cụm container để chẩn đoán lỗi:*
  ```bash
  docker compose logs -f --tail=50
  ```
  *(Giải thích): Lệnh này sẽ ghép log của tất cả các container lại, phân màu và hiển thị tiền tố dịch vụ ở đầu dòng để chúng ta dễ dàng theo dõi trình tự lỗi.*
- *Bước 3: Kiểm tra trạng thái sẵn sàng nhận kết nối của PostgreSQL:*
  ```bash
  docker exec -it quickbite-db pg_isready -U postgres
  ```
  *Màn hình trả về thông báo accepting connections là DB đã sẵn sàng.*
- *Bước 4: Chạy câu lệnh SQL trực tiếp từ máy host để test kết nối:*
  ```bash
  docker compose exec quickbite-db psql -U postgres -c "SELECT 1;"
  ```

Đó là toàn bộ quy trình thiết lập, đóng gói và vận hành một hệ thống nhiều container bằng Docker Compose. Hy vọng các bạn đã nắm vững tư duy và các câu lệnh thực tế này. Hẹn gặp lại các bạn ở các Session tiếp theo!
