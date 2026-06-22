# QUIZ LESSONS - SESSION 05

# LESSON 01: Kiến trúc đa dịch vụ và thiết kế thực thể trong hệ thống phân tán QuickBite

## Q1

Thử thách lớn nhất đối với việc thiết lập mối quan hệ giữa các bảng dữ liệu trong mô hình thiết kế Database-per-Service của kiến trúc microservices là gì?

[A]
Trình biên dịch Java sẽ báo lỗi cú pháp nếu phát hiện hai class thực thể nằm ở hai service khác nhau có cấu trúc trường giống nhau.
[EXP]
Trình biên dịch Java biên dịch từng microservice độc lập, không kiểm tra hay so khớp cấu trúc class thực thể giữa các dự án khác nhau.
[B]
Hệ quản trị cơ sở dữ liệu PostgreSQL không hỗ trợ chạy nhiều database độc lập trên cùng một container.
[EXP]
PostgreSQL hoàn toàn hỗ trợ chạy nhiều database độc lập trên cùng một instance/container.
[C]
Không thể sử dụng các quan hệ khóa ngoại vật lý hoặc liên kết JPA trực tiếp (như `@ManyToOne`, `@OneToMany`) xuyên cơ sở dữ liệu; bắt buộc phải liên kết lỏng qua ID và giao tiếp qua REST API.
[EXP]
Chính xác. Vì mỗi dịch vụ quản lý một cơ sở dữ liệu logic biệt lập (nằm trên các database server khác nhau), hệ thống không thể thiết lập khóa ngoại vật lý và các annotation liên kết JPA truyền thống. Các thực thể bắt buộc phải liên kết lỏng bằng trường ID (`Long`) và gọi API để lấy dữ liệu chéo.
[D]
Docker Engine tự động chặn toàn bộ các kết nối mạng giữa các container chứa cơ sở dữ liệu khác nhau.
[EXP]
Docker Engine không chặn kết nối mạng; các container hoàn toàn giao tiếp được với nhau nếu nằm chung mạng ảo (bridge network).


@correct: C
@point: 20

## Q2

Mục đích cốt lõi của việc áp dụng mô hình Snapshot (Snapshot Pattern) trong việc lưu trữ thông tin chi tiết đơn hàng (OrderDetail/OrderItem) là gì?

[A]
Giảm thiểu tối đa dung lượng bộ nhớ lưu trữ của cơ sở dữ liệu `order-service`.
[EXP]
Áp dụng Snapshot làm tăng dung lượng lưu trữ do phải nhân bản dữ liệu, nhưng đây là sự đánh đổi bắt buộc để đảm bảo tính toàn vẹn lịch sử giao dịch.
[B]
Cho phép hệ thống tự động đồng bộ hóa thông tin đơn hàng lên các nền tảng lưu trữ đám mây.
[EXP]
Snapshot Pattern giải quyết bài toán toàn vẹn dữ liệu nghiệp vụ, không liên quan đến việc đồng bộ hóa hạ tầng đám mây.
[C]
Bảo toàn lịch sử giao dịch bằng cách chụp và lưu cứng các thông tin động (như tên món, giá bán) tại thời điểm đặt đơn, tránh bị sai lệch dữ liệu khi nhà hàng thay đổi thực đơn trong tương lai.
[EXP]
Chính xác. Nếu đơn hàng chỉ lưu ID món ăn và truy xuất động từ `restaurant-service`, khi nhà hàng thay đổi giá hoặc tên món, thông tin hóa đơn lịch sử hiển thị cho khách hàng sẽ bị sai lệch so với thời điểm giao dịch phát sinh. Snapshot giúp lưu trữ bất biến thông tin tại thời điểm mua.
[D]
Ngăn chặn hoàn toàn việc khách hàng thực hiện thay đổi địa chỉ giao hàng sau khi đơn hàng đã được xác nhận.
[EXP]
Việc chặn đổi địa chỉ thuộc về logic nghiệp vụ kiểm soát trạng thái đơn hàng của `order-service`, không phải mục tiêu của Snapshot Pattern.


@correct: C
@point: 20

## Q3

Trong kiến trúc phân tán QuickBite, thực thể `Order` liên kết thông tin khách hàng từ `user-service` theo cơ chế nào để đảm bảo nguyên tắc cô lập dữ liệu?

[A]
Sử dụng chung một bảng dữ liệu dùng chung trong database được chia sẻ giữa hai dịch vụ.
[EXP]
Mô hình Database-per-Service cấm việc chia sẻ chung bảng dữ liệu trực tiếp giữa các microservices để tránh ràng buộc cứng.
[B]
Tự động sao chép toàn bộ bảng dữ liệu người dùng từ `user-service` sang database của `order-service` định kỳ.
[EXP]
Việc sao chép toàn bộ bảng dữ liệu vi phạm nguyên tắc cô lập và dẫn đến dư thừa dữ liệu cực lớn, khó kiểm soát nhất quán.
[C]
Sử dụng annotation `@ManyToOne` của JPA để liên kết trực tiếp bảng `orders` với bảng `users` nằm ở database khác.
[EXP]
JPA không hỗ trợ liên kết trực tiếp chéo database logic nằm ở các service khác nhau mà không thông qua cơ chế tích hợp đặc biệt.
[D]
Lưu trữ khóa chính dưới dạng một trường số nguyên thông thường (`customerId` kiểu `Long`) thay vì khai báo đối tượng thực thể `User`.
[EXP]
Chính xác. Để đảm bảo tính lỏng lẻo (loose coupling) và cô lập dữ liệu, thực thể `Order` chỉ lưu trường `customerId` để tham chiếu. Khi cần thông tin chi tiết, `order-service` sẽ gọi API sang `user-service` để lấy dữ liệu.


@correct: D
@point: 20

## Q4

Trong thiết kế hệ thống QuickBite, dịch vụ nào dưới đây chịu trách nhiệm trực tiếp quản lý ví tài khoản (`UserWallet`) và thực hiện trừ tiền mô phỏng khi thanh toán đơn hàng?

