# QUIZ LESSONS - SESSION 10

# LESSON 01: Giới thiệu VPS và thiết lập kết nối cơ bản qua Bitvise Client

## Q1

Mô hình máy chủ ảo VPS (Virtual Private Server) có đặc điểm nào dưới đây so với dịch vụ Cloud Server (như AWS EC2)?

[A]
Có khả năng tự động co giãn tài nguyên (Auto-scaling) cực kỳ linh hoạt theo lưu lượng truy cập thực tế.
[EXP]
Sai. Đây là đặc điểm nổi bật của Cloud Server nhờ tính năng tự động co giãn linh hoạt, chứ không phải VPS truyền thống.
[B]
Được cài đặt sẵn toàn bộ các thành phần runtime của ứng dụng Spring Boot và cơ sở dữ liệu khi khởi tạo.
[EXP]
Sai. Cả VPS và Cloud Server thường chỉ cung cấp một hệ điều hành sạch (clean OS) khi mới khởi tạo, người dùng phải tự cài đặt và cấu hình môi trường.
[C]
Cung cấp tài nguyên phần cứng cố định trên một máy chủ vật lý (Physical Node) và khó thực hiện nâng cấp tài nguyên mà không gây gián đoạn dịch vụ.
[EXP]
Chính xác. VPS được phân hoạch tài nguyên cố định trên một node vật lý nhất định. Việc nâng cấp tài nguyên thường yêu cầu phải khởi động lại (restart) máy chủ.
[D]
Bị giới hạn hoàn toàn quyền truy cập quản trị hệ điều hành (Root OS) và không thể can thiệp sâu vào nhân hệ điều hành (Kernel).
[EXP]
Sai. VPS vẫn cấp toàn quyền quản trị cao nhất (root) cho người sử dụng để cấu hình và tùy biến hệ thống.

@correct: C
@point: 20

## Q2

Khi sử dụng các công cụ dòng lệnh (như PowerShell hoặc Terminal) ở máy tính cá nhân (local) để kết nối SSH vào VPS lần đầu tiên, hệ thống hiển thị thông điệp cảnh báo "The authenticity of host... can't be established". Lập trình viên nên xử lý như thế nào?

[A]
Nhập "yes" để xác nhận lưu vân tay khóa bảo mật (host key fingerprint) của máy chủ vào máy local.
[EXP]
Chính xác. Đây là cảnh báo an toàn tiêu chuẩn trong lần đầu kết nối. Nhập "yes" giúp lưu thông tin nhận diện của VPS vào tệp tin known_hosts ở máy local để kiểm tra ở các lần sau.
[B]
Ngắt kết nối lập tức và yêu cầu nhà cung cấp VPS cài lại toàn bộ hệ điều hành vì máy chủ đã bị xâm nhập.
[EXP]
Sai. Đây là thông báo bảo mật tiêu chuẩn khi kết nối lần đầu tới bất kỳ máy chủ SSH mới nào, không phải dấu hiệu bị tấn công.
[C]
Đổi cổng SSH mặc định từ cổng 22 sang một cổng khác ngẫu nhiên trên máy local để bỏ qua bước kiểm tra khóa.
[EXP]
Sai. Việc đổi cổng kết nối không giúp bỏ qua cảnh báo nhận diện fingerprint này của giao thức SSH.
[D]
Sử dụng tài khoản quản trị phụ thay thế cho tài khoản root để tránh hiển thị cảnh báo bảo mật hệ thống.
[EXP]
Sai. Cảnh báo này xuất hiện dựa trên địa chỉ IP và khóa nhận diện của host, hoàn toàn độc lập với tài khoản người dùng đăng nhập.

@correct: A
@point: 20

## Q3

Khi bạn không thể kết nối SSH vào một VPS mới được khởi tạo từ nhà cung cấp dịch vụ đám mây (Cloud Provider), bước kiểm tra cấu hình mạng nào sau đây là quan trọng và cần được ưu tiên hàng đầu?

