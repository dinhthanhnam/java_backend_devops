# QUIZ LESSON FORMAT

## **1. NGUYÊN TẮC CỐT LÕI (5 "KHÔNG" & "NÊN")**
* **Đơn nhiệm:** Mỗi câu hỏi chỉ đánh giá **duy nhất 1** kiến thức/kỹ năng. Không lồng ghép nhiều lỗi (cú pháp, logic, hiệu năng) vào cùng một câu.
* **Ngữ cảnh hóa:** Ưu tiên đưa vào **tình huống thực tế (Scenario)** thay vì hỏi định nghĩa suông.
* **Nhiễu thông minh:** Các phương án sai phải dựa trên **lỗi phổ biến** (ví dụ: lỗi lặp thừa/thiếu 1 lần). Tuyệt đối không dùng đáp án ngớ ngẩn như "Cả 3 đều sai".
* **Đồng nhất:** Các phương án phải tương đồng về độ dài, cấu trúc ngữ pháp và phạm vi kiến thức.
* **Sắc bén (No clues):** Không để lộ từ khóa gợi ý giữa câu hỏi và đáp án.

* **Số lượng:** 5 câu.
* **Mục tiêu:** Nhận diện cú pháp, thông hiểu luồng chạy cơ bản và sàng lọc lỗ hổng kiến thức.
* **Cấu trúc:**
    1. Định nghĩa/Cú pháp.
    2. Luồng thực thi.
    3. Đọc hiểu code mẫu.
    4. Phân biệt/So sánh.
    5. Dự đoán kết quả (có bẫy nhẹ).

Format ở định dạng **Markdown** như sau:

## Q<number>

<Nội dung câu hỏi>

[A]
<Nội dung đáp án A>
[EXP]
<Nội dung giải thích cho A>
[B]
<Nội dung đáp án B>
[EXP]
<Nội dung giải thích cho B>
[C]
<Nội dung đáp án C>
[EXP]
<Nội dung giải thích cho C>
[D]
<Nội dung đáp án D>
[EXP]
<Nội dung giải thích cho D>


@correct: A [hoặc B, C, D]
@point: 20 (default)

* **Tiêu chuẩn:** Sinh viên chỉ mất tối đa **44 giây/câu** để trả lời.


# QUIZ ĐẦU GIỜ FORMAT

## Q<number>

<Nội dung câu hỏi>

[A]
<Nội dung đáp án A>
[EXP]
<Nội dung giải thích cho A>
[B]
<Nội dung đáp án B>
[EXP]
<Nội dung giải thích cho B>
[C]
<Nội dung đáp án C>
[EXP]
<Nội dung giải thích cho C>
[D]
<Nội dung đáp án D>
[EXP]
<Nội dung giải thích cho D>

@correct: A [hoặc B, C, D]
@difficulty: 4 [Hoặc 5, 6, 7, 8, 9] (Đại diện cho mức độ khó của câu hỏi)
@category: 0 [Hoặc 1] (Đại diện cho nội dung câu hỏi thuộc bài cũ (0) hay bài mới (1))

Nếu đáp án có chứa code, code phải cực kì ngắn gọn, hoặc chỉ là pseudo code, và **TUYỆT ĐỐI** không được phép xuống dòng.

Mã mẫu (code ví dụ) sử dụng trong nội dung câu hỏi (Question description) phải được viết trong khối code block (fenced code block) riêng biệt, có định dạng thụt lề (indentation) rõ ràng, chuẩn chỉnh và xuống dòng đúng quy chuẩn lập trình để sinh viên dễ đọc hiểu. Quy tắc không xuống dòng chỉ áp dụng cho mã nguồn nằm bên trong các đáp án lựa chọn [A], [B], [C], [D].

Đáp án của các câu phải có sự ngẫu nhiên khách quan, không được phép để quá nhiều câu liên tiếp cùng chung 1 đáp án (ví dụ, 5 câu liên tiếp chung đáp án C là không được).

Bạn sẽ **cố gắng tối đa (nhưng không ép buộc)** để các đáp án có **độ dài tương đối bằng nhau**, nhằm tránh việc sinh viên dễ đoán đáp án đúng dựa trên độ dài.

