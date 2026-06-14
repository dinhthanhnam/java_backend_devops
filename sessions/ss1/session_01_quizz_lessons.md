# QUIZ LESSONS - SESSION 01

# LESSON 01: Tổng quan DevOps và hạn chế của triển khai thủ công

## Q1

Quá trình triển khai phần mềm (Deployment) được định nghĩa như thế nào trong quy trình phát triển sản phẩm thực tế?

[A]
Quá trình chạy ứng dụng trên máy cá nhân của lập trình viên bằng cách sử dụng các nút Run/Debug trên công cụ IDE.
[EXP]
Đây là quá trình phát triển (Development) và chạy thử nghiệm trên môi trường sandbox cá nhân của lập trình viên (localhost), không phải triển khai thực tế.
[B]
Quá trình đưa ứng dụng từ môi trường phát triển của lập trình viên lên máy chủ thực tế (Production Server) để người dùng cuối có thể truy cập.
[EXP]
Đây chính là định nghĩa chính xác về triển khai phần mềm (Deployment), giúp sản phẩm tiếp cận người dùng cuối và mang lại doanh thu.
[C]
Quá trình thiết lập các bài kiểm thử tự động nhằm kiểm tra xem mã nguồn mới có phát sinh lỗi cú pháp hay không.
[EXP]
Đây là một bước kiểm soát chất lượng trong giai đoạn Tích hợp liên tục (CI), không phải toàn bộ định nghĩa của quá trình triển khai.
[D]
Quá trình cài đặt hệ điều hành và phân quyền các tài khoản quản trị trên máy chủ ảo trước khi vận hành ứng dụng.
[EXP]
Đây là công việc thiết lập hạ tầng và phân quyền hệ thống (Infrastructure Setup / Provisioning), là điều kiện cần chứ không phải định nghĩa triển khai phần mềm.


@correct: B
@point: 20

## Q2

Trong quy trình cập nhật mã nguồn thủ công từ máy local lên máy chủ Linux, thứ tự thực hiện đúng của các hành động nào dưới đây là chuẩn xác?
- (1) SSH vào server Linux để ra lệnh reload và restart dịch vụ qua Systemd.
- (2) Đóng gói ứng dụng thành file JAR tại máy local (ví dụ chạy `./gradlew clean bootJar`).
- (3) Sử dụng lệnh SCP để truyền file JAR vừa build lên thư mục tạm `/tmp` của server.

[A]
Thứ tự thực hiện các bước triển khai thủ công là: (1) -> (2) -> (3)
[EXP]
Thứ tự này không đúng logic vì bạn không thể SSH vào restart dịch vụ trước khi đóng gói và truyền file JAR mới lên server.
[B]
Thứ tự thực hiện các bước triển khai thủ công là: (2) -> (1) -> (3)
[EXP]
Thứ tự này sai vì bạn không thể SSH vào restart dịch vụ (bước 1) trước khi truyền tải file JAR mới lên server (bước 3).
[C]
Thứ tự thực hiện các bước triển khai thủ công là: (2) -> (3) -> (1)
[EXP]
Thứ tự này hoàn toàn chính xác: trước hết cần đóng gói mã nguồn tại local -> truyền file JAR đã đóng gói lên server -> SSH vào server để restart dịch vụ chạy file mới.
[D]
Thứ tự thực hiện các bước triển khai thủ công là: (3) -> (2) -> (1)
[EXP]
Thứ tự này không thể thực hiện được vì bạn không thể truyền file JAR (bước 3) trước khi chạy lệnh đóng gói ra file JAR đó (bước 2).


@correct: C
@point: 20

## Q3

Cho đoạn cấu hình dịch vụ Systemd sau đây trên server Linux:

```ini
[Service]
WorkingDirectory=/opt/quickbite
ExecStart=/usr/bin/java -jar /opt/quickbite/user-service-0.0.1.jar
Restart=always
RestartSec=10
```

Thuộc tính `RestartSec=10` có ý nghĩa như thế nào đối với ứng dụng?

[A]
Giới hạn thời gian chạy tối đa của ứng dụng là 10 giây trước khi hệ thống tự động tắt đi và khởi chạy lại dịch vụ.
[EXP]
Cấu hình này không giới hạn thời gian chạy của ứng dụng mà chỉ quy định thời gian chờ trước khi tiến hành khởi động lại.
[B]
Thiết lập thời gian chờ 10 giây trước khi tiến hành khởi động lại ứng dụng nếu tiến trình bị sập hoặc dừng đột ngột.
[EXP]
Đây là ý nghĩa chính xác của `RestartSec=10`. Thuộc tính này chỉ định thời gian delay (chờ) 10 giây trước khi Systemd khởi chạy lại dịch vụ bị sập.
[C]
Chỉ cho phép ứng dụng tự động khởi động lại tối đa 10 lần, sau đó dịch vụ Systemd sẽ bị vô hiệu hóa hoàn toàn.
[EXP]
Systemd không giới hạn số lần restart ở thuộc tính này. Việc restart sẽ diễn ra liên tục mỗi khi sập do có cấu hình `Restart=always`.
[D]
Trì hoãn việc khởi chạy ứng dụng lần đầu tiên 10 giây tính từ thời điểm hệ điều hành Linux khởi động thành công.
[EXP]
Cấu hình này không trì hoãn lần khởi chạy đầu tiên mà chỉ áp dụng khi tiến trình Java bị sập đột ngột trong lúc đang chạy.


@correct: B
@point: 20

## Q4

Sự khác biệt mấu chốt giữa việc cấu hình Systemd tự khởi động lại ứng dụng với việc áp dụng triết lý DevOps đầy đủ là gì?

[A]
Systemd chỉ quản lý tiến trình tĩnh cục bộ; DevOps yêu cầu tự động hóa toàn bộ vòng đời từ build, test đến deploy.
[EXP]
Systemd chỉ là một công cụ quản lý tiến trình cục bộ trên một server đơn lẻ, không tự động kéo code từ Git, không tự chạy test hay đóng gói. DevOps là sự kết hợp văn hóa, quy trình và tự động hóa khép kín.
[B]
Systemd chạy ứng dụng dưới quyền root; DevOps bắt buộc ứng dụng phải chạy trên Docker với tài khoản hệ thống chuyên dụng.
[EXP]
Systemd có thể cấu hình chạy ứng dụng dưới quyền user bất kỳ thông qua từ khóa `User=...` và DevOps không bắt buộc phải chạy Docker ở mọi trường hợp.
[C]
Systemd không thể cấu hình biến môi trường; DevOps chỉ tập trung vào việc quản lý biến cấu hình thông qua file cấu hình.
[EXP]
Systemd hoàn toàn hỗ trợ cấu hình biến môi trường thông qua thuộc tính `Environment=...`. Phát biểu này sai về mặt kỹ thuật.
[D]
Systemd chỉ hỗ trợ các ứng dụng Java Spring Boot; DevOps là phương pháp luận áp dụng được cho mọi ngôn ngữ lập trình.
[EXP]
Systemd hỗ trợ quản lý tiến trình cho mọi loại ứng dụng (NodeJS, Go, Python...), không chỉ riêng ứng dụng Java Spring Boot.


