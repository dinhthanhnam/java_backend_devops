# KỊCH BẢN TRÌNH BÀY — SESSION 08

## Đóng gói Docker Image & Đẩy lên Registry

Kịch bản đi theo `session_08_slide_content.md`. Khi quay từng video riêng, bắt đầu từ divider của lesson tương ứng. Giọng nói bình tĩnh, giải thích rõ nguyên nhân trước khi đọc lệnh; không đọc lại nguyên văn toàn bộ bullet trên slide.

## Slide 01 — Tiêu đề chính

Chào các bạn. Session này nối tiếp bước build JAR ở CI bằng một đầu ra gần với vận hành hơn: Docker image. Chúng ta sẽ đi từ cách build image, tổ chức Dockerfile, đặt version, lưu image lên Registry cho đến cách một workflow khác lấy đúng image đó về kiểm tra. Ví dụ xuyên suốt là `user-service` của QuickBite.

## Slide 02 — Nội dung session

Session gồm năm lesson nối tiếp nhau. Lesson đầu đưa bước build image vào pipeline hai Job. Lesson hai chuyển sang Dockerfile multi-stage và tối ưu cách tổ chức layer. Lesson ba giới thiệu image name, tag, digest và cách push image từ local lên GHCR. Lesson bốn sử dụng `GITHUB_TOKEN` để pull image và phân biệt các mức kiểm tra. Lesson cuối ghép toàn bộ thao tác thành một bài thực hành hoàn chỉnh với `user-service`.

## Slide 03 — Divider Lesson 01

Trong Lesson 01, ta trả lời câu hỏi: nếu JAR đã được CI tạo ra, làm sao đóng gói JAR đó thành image mà vẫn truy vết được nó xuất phát từ job nào và commit nào?

## Slide 04 — Vì sao image cũng phải được build trong pipeline

Nếu CI chỉ build JAR còn image được tạo thủ công ở local, hai đầu ra có thể không còn cùng một trạng thái source. Máy local có thể có thay đổi chưa commit, Dockerfile khác revision, hoặc JAR cũ. Đưa Docker build vào pipeline không tự động loại bỏ mọi rủi ro, nhưng tạo được đường đi có thể kiểm tra: run nào tạo JAR, job nào tạo image và log nào xác nhận kết quả. Đó là giá trị cốt lõi của CI trong tình huống này.

## Slide 05 — Hai jobs xác định rõ điểm chuyển giao từ JAR sang image

Trong mô hình này, Job `build_jar` chạy Gradle, tạo file JAR rồi upload kết quả dưới tên artifact `app-jar`. Job `build_image` khai báo `needs: build_jar`, vì vậy nó chỉ bắt đầu khi Job trước hoàn thành thành công. Sau đó Job này checkout lại mã nguồn để lấy Dockerfile, download JAR vào thư mục `build/libs` và chạy `docker build`. Mỗi Job có không gian làm việc riêng, nên file JAR không tự xuất hiện ở Job sau; `needs` điều khiển thứ tự, còn artifact đảm nhiệm việc truyền file.

## Slide 06 — Docker socket là năng lực build image và cũng là quyền nhạy cảm

Self-hosted Runner của QuickBite mount `/var/run/docker.sock`, vì vậy lệnh `docker build` trong workflow được chuyển trực tiếp tới Docker daemon của máy host. Cách làm này không cần khởi tạo Docker-in-Docker và có thể tận dụng những image cùng layer cache đã có trên host. Tuy nhiên, Docker socket không phải một volume dữ liệu thông thường; Job truy cập được socket cũng có quyền rất cao đối với Docker host. Vì vậy, Runner này chỉ nên chạy workflow, action và mã nguồn đã được kiểm soát.

## Slide 07 — Demo: Build image từ JAR artifact trong workflow

Ở slide này, workflow image được đặt trong file `image.yml`, nằm song song với `ci.yml` hiện có. Job `build_image` chờ `build_jar` hoàn thành thông qua `needs`, checkout mã nguồn để lấy Dockerfile, rồi download artifact `app-jar` vào thư mục `build/libs`. Dockerfile đơn tầng lấy JAR từ thư mục đó và đóng gói thành image `user-service:ci`. Sau khi build, lệnh `docker image inspect` được dùng để xác nhận image đã tồn tại trên Runner. Phần demo này chỉ dừng ở bước build; việc gắn version và push lên Registry sẽ được thực hiện ở Lesson 03.

