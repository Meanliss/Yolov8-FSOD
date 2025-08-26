# Few-Shot Object Detection với YOLOv8 trên DOTA v1.5
*Một thử nghiệm về phương pháp huấn luyện 2 giai đoạn cho các lớp đối tượng hiếm (Few-Shot Learning).*

---

## 📝 Tổng quan Dự án

Dự án này trình bày một phương pháp học ít mẫu (few-shot learning) cho bài toán phát hiện vật thể, sử dụng mô hình **YOLOv8** trên một phiên bản tùy chỉnh của bộ dữ liệu **DOTA v1.5**. Mục tiêu là kiểm tra khả năng của mô hình trong việc học và nhận diện các lớp đối tượng hiếm (novel classes) chỉ với một vài ví dụ.

Do giới hạn về tài nguyên, một bộ dữ liệu con đã được tạo ra từ DOTA v1.5, bao gồm **3 lớp đối tượng**, được phân loại như sau:
* **2 Base Classes:** Các lớp phổ biến, có nhiều dữ liệu.
* **1 Novel Class:** Lớp hiếm, có rất ít dữ liệu.

---

## ⚙️ Phương pháp & Kỹ thuật

Phương pháp được lấy cảm hứng từ TFA (Two-stage Fine-tuning Approach) và được điều chỉnh để áp dụng cho kiến trúc YOLOv8.

### Giai đoạn 1: Base Training
Mô hình YOLOv8 được huấn luyện trên toàn bộ dữ liệu của 3 lớp. Giai đoạn này nhằm mục đích trang bị cho mô hình kiến thức nền tảng về các đặc trưng chung (cạnh, góc, texture) và khả năng nhận diện tốt các lớp Base Class.

### Giai đoạn 2: Fine-tuning cho Lớp Novel
"Base model" từ Giai đoạn 1 được tinh chỉnh trên một tập dữ liệu được chuẩn bị đặc biệt, với tỉ lệ **80%** ảnh chứa Novel Class và **20%** ảnh chỉ chứa Base Class.

Một kỹ thuật then chốt được sử dụng trong giai đoạn này là **Đóng băng các lớp (Layer Freezing)**. Hầu hết các lớp trong mạng (phần backbone - chịu trách nhiệm trích xuất đặc trưng) được đóng băng, chỉ để lại các lớp cuối cùng (lớp detection head - chịu trách nhiệm phân loại và định vị) được phép cập nhật trọng số.

> **Logic đằng sau Layer Freezing:** Kỹ thuật này giữ lại toàn bộ kiến thức về đặc trưng hình ảnh mà mô hình đã học được ở Giai đoạn 1. Việc chỉ cập nhật các lớp cuối giúp mô hình tập trung toàn bộ "năng lượng học" vào việc liên kết các đặc trưng hiện có với một cái nhãn mới (Novel Class), thay vì phải học lại từ đầu. Điều này giúp tăng hiệu quả huấn luyện cho lớp hiếm và chống lại hiện tượng "quên" kiến thức về các lớp phổ biến (Catastrophic Forgetting).

---

## 📊 Kết quả & Phân tích

Kết quả so sánh hiệu suất giữa mô hình sau Giai đoạn 1 và Giai đoạn 2 cho thấy:

* **Về Novel Class:** Khả năng nhận diện lớp hiếm **không cho thấy sự cải thiện**. Mô hình gần như không đưa ra dự đoán nào cho lớp này.
* **Về Chỉ số Tổng quan (mAP):** Mặc dù vậy, chỉ số mAP tổng thể của mô hình sau Giai đoạn 2 **thực sự đã tăng lên** so với Giai đoạn 1.

---

## 💡 Thảo luận & Hướng phát triển

Việc mô hình chưa nhận diện được Novel Class có thể được giải thích là do **số lượng mẫu và thời gian huấn luyện (epoch) còn quá ít**.

Tuy nhiên, sự cải thiện của chỉ số mAP tổng thể là một tín hiệu tích cực, cho thấy phương pháp fine-tuning đã giúp mô hình cân bằng và tối ưu hóa các quyết định của mình. Điều này cho thấy tiềm năng của phương pháp: nếu được huấn luyện với quy mô lớn hơn, khả năng nhận diện Novel Class có thể sẽ được cải thiện đáng kể.

**Các hướng phát triển tiếp theo:**
* Tăng số lượng ảnh cho Novel Class trong Giai đoạn 2.
* Tăng số epoch huấn luyện cho Giai đoạn 2.
* Thử nghiệm các chiến lược fine-tuning khác (ví dụ: unfreeze một vài lớp cuối của backbone).

---

## 🛠️ Yêu cầu Cài đặt

Dự án được xây dựng dựa trên các thư viện chính sau:
* `ultralytics`
* `torch`
* `numpy`
* `opencv-python`