[A]
Kiểm tra cấu hình Firewall / Security Group trên trang quản trị của nhà cung cấp dịch vụ để đảm bảo cổng 22 đã được mở cho địa chỉ IP của bạn.
[EXP]
Chính xác. Các nhà cung cấp đám mây thường áp dụng một lớp tường lửa bên ngoài (Security Group). Nếu cổng 22 chưa được mở tại đây, mọi yêu cầu kết nối SSH từ ngoài vào đều bị chặn trước khi tới được VPS.
[B]
Cài đặt lại toàn bộ hệ điều hành Ubuntu trên VPS vì mặc định các máy chủ mới không hỗ trợ giao thức SSH.
[EXP]
Sai. Hầu hết các bản phân phối Ubuntu Server dành cho VPS đều được cài đặt và kích hoạt sẵn dịch vụ SSH (openssh-server).
[C]
Khởi động lại máy tính cá nhân của bạn để cập nhật lại địa chỉ IP mạng nội bộ (Private IP).
[EXP]
Sai. Việc khởi động lại máy tính cá nhân không giải quyết được vấn đề cấu hình tường lửa phía máy chủ hay nhà cung cấp.
[D]
Gửi yêu cầu hỗ trợ kỹ thuật nhờ nhà cung cấp cấp lại một cổng kết nối khác thay thế cho cổng mặc định 22.
[EXP]
Sai. Cổng 22 là cổng tiêu chuẩn của SSH, sự cố không kết nối được là do cấu hình chặn cổng chứ không phải do bản thân số cổng 22 bị lỗi.

@correct: A
@point: 20

## Q4

Khi muốn sao chép nhanh một tệp tin (ví dụ: file cấu hình ứng dụng hoặc file chạy .jar) từ máy tính cá nhân (local) lên máy chủ VPS thông qua dòng lệnh, lập trình viên nên sử dụng công cụ/lệnh nào chạy trên nền tảng SSH?

[A]
Sử dụng lệnh cp (Copy) thông thường.
[EXP]
Sai. Lệnh cp chỉ dùng để sao chép tệp tin nội bộ giữa các thư mục trên cùng một máy chủ, không hỗ trợ truyền tải qua mạng.
[B]
Sử dụng lệnh scp (Secure Copy Protocol) hoặc giao thức truyền file SFTP (SSH File Transfer Protocol).
[EXP]
Chính xác. Cả scp và SFTP đều là các phương thức truyền tải tệp tin an toàn được xây dựng trên nền tảng giao thức SSH (chạy qua cổng 22 mặc định).
[C]
Sử dụng lệnh ftp (File Transfer Protocol) truyền thống.
[EXP]
Sai. FTP truyền thống truyền dữ liệu (bao gồm cả mật khẩu) dưới dạng văn bản rõ (clear text), không an toàn và không đi qua đường truyền mã hóa của SSH.
[D]
Sử dụng lệnh ping kết hợp với cờ truyền dữ liệu.
[EXP]
Sai. Lệnh ping sử dụng giao thức ICMP chỉ để kiểm tra trạng thái kết nối mạng, không có tính năng truyền tải tệp tin.

@correct: B
@point: 20

## Q5

Mục đích chính của việc chạy chuỗi lệnh sudo apt update && sudo apt upgrade -y ngay khi nhận bàn giao một VPS mới cài đặt hệ điều hành Ubuntu là gì?

[A]
Giải phóng dung lượng ổ cứng SSD bằng cách dọn dẹp các tệp tin log hệ thống sinh ra trong quá trình cài đặt OS.
[EXP]
Sai. Lệnh này dùng để tải và cập nhật phần mềm, thậm chí có thể làm tăng nhẹ dung lượng lưu trữ do cài đặt các bản vá lỗi và phiên bản mới hơn.
[B]
Đồng bộ hóa múi giờ hệ thống (Timezone) của VPS với múi giờ thực tế của máy tính cá nhân người quản trị.
[EXP]
Sai. Đồng bộ múi giờ yêu cầu cấu hình các công cụ quản lý thời gian như timedatectl, việc cập nhật gói phần mềm không tự động thay đổi múi giờ.
[C]
Đổi tên đăng nhập của tài khoản quản trị tối cao từ root sang một tên người dùng khác để tăng tính bảo mật.
[EXP]
Sai. Việc đổi tên tài khoản hoặc cấu hình phân quyền người dùng không nằm trong chức năng của hai lệnh quản lý gói phần mềm này.
[D]
Cập nhật danh sách gói phần mềm và tiến hành nâng cấp, vá các lỗ hổng bảo mật của các thư viện hệ thống hiện có trên hệ điều hành.
[EXP]
Chính xác. Các bản cài đặt OS mặc định của nhà cung cấp thường không chứa các bản vá mới nhất. Việc chạy chuỗi lệnh này giúp hệ thống cập nhật các bản vá bảo mật mới nhất để phòng ngừa mã độc và lỗ hổng bảo mật.

