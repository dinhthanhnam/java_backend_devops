# QUIZ LESSONS - SESSION 11

# LESSON 01: Khái niệm Reverse Proxy và vai trò của Nginx trong kiến trúc Microservices

## Q1

Điểm khác biệt cốt lõi về vai trò đại diện giữa Forward Proxy và Reverse Proxy là gì?

[A]
Forward Proxy đại diện cho máy chủ đích để nhận dữ liệu, trong khi Reverse Proxy đại diện cho mạng Internet toàn cầu để phân phối luồng dữ liệu.
[EXP]
Sai. Cả hai loại proxy đều không đại diện cho mạng Internet toàn cục mà đóng vai trò trung gian đứng trước Client hoặc Server.
[B]
Forward Proxy đứng trước Client và làm đại diện cho Client; Reverse Proxy đứng trước Server và làm đại diện cho Server.
[EXP]
Chính xác. Forward Proxy giúp Client gửi yêu cầu ẩn danh ra ngoài (như VPN), còn Reverse Proxy đại diện cho hệ thống máy chủ để đón nhận và phân phối các yêu cầu từ bên ngoài gửi vào.
[C]
Forward Proxy chỉ cho phép truyền dữ liệu qua HTTP, trong khi Reverse Proxy bắt buộc toàn bộ dữ liệu truyền qua HTTPS.
[EXP]
Sai. Việc sử dụng giao thức HTTP hay HTTPS là cấu hình của từng proxy, không phải điểm khác biệt cốt lõi để phân biệt Forward và Reverse.
[D]
Forward Proxy hoạt động ở tầng vật lý của mạng, còn Reverse Proxy chỉ hoạt động ở tầng ứng dụng Java Spring Boot.
[EXP]
Sai. Cả hai loại proxy đều hoạt động ở các tầng mạng của hệ điều hành, độc lập với Java Spring Boot.

@correct: B
@point: 20

## Q2

Tại sao việc đặt Nginx làm Reverse Proxy đứng ở biên hệ thống (cổng 80/443) lại giúp giảm diện tấn công (Attack Surface) của VPS?

[A]
Vì Nginx sẽ tự động phát hiện và chặn các cuộc tấn công DDoS bằng cách tắt kết nối Internet của toàn bộ máy chủ VPS ngay lập tức.
[EXP]
Sai. Nginx không tắt kết nối mạng của VPS khi bị DDoS mà lọc các request hoặc trả về mã lỗi thích hợp.
[B]
Vì Nginx tự động mã hóa toàn bộ dữ liệu trong cơ sở dữ liệu PostgreSQL khiến hacker không thể đọc được thông tin.
[EXP]
Sai. Nginx chỉ xử lý lưu lượng mạng (HTTP/HTTPS), không can thiệp vào việc mã hóa dữ liệu lưu trữ vật lý trong Database.
[C]
Vì tường lửa VPS chỉ cần mở duy nhất cổng 80/443 của Nginx, toàn bộ các cổng microservices khác được chặn và cô lập an toàn ở localhost.
[EXP]
Chính xác. Khi có Nginx làm Reverse Proxy, các cổng dịch vụ nội bộ (như 8080, 8081...) không cần mở ra Internet mà được ẩn giấu hoàn toàn, chỉ cho phép Nginx chuyển tiếp request vào.
[D]
Vì Nginx tự động chuyển đổi mã nguồn Java Spring Boot thành dạng mã máy nhị phân trước khi phản hồi cho Client.
[EXP]
Sai. Nginx chỉ chuyển tiếp các gói tin HTTP/HTTPS, việc biên dịch mã nguồn Java là do JVM/JDK thực hiện.

@correct: C
@point: 20

## Q3

Mô hình "SSL Termination" tại Nginx Reverse Proxy hoạt động theo cơ chế nào để tối ưu hiệu năng cho hệ thống microservices?

