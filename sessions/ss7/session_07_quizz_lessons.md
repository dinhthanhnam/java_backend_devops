# QUIZ LESSONS - SESSION 07: QUY TRÌNH CI/CD VỚI GITHUB ACTIONS

# LESSON 01: Nền tảng CI/CD: Tổng quan GitHub Actions và Kiến trúc Runner

## Q1

Khi cấu hình một Self-hosted runner thông qua Docker Compose (sử dụng image myoung34/github-runner), biến môi trường nào chứa tham số bảo mật bắt buộc để định danh và cho phép Runner kết nối với GitHub?

[A]
RUNNER_PASSWORD
[EXP]
Hệ thống sử dụng token bảo mật định danh tự động sinh ra thay vì mật khẩu truyền thống của người dùng.
[B]
RUNNER_SSH_KEY
[EXP]
Runner kết nối thông qua giao thức HTTPS và sử dụng token, không sử dụng giao thức SSH để kết nối về máy chủ.
[C]
RUNNER_TOKEN
[EXP]
Chính xác. Chuỗi token xác thực sinh ra từ giao diện GitHub cung cấp mức độ bảo mật cần thiết để xác minh tính hợp lệ của máy chủ Runner khi cấu hình qua Docker.
[D]
RUNNER_AUTH_CODE
[EXP]
Biến môi trường này không tồn tại trong cấu hình chuẩn của hệ thống Docker image đang sử dụng.

@correct: C
@point: 20

## Q2

Cơ chế hoạt động giao tiếp giữa GitHub Server và GitHub Actions Runner diễn ra theo cách thức nào?

[A]
GitHub Server chủ động thiết lập kết nối (Push) SSH vào máy chủ Runner để kích hoạt lệnh.
[EXP]
Server không có quyền chủ động kết nối hay vượt tường lửa (firewall) để SSH vào Runner cục bộ.
[B]
Runner liên tục gửi yêu cầu truy vấn (Polling) lên GitHub Server để tự nhận các luồng công việc mới.
[EXP]
Chính xác. Kiến trúc Runner dựa trên cơ chế Pull (long-polling), chủ động kiểm tra trạng thái công việc trên Server để thực thi.
[C]
Lập trình viên bắt buộc mở cổng mạng (port) của thiết bị để Server truyền tập dữ liệu công việc về.
[EXP]
Runner thiết lập kết nối ngoại mạng (Outbound connection) nên hoàn toàn không cần cấu hình mở cổng mạng nội bộ.
[D]
Runner trực tiếp rà soát và sao chép mã nguồn từ thiết bị phát triển thông qua Docker Network.
[EXP]
Runner trực tiếp sao chép (clone) mã nguồn từ GitHub Server, không đọc từ máy chủ cá nhân.

@correct: B
@point: 20

## Q3

Đâu là đặc thù phân biệt lớn nhất giữa GitHub-hosted runner và Self-hosted runner trong quá trình xử lý công việc?

[A]
GitHub-hosted runner cung cấp môi trường máy ảo hoàn toàn sạch sẽ bị hủy ngay sau lượt chạy, còn Self-hosted runner duy trì trạng thái hệ thống giữa các lượt chạy.
[EXP]
Chính xác. Máy ảo GitHub-hosted cung cấp một môi trường cách ly nghiêm ngặt, trong khi Self-hosted lưu giữ trực tiếp trạng thái cài đặt tại hệ thống.
[B]
GitHub-hosted runner chỉ có khả năng biên dịch Java, trong khi Self-hosted runner có khả năng biên dịch đa ngôn ngữ.
[EXP]
Cả hai môi trường đều tương thích với quy trình cài đặt và biên dịch của mọi ngôn ngữ lập trình.
[C]
Self-hosted runner mặc định bị tính phí dựa trên từng phút chạy, còn GitHub-hosted runner luôn miễn phí.
[EXP]
Self-hosted runner hoàn toàn không bị hệ thống tính thời gian chạy; ngược lại, GitHub-hosted runner có hạn mức miễn phí mỗi tháng.
[D]
GitHub-hosted runner yêu cầu duy trì kết nối Internet liên tục, còn Self-hosted runner có thể hoạt động ngoại tuyến.
[EXP]
Bất cứ dạng Runner nào cũng cần tiếp nhận tín hiệu từ hệ thống GitHub trung tâm thông qua mạng Internet.

