# NỘI DUNG SLIDE — SESSION 08

## Đóng gói Docker Image & Đẩy lên Registry

Tài liệu này là nội dung trực tiếp để dựng Google Slides. Bố cục, typography và nhịp trình bày bám theo file mẫu `(Slide) Session 7.pptx`: slide đầu là tiêu đề, slide kế tiếp là agenda, mỗi lesson bắt đầu bằng một divider nền đỏ, và phần cuối gồm tổng kết cùng slide kết thúc. Số slide được quyết định bởi mức độ cần thiết để giải thích rõ nội dung, không bị giới hạn theo số slide của file mẫu.

---

## Slide 01 — Tiêu đề chính

**Session 08:**

**Đóng gói Docker Image & Đẩy lên Registry**

Không thêm câu dẫn, sơ đồ hay bullet trên slide này.

---

## Slide 02 — Nội dung session

1. Quy trình Build Docker image trong pipeline CI/CD
2. Tối ưu hóa Dockerfile cho Production (Multi-stage build)
3. Phiên bản hóa và đẩy Docker image lên Registry từ Local
4. Sử dụng Docker Image từ Registry trong Luồng CI/CD
5. Kịch bản Thực hành Tổng hợp với user-service

---

## Slide 03 — Divider Lesson 01

**01**

**Quy trình Build Docker image trong pipeline CI/CD**

---

## Slide 04 — Vì sao image cũng phải được build trong pipeline

### Sự lệch pha giữa artifact CI và image tạo thủ công

**Build image ở local**

- CI chỉ xác nhận JAR được build từ commit đã push.
- Image lại có thể được tạo từ source, Dockerfile hoặc JAR đang thay đổi trên máy cá nhân.
- Khi cần truy vết lỗi, khó khẳng định image đã phát hành tương ứng với commit nào.

**Build image trong pipeline**

- Bước đóng gói trở thành một job có log, trạng thái và điều kiện chạy rõ ràng.
- Image được tạo từ đúng revision đã đi qua bước kiểm tra CI.
- Tag image tạo đầu vào có thể nhận diện cho Registry hoặc bước kiểm tra kế tiếp.

*Minh họa:* hai luồng song song: `CI build JAR → local build image` và `CI build JAR → CI build image`. Luồng thứ hai được tô nổi bật.

---

## Slide 05 — Hai jobs xác định rõ điểm chuyển giao từ JAR sang image

### Artifact giúp jobs trao đổi đúng tệp đầu ra

```text
git push
   │
   ├── build_jar: ./gradlew bootJar
   │       └── upload app-jar
   │
   └── build_image (needs: build_jar)
           ├── checkout Dockerfile
           ├── download app-jar
           └── docker build
```

- `build_jar` chịu trách nhiệm biên dịch và lưu JAR dưới tên artifact `app-jar`.
- `build_image` chỉ được chạy khi `build_jar` thành công nhờ `needs`.
- Artifact là cơ chế truyền file có chủ đích giữa hai job; không nên giả định workspace của job trước vẫn còn ở job sau.

*Minh họa:* pipeline nằm ở giữa slide; một mũi tên artifact nối hai job và ghi rõ `app-jar`.

---

## Slide 06 — Docker socket là năng lực build image và cũng là quyền nhạy cảm

### Self-hosted Runner của QuickBite truy cập Docker daemon của host

**Lợi ích vận hành**

- Runner mount `/var/run/docker.sock` để lệnh `docker build` giao tiếp với Docker daemon của host.
- Có thể dùng cache layer sẵn có trên builder host và không cần chạy Docker-in-Docker.
- Phù hợp khi Runner được quản trị trên máy chủ tin cậy.

**Rủi ro phải kiểm soát**

- Job truy cập được socket có quyền rất cao đối với Docker host.
- Chỉ chạy workflow, action và mã nguồn đã được tin cậy, đặc biệt với pull request từ nguồn bên ngoài.
- Không xem Docker socket là một volume dữ liệu thông thường.

