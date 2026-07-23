# PROMPT CHO GAMMA: AUTOMATION CI/CD WITH GITHUB ACTIONS (SESSION 7)

## 1. CONTEXT & ROLE
* **Role:** DevOps Architect kiêm Giảng viên cao cấp. Giọng điệu chuyên nghiệp, trực diện, đi thẳng vào bản chất kỹ thuật và thực tiễn vận hành hệ thống.
* **Target Audience:** Kỹ sư phần mềm, học viên đang học cách tự động hóa quy trình tích hợp mã nguồn (CI) cho hệ thống Spring Boot Microservices (dự án QuickBite).
* **Objective:** Giải thích bản chất CI/CD, kiến trúc phân tán giữa GitHub Server và Runner (giao tiếp REST API / Long-polling), Hosted vs Self-hosted runner, cấu trúc thư mục và cú pháp tệp YAML, cơ chế chạy song song (Parallel) mặc định và cách chuyển sang chạy tuần tự (Sequential) bằng từ khóa `needs`, cơ chế cô lập Job (Job Isolation), quy trình đóng gói Spring Boot bằng Gradle Wrapper (`gradlew`), và quy trình 4 bước chẩn đoán lỗi console qua các kịch bản thực tế (Permission denied, Compilation failed, Test failed).

---

## 2. PARTITION INSTRUCTIONS
* **Số lượng slide khuyến nghị:** 15 - 18 slides.
* **Nguyên tắc phân bổ nội dung:**
  * **LESSON 01 (Tổng quan & Runner):** Hạn chế của việc build thủ công, định nghĩa CI/CD, kiến trúc điều phối Server - Runner (Long-polling) và so sánh Hosted vs Self-hosted runner.
  * **LESSON 02 (Workflow YAML):** Thư mục ẩn bắt buộc `.github/workflows/`, cú pháp YAML cơ bản (thụt lề khoảng trắng, cấm dùng Tab), các từ khóa nền tảng (`name`, `on`, `env`, `jobs`, `runs-on`, `steps`, `uses`, `run`), và viết file workflow in thông tin môi trường.
  * **LESSON 03 (Job Coordination & Isolation):** Luồng chạy song song mặc định của các jobs, từ khóa `needs` để thiết lập luồng tuần tự, cơ chế cô lập job (Job Isolation), và viết workflow 3 jobs mô phỏng (2 job song song thu thập thông tin, 1 job in kết quả).
  * **LESSON 04 (Biên dịch Spring Boot với Gradle):** Giới hạn tài nguyên Hosted runner, tối ưu hóa pipeline dưới 1.5 phút bằng cách bỏ qua test (`bootJar`), cấp quyền thực thi `chmod +x ./gradlew`, fallback cấu hình database trong `application.yml`, và lưu trữ thành phẩm bằng `actions/upload-artifact@v4`.
  * **LESSON 05 (Chẩn đoán sự cố & Log):** Vị trí đọc log console, quy trình 4 bước rà soát lỗi hệ thống, chẩn đoán 3 kịch bản lỗi thực tế (Permission denied - exit code 126, compile FAILED - exit code 1, test FAILED - exit code 1).
  * **Độ thoáng đãng:** Trình bày ngắn gọn, súc tích, đi thẳng vào bản chất kỹ thuật. Loại bỏ hoàn toàn các câu từ suồng sã.

---

## 3. HIERARCHICAL OUTLINE (DÀN Ý CHI TIẾT)

### LESSON 01: Tổng quan GitHub Actions và Kiến trúc Runner

#### Slide 1: Sự bế tắc của việc biên dịch và kiểm thử thủ công
* **Vấn đề thực tế trong dự án QuickBite:**
  * Mỗi khi cập nhật mã nguồn (ví dụ: sửa đổi logic trong `user-service`), lập trình viên phải chạy unit test cục bộ, biên dịch thủ công ra file JAR (`./gradlew bootJar`) trước khi push lên Git.
  * **Thách thức:** Lập trình viên thường bỏ qua các bước kiểm tra, trực tiếp push code lỗi lên Git khiến code gãy biên dịch vẫn được merge vào nhánh chính.
  * Môi trường không nhất quán: "Chạy tốt trên máy tôi" nhưng sập trên máy người khác do lệch phiên bản JDK (17 vs 21).