[A]
`user-service`
[EXP]
Đúng. `user-service` quản lý thông tin định danh (`User`), ví tiền (`UserWallet`) và các địa chỉ nhận hàng (`UserAddress`). Mọi yêu cầu liên quan đến số dư ví và trừ tiền đều phải gọi API đến dịch vụ này.
[B]
`notification-service`
[EXP]
`notification-service` chỉ chịu trách nhiệm tiếp nhận sự kiện và gửi thông báo, không tham gia vào luồng xử lý tài chính.
[C]
`order-service`
[EXP]
`order-service` chỉ quản lý thông tin đơn hàng và lịch sử trạng thái đơn, khi thanh toán phải gửi yêu cầu trừ tiền sang `user-service`.
[D]
`restaurant-service`
[EXP]
`restaurant-service` quản lý thực đơn và thông tin cửa hàng, không quản lý ví tiền hay tài khoản của khách hàng.


@correct: A
@point: 20

## Q5

Spring Cloud OpenFeign đóng vai trò gì trong việc triển khai giao tiếp giữa các dịch vụ trong Spring Boot?

[A]
Tự động biên dịch mã nguồn Java thành Docker Image mà không cần Dockerfile.
[EXP]
OpenFeign là thư viện giao tiếp mạng REST, không liên quan đến việc đóng gói hay biên dịch Docker Image.
[B]
Quản lý và cấp phát địa chỉ IP tĩnh cho các container trong mạng ảo Docker.
[EXP]
Địa chỉ IP của container do Docker Daemon quản lý, OpenFeign chỉ sử dụng tên dịch vụ để DNS của Docker phân giải IP tương ứng.
[C]
Mã hóa toàn bộ cơ sở dữ liệu của các microservices để đảm bảo an toàn thông tin.
[EXP]
OpenFeign không can thiệp vào tầng lưu trữ dữ liệu hay mã hóa cơ sở dữ liệu của ứng dụng.
[D]
Cung cấp một giải pháp khai báo (Declarative) giúp viết REST Client dưới dạng các Interface trực quan, tự động cấu hình và gọi API dựa trên tên dịch vụ.
[EXP]
Đúng. OpenFeign giúp loại bỏ boilerplate code của việc viết RestTemplate thủ công. Lập trình viên chỉ cần định nghĩa một Interface đi kèm annotation `@FeignClient(name = "service-name")`, Spring Cloud sẽ tự động sinh mã thực thi cuộc gọi HTTP.


@correct: D
@point: 20

---

# LESSON 02: Quy trình chạy Spring Boot Service cùng cơ sở dữ liệu bằng Docker Compose

## Q1

Mục đích chính của việc sử dụng tiền tố hóa tên dịch vụ (ví dụ: `USER_DB_NAME`, `RESTAURANT_DB_NAME`) trong tệp cấu hình `.env` dùng chung của dự án đa container là gì?

[A]
Bắt buộc hệ điều hành host phải mã hóa các tệp tin cấu hình trước khi nạp vào container.
[EXP]
Tiền tố hóa biến môi trường không có chức năng mã hóa dữ liệu trên máy host.
[B]
Để Docker Daemon tự động tạo các database logic tương ứng trong PostgreSQL.
[EXP]
Docker Daemon không tự động tạo database dựa trên tên biến môi trường trong `.env`; việc này do script khởi tạo `init-db.sql` đảm nhận.
[C]
Giúp giảm thời gian biên dịch mã nguồn Spring Boot khi đóng gói image.
[EXP]
Biến môi trường trong `.env` được nạp vào lúc runtime (khởi chạy container), hoàn toàn không ảnh hưởng đến thời gian biên dịch mã nguồn.
[D]
Tránh việc các khóa cấu hình bị trùng lặp và ghi đè giá trị lẫn nhau khi Docker Compose nạp biến môi trường dùng chung cho nhiều dịch vụ.
[EXP]
Chính xác. Nếu tất cả các dịch vụ đều sử dụng chung một key là `DB_NAME` trong file `.env`, Docker Compose sẽ không phân biệt được và có thể nạp sai cấu hình cho các service. Việc tiền tố hóa giúp phân định rõ ràng tham số của từng service trước khi ánh xạ vào biến môi trường của container.


@correct: D
@point: 20

## Q2

Khi thiết lập tệp `docker-compose.yml` cho các dịch vụ backend chạy cùng một database PostgreSQL độc lập có sẵn từ trước, cấu hình nào giúp các container kết nối được với database đó?

[A]
Ánh xạ toàn bộ các cổng kết nối của database ra ngoài internet.
[EXP]
Việc mở cổng database ra internet gây nguy cơ bảo mật nghiêm trọng và không cần thiết cho giao tiếp mạng nội bộ của Docker.
[B]
Cài đặt trực tiếp dịch vụ PostgreSQL vào bên trong từng container backend.
[EXP]
Điều này vi phạm nguyên tắc cô lập container và làm phình to kích thước image, lãng phí tài nguyên hệ thống.
[C]
Sử dụng chung một tệp cấu hình `application.yml` tĩnh cho tất cả các microservices.
[EXP]
Mỗi microservice có logic và thông số kết nối database khác nhau, bắt buộc phải sở hữu file cấu hình riêng biệt.
[D]
Khai báo chung một mạng ảo và đặt thuộc tính `external: true` cho mạng đó để tái sử dụng mạng ngoài do database stack tạo ra.
[EXP]
Đúng. Để kết nối với container database PostgreSQL đang chạy độc lập từ trước, các service backend phải tham gia chung vào mạng ảo của database. Trong file Compose của backend, ta khai báo mạng này và set `external: true` để yêu cầu Compose sử dụng lại mạng sẵn có thay vì tạo mạng mới.


@correct: D
@point: 20

## Q3

Tệp tin khởi tạo SQL (`init-db.sql`) được cấu hình trong container PostgreSQL hoạt động theo cơ chế nào khi khởi chạy hệ thống?

[A]
Được chạy định kỳ sau mỗi 24 giờ để tự động sao lưu dữ liệu của hệ thống.
[EXP]
`init-db.sql` là script khởi tạo cấu trúc và tài khoản, không phải công cụ sao lưu định kỳ.
[B]
Tự động thực thi lại toàn bộ câu lệnh mỗi khi có một microservice kết nối vào database.
[EXP]
Script không tự chạy lại khi có kết nối mới để bảo vệ dữ liệu hiện có không bị reset hoặc ghi đè.
[C]
Chỉ chạy khi máy chủ vật lý bị mất kết nối internet nhằm kích hoạt chế độ dự phòng.
[EXP]
Hoạt động của script hoàn toàn cục bộ trong container, không phụ thuộc vào trạng thái kết nối internet của máy host.
[D]
Chỉ chạy một lần duy nhất khi container database được khởi tạo lần đầu tiên và thư mục dữ liệu (Volume) còn trống.
[EXP]
Đúng. Thư mục `/docker-entrypoint-initdb.d/` của image Postgres chính thức chỉ thực thi các file script `.sql` hoặc `.sh` một lần duy nhất khi khởi tạo container database lần đầu (khi chưa có dữ liệu trong volume). Ở các lần khởi động sau, script này sẽ bị bỏ qua để tránh ghi đè dữ liệu.


