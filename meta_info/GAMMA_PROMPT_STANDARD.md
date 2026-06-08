# TIÊU CHUẨN SOẠN THẢO PROMPT CHO GAMMA (GAMMA PROMPT STANDARD)

Tài liệu này định nghĩa tiêu chuẩn thiết lập câu lệnh (prompt) gửi cho Gamma AI. Mục tiêu là cung cấp một **dàn ý nội dung giàu chi tiết, có phân cấp cấu trúc rõ ràng**, đồng thời ra lệnh cho Gamma **tự động phân bổ và quyết định số lượng slide tối ưu**, tránh việc gò ép cứng nhắc làm nhồi nhét chữ trên một trang.

---

## 1. Cấu trúc Khung của một Prompt Gamma Linh hoạt

Thay vì định nghĩa cứng nhắc `Slide 1`, `Slide 2`..., một prompt tối ưu cho Gamma sẽ gồm 4 phần chính:

```text
┌────────────────────────────────────────────────────────┐
│ 1. CONTEXT & ROLE (Bối cảnh khóa học & Vai trò giảng)   │
├────────────────────────────────────────────────────────┤
│ 2. DESIGN & VISUAL SPECS (Quy chuẩn màu sắc & Bố cục)   │
├────────────────────────────────────────────────────────┤
│ 3. PARTITION INSTRUCTIONS (Chỉ thị tự động chia slide)  │
├────────────────────────────────────────────────────────┤
│ 4. HIERARCHICAL OUTLINE (Dàn ý chi tiết giàu thông tin) │
└────────────────────────────────────────────────────────┘
```

---

## 2. Đặc tả chi tiết từng phần trong Prompt

### 2.1 Bối cảnh & Vai trò (Context & Role)
* **Vai trò:** DevOps Architect kiêm Giảng viên cao cấp. Giọng điệu trực diện, thực chiến, dùng ngôn ngữ ngành ("ăn hành", "tech lead", "sập server", "lệch cấu hình").
* **Đối tượng:** Kỹ sư phần mềm, intern mới vào dự án QuickBite.

### 2.2 Quy chuẩn Thiết kế (Design & Visual Specs)
* **Bảng màu:** Dark Mode chủ đạo (nền `#121214`), màu nhấn Neon Emerald Green (trạng thái chạy ngon/thành công) và Neon Red/Orange (cảnh báo lỗi hệ thống).
* **Typography:** Font chữ hiện đại (*Inter*, *Outfit*).

### 2.3 Chỉ thị Phân tách Slide Động (Partition Instructions)
Đây là phần quan trọng nhất để Gamma tự điều phối slide:
* **Yêu cầu phân tách:** *"Dựa trên dàn ý chi tiết dưới đây, hãy tự động phân tích và chia nhỏ nội dung thành số lượng slide phù hợp (khuyến nghị từ 12-18 slides). Không được nhồi nhét nhiều chủ đề vào cùng một slide. Đảm bảo nguyên tắc: **Một slide chỉ trình bày một thông điệp/khái niệm cốt lõi** để tạo không gian thoáng đãng, dễ đọc."*
* **Yêu cầu layout:** Tự chọn bố cục phù hợp với nội dung (ví dụ: Timeline cho quy trình, 2 cột cho so sánh, Grid cards cho phân loại thành phần).

### 2.4 Dàn ý Chi tiết giàu thông tin (Hierarchical Outline)
Cung cấp nội dung dưới dạng sơ đồ cây phân cấp giàu chi tiết, đầy đủ ví dụ thực tế và sơ đồ ASCII:
* **Chủ đề chính (H1):** Định vị phần nội dung lớn.
* **Chủ đề phụ (H2):** Phân rã bài học.
* **Nội dung gạch đầu dòng (Bullet points):** Có cấu trúc chi tiết, giải thích rõ nguyên nhân, hậu quả và giải pháp.
* **Ví dụ thực tế / Lệnh chạy / Sơ đồ ASCII:** Đóng vai trò làm bằng chứng trực quan để Gamma tự động đóng gói vào khối code hoặc diagram.

---

## 3. Hướng dẫn sử dụng
1. Copy toàn bộ nội dung prompt được thiết kế theo tiêu chuẩn này.
2. Dán vào ô nhập liệu của Gamma App (**Text to Presentation**).
3. Chọn một theme tối (Dark Theme) và bấm **Generate**. Gamma sẽ tự động chia slide và thiết kế bố cục tối ưu nhất cho từng phần của dàn ý.
