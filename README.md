# 🚀 NLP Mini Contest 2026 - Joint Intent and Slot Filling

Repository này chứa mã nguồn giải quyết bài toán đa nhiệm: Phân loại Ý định đa nhãn (Multi-label Intent) và Trích xuất Thực thể (Slot Filling) cho trợ lý ảo bằng tiếng Việt.

## 📊 Đặc tả Dữ liệu (Dataset)
Tập dữ liệu của dự án được xây dựng dựa trên nguồn MASSIVE 1.1 (Amazon, giấy phép CC BY 4.0) dành riêng cho ngữ liệu tiếng Việt (vi-VN)[cite: 14]. Ban tổ chức đã xử lý câu trùng và ghép thêm câu nhiều ý định[cite: 14].

* **Train / Dev set:** Gồm 14.191 câu huấn luyện và 1.275 câu đánh giá có sẵn nhãn[cite: 14].
* **Test set:** Gồm 2.727 câu ẩn nhãn dùng để chấm điểm trên hệ thống[cite: 14].
* **Không gian nhãn:** Gồm 60 ý định (Intent) và 54 loại thực thể (Slot) được gán theo chuẩn BIO[cite: 14].
* **Đánh giá (Metric):** Điểm chung cuộc là trung bình cộng của `Macro-F1 Intent` và `Micro-F1 Slot`[cite: 14].

## 🧠 Cấu trúc Thư mục (Project Structure)
```text
├── data/
│   ├── train.csv, dev.csv, test.csv
│   └── intents.txt, slot_types.txt
├── 1_baseline.py               # Thuật toán TF-IDF kết hợp Logistic Regression & SVM
├── 2_xlmr_joint.py             # Kiến trúc XLM-Roberta chia 2 phân nhánh Linear Head
├── 3_ensemble.py               # Hợp nhất kết quả từ 2 mô hình (Ensemble Prediction)
├── requirements.txt
└── README.md
