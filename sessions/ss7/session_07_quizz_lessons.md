# QUIZ LESSONS - SESSION 07

# LESSON 01: Tổng quan GitLab CI/CD và Kiến trúc GitLab Runner

## Q1

Khi cài đặt Local Specific Runner bằng Docker Compose, tệp tin cấu hình nào sau đây được sử dụng để định nghĩa dịch vụ Runner?

[A]
.gitlab-ci.yml
[EXP]
Đây là tệp tin kịch bản CI/CD của dự án, không dùng để khởi chạy dịch vụ Runner.
[B]
docker-compose.yml
[EXP]
Chính xác. Dịch vụ Runner được khởi chạy bằng cách định nghĩa container trong tệp docker-compose.yml.
[C]
application.yml
[EXP]
Đây là tệp cấu hình của ứng dụng Spring Boot, không liên quan đến dịch vụ Runner.
[D]
Dockerfile
[EXP]
Đây là tệp định nghĩa cách build Docker image, không dùng để quản lý dịch vụ Runner.

@correct: B
@point: 20

## Q2

Khi lập trình viên thực hiện push code lên GitLab, quá trình nào sau đây diễn ra để Runner nhận công việc?

[A]
GitLab Server chủ động gửi lệnh SSH để kích hoạt Runner.
[EXP]
Server không kết nối SSH trực tiếp đến Runner.
[B]
Runner liên tục gửi yêu cầu HTTP (polling) lên GitLab Server để tự nhận các job mới.
[EXP]
Chính xác. Runner hoạt động theo cơ chế pull, chủ động gửi yêu cầu để lấy job từ Server về thực thi.
[C]
Lập trình viên phải mở cổng mạng của máy host để Server truyền job về.
[EXP]
Runner chủ động tạo kết nối ra ngoài nên không cần mở cổng mạng máy host.
[D]
Runner tự động đọc trực tiếp mã nguồn từ máy cá nhân thông qua Docker Network.
[EXP]
Runner clone mã nguồn từ GitLab Server, không đọc từ máy cá nhân.

@correct: B
@point: 20

## Q3

Trong lệnh đăng ký Runner sau:
```bash
docker compose exec gitlab-runner gitlab-runner register \
  --url "https://gitlab.com/" \
  --token "GLRT-12345" \
  --executor "docker" \
  --docker-image "alpine:latest"
```
Tham số `--executor "docker"` có ý nghĩa gì?

[A]
Chỉ định mỗi công việc (job) sẽ được thực thi bên trong một container Docker cô lập.
[EXP]
Chính xác. Executor docker sẽ tạo một container mới từ image cho trước để chạy các lệnh của job một cách độc lập.
[B]
Yêu cầu Runner chỉ được phép chạy trên hệ điều hành Windows.
[EXP]
Executor docker chạy các container Linux/Windows chứ không giới hạn hệ điều hành máy host.
[C]
Khởi chạy trực tiếp cơ sở dữ liệu PostgreSQL bên trong Runner.
[EXP]
Tham số này cấu hình môi trường chạy job, không liên quan đến khởi chạy PostgreSQL.
[D]
Tự động đẩy mã nguồn của dự án lên Docker Hub registry.
[EXP]
Đăng ký Runner không liên quan đến việc đẩy code lên Docker Hub.

@correct: A
@point: 20

## Q4

Điểm khác biệt cơ bản giữa Shared Runner và Specific Runner trong GitLab CI/CD là gì?

[A]
Shared Runner do GitLab cung cấp dùng chung; Specific Runner do người dùng tự cài đặt và đăng ký riêng cho dự án.
[EXP]
Chính xác. Shared Runner là tài nguyên dùng chung của GitLab; Specific Runner là Runner tự host riêng cho dự án cụ thể.
[B]
Shared Runner chỉ chạy được mã nguồn Java, còn Specific Runner chạy được mọi ngôn ngữ.
[EXP]
Cả hai loại Runner đều chạy được mọi ngôn ngữ tùy thuộc vào Executor và image cấu hình.
[C]
Specific Runner bắt buộc phải chạy bằng Shell Executor, còn Shared Runner chạy bằng Docker Executor.
[EXP]
Cả hai loại Runner đều hỗ trợ cấu hình bất kỳ Executor nào (Docker, Shell...).
[D]
Shared Runner không cần kết nối internet để hoạt động, còn Specific Runner bắt buộc phải có.
[EXP]
Cả hai loại Runner đều cần kết nối internet để giao tiếp với GitLab Server.

