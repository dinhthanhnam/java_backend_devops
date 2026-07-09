# QUIZ LESSONS - SESSION 10

# LESSON 01: Giới thiệu VPS và thiết lập kết nối cơ bản qua Bitvise Client

## Q1

Mô hình máy chủ ảo VPS có đặc điểm nào dưới đây so với dịch vụ Cloud Server (như AWS EC2)?

[A]
Có khả năng tự động co giãn tài nguyên (Auto-scaling) cực kỳ linh hoạt theo lưu lượng truy cập thực tế.
[EXP]
Sai. Đây là đặc điểm nổi bật của Cloud Server chứ không phải VPS truyền thống.
[B]
Được cài đặt sẵn toàn bộ các thành phần runtime của ứng dụng Spring Boot và cơ sở dữ liệu khi khởi tạo.
[EXP]
Sai. Cả VPS và Cloud Server thường chỉ cung cấp hệ điều hành sạch khi mới khởi tạo, người dùng phải tự cấu hình.
[C]
Cung cấp tài nguyên phần cứng cố định trên một node vật lý và khó thực hiện nâng cấp nóng không gián đoạn.
[EXP]
Chính xác. VPS thường có tài nguyên cố định và bị giới hạn bởi node vật lý chứa nó, việc nâng cấp thường yêu cầu restart máy chủ.
[D]
Bị giới hạn hoàn toàn quyền truy cập quản trị hệ điều hành (Root OS) và không thể can thiệp sâu vào kernel.
[EXP]
Sai. VPS vẫn cấp toàn quyền quản trị cao nhất (root) cho người sử dụng.

@correct: C
@point: 20

## Q2

Khi sử dụng các công cụ dòng lệnh mặc định (như PowerShell/CMD) ở máy local để kết nối SSH vào VPS lần đầu tiên, hệ thống hiển thị thông điệp "The authenticity of host... can't be established". Lập trình viên nên xử lý như thế nào?

[A]
Nhập "yes" để xác nhận lưu vân tay khóa bảo mật (host key fingerprint) của máy chủ vào máy local.
[EXP]
Chính xác. Đây là cảnh báo an toàn lần đầu kết nối, nhập "yes" giúp lưu thông tin nhận diện của VPS để kiểm tra ở các lần sau.
[B]
Ngắt kết nối lập tức và yêu cầu nhà cung cấp VPS cài lại toàn bộ hệ điều hành vì máy chủ đã bị xâm nhập.
[EXP]
Sai. Đây là thông báo tiêu chuẩn khi kết nối lần đầu tới bất kỳ máy chủ SSH mới nào, không phải dấu hiệu bị hack.
[C]
Đổi cổng SSH mặc định từ 22 sang một cổng khác ngẫu nhiên trên máy local để bypass qua bước kiểm tra khóa.
[EXP]
Sai. Việc đổi cổng không giúp bypass qua cảnh báo nhận diện fingerprint này của giao thức SSH.
[D]
Sử dụng tài khoản quản trị phụ thay thế cho tài khoản root để tránh hiển thị cảnh báo bảo mật hệ thống.
[EXP]
Sai. Cảnh báo này xuất hiện dựa trên địa chỉ host (IP/Port), hoàn toàn độc lập với tài khoản người dùng đăng nhập.

@correct: A
@point: 20

## Q3

Khi lưu thông tin phiên làm việc trên Bitvise SSH Client để phục vụ kết nối nhanh sau này, tệp tin cấu hình profile sẽ được lưu với phần mở rộng (đuôi file) nào?

[A]
Tệp tin cấu hình lưu thông tin phiên có đuôi `.bjp`
[EXP]
Sai. Đuôi `.bjp` không phải định dạng tệp tin profile của ứng dụng Bitvise SSH Client.
[B]
Tệp tin cấu hình lưu thông tin phiên có đuôi `.tpl`
[EXP]
Chính xác. Bitvise SSH Client xuất và nạp các profile cấu hình kết nối dưới dạng tệp tin `.tpl`.
[C]
Tệp tin cấu hình lưu thông tin phiên có đuôi `.ssh`
[EXP]
Sai. Thư mục `.ssh` dùng để lưu trữ key xác thực, không phải đuôi file profile của Bitvise.
[D]
Tệp tin cấu hình lưu thông tin phiên có đuôi `.conf`
[EXP]
Sai. Định dạng `.conf` thường dùng cho các tệp cấu hình hệ thống Linux, không phải profile Bitvise.