@correct: A
@point: 20

## Q4

Đâu là lợi thế của việc sử dụng Runner triển khai bằng Docker Compose so với việc cài đặt Runner trực tiếp lên hệ điều hành của máy chủ?

[A]
Tự động hóa quá trình đóng gói và cô lập môi trường thực thi, tránh xung đột phần mềm với các dịch vụ khác trên máy chủ.
[EXP]
Chính xác. Docker Compose giúp cô lập runner, dễ dàng cấu hình, dọn dẹp và đảm bảo tính nhất quán môi trường so với cài trực tiếp.
[B]
Cho phép Runner bỏ qua các lớp bảo mật mạng nội bộ.
[EXP]
Docker Compose không bỏ qua bảo mật mạng; mạng vẫn tuân thủ firewall của host.
[C]
Tăng tốc độ đường truyền mạng lên gấp đôi.
[EXP]
Docker không thể tăng tốc giới hạn vật lý của đường truyền mạng.
[D]
Không yêu cầu cấu hình token bảo mật từ GitHub.
[EXP]
Dù triển khai qua Docker thì Runner vẫn bắt buộc cần RUNNER_TOKEN để xác thực.

@correct: A
@point: 20

## Q5

Thành phần nào trong kiến trúc GitHub Actions chịu trách nhiệm lắng nghe các sự kiện (event) và quyết định khi nào kích hoạt luồng công việc?

[A]
GitHub Actions Runner.
[EXP]
Runner chỉ chịu trách nhiệm nhận và thực thi công việc, không đảm nhận việc lắng nghe sự kiện của repository.
[B]
GitHub Server (Nền tảng trung tâm).
[EXP]
Chính xác. GitHub Server lắng nghe các sự kiện (như push, pull request) và tự động kích hoạt tiến trình CI/CD dựa trên cấu hình workflow.
[C]
Docker Daemon của máy host.
[EXP]
Docker Daemon dùng để chạy container, không tương tác trực tiếp với các sự kiện GitHub.
[D]
Webhook tích hợp bên thứ ba.
[EXP]
Hệ thống sử dụng cơ chế tích hợp nội bộ của GitHub, không cần webhook bên ngoài cho các luồng Actions tiêu chuẩn.

@correct: B
@point: 20


# LESSON 02: Xây dựng cấu trúc Workflow CI/CD cơ bản

## Q1

Trong định dạng file Workflow của GitHub Actions, tính năng nào chịu trách nhiệm thiết lập các yếu tố kích hoạt quy trình (như sự kiện cập nhật mã nguồn)?

[A]
runs-on
[EXP]
Thuộc tính `runs-on` dùng để chỉ định nền tảng máy chủ thực thi công việc.
[B]
jobs
[EXP]
Khối `jobs` làm nhiệm vụ định nghĩa danh sách công việc sẽ tiến hành, không phải sự kiện kích hoạt.
[C]
on
[EXP]
Chính xác. Khối lệnh `on` chuyên xử lý việc khai báo các sự kiện (ví dụ như `push`, `pull_request`) để khởi động tiến trình.
[D]
steps
[EXP]
Chuỗi `steps` đại diện cho các bước thực thi độc lập bên trong công việc (job).

@correct: C
@point: 20

## Q2

Mã lệnh nào dưới đây áp dụng thư viện Action đóng gói sẵn từ cộng đồng để thiết lập môi trường Java 17?

[A]
run: java install 17
[EXP]
Câu lệnh shell truyền thống phụ thuộc vào từng hệ điều hành, không phải Action tiêu chuẩn của hệ thống.
[B]
uses: actions/setup-java@v5
[EXP]
Chính xác. Cú pháp `uses` gọi ra Action tiêu chuẩn `actions/setup-java`, giúp nhanh chóng cấu hình biến môi trường và JDK cần thiết.
[C]
image: eclipse-temurin:17
[EXP]
Từ khóa `image` là cú pháp của các nền tảng khác (như GitLab CI), không phải đặc thù của thư viện GitHub Actions.
[D]
needs: java-environment
[EXP]
Khóa `needs` dùng để kiểm soát mối quan hệ phụ thuộc tuần tự giữa các công việc (jobs).

