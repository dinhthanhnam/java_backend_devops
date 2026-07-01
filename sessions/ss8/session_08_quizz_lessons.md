# QUIZ LESSONS - SESSION 08

# LESSON 01: Quy trình Build Docker image trong luồng CI/CD

## Q1

Khi thiết lập Runner để chạy các lệnh Docker trực tiếp trên Docker Daemon của máy host (cơ chế DooD), cấu hình volumes của dịch vụ Runner cần khai báo mount đường dẫn nào sau đây?

[A]
"/var/run/docker.sock:/var/run/docker.sock"
[EXP]
Chính xác. Việc mount Docker socket vật lý của máy host vào container của job cho phép chuyển tiếp các lệnh Docker CLI từ container về daemon của host xử lý.
[B]
"/var/run/docker.sock:/etc/github-runner"
[EXP]
Sai. Đường dẫn bên phải không phải là socket giao tiếp.
[C]
"/etc/github-runner:/var/run/docker.sock"
[EXP]
Sai. Đường dẫn bên trái không phải là socket ở máy host.
[D]
"/cache:/cache"
[EXP]
Sai. Volume này dùng để lưu bộ nhớ đệm (cache), không liên quan đến việc chia sẻ Docker socket.

@correct: A
@point: 20

## Q2

Trong workflow 2 job biên dịch và đóng gói image của bài học, tại sao job build image ở sau cần khai báo thuộc tính `needs: [build_jar]`?

[A]
Để bắt buộc GitHub Actions Runner phải chạy hai job này song song nhằm tiết kiệm thời gian.
[EXP]
Khai báo needs không làm các job chạy song song; ngược lại nó thiết lập sự phụ thuộc tuần tự.
[B]
Để tự động kế thừa toàn bộ biến môi trường đã định nghĩa từ job biên dịch JAR trước đó.
[EXP]
Biến môi trường không kế thừa qua needs.
[C]
Để GitHub Actions đảm bảo job biên dịch JAR phải hoàn thành thành công trước khi bắt đầu job build image.
[EXP]
Chính xác. Khai báo needs đảm bảo thứ tự chạy của các job trong workflow. Cùng kết hợp với actions/download-artifact, Runner sẽ lấy file JAR từ job trước để phục vụ build image.
[D]
Để tự động khởi chạy dịch vụ Docker-in-Docker (DinD) phụ trợ cho tiến trình đóng gói.
[EXP]
Needs chỉ quản lý thứ tự thực thi, không kích hoạt dịch vụ dind.

@correct: C
@point: 20

## Q3

Cho đoạn cấu hình job trong GitHub Actions sau:
```yaml
build_image:
  runs-on: [self-hosted, quickbite]
  steps:
    - uses: actions/checkout@v5
    - run: docker build -t user-service:latest .
```
Tại sao job này không cần khởi tạo dịch vụ Docker daemon bên trong mà vẫn có thể thực thi lệnh `docker build` thành công?

[A]
Vì Runner sử dụng cơ chế DooD (Docker-outside-of-Docker) chia sẻ trực tiếp Docker socket của máy host.
[EXP]
Chính xác. Các lệnh Docker CLI chạy trong job sẽ được chuyển tiếp qua socket để thực thi trực tiếp trên Docker daemon của máy host.
[B]
Vì action checkout mặc định cài đặt Docker.
[EXP]
Action checkout chỉ kéo mã nguồn, không cài đặt Docker.
[C]
Vì GitHub tự động cung cấp dịch vụ Docker daemon từ xa cho mọi Runner tự host.
[EXP]
GitHub không cung cấp daemon từ xa cho self-hosted runner; việc thực thi phụ thuộc vào cấu hình máy host.
[D]
Vì lệnh docker build được Runner tự động chuyển đổi thành các lệnh shell cơ bản.
[EXP]
Runner không chuyển đổi lệnh; lệnh vẫn chạy thông qua Docker CLI có sẵn trên máy host.

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
Cơ chế bảo mật thông tin đăng nhập do cấu hình biến bảo mật quản lý, không phụ thuộc vào DooD hay DinD.
[C]
Không đòi hỏi cấu hình phức tạp với privileged (đặc quyền) và tận dụng trực tiếp cache image có sẵn trên host.
[EXP]
Chính xác. DooD không cần privileged trên Runner (chỉ cần chia sẻ socket) và tận dụng trực tiếp cache image có sẵn trên host để tăng tốc độ build.
[D]
Tự động dọn dẹp các image rác sinh ra sau khi kết thúc pipeline CI/CD.
[EXP]
DooD không tự động dọn dẹp image; các image build xong sẽ lưu trực tiếp trên Docker daemon của host.

