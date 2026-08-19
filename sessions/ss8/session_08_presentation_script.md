# KỊCH BẢN TRÌNH BÀY — SESSION 08

## Đóng gói Docker Image & Đẩy lên Registry

Kịch bản đi theo `session_08_slide_content.md`. Khi quay từng video riêng, bắt đầu từ divider của lesson tương ứng. Giọng nói bình tĩnh, giải thích rõ nguyên nhân trước khi đọc lệnh; không đọc lại nguyên văn toàn bộ bullet trên slide.

## Slide 01 — Tiêu đề chính

Chào các bạn. Session này nối tiếp bước build JAR ở CI bằng một đầu ra gần với vận hành hơn: Docker image. Chúng ta sẽ đi từ cách build image, tổ chức Dockerfile, đặt version, lưu image lên Registry cho đến cách một workflow khác lấy đúng image đó về kiểm tra. Ví dụ xuyên suốt là `user-service` của QuickBite.

## Slide 02 — Agenda

Năm lesson đi theo một chuỗi liên tục. Lesson đầu đưa image vào pipeline; lesson hai làm rõ multi-stage build; lesson ba nói về tag, digest và GHCR; lesson bốn dùng lại image từ Registry trong workflow; lesson cuối ghép toàn bộ thành một lab có bằng chứng kỹ thuật. Khi quay riêng từng lesson, phần divider sẽ nhắc lại đúng phạm vi của lesson đó.

## Slide 03 — Divider Lesson 01

Trong Lesson 01, ta trả lời câu hỏi: nếu JAR đã được CI tạo ra, làm sao đóng gói JAR đó thành image mà vẫn truy vết được nó xuất phát từ job nào và commit nào?

## Slide 04 — Vì sao image cũng phải được build trong pipeline

Nếu CI chỉ build JAR còn image được tạo thủ công ở local, hai đầu ra có thể không còn cùng một trạng thái source. Máy local có thể có thay đổi chưa commit, Dockerfile khác revision, hoặc JAR cũ. Đưa Docker build vào pipeline không tự động loại bỏ mọi rủi ro, nhưng tạo được đường đi có thể kiểm tra: run nào tạo JAR, job nào tạo image và log nào xác nhận kết quả. Đó là giá trị cốt lõi của CI trong tình huống này.

## Slide 05 — Điểm chuyển giao từ JAR sang image

Ở mô hình hai jobs, `build_jar` chạy Gradle và upload JAR dưới tên artifact `app-jar`. Job `build_image` khai báo `needs: build_jar`, checkout Dockerfile, download artifact đúng vào `build/libs`, rồi mới gọi `docker build`. Đừng dựa vào giả định workspace của job trước vẫn còn nguyên ở job sau. Artifact và `needs` làm cho việc truyền file cùng thứ tự thực thi trở nên tường minh.

## Slide 06 — Docker socket và quyền trên host

QuickBite mount `/var/run/docker.sock` vào self-hosted Runner. Nhờ vậy, lệnh Docker trong Runner gọi Docker daemon của host, tận dụng layer cache và không cần Docker-in-Docker. Đổi lại, workflow truy cập socket có quyền rất cao trên Docker host. Vì vậy chỉ chạy workflow, action và mã nguồn đáng tin cậy trên Runner này; đây không phải một volume dữ liệu thông thường.

## Slide 07 — Demo build image từ JAR artifact

Trong demo, chúng ta tạo `image.yml` cạnh `ci.yml`, không ghi đè workflow đã có. Dockerfile đơn tầng của `user-service` copy `build/libs/user-service.jar` vào image JRE 17. Tôi giải thích từng step: `needs` khóa thứ tự, download artifact đặt JAR đúng vị trí Dockerfile cần, và `docker build -t user-service:ci .` tạo tag tạm để kiểm tra. Sau run, `docker image inspect user-service:ci` xác nhận image tồn tại; chưa push Registry ở bước này.

## Slide 08 — Divider Lesson 02

