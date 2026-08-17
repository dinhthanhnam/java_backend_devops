# NỘI DUNG SLIDE — SESSION 07: TỰ ĐỘNG HÓA QUY TRÌNH CI/CD VỚI GITHUB ACTIONS

## Slide map

| # | Lesson | Loại | Trọng tâm |
|---:|---|---|---|
| 1–2 | Mở đầu | Title, agenda | Mục tiêu buổi học |
| 3–8 | Lesson 01 | Divider, theory, demo | CI/CD và Runner |
| 9–13 | Lesson 02 | Divider, theory, demo | Workflow YAML đầu tiên |
| 14–17 | Lesson 03 | Divider, theory, demo | Jobs, `needs`, isolation |
| 18–21 | Lesson 04 | Divider, theory, demo | Gradle và artifact |
| 22–25 | Lesson 05 | Divider, debug | Đọc log và ba lỗi thường gặp |
| 26 | Kết | Summary | Checklist áp dụng |

---

## Slide 01 — Title

- **Tiêu đề:** Tự động hóa quy trình CI/CD với GitHub Actions
- **Phụ đề:** Session 07 · QuickBite Food Delivery Microservices
- **Visual:** Một đường đi tối giản: `git push` → GitHub Actions → Runner → JAR artifact.
- **Ý chính:** Một thay đổi nhỏ có thể được kiểm tra và đóng gói theo cùng một quy trình lặp lại.

## Slide 02 — Bản đồ buổi học

- **Tiêu đề:** Từ commit đến artifact qua 5 phần ngắn
- **Nội dung hiển thị:**
  1. CI/CD và Runner
  2. Workflow YAML
  3. Điều phối jobs
  4. Build Spring Boot
  5. Đọc log và xử lý lỗi
- **Visual:** Timeline 5 mốc; đánh dấu phần thực hành ở mốc 2, 3, 4 và 5.

## Slide 03 — Divider Lesson 01

- **Nội dung:** `01` · Tổng quan CI/CD với GitHub Actions và Kiến trúc Runner

## Slide 04 — Build thủ công không tạo được phản hồi nhất quán

- **Nội dung hiển thị:** So sánh hai cột.
  - **Thủ công:** mỗi người tự chạy lệnh; môi trường khác nhau; lỗi phát hiện muộn.
  - **CI:** push kích hoạt cùng workflow; kết quả và log lưu tại một nơi; artifact có thể tải lại.
- **Visual:** Cột trái là ba terminal rời rạc; cột phải là một pipeline thẳng.
- **Callout:** CI không thay thế review code; CI tạo phản hồi kỹ thuật lặp lại được.

## Slide 05 — CI, Delivery và Deployment là ba mức tự động hóa khác nhau

- **Nội dung hiển thị:**
  - **Continuous Integration:** build/test sau thay đổi mã nguồn.
  - **Continuous Delivery:** tạo artifact sẵn sàng phát hành.
  - **Continuous Deployment:** tự đưa artifact đã đạt điều kiện lên môi trường đích.
- **Visual:** Ba tầng tăng dần, đánh dấu phạm vi Session 07 là CI và artifact delivery cơ bản.

## Slide 06 — GitHub điều phối, Runner mới thực thi lệnh

- **Nội dung hiển thị:**
  - Workflow được kích hoạt bởi sự kiện như `push`.
  - GitHub xếp lịch job phù hợp nhãn Runner.
  - Runner chủ động kết nối, nhận job, checkout và chạy step.
  - Log và trạng thái được gửi ngược về GitHub.
- **Visual:** Sơ đồ `Developer → GitHub → queue → Runner → Gradle/JAR`, mũi tên Runner nhận job hướng từ Runner tới GitHub trước.

## Slide 07 — Chọn Runner theo trách nhiệm vận hành

- **Nội dung hiển thị:**
  - **GitHub-hosted:** VM do GitHub quản lý; môi trường mới; phù hợp workflow phổ biến.
  - **Self-hosted:** hạ tầng do đội ngũ quản lý; chủ động tài nguyên, mạng nội bộ và cache; phải tự bảo mật/dọn dẹp.
- **Visual:** Bảng so sánh 2 cột: quyền kiểm soát, bảo trì, truy cập nội bộ.
- **Callout:** QuickBite dùng self-hosted runner cho lab, không đồng nghĩa mọi dự án phải dùng loại này.

