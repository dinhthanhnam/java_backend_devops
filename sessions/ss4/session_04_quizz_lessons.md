# QUIZ LESSONS - SESSION 04

# LESSON 01: Dockerfile và cách đóng gói ứng dụng Spring Boot thực tế

## Q1

Vai trò chính của tệp tin `Dockerfile` trong quy trình đóng gói và triển khai ứng dụng là gì?

[A]
Tự động biên dịch mã nguồn Java từ dạng text sang file bytecode `.class` mà không cần Maven hoặc Gradle.
[EXP]
Dockerfile không thay thế các công cụ biên dịch mã nguồn như Maven hay Gradle, ứng dụng vẫn cần được build ra file JAR trước khi Dockerfile thực hiện đóng gói.
[B]
Làm "nguồn sự thật duy nhất" (Source of Truth) chứa đặc tả môi trường chạy của ứng dụng một cách nhất quán.
[EXP]
Dockerfile đóng vai trò là tài liệu đặc tả chuẩn hóa duy nhất mô tả môi trường chạy, giúp bất kỳ máy chủ hay thành viên nào cũng có thể đóng gói ra Docker Image hoạt động nhất quán, chính xác.
[C]
Theo dõi hiệu năng tiêu thụ CPU và RAM của ứng dụng trong thời gian thực.
[EXP]
Dockerfile là bản thiết kế tĩnh dùng để đóng gói image, không có chức năng theo dõi hiệu năng của container khi đang chạy.
[D]
Ánh xạ trực tiếp các cổng mạng nội bộ của container ra ngoài máy host vật lý khi container khởi chạy.
[EXP]
Dockerfile chỉ khai báo cổng giao tiếp nội bộ qua chỉ thị EXPOSE chứ không tự động ánh xạ cổng ra ngoài máy host vật lý; việc ánh xạ cổng do cờ `-p` của lệnh `docker run` hoặc cấu hình `ports` trong Docker Compose đảm nhận.


@correct: B
@point: 20

## Q2

Tại sao trong môi trường Production, việc sử dụng base image `eclipse-temurin:17-jre-alpine` lại được khuyến nghị hơn so với các image chứa JDK đầy đủ?

[A]
Vì JRE Alpine chứa sẵn các driver kết nối hiệu năng cao tới cơ sở dữ liệu PostgreSQL.
[EXP]
Base image Java không tích hợp sẵn driver PostgreSQL; driver này phải được khai báo trong dependency của ứng dụng Spring Boot.
[B]
Vì JDK không hỗ trợ khởi chạy các file ứng dụng đã được đóng gói dưới dạng file JAR.
[EXP]
JDK hoàn toàn hỗ trợ chạy file JAR (vì JDK bao gồm cả JRE), nhưng sử dụng JDK trên Production gây lãng phí dung lượng và tăng rủi ro bảo mật.
[C]
Vì JRE Alpine tự động tối ưu hóa cấu hình bộ nhớ Heap Size của máy ảo Java (JVM) dựa trên tài nguyên của máy host.
[EXP]
JVM không tự động tối ưu hóa cấu hình Heap Size dựa trên hệ điều hành Alpine; việc cấu hình Heap Size vẫn yêu cầu các tham số JVM cụ thể khi khởi chạy.
[D]
Vì JRE Alpine giúp tối ưu hóa dung lượng ổ đĩa và tăng tính bảo mật bằng cách loại bỏ các công cụ phát triển không cần thiết, giảm bề mặt tấn công.
[EXP]
Đúng. Môi trường Production chỉ cần chạy ứng dụng nên sử dụng JRE (Java Runtime Environment) là đủ. Phiên bản Alpine siêu nhẹ giúp giảm dung lượng image xuống tối thiểu và loại bỏ các công cụ biên dịch/gỡ lỗi của JDK nhằm hạn chế lỗ hổng bảo mật.


@correct: D
@point: 20

## Q3

Sự khác biệt cốt lõi về cơ chế xử lý tín hiệu tắt máy (Graceful Shutdown) giữa việc viết chỉ thị chạy dạng Exec Form (`["java", "-jar", "app.jar"]`) và Shell Form (`java -jar app.jar`) là gì?

[A]
Exec Form chỉ chạy được trên hệ điều hành Linux; Shell Form bắt buộc phải chạy trên Windows.
[EXP]
Cả hai định dạng đều chạy được trên cả Linux và Windows, không bị giới hạn bởi hệ điều hành máy host.
[B]
Shell Form tự động lưu trữ trạng thái bộ nhớ Heap của ứng dụng trước khi tắt; Exec Form thì không.
[EXP]
Cả hai định dạng đều không tự động sao lưu bộ nhớ Heap khi tắt trừ khi có cấu hình tham số đặc biệt của JVM.
[C]
Exec Form cho phép ứng dụng chạy trực tiếp dưới tiến trình PID 1, nhận trực tiếp tín hiệu `SIGTERM` để đóng kết nối an toàn; Shell Form chạy qua tiến trình con của shell nên không nhận được `SIGTERM` trực tiếp và bị buộc dừng bằng `SIGKILL` sau 10 giây.
[EXP]
Chính xác. Exec Form (dạng mảng) khởi chạy ứng dụng trực tiếp dưới PID 1, do đó nhận được tín hiệu `SIGTERM` của Docker để thực hiện tắt máy an toàn. Shell Form khởi chạy qua trình shell trung gian `/bin/sh -c`, khiến tiến trình shell chiếm PID 1 và không chuyển tiếp tín hiệu `SIGTERM` đến Java, dẫn đến ứng dụng bị tắt đột ngột bằng `SIGKILL`.
[D]
Shell Form giải phóng RAM nhanh hơn Exec Form do tiến trình shell tự động dọn dẹp các luồng chạy ngầm của Java.
[EXP]
Shell Form không giải phóng RAM nhanh hơn mà ngược lại, việc tắt đột ngột bằng SIGKILL có thể làm rò rỉ dữ liệu hoặc hỏng file chưa kịp lưu.


@correct: C
@point: 20

## Q4

Chỉ thị `EXPOSE 8081` trong một Dockerfile đóng vai trò gì khi container khởi chạy?