Lesson 02 chuyển sang Dockerfile tự đủ. Ta không bỏ mô hình artifact ở lesson trước; ta dùng nó để hiểu rõ hơn vì sao multi-stage build lại tách môi trường build khỏi runtime.

## Slide 09 — Bản chất multi-stage build

Builder stage cần JDK, Gradle Wrapper, source và dependency để tạo JAR. Runtime stage chỉ cần JRE phù hợp cùng JAR đã tạo. `COPY --from=builder` là ranh giới rõ ràng: chỉ artifact cần chạy đi sang runtime. Nhờ đó source code, Gradle Wrapper và công cụ biên dịch không trở thành thành phần bắt buộc của runtime image.

## Slide 10 — Dockerfile Java 17 cho `user-service`

Khi đọc Dockerfile này, cần nhìn hai stage thay vì chỉ nhìn lệnh. Builder dùng JDK 17 để chạy Gradle; runtime dùng JRE 17 để chạy JAR. `user-service` dùng Java 17, còn `restaurant-service`, `order-service` và `notification-service` dùng Java 21. Không sao chép base image giữa các service mà không kiểm tra toolchain và tên JAR thực tế.

## Slide 11 — Cache Gradle phụ thuộc thứ tự layer

Ta copy Gradle Wrapper và build script trước, chạy bước dependency, sau đó mới copy `src` và chạy `bootJar`. Khi chỉ sửa source Java, builder còn cache hợp lệ có thể tái sử dụng layer dependency. Đây là khả năng tối ưu, không phải cam kết thời gian build. `.dockerignore` cũng quan trọng: loại `.git`, `.gradle`, `build` và `.env` để context nhỏ hơn, không mang cache local hoặc cấu hình nhạy cảm vào builder.

## Slide 12 — Demo so sánh hai cách đóng gói

Build một image single-stage và một image multi-stage từ cùng source, rồi xem `docker image ls` và `docker history`. Mục tiêu không phải thi xem image nào có một con số dung lượng đẹp hơn. Chúng ta xác nhận runtime image multi-stage chỉ nhận JAR, không cần giữ source hoặc Gradle Wrapper. Nếu kết quả khác nhau giữa hai máy, hãy nhìn base image, cache và build context trước khi kết luận.

## Slide 13 — Divider Lesson 03

Image vừa build chỉ nằm trên Docker daemon hiện tại. Lesson này đưa image sang GitHub Container Registry để Runner hoặc máy khác có thể truy xuất lại bằng một reference rõ ràng.

## Slide 14 — Image name, tag và digest

`ghcr.io/<namespace>/user-service` xác định package. Tag `1.0.0` là nhãn dễ đọc để chọn release, nhưng tag có thể bị push lại. Digest `sha256:...` gắn với nội dung image cụ thể. Vì thế tag hữu ích cho giao tiếp, còn digest hữu ích khi cần khóa đúng artifact đã được kiểm chứng. `latest` không nên là căn cứ duy nhất để rollback production.

## Slide 15 — Xác thực và push từ local

Ở local, dùng PAT có scope phù hợp và truyền qua `--password-stdin`; tuyệt đối không đặt token trực tiếp trong lệnh, Dockerfile hay YAML. Sau login, `docker tag` chỉ tạo một reference GHCR cho image local, không build lại image. `docker push` upload các layer cần thiết và trả về digest manifest. Digest đó cần được ghi vào báo cáo lab cùng image reference, thay vì chỉ chụp màn hình package.

## Slide 16 — Local push chỉ là bước học

Push local giúp chúng ta hiểu đầy đủ CLI, namespace, tag và quyền package. Nhưng image phát hành nên được CI build từ commit đã qua kiểm chứng; image local có thể chứa thay đổi chưa commit hoặc dùng môi trường khác Runner. Khi chuyển sang vận hành, ưu tiên `GITHUB_TOKEN` với quyền tối thiểu hơn là PAT cá nhân trong workflow.

## Slide 17 — Divider Lesson 04

Lesson 04 dùng lại image đã nằm trên Registry. Trọng tâm không còn là build source mà là pull đúng reference, cấp đúng quyền và chọn đúng mức kiểm tra.