[A]
Nginx yêu cầu Client tự giải mã toàn bộ dữ liệu trước khi gửi request tới máy chủ VPS để tiết kiệm CPU.
[EXP]
Sai. Client (Trình duyệt) thực hiện mã hóa request và giải mã response; việc giải mã request HTTPS gửi đến do máy chủ đảm nhận.
[B]
Nginx tiếp nhận kết nối HTTPS bảo mật, tiến hành giải mã dữ liệu, rồi chuyển tiếp request dạng HTTP thường vào các service backend bên trong.
[EXP]
Chính xác. Bằng cách thực hiện giải mã SSL tại biên (Nginx), các service backend phía sau (như Spring Boot) chỉ cần xử lý request HTTP thường, giúp tiết kiệm tài nguyên CPU cho các container Java.
[C]
Nginx từ chối tất cả kết nối HTTPS cổng 443 và buộc Client phải chuyển hẳn sang dùng HTTP thường cổng 80 để tối ưu tốc độ.
[EXP]
Sai. Nginx vẫn nhận kết nối HTTPS cổng 443 bình thường để bảo mật đường truyền giữa Client và VPS.
[D]
Nginx tự động sinh ra một khóa bảo mật mới cho mỗi request của người dùng và lưu trữ khóa đó trong container database.
[EXP]
Sai. Nginx sử dụng khóa SSL cố định cấu hình trên máy chủ để bắt tay bảo mật (SSL Handshake), không sinh khóa mới cho từng request để lưu vào DB.

@correct: B
@point: 20

## Q4

Lợi ích của việc Nginx có khả năng xử lý trực tiếp các yêu cầu tải tĩnh (Static Content) là gì?

[A]
Giúp tự động sửa các lỗi cú pháp HTML/CSS của lập trình viên trước khi hiển thị cho người dùng cuối.
[EXP]
Sai. Nginx chỉ trả file tĩnh dưới dạng nguyên bản, không có khả năng phân tích hay tự sửa lỗi cú pháp code Frontend.
[B]
Ngăn chặn hoàn toàn việc người dùng tải các file ảnh có kích thước lớn lên máy chủ VPS để bảo vệ ổ đĩa.
[EXP]
Sai. Nginx chỉ hỗ trợ giới hạn dung lượng tải lên qua cấu hình, lợi ích chính của static content ở đây là cải thiện tốc độ phản hồi file tĩnh.
[C]
Giải phóng tài nguyên cho Spring Boot bằng cách trực tiếp trả các file HTML/CSS/JS mà không cần gọi vào ứng dụng Java.
[EXP]
Chính xác. Nginx đọc ghi file cực nhanh trên đĩa cứng; việc để Nginx gánh tải file tĩnh giúp Spring Boot chỉ cần tập trung xử lý các API nghiệp vụ cốt lõi.
[D]
Tự động biên dịch mã nguồn React/Angular của ứng dụng thành file JAR để chạy ngầm trên VPS.
[EXP]
Sai. Nginx không biên dịch mã nguồn React/Angular ra file JAR, đó là nhiệm vụ của Node.js/Build tools ở local hoặc CI/CD.

@correct: C
@point: 20

## Q5

Nếu không sử dụng Nginx Reverse Proxy mà mở trực tiếp các cổng `8080-8084` của microservices ra Internet, rủi ro lớn nhất về mặt quản trị là gì?

[A]
Client bên ngoài bắt buộc phải sử dụng hệ điều hành Ubuntu mới có thể gửi được request tới hệ thống.
[EXP]
Sai. Client sử dụng bất kỳ hệ điều hành nào (Windows, macOS, iOS, Android) đều gửi được request qua giao thức HTTP/HTTPS.
[B]
Khó quản lý chứng chỉ SSL/HTTPS do phải cấu hình và cập nhật mã hóa bảo mật trên từng container Spring Boot riêng lẻ.
[EXP]
Chính xác. Khi mở trực tiếp nhiều cổng, bạn sẽ phải cài đặt và cấu hình chứng chỉ SSL trên từng container dịch vụ đơn lẻ, gây khó khăn cho việc gia hạn và quản lý tập trung.
[C]
Docker sẽ tự động ngắt kết nối mạng của VPS nếu phát hiện có nhiều hơn 2 cổng dịch vụ đang mở ra ngoài.
[EXP]
Sai. Docker hoàn toàn cho phép map bao nhiêu cổng tùy ý ra host nếu tài nguyên phần cứng cho phép.
[D]
Toàn bộ mã nguồn Java của các service sẽ bị hiển thị công khai trên giao diện web của Client khi truy cập.
[EXP]
Sai. Spring Boot chỉ trả về kết quả API (JSON/XML), không bao giờ tự động hiển thị mã nguồn Java thô cho Client.

@correct: B
@point: 20

---

# LESSON 02: Cấu trúc tệp cấu hình Nginx và các khối khai báo chính (Configuration Blocks)

## Q1

