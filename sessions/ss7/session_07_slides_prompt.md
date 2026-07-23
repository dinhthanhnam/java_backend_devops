# HƯỚNG DẪN CẤU TRÚC VÀ NỘI DUNG SLIDE (HSSF) - SESSION 07
## CHỦ ĐỀ: TỰ ĐỘNG HÓA QUY TRÌNH CI/CD VỚI GITHUB ACTIONS

Tài liệu này cung cấp thiết kế chi tiết từng slide theo chuẩn **HSSF (HTML Slide System Framework)** của Rikkei Education. Mỗi slide được định nghĩa rõ ràng về nhãn (`data-hssf-label`), cấu trúc layout component, nội dung hiển thị (tiếng Việt), các đoạn mã (code) và nội dung thuyết trình (Speaker Notes).

---

## I. BẢN ĐỒ SLIDE (SLIDE MAP)

| # | `data-hssf-label` | Chủ đề Slide (Focus) | HSSF Components chính |
|---|-------------------|----------------------|-----------------------|
| 1 | Title | Tiêu đề chính | `hssf-slide--title`, `hssf-title-block` |
| 2 | Agenda | Lộ trình buổi học | `hssf-header`, `hssf-agenda` |
| 3 | Sec-01 | Phân đoạn 01: Giới thiệu GitHub Actions | `hssf-slide--section`, `hssf-section-block` |
| 4 | Pain-Manual | Hạn chế của việc build & kiểm thử thủ công | `hssf-compare`, `hssf-callout--danger` |
| 5 | Concept-CICD | CI/CD là gì? | `hssf-grid`, `hssf-card` |
| 6 | Flow-Architecture | Kiến trúc điều phối Server - Runner | `hssf-columns--2`, `hssf-flow` |
| 7 | Runner-Types | Phân loại Runner: Hosted vs Self-hosted | `hssf-compare`, `hssf-callout--info` |
| 8 | Sec-02 | Phân đoạn 02: Cấu trúc Workflow | `hssf-slide--section`, `hssf-section-block` |
| 9 | Workflow-Syntax | Cấu trúc file yaml của Workflow | `hssf-columns`, `hssf-code`, `hssf-defs` |
| 10 | Workflow-Demo | File YAML Workflow cơ bản đầu tiên | `hssf-code`, `hssf-callout--success` |
| 11 | Sec-03 | Phân đoạn 03: Điều phối và Phân tách Jobs | `hssf-slide--section`, `hssf-section-block` |
| 12 | Job-Dependency | Cơ chế phụ thuộc giữa các Jobs (needs) | `hssf-columns`, `hssf-flow` |
| 13 | Matrix-Strategy | Cấu hình Matrix Strategy | `hssf-columns`, `hssf-code`, `hssf-list` |
| 14 | Sec-04 | Phân đoạn 04: Biên dịch tự động Spring Boot | `hssf-slide--section`, `hssf-section-block` |
| 15 | Gradle-Build | Workflow biên dịch Spring Boot với Gradle | `hssf-code`, `hssf-callout--warning` |
| 16 | Cache-Gradle | Cơ chế Cache Dependency trong CI/CD | `hssf-compare`, `hssf-callout--tip` |
| 17 | Sec-05 | Phân đoạn 05: Debug và Quản trị Log | `hssf-slide--section`, `hssf-section-block` |
| 18 | Debug-Log | Chẩn đoán sự cố & Debug log trong CI/CD | `hssf-steps`, `hssf-list` |
| 19 | Summary | Tổng kết bài học | `hssf-header`, `hssf-list` |
| 20 | End | Kết thúc slide | `hssf-brand-end` |

---

## II. CHI TIẾT CẤU TRÚC VÀ NỘI DUNG TỪNG SLIDE

