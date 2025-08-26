Few-Shot Object Detection trên DOTA v1.5 với YOLOv8
Đây là một dự án demo thử nghiệm phương pháp học ít mẫu (few-shot learning) cho bài toán phát hiện vật thể, sử dụng mô hình YOLOv8 trên một phiên bản tùy chỉnh của bộ dữ liệu DOTA v1.5.

📝 Tổng quan
Trong các bài toán thực tế, việc thu thập và gán nhãn cho các lớp đối tượng hiếm (novel class) thường rất tốn kém. Hướng tiếp cận few-shot learning giúp mô hình học cách nhận biết các lớp mới chỉ với một vài ví dụ.

Do thời gian và tài nguyên có hạn, dự án này được thực hiện trên một bộ dữ liệu đã qua xử lý:

Tạo Dataset con: Từ bộ DOTA v1.5 gốc, dữ liệu được cắt và lọc để tạo ra một bộ dataset nhỏ hơn chỉ chứa 3 lớp đối tượng.

Phân loại Class: 3 lớp này được chia thành:

2 Base Classes: Các lớp phổ biến, có nhiều dữ liệu.

1 Novel Class: Lớp hiếm, có rất ít dữ liệu.

Mục tiêu của dự án là kiểm chứng một phương pháp huấn luyện 2 giai đoạn để cải thiện khả năng nhận diện Novel Class mà không làm suy giảm hiệu suất trên các Base Classes.

⚙️ Phương pháp tiếp cận
Phương pháp được lấy cảm hứng từ TFA (Two-stage Fine-tuning Approach), thường thấy trong các mô hình như Faster R-CNN, và được áp dụng cho YOLOv8.

Giai đoạn 1: Base Training
Ở giai đoạn này, mô hình YOLOv8 được huấn luyện một cách thông thường trên toàn bộ dữ liệu của 3 lớp.

Mục tiêu: Giúp mô hình học các đặc trưng chung (cạnh, góc, texture) và học cách nhận diện tốt 2 lớp Base Class. Kết quả của giai đoạn này là một "base model" có nền tảng kiến thức vững chắc.

Giai đoạn 2: Fine-tuning cho Lớp Novel
Đây là giai đoạn cốt lõi của thử nghiệm. "Base model" từ Giai đoạn 1 được tinh chỉnh (fine-tune) trên một tập dữ liệu được chuẩn bị đặc biệt.

Chuẩn bị dữ liệu: Một tập dữ liệu mới được tạo ra với tỉ lệ mất cân bằng có chủ đích:

80% dữ liệu chứa các ảnh có Novel Class.

20% dữ liệu là các ảnh chỉ chứa Base Class.

Mục đích là để "ép" mô hình tập trung học vào các đặc điểm của lớp hiếm.

Đóng băng các lớp (Layer Freezing): Trước khi huấn luyện, hầu hết các lớp trong mạng (phần backbone) được đóng băng (freeze), chỉ để lại các lớp cuối cùng (lớp detection head) được phép cập nhật trọng số.

🧠 Giải thích về Layer Freezing
Có thể hình dung mô hình YOLOv8 như một chuyên gia phân tích hình ảnh. Các lớp đầu và giữa (backbone) giống như đôi mắt và vùng não thị giác. Sau Giai đoạn 1, "đôi mắt" này đã rất giỏi trong việc nhận ra các đặc trưng cơ bản như hình dạng tàu thủy, cánh máy bay (Base Classes).

Việc đóng băng các lớp này tương đương với việc ra lệnh: "Giữ nguyên khả năng nhìn và nhận biết đặc trưng đã học được, không thay đổi nó."

Các lớp cuối (detection head) thì giống như vùng não ra quyết định và dán nhãn. Bằng cách chỉ huấn luyện các lớp này, thông điệp được gửi đến là: "Sử dụng khả năng nhìn siêu việt có sẵn để học cách dán một cái nhãn mới (Novel Class) cho một vật thể lạ. Không cần học lại cách nhìn, chỉ cần học cách gọi tên cái mới."

Kết quả: Kỹ thuật này giúp mô hình dồn toàn bộ tài nguyên học vào việc nhận diện Novel Class mà không làm xáo trộn hay "quên" đi kiến thức về các Base Class đã được huấn luyện kỹ. Đây là cách để chống lại hiện tượng "Catastrophic Forgetting" (Quên thảm khốc).

📊 Kết quả và Phân tích
So sánh hiệu suất của model sau Giai đoạn 1 và Giai đoạn 2 cho thấy các kết quả sau:

Khả năng nhận diện Novel Class: Khả năng đoán đúng Novel Class không tăng sau Giai đoạn 2. Mô hình gần như không đưa ra dự đoán nào cho lớp này (chỉ có một lần dự đoán nhưng không chính xác).

Chỉ số tổng quan: Tuy nhiên, một điểm sáng là các chỉ số tổng quan (ví dụ: mAP) của mô hình sau Giai đoạn 2 thực sự đã cải thiện so với Giai đoạn 1.

Thảo luận
Việc mô hình chưa nhận diện được Novel Class có thể là do số lượng mẫu và thời gian huấn luyện (epoch) còn quá ít, khiến mô hình chưa có đủ cơ hội để học các đặc trưng của lớp hiếm.

Mặc dù vậy, việc chỉ số tổng quan tăng cho thấy phương pháp fine-tuning về cơ bản là có hiệu quả, giúp mô hình cân bằng lại quyết định của mình tốt hơn. Có thể giả định rằng, nếu thử nghiệm này được mở rộng với quy mô lớn hơn (nhiều dữ liệu cho lớp novel hơn và training lâu hơn), phương pháp tiếp cận 2 giai đoạn này hoàn toàn có tiềm năng cải thiện đáng kể khả năng phát hiện các lớp đối tượng hiếm.

🚀 Hướng phát triển
Tăng số lượng ảnh cho Novel Class trong Giai đoạn 2.

Tăng số epoch huấn luyện cho Giai đoạn 2.

Thử nghiệm các chiến lược fine-tuning khác.
