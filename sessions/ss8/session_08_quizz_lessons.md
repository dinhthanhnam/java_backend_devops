# QUIZ LESSONS - SESSION 08

# LESSON 01: Quy trình Build Docker image trong pipeline CI/CD

## Q1

Khi thiết lập Local Specific Runner để chạy các lệnh Docker trực tiếp trên Docker Daemon của máy host (cơ chế DooD), cấu hình volumes của dịch vụ Runner cần khai báo mount đường dẫn nào sau đây?

[A]
"/var/run/docker.sock:/var/run/docker.sock"
[EXP]
Chính xác. Việc mount Docker socket vật lý của máy host vào container của job cho phép chuyển tiếp các lệnh Docker CLI từ container về daemon của host xử lý.
[B]
"/var/run/docker.sock:/etc/gitlab-runner"
[EXP]
Sai. Đường dẫn bên phải là thư mục lưu cấu hình của Runner, không phải socket giao tiếp.
[C]
"/etc/gitlab-runner:/var/run/docker.sock"
[EXP]
Sai. Đường dẫn bên trái là thư mục cấu hình của Runner ở máy host, không phải socket.
[D]
"/cache:/cache"
[EXP]
Sai. Volume này dùng để lưu bộ nhớ đệm (cache) của các job, không liên quan đến việc chia sẻ Docker socket.

@correct: A
@point: 20

## Q2

Trong pipeline 2 stage biên dịch và đóng gói image của bài học, tại sao job build image ở stage sau cần khai báo thuộc tính `dependencies: - build_jar_job`?

[A]
Để bắt buộc GitLab Runner phải chạy hai job này song song nhằm tiết kiệm thời gian.
[EXP]
Khai báo dependencies không làm các job chạy song song; các stage chạy tuần tự.
[B]
Để tự động kế thừa toàn bộ biến môi trường đã định nghĩa từ job biên dịch JAR trước đó.
[EXP]
Biến môi trường được quản lý toàn cục hoặc theo từng job, không kế thừa qua dependencies.
[C]
Để GitLab Runner tự động tải file JAR artifacts đã được lưu trữ ở stage trước về thư mục làm việc sạch sẽ của job hiện tại.
[EXP]
Chính xác. Mỗi job chạy trên một container độc lập và sạch sẽ. Nhờ thuộc tính dependencies, Runner sẽ tải file JAR artifacts từ stage trước về workspace của container hiện tại để phục vụ build image.
[D]
Để tự động khởi chạy dịch vụ Docker-in-Docker (DinD) phụ trợ cho tiến trình đóng gói.
[EXP]
Dependencies chỉ quản lý truyền tải artifacts, không kích hoạt dịch vụ dind.

@correct: C
@point: 20

## Q3

Cho đoạn cấu hình pipeline CI/CD sau:
```yaml
build_docker_image_job:
  stage: build_image
  image: docker:latest
  tags:
    - quickbite
  script:
    - docker build -t user-service:latest .
```
Tại sao job này không cần khai báo từ khóa `services: - docker:dind` nhưng vẫn có thể thực thi lệnh `docker build` thành công?

[A]
Vì Runner sử dụng cơ chế DooD (Docker-outside-of-Docker) chia sẻ trực tiếp Docker socket của máy host.
[EXP]
Chính xác. Các lệnh Docker CLI chạy trong container của job sẽ được chuyển tiếp qua socket để thực thi trực tiếp trên Docker daemon của máy host.
[B]
Vì image `docker:latest` đã được tích hợp sẵn một Docker daemon chạy ngầm bên trong.
[EXP]
Image docker:latest chỉ chứa Docker CLI (client), không tự chạy Docker daemon bên trong nếu không cấu hình dind.
[C]
Vì GitLab Server tự động cung cấp dịch vụ Docker daemon từ xa cho mọi Runner đăng ký.
[EXP]
GitLab Server không cung cấp daemon từ xa; việc thực thi phụ thuộc vào cấu hình Runner local.
[D]
Vì lệnh docker build được Runner tự động chuyển đổi thành tập lệnh shell chạy trực tiếp trên host.
[EXP]
Runner không chuyển đổi lệnh; lệnh vẫn chạy thông qua Docker CLI bên trong container của job.

@correct: A
@point: 20

## Q4

Khi so sánh hai mô hình chạy Docker trong CI/CD, ưu điểm vượt trội nhất của Docker-outside-of-Docker (DooD) chia sẻ Docker socket so với Docker-in-Docker (DinD) là gì?