@correct: A
@point: 20

## Q5

Khi viết tệp docker-compose.yml cho Local Runner, nếu bỏ qua phần mount volumes `- /var/run/docker.sock:/var/run/docker.sock`, chuyện gì sẽ xảy ra khi Runner chạy job dùng Docker Executor?

[A]
Runner vẫn chạy bình thường nhưng tốc độ tải image sẽ chậm hơn.
[EXP]
Thiếu socket sẽ khiến Runner hoàn toàn không chạy được job dùng Docker Executor do không thể điều khiển Docker Engine.
[B]
Runner không thể giao tiếp với Docker Daemon của máy host để tạo container phụ, dẫn đến các job bị lỗi.
[EXP]
Chính xác. Mount Docker socket là bắt buộc để Runner (chạy dạng container) có quyền ra lệnh cho Docker của host tạo container mới chạy job.
[C]
GitLab Server sẽ tự động từ chối đăng ký tài khoản Runner này.
[EXP]
Runner vẫn đăng ký thành công với Server, lỗi chỉ xảy ra khi chạy job.
[D]
File cấu hình đăng ký Runner trong thư mục /etc/gitlab-runner sẽ bị xóa sạch khi khởi động lại.
[EXP]
Việc mất cấu hình do thiếu volume lưu trữ cấu hình /etc/gitlab-runner, không liên quan đến Docker socket.

@correct: B
@point: 20

# LESSON 02: Cấu trúc và Cú pháp file .gitlab-ci.yml

## Q1

Trong file .gitlab-ci.yml, thuộc tính nào bắt buộc phải có trong mỗi job để định nghĩa các câu lệnh sẽ thực thi?

[A]
image
[EXP]
Image có thể kế thừa từ khai báo toàn cục, không bắt buộc khai báo riêng ở từng job.
[B]
variables
[EXP]
Variables là tùy chọn, không bắt buộc phải có trong khai báo job.
[C]
script
[EXP]
Chính xác. Script là thuộc tính bắt buộc của job, chứa danh sách các câu lệnh shell để chạy trong container.
[D]
tags
[EXP]
Tags là tùy chọn để định tuyến job, không bắt buộc về mặt cú pháp YAML của job.

@correct: C
@point: 20

## Q2

Trong một job cấu hình như sau, khối lệnh nào sẽ được thực thi đầu tiên khi job bắt đầu chạy?
```yaml
test_job:
  before_script:
    - echo "A"
  script:
    - echo "B"
  after_script:
    - echo "C"
```

[A]
Khối lệnh in ra chữ "B" trong script.
[EXP]
Script chính chỉ chạy sau khi before_script hoàn thành.
[B]
Khối lệnh in ra chữ "A" trong before_script.
[EXP]
Chính xác. before_script chứa các lệnh chuẩn bị được thực thi trước kịch bản chính script.
[C]
Khối lệnh in ra chữ "C" trong after_script.
[EXP]
after_script luôn chạy cuối cùng để dọn dẹp hoặc in log sau khi job kết thúc.
[D]
Cả ba khối lệnh "A", "B", "C" được chạy song song cùng một lúc.
[EXP]
Các khối lệnh được thực thi tuần tự: before_script -> script -> after_script.

@correct: B
@point: 20

## Q3

Cho file cấu hình .gitlab-ci.yml sau:
```yaml
image: eclipse-temurin:17-jdk-alpine
stages:
  - info

print_job:
  stage: info
  tags:
    - quickbite
  script:
    - echo "Branch is $CI_COMMIT_BRANCH"
```
Biến `$CI_COMMIT_BRANCH` được khai báo ở đâu để sử dụng?

[A]
Lập trình viên phải tự khai báo trong khối variables toàn cục.
[EXP]
Biến định sẵn không cần lập trình viên khai báo thủ công trong khối variables.
[B]
Được nạp tự động bởi GitLab CI/CD (Predefined Variable) để cung cấp thông tin nhánh Git.
[EXP]
Chính xác. Đây là biến môi trường có sẵn cung cấp tên nhánh hiện tại kích hoạt pipeline.
[C]
Được định nghĩa trong tệp cấu hình application.yml của Spring Boot.
[EXP]
application.yml là cấu hình runtime của Java ứng dụng, không cung cấp biến cho GitLab CI.
[D]
Được đọc từ tệp tin .gitignore của repository.
[EXP]
.gitignore chỉ khai báo các file cần bỏ qua, không chứa biến môi trường.

