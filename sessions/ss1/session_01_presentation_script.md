## lesson 1

Chào các bạn sinh viên, rất vui được đồng hành với các bạn trong môn học Devops này.
Có thể một số bạn đã nghe đến cụm từ Devops.
Thế nhưng thầy tin rằng rất nhiều trong số chúng ta chưa hiểu rõ về Devops đâu.
Vì thế môn học này là để chúng ta đi bóc tách và giải mã Devops.

Thì tại thời điểm này của môn học

Các bạn chỉ cần hiểu Devops là việc triển khai ứng dung web (ví dụ như spring boot) lên trên server rồi public nó ra internet.

Được rồi, chúng ta sẽ đến với bài đầu tiên, tổng quan Devops và Quy trình CI/CD.

Bài học này có 6 nội dung:
1. Tổng quan Devops, hạn chế của triển khai thủ công
2. Khái niệm CI CD
3. 3 Môi trường Dev Staging và Production
4. Nhắc lại kiến trúc Microservice
5. Linux và vao trò của Linux trong Devops
6. Một vài lệnh quản lý quyền và mạng trong Linux.

Nội dung đầu tiên: Tổng quan Devops và hạn chế của triển khai thủ công.

Qua các môn học Java trước, webservice, rồi microservice, chắc hẳn các bạn đã biết cách chạy ứng dụng web là như thế nào, chính là việc bấm nút Run trên IDE của chúng ta.

Tuy nhiên, nhiêu đó chỉ giúp các bạn là một developer, còn trong bối cảnh cạnh tranh của thị trường IT hiện tại, để có lợi thế tìm việc, chúng ta cần phải có tư duy không chỉ dừng lại ở mức code được, mà phải là người am hiểu hệ thống, là một kỹ sư

Và cái tư duy kĩ sư đó mà thầy muốn nhắc đến là tư duy Production Ready.

Tức là: 

Code xong là có thể đóng gói được, tự động kiểm thử, tự động deploy.

Sau khi deploy lên thì có thể giám sát liên tục trạng thái của ứng dụng được.

Và bất kì vấn đề nào xảy ra trong vòng đời của ứng dụng, chúng ta đều kiểm soát được. 

Để làm được việc đó, chắc chắn chúng ta cần rất nhiều công cụ hỗ trợ.

Và môn học này, là việc chúng ta đi tìm hiểu, ứng dụng các công cụ đó.

Chúng ta sẽ đi tìm hiểu thử các kịch bản triển khai mà đã từng là ác mộng trong quá khứ khi chưa có CI CD.

Ở đây chúng ta có một dự án giả định tên là Quickbite, là một nền tảng giao đồ ăn, và được phát triển theo kiến trúc microservice.

Thế thì có các lỗi gì có thể xảy ra khi triển khai?

Đầu tiên là vòng lặp sửa code vô tận, cứ mỗi lần sửa code là một lần build lại rồi sử dụng scp để đưa file jar lên server, rồi đăng nhập vào server, restart ứng dụng thủ công.

Thế nhưng chúng ta chỉ cần copy nhầm config local lên Prod. Dữ liệu thật bị ghi vào DB test.

Các service bị lệch môi trường, nghe thì đơn giản như rất hay mắc phải, code trên dev thì viết trên java 21, nhưng server thì lại cài nhầm jre 17.

Các phiên bản file jar thì lung tung, cuối cùng không biết cái nào mới là cái dùng ổn định.

Gần như mù tịt về logging, bởi vì hệ thống log mặc định sử dụng journalctl là không đủ.

Trong quá khứ, khi đội devs và đội ops tách biệt, thế là suốt ngày cãi nhau.

Thế thì Devops là gì?
Devops là kết hợp của Development và Operations - Đây không phải là một vị trí công việc.
Nó là sự giao thoa giữa văn hoá, quy trình, và công cụ để tự động hoá vòng đời phát hành phần mềm.

Thế khi nào là chúng ta đã nắm được devops, khi nào là chúng ta đã biết, hay đã giỏi Devops? Có thể được tóm gọn qua 3 trụ cột chính sau:
Sự am hiểu về hệ thống thực tế, về hạ tầng, về phần cứng.
Sự am hiểu về công nghệ sử dụng (Java Runtime)
Cuối cùng là biết xây dựng CI CD Pipeline, và CI CD Pipeline cũng là sản phẩm cốt lõi của Devops.