@correct: B
@point: 20

## Q3

Nếu lập trình viên vô tình sử dụng phím Tab để thực hiện thao tác thụt lề cấu hình trong tệp `.github/workflows/ci.yml`, phản ứng của hệ thống GitHub là gì?

[A]
GitHub tự động thiết lập lại cấu trúc thành định dạng JSON để biên dịch tiếp.
[EXP]
Nền tảng không thực hiện hiệu chỉnh mã nguồn hay tự động đổi định dạng tệp cấu hình của dự án.
[B]
Quá trình khởi chạy thất bại và hệ thống thông báo lỗi biên dịch cú pháp (YAML syntax error).
[EXP]
Chính xác. Tiêu chuẩn quốc tế của định dạng YAML nghiêm cấm sử dụng phím Tab trong việc cấu trúc tầng lớp và chỉ cho phép khoảng trắng (spaces).
[C]
Luồng công việc hoạt động chậm lại nhưng vẫn đảm bảo kết quả chính xác.
[EXP]
Lỗi cú pháp sẽ ngăn chặn tiến trình trước khi quá trình phân bổ tài nguyên diễn ra.
[D]
Máy chủ Runner được yêu cầu sử dụng hệ điều hành Ubuntu thay vì Linux mặc định.
[EXP]
Phân bổ hệ điều hành là do từ khóa `runs-on` quy định, không liên quan đến lỗi sai cấu trúc YAML.

@correct: B
@point: 20

## Q4

Đường dẫn thư mục bắt buộc nào phải được tạo trong repository để GitHub tự động nhận diện và chạy các file cấu hình workflow?

[A]
`.github/workflows/`
[EXP]
Chính xác. GitHub yêu cầu tất cả các file cấu hình CI/CD (YAML) phải đặt chính xác trong thư mục `.github/workflows/` tại gốc của repository.
[B]
`.gitlab-ci/`
[EXP]
Thư mục này không có ý nghĩa với hệ thống GitHub Actions.
[C]
`workflows/`
[EXP]
Nếu thư mục workflows không được đặt bên trong thư mục ẩn `.github`, hệ thống sẽ không nhận diện được.
[D]
`.github/actions/`
[EXP]
Thư mục này dùng để định nghĩa các custom actions riêng lẻ, không dùng để chứa các file workflow tổng thể.

@correct: A
@point: 20

## Q5

Trong khối cấu hình `runs-on: [self-hosted, quickbite]`, ý nghĩa của các nhãn (labels) này là gì?

[A]
Xác định hệ điều hành cài đặt và tên máy chủ vật lý cụ thể.
[EXP]
Nhãn không mang thông tin mặc định về hệ điều hành, nó là từ khóa tuỳ chỉnh.
[B]
Định tuyến công việc tới các máy chủ Runner được gắn nhãn tương ứng.
[EXP]
Chính xác. Hệ thống sẽ so khớp các nhãn (tags/labels) được cấu hình trong file yaml với nhãn của runner đang hoạt động để phân bổ job. Ở đây job sẽ gửi đến runner có nhãn self-hosted và quickbite.
[C]
Chỉ định tên tài khoản GitHub được phép chạy job này.
[EXP]
Các nhãn không liên quan đến quyền của người dùng trên GitHub.
[D]
Kích hoạt luồng chạy nhanh (quickbite) bỏ qua các bước kiểm tra.
[EXP]
"quickbite" chỉ là một nhãn do người dùng định nghĩa, không mang ý nghĩa kích hoạt chức năng ẩn của hệ thống.

@correct: B
@point: 20


# LESSON 03: Cơ chế điều phối và phân tách Jobs trong luồng CI/CD

## Q1

Khi bạn thiết lập nhiều `jobs` cấu hình song song không mang từ khóa ràng buộc phụ thuộc, quy trình mặc định của GitHub Actions diễn ra như thế nào?