Tại sao DevOps được khuyến nghị giữ nguyên tệp cấu hình gốc `nginx.conf` và chỉ viết các tệp cấu hình con trong thư mục `/etc/nginx/conf.d/`?

[A]
Vì tệp cấu hình gốc `nginx.conf` bị khóa quyền ghi ở cấp độ phần cứng VPS và không thể chỉnh sửa.
[EXP]
Sai. Với quyền root/sudo, bạn hoàn toàn chỉnh sửa được file `nginx.conf` gốc, nhưng việc này không được khuyến nghị.
[B]
Để tránh rủi ro làm hỏng cấu hình toàn cục và dễ dàng quản lý, bật/tắt cấu hình của từng dự án một cách độc lập.
[EXP]
Chính xác. File gốc chứa các thiết lập hệ thống phức tạp. Sử dụng thư mục `conf.d/` giúp phân tách cấu hình rõ ràng, tránh xung đột và dễ quản lý dự án.
[C]
Vì Let's Encrypt chỉ chấp nhận cấp phát chứng chỉ SSL cho các tệp cấu hình nằm trong thư mục `/etc/nginx/conf.d/`.
[EXP]
Sai. Certbot có thể tự động cấu hình SSL trên bất kỳ tệp cấu hình hợp lệ nào được Nginx nạp, không bắt buộc phải ở `conf.d/`.
[D]
Vì Nginx sẽ chạy nhanh gấp đôi nếu các chỉ thị cấu hình được chia nhỏ ra nhiều tệp tin khác nhau.
[EXP]
Sai. Việc chia nhỏ file cấu hình chỉ phục vụ cho mục đích quản trị và bảo trì của con người, không ảnh hưởng đến tốc độ thực thi của Nginx.

@correct: B
@point: 20

## Q2

Khác biệt cơ bản về mặt cấu trúc viết file giữa tệp cấu hình con trong `/etc/nginx/conf.d/` và tệp gốc `/etc/nginx/nginx.conf` là gì?

[A]
Các tệp cấu hình con bắt buộc phải bắt đầu bằng khối `main { ... }` để liên kết với tệp gốc.
[EXP]
Sai. Không tồn tại khối nào tên là `main { ... }` trong cú pháp cấu hình của Nginx.
[B]
Các tệp cấu hình con chỉ chứa các khối `server` và `location` độc lập, không cần bọc bởi khối `http` hay `events`.
[EXP]
Chính xác. Do tệp cấu hình con được nạp qua lệnh `include` nằm sẵn trong khối `http` của tệp gốc, nên các file con chỉ cần khai báo trực tiếp các block `server` và `location` tương ứng.
[C]
Các tệp cấu hình con bắt buộc phải được mã hóa dưới dạng file nhị phân trước khi đưa vào thư mục `conf.d/`.
[EXP]
Sai. Tất cả các tệp cấu hình Nginx đều là tệp văn bản thô (Plain text).
[D]
Các tệp cấu hình con không được phép sử dụng chỉ thị `listen` để khai báo cổng mạng.
[EXP]
Sai. Chỉ thị `listen` là bắt buộc trong khối `server` của các tệp con để định nghĩa cổng lắng nghe của máy chủ ảo.

@correct: B
@point: 20

## Q3

Trong các khối `location` dưới đây, khối nào có độ ưu tiên so khớp đường dẫn URI cao nhất khi Nginx nhận được request?

[A]
`location /api/v1`
[EXP]
Sai. Đây là khớp tiền tố mặc định, có độ ưu tiên thấp nhất trong các modifier.
[B]
`location ~* \.json$`
[EXP]
Sai. Khớp Regex có độ ưu tiên cao hơn khớp tiền tố mặc định nhưng vẫn thấp hơn khớp chính xác.
[C]
`location = /api/v1/users`
[EXP]
Chính xác. Modifier `=` biểu thị khớp chính xác (Exact match). Nginx sẽ ưu tiên kiểm tra và áp dụng ngay cấu hình này nếu URI khớp hoàn hảo 100%.
[D]
`location ^~ /api/v1/`
[EXP]
Sai. Modifier `^~` biểu thị khớp tiền tố ưu tiên, có độ ưu tiên cao hơn Regex nhưng vẫn thấp hơn khớp chính xác `=`.

@correct: C
@point: 20

## Q4

Sự khác biệt về hành vi so khớp đường dẫn của modifier `~` so với `~*` trong khối location của Nginx là gì?