@correct: B
@point: 20

## Q4

Để đảm bảo job CI chỉ được phân phối và thực thi trên máy Runner cục bộ (Local Specific Runner) có tag quickbite thay vì bị các Shared Runner dùng chung khác giành quyền thực thi, ta sử dụng từ khóa nào trong job?

[A]
stage: quickbite
[EXP]
Stage dùng để phân chia giai đoạn chạy của pipeline, không dùng để chỉ định Runner.
[B]
image: quickbite
[EXP]
Image dùng để chỉ định Docker image làm môi trường chạy, không chỉ định Runner.
[C]
only: [quickbite]
[EXP]
Only dùng để giới hạn nhánh Git (như main, dev), không định tuyến theo nhãn Runner.
[D]
tags: [quickbite]
[EXP]
Chính xác. Khai báo tags giúp định hướng job về đúng máy Runner có nhãn tags tương ứng.

@correct: D
@point: 20

## Q5

Nếu lập trình viên vô tình sử dụng phím Tab để thụt lề các dòng cấu hình trong file .gitlab-ci.yml, kết quả nhận được trên giao diện GitLab khi push code lên là gì?

[A]
GitLab Server tự động chuyển đổi phím Tab thành khoảng trắng và chạy bình thường.
[EXP]
GitLab Server không tự ý chỉnh sửa hay chuyển đổi nội dung file cấu hình của người dùng.
[B]
Pipeline bị báo lỗi cú pháp (YAML syntax error) và từ chối khởi chạy.
[EXP]
Chính xác. Định dạng YAML nghiêm cấm sử dụng phím Tab để thụt lề, việc dùng Tab sẽ gây lỗi cú pháp tệp cấu hình.
[C]
Job vẫn chạy nhưng các biến môi trường sẽ bị rỗng.
[EXP]
Lỗi cú pháp YAML sẽ ngăn chặn toàn bộ pipeline khởi chạy ngay từ đầu, không có job nào được thực thi.
[D]
Hệ thống tự động chuyển job sang chạy bằng Shell Executor thay vì Docker Executor.
[EXP]
Lỗi cú pháp file cấu hình làm sập pipeline, không liên quan đến việc thay đổi Executor.

@correct: B
@point: 20

# LESSON 03: Cơ chế hoạt động của Pipeline và Phân tách Stages

## Q1

Theo cơ chế hoạt động của GitLab CI/CD, các job thuộc cùng một stage (giai đoạn) sẽ được thực thi như thế nào?

[A]
Luôn luôn chạy tuần tự từ trên xuống dưới theo thứ tự khai báo trong file cấu hình.
[EXP]
Các job trong cùng stage không chạy tuần tự mà chạy song song.
[B]
Chạy song song cùng một lúc nếu hệ thống có đủ tài nguyên Runner trống.
[EXP]
Chính xác. Các job cùng stage được phân phối thực thi đồng thời để tối ưu hóa thời gian chạy của pipeline.
[C]
Job nào có dung lượng mã nguồn nhỏ hơn sẽ được chạy trước.
[EXP]
Dung lượng file không ảnh hưởng đến thứ tự hay điều phối thực thi job của Runner.
[D]
Chạy tuần tự và chia sẻ chung một ổ đĩa tạm thời trong suốt quá trình chạy.
[EXP]
Mỗi job chạy song song trên một container cô lập hoàn toàn, không dùng chung ổ đĩa trực tiếp.

@correct: B
@point: 20

## Q2

Cho danh sách các giai đoạn sau:
```yaml
stages:
  - compile
  - test
```
Nếu job thuộc stage compile bị lỗi (Failed), stage test tiếp theo sẽ diễn ra như thế nào?