@correct: C
@point: 20

## Q5

Trong GitHub Actions, điều gì sẽ xảy ra nếu một bước (step) dọn dẹp container bằng `docker rm` được khai báo sau một bước kiểm thử bị lỗi (fail)?

[A]
Bước dọn dẹp vẫn luôn chạy vì GitHub Actions mặc định chạy tất cả các step bất chấp lỗi.
[EXP]
Mặc định nếu một step lỗi, các step sau sẽ bị bỏ qua (skip).
[B]
Bước dọn dẹp sẽ bị bỏ qua, dẫn đến container kiểm thử vẫn đang chạy trên Runner.
[EXP]
Chính xác. Nếu không có cấu hình đặc biệt như `if: always()`, step bị fail sẽ khiến workflow dừng lại, làm step dọn dẹp không được thực thi.
[C]
Hệ thống sẽ tạm dừng và chờ lập trình viên vào dọn dẹp thủ công.
[EXP]
Không có tính năng tạm dừng chờ dọn dẹp; workflow sẽ kết thúc với trạng thái failed.
[D]
Container kiểm thử tự động bị dừng bởi hệ điều hành.
[EXP]
Hệ điều hành không tự biết để dừng container, container sẽ tiếp tục chạy ngầm.

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

Tại sao việc áp dụng kỹ thuật Multi-stage build trong Dockerfile lại cho phép chúng ta tinh giản tệp cấu hình `.github/workflows/ci.yml` về mức tối giản nhất?

[A]
Vì Docker tự động tối ưu hóa tệp cấu hình YAML khi phát hiện Dockerfile đa tầng.
[EXP]
Docker và GitHub Actions là hai hệ thống độc lập; Docker không tự ý can thiệp vào cấu hình YAML.
[B]
Vì GitHub Actions không hỗ trợ chạy nhiều job khi dự án có chứa Dockerfile đa tầng.
[EXP]
GitHub Actions vẫn hỗ trợ chạy nhiều job bình thường; việc rút gọn job là do quyết định thiết kế tối ưu hệ thống.
[C]
Vì toàn bộ luồng biên dịch mã nguồn (JDK) và đóng gói thành phẩm (JRE) đã được tích hợp khép kín bên trong Dockerfile.
[EXP]
Chính xác. Khi Dockerfile tự đảm nhận khâu biên dịch, workflow không cần tạo job biên dịch riêng biệt và tải/lên artifact nữa, chỉ cần chạy duy nhất lệnh docker build.
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
Image được build ra chỉ chạy được trên các môi trường cloud.
[EXP]
Image này là image ứng dụng tiêu chuẩn, chạy được trên bất kỳ môi trường container nào hỗ trợ Docker.

@correct: C
@point: 20

## Q4

Tại sao thời gian build image bằng Multi-stage build trên CI/CD lại kéo dài rất lâu (thường mất 4-5 phút) so với khi chạy tại máy local cá nhân của lập trình viên?

[A]
Vì GitHub giới hạn băng thông truyền tải mạng của mọi job chạy trên Runner.
[EXP]
Băng thông mạng của job phụ thuộc vào hạ tầng mạng của máy host tự chạy Runner, GitHub không bóp băng thông.
[B]
Vì tiến trình chạy Gradle bên trong container builder tạm thời không truy cập được thư mục cache Gradle của máy host.
[EXP]
Chính xác. Lệnh RUN trong Dockerfile khởi chạy container builder độc lập và sạch sẽ, không có cache gradle dependencies nên bắt buộc phải tải lại toàn bộ dependencies từ Maven Central.
[C]
Vì Runner bắt buộc phải khởi tạo một container phụ trợ để biên dịch mã nguồn Java.
[EXP]
Giáo trình đang dùng DooD chia sẻ socket, không sử dụng container phụ trợ độc lập cho biên dịch.
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
Lệnh build vẫn thực thi bình thường, không báo lỗi cú pháp do thiếu .dockerignore.
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

Khi tạo Personal Access Token (PAT) trên GitHub để phục vụ cho các lệnh đăng nhập và đẩy Docker image từ máy local lên Container Registry, quyền (scopes) nào bắt buộc phải được tích chọn?

