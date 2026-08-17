# KỊCH BẢN TRÌNH BÀY — SESSION 07: TỰ ĐỘNG HÓA QUY TRÌNH CI/CD VỚI GITHUB ACTIONS

**Thời lượng mục tiêu:** 25–35 phút, gồm khoảng 15–20 phút lý thuyết có minh họa và 10–15 phút demo. Nếu chỉ trình bày bản rút gọn, phần lời cho slide 1–7 đã đủ quá 5 phút.

## Slide 01 — Mở đầu

Chào các bạn. Trong buổi này, chúng ta sẽ không xem CI/CD như một hệ thống phức tạp cần triển khai ngay từ đầu. Ta bắt đầu bằng một câu hỏi đơn giản: sau khi đẩy thay đổi lên repository, làm sao cả đội có một cách nhất quán để biết mã nguồn còn build được hay không?

Ví dụ xuyên suốt là `user-service` của QuickBite. Mục tiêu cuối buổi là đọc được một workflow nhỏ, biết nó chạy ở đâu, tạo được JAR artifact, và biết mở log đúng chỗ khi pipeline thất bại.

## Slide 02 — Lộ trình

Chúng ta đi theo đúng đường đi của một thay đổi mã nguồn: đầu tiên là GitHub Actions và Runner, sau đó là file workflow, cách nhiều jobs phối hợp, quy trình build Gradle, và cuối cùng là đọc log lỗi. Phần thực hành không tách rời lý thuyết: mỗi phần sẽ dùng lại QuickBite để kiểm chứng ý vừa học.

## Slide 03 — Chuyển phần 1

Phần đầu tiên trả lời hai câu hỏi: tại sao cần CI, và khi workflow được kích hoạt thì lệnh build thực sự chạy ở máy nào.

## Slide 04 — Build thủ công

Khi mỗi lập trình viên tự build trên máy mình, kết quả phụ thuộc vào JDK, Gradle cache, cấu hình môi trường và thao tác cá nhân. Vấn đề không phải là build thủ công luôn sai, mà là đội ngũ không có một phản hồi chung ngay sau khi source thay đổi.

CI đưa các bước đó vào một workflow có version control. Khi push xảy ra, workflow lặp lại cùng các bước, giữ log và trạng thái tại một chỗ. Nhờ vậy, người review biết được thay đổi có qua được kiểm tra kỹ thuật cơ bản trước khi đi xa hơn hay không.

## Slide 05 — Phân biệt các mức tự động hóa

Continuous Integration là phần chúng ta làm trọng tâm hôm nay: tự động build hoặc test khi mã nguồn thay đổi. Continuous Delivery đi thêm một bước là giữ artifact sẵn sàng phát hành. Continuous Deployment là tự động triển khai artifact đã đạt điều kiện.

Vì vậy Session 07 dừng ở JAR artifact. Docker image, registry và deploy sẽ là các công việc tiếp nối; chưa cần trộn tất cả vào một workflow đầu tiên.

## Slide 06 — GitHub và Runner

GitHub lưu workflow, nhận sự kiện push và điều phối job. Nhưng GitHub không tự chạy `./gradlew bootJar` thay chúng ta. Lệnh đó chạy ở Runner.

Điểm cần nhớ là self-hosted Runner chủ động kết nối tới GitHub để nhận job. Sau khi nhận job, Runner checkout source, chạy steps, rồi gửi trạng thái và log về giao diện Actions. Đây là lý do chúng ta có thể đọc được từng step đã chạy gì và lỗi ở đâu.

## Slide 07 — Chọn Runner

GitHub-hosted Runner giảm phần vận hành vì GitHub chuẩn bị môi trường. Self-hosted Runner phù hợp khi cần kiểm soát tài nguyên, sử dụng mạng nội bộ hoặc tái dùng cache. Đổi lại, đội vận hành phải lo cập nhật hệ điều hành, quyền truy cập và dọn dẹp workspace.

QuickBite dùng self-hosted Runner cho lab để chúng ta quan sát rõ hạ tầng thực thi. Đây là lựa chọn của bài lab, không phải quy tắc rằng mọi dự án phải tự vận hành Runner.

## Slide 08 — Demo Runner

**Chuẩn bị:** Mở terminal trên Ubuntu có Docker và truy cập được GitHub. Không trình chiếu hoặc gõ token thật lên màn hình. Token/PAT đã được nạp vào biến môi trường `PERSONAL_ACCESS_TOKEN_FOR_RUNNER` hoặc file secret ngoài repository.

Tôi mở thư mục `~/quickbite/action-runner` và cho các bạn xem `docker-compose.yml`. Các chi tiết đáng chú ý là runner cấp organization, các labels `self-hosted,quickbite,backend,ubuntu`, và Docker socket được mount để phục vụ các tác vụ Docker ở những session sau.

