# NLP - Multi-label Intent and Slot Filling

Repository này chứa mã nguồn giải quyết bài toán đa nhiệm: Phân loại Ý định đa nhãn (Multi-label Intent) và Trích xuất Thực thể (Slot Filling) cho hệ thống trợ lý ảo tiếng Việt.

## Mô tả Bài toán
Hệ thống trợ lý ảo cần hiểu được câu lệnh tiếng Việt của người dùng (ví dụ: "gọi tôi dậy lúc chín giờ sáng ngày thứ sáu") thông qua hai nhiệm vụ thực thi song song:
1. **Intent (Ý định):** Xác định mục tiêu của người dùng muốn trợ lý làm gì; khoảng 13% số câu ghép hai yêu cầu bằng các từ nối như "và", "rồi", "sau đó", "với lại", "xong thì", "nhân tiện". Trong bài nộp, các ý định đa nhãn được phân tách bằng dấu `#` (ví dụ: `alarm_set#weather_query`).
2. **Slot (Thực thể):** Trích xuất các cụm từ mang giá trị tham số của yêu cầu (như thời gian, địa điểm, thiết bị...). Mỗi token (âm tiết) trong câu sẽ được gán một nhãn duy nhất theo tiêu chuẩn BIO (Begin - Inside - Outside). Ví dụ cụm "chín giờ sáng" được gán chuỗi `B-time I-time I-time`.

## 🎯 Nhiệm vụ

### Intent đa nhãn
Mỗi câu có một tập intent. Trong bài nộp, các intent của một câu được nối với nhau bằng dấu `#` và thứ tự không quan trọng.

| Câu | intent |
| :--- | :--- |
| `gọi tôi dậy lúc chín giờ sáng ngày thứ sáu` | `alarm_set` |
| `cài báo thức trong hai giờ kể từ bây giờ và thời tiết hôm nay thế nào` | `alarm_set#weather_query` |
| `im lặng` | `audio_volume_mute` |

Số câu của mỗi intent **chênh lệch rất nhiều**: `calendar_set` có hơn một nghìn câu train, trong khi `cooking_query` chỉ có vài câu.

### Slot filling theo nhãn BIO
Mỗi token nhận đúng một nhãn. Nhãn `B-<loại>` đánh dấu token đầu của một cụm, `I-<loại>` đánh dấu các token tiếp theo trong cùng cụm, và `0` dành cho token không thuộc cụm nào. Số nhãn của một câu phải **bằng đúng số token** của câu đó.

```text
text : gọi tôi dậy lúc chín giờ sáng ngày thứ sáu
slots: 0 0 0 0 B-time I-time I-time 0 B-date I-date
```

## Phương pháp thực hiện 
Hệ thống chấm điểm đánh giá hiệu suất thông qua trung bình cộng của `Macro-F1 Intent` và `Micro-F1 Slot`. Nhằm tối ưu hóa đồng thời cả hai bài toán, dự án áp dụng chiến lược **Ensemble Prediction** bằng cách chắt lọc điểm mạnh từ 2 mô hình khác biệt:

* **1. Mô hình Baseline:**
  * Sử dụng thuật toán `TF-IDF` kết hợp `Logistic Regression` (OneVsRest).
  * *Ưu điểm:* Khả năng phân loại Intent cực kỳ nhanh và chính xác, đạt F1 0.77 trên tập dev.
  * *Nhược điểm:* Việc gán nhãn Slot sử dụng bộ phân loại tuyến tính với cửa sổ token giới hạn (chỉ nhìn 2 âm tiết mỗi bên). Sự thiếu hụt ngữ cảnh toàn câu khiến Slot F1 chỉ đạt 0.55.
* **2. Mô hình Deep Learning (Tối ưu hóa Slot):**
  * Xây dựng kiến trúc `Joint Model` với bộ mã hóa dùng chung (`XLM-Roberta-Base`) kết nối ra 2 mạng tuyến tính riêng (Linear Heads) cho Intent và Slot.
  * *Kỹ thuật cốt lõi:* Khắc phục triệt để hiện tượng Intent không học được (loss bị lấn át) bằng cách phân hóa Learning Rate. Lõi XLM-R bảo toàn kiến thức với tốc độ học chậm (`3e-5`), trong khi 2 Heads được học cấp tốc (`1e-3`).
  * *Kết quả:* Nhờ cơ chế Attention đọc toàn bộ ngữ cảnh câu, điểm Slot nhảy vọt lên 0.72 F1.
* **3. Kỹ thuật Ensemble (Late Fusion):**
  * Tiến hành hợp nhất (merge) kết quả tốt nhất của cả 2 phương pháp: Cột `intent` lấy từ mô hình Baseline ghép với cột `slots` lấy từ mô hình XLM-R.
  * Đạt mức điểm chung cuộc 0.75 F1 mà không phát sinh thêm chi phí tính toán GPU ở bước cuối.

## Cấu trúc thư mục nlp_data_multi_intent_and_slot_filling lưu trên Google Drive

```text
├── train.csv                  # Dữ liệu huấn luyện
├── dev.csv                    # Dữ liệu đánh giá cục bộ
├── test.csv                   # Dữ liệu kiểm tra (để dự đoán nộp bài)
├── intents.txt                # Danh sách các nhãn Intent
├── slot_types.txt             # Danh sách các nhãn Slot
├── sample_submission.csv      # File mẫu định dạng nộp bài
├── scorer.py                  # Trình chấm điểm tại chỗ (Local Evaluation)
└── README.md                  # Tài liệu mô tả dự án
