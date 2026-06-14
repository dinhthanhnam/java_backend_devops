# QUIZ LESSONS - SESSION 02

# LESSON 01: Khái niệm container và so sánh Docker với máy ảo

## Q1

Ảo hóa ở cấp độ hệ điều hành (Container) khác biệt như thế nào về mặt kiến trúc so với ảo hóa ở cấp độ phần cứng (Virtual Machine)?

[A]
Container sử dụng chung nhân hệ điều hành của máy host; Virtual Machine bắt buộc phải mang theo một hệ điều hành khách (Guest OS).
[EXP]
Đây là khác biệt cốt lõi. Container chia sẻ chung nhân (Shared Kernel) của Host OS nên rất nhẹ, còn VM bắt buộc ảo hóa phần cứng và chạy một Guest OS hoàn chỉnh.
[B]
Container tự ảo hóa CPU và RAM riêng biệt; Virtual Machine chạy trực tiếp trên nhân hệ điều hành của máy host.
[EXP]
Phát biểu bị ngược. VM ảo hóa phần cứng để tạo CPU/RAM ảo, còn Container chạy trực tiếp tiến trình trên nhân của Host OS.
[C]
Container yêu cầu phần mềm Hypervisor để quản lý; Virtual Machine chạy trực tiếp thông qua công cụ Docker Engine.
[EXP]
Phát biểu bị ngược. VM yêu cầu Hypervisor để ảo hóa phần cứng, còn Container được quản lý trực tiếp qua Docker Engine.
[D]
Container khóa cứng dung lượng RAM tĩnh khi cấp phát; Virtual Machine tự động co giãn tài nguyên RAM theo tải thực tế.
[EXP]
Phát biểu bị ngược. VM thường khóa cứng RAM tĩnh khi khởi tạo, còn Container cho phép tiêu hao tài nguyên động theo tải thực tế.


@correct: A
@point: 20

## Q2

Tại sao Docker Container có tốc độ khởi động (boot) gần như tức thì (tính bằng mili-giây) trong khi máy ảo VM phải mất hàng phút?

[A]
Vì container sử dụng CPU ảo có tốc độ xử lý nhanh hơn rất nhiều so với CPU vật lý được cấp cho máy ảo.
[EXP]
Container sử dụng trực tiếp CPU vật lý của máy host chứ không có khái niệm CPU ảo nhanh hơn.
[B]
Vì container không phải khởi động lại toàn bộ nhân hệ điều hành mà chỉ khởi chạy tiến trình ứng dụng trên nhân dùng chung.
[EXP]
Đúng. Container chỉ đơn thuần là khởi chạy một tiến trình trên nhân Host OS có sẵn nên tốc độ là mili-giây, còn VM phải nạp toàn bộ Guest OS từ đầu.
[C]
Vì Docker Daemon tự động tắt toàn bộ các tiến trình chạy ngầm của hệ điều hành host trước khi container khởi động.
[EXP]
Docker Daemon không tắt tiến trình của máy host khi khởi động container, các tiến trình host vẫn chạy bình thường.
[D]
Vì container tự động bỏ qua bước nạp các biến môi trường hệ thống trong suốt quá trình khởi chạy tiến trình.
[EXP]
Container vẫn nạp đầy đủ các biến môi trường được cấu hình khi khởi chạy tiến trình để ứng dụng hoạt động chính xác.


@correct: B
@point: 20

## Q3

Trong cơ chế cô lập tiến trình của container Linux, tính năng `Namespaces` và `Cgroups` đóng vai trò cụ thể nào?

[A]
Namespaces giới hạn tài nguyên CPU/RAM sử dụng; Cgroups chịu trách nhiệm phân vùng cô lập mạng và tiến trình.
[EXP]
Phát biểu bị ngược. Namespaces chịu trách nhiệm cô lập tầm nhìn hệ thống, còn Cgroups giới hạn tài nguyên CPU/RAM.
[B]
Namespaces mã hóa các file dữ liệu của container; Cgroups thực hiện kết nối mạng giữa các container với nhau.
[EXP]
Namespaces không có vai trò mã hóa file, và Cgroups không chịu trách nhiệm kết nối mạng hệ thống.
[C]
Namespaces phân chia cổng mạng nội bộ cho container; Cgroups phân tích và ghi lại log hoạt động của các tiến trình.
[EXP]
Cgroups không ghi lại log hoạt động tiến trình mà giới hạn tài nguyên phần cứng.
[D]
Namespaces tạo ra sự cô lập về tầm nhìn hệ thống; Cgroups giới hạn lượng tài nguyên phần cứng tối đa được sử dụng.
[EXP]
Đúng. Namespaces cô lập tài nguyên hệ thống (như Process ID, Network, Mount...) giúp container tưởng nó đang chạy một mình. Cgroups (Control Groups) giới hạn CPU/RAM mà tiến trình đó được phép tiêu thụ.