```bash
cd ~/quickbite/action-runner
docker compose up -d
docker compose ps
docker compose logs --tail 50 action-runner
```

Sau đó chuyển sang GitHub Organization settings, mục Actions runners. Kết quả cần thấy là runner online hoặc idle và có labels tương ứng. Nếu runner không online, kiểm tra log container trước; không tạo token mới một cách ngẫu nhiên.

## Slide 09 — Chuyển phần 2

Bây giờ Runner đã có thể nhận việc. Câu hỏi tiếp theo là: GitHub nhận biết công việc cần chạy từ file nào?

## Slide 10 — Đường dẫn workflow

GitHub chỉ nhận diện workflow khi file YAML nằm trong `.github/workflows/` từ root repository. Đây là lỗi rất hay gặp: YAML có thể đúng, nhưng đặt sai thư mục thì GitHub không thấy workflow nào cả.

Với QuickBite, workflow chưa có sẵn trong source. Trong demo, chúng ta tạo mới `.github/workflows/ci.yml` bên trong repository `user-service`, sau đó commit đúng file đó.

## Slide 11 — Cấu trúc workflow

Đọc từ trên xuống dưới: `name` là tên hiển thị; `on` là event; `jobs` là các đơn vị công việc; `runs-on` chọn Runner theo labels; `steps` là các thao tác trong job. `actions/checkout` không phải phép màu, nó đơn giản là action lấy source vào workspace của Runner.

Đừng cố ghi hết logic vào một step. Các step nhỏ, đặt tên rõ, sẽ làm log dễ đọc và dễ khoanh vùng khi có lỗi.

## Slide 12 — Demo workflow đầu tiên

Tôi tạo `ci.yml` cho `user-service`. Workflow này chỉ checkout source, cài Temurin 17 và in vài thông tin môi trường. `user-service` dùng Java 17 trong Gradle toolchain, nên đây là phiên bản phải khớp.

Sau khi tạo file, tôi kiểm tra YAML bằng mắt: dùng spaces thay Tab, `jobs` cùng cấp, và `steps` nằm đúng bên trong job. Đây là workflow quan sát môi trường, chưa build JAR; mục tiêu là xác nhận đường đi GitHub tới Runner trước.

## Slide 13 — Quan sát kết quả workflow

Tôi commit và push workflow lên nhánh đã khai báo. Trên tab Actions, ta không chỉ nhìn dấu xanh. Ta mở run, xác nhận đúng workflow, đúng nhánh, đúng Runner và output `java -version` là Java 17.

Nếu không có run, ba nơi cần kiểm tra theo thứ tự là: file đã nằm đúng `.github/workflows`, event `push` có khớp nhánh hay không, rồi mới tới Runner. Thứ tự này tránh việc debug hạ tầng khi lỗi thực ra chỉ là sai đường dẫn.

## Slide 14 — Chuyển phần 3

Một workflow thực tế thường không chỉ có một job. Ta cần biết job nào có thể chạy cùng lúc và job nào cần chờ kết quả của job khác.

## Slide 15 — `needs` tạo quan hệ phụ thuộc

Mặc định, các jobs không phụ thuộc nhau có thể chạy song song. Khi thêm `needs`, ta nói rõ rằng job sau chỉ có ý nghĩa khi job trước đã thành công. Nếu job trước thất bại, job phụ thuộc sẽ không chạy, vì đầu vào của nó không đáng tin cậy.

Điều này giúp workflow mô tả đúng logic thay vì chỉ mô tả thứ tự lệnh. Chúng ta sẽ thử hai kiểm tra độc lập rồi mới in báo cáo tổng hợp.

## Slide 16 — Isolation giữa jobs

Một lỗi tư duy phổ biến là tạo file ở job A rồi mong job B nhìn thấy nó ngay. Mỗi job phải được thiết kế như một đơn vị độc lập. Khi thật sự cần truyền dữ liệu, dùng artifact hoặc output có chủ đích.

Self-hosted Runner có thể còn cache hoặc file cũ trên máy, nhưng đó không phải contract của workflow. Workflow đáng tin cậy phải chạy đúng ngay cả khi không dựa vào trạng thái còn sót lại.

## Slide 17 — Demo ba jobs

Tôi thêm hai jobs đầu: một job chạy `uname -a`, job còn lại chạy `whoami`. Không có `needs` nên chúng có thể bắt đầu song song. Job `report` khai báo `needs` cho cả hai.

