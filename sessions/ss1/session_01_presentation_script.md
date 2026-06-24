Chào các bạn sinh viên, rất vui được đồng hành với các bạn trong môn học Devops này.
Có thể một số bạn đã nghe đến cụm từ Devops, nhưng thầy tin là rất nhiều trong số chúng ta chưa hiểu rõ về Devops đâu, vì thế môn học này là để chúng ta đi bóc tách và giải mã Devops.

Tại thời điểm này của môn học, thì các bạn chỉ cần hiểu Devops là việc triển khai ứng dung web (ví dụ như spring boot) lên trên server và public nó ra internet.

Được rồi, chúng ta sẽ đến với bài đầu tiên, tổng quan Devops và Quy trình CI/CD.

Bài học này có 6 nội dung:
1. Tổng quan Devops, hạn chế của triển khai thủ công
2. Khái niệm CI CD
3. 3 Môi trường Dev Staging và Production
4. Nhắc lại kiến trúc Microservice
5. Linux và vao trò của Linux trong Devops
6. Một vài lệnh quản lý quyền và mạng trong Linux.

Nội dung đầu tiên: Tổng quan Devops và hạn chế của triển khai thủ công.

Qua các môn học Java trước, webservice, rồi microservice, chắc hẳn các bạn đã biết cách chạy ứng dụng web là như thế nào, bấm nút run ở đâu.

Tuy nhiên, nhiêu đó chỉ giúp các bạn là một coder, còn để trở thành một kĩ sư trong thời đại này, việc biết chạy code thôi là chưa đủ.

Thế thì nó xuất hiện một lối tư duy mới, tư duy Production Ready ngay từ đầu.

Code xong là có thể đóng gói được, tự động kiểm thử, tự động deploy, sau khi deploy lên thì có thể giám sát liên tục trạng thái của ứng dụng được, và bất kì vấn đề nào xảy ra trong vòng đời của ứng dụng, chúng ta đều kiểm soát được. 

Để làm được việc đó, chắc chắn chúng ta cần rất nhiều công cụ hỗ trợ, và trong môn học này, đi tìm hiểu và học các công cụ đó là mục tiêu và là điều tất yếu.