Bạn sẽ **tạo đúng số lượng câu hỏi theo yêu cầu**.

Bạn sẽ tuân thủ các nguyên tắc thiết kế sau

## **1. NGUYÊN TẮC CỐT LÕI (5 "KHÔNG" & "NÊN")**
* **Đơn nhiệm:** Mỗi câu hỏi chỉ đánh giá **duy nhất 1** kiến thức/kỹ năng. Không lồng ghép nhiều lỗi (cú pháp, logic, hiệu năng) vào cùng một câu.
* **Ngữ cảnh hóa:** Ưu tiên đưa vào **tình huống thực tế (Scenario)** thay vì hỏi định nghĩa suông.
* **Nhiễu thông minh:** Các phương án sai phải dựa trên **lỗi phổ biến** (ví dụ: lỗi lặp thừa/thiếu 1 lần). Tuyệt đối không dùng đáp án ngớ ngẩn như "Cả 3 đều sai".
* **Đồng nhất:** Các phương án phải tương đồng về độ dài, cấu trúc ngữ pháp và phạm vi kiến thức.
* **Sắc bén (No clues):** Không để lộ từ khóa gợi ý giữa câu hỏi và đáp án.

## **QUY TRÌNH SOẠN QUIZZ ĐẦU GIỜ (KIỂM TRA KIẾN THỨC CŨ, VÀ CHUẨN BỊ KIẾN THỨC MỚI)**

* **Số lượng:** 30 câu.
* **Phân bổ:**
    * **10 câu bài cũ (Trọng tâm Vận dụng/Phân tích/Sáng tạo):** Kiểm tra kỹ năng fix bug, tối ưu hiệu năng và phản xạ lập trình.
    * **5 câu bài mới (Trọng tâm Nhận diện/Thông hiểu):** Kiểm tra việc tự học lý thuyết cơ bản qua E-learning.

* **Ma trận câu hỏi:**
| STT | Phạm vi | Mức độ | Số lượng | Mục tiêu cụ thể |
|-----|--------|--------|---------|----------------|
| Câu 1-8 | Bài cũ | Vận dụng chuyên sâu | 8 câu | Hiện thực hóa cú pháp vào bài toán thực tế nhỏ. |
| Câu 9-14 | Bài cũ | Phân tích chuyên sâu | 6 câu | Đọc hiểu luồng dữ liệu và phát hiện lỗi logic (Debug). |
| Câu 15-20 | Bài cũ | Sáng tạo | 6 câu | Tư duy kiến trúc, tối ưu hiệu năng và giải quyết vấn đề hệ thống. |
| Câu 21-24 | Bài mới | Thông hiểu | 4 câu | Xác nhận tiếp thu khái niệm, thuật ngữ và mục đích công nghệ mới. |
| Câu 25-28 | Bài mới | Vận dụng sơ bộ | 4 câu | Nhận diện cú pháp chuẩn và cách triển khai cơ bản. |
| Câu 29-30 | Bài mới | Phân tích sơ bộ | 2 câu | So sánh công nghệ mới với giải pháp đã biết. |

Quy đổi mức độ khó:
Vận dụng chuyên sâu = 4
Phân tích chuyên sâu = 5
Sáng tạo = 6
Thông hiểu = 7
Vận dụng sơ bộ = 8
Phân tích sơ bộ = 9

* **Tiêu chuẩn:** Sinh viên chỉ mất tối đa **45 giây/câu** để trả lời.


**Tất cả các câu hỏi phải được viết bằng tiếng Việt.**

*Quiz đầu giờ là kết hợp kiến thức của bài cũ và bài mới, nên cần tham khảo bài đọc tương ứng để tạo câu hỏi.*


# QUIZ CUỐI GIỜ FORMAT

## Q<number>

<Nội dung câu hỏi>

[A]
<Nội dung đáp án A>
[EXP]
<Nội dung giải thích cho A>
[B]
<Nội dung đáp án B>
[EXP]
<Nội dung giải thích cho B>
[C]
<Nội dung đáp án C>
[EXP]
<Nội dung giải thích cho C>
[D]
<Nội dung đáp án D>
[EXP]
<Nội dung giải thích cho D>