[A]
Không yêu cầu máy host Runner phải kết nối mạng Internet khi chạy job đóng gói.
[EXP]
Cả hai mô hình đều cần kết nối mạng để tải base image nếu chưa có sẵn.
[B]
DooD cho phép bảo mật tuyệt đối thông tin đăng nhập Registry của lập trình viên.
[EXP]
Cơ chế bảo mật thông tin đăng nhập do cấu hình biến bảo mật của GitLab quản lý, không phụ thuộc vào DooD hay DinD.
[C]
Không đòi hỏi Runner phải chạy ở chế độ privileged (đặc quyền) và tối giản file cấu hình YAML.
[EXP]
Chính xác. DooD không cần privileged trên Runner (chỉ cần chia sẻ socket) và loại bỏ khai báo services phụ trợ, đồng thời tận dụng trực tiếp cache image có sẵn trên host để tăng tốc độ build.
[D]
Tự động dọn dẹp các image rác sinh ra sau khi kết thúc pipeline CI/CD.
[EXP]
DooD không tự động dọn dẹp image; các image build xong sẽ lưu trực tiếp trên Docker daemon của host.

@correct: C
@point: 20

## Q5

Học viên cấu hình một job kiểm tra trạng thái Docker sau khi build image như sau:
```yaml
build_docker_image_job:
  stage: build_image
  script:
    - docker build -t user-service:latest .
  after_script:
    - docker images | grep non-existent-image
```
Nếu tiến trình build ở `script` thành công, nhưng lệnh kiểm tra trong `after_script` thất bại (exit code khác 0), trạng thái cuối cùng của job trên GitLab CI là gì?

[A]
Failed (Thất bại) vì mọi lỗi phát sinh trong after_script đều được tính vào kết quả của job.
[EXP]
Lỗi trong after_script được GitLab Runner bỏ qua, không làm ảnh hưởng đến kết quả job.
[B]
Passed (Thành công) vì trạng thái của job chỉ được đánh giá dựa trên kết quả thực thi của khối script chính.
[EXP]
Chính xác. Khối after_script chạy trong một shell độc lập sau script chính; nếu script chính thành công thì job vẫn được báo Passed bất kể sau đó có lỗi ở after_script.
[C]
Canceled (Bị hủy) do hệ thống phát hiện xung đột trạng thái giữa script và after_script.
[EXP]
Không có trạng thái hủy tự động do xung đột này; job chỉ bị hủy bởi người dùng hoặc hệ thống gặp sự cố.
[D]
Warning (Cảnh báo) để nhắc nhở lập trình viên kiểm tra lại log của after_script.
[EXP]
GitLab CI không có trạng thái Warning riêng cho trường hợp lỗi after_script; job hiển thị Passed bình thường.

@correct: B
@point: 20

# LESSON 02: Tối ưu hóa Dockerfile cho Production (Multi-stage build)

## Q1

Trong một Dockerfile đa tầng (Multi-stage build) cấu hình cho Spring Boot, lệnh nào dưới đây dùng để sao chép file JAR đã biên dịch từ stage builder sang stage chạy runtime?

[A]
`COPY --from=builder /app/build/libs/*.jar app.jar`
[EXP]
Chính xác. Cờ --from=builder chỉ định Docker sao chép file từ không gian làm việc của stage được đặt tên là builder trước đó.
[B]
`COPY /app/build/libs/*.jar app.jar`
[EXP]
Sai. Lệnh này chỉ sao chép tệp từ build context ở máy local (host) vào container, không lấy từ stage builder.
[C]
`ADD --from=builder /app/build/libs/*.jar app.jar`
[EXP]
Sai. Cú pháp chuẩn của Multi-stage build sử dụng lệnh COPY, không dùng lệnh ADD cho thuộc tính --from.
[D]
`COPY --builder /app/build/libs/*.jar app.jar`
[EXP]
Sai. Cú pháp chính xác của cờ chỉ định nguồn stage là --from=<stage_name>, không phải --builder.

@correct: A
@point: 20

## Q2

Tại sao việc áp dụng kỹ thuật Multi-stage build trong Dockerfile lại cho phép chúng ta tinh giản tệp cấu hình `.gitlab-ci.yml` về mức tối giản nhất?