@correct: D
@point: 20

## Q4

Sự khác biệt mấu chốt về việc cấp phát tài nguyên bộ nhớ RAM giữa Máy ảo (VM) và Docker Container là gì?

[A]
VM tiêu hao RAM động theo nhu cầu thực tế của dịch vụ; Docker Container khóa cứng RAM tĩnh ngay khi khởi tạo.
[EXP]
Phát biểu bị ngược. VM khóa cứng tài nguyên RAM, còn container tiêu hao động theo nhu cầu của tiến trình.
[B]
VM không tiêu tốn tài nguyên RAM cho hệ điều hành khách; Docker Container tiêu tốn RAM cho lớp Docker Engine.
[EXP]
VM tiêu tốn rất nhiều RAM để duy trì hệ điều hành khách (Guest OS). Docker Engine tiêu tốn tài nguyên cực kỳ tối giản.
[C]
VM chia sẻ chung tài nguyên RAM với máy chủ vật lý; Docker Container ảo hóa hoàn toàn RAM và không dùng RAM của host.
[EXP]
Docker Container chạy trực tiếp trên RAM vật lý của máy host, không ảo hóa RAM độc lập như máy ảo VM.
[D]
VM khóa cứng tài nguyên RAM tĩnh; Docker Container tự động tiêu hao RAM động theo nhu cầu hoạt động thực tế.
[EXP]
Chính xác. VM khi được cấp 2GB RAM sẽ chiếm dụng cứng 2GB từ máy host. Docker Container cấp phát tài nguyên động và chỉ dùng đúng lượng RAM tiến trình cần tại thời điểm chạy.


@correct: D
@point: 20

## Q5

Điều gì có thể xảy ra nếu có một lỗ hổng bảo mật nghiêm trọng xuất hiện ở nhân hệ điều hành (Kernel Vulnerability) của máy chủ host chạy Docker?

[A]
Docker Daemon sẽ tự động phát hiện và chuyển đổi container sang chạy trên một nhân hệ điều hành khách ảo độc lập.
[EXP]
Docker Daemon không có khả năng tự động tạo máy ảo để di chuyển container sang chạy nhân ảo độc lập.
[B]
Các container sẽ bị mất kết nối mạng nội bộ do Namespaces tự động đóng các cổng kết nối mạng để bảo vệ hệ thống.
[EXP]
Namespaces không tự động đóng các cổng kết nối mạng khi phát hiện có lỗ hổng Kernel.
[C]
Hacker trong container có thể khai thác lỗi để "vượt ngục" (Container Escape) chiếm quyền điều khiển toàn bộ máy chủ host.
[EXP]
Chính xác. Vì container dùng chung nhân Kernel của host (Shared Kernel), nếu Kernel có lỗ hổng, hacker có thể leo thang đặc quyền để thoát ra khỏi container và kiểm soát máy host vật lý.
[D]
Ứng dụng Spring Boot trong container sẽ tự động biên dịch lại mã nguồn để tương thích với phiên bản vá lỗi của Kernel.
[EXP]
Ứng dụng Spring Boot không thể tự biên dịch lại mã nguồn khi phát hiện lỗ hổng ở tầng nhân hệ điều hành máy host.


@correct: C
@point: 20

---

# LESSON 02: Docker image, container và registry

## Q1

Dưới góc nhìn của lập trình hướng đối tượng (OOP), mối quan hệ tương quan giữa Docker Image và Docker Container được mô tả như thế nào?

[A]
Docker Image tương tự như một Object (Thực thể sống); Docker Container tương tự như một Class (Bản thiết kế tĩnh).
[EXP]
Phát biểu bị ngược. Image mới là bản thiết kế tĩnh, còn Container mới là thực thể chạy động trong bộ nhớ.
[B]
Docker Image tương tự như một Class (Bản thiết kế tĩnh); Docker Container tương tự như một Object (Thực thể sống).
[EXP]
Đúng. Docker Image giống Class trong OOP: chỉ đọc và đóng vai trò làm bản thiết kế. Docker Container giống Object: thực thể sống được khởi tạo từ Class đó.
[C]
Docker Image tương tự như một Method (Phương thức); Docker Container tương tự như một Attribute (Thuộc tính).
[EXP]
Khái niệm method và attribute không phản ánh đúng quan hệ bản thiết kế tĩnh và thực thể chạy động của Image và Container.
[D]
Docker Image tương tự như một Interface (Giao diện); Docker Container tương tự như một Package (Gói mã nguồn).
[EXP]
Khái niệm interface và package hoàn toàn không phản ánh mối quan hệ giữa Image và Container.


@correct: B
@point: 20

## Q2