[A]
Modifier `~` dùng để so khớp đường dẫn tuyệt đối, còn `~*` dùng để so khớp đường dẫn tương đối.
[EXP]
Sai. Cả hai modifier này đều dùng Regex để so khớp đường dẫn URI của request.
[B]
Modifier `~` thực hiện so khớp phân biệt chữ hoa/chữ thường, còn `~*` thực hiện so khớp KHÔNG phân biệt chữ hoa/chữ thường.
[EXP]
Chính xác. Đây là định nghĩa cú pháp Regex của Nginx: `~` phân biệt hoa thường, `~*` bỏ qua sự khác biệt hoa thường.
[C]
Modifier `~` chỉ hỗ trợ các tệp tin hình ảnh, còn `~*` hỗ trợ toàn bộ các định dạng tệp tin khác.
[EXP]
Sai. Định dạng tệp tin được quyết định bởi biểu thức chính quy (Regex pattern) đi kèm, không phụ thuộc vào modifier.
[D]
Modifier `~` yêu cầu Nginx phải kiểm tra lại cơ sở dữ liệu trước khi quyết định so khớp.
[EXP]
Sai. Nginx so khớp chuỗi URI thuần túy bằng bộ nhớ đệm, hoàn toàn không tương tác với bất kỳ cơ sở dữ liệu nào.

@correct: B
@point: 20

## Q5

Khi Nginx tìm thấy một location khớp tiền tố ưu tiên `location ^~ /assets/`, hành vi tiếp theo của nó là gì?

[A]
Tiếp tục quét qua tất cả các location sử dụng Regular Expression (`~` hoặc `~*`) để tìm kiếm so khớp tốt hơn.
[EXP]
Sai. Đây là hành vi của khớp tiền tố mặc định (không dùng modifier). Với `^~`, Nginx sẽ dừng quét Regex.
[B]
Tự động tải toàn bộ nội dung trong thư mục `/assets/` về bộ nhớ RAM của VPS để tăng tốc độ phản hồi.
[EXP]
Sai. Nginx chỉ áp dụng cấu hình xử lý request, không tự động load thư mục lên RAM.
[C]
Áp dụng luôn cấu hình trong khối này để xử lý request và dừng ngay tiến trình quét qua các khối location Regex khác.
[EXP]
Chính xác. Modifier `^~` có ý nghĩa là "khớp tiền tố và dừng quét Regex" (non-regular expression match), giúp tối ưu hiệu năng bỏ qua bước duyệt Regex.
[D]
Chuyển hướng request này sang cổng mặc định của database PostgreSQL.
[EXP]
Sai. Nginx chỉ bẻ hướng xử lý dựa trên các chỉ thị bên trong khối, không tự động chuyển hướng sang DB.

@correct: C
@point: 20

---

# LESSON 03: Triển khai Nginx làm Reverse Proxy chuyển tiếp yêu cầu đến API Gateway

## Q1

Để cấu hình Nginx chuyển tiếp yêu cầu từ cổng 80 của VPS vào cổng 8080 của API Gateway, chỉ thị nào sau đây được sử dụng trong khối `location`?

[A]
`proxy_pass http://127.0.0.1:8080;`
[EXP]
Chính xác. Chỉ thị `proxy_pass` kèm địa chỉ máy chủ đích (Upstream) được sử dụng để chuyển tiếp toàn bộ request khớp với location đó tới target server.
[B]
`forward_to http://127.0.0.1:8080;`
[EXP]
Sai. Trong cú pháp cấu hình Nginx không tồn tại chỉ thị nào tên là `forward_to`.
[C]
`redirect_port 8080;`
[EXP]
Sai. Không tồn tại chỉ thị `redirect_port` trong Nginx.
[D]
`proxy_redirect http://127.0.0.1:8080;`
[EXP]
Sai. Chỉ thị `proxy_redirect` dùng để thay đổi các Location headers trong HTTP response từ upstream gửi về, không phải để chuyển tiếp request.

@correct: A
@point: 20

## Q2

Tại sao việc cấu hình chỉ thị `proxy_set_header X-Real-IP $remote_addr;` lại cực kỳ quan trọng khi thiết lập Reverse Proxy?

