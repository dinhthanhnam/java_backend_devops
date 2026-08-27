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

Ở lesson trước, chúng ta đã build được image, nhưng image đó mới chỉ tồn tại trên máy local. Nếu Runner hoặc một máy khác cần sử dụng, chúng ta phải đưa image lên Registry. Trong lesson này, chúng ta sẽ gắn cho image một tên và phiên bản rõ ràng, sau đó push lên GitHub Container Registry để các môi trường khác có thể pull về đúng phiên bản cần dùng.

## Slide 14 — Tên image, tag và digest phục vụ ba mục đích khác nhau

Khi đưa image lên Registry, chúng ta cần phân biệt ba thành phần. Image name, ví dụ `ghcr.io/<namespace>/user-service`, là địa chỉ cho biết image được lưu ở đâu. Tag như `1.0.0` là nhãn phiên bản giúp con người dễ chọn image cần dùng. Tuy nhiên, cùng một tag vẫn có thể được push lại và trỏ sang một image mới. Digest `sha256:...` thì khác: nó được tạo từ nội dung image, nên khi nội dung thay đổi, digest cũng thay đổi. Vì vậy, chúng ta dùng tag để quản lý phiên bản cho thuận tiện, còn dùng digest khi cần xác định chính xác một image không bị thay đổi.

## Slide 15 — Push image từ local cần xác thực an toàn và tag có ý nghĩa

Để đưa image từ máy local lên GHCR, trước tiên chúng ta đăng nhập Docker bằng PAT có quyền đọc và ghi package. Sau đó, dùng `docker tag` để gắn cho image local một địa chỉ đầy đủ, ví dụ `ghcr.io/<namespace>/user-service:1.0.0`. Lệnh này chỉ đặt thêm tên cho image hiện có, không build lại image. Cuối cùng, chạy `docker push` để tải image lên GHCR. Khi lệnh hoàn thành, hãy mở GitHub Packages và kiểm tra package `user-service` đã có tag `1.0.0`. Như vậy, ba bước cần nhớ là: đăng nhập, gắn tag và push.

## Slide 16 — Local push giúp hiểu CLI, còn CI mới là đích phát hành đáng tin cậy

Quy trình local vừa thực hiện có mục đích giúp chúng ta nhìn rõ từng thao tác: đăng nhập Registry, build image, gắn tag và push. Tuy nhiên, máy local có thể chứa source chưa commit hoặc có môi trường khác với hệ thống CI, nên image phát hành từ local không bảo đảm tính nhất quán. Trong quy trình thực tế, CI/CD cần chịu trách nhiệm build và push image từ mã nguồn đã được kiểm soát. Khi workflow đã đáp ứng được yêu cầu, nên dùng `GITHUB_TOKEN` với quyền tối thiểu thay vì đưa PAT cá nhân vào pipeline. Vì vậy, hãy xem local push là bài thực hành để hiểu công cụ, không phải cách phát hành production.

## Slide 17 — Divider Lesson 04

Lesson 04 chuyển sang chiều ngược lại: workflow sẽ lấy image đã có trên GHCR về Runner. Chúng ta sẽ đi qua đầy đủ các bước cấp quyền đọc package, đăng nhập Registry, xác định đúng image reference, chạy `docker pull`, rồi kiểm tra image ở nhiều mức khác nhau.

## Slide 18 — Workflow đăng nhập GHCR và pull đúng image

Bây giờ chúng ta thực hiện đúng mục tiêu của lesson: để một Job trên Runner lấy image từ GHCR về sử dụng. Quy trình gồm bốn bước: cấp quyền đọc package, đăng nhập Registry, xác định đúng image cần lấy và chạy lệnh pull.

Trước hết, Job khai báo `packages: read`. Quyền này cho phép `GITHUB_TOKEN` đọc package trên GHCR. Khác với thao tác ở máy local, chúng ta không cần tự tạo PAT rồi lưu vào secrets. Khi Job bắt đầu, GitHub tự cấp `GITHUB_TOKEN`; `docker/login-action` sử dụng token đó cùng `github.actor` để đăng nhập Docker vào `ghcr.io`.

Sau khi đăng nhập, Job sử dụng biến `IMAGE_REF`. Giá trị này phải ghi đủ bốn thành phần: Registry `ghcr.io`, namespace sở hữu package, tên image `user-service` và tag `1.0.0`. Đây chính là địa chỉ để Docker biết phải tải image nào. Lệnh `docker pull "$IMAGE_REF"` yêu cầu GHCR trả về manifest của image, sau đó Docker tải những layer Runner chưa có và lưu image vào Docker daemon trên Runner. Cuối cùng, `docker image inspect "$IMAGE_REF"` xác nhận image với đúng reference đó đã có trên máy.