@correct: D
@point: 20

## Q4

Trong tệp `docker-compose.yml`, khai báo nào dưới đây chỉ ra rằng một mạng ảo được quản lý bên ngoài file Compose hiện tại và không được tự động tạo mới?

[A]
```yaml
networks:
  quickbite-net:
    internal: true
```
[EXP]
Thuộc tính `internal` dùng để hạn chế container trong mạng không được kết nối ra ngoài internet, không có ý nghĩa chỉ định mạng ngoài sẵn có.
[B]
```yaml
networks:
  quickbite-net:
    reuse: true
```
[EXP]
Từ khóa `reuse: true` không tồn tại trong đặc tả cú pháp cấu hình mạng của Docker Compose.
[C]
```yaml
networks:
  quickbite-net:
    external: true
```
[EXP]
Chính xác. Khai báo thuộc tính `external: true` thông báo cho Docker Compose biết mạng `quickbite-net` đã được khởi tạo trước đó bởi một tiến trình hoặc file Compose khác, Compose chỉ cần kết nối các container hiện tại vào mạng này mà không tạo mới.
[D]
```yaml
networks:
  quickbite-net:
    driver: bridge
```
[EXP]
Khai báo này yêu cầu Compose tự khởi tạo một mạng ảo mới sử dụng driver bridge cục bộ cho dự án hiện hành.


@correct: C
@point: 20

## Q5

Để đảm bảo nguyên tắc cô lập dữ liệu cao nhất giữa các dịch vụ `user-service` và `restaurant-service` khi chạy chung một container PostgreSQL vật lý, kỹ sư hệ thống nên cấu hình như thế nào?

[A]
Cấu hình cho cả hai dịch vụ sử dụng chung tài khoản quản trị tối cao (`postgres`) để đơn giản hóa việc kết nối.
[EXP]
Sử dụng tài khoản `postgres` tối cao cho tất cả dịch vụ tạo ra lỗ hổng bảo mật lớn, khi một service bị tấn công, hacker có thể kiểm soát toàn bộ cơ sở dữ liệu của các service khác.
[B]
Mỗi dịch vụ kết nối vào một cơ sở dữ liệu logic riêng biệt và sử dụng tài khoản truy cập được phân quyền hạn chế riêng cho database đó.
[EXP]
Chính xác. Dù chạy chung một database server vật lý để tiết kiệm tài nguyên khi dev, việc tách biệt database logic (ví dụ: `quickbite_user` và `quickbite_restaurant`) kết hợp phân quyền tài khoản riêng giúp ngăn chặn việc gọi chéo dữ liệu trái phép ở tầng cơ sở dữ liệu.
[C]
Chỉ cho phép một dịch vụ hoạt động tại một thời điểm để tránh xung đột đọc ghi.
[EXP]
Các microservices phải hoạt động song song đồng thời để phục vụ các yêu cầu nghiệp vụ phức tạp của hệ thống.
[D]
Bắt buộc cả hai dịch vụ phải sử dụng chung một database logic nhưng chia nhỏ bảng bằng tiền tố.
[EXP]
Dùng chung database logic làm mất đi tính cô lập dữ liệu, các service có thể truy cập và sửa đổi bảng của nhau dễ dàng, vi phạm nguyên tắc microservices.


@correct: B
@point: 20

---

# LESSON 03: Giao tiếp liên dịch vụ qua Docker Network trong kiến trúc phân tán

## Q1

Cơ chế phân giải tên miền tự động (Service Discovery) qua DNS nội bộ của Docker Network hoạt động như thế nào khi container A gửi request tới `http://restaurant-service:8082`?

[A]
Docker Daemon chặn kết nối và yêu cầu lập trình viên phải cung cấp địa chỉ IP tĩnh vật lý của container đích.
[EXP]
Docker Daemon hỗ trợ tự động phân giải tên miền, giải phóng lập trình viên khỏi việc cấu hình IP tĩnh thủ công.
[B]
DNS Server tích hợp của Docker tự động tra cứu tên container `restaurant-service` và phân giải thành địa chỉ IP ảo hiện hành của container đó trong mạng.
[EXP]
Chính xác. Docker Network tích hợp một DNS Server nội bộ. Khi container gọi tên dịch vụ/tên container khác, DNS này sẽ tự động chuyển đổi tên đó thành địa chỉ IP động hiện tại của container đích, giúp kết nối luôn thông suốt mà không cần biết trước IP.
[C]
Request được định tuyến ra mạng internet công cộng để phân giải qua các DNS Server toàn cầu của Google (8.8.8.8).
[EXP]
Đây là kết nối mạng nội bộ ảo của Docker, các DNS toàn cầu không quản lý và không thể phân giải địa chỉ IP nội bộ của container.
[D]
Docker CLI tự động sửa đổi file cấu hình mạng `/etc/hosts` trên máy host vật lý để chuyển hướng request.
[EXP]
Docker không can thiệp hay sửa đổi file `/etc/hosts` trên hệ điều hành host; mọi quá trình phân giải diễn ra nội bộ trong mạng ảo của container.


@correct: B
@point: 20

## Q2

Tại sao trong môi trường Staging/Production, việc đóng toàn bộ các cổng mạng của các microservices nội bộ (như `8081`, `8082`) ra ngoài máy host vật lý lại là một quy chuẩn bảo mật bắt buộc?