[A]
Vì Docker tự động tối ưu hóa tệp cấu hình YAML của GitLab khi phát hiện Dockerfile đa tầng.
[EXP]
Docker và GitLab CI là hai hệ thống độc lập; Docker không tự ý can thiệp vào cấu hình YAML của GitLab.
[B]
Vì GitLab CI/CD không hỗ trợ chạy nhiều job khi dự án có chứa Dockerfile đa tầng.
[EXP]
GitLab CI/CD vẫn hỗ trợ chạy nhiều job bình thường; việc rút gọn job là do quyết định thiết kế tối ưu hệ thống.
[C]
Vì toàn bộ luồng biên dịch mã nguồn (JDK) và đóng gói thành phẩm (JRE) đã được tích hợp khép kín bên trong Dockerfile.
[EXP]
Chính xác. Khi Dockerfile tự đảm nhận khâu biên dịch, pipeline không cần tạo job biên dịch riêng biệt và truyền tải artifacts nữa, chỉ cần chạy duy nhất lệnh docker build.
[D]
Vì Runner sẽ tự động biên dịch mã nguồn Java ra file JAR mà không cần gọi lệnh Gradle.
[EXP]
Runner vẫn phải gọi lệnh gradle biên dịch, nhưng lệnh này được chạy bên trong container builder của Dockerfile.

@correct: C
@point: 20

## Q3

Cho Dockerfile đa tầng của dịch vụ `user-service` như sau:
```dockerfile
FROM eclipse-temurin:17-jdk-alpine AS builder
WORKDIR /app
COPY . .
RUN chmod +x ./gradlew && ./gradlew bootJar --no-daemon

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /app/build/libs/*.jar app.jar
EXPOSE 8081
ENTRYPOINT ["java", "-jar", "app.jar"]
```
Khẳng định nào sau đây là chính xác về đặc điểm của Docker image được tạo thành?

[A]
Image cuối cùng chứa cả bộ JDK 17 đầy đủ và mã nguồn thô Java của ứng dụng.
[EXP]
Toàn bộ JDK và mã nguồn thô chỉ nằm ở stage 1 (builder) và bị loại bỏ ở stage runtime.
[B]
Tập tin Gradle Wrapper (`./gradlew`) bắt buộc phải được cài đặt toàn cục trên máy host Runner.
[EXP]
Gradle Wrapper chạy trực tiếp trong container builder nhờ file sao chép sang, không cần cài trên host.
[C]
Image cuối cùng chứa môi trường JRE siêu gọn nhẹ và tệp tin JAR thương phẩm, bỏ lại các công cụ biên dịch nặng nề.
[EXP]
Chính xác. Giúp tối thiểu hóa dung lượng image cuối cùng và tăng tính bảo mật (không lộ mã nguồn thô ở môi trường production).
[D]
Image được build ra chỉ chạy được trên các container có cấu hình Docker-in-Docker (DinD).
[EXP]
Image này là image ứng dụng tiêu chuẩn, chạy được trên bất kỳ môi trường container nào hỗ trợ Docker.

@correct: C
@point: 20

## Q4

Tại sao thời gian build image bằng Multi-stage build trên pipeline CI/CD của GitLab Runner lại kéo dài rất lâu (thường mất 4-5 phút) so với khi chạy tại máy local cá nhân của lập trình viên?

[A]
Vì GitLab Server giới hạn băng thông truyền tải mạng của mọi job chạy trên Runner.
[EXP]
Băng thông mạng của job phụ thuộc vào hạ tầng mạng của Runner, GitLab không bóp băng thông của Runner tự host.
[B]
Vì tiến trình chạy Gradle bên trong container builder tạm thời không truy cập được thư mục cache Gradle của máy host.
[EXP]
Chính xác. Lệnh RUN trong Dockerfile khởi chạy container builder độc lập và sạch sẽ, không có cache gradle dependencies nên bắt buộc phải tải lại toàn bộ dependencies từ Maven Central.
[C]
Vì GitLab Runner bắt buộc phải khởi tạo một container dind phụ trợ để biên dịch mã nguồn Java.
[EXP]
Giáo trình đang dùng DooD chia sẻ socket, không sử dụng container dind phụ trợ cho biên dịch.
[D]
Vì Dockerfile đa tầng bắt buộc phải tải lại base image JDK từ Docker Hub ở mỗi lượt chạy job.
[EXP]
Nhờ DooD chia sẻ socket, base image JDK sẽ được lưu cache trên host daemon và không cần tải lại nếu không thay đổi tag.

@correct: B
@point: 20

## Q5

Học viên viết Dockerfile đa tầng sử dụng câu lệnh `COPY . .` tại stage builder, nhưng lại quên không tạo tệp tin `.dockerignore` tại thư mục gốc của dự án. Khi thực hiện lệnh `docker build`, chuyện gì sẽ xảy ra?

