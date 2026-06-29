# Kịch bản Thuyết trình - Session 01: Tổng quan DevOps & CI/CD

---

## Lesson 1: Tổng quan DevOps và Hạn chế của triển khai thủ công

**(Mở đầu - Khơi gợi sự hứng thú)**
Xin chào tất cả các bạn học viên! Rất vui được đồng hành cùng các bạn trong một hành trình hoàn toàn mới mang tên là: **DevOps**. 
Thực ra thì, nếu các bạn thường xuyên theo dõi các nền tảng tuyển dụng như ITviec hay TopCV, thì chắc chắn các bạn cũng đã nhận ra DevOps đang là một trong những từ khóa "hot" nhất và được săn đón nhiều nhất hiện nay rồi.

Vậy thì, DevOps có đơn thuần chỉ là đẩy code lên server và làm cho trang web chạy được trên Internet không? Chắc chắn là không rồi. DevOps rộng lớn và thú vị hơn thế rất nhiều. Và thầy hy vọng là môn học này sẽ truyền cảm hứng cho các bạn, mở ra một định hướng phát triển sự nghiệp mới mẻ và khác biệt hơn so với con đường lập trình web truyền thống.

**(Nội dung chính)**
Thế thì hôm nay, chúng ta sẽ bắt đầu với chuỗi bài học đầu tiên: **Tổng quan DevOps và Quy trình CI/CD**. Bài học hôm nay của chúng ta sẽ gồm 6 nội dung chính:
1. Tổng quan DevOps và hạn chế của triển khai thủ công
2. Khái niệm CI/CD
3. 3 môi trường cơ bản: Dev, Staging và Production
4. Nhắc lại kiến trúc Microservices
5. Hệ điều hành Linux và vai trò trong triển khai hệ thống
6. Quản lý quyền và các lệnh mạng cơ bản trong Linux

**[Slide: Hạn chế của triển khai thủ công]**
Qua các môn học trước về Java, Web Service hay Microservices, chắc hẳn các bạn đã quen với việc chạy ứng dụng bằng cách bấm nút "Run" trên IDE rồi đúng không? Tuy nhiên, nhiêu đó chỉ giúp các bạn dừng lại ở mức là một "Developer" thôi. 

Còn trong bối cảnh cạnh tranh hiện tại, để có lợi thế vượt trội khi đi xin việc, các bạn cần trang bị cho mình một **Tư duy Kỹ sư (Engineer Mindset)** – tức là tư duy của một người am hiểu toàn bộ hệ thống. Đó chính là cái gọi là tư duy **Production Ready**:
- Nghĩa là code xong phải tự động đóng gói được, tự động kiểm thử và tự động triển khai được.
- Và khi ứng dụng chạy, chúng ta phải liên tục giám sát được trạng thái của nó.
- Tóm lại là phải kiểm soát được bất kỳ vấn đề nào xảy ra trong suốt vòng đời của ứng dụng.

Thế thì để làm được việc đó, trong môn học này chúng ta chắc chắn sẽ cần sự hỗ trợ của rất nhiều công cụ. Nhưng trước tiên, hãy cùng nhìn lại một chút về quá khứ để hiểu tại sao chúng ta lại cần đến CI/CD nhé. Trong quá khứ, việc triển khai thủ công từng là một "ác mộng". Các lỗi thường gặp có thể kể đến như là:
- **Đầu tiên là vòng lặp sửa code thủ công:** Cứ sửa code là lại phải tự build lại, dùng lệnh `scp` đẩy file lên server, đăng nhập vào server rồi lại restart ứng dụng bằng tay, rất mất thời gian.
- **Tiếp theo là sai sót cấu hình:** Chúng ta rất dễ nhầm lẫn, ví dụ cấu hình nhầm database test sang database thật, gây mất dữ liệu nghiêm trọng.
- **Hay đơn giản hơn là lệch pha môi trường:** Code thì được viết và biên dịch bằng Java 21, nhưng server lại chỉ cài Java 17.
- **Và việc quản lý phiên bản cũng rất khó khăn:** Rất khó để kiểm soát các file `.jar` đang chạy thuộc về phiên bản nào.
- **Và cuối cùng là "mù" log:** Trong một hệ thống Microservices phức tạp với hàng chục service gọi chéo nhau, nếu chỉ đọc log thủ công từng service thì sẽ không thể nào dò ra được lỗi.