### SLIDE 1: Title
* **HSSF Classes:** `hssf-slide hssf-slide--title`
* **Layout / Components:**
  * `hssf-title-block` có `hssf-accent--bar-left`
  * Eyebrow: `SESSION 07 · DEVOPS IN ACTION`
  * Title: `Tự động hóa quy trình CI/CD với GitHub Actions`
  * Meta: `Rikkei Academy · Bộ môn DevOps`
* **Speaker Notes:** Chào các bạn học viên. Hôm nay chúng ta bước sang Session 07 để nghiên cứu một chủ đề cực kỳ quan trọng trong DevOps: tự động hóa quy trình kiểm thử và biên dịch (CI) sử dụng công cụ GitHub Actions.

---

### SLIDE 2: Agenda
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Nội dung bài học", Subtitle: "5 phần trọng tâm về CI/CD")
  * `hssf-agenda`:
    * Lộ trình 1: Tổng quan về CI/CD và kiến trúc Runner
    * Lộ trình 2: Xây dựng cấu trúc Workflow CI/CD cơ bản (YAML)
    * Lộ trình 3: Cơ chế điều phối và phân tách Jobs
    * Lộ trình 4: Thực hành: Tự động biên dịch Spring Boot bằng Gradle
    * Lộ trình 5: Chẩn đoán sự cố và quản trị log hệ thống CI/CD
* **Speaker Notes:** Lộ trình bài học gồm 5 phần. Chúng ta đi từ lý thuyết tổng quan, cấu pháp file cấu hình, cách chia nhỏ job, biên dịch thực tế app Spring Boot, và cuối cùng là kỹ năng debug pipeline.

---

### SLIDE 3: Sec-01 (Divider)
* **HSSF Classes:** `hssf-slide hssf-slide--section`
* **Layout / Components:**
  * `hssf-section-block`
  * Big Number: `01`
  * Title: `Tổng quan GitHub Actions & Kiến trúc Runner`
* **Speaker Notes:** Phần 1: Tìm hiểu khái niệm CI/CD và cách GitHub Actions phân tách nhiệm vụ giữa máy chủ điều phối và máy chủ thực thi (Runner).

---

### SLIDE 4: Pain-Manual (Hạn chế build thủ công)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Vấn đề của quy trình thủ công", Subtitle: "Tại sao không nên tự build & test ở local?")
  * `hssf-compare` chia 2 cột:
    * Cột Trái (Quy trình thủ công & rủi ro):
      * Lập trình viên quên chạy test hoặc build thử trước khi push.
      * Code lỗi biên dịch vẫn được merge vào nhánh chính.
      * "Chạy tốt trên máy tôi" — Xung đột phiên bản JDK giữa các máy.
    * Cột Phải (Hệ thống CI/CD tự động):
      * Tự động chạy test & build tập trung mỗi khi push code mới.
      * Chặn merge code nếu build lỗi.
      * Đảm bảo môi trường biên dịch đồng nhất (Docker/VM chuẩn).
  * `hssf-callout--danger` ở chân slide: "Hệ thống CI/CD giúp loại bỏ hoàn toàn yếu tố chủ quan của con người trong khâu kiểm soát chất lượng code."
* **Speaker Notes:** Rủi ro lớn nhất của build thủ công là dev quên chạy test hoặc môi trường của dev khác với production. CI/CD giải quyết việc này bằng cách tự động chạy test tập trung trên server sạch.

---

### SLIDE 5: Concept-CICD (Khái niệm CI/CD)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Khái niệm CI/CD", Subtitle: "Trái tim của quy trình vận hành DevOps")
  * `hssf-grid--2` chứa 4 card `hssf-card--outline`:
    * Card 1: Icon `🔄` | Title: Continuous Integration (CI) | Body: Tự động hợp nhất code mới, chạy lint, biên dịch (compile) và chạy unit tests.
    * Card 2: Icon `📦` | Title: Continuous Delivery (CD) | Body: Đóng gói sản phẩm (JAR, Docker image) và sẵn sàng để deploy lên môi trường staging/production.
    * Card 3: Icon `🚀` | Title: Continuous Deployment | Body: Tự động triển khai code mới lên thẳng production mà không cần phê duyệt thủ công.
    * Card 4: Icon `🛠️` | Title: Automation Tool | Body: GitHub Actions tích hợp sâu vào GitHub repository, giúp kích hoạt pipeline từ các event của Git.