[A]
Docker daemon sao chép toàn bộ tệp tin, bao gồm cả các thư mục build/ và .gradle/ local vào container, gây tăng dung lượng build context và rủi ro xung đột biên dịch.
[EXP]
Chính xác. Thiếu .dockerignore khiến toàn bộ dữ liệu tạm ở máy local bị copy vào container builder, làm chậm tiến trình gửi build context và có thể gây lỗi compile do ghi đè file.
[B]
Tiến trình build image sẽ bị trình biên dịch của Docker từ chối và báo lỗi cú pháp ngay lập tức.
[EXP]
Lệnh build vẫn thực thi bình thường, không báo lỗi cú pháp YAML hay Dockerfile do thiếu .dockerignore.
[C]
Docker sẽ tự động bỏ qua các thư mục tạm thời này nhờ cơ chế phát hiện tự động của Engine.
[EXP]
Docker Engine không tự động bỏ qua nếu không được khai báo rõ ràng trong tệp .dockerignore.
[D]
Dung lượng của image runtime cuối cùng sẽ tăng vọt lên hàng gigabytes do chứa các thư mục rác này.
[EXP]
Các thư mục rác này chỉ nằm ở stage builder và không bị copy sang stage runtime (do lệnh COPY --from=builder chỉ lấy file JAR cụ thể).

@correct: A
@point: 20

# LESSON 03: Phiên bản hóa và Đẩy Docker Image lên Registry từ Local

## Q1

Khi tạo Personal Access Token (PAT) trên GitLab để phục vụ cho các lệnh đăng nhập và đẩy Docker image từ máy local lên Container Registry, hai quyền (scopes) nào bắt buộc phải được tích chọn?

[A]
`api` và `read_user`
[EXP]
Các quyền này dùng để tương tác với API hệ thống và đọc thông tin user, không dùng cho Registry.
[B]
`read_repository` và `write_repository`
[EXP]
Các quyền này cấp quyền truy cập đọc/ghi mã nguồn Git, không liên quan đến Container Registry.
[C]
`write_registry` và `read_registry`
[EXP]
Chính xác. Đây là hai scope tối thiểu cho phép Docker client xác thực quyền ghi (push) và đọc (pull) image đối với kho lưu trữ.
[D]
`sudo` và `admin_mode`
[EXP]
Đây là các quyền quản trị cao cấp của hệ thống GitLab, không cấp cho mục đích đăng nhập Registry thông thường.

@correct: C
@point: 20

## Q2

Để đẩy thành công một Docker image được build ở máy local lên GitLab Container Registry của dự án `user-service` thuộc namespace `backend_fullskill_devops`, chuỗi các lệnh Docker CLI nào sau đây được thực hiện theo đúng trình tự?

[A]
`docker push ...` -> `docker tag ...` -> `docker login ...`
[EXP]
Trình tự này sai vì không thể push khi chưa login và chưa gắn tag Registry.
[B]
`docker login ...` -> `docker push ...` -> `docker tag ...`
[EXP]
Trình tự này sai vì lệnh push chạy trước khi tag image theo đúng đường dẫn Registry.
[C]
`docker tag ...` -> `docker login ...` -> `docker push ...`
[EXP]
Trình tự này có thể chạy được nhưng thông thường cần login thành công trước khi thực hiện đẩy image.
[D]
`docker login ...` -> `docker tag ...` -> `docker push ...`
[EXP]
Chính xác. Quy trình chuẩn là đăng nhập để xác thực -> gắn tag image nguồn thành đường dẫn Registry -> đẩy image lên kho lưu trữ.

@correct: D
@point: 20

## Q3

Một lập trình viên chạy câu lệnh sau trên terminal tại máy local:
```bash
docker tag user-service:1.0.0 registry.gitlab.com/backend_fullskill_devops/user-service:1.0.0
```
Mục đích thực tế của lệnh này là gì?

[A]
Đẩy trực tiếp các layer của image user-service:1.0.0 lên máy chủ registry.gitlab.com.
[EXP]
Lệnh docker tag chỉ hoạt động cục bộ, không gửi dữ liệu lên mạng; lệnh docker push mới đẩy image lên.
[B]
Tạo một bí danh (alias) trỏ đến image nguồn và định dạng lại tên theo đúng địa chỉ Registry để chuẩn bị push.
[EXP]
Chính xác. Lệnh tag tạo ra một nhãn mới trỏ đến cùng một Image ID có sẵn trong local Docker Engine để định tuyến khi push.
[C]
Biên dịch lại mã nguồn Java bên trong image theo cấu hình của registry.gitlab.com.
[EXP]
Lệnh docker tag không can thiệp vào cấu trúc bên trong hay biên dịch lại image.
[D]
Xóa bỏ hoàn toàn tag cũ user-service:1.0.0 khỏi Docker Engine của máy local.
[EXP]
Lệnh này tạo thêm tag mới chứ không xóa tag cũ; cả hai tag sẽ cùng tồn tại và trỏ chung một Image ID.