*Minh họa:* `Runner container → /var/run/docker.sock → Docker daemon trên host`; gắn biểu tượng cảnh báo tại socket.

---

## Slide 07 — Demo: Build image từ JAR artifact trong workflow

### Tạo `image.yml` song song với workflow CI hiện có

```yaml
build_image:
  needs: build_jar
  runs-on: [self-hosted, quickbite]
  steps:
    - uses: actions/checkout@v5
    - uses: actions/download-artifact@v4
      with: { name: app-jar, path: build/libs }
    - run: docker build -t user-service:ci .
```

**Giải thích và lưu ý**

- Thêm `user-service/.github/workflows/image.yml`; không ghi đè `ci.yml` đang có.
- Dockerfile đơn tầng của `user-service` sao chép `build/libs/user-service.jar` vào image Java 17.
- Job chỉ hoàn thành khi `docker image inspect user-service:ci` trả exit code `0` trên Runner.
- Demo này dừng ở bước build; việc gắn phiên bản và push Registry được thực hiện ở Lesson 03.

---

## Slide 08 — Divider Lesson 02

**02**

**Tối ưu hóa Dockerfile cho Production (Multi-stage build)**

---

## Slide 09 — Multi-stage build tách môi trường build khỏi môi trường chạy

### Chỉ JAR cần đi từ builder sang runtime

**Builder stage**

- Cần JDK, Gradle Wrapper, source code và dependency để biên dịch ứng dụng.
- Tạo JAR bằng `./gradlew bootJar`.

**Runtime stage**

- Chỉ cần JRE phù hợp với service và JAR đã được tạo.
- Nhận artifact qua `COPY --from=builder`.

**Ý nghĩa**

- Runtime image không mang theo Gradle Wrapper, source code và công cụ biên dịch nếu chúng không cần thiết khi chạy.

*Minh họa:* hai khối lớn `builder` và `runtime`; chỉ `user-service.jar` đi qua mũi tên ở giữa.

---

## Slide 10 — Dockerfile multi-stage của `user-service` phải nhất quán Java 17

### Builder dùng JDK, runtime dùng JRE

```dockerfile
FROM eclipse-temurin:17-jdk-alpine AS builder
WORKDIR /app
COPY gradlew ./
COPY gradle ./gradle
COPY build.gradle settings.gradle ./
RUN chmod +x gradlew && ./gradlew dependencies --no-daemon
COPY src ./src
RUN ./gradlew bootJar --no-daemon

FROM eclipse-temurin:17-jre-alpine AS runtime
WORKDIR /app
COPY --from=builder /app/build/libs/user-service.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Lưu ý:** `user-service` dùng Java 17. `restaurant-service`, `order-service` và `notification-service` phải dùng base image Java 21 tương ứng.

---

## Slide 11 — Thứ tự layer quyết định khả năng tái sử dụng cache Gradle

### Copy các tệp thay đổi ít trước source Java

```dockerfile
COPY gradlew ./
COPY gradle ./gradle
COPY build.gradle settings.gradle ./
RUN ./gradlew dependencies --no-daemon

