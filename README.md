# NLP - Multi-label Intent and Slot Filling

Repository này chứa mã nguồn giải quyết bài toán đa nhiệm: Phân loại Ý định đa nhãn (Multi-label Intent) và Trích xuất Thực thể (Slot Filling) cho hệ thống trợ lý ảo tiếng Việt.

## Mô tả Bài toán
Hệ thống trợ lý ảo cần hiểu được câu lệnh tiếng Việt của người dùng (ví dụ: "gọi tôi dậy lúc chín giờ sáng ngày thứ sáu") thông qua hai nhiệm vụ thực thi song song:
1. **Intent (Ý định):** Xác định mục tiêu của người dùng muốn trợ lý làm gì; khoảng 13 % số câu ghép hai yêu cầu bằng các từ nối như "và", "rồi", "sau đó", "với lại", "xong thì", "nhân tiện". Trong bài nộp, các ý định đa nhãn được phân tách bằng dấu `#` (ví dụ: `alarm_set#weather_query`)[cite: 15].
2. **Slot (Thực thể):** Trích xuất các cụm từ mang giá trị tham số của yêu cầu (như thời gian, địa điểm, thiết bị...)[cite: 11]. Mỗi token (âm tiết) trong câu sẽ được gán một nhãn duy nhất theo tiêu chuẩn BIO (Begin - Inside - Outside)[cite: 11, 15]. Ví dụ cụm "chín giờ sáng" được gán chuỗi `B-time I-time I-time`[cite: 15].

## Phương pháp Thực hiện (Methodology)
Hệ thống chấm điểm đánh giá hiệu suất thông qua trung bình cộng của `Macro-F1 Intent` và `Micro-F1 Slot`[cite: 14]. Nhằm tối ưu hóa đồng thời cả hai bài toán, dự án áp dụng chiến lược **Ensemble Prediction** bằng cách chắt lọc điểm mạnh từ 2 mô hình khác biệt[cite: 25, 29]:

* **1. Mô hình Baseline (Tối ưu hóa Intent):**
  * Sử dụng thuật toán `TF-IDF` kết hợp `Logistic Regression` (OneVsRest)[cite: 15].
  * *Ưu điểm:* Khả năng phân loại Ý định (Intent) cực kỳ nhanh và chính xác, đạt F1 0.77 trên tập dev[cite: 25, 29].
  * *Nhược điểm:* Việc gán nhãn Slot sử dụng bộ phân loại tuyến tính với cửa sổ token giới hạn (chỉ nhìn 2 âm tiết mỗi bên)[cite: 15, 25]. Sự thiếu hụt ngữ cảnh toàn câu khiến Slot F1 chỉ đạt 0.55[cite: 25, 29].
* **2. Mô hình Deep Learning (Tối ưu hóa Slot):**
  * Xây dựng kiến trúc `Joint Model` với bộ mã hóa dùng chung (`XLM-Roberta-Base`) kết nối ra 2 mạng tuyến tính riêng (Linear Heads) cho Intent và Slot[cite: 27, 28].
  * *Kỹ thuật cốt lõi:* Khắc phục triệt để hiện tượng Intent không học được (loss bị lấn át) bằng cách phân hóa Learning Rate[cite: 27]. Lõi XLM-R bảo toàn kiến thức với tốc độ học chậm (`3e-5`), trong khi 2 Heads được học cấp tốc (`1e-3`)[cite: 27, 28].
  * *Kết quả:* Nhờ cơ chế Attention đọc toàn bộ ngữ cảnh câu, điểm Slot nhảy vọt lên 0.72 F1[cite: 25, 29].
* **3. Kỹ thuật Ensemble (Late Fusion):**
  * Tiến hành hợp nhất (merge) kết quả tốt nhất của cả 2 phương pháp: Cột `intent` lấy từ mô hình Baseline ghép với cột `slots` lấy từ mô hình XLM-R[cite: 29].
  * Đạt mức điểm chung cuộc 0.75 F1 mà không phát sinh thêm chi phí tính toán GPU ở bước cuối[cite: 29].

## Đặc tả Dữ liệu (Dataset)
Tập dữ liệu xây dựng từ nguồn MASSIVE 1.1 (Amazon) đã được chuẩn hóa riêng cho ngữ liệu tiếng Việt (vi-VN)[cite: 14].
* **Train / Dev set:** Gồm 14.191 câu huấn luyện và 1.275 câu đánh giá có đủ nhãn[cite: 11, 14].
* **Test set:** Gồm 2.727 câu ẩn nhãn dùng để dự đoán[cite: 11, 14].
* **Không gian nhãn:** Tổng cộng 60 ý định (Intent) và 54 loại thực thể (Slot)[cite: 11, 14].

## Cấu trúc Thư mục (Project Structure)
```text
├── data/
│   ├── train.csv, dev.csv, test.csv
│   └── intents.txt, slot_types.txt
├── 1_baseline.py               # Thuật toán TF-IDF kết hợp Logistic Regression & SVM
├── 2_xlmr_joint.py             # Kiến trúc XLM-Roberta chia 2 phân nhánh Linear Head
├── 3_ensemble.py               # Hợp nhất kết quả từ 2 mô hình (Ensemble Prediction)
├── requirements.txt
└── README.md