[A]
Chỉ mang tính chất tài liệu hóa và khai báo cổng giao tiếp nội bộ giữa các container trong mạng ảo; không tự động ánh xạ cổng này ra ngoài máy host.
[EXP]
Chính xác. Chỉ thị EXPOSE chỉ đóng vai trò tài liệu hóa để lập trình viên hoặc kỹ sư DevOps biết cổng dịch vụ lắng nghe bên trong container. Cổng này không tự động được mở ra ngoài máy host trừ khi có cấu hình tường minh bằng tham số `-p` hoặc cấu hình `ports`.
[B]
Cấu hình cho máy ảo Java (JVM) lắng nghe các kết nối HTTP trên cổng được khai báo.
[EXP]
Cổng lắng nghe của JVM được cấu hình thông qua mã nguồn hoặc file cấu hình `application.yml` của Spring Boot, không phụ thuộc vào chỉ thị EXPOSE của Dockerfile.
[C]
Bắt buộc Docker Engine phải chặn toàn bộ các cổng kết nối khác ngoài cổng `8081`.
[EXP]
Docker Engine không chặn các cổng khác; các dịch vụ bên trong container vẫn có thể lắng nghe ở bất kỳ cổng nào dù không khai báo EXPOSE.
[D]
Tự động ánh xạ cổng `8081` của container ra cổng ngẫu nhiên trên máy host vật lý.
[EXP]
EXPOSE không tự động ánh xạ cổng ra máy host; việc ánh xạ ngẫu nhiên chỉ xảy ra khi chạy lệnh với cờ `-P` (viết hoa).


@correct: A
@point: 20

## Q5

Khi viết Dockerfile cho ứng dụng Spring Boot, chỉ thị `WORKDIR /app` có ý nghĩa như thế nào đối với các chỉ thị tiếp theo như `COPY` hoặc `ENTRYPOINT`?

[A]
Định nghĩa thư mục trên máy host sẽ được mount vĩnh viễn với container.
[EXP]
Việc mount thư mục từ máy host được cấu hình thông qua volume chứ không phải thông qua chỉ thị WORKDIR.
[B]
Yêu cầu Docker sao chép toàn bộ mã nguồn từ thư mục `/app` trên máy host vào container.
[EXP]
WORKDIR không sao chép mã nguồn từ máy host; việc sao chép do chỉ thị COPY hoặc ADD đảm nhận.
[C]
Cấu hình đường dẫn lưu trữ các file cấu hình tạm của hệ điều hành container.
[EXP]
Hệ điều hành container tự quản lý các thư mục hệ thống (như `/tmp`), WORKDIR không làm thay đổi đường dẫn lưu file cấu hình tạm này.
[D]
Thiết lập `/app` làm thư mục làm việc hiện hành bên trong container; các chỉ thị sau sẽ thực thi với đường dẫn tương đối so với thư mục này.
[EXP]
Đúng.WORKDIR thiết lập thư mục làm việc hiện hành. Lệnh `COPY build/libs/user-service.jar app.jar` sẽ sao chép file vào `/app/app.jar` và `ENTRYPOINT ["java", "-jar", "app.jar"]` sẽ tìm file `app.jar` ngay tại thư mục `/app`.


@correct: D
@point: 20

---

# LESSON 02: Docker Compose và khái niệm hệ thống nhiều container

## Q1

Vấn đề chính nào dưới đây xảy ra khi quản lý thủ công một hệ thống microservices gồm nhiều container (như Database, User Service, Restaurant Service) bằng các lệnh `docker run` đơn lẻ?

[A]
Hệ điều hành host tự động khóa các cổng mạng nếu phát hiện nhiều hơn hai container đang chạy.
[EXP]
Hệ điều hành host không tự động khóa cổng mạng; số lượng container chạy và mở cổng chỉ bị giới hạn bởi tài nguyên phần cứng máy host.
[B]
Docker Engine tự động giới hạn băng thông mạng giữa các container nếu chúng không được chạy cùng một lệnh.
[EXP]
Docker Engine không giới hạn băng thông mạng giữa các container chạy riêng lẻ.
[C]
Hiện tượng biến động địa chỉ IP (IP Drift) của database khi khởi động lại khiến các backend mất kết nối và yêu cầu cấu hình lại thủ công.
[EXP]
Đúng. Khi tắt và bật lại container database bằng lệnh run thủ công, Docker Engine sẽ cấp phát một IP động mới (IP Drift). Do đó, cấu hình kết nối cứng IP ở backend sẽ bị lỗi, đòi hỏi lập trình viên phải tra cứu IP mới và khởi chạy lại các container backend.
[D]
Các container độc lập không thể chia sẻ chung tài nguyên CPU vật lý của máy host.
[EXP]
Tất cả các container trên cùng một host đều chia sẻ tài nguyên CPU vật lý của máy host thông qua cơ chế lập lịch của nhân hệ điều hành.


@correct: C
@point: 20

## Q2

Trong kiến trúc Microservices, lý do quan trọng nhất của việc chia nhỏ hệ thống thành các container độc lập thay vì chạy tất cả dịch vụ trong một container duy nhất là gì?

[A]
Để Docker Daemon có thể chia sẻ trực tiếp bộ nhớ RAM vật lý giữa các dịch vụ mà không cần mạng.
[EXP]
Các container hoạt động độc lập và cô lập về bộ nhớ; chúng giao tiếp với nhau qua giao thức mạng ảo chứ không chia sẻ trực tiếp RAM.
[B]
Giúp giảm tổng dung lượng lưu trữ của Docker Image xuống mức tối thiểu.
[EXP]
Việc chia nhỏ thành nhiều image riêng biệt thực tế có thể làm tăng tổng dung lượng lưu trữ trên đĩa, nhưng mang lại lợi ích lớn về kiến trúc và vận hành.
[C]
Bắt buộc các dịch vụ phải sử dụng chung một phiên bản Java Runtime Environment (JRE).
[EXP]
Ngược lại, chia nhỏ container cho phép mỗi dịch vụ sử dụng phiên bản JRE/JDK riêng biệt phù hợp với nhu cầu phát triển.
[D]
Đảm bảo tính đa dạng công nghệ, cô lập lỗi và cho phép mở rộng độc lập từng dịch vụ theo nhu cầu tải thực tế.
[EXP]
Chính xác. Mỗi dịch vụ có thể sử dụng các phiên bản công nghệ khác nhau (ví dụ: Java 17, Java 21, PostgreSQL). Việc chia nhỏ giúp cô lập lỗi (một dịch vụ crash không làm sập dịch vụ khác) và cho phép scale up riêng biệt dịch vụ chịu tải lớn mà không lãng phí tài nguyên.