* **Giải pháp:** Thiết lập hệ thống kiểm thử và biên dịch tự động, tập trung hóa trên một máy chủ độc lập (CI/CD).

#### Slide 2: Khái niệm cốt lõi CI/CD
* **Continuous Integration (CI - Tích hợp liên tục):**
  * Tự động hóa khâu kéo code mới, kiểm tra chất lượng cú pháp (Linter), biên dịch ứng dụng và chạy bộ Unit/Integration tests.
  * Mục tiêu: Đảm bảo code mới tích hợp vào kho lưu trữ luôn ở trạng thái biên dịch thành công.
* **Continuous Delivery / Deployment (CD - Chuyển giao/Triển khai liên tục):**
  * Delivery: Tự động đóng gói (JAR, Docker image) và đẩy lên registry. Deploy lên môi trường Staging/Production cần xác nhận thủ công.
  * Deployment: Tự động hóa hoàn toàn khâu triển khai lên Production không qua can thiệp vật lý.

#### Slide 3: Kiến trúc điều phối Server - Runner
* **Sơ đồ luồng giao tiếp:**
  ```text
  [ GitHub Server (Web UI/Repo) ] <─── ( REST API via Long-Polling ) ───> [ Actions Runner (Agent VM/OS) ]
                                                                                   │
                                                                           (Thực thi Jobs)
                                                                                   ▼
                                                                           [ Môi trường Host ]
  ```
* **Thành phần điều phối:**
  * **GitHub Server:** Nơi lưu trữ mã nguồn, phát hiện Git events (push, pull request) và điều phối công việc.
  * **GitHub Actions Runner (Agent):** Tiến trình độc lập chạy trên máy chủ. Nó chủ động gửi request kéo job (long-polling) để thực thi.
  * *Ưu điểm:* Không cần mở bất kỳ cổng inbound (đầu vào) nào trên máy chủ Runner, chống nguy cơ tấn công mạng.

#### Slide 4: Phân loại Runner: Hosted vs Self-hosted
* **So sánh đặc thù tài nguyên:**
  * **GitHub-hosted runner:**
    * Do GitHub quản lý, khởi tạo máy ảo mới tinh sạch sẽ cho mỗi job (Linux, Windows hoặc macOS).
    * Bị giới hạn cấu hình phần cứng (thường 2 vCPU, 7GB RAM) và giới hạn thời gian chạy miễn phí.
  * **Self-hosted runner:**
    * Máy ảo/VPS hoặc máy vật lý do người dùng tự cài đặt ứng dụng Runner và đăng ký vào GitHub repository.
    * Quyền kiểm soát hoàn toàn về phần cứng, giữ được cache cục bộ giúp build nhanh hơn, không bị tính phí thời gian chạy.

---

### LESSON 02: Xây dựng cấu trúc Workflow CI/CD cơ bản

#### Slide 5: Quy định cấu trúc thư mục và cú pháp YAML
* **Thư mục workflows bắt buộc:**
  * GitHub Actions yêu cầu tất cả các tệp tin cấu hình Workflow (ví dụ: `ci.yml`) phải được đặt trong thư mục ẩn có đường dẫn chuẩn xác là `.github/workflows/` tính từ thư mục gốc của repository.
* **Quy tắc cú pháp YAML cơ bản:**
  * Định nghĩa dữ liệu theo cặp `key: value` (bắt buộc có khoảng trắng sau dấu `:`).
  * Phân cấp cấu trúc bằng **thụt lề bằng khoảng trắng (spaces)**.
  * *Cảnh báo:* Nghiêm cấm sử dụng phím Tab để thụt lề (gây lỗi cú pháp YAML syntax error). Sử dụng dấu gạch ngang `-` kèm khoảng trắng để biểu diễn mảng (danh sách).