[A]
Để ngăn chặn các truy cập trực tiếp trái phép từ bên ngoài máy host hoặc internet vào các dịch vụ nội bộ, buộc mọi traffic phải đi qua cửa ngõ kiểm soát duy nhất là API Gateway.
[EXP]
Chính xác. Nếu mở cổng microservices ra ngoài máy host, kẻ tấn công có thể quét cổng và gọi trực tiếp vào API nội bộ (vốn không có hoặc ít lớp bảo vệ hơn), bỏ qua hoàn toàn các bộ lọc an ninh, kiểm tra quyền hạn (Authorization) đặt tại API Gateway.
[B]
Để giảm thiểu lượng điện năng tiêu thụ và tải xử lý của các card mạng vật lý trên máy chủ.
[EXP]
Việc đóng cổng mạng là quy chuẩn an ninh thông tin, không liên quan đến việc tiết kiệm điện năng card mạng vật lý.
[C]
Do Docker Engine chỉ hỗ trợ ánh xạ tối đa một cổng mạng ra ngoài máy host vật lý tại một thời điểm.
[EXP]
Docker Engine hoàn toàn hỗ trợ ánh xạ không giới hạn số lượng cổng ra máy host, tùy thuộc vào số lượng cổng còn trống của máy host.
[D]
Vì các ứng dụng Spring Boot không thể khởi chạy nếu cấu hình cổng của nó bị trùng lặp với cổng của máy host.
[EXP]
Cổng của container là độc lập trong mạng ảo; nó chỉ bị trùng lặp nếu ta cố tình ánh xạ (NAT) trùng cổng ra ngoài máy host.


@correct: A
@point: 20

## Q3

Khi cấu hình OpenFeign client với nhãn `@FeignClient(name = "restaurant-service")`, làm thế nào thư viện này biết được địa chỉ IP của container đích để gửi request?

[A]
Nó đọc thông tin địa chỉ IP được lưu cứng trong file cấu hình `application.yml` của dịch vụ gọi.
[EXP]
Sử dụng OpenFeign kết hợp Docker DNS giúp loại bỏ việc lưu cứng địa chỉ IP trong file cấu hình, đảm bảo tính linh hoạt khi container đổi IP.
[B]
Nó tự động thực hiện quét toàn bộ dải mạng LAN vật lý của công ty để tìm kiếm container có tên tương ứng.
[EXP]
Feign chỉ hoạt động trong phạm vi mạng ảo của Docker được cấu hình, không thực hiện quét mạng LAN vật lý.
[C]
Nó yêu cầu hệ điều hành host phải cài đặt sẵn một dịch vụ DNS bên thứ ba như Consul hoặc Eureka.
[EXP]
Mạng ảo của Docker đã tích hợp sẵn DNS phân giải tên container, do đó không bắt buộc phải cài thêm Consul hay Eureka cho các hệ thống quy mô vừa và nhỏ.
[D]
Nó sử dụng tên dịch vụ `"restaurant-service"` làm hostname trong URL; DNS nội bộ của Docker Network sẽ tự động phân giải hostname này thành IP của container tương ứng.
[EXP]
Đúng. Nhãn `@FeignClient(name = "restaurant-service")` chỉ định tên đích. Khi thực thi gọi API, Feign sẽ tạo request tới `http://restaurant-service/...`. Docker Network nhận diện hostname này và DNS sẽ phân giải ra địa chỉ IP nội bộ chính xác của container.


@correct: D
@point: 20

## Q4

Nếu trong tệp `docker-compose.yml`, cấu hình của dịch vụ `user-service` hoàn toàn không khai báo thuộc tính `ports`, điều này ảnh hưởng như thế nào đến khả năng kết nối của hệ thống?

[A]
Toàn bộ các container khác trong cùng mạng Compose cũng bị chặn hoàn toàn kết nối tới `user-service`.
[EXP]
Kết nối nội bộ giữa các container trong cùng mạng ảo không bị ảnh hưởng bởi việc không ánh xạ cổng ra ngoài host.
[B]
`user-service` sẽ không thể khởi chạy thành công do lỗi thiếu cổng lắng nghe của hệ thống.
[EXP]
Dịch vụ vẫn khởi chạy bình thường và lắng nghe ở cổng nội bộ khai báo trong code (ví dụ: 8081).
[C]
Các container khác trong cùng mạng ảo vẫn kết nối được tới `user-service` bình thường, nhưng các ứng dụng chạy trực tiếp ở máy host (hoặc bên ngoài internet) không thể kết nối trực tiếp.
[EXP]
Chính xác. Việc không khai báo `ports` có nghĩa là không ánh xạ cổng ra máy host. Container vẫn mở cổng nội bộ nên các container khác cùng mạng ảo vẫn gọi được bình thường. Máy host vật lý sẽ bị chặn kết nối trực tiếp, giúp tăng tính an toàn cho dịch vụ.
[D]
Docker Engine sẽ tự động gán cổng mặc định là `80` cho container này để kết nối ra ngoài máy host.
[EXP]
Docker Engine không tự động ánh xạ cổng mặc định nếu lập trình viên không khai báo thuộc tính `ports`.


@correct: C
@point: 20

## Q5

Chuyện gì xảy ra với kết nối mạng giữa các dịch vụ khi một container (ví dụ: `restaurant-service`) bị lỗi và được Docker Compose tạo mới (recreate)?

[A]
Container mới bắt buộc phải sử dụng lại chính xác địa chỉ IP cũ để tránh xung đột cổng mạng.
[EXP]
Docker cấp phát IP động từ dải IP của mạng bridge, không bắt buộc container phải giữ nguyên IP cũ.
[B]
Các dịch vụ khác sẽ tự động chuyển sang kết nối trực tiếp qua giao lộ mạng của máy host vật lý.
[EXP]
Các dịch vụ chỉ kết nối qua cấu hình mạng ảo được định nghĩa, không tự động chuyển đổi cấu hình sang card mạng vật lý của host.
[C]
Hệ thống mạng bị tê liệt hoàn toàn và yêu cầu kỹ sư hệ thống phải khởi động lại toàn bộ máy chủ vật lý.
[EXP]
Docker tự động xử lý cập nhật mạng, không yêu cầu khởi động lại máy chủ vật lý hay làm sập toàn bộ hệ thống.
[D]
Container mới nhận một địa chỉ IP ảo mới, nhưng kết nối mạng vẫn thông suốt nhờ DNS nội bộ tự động cập nhật bản ghi phân giải tên miền tương ứng với IP mới.
[EXP]
Chính xác. Khi container bị tái tạo, địa chỉ IP của nó thường thay đổi. Tuy nhiên, vì các service gọi nhau bằng tên container thông qua DNS nội bộ, Docker DNS sẽ cập nhật IP mới này vào bảng phân giải, đảm bảo các cuộc gọi tiếp theo không bị lỗi.


@correct: D
@point: 20

---

# LESSON 04: API Gateway và điểm truy cập tập trung trong hệ thống Microservices

## Q1

