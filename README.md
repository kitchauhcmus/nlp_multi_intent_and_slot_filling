# NLP - Joint Intent and Slot Filling

Repository này chứa mã nguồn giải quyết bài toán đa nhiệm: Phân loại Ý định đa nhãn (Multi-label Intent) và Trích xuất Thực thể (Slot Filling) cho trợ lý ảo Tiếng Việt.

## Chiến lược & Kết quả
Bài toán đánh giá dựa trên `Macro-F1 Intent` và `Micro-F1 Slot`. Để đạt hiệu suất tối ưu (0.75 F1), hệ thống áp dụng kỹ thuật **Ensemble Prediction**:
1. **Intent (Baseline):** Sử dụng `TF-IDF` kết hợp `Logistic Regression (OneVsRest)` cho tốc độ nhanh và độ chính xác Intent rất tốt (0.77 F1).
2. **Slot (XLM-R Joint Model):** Xây dựng bộ mã hóa chia sẻ (`XLM-Roberta-Base`) với 2 Linear Heads. Áp dụng kỹ thuật phân tách Learning Rate (`3e-5` cho Encoder, `1e-3` cho Heads) để chống lại sự mất cân bằng Loss. Mô hình này xuất sắc ở bài toán Slot (0.72 F1).
3. **Ensemble:** Hợp nhất cột `intent` từ Baseline và cột `slots` từ XLM-R để nộp bài.

## Cách chạy dự án
1. Cài đặt thư viện: `pip install -r requirements.txt`
2. Đặt dữ liệu vào thư mục `data/`.
3. Chạy Baseline: `python 1_baseline.py`
4. Chạy XLM-R: `python 2_xlmr_joint.py`
5. Ghép file nộp bài: `python 3_ensemble.py`
6. Upload file `sub_ghep.csv` lên hệ thống nền tảng.