[A]
Vẫn tiếp tục chạy bình thường để kiểm tra lỗi logic.
[EXP]
Stage test sẽ bị dừng lại để tránh lãng phí tài nguyên khi code không compile được.
[B]
Bị hủy bỏ (skipped) và toàn bộ pipeline báo trạng thái thất bại.
[EXP]
Chính xác. Quy trình chạy stage là tuần tự; bất kỳ stage nào bị lỗi sẽ làm dừng toàn bộ các stage chạy sau.
[C]
Chờ lập trình viên nhấn nút xác nhận trên giao diện Web UI rồi mới chạy tiếp.
[EXP]
Pipeline tự động dừng hoàn toàn khi có lỗi nghiêm trọng ở các stage trước mà không chờ xác nhận.
[D]
Tự động chạy lại stage compile thêm 3 lần trước khi chạy stage test.
[EXP]
Hệ thống không tự động chạy lại trừ khi được định cấu hình bằng thuộc tính retry.

@correct: B
@point: 20

## Q3

Đọc cấu hình pipeline giả lập sau:
```yaml
stages:
  - info
  - print

job_1:
  stage: info
  script:
    - echo "A"

job_2:
  stage: print
  script:
    - echo "B"
```
Quy trình thực thi của job_1 và job_2 diễn ra như thế nào?

[A]
job_1 chạy và hoàn thành trước, sau đó job_2 mới bắt đầu chạy.
[EXP]
Chính xác. Vì stage info được khai báo trước stage print, nên job_1 thuộc stage info phải chạy xong thì job_2 ở stage print mới được kích hoạt.
[B]
job_2 chạy và hoàn thành trước, sau đó job_1 mới bắt đầu chạy.
[EXP]
Stage print nằm sau stage info nên job_2 không bao giờ chạy trước job_1.
[C]
Cả hai job job_1 và job_2 được kích hoạt chạy song song cùng lúc.
[EXP]
Hai job thuộc hai stage khác nhau phải chạy tuần tự theo thứ tự stage, không chạy song song.
[D]
Chỉ có job_1 chạy, còn job_2 bị hủy bỏ mặc định do khác stage.
[EXP]
job_2 vẫn chạy bình thường sau khi job_1 hoàn thành thành công.

@correct: A
@point: 20

## Q4

Cơ chế cô lập (Job Isolation) của Docker Executor bảo đảm điều gì giữa các job chạy trong pipeline?

[A]
Các job chia sẻ chung dung lượng RAM của máy host để tăng tốc độ chạy.
[EXP]
Các job chạy trên container độc lập, không dùng chung bộ nhớ trực tiếp để tránh xung đột.
[B]
Mỗi job được thực thi trong một container mới tinh, sạch sẽ và container này bị hủy hoàn toàn khi job kết thúc.
[EXP]
Chính xác. Mỗi job chạy trong container cô lập riêng, mọi file tạm phát sinh sẽ bị xóa sạch khi job hoàn thành để tránh để lại rác hệ thống.
[C]
Giúp các job có thể ghi đè và chỉnh sửa trực tiếp mã nguồn trên nhánh chính (main branch).
[EXP]
Runner chỉ clone mã nguồn về chạy thử nghiệm, không có quyền tự ý ghi đè mã nguồn trên Git repo gốc.
[D]
Chặn hoàn toàn kết nối internet của các job để bảo mật thông tin mã nguồn.
[EXP]
Job vẫn cần mạng để pull image và clone code cũng như gửi trả log về GitLab Server.

@correct: B
@point: 20

## Q5

Cho tệp cấu hình .gitlab-ci.yml sau:
```yaml
stages:
  - info
  - print

job_info:
  stage: info
  script:
    - echo "Hello" > message.txt

job_print:
  stage: print
  script:
    - cat message.txt
```
Kết quả khi job_print thực thi câu lệnh cat message.txt là gì?

[A]
In ra nội dung "Hello" do tệp tin được tự động chuyển giao giữa các job.
[EXP]
File tạm sinh ra ở container job trước không tự động chuyển giao sang container của job sau.
[B]
Báo lỗi không tìm thấy file (No such file or directory) và job bị thất bại.
[EXP]
Chính xác. Do tính cô lập, container của job_info bị hủy kéo theo file message.txt biến mất. job_print chạy trên container mới tinh nên không có file này.
[C]
Hiển thị dòng chữ rỗng nhưng job vẫn hoàn thành thành công.
[EXP]
Lệnh cat khi lỗi đường dẫn sẽ trả về mã exit code lỗi, khiến job bị thất bại (Failed).
[D]
GitLab Server tự động báo lỗi cú pháp YAML ngay khi push file cấu hình lên.
[EXP]
File cấu hình đúng cú pháp YAML nên không báo lỗi cú pháp, lỗi chỉ xảy ra khi chạy lệnh shell của job.