[A]
Thực thi tuần tự từ trên xuống dưới dựa trên thứ tự khai báo.
[EXP]
Cơ chế tự nhiên của khối lệnh sẽ không thực thi theo tuyến tính trừ khi có ràng buộc.
[B]
Phân bổ toàn bộ các công việc để thực thi song song (Parallel) cùng lúc nhằm tối đa hóa hiệu suất.
[EXP]
Chính xác. Hệ thống sẽ ưu tiên phân bổ đồng thời để tiết kiệm tổng thể thời gian thực thi của cả luồng quy trình.
[C]
Yêu cầu lập trình viên can thiệp thủ công trên giao diện Web UI để chọn thứ tự thực thi.
[EXP]
Khối CI/CD được thiết kế theo xu hướng tự động hóa hoàn toàn, không yêu cầu các quyết định thủ công cản trở luồng chạy.
[D]
Tự động bỏ qua công việc cuối cùng nhằm phân bổ tài nguyên tối ưu.
[EXP]
Không một công việc nào bị bỏ qua trừ trường hợp có thiết lập từ khoá loại trừ như `if`.

@correct: B
@point: 20

## Q2

Cho cấu hình một luồng công việc chứa 2 công việc A và B. Nếu B được thiết lập `needs: [A]`, chuyện gì xảy ra nếu công việc A thất bại (Failed)?

[A]
Công việc B tiếp tục khởi động nhằm truy xuất thông báo lỗi của A.
[EXP]
Khi hệ thống có lỗi, công việc B không được phép khởi động.
[B]
Công việc B tự động chuyển vào trạng thái bị bỏ qua (Skipped) và luồng công việc kết thúc.
[EXP]
Chính xác. Để giảm thiểu lãng phí và nguy cơ hỏng hệ thống, khi một khối lượng công việc cấp trên gặp sự cố, hệ thống chặn kích hoạt toàn bộ luồng phía dưới.
[C]
Hệ thống tiến hành chạy lại công việc A tối đa ba lần trước khi huỷ luồng.
[EXP]
Nền tảng mặc định không tự động thử nghiệm lại tiến trình (retry) trừ phi có sử dụng luồng Action bên ngoài can thiệp.
[D]
Công việc B sẽ tạm ngừng 5 phút để công việc A kịp khôi phục mã lỗi.
[EXP]
Tiến trình sẽ kết thúc dứt khoát và không cấu hình thời gian đệm chờ lỗi.

@correct: B
@point: 20

## Q3

Đặc thù cô lập (Job Isolation) của môi trường Runner ảnh hưởng thế nào đến luồng dữ liệu của hai công việc chạy liên tiếp?

[A]
Công việc sau tự động nạp các mã nguồn đã tải xuống của công việc trước vào bộ đệm.
[EXP]
Tài nguyên máy ảo bị xoá bỏ giữa các quy trình, ngăn cản khả năng này.
[B]
Các tệp tạm thời tạo ra ở công việc trước biến mất, bắt buộc công việc sau phải dùng Action tạo tải (Checkout/Artifacts) để thiết lập lại vùng làm việc.
[EXP]
Chính xác. Quá trình dọn dẹp hệ thống liên tục yêu cầu kĩ sư tự động khai báo từng công đoạn chia sẻ tệp (như upload/download artifact) ở mỗi job riêng.
[C]
Bộ nhớ ngẫu nhiên (RAM) của hai phiên chạy được kết nối chung vào hệ thống.
[EXP]
Tính cô lập cô lập dữ liệu tuyệt đối làm phân lập tài nguyên cả ở cấu trúc bộ nhớ vật lí.
[D]
Hệ thống tự động đồng bộ hóa thời gian làm việc để giảm thời gian phản hồi.
[EXP]
Việc đồng bộ hoá hệ thống tập trung vào mặt định thời gian hệ thống chung, không liên quan đến cơ chế luồng dữ liệu.

@correct: B
@point: 20

## Q4