@correct: B
@point: 20

## Q4

Sau khi kết nối thành công vào VPS bằng Bitvise SSH Client, làm thế nào để lập trình viên mở trình quản lý tệp tin đồ họa SFTP để duyệt và kéo thả file?

[A]
Hệ thống sẽ tự động bật đồng thời cả terminal console và cửa sổ SFTP lên màn hình ngay khi log in thành công.
[EXP]
Sai. Bitvise không tự động mở các cửa sổ này mà người dùng phải thao tác kích hoạt thủ công.
[B]
Gõ lệnh `sftp root@localhost` trực tiếp trên terminal console của Bitvise để chuyển đổi giao diện dòng lệnh.
[EXP]
Sai. Lệnh này dùng để chạy sftp dạng CLI, không giúp mở giao diện quản lý tệp tin đồ họa trực quan của Bitvise.
[C]
Click vào biểu tượng "New SFTP window" ở danh sách tùy chọn bên thanh menu bên trái của ứng dụng Bitvise.
[EXP]
Chính xác. Bitvise thiết kế các nút chức năng "New terminal console" và "New SFTP window" ở menu trái để người dùng mở khi cần.
[D]
Kéo thả file trực tiếp từ máy local vào cửa sổ terminal console đang chạy để hệ thống tự động upload file.
[EXP]
Sai. Bạn không thể kéo thả file vào cửa sổ dòng lệnh terminal console để thực hiện truyền tải dữ liệu.

@correct: C
@point: 20

## Q5

Mục đích chính của việc chạy chuỗi lệnh `sudo apt update && sudo apt upgrade -y` ngay khi nhận bàn giao một VPS mới từ nhà cung cấp là gì?

[A]
Giải phóng dung lượng ổ cứng SSD bằng cách dọn dẹp các tệp tin log rác sinh ra trong quá trình cài đặt OS.
[EXP]
Sai. Lệnh này dùng để tải và nâng cấp phần mềm, ngược lại nó có thể làm tăng nhẹ dung lượng lưu trữ do cài các bản vá mới.
[B]
Đồng bộ hóa múi giờ hệ thống của VPS với múi giờ thực tế tại máy local của người quản trị viên.
[EXP]
Sai. Đồng bộ múi giờ yêu cầu các lệnh cấu hình thời gian khác như timedatectl, apt upgrade không tự đổi múi giờ.
[C]
Đổi tên đăng nhập của tài khoản quản trị tối cao từ root sang một tên người dùng khác an toàn hơn.
[EXP]
Sai. Việc đổi tên tài khoản hoặc cấu hình user không nằm trong chức năng của hai lệnh nâng cấp gói phần mềm này.
[D]
Cập nhật danh sách gói và tiến hành vá các lỗ hổng bảo mật của các thư viện hệ thống đã lỗi thời.
[EXP]
Chính xác. Các bản phân phối OS có sẵn của nhà cung cấp thường cũ. Việc chạy chuỗi lệnh này giúp hệ thống an toàn hơn trước các lỗ hổng bảo mật mới.

@correct: D
@point: 20

---

# LESSON 02: Tạo User mới và bảo mật VPS bằng cơ chế khóa SSH (SSH Hardening)

## Q1

Mục đích cốt lõi của việc áp dụng nguyên tắc đặc quyền tối thiểu (Least Privilege) bằng cách tạo user `deployer` thay vì sử dụng trực tiếp tài khoản `root` là gì?