[A]
`repo` và `workflow`
[EXP]
Các quyền này dùng để tương tác với Git repo và cấu hình workflow, không đủ để đẩy packages.
[B]
`read:org` và `write:org`
[EXP]
Các quyền này quản lý organization, không liên quan đến Container Registry.
[C]
`write:packages` và `read:packages`
[EXP]
Chính xác. Đây là hai scope thiết yếu cho phép Docker client xác thực quyền ghi (push) và đọc (pull) đối với kho packages (GHCR).
[D]
`admin:enterprise`
[EXP]
Đây là các quyền quản trị cao cấp, không cấp cho mục đích đăng nhập Registry thông thường.

@correct: C
@point: 20

## Q2

Để đẩy thành công một Docker image được build ở máy local lên GitHub Container Registry của tài khoản `nguyena` dự án `user-service`, chuỗi các lệnh Docker CLI nào sau đây được thực hiện theo đúng trình tự?

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
docker tag user-service:1.0.0 ghcr.io/nguyena/user-service:1.0.0
```
Mục đích thực tế của lệnh này là gì?

[A]
Đẩy trực tiếp các layer của image user-service:1.0.0 lên máy chủ ghcr.io.
[EXP]
Lệnh docker tag chỉ hoạt động cục bộ, không gửi dữ liệu lên mạng; lệnh docker push mới đẩy image lên.
[B]
Tạo một bí danh (alias) trỏ đến image nguồn và định dạng lại tên theo đúng địa chỉ Registry để chuẩn bị push.
[EXP]
Chính xác. Lệnh tag tạo ra một nhãn mới trỏ đến cùng một Image ID có sẵn trong local Docker Engine để định tuyến khi push.
[C]
Biên dịch lại mã nguồn Java bên trong image theo cấu hình của ghcr.io.
[EXP]
Lệnh docker tag không can thiệp vào cấu trúc bên trong hay biên dịch lại image.
[D]
Xóa bỏ hoàn toàn tag cũ user-service:1.0.0 khỏi Docker Engine của máy local.
[EXP]
Lệnh này tạo thêm tag mới chứ không xóa tag cũ; cả hai tag sẽ cùng tồn tại và trỏ chung một Image ID.

@correct: B
@point: 20

## Q4

Tại sao việc sử dụng Personal Access Token (PAT) lại được khuyến nghị và an toàn hơn so với việc sử dụng mật khẩu tài khoản GitHub chính để đăng nhập Docker Registry từ máy local?

[A]
Vì Personal Access Token có thể giới hạn phạm vi quyền truy cập (chỉ với packages) và dễ dàng thu hồi mà không cần đổi mật khẩu chính.
[EXP]
Chính xác. PAT cho phép cấu hình giới hạn quyền; nếu bị lộ, lập trình viên chỉ cần revoke token mà không ảnh hưởng đến toàn bộ tài khoản GitHub.
[B]
Vì Docker CLI sẽ tự động từ chối đăng nhập nếu lập trình viên nhập mật khẩu tài khoản chính.
[EXP]
GitHub tự chối xác thực bằng mật khẩu tài khoản chính qua command line, thay vào đó bắt buộc dùng PAT hoặc thiết bị xác thực, nhưng đó là quy định nền tảng chứ không phải tự động từ chối của bản thân client.
[C]
Vì sử dụng Personal Access Token giúp tăng tốc độ tải các layer của image lên Registry gấp hai lần.
[EXP]
Mã xác thực không ảnh hưởng đến tốc độ truyền tải dữ liệu của lệnh push.
[D]
Vì Personal Access Token tự động mã hóa dữ liệu image trong quá trình truyền tải.
[EXP]
Việc mã hóa dữ liệu do giao thức TLS của HTTPS đảm nhận, không phụ thuộc vào loại token.

@correct: A
@point: 20

## Q5

Lập trình viên thực hiện build image thành công tại máy local và chạy lệnh `docker tag` đúng định dạng, sau đó thực thi ngay lệnh `docker push` mà chưa đăng nhập GHCR thành công. Kết quả nhận được từ terminal là gì?

[A]
Lệnh push vẫn thành công bình thường vì GHCR mặc định cho phép đẩy image tự do.
[EXP]
GHCR yêu cầu quyền truy cập hợp lệ để ghi (push).
[B]
Docker CLI sẽ tự động dừng lại và hiển thị trình duyệt yêu cầu đăng nhập.
[EXP]
Docker CLI không hiển thị giao diện trình duyệt web; nó chỉ thao tác trên terminal.
[C]
Terminal báo lỗi từ chối truy cập `denied: unauthenticated` do chưa xác thực.
[EXP]
Chính xác. Khi chưa đăng nhập hoặc thiếu token, server sẽ phản hồi mã lỗi từ chối quyền truy cập đối với yêu cầu đẩy image.
[D]
Hệ thống GitHub sẽ tự động khóa tài khoản cá nhân của lập trình viên.
[EXP]
Hệ thống chỉ từ chối truy cập, không tự động khóa tài khoản người dùng.

@correct: C
@point: 20

# LESSON 04: Sử dụng Docker Image từ Registry trong Luồng CI/CD

## Q1

Trong GitHub Actions, biến nào lưu trữ token bảo mật tạm thời tự động sinh ra cho job để đăng nhập GHCR?

[A]
`${{ github.actor }}`
[EXP]
Biến này lưu tên đăng nhập người kích hoạt workflow, không phải token bảo mật.
[B]
`${{ secrets.GITHUB_TOKEN }}`
[EXP]
Chính xác. Đây là biến token tạm thời tự động do GitHub sinh ra dùng để xác thực trong workflow.
[C]
`${{ github.token }}`
[EXP]
Cách gọi này cũng có thể dùng nhưng biến chính thống cho secret thường ở block secrets.
[D]
`${{ secrets.PAT_TOKEN }}`
[EXP]
Đây là tên biến người dùng tự định nghĩa, không phải token tự động sinh ra.

@correct: B
@point: 20

## Q2

Để biến `${{ secrets.GITHUB_TOKEN }}` có quyền tải image (pull) từ GHCR về chạy, job trong GitHub Actions cần khai báo phân quyền nào?

[A]
`permissions: write-all`
[EXP]
Quyền này quá rộng và không được khuyến khích vì lý do bảo mật.
[B]
`permissions: packages: read`
[EXP]
Chính xác. Khai báo quyền packages read cho phép GITHUB_TOKEN tải image về mà vẫn tuân thủ nguyên tắc quyền tối thiểu.
[C]
`permissions: contents: write`
[EXP]
Quyền này cho phép ghi vào mã nguồn repo, không liên quan đến packages.
[D]
Không cần cấu hình, mặc định đã có quyền admin cao nhất.
[EXP]
Mặc định GITHUB_TOKEN bị giới hạn quyền rất chặt chẽ, phải khai báo cụ thể.

@correct: B
@point: 20

## Q3

Khi muốn kéo image từ GHCR với tên kho lưu trữ (`github.repository`) có chứa chữ cái hoa, tại sao lập trình viên cần sử dụng lệnh `tr '[:upper:]' '[:lower:]'`?

[A]
Để mã hóa tên kho lưu trữ chống tấn công.
[EXP]
Lệnh tr chỉ chuyển đổi viết hoa thành viết thường, không mã hóa.
[B]
Vì Docker image name không cho phép sử dụng ký tự viết hoa.
[EXP]
Chính xác. Docker engine yêu cầu tên tham chiếu image (repository) phải hoàn toàn là chữ viết thường. Biến github.repository giữ nguyên chữ hoa nếu kho GitHub có chữ hoa, nên phải hạ xuống chữ thường.
[C]
Để tương thích với cú pháp YAML của GitHub Actions.
[EXP]
YAML vẫn hỗ trợ chữ hoa bình thường.
[D]
Để tự động nhận diện và thay thế khoảng trắng thành dấu gạch ngang.
[EXP]
Lệnh `tr '[:upper:]' '[:lower:]'` không biến đổi khoảng trắng.

@correct: B
@point: 20

## Q4

Tại sao việc chuyển đổi sang kịch bản kéo image từ Registry về chạy thử trong CI/CD lại giúp rút ngắn thời gian chạy của job xuống cực kỳ nhanh (chỉ khoảng 10-15 giây)?

[A]
Vì Runner không phải thực hiện các tác vụ biên dịch mã nguồn Java và tải dependencies của Gradle từ Maven Central.
[EXP]
Chính xác. Việc build image đã được thực hiện ở local trước đó; Runner chỉ chạy lệnh docker pull để tải các layer image đã đóng gói sẵn, giúp tiết kiệm thời gian biên dịch.
[B]
Vì GitHub Server tự động cache toàn bộ dung lượng của image trên RAM của máy host Runner.
[EXP]
GitHub không lưu cache image trên RAM; image được tải về ổ đĩa cứng của host Runner qua Docker engine.
[C]
Vì lệnh docker pull được thực thi trực tiếp trên hệ thống mạng nội bộ siêu tốc của máy chủ GitHub.
[EXP]
Tiến trình kéo image chạy trên Runner của người dùng (tự host), tốc độ mạng do mạng cục bộ quyết định.
[D]
Vì Runner tự động bỏ qua bước kiểm tra chữ ký số bảo mật của Docker image.
[EXP]
Tốc độ nhanh là do loại bỏ khâu biên dịch và đóng gói, không liên quan đến việc bỏ qua xác thực bảo mật.

@correct: A
@point: 20

## Q5

Học viên cấu hình lệnh chạy container kiểm thử verify ứng dụng trong job như sau:
```yaml
    - run: |
        docker login ghcr.io -u ${{ github.actor }} -p ${{ secrets.GITHUB_TOKEN }}
        docker pull ghcr.io/nguyena/user-service:1.0.0
        docker run --name test_app -p 8081:8081 ghcr.io/nguyena/user-service:1.0.0
        echo "Verify success"