Làm thế nào để cấu trúc một workflow mà một job phân tích mã nguồn (lint) và một job kiểm thử (test) chạy đồng thời, sau đó job triển khai (deploy) chỉ chạy khi cả hai job trước đều thành công?

[A]
Đặt cả 3 job trong cùng một file YAML và không cấu hình gì thêm.
[EXP]
Nếu không cấu hình gì thêm, cả 3 job sẽ chạy song song đồng thời.
[B]
Cấu hình `needs: [lint, test]` cho job deploy, trong khi job lint và test không sử dụng từ khóa needs.
[EXP]
Chính xác. Bỏ trống needs ở hai job đầu giúp chúng mặc định chạy song song, và needs ở job deploy sẽ chặn nó cho đến khi hai job kia hoàn thành thành công.
[C]
Sử dụng từ khóa `parallel: true` ở cả 3 job.
[EXP]
Cú pháp GitHub Actions không sử dụng từ khóa `parallel: true` cho mục đích này.
[D]
Cấu hình `depends_on: [deploy]` cho job lint và test.
[EXP]
Cú pháp phụ thuộc là `needs`, không phải `depends_on`, và mối quan hệ phụ thuộc bị ngược.

@correct: B
@point: 20

## Q5

Khi sử dụng cơ chế cô lập Job (Job Isolation), nếu bạn cài đặt một phần mềm bằng dòng lệnh `apt-get install` trong `job_1`, phần mềm đó có sẵn sàng trong `job_2` không?

[A]
Có, nếu `job_2` có khai báo `needs: [job_1]`.
[EXP]
Từ khóa needs chỉ quy định thứ tự chạy, không vô hiệu hóa tính cô lập môi trường.
[B]
Có, vì cả hai job đều chạy trên cùng một máy chủ Runner.
[EXP]
Dù chạy chung một runner, các tiến trình công việc được Runner thiết lập cách ly dữ liệu tạm (workspace) và môi trường.
[C]
Không, vì mỗi job được chạy trên một môi trường hoặc phiên làm việc cô lập.
[EXP]
Chính xác. Job Isolation đảm bảo sự độc lập tuyệt đối giữa các tiến trình, không có tính kế thừa các sửa đổi về môi trường và phần mềm đã được cài đặt.
[D]
Có, nếu hệ thống sử dụng GitHub-hosted runner.
[EXP]
GitHub-hosted runner cấp phát một máy ảo hoàn toàn mới cho mỗi job, vì vậy tính cô lập càng được đảm bảo tuyệt đối.

@correct: C
@point: 20


# LESSON 04: Ứng dụng CI/CD: Biên dịch tự động Spring Boot với Gradle

## Q1

Mục tiêu chính khi lập trình viên thực hiện câu lệnh `chmod +x ./gradlew` trước khối lệnh biên dịch mã nguồn Gradle là gì?

[A]
Thiết lập thư mục Gradle Wrapper dưới quyền người dùng Root trên máy ảo Linux.
[EXP]
Lệnh chmod cung cấp quyền cho tệp, không can thiệp vào quy trình tài khoản Root máy chủ.
[B]
Cưỡng ép máy chủ nền tảng Linux cấp quyền khởi chạy mã thực thi (execute bit) cho tiện ích Wrapper, giải quyết lỗi Permission Denied.
[EXP]
Chính xác. Bởi vì các mã nguồn sinh ra trên nền tảng Windows thường thiếu quyền thực thi khi đồng bộ lên hệ thống, việc cấp quyền tường minh tránh thông báo lỗi thoát quyền truy cập.
[C]
Nén toàn bộ thư mục Gradle vào một bản phân phối nhị phân độc lập.
[EXP]
Quá trình tạo gói là tác vụ độc lập của hệ thống Gradle, không thực hiện qua lệnh shell cơ bản này.
[D]
Ra lệnh cho hệ thống tự động đăng xuất tải nguyên từ Git để lưu vùng nhớ.
[EXP]
Lệnh phân quyền là cơ sở điều hành của hạt nhân hệ thống, không thay thế luồng tài nguyên của nền tảng kho lưu trữ.

@correct: B
@point: 20

## Q2