@correct: D
@point: 20

---

# LESSON 02: Tạo User mới và bảo mật VPS bằng cơ chế khóa SSH (SSH Hardening)

## Q1

Mục đích cốt lõi của việc áp dụng nguyên tắc đặc quyền tối thiểu (Least Privilege) bằng cách tạo tài khoản người dùng thường (ví dụ: deployer) thay vì sử dụng trực tiếp tài khoản root cho các hoạt động vận hành hàng ngày là gì?

[A]
Ngăn chặn hoàn toàn các cuộc tấn công dò quét mật khẩu (Brute-force) nhắm vào cổng kết nối SSH của VPS.
[EXP]
Sai. Tạo user mới chỉ hạn chế rủi ro phá hoại của tài khoản root. Để chống Brute-force triệt để cần tắt xác thực bằng mật khẩu hoặc sử dụng các công cụ như Fail2ban.
[B]
Giảm thiểu thiệt hại do lỗi thao tác ngoài ý muốn của quản trị viên và kiểm soát chặt chẽ các hành động nâng quyền thông qua lệnh sudo.
[EXP]
Chính xác. Thao tác bằng quyền root trực tiếp rất dễ gây ra lỗi hệ thống nghiêm trọng (như xóa nhầm file hệ thống). Dùng user thường và nâng quyền bằng sudo khi cần thiết giúp ghi nhận nhật ký (log) hành động và bảo vệ các thư mục quan trọng.
[C]
Tăng tốc độ xử lý các tác vụ biên dịch mã nguồn Java Spring Boot trên VPS nhờ giảm tải tài nguyên hệ thống.
[EXP]
Sai. Quyền hạn của tài khoản người dùng không ảnh hưởng trực tiếp đến hiệu năng tính toán của CPU hay bộ nhớ RAM khi chạy ứng dụng.
[D]
Tự động mã hóa toàn bộ dữ liệu lưu trữ trên ổ cứng để bảo vệ dữ liệu thô của ứng dụng.
[EXP]
Sai. Cơ chế phân quyền người dùng Linux không thực hiện chức năng mã hóa toàn bộ đĩa cứng vật lý (Full Disk Encryption).

@correct: B
@point: 20

## Q2

Trong cơ chế xác thực bằng khóa SSH Key sử dụng thuật toán mã hóa bất đối xứng, khóa riêng tư (Private Key) được lưu giữ ở đâu?

[A]
Được đưa lên cấu hình trong tệp tin ~/.ssh/authorized_keys` của tài khoản người dùng trên máy chủ VPS.
[EXP]
Sai. Tệp authorized_keys trên máy chủ VPS dùng để chứa khóa công khai (Public Key), không chứa khóa riêng tư.
[B]
Được lưu trữ an toàn và bảo mật trên máy tính cá nhân (local) của lập trình viên và tuyệt đối không chia sẻ cho bất kỳ ai.
[EXP]
Chính xác. Khóa riêng tư (Private Key) dùng để giải mã thử thách xác thực từ máy chủ gửi về và phải được bảo vệ tuyệt mật tại máy trạm (client).
[C]
Được tự động đăng ký và lưu trữ tập trung trên máy chủ quản lý mã nguồn GitHub của doanh nghiệp.
[EXP]
Sai. GitHub chỉ lưu trữ Public Key của bạn để xác thực quyền truy cập kho mã nguồn, không bao giờ yêu cầu hay lưu trữ Private Key.
[D]
Được đính kèm trực tiếp vào tệp tin cấu hình docker-compose.yml` để các dịch vụ container có thể sử dụng chung.
[EXP]
Sai. Không bao giờ được để lộ Private Key trong các tệp tin cấu hình ứng dụng hay mã nguồn dự án để tránh rò rỉ bảo mật nghiêm trọng.

@correct: B
@point: 20

## Q3

Để tạo người dùng mới tên là deployer và cấp quyền quản trị (quyền thực thi qua lệnh sudo) trên hệ điều hành Ubuntu Server, chuỗi lệnh nào sau đây là chính xác?