* **Speaker Notes:** CI là tích hợp liên tục (tập trung vào build/test). CD có hai nhánh: Continuous Delivery (sẵn sàng deploy) và Continuous Deployment (tự động triển khai lên Production).

---

### SLIDE 6: Flow-Architecture (Kiến trúc Runner)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Kiến trúc GitHub Actions Runner")
  * `hssf-columns hssf-columns--2 hssf-columns--loose hssf-fill hssf-columns--center`:
    * Cột 1 (Luồng dọc): `hssf-flow hssf-flow--col`
      * Node 1 (Primary): `GitHub Server (Web/Git)` (sub: Nhận diện git push event)
      * Edge (labeled: Long-polling / REST) →
      * Node 2 (Soft): `GitHub Actions Runner` (sub: Máy chủ thực thi công việc)
      * Edge (labeled: Execute Job) →
      * Node 3 (Outline): `Môi trường Host OS` (sub: Biên dịch ứng dụng)
    * Cột 2: `hssf-stack`
      * Giải thích cơ chế giao tiếp:
        * **1. Không mở cổng:** Runner chủ động gửi request kéo công việc về (Long-polling), giúp máy chủ build không cần mở cổng SSH ra Internet.
        * **2. Cô lập:** Runner tải mã nguồn về môi trường riêng để biên dịch rồi gửi log kết quả về cho GitHub Server hiển thị.
* **Speaker Notes:** Kiến trúc cực kỳ thông minh. Runner liên tục gửi request đến GitHub qua giao thức polling để nhận job. Nhờ vậy, máy chủ Runner nằm trong mạng nội bộ vẫn hoạt động tốt mà không cần mở cổng inbound.

---

### SLIDE 7: Runner-Types (Phân loại Runner)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Hosted Runner vs Self-hosted Runner")
  * `hssf-compare`:
    * Cột Trái (GitHub-hosted Runner):
      * Do GitHub quản lý hoàn toàn dưới dạng máy ảo (VM).
      * Máy ảo luôn sạch sẽ (khởi tạo mới tinh cho mỗi job).
      * Giới hạn cấu hình phần cứng (thường 2 vCPU, 7GB RAM).
      * Bị tính phí thời gian chạy (sau khi hết quota miễn phí).
    * Cột Phải (Self-hosted Runner):
      * Người dùng tự thiết lập trên máy ảo (VM)/VPS hoặc máy vật lý riêng.
      * Giữ được bộ nhớ đệm (cache) giữa các lần build giúp build nhanh hơn.
      * Tùy biến cấu hình không giới hạn và hoàn toàn miễn phí thời gian chạy.
  * `hssf-callout--info`: "Đối với các ứng dụng Java Spring Boot đòi hỏi nhiều RAM khi biên dịch, sử dụng Self-hosted runner là giải pháp tối ưu chi phí."
* **Speaker Notes:** Hosted runner sạch nhưng cấu hình yếu và giới hạn thời gian chạy. Self-hosted runner do mình tự quản lý, cấu hình mạnh hơn, giữ được cache giúp biên dịch Java nhanh chóng.

---

### SLIDE 8: Sec-02 (Divider)
* **HSSF Classes:** `hssf-slide hssf-slide--section`
* **Layout / Components:**
  * `hssf-section-block`
  * Big Number: `02`
  * Title: `Xây dựng cấu trúc Workflow CI/CD cơ bản`
* **Speaker Notes:** Phần 2: Tìm hiểu cấu trúc và cú pháp của tệp cấu hình YAML (`.github/workflows/*.yml`) để định nghĩa pipeline.

