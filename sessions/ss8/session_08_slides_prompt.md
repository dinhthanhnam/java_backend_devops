# PROMPT CHO GEMINI — GOOGLE SLIDES SESSION 08

## 1. Nhiệm vụ và nguồn nội dung

Bạn là giảng viên DevOps và chuyên gia thiết kế slide đào tạo kỹ thuật. Hãy tạo một bộ **Google Slides 16:9** bằng tiếng Việt cho Session 08 của học phần Java Backend & DevOps. Số slide được quyết định bởi mật độ nội dung; không thêm hoặc gộp slide chỉ để khớp số lượng của file mẫu.

- Chủ đề: **Đóng gói Docker Image & Đẩy lên Registry**.
- Hệ thống xuyên suốt: **QuickBite Food Delivery Microservices Platform**.
- Đối tượng: học viên đã biết Docker Compose, Gradle Wrapper và GitHub Actions cơ bản.
- Mục tiêu: học viên hiểu cách đóng gói Spring Boot thành Docker image, tối ưu Dockerfile, gắn phiên bản, push/pull image qua GitHub Container Registry và kiểm tra image đúng mức trong workflow.

Sử dụng `session_08_slide_content.md` làm **nguồn nội dung duy nhất**. Không tự thêm chủ đề ngoài Docker, GitHub Actions, GitHub Container Registry và QuickBite. Giữ nguyên các tên lesson sau, đặc biệt trên divider:

1. Quy trình Build Docker image trong pipeline CI/CD
2. Tối ưu hóa Dockerfile cho Production (Multi-stage build)
3. Phiên bản hóa và đẩy Docker image lên Registry từ Local
4. Sử dụng Docker Image từ Registry trong Luồng CI/CD
5. Kịch bản Thực hành Tổng hợp với user-service

## 2. Trình tự slide bắt buộc

| Slide | Vai trò |
|---|---|
| 01 | Tiêu đề chính |
| 02 | Agenda |
| 03 | Divider Lesson 01 |
| 04–07 | Nội dung Lesson 01 |
| 08 | Divider Lesson 02 |
| 09–12 | Nội dung Lesson 02 |
| 13 | Divider Lesson 03 |
| 14–16 | Nội dung Lesson 03 |
| 17 | Divider Lesson 04 |
| 18–19 | Nội dung Lesson 04 |
| 20 | Divider Lesson 05 |
| 21–25 | Nội dung Lesson 05 |
| 26 | Tổng kết Session 08 |
| 27 | Slide kết thúc |

Mỗi lesson là một cụm độc lập để quay video riêng. Không trộn nội dung giữa lesson; divider là điểm bắt đầu của video lesson tương ứng.

## 3. Tham chiếu thiết kế bắt buộc từ `(Slide) Session 7.pptx`

Đây là **mẫu trực tiếp** cho thiết kế Session 08. Giữ đồng nhất font, cỡ chữ, khoảng cách, lề, footer, số trang và nhịp chuyển bố cục. Không sáng tạo một theme mới, không dùng layout kiểu dashboard hay card grid dày đặc.

### Typography

- Dùng **Montserrat** xuyên suốt. Tiêu đề dùng Montserrat ExtraBold; heading của divider dùng Montserrat; agenda và nội dung thông thường dùng Montserrat; slide tổng kết dùng Montserrat Black cho chữ `TỔNG KẾT`.
- Không thay font bằng Inter, Be Vietnam Pro, Arial hoặc font khác.
- Slide 01: `Session 08:` và title chính đều 30 pt, Montserrat ExtraBold.
- Slide 02: chữ `NỘI DUNG` dọc bên trái 60 pt Montserrat Black; danh sách agenda 24 pt Montserrat.
- Divider: số lesson trong vòng tròn 70 pt; tên lesson 40 pt. Dù title dài, giữ một cỡ chữ này; cho phép ngắt dòng có chủ ý, không thu nhỏ font.
- Slide nội dung: title chính 26.5 pt Montserrat ExtraBold; subtitle 18 pt Montserrat ExtraBold; body tối thiểu 14 pt. Chỉ dùng body 20–24 pt cho layout ít chữ giống slide ví dụ, không dùng cỡ chữ khác tùy tiện.
- Code block dùng Courier New nền tối; ưu tiên 8.5–10 pt như mẫu. Nếu code không vừa, rút ngắn code hoặc tách ý, không thu nhỏ tiếp.
- Số slide góc phải dưới 12 pt Montserrat. Không để số slide trùng lặp.

### Spacing và bố cục

- Dùng khung 16:9, cùng khoảng trắng và lề như mẫu. Trên các slide nội dung, title bắt đầu khoảng `x = 66`, `y = 37`; subtitle khoảng `y = 101`. Không đẩy title sát mép trên hoặc sát logo.
- Với layout hai khối, hai panel bắt đầu gần `x = 66` và `x = 492`, rộng khoảng 402 mỗi panel, có khe giữa khoảng 24; phần đệm trong panel khoảng 18–21. Hai panel phải cùng mép trên, cùng chiều cao và cùng nhịp dòng.
- Không tự thay đổi khoảng cách giữa title, subtitle, nội dung, footer và số trang theo từng slide. Nội dung dài phải được biên tập lại trước, không bóp nhỏ chữ hoặc làm panel cao thấp tùy tiện.
- Mỗi slide nội dung chỉ có một thông điệp chính. Tối đa hai panel nội dung hoặc một vùng code kèm một vùng giải thích; tránh ba hoặc bốn cột nhỏ.
- Giữ logo Rikkei Academy ở góc phải trên, footer bản quyền ở góc trái dưới và tam giác đỏ/số trang góc phải dưới như trong mẫu. Dùng đúng asset từ mẫu; không tạo lại logo bằng AI.