[A]
`sudo adduser deployer && sudo usermod -aG sudo deployer`
[EXP]
Chính xác. Lệnh adduser tạo người dùng mới cùng thư mục home, và usermod -aG sudo thêm người dùng đó vào nhóm quản trị sudo để có đặc quyền thực thi lệnh quản trị.
[B]
`sudo useradd deployer && sudo groupadd sudo deployer`
[EXP]
Sai. Lệnh groupadd dùng để tạo nhóm người dùng mới chứ không phải để thêm một người dùng vào nhóm có sẵn.
[C]
`sudo createuser deployer && sudo grant privilege to deployer`
[EXP]
Sai. Các lệnh này không phải cú pháp quản lý người dùng chuẩn của Linux (đây là cú pháp lệnh trong quản trị cơ sở dữ liệu SQL).
[D]
`sudo newuser deployer && sudo chmod +x /home/deployer`
[EXP]
Sai. Không có lệnh newuser mặc định trên Ubuntu, và chmod +x chỉ cấp quyền thực thi tệp tin chứ không cấp đặc quyền quản trị sudo.

@correct: A
@point: 20

## Q4

Sau khi hoàn tất thay đổi cấu hình bảo mật SSH Hardening trong tệp tin /etc/ssh/sshd_config, lập trình viên bắt buộc phải thực hiện lệnh nào dưới đây để cấu hình mới chính thức có hiệu lực?

[A]
`sudo systemctl restart ssh`
[EXP]
Chính xác. Dịch vụ SSH Daemon cần được khởi động lại (restart) hoặc tải lại cấu hình (reload) để áp dụng các thay đổi trong tệp sshd_config.
[B]
`sudo ufw reload`
[EXP]
Sai. Lệnh này dùng để tải lại cấu hình của tường lửa UFW, không tác động trực tiếp tới cấu hình của dịch vụ SSH.
[C]
`sudo apt update`
[EXP]
Sai. Lệnh này dùng để cập nhật danh sách các gói phần mềm từ kho lưu trữ, không liên quan đến việc nạp lại cấu hình dịch vụ hệ thống.
[D]
`newgrp docker`
[EXP]
Sai. Lệnh này dùng để áp dụng quyền nhóm docker cho phiên làm việc hiện tại, không liên quan đến dịch vụ SSH.

@correct: A
@point: 20

## Q5

Trong tệp cấu hình /etc/ssh/sshd_config, thiết lập nào sau đây giúp vô hiệu hóa hoàn toàn phương thức xác thực bằng mật khẩu thông thường để buộc người dùng phải đăng nhập qua SSH Key?

[A]
`PubkeyAuthentication no`
[EXP]
Sai. Thiết lập này sẽ tắt tính năng xác thực bằng khóa SSH Key, buộc phải đăng nhập bằng mật khẩu (đi ngược lại mục tiêu bảo mật).
[B]
`PermitRootLogin no`
[EXP]
Sai. Cấu hình này chặn đăng nhập trực tiếp bằng tài khoản root, nhưng không tắt cơ chế đăng nhập bằng mật khẩu của các tài khoản người dùng khác.
[C]
`PasswordAuthentication no`
[EXP]
Chính xác. Thiết lập PasswordAuthentication no ngăn chặn hoàn toàn việc đăng nhập bằng mật khẩu thông thường, chặn đứng các cuộc tấn công dò mật khẩu tự động.
[D]
`StrictModes no`
[EXP]
Sai. Cấu hình này kiểm tra quyền hạn của các tệp tin cấu hình thư mục cá nhân .ssh, không quản lý phương thức xác thực mật khẩu.

@correct: C
@point: 20

---

# LESSON 03: Thiết lập Tường lửa UFW và Cài đặt Docker trên Ubuntu Server

## Q1

Tại sao lập trình viên nên ưu tiên cài đặt Docker Engine từ Docker Repository chính thức của Docker thay vì sử dụng gói cài đặt mặc định (docker.io) trong kho lưu trữ của Ubuntu?