## Lesson 2
Chúng ta sẽ đến với bài số 2, đó là Khái niệm CI CD (Quy trình buid, test, deploy)

Ở đây thầy sẽ lấy một ví dụ hết sức thực tế, về một bạn intern, rất tích cực code và đẩy commit mới lên, tuy nhiên như thế thì anh Techlead sẽ không thể nào review hết được, vì làm gì có thời gian ạ.

Nhưng may mắn là có một thứ rất hay đó là CI CD pipeline, đại khái là
Khi intern đẩy commit lên, code sẽ được build thử, được chạy test thử, sau khi nó xanh (passed) thì anh Techlead mới vào review code để quyết định xem làm gì tiếp với nó.

Oke chúng ta đã nói nhiều về CI CD rồi, thế còn khái niệm nó là gì?
CI là viết tắt của Continuous Integration (Tích hợp liên tục), nghĩa là tự động compile và chạy unit test ngay khi dev gõ git push.
Cái CI này nó gắn với một triết lý, gọi là Fail fast, nói chung là, lỗi càng sớm, càng giảm được rủ ro mất tiền sau này.

Còn CD, thì lại có 2 khái niệm:
1 là Continuous Delivery, nghĩa là sau khi CI xong, thì code tự động đóng gói và sẵn sàng để Deploy, nhưng vẫn cần phê duyệt của con người.
Ngược lại Continous Deployment, nghĩa là tự động hoá 100% không cần cả phê duyệt nữa luôn, thế thì bản chất là Coutinous Deployment là quy trình đầy đủ hơn của Delivery thôi, khi mà chúng ta hoàn toàn có thể tin tưởng cái Pipeline mà mình đã xây.

Lưu ý là: lệnh git push, chính là nút kích hoạt cho máy chủ CI hoạt động.

Sơ đồ pipeline cơ bản có thể tóm gọn thành 4 giai đoạn (stage) sau, và bất kì giai đoạn nào fail, nghĩa là cả pipeline fail.

Giai đoạn đầu tiện là Compile, cái này thì áp dụng cho Java và các ngôn ngữ compile khác như node.

Giai đoạn tiếp theo là chạy test, chạy test là bắt buộc, chẳng qua là trước nay chúng ta ít khi quan tâm đến cái test này thôi, chứ nó là một phần quan trọng của triết lý fail fast.

Giai đoạn ba là đóng gói, ở đây là đóng gói Jar, hoặc đóng gói Docker, chính xác hơn là cả 2, đóng gói Jar trước, sau đó đóng gói Docker.

Giai đoạn cuối cùng là Deploy nữa thôi, tự động triển khai lên Production.

Ở đây, có một hiểu lầm rất phổ biến, đó là việc viết một script để thực hiện việc copy file Jar từ local lên Server cũng là CI CD.
Sai hoàn toàn, đấy chỉ là CD, còn CI mới là cái quan trọng.


## Lesson 3 Môi trường dev, staging, và production

Trong bất kì dự án thực tế nào, luôn luôn có ít nhất 3 môi trường như thế này, mỗi môi trường áp dụng các cấu hình khác nhau và cấp độ bảo mật khác nhau.

Môi trường dev, chính máy Local của lập trình viên, thì có dữ liệu tự giả lập, mức độ ổn định yêu cầu thấp, mục tiêu duy nhất là code nhanh và debug trực tiếp.

Môi trường Staging thì là môi trường độc lập, mô phỏng giống với production tới 99&, dữ liệu cũng được mô phỏng thật, nhưng không được chứa dữ liệu nhạy cảm. 
Mục đích của môi trường Staging là để chạy Integration Test và UAT (kiểm thử chấp nhận) để nghiệm thu tính năng.

Còn môi trường Production là môi trường thật, chạy thật và phục vụ cho người dùng cuối (end-user), dữ liệu thật và nhạy cảm, yêu cầu bảo mật tuyệt đối và uptime 99.99%