Tại sao việc thay thế tác vụ `./gradlew build` thành `./gradlew bootJar` lại giúp tối ưu hóa tổng thể luồng công việc (workflow) một cách hiệu quả?

[A]
Tác vụ `bootJar` có cơ chế biên dịch ngầm thông minh hơn so với thư viện mặc định.
[EXP]
Khả năng biên dịch lõi của hệ thống Java hoàn toàn như nhau.
[B]
Tác vụ `bootJar` đóng gói bản phát hành mà không cần thực thi quy trình kiểm thử tự động (Unit Test), giảm áp lực thời gian đáng kể.
[EXP]
Chính xác. Tác vụ `bootJar` thuần tuý là quá trình chuyển đổi sang mã máy và đóng gói, loại bỏ hoàn toàn các chu kỳ tải cấu trúc ngữ cảnh ứng dụng rườm rà.
[C]
Lệnh `build` yêu cầu khởi tạo hai Container độc lập để tạo dữ liệu so sánh.
[EXP]
Toàn bộ quá trình vòng đời nằm nguyên ở môi trường của Runner, không sử dụng tài nguyên Container đa lớp.
[D]
Cấu trúc `bootJar` tương thích với chuẩn đóng gói của nền tảng Docker tốt hơn.
[EXP]
Tệp dữ liệu đầu ra không ảnh hưởng đến độ tương thích, cả hai đều có thể phân tách cấu trúc hình ảnh Docker tương đương.

@correct: B
@point: 20

## Q3

Action `actions/upload-artifact@v4` giải quyết thách thức quản lý dữ liệu đặc thù nào của kiến trúc GitHub Actions?

[A]
Khắc phục vấn đề tự động xoá sổ các tệp tin sản phẩm khi một công việc kết thúc nhờ tính chất Cô lập công việc (Job Isolation).
[EXP]
Chính xác. Khả năng đồng bộ các tệp tin lưu vào kho chứa tập trung của GitHub cho phép bảo vệ thành quả của công việc xây dựng mã nguồn.
[B]
Đảm bảo tốc độ truy xuất cơ sở dữ liệu hệ thống PostgreSQL đạt chuẩn công nghiệp.
[EXP]
Việc sao lưu bản phân phối tĩnh không tương thích và không can thiệp được vào hạ tầng kết nối truy xuất động.
[C]
Hợp nhất các tệp cấu hình Gradle với các tệp quản lý cấu trúc của hệ thống Maven.
[EXP]
Sự hợp nhất công cụ hoàn toàn bất khả thi thông qua các Action xuất dữ liệu.
[D]
Ngăn chặn các tệp nguồn có khả năng dính vi-rút tải lên hệ thống.
[EXP]
Hệ thống không cung cấp lõi bảo mật phần mềm cho dữ liệu tạo tác.

@correct: A
@point: 20

## Q4

Tại sao thư mục `build/` (nơi chứa kết quả biên dịch của Spring Boot) không thể được tải trực tiếp về máy cá nhân từ giao diện một job mà phải thông qua việc khai báo hành động `actions/upload-artifact@v4`?

[A]
Vì thư mục này chứa mã độc có thể làm hại máy chủ.
[EXP]
Đây là kết quả biên dịch bình thường của Spring Boot, không chứa mã độc mặc định.
[B]
Vì bộ nhớ cục bộ của Runner chỉ tồn tại tạm thời và sẽ bị xoá đi ngay khi tiến trình job hoàn tất.
[EXP]
Chính xác. Mọi dữ liệu sinh ra trên môi trường runner sẽ bị tiêu hủy theo tiến trình để tiết kiệm dung lượng, phải dùng action upload để đẩy lên hệ thống máy chủ lưu trữ lâu dài của GitHub.
[C]
Vì kích thước thư mục build luôn vượt quá dung lượng cho phép của GitHub.
[EXP]
Dung lượng thư mục build thường nhỏ hơn hạn mức lưu trữ artifact của GitHub rất nhiều.
[D]
Vì hệ điều hành Linux bảo mật nên chặn quyền đọc thư mục build.
[EXP]
Runner hoàn toàn có quyền đọc các file sinh ra trong workspace của nó.