[A]
Vì gói cài đặt mặc định của Ubuntu yêu cầu cấu hình thêm các dịch vụ ảo hóa phụ trợ rất phức tạp trên máy chủ.
[EXP]
Sai. Gói docker.io cài đặt trực tiếp, vấn đề lớn nhất của nó là phiên bản thường cũ hơn và thiếu các tính năng mới.
[B]
Vì gói chính thức của Docker giúp tăng gấp đôi hiệu năng xử lý cơ sở dữ liệu của các container.
[EXP]
Sai. Hiệu năng ứng dụng phụ thuộc vào cấu hình hệ thống và tài nguyên phần cứng, nguồn cài đặt Docker không làm tăng hiệu năng ứng dụng trực tiếp như vậy.
[C]
Vì Docker Repository chính thức cung cấp phiên bản Docker Engine mới nhất, ổn định và nhận được các bản vá bảo mật kịp thời nhất từ chính nhà phát triển.
[EXP]
Chính xác. Việc dùng kho chính thức đảm bảo hệ thống luôn được cập nhật bản vá bảo mật và các tính năng mới nhất của Docker một cách đồng bộ.
[D]
Vì gói cài đặt mặc định của Ubuntu tự động kích hoạt tính năng chặn kết nối mạng nội bộ của toàn bộ container.
[EXP]
Sai. Gói docker.io vẫn cho phép các container giao tiếp mạng bình thường theo cấu hình mặc định.

@correct: C
@point: 20

## Q2

Sau khi thêm tài khoản người dùng deployer vào nhóm quản trị docker, tại sao lập trình viên cần chạy lệnh newgrp docker trong phiên làm việc (session) terminal hiện tại?

