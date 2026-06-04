# THIẾT KẾ BÀI TẬP VỀ NHÀ (HOMEWORK DESIGN SPECIFICATION)

Tài liệu này định nghĩa triết lý, cấu trúc, phân bổ độ khó và định dạng nộp bài tập về nhà áp dụng cho toàn bộ học phần **"DevOps thực chiến với Microservices"** (áp dụng cho nền tảng QuickBite).

---

## 1. Triết lý thiết kế bài tập về nhà

Để bám sát triết lý **"Thực hành là trung tâm"** của học phần, bài tập về nhà không kiểm tra lý thuyết suông mà tập trung 100% vào **vận hành thực tế** và **tự động hóa hệ thống**.
* **Giải quyết nỗi đau cụ thể:** Sinh viên làm bài tập để tự tay giải quyết một vấn đề thực tế phát sinh (ví dụ: trùng port, lỗi quyền, cấu hình sai môi trường, tối ưu hóa script deploy).
* **Bảo toàn ngữ cảnh QuickBite:** Toàn bộ bài tập phải dùng đúng ngữ cảnh của hệ thống QuickBite (ban đầu là `user-service`, sau đó mở rộng thành đa dịch vụ ở các session sau).
* **Tránh sao chép thụ động:** Giảm dần sự hỗ trợ "cầm tay chỉ việc" khi độ khó tăng lên nhằm rèn luyện tư duy tự giải quyết vấn đề cho sinh viên.

---

## 2. Cơ cấu số lượng và phân bổ độ khó

Mỗi Session sẽ thiết kế đúng **6 bài tập**, được chia làm **3 mức độ** với phân bổ điểm và yêu cầu như sau:

| Mức độ | Số bài | Yêu cầu kỹ thuật | Phương pháp tiếp cận | Kết quả đánh giá |
| :--- | :---: | :--- | :--- | :---: |
| **Khá** | 4 bài | Vận hành cơ bản các cấu hình, công cụ và dòng lệnh cốt lõi đã học trong các Lesson của Session. | **Có hướng dẫn từng bước** (Step-by-step). Sinh viên chỉ cần làm theo và hiểu bản chất để lấy phản xạ. | **Đạt / Không đạt** |
| **Giỏi** | 1 bài | Xâu chuỗi nhiều công cụ hoặc cấu hình lại hệ thống linh hoạt hơn, mở rộng nhẹ ngoài nội dung học. | **Chỉ cung cấp gợi ý/từ khóa** (Hints). Sinh viên tự liên kết kiến thức để tìm giải pháp. | **Đạt / Không đạt** |
| **Xuất sắc**| 1 bài | Tự động hóa ở mức cao, tối ưu hiệu năng hoặc chẩn đoán, xử lý sự cố nâng cao sát thực tế Production. | **Tự nghiên cứu hoàn toàn** (Independent). Chỉ đưa ra yêu cầu đầu ra, không cung cấp kịch bản. | **Đạt / Không đạt** |

---

## 3. Cấu trúc chuẩn của một bài tập (Format)

Mỗi bài tập trong tài liệu bài tập của Session phải tuân thủ nghiêm ngặt cấu trúc 4 phần sau:

### 3.1 Mục tiêu mong muốn đạt được (Learning Objectives)
* Nêu rõ kỹ năng thực hành cụ thể sinh viên sẽ làm chủ sau khi hoàn thành bài tập (ví dụ: viết được script chạy ngầm, cấu hình kiểm tra port tự động).
* Không dùng từ chung chung như "hiểu về...", hãy dùng các động từ hành động: *đóng gói được, xử lý được, tự động hóa được, chẩn đoán được...*

### 3.2 Mô tả yêu cầu (Requirements)
* Nêu rõ ngữ cảnh thực tế của hệ thống QuickBite và yêu cầu kỹ thuật chi tiết.
* Liệt kê rõ các ràng buộc kỹ thuật (ví dụ: không được dùng user `root`, phải dùng biến môi trường để cấu hình cổng mạng).

### 3.3 Kiểm thử và kết quả mong muốn (Verification & Expected Results)
* Chỉ rõ câu lệnh cụ thể sinh viên cần chạy để tự kiểm tra kết quả bài làm của mình.
* Mô tả chi tiết log đầu ra (console output) hoặc trạng thái hệ thống thành công trông như thế nào.
* Quy định rõ các hình ảnh (screenshot) hoặc file log sinh viên cần chụp lại để làm bằng chứng (evidence).

### 3.4 Hướng dẫn nộp bài (Submission Guidelines)
* Định cấu trúc thư mục nộp bài trên kho lưu trữ Git cá nhân của sinh viên.
* Ví dụ:
  ```text
  github-repo/
  └── homework/
      └── session_01/
          ├── exercise_01/
          │   ├── quickbite-user.service
          │   └── evidence.png
          └── exercise_02/
              └── initial-script.sh
  ```
* Nêu rõ tên file và định dạng file cần đẩy lên Git (ví dụ: `.sh`, `.yml`, `.service`, `.png`).

---

## 4. Tiêu chí đánh giá hoàn thành bài tập (Pass/Fail)

Học phần này đánh giá kết quả bài tập theo cơ chế **Đạt (Pass)** hoặc **Không đạt (Fail)** cho từng bài tập dựa trên các tiêu chí sau:

* **Tính hoạt động:** File script (`.sh`), file cấu hình dịch vụ (`.service`) hoặc mã nguồn phải chạy được trên môi trường tiêu chuẩn mà không gặp lỗi cú pháp.
* **Tính bảo mật:** Không sử dụng tài khoản `root` để khởi chạy ứng dụng (phải dùng user `quickbite`), không để lộ thông tin mật khẩu nhạy cảm trực tiếp trong file code (phải dùng biến môi trường).
* **Đầy đủ bằng chứng:** Cung cấp đầy đủ hình ảnh chụp màn hình kết quả chạy lệnh thực tế trên Terminal hoặc file log hệ thống theo đúng yêu cầu kiểm thử của từng bài.
* **Đúng cấu trúc thư mục:** Nộp bài và lưu trữ file đúng cấu trúc thư mục quy định trên Git repository cá nhân.