@correct: A
@point: 20

## Q5

Chuyện gì sẽ xảy ra nếu một ứng dụng Java Spring Boot được triển khai thủ công lên một VPS có cấu hình tối giản (1 vCPU, 1GB RAM) mà lập trình viên không cấu hình giới hạn dung lượng bộ nhớ Heap cho JVM (JVM Heap size) khi ứng dụng gặp lượng traffic lớn?

[A]
Ứng dụng sẽ tự động phân bổ tài nguyên từ CPU sang RAM để tránh tình trạng tràn bộ nhớ vật lý của VPS.
[EXP]
JVM và hệ điều hành không thể tự động chuyển đổi tài nguyên giữa CPU và RAM. Đây là phát biểu phi vật lý.
[B]
Hệ điều hành Linux sẽ kích hoạt tiến trình OOM Killer để tắt tiến trình Java nhằm bảo vệ tài nguyên hệ thống.
[EXP]
Nếu không cấu hình giới hạn Heap size, JVM có thể chiếm dụng vượt quá giới hạn RAM vật lý của VPS (1GB). Lúc này, Linux Kernel sẽ kích hoạt Out of Memory (OOM) Killer để dừng khẩn cấp tiến trình Java để tránh sập hệ điều hành.
[C]
Ứng dụng sẽ tự động tối ưu hóa và giải phóng bộ nhớ Heap liên tục mà không gây ảnh hưởng đến hiệu năng của server.
[EXP]
JVM chỉ giải phóng bộ nhớ qua Garbage Collection (GC), nhưng nếu dung lượng RAM quá thấp và traffic tăng cao, GC không thể cứu vãn tình trạng thiếu bộ nhớ vật lý.
[D]
Hệ thống sẽ tự động cấu hình lại bộ nhớ của VPS để đáp ứng lượng tải thực tế từ người dùng cuối.
[EXP]
Hạ tầng VPS không thể tự động nâng cấp dung lượng RAM vật lý nếu không có cấu hình Auto-scaling tầng hạ tầng đám mây được thiết lập trước.


@correct: B
@point: 20

---

# LESSON 02: Khái niệm CI/CD (quy trình build, test, deploy)

## Q1

Vai trò cốt lõi của "lưới an toàn" Unit Test trong giai đoạn Tích hợp liên tục (Continuous Integration - CI) là gì?

[A]
Đảm bảo ứng dụng chạy mượt mà trên môi trường Production mà không cần phải thực hiện cấu hình các cổng kết nối mạng.
[EXP]
Unit Test kiểm tra logic code ở mức đơn vị chứ không kiểm tra hoặc giải quyết các vấn đề cấu hình mạng hay chạy trên Production.
[B]
Tự động gửi email thông báo cho Tech Lead biết khi nào lập trình viên hoàn thành việc đẩy code mới lên kho chứa.
[EXP]
Việc gửi email thông báo là tính năng của Git Platform (như GitHub/GitLab), không phải vai trò cốt lõi của việc chạy Unit Test.
[C]
Tự động kiểm tra để phát hiện lỗi logic và đảm bảo code mới đẩy lên không làm phá vỡ các tính năng sẵn có.
[EXP]
Unit Test hoạt động như một lưới lọc tự động, giúp phát hiện lỗi logic sớm (Fail-fast) và đảm bảo code mới không làm ảnh hưởng các chức năng cũ.
[D]
Đóng gói mã nguồn thành các file thực thi có định dạng JAR hoặc Docker Image để sẵn sàng triển khai lên server.
[EXP]
Đóng gói mã nguồn là nhiệm vụ của giai đoạn Package trong pipeline, không phải nhiệm vụ của Unit Test.


@correct: C
@point: 20

## Q2

Theo nguyên lý nghiêm ngặt của đường ống (Pipeline) CI/CD tiêu chuẩn, điều gì sẽ xảy ra nếu một bài kiểm thử (Unit Test) bị thất bại ở giai đoạn Test?

[A]
Toàn bộ đường ống sẽ dừng lại ngay lập tức, hủy bỏ quá trình đóng gói và gửi thông báo lỗi cho lập trình viên.
[EXP]
Đây là cơ chế Fail-fast (thất bại sớm) của pipeline CI/CD để bảo vệ môi trường chạy tránh các lỗi logic thô chưa được kiểm soát.
[B]
Đường ống sẽ tự động bỏ qua bài test bị lỗi để tiếp tục thực hiện đóng gói và deploy ứng dụng lên server.
[EXP]
Pipeline sẽ không bao giờ bỏ qua lỗi test để tiếp tục vì điều này sẽ làm giảm chất lượng sản phẩm và gây rủi ro lỗi cho hệ thống thật.
[C]
Đường ống vẫn đóng gói ứng dụng thành file JAR nhưng sẽ cảnh báo lỗi khi khởi chạy trên môi trường Production.
[EXP]
Pipeline sẽ dừng lại ngay lập tức và không thực hiện đóng gói file JAR ở bước tiếp theo để đảm bảo tính an toàn.
[D]
Đường ống sẽ tự động sửa đổi mã nguồn dựa trên lịch sử commit trước đó để vượt qua bài kiểm thử bị thất bại.
[EXP]
Pipeline không có khả năng tự động sửa đổi logic mã nguồn của lập trình viên để vượt qua các bài kiểm thử bị lỗi.


@correct: A
@point: 20

## Q3

Một bạn Intern cấu hình một script đơn giản tự động copy file JAR lên server mỗi khi đẩy code lên Git mà không chạy Unit Test. Phát biểu nào dưới đây mô tả chính xác nhất về quy trình này?