### Màu sắc và họa tiết

- Slide nội dung: nền trắng hoặc trắng rất nhạt; title/subtitle đỏ đậm cùng tông mẫu. Dùng panel xanh tím rất nhạt và cam rất nhạt với viền mảnh như mẫu khi cần đối chiếu hai ý.
- Divider và slide kết thúc: nền đỏ đậm phủ toàn slide, có texture hình học tinh tế; logo trắng ở góc trái trên; cụm chấm trắng ở cạnh phải. Title và số lesson màu trắng.
- Slide 01, 02, 25 dùng đúng bố cục nền trắng, mũi tên đỏ và cụm chấm đỏ của mẫu. Không thay bằng ảnh stock, gradient, icon 3D hoặc hiệu ứng phát sáng.
- Chỉ dùng icon line đơn giản khi thực sự cần: Docker socket, artifact hoặc cảnh báo. Không dùng nhiều icon trang trí.

## 4. Quy tắc áp dụng theo loại slide

- **Slide 01:** chỉ `Session 08:` và title. Không phụ đề, quote, agenda hay sơ đồ.
- **Slide 02:** chỉ agenda 5 lesson, đánh số như nội dung nguồn; giữ chữ `NỘI DUNG` dọc ở bên trái.
- **Slide divider 03, 08, 13, 17, 20:** dùng đúng layout divider đỏ của Slide 3/8/13/17/20 trong mẫu. Chỉ thay số lesson và title lesson; title phải là title curriculum nguyên văn.
- **Slide nội dung:** dùng các layout nền sáng của mẫu. Với slide so sánh, dùng hai panel cân xứng. Với quy trình, đặt flow đơn giản hoặc bốn bước ngang. Với code, đặt code block nền tối bên trái và phần giải thích nền sáng bên phải. Không nhét một đoạn văn dài vào slide.
- **Slide 26:** dùng layout tổng kết của Slide 25 mẫu: chữ `TỔNG KẾT` lớn bên trái và danh sách 5 ý bên phải; không thêm CTA hoặc nội dung mới.
- **Slide 27:** dùng layout kết thúc nền đỏ của Slide 26 mẫu, giữ nguyên câu nhận diện học viện trong nguồn nội dung.

## 5. Chuẩn ngôn ngữ

- Viết tiếng Việt đầy đủ, tự nhiên và chính xác. Không biến nội dung thành slogan, cụm từ rời rạc hoặc câu giật gân.
- Giữ nhất quán các thuật ngữ: Docker image, layer, build context, registry, tag, digest, artifact, Runner, workflow, cache.
- Tiêu đề nội dung có thể diễn đạt thông điệp kỹ thuật; title divider phải giữ nguyên title lesson trong curriculum.
- Không đưa lời dẫn của người thuyết trình, thời lượng, ghi chú sản xuất slide hoặc hướng dẫn cho Gemini lên slide.

## 6. Sự chính xác kỹ thuật bắt buộc

- `user-service` dùng Java 17. `restaurant-service`, `order-service`, `notification-service` dùng Java 21.
- QuickBite hiện có Dockerfile đơn tầng cho `user-service`, Dockerfile multi-stage cho `restaurant-service`, và `user-service/.github/workflows/ci.yml`. Ví dụ image pipeline phải là file `image.yml` song song `ci.yml`, không ghi đè workflow hiện có.
- Self-hosted Runner truy cập Docker daemon qua `/var/run/docker.sock`. Nêu cả lợi ích cache lẫn rủi ro quyền cao trên host; chỉ chạy workflow tin cậy.
- Workflow publish GHCR dùng `GITHUB_TOKEN` với quyền tối thiểu: `contents: read`, `packages: write`. Workflow chỉ pull private image dùng `packages: read`.
- Khi thao tác local với PAT, phải dùng `--password-stdin`; không lộ token trong command, YAML, Dockerfile, ảnh hoặc log. Chỉ dùng placeholder `${CR_PAT}` hoặc `<token được lưu ngoài repository>`.
- Không hứa hẹn thời gian build/pull hoặc dung lượng image cố định. Cache phụ thuộc builder, build context và thay đổi source.
- `docker ps` không chứng minh Spring Boot đã sẵn sàng phục vụ. Smoke test `java -version` chỉ xác nhận runtime; kiểm tra ứng dụng cần PostgreSQL, `quickbite-net`, biến `DB_*` và health endpoint phù hợp.

## 7. Kiểm tra trước khi xuất

- Các slide đi đúng thứ tự của nội dung nguồn và đúng title lesson; số lượng cuối cùng phải phục vụ việc giải thích rõ ràng, không bị ép theo Session 07.
- Font, font size, spacing, logo, footer và số trang nhất quán với `(Slide) Session 7.pptx` từ đầu đến cuối.
- Không có chữ tràn, title tự xuống dòng ngoài chủ đích, số trang lặp hoặc panel lệch nhau.
- Mọi đoạn YAML, Dockerfile và Bash giữ đúng thụt lề; không chứa token thật.
- Không có nội dung ngoài phạm vi Session 08 như Kubernetes, Docker Swarm, Docker Hub hay công cụ CI khác.