## Slide 08 — Demo: Kiểm tra self-hosted runner của QuickBite

- **Mục tiêu:** Quan sát cấu hình runner hiện có và trạng thái online.
- **Thao tác hiển thị:**
  ```bash
  cd ~/quickbite/action-runner
  # export PERSONAL_ACCESS_TOKEN_FOR_RUNNER=...  # không in token
  docker compose up -d
  docker compose ps
  docker compose logs --tail 50 action-runner
  ```
- **Dấu hiệu thành công:** Runner xuất hiện `Idle`/`Online` trong GitHub Organization settings; nhãn có `self-hosted`, `quickbite`, `backend`, `ubuntu`.
- **Visual:** Screenshot placeholder của trang Runner + terminal output gọn.

## Slide 09 — Divider Lesson 02

- **Nội dung:** `02` · Xây dựng cấu trúc Workflow CI/CD cơ bản

## Slide 10 — Workflow chỉ được GitHub nhận diện ở đúng đường dẫn

- **Nội dung hiển thị:**
  ```text
  user-service/
  └── .github/
      └── workflows/
          └── ci.yml
  ```
  - YAML dùng khoảng trắng, không dùng Tab.
  - Workflow là file cấu hình theo version control.
- **Visual:** Cây thư mục với `.github/workflows/ci.yml` tô điểm nhấn.

## Slide 11 — Một workflow có trigger, jobs và steps

- **Nội dung hiển thị:**
  ```yaml
  name: Print Environment Info
  on: [push]
  jobs:
    print_env:
      runs-on: [self-hosted, quickbite]
      steps:
        - uses: actions/checkout@v5
        - run: java -version
  ```
- **Visual:** Annotation bên phải giải thích `name`, `on`, `jobs`, `runs-on`, `steps`.

## Slide 12 — Demo: Workflow đầu tiên kiểm tra môi trường Java 17

- **Nội dung hiển thị:**
  - Tạo mới `.github/workflows/ci.yml` trong repository `user-service`.
  - Trigger: push vào `main`.
  - `actions/checkout@v5` lấy source.
  - `actions/setup-java@v5` cấu hình Temurin 17.
  - In `java -version`, `GITHUB_REF_NAME`, tên project.
- **Visual:** Code block chỉ 12–14 dòng quan trọng; phần env/log được chú thích ngoài code.

## Slide 13 — Một lần chạy thành công phải để lại bằng chứng dễ đọc

- **Nội dung hiển thị:**
  1. Commit workflow và push.
  2. Mở tab Actions, chọn run mới nhất.
  3. Xác nhận job xanh, nhánh đúng, JDK 17 đúng.
- **Visual:** Ba bước nhỏ nối tiếp và mock log 3 dòng.
- **Callout:** Không có run nào xuất hiện: kiểm tra đường dẫn, YAML và trigger trước khi nghi ngờ Runner.

## Slide 14 — Divider Lesson 03

- **Nội dung:** `03` · Cơ chế điều phối và phân tách Jobs trong luồng CI/CD

## Slide 15 — Jobs chạy song song cho đến khi khai báo phụ thuộc

- **Nội dung hiển thị:**
  - Không có `needs`: jobs độc lập có thể chạy song song.
  - Có `needs`: job sau chỉ chạy khi job trước thành công.
  - Một job lỗi khiến job phụ thuộc không chạy.
- **Visual:** Dependency graph: `system-info` và `runner-user` → `report`.

## Slide 16 — Job isolation buộc ta truyền kết quả một cách rõ ràng

- **Nội dung hiển thị:**
  - Mỗi job có lifecycle và workspace riêng.
  - Không giả định file tạo ở job A tự có ở job B.
  - Dùng artifact hoặc output khi cần chuyển dữ liệu.
- **Visual:** Hai workspace tách nhau, một package artifact đi qua giữa.
- **Callout:** Self-hosted runner có thể lưu trạng thái máy, nhưng workflow vẫn phải coi job là độc lập.

## Slide 17 — Demo: Hai jobs song song, một job tổng hợp

- **Nội dung hiển thị:**
  ```yaml
  system_info: { run: uname -a }
  runner_user: { run: whoami }
  report:
    needs: [system_info, runner_user]
    run: echo "Both checks completed"
  ```