[A]
Đây là một quy trình CI/CD hoàn chỉnh vì đã tự động hóa việc đưa mã nguồn mới lên chạy trên môi trường Production.
[EXP]
Quy trình này hoàn toàn thiếu đi bước kiểm soát chất lượng mã nguồn tự động ở môi trường độc lập, nên không thể gọi là CI/CD hoàn chỉnh.
[B]
Đây chỉ là tự động hóa thao tác triển khai (CD), hoàn toàn thiếu đi cấu phần kiểm soát chất lượng mã nguồn (CI).
[EXP]
Chính xác. Việc chỉ tự động copy file mới chỉ là phần CD (Deployment). Nếu không có bước biên dịch độc lập và chạy Unit Test để lọc lỗi logic, bạn chưa hề có CI.
[C]
Đây là quy trình Continuous Delivery chuẩn vì nó đã tạo ra file đóng gói và chuyển lên server thành công.
[EXP]
Continuous Delivery yêu cầu một quy trình kiểm soát chất lượng nghiêm ngặt trước đó thông qua CI và thường có bước phê duyệt thủ công.
[D]
Đây là quy trình Continuous Integration vì đã tích hợp thành công mã nguồn từ máy local lên máy chủ Git tập trung.
[EXP]
Tích hợp mã nguồn lên Git chỉ là bước kích hoạt (trigger), CI đòi hỏi quá trình tự động biên dịch và chạy kiểm thử tự động.


@correct: B
@point: 20

## Q4

Điểm khác biệt mấu chốt để phân biệt giữa Continuous Delivery (Chuyển giao liên tục) và Continuous Deployment (Triển khai liên tục) là gì?

[A]
Môi trường triển khai của Continuous Delivery là Staging, còn của Continuous Deployment bắt buộc phải là Production.
[EXP]
Cả hai quy trình đều có đích đến cuối cùng là môi trường Production, sự khác biệt nằm ở cơ chế phê duyệt khi đưa code lên.
[B]
Continuous Delivery chỉ đóng gói file JAR, còn Continuous Deployment bắt buộc phải đóng gói ứng dụng dưới dạng Docker Image.
[EXP]
Cả hai đều có thể sử dụng file JAR hoặc Docker Image để triển khai, định dạng đóng gói không phải điểm phân biệt hai khái niệm này.
[C]
Continuous Delivery chạy toàn bộ các bài Unit Test, còn Continuous Deployment chỉ chạy các bài kiểm thử hiệu năng hệ thống.
[EXP]
Cả hai quy trình đều bắt buộc phải chạy đầy đủ Unit Test ở giai đoạn CI để kiểm soát chất lượng mã nguồn trước khi deploy.
[D]
Continuous Delivery cần nhấn nút duyệt thủ công (Manual Approval) để lên Prod, còn Continuous Deployment tự động hóa 100%.
[EXP]
Đây là điểm khác biệt cốt lõi. Continuous Delivery cần con người bấm nút duyệt để deploy lên Prod (ví dụ để kiểm soát thời điểm phát hành), còn Continuous Deployment tự động hoàn toàn không cần can thiệp.


@correct: D
@point: 20

## Q5

Giả sử một dự án Java Spring Boot đang chạy pipeline CI/CD. Ở commit mới nhất, lập trình viên sửa code lỗi cú pháp dẫn đến việc biên dịch thất bại ở bước `Compile`. Dự đoán kết quả đối với ứng dụng đang chạy trên server Production.

[A]
Ứng dụng trên server Production sẽ bị crash ngay lập tức vì nhận được file JAR mới nhưng không thể chạy được.
[EXP]
Vì biên dịch thất bại ở bước đầu tiên nên file JAR mới chưa được đóng gói và chuyển lên server, do đó server Production không bị ảnh hưởng.
[B]
Hệ thống sẽ tự động khôi phục (rollback) mã nguồn trên máy local của lập trình viên về phiên bản commit trước đó.
[EXP]
Pipeline không can thiệp để thay đổi code trên máy local của nhà phát triển mà chỉ dừng luồng chạy của đường ống tự động.
[C]
Ứng dụng trên server Production vẫn hoạt động bình thường với phiên bản cũ, vì file JAR mới chưa từng được tạo ra.
[EXP]
Đúng. Đây là cơ chế tự bảo vệ của pipeline: bất kỳ bước nào thất bại, pipeline dừng ngay lập tức, đảm bảo phiên bản cũ ổn định trên Prod không bị ghi đè bởi code lỗi.
[D]
Hệ điều hành trên server Production sẽ báo lỗi phân quyền khi cố gắng nạp file cấu hình của phiên bản bị lỗi biên dịch.
[EXP]
Hệ điều hành trên server Production không hề nhận được file cấu hình hay mã nguồn mới của commit lỗi này nên không có lỗi phân quyền nào xảy ra.


@correct: C
@point: 20

---

# LESSON 03: Mô hình môi trường Dev, Staging và Production

## Q1

Trong file cấu hình `application.yml` của Spring Boot, cú pháp placeholder `${QUICKBITE_DB_USER:postgres}` có ý nghĩa gì?

[A]
Tìm biến môi trường `QUICKBITE_DB_USER`; nếu có thì nạp giá trị đó, nếu không có thì tự động dùng giá trị mặc định `postgres`.
[EXP]
Đây là cú pháp nạp cấu hình động của Spring Boot. Nó ưu tiên lấy giá trị từ biến môi trường hệ thống, nếu trống sẽ fallback về giá trị mặc định sau dấu hai chấm.
[B]
Thiết lập biến môi trường `QUICKBITE_DB_USER` vào hệ điều hành của server với giá trị mặc định được chỉ định là `postgres`.
[EXP]
Spring Boot chỉ đọc cấu hình chứ không có nhiệm vụ thiết lập hay thay đổi biến môi trường của hệ điều hành.
[C]
Chỉ chấp nhận giá trị từ biến môi trường `QUICKBITE_DB_USER` và sẽ báo lỗi khởi chạy nếu biến này có giá trị là `postgres`.
[EXP]
Cú pháp này hoàn toàn cho phép chạy với giá trị `postgres` khi đóng vai trò là giá trị fallback mặc định.
[D]
Tạo ra một tài khoản cơ sở dữ liệu mới tên là `postgres` trong hệ quản trị cơ sở dữ liệu PostgreSQL của môi trường Staging.
[EXP]
Cấu hình này chỉ định thông tin đăng nhập của ứng dụng, không có chức năng khởi tạo tài khoản mới trong cơ sở dữ liệu PostgreSQL.