Thế thì để giao tiếp tốt giữa 3 môi trường này, chúng ta cần tuân thủ nguyên tắc "Build Once, Run Anywhere", đấy là khi mà chúng ta chỉ build ra 1 file Jar duy nhất, nhưng lại có thể dùng ở bất kì đâu, điểm khác biệt chính là ở biến môi trường chúng ta nạp vào lúc chạy ứng dụng này.

Nạp biến môi trường là cái gì, nghe mơ hồ quá?
Trước nay chúng ta hẳn là đã quen với việc hardcode database url vào trong application.yml, không chỉ build ra file jar bị cứng, mà còn rủi ro bảo mật khi chúng ta đẩy file application này lên trên repo mà có nhiều người có quyền truy cập.

Spring mặc định cung cấp cho chúng ta một cái rất hay để không cần fix cứng database url, mà có thể nạp khi khởi chạy, cơ chế này gọi là nội suy biến môi trường.

Cú pháp của nó là sử dụng dấu đô la và ngoặc nhọn, nếu fallback thì thêm nội dung sau dấu : nữa để fallback mặc định nếu không có biến tương ứng được nạp vào.

Thế khi đó biến đó được lấy từ đâu?

Lấy từ trong cái shell đang hoạt động khi đó, và sử dụng lệnh export TÊN BIẾN = giá trị

sau đó khởi chạy ứng dụng bằng lệnh java -jar user-service.jar


## lesson 4 Kiến trúc triển khai hệ thống Microservices

Chúng ta đều biết microservices là hệ thống phức tạp, gồm nhiều các service giao tiếp với nhau
Nếu như trực tiếp để lộ ip của các services ra ngoài internet (10.0.1.15:8081) thì sẽ rất nguy hiểm:
Thứ nhất là lộ cổng nội bộ, phơi bày hoàn toàn các port nội bộ ra internet, hacker có thể quét được và tấn công vào từng service.
Thứ hai là SSL rất phúc tạp khi phải quản lý chứng chỉ HTTPS cho từng cổng của từng service.
Thứ ba là cứng nhắc IP, khi thay đổi IP server hoặc khi cần scale hệ thống lên, thì các ứng dụng client (web, app mobile) phải cập nhật lại endpoint API, khiến cho hệ thống rất thiếu linh hoạt và khó bảo trì.

Chúng ta sẽ tham khảo khiến trúc phân lớp sau của hệ thống Quickbite, cũng là hệ thống tiêu chuẩn để chúng ta thực hành trong môn học này.

Quickbite chỉ có đúng 1 điểm duy nhất ra ngoài internet, vị trí này gọi là Reverse Proxy, và Nginx đảm nhận vai trò này.

Tiếp theo đến bên dưới là thuộc về mạng nội bộ, có API gateway, là chỉ đường đến các service con khác, API Gateway thì các bạn đều đã học rồi.

Bên dưới nữa là các service, và mỗi service có database riêng.

Quickbite cũng tuân thủ nguyên tắt Database-per-service, mỗi service chỉ truy cập đến database của riêng nó, về mặt vật lý có thể dùng chung một cụm Postgressql, nhưng logic database là tách rời hoàn toàn khỏi nhau, các service không được phép ghi chéo databse của nhau.

Phân biệt Nginx và API gateway.
Chúng ta sẽ đặt câu hỏi là Nginx để làm gì đâu, API gateway làm được mà?
Nhưng thực tế là không phải, Nginx làm rất nhiều, Nginx xử lý request bất đồng bộ tốt hơn, Nginx là tấm khiên, là nơi xử lý SSL (SSL Termination)
Ngoài ra, nó cũng có thể trả các tài nguyên tĩnh như là html css js cho người dùng.

Còn API gateway chỉ là thằng đứng sau tiếp nhận mọi request của Nginx để điều phối đến các service thôi, sử dụng cơ chế định tuyến động, gateway cũng làm việc xác thực tập trung (authentication) và Rate Limiting (chống DDOS)

Kết quả cuối cùng của thiết kế này, là chỉ có cổng 443, và 80 là ra ngoài internet, cái này là dĩ nhiên rồi, bởi vì web mặc định 443 cho https, và 80 cho http, còn lại các port khác chỉ hoạt động ở bên trong, không để lộ ra ngoài.