---

### SLIDE 9: Workflow-Syntax (Cú pháp YAML)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Cấu trúc Workflow YAML")
  * `hssf-columns hssf-columns--1-2 hssf-columns--loose hssf-fill hssf-columns--center`:
    * Cột 1: `hssf-defs`
      * `name`: Tên của workflow
      * `on`: Sự kiện kích hoạt (push, pull_request)
      * `jobs`: Tập hợp các công việc chạy song song
      * `runs-on`: Loại Runner thực thi (ubuntu-latest, self-hosted)
      * `steps`: Các bước chạy tuần tự trong job
      * `uses`: Gọi các Actions có sẵn từ chợ (Marketplace)
    * Cột 2: `hssf-code` (Cú pháp phân cấp của file YAML)
      ```yaml
      name: Basic CI
      on: [push]
      jobs:
        build-job:
          runs-on: ubuntu-latest
          steps:
            - name: Checkout Code
              uses: actions/checkout@v3
            - name: Run Script
              run: echo "Hello World"
      ```
* **Speaker Notes:** File workflow được lưu dưới dạng tệp YAML. Chú ý cấu trúc phân cấp: 1 Workflow chứa nhiều Jobs, 1 Job chứa nhiều Steps chạy tuần tự trên 1 Runner.

---

### SLIDE 10: Workflow-Demo (File Workflow đầu tiên)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Triển khai file Workflow hoàn chỉnh", Subtitle: "Lưu tại thư mục .github/workflows/ci.yml")
  * `hssf-code` (ci.yml / yaml):
    ```yaml
    name: QuickBite CI Pipeline

    on:
      push:
        branches: [ main, develop ]
      pull_request:
        branches: [ main ]

    jobs:
      test-code:
        runs-on: ubuntu-latest
        steps:
          - name: Clone source code from Git
            uses: actions/checkout@v3

          - name: Execute Shell Script
            run: |
              echo "Starting test suite..."
              echo "All tests passed successfully."
    ```
  * `hssf-callout--success`: "Chỉ cần đẩy tệp này lên nhánh main, GitHub Actions sẽ tự động phát hiện và kích hoạt pipeline ngay lập tức."
* **Speaker Notes:** Đây là file workflow đơn giản. Event được cấu hình kích hoạt khi có push vào main/develop hoặc pull request vào main. Job chạy trên máy ubuntu-latest.

---

### SLIDE 11: Sec-03 (Divider)
* **HSSF Classes:** `hssf-slide hssf-slide--section`
* **Layout / Components:**
  * `hssf-section-block`
  * Big Number: `03`
  * Title: `Cơ chế điều phối và phân tách Jobs`
* **Speaker Notes:** Phần 3: Cách tối ưu pipeline bằng cách chia nhỏ thành các Jobs chạy song song hoặc phụ thuộc tuần tự.

---

### SLIDE 12: Job-Dependency (Sự phụ thuộc Jobs)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Cơ chế Jobs song song và phụ thuộc")
  * `hssf-columns hssf-columns--2 hssf-columns--loose hssf-fill hssf-columns--center`:
    * Cột 1 (Luồng dọc): `hssf-flow hssf-flow--col`
      * Node 1 (Soft): `Job: Linter` (sub: Kiểm tra cú pháp)
      * Node 2 (Soft): `Job: Unit Test` (sub: Chạy bộ kiểm thử)
      * Edge (labeled: needs) →
      * Node 3 (Primary): `Job: Build & Packaging` (sub: Chỉ chạy nếu Linter & Test đạt)
    * Cột 2: `hssf-stack`
      * Cấu hình từ khóa `needs` trong YAML:
        * Mặc định: Các jobs chạy song song độc lập.
        * Từ khóa `needs`: Tạo thứ tự phụ thuộc.
      * `hssf-callout--info`: "Không chạy build nếu linter hoặc test bị lỗi để tiết kiệm tài nguyên hạ tầng."