@correct: A
@point: 20

## Q2

Theo nguyên tắc vàng của DevOps "Build once, run anywhere", hành động nào dưới đây là ĐÚNG khi chuyển giao ứng dụng qua các môi trường khác nhau?

[A]
Biên dịch lại mã nguồn của ứng dụng cho từng môi trường để đảm bảo tương thích với các phiên bản hệ điều hành khác nhau.
[EXP]
Biên dịch lại làm mất tính nhất quán của phiên bản (Consistency). File chạy trên Prod phải chính xác là file đã được kiểm thử thành công trên Staging.
[B]
Build một file JAR duy nhất từ máy chủ CI và nạp các tham số cấu hình khác nhau bằng biến môi trường khi khởi chạy.
[EXP]
Đây chính là nguyên tắc cốt lõi: chỉ đóng gói ứng dụng một lần duy nhất, sự khác biệt giữa các môi trường (Dev, Staging, Prod) được giải quyết bằng các biến môi trường nạp ở runtime.
[C]
Sử dụng các file cấu hình cứng như `application-dev.yml` hay `application-prod.yml` lưu trực tiếp trên kho lưu trữ Git.
[EXP]
Cách này vi phạm nguyên tắc bảo mật và quản lý cấu hình tập trung vì lưu thông tin nhạy cảm của các môi trường (đặc biệt là Prod) lên Git.
[D]
Cài đặt các phiên bản Java Runtime (JRE) khác nhau trên mỗi máy chủ để tối ưu hóa hiệu năng cho từng môi trường cụ thể.
[EXP]
Để đảm bảo tính nhất quán và tránh lỗi lệch pha môi trường chạy (Java Runtime Mismatch), các môi trường cần chạy chung một phiên bản Java thống nhất.


@correct: B
@point: 20

## Q3

Lập trình viên cấu hình dòng password kết nối database trong file `application.yml` như sau:

```yaml
spring:
  datasource:
    password: ${QUICKBITE_DB_PASS}
```

Chuyện gì xảy ra nếu lập trình viên khởi chạy ứng dụng bằng lệnh `java -jar user-service-0.0.1.jar` ở máy local mà không nạp biến môi trường `QUICKBITE_DB_PASS`?

[A]
Ứng dụng sẽ tự động sử dụng giá trị mặc định là chuỗi rỗng để tiến hành kết nối đến cơ sở dữ liệu.
[EXP]
Spring Boot không tự động chuyển đổi thành chuỗi rỗng nếu không có dấu hai chấm biểu thị giá trị mặc định phía sau tên biến.
[B]
Ứng dụng sẽ khởi chạy bình thường nhưng sẽ bị crash khi nhận được request đọc ghi dữ liệu đầu tiên từ phía Client.
[EXP]
Ứng dụng sẽ bị crash ngay từ giai đoạn khởi chạy (bootstrap) do lỗi nạp cấu hình cơ sở dữ liệu, không đợi đến khi nhận request.
[C]
Ứng dụng sẽ báo lỗi khởi chạy thất bại ngay lập tức vì không thể phân giải được placeholder cấu hình password.
[EXP]
Chính xác. Vì không có dấu hai chấm quy định giá trị mặc định và hệ thống không tìm thấy biến môi trường `QUICKBITE_DB_PASS`, Spring Boot sẽ quăng lỗi khởi chạy thất bại.
[D]
Hệ điều hành sẽ tự động tạo một biến môi trường tạm thời để cấp quyền truy cập cơ sở dữ liệu cho ứng dụng.
[EXP]
Hệ điều hành không thể tự đoán và sinh ra mật khẩu cơ sở dữ liệu cho ứng dụng khi khởi chạy.


@correct: C
@point: 20

## Q4

Phát biểu nào sau đây so sánh ĐÚNG về đặc điểm dữ liệu và yêu cầu độ ổn định giữa môi trường Staging và Production?

[A]
Staging sử dụng dữ liệu giả lập tự tạo và yêu cầu độ ổn định thấp; Production sử dụng dữ liệu thật và yêu cầu độ ổn định tuyệt đối.
[EXP]
Staging cần mô phỏng dữ liệu thật để kiểm thử chân thực nhất có thể, yêu cầu độ ổn định ở mức trung bình đến cao (giống Prod 99%).
[B]
Staging sử dụng dữ liệu thật nhạy cảm và yêu cầu độ ổn định trung bình; Production sử dụng dữ liệu thật và yêu cầu độ ổn định tuyệt đối.
[EXP]
Staging không được phép chứa dữ liệu thật nhạy cảm của khách hàng để đảm bảo tính bảo mật và tuân thủ dữ liệu.
[C]
Staging sử dụng dữ liệu thật của người dùng cuối và yêu cầu độ ổn định tuyệt đối; Production chỉ dùng dữ liệu test của QA/QC.
[EXP]
Phát biểu này bị ngược. Production mới là nơi chứa dữ liệu thật của người dùng cuối và yêu cầu độ ổn định tuyệt đối.
[D]
Staging mô phỏng dữ liệu thật không nhạy cảm với độ ổn định cao; Production chạy dữ liệu thật và độ ổn định yêu cầu tuyệt đối.
[EXP]
Đúng. Staging (UAT) cần mô phỏng dữ liệu gần nhất với thực tế nhưng đã loại bỏ thông tin nhạy cảm, đồng thời làm bệ phóng thử nghiệm trước khi đẩy lên môi trường Prod có tính ổn định tuyệt đối.


@correct: D
@point: 20

## Q5

Tại sao việc commit các file cấu hình cứng chứa thông tin mật khẩu của môi trường Production (ví dụ: `application-prod.yml`) lên một Private Repository trên GitHub/GitLab vẫn được coi là một hành vi vi phạm nguyên tắc bảo mật nghiêm trọng?