Như vậy, `GITHUB_TOKEN` chỉ giải quyết bước xác thực; nó không tự pull image. Thao tác pull thực sự vẫn do Docker thực hiện dựa trên `IMAGE_REF`. Mạch cần nhớ là: Job có quyền đọc, Docker đăng nhập GHCR, Docker pull đúng reference, rồi image mới xuất hiện trên Runner để bước tiếp theo sử dụng.

## Slide 19 — Chạy image vừa pull và kiểm tra ứng dụng trên Runner

Ở Slide 18, image mới chỉ được tải về Docker daemon của Runner. Bước tiếp theo là tạo container từ image đó và kiểm tra `user-service` có thực sự khởi động được hay không.

Trước khi chạy, Runner phải có môi trường mà ứng dụng cần. Trong ví dụ này, PostgreSQL đã hoạt động trong network `quickbite-net`, còn file `.env` chứa các biến `DB_*` phù hợp. Lệnh `docker run -d` tạo container `verify_user_service`, nối container vào network, truyền cấu hình database và ánh xạ cổng `8081`. Từ thời điểm này, container mới bắt đầu chạy file JAR bên trong image.

Spring Boot không sẵn sàng ngay lập tức, nên workflow không gọi health endpoint đúng một lần rồi kết luận thất bại. Vòng lặp trên slide gọi `/actuator/health` mỗi năm giây, tối đa mười hai lần. Nếu endpoint phản hồi thành công, script kết thúc với mã `0` và bước kiểm tra được tính là pass. Nếu hết 60 giây mà ứng dụng vẫn chưa sẵn sàng, workflow in `docker logs` để chúng ta thấy nguyên nhân, chẳng hạn sai thông tin database hoặc ứng dụng không mở được cổng, rồi kết thúc với mã lỗi.

Cuối cùng, bước dọn dẹp sử dụng `if: always()`. Điều đó có nghĩa là dù health check pass hay fail, container kiểm tra vẫn được xóa khỏi Runner. Toàn bộ quy trình của Lesson 4 lúc này đã hoàn chỉnh: đăng nhập GHCR, pull đúng image, chạy container từ image, kiểm tra ứng dụng và dọn tài nguyên sau khi hoàn thành.

## Slide 20 — Divider Lesson 05

Lesson cuối ghép các thao tác đã học thành một luồng hoàn chỉnh với `user-service`. Chúng ta sẽ build và push image từ local lên GHCR, sau đó để GitHub Actions đăng nhập Registry, pull đúng image về Runner, chạy ứng dụng cùng PostgreSQL, kiểm tra health endpoint và dọn tài nguyên. Việc tách local và CI ở bài này giúp chúng ta quan sát rõ từng bước; trong quy trình thực tế, phần build và publish cũng nên được tự động hóa trên CI/CD.

## Slide 21 — Kịch bản thực hành đi từ Local tới CI/CD

Bài thực hành có bốn bước và phải làm theo thứ tự. Đầu tiên, ta build image `user-service:1.0.0` tại local. Bước hai dùng PAT để đăng nhập GHCR, gắn Registry tag và push image. Bước ba chuẩn bị môi trường chạy trên Runner: PostgreSQL phải hoạt động trong network `quickbite-net`, còn cấu hình `DB_*` được lưu trong Repository secret `USER_SERVICE_ENV`. Cuối cùng, `ci.yml` đăng nhập GHCR bằng `GITHUB_TOKEN`, pull image, chạy container, chờ health check và luôn dọn tài nguyên. Như vậy, bài thực hành không dừng ở việc Runner tải được image mà phải chứng minh ứng dụng từ image đó thực sự khởi động thành công.

## Slide 22 — Bước 1: Build image multi-stage tại máy local

Tại thư mục gốc của `user-service`, Dockerfile chuẩn bị Gradle Wrapper và file build trước, chạy bước tải dependency, sau đó mới copy thư mục `src` và chạy `bootJar`. Builder stage sử dụng JDK 17 để tạo JAR; runtime stage dùng JRE 17 và chỉ nhận file đó dưới tên `app.jar`. Sau khi kiểm tra Dockerfile, chạy `docker build -t user-service:1.0.0 .`. Khi lệnh hoàn thành, image phiên bản `1.0.0` đã tồn tại ở máy local và có thể chuyển sang bước đăng nhập Registry.

## Slide 23 — Bước 2: PAT login, tag và push image lên GHCR