Quy trình phân phối và khởi chạy một ứng dụng Docker từ kho lưu trữ (Docker Registry) về máy host được thực hiện như thế nào?

[A]
Tải (pull) Image từ Registry về máy host -> Khởi chạy (run) Container từ Image đó để tạo tiến trình ứng dụng.
[EXP]
Đây là luồng hoạt động chính xác. Ta dùng lệnh pull để kéo bản thiết kế (Image) về, rồi dùng lệnh run để tạo thực thể container chạy trên RAM/CPU.
[B]
Tải mã nguồn về -> Biên dịch ra file JAR -> Đóng gói thành Image -> Chạy container trực tiếp trên máy chủ.
[EXP]
Quy trình này mô tả quá trình build và chạy ở máy local hoặc CI server, không phải quy trình phân phối và chạy từ Registry về server.
[C]
Tải file đóng gói JAR về máy -> Gửi yêu cầu qua REST API -> Daemon tự động chạy ứng dụng mà không cần Image.
[EXP]
Docker bắt buộc phải sử dụng Image để khởi chạy container, không chạy trực tiếp file JAR mà không thông qua Image.
[D]
Khởi chạy Container trực tiếp từ Registry -> Registry tự động giải nén và nạp tiến trình vào RAM của máy host.
[EXP]
Hệ thống không chạy trực tiếp container từ xa mà bắt buộc phải tải Image về lưu trữ cục bộ ở máy host trước khi khởi chạy.


@correct: A
@point: 20

## Q3

Do đặc tính bất biến (Immutability) của Docker Image, chuyện gì sẽ xảy ra nếu lập trình viên truy cập vào container đang chạy và thay đổi một file cấu hình hệ thống?

[A]
Thay đổi đó được áp dụng ngay lập tức vào Docker Image gốc để tất cả các container sau này đều nhận được cấu hình mới.
[EXP]
Docker Image là chỉ đọc (Read-only) và bất biến, không bao giờ tự động cập nhật khi bạn thay đổi file trong container.
[B]
Thay đổi đó sẽ chỉ tồn tại trên lớp Writable Layer của riêng container đó và biến mất vĩnh viễn khi container bị xóa.
[EXP]
Chính xác. Mọi thao tác ghi/sửa đổi file trong lúc container đang chạy chỉ được ghi lên lớp Writable Layer tạm thời của container đó và sẽ biến mất khi xóa container.
[C]
Docker Engine sẽ tự động báo lỗi biên dịch và dừng hoạt động của container để bảo vệ tính toàn vẹn của Image.
[EXP]
Docker Engine hoàn toàn cho phép ghi vào Writable Layer và không báo lỗi biên dịch hay dừng container khi có thay đổi file.
[D]
Lớp Writable Layer sẽ tự động tạo một phiên bản backup của Image trên Docker Hub để lập trình viên tải về.
[EXP]
Docker không tự động backup hay đẩy các thay đổi trong container lên Docker Hub.


@correct: B
@point: 20

## Q4

Điểm khác biệt cơ bản giữa Public Registry (như Docker Hub) và Private Registry của doanh nghiệp là gì?

[A]
Public Registry chỉ hỗ trợ lưu trữ các image hệ điều hành; Private Registry chỉ hỗ trợ lưu trữ các file thực thi JAR.
[EXP]
Cả hai loại Registry đều hỗ trợ lưu trữ bất kỳ loại Docker Image nào, không phân biệt hệ điều hành hay ứng dụng Java.
[B]
Public Registry yêu cầu phí bản quyền khi tải image; Private Registry hoàn toàn miễn phí cho tất cả mọi người sử dụng.
[EXP]
Docker Hub (Public Registry) cho phép tải miễn phí hàng triệu image công cộng. Private Registry thường tốn chi phí quản lý của doanh nghiệp.
[C]
Public Registry tự động cập nhật code từ Git; Private Registry đòi hỏi con người phải trực tiếp biên dịch mã nguồn thủ công.
[EXP]
Quy trình cập nhật code từ Git thuộc về CI/CD pipeline, không phải tính chất phân biệt giữa Public và Private Registry.
[D]
Public Registry chứa các image chính thức công cộng; Private Registry giới hạn quyền truy cập để bảo mật code nội bộ.
[EXP]
Chính xác. Public Registry dành cho cộng đồng chia sẻ image công khai. Private Registry được doanh nghiệp dựng riêng để bảo mật các image chứa mã nguồn thương mại của dự án.


@correct: D
@point: 20

## Q5

Một lập trình viên khởi chạy một container PostgreSQL từ image gốc, lưu vào đó 100 thông tin khách hàng. Sau đó, lập trình viên chạy lệnh xóa container này đi (`docker rm`) và chạy lệnh tạo một container PostgreSQL mới từ image ban đầu. Dự đoán số phận của 100 dữ liệu khách hàng đó.