**[Slide: Xung đột giữa Dev và Ops]**
Một vấn đề lớn khác trong quá khứ đó là mâu thuẫn giữa đội ngũ Development (Dev) và Operations (Ops).
- **Đội Dev** thì luôn muốn đẩy tính năng mới nhanh nhất có thể và fix bug ngay lập tức.
- Nhưng **Đội Ops** thì lại ưu tiên sự ổn định, sợ rủi ro sập hệ thống khi có thay đổi.
Cái mâu thuẫn này nó là một rào cản rất lớn, cho đến khi triết lý DevOps ra đời.

**[Slide: DevOps là gì?]**
Vậy tóm lại DevOps là gì? DevOps thực chất là sự kết hợp của **Development** và **Operations**. Các bạn hãy nhớ, đây không chỉ là một chức danh công việc, mà nó là một **văn hóa, quy trình và một tập hợp các công cụ** nhằm tự động hóa vòng đời phát triển phần mềm.

Thế thì làm sao để biết khi nào một người giỏi DevOps? Chúng ta sẽ dựa vào 3 trụ cột chính:
1. Sự am hiểu hệ thống thực tế (Hạ tầng, Mạng, Phần cứng).
2. Sự am hiểu công nghệ lõi đang sử dụng (Ví dụ như Java Runtime).
3. Và cuối cùng là khả năng xây dựng CI/CD Pipeline – Đây cũng chính là sản phẩm cốt lõi của DevOps.

---

## Lesson 2: Khái niệm CI/CD (Quy trình Build, Test, Deploy)

**[Câu chuyện thực tế]**
Bây giờ chúng ta thử tưởng tượng một tình huống thực tế như thế này: Có một bạn thực tập sinh rất chăm chỉ, liên tục viết code và đẩy hàng tá commit lên hệ thống mỗi ngày. Anh Tech Lead thì làm gì có đủ thời gian để kéo code về, review và test thủ công từng cái commit một được. 

Giải pháp ở đây chính là cái **CI/CD Pipeline**. Nhờ có Pipeline, đại khái là mỗi khi bạn intern đẩy commit lên, code sẽ tự động được build và chạy test thử. Chỉ khi nào tất cả các bài test đều xanh (tức là Pass), thì anh Tech Lead mới cần vào xem xét và quyết định.

**[Slide: CI/CD là gì?]**
- Đầu tiên, **CI (Continuous Integration - Tích hợp liên tục):** Nghĩa là tự động compile và chạy Unit Test ngay khi developer gõ lệnh `git push`. Cái CI này nó gắn liền với một triết lý là **"Fail fast"** - hiểu đơn giản là lỗi phát hiện càng sớm, thì chi phí sửa chữa lại càng thấp.
- Tiếp theo là **CD thì lại có hai khái niệm:**
  - 1 là **Continuous Delivery (Phân phối liên tục):** Code sau khi vượt qua CI sẽ được đóng gói sẵn sàng để deploy lên môi trường Production, nhưng lúc này vẫn cần một cái "click" phê duyệt từ con người.
  - Ngược lại, **Continuous Deployment (Triển khai liên tục):** Tức là tự động hóa 100% luôn. Code tự động được đẩy thẳng lên Production mà không cần ai phê duyệt cả. Đây có thể coi là bước trưởng thành nhất, khi đội ngũ đã hoàn toàn tin tưởng vào các bài test tự động của mình rồi.

*(Nhấn mạnh)*: Các bạn lưu ý là, cái lệnh `git push` chính là "công tắc" kích hoạt toàn bộ máy chủ CI/CD hoạt động nhé!