@correct: B
@point: 20

## Q5

Khi sử dụng `actions/setup-java@v5`, thuộc tính `distribution` (ví dụ: `temurin`) dùng để làm gì?

[A]
Định nghĩa phiên bản của ngôn ngữ Java.
[EXP]
Phiên bản Java được định nghĩa ở thuộc tính `java-version`.
[B]
Chọn nhà cung cấp bản phân phối của JDK do có nhiều tổ chức cùng đóng gói JDK.
[EXP]
Chính xác. Hệ sinh thái Java có nhiều tổ chức phân phối (distributions) bộ cài JDK khác nhau dựa trên mã nguồn mở OpenJDK (ví dụ: Adoptium/Temurin, Amazon Corretto, Microsoft).
[C]
Kích hoạt thuật toán phân phối mã nguồn tới nhiều máy chủ Runner.
[EXP]
Đây là thiết lập môi trường Java, không liên quan đến việc phân phối mã nguồn trên hệ thống mạng.
[D]
Chỉ định vùng địa lý (region) của máy chủ GitHub sẽ xử lý mã nguồn.
[EXP]
Vùng địa lý không được cấu hình qua khối setup-java.

@correct: B
@point: 20


# LESSON 05: Chẩn đoán sự cố và quản trị log hệ thống CI/CD

## Q1

Quy trình ưu tiên tiêu chuẩn khi đối mặt với luồng công việc trả về trạng thái thất bại (Failed) là gì?

[A]
Khôi phục mã nguồn về trạng thái commit nguyên bản nhất có thể và cập nhật thủ công.
[EXP]
Sự tuỳ tiện chỉnh sửa gây hỗn loạn kĩ thuật không có phương pháp giải trình rủi ro.
[B]
Kích hoạt giao diện Tab Actions, truy cập log của công việc cụ thể, sau đó cuộn màn hình hiển thị để kiểm tra trực tiếp mã thoát (Exit code) và thông báo lỗi.
[EXP]
Chính xác. Đọc nhật ký chạy công việc từ cấp độ thấp mang tính chất quan trọng nhất giúp thiết lập vùng cần bảo trì và xác định lỗi.
[C]
Thay thế Action `actions/checkout` bằng công cụ tải mã nguồn cơ sở của nền tảng.
[EXP]
Lỗi quy trình phát sinh từ rất nhiều loại, không chỉ lỗi do nạp khối lượng mã nguồn.
[D]
Thay đổi phiên bản cài đặt Ubuntu mặc định trên máy chủ chạy thuật toán xuống mức tiêu chuẩn cũ.
[EXP]
Thay thế hệ điều hành không thể sửa chữa các lỗi logic hoặc cú pháp của tệp cấu trúc dự án.

@correct: B
@point: 20

## Q2

Mã thoát (Exit Code) mang tín hiệu số `126` trong môi trường hệ điều hành chuẩn của GitHub Actions đại diện cho vấn đề gì?

[A]
Hệ thống mạng truy vấn thất bại do kết nối máy chủ ngoại miền quá hạn.
[EXP]
Lỗi mạng sẽ có hệ số định danh cảnh báo ngoại tuyến tương ứng.
[B]
Lệnh không thể được thực thi do thiếu tính năng phân quyền hệ thống chạy tệp.
[EXP]
Chính xác. Hệ số thoát này cảnh báo về lỗi quyền truy cập tệp khi công cụ (như gradlew) không có quyền execute bit trên Linux.
[C]
Tệp khai báo thuật toán Java bị lệch hoặc sai lỗi chính tả.
[EXP]
Lỗi ngữ pháp nội bộ lớp sẽ cung cấp Exit Code bằng `1` kết hợp thông tin lỗi thư viện.
[D]
Hệ thống từ chối khởi động quá trình do quá trình vượt khối lượng cho phép.
[EXP]
Giới hạn hệ thống sẽ kích hoạt lệnh chẩn đoán tài nguyên quá tải.

@correct: B
@point: 20

## Q3

Nguyên nhân phổ biến làm phát sinh lỗi kiểm thử "Test FAILED" nhưng nhật ký biên dịch mã nguồn Java vẫn thể hiện "Passed" thành công là gì?