[A]
Để Nginx tự động giới hạn băng thông truy cập của Client dựa trên địa chỉ IP của họ.
[EXP]
Sai. Việc giới hạn băng thông hoặc giới hạn request sử dụng các module limit chuyên biệt, không phụ thuộc vào header này.
[B]
Để bảo toàn địa chỉ IP thực tế của Client, giúp ứng dụng backend nhận diện đúng IP nguồn thay vì chỉ thấy IP của Nginx (127.0.0.1).
[EXP]
Chính xác. Khi qua proxy, request mới gửi đến backend sẽ có IP nguồn là IP của Nginx. Dòng cấu hình này giúp nạp IP thực của Client vào header `X-Real-IP` để backend đọc và xử lý.
[C]
Để mã hóa địa chỉ IP của Client trước khi truyền tin nhằm đảm bảo an toàn thông tin cá nhân.
[EXP]
Sai. Header này lưu IP dưới dạng văn bản rõ ràng (Plain text) để backend sử dụng trực tiếp, không mã hóa.
[D]
Để Nginx tự động kiểm tra xem IP của Client có nằm trong danh sách đen (Blacklist) của hệ điều hành hay không.
[EXP]
Sai. Việc chặn IP được thực hiện bởi tường lửa UFW hoặc chỉ thị `allow/deny` của Nginx, không liên quan đến việc đính kèm header chuyển tiếp này.

@correct: B
@point: 20

## Q3

Trước khi chạy lệnh nạp lại cấu hình Nginx trên Production, DevOps nên thực hiện câu lệnh nào để giảm thiểu rủi ro làm sập hệ thống?

[A]
`sudo systemctl restart nginx`
[EXP]
Sai. Lệnh restart tắt hoàn toàn dịch vụ và bật lại, nếu cấu hình lỗi hệ thống sẽ bị downtime.
[B]
`sudo nginx -t`
[EXP]
Chính xác. Lệnh `nginx -t` kiểm tra cú pháp của toàn bộ các file cấu hình Nginx. Nếu có lỗi (thiếu dấu `;`, sai từ khóa...), nó sẽ báo lỗi cụ thể ở dòng nào để sửa trước khi áp dụng.
[C]
`sudo systemctl status nginx`
[EXP]
Sai. Lệnh này chỉ xem trạng thái hoạt động hiện tại của dịch vụ Nginx, không có chức năng kiểm tra cú pháp file config mới sửa.
[D]
`sudo nginx -s stop`
[EXP]
Sai. Lệnh này tắt ngay lập tức tiến trình Nginx, gây downtime dịch vụ.

@correct: B
@point: 20

## Q4

Cơ chế hoạt động của lệnh `systemctl reload nginx` giúp đạt được trạng thái "Zero-Downtime Hot Reload" như thế nào?

[A]
Nginx sẽ tự động bỏ qua toàn bộ các lỗi cú pháp trong cấu hình mới để tiếp tục chạy cấu hình cũ.
[EXP]
Sai. Nếu cấu hình mới lỗi cú pháp, lệnh reload sẽ thất bại và báo lỗi ngay lập tức, không tự bỏ qua.
[B]
Master process spawn ra các worker mới chạy cấu hình mới, đồng thời ra lệnh cho các worker cũ xử lý nốt các kết nối dở dang rồi tự hủy nhẹ nhàng.
[EXP]
Chính xác. Cơ chế này giúp các request mới được xử lý bằng cấu hình mới, các request cũ không bị ngắt đột ngột, đảm bảo hệ thống hoạt động liên tục 100%.
[C]
Nginx sẽ tạm thời lưu trữ toàn bộ request của khách hàng vào database trong thời gian dịch vụ tạm dừng 5 giây.
[EXP]
Sai. Nginx reload diễn ra tức thời ở tầng bộ nhớ của OS, hoàn toàn không ghi request vào database.
[D]
Nginx tự động nhân bản thêm một VPS phụ chạy song song để gánh tải trong quá trình reload cấu hình.
[EXP]
Sai. Lệnh reload chỉ diễn ra trên chính máy chủ VPS hiện hành, không có khả năng tự mua thêm VPS mới.

@correct: B
@point: 20

## Q5

Khi chuyển tiếp request tới API Gateway thông qua chỉ thị `proxy_pass http://127.0.0.1:8080;`, tại sao chúng ta nên cấu hình thêm `proxy_set_header X-Forwarded-Proto $scheme;`?