[A]
Vì các máy chủ của GitHub/GitLab sẽ tự động quét và vô hiệu hóa các file cấu hình có chứa từ khóa liên quan đến mật khẩu.
[EXP]
Các nền tảng Git chỉ có thể cảnh báo rò rỉ (secret scanning) chứ không tự động vô hiệu hóa hay xóa file cấu hình của bạn.
[B]
Vì kho lưu trữ riêng tư vẫn có nhiều người truy cập (dev, tester, thực tập sinh), làm lộ lọt thông tin quản trị hệ thống thật.
[EXP]
Chính xác. Private repo vẫn được phân quyền truy cập rộng rãi cho toàn bộ đội ngũ dự án (gồm cả các cộng tác viên, thực tập sinh thời vụ). Mật khẩu Prod chỉ được phép xuất hiện ở runtime trên máy chủ Production.
[C]
Vì việc commit file cấu hình lên Git sẽ làm tăng dung lượng file JAR khi đóng gói, gây lãng phí tài nguyên của máy chủ.
[EXP]
Dung lượng một file cấu hình yml là cực nhỏ (vài KB), không ảnh hưởng đáng kể đến kích thước file JAR hay hiệu năng của máy chủ.
[D]
Vì hệ thống CI/CD sẽ báo lỗi biên dịch khi phát hiện file cấu hình của môi trường Production tồn tại trong mã nguồn của Dev.
[EXP]
Hệ thống CI/CD vẫn biên dịch bình thường và không ngăn chặn việc này bằng lỗi biên dịch, đây thuần túy là lỗi quản trị bảo mật của con người.


@correct: B
@point: 20

---

# LESSON 04: Kiến trúc triển khai hệ thống Microservices và luồng đi của dữ liệu

## Q1

Thành phần Nginx đóng vai trò gì ở Edge Layer (Lớp biên giới) trong kiến trúc triển khai tổng thể của QuickBite?

[A]
Chạy các logic nghiệp vụ của ứng dụng Spring Boot và quản lý kết nối trực tiếp đến PostgreSQL Database.
[EXP]
Nginx không chạy mã nguồn Java Spring Boot và không tương tác trực tiếp với cơ sở dữ liệu PostgreSQL.
[B]
Đóng vai trò là tấm khiên Reverse Proxy ở rìa ngoài hệ thống, tiếp nhận và điều hướng request từ Public Internet.
[EXP]
Đây là vai trò cốt lõi của Nginx. Nó nhận các request HTTP/HTTPS từ người dùng, thực hiện SSL Termination và điều hướng đến các thành phần bên trong.
[C]
Quản lý các biến môi trường và thiết lập quyền truy cập cho các tài khoản chạy dịch vụ trên server Linux.
[EXP]
Việc quản lý biến môi trường và phân quyền thuộc trách nhiệm của hệ điều hành Linux, không liên quan đến Nginx.
[D]
Thực hiện tự động hóa việc kiểm soát chất lượng mã nguồn mỗi khi lập trình viên thực hiện đẩy code lên Git.
[EXP]
Đây là nhiệm vụ của CI/CD Engine (như GitHub Actions, GitLab CI), Nginx không tham gia vào quá trình xây dựng mã nguồn.


@correct: B
@point: 20

## Q2

Khi người dùng truy cập vào API backend (`api.quickbite.com`), luồng đi chuẩn xác của dữ liệu qua các thành phần hệ thống là gì?

[A]
Client -> API Gateway -> Nginx -> user-service -> PostgreSQL DB
[EXP]
Luồng này sai vì Nginx nằm ở ngoài cùng tiếp nhận kết nối trước, sau đó mới chuyển tiếp đến API Gateway ở phía sau.
[B]
Client -> user-service -> API Gateway -> Nginx -> PostgreSQL DB
[EXP]
Luồng này sai vì Client không thể kết nối trực tiếp đến `user-service` nằm trong mạng nội bộ.
[C]
Client -> Nginx -> API Gateway -> user-service -> PostgreSQL DB
[EXP]
Đây là luồng dữ liệu chuẩn xác và bảo mật: Client gửi request đến Nginx -> Nginx proxy đến API Gateway -> Gateway định tuyến đến `user-service` -> Service truy vấn dữ liệu từ PostgreSQL.
[D]
Client -> Nginx -> user-service -> API Gateway -> PostgreSQL DB
[EXP]
Luồng này sai vì Nginx chuyển tiếp request đến API Gateway, chứ không đi trực tiếp vào các microservices nội bộ như `user-service`.


@correct: C
@point: 20

## Q3

Khi cấu hình sơ đồ kiến trúc triển khai của QuickBite, tại sao các service nghiệp vụ như `user-service` và `order-service` lại được thiết kế nằm ẩn hoàn toàn trong mạng nội bộ (Private Network)?

[A]
Để ngăn chặn các truy cập trực tiếp từ Internet vào service, phòng ngừa hacker scan cổng và tấn công trực diện.
[EXP]
Đây là lý do bảo mật quan trọng nhất. Bằng cách giấu các dịch vụ nghiệp vụ trong mạng nội bộ, hacker không thể dò quét cổng hoặc khai thác lỗi dịch vụ trực tiếp từ internet.
[B]
Để giảm thiểu chi phí băng thông truyền tải dữ liệu giữa các máy chủ dịch vụ của hệ thống QuickBite.
[EXP]
Việc đặt trong mạng nội bộ giúp tối ưu kết nối nội bộ nhưng mục đích cốt lõi và quan trọng nhất vẫn là bảo mật hệ thống.
[C]
Để giúp Nginx có thể dễ dàng quản lý việc giải mã HTTPS và phân tải tĩnh mà không cần đi qua API Gateway.
[EXP]
Nginx thực hiện SSL Termination độc lập ở Edge Layer, vị trí của các microservice trong mạng nội bộ không phục vụ chức năng này của Nginx.
[D]
Để đảm bảo rằng các service nghiệp vụ có thể chia sẻ chung một cơ sở dữ liệu vật lý PostgreSQL duy nhất.
[EXP]
Việc đặt trong mạng nội bộ không quyết định hay ảnh hưởng đến việc các service có dùng chung máy chủ database vật lý hay không.


@correct: A
@point: 20

## Q4

Sự khác biệt mấu chốt về chức năng giữa Nginx và API Gateway trong mô hình triển khai Microservices là gì?