[A]
Ứng dụng sẽ tự động sử dụng giá trị mặc định là chuỗi rỗng để tiến hành kết nối đến cơ sở dữ liệu.
[EXP]
Phát biểu này mô tả cấu hình placeholder của Spring Boot, không liên quan đến việc lưu trữ dữ liệu PostgreSQL.
[B]
100 dữ liệu khách hàng vẫn tồn tại vì Docker Engine tự động tạo cơ chế đồng bộ dữ liệu giữa các container cùng tên.
[EXP]
Docker không tự động đồng bộ dữ liệu giữa các container mới tạo từ cùng một image trừ khi cấu hình volume dùng chung.
[C]
100 dữ liệu khách hàng bị mất hoàn toàn vì dữ liệu chỉ được ghi tạm thời trên lớp Writable Layer của container bị xóa.
[EXP]
Đúng. Do dữ liệu ghi vào Writable Layer của container cũ, khi xóa container (`docker rm`) thì lớp ghi này bị hủy bỏ theo, dẫn đến mất dữ liệu.
[D]
100 dữ liệu khách hàng bị mất hoàn toàn nhưng có thể khôi phục từ thư mục tạm `/tmp` của hệ điều hành Linux máy host.
[EXP]
Dữ liệu của container không nằm trong `/tmp` của host và không thể khôi phục một cách đơn giản như vậy khi container đã bị xóa.


@correct: C
@point: 20

---

# LESSON 03: Cài đặt Docker và kiểm tra môi trường

## Q1

Trong kiến trúc Client-Server của Docker Engine, tiến trình `dockerd` (Docker Daemon) đóng vai trò gì?

[A]
Là giao diện dòng lệnh giúp người dùng nhập lệnh và gửi yêu cầu đến server.
[EXP]
Đây là vai trò của Docker CLI (Client), không phải của Docker Daemon (`dockerd`).
[B]
Là cầu nối trung gian định nghĩa các interface giao tiếp REST API của hệ thống.
[EXP]
REST API chỉ là giao thức/cầu nối truyền thông chứ không phải là tiến trình Docker Daemon trực tiếp quản lý hệ thống.
[C]
Chịu trách nhiệm trực tiếp quản lý, tải image, khởi tạo container và phân chia mạng.
[EXP]
Chính xác. Docker Daemon (`dockerd`) là tiến trình chạy ngầm đóng vai trò là "trái tim" của hệ thống, thực thi mọi thao tác quản lý tài nguyên của Docker.
[D]
Là trình soạn thảo văn bản giúp lập trình viên sửa đổi các file cấu hình hệ thống.
[EXP]
Trình soạn thảo văn bản trên Linux là các công cụ như `nano` hay `vi`, không liên quan đến Docker Daemon.


@correct: C
@point: 20

## Q2

Khi bạn chạy lệnh `docker run hello-world` lần đầu tiên trên một máy chủ Linux vừa được cài đặt Docker, luồng hoạt động nội bộ nào dưới đây diễn ra ĐÚNG?

[A]
CLI gửi yêu cầu -> Daemon kiểm tra local -> Không thấy image -> Tải image từ Docker Hub -> Khởi chạy container.
[EXP]
Đây là luồng hoạt động chuẩn xác: Daemon kiểm tra bộ nhớ local không có image sẽ tự động liên kết lên Docker Hub để pull về, sau đó mới khởi chạy container.
[B]
CLI gửi yêu cầu -> Daemon tải trực tiếp từ Docker Hub -> Ghi đè vào local -> Dừng tiến trình và báo lỗi cú pháp.
[EXP]
Daemon chỉ tải từ Docker Hub khi local không có, và quá trình chạy hello-world thành công sẽ in thông tin chào mừng chứ không báo lỗi cú pháp.
[C]
CLI kiểm tra local -> Tải trực tiếp từ Docker Hub -> Gửi file JAR cho Daemon -> Daemon giải nén và khởi chạy.
[EXP]
Việc kiểm tra cục bộ và tải image do Daemon thực hiện thông qua REST API, CLI không trực tiếp tải và không gửi file JAR cho Daemon.
[D]
CLI gửi yêu cầu -> Daemon kiểm tra local -> Báo lỗi `Unable to find image` và dừng toàn bộ tiến trình hệ thống.
[EXP]
Khi không tìm thấy image cục bộ, Daemon sẽ tiến hành pull từ Docker Hub chứ không báo lỗi dừng tiến trình hệ thống.


@correct: A
@point: 20

## Q3

Để kích hoạt bộ quản lý dịch vụ `systemd` trên phiên bản Linux Ubuntu chạy trong WSL 2 trên Windows, lập trình viên cần làm gì?