@correct: D
@point: 20

## Q3

Docker Compose được định nghĩa là một công cụ quản lý cấu hình theo phong cách khai báo (Declarative). Điều này có nghĩa là gì?

[A]
Người dùng định nghĩa trạng thái mong muốn của hệ thống trong file YAML và Docker Compose tự động điều phối để đạt được trạng thái đó.
[EXP]
Đúng. Phong cách khai báo (Declarative) cho phép ta mô tả cấu trúc hệ thống (chạy những dịch vụ nào, cổng nào, volume nào...) trong file YAML. Docker Compose sẽ tự động thực hiện các bước hạ tầng cần thiết để đưa hệ thống về đúng trạng thái mong muốn đó.
[B]
Hệ thống chỉ cho phép khai báo các tham số tĩnh và không thể thay đổi khi ứng dụng đang chạy.
[EXP]
Hệ thống hoàn toàn cho phép thay đổi cấu hình runtime (thông qua cập nhật file cấu hình và chạy lại lệnh up) mà không bị bó buộc vào các cấu hình tĩnh ban đầu.
[C]
Người dùng phải viết các kịch bản dòng lệnh shell script tuần tự để chỉ đạo Docker Engine cách khởi chạy từng container.
[EXP]
Đây là phong cách mệnh lệnh (Imperative), đại diện bởi việc chạy chuỗi câu lệnh `docker run` thủ công, không phải phong cách khai báo của Compose.
[D]
Docker Compose tự động chuyển đổi mã nguồn Java Spring Boot thành các tệp cấu hình hệ thống Linux.
[EXP]
Docker Compose không can thiệp vào quá trình biên dịch mã nguồn hay cấu hình mã nguồn Java của Spring Boot.


@correct: A
@point: 20

## Q4

Quy trình 3 bước nền tảng khi làm việc với Docker Compose để khởi chạy một ứng dụng phức tạp là gì?

[A]
Biên dịch mã nguồn (Gradle) -> Đóng gói file JAR -> Khởi chạy trực tiếp bằng lệnh java -jar.
[EXP]
Đây là quy trình chạy ứng dụng Java truyền thống trên môi trường máy host, chưa áp dụng công nghệ container hóa và Docker Compose.
[B]
Đẩy image lên Docker Hub -> Tải image về máy host -> Khởi chạy container bằng lệnh docker run.
[EXP]
Quy trình này mô tả việc phân phối image thủ công, không phản ánh quy trình phát triển và vận hành tích hợp của Docker Compose.
[C]
Tạo volume -> Thiết lập mạng ảo -> Ánh xạ cổng kết nối thủ công bằng lệnh docker network connect.
[EXP]
Đây là các bước thủ công cấp độ thấp của Docker CLI, Docker Compose tự động hóa toàn bộ các bước này mà không cần lập trình viên can thiệp thủ công.
[D]
Đóng gói môi trường chạy (Dockerfile) -> Khai báo cấu trúc hệ thống (docker-compose.yml) -> Vận hành hệ thống (docker compose CLI).
[EXP]
Đây là quy trình chuẩn: Đầu tiên viết Dockerfile để định nghĩa cách đóng gói từng dịch vụ -> Tiếp theo viết file docker-compose.yml để mô tả cách các dịch vụ liên kết với nhau -> Cuối cùng chạy lệnh CLI (`docker compose up`) để vận hành.


@correct: D
@point: 20

## Q5

Nhận định nào sau đây là ĐÚNG về phạm vi sử dụng thực tế của Docker Compose?

[A]
Chỉ hỗ trợ quản lý các container chạy cơ sở dữ liệu như PostgreSQL hay MySQL, không hỗ trợ web app.
[EXP]
Docker Compose hỗ trợ chạy bất kỳ loại ứng dụng nào được đóng gói thành container (Web, API, DB, Cache...).
[B]
Là giải pháp tối ưu nhất để tự động co giãn (Auto Scaling) hệ thống trên hàng trăm máy chủ vật lý ở môi trường Production lớn.
[EXP]
Trên môi trường Production lớn chạy đa máy chủ (Multi-host), cần các công cụ điều phối container chuyên dụng như Kubernetes hoặc Docker Swarm thay vì Docker Compose.
[C]
Thích hợp cho môi trường local phát triển, môi trường kiểm thử (Staging) hoặc môi trường Production quy mô nhỏ chạy trên một máy chủ đơn lẻ (Single Host).
[EXP]
Đúng. Docker Compose được thiết kế để quản lý các nhóm container chạy trên cùng một máy chủ vật lý/ảo (Single Host). Nó cực kỳ tiện lợi cho môi trường phát triển local hoặc môi trường staging nhỏ.
[D]
Chỉ có thể hoạt động khi máy chủ host không cài đặt bất kỳ công cụ quản lý container nào khác như Kubernetes.
[EXP]
Docker Compose chạy độc lập thông qua Docker Engine, hoàn toàn có thể cùng tồn tại với các công cụ quản lý container khác trên cùng một máy chủ.


@correct: C
@point: 20

---

# LESSON 03: Cấu trúc file docker-compose.yml (services, image, build)

## Q1

Khi soạn thảo tệp tin cấu hình `docker-compose.yml`, quy tắc định dạng cú pháp nào bắt buộc phải tuân thủ để tránh lỗi cú pháp?