[A]
Để backend Spring Boot biết được Client ban đầu truy cập qua giao thức bảo mật nào (HTTP hay HTTPS) và đưa ra phản hồi phù hợp.
[EXP]
Chính xác. Vì Nginx giải mã SSL rồi chuyển tiếp HTTP thường tới backend, Spring Boot sẽ chỉ thấy request HTTP thường đi vào. Header này giúp truyền thông tin giao thức gốc cho backend biết để xử lý đúng logic (ví dụ dựng link redirect).
[B]
Để bắt buộc Nginx phải tự động mã hóa lại gói tin bằng một thuật toán bảo mật mới trước khi gửi đi.
[EXP]
Sai. Chỉ thị này chỉ truyền thông tin giao thức dưới dạng chuỗi văn bản (như "https"), không thực hiện mã hóa gói tin nội bộ.
[C]
Để đổi giao thức kết nối nội bộ của các container từ TCP sang UDP nhằm tăng tốc độ truyền tải.
[EXP]
Sai. HTTP/HTTPS hoạt động bắt buộc trên nền tảng giao thức truyền tải tin cậy TCP, không thể tự chuyển sang UDP.
[D]
Để Spring Boot tự động kích hoạt tính năng CORS filter đối với toàn bộ các tên miền ngoài danh sách.
[EXP]
Sai. CORS được cấu hình độc lập qua mã nguồn Java Spring Boot hoặc Spring Cloud Gateway, không tự động kích hoạt qua header này.

@correct: A
@point: 20

---

# LESSON 04: Cấu hình tên miền (Domain) và mã hóa bảo mật SSL/HTTPS với Let's Encrypt

## Q1

Để trỏ tên miền phụ `api.quickbite.com` về địa chỉ IP công khai của máy chủ VPS, bạn cần cấu hình loại bản ghi DNS nào?

[A]
Bản ghi `MX (Mail Exchanger)`
[EXP]
Sai. Bản ghi MX dùng để định tuyến hệ thống Email, không dùng để trỏ tên miền về IP máy chủ Web.
[B]
Bản ghi `CNAME (Canonical Name)`
[EXP]
Sai. CNAME dùng để ánh xạ một tên miền sang một tên miền khác (alias), không trỏ trực tiếp về địa chỉ IP thô.
[C]
Bản ghi `TXT`
[EXP]
Sai. Bản ghi TXT dùng để lưu trữ các thông tin văn bản cấu hình (như xác thực SPF, DKIM cho email), không dùng để định tuyến IP.
[D]
Bản ghi `A (Address Record)`
[EXP]
Chính xác. Bản ghi A dùng để ánh xạ trực tiếp tên miền hoặc subdomain về một địa chỉ IPv4 công khai của máy chủ vật lý/VPS.

@correct: D
@point: 20

## Q2

Let's Encrypt và công cụ Certbot có mối quan hệ như thế nào trong quy trình thiết lập bảo mật HTTPS?

[A]
Let's Encrypt là công cụ dòng lệnh cài trên VPS, còn Certbot là trang web bán chứng chỉ SSL trả phí.
[EXP]
Sai. Let's Encrypt là tổ chức phát hành chứng chỉ miễn phí; Certbot là client tự động hóa chạy trên VPS.
[B]
Certbot là client tự động tương tác để xác thực tên miền và tải xuống chứng chỉ SSL miễn phí do tổ chức CA Let's Encrypt phát hành.
[EXP]
Chính xác. Certbot chạy trên máy chủ VPS, tự thực hiện các thủ tục bắt tay xác thực (challenge) với Let's Encrypt Server để tải và cấu hình SSL tự động cho Web server.
[C]
Certbot là một phần mềm diệt virus bảo vệ VPS, còn Let's Encrypt là thuật toán mã hóa dữ liệu của database.
[EXP]
Sai. Cả hai công cụ này đều phục vụ cho việc cấp phát và cấu hình chứng chỉ bảo mật SSL/TLS cho web server.
[D]
Let's Encrypt sẽ tự động khóa tài khoản GitHub của bạn nếu phát hiện bạn không sử dụng Certbot để bảo mật VPS.
[EXP]
Sai. Let's Encrypt và GitHub là hai tổ chức hoạt động hoàn toàn độc lập, không liên quan đến việc quản lý tài khoản của nhau.

@correct: B
@point: 20

## Q3

Khi chạy lệnh `sudo certbot --nginx -d api.quickbite.com` thành công, Certbot sẽ thực hiện cấu hình chuyển hướng (HTTP sang HTTPS) bằng mã trạng thái HTTP nào?