@correct: B
@point: 20

# LESSON 04: Build ứng dụng Spring Boot bằng Maven/Gradle trong CI

## Q1

Lệnh Gradle nào sau đây giúp đóng gói dự án Spring Boot thành file JAR thực thi nhanh nhất trong môi trường CI bằng cách bỏ qua các bài kiểm thử tự động?

[A]
./gradlew compileJava
[EXP]
Lệnh này chỉ compile code Java sang class chứ không tạo ra file JAR thực thi.
[B]
./gradlew bootJar -x test
[EXP]
Chính xác. bootJar giúp đóng gói file JAR thực thi và -x test loại bỏ bước chạy test giúp rút ngắn tối đa thời gian chạy.
[C]
./gradlew test
[EXP]
Lệnh này chỉ chạy unit test chứ không tạo ra file JAR đóng gói sản phẩm.
[D]
./gradlew build
[EXP]
Lệnh này chạy đầy đủ bao gồm cả kiểm thử, tốn nhiều thời gian chạy trên Runner.

@correct: B
@point: 20

## Q2

Từ khóa artifacts trong file .gitlab-ci.yml dùng để làm gì sau khi kết thúc job build file JAR?

[A]
Khai báo image JDK mặc định cho Runner tải về làm môi trường chạy.
[EXP]
Image JDK được khai báo bằng từ khóa image, không dùng artifacts.
[B]
Chỉ định các tệp tin kết quả (như file JAR) cần được lưu giữ lại trên GitLab Server để sử dụng cho các job sau hoặc tải xuống.
[EXP]
Chính xác. Do tính cô lập của container, artifacts giúp nén và giữ lại file JAR đích trước khi container chạy job build bị hủy.
[C]
Tự động khởi chạy ứng dụng Spring Boot lên môi trường Production.
[EXP]
Deploy ứng dụng lên production yêu cầu kịch bản deploy riêng, artifacts chỉ có vai trò lưu trữ tệp tin sản phẩm.
[D]
Dọn dẹp sạch sẽ toàn bộ thư mục build để giải phóng bộ nhớ cho máy host.
[EXP]
Artifacts làm nhiệm vụ lưu giữ file chứ không phải dọn dẹp giải phóng bộ nhớ máy host.

@correct: B
@point: 20

## Q3

Cho cấu hình job build sau:
```yaml
build_job:
  stage: build
  script:
    - chmod +x ./gradlew
    - ./gradlew bootJar -x test
```
Lệnh `chmod +x ./gradlew` có vai trò gì trước khi chạy lệnh build?

[A]
Tải Gradle Wrapper từ internet về máy Runner để sử dụng.
[EXP]
Gradle Wrapper đã được đính kèm sẵn trong mã nguồn dự án, không cần tải mới.
[B]
Cấu hình cổng mạng của container để Spring Boot có thể chạy ở port 8080.
[EXP]
Lệnh phân quyền file cục bộ không liên quan đến cấu hình port mạng của container.
[C]
Đăng ký tài khoản Runner mới vào máy chủ GitLab Server.
[EXP]
Việc đăng ký Runner được thực hiện qua CLI riêng, không dùng lệnh chmod của tệp tin dự án.
[D]
Cấp quyền thực thi cho file chạy ./gradlew trên môi trường Linux của Runner.
[EXP]
Chính xác. Lệnh này cấp quyền chạy cho wrapper để tránh lỗi Permission Denied trên môi trường Runner Linux.

@correct: D
@point: 20

## Q4

Khi viết cấu hình kết nối database trong application.yml của các microservices, cách viết nào dưới đây giúp việc biên dịch và đóng gói JAR trong CI/CD diễn ra trơn tru mà không yêu cầu phải có một container database PostgreSQL thực tế đang chạy tại thời điểm build?