[A]
Tất cả các giá trị cấu hình dạng số bắt buộc phải đặt trong dấu ngoặc kép.
[EXP]
Các giá trị dạng số hoặc boolean có thể viết trực tiếp không cần dấu ngoặc kép (ngoại trừ cổng mạng như `"8080:8080"` để tránh YAML tự động convert sang hệ cơ số 60).
[B]
Phải khai báo từ khóa `version` ở cuối tệp tin sau khi đã định nghĩa xong các dịch vụ.
[EXP]
Từ khóa `version` thường được khai báo ở ngay dòng đầu tiên của tệp tin cấu hình để chỉ định phiên bản định dạng sử dụng.
[C]
Phải sử dụng thụt lề bằng khoảng trắng (Spaces) để phân cấp các khối cấu hình, tuyệt đối không được sử dụng phím Tab.
[EXP]
Chính xác. Định dạng YAML cực kỳ nghiêm ngặt về việc thụt lề để xác định mối quan hệ cha-con giữa các thuộc tính. Lập trình viên bắt buộc phải dùng dấu cách (thường là 2 hoặc 4 khoảng trắng) và tuyệt đối không được sử dụng phím Tab vì sẽ gây lỗi biên dịch tệp tin.
[D]
Tên của các dịch vụ trong khối `services` phải trùng khớp hoàn toàn với tên thư mục chứa mã nguồn.
[EXP]
Tên dịch vụ là do lập trình viên tự đặt tùy ý bên trong file Compose để định danh dịch vụ, không bắt buộc phải trùng tên thư mục mã nguồn.


@correct: C
@point: 20

## Q2

Trong cấu hình của một service thuộc tệp `docker-compose.yml`, việc khai báo thuộc tính `build` với các khóa `context` và `dockerfile` có ý nghĩa gì?

[A]
Cấu hình cho container tự động biên dịch lại mã nguồn Java mỗi khi có sự thay đổi file trên máy host.
[EXP]
Thuộc tính build chỉ chạy một lần khi dựng cụm hoặc khi có yêu cầu build lại, không tự động theo dõi và biên dịch lại code Java thời gian thực khi đang chạy.
[B]
Chỉ định thư mục chứa mã nguồn và tên tệp tin Dockerfile để Compose tự động build thành image cục bộ thay vì kéo image có sẵn từ registry.
[EXP]
Đúng. Khai báo `build` yêu cầu Compose tự đóng gói image từ mã nguồn tại local. Khóa `context` chỉ định đường dẫn đến thư mục chứa code và Dockerfile, còn `dockerfile` chỉ ra file Dockerfile tương ứng để thực hiện build.
[C]
Thiết lập đường dẫn mount tài nguyên tĩnh giữa máy host và container đang chạy.
[EXP]
Việc mount tài nguyên tĩnh do thuộc tính `volumes` đảm nhận, không phải thuộc tính `build`.
[D]
Yêu cầu Docker Engine tự động tải tệp Dockerfile từ kho lưu trữ Docker Hub về máy.
[EXP]
Dockerfile dùng để build cục bộ nằm trên máy host phát triển, không phải được tải về từ Docker Hub.


@correct: B
@point: 20

## Q3

Điều gì xảy ra nếu trong cấu hình của một service, lập trình viên khai báo cả hai từ khóa `build` và `image`?

[A]
Hệ thống tự động đẩy image sau khi build lên Docker Hub với nhãn được khai báo ở khóa `image`.
[EXP]
Lệnh build cục bộ của Compose không tự động push image lên Docker Hub; việc push image đòi hỏi chạy lệnh `docker push` hoặc cấu hình CI/CD riêng.
[B]
Compose sẽ bỏ qua thuộc tính `build` và tiến hành tải image từ Docker Hub về để chạy.
[EXP]
Compose ưu tiên build từ Dockerfile cục bộ khi thuộc tính `build` được khai báo, không bỏ qua để tải image từ xa.
[C]
Compose sẽ build image mới từ Dockerfile và tự động đặt tên (tag) cho image đó theo giá trị khai báo ở từ khóa `image`.
[EXP]
Đúng. Khi khai báo cả hai, Compose sẽ build image từ mã nguồn cục bộ theo cấu hình `build`, sau đó đặt tên cho image đầu ra theo nhãn được định nghĩa tại `image` thay vì tự sinh ra một tên mặc định.
[D]
Docker Engine sẽ báo lỗi cú pháp và chặn không cho phép khởi chạy hệ thống do xung đột khai báo.
[EXP]
Đây là cấu hình hoàn toàn hợp lệ và là best practice để đặt tên tường minh cho các image tự build cục bộ trong dự án.


@correct: C
@point: 20

## Q4

Khi mã nguồn Java local của dịch vụ thay đổi và file JAR mới đã được build trên host, lệnh nào dưới đây giúp Compose đóng gói lại image mới và khởi chạy lại container?

[A]
`docker compose start`
[EXP]
Lệnh `start` chỉ kích hoạt lại container đã bị dừng trước đó, không đóng gói hay tái tạo container từ code mới.
[B]
`docker compose up -d --force-recreate`
[EXP]
Cờ `--force-recreate` buộc tạo lại container từ image cũ sẵn có ở local, không thực hiện rebuild lại Dockerfile nếu không truyền cờ `--build`.
[C]
`docker compose restart`
[EXP]
Lệnh `restart` chỉ đơn thuần khởi động lại container hiện tại từ image cũ đang có, không chạy lại quá trình build nên ứng dụng vẫn sẽ chạy bản code cũ.
[D]
`docker compose up -d --build`
[EXP]
Chính xác. Cờ `--build` bắt buộc Docker Compose phải thực hiện lại quá trình build image từ Dockerfile dựa trên file JAR mới đã biên dịch, sau đó hủy container cũ và dựng container mới từ image vừa build này.


@correct: D
@point: 20

## Q5

Khai báo `depends_on` trong tệp `docker-compose.yml` có vai trò chính là gì?