## Slide 08 — Divider Lesson 02

Lesson 02 chuyển sang Dockerfile tự đủ. Ta không bỏ mô hình artifact ở lesson trước; ta dùng nó để hiểu rõ hơn vì sao multi-stage build lại tách môi trường build khỏi runtime.

## Slide 09 — Multi-stage build tách môi trường build khỏi môi trường chạy

Builder stage cần JDK, Gradle Wrapper, source và dependency để tạo JAR. Runtime stage chỉ cần JRE phù hợp cùng JAR đã tạo. `COPY --from=builder` là ranh giới rõ ràng: chỉ artifact cần chạy đi sang runtime. Nhờ đó source code, Gradle Wrapper và công cụ biên dịch không trở thành thành phần bắt buộc của runtime image.

## Slide 10 — Dockerfile multi-stage của `user-service` phải nhất quán Java 17

Dockerfile này có hai stage nhưng cùng phải thống nhất Java 17. Builder stage dùng JDK 17, chuẩn bị Gradle Wrapper và các file cấu hình trước, tải dependency, rồi mới copy source và chạy `bootJar`. Runtime stage dùng JRE 17 và chỉ nhận file `user-service.jar` từ builder dưới tên `app.jar`. Cách tách này giúp image cuối không phải mang theo source, Gradle và JDK. Khi áp dụng cho service khác, cần kiểm tra đúng phiên bản Java của service đó thay vì sao chép nguyên base image của `user-service`.

## Slide 11 — Thứ tự layer quyết định khả năng tái sử dụng cache Gradle

Để Docker cache có cơ hội tái sử dụng phần dependency, ta copy Gradle Wrapper và các file build trước, chạy bước tải dependency, sau đó mới copy thư mục `src` và chạy `bootJar`. Khi chỉ source Java thay đổi, các layer phía trước vẫn có thể còn hợp lệ và không phải thực hiện lại toàn bộ. Đây chỉ là khả năng tái sử dụng cache, không phải cam kết rằng lần build nào cũng nhanh. Bên cạnh đó, `.dockerignore` cần loại `.git`, `.gradle`, `build` và `.env` để build context gọn hơn và không đưa file local hoặc cấu hình nhạy cảm vào builder.

## Slide 12 — Demo: So sánh single-stage và multi-stage bằng cùng một source

Trong phần demo, chúng ta build một image bằng Dockerfile single-stage và một image bằng Dockerfile multi-stage từ cùng một source. Sau đó dùng `docker image ls` để xem các image đã tạo và `docker history` để quan sát các layer của bản multi-stage. Điểm cần so sánh không chỉ là dung lượng, mà là thành phần đi vào runtime image: bản multi-stage chỉ nhận JAR từ builder, không cần giữ source hoặc Gradle Wrapper. Kết quả cụ thể có thể khác nhau tùy base image và cache, vì vậy không nên biến một con số dung lượng hoặc thời gian build thành kết luận cố định.

## Slide 13 — Divider Lesson 03

Image vừa build chỉ nằm trên Docker daemon hiện tại. Lesson này đưa image sang GitHub Container Registry để Runner hoặc máy khác có thể truy xuất lại bằng một reference rõ ràng.

## Slide 14 — Tên image, tag và digest phục vụ ba mục đích khác nhau

Ba thành phần trên slide trả lời ba câu hỏi khác nhau. Image name `ghcr.io/<namespace>/user-service` cho biết package nằm ở đâu. Tag như `1.0.0` là nhãn phiên bản dễ đọc để chọn release, nhưng tag vẫn có thể bị push lại. Digest `sha256:...` gắn với một nội dung image cụ thể, nên phù hợp khi cần khóa đúng bản đã kiểm chứng. Vì vậy, tag thuận tiện cho việc quản lý phiên bản, còn digest giúp tái lập chính xác; riêng `latest` không đủ rõ ràng để làm căn cứ rollback production.

## Slide 15 — Push image từ local cần xác thực an toàn và tag có ý nghĩa

Khi push từ local, Docker CLI đăng nhập GHCR bằng PAT có quyền package phù hợp. Token được truyền qua standard input để không xuất hiện trực tiếp trong câu lệnh, Dockerfile hoặc YAML. Sau khi login, `docker tag` tạo thêm reference `ghcr.io/<namespace>/user-service:1.0.0` cho image local; thao tác này không build lại image. Lệnh `docker push` tiếp tục tải các layer lên Registry và trả về thông tin digest. Để kiểm tra kết quả, cần xem log push thành công, xác nhận package trên GitHub và ghi lại image reference cùng digest thay vì chỉ dựa vào một ảnh chụp giao diện.