Sau khi push, ta mở graph ở tab Actions. Điều cần quan sát không phải chỉ là ba dấu xanh: `report` phải nằm sau hai job đầu. Nếu `report` chạy trước, nghĩa là YAML chưa thể hiện đúng phụ thuộc.

## Slide 18 — Chuyển phần 4

Khi đã có Runner và workflow, ta chuyển sang công việc CI hữu ích đầu tiên cho backend: build một Spring Boot service thành JAR.

## Slide 19 — JDK và Gradle Wrapper

Trước khi build, Runner phải có đúng JDK. QuickBite không dùng một phiên bản Java duy nhất: `user-service` dùng Java 17, trong khi restaurant, order và notification service dùng Java 21. Pipeline phải đọc cấu hình project, không đoán theo máy của người viết workflow.

Gradle Wrapper đi cùng source để mọi Runner dùng đúng Gradle version mà project kỳ vọng. Vì thế chúng ta chạy `./gradlew`, không giả định Gradle cài toàn cục trên server.

## Slide 20 — `bootJar` và artifact

`bootJar` tạo Spring Boot executable JAR. Trong `user-service`, file đầu ra có tên `user-service.jar` dưới `build/libs`. Sau build, upload artifact giúp người review hoặc người vận hành tải lại đúng file mà workflow vừa tạo.

Khi cần chạy test, dùng task như `build` hoặc `test`. Không nên nói `bootJar` đã thay thế hoàn toàn kiểm thử; nó chỉ là mục tiêu build tinh giản của demo này.

## Slide 21 — Demo build artifact

Tôi tạo workflow build gồm checkout, setup Java 17, `chmod +x ./gradlew`, `./gradlew bootJar` và upload artifact. Sau push, ta mở job build từng step. Khi thành công, kéo xuống cuối run để xem artifact `user-service-jar`.

Đây là thời điểm dừng hợp lý của Session 07: chúng ta đã có đầu ra JAR truy xuất được. Chưa build Docker image ở đây để giữ pipeline nhỏ, dễ đọc và dễ debug.

## Slide 22 — Chuyển phần 5

Một pipeline hữu ích không phải là pipeline không bao giờ lỗi. Điều quan trọng là khi lỗi, ta biết đọc log theo một phương pháp thay vì thử sửa ngẫu nhiên.

## Slide 23 — Quy trình debug

Khi thấy run đỏ, tôi bắt đầu từ job bị lỗi, sau đó mở đúng step, tìm lỗi đầu tiên có ngữ cảnh và đọc exit code. Tiếp theo mới phân loại lỗi: YAML, quyền, compile, test hoặc dependency.

Mục tiêu là tạo thay đổi nhỏ nhất có thể, chạy lại và xác nhận đúng giả thuyết. Đọc phần cuối log có ích, nhưng không được bỏ qua lỗi đầu tiên xuất hiện trước các lỗi dây chuyền.

## Slide 24 — Demo lỗi permission

Để demo ổn định, tôi không chỉ xóa bước `chmod +x`, vì executable bit có thể đã được Git giữ lại. Trên branch demo, tôi cố ý thêm `chmod -x ./gradlew` ngay trước lệnh build. Lần chạy sau sẽ báo `Permission denied`, thường kèm exit code 126.

Tôi chỉ cho học viên dòng lỗi, sau đó khôi phục workflow thành `chmod +x ./gradlew` và chạy lại. Demo này không đụng logic Java; nó tách rõ lỗi quyền thực thi khỏi lỗi source code.

## Slide 25 — Demo compile và test failure

Với lỗi compile, trên branch demo ta đổi tạm `UserRepository` thành `UserReposotory` trong `UserService.java`. Log Gradle sẽ chỉ ra `cannot find symbol`, file và dòng liên quan. Cách xử lý là sửa source và có thể chạy `./gradlew compileJava` trước khi push lại.

Với lỗi test, `UserServiceApplicationTests` hiện có test `contextLoads`. Ta thêm tạm một assertion sai trên branch demo, chạy workflow với `./gradlew build`, rồi quan sát test name và assertion failure. Sau demo, hoàn nguyên toàn bộ thay đổi bằng `git restore` hoặc bỏ branch demo. Không đưa lỗi cố ý vào `main`.

## Slide 26 — Kết thúc

Chúng ta kết lại bằng năm điểm: Runner online, workflow đúng vị trí, JDK đúng service, JAR được lưu thành artifact, và log được đọc từ job đến exit code. Một workflow nhỏ nhưng rõ ràng là nền tảng tốt hơn một pipeline dài, nhiều bước nhưng khó tin cậy.

Sau buổi này, các bạn có thể áp dụng cùng khuôn mẫu cho service khác của QuickBite: trước hết đổi JDK đúng toolchain, sau đó giữ workflow và cách đọc log nhất quán.