[A]
Ánh xạ cổng kết nối mạng giữa các container phụ thuộc với nhau.
[EXP]
Việc kết nối mạng được cấu hình qua thuộc tính `networks`, `depends_on` chỉ kiểm soát thứ tự khởi chạy của tiến trình.
[B]
Tự động đồng bộ hóa biến môi trường từ container này sang container khác khi khởi động.
[EXP]
`depends_on` không chia sẻ hay đồng bộ hóa biến môi trường giữa các container.
[C]
Bắt buộc container backend phải đợi cho đến khi cơ sở dữ liệu PostgreSQL sẵn sàng nhận kết nối mới được khởi chạy.
[EXP]
Mặc định, `depends_on` chỉ đợi container phụ thuộc ở trạng thái "chạy" (running) chứ không đợi dịch vụ bên trong sẵn sàng nhận kết nối (healthy). Để đợi database sẵn sàng hoàn toàn, cần cấu hình thêm điều kiện healthcheck.
[D]
Thiết lập thứ tự khởi chạy giữa các container trong cụm (ví dụ: khởi chạy database trước khi khởi chạy backend).
[EXP]
Đúng. `depends_on` định nghĩa sự phụ thuộc về mặt thứ tự khởi chạy của các dịch vụ. Nếu backend phụ thuộc vào database, Compose sẽ khởi động container database trước rồi mới đến container backend.


@correct: D
@point: 20

---

# LESSON 04: Biến môi trường và cấu hình port

## Q1

Sự khác biệt mấu chốt giữa thuộc tính `ports` và `expose` trong cấu hình Docker Compose là gì?

[A]
`ports` yêu cầu quyền root để kích hoạt; `expose` có thể cấu hình bởi bất kỳ user nào.
[EXP]
Việc khởi chạy Docker Compose nói chung đều yêu cầu user có quyền tương tác với Docker Daemon, không có sự phân biệt quyền hạn riêng giữa ports và expose.
[B]
`ports` ánh xạ cổng từ container ra máy host để truy cập được từ bên ngoài; `expose` chỉ khai báo cổng giao tiếp nội bộ giữa các container trong mạng ảo, máy host không kết nối trực tiếp được.
[EXP]
Chính xác. `ports` thực hiện NAT cổng từ máy host vào container (ví dụ: `"8080:8080"` giúp truy cập qua `localhost:8080`). Còn `expose` chỉ mang tính chất khai báo cổng cho các container giao tiếp nội bộ với nhau trong cùng mạng bridge ảo, bên ngoài máy host không thể gọi trực tiếp được.
[C]
`ports` dùng cho các container chạy ứng dụng Java; `expose` dùng riêng cho các container chạy cơ sở dữ liệu.
[EXP]
Cả hai thuộc tính đều có thể áp dụng cho bất kỳ loại container nào (Java, Database, Nginx...) tùy thuộc vào nhu cầu mở cổng kết nối.
[D]
`expose` tự động mở cổng ra ngoài Internet; `ports` chỉ cho phép truy cập từ mạng nội bộ công ty.
[EXP]
Ngược lại, `ports` mới là thuộc tính giúp mở kết nối ra ngoài máy host (và internet nếu máy host có IP công cộng), còn `expose` giữ kết nối cô lập hoàn toàn bên trong mạng ảo Docker.


@correct: B
@point: 20

## Q2

Tại sao việc viết cứng (hardcode) các thông số nhạy cảm như mật khẩu database trực tiếp vào tệp `docker-compose.yml` lại bị coi là một hành vi không an toàn?

[A]
Khiến container không thể tự động khởi động lại khi gặp sự cố phần cứng.
[EXP]
Cơ chế tự khởi động lại được cấu hình qua thuộc tính `restart`, không bị ảnh hưởng bởi việc khai báo mật khẩu tĩnh hay động.
[B]
Gây rò rỉ thông tin nhạy cảm khi file này được commit lên Git (Hardcoded Credentials Leak) và vi phạm nguyên lý "Build một lần, chạy mọi nơi".
[EXP]
Đúng. Khi đẩy file chứa mật khẩu lên kho mã nguồn chung, bất kỳ ai có quyền truy cập Git đều có thể đọc được mật khẩu (lộ thông tin). Ngoài ra, viết cứng thông số làm mất đi tính linh hoạt khi triển khai ứng dụng trên các môi trường Dev, Staging, Production khác nhau.
[C]
Làm giảm hiệu năng đọc ghi dữ liệu của hệ quản trị cơ sở dữ liệu.
[EXP]
Cách khai báo mật khẩu là cấu hình đầu vào lúc khởi chạy, hoàn toàn không ảnh hưởng đến hiệu năng hoạt động cơ sở dữ liệu PostgreSQL.
[D]
Khiến Docker Daemon không thể mã hóa dữ liệu truyền tải giữa các container.
[EXP]
Việc mã hóa dữ liệu truyền tải trong mạng phụ thuộc vào giao thức bảo mật của ứng dụng (như SSL/TLS) chứ không liên quan đến cách khai báo mật khẩu trong file Compose.


@correct: B
@point: 20

## Q3

Cơ chế nội suy biến (Interpolation) trong Docker Compose hoạt động như thế nào khi gặp cú pháp `${DB_PASSWORD}` trong tệp `docker-compose.yml`?

[A]
Hệ thống tự động tạo một mật khẩu ngẫu nhiên có tính bảo mật cao để thay thế biến đó.
[EXP]
Hệ thống không tự động sinh mật khẩu ngẫu nhiên cho biến môi trường nếu không được cấu hình logic sinh chuỗi cụ thể trong mã nguồn.
[B]
Trình biên dịch Java sẽ nạp biến này từ file cấu hình `application.yml` của ứng dụng Spring Boot.
[EXP]
Nội suy biến diễn ra ở tầng Docker Compose trước khi khởi chạy container; sau khi container chạy, biến môi trường mới được truyền vào hệ điều hành của container để JVM đọc.
[C]
Docker Engine sẽ yêu cầu lập trình viên nhập mật khẩu trực tiếp từ bàn phím lúc chạy lệnh up.
[EXP]
Docker Compose không dừng lại để yêu cầu nhập liệu tương tác từ bàn phím khi thực thi lệnh up.
[D]
Compose sẽ tự động tìm kiếm tệp tin `.env` cùng cấp thư mục để đọc giá trị của biến `DB_PASSWORD` và điền vào tệp cấu hình.
[EXP]
Chính xác. Khi gặp cú pháp `${VARIABLE}`, Docker Compose mặc định tìm tệp `.env` nằm cùng thư mục chạy lệnh, lấy giá trị tương ứng để điền vào cấu hình YAML trước khi thực thi khởi chạy container.