COPY src ./src
RUN ./gradlew bootJar --no-daemon
```

- Các tệp cấu hình Gradle thường thay đổi ít hơn `src/`, vì vậy layer dependency có cơ hội được tái sử dụng khi chỉ sửa source.
- Cache chỉ có hiệu lực khi builder còn cache hợp lệ; multi-stage build không tự động khiến mọi lần build nhanh hơn.
- `.dockerignore` nên loại `.git/`, `.gradle/`, `build/` và `.env` để build context nhỏ, sạch và không mang cấu hình nhạy cảm vào builder.

*Minh họa:* chồng layer từ `Gradle files` đến `Dependencies`, `Source` và `JAR`; đánh dấu layer dependency là “có thể tái sử dụng”.

---

## Slide 12 — Demo: So sánh single-stage và multi-stage bằng cùng một source

### Quan sát thành phần runtime thay vì chỉ nhìn dung lượng image

```bash
docker build -t user-service:single -f Dockerfile.single .
docker build -t user-service:multi .
docker image ls user-service
docker history user-service:multi
```

**Tiêu chí quan sát**

- Runtime image multi-stage chỉ nhận JAR từ builder stage.
- Lịch sử layer cho thấy source và Gradle Wrapper không cần hiện diện trong runtime stage.
- Không đưa ra cam kết thời gian build hoặc kích thước cố định; kết quả phụ thuộc base image, cache và thay đổi của source.

---

## Slide 13 — Divider Lesson 03

**03**

**Phiên bản hóa và đẩy Docker image lên Registry từ Local**

---

## Slide 14 — Tên image, tag và digest phục vụ ba mục đích khác nhau

### Registry lưu image theo địa chỉ và định danh phiên bản

**Image name**

`ghcr.io/<namespace>/user-service` xác định package và nơi image được lưu.

**Tag**

`1.0.0` là nhãn version dễ đọc, tiện chọn release, nhưng có thể bị push lại.

**Digest**

`sha256:...` xác định nội dung image cụ thể; dùng khi cần tái lập chính xác bản đã kiểm chứng.

*Minh họa:* một image name dẫn đến tag `1.0.0`, tag `latest` và một digest cố định. Nêu rõ `latest` không đủ rõ để rollback production.

---

## Slide 15 — Push image từ local cần xác thực an toàn và tag có ý nghĩa

### Đăng nhập GHCR qua standard input, không để token trong câu lệnh

```bash
export CR_PAT='<token được lưu ngoài repository>'
printf '%s' "$CR_PAT" \
  | docker login ghcr.io -u <github_username> --password-stdin

docker tag user-service:1.0.0 \
  ghcr.io/<namespace>/user-service:1.0.0
docker push ghcr.io/<namespace>/user-service:1.0.0
```

**Giải thích và lưu ý**

- PAT classic cần đúng quyền, ví dụ `write:packages` để push và `read:packages` để pull private package.
- Không dùng mật khẩu GitHub; không đặt token vào Dockerfile, YAML, `.env` bị commit, ảnh slide hay log.
- Sau khi push, ghi lại digest Docker in ra. Digest là bằng chứng cho nội dung image đã phát hành.

---

## Slide 16 — Local push giúp hiểu CLI, còn CI mới là đích phát hành đáng tin cậy

### Local để thực hành, CI để phát hành

- Push image từ local giúp người học nắm được quy trình đăng nhập registry, đặt tên image, gắn tag và kiểm tra kết quả `docker push`.
- Tuy nhiên, image tạo từ local có thể chứa thay đổi chưa được commit hoặc chưa qua kiểm tra đầy đủ. Vì vậy, local push chỉ nên dùng cho thực hành và thử nghiệm.
- Trong quy trình phát hành, CI cần build và publish image từ một commit đã được kiểm tra thành công. Nhờ đó, image trên registry có thể truy vết về đúng phiên bản mã nguồn.
- Với GitHub Actions, nên dùng `GITHUB_TOKEN` cùng quyền tối thiểu cần thiết thay cho PAT cá nhân, nếu token mặc định đáp ứng được yêu cầu.

*Minh họa:* bên trái là `Local practice`, bên phải là `CI publish from verified commit`, nối bằng mũi tên thể hiện quá trình chuyển từ thực hành sang phát hành.
---

## Slide 17 — Divider Lesson 04

**04**

**Sử dụng Docker Image từ Registry trong Luồng CI/CD**

---

## Slide 18 — Workflow đăng nhập GHCR và pull đúng image

### Luồng thực hiện: cấp quyền → đăng nhập → pull → xác nhận

```yaml
jobs:
  pull_image:
    runs-on: [self-hosted, quickbite]
    permissions:
      packages: read
    env:
      IMAGE_REF: ghcr.io/<namespace>/user-service:1.0.0
    steps:
      - name: Đăng nhập GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Pull và kiểm tra image
        run: |
          docker pull "$IMAGE_REF"
          docker image inspect "$IMAGE_REF"