[A]
`HTTP 302 Found`
[EXP]
Sai. Mã 302 là chuyển hướng tạm thời (Temporary Redirect), không được Certbot cấu hình mặc định để tối ưu SEO.
[B]
`HTTP 404 Not Found`
[EXP]
Sai. Mã 404 biểu thị tài nguyên không tồn tại, không phải mã lệnh chuyển hướng giao thức.
[C]
`HTTP 301 Moved Permanently`
[EXP]
Chính xác. Certbot cấu hình redirect vĩnh viễn (301 Redirect) từ cổng 80 lên cổng 443 để thông báo cho trình duyệt và các bot tìm kiếm luôn sử dụng phiên bản HTTPS bảo mật cho các lần truy cập sau.
[D]
`HTTP 502 Bad Gateway`
[EXP]
Sai. Mã 502 là lỗi xảy ra khi Nginx không kết nối được tới ứng dụng backend phía sau, không phải mã chuyển hướng.

@correct: C
@point: 20

## Q4

Tại sao việc bật chế độ Proxy (đám mây màu cam) của Cloudflare lại có thể khiến lệnh xin cấp chứng chỉ SSL bằng Certbot (`HTTP-01 challenge`) thất bại?

[A]
Vì Cloudflare tự động xóa toàn bộ các tệp tin cấu hình con trong thư mục `/etc/nginx/conf.d/` của VPS.
[EXP]
Sai. Cloudflare chỉ can thiệp vào định tuyến DNS và CDN, không có quyền can thiệp vào hệ thống tệp tin cục bộ của VPS.
[B]
Vì Let's Encrypt Server không thể truy cập trực tiếp tệp tin xác thực tạm thời trên VPS do bị tường lửa/proxy của Cloudflare chặn hoặc chuyển hướng.
[EXP]
Chính xác. Let's Encrypt xác thực bằng cách gọi HTTP tới tên miền để tìm file xác thực tạm thời. Đi qua CDN/Proxy Cloudflare có thể khiến gói tin bị chặn, chuyển hướng HTTPS hoặc cache lỗi, dẫn đến Let's Encrypt không đọc được file và báo lỗi xác thực.
[C]
Vì Cloudflare bắt buộc VPS phải sử dụng hệ điều hành Windows Server mới cho phép xác thực.
[EXP]
Sai. Cloudflare hoàn toàn tương thích và hỗ trợ mọi hệ điều hành máy chủ.
[D]
Vì Let's Encrypt chỉ cấp phát chứng chỉ SSL cho các VPS không sử dụng bất kỳ dịch vụ DNS nào.
[EXP]
Sai. Để có tên miền chạy web, bắt buộc phải có dịch vụ quản lý DNS, Let's Encrypt luôn hoạt động dựa trên DNS của tên miền.

@correct: B
@point: 20

## Q5

Chứng chỉ SSL miễn phí do Let's Encrypt phát hành có thời hạn hiệu lực là bao lâu và giải pháp quản trị tự động là gì?

[A]
Hiệu lực 30 ngày; DevOps phải chạy lệnh xin cấp mới thủ công vào mỗi ngày cuối tháng.
[EXP]
Sai. Thời hạn hiệu lực của chứng chỉ Let's Encrypt là 90 ngày.
[B]
Hiệu lực 1 năm; Certbot sẽ gửi mã kích hoạt mới qua tin nhắn SMS tới số điện thoại của quản trị viên để nhập thủ công.
[EXP]
Sai. Quy trình gia hạn của Let's Encrypt hoàn toàn tự động hóa qua Internet, không sử dụng SMS.
[C]
Hiệu lực 90 ngày; Certbot tự động thiết lập tiến trình chạy ngầm (cron job hoặc systemd timer) để tự động kiểm tra và gia hạn trước khi hết hạn.
[EXP]
Chính xác. Chứng chỉ có hạn 90 ngày để đảm bảo an toàn. Certbot tự cấu hình cron job chạy 2 lần mỗi ngày để tự gia hạn các chứng chỉ còn hạn dưới 30 ngày một cách tự động, DevOps không cần can thiệp.
[D]
Hiệu lực vĩnh viễn; sau khi cài đặt thành công, DevOps không bao giờ cần phải quan tâm đến việc gia hạn nữa.
[EXP]
Sai. Let's Encrypt không phát hành chứng chỉ vĩnh viễn vì lý do bảo mật và kiểm soát quyền sở hữu tên miền.

@correct: C
@point: 20