[A]
Nền tảng Gradle lỗi phân tách mô-đun hoặc tự động khởi động các phiên bản phần mềm Java ngẫu nhiên.
[EXP]
Phiên bản và cấu trúc phần mềm được hệ thống khoá tuyệt đối dựa trên thiết lập Workflow.
[B]
Cú pháp ngôn ngữ Java bảo đảm độ chính xác, nhưng giá trị logic kết quả tính toán không khớp với mục tiêu kỳ vọng của khối kiểm thử.
[EXP]
Chính xác. Giai đoạn Compiler chỉ đánh giá về định dạng cú pháp cấu trúc; trong khi Assertion kiểm thử xác nhận về độ toàn vẹn của logic chức năng.
[C]
Hệ thống tệp tin GitHub bị ngắt kết nối do các luồng công việc gây xung đột băng thông phân phối.
[EXP]
Giai đoạn biên dịch phần mềm không truy cập dữ liệu luồng giao dịch trong nội tại các kiểm thử đã đóng kín.
[D]
Kĩ sư đã quên kích hoạt cài đặt tác vụ nạp biến môi trường tự động `setup-java`.
[EXP]
Thiếu Java, hệ thống sẽ kết thúc thất bại trực tiếp ở quá trình tạo luồng trung gian chứ không biên dịch tệp.

@correct: B
@point: 20

## Q4

Khi cấu hình workflow bị lỗi "YAML syntax error" và hiển thị dấu X đỏ ngay sau khi push code, bước kiểm tra đầu tiên lập trình viên nên thực hiện là gì?

[A]
Kiểm tra kết nối mạng của máy chủ Runner.
[EXP]
Lỗi cấu trúc YAML xảy ra ở phía GitHub Server trước cả khi Server gửi lệnh cho Runner.
[B]
Mở lại tệp `.github/workflows/ci.yml` kiểm tra xem có vô tình sử dụng phím Tab thay vì Space để thụt lề hay không.
[EXP]
Chính xác. Lỗi phân tích cú pháp YAML (syntax error) rất hay xảy ra do định dạng khoảng trắng bị lệch hoặc lập trình viên lỡ nhấn phím Tab.
[C]
Khởi động lại Docker container của tiến trình Runner.
[EXP]
Quá trình runner khởi động lại không tác động đến lỗi cấu trúc cú pháp của tệp YAML.
[D]
Đổi tên repository trên GitHub để reset bộ nhớ đệm.
[EXP]
Đổi tên repository không giải quyết được lỗi cú pháp phần mềm.

@correct: B
@point: 20

## Q5

Điều gì sẽ xảy ra nếu lập trình viên không gán đúng các nhãn (labels) trên file YAML so với nhãn của Self-hosted runner (ví dụ khai báo `runs-on: [self-hosted, sai_ten]`)?

[A]
GitHub sẽ tự động bỏ qua nhãn bị sai và vẫn gửi job cho Runner gần nhất.
[EXP]
Hệ thống định tuyến của GitHub Actions yêu cầu so khớp chính xác cấu hình nhãn, không được tự động bỏ qua.
[B]
GitHub tự động phân bổ job chạy trên GitHub-hosted runner miễn phí để thay thế.
[EXP]
GitHub Server không tự ý chuyển hướng môi trường nếu lập trình viên đã chỉ định cụ thể các nhãn hệ thống.
[C]
Luồng công việc sẽ bị treo ở trạng thái chờ "Queued" hoặc "Waiting for a runner" vô thời hạn do không tìm thấy Runner khớp cấu hình.
[EXP]
Chính xác. Khi nhãn yêu cầu (requested labels) không khớp với nhãn của bất kỳ runner khả dụng nào đang hoạt động, tiến trình sẽ bị mắc kẹt ở hàng đợi cho đến khi bị hủy vì timeout.
[D]
Runner tự sửa tên nhãn của mình để phù hợp với file cấu hình.
[EXP]
Runner không thể tự động thay đổi cấu hình nội bộ của mình để chạy file bị sai.

@correct: C
@point: 20