```

- `packages: read` cho phép token của Job đọc package trên GHCR.
- `docker/login-action` dùng `GITHUB_TOKEN` do GitHub tự cấp để Docker xác thực với `ghcr.io`; không cần tạo PAT cho workflow.
- `IMAGE_REF` phải chứa đủ Registry, namespace, tên image và tag cần lấy.
- `docker pull` tải manifest cùng các layer về Docker daemon của Runner; `docker image inspect` xác nhận image đã tồn tại tại đó.
- Job khác cần pull image sẽ nhận token riêng và cũng phải được cấp quyền phù hợp.

**Minh họa trên slide:** `Job bắt đầu` → `packages: read` → `Login ghcr.io` → `Pull IMAGE_REF` → `Image có trên Runner`.
---

## Slide 19 — Chạy image vừa pull và kiểm tra ứng dụng trên Runner

### Pull xong phải chạy container và kiểm tra health endpoint

**Điều kiện trước khi chạy:** PostgreSQL đã hoạt động trong network `quickbite-net`; file `.env` chứa đúng các biến `DB_*` của `user-service`.

```yaml
- name: Chạy và kiểm tra user-service
  run: |
    docker run -d \
      --name verify_user_service \
      --network quickbite-net \
      --env-file .env \
      -p 8081:8081 \
      "$IMAGE_REF"

    for i in {1..12}; do
      curl --fail http://localhost:8081/actuator/health && exit 0
      sleep 5
    done

    docker logs verify_user_service
    exit 1

- name: Dọn container
  if: always()
  run: docker rm -f verify_user_service || true
```

- `docker run -d` tạo container từ image vừa pull và chạy ứng dụng ở chế độ nền.
- `--network` và `--env-file` cung cấp kết nối cùng cấu hình database mà ứng dụng cần.
- Vòng lặp chờ tối đa 60 giây vì Spring Boot cần thời gian khởi động; health trả thành công thì Job mới pass.
- Nếu ứng dụng không sẵn sàng, `docker logs` cung cấp nguyên nhân trước khi Job fail.
- `if: always()` bảo đảm container được dọn dù bước kiểm tra thành công hay thất bại.

**Luồng trên Runner:** `Image đã pull` → `docker run` → `Health thành công` → `Dọn container`.

---

## Slide 20 — Divider Lesson 05

**05**

**Kịch bản Thực hành Tổng hợp với user-service**

---

## Slide 21 — Kịch bản thực hành đi từ Local tới CI/CD

### Bốn bước phải thực hiện theo đúng thứ tự

1. Build image `user-service:1.0.0` tại máy local.
2. Dùng PAT để đăng nhập, gắn tag Registry và push image lên GHCR.
3. Chuẩn bị PostgreSQL, network `quickbite-net` và secret `USER_SERVICE_ENV` cho Runner.
4. Cập nhật `ci.yml` để đăng nhập GHCR, pull image, chạy health check và dọn dẹp.

**Lưu ý:** quy trình local build/push rồi CI pull/verify chỉ dùng để học từng thao tác. Trong môi trường thực tế, build, push và deploy nên chạy tự động trên CI/CD.

*Minh họa:* `Local build & push` → `GHCR` → `Runner pull` → `Run + health check` → `Cleanup`.

---

## Slide 22 — Bước 1: Build image multi-stage tại máy local

### Dùng Dockerfile đã học ở Lesson 02 cho `user-service`

```bash
docker build -t user-service:1.0.0 .
```

- Dockerfile có builder stage dùng JDK 17 để tạo JAR và runtime stage dùng JRE 17 để chạy `app.jar`.
- Chạy lệnh tại thư mục gốc của `user-service`.
- Chỉ chuyển sang bước tiếp theo khi build local hoàn thành và đã có image `user-service:1.0.0`.

---

## Slide 23 — Bước 2: PAT login, tag và push image lên GHCR

### Local dùng PAT; workflow ở bước sau dùng `GITHUB_TOKEN`

```bash
docker login ghcr.io -u <github_username_cua_ban>