* **Speaker Notes:** Mặc định các job chạy song song. Bằng cách sử dụng từ khóa `needs`, ta bắt buộc Job Build chỉ chạy khi Job Linter và Job Test chạy thành công.

---

### SLIDE 13: Matrix-Strategy (Chiến lược Matrix)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Chiến lược Matrix", Subtitle: "Chạy thử nghiệm trên nhiều phiên bản môi trường đồng thời")
  * `hssf-columns hssf-columns--1-1 hssf-columns--loose hssf-fill`:
    * Cột 1: `hssf-code` (Cú pháp matrix / yaml)
      ```yaml
      jobs:
        test:
          runs-on: ubuntu-latest
          strategy:
            matrix:
              java-version: [17, 21]
              os: [ubuntu-latest, windows-latest]
          steps:
            - uses: actions/setup-java@v3
              with:
                java-version: ${{ matrix.java-version }}
      ```
    * Cột 2: `hssf-stack`
      * Ứng dụng thực tế:
        * Chạy đồng thời 4 jobs kiểm thử:
          * Java 17 chạy trên Ubuntu
          * Java 17 chạy trên Windows
          * Java 21 chạy trên Ubuntu
          * Java 21 chạy trên Windows
      * `hssf-list`:
        * Tiết kiệm công sức viết code cấu hình lặp lại.
        * Phát hiện sớm lỗi không tương thích phiên bản.
* **Speaker Notes:** Matrix Strategy giúp chạy thử nghiệm trên nhiều tổ hợp cấu hình (hệ điều hành, phiên bản JDK) cùng lúc mà không cần viết lặp code workflow.

---

### SLIDE 14: Sec-04 (Divider)
* **HSSF Classes:** `hssf-slide hssf-slide--section`
* **Layout / Components:**
  * `hssf-section-block`
  * Big Number: `04`
  * Title: `Biên dịch tự động Spring Boot với Gradle`
* **Speaker Notes:** Phần 4: Thực hành xây dựng pipeline hoàn chỉnh biên dịch ứng dụng Spring Boot thành tệp JAR bằng công cụ Gradle.

---

### SLIDE 15: Gradle-Build (Workflow biên dịch Gradle)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Cấu hình build Spring Boot với Gradle")
  * `hssf-code` (gradle_ci.yml / yaml):
    ```yaml
    jobs:
      build-spring-app:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v3

          - name: Setup Java JDK 17
            uses: actions/setup-java@v3
            with:
              java-version: '17'
              distribution: 'temurin'

          - name: Grant execute permission for gradlew
            run: chmod +x gradlew

          - name: Compile and Build JAR
            run: ./gradlew bootJar
    ```
  * `hssf-callout--warning`: "Cần cấp quyền thực thi `chmod +x gradlew` trước khi chạy lệnh build, nếu không pipeline sẽ sập do lỗi Permission denied."
* **Speaker Notes:** Các bước cơ bản: checkout code, cài đặt JDK 17 (dùng bản phân phối Temurin), cấp quyền thực thi cho file wrapper gradlew, cuối cùng chạy lệnh `./gradlew bootJar` để build.

---

### SLIDE 16: Cache-Gradle (Tối ưu hóa Cache)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Tối ưu hóa thời gian build với Cache", Subtitle: "Tránh tải lại hàng trăm MB thư viện Gradle ở mỗi lần chạy")
  * `hssf-compare`:
    * Cột Trái (Không sử dụng Cache):
      * Mỗi lần build, runner phải tải mới toàn bộ Spring dependencies từ Internet.
      * Thời gian build kéo dài (3 - 5 phút).
      * Tiêu tốn băng thông mạng vô ích.
    * Cột Phải (Có sử dụng Cache):
      * Tái sử dụng các thư viện đã tải từ các lần build trước.
      * Chỉ tải thêm các thư viện mới phát sinh.
      * Thời gian build giảm sâu (dưới 1 phút).
  * `hssf-callout--tip` ở chân slide: "Sử dụng tính năng cache tích hợp sẵn của Action `setup-java` bằng cách cấu hình: `cache: 'gradle'`."