[A]
Nginx xử lý logic nghiệp vụ phần mềm; API Gateway xử lý các tác vụ ở tầng hạ tầng mạng và giải mã HTTPS.
[EXP]
Nginx mới là thành phần xử lý ở tầng hạ tầng mạng (SSL Termination, HTTP Proxy), còn API Gateway hoạt động ở tầng ứng dụng.
[B]
Nginx xử lý việc định tuyến động theo đường dẫn; API Gateway chịu trách nhiệm phân tải tĩnh cho ứng dụng Frontend.
[EXP]
Nginx thường chịu trách nhiệm phân tải và lưu trữ file tĩnh cho Frontend, còn API Gateway đảm nhiệm định tuyến động và lọc request tầng ứng dụng.
[C]
Nginx chỉ hoạt động trên môi trường Staging; API Gateway là thành phần bắt buộc phải có trên môi trường Production.
[EXP]
Cả Nginx và API Gateway đều là các thành phần kiến trúc cần thiết trên cả môi trường Staging lẫn Production.
[D]
Nginx giải quyết bài toán ở tầng mạng/hiệu năng; API Gateway xử lý logic ứng dụng như định tuyến động, Auth Filter.
[EXP]
Chính xác. Nginx hoạt động tối ưu ở tầng mạng (Reverse proxy, SSL termination, static server). API Gateway (ví dụ Spring Cloud Gateway) chạy ở tầng ứng dụng để xử lý logic như xác thực, phân quyền và routing động.


@correct: D
@point: 20

## Q5

Điều gì sẽ xảy ra nếu lập trình viên cho phép Client (Web/Mobile) kết nối trực tiếp đến IP và cổng của từng microservice (ví dụ `http://10.0.1.15:8080`) thay vì đi qua Nginx và API Gateway?

[A]
Hệ thống sẽ hoạt động nhanh hơn do giảm bớt các bước trung gian nhưng lại đối mặt với lỗ hổng bảo mật nghiêm trọng.
[EXP]
Kết nối trực tiếp bỏ qua các chốt chặn bảo mật (tường lửa, HTTPS tập trung, Auth Filter), làm lộ địa chỉ IP thật của server và tạo ra các lỗ hổng bảo mật chết người.
[B]
Ứng dụng Client sẽ bị crash ngay lập tức vì Spring Boot không hỗ trợ giao thức kết nối trực tiếp không qua gateway.
[EXP]
Spring Boot chạy độc lập hoàn toàn có thể tiếp nhận request trực tiếp từ Client qua giao thức HTTP thông thường.
[C]
Hệ quản trị cơ sở dữ liệu PostgreSQL sẽ tự động chặn các kết nối từ client để bảo vệ an toàn cho dữ liệu.
[EXP]
PostgreSQL chỉ quản lý kết nối từ backend đến nó, không liên quan và không thể tự động can thiệp vào kết nối giữa Client và backend.
[D]
Đường ống CI/CD sẽ tự động dừng lại và cảnh báo lỗi bảo mật ở bước kiểm tra cú pháp của file cấu hình.
[EXP]
Đường ống CI/CD không có nhiệm vụ phân tích kiến trúc mạng và phát hiện việc mở cổng dịch vụ trực tiếp ra ngoài.


@correct: A
@point: 20

---

# LESSON 05: Hệ điều hành Linux và vai trò trong triển khai hệ thống

## Q1

Để nạp lại cấu hình của file `.bashrc` ngay lập tức trên Terminal hiện hành mà không cần phải thực hiện đăng xuất (logout), bạn sử dụng câu lệnh nào?

[A]
`sudo systemctl restart bashrc`
[EXP]
`.bashrc` là file cấu hình shell của người dùng, không phải là một background service được quản lý bởi Systemd.
[B]
`source ~/.bashrc`
[EXP]
Chính xác. Lệnh `source` (hoặc dấu chấm `.`) sẽ nạp và thực thi các khai báo trong file cấu hình `.bashrc` ngay lập tức trên session terminal hiện tại.
[C]
`nginx -s reload`
[EXP]
Lệnh này dùng để nạp lại cấu hình của máy chủ web Nginx, không liên quan đến biến môi trường trong cấu hình shell của hệ điều hành.
[D]
`chmod +x ~/.bashrc`
[EXP]
Lệnh này chỉ cấp quyền thực thi cho file `.bashrc`, không có tác dụng nạp lại các biến môi trường vào session làm việc hiện tại.


@correct: B
@point: 20

## Q2

Quy trình cập nhật cấu hình cho máy chủ Nginx một cách an toàn và không làm gián đoạn các kết nối hiện tại của khách hàng được thực hiện như thế nào?

[A]
Chạy `sudo nginx -t` để kiểm tra lỗi cú pháp, sau đó chạy `sudo nginx -s reload` để tải cấu hình mới.
[EXP]
Đây là quy trình chuẩn. `nginx -t` kiểm tra cú pháp file config tránh làm sập Nginx nếu có lỗi, sau đó `nginx -s reload` nạp cấu hình mới một cách trơn tru mà không ngắt kết nối hiện tại.
[B]
Chạy `sudo systemctl stop nginx` để dừng dịch vụ, chỉnh sửa cấu hình rồi chạy `sudo systemctl start nginx`.
[EXP]
Cách này gây gián đoạn dịch vụ (downtime) vì phải dừng máy chủ web trong lúc chỉnh sửa và khởi động lại.
[C]
Khởi động lại hệ điều hành Linux để tự động nạp lại toàn bộ file cấu hình của Nginx từ thư mục lưu trữ.
[EXP]
Việc khởi động lại hệ điều hành gây thời gian chết rất lớn cho toàn bộ hệ thống và là hành động cực kỳ tối kỵ trong vận hành.
[D]
Chạy `source ~/.bashrc` để nạp các biến môi trường của Nginx, sau đó dùng lệnh `kill -9` để kết thúc tiến trình cũ.
[EXP]
Reload Nginx không liên quan đến `.bashrc` và dùng `kill -9` sẽ cưỡng bức ngắt toàn bộ kết nối hiện tại của khách hàng.


@correct: A
@point: 20

## Q3

Đoạn script khởi tạo hệ thống ban đầu (`initial-script.sh`) dưới đây thực hiện các nhiệm vụ gì?

```bash
#!/bin/bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install -y openjdk-17-jdk curl git
```

Chọn câu trả lời mô tả chính xác và đầy đủ nhất:

[A]
Tự động tải mã nguồn dự án từ Git, cài đặt môi trường Java JDK 17 và khởi chạy dịch vụ Nginx trên server Linux.
[EXP]
Script này chỉ cài đặt công cụ git chứ không có lệnh tải mã nguồn và không cài đặt hay khởi chạy dịch vụ Nginx.
[B]
Đóng gói file JAR của Spring Boot, cấp quyền thực thi cho file script và tự động restart ứng dụng bằng Systemd.
[EXP]
Script này không chứa các lệnh đóng gói hay thao tác với dịch vụ Systemd của ứng dụng.
[C]
Cập nhật hệ điều hành, nâng cấp phần mềm bảo mật, cài đặt môi trường Java JDK 17 và hai công cụ curl, git.
[EXP]
Đúng. Script thực hiện cập nhật danh sách gói (`update`), nâng cấp hệ thống (`upgrade`), cài JDK 17 (`openjdk-17-jdk`) và cài thêm curl, git.
[D]
Cấu hình các biến môi trường cho cơ sở dữ liệu PostgreSQL và phân quyền truy cập cho tài khoản người dùng hệ thống.
[EXP]
Script không có dòng lệnh nào thiết lập biến môi trường hay phân quyền truy cập người dùng hệ thống.