[A]
url: jdbc:postgresql://localhost:5432/quickbite
[EXP]
Cách viết cố định host này vẫn gây lỗi nếu lúc khởi chạy context không kết nối được tới địa chỉ localhost.
[B]
url: jdbc:postgresql://${DB_HOST:localhost}:5432/quickbite
[EXP]
Chính xác. Cú pháp fallback `${VAR_NAME:default_value}` giúp lấy giá trị mặc định localhost khi biến môi trường hệ thống không được nạp.
[C]
url: jdbc:postgresql://postgres-db:5432/quickbite
[EXP]
Cách viết cố định host ảo của Docker Compose vẫn gây lỗi kết nối nếu container database không chạy lúc build.
[D]
Xóa bỏ hoàn toàn dòng cấu hình url của cơ sở dữ liệu datasource.
[EXP]
Xóa cấu hình url làm Spring Boot gặp lỗi ngay lập tức do thiếu thông số cấu hình DataSource bắt buộc.

@correct: B
@point: 20

## Q5

Dự án user-service sử dụng Java 17. Nếu lập trình viên cấu hình dòng image đầu tiên trong file .gitlab-ci.yml là `image: eclipse-temurin:21-jdk-alpine` và chạy build, kết quả thế nào?

[A]
Pipeline bị báo lỗi cú pháp YAML ngay khi push code và không chạy.
[EXP]
Cấu hình image đúng cú pháp YAML nên không báo lỗi cú pháp.
[B]
Job build vẫn chạy thành công bình thường và tạo ra file JAR Java 17.
[EXP]
Chính xác. JDK 21 có khả năng biên dịch và tương thích ngược hoàn toàn với mã nguồn Java 17, do đó quá trình build diễn ra thành công.
[C]
Trình biên dịch báo lỗi không hỗ trợ tương thích ngược và dừng job.
[EXP]
JDK phiên bản cao hơn (Java 21) luôn tương thích ngược với mã nguồn thấp hơn (Java 17).
[D]
Runner tự động dừng và yêu cầu cài đặt lại hệ điều hành của máy host.
[EXP]
Phiên bản JDK trong container hoàn toàn độc lập với hệ điều hành máy host và không yêu cầu cài đặt lại máy host.

@correct: B
@point: 20

# LESSON 05: Thực hành phân tích log và chẩn đoán lỗi build trên GitLab CI

## Q1

Khi pipeline CI/CD bị báo lỗi đỏ (Failed), quy trình nào sau đây giúp định vị nguyên nhân lỗi nhanh nhất trên giao diện GitLab?

[A]
Thay đổi ngẫu nhiên code Java ở máy local rồi push lên liên tục.
[EXP]
Việc sửa code ngẫu nhiên không có căn cứ và gây lãng phí thời gian chạy lại pipeline.
[B]
Vào mục Build -> Jobs, click mở Job bị lỗi, cuộn xuống dưới cùng của Console Log để kiểm tra exit code và câu lệnh gây lỗi.
[EXP]
Chính xác. Log cuối cùng chỉ ra chính xác câu lệnh và mã lỗi trả về giúp khoanh vùng nguyên nhân nhanh nhất.
[C]
Khởi động lại Docker Desktop của máy cá nhân để xóa sạch log cũ.
[EXP]
Khởi động lại Docker local không ảnh hưởng đến log đã lưu trữ trên GitLab Server.
[D]
Đổi tên tệp tin .gitlab-ci.yml thành tên khác để kích hoạt lại pipeline.
[EXP]
Đổi tên tệp cấu hình làm GitLab Server không nhận diện được kịch bản CI/CD và không chạy pipeline nữa.

@correct: B
@point: 20

## Q2

Trong kịch bản thực hành lỗi phân quyền (Permission Denied) do thiếu lệnh chmod +x ./gradlew, dòng lỗi nào sau đây sẽ xuất hiện trong log của job build?

[A]
Task :compileJava FAILED
[EXP]
Lỗi này xuất hiện khi biên dịch mã nguồn Java thất bại, không phải lỗi phân quyền chạy file Wrapper.
[B]
./gradlew: Permission denied
[EXP]
Chính xác. Log báo lỗi trực tiếp do hệ điều hành Linux từ chối quyền chạy tệp tin ./gradlew.
[C]
AssertionError: expected:<200> but was:<500>
[EXP]
Đây là lỗi assert của unit test thất bại, không phải lỗi quyền chạy tệp tin.
[D]
Cannot connect to the Docker daemon
[EXP]
Lỗi này do thiếu socket Docker hoặc service dind khi build image, không phải lỗi chạy wrapper.