[A]
Ngăn chặn hoàn toàn các cuộc tấn công dò quét mật khẩu (Brute-force) nhắm vào cổng kết nối SSH của VPS.
[EXP]
Sai. Tạo user mới chỉ hạn chế rủi ro root, để chống Brute-force triệt để cần tắt xác thực mật khẩu (PasswordAuthentication no).
[B]
Giảm thiểu thiệt hại do lỗi thao tác của quản trị viên và kiểm soát chặt chẽ các hành động nâng quyền qua lệnh `sudo`.
[EXP]
Chính xác. Thao tác bằng root dễ gây lỗi hệ thống nghiêm trọng nếu gõ sai lệnh. Dùng user thường và sudo giúp bảo vệ hệ thống tốt hơn.
[C]
Tăng tốc độ xử lý các tác vụ biên dịch mã nguồn Java Spring Boot trên VPS nhờ giảm tải tài nguyên hệ thống.
[EXP]
Sai. Việc dùng user thường không ảnh hưởng đến hiệu năng tính toán của CPU hay RAM khi chạy ứng dụng.
[D]
Tự động mã hóa toàn bộ dữ liệu lưu trữ trên ổ cứng SSD của máy chủ VPS để tránh bị rò rỉ dữ liệu thô.
[EXP]
Sai. SSH và phân quyền user thường không thực hiện nhiệm vụ mã hóa toàn bộ đĩa cứng vật lý (Full Disk Encryption).

@correct: B
@point: 20

## Q2

Trong cơ chế xác thực bằng khóa SSH Key sử dụng thuật toán mã hóa bất đối xứng, khóa riêng tư (Private Key) được lưu giữ ở đâu?

[A]
Được đẩy lên cấu hình trong tệp `~/.ssh/authorized_keys` của tài khoản người dùng trên máy chủ VPS.
[EXP]
Sai. Tệp authorized_keys trên VPS dùng để chứa khóa công khai (Public Key), không phải khóa riêng tư.
[B]
Được lưu trữ an toàn và bảo mật trên máy tính cá nhân (local) của lập trình viên và tuyệt đối không chia sẻ.
[EXP]
Chính xác. Private Key dùng để giải mã thử thách xác thực và phải được giữ tuyệt mật tại máy client (máy local).
[C]
Được tự động đăng ký và lưu trữ tập trung trên máy chủ quản lý mã nguồn GitHub của doanh nghiệp.
[EXP]
Sai. GitHub chỉ lưu trữ Public Key của bạn để xác thực quyền clone/push code, không bao giờ giữ Private Key của bạn.
[D]
Được đính kèm trực tiếp vào tệp tin cấu hình docker-compose.yml để các container có thể sử dụng chung.
[EXP]
Sai. Không bao giờ được để lộ Private Key trong các tệp cấu hình ứng dụng hay mã nguồn dự án.

@correct: B
@point: 20

## Q3

Để tạo người dùng mới tên là `deployer` và cấp quyền quản trị tối cao (quyền thực thi qua lệnh `sudo`) trên Ubuntu Server, chuỗi lệnh nào sau đây là chính xác?

[A]
`sudo adduser deployer && sudo usermod -aG sudo deployer`
[EXP]
Chính xác. Lệnh adduser tạo người dùng mới, và usermod -aG sudo thêm người dùng đó vào nhóm quản trị sudo để có quyền chạy lệnh quản trị.
[B]
`sudo useradd deployer && sudo groupadd sudo deployer`
[EXP]
Sai. Lệnh groupadd dùng để tạo nhóm mới chứ không phải để thêm user vào nhóm hiện có.
[C]
`sudo createuser deployer && sudo grant privilege to deployer`
[EXP]
Sai. Các lệnh này không phải cú pháp quản trị người dùng chuẩn của Linux (đây là các lệnh tương tự SQL/Database).
[D]
`sudo newuser deployer && sudo chmod +x /home/deployer`
[EXP]
Sai. Không có lệnh newuser mặc định trên Ubuntu, chmod +x chỉ cấp quyền thực thi tệp tin chứ không cấp quyền sudo.

@correct: A
@point: 20

## Q4

Sau khi hoàn tất cấu hình SSH Hardening trong tệp `/etc/ssh/sshd_config`, lập trình viên bắt buộc phải chạy lệnh nào sau đây để các cấu hình bảo mật mới chính thức có hiệu lực?