@correct: B
@point: 20

## Q4

Tại sao việc sử dụng Personal Access Token (PAT) lại được khuyến nghị và an toàn hơn so với việc sử dụng mật khẩu tài khoản GitLab chính để đăng nhập Docker Registry từ máy local?

[A]
Vì Personal Access Token có thể giới hạn phạm vi quyền truy cập và dễ dàng thu hồi mà không cần đổi mật khẩu chính.
[EXP]
Chính xác. PAT cho phép cấu hình giới hạn chỉ được đọc/ghi registry; nếu bị lộ, lập trình viên chỉ cần revoke token mà không ảnh hưởng đến mật khẩu tài khoản chính.
[B]
Vì Docker CLI sẽ tự động từ chối đăng nhập nếu lập trình viên nhập mật khẩu tài khoản chính.
[EXP]
Docker CLI vẫn chấp nhận mật khẩu chính nếu tài khoản không bật bảo mật 2 lớp (2FA), nhưng điều này không an toàn.
[C]
Vì sử dụng Personal Access Token giúp tăng tốc độ tải các layer của image lên Registry gấp hai lần.
[EXP]
Mã xác thực không ảnh hưởng đến băng thông mạng hay tốc độ truyền tải dữ liệu của lệnh push.
[D]
Vì Personal Access Token tự động mã hóa toàn bộ dữ liệu mã nguồn trong quá trình truyền tải.
[EXP]
Việc mã hóa dữ liệu truyền tải do giao thức HTTPS/TLS của Docker và Registry đảm nhận, không phụ thuộc vào loại token.

@correct: A
@point: 20

## Q5

Lập trình viên thực hiện build image thành công tại máy local và chạy lệnh `docker tag` đúng định dạng, sau đó thực thi ngay lệnh `docker push` mà quên chưa chạy lệnh `docker login`. Kết quả nhận được từ terminal là gì?

[A]
Lệnh push vẫn thành công bình thường vì GitLab Container Registry mặc định cho phép đẩy image tự do.
[EXP]
GitLab Container Registry là private, bắt buộc phải đăng nhập xác thực để ghi dữ liệu.
[B]
Docker CLI sẽ tự động dừng lại và hiển thị giao diện yêu cầu đăng ký tài khoản GitLab mới.
[EXP]
Docker CLI không hiển thị giao diện đăng ký tài khoản; nó chỉ trả về mã lỗi trên terminal.
[C]
Terminal báo lỗi từ chối truy cập `denied: requested access to the resource is denied` do chưa xác thực.
[EXP]
Chính xác. Khi chưa đăng nhập, Registry server sẽ phản hồi mã lỗi từ chối quyền truy cập (unauthorized) đối với yêu cầu đẩy image.
[D]
Hệ thống GitLab sẽ tự động khóa tài khoản cá nhân của lập trình viên do phát hiện truy cập trái phép.
[EXP]
Hệ thống chỉ từ chối yêu cầu đẩy image từ CLI, không tự động khóa tài khoản người dùng vì lỗi này.

@correct: C
@point: 20

# LESSON 04: Sử dụng Docker Image từ Registry trong Pipeline CI/CD

## Q1

Trong môi trường chạy job của GitLab Runner, biến môi trường ẩn định sẵn (Predefined Variable) nào lưu trữ token đăng nhập tạm thời chỉ có hiệu lực trong vòng đời của job để truy cập Registry?

[A]
`$CI_JOB_TOKEN`
[EXP]
Biến này dùng để xác thực các API chung của GitLab, không dùng cho lệnh đăng nhập chuẩn của Docker Registry của dự án.
[B]
`$CI_REGISTRY_PASSWORD`
[EXP]
Chính xác. Biến này chứa token xác thực tạm thời do GitLab tự động sinh ra cho lượt chạy pipeline để đăng nhập vào Registry của dự án.
[C]
`$CI_REGISTRY_USER`
[EXP]
Biến này lưu tên đăng nhập tạm thời (username), không phải mật khẩu/token xác thực.
[D]
`$CI_REGISTRY_IMAGE`
[EXP]
Biến này chứa đường dẫn thư mục lưu image của dự án trên Registry, không chứa thông tin xác thực.

