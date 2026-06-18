# TIÊU CHUẨN SOẠN THẢO PROMPT SLIDE CHO GAMMA AI (SLIDE DEVELOPMENT GUIDELINE)

Tài liệu này định nghĩa tiêu chuẩn và phương pháp thiết kế câu lệnh (prompt) gửi cho Gamma AI để tạo slide bài giảng cho học phần. Mục tiêu là cung cấp một cấu trúc prompt chuẩn hóa, đảm bảo tính chuyên nghiệp, cô đọng cao về mặt nội dung, để Gamma AI có thể kết xuất (render) chính xác mà không tự ý suy diễn thông tin.

---

## 1. Cấu trúc Khung của một Prompt Slide chuẩn

Một slide prompt hoàn chỉnh gửi cho Gamma AI bao gồm 3 phần chính sau:

```text
┌────────────────────────────────────────────────────────┐
│ 1. CONTEXT & ROLE (Bối cảnh khóa học & Vai trò giảng)   │
├────────────────────────────────────────────────────────┤
│ 2. PARTITION INSTRUCTIONS (Chỉ thị phân tách slide)    │
├────────────────────────────────────────────────────────┤
│ 3. HIERARCHICAL OUTLINE (Dàn ý chi tiết cung cấp text)  │
└────────────────────────────────────────────────────────┘
```

### 1.1 Context & Role (Bối cảnh & Vai trò)
* **Vai trò:** DevOps Architect kiêm Giảng viên cao cấp. Giọng điệu chuyên nghiệp, trực diện, hàn lâm thực hành, giảng giải rõ ràng từ bản chất kỹ thuật.
* **Đối tượng:** Kỹ sư phần mềm, học viên đang phát triển hệ thống Spring Boot Microservices (QuickBite).

### 1.2 Partition Instructions (Chỉ thị phân bổ slide)
* **Số lượng:** Khuyến nghị từ 12 - 20 slides tùy theo độ dài session.
* **Nguyên tắc phân chia:** Một slide chỉ trình bày một thông điệp/khái niệm cốt lõi để đảm bảo không gian thoáng đãng, dễ đọc.

---

## 2. Ba Nguyên tắc cốt lõi khi thiết kế Dàn ý (Hierarchical Outline)

### Nguyên tắc 1: Phong cách chuyên nghiệp và cực kỳ cô đọng
* **Tính chuyên nghiệp:** Ngôn ngữ sử dụng phải chuẩn mực kỹ thuật, nghiêm túc và học thuật. Loại bỏ hoàn toàn các cách nói suồng sã, giật gân, ví von quá đà (như "ăn hành", "intern", "ngây thơ", "khờ khạo", "loay hoay").
* **Độ cô đọng cao:** Ưu tiên cấu trúc câu ngắn gọn, trực diện, đi thẳng vào bản chất kỹ thuật. Loại bỏ các từ đệm, từ nối hoặc các câu chuyện dẫn nhập mang tính giật gân không cần thiết.
* **Cú pháp:** Sử dụng mã nguồn thực tế (code block), bảng biểu Markdown và sơ đồ dòng chảy ASCII trực tiếp trong dàn ý để minh họa trực quan.

### Nguyên tắc 2: Đồng bộ tiêu đề và trình tự của Lesson
* Các phần lớn của dàn ý slide prompt phải tương ứng trực tiếp với danh sách các Lesson trong session (ví dụ: `LESSON 01`, `LESSON 02`...) và đi theo đúng trình tự bài học.

### Nguyên tắc 3: Thiết kế nhịp độ (Pacing) bài học
* **Lesson đầu tiên của Session (Lesson 01 - Mở đầu & Dẫn dắt):**
  * Mang tính chất đặt vấn đề, so sánh tư duy hệ thống và phân tích các trường hợp lỗi triển khai thực tế.
  * Có phần chuyển tiếp (transition) dẫn dắt logic sang các bài học kỹ thuật phía sau.
* **Các Lesson tiếp theo (Lesson 02 trở đi - Đi thẳng vào giải pháp):**
  * Bỏ qua các phần dẫn nhập dông dài.
  * Đi thẳng vào định nghĩa cốt lõi, cơ chế hoạt động, sơ đồ kỹ thuật và các ví dụ mã nguồn/câu lệnh thực hành ngay từ slide đầu tiên của lesson đó.

---

## 3. Template mẫu cho một chủ đề trong Dàn ý (Hierarchical Outline)

Dưới đây là cấu trúc gợi ý khi viết nội dung cho một Lesson trong prompt:

```markdown
### LESSON [Số thứ tự]: [Tên bài học giống hệt tiêu đề bài đọc]

* **Vấn đề thực tế (Khởi đầu nhanh):**
  * Mô tả ngắn gọn tình huống thực tế hoặc lỗi kỹ thuật phát sinh nếu không sử dụng công nghệ này.
* **Khái niệm cốt lõi / Định nghĩa:**
  * Giải thích chi tiết bản chất kỹ thuật bằng ngôn ngữ chuyên nghiệp (Cung cấp sẵn text giải thích ngắn gọn).
* **Sơ đồ luồng dữ liệu / Luồng hoạt động (nếu có):**
  * Sử dụng sơ đồ ASCII đơn giản để mô tả trực quan dòng chảy của dữ liệu hoặc các bước thực hiện.
* **Ví dụ thực hành & Giải thích cấu hình:**
  * Cung cấp các khối lệnh chạy thực tế hoặc cấu hình hoàn chỉnh.
  * Giải thích ý nghĩa của các tham số quan trọng trong câu lệnh/cấu hình đó để người đọc hiểu bản chất.
```

---

## 4. Quy trình xây dựng Prompt Slide từ Bài đọc

1. **Bước 1 (Đọc & Lọc):** Đọc kỹ các bài đọc (`session_XX_lesson_YY.md`) của Session hiện tại để thu thập các khái niệm cốt lõi, ví dụ lệnh và file cấu hình.
2. **Bước 2 (Viết Lesson 01 - Tạo động lực chuyên nghiệp):** Thiết kế slide cho Lesson 01 tập trung vào so sánh tư duy và các trường hợp lỗi triển khai thực tế của hệ thống QuickBite.
3. **Bước 3 (Triển khai các Lesson sau - Thực chiến & Trực quan):** Chuyển đổi các phần kiến thức cốt lõi của các bài học tiếp theo thành các slide chứa lời giải thích rõ ràng và ví dụ cụ thể. Tối ưu hóa các câu lệnh và cấu hình thành dạng dễ copy.
4. **Bước 4 (Ghép Khung & Kiểm tra):** Ghép dàn ý chi tiết vào khung prompt chuẩn (Context, Partition) và kiểm tra lại sự đồng bộ về tiêu đề/thứ tự với các bài học gốc.