@correct: C
@point: 20

## Q4

Sự khác biệt về phạm vi tác dụng và vòng đời của biến môi trường được khai báo bằng lệnh `export DB_PASS=123` so với biến được ghi vào file `.bashrc` là gì?

[A]
Lệnh `export` tạo biến vĩnh viễn trên toàn hệ thống; biến trong `.bashrc` chỉ có tác dụng trong phiên terminal hiện tại.
[EXP]
Phát biểu này bị ngược. Lệnh `export` trực tiếp trên terminal chỉ tạo biến tạm thời, còn ghi vào `.bashrc` mới giúp lưu trữ lâu dài.
[B]
Lệnh `export` chỉ chạy được trên hệ điều hành Windows; biến trong `.bashrc` là định dạng cấu hình dành riêng cho Linux.
[EXP]
Cả hai cách này đều là các câu lệnh và cấu hình trên hệ điều hành Linux, không dùng cho Windows.
[C]
Lệnh `export` yêu quyền root để thực thi; biến trong `.bashrc` có thể được cấu hình bởi bất kỳ tài khoản thông thường nào.
[EXP]
Lệnh `export` thông thường không đòi hỏi quyền root (sudo) để khai báo biến môi trường cho session của user hiện tại.
[D]
Lệnh `export` tạo biến tạm thời sẽ mất khi đóng terminal; biến trong `.bashrc` tự động nạp ở mọi phiên làm việc mới của user.
[EXP]
Chính xác. Biến khai báo qua `export` trực tiếp sẽ biến mất khi tắt session shell. Biến ghi vào `.bashrc` sẽ tồn tại vĩnh viễn và tự động nạp mỗi khi user đăng nhập terminal mới.


@correct: D
@point: 20

## Q5

Giả sử bạn chạy lệnh `ps -ef | grep java` và phát hiện ứng dụng Java Spring Boot đang chạy với ID tiến trình (PID) là `14522`. Khi thực hiện lệnh `kill -9 14522`, điều gì sẽ xảy ra?

[A]
Tiến trình Java sẽ nhận được tín hiệu dừng và tiến hành giải phóng bộ nhớ từ từ trước khi tắt hoàn toàn dịch vụ.
[EXP]
Tín hiệu dừng thông thường (SIGTERM) mới giúp tiến trình tắt từ từ. Lệnh `kill -9` gửi tín hiệu SIGKILL để kết liễu tiến trình ngay lập tức.
[B]
Hệ điều hành Linux sẽ lập tức chấm dứt (kill) tiến trình Java một cách cưỡng bức mà không chờ giải phóng tài nguyên.
[EXP]
Chính xác. Cờ `-9` (SIGKILL) ra lệnh cho hệ điều hành buộc dừng tiến trình ngay lập tức mà không cho phép tiến trình kịp dọn dẹp hay lưu trữ trạng thái.
[C]
Dịch vụ Systemd sẽ tự động nhận diện và cấu hình lại cổng chạy của ứng dụng Java sang cổng dự phòng `8081`.
[EXP]
Systemd không thể tự động thay đổi cổng chạy của ứng dụng Java sang một cổng khác khi tiến trình bị tắt.
[D]
Ứng dụng Java sẽ tự động khởi động lại và nạp lại toàn bộ cấu hình từ file `.bashrc` của user chạy dịch vụ.
[EXP]
Việc khởi động lại chỉ xảy ra nếu tiến trình này được cấu hình chạy dưới dạng dịch vụ của Systemd với tùy chọn `Restart=always`.


@correct: B
@point: 20

---

# LESSON 06: Quản lý quyền và các lệnh mạng cơ bản trong Linux

## Q1

Khi tạo người dùng hệ thống chuyên dụng bằng lệnh `useradd` để chạy ứng dụng Java Spring Boot, tùy chọn nào dưới đây giúp vô hiệu hóa shell đăng nhập nhằm đảm bảo an toàn bảo mật?

[A]
Tùy chọn `-r` để đánh dấu đây là tài khoản hệ thống.
[EXP]
Tùy chọn `-r` tạo một user hệ thống với UID thấp, nhưng không tự động vô hiệu hóa shell đăng nhập của user đó.
[B]
Tùy chọn `-s /bin/false` để ngăn chặn việc mở shell đăng nhập.
[EXP]
Chính xác. Chỉ định shell đăng nhập là `/bin/false` (hoặc `/usr/sbin/nologin`) sẽ vô hiệu hóa hoàn toàn khả năng đăng nhập trực tiếp hoặc qua SSH của tài khoản này, giảm thiểu rủi ro bảo mật nếu ứng dụng bị chiếm quyền.
[C]
Tùy chọn `-g quickbite` để gán user vào nhóm tương ứng.
[EXP]
Tùy chọn này chỉ xác định primary group (nhóm chính) cho user mới tạo, không có vai trò bảo mật shell đăng nhập.
[D]
Tùy chọn `-m` để yêu cầu hệ thống không tạo thư mục home.
[EXP]
Tùy chọn `-m` dùng để tạo thư mục home cho user (thường dùng `-M` hoặc không truyền để tránh tạo home directory cho system account).


@correct: B
@point: 20

## Q2

Khi ứng dụng `user-service` không thể kết nối tới PostgreSQL ở IP `10.0.1.20`, lập trình viên cần thực hiện chẩn đoán lỗi mạng theo thứ tự các bước nào sau đây?
- (1) Sử dụng `curl -v 10.0.1.20:5432` hoặc `telnet` để kiểm tra cổng kết nối database có phản hồi hay không.
- (2) Sử dụng `ping 10.0.1.20` để kiểm tra kết nối vật lý giữa hai máy chủ có thông suốt hay không.