[A]
Để kích hoạt dịch vụ nền Docker Daemon bắt đầu chạy cùng hệ điều hành của máy chủ VPS.
[EXP]
Sai. Việc quản lý dịch vụ nền khởi chạy cần dùng lệnh quản lý dịch vụ hệ thống như sudo systemctl start docker.
[B]
Để áp dụng quyền hạn mới của nhóm docker cho phiên làm việc hiện tại mà không yêu cầu người dùng phải đăng xuất (logout) và đăng nhập lại.
[EXP]
Chính xác. Hệ điều hành Linux chỉ nạp lại quyền hạn của các nhóm người dùng khi bắt đầu một phiên đăng nhập mới. Lệnh newgrp docker giúp nạp lại quyền hạn ngay tại phiên hiện tại.
[C]
Để tự động tạo ra một card mạng ảo (Bridge Network) dùng chung cho cụm ứng dụng microservices.
[EXP]
Sai. Việc tạo mạng ảo được thực hiện qua lệnh docker network create hoặc định nghĩa trong cấu hình Docker Compose.
[D]
Để gán toàn quyền sở hữu vật lý tệp tin cấu hình docker-compose.yml` cho tài khoản người dùng deployer.
[EXP]
Sai. Lệnh này không thay đổi quyền sở hữu (owner/group owner) của các tệp tin trên ổ cứng.

@correct: B
@point: 20

## Q3

Nguyên tắc thiết lập tường lửa UFW (Uncomplicated Firewall) an toàn và bảo mật cho máy chủ VPS chạy môi trường Production là gì?

[A]
Cho phép toàn bộ kết nối đi vào mặc định (Default Allow Incoming) và chỉ cấu hình chặn một số cổng dịch vụ nhạy cảm như cổng database 5432.
[EXP]
Sai. Thiết lập này cực kỳ nguy hiểm vì các dịch vụ mới mở sau này sẽ bị lộ mặc định ra ngoài Internet nếu người quản trị quên cấu hình chặn.
[B]
Chặn toàn bộ lưu lượng đi ra ngoài Internet (Default Deny Outgoing) để tránh rò rỉ mã nguồn dự án.
[EXP]
Sai. Chặn toàn bộ kết nối đi ra sẽ khiến VPS không thể tải gói phần mềm, cập nhật hệ thống hoặc kết nối tới các dịch vụ bên ngoài như API hay GitHub.
[C]
Chặn toàn bộ kết nối đi vào mặc định (Default Deny Incoming), chỉ mở một số cổng dịch vụ thiết yếu đã được xác định trước (ví dụ cổng 22 cho SSH, cổng 80/443 cho web).
[EXP]
Chính xác. Đây là nguyên tắc bảo mật tiêu chuẩn (Zero Trust / Least Privilege ở tầng mạng). Chỉ cho phép những gì thực sự cần thiết, còn lại chặn toàn bộ mặc định để giảm thiểu diện tấn công.
[D]
Vô hiệu hóa hoàn toàn tường lửa UFW trên máy host và chuyển giao nhiệm vụ lọc gói tin cho các container tự quản lý.
[EXP]
Sai. Các container chạy trên Docker mặc định không tự tích hợp tường lửa lọc gói tin từ bên ngoài Internet. Máy chủ host luôn cần tường lửa bảo vệ.

@correct: C
@point: 20

## Q4

Điều gì sẽ xảy ra nếu lập trình viên thực hiện kích hoạt tường lửa bằng lệnh sudo ufw enable khi chưa thiết lập quy tắc cho phép kết nối cổng SSH (sudo ufw allow 22/tcp hoặc dịch vụ ssh)?

[A]
Hệ thống sẽ hiển thị cảnh báo lỗi cú pháp và từ chối chạy lệnh kích hoạt để bảo vệ phiên kết nối hiện tại.
[EXP]
Sai. UFW vẫn sẽ được kích hoạt bình thường theo yêu cầu mà không có cơ chế tự động ngăn chặn hành động này.
[B]
Phiên kết nối SSH hiện tại sẽ bị ngắt lập tức và lập trình viên bị khóa quyền truy cập vào VPS từ xa.
[EXP]
Chính xác. Do chính sách mặc định của tường lửa là chặn toàn bộ kết nối đi vào (Default Deny Incoming), việc kích hoạt tường lửa khi chưa mở cổng 22 sẽ chặn đứng mọi kết nối SSH đến máy chủ.
[C]
Ứng dụng Bitvise SSH Client sẽ tự động phát hiện sự cố và gửi yêu cầu cấu hình mở lại cổng 22 từ xa.
[EXP]
Sai. Bitvise SSH Client chỉ hoạt động ở máy local, không thể can thiệp ghi đè cấu hình tường lửa hệ thống khi kết nối đã bị chặn.
[D]
Hệ điều hành VPS sẽ tự động khởi động lại (restart) máy chủ để khôi phục cấu hình mạng ban đầu.
[EXP]
Sai. Hệ điều hành không tự động restart khi mất kết nối mạng. Cấu hình tường lửa đã kích hoạt sẽ được lưu trữ và áp dụng liên tục.

@correct: B
@point: 20

## Q5

Tại sao các container (ví dụ: container ứng dụng Spring Boot backend và database PostgreSQL) chạy chung một mạng ảo của Docker (Docker Network) vẫn giao tiếp với nhau bình thường dù tường lửa UFW trên máy chủ host đã cấu hình chặn cổng database (ví dụ cổng 5432 của PostgreSQL)?

[A]
Vì các container giao tiếp nội bộ thông qua card mạng ảo (Bridge Network) riêng do Docker quản lý, lưu lượng không đi qua các card mạng vật lý chịu sự kiểm soát trực tiếp của UFW.
[EXP]
Chính xác. Tường lửa UFW trên máy host mặc định lọc các gói tin đi qua card mạng vật lý kết nối với Internet. Giao tiếp mạng nội bộ giữa các container trong cùng Docker Network được định tuyến trực tiếp qua bridge của Docker nên không bị ảnh hưởng bởi các quy tắc chặn của UFW đối với card mạng ngoài.
[B]
Vì Docker Daemon tự động cấu hình ghi đè và bypass toàn bộ hệ thống tường lửa UFW của máy host đối với tất cả các cổng.
[EXP]
Sai. Docker chỉ can thiệp vào bảng định tuyến iptables cho các cổng được ánh xạ ra ngoài (Publish Port), giao tiếp thuần nội bộ giữa các container không cần đi qua bộ lọc của UFW.
[C]
Vì cơ sở dữ liệu PostgreSQL tự động đổi cổng giao tiếp nội bộ sang cổng khác khi phát hiện tường lửa UFW hoạt động.
[EXP]
Sai. Cơ sở dữ liệu vẫn chạy cố định ở cổng cấu hình (mặc định 5432), không tự động thay đổi cổng chạy dịch vụ.
[D]
Vì tường lửa UFW chỉ có hiệu lực bảo vệ đối với các ứng dụng viết bằng Java.
[EXP]
Sai. Tường lửa UFW hoạt động ở tầng mạng của hệ điều hành, hoàn toàn độc lập với ngôn ngữ lập trình của các ứng dụng chạy trên máy chủ.

@correct: A
@point: 20

---

# LESSON 04: Đồng bộ cấu hình qua Git bằng SSH Key và vận hành cụm Microservices

## Q1

Trong quy trình đồng bộ mã nguồn dự án từ GitHub về VPS, mục đích của việc sinh cặp khóa SSH Key trên máy chủ VPS là gì?

[A]
Để cho phép máy tính cá nhân của lập trình viên có thể kết nối thẳng vào thư mục làm việc trên VPS mà không cần mật khẩu.
[EXP]
Sai. Khóa dùng để kết nối từ máy local vào VPS phải được tạo ở máy local, chứ không phải tạo trên VPS.
[B]
Để làm khóa mã hóa bảo mật cho nội dung của các tệp tin cấu hình môi trường .env chạy trên VPS.
[EXP]
Sai. Khóa SSH Key dùng để xác thực kết nối truyền tải, không dùng để thực hiện mã hóa nội dung tệp tin cấu hình ứng dụng.
[C]
Để VPS xác thực danh tính với GitHub, từ đó cấp quyền tải (clone/pull) các kho mã nguồn riêng tư (Private Repository) của bạn về máy chủ.
[EXP]
Chính xác. Khóa công khai (Public Key) sinh ra trên VPS sẽ được đăng ký với GitHub nhằm cấp quyền cho VPS xác thực danh tính khi thực hiện các lệnh Git qua giao thức SSH.
[D]
Để tự động đồng bộ hóa các thay đổi mã nguồn trực tiếp từ máy tính cá nhân lên môi trường Production của VPS.
[EXP]
Sai. Việc tự động đồng bộ thay đổi yêu cầu các công cụ CI/CD hoặc webhook. Khóa SSH Key chỉ đóng vai trò là phương thức xác thực quyền truy cập kho mã nguồn.

@correct: C
@point: 20

## Q2

Khóa công khai (Public Key) sinh ra trên VPS để kết nối với GitHub cần được cấu hình ở vị trí nào sau đây trên giao diện GitHub?

[A]
Mục SSH and GPG keys trong phần Settings (Cài đặt) tài khoản GitHub cá nhân của bạn.
[EXP]
Chính xác. Bạn cần truy cập Settings tài khoản cá nhân trên GitHub -> SSH and GPG keys -> chọn New SSH key để thêm khóa công khai của máy chủ VPS vào danh sách xác thực.
[B]
Thư mục .ssh/authorized_keys` của tài khoản người dùng root trên máy chủ VPS.
[EXP]
Sai. Đây là nơi chứa Public Key của máy local để đăng nhập vào máy chủ VPS, không phải nơi cấu hình để VPS kết nối tới GitHub.
[C]
Mục Secrets and variables -> Actions trong phần Settings của Repository dự án trên GitHub.
[EXP]
Sai. Mục này dùng để lưu trữ các biến bảo mật cho luồng chạy tự động hóa (GitHub Actions), không phải nơi đăng ký khóa SSH xác thực tài khoản.
[D]
Ghi trực tiếp vào thuộc tính credentials của tệp tin cấu hình docker-compose.yml.
[EXP]
Sai. Tệp tin docker-compose.yml định nghĩa cấu hình chạy container, không chứa thông tin cấu hình xác thực SSH Key với GitHub.