Vì thao tác này diễn ra trên máy local, Docker CLI cần đăng nhập bằng Personal Access Token, không phải `GITHUB_TOKEN` của workflow. Ta tạo PAT classic có quyền `read:packages` và `write:packages`, rồi chạy `docker login ghcr.io -u <github_username_cua_ban>` và nhập PAT khi Docker hỏi mật khẩu. Sau khi login thành công, gắn image local vào đúng địa chỉ `ghcr.io/<repository_namespace>/user-service:1.0.0`, rồi chạy `docker push` với địa chỉ đó. Bước này chỉ hoàn tất khi mở GitHub Packages và thấy package `user-service` có tag `1.0.0`.

## Slide 24 — Bước 3 và 4: Chuẩn bị môi trường rồi pull và verify

Trước khi chạy workflow, chúng ta phải chuẩn bị dependency của ứng dụng. PostgreSQL cần hoạt động trong network `quickbite-net`. Các biến `DB_*` không ghi trực tiếp vào YAML mà được lưu trong Repository secret `USER_SERVICE_ENV`; khi Job chạy, nội dung secret được ghi tạm vào thư mục `RUNNER_TEMP` để Docker sử dụng qua `--env-file`.

Job `test_image` được cấp `packages: read` và đăng nhập GHCR bằng `GITHUB_TOKEN`. Biến `IMAGE_REF` ghi rõ namespace, tên `user-service` và tag `1.0.0`; nhờ đó, `docker pull` lấy đúng image đã push ở bước trước. Sau khi pull, Job chạy container `verify_user_service`, nối nó vào `quickbite-net`, truyền file môi trường và ánh xạ cổng `8081`.

Workflow gọi `/actuator/health` mỗi năm giây trong tối đa một phút. Nếu ứng dụng sẵn sàng, Job pass. Nếu ứng dụng không lên, `docker logs` được in ngay trong log Actions trước khi Job fail để chúng ta biết lỗi nằm ở database, network hay cấu hình. Bước cuối dùng `if: always()` để xóa container và file môi trường tạm trong cả hai trường hợp thành công và thất bại. Đây chính là quy trình đã học ở Lesson 4 được áp dụng đầy đủ vào bài thực hành.

## Slide 25 — Hoàn tất báo cáo: ba minh chứng và ba lỗi cần tránh

Báo cáo cần chứng minh đủ cả ba chặng của luồng. Minh chứng thứ nhất là Terminal local cho thấy image được build và push thành công. Minh chứng thứ hai là GitHub Packages có `user-service` với tag `1.0.0`. Minh chứng cuối cùng là log của Job `test_image`, trong đó Runner pull đúng `IMAGE_REF` và health check thành công.

Nếu gặp `manifest unknown`, hãy kiểm tra namespace, tên image và tag vì chỉ cần sai một thành phần là Docker sẽ tìm sang reference khác. Nếu gặp `denied`, kiểm tra quyền truy cập package và `packages: read`. Nếu pull thành công nhưng health không lên, image đã đến Runner nên lỗi nằm ở giai đoạn chạy ứng dụng; lúc đó đọc `docker logs`, rồi kiểm tra PostgreSQL, network `quickbite-net` và các biến `DB_*`. Chỉ khi health check pass và bước cleanup hoàn thành thì bài thực hành mới được coi là đạt yêu cầu.

## Slide 26 — Tổng kết Session 08

Trong Session 08, chúng ta đã đi qua toàn bộ vòng đời của một Docker image. Trước hết, image được build trong pipeline để gắn với đúng phiên bản mã nguồn. Tiếp theo, Dockerfile multi-stage tách môi trường dùng để biên dịch khỏi môi trường chỉ dùng để chạy ứng dụng. Khi phát hành, image được gắn tag và lưu trên GitHub Container Registry. Ở máy local, Docker CLI đăng nhập bằng PAT; còn trong GitHub Actions, mỗi Job sử dụng `GITHUB_TOKEN` do GitHub tự cấp với quyền vừa đủ. Cuối cùng, bài thực hành ghép các bước thành một quy trình hoàn chỉnh: build image, push lên GHCR, để `ci.yml` pull image về kiểm tra và dọn dẹp container sau khi chạy.

## Slide 27 — Kết thúc

Nội dung Session 08 kết thúc tại đây. Các bạn hãy hoàn thành bài thực hành, kiểm tra đủ ba minh chứng gồm Terminal local, GitHub Packages và log của job `test_image`, sau đó đối chiếu lại các lỗi thường gặp trước khi nộp bài.