- **Visual:** GitHub Actions graph tương ứng; mũi tên chỉ từ hai job đầu tới `report`.
- **Dấu hiệu thành công:** `report` chỉ bắt đầu sau khi hai job đầu xanh.

## Slide 18 — Divider Lesson 04

- **Nội dung:** `04` · Ứng dụng CI/CD: Biên dịch tự động Spring Boot với Gradle

## Slide 19 — Runner phải dùng đúng JDK mà project khai báo

- **Nội dung hiển thị:**
  - `user-service`: Gradle toolchain Java 17.
  - `restaurant-service`, `order-service`, `notification-service`: Java 21.
  - Gradle Wrapper giữ phiên bản Gradle theo source, không phụ thuộc Gradle cài sẵn trên Runner.
- **Visual:** Bảng service–JDK–lệnh build.

## Slide 20 — `bootJar` tạo JAR chạy được, artifact giữ lại đầu ra CI

- **Nội dung hiển thị:**
  ```bash
  chmod +x ./gradlew
  ./gradlew bootJar
  # build/libs/user-service.jar
  ```
  - `bootJar`: đóng gói Spring Boot executable JAR.
  - `upload-artifact`: lưu JAR để tải lại từ run.
  - `build`: dùng khi cần bao gồm test.
- **Visual:** Source → Gradle Wrapper → `user-service.jar` → Artifact.

## Slide 21 — Demo: Pipeline build tối thiểu cho `user-service`

- **Nội dung hiển thị:**
  1. Checkout source.
  2. Setup Temurin 17.
  3. Cấp execute bit cho Wrapper.
  4. Chạy `./gradlew bootJar`.
  5. Upload `build/libs/*.jar` với tên `user-service-jar`.
- **Visual:** Code block workflow rút gọn ở trái; panel phải là vị trí Artifacts trong run thành công.
- **Dấu hiệu thành công:** Có `user-service.jar` trong artifact; không cần build Docker image ở Session 07.

## Slide 22 — Divider Lesson 05

- **Nội dung:** `05` · Chẩn đoán sự cố và quản trị log hệ thống CI/CD

## Slide 23 — Đọc log theo thứ tự job → step → lỗi đầu tiên → exit code

- **Nội dung hiển thị:**
  1. Xác định job đỏ.
  2. Mở step thất bại.
  3. Đọc lỗi đầu tiên có ngữ cảnh.
  4. Phân loại: workflow, quyền, compile, test, dependency.
  5. Sửa nhỏ nhất có thể và chạy lại.
- **Visual:** Lưu đồ dọc 5 bước.

## Slide 24 — Ca lỗi 1: Permission denied là lỗi quyền thực thi, không phải lỗi Java

- **Nội dung hiển thị:**
  ```text
  ./gradlew: Permission denied
  Process completed with exit code 126
  ```
  - Nguyên nhân demo: workflow cố ý chạy `chmod -x ./gradlew`.
  - Khắc phục: `chmod +x ./gradlew` trước `./gradlew bootJar`.
- **Visual:** Log card đỏ và code fix màu xanh.

## Slide 25 — Ca lỗi 2 và 3: Compile lỗi khác với test lỗi

- **Nội dung hiển thị:** So sánh hai cột.
  - **Compilation failed:** `cannot find symbol`, chỉ file/dòng lỗi; sửa source rồi chạy `./gradlew compileJava`.
  - **Test failed:** compile xong nhưng assertion/context test thất bại; xem test name rồi chạy `./gradlew test`.
- **Visual:** Hai log snippets ngắn, mỗi snippet tô đậm dòng kết luận.
- **Callout:** Tạo lỗi trên branch demo, không commit lỗi cố ý vào `main`.

## Slide 26 — Checklist áp dụng sau Session 07

- **Nội dung hiển thị:**
  - Runner online và có đúng labels.
  - Workflow nằm trong `.github/workflows/`.
  - JDK khớp Gradle toolchain của service.
  - Build tạo artifact có thể tải lại.
  - Khi lỗi: đọc job, step, lỗi đầu tiên và exit code.
- **Visual:** Checklist 5 dòng + đường đi `push → run → artifact → debug`.
- **Kết:** “Tự động hóa bắt đầu từ một workflow nhỏ, rõ ràng và kiểm chứng được.”