@correct: B
@point: 20

## Q2

Khi Runner chạy job, tại sao giá trị của biến `$CI_REGISTRY_PASSWORD` lại hiển thị dưới dạng chuỗi đầu ra `[MASKED]` trên giao diện điều khiển Console Log của GitLab?

[A]
Do tệp cấu hình .gitlab-ci.yml quy định ẩn đi bằng các lệnh kiểm duyệt chuỗi.
[EXP]
Khai báo YAML không quy định việc che dấu log; đây là tính năng bảo mật mặc định của GitLab CI/CD Server.
[B]
Do GitLab CLI tự động phát hiện và mã hóa dữ liệu trước khi gửi về máy Runner.
[EXP]
Giá trị biến vẫn được gửi nguyên bản đến Runner để thực thi script, chỉ có phần log in ra màn hình được lọc.
[C]
Do Docker CLI tự động che dấu thông tin mật khẩu khi ghi nhận tham số truyền vào cờ -p.
[EXP]
Docker CLI không tự che dấu log đầu ra của GitLab; đây là hành vi lọc luồng log của GitLab Server.
[D]
Để bảo vệ an toàn thông tin, tránh lộ lọt token bảo mật nhạy cảm ra màn hình log công khai khi chạy job.
[EXP]
Chính xác. GitLab Server có cơ chế lọc (mask) tự động tất cả các giá trị của biến bảo mật xuất hiện trên log để ngăn ngừa lộ mật khẩu.

@correct: D
@point: 20

## Q3

Đối với dự án `user-service` nằm trong không gian làm việc `backend_fullskill_devops`, biến môi trường định sẵn `$CI_REGISTRY_IMAGE` trong pipeline CI/CD sẽ tự động phân giải thành giá trị nào?

[A]
`registry.gitlab.com/backend_fullskill_devops`
[EXP]
Đường dẫn này chỉ dừng lại ở tên group/namespace, thiếu tên dự án cụ thể.
[B]
`registry.gitlab.com/user-service`
[EXP]
Đường dẫn này thiếu phần namespace/group cha chứa dự án.
[C]
`registry.gitlab.com/backend_fullskill_devops/user-service`
[EXP]
Chính xác. Biến này phân giải đầy đủ gồm: domain registry (`registry.gitlab.com`) + namespace group (`backend_fullskill_devops`) + tên project (`user-service`).
[D]
`https://gitlab.com/backend_fullskill_devops/user-service/container_registry`
[EXP]
Đây là URL giao diện Web UI dùng trên trình duyệt, không phải địa chỉ Registry dùng cho Docker CLI.

@correct: C
@point: 20

## Q4

Tại sao việc chuyển đổi sang kịch bản kéo image từ Registry về chạy thử trong CI/CD lại giúp rút ngắn thời gian chạy của job xuống cực kỳ nhanh (chỉ khoảng 10-15 giây)?

[A]
Vì Runner không phải thực hiện các tác vụ biên dịch mã nguồn Java và tải dependencies của Gradle từ Maven Central.
[EXP]
Chính xác. Việc build image đã được thực hiện ở local trước đó; Runner chỉ chạy lệnh docker pull để tải các layer image đã đóng gói sẵn, giúp tiết kiệm thời gian biên dịch.
[B]
Vì GitLab Server tự động cache toàn bộ dung lượng của image trên RAM của máy host Runner.
[EXP]
GitLab không lưu cache image trên RAM; image được tải về ổ đĩa cứng của host Runner qua Docker engine.
[C]
Vì lệnh docker pull được thực thi trực tiếp trên hệ thống mạng nội bộ siêu tốc của GitLab Server.
[EXP]
Tiến trình kéo image chạy trên Runner của người dùng (tự host), không chạy trên hạ tầng mạng của GitLab Server.
[D]
Vì Runner tự động bỏ qua bước kiểm tra chữ ký số bảo mật của Docker image khi tải từ Registry.
[EXP]
Tốc độ nhanh là do loại bỏ khâu biên dịch và đóng gói, không liên quan đến việc bỏ qua xác thực bảo mật.

@correct: A
@point: 20

## Q5

Học viên cấu hình lệnh chạy container kiểm thử verify ứng dụng trong script của `.gitlab-ci.yml` như sau:
```yaml
test_job:
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker pull $CI_REGISTRY_IMAGE:1.0.0
    - docker run --name test_app -p 8081:8081 $CI_REGISTRY_IMAGE:1.0.0
    - echo "Verify success"
```
Khi job này được kích hoạt chạy trên GitLab CI, hiện tượng gì sẽ xảy ra?