[A]
Thêm dòng lệnh `export SYSTEMD=true` vào cuối file cấu hình `.bashrc` của tài khoản root hệ thống Linux.
[EXP]
Khai báo biến môi trường trong `.bashrc` không thể kích hoạt dịch vụ quản lý hệ thống `systemd` của WSL 2.
[B]
Chạy lệnh `sudo systemctl start systemd` trực tiếp trên cửa sổ Terminal của Ubuntu trong môi trường WSL.
[EXP]
Lệnh này không thể chạy được nếu cấu hình khởi động của WSL 2 chưa cho phép kích hoạt dịch vụ `systemd` từ file cấu hình hệ thống.
[C]
Khởi động lại máy tính Windows để hệ điều hành tự động cập nhật cấu hình dịch vụ chạy ngầm cho WSL.
[EXP]
Khởi động lại Windows không tự động kích hoạt `systemd` nếu bạn chưa tạo file cấu hình `/etc/wsl.conf` tương ứng.
[D]
Cấu hình dòng `systemd=true` trong file `/etc/wsl.conf` ở Ubuntu, sau đó chạy `wsl --shutdown` ở PowerShell Windows.
[EXP]
Đây là giải pháp chính xác. Bạn ghi cấu hình `systemd=true` vào `/etc/wsl.conf` để yêu cầu WSL khởi chạy systemd khi boot, sau đó tắt hẳn WSL qua PowerShell để nạp cấu hình mới.


@correct: D
@point: 20

## Q4

Tại sao trong thực tế vận hành DevOps, lập trình viên bắt buộc phải cài đặt **Docker Engine** gốc thay vì cài **Docker Desktop** trên các máy chủ Production chạy Cloud?

[A]
Docker Desktop chỉ hỗ trợ chạy ứng dụng NodeJS, còn Docker Engine gốc mới chạy được ứng dụng Java Spring Boot.
[EXP]
Cả hai phiên bản đều hỗ trợ chạy bất kỳ loại container nào (NodeJS, Java, Python...), đây không phải là lý do phân biệt.
[B]
Docker Desktop đòi hỏi giao diện đồ họa GUI và tiêu tốn nhiều tài nguyên, không phù hợp cho server Cloud tối giản.
[EXP]
Chính xác. Server Cloud chạy Linux thường là bản tối giản không có giao diện đồ họa (chỉ dùng CLI). Việc cài Docker Desktop sẽ gây lãng phí lớn tài nguyên CPU/RAM để chạy các dịch vụ giao diện và máy ảo không cần thiết.
[C]
Docker Engine gốc tự động tối ưu hóa bộ nhớ RAM cho hệ điều hành khách mà không cần thiết lập file cấu hình.
[EXP]
Docker Engine gốc chạy trực tiếp trên Host OS và không có hệ điều hành khách (Guest OS) nào cần quản lý tối ưu hóa RAM.
[D]
Docker Engine gốc hỗ trợ kết nối trực tiếp đến PostgreSQL vật lý mà không cần thông qua lớp mạng nội bộ.
[EXP]
Docker Engine gốc vẫn sử dụng cơ chế mạng nội bộ của container để liên kết cơ sở dữ liệu, đảm bảo bảo mật.


@correct: B
@point: 20

## Q5

Khi gõ lệnh `docker ps` trên Terminal của Ubuntu Linux, hệ thống trả về lỗi: `Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?`. Dự đoán nguyên nhân gây ra lỗi này.

[A]
Do Docker Daemon (`dockerd`) chưa được khởi động hoặc tài khoản chạy lệnh chưa có quyền truy cập Unix socket.
[EXP]
Đây là nguyên nhân chính xác. Lỗi này xảy ra khi Client không tìm thấy tiến trình Daemon đang hoạt động, hoặc bạn chạy lệnh bằng user thường chưa được phân quyền vào nhóm `docker`.
[B]
Do Docker CLI bị hỏng hoặc viết sai cú pháp lệnh và cần phải cài đặt lại phiên bản Docker Client mới nhất.
[EXP]
Docker CLI vẫn hoạt động bình thường mới in ra được thông báo lỗi kết nối Unix socket này.
[C]
Do máy chủ Linux bị mất kết nối internet và không thể truy cập vào kho lưu trữ Docker Hub để đồng bộ thông tin.
[EXP]
Lệnh `docker ps` chỉ liệt kê các container cục bộ đang chạy trên máy host, không yêu cầu kết nối internet lên Docker Hub.
[D]
Do cổng mạng `8080` của máy host đang bị chiếm dụng bởi một dịch vụ Java Spring Boot chạy ngầm khác.
[EXP]
Xung đột cổng `8080` của ứng dụng Spring Boot không ảnh hưởng đến kết nối Unix socket `/var/run/docker.sock` của Docker Daemon.


@correct: A
@point: 20

---

# LESSON 04: Các lệnh Docker cơ bản trong vòng đời container