docker tag user-service:1.0.0 ghcr.io/<repository_namespace>/user-service:1.0.0
docker push ghcr.io/<repository_namespace>/user-service:1.0.0
```

- PAT classic dùng cho Docker CLI local cần quyền `read:packages` và `write:packages`; không dùng mật khẩu tài khoản GitHub.
- Thay `<repository_namespace>` bằng username hoặc Organization sở hữu package.
- Sau khi push, mở GitHub **Packages** và xác nhận `user-service` có tag `1.0.0`.

---

## Slide 24 — Bước 3 và 4: Chuẩn bị môi trường rồi pull và verify

### PostgreSQL phải sẵn sàng; workflow dùng `GITHUB_TOKEN` để pull image

Tạo Repository secret `USER_SERVICE_ENV` chứa các biến `DB_*`. Trên Runner, PostgreSQL phải chạy trong network `quickbite-net` trước khi Job bắt đầu.

```yaml
jobs:
  test_image:
    runs-on: [self-hosted, quickbite]
    permissions:
      packages: read
    env:
      IMAGE_REF: ghcr.io/<namespace>/user-service:1.0.0
      USER_SERVICE_ENV: ${{ secrets.USER_SERVICE_ENV }}
    steps:
      - name: Đăng nhập GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Pull và kiểm tra user-service
        run: |
          printf '%s' "$USER_SERVICE_ENV" > "$RUNNER_TEMP/user-service.env"
          docker pull "$IMAGE_REF"
          docker run -d --name verify_user_service \
            --network quickbite-net \
            --env-file "$RUNNER_TEMP/user-service.env" \
            -p 8081:8081 "$IMAGE_REF"

          for i in {1..12}; do
            curl --silent --fail http://localhost:8081/actuator/health && exit 0
            sleep 5
          done

          docker logs verify_user_service
          exit 1

      - name: Dọn tài nguyên kiểm tra
        if: always()
        run: |
          docker rm -f verify_user_service || true
          rm -f "$RUNNER_TEMP/user-service.env"
```

- Job chỉ pass khi `/actuator/health` phản hồi thành công trong tối đa 60 giây.
- Nếu ứng dụng không lên, workflow in `docker logs` trước khi báo lỗi.
- Bước `if: always()` luôn xóa container và file môi trường tạm khỏi self-hosted Runner.

---

## Slide 25 — Hoàn tất báo cáo: ba minh chứng và ba lỗi cần tránh

### Chỉ coi lab hoàn thành khi đủ bằng chứng

| Ba minh chứng cần nộp | Nếu gặp lỗi |
|---|---|
| Terminal local build và `docker push` thành công | `manifest unknown`: kiểm tra lại namespace, image name và tag trong `IMAGE_REF` |
| GitHub Packages có `user-service:1.0.0` | `denied`: kiểm tra quyền package và `packages: read` |
| Job `test_image` pull được image và health check pass | Health không lên: đọc `docker logs`, kiểm tra PostgreSQL, network và các biến `DB_*` |

**Kết quả cần đạt:** image được build và push lên GHCR; Runner pull đúng reference, chạy `user-service`, nhận health thành công và dọn sạch tài nguyên kiểm tra.

---

## Slide 26 — Tổng kết Session 08

**TỔNG KẾT**

- Build image trong CI để bảo đảm đúng phiên bản mã nguồn đã kiểm tra.
- Multi-stage dùng JDK để build, chỉ giữ JRE và JAR ở runtime.
- Tag dùng để quản lý phiên bản; GHCR lưu trữ và phân phối image.
- Máy local dùng PAT; workflow dùng `GITHUB_TOKEN` theo từng Job.
- `ci.yml` thực hiện pull image, chạy verify và dọn dẹp container.

---

## Slide 27 — Kết thúc

**KẾT THÚC**

**HỌC VIỆN ĐÀO TẠO LẬP TRÌNH CHẤT LƯỢNG NHẬT BẢN**