[A]
`sudo systemctl restart ssh`
[EXP]
Chính xác. Dịch vụ SSH Daemon cần được restart để tải lại tệp cấu hình sshd_config mới sửa đổi.
[B]
`sudo ufw reload`
[EXP]
Sai. Lệnh này dùng để nạp lại cấu hình tường lửa UFW, không ảnh hưởng đến dịch vụ cấu hình SSH.
[C]
`sudo apt update`
[EXP]
Sai. Lệnh này chỉ dùng để cập nhật danh sách gói phần mềm, không nạp lại cấu hình dịch vụ hệ thống.
[D]
`newgrp docker`
[EXP]
Sai. Lệnh này dùng để cập nhật quyền nhóm docker cho session terminal hiện tại, không liên quan đến SSH.

@correct: A
@point: 20

## Q5

Trong tệp cấu hình `/etc/ssh/sshd_config`, cấu hình nào sau đây giúp vô hiệu hóa hoàn toàn phương thức xác thực bằng mật khẩu thông thường để chống tấn công Brute-force?

[A]
`PubkeyAuthentication no`
[EXP]
Sai. Cấu hình này sẽ tắt xác thực bằng khóa SSH Key, buộc phải dùng mật khẩu (ngược lại với yêu cầu).
[B]
`PermitRootLogin no`
[EXP]
Sai. Cấu hình này chỉ chặn đăng nhập bằng tài khoản root, không tắt cơ chế đăng nhập bằng mật khẩu của các user khác.
[C]
`PasswordAuthentication no`
[EXP]
Chính xác. Thiết lập PasswordAuthentication no ngăn chặn hoàn toàn việc đăng nhập bằng mật khẩu, buộc phải có SSH Key.
[D]
`StrictModes no`
[EXP]
Sai. Cấu hình này kiểm tra quyền hạn của các tệp tin cấu hình SSH, không quản lý phương thức xác thực mật khẩu.

@correct: C
@point: 20

---

# LESSON 03: Thiết lập Tường lửa UFW và Cài đặt Docker trên Ubuntu Server

## Q1

Tại sao chúng ta nên ưu tiên cài đặt Docker Engine từ Docker Repository chính thức thay vì sử dụng gói cài đặt mặc định của Ubuntu (`docker.io`)?

[A]
Vì gói cài đặt của Ubuntu yêu cầu cấu hình thêm dịch vụ ảo hóa phụ trợ rất phức tạp trên máy chủ.
[EXP]
Sai. Gói docker.io không yêu cầu thêm ảo hóa phức tạp, vấn đề lớn nhất của nó là phiên bản cũ.
[B]
Vì gói chính thức của Docker giúp tăng tốc hiệu năng xử lý cơ sở dữ liệu bên trong container lên gấp đôi.
[EXP]
Sai. Hiệu năng ứng dụng phụ thuộc vào tài nguyên phần cứng và cấu hình ứng dụng, không do nguồn cài đặt docker quyết định.
[C]
Vì Docker Repository cung cấp phiên bản mới nhất, ổn định và cập nhật nhanh chóng các bản vá bảo mật chính hãng.
[EXP]
Chính xác. Kho cài đặt chính thức của Docker đảm bảo bạn luôn có phiên bản mới nhất cùng các bản vá lỗi bảo mật kịp thời.
[D]
Vì gói cài đặt mặc định của Ubuntu tự động kích hoạt tính năng chặn kết nối mạng của toàn bộ container.
[EXP]
Sai. Gói docker.io vẫn cho phép các container giao tiếp mạng bình thường theo cấu hình mặc định của Docker.

@correct: C
@point: 20

## Q2

Sau khi thêm người dùng `deployer` vào nhóm `docker`, tại sao lập trình viên cần chạy lệnh `newgrp docker` trong session terminal hiện tại?

[A]
Để kích hoạt dịch vụ Docker Daemon khởi chạy ngầm cùng hệ thống của máy chủ VPS.
[EXP]
Sai. Khởi chạy dịch vụ docker cần dùng các lệnh như systemctl start docker.
[B]
Để cập nhật quyền hạn của nhóm docker cho phiên làm việc hiện tại mà không cần logout và đăng nhập lại.
[EXP]
Chính xác. Unix mặc định chỉ cập nhật quyền nhóm mới khi mở session mới, newgrp giúp áp dụng quyền ngay lập tức.
[C]
Để tự động tạo ra một card mạng ảo (bridge network) dùng chung cho cụm microservices QuickBite.
[EXP]
Sai. Tạo mạng ảo là nhiệm vụ của lệnh docker network create hoặc docker compose.
[D]
Để gán toàn quyền sở hữu tệp cấu hình docker-compose.yml cho tài khoản của người dùng deployer.
[EXP]
Sai. Lệnh này không thay đổi quyền sở hữu (owner) vật lý của các file trên đĩa cứng.