* **Speaker Notes:** Build Java thường tốn nhiều thời gian tải thư viện dependency từ maven central. Hãy bật cache để giảm thời gian build từ vài phút xuống dưới 1 phút.

---

### SLIDE 17: Sec-05 (Divider)
* **HSSF Classes:** `hssf-slide hssf-slide--section`
* **Layout / Components:**
  * `hssf-section-block`
  * Big Number: `05`
  * Title: `Chẩn đoán sự cố & Quản trị Log hệ thống CI/CD`
* **Speaker Notes:** Phần cuối: Cách đọc log lỗi của GitLab/GitHub Actions để phân tích nguyên nhân và sửa lỗi sập pipeline.

---

### SLIDE 18: Debug-Log (Sửa lỗi Pipeline)
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Quy trình chẩn đoán lỗi trong CI/CD")
  * `hssf-steps`:
    * Step 1: Xác định Job bị lỗi (có biểu tượng dấu X đỏ).
    * Step 2: Tìm Step bị sập để đọc log chi tiết ở màn hình Console.
    * Step 3: Phân loại lỗi phổ biến (Lỗi cú pháp YAML, Sai JDK, Thiếu file cấu hình DB).
    * Step 4: Bật tùy chọn Debug log (`ACTIONS_RUNNER_DEBUG: true`) nếu cần xem chi tiết hành vi của Runner.
  * `hssf-list` (Các lỗi kinh điển):
    * *Permission denied:* Quên chmod +x cho file gradlew.
    * *Out of memory:* Máy ảo Runner bị hết RAM vật lý khi build JVM.
* **Speaker Notes:** Quy trình chẩn đoán gồm 4 bước. Hãy chú ý tìm đúng dấu X đỏ, đọc console. Các lỗi thường gặp gồm: phân cấp sai file YAML, quên chmod +x, hoặc lỗi RAM (OOM) của máy ảo.

---

### SLIDE 19: Summary
* **HSSF Classes:** `hssf-slide hssf-slide--content`
* **Layout / Components:**
  * `hssf-header` (Title: "Tổng kết Session 07", Subtitle: "Nền tảng tự động hóa CI/CD vững chắc")
  * `hssf-list`:
    * CI/CD giúp tích hợp và kiểm thử code tự động, loại bỏ rủi ro thủ công.
    * Kiến trúc Runner phân tách: Server điều phối, Runner thực thi.
    * File Workflow YAML lưu trong thư mục `.github/workflows/` của Repo.
    * Tận dụng `needs` để thiết lập thứ tự và `matrix` để chạy đa môi trường.
    * Tối ưu hóa pipeline biên dịch Spring Boot bằng cách bật cơ chế cache của Gradle.
* **Speaker Notes:** Tóm lại, Session 07 cung cấp tư duy và công cụ tự động hóa khâu build/test. Bật cache và quản lý job thông minh sẽ nâng cao năng suất của toàn bộ dự án.

---

### SLIDE 20: End
* **HSSF Classes:** `hssf-slide hssf-slide--section`
* **Layout / Components:**
  * `hssf-brand-end`
    * Kicker: `DEVOPS IN ACTION`
    * Title: `HỌC VIỆN ĐÀO TẠO LẬP TRÌNH CHẤT LƯỢNG NHẬT BẢN`
    * Org: `Rikkei Education`
  * `hssf-footer--light hssf-footer--nopage`
* **Speaker Notes:** Cảm ơn các bạn. Hẹn gặp lại các bạn trong Session 08: Đóng gói Docker Image và Đẩy lên Registry.