[A]
Job hoàn thành thành công (Passed) ngay lập tức mà không gặp bất kỳ lỗi nào.
[EXP]
Job không thể hoàn thành Passed lập tức vì lệnh docker run sẽ chặn tiến trình chạy các lệnh tiếp theo.
[B]
Job CI/CD bị treo vô hạn thời gian cho đến khi gặp lỗi Timeout của hệ thống.
[EXP]
Chính xác. Do lệnh docker run thiếu cờ -d (detached mode) để chạy ngầm dưới nền, container sẽ chạy chiếm quyền điều khiển shell của Runner và chặn đứng các dòng lệnh tiếp theo.
[C]
Trình biên dịch GitLab CI sẽ báo lỗi cú pháp YAML và từ chối chạy pipeline ngay khi push file.
[EXP]
Cú pháp file YAML hoàn toàn hợp lệ, lỗi chỉ phát sinh ở runtime khi chạy lệnh docker run.
[D]
Runner tự động chuyển sang chế độ chạy ngầm và in dòng chữ "Verify success" ra log console sau 5 giây.
[EXP]
Runner không tự sửa lệnh để thêm cờ chạy ngầm; job sẽ bị chặn tại dòng lệnh docker run.

@correct: B
@point: 20

# LESSON 05: Kịch bản Thực hành Tổng hợp với user-service

## Q1

Trong kịch bản thực hành tổng hợp, lệnh khởi chạy container verify ứng dụng được viết như sau:
```bash
docker run -d --name verify_app -p 8081:8081 $CI_REGISTRY_IMAGE:1.0.0
```
Ý nghĩa của tham số `-d` và `-p 8081:8081` trong câu lệnh trên là gì?

[A]
Chạy container ở chế độ chạy ngầm dưới nền (detached) và ánh xạ cổng 8081 của container ra cổng 8081 của Runner.
[EXP]
Chính xác. Cờ -d giải phóng shell để Runner tiếp tục chạy các lệnh sau; cờ -p ánh xạ cổng mạng để phục vụ việc kiểm tra trạng thái ứng dụng.
[B]
Chạy container ở chế độ debug và mở cổng 8081 cho phép kết nối cơ sở dữ liệu PostgreSQL bên ngoài.
[EXP]
Tham số này cấu hình cho ứng dụng Spring Boot, không liên quan đến cổng của PostgreSQL (mặc định 5432).
[C]
Tự động dọn dẹp (delete) container khi dừng và gán quyền đặc quyền privileged trên cổng 8081.
[EXP]
Cờ dọn dẹp container khi dừng là --rm, không phải -d; cờ -p chỉ ánh xạ cổng thông thường.
[D]
Cấp quyền truy cập trực tiếp (direct) vào thư mục mã nguồn ở máy local qua cổng 8081.
[EXP]
Lệnh này chạy trên Runner CI/CD, không kết nối hay mount thư mục mã nguồn từ máy local của lập trình viên.

@correct: A
@point: 20

## Q2

Để cập nhật một tính năng mới trong code của ứng dụng và thực thi kiểm thử pipeline CI/CD verify phiên bản mới (ví dụ tag `1.0.1`), lập trình viên cần thực hiện chuỗi thao tác nào sau đây?

[A]
Git push code -> Cập nhật tag 1.0.1 trong .gitlab-ci.yml -> Build & Push image 1.0.1 từ local.
[EXP]
Trình tự này sai vì nếu push code trước, pipeline CI/CD sẽ sập ở bước kéo image do Registry chưa có tag 1.0.1.
[B]
Cập nhật tag 1.0.1 trong .gitlab-ci.yml -> Git push code -> Build & Push image 1.0.1 từ local.
[EXP]
Trình tự này sai vì pipeline vẫn chạy lỗi khi nhận commit push code do image chưa được đẩy lên Registry.
[C]
Build & Push image 1.0.1 từ local -> Cập nhật tag 1.0.1 trong .gitlab-ci.yml -> Git push code lên GitLab.
[EXP]
Chính xác. Phải đẩy image mới lên Registry trước để chuẩn bị sẵn tài nguyên, sau đó sửa tag cấu hình trong YAML rồi mới push code kích hoạt pipeline kéo image về chạy thử.
[D]
Chạy lệnh docker pull ở local -> Cập nhật .gitlab-ci.yml -> Đẩy trực tiếp mã nguồn JAR lên GitLab Server.
[EXP]
Lệnh docker pull ở local không tạo ra image mới; mã nguồn JAR không được đẩy trực tiếp lên GitLab Server mà truyền qua Git.