[A]
Thực hiện bước (1) trước, nếu thành công thì thực hiện bước (2) để kiểm tra tính toàn vẹn của dữ liệu.
[EXP]
Quy trình này không hợp lý vì ta cần kiểm tra kết nối vật lý cơ bản (ping) trước khi kiểm tra trạng thái cổng dịch vụ (curl).
[B]
Chỉ cần thực hiện bước (1) vì lệnh curl đã bao gồm khả năng kiểm tra kết nối vật lý của đường truyền mạng.
[EXP]
Mặc dù curl kiểm tra cổng, việc chạy ping trước vẫn là bước cơ bản để xác nhận máy chủ đích có online và phản hồi mạng hay không.
[C]
Thực hiện bước (2) để xác nhận kết nối vật lý, sau đó thực hiện bước (1) để kiểm tra trạng thái cổng dịch vụ.
[EXP]
Đây là quy trình chẩn đoán chuẩn: kiểm tra thông đường truyền vật lý (ping) -> nếu thông thì kiểm tra xem cổng dịch vụ đích (5432) có mở và phản hồi hay không.
[D]
Chỉ cần thực hiện bước (2) vì ping thành công đồng nghĩa với việc cổng kết nối database đã được mở sẵn.
[EXP]
Ping thành công chỉ chứng minh máy chủ đó đang chạy và thông mạng vật lý, không đảm bảo cổng 5432 của PostgreSQL có mở hay không.


@correct: C
@point: 20

## Q3

Khi cấu hình thư mục dự án trên server Linux bằng lệnh `sudo chmod -R 750 /opt/quickbite`, ý nghĩa phân quyền của con số `750` đối với thư mục này là gì?

[A]
Chủ sở hữu có toàn quyền (rwx); Nhóm sở hữu có quyền Đọc/Thực thi (r-x); Các tài khoản khác không có quyền gì (---).
[EXP]
Chính xác. Số 7 đại diện cho chủ sở hữu (rwx - Đọc, Ghi, Thực thi); số 5 đại diện cho group (r-x - Đọc, Thực thi); số 0 dành cho others (--- - không có quyền gì).
[B]
Chủ sở hữu có quyền Đọc/Ghi (rw-); Nhóm sở hữu có quyền Thực thi (r-x); Các tài khoản khác có quyền Đọc (r--).
[EXP]
Ý nghĩa phân quyền này tương đương với số `654`, không phải số `750`.
[C]
Chủ sở hữu và nhóm sở hữu có toàn quyền (rwx); Các tài khoản khác trên hệ thống chỉ có quyền Đọc và Thực thi (r-x).
[EXP]
Ý nghĩa phân quyền này tương đương với số `775`, không phải số `750`.
[D]
Chủ sở hữu có quyền Đọc/Thực thi (r-x); Nhóm sở hữu có toàn quyền (rwx); Các tài khoản khác không có quyền gì (---).
[EXP]
Ý nghĩa phân quyền này tương đương với số `570`, không phải số `750`.


@correct: A
@point: 20

## Q4

Tại sao việc chạy ứng dụng Java Spring Boot bằng tài khoản hệ thống chuyên dụng (ví dụ: `quickbite`) được coi là an toàn hơn nhiều so với việc chạy dưới quyền user `root`?

[A]
Tài khoản `quickbite` có khả năng tự động tối ưu hóa tài nguyên RAM JVM tốt hơn tài khoản `root`.
[EXP]
Việc tối ưu hóa RAM JVM phụ thuộc vào cấu hình của JVM và mã nguồn ứng dụng, không bị ảnh hưởng bởi quyền hạn của tài khoản chạy.
[B]
Tài khoản `quickbite` được tích hợp sẵn các công cụ chẩn đoán mạng nâng cao như `ss` và `ping`.
[EXP]
Các công cụ chẩn đoán mạng như `ss` và `ping` là của hệ thống, tài khoản root mới là tài khoản có toàn quyền chạy đầy đủ các tùy chọn của các công cụ này.
[C]
Nếu ứng dụng bị hacker tấn công, quyền hạn của hacker sẽ bị giới hạn trong phạm vi của user đó, tránh làm sập toàn bộ OS.
[EXP]
Đúng. Đây là nguyên tắc phân quyền tối thiểu (Least Privilege). Nếu ứng dụng bị hacker khai thác, hacker chỉ có quyền hạn hạn chế của user `quickbite`, không thể phá hủy hệ thống hoặc can thiệp vào các tiến trình khác như tài khoản `root`.
[D]
Tài khoản `quickbite` cho phép ứng dụng tự động bỏ qua các bài Unit Test bị lỗi để tiến hành deploy nhanh hơn.
[EXP]
Việc chạy hay bỏ qua Unit Test do cấu hình pipeline CI/CD quyết định, tài khoản chạy ứng dụng trên server không can thiệp vào việc này.


@correct: C
@point: 20

## Q5

Khi khởi chạy `user-service`, ứng dụng bị crash và ném ra lỗi `Port 8080 was already in use`. Bạn chạy lệnh `sudo ss -tulpn | grep :8080` để tìm kiếm tiến trình chiếm dụng. Chuyện gì xảy ra nếu bạn không sử dụng quyền `sudo` khi chạy lệnh `ss` này?

[A]
Lệnh `ss` vẫn hiển thị danh sách cổng mở nhưng không hiển thị thông tin PID và tên tiến trình đang chiếm dụng.
[EXP]
Chính xác. Nếu không có quyền quản trị (sudo), lệnh `ss` không được phép truy cập vào các file mô tả tiến trình của hệ điều hành để đọc thông tin PID và tên tiến trình (Process Name).
[B]
Lệnh `ss` sẽ bị hệ điều hành Linux chặn quyền truy cập và trả về lỗi `Permission denied` ngay lập tức.
[EXP]
Lệnh `ss` vẫn có thể chạy được dưới quyền user thường để xem danh sách cổng mạng mở của chính user đó mà không báo lỗi Permission denied.
[C]
Lệnh `ss` sẽ tự động chuyển hướng kết nối và hiển thị thông tin của các cổng mạng thuộc môi trường Staging.
[EXP]
Lệnh `ss` chỉ hiển thị thông tin cổng cục bộ của chính máy chủ đang chạy, không thể tự động chuyển hướng sang máy chủ Staging khác.
[D]
Lệnh `ss` sẽ tự động tắt tiến trình đang chiếm dụng cổng 8080 mà không cần bạn phải chạy lệnh `kill -9`.
[EXP]
Lệnh `ss` chỉ là một công cụ chẩn đoán thông tin mạng, không có khả năng tự kết liễu tiến trình đang hoạt động.


@correct: A
@point: 20