#### Slide 6: Phân cấp cấu trúc Workflow
* **Cây phân cấp của luồng thực thi:**
  * **Workflow:** Toàn bộ kịch bản tích hợp tự động hóa (lưu tại `.github/workflows/`).
  * **Jobs:** Tập hợp các công việc. Các job mặc định chạy song song và độc lập trên các Runner khác nhau.
  * **Steps:** Các bước chạy tuần tự trong 1 Job. Tất cả step của 1 job chạy trên cùng 1 Runner và chia sẻ chung thư mục làm việc.
  * **Actions:** Các khối lệnh viết sẵn được đóng gói lại (gọi qua từ khóa `uses`), ví dụ clone mã nguồn, cài JDK.
  * **Run:** Thực thi các lệnh shell command thông thường (ví dụ: `run: java -version`).

#### Slide 7: Các từ khóa cú pháp nền tảng
* **Từ khóa điều phối:**
  * `name`: Tên của workflow hiển thị trên giao diện GitHub Actions.
  * `on`: Định nghĩa các sự kiện kích hoạt (ví dụ: `push` hoặc `pull_request` lọc theo nhánh).
  * `env`: Khai báo biến môi trường dùng chung trong workflow.
* **Định nghĩa Jobs và Steps:**
  * `jobs`: Chứa một hoặc nhiều công việc chạy song song.
  * `runs-on`: Hệ điều hành hoặc nhãn (label) của Runner thực thi (ví dụ: `[self-hosted, quickbite]`).
  * `steps`: Các bước chạy tuần tự trong job. Sử dụng `uses` để gọi action đóng gói sẵn (như `actions/checkout@v5` để clone code, `actions/setup-java@v5` để cài JDK), hoặc `run` để thực thi shell script.

#### Slide 8: Thực hành viết file Workflow in thông tin môi trường
* Mã nguồn tệp `.github/workflows/ci.yml` của dự án `user-service`:
```yaml
name: Print Environment Info

on:
  push:
    branches:
      - main

env:
  PROJECT_NAME: "QuickBite-User-Service"

jobs:
  print_env_job:
    runs-on: [self-hosted, quickbite]
    steps:
      - name: Lấy mã nguồn về máy Runner
        uses: actions/checkout@v5
        
      - name: Thiết lập môi trường Java 17
        uses: actions/setup-java@v5
        with:
          java-version: '17'
          distribution: 'temurin'
          
      - name: In thông tin môi trường
        run: |
          echo "Chuẩn bị in thông tin môi trường cho dự án ${PROJECT_NAME}..."
          java -version
          echo "Nhánh Git đang chạy workflow là ${GITHUB_REF_NAME}"
```
* **Biến môi trường mặc định:** `GITHUB_REF_NAME` là biến do hệ thống tự động tiêm vào môi trường của Runner, không cần khai báo trong khối `env`.

---

### LESSON 03: Cơ chế điều phối và phân tách Jobs trong luồng CI/CD

#### Slide 9: Chạy song song mặc định và Chuyển đổi tuần tự bằng `needs`
* **Hành vi song song (Parallel):**
  * Các job ngang hàng được khai báo trong workflow sẽ tự động chạy song song cùng lúc nhằm tăng tốc độ xử lý của chu trình CI/CD.
* **Luồng tuần tự với `needs`:**
  * Sử dụng từ khóa `needs` để thiết lập sự phụ thuộc giữa các job.
  * Ví dụ: Job B khai báo `needs: [Job A]`. Job B sẽ bị khóa ở trạng thái pending cho đến khi Job A hoàn thành thành công.
  * *Chốt chặn bảo mật:* Nếu Job A bị lỗi (Failed), các job phụ thuộc phía sau (Job B) sẽ tự động bị bỏ qua (Skipped) để bảo vệ hệ thống khỏi việc tiếp tục sử dụng code lỗi.