@correct: A [hoặc B, C, D]
@difficulty: 6 [Hoặc 10, 11] (Đại diện cho mức độ nhận thức của câu hỏi)

@category: 1 (Mặc định)


Bạn sẽ **cố gắng tối đa (nhưng không ép buộc)** để các đáp án có **độ dài tương đối bằng nhau**, nhằm tránh việc sinh viên dễ đoán đáp án đúng dựa trên độ dài.

Nếu đáp án có chứa code, code phải cực kì ngắn gọn, hoặc chỉ là pseudo code, và **TUYỆT ĐỐI** không được phép xuống dòng.

Mã mẫu (code ví dụ) sử dụng trong nội dung câu hỏi (Question description) phải được viết trong khối code block (fenced code block) riêng biệt, có định dạng thụt lề (indentation) rõ ràng, chuẩn chỉnh và xuống dòng đúng quy chuẩn lập trình để sinh viên dễ đọc hiểu. Quy tắc không xuống dòng chỉ áp dụng cho mã nguồn nằm bên trong các đáp án lựa chọn [A], [B], [C], [D].

Đáp án của các câu phải có sự ngẫu nhiên khách quan, không được phép để quá nhiều câu liên tiếp cùng chung 1 đáp án (ví dụ, 5 câu liên tiếp chung đáp án C là không được).

Bạn sẽ **tạo đúng số lượng câu hỏi theo yêu cầu**.

Bạn sẽ tuân thủ các nguyên tắc thiết kế sau

## **1. NGUYÊN TẮC CỐT LÕI (5 "KHÔNG" & "NÊN")**
* **Đơn nhiệm:** Mỗi câu hỏi chỉ đánh giá **duy nhất 1** kiến thức/kỹ năng. Không lồng ghép nhiều lỗi (cú pháp, logic, hiệu năng) vào cùng một câu.
* **Ngữ cảnh hóa:** Ưu tiên đưa vào **tình huống thực tế (Scenario)** thay vì hỏi định nghĩa suông.
* **Nhiễu thông minh:** Các phương án sai phải dựa trên **lỗi phổ biến** (ví dụ: lỗi lặp thừa/thiếu 1 lần). Tuyệt đối không dùng đáp án ngớ ngẩn như "Cả 3 đều sai".
* **Đồng nhất:** Các phương án phải tương đồng về độ dài, cấu trúc ngữ pháp và phạm vi kiến thức.
* **Sắc bén (No clues):** Không để lộ từ khóa gợi ý giữa câu hỏi và đáp án.

## **QUY TRÌNH SOẠN QUIZZ CUỐI GIỜ (KIỂM TRA LẠI KIẾN THỨC VỪA HỌC)**

* **Số lượng:** 30 câu.
* **Cấu trúc:**
    * **6 câu Vận dụng:** Triển khai đúng cú pháp/chức năng công nghệ mới.
    * **5 câu Phân tích:** Đọc luồng code, bắt lỗi logic.
    * **4 câu Sáng tạo:** Lựa chọn giải pháp tối ưu (Best Practice) cho bài toán thực tế.

* **Ma trận câu hỏi:**
| STT | Mức độ | Số lượng | Mục tiêu đánh giá |
|-----|--------|---------|------------------|
| Câu 1-12 | Vận dụng | 12 câu | Kiểm tra khả năng triển khai đúng cú pháp và chức năng cơ bản của công nghệ mới. |
| Câu 13-22 | Phân tích | 10 câu | Kiểm tra khả năng "đọc" luồng code, bắt lỗi logic và hiểu bản chất cơ chế vận hành. |
| Câu 23-30 | Sáng tạo | 8 câu | Đánh giá tư duy lựa chọn giải pháp tốt nhất (Best Practice) cho bài toán thực tế. |

Quy đổi mức độ khó:
Vận dụng = 10
Phân tích = 11
Sáng tạo = 6


* **Tiêu chuẩn:** Sinh viên chỉ mất tối đa **44 giây/câu** để trả lời.


**Tất cả các câu hỏi phải được viết bằng tiếng Việt.**

*Quiz cuối giờ chỉ tập trung vào kiến thức của bài mới, nên cần tham khảo bài đọc tương ứng để tạo câu hỏi.*