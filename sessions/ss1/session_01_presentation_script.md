Chào các bạn sinh viên, rất vui được đồng hành với các bạn trong môn học Devops này.
Có thể một số bạn đã nghe đến cụm từ Devops.
Thế nhưng thầy tin rằng rất nhiều trong số chúng ta chưa hiểu rõ về Devops đâu.
Vì thế môn học này là để chúng ta đi bóc tách và giải mã Devops.

Thì tại thời điểm này của môn học

Các bạn chỉ cần hiểu Devops là việc triển khai ứng dung web (ví dụ như spring boot) lên trên server rồi public nó ra internet.

Được rồi, chúng ta sẽ đến với bài đầu tiên, tổng quan Devops và Quy trình CI/CD.

Bài học này có 6 nội dung:
1. Tổng quan Devops, hạn chế của triển khai thủ công
2. Khái niệm CI CD
3. 3 Môi trường Dev Staging và Production
4. Nhắc lại kiến trúc Microservice
5. Linux và vao trò của Linux trong Devops
6. Một vài lệnh quản lý quyền và mạng trong Linux.

Nội dung đầu tiên: Tổng quan Devops và hạn chế của triển khai thủ công.

Qua các môn học Java trước, webservice, rồi microservice, chắc hẳn các bạn đã biết cách chạy ứng dụng web là như thế nào, chính là việc bấm nút Run trên IDE của chúng ta.

Tuy nhiên, nhiêu đó chỉ giúp các bạn là một coder, còn trong bối cảnh hiện nay, để cạnh tranh trong thị trường việc làm, chúng ta cần có tư duy của một kĩ sư.

Và cái tư duy kĩ sư đó mà thầy muốn nhắc đến là tư duy Production Ready.

Tức là: 

Code xong là có thể đóng gói được, tự động kiểm thử, tự động deploy.

Sau khi deploy lên thì có thể giám sát liên tục trạng thái của ứng dụng được.

Và bất kì vấn đề nào xảy ra trong vòng đời của ứng dụng, chúng ta đều kiểm soát được. 

Để làm được việc đó, chắc chắn chúng ta cần rất nhiều công cụ hỗ trợ.

Và môn học này, là việc chúng ta đi tìm hiểu, ứng dụng các công cụ đó.

Chúng ta sẽ đi tìm hiểu thử các kịch bản triển khai mà đã từng là ác mộng trong quá khứ khi chưa có CI CD.

Ở đây chúng ta có một dự án giả định tên là Quickbite, là một nền tảng giao đồ ăn, và được phát triển theo kiến trúc microservice.