@correct: B
@point: 20

## Q3

Nguyên tắc thiết lập tường lửa UFW an toàn và bảo mật cho máy chủ VPS chạy môi trường Production là gì?

[A]
Cho phép toàn bộ kết nối đi vào (Default Allow Incoming) và chỉ chặn một số cổng database nhạy cảm như 5432.
[EXP]
Sai. Thiết lập này cực kỳ nguy hiểm vì các cổng dịch vụ mới mở sau này sẽ bị lộ mặc định ra ngoài Internet.
[B]
Chặn toàn bộ lưu lượng đi ra ngoài Internet (Default Deny Outgoing) để tránh rò rỉ mã nguồn dự án.
[EXP]
Sai. Chặn outgoing khiến VPS không thể tải thư viện, cập nhật phần mềm hoặc đồng bộ mã nguồn từ GitHub.
[C]
Chặn toàn bộ kết nối đi vào mặc định (Default Deny Incoming), chỉ mở các cổng dịch vụ thiết yếu (như 22, 80, 443).
[EXP]
Chính xác. Đây là nguyên tắc "Default Deny" chuẩn bảo mật, chỉ mở những cổng thực sự cần thiết để giảm thiểu diện tấn công.
[D]
Tắt hoàn toàn tường lửa UFW và chuyển giao nhiệm vụ lọc gói tin cho tường lửa nội bộ của các container Docker.
[EXP]
Sai. Docker không tự tích hợp tường lửa lọc gói tin từ ngoài Internet, máy chủ vẫn cần tường lửa UFW ở tầng host bảo vệ.

@correct: C
@point: 20

## Q4

Điều gì sẽ xảy ra nếu lập trình viên thực hiện kích hoạt tường lửa bằng lệnh `sudo ufw enable` khi chưa chạy lệnh cho phép kết nối cổng SSH (`sudo ufw allow 22/tcp`)?

[A]
Hệ thống sẽ hiển thị lỗi cú pháp và tự động từ chối chạy lệnh kích hoạt tường lửa để bảo vệ kết nối.
[EXP]
Sai. UFW vẫn kích hoạt bình thường theo yêu cầu của bạn mà không tự động chặn lệnh hay sửa lỗi hộ bạn.
[B]
Phiên kết nối SSH hiện tại sẽ bị ngắt lập tức và bạn bị khóa quyền truy cập vào VPS từ xa.
[EXP]
Chính xác. Vì chính sách mặc định chặn incoming, cổng 22 bị đóng sẽ ngắt toàn bộ kết nối SSH hiện tại và tương lai.
[C]
Bitvise SSH Client sẽ tự động phát hiện và thêm luật mở cổng 22 từ xa vào cấu hình của VPS.
[EXP]
Sai. Bitvise chỉ là phần mềm client ở máy local, không thể can thiệp ghi đè hay tự sửa cấu hình tường lửa hệ thống trên VPS.
[D]
Hệ điều hành VPS sẽ tự động thực hiện restart máy chủ để phục hồi lại cấu hình mạng ban đầu.
[EXP]
Sai. Hệ điều hành không tự động restart để khôi phục mạng; cấu hình tường lửa đã ghi xuống đĩa cứng sẽ được áp dụng mãi.

@correct: B
@point: 20

## Q5

Tại sao các container (như dịch vụ backend và database) chạy chung mạng Docker Network vật lý vẫn giao tiếp được bình thường dù tường lửa UFW trên máy host đã chặn cổng database (ví dụ cổng `5432` của PostgreSQL)?