@correct: A
@point: 20

## Q3

Nguyên tắc quản lý tệp tin cấu hình môi trường chứa thông tin nhạy cảm (như .env) trong các dự án phần mềm chuyên nghiệp là gì?

[A]
Khai báo tệp tin .env trong tệp cấu hình .gitignore để không bao giờ đẩy lên Git Repository, và tiến hành cấu hình thủ công tệp tin này trực tiếp trên từng máy chủ môi trường.
[EXP]
Chính xác. Tệp tin .env chứa các thông tin nhạy cảm và cấu hình riêng biệt của từng môi trường (mật khẩu database, mã khóa bảo mật JWT). Việc loại bỏ khỏi Git giúp tránh rò rỉ thông tin bảo mật và đảm bảo tính độc lập giữa các môi trường.
[B]
Đẩy tệp tin .env lên Git Repository chung để tất cả thành viên dự án và máy chủ VPS có cùng cấu hình đồng bộ.
[EXP]
Sai. Đẩy tệp tin cấu hình nhạy cảm lên Git sẽ gây rủi ro bảo mật lớn (lộ mật khẩu cơ sở dữ liệu) và khiến việc cấu hình khác nhau giữa các môi trường gặp khó khăn.
[C]
Cấu hình quyền đọc ghi công khai (ví dụ: chmod 777) cho tệp tin .env trên VPS để tất cả tiến trình đều truy cập được.
[EXP]
Sai. Tệp tin .env chứa thông tin nhạy cảm nên cần được giới hạn quyền đọc (chỉ cho phép tài khoản chạy ứng dụng đọc ghi) để tránh bị rò rỉ nội bộ.
[D]
Nhúng trực tiếp toàn bộ các giá trị cấu hình của tệp tin .env vào mã nguồn Java trước khi thực hiện đóng gói ứng dụng.
[EXP]
Sai. Cách làm này vi phạm nguyên tắc tách biệt cấu hình khỏi mã nguồn ứng dụng, khiến ứng dụng mất đi tính linh hoạt khi triển khai trên các môi trường khác nhau.