@correct: D
@point: 20

## Q4

Theo quy chuẩn bảo mật DevOps, tệp tin `.env` chứa mật khẩu thật của môi trường local nên được quản lý như thế nào trên Git?

[A]
Đưa tệp `.env` vào tệp `.gitignore` để tránh đẩy lên Git, đồng thời tạo tệp `.env.example` chứa các biến mẫu để hướng dẫn cấu hình.
[EXP]
Chính xác. Tệp `.env` chứa thông tin bảo mật nhạy cảm của môi trường thực tế nên phải được đưa vào `.gitignore` để giữ bí mật cục bộ. File `.env.example` chỉ chứa tên biến và giá trị mẫu được đẩy lên Git làm tài liệu hướng dẫn cho các lập trình viên khác tự thiết lập.
[B]
Commit tệp `.env` bình thường nhưng mã hóa nội dung bằng các công cụ zip có mật khẩu.
[EXP]
Git không tự động giải nén và nạp cấu hình từ file zip mã hóa; việc này làm gián đoạn quy trình chạy tự động của hệ thống.
[C]
Chỉ commit tệp `.env` lên các nhánh (branch) phát triển cá nhân, không merge vào nhánh chính.
[EXP]
Đẩy file nhạy cảm lên bất kỳ nhánh nào trên kho lưu trữ công cộng/chung đều tạo ra lỗ hổng bảo mật nghiêm trọng.
[D]
Sử dụng thuộc tính ẩn của Git để ngăn không cho các thành viên khác trong dự án tải tệp tin này về.
[EXP]
Git không có tính năng ẩn file đối với một số thành viên trong cùng một kho lưu trữ chung.


@correct: A
@point: 20

## Q5

Lệnh nào dưới đây giúp kỹ sư DevOps kiểm tra tính chính xác của việc nội suy biến và hiển thị toàn bộ cấu hình thực tế sẽ chạy của Docker Compose?

[A]
`docker compose config`
[EXP]
Chính xác. Lệnh `docker compose config` phân tích tệp `docker-compose.yml`, thực hiện nạp các giá trị từ `.env` vào vị trí nội suy, kiểm tra lỗi cú pháp và in ra toàn bộ cấu hình cuối cùng sẽ được thực thi trên Terminal.
[B]
`docker compose verify`
[EXP]
Không tồn tại lệnh `docker compose verify` để kiểm tra cú pháp file cấu hình Compose.
[C]
`docker compose inspect`
[EXP]
Lệnh `inspect` là lệnh của Docker CLI dùng để xem thông tin chi tiết của một đối tượng cụ thể (như container, image, network) đang chạy, không dùng để kiểm tra cấu hình file Compose trước khi khởi chạy.
[D]
`docker compose check`
[EXP]
Không tồn tại lệnh `docker compose check` trong bộ công cụ CLI mặc định của Docker Compose.


@correct: A
@point: 20

---

# LESSON 05: Volume và network trong Docker Compose

## Q1

Ưu điểm lớn nhất của việc sử dụng Named Volume so với Bind Mount khi lưu trữ dữ liệu cho container cơ sở dữ liệu PostgreSQL trong Docker Compose là gì?

[A]
Named Volume hỗ trợ mã hóa dữ liệu tự động ở tầng phần cứng ổ đĩa.
[EXP]
Docker Volume không tự động tích hợp tính năng mã hóa dữ liệu ở tầng vật lý phần cứng của máy host.
[B]
Do Docker tự quản lý thư mục lưu trữ ẩn trên máy host, đảm bảo hiệu năng cao, độc lập với cấu trúc thư mục của host và tránh việc xóa nhầm từ host.
[EXP]
Chính xác. Named Volume được Docker quản lý tại một thư mục ẩn chuyên dụng trên máy host. Điều này giúp tăng hiệu năng I/O, độc lập cấu trúc OS của host (không cần quan tâm đường dẫn vật lý chạy trên Windows hay Linux), và cực kỳ an toàn vì người dùng thông thường khó can thiệp xóa nhầm file.
[C]
Named Volume cho phép chia sẻ dữ liệu trực tiếp với các máy chủ khác qua môi trường internet.
[EXP]
Named Volume mặc định chỉ lưu trữ cục bộ trên máy host chạy Docker Engine, không tự động đồng bộ qua internet.
[D]
Named Volume tự động dọn dẹp các bản ghi dữ liệu rác sau mỗi 24 giờ hoạt động.
[EXP]
Docker không có cơ chế phân tích cấu trúc dữ liệu bên trong database để dọn dẹp dữ liệu rác của ứng dụng.


@correct: B
@point: 20

## Q2

Trong Docker Compose, khi nào lập trình viên nên ưu tiên sử dụng Bind Mount thay vì Named Volume?

[A]
Khi cần liên kết trực tiếp một thư mục cấu hình hoặc tệp tin mã nguồn cụ thể từ máy host vào container để phục vụ việc chỉnh sửa trực tiếp.
[EXP]
Đúng. Bind Mount ánh xạ một đường dẫn cụ thể trên máy host vào container. Điều này rất hữu ích khi cần mount mã nguồn (để hot-reload khi dev) hoặc mount các file cấu hình ngoại vi cần chỉnh sửa liên tục từ bên ngoài.
[B]
Khi lưu trữ cơ sở dữ liệu Production yêu cầu hiệu năng đọc ghi đĩa ở mức tối đa.
[EXP]
Đối với cơ sở dữ liệu Production, Named Volume là lựa chọn tối ưu hơn nhờ hiệu năng tốt và độ an toàn quản lý cao.
[C]
Khi muốn Docker Engine tự động sao lưu dữ liệu lên các dịch vụ lưu trữ đám mây.
[EXP]
Docker Engine không tự động tích hợp cơ chế sao lưu dữ liệu lên đám mây cho cả hai hình thức lưu trữ này.
[D]
Khi cần bảo mật tuyệt đối dữ liệu, không cho phép hệ điều hành host truy cập.
[EXP]
Cả hai cơ chế đều nằm trên ổ đĩa của máy host, hệ điều hành host (đặc biệt là quyền root) hoàn toàn truy cập được dữ liệu này.