Bất cập lớn nhất đối với ứng dụng phía Client (Web, Mobile) khi phải giao tiếp trực tiếp với nhiều microservices độc lập mà không qua API Gateway là gì?

[A]
Tốc độ mạng internet của khách hàng sẽ bị suy giảm nghiêm trọng khi gửi nhiều yêu cầu cùng lúc.
[EXP]
Tốc độ mạng phụ thuộc vào băng thông và nhà mạng của khách hàng, không bị ảnh hưởng trực tiếp bởi kiến trúc có hay không có Gateway.
[B]
Tải xử lý của thiết bị di động của khách hàng sẽ bị quá tải do phải mã hóa dữ liệu nhiều lần.
[EXP]
Client không bị quá tải do mã hóa dữ liệu; việc kết nối trực tiếp chỉ gây khó khăn về mặt quản lý cấu hình và bảo mật.
[C]
Client bắt buộc phải sử dụng hệ điều hành Linux mới có thể gửi được request đến các microservices.
[EXP]
Client chạy trên bất kỳ hệ điều hành nào (iOS, Android, Windows...) đều có thể gửi request HTTP tiêu chuẩn tới server.
[D]
Client phải quản lý quá nhiều địa chỉ endpoint/port khác nhau, dễ đụng phải lỗi chia sẻ tài nguyên chéo nguồn (CORS) trên trình duyệt, và trùng lặp mã nguồn xử lý xác thực (Auth).
[EXP]
Chính xác. Khi gọi trực tiếp, client phải cấu hình nhiều IP/cổng dịch vụ khác nhau. Trình duyệt sẽ chặn kết nối chéo cổng (CORS) nếu không cấu hình cho phép ở tất cả service. Ngoài ra, các dịch vụ đều phải tự viết code giải mã token/kiểm tra quyền gây trùng lặp mã nguồn lớn.


@correct: D
@point: 20

## Q2

API Gateway đóng vai trò làm "lá chắn an ninh" cho hệ thống microservices nội bộ bằng cơ chế nào?

[A]
Bắt buộc toàn bộ người dùng phải thay đổi mật khẩu định kỳ mỗi khi truy cập vào hệ thống.
[EXP]
Yêu cầu đổi mật khẩu thuộc về logic nghiệp vụ quản lý tài khoản của `user-service`, không do Gateway tự động cưỡng chế.
[B]
Làm điểm truy cập tập trung duy nhất, che giấu hoàn toàn cấu trúc mạng ảo nội bộ, địa chỉ IP và các cổng dịch vụ thực tế của các microservices phía sau.
[EXP]
Chính xác. Nhờ có API Gateway làm cửa ngõ duy nhất mở ra ngoài, các microservices nội bộ có thể đóng kín toàn bộ cổng mạng với thế giới bên ngoài. Hacker bên ngoài không thể dò quét hoặc biết được cấu trúc mạng và IP nội bộ của hệ thống.
[C]
Thực hiện mã hóa cứng toàn bộ ổ đĩa lưu trữ dữ liệu của container database PostgreSQL.
[EXP]
Mã hóa ổ đĩa là tính năng của tầng lưu trữ/hệ điều hành, không thuộc phạm vi xử lý của API Gateway.
[D]
Tự động ngắt kết nối internet của toàn bộ máy chủ khi phát hiện có yêu cầu truy cập từ địa chỉ IP lạ.
[EXP]
API Gateway không ngắt kết nối internet của máy chủ; nó chỉ lọc và điều hướng các request hợp lệ.


@correct: B
@point: 20

## Q3

Sự khác biệt bản chất về mặt chức năng giữa một API Gateway cấp độ ứng dụng (như Spring Cloud Gateway) và một Reverse Proxy cấp độ hạ tầng (như Nginx) là gì?

[A]
API Gateway chỉ chạy được trên môi trường đám mây; Reverse Proxy chỉ chạy được trên các máy chủ vật lý đặt tại văn phòng.
[EXP]
Cả hai công cụ đều có thể chạy trên bất kỳ hạ tầng nào (Cloud, On-premise, VM, Docker...).
[B]
API Gateway tự động thay thế toàn bộ mã nguồn của các microservices phía sau khi có lỗi xảy ra.
[EXP]
API Gateway chỉ điều hướng request, không có khả năng thay thế hay can thiệp vào mã nguồn của các dịch vụ khác.
[C]
Reverse Proxy bắt buộc phải sử dụng giao thức truyền tải dữ liệu dạng nhị phân; API Gateway chỉ hỗ trợ truyền tải văn bản thuần túy.
[EXP]
Cả hai đều hỗ trợ các giao thức mạng phổ biến như HTTP/1.1, HTTP/2, WebSockets...
[D]
API Gateway cấp độ ứng dụng tích hợp sâu với code ứng dụng để xử lý logic phức tạp (như xác thực quyền, giới hạn tải động); Reverse Proxy cấp độ hạ tầng tối ưu cho định tuyến hiệu năng cao, SSL Termination và phân phối tài nguyên tĩnh.
[EXP]
Chính xác. API Gateway (Spring Cloud Gateway) chạy trên nền JVM nên dễ dàng viết code custom filter can thiệp sâu vào logic nghiệp vụ (Auth, Rate Limiting). Nginx (Reverse Proxy) viết bằng C, tối ưu cực tốt ở tầng mạng/hạ tầng để cân bằng tải, phân phối file tĩnh và xử lý SSL.


@correct: D
@point: 20

## Q4

Làm thế nào API Gateway giúp giải quyết triệt để vấn đề "CORS Hell" trong hệ thống gồm nhiều dịch vụ?

[A]
Nó yêu cầu trình duyệt của khách hàng phải tắt tính năng bảo mật CORS trước khi truy cập ứng dụng.
[EXP]
Hệ thống không thể can thiệp cài đặt trình duyệt của người dùng cuối để tắt các tính năng bảo mật mặc định.
[B]
API Gateway tự động chuyển đổi toàn bộ request từ Client thành giao thức HTTPS bảo mật cao để vượt qua bộ lọc CORS.
[EXP]
CORS là chính sách bảo mật dựa trên nguồn gốc (Origin - Domain/Port), việc đổi sang HTTPS không tự động giải quyết được lỗi CORS nếu domain gọi khác nhau.
[C]
Nó thực hiện mã hóa toàn bộ mã nguồn Frontend để trình duyệt không phát hiện ra nguồn gốc gọi API.
[EXP]
Mã nguồn frontend chạy trên trình duyệt người dùng, API Gateway không thể can thiệp mã hóa mã nguồn frontend để bypass CORS.
[D]
Cấu hình CORS tập trung duy nhất tại API Gateway; các microservices phía sau chỉ nhận request nội bộ từ Gateway nên không cần cấu hình CORS độc lập.
[EXP]
Chính xác. Vì Client chỉ gửi request tới địa chỉ duy nhất của API Gateway, ta chỉ cần cấu hình cho phép CORS tại đầu vào Gateway. Khi Gateway chuyển tiếp request vào mạng nội bộ tới các service, đó là giao tiếp server-to-server nên không bị hạn chế bởi chính sách CORS của trình duyệt.