## Slide 16 — Local push giúp hiểu CLI, còn CI mới là đích phát hành đáng tin cậy

Quy trình local vừa thực hiện có mục đích giúp chúng ta nhìn rõ từng thao tác: đăng nhập Registry, build image, gắn tag và push. Tuy nhiên, máy local có thể chứa source chưa commit hoặc có môi trường khác với hệ thống CI, nên image phát hành từ local không bảo đảm tính nhất quán. Trong quy trình thực tế, CI/CD cần chịu trách nhiệm build và push image từ mã nguồn đã được kiểm soát. Khi workflow đã đáp ứng được yêu cầu, nên dùng `GITHUB_TOKEN` với quyền tối thiểu thay vì đưa PAT cá nhân vào pipeline. Vì vậy, hãy xem local push là bài thực hành để hiểu công cụ, không phải cách phát hành production.

## Slide 17 — Divider Lesson 04

Lesson 04 chuyển sang cách workflow sử dụng image đã có trên Registry. Trọng tâm là hiểu `GITHUB_TOKEN` được GitHub cấp như thế nào, Job pull và Job publish cần quyền gì, rồi phân biệt giữa việc pull thành công, kiểm tra Java runtime và kiểm tra ứng dụng đã thực sự sẵn sàng.

## Slide 18 — `GITHUB_TOKEN`: GitHub tự cấp cho từng Job

`GITHUB_TOKEN` không phải token chúng ta tự tạo rồi lưu vào Repository secrets. Mỗi khi một Job bắt đầu, GitHub tự cấp cho Job đó một token riêng; workflow chỉ tham chiếu token qua `${{ secrets.GITHUB_TOKEN }}` hoặc `github.token`. Khi Job kết thúc, token của Job đó cũng hết hiệu lực. Block `permissions` không tạo token mà chỉ giới hạn token được phép làm gì: Job pull image cần `packages: read`, còn Job publish image cần `packages: write`. Hai Job nhận hai token riêng, không dùng chung một token cho toàn workflow. Nếu `GITHUB_TOKEN` đã đủ quyền truy cập package, không cần đưa PAT cá nhân vào workflow.

## Slide 19 — Pull thành công chưa có nghĩa ứng dụng đã sẵn sàng

Ở slide này, chúng ta phân biệt ba mức kiểm tra. Khi `docker pull` thành công, Runner mới chỉ tải được image từ GHCR; điều đó chưa chứng minh Spring Boot đã chạy. Lệnh `docker run --entrypoint java ... -version` đi thêm một bước khi xác nhận Java runtime bên trong image hoạt động, nhưng vẫn chưa cho biết ứng dụng có kết nối được database hoặc mở HTTP endpoint hay không. Muốn kết luận `user-service` đã sẵn sàng phục vụ, cần chạy đủ PostgreSQL, network `quickbite-net` và các biến `DB_*`, rồi gọi `/actuator/health`; endpoint trả `UP` mới là health check thành công. `docker ps` chỉ cho biết container còn chạy, không thay thế được health check của ứng dụng.

## Slide 20 — Divider Lesson 05

Lesson cuối là bài thực hành tổng hợp với `user-service`. Chúng ta sẽ build image ở local, dùng PAT để push image lên GHCR, rồi để workflow CI/CD pull image đó về và chạy kiểm tra. Cách tách local và CI này chỉ phục vụ mục tiêu học từng thao tác; trong quy trình thực tế, build, push và deploy nên được tự động hóa hoàn toàn trên CI/CD.

## Slide 21 — Kịch bản thực hành đi từ Local tới CI/CD

Bài thực hành có bốn bước và phải làm theo thứ tự. Đầu tiên, ta viết Dockerfile multi-stage và build image `user-service:1.0.0` ở local. Tiếp theo, ta dùng PAT để Docker CLI đăng nhập GHCR. Sau đó, ta gắn tag Registry, push image và kiểm tra tag `1.0.0` trên GitHub Packages. Khi image đã có trên Registry, ta mới cập nhật `ci.yml` để Runner pull image, chạy container kiểm tra rồi dọn dẹp. Nếu đẩy workflow trước khi push image, bước pull sẽ không có image để tải.