@correct: C
@point: 20

## Q3

Trong script của job verify ứng dụng Spring Boot, tại sao lệnh `sleep 5` lại được chèn vào ngay sau lệnh khởi chạy `docker run`?

[A]
Để cung cấp thời gian chờ cho Runner kết nối internet để đồng bộ cấu hình với GitLab Server.
[EXP]
Runner và GitLab Server giao tiếp độc lập, không liên quan đến tiến trình khởi chạy của container ứng dụng.
[B]
Để kiểm soát tốc độ ghi log của container, tránh tràn bộ nhớ đệm console log của Runner.
[EXP]
Lệnh sleep không dùng để lọc hay kiểm soát tốc độ ghi log của ứng dụng Spring Boot.
[C]
Để bắt buộc container phải dừng hoạt động tạm thời trước khi tiến hành dọn dẹp môi trường.
[EXP]
Lệnh sleep chỉ làm tạm ngưng shell script của job, không làm dừng container đang chạy ngầm dưới nền.
[D]
Để trì hoãn việc chạy lệnh tiếp theo, giúp ứng dụng Spring Boot có đủ thời gian hoàn tất quá trình khởi động.
[EXP]
Chính xác. Spring Boot cần vài giây để boot ứng dụng; lệnh sleep 5 đảm bảo khi chạy lệnh docker ps hoặc gọi API kiểm tra ở dòng sau thì ứng dụng đã sẵn sàng chạy.

@correct: D
@point: 20

## Q4

Tại sao việc dọn dẹp container (`docker rm -f verify_app`) ở cuối job thực hành tổng hợp lại được viết trực tiếp trong khối `script` chính thay vì đưa vào `after_script`?

[A]
Vì after_script không thể truy cập các biến môi trường như `$CI_REGISTRY_IMAGE` của job.
[EXP]
Biến môi trường định sẵn vẫn khả dụng trong after_script bình thường.
[B]
Để Runner có thể phát hiện và bắt lỗi chính xác nếu tiến trình dọn dẹp thất bại, bảo vệ tính toàn vẹn của job.
[EXP]
Chính xác. Lỗi phát sinh trong after_script sẽ bị Runner bỏ qua và job vẫn báo Passed. Viết vào script chính giúp Runner phát hiện ngay lỗi dọn dẹp để báo Failed cho job.
[C]
Vì after_script sẽ tự động khởi chạy lại container verify_app nếu phát hiện container bị dừng.
[EXP]
Khối after_script không tự động khởi chạy lại container; nó chỉ chạy các câu lệnh do lập trình viên khai báo.
[D]
Để tiết kiệm băng thông mạng truyền tải giữa máy host Runner và máy chủ GitLab Server.
[EXP]
Lệnh xóa container chạy cục bộ trên host Docker daemon, không tiêu tốn băng thông truyền tải mạng.

@correct: B
@point: 20

## Q5

Học viên cấu hình tệp tin `.gitlab-ci.yml` kéo image `1.0.0` về verify. Do sơ xuất, học viên push tệp tin cấu hình lên GitLab Server để kích hoạt pipeline trước khi thực hiện build và đẩy image `1.0.0` từ local lên Registry. Kết quả là gì?

[A]
Job verify thất bại ngay lập tức ở bước docker pull do kho lưu trữ Container Registry của dự án đang trống rỗng.
[EXP]
Chính xác. Khi Registry chưa có image và tag tương ứng, lệnh docker pull của Runner sẽ không thể tìm thấy manifest của image và trả về lỗi, khiến job thất bại.
[B]
Runner sẽ tự động biên dịch mã nguồn Java từ thư mục Git để tự đóng gói thành image thay thế.
[EXP]
Runner không tự động biên dịch thay thế nếu script của job chỉ khai báo lệnh docker pull and docker run.
[C]
Job vẫn báo Passed bình thường vì GitLab CI tự động bỏ qua lỗi kéo image nếu phát hiện Registry trống.
[EXP]
Lỗi docker pull là lỗi nghiêm trọng; Runner sẽ dừng thực thi lập tức và đánh dấu job hiển thị Failed.
[D]
Docker Engine của máy host Runner sẽ tự động tạo ra một image giả lập (mock image) để chạy test.
[EXP]
Docker Engine không tự sinh image giả lập; nó bắt buộc phải kéo được image thật hoặc báo lỗi sập.

@correct: A
@point: 20