#### Slide 10: Cơ chế cô lập công việc (Job Isolation)
* **Nguyên lý cô lập:**
  * Mỗi một job chạy trong một không gian riêng biệt, sạch sẽ.
  * File và dữ liệu sinh ra tại thư mục cục bộ của Job A sẽ **không** được tự động chia sẻ hay dùng chung bởi Job B.
  * Để Job B có mã nguồn, bạn bắt buộc phải khai báo lại lệnh `actions/checkout@v5` ở bước thực thi đầu tiên của Job B (lệnh checkout ở Job A không có tác dụng với Job B).
  * Nếu muốn chia sẻ tệp tin giữa các Job, bắt buộc phải dùng các hành động upload/download artifacts trung gian.

#### Slide 11: Thực hành cấu hình luồng 3 Jobs phụ thuộc
* Cấu hình workflow gồm `job_info_1` và `job_info_2` chạy song song, sau đó `job_print` phụ thuộc chạy cuối cùng:
```yaml
jobs:
  job_info_1:
    runs-on: [self-hosted, quickbite]
    steps:
      - name: Thu thập thông tin OS
        run: uname -a

  job_info_2:
    runs-on: [self-hosted, quickbite]
    steps:
      - name: Thu thập thông tin User
        run: whoami

  job_print:
    needs: [job_info_1, job_info_2]
    runs-on: [self-hosted, quickbite]
    steps:
      - name: In kết quả
        run: echo "Chỉ chạy sau khi job_info_1 và job_info_2 thành công."
```

---

### LESSON 04: Ứng dụng CI/CD: Biên dịch tự động Spring Boot với Gradle

#### Slide 12: Chuyển dịch quy trình đóng gói JAR từ Local lên CI
* **Quy trình thủ công trước đây (Local Build):**
  * Nhà phát triển phải tự cài đặt JDK phù hợp trên máy cá nhân.
  * Tự chạy lệnh `./gradlew bootJar` ở máy local để biên dịch ra file JAR rồi mới upload thủ công lên máy chủ.
  * Nhược điểm: Phụ thuộc vào môi trường máy cá nhân, dễ xảy ra lỗi lệch phiên bản JDK.
* **Quy trình tự động hóa mới (CI Build):**
  * Đưa toàn bộ quy trình biên dịch và đóng gói JAR vào chạy tự động trên GitHub Actions Server.
  * Runner tự động chuẩn bị môi trường sạch, cài đặt JDK chuẩn thông qua Action và chạy đóng gói JAR tự động mỗi khi push code.
  * Để tối ưu tài nguyên trên môi trường Hosted Runner, kịch bản CI sẽ chạy tác vụ `bootJar` (chỉ tập trung đóng gói file JAR thành phẩm) thay vì chạy toàn bộ vòng đời `build` kèm test.

#### Slide 13: Thực hành Workflow biên dịch và lưu trữ Artifact
* Kịch bản đóng gói ứng dụng `user-service` và tải thành phẩm lên không gian lưu trữ của GitHub:
```yaml
name: Build Spring Boot App

on:
  push:
    branches:
      - main

jobs:
  build_executable_jar:
    runs-on: [self-hosted, quickbite]
    steps:
      - name: Checkout mã nguồn
        uses: actions/checkout@v5
        
      - name: Cài đặt môi trường JDK 17
        uses: actions/setup-java@v5
        with:
          java-version: '17'
          distribution: 'temurin'
          
      - name: Cấp quyền thực thi cho Gradle Wrapper
        run: chmod +x ./gradlew
        
      - name: Biên dịch và đóng gói tệp JAR
        run: ./gradlew bootJar
        
      - name: Lưu trữ sản phẩm (Artifact)
        uses: actions/upload-artifact@v4
        with:
          name: user-service-jar
          path: build/libs/*.jar
          retention-days: 3
```
* *Đồng bộ JDK:* Nếu là `restaurant-service`, thay đổi tham số `java-version` thành `'21'`.

---

### LESSON 05: Chẩn đoán sự cố và quản trị log hệ thống CI/CD