## Slide 22 — Bước 1: Build image multi-stage tại máy local

Tại thư mục gốc của `user-service`, Dockerfile chuẩn bị Gradle Wrapper và file build trước, chạy bước tải dependency, sau đó mới copy thư mục `src` và chạy `bootJar`. Builder stage sử dụng JDK 17 để tạo JAR; runtime stage dùng JRE 17 và chỉ nhận file đó dưới tên `app.jar`. Sau khi kiểm tra Dockerfile, chạy `docker build -t user-service:1.0.0 .`. Khi lệnh hoàn thành, image phiên bản `1.0.0` đã tồn tại ở máy local và có thể chuyển sang bước đăng nhập Registry.

## Slide 23 — Bước 2 và 3: PAT login, tag và push image lên GHCR

Vì thao tác này diễn ra trên máy local, Docker CLI cần đăng nhập bằng Personal Access Token, không phải `GITHUB_TOKEN` của workflow. Ta tạo PAT classic có quyền `read:packages` và `write:packages`, rồi chạy `docker login ghcr.io -u <github_username_cua_ban>` và nhập PAT khi Docker hỏi mật khẩu. Sau khi login thành công, gắn image local vào đúng địa chỉ `ghcr.io/<repository_namespace>/user-service:1.0.0`, rồi chạy `docker push` với địa chỉ đó. Bước này chỉ hoàn tất khi mở GitHub Packages và thấy package `user-service` có tag `1.0.0`.

## Slide 24 — Bước 4: `ci.yml` pull image và chạy verify trên Runner

Sau khi image đã có trên GHCR, ta cập nhật `.github/workflows/ci.yml`. Job `test_image` chạy trên self-hosted Runner có nhãn `quickbite`, cấp `packages: read` và `contents: read`, rồi đăng nhập GHCR bằng `${{ secrets.GITHUB_TOKEN }}` do GitHub tự cấp cho Job. Job tạo `IMAGE_TAG` từ tên repository, pull tag `1.0.0`, chạy container `verify_app` ở chế độ nền với cổng `8081:8081`, chờ năm giây, kiểm tra bằng `docker ps` và xóa container bằng `docker rm -f verify_app || true`. Sau khi push `ci.yml` lên `main`, mở tab Actions để xem job `test_image`. Nếu container biến mất khỏi `docker ps`, hãy chạy lại tại local và dùng `docker logs <container_name>` để tìm lỗi cấu hình hoặc cổng.

## Slide 25 — Hoàn tất báo cáo: ba minh chứng và ba lỗi cần tránh

Báo cáo của bài thực hành cần có ba minh chứng: ảnh Terminal local cho thấy build và `docker push` thành công, ảnh GitHub Packages có tag `1.0.0`, và log GitHub Actions của job `test_image`. Nếu workflow báo không tìm thấy image, nguyên nhân thường là push `ci.yml` trước khi push image. Nếu báo `denied: unauthenticated`, kiểm tra lại `permissions: packages: read`. Nếu container tạo được nhưng sập ngay, `docker ps` sẽ không hiển thị nó; hãy debug ở local bằng `docker logs <container_name>` trước khi push lại image.

## Slide 26 — Tổng kết Session 08

Trong Session 08, chúng ta đã đi qua toàn bộ vòng đời của một Docker image. Trước hết, image được build trong pipeline để gắn với đúng phiên bản mã nguồn. Tiếp theo, Dockerfile multi-stage tách môi trường dùng để biên dịch khỏi môi trường chỉ dùng để chạy ứng dụng. Khi phát hành, image được gắn tag và lưu trên GitHub Container Registry. Ở máy local, Docker CLI đăng nhập bằng PAT; còn trong GitHub Actions, mỗi Job sử dụng `GITHUB_TOKEN` do GitHub tự cấp với quyền vừa đủ. Cuối cùng, bài thực hành ghép các bước thành một quy trình hoàn chỉnh: build image, push lên GHCR, để `ci.yml` pull image về kiểm tra và dọn dẹp container sau khi chạy.

## Slide 27 — Kết thúc

Nội dung Session 08 kết thúc tại đây. Các bạn hãy hoàn thành bài thực hành, kiểm tra đủ ba minh chứng gồm Terminal local, GitHub Packages và log của job `test_image`, sau đó đối chiếu lại các lỗi thường gặp trước khi nộp bài.