## Q1

Khi khởi chạy container bằng lệnh `docker run`, tham số nào dưới đây giúp ánh xạ cổng mạng của máy host vào cổng mạng nội bộ của container?

[A]
Tham số `-d` (Detached mode)
[EXP]
Cờ `-d` dùng để chạy container dưới nền (background) giải phóng Terminal, không có vai trò cấu hình cổng mạng.
[B]
Tham số `-p host_port:container_port`
[EXP]
Chính xác. Tham số `-p` (Port mapping) giúp định tuyến dữ liệu từ cổng bên ngoài máy host vào cổng dịch vụ bên trong container.
[C]
Tham số `-e KEY=VALUE`
[EXP]
Tham số `-e` dùng để truyền biến môi trường vào bên trong container khi khởi chạy, không cấu hình cổng mạng.
[D]
Tham số `--name [tên]`
[EXP]
Tham số `--name` dùng để đặt tên tường minh cho container thay vì để Docker tự sinh tên ngẫu nhiên.


@correct: B
@point: 20

## Q2

Vòng đời hoạt động của một container trải qua các trạng thái tuần tự nào dưới đây?

[A]
Running (Đang hoạt động) -> Created (Đã tạo) -> Stopped (Đã dừng) -> Destroyed (Đã xóa).
[EXP]
Container không thể ở trạng thái Running trước khi được khởi tạo ở trạng thái Created.
[B]
Created (Đã tạo) -> Stopped (Đã dừng) -> Running (Đang hoạt động) -> Destroyed (Đã xóa).
[EXP]
Container không thể chuyển trực tiếp từ Stopped sang Destroyed rồi mới quay lại hoạt động hoặc tắt tiến trình.
[C]
Created (Đã tạo) -> Running (Đang hoạt động) -> Stopped (Đã dừng) -> Destroyed (Đã xóa).
[EXP]
Đúng. Tiến trình vòng đời chuẩn: Container được tạo cấu hình (`Created`) -> kích hoạt chạy tiến trình (`Running`) -> tắt tiến trình chính (`Stopped`) -> xóa bỏ file khỏi đĩa (`Destroyed`).
[D]
Running (Đang hoạt động) -> Stopped (Đã dừng) -> Created (Đã tạo) -> Destroyed (Đã xóa).
[EXP]
Lập trình viên không thể chạy container khi nó chưa được tạo (Created) trên hệ thống.


@correct: C
@point: 20

## Q3

Bạn Intern khởi chạy container PostgreSQL bằng lệnh sau:

```bash
docker run -d --name quickbite-db postgres:15-alpine
```

Tại sao ứng dụng Spring Boot chạy trực tiếp ở máy local lại gặp lỗi Connection Refused khi kết nối vào database này ở cổng 5432?

[A]
Do thiếu tham số `-d` khiến container bị tắt ngay khi bạn Intern đóng Terminal.
[EXP]
Bạn Intern đã dùng cờ `-d` nên container vẫn chạy dưới nền khi đóng Terminal, lỗi kết nối không do cờ này.
[B]
Do thiếu tham số `-e` để thiết lập mật khẩu đăng nhập mặc định cho cơ sở dữ liệu Postgres.
[EXP]
Thiếu mật khẩu có thể gây lỗi xác thực thông tin đăng nhập (Auth Failed) chứ không gây lỗi từ chối kết nối (Connection Refused).
[C]
Do Docker Daemon tự động chặn kết nối từ máy local để bảo vệ an toàn cho container.
[EXP]
Docker Daemon không tự động chặn kết nối nếu lập trình viên đã cấu hình ánh xạ cổng mạng chính xác.
[D]
Do thiếu tham số `-p` để ánh xạ cổng `5432` từ máy host vào cổng `5432` nội bộ của container.
[EXP]
Đúng. Vì không có cờ `-p 5432:5432`, cổng 5432 của PostgreSQL chỉ mở trong mạng nội bộ cô lập của container. Ứng dụng chạy trực tiếp ở máy host không thể tìm thấy cổng này.


@correct: D
@point: 20

## Q4

Sự khác biệt mấu chốt giữa trạng thái Stopped (đã dừng) và Destroyed (đã xóa) của một container là gì?