@correct: D
@point: 20

## Q5

Trong kiến trúc Microservices, việc trùng lặp mã nguồn xử lý Xác thực (Authentication) và Phân quyền (Authorization) được API Gateway giải quyết như thế nào?

[A]
Gateway tự động đồng bộ mã nguồn Java chứa logic xác thực sang tất cả các dịch vụ nội bộ khi khởi chạy.
[EXP]
Gateway không tự sao chép hay đồng bộ mã nguồn của nó sang các container khác.
[B]
Đưa logic xác thực và giải mã token JWT về xử lý tập trung tại API Gateway; Gateway chỉ chuyển tiếp các request hợp lệ kèm thông tin User đã được xác thực xuống các dịch vụ phía sau.
[EXP]
Đúng. Bằng cách thực hiện xác thực tập trung tại Gateway, các microservices phía sau không cần viết lại logic giải mã JWT hay check quyền. Gateway sẽ verify token, đính kèm thông tin user (ví dụ qua header) rồi gửi xuống nội bộ, giúp code nội bộ cực kỳ gọn nhẹ.
[C]
Nó tự động chuyển đổi mọi tài khoản người dùng thành quyền quản trị tối cao (Admin) khi đi qua cổng kiểm soát.
[EXP]
Điều này vi phạm nguyên tắc phân quyền và tạo ra lỗ hổng bảo mật nghiêm trọng cho hệ thống.
[D]
Nó yêu cầu cơ sở dữ liệu của tất cả dịch vụ phải sử dụng chung một bảng lưu trữ thông tin mật khẩu của người dùng.
[EXP]
Việc chia sẻ chung bảng dữ liệu vi phạm nguyên tắc cô lập dữ liệu của kiến trúc microservices.


@correct: B
@point: 20

---

# LESSON 05: Spring Cloud Gateway và định tuyến yêu cầu trong kiến trúc đa dịch vụ

## Q1

Một Route (Tuyến định tuyến) tiêu chuẩn trong tệp cấu hình của Spring Cloud Gateway được cấu thành từ các thành phần chính nào?

[A]
Route ID, Destination URI, Predicates (Điều kiện khớp) và Filters (Bộ lọc xử lý).
[EXP]
Chính xác. Cấu trúc một route gồm: `id` (định danh duy nhất), `uri` (địa chỉ đích của service phía sau), `predicates` (dùng để so khớp xem request có thuộc route này không, ví dụ khớp path), và `filters` (dùng để sửa đổi request/response).
[B]
API Version, Request Body, Response Status và Error Message.
[EXP]
Đây là các thành phần của một chu kỳ HTTP Request/Response thông thường, không phải thành phần định nghĩa tuyến đường của Gateway.
[C]
Service Name, Port Mapping, Database Connection và CPU Limits.
[EXP]
Đây là các thông số cấu hình hạ tầng của Docker Compose và JVM, không phải thành phần cấu tạo nên Route của Spring Cloud Gateway.
[D]
SSL Certificate, Domain Name, Load Balancer và Cache Control.
[EXP]
Đây là các cấu hình thường gặp ở tầng Reverse Proxy hạ tầng (như Nginx), không cấu thành nên đối tượng Route trong Spring Cloud Gateway.


@correct: A
@point: 20

## Q2

Bộ lọc `StripPrefix=2` trong cấu hình Filter của Spring Cloud Gateway thực hiện nhiệm vụ gì đối với request path `/api/v1/users/me` trước khi chuyển tiếp tới service đích?

[A]
Chặn kết nối nếu đường dẫn chứa nhiều hơn 2 dấu gạch chéo `/`.
[EXP]
Bộ lọc không chặn kết nối; việc chặn dựa trên các điều kiện của Predicates hoặc các filter bảo mật chuyên dụng.
[B]
Nhân bản request thành 2 yêu cầu độc lập gửi tới 2 service khác nhau.
[EXP]
StripPrefix chỉ thay đổi cấu trúc chuỗi đường dẫn của request hiện tại, không nhân bản hay nhân đôi request.
[C]
Cắt bỏ 2 phần tử đầu tiên của đường dẫn (ở đây là `/api` và `/v1`), chuyển tiếp đường dẫn đã rút gọn là `/users/me` tới service đích.
[EXP]
Chính xác. Bộ lọc `StripPrefix=n` dùng để loại bỏ `n` phần tử đầu tiên trong chuỗi dẫn đường dẫn URL. Với `StripPrefix=2`, đường dẫn `/api/v1/users/me` sẽ bị cắt `/api/v1` và chỉ gửi phần `/users/me` xuống dịch vụ nội bộ.
[D]
Thêm tiền tố `/api/v1` vào trước đường dẫn gốc của request.
[EXP]
Đây là chức năng của bộ lọc thêm tiền tố (PrefixPath), không phải của StripPrefix (cắt bớt tiền tố).


@correct: C
@point: 20

## Q3

Khi tích hợp dịch vụ API Gateway (`gateway-service`) vào hệ thống `docker-compose.yml` chung, cổng mạng nào của hệ thống nên được ánh xạ (ports) ra ngoài máy host vật lý?