#### Slide 14: Quy trình 4 bước chẩn đoán lỗi Console
* **Các bước xử lý sự cố chuẩn:**
  1. **Xác định Job lỗi:** Tìm job có biểu tượng dấu X đỏ trong tab Actions.
  2. **Truy cập Console Log:** Bấm chọn job bị lỗi, mở rộng step bị sập để đọc log.
  3. **Phân tích dòng log lỗi:** Cuộn xuống phần cuối log để kiểm tra mã thoát (Exit code) và lần ngược lên trên để đọc thông tin lỗi chi tiết (Error Trace).
  4. **Sửa lỗi và kiểm chứng cục bộ:** Sửa mã nguồn/cấu hình, chạy kiểm tra cục bộ trước khi push commit mới lên Git.

#### Slide 15: Kịch bản lỗi 1 - Lỗi phân quyền Wrapper (Permission Denied)
* **Dấu hiệu nhận biết trong log console:**
  ```text
  Run ./gradlew bootJar
  /home/runner/work/_temp/...: line 2: ./gradlew: Permission denied
  Error: Process completed with exit code 126.
  ```
* **Bản chất lỗi:** Tệp `./gradlew` bị mất thuộc tính quyền thực thi (execute bit) do được viết trên Windows và push lên môi trường máy chủ chạy Linux của Runner.
* **Cách khắc phục:** Thêm bước chạy `run: chmod +x ./gradlew` trước lệnh biên dịch trong file YAML.

#### Slide 16: Kịch bản lỗi 2 - Lỗi biên dịch mã nguồn (Compilation Failed)
* **Dấu hiệu nhận biết trong log console:**
  ```text
  Run ./gradlew bootJar
  > Task :compileJava FAILED
  /home/runner/.../UserService.java:24: error: cannot find symbol
          private UserReposotory userRepository;
                  ^
  Error: Process completed with exit code 1.
  ```
* **Bản chất lỗi:** Trình biên dịch Java Compiler từ chối tạo bytecode do lỗi cú pháp code Java hoặc import sai thư viện ở dòng cụ thể (dòng 24).
* **Cách khắc phục:** Định vị file Java bị lỗi theo mô tả đường dẫn trong log, sửa lại cú pháp code Java chuẩn xác.

#### Slide 17: Kịch bản lỗi 3 - Lỗi kiểm thử thất bại (Test Failed)
* **Dấu hiệu nhận biết trong log console (khi chạy lệnh `./gradlew build`):**
  ```text
  UserServiceApplicationTests > testCreateUser() FAILED
      org.opentest4j.AssertionFailedError at UserServiceApplicationTests.java:17
  2 tests completed, 1 failed
  > Task :test FAILED
  Error: Process completed with exit code 1.
  ```
* **Bản chất lỗi:** Việc biên dịch lớp đã hoàn tất nhưng giá trị khẳng định logic (Assertion) trong file test bị sai lệch, cơ chế Gradle kích hoạt "Fail-fast" để hủy bỏ việc tạo tệp JAR bị lỗi.
* **Cách khắc phục:** Rà soát lại logic nghiệp vụ hoặc sửa lại assertion trong file test cho đúng thiết kế.

#### Slide 18: Tổng kết bài học
* **Tự động hóa CI/CD:** Thay thế quy trình đóng gói và kiểm tra thủ công tại máy local bằng quy trình tự động hóa tập trung trên máy chủ CI.
* **Kiến trúc Server - Runner:** Giao tiếp qua giao thức Long-polling an toàn, Runner chủ động kéo job về chạy và trả log mà không cần mở cổng inbound.
* **Cấu trúc kịch bản YAML:** Lưu trữ bắt buộc tại thư mục `.github/workflows/`, chú ý phân cấp thụt lề bằng khoảng trắng và cấm sử dụng phím Tab.
* **Điều phối Jobs:** Mặc định chạy song song, sử dụng từ khóa `needs` để ràng buộc tuần tự và tận dụng cơ chế cô lập Job (Job Isolation).
* **Đóng gói Spring Boot:** Sử dụng Gradle Wrapper (`gradlew`) thống nhất phiên bản và lưu trữ tệp JAR thành phẩm qua Action Upload-Artifact.
* **Chẩn đoán log console:** Tìm đúng step lỗi có dấu X đỏ, kiểm tra Exit Code (ví dụ: 126 là thiếu quyền thực thi) và Error Trace để sửa lỗi.