@correct: A
@point: 20

## Q3

Cơ chế Service Discovery (Phát hiện dịch vụ) hoạt động như thế nào trong mạng ảo (Network) được khởi tạo bởi Docker Compose?

[A]
Các container tự động gửi tín hiệu broadcast để tìm kiếm địa chỉ MAC vật lý của nhau.
[EXP]
Hệ thống mạng Docker sử dụng DNS nội bộ điều phối bởi Docker Daemon để phân giải IP, không dựa trên quét broadcast địa chỉ MAC vật lý.
[B]
DNS nội bộ của mạng tự động phân giải tên dịch vụ (ví dụ: `quickbite-db`) thành địa chỉ IP động tương ứng của container đó để các dịch vụ khác kết nối.
[EXP]
Chính xác. Khi các container tham gia cùng một mạng ảo tùy biến của Compose, Docker sẽ kích hoạt một máy chủ DNS nội bộ. Nhờ đó, backend có thể kết nối tới database thông qua tên dịch vụ cấu hình trong file compose (ví dụ: `jdbc:postgresql://quickbite-db:5432/...`) mà không cần quan tâm IP động của database bị thay đổi.
[C]
Docker Daemon tự động quét và mở tất cả các cổng mạng của host để container kết nối ra ngoài.
[EXP]
Service Discovery là cơ chế phân giải tên miền nội bộ ảo, không liên quan đến việc mở cổng kết nối vật lý của máy host ra ngoài.
[D]
Trình duyệt của máy host tự động cập nhật file `/etc/hosts` để nhận dạng các container.
[EXP]
Docker không can thiệp hay tự động chỉnh sửa file cấu hình mạng `/etc/hosts` trên hệ điều hành của máy host.


@correct: B
@point: 20

## Q4

Điều gì xảy ra đối với dữ liệu lưu giữ trong Named Volume khi ta thực hiện câu lệnh dừng và dọn dẹp hệ thống `docker compose down`?

[A]
Dữ liệu tự động được nén thành file `.tar` và lưu vào thư mục `/tmp` của máy host.
[EXP]
Hệ thống không tự động nén hay sao lưu dữ liệu sang thư mục khác khi dọn dẹp container.
[B]
Dữ liệu vẫn được giữ an toàn 100% trên ổ đĩa máy host; chỉ bị xóa nếu lập trình viên truyền thêm cờ `-v` (`docker compose down -v`).
[EXP]
Đúng. Lệnh `docker compose down` chỉ hủy bỏ các container và mạng ảo của dự án. Dữ liệu trong Named Volume vẫn được giữ nguyên vẹn trên máy host. Dữ liệu chỉ bị xóa vĩnh viễn khi ta cố ý chạy kèm cờ `-v` (volumes).
[C]
Toàn bộ dữ liệu trong volume sẽ bị xóa sạch cùng với container để giải phóng không gian bộ nhớ.
[EXP]
Docker bảo vệ dữ liệu volume một cách an toàn; mặc định lệnh down không xóa Named Volume của bạn.
[D]
Dữ liệu bị khóa và chỉ có thể truy cập lại khi khởi chạy bằng lệnh `docker compose start`.
[EXP]
Dữ liệu không bị khóa; bạn hoàn toàn có thể khởi chạy một container mới tinh và mount vào volume cũ để tiếp tục sử dụng bình thường.


@correct: B
@point: 20

## Q5

Mạng ảo mặc định (Default Network) được tạo ra khi chạy lệnh `docker compose up` thuộc loại (driver) nào dưới đây?

[A]
Bridge (Mạng cầu ảo nội bộ giúp cô lập các container trên cùng một host).
[EXP]
Chính xác. Docker Compose mặc định khởi tạo một mạng ảo dạng `bridge` đặt tên theo thư mục dự án. Các container trong mạng này có thể tự do giao tiếp với nhau qua DNS nội bộ nhưng bị cô lập an toàn với các mạng khác và bên ngoài máy host.
[B]
Host (Container sử dụng chung hoàn toàn cấu hình mạng của máy host).
[EXP]
Driver Host loại bỏ sự cô lập mạng của container; Docker Compose mặc định không sử dụng driver này để đảm bảo tính an toàn.
[C]
Overlay (Mạng ảo kết nối nhiều máy chủ Docker Host vật lý khác nhau).
[EXP]
Driver Overlay chỉ được sử dụng khi chạy ở chế độ cluster đa máy chủ (như Docker Swarm), không phải driver mặc định của Compose đơn máy chủ.
[D]
None (Container hoàn toàn cô lập và không có cấu hình mạng mạng kết nối).
[EXP]
Driver None ngắt kết nối mạng của container; sử dụng driver này sẽ khiến các dịch vụ không thể giao tiếp được với nhau.


@correct: A
@point: 20

---

# LESSON 06: Quản lý vòng đời hệ thống với Docker Compose

## Q1

Sự khác biệt cốt lõi về mặt tài nguyên hệ thống giữa lệnh `docker compose stop` và `docker compose down` là gì?

[A]
`stop` chỉ dừng tiến trình, container vẫn tồn tại và giữ cấu hình trên đĩa; `down` dừng và xóa bỏ hoàn toàn container cùng mạng ảo của dự án khỏi hệ thống.
[EXP]
Chính xác. Lệnh `stop` gửi tín hiệu dừng tiến trình bên trong container nhưng giữ nguyên trạng thái container ở ổ đĩa (Writable Layer và cấu hình mạng vẫn tồn tại). Lệnh `down` dọn dẹp triệt để bằng cách xóa hẳn container, mạng ảo ra khỏi hệ thống, giải phóng hoàn toàn RAM và tài nguyên quản lý.
[B]
`stop` chỉ dừng các dịch vụ Java; `down` dừng toàn bộ các container cơ sở dữ liệu.
[EXP]
Cả hai lệnh đều tác động lên toàn bộ các dịch vụ được khai báo trong file cấu hình Compose của dự án, không phân biệt loại công nghệ.
[C]
`stop` giải phóng hoàn toàn bộ nhớ đĩa; `down` giữ lại các container ở trạng thái chờ kích hoạt.
[EXP]
Phát biểu bị ngược. `stop` giữ lại container trên đĩa, còn `down` xóa hoàn toàn container khỏi đĩa.
[D]
`down` bắt buộc phải xóa Named Volume; `stop` luôn giữ lại Named Volume.
[EXP]
Lệnh `down` mặc định không xóa Named Volume trừ khi có cờ `-v`; cả hai lệnh đều giữ an toàn cho dữ liệu volume của bạn.