```
Khi job này được kích hoạt, hiện tượng gì sẽ xảy ra?

[A]
Job hoàn thành thành công (Passed) ngay lập tức mà không gặp bất kỳ lỗi nào.
[EXP]
Job không thể hoàn thành Passed lập tức vì lệnh docker run sẽ chặn tiến trình chạy các lệnh tiếp theo.
[B]
Job bị treo vô hạn thời gian cho đến khi bị hủy bởi hệ thống do timeout.
[EXP]
Chính xác. Do lệnh docker run thiếu cờ -d (detached mode) để chạy ngầm dưới nền, container sẽ chạy chiếm quyền điều khiển shell của Runner và chặn đứng các lệnh phía sau.
[C]
GitHub Actions sẽ báo lỗi cú pháp YAML và từ chối chạy ngay khi push file.
[EXP]
Cú pháp YAML hợp lệ, lỗi là do hành vi của lệnh.
[D]
Runner tự động thêm tham số -d và in ra Verify success.
[EXP]
Runner không thể tự động sửa đổi lệnh của lập trình viên.

@correct: B
@point: 20

# LESSON 05: Kịch bản Thực hành Tổng hợp với user-service

## Q1

Trong kịch bản thực hành tổng hợp, lệnh khởi chạy container verify ứng dụng được viết như sau:
```bash
docker run -d --name verify_app -p 8081:8081 $IMAGE_TAG
```
Ý nghĩa của tham số `-d` và `-p 8081:8081` trong câu lệnh trên là gì?

[A]
Chạy container ở chế độ chạy ngầm dưới nền (detached) và ánh xạ cổng 8081 của container ra cổng 8081 của Runner.
[EXP]
Chính xác. Cờ -d giải phóng shell để Runner tiếp tục chạy các lệnh sau; cờ -p ánh xạ cổng mạng để phục vụ việc kiểm tra trạng thái ứng dụng.
[B]
Chạy container ở chế độ debug và mở cổng 8081 cho phép kết nối bên ngoài.
[EXP]
Tham số này không liên quan đến chế độ debug riêng biệt của Spring Boot.
[C]
Tự động dọn dẹp (delete) container khi dừng và gán quyền đặc quyền privileged trên cổng 8081.
[EXP]
Cờ dọn dẹp container khi dừng là --rm, không phải -d.
[D]
Cấp quyền truy cập trực tiếp (direct) vào thư mục mã nguồn ở máy local qua cổng 8081.
[EXP]
Lệnh này không liên quan đến thư mục mã nguồn máy local.

@correct: A
@point: 20

## Q2

Để cập nhật một tính năng mới trong code của ứng dụng và thực thi kiểm thử pipeline CI/CD verify phiên bản mới (ví dụ tag `1.0.1`), lập trình viên cần thực hiện chuỗi thao tác nào sau đây?

[A]
Git push code -> Cập nhật tag 1.0.1 trong ci.yml -> Build & Push image 1.0.1 từ local.
[EXP]
Trình tự này sai vì nếu push code trước, workflow sẽ sập ở bước kéo image do Registry chưa có tag 1.0.1.
[B]
Cập nhật tag 1.0.1 trong ci.yml -> Git push code -> Build & Push image 1.0.1 từ local.
[EXP]
Trình tự này sai vì workflow vẫn chạy lỗi do image chưa được đẩy lên Registry.
[C]
Build & Push image 1.0.1 từ local -> Cập nhật tag 1.0.1 trong ci.yml -> Git push code lên GitHub.
[EXP]
Chính xác. Phải đẩy image mới lên Registry trước để chuẩn bị sẵn tài nguyên, sau đó sửa tag cấu hình trong YAML rồi mới push code kích hoạt workflow kéo image về chạy thử.
[D]
Chạy lệnh docker pull ở local -> Cập nhật ci.yml -> Đẩy trực tiếp mã nguồn JAR lên GitHub.
[EXP]
Mã nguồn JAR không được lưu trong Git và pull image không thay thế build image.

@correct: C
@point: 20

## Q3

Trong script của job verify ứng dụng Spring Boot, tại sao lệnh `sleep 5` lại được chèn vào ngay sau lệnh khởi chạy `docker run`?

[A]
Để cung cấp thời gian chờ cho Runner kết nối internet với GitHub.
[EXP]
Runner và GitHub giao tiếp độc lập với luồng lệnh ứng dụng.
[B]
Để kiểm soát tốc độ ghi log của container, tránh tràn bộ nhớ đệm.
[EXP]
Lệnh sleep không dùng để lọc hay kiểm soát log.
[C]
Để bắt buộc container phải dừng hoạt động tạm thời trước khi tiến hành dọn dẹp.
[EXP]
Lệnh sleep làm tạm ngưng shell script, container chạy detached vẫn tiếp tục hoạt động.
[D]
Để trì hoãn việc chạy lệnh tiếp theo, giúp ứng dụng Spring Boot có đủ thời gian hoàn tất quá trình khởi động trước khi gọi lệnh `docker ps`.
[EXP]
Chính xác. Spring Boot cần vài giây để boot ứng dụng; lệnh sleep 5 đảm bảo kiểm tra trạng thái chính xác nhất.

@correct: D
@point: 20

## Q4

Để đảm bảo dọn dẹp môi trường (xóa container `verify_app`) ngay cả khi một số lệnh kiểm tra trước đó bị lỗi trong GitHub Actions, ta nên làm gì?

[A]
Tách thành step mới và dùng lệnh bash cơ bản.
[EXP]
Step mới sẽ mặc định bị bỏ qua nếu step trước lỗi.
[B]
Sử dụng toán tử `|| true` ở cuối chuỗi kiểm tra hoặc cấu hình `if: always()` ở step dọn dẹp.
[EXP]
Chính xác. Điều này giúp hệ thống không hủy dọn dẹp do lỗi của bước kiểm thử.
[C]
Gửi thông báo email cho quản trị viên vào dọn dẹp.
[EXP]
Không tự động hóa.
[D]
Cấu hình Docker engine trên máy host tự xóa container.
[EXP]
Docker engine không tự động xóa nếu không có cờ cấu hình --rm.

@correct: B
@point: 20

## Q5

Học viên cấu hình tệp tin `ci.yml` kéo image `1.0.0` về verify. Do sơ xuất, học viên push tệp tin cấu hình lên GitHub để kích hoạt workflow trước khi thực hiện build và đẩy image `1.0.0` từ local lên Registry. Kết quả là gì?

[A]
Job verify thất bại ngay lập tức ở bước docker pull do kho lưu trữ Container Registry chưa có image đó.
[EXP]
Chính xác. Khi Registry chưa có image và tag tương ứng, lệnh docker pull của Runner sẽ không thể tìm thấy image và trả về lỗi, khiến job thất bại.
[B]
Runner sẽ tự động biên dịch mã nguồn Java để tạo image thay thế.
[EXP]
Runner không thể tự thay đổi workflow.
[C]
Job vẫn báo Passed bình thường vì GitHub Actions tự bỏ qua lỗi docker pull.
[EXP]
Lỗi docker pull sẽ làm step fail lập tức.
[D]
Docker Engine của host sẽ tự sinh ra mock image để vượt qua bài test.
[EXP]
Docker Engine không tạo mock image.

@correct: A
@point: 20