**[Slide: 4 Giai đoạn của Pipeline cơ bản]**
Một sơ đồ pipeline cơ bản sẽ gồm 4 giai đoạn, và nếu bất kỳ giai đoạn nào thất bại (fail), thì toàn bộ pipeline sẽ dừng lại ngay lập tức:
1. **Đầu tiên là Compile:** Biên dịch mã nguồn (cái này thì áp dụng cho các ngôn ngữ như Java hay NodeJS...).
2. **Tiếp theo là Test:** Chạy các bài test tự động. Đây là phần bắt buộc và là cốt lõi của triết lý "fail fast" mà có thể trước nay các bạn chưa chú trọng nhiều.
3. **Thứ ba là Build/Package (Đóng gói):** Đóng gói thành file `.jar` và sau đó đóng gói tiếp thành một Docker Image.
4. **Và cuối cùng là Deploy:** Triển khai tự động bản đóng gói đó lên server thôi.

*(Lưu ý)*: Ở đây có một hiểu lầm rất phổ biến, nhiều người cứ nghĩ việc viết một cái script tự động copy file `.jar` lên server cũng là CI/CD. Hoàn toàn sai nhé! Đó cao lắm thì chỉ là một phần nhỏ của CD thôi, còn phần quan trọng nhất phải là cái **CI** (tức là kiểm thử và tích hợp cơ).

---

## Lesson 3: Môi trường Dev, Staging, và Production

**[Slide: Các loại môi trường]**
Thực tế thì trong bất kỳ dự án nào, luôn luôn tồn tại ít nhất là 3 môi trường biệt lập thế này, mỗi môi trường sẽ có mức độ bảo mật và cấu hình khác nhau:
1. **Đầu tiên là Dev (Development):** Tức là môi trường máy local của lập trình viên. Dữ liệu thì tự giả lập, độ ổn định cũng không quan trọng. Mục tiêu duy nhất chỉ là để code nhanh và debug dễ dàng.
2. **Tiếp đến là Staging:** Đây là môi trường "Bản sao hoàn hảo" của Production (giống đến 99%). Dữ liệu cũng được làm cho giống thật nhưng không được chứa thông tin nhạy cảm. Mục đích của nó là để chạy Integration Test, kiểm thử giao diện và nghiệm thu (UAT) trước khi tung ra sản phẩm.
3. **Và cuối cùng là Production:** Đây là môi trường chạy thật, nơi người dùng cuối trực tiếp sử dụng. Dữ liệu là thật, độ bảo mật phải tuyệt đối và yêu cầu là hệ thống phải luôn hoạt động (Uptime 99.99%).

**[Slide: Build Once, Run Anywhere]**
Thế thì để quản lý tốt 3 môi trường này, chúng ta cần phải áp dụng một nguyên tắc gọi là: **"Build Once, Run Anywhere"** (Build 1 lần, Chạy mọi nơi). Tức là sao? Nghĩa là chúng bản chất chỉ tạo ra đúng 1 file `.jar` duy nhất ở giai đoạn CI thôi, và mang chính cái file đó đi chạy ở Dev, Staging hay Production.

Vậy thì làm sao để ứng dụng biết nó đang cần kết nối với Database nào? Câu trả lời chính là: **Biến môi trường (Environment Variables)**.
Trước đây, hẳn là các bạn đã quen với việc "hardcode" đường dẫn database thẳng vào file `application.yml` rồi đúng không. Điều này cực kỳ rủi ro về mặt bảo mật và làm cho file `.jar` bị đóng cứng, không thể linh hoạt. 

Spring Boot thì lại hỗ trợ nội suy biến môi trường rất tốt bằng cú pháp `${TEN_BIEN:gia_tri_mac_dinh}`. Khi khởi chạy, chúng ta chỉ cần nạp biến môi trường vào thông qua shell (ví dụ như lệnh `export DATABASE_URL=...`), rồi sau đó chạy lệnh `java -jar application.jar` là xong.

---

## Lesson 4: Kiến trúc triển khai hệ thống Microservices