[A]
Mở tất cả các cổng của microservices nội bộ (`8081`, `8082`, `8083`) và đóng cổng của Gateway.
[EXP]
Cấu hình này khiến Gateway mất tác dụng làm cửa ngõ bảo vệ và phơi bày toàn bộ các dịch vụ nội bộ ra ngoài.
[B]
Chỉ mở duy nhất cổng `8080` (hoặc cổng chạy của Gateway) ra ngoài máy host; đóng toàn bộ cổng kết nối trực tiếp của các microservices nội bộ.
[EXP]
Chính xác. Quy chuẩn thiết kế an toàn yêu cầu chỉ mở cổng của Gateway (ví dụ: `8080`) ra ngoài để làm cửa ngõ tiếp nhận duy nhất. Các microservices khác chạy ẩn hoàn toàn trong mạng ảo Docker và chỉ giao tiếp với Gateway, không mở cổng trực tiếp ra host.
[C]
Mở cổng `5432` của database PostgreSQL ra ngoài và đóng toàn bộ các cổng chạy web service.
[EXP]
Mở cổng database ra ngoài internet mà không có lớp bảo vệ mạng cực kỳ nguy hiểm, dễ bị tấn công brute force hoặc rò rỉ dữ liệu.
[D]
Không ánh xạ bất kỳ cổng nào ra ngoài máy host để đảm bảo tính an toàn tuyệt đối.
[EXP]
Nếu không mở bất kỳ cổng nào (kể cả Gateway), Client bên ngoài (Web/Mobile) sẽ hoàn toàn không thể kết nối và sử dụng các dịch vụ của hệ thống.


@correct: B
@point: 20

## Q4

Thành phần `Predicate` trong Spring Cloud Gateway đóng vai trò cụ thể nào dưới đây?

[A]
Tự động biên dịch mã nguồn Java sang ngôn ngữ máy trước khi điều hướng.
[EXP]
Predicate là logic so khớp gói tin mạng ở thời điểm runtime, không tham gia vào quá trình biên dịch mã nguồn.
[B]
Thực hiện mã hóa dữ liệu của gói tin response gửi trả về cho Client.
[EXP]
Mã hóa dữ liệu response thường do các Filter hoặc cấu hình bảo mật SSL của web server đảm nhận.
[C]
Là tập hợp các điều kiện logic (như so khớp Path, Header, Method) để Gateway quyết định xem có áp dụng Route này cho request hiện tại hay không.
[EXP]
Đúng. Predicate hoạt động như một bộ lọc điều kiện. Nếu request thỏa mãn điều kiện của Predicate (ví dụ: đường dẫn bắt đầu bằng `/api/v1/orders/**`), Gateway sẽ chọn route tương ứng để xử lý tiếp.
[D]
Chịu trách nhiệm ghi đè các tham số cấu hình database của ứng dụng Spring Boot.
[EXP]
Việc nạp cấu hình database do tầng Spring Boot Core/Spring Cloud Config xử lý, không liên quan đến Predicate của Gateway.


@correct: C
@point: 20

## Q5

Trong file cấu hình định tuyến của Spring Cloud Gateway, nếu cấu hình URI của một route dạng `uri: http://user-service:8081`, tên miền `user-service` đại diện cho đối tượng nào?

[A]
Tên của container/dịch vụ chạy ứng dụng `user-service` được định nghĩa trong mạng ảo của Docker Compose.
[EXP]
Đúng. Trong mạng ảo Docker, tên service trong file Compose đóng vai trò là hostname. Khi cấu hình `http://user-service:8081`, Gateway sẽ gửi yêu cầu trực tiếp tới container mang tên này, nhờ DNS nội bộ của Docker phân giải IP.
[B]
Tên miền public đã được đăng ký bản quyền trên các nhà cung cấp DNS toàn cầu.
[EXP]
Đây là hostname nội bộ trong mạng ảo Docker, không thể truy cập từ internet công cộng nếu không đi qua Gateway.
[C]
Địa chỉ IP vật lý cố định của máy chủ host chạy hệ thống.
[EXP]
Đây là tên container ảo, không phải địa chỉ IP vật lý của máy host. Địa chỉ IP của container là động và do Docker tự quản lý.
[D]
Tên của class thực thể chính được khai báo trong mã nguồn Java của Gateway.
[EXP]
Hostname trong URI cấu hình định tuyến đại diện cho thực thể mạng (container/service), không liên quan đến các class Java trong code.


@correct: A
@point: 20

---

# LESSON 06: Luồng yêu cầu đặt hàng end-to-end từ Client qua API Gateway

## Q1

Tại sao các giao dịch database truyền thống sử dụng annotation `@Transactional` lại không thể tự động bảo toàn tính nhất quán dữ liệu xuyên suốt các dịch vụ trong kiến trúc microservices?

[A]
Vì mỗi dịch vụ sở hữu cơ sở dữ liệu logic cô lập riêng biệt; cơ chế rollback của một database không thể tự động can thiệp và hoàn tác dữ liệu trên database của dịch vụ khác qua kết nối mạng.
[EXP]
Chính xác. `@Transactional` hoạt động dựa trên kết nối cục bộ của một database cụ thể. Trong microservices (Database-per-service), khi `order-service` gọi REST API sang `user-service` để trừ tiền, hai hành động này diễn ra trên hai database độc lập qua môi trường mạng. Nếu bước sau lỗi, database của `order-service` rollback cũng không thể tự động hoàn tiền trong database của `user-service`.
[B]
Do Spring Boot tự động vô hiệu hóa tính năng `@Transactional` khi ứng dụng được đóng gói thành Docker Image.
[EXP]
Annotation `@Transactional` vẫn hoạt động bình thường trong container đối với các tác vụ nội bộ cơ sở dữ liệu của chính service đó.
[C]
Vì Docker Engine không hỗ trợ giao thức truyền tải dữ liệu có tính chất giao dịch (Transaction Protocol).
[EXP]
Docker Engine chỉ cung cấp hạ tầng ảo hóa mạng và tài nguyên, việc quản lý giao dịch hoàn toàn nằm ở tầng ứng dụng và hệ quản trị cơ sở dữ liệu.
[D]
Do các database của microservices bắt buộc phải sử dụng các hệ quản trị cơ sở dữ liệu khác nhau.
[EXP]
Dù các microservices dùng chung một loại cơ sở dữ liệu (ví dụ đều dùng PostgreSQL), tính cô lập vật lý/logic giữa các DB vẫn ngăn cản việc dùng chung transaction cục bộ.


@correct: A
@point: 20

## Q2

Trong kiến trúc microservices, cơ chế Giao dịch bù (Compensating Transaction) đóng vai trò gì để giải quyết sự cố phân tán?