[A]
Vì các container giao tiếp nội bộ thông qua card mạng ảo riêng do Docker quản lý, không đi qua card mạng vật lý của host.
[EXP]
Chính xác. UFW ở máy host chỉ lọc gói tin đi qua các card mạng vật lý nối ra ngoài Internet, không lọc lưu lượng mạng nội bộ của Docker Bridge.
[B]
Vì Docker tự động cấu hình bypass qua toàn bộ hệ thống tường lửa UFW của máy host đối với tất cả các cổng.
[EXP]
Sai. Docker chỉ can thiệp NAT port ra ngoài Internet, giao tiếp thuần nội bộ giữa các container không cần đi qua filter của UFW.
[C]
Vì database PostgreSQL tự động đổi cổng giao tiếp nội bộ sang cổng 80 khi phát hiện tường lửa UFW hoạt động.
[EXP]
Sai. Database vẫn chạy cố định ở cổng cấu hình (ví dụ 5432), không tự động đổi cổng giao tiếp.
[D]
Vì tường lửa UFW chỉ có hiệu lực bảo vệ đối với các dịch vụ viết bằng Java Spring Boot.
[EXP]
Sai. UFW lọc gói tin ở tầng mạng của hệ điều hành host, hoàn toàn độc lập với ngôn ngữ lập trình của ứng dụng.

@correct: A
@point: 20

---

# LESSON 04: Đồng bộ cấu hình qua Git bằng SSH Key và vận hành cụm Microservices

## Q1

Trong quy trình đồng bộ mã nguồn dự án từ GitHub về VPS, mục đích của việc sinh khóa SSH Key trên VPS là gì?

[A]
Để cho phép máy tính cá nhân của lập trình viên có thể SSH thẳng vào thư mục làm việc trên VPS không cần mật khẩu.
[EXP]
Sai. Khóa dùng để local SSH vào VPS được tạo ở máy local, không phải tạo trên VPS.
[B]
Để cài đặt làm khóa mã hóa bảo mật cho các tệp tin cấu hình môi trường `.env` chạy trên VPS.
[EXP]
Sai. SSH Key dùng để xác thực kết nối, không dùng để mã hóa nội dung tệp tin `.env`.
[C]
Để VPS xác thực danh tính và có quyền clone các repository riêng tư (Private Repository) từ tài khoản GitHub của bạn.
[EXP]
Chính xác. Khóa sinh trên VPS sẽ được thêm vào tài khoản GitHub nhằm cấp quyền xác thực cho VPS khi giao tiếp với GitHub qua giao thức SSH.
[D]
Để tự động đồng bộ hóa các thay đổi mã nguồn từ máy local lên thẳng môi trường Production của VPS.
[EXP]
Sai. Đồng bộ trực tiếp cần các luồng CI/CD; SSH Key chỉ đóng vai trò là phương thức xác thực quyền truy cập kho code.

@correct: C
@point: 20

## Q2

Khóa công khai (Public Key) sinh ra trên VPS để kết nối với GitHub được cấu hình ở vị trí nào sau đây?

[A]
Mục **SSH and GPG keys** trong phần Settings tài khoản GitHub cá nhân của bạn.
[EXP]
Chính xác. Đăng nhập GitHub, vào Settings tài khoản cá nhân -> SSH and GPG keys -> New SSH key để thêm khóa công khai của VPS.
[B]
Thư mục `.ssh/authorized_keys` của tài khoản người dùng root trên máy chủ VPS.
[EXP]
Sai. Đây là nơi chứa Public Key của máy local để đăng nhập vào VPS, không phải nơi cấu hình để VPS kết nối tới GitHub.
[C]
Mục **Secrets and variables** -> **Actions** trong phần Settings của Repository dự án trên GitHub.
[EXP]
Sai. Mục này dùng để lưu trữ các biến bảo mật cho luồng chạy CI/CD (GitHub Actions), không phải nơi khai báo SSH Key tài khoản.
[D]
Được ghi trực tiếp vào thuộc tính credentials của tệp tin cấu hình docker-compose.yml.
[EXP]
Sai. Tệp docker-compose.yml không chứa thông tin xác thực SSH Key kết nối tới tài khoản GitHub.

@correct: A
@point: 20

## Q3

Nguyên tắc quản lý tệp cấu hình môi trường `.env` trong các dự án phần mềm chuyên nghiệp là gì?