[A]
Trạng thái Stopped vẫn giữ lại dữ liệu ghi tạm thời trên ổ đĩa; trạng thái Destroyed xóa sạch hoàn toàn container khỏi hệ thống.
[EXP]
Chính xác. Khi stop container, tiến trình dừng và giải phóng RAM/CPU nhưng Writable Layer vẫn nằm trên ổ đĩa máy host. Khi chạy lệnh `docker rm` (Destroyed), lớp Writable Layer này bị xóa vĩnh viễn.
[B]
Trạng thái Stopped giải phóng hoàn toàn bộ nhớ ổ đĩa; trạng thái Destroyed vẫn giữ lại cấu hình cổng mạng trên máy host.
[EXP]
Phát biểu bị ngược. Stopped giữ lại ổ đĩa, còn Destroyed giải phóng hoàn toàn cả cấu hình cổng và ổ đĩa của container.
[C]
Trạng thái Stopped chỉ dùng cho container PostgreSQL; trạng thái Destroyed dùng cho mọi loại container khác.
[EXP]
Cả hai trạng thái Stopped và Destroyed đều áp dụng cho tất cả các loại container chạy trên Docker.
[D]
Trạng thái Stopped yêu cầu quyền root để khôi phục; trạng thái Destroyed có thể được khôi phục bởi bất kỳ tài khoản nào.
[EXP]
Khi container đã bị xóa (Destroyed), không tài khoản nào (kể cả root) có thể khôi phục lại trực tiếp nếu không có bản backup dữ liệu trước đó.


@correct: A
@point: 20

## Q5

Việc sử dụng cờ `-f` để xóa cưỡng chế một container đang hoạt động (`docker rm -f`) có thể dẫn đến rủi ro nào đối với hệ thống?

[A]
Làm hỏng toàn bộ các Docker Image gốc đang được lưu trữ cục bộ trên ổ đĩa của máy host.
[EXP]
Xóa container không làm ảnh hưởng hay hỏng các Docker Image gốc (chỉ đọc) lưu trữ ở local.
[B]
Gây mất dữ liệu chưa kịp ghi hoặc hỏng file do tiến trình bị tắt đột ngột (tương tự lệnh kill -9).
[EXP]
Đúng. Lệnh `docker rm -f` sử dụng tín hiệu SIGKILL để buộc dừng khẩn cấp tiến trình của container mà không cho phép ứng dụng kịp đóng kết nối và ghi dữ liệu dở dang lên ổ đĩa.
[C]
Gây sập tiến trình Docker Daemon và yêu cầu phải cài đặt lại Docker Engine từ đầu.
[EXP]
Docker Daemon vẫn hoạt động bình thường khi xử lý lệnh xóa cưỡng chế container, không bị crash hệ thống.
[D]
Làm mất các cấu hình biến môi trường vĩnh viễn đã được ghi trong file `.bashrc` của user.
[EXP]
Biến môi trường trong `.bashrc` nằm trên hệ điều hành host, hoàn toàn độc lập và không bị ảnh hưởng bởi việc xóa container.


@correct: B
@point: 20

---

# LESSON 05: Kiểm tra log và truy cập container (logs, exec)

## Q1

Khi sử dụng lệnh `docker exec -it` để truy cập vào shell của container đang chạy, ý nghĩa cụ thể của cờ `-it` là gì?

[A]
`-i` để tự động hóa việc khởi chạy; `-t` để chỉ định múi giờ cho container.
[EXP]
Cờ này không có chức năng thiết lập múi giờ hay tự động hóa quá trình khởi chạy container.
[B]
`-i` để giữ luồng nhập dữ liệu mở; `-t` để cung cấp một Terminal ảo.
[EXP]
Chính xác. `-i` (interactive) giữ luồng nhập dữ liệu STDIN luôn mở để nhận phím gõ từ bàn phím. `-t` (tty) cấp một Terminal ảo để hiển thị shell tương tác.
[C]
`-i` để bỏ qua các kiểm thử tự động; `-t` để kiểm tra cổng kết nối mạng.
[EXP]
Cờ `-it` chỉ phục vụ giao tiếp Terminal, không liên quan đến việc kiểm thử mã nguồn hay cấu hình mạng.
[D]
`-i` để chạy container dưới nền; `-t` để đặt tên tường minh cho tiến trình.
[EXP]
Chạy dưới nền là cờ `-d`, còn đặt tên là cờ `--name`, cờ `-it` không thực hiện các chức năng này.


@correct: B
@point: 20

## Q2

Docker Daemon tự động thu thập và lưu trữ log của container từ các luồng đầu ra nào của tiến trình chính?

[A]
Luồng dữ liệu vào (`STDIN`) và luồng cấu hình biến môi trường (`ENV`).
[EXP]
`STDIN` là luồng nhập từ bàn phím và `ENV` là biến môi trường, không phải luồng đầu ra lưu trữ log của ứng dụng.
[B]
Luồng ghi nhận lỗi hệ thống (`SYS_ERR`) và luồng kiểm tra cổng mạng (`PORT`).
[EXP]
Hệ thống log của container không lưu trữ riêng luồng kiểm tra cổng mạng.
[C]
Luồng đầu ra tiêu chuẩn (`STDOUT`) và luồng báo lỗi tiêu chuẩn (`STDERR`).
[EXP]
Chính xác. Docker Daemon bắt và ghi nhận toàn bộ thông tin xuất ra từ hai luồng mặc định là `STDOUT` (Standard Output) và `STDERR` (Standard Error) của tiến trình chính.
[D]
Luồng ghi đè file (`WRITE`) và luồng kết nối cơ sở dữ liệu (`DB_LOG`).
[EXP]
Docker Daemon không bắt log từ luồng ghi đè file trực tiếp trên ổ đĩa của ứng dụng.