[A]
Tự động biên dịch lại mã nguồn của các dịch vụ để tự sửa lỗi logic.
[EXP]
Giao dịch bù là logic nghiệp vụ được lập trình sẵn để xử lý hoàn tác dữ liệu khi có lỗi xảy ra ở runtime, không biên dịch lại code.
[B]
Tự động gửi email thông báo lỗi cho toàn bộ đội ngũ phát triển hệ thống để sửa lỗi thủ công.
[EXP]
Mục tiêu của giao dịch bù là tự động hoàn tác để đảm bảo trải nghiệm khách hàng và tính chính xác của số dư ví ngay lập tức, không phụ thuộc vào việc can thiệp thủ công.
[C]
Thực hiện các hành động hoàn tác bằng code (như gọi API cộng lại tiền vào ví) để khôi phục trạng thái nhất quán của hệ thống khi một bước trong chuỗi nghiệp vụ phân tán bị thất bại.
[EXP]
Chính xác. Khi một bước trong luồng nghiệp vụ phân tán bị lỗi (ví dụ: nhà hàng từ chối đơn hàng sau khi khách đã bị trừ tiền), hệ thống phải chủ động kích hoạt một giao dịch bù (Compensating Transaction) bằng code để hoàn tác các bước thành công trước đó (gọi API trả lại tiền vào ví của khách).
[D]
Yêu cầu hệ quản trị cơ sở dữ liệu tự động khôi phục lại dữ liệu từ tệp sao lưu (Backup file) gần nhất.
[EXP]
Đây là quá trình khôi phục thảm họa (Disaster Recovery), không phải cơ chế xử lý nhất quán giao dịch phân tán thời gian thực.


@correct: C
@point: 20

## Q3

Trong luồng nghiệp vụ đặt món ăn của QuickBite, nếu nhà hàng từ chối chuẩn bị món ăn vì hết nguyên liệu, dịch vụ nào chịu trách nhiệm kích hoạt giao dịch bù để hoàn tiền cho khách hàng?

[A]
`order-service`
[EXP]
Đúng. `order-service` đóng vai trò điều phối luồng nghiệp vụ đặt đơn. Khi nhận tín hiệu từ chối từ `restaurant-service`, `order-service` sẽ chuyển trạng thái đơn thành `CANCELLED` và chủ động gửi yêu cầu gọi API sang `user-service` để hoàn tiền vào ví khách hàng.
[B]
`notification-service`
[EXP]
`notification-service` chỉ gửi thông báo thông tin tới người dùng, không tham gia điều phối luồng tài chính hoàn tiền.
[C]
`user-service`
[EXP]
`user-service` chỉ quản lý ví và thực hiện cộng/trừ tiền khi nhận yêu cầu, không tự biết trạng thái đơn hàng để chủ động hoàn tiền nếu không có yêu cầu từ `order-service`.
[D]
`restaurant-service`
[EXP]
`restaurant-service` chỉ quản lý thực đơn và báo trạng thái từ chối của nhà hàng, việc điều phối tài chính của đơn hàng thuộc trách nhiệm của `order-service`.


@correct: A
@point: 20

## Q4

Khi thực hiện tích hợp và chạy thử nghiệm luồng đặt hàng end-to-end qua Gateway, việc sử dụng lệnh stream nhật ký tập trung `docker compose logs -f` mang lại lợi ích gì cho việc gỡ lỗi?

[A]
Hiển thị toàn bộ nhật ký của tất cả các container đan xen theo trình tự thời gian thực, giúp kỹ sư dễ dàng theo dõi đường đi của request và vị trí phát sinh lỗi liên dịch vụ.
[EXP]
Chính xác. Lệnh `logs -f` gộp chung luồng log của tất cả container. Nhìn vào log tập trung được phân biệt màu sắc và nhãn dịch vụ, kỹ sư có thể theo dõi chính xác trình tự: Gateway nhận request -> chuyển sang Order -> gọi sang User trừ tiền -> gọi sang Restaurant... Từ đó phát hiện nhanh dịch vụ nào bị sập hoặc trả về lỗi.
[B]
Tự động sửa lỗi logic trong mã nguồn Spring Boot của dịch vụ phát sinh lỗi.
[EXP]
Lệnh xem log chỉ hiển thị thông tin nhật ký đầu ra (STDOUT/STDERR), không có khả năng tự sửa lỗi trong mã nguồn.
[C]
Tự động chặn đứng các cuộc tấn công từ chối dịch vụ (DDoS) đi vào hệ thống.
[EXP]
Đây là tính năng của tường lửa (Firewall) hoặc API Gateway cấu hình rate limiting, không phải của lệnh xem log.
[D]
Tăng tốc độ xử lý các truy vấn cơ sở dữ liệu của PostgreSQL lên gấp đôi.
[EXP]
Stream log không ảnh hưởng đến tốc độ hay hiệu năng xử lý truy vấn của cơ sở dữ liệu.


@correct: A
@point: 20

## Q5

Trong quy trình đặt đơn 6 bước của QuickBite, tại sao đơn hàng phải được lưu vào cơ sở dữ liệu ở trạng thái `PENDING` ngay từ bước đầu tiên trước khi thực hiện thanh toán?

[A]
Để bắt buộc khách hàng không được phép hủy đơn trong suốt quá trình xử lý thanh toán.
[EXP]
Trạng thái `PENDING` chỉ là bước đệm kỹ thuật để hệ thống xử lý, không liên quan đến việc hạn chế quyền hủy đơn của khách hàng.
[B]
Vì cơ sở dữ liệu PostgreSQL không cho phép tạo đơn hàng ở trạng thái nào khác ngoài `PENDING`.
[EXP]
PostgreSQL chỉ lưu trữ dữ liệu dạng text/enum theo thiết kế của lập trình viên, không áp đặt trạng thái mặc định nào cho đơn hàng.
[C]
Để lưu vết và khởi tạo trạng thái giao dịch trên hệ thống, làm căn cứ đối chiếu và theo dõi tiến độ xử lý của các bước tiếp theo (Thanh toán, Báo món).
[EXP]
Đúng. Việc lưu đơn ở trạng thái `PENDING` ngay từ đầu giúp hệ thống ghi nhận có một giao dịch đang được xử lý. Nếu các bước sau (như trừ tiền ở `user-service`) bị lỗi mạng hoặc crash, hệ thống vẫn có dữ liệu đơn hàng `PENDING` để đối soát, xử lý lỗi hoặc thực hiện giao dịch bù tương ứng.
[D]
Để API Gateway tự động tối ưu hóa băng thông mạng cho request hiện tại.
[EXP]
Lưu trữ trạng thái đơn hàng trong DB là tác vụ của `order-service`, hoàn toàn độc lập với việc quản lý băng thông của API Gateway.


@correct: C
@point: 20