**[Slide: Vấn đề bảo mật Microservices]**
Chúng ta đều biết Microservices là một hệ thống rất lớn, gồm nhiều dịch vụ nhỏ giao tiếp với nhau. Thế thì các bạn hãy thử tưởng tượng, nếu chúng ta để lộ trực tiếp địa chỉ IP và Port nội bộ của từng service ra ngoài Internet (ví dụ như để lộ port `10.0.1.15:8081` cho User Service, hay `10.0.1.16:8082` cho Order Service) thì hậu quả sẽ ra sao? Việc này sẽ cực kỳ nguy hiểm các bạn ạ:
- **Thứ nhất là rủi ro bảo mật:** Hacker có thể dễ dàng dò quét các port này và tấn công trực tiếp vào từng service. Hành động này không khác gì chúng ta xây một ngôi nhà mà lại mở toang mọi cánh cửa sổ mời trộm vào vậy.
- **Thứ hai là quản lý SSL rất phức tạp:** Nếu đưa trực tiếp ra ngoài, mỗi service sẽ lại phải tự gắn một chứng chỉ HTTPS riêng. Các bạn cứ tưởng tượng hệ thống có 50 service mà phải cài và gia hạn SSL cho cả 50 cái thì rất mệt mỏi và dễ xảy ra sai sót.
- **Và thứ ba là sự cứng nhắc của hệ thống:** Giả sử khi server bị chết, chúng ta phải đổi IP, hoặc khi có sự kiện cần tăng số lượng server lên để chịu tải, thì các client (như web, mobile app) sẽ phải cập nhật lại đường dẫn API. Việc này là bất khả thi trên thực tế và sẽ gây ra gián đoạn hệ thống trên diện rộng.

**[Slide: Kiến trúc Quickbite]**
Để giải quyết triệt để cái bài toán đau đầu trên, chúng ta sẽ áp dụng một kiến trúc mạng phân lớp tiêu chuẩn. Và thầy sẽ mượn luôn dự án **Quickbite** – dự án thực hành xuyên suốt của môn học này – để làm ví dụ nhé. 

Vậy hệ thống Quickbite được tổ chức như thế nào?
1. **Đầu tiên là lớp Reverse Proxy (Nginx):** Hệ thống Quickbite chỉ mở ra đúng một "cửa ngõ" duy nhất kết nối với Internet thôi. Thằng Nginx này sẽ đóng vai trò là một người bảo vệ cổng (Gatekeeper), đứng ra tiếp nhận mọi request từ bên ngoài vào.
2. **Bên dưới là lớp API Gateway:** Thằng này thì nằm hoàn toàn bên trong mạng nội bộ. Gateway có nhiệm vụ nhận yêu cầu từ Nginx và bắt đầu phân luồng, định tuyến động (Dynamic Routing) đến các service nhỏ bên trong.
3. **Phía sau nữa là các Services độc lập:** Đây là nơi chứa logic nghiệp vụ, như User Service, Order Service,...
4. **Và cuối cùng là Database:** Quickbite tuân thủ chặt chẽ cái nguyên tắc gọi là **Database-per-service**: Tức là mỗi service sở hữu một database riêng biệt. Về mặt vật lý, chúng ta có thể cài chung trên một cụm server PostgreSQL cho tiết kiệm, nhưng về mặt logic thì chúng cách ly hoàn toàn. Tuyệt đối service này không được phép truy cập hay ghi chéo vào database của service khác. Tại sao lại thế? Là để giả sử khi service Order bị sập database, thì service User vẫn hoạt động bình thường, đảm bảo tính sẵn sàng cao (High Availability) cho toàn bộ hệ thống.