## Slide 18 — `GITHUB_TOKEN` và quyền tối thiểu

Slide này chỉ cần chốt bốn ý. [Dừng một nhịp]

Thứ nhất, `GITHUB_TOKEN` **không phải** token chúng ta tự tạo, rồi tự lưu vào Repository secrets. Mỗi khi một Job bắt đầu, GitHub tự cấp cho Job đó một token riêng. Vì vậy, khi nhìn thấy dòng `password: ${{ secrets.GITHUB_TOKEN }}`, các bạn hiểu đơn giản là: *bước login đang lấy token GitHub vừa cấp để đăng nhập GHCR*.

Thứ hai, Job nào xong thì token của Job đó cũng hết hiệu lực. [Chỉ vào hai luồng minh họa] Job pull nhận Token A; Job publish nhận Token B. Đây là hai token khác nhau, không phải một token dùng chung cho toàn bộ workflow.

Thứ ba, block `permissions` **không tạo ra token**. Nó chỉ giới hạn token GitHub đã cấp được phép làm gì. Với Job ở bên trái, nhiệm vụ là kéo image về để kiểm tra, nên chỉ cần `packages: read`. Với Job ở bên phải, nhiệm vụ là đẩy image mới lên GHCR, nên cần `packages: write`. Nếu Job publish có checkout source thì cấp thêm `contents: read` cho đúng bước checkout.

Cuối cùng, trong trường hợp `GITHUB_TOKEN` đã có quyền truy cập package cần thiết, không đưa PAT cá nhân vào workflow. PAT chỉ là phương án bổ sung khi token mặc định thực sự không đủ quyền với tài nguyên cần dùng. [Dừng] Như vậy, nguyên tắc rất ngắn gọn: **GitHub tự cấp token theo Job; ta chỉ cấp đúng quyền cho Job đó.**

## Slide 19 — Pull image và kiểm tra đúng mức

Ở slide này, chúng ta phân biệt ba mức kiểm tra. Khi `docker pull` thành công, Runner mới chỉ tải được image từ GHCR; điều đó chưa chứng minh Spring Boot đã chạy. Lệnh `docker run --entrypoint java ... -version` đi thêm một bước: nó xác nhận image có Java runtime hoạt động, nhưng vẫn chưa cho biết ứng dụng có kết nối được PostgreSQL hay có mở HTTP endpoint hay không. Muốn kết luận `user-service` đã sẵn sàng phục vụ, ta phải chạy đủ PostgreSQL, network `quickbite-net` và các biến `DB_*`, rồi gọi `/actuator/health`; endpoint trả `UP` mới là health check thành công. Vì vậy, `docker ps` chỉ cho biết container còn chạy; hãy nhớ: pull kiểm tra việc tải image, `java -version` kiểm tra runtime, còn health endpoint mới kiểm tra ứng dụng sẵn sàng.

## Slide 20 — Divider Lesson 05

Lesson cuối là bài thực hành tổng hợp với `user-service`. Chúng ta sẽ build image ở local, dùng PAT để push image lên GHCR, rồi để workflow CI/CD pull image đó về và chạy kiểm tra. Cách tách local và CI này chỉ phục vụ mục tiêu học từng thao tác; trong quy trình thực tế, build, push và deploy nên được tự động hóa hoàn toàn trên CI/CD.

## Slide 21 — Chuẩn bị lab

Bài thực hành có bốn bước và phải làm theo thứ tự. Đầu tiên, ta viết Dockerfile multi-stage và build image `user-service:1.0.0` ở local. Tiếp theo, ta dùng PAT để Docker CLI đăng nhập GHCR. Sau đó, ta gắn tag Registry, push image và kiểm tra tag `1.0.0` trên GitHub Packages. Khi image đã có trên Registry, ta mới cập nhật `ci.yml` để Runner pull image, chạy container kiểm tra rồi dọn dẹp. Nếu đẩy workflow trước khi push image, bước pull sẽ không có image để tải.

## Slide 22 — Bước 1: Build image