[A]
Khai báo tệp `.env` trong `.gitignore` để không bao giờ push lên Git, và tự cấu hình thủ công tệp này trên từng môi trường.
[EXP]
Chính xác. Tệp `.env` chứa thông tin nhạy cảm (mật khẩu db, jwt key), bắt buộc phải bỏ qua khi commit Git và cấu hình riêng ở mỗi server.
[B]
Push tệp `.env` lên Git Repository để tất cả các thành viên trong dự án và hệ thống VPS có cấu hình đồng bộ nhất quán.
[EXP]
Sai. Push `.env` lên Git sẽ làm lộ thông tin bảo mật và mật khẩu của môi trường Production ra bên ngoài.
[C]
Cấu hình quyền đọc ghi rộng rãi (chmod 777) cho file `.env` trên VPS để tất cả các ứng dụng hệ thống đều truy cập được.
[EXP]
Sai. File `.env` chứa thông tin nhạy cảm nên cần được giới hạn quyền đọc (chỉ cho phép user deployer/docker đọc).
[D]
Tích hợp trực tiếp toàn bộ nội dung của tệp `.env` vào mã nguồn của file JAR khi thực hiện build ứng dụng.
[EXP]
Sai. Làm vậy sẽ phá vỡ nguyên lý tách biệt cấu hình khỏi mã nguồn, khiến ứng dụng khó chuyển đổi môi trường linh hoạt.

@correct: A
@point: 20

## Q4

Lệnh nào dưới đây giúp lập trình viên giám sát tài nguyên phần cứng (CPU, dung lượng RAM tiêu thụ) của từng container đang chạy trên VPS theo thời gian thực?

[A]
`docker stats`
[EXP]
Chính xác. Lệnh docker stats hiển thị bảng thống kê luồng sử dụng tài nguyên (CPU, RAM, Network I/O) của tất cả container đang chạy.
[B]
`docker compose ps`
[EXP]
Sai. Lệnh này chỉ hiển thị danh sách các container kèm trạng thái sống/chết (Up/Down) và cổng chạy, không đo đạc tài nguyên.
[C]
`docker compose logs -f`
[EXP]
Sai. Lệnh này hiển thị log đầu ra (Console output) của các container, không đo đạc lượng CPU hay RAM tiêu thụ.
[D]
`docker version`
[EXP]
Sai. Lệnh này hiển thị thông tin phiên bản của ứng dụng Docker CLI và Docker Daemon đang cài đặt trên máy.

@correct: A
@point: 20

## Q5

Khi chạy ứng dụng Spring Boot bằng Docker Compose trên VPS, container của một dịch vụ Microservices báo trạng thái `Exit Code 137` và dừng hoạt động. Nguyên nhân chính của lỗi này là gì?

[A]
Cơ sở dữ liệu PostgreSQL bị sập khiến ứng dụng Spring Boot không thể kết nối và tự động ném ra mã lỗi.
[EXP]
Sai. Lỗi kết nối database thường làm ứng dụng ném ngoại lệ và dừng với Exit Code 1 thông thường, không phải 137.
[B]
Cổng chạy ứng dụng (ví dụ 8081) đang bị chiếm dụng bởi một tiến trình khác đang chạy ngoài máy host.
[EXP]
Sai. Trùng cổng chạy thường báo lỗi BindException và dừng với Exit Code 1 ngay khi khởi động.
[C]
Hệ điều hành (Kernel) đã cưỡng bức tắt container bằng tín hiệu SIGKILL do hệ thống bị cạn kiệt bộ nhớ RAM vật lý (OOM Killer).
[EXP]
Chính xác. Exit Code 137 thường chỉ ra tiến trình bị kill bằng SIGKILL (9) từ hệ điều hành do cạn kiệt bộ nhớ (Out Of Memory).
[D]
Tệp tin cấu hình môi trường `.env` bị thiếu một số biến môi trường quan trọng phục vụ kết nối.
[EXP]
Sai. Thiếu biến môi trường sẽ báo lỗi cấu hình ứng dụng Java và trả về Exit Code thông thường, không kích hoạt OOM Killer.

@correct: C
@point: 20