@correct: C
@point: 20

## Q3

Giả sử lập trình viên chạy lệnh sau trên Terminal của máy host:

```bash
docker exec quickbite-db ls -la /
```

Phát biểu nào sau đây mô tả đúng kết quả trả về của lệnh này?

[A]
Hiển thị danh sách các thư mục bên trong container ra Terminal máy host rồi kết thúc lệnh.
[EXP]
Đúng. Lệnh `ls -la /` không yêu cầu tương tác nhập liệu từ bàn phím. Do đó, Docker sẽ chạy lệnh, in kết quả ra màn hình máy host rồi tắt tiến trình shell phụ ngay lập tức mà không cần cờ `-it`.
[B]
Hệ thống báo lỗi thiếu cờ `-it` và chặn không cho phép thực hiện lệnh hiển thị file.
[EXP]
Docker hoàn toàn cho phép thực hiện các lệnh không tương tác mà không bắt buộc phải truyền cờ `-it`.
[C]
Tự động mở một shell tương tác và giữ Terminal luôn mở để lập trình viên gõ phím.
[EXP]
Thiếu cờ `-it` và lệnh truyền vào là `ls -la /` (chạy xong tự tắt) nên hệ thống không thể duy trì shell tương tác.
[D]
Hiển thị danh sách thư mục của máy host thay vì hiển thị thư mục của container.
[EXP]
Lệnh `docker exec` bắt buộc chạy tiến trình bên trong container, kết quả hiển thị phải là cấu trúc thư mục của container.


@correct: A
@point: 20

## Q4

Sự khác biệt bản chất giữa hai lệnh `docker run -it [image] sh` và `docker exec -it [container] sh` là gì?

[A]
`docker run` chạy trên container đã dừng; `docker exec` tạo một container hoàn toàn mới từ Registry.
[EXP]
Phát biểu bị ngược. `docker run` tạo container mới, còn `docker exec` chạy lệnh trên container đang hoạt động.
[B]
`docker run` bắt buộc phải chạy dưới quyền root; `docker exec` có thể chạy bởi user thường không cần sudo.
[EXP]
Cả hai lệnh đều có thể chạy bằng user thường nếu user đó đã được phân quyền vào nhóm `docker` từ trước.
[C]
`docker run` chỉ dùng cho các ứng dụng Java; `docker exec` dùng riêng cho các ứng dụng cơ sở dữ liệu.
[EXP]
Cả hai lệnh đều dùng được cho bất kỳ loại image/container nào (Java, Database, Web server...).
[D]
`docker run` tạo một container mới tinh từ image; `docker exec` chạy shell trên container đang hoạt động.
[EXP]
Chính xác. `docker run` luôn khởi tạo một container mới từ image gốc. `docker exec` chỉ tạo một tiến trình phụ chạy bên trong một container có sẵn đang hoạt động.


@correct: D
@point: 20

## Q5

Khi bạn đang ở trong shell tương tác của container được tạo bởi lệnh `docker exec -it quickbite-db sh`, chuyện gì xảy ra nếu bạn gõ lệnh `exit`?

[A]
Container `quickbite-db` sẽ bị dừng hoạt động (stop) ngay lập tức vì tiến trình chính bị tắt.
[EXP]
Lệnh `exit` chỉ tắt tiến trình shell phụ do `exec` sinh ra, tiến trình chính (PostgreSQL) của container vẫn chạy bình thường.
[B]
Tiến trình shell con sẽ tắt và bạn quay về shell máy host, container chính vẫn hoạt động bình thường.
[EXP]
Chính xác. `docker exec` tạo tiến trình con chạy song song. Khi gõ `exit`, bạn chỉ thoát và kết liễu tiến trình shell con đó mà không ảnh hưởng đến container chính.
[C]
Toàn bộ dữ liệu của cơ sở dữ liệu PostgreSQL bên trong container sẽ bị xóa sạch khỏi ổ đĩa.
[EXP]
Thoát shell không thực hiện việc xóa dữ liệu trên lớp Writable Layer của container.
[D]
Docker Engine sẽ tự động kích hoạt tiến trình OOM Killer để tắt dịch vụ Docker Daemon chạy ngầm.
[EXP]
OOM Killer chỉ kích hoạt khi cạn kiệt RAM vật lý, việc thoát shell tương tác không thể kích hoạt tiến trình này.


@correct: B
@point: 20