@correct: A
@point: 20

## Q2

Tính năng ưu việt nhất của lệnh stream nhật ký `docker compose logs -f` khi vận hành một hệ thống gồm nhiều dịch vụ (Multi-service Stack) là gì?

[A]
Tự động tổng hợp log của tất cả các dịch vụ theo dòng thời gian, phân biệt màu sắc và tiền tố tên dịch vụ ở đầu dòng để dễ dàng chẩn đoán lỗi liên kết.
[EXP]
Chính xác. Điểm mạnh của logs trong Compose là khả năng stream log tập trung. Log của từng container được gộp chung theo trình tự thời gian thực, gán màu sắc riêng biệt và dán nhãn tên dịch vụ ở đầu dòng giúp lập trình viên phát hiện chuỗi sự kiện lỗi giữa các dịch vụ rất nhanh chóng.
[B]
Cho phép ghi đè cấu hình ghi log của ứng dụng Spring Boot sang dạng file JSON.
[EXP]
Docker Compose chỉ thu thập log đầu ra, việc định dạng log sang JSON thuộc về cấu hình của framework log (như Logback) bên trong mã nguồn Spring Boot.
[C]
Tự động gửi cảnh báo qua email cho quản trị viên khi phát hiện dòng log báo lỗi kết nối.
[EXP]
Docker Compose không hỗ trợ tính năng tự động gửi email cảnh báo; đây là nhiệm vụ của các hệ thống giám sát log chuyên nghiệp (như ELK, Grafana Loki).
[D]
Tự động lọc bỏ các log cảnh báo (Warning) và chỉ hiển thị các log lỗi nghiêm trọng (Critical).
[EXP]
Lệnh hiển thị toàn bộ luồng log ra tiêu chuẩn (STDOUT/STDERR), không tự động lọc bỏ các log warning trừ khi ta sử dụng thêm công cụ lọc pipe (như grep).


@correct: A
@point: 20

## Q3

Để kiểm tra nhanh trạng thái sẵn sàng kết nối của container database PostgreSQL đang chạy trong cụm mà không cần cài đặt tool client ở máy host, lệnh nào dưới đây là phù hợp nhất?

[A]
`docker compose exec quickbite-db pg_isready -U postgres`
[EXP]
Đúng. Lệnh này sử dụng công cụ `exec` để chạy trực tiếp lệnh kiểm tra tính sẵn sàng `pg_isready` có sẵn bên trong container database PostgreSQL, trả về kết quả trạng thái kết nối ngay tại Terminal của host.
[B]
`docker compose logs -f quickbite-db`
[EXP]
Logs chỉ hiển thị lịch sử in log ra màn hình, không thực thi kiểm tra chủ động trạng thái sẵn sàng kết nối hiện tại của PostgreSQL.
[C]
`docker compose run quickbite-db psql -h localhost`
[EXP]
Lệnh `run` sẽ cố gắng khởi tạo một container database PostgreSQL mới tinh thay vì chạy lệnh trên container database đang hoạt động, gây lãng phí tài nguyên và không kiểm tra đúng container mục tiêu.
[D]
`docker compose start quickbite-db`
[EXP]
Lệnh `start` dùng để khởi động lại container đã dừng, không có chức năng chạy lệnh kiểm tra kết nối bên trong container đang chạy.


@correct: A
@point: 20

## Q4

Khi lập trình viên gõ tổ hợp phím `Ctrl + C` để thoát khỏi màn hình stream log của lệnh `docker compose logs -f`, trạng thái của cụm container sẽ như thế nào?

[A]
Toàn bộ các container trong cụm sẽ chuyển sang trạng thái dừng (Stopped) ngay lập tức.
[EXP]
Việc dừng container đòi hỏi chạy lệnh `docker compose stop` hoặc chạy lệnh `up` ở chế độ foreground (không có cờ `-d`) rồi nhấn Ctrl+C.
[B]
Cụm container vẫn hoạt động bình thường dưới nền; việc thoát log chỉ dừng việc hiển thị trên Terminal.
[EXP]
Đúng. Nhấn `Ctrl + C` chỉ ngắt kết nối hiển thị log của công cụ CLI trên Terminal hiện tại, hoàn toàn không gửi tín hiệu dừng tới các container. Các dịch vụ vẫn tiếp tục chạy bình thường dưới nền.
[C]
Tiến trình Java Spring Boot sẽ thực hiện việc shutdown an toàn (Graceful Shutdown).
[EXP]
Ứng dụng Spring Boot vẫn hoạt động bình thường và không thực hiện quá trình shutdown.
[D]
Docker Engine sẽ tự động kích hoạt tiến trình xóa sạch cụm (Down).
[EXP]
Docker Engine không bao giờ tự động thực hiện lệnh down khi người dùng chỉ thoát xem log.


@correct: B
@point: 20

## Q5

Lệnh nào dưới đây giúp liệt kê danh sách các container thuộc dự án hiện hành kèm theo thông tin chi tiết về trạng thái hoạt động và cổng ánh xạ mạng thực tế?

[A]
`docker compose status`
[EXP]
Không tồn tại lệnh `docker compose status` để liệt kê danh sách container trong Docker Compose.
[B]
`docker compose ps`
[EXP]
Chính xác. Lệnh `docker compose ps` liệt kê tất cả các container thuộc stack hiện tại, chỉ ra ID, trạng thái chạy (Up/Exit), thời gian hoạt động và thông tin ánh xạ port của từng dịch vụ.
[C]
`docker compose show`
[EXP]
Không tồn tại lệnh `docker compose show` để hiển thị danh sách container trong Docker Compose.
[D]
`docker compose list`
[EXP]
Không tồn tại lệnh `docker compose list` trong bộ công cụ CLI mặc định của Docker Compose.


@correct: B
@point: 20