Tại thư mục gốc của `user-service`, ta dùng lại Dockerfile multi-stage đã học ở Lesson 02. Builder stage dùng JDK 17 để chạy Gradle và tạo JAR; runtime stage chỉ giữ JRE 17 cùng `app.jar`. Lệnh cần chạy là `docker build -t user-service:1.0.0 .`. Khi lệnh hoàn thành, image `user-service:1.0.0` đã tồn tại ở máy local và sẵn sàng cho bước đăng nhập Registry.

## Slide 23 — Bước 2 và 3: PAT login, tag và push

Vì thao tác này diễn ra trên máy local, Docker CLI cần đăng nhập bằng Personal Access Token, không phải `GITHUB_TOKEN` của workflow. Ta tạo PAT classic có quyền `read:packages` và `write:packages`, rồi chạy `docker login ghcr.io -u <github_username_cua_ban>` và nhập PAT khi Docker hỏi mật khẩu. Sau khi login thành công, gắn image local vào đúng địa chỉ `ghcr.io/<repository_namespace>/user-service:1.0.0`, rồi chạy `docker push` với địa chỉ đó. Bước này chỉ hoàn tất khi mở GitHub Packages và thấy package `user-service` có tag `1.0.0`.

## Slide 24 — Bước 4: Workflow pull và chạy verify

Sau khi image đã có trên GHCR, ta cập nhật `.github/workflows/ci.yml`. Job `test_image` chạy trên self-hosted Runner có nhãn `quickbite`, cấp `packages: read` và `contents: read`, rồi đăng nhập GHCR bằng `${{ secrets.GITHUB_TOKEN }}` do GitHub tự cấp cho Job. Job tạo `IMAGE_TAG` từ tên repository, pull tag `1.0.0`, chạy container `verify_app` ở chế độ nền với cổng `8081:8081`, chờ năm giây, kiểm tra bằng `docker ps` và xóa container bằng `docker rm -f verify_app || true`. Sau khi push `ci.yml` lên `main`, mở tab Actions để xem job `test_image`. Nếu container biến mất khỏi `docker ps`, hãy chạy lại tại local và dùng `docker logs <container_name>` để tìm lỗi cấu hình hoặc cổng.

## Slide 25 — Minh chứng và lỗi thường gặp

Báo cáo của bài thực hành cần có ba minh chứng: ảnh Terminal local cho thấy build và `docker push` thành công, ảnh GitHub Packages có tag `1.0.0`, và log GitHub Actions của job `test_image`. Nếu workflow báo không tìm thấy image, nguyên nhân thường là push `ci.yml` trước khi push image. Nếu báo `denied: unauthenticated`, kiểm tra lại `permissions: packages: read`. Nếu container tạo được nhưng sập ngay, `docker ps` sẽ không hiển thị nó; hãy debug ở local bằng `docker logs <container_name>` trước khi push lại image.

## Slide 26 — Tổng kết Session 08

Trong Session 08, chúng ta đã đi qua toàn bộ vòng đời của một Docker image. Trước hết, image được build trong pipeline để gắn với đúng phiên bản mã nguồn. Tiếp theo, Dockerfile multi-stage tách môi trường dùng để biên dịch khỏi môi trường chỉ dùng để chạy ứng dụng. Khi phát hành, image được gắn tag và lưu trên GitHub Container Registry. Ở máy local, Docker CLI đăng nhập bằng PAT; còn trong GitHub Actions, mỗi Job sử dụng `GITHUB_TOKEN` do GitHub tự cấp với quyền vừa đủ. Cuối cùng, bài thực hành ghép các bước thành một quy trình hoàn chỉnh: build image, push lên GHCR, để `ci.yml` pull image về kiểm tra và dọn dẹp container sau khi chạy.

## Slide 27 — Kết thúc

Nội dung Session 08 kết thúc tại đây. Các bạn hãy hoàn thành bài thực hành, kiểm tra đủ ba minh chứng gồm Terminal local, GitHub Packages và log của job `test_image`, sau đó đối chiếu lại các lỗi thường gặp trước khi nộp bài.