**[Slide: Phân biệt Nginx và API Gateway]**
Đến đây chắc các bạn sẽ hỏi thầy là: "Thầy ơi, tại sao lại cần thêm Nginx, trong khi em thấy API Gateway làm được hết mọi việc rồi cơ mà?" 
Đúng là Gateway rất mạnh, nhưng thực tế triển khai thì mỗi công cụ lại có một thế mạnh đặc thù riêng:
- **Nginx** được sinh ra để làm lớp khiên chắn. Nó rất mạnh trong việc xử lý request bất đồng bộ và chịu tải cực kỳ khủng khiếp. Nó sẽ đứng ra gánh vác việc giải mã chứng chỉ SSL (thuật ngữ gọi là SSL Termination) để giảm tải cho các dịch vụ phía sau. Ngoài ra, Nginx cũng kiêm luôn việc trả về các file tĩnh (như hình ảnh, HTML, CSS, JS) cho người dùng một cách cực kỳ trơn tru và nhanh chóng.
- Còn **API Gateway** thì nó lại tập trung vào logic của API hơn. Nó sẽ đứng ra kiểm tra xem người dùng này có quyền truy cập không thông qua xác thực tập trung (Authentication như check JWT Token), rồi thì giới hạn tốc độ (Rate Limiting) để chống việc có ai đó spam request làm sập server, và định tuyến request tới đúng service đang cần.

**[Kết luận Bài 4]**
Vậy nói tóm lại, nhờ áp dụng kiến trúc phân lớp này, toàn bộ hệ thống khổng lồ của chúng ta chỉ mở đúng hai cổng ra Internet là cổng 443 (cho HTTPS) và cổng 80 (cho HTTP). Tất cả các dịch vụ nội bộ còn lại thì đều được giấu an toàn tuyệt đối ở phía sau bức tường lửa và Nginx.

---

## Lesson 5: Hệ điều hành Linux và vai trò trong triển khai hệ thống

**[Slide 12: Vai trò của Linux & Lệnh khởi tạo hệ thống]**
Thế thì từ nãy đến giờ chúng ta nói rất nhiều về hệ thống, về tự động hóa. Vậy hệ điều hành nào là tiêu chuẩn để vận hành các hệ thống này? Chắc chắn đó là **Linux**. Tại sao lại thế? 
Bởi vì công nghệ Container cốt lõi như Docker luôn dùng chung nhân hệ điều hành (thuật ngữ gọi là Shared Kernel), và cái nhân đó bắt buộc phải là Linux Kernel.

Bây giờ thầy sẽ chuyển sang phần thực hành, bắt đầu bằng việc chuẩn bị một server Linux để chạy code Spring Boot.

**(Nhấn mạnh: Chuyển màn hình từ Slide sang Terminal)**

**[Live Demo 1: Cài đặt JDK 17 bằng Script tự động]**
- *Để cài đặt công cụ cần thiết trên Ubuntu, thay vì gõ tay từng lệnh rất dễ sai, thầy sẽ tạo hẳn một kịch bản tự động bằng file script. Thầy tạo file `initial-script.sh`.*
- *Bên trong file, thầy gõ lệnh cập nhật hệ thống `sudo apt-get update && sudo apt-get upgrade -y` và lệnh cài đặt Java `sudo apt-get install -y openjdk-17-jdk curl git`.*
- *Lưu file lại, thầy sẽ dùng lệnh `chmod +x initial-script.sh` để cấp quyền thực thi cho nó.*
- *Chạy file script bằng lệnh `./initial-script.sh`. Đó, các bạn thấy không, server đang tự động cài Java 17 chuẩn chỉnh luôn.*

**[Slide 13: Thao tác file, thư mục và quản lý dịch vụ]**
Ngoài các lệnh thao tác file cơ bản như `pwd`, `ls -la`, tạo thư mục `mkdir -p` hay xóa file `rm -rf`, thì kỹ năng quan trọng nhất của DevOps là quản lý biến môi trường và tiến trình hệ thống.