@correct: B
@point: 20

## Q3

Quan sát đoạn log lỗi compiler sau của dự án user-service:
```text
> Task :compileJava FAILED
/builds/user-service/src/main/java/com/quickbite/user/service/UserService.java:24: error: cannot find symbol
        private UserReposotory userRepository;
                ^
```
Nguyên nhân gây ra lỗi sập build trên là gì?

[A]
Viết sai chính tả tên class UserRepository thành UserReposotory ở dòng 24 trong file UserService.java.
[EXP]
Chính xác. Log báo rõ lỗi cú pháp không tìm thấy class UserReposotory tại dòng 24 của file UserService.java.
[B]
Thiếu lệnh cấp quyền chmod +x cho Gradle Wrapper.
[EXP]
Thiếu lệnh cấp quyền sẽ báo lỗi Permission denied chứ không chạy đến bước compileJava.
[C]
Phiên bản JDK của Runner không tương thích với dự án.
[EXP]
Lỗi class version mismatch sẽ hiển thị mã phiên bản class không tương thích, không báo lỗi cannot find symbol.
[D]
Chưa bật container database PostgreSQL chạy song song.
[EXP]
Spring Boot build JAR không đòi hỏi database thật hoạt động nhờ cấu hình fallback trong application.yml.

@correct: A
@point: 20

## Q4

Sự khác biệt rõ rệt nhất về log hiển thị giữa lỗi biên dịch mã nguồn (Compilation Failed) và lỗi kiểm thử thất bại (Test FAILED) là gì?

[A]
Compilation Failed báo lỗi ở tệp cấu hình YAML; Test FAILED báo lỗi ở file Dockerfile.
[EXP]
Cả hai lỗi đều nằm ở tầng ứng dụng Java, không liên quan đến tệp cấu hình YAML hay Dockerfile.
[B]
Compilation Failed báo lỗi cú pháp từ trình biên dịch (Task :compileJava FAILED); Test FAILED báo lỗi logic khi thực hiện các phép so sánh kiểm thử (Task :test FAILED đi kèm chi tiết các assert bị lệch).
[EXP]
Chính xác. Compilation Failed chặn đứng việc biên dịch code; Test FAILED xảy ra sau khi biên dịch xong và chạy testcase không khớp kết quả mong đợi.
[C]
Compilation Failed chỉ xuất hiện trên Windows; Test FAILED chỉ xuất hiện trên Linux.
[EXP]
Cả hai lỗi đều xuất hiện trên mọi hệ điều hành có chạy trình biên dịch Java và Junit.
[D]
Compilation Failed tự động được Runner sửa lỗi; Test FAILED yêu cầu lập trình viên can thiệp.
[EXP]
Runner không tự động sửa bất kỳ lỗi mã nguồn nào của lập trình viên.

@correct: B
@point: 20

## Q5

Lập trình viên sửa lỗi unit test cho dự án ở local và chạy ./gradlew test thành công. Tuy nhiên, khi push code lên GitLab, pipeline vẫn báo lỗi đỏ ở bước chạy test. Nguyên nhân phổ biến nhất gây ra hiện tượng này là gì?

[A]
Lập trình viên đã sửa code ở local nhưng quên commit và push các thay đổi đó lên Git repository.
[EXP]
Chính xác. Đây là sơ suất phổ biến: code chạy tốt ở local nhưng chưa được đẩy lên Git khiến Runner vẫn lấy bản code cũ bị lỗi để chạy test.
[B]
Runner tự động lưu cache kết quả chạy test thất bại của lần trước để áp đặt cho lần này.
[EXP]
Runner chạy trên container mới tinh và không lưu trữ hay áp đặt kết quả test của các lượt chạy trước.
[C]
Hệ điều hành Linux của Runner tự động chặn các lớp kiểm thử JUnit chạy trên môi trường Docker.
[EXP]
JUnit chạy hoàn toàn bình thường trong container Docker của Runner.
[D]
Môi trường CI bắt buộc phải chạy test tuần tự từng file và không cho phép chạy song song.
[EXP]
Gradle chạy test tuần tự hay song song tùy thuộc cấu hình dự án, không tự động dừng job vì lý do này.

@correct: A
@point: 20