@correct: A
@point: 20

## Q4

Lệnh nào dưới đây giúp lập trình viên giám sát tài nguyên phần cứng (tỷ lệ CPU, dung lượng RAM tiêu thụ) của từng container đang chạy trên máy chủ VPS theo thời gian thực?

[A]
`docker stats`
[EXP]
Chính xác. Lệnh docker stats hiển thị một bảng thống kê cập nhật liên tục về hiệu năng sử dụng tài nguyên (CPU, RAM, Network I/O) của toàn bộ các container đang chạy trên máy chủ.
[B]
`docker compose ps`
[EXP]
Sai. Lệnh này hiển thị danh sách các dịch vụ kèm trạng thái hoạt động (Up/Down) và cổng chạy ứng dụng, không hiển thị dữ liệu tiêu thụ tài nguyên phần cứng.
[C]
`docker compose logs -f`
[EXP]
Sai. Lệnh này dùng để theo dõi liên tục nhật ký hoạt động (Console Logs) của các container, không đo đạc thông số tài nguyên.
[D]
`docker version`
[EXP]
Sai. Lệnh này hiển thị thông tin phiên bản chi tiết của phần mềm Docker Client và Docker Engine đang chạy trên máy chủ.

@correct: A
@point: 20

## Q5

Khi chạy ứng dụng Spring Boot bằng Docker Compose trên VPS, container của một dịch vụ Microservices đột ngột dừng hoạt động và hiển thị trạng thái Exit Code 137. Nguyên nhân phổ biến nhất của lỗi này là gì?

[A]
Cơ sở dữ liệu PostgreSQL bị sập khiến ứng dụng Spring Boot không thể kết nối và tự động ném ra lỗi dừng hệ thống.
[EXP]
Sai. Lỗi không kết nối được cơ sở dữ liệu thường làm ứng dụng ném ngoại lệ kết nối và dừng với mã Exit Code 1 thông thường.
[B]
Cổng chạy ứng dụng (ví dụ 8081) đang bị chiếm dụng bởi một tiến trình khác đang chạy ngoài máy host.
[EXP]
Sai. Lỗi trùng cổng chạy thường ném ra lỗi ngoại lệ BindException và trả về mã dừng ứng dụng Exit Code 1 ngay khi khởi động.
[C]
Hệ điều hành (Kernel) đã gửi tín hiệu cưỡng bức tắt container bằng lệnh SIGKILL (tín hiệu số 9) do hệ thống bị cạn kiệt bộ nhớ RAM vật lý (cơ chế OOM Killer).
[EXP]
Chính xác. Mã trạng thái Exit Code 137 là kết quả của việc tiến trình bị buộc dừng đột ngột bằng tín hiệu SIGKILL (128 + 9 = 137). Trực trạng này xảy ra phổ biến khi máy chủ bị cạn kiệt bộ nhớ và cơ chế Out-Of-Memory Killer (OOM Killer) của Linux can thiệp để bảo vệ hệ điều hành.
[D]
Tệp tin cấu hình môi trường .env bị thiếu một số biến cấu hình bắt buộc để ứng dụng khởi động.
[EXP]
Sai. Thiếu cấu hình/biến môi trường thường làm ứng dụng dừng trong quá trình khởi chạy (như lỗi cấu hình Spring Boot) và trả về mã lỗi thông thường, chứ không kích hoạt cơ chế cưỡng bức của hệ điều hành.

@correct: C
@point: 20