**[Live Demo 2: Cấu hình biến môi trường và Quản lý hệ thống]**
- *Đầu tiên, để cấu hình biến môi trường vĩnh viễn, thầy sẽ chèn biến cấu hình vào thẳng file hệ thống bằng lệnh: `echo "export QUICKBITE_DB_USER=staging_admin" >> ~/.bashrc`.*
- *Và để nó nhận cấu hình ngay lập tức mà không cần khởi động lại, thầy dùng `source ~/.bashrc`.*
- *Tiếp theo, giả sử hệ thống có một tiến trình Java đang bị treo. Thầy sẽ dùng `ps -ef | grep java` để dò ra tiến trình đó. Thấy số PID của nó rồi, thầy dùng lệnh `kill -9 [PID]` để tiêu diệt dứt điểm.*
- *Nói về Nginx, có 2 lệnh kinh điển các bạn phải nằm lòng: Thứ nhất là `sudo nginx -t` để test file cấu hình trước xem có lỗi cú pháp không. Thứ hai là `sudo nginx -s reload` để cập nhật cấu hình mới mà không làm mất kết nối của client.*

---

## Lesson 6: Quản lý quyền và các lệnh mạng cơ bản trong Linux

**[Slide 14: Bảo mật hệ thống - Tạo người dùng chuyên dụng]**
Nói đến server là phải nói đến bảo mật. Các bạn có biết điều tối kỵ nhất trên Linux là gì không? Đó là chạy ứng dụng Web bằng quyền quản trị viên cao nhất - quyền `root`. Nếu hacker tấn công được ứng dụng, họ sẽ chiếm trọn cả máy chủ. Để ngăn chặn, chúng ta phải tạo ra các tài khoản hệ thống (system user) chỉ dùng để chạy app và không có quyền truy cập vỏ sò (shell).

**[Live Demo 3: Tạo người dùng bảo mật]**
- *Đầu tiên thầy sẽ tạo một nhóm riêng: `sudo groupadd quickbite`.*
- *Tiếp theo là lệnh mấu chốt: `sudo useradd -r -g quickbite -s /bin/false quickbite`.*
- *Tại sao lệnh này an toàn? Chữ `-r` ở đây là để tạo tài khoản hệ thống không có thư mục cá nhân. Còn `-s /bin/false` là để cấm tiệt tài khoản này login qua SSH. Do vậy, dù hacker có mò được pass thì cũng không thể gõ được bất kỳ lệnh phá hoại nào.*

**[Slide 15: Phân quyền thư mục & Chẩn đoán mạng]**
Có user rồi, chúng ta phải thiết lập khu vực an toàn cho thư mục chứa mã nguồn của chúng ta. Đồng thời, khi server gặp sự cố, kỹ năng chẩn đoán mạng là cứu cánh đầu tiên.

**[Live Demo 4: Phân quyền và Chẩn đoán mạng]**
- *Thầy sẽ tạo nơi lưu trữ bằng lệnh `sudo mkdir -p /opt/quickbite/user-service`.*
- *Chuyển quyền sở hữu cho anh user bảo mật vừa tạo bằng lệnh `sudo chown -R quickbite:quickbite /opt/quickbite`.*
- *Cuối cùng là chốt chặn phân quyền: `sudo chmod -R 750 /opt/quickbite` (chủ sở hữu được Đọc/Ghi/Chạy, nhưng những người dùng khác tuyệt đối không có quyền gì).*

- *Còn về mạng thì sao? Thầy sẽ gõ `ip addr` để xem IP hiện tại của server.*
- *Dùng lệnh `curl -I https://google.com` để gửi một HTTP Request thần tốc, test xem mạng và dịch vụ web kia có đang sống hay không.*
- *Và đặc sản cuối cùng, kịch bản lỗi "Port 8080 already in use". Thầy sẽ dò ra thủ phạm đang chiếm cổng bằng lệnh `sudo ss -tulpn | grep :8080`. Trả về PID là 14522. Vậy thì kết liễu nó bằng lệnh `sudo kill -9 14522`. Cổng 8080 lại rảnh rang để chạy app của chúng ta!*

Đó là toàn bộ những kỹ năng cốt lõi nhất của Linux mà các bạn cần nắm để bước vào thế giới DevOps thực chiến. Nó cũng khép lại nội dung Session 1 ngày hôm nay. Hẹn gặp lại các bạn trong bài học tiếp theo!
