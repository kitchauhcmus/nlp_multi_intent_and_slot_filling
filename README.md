# NLP - Multi-label Intent and Slot Filling

Repository này chứa mã nguồn giải quyết bài toán đa nhiệm: Phân loại Ý định đa nhãn (Multi-label Intent) và Trích xuất Thực thể (Slot Filling) cho hệ thống trợ lý ảo tiếng Việt.

## Tổng quan

Một trợ lý ảo nhận **câu lệnh tiếng Việt** của người dùng, ví dụ "gọi tôi dậy lúc chín giờ sáng ngày thứ sáu", và phải hiểu hai điều cùng lúc:

1. **Ý định (intent)**: người dùng muốn trợ lý làm gì. Một câu có thể chứa **một hoặc nhiều** ý định; khoảng 13 % số câu ghép hai yêu cầu bằng các từ nối như "và", "rồi", "sau đó", "với lại", "xong thì", "nhân tiện".
2. **Thông tin cần điền (slot)**: các cụm từ mang giá trị tham số của yêu cầu, như thời gian, địa điểm, tên người hay tên bài hát. Mỗi token của câu được gán một nhãn theo sơ đồ **BIO**.

Dữ liệu của bài gồm:

* 14.191 câu train và 1.275 câu dev có đủ nhãn intent và slot.
* 2.727 câu test chỉ có văn bản; nhãn của tập test được ẩn.
* 60 intent thuộc 18 nhóm chức năng (báo thức, lịch, thời tiết, nhạc, email, nhà thông minh, tin tức, giao thông...) và 54 loại slot.
* Văn bản đã được chuyển về chữ thường. Mỗi token là một **âm tiết** và các token cách nhau đúng một dấu cách.

Với mỗi câu test, thí sinh nộp tập intent và chuỗi nhãn BIO của câu đó. Điểm của bài nộp là trung bình cộng của điểm intent và điểm slot, nên hai phần quan trọng ngang nhau.

## Nhiệm vụ

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
Với câu ghép hai yêu cầu, từ nối mang nhãn `0` và các cụm slot của cả hai vế được giữ nguyên:

```text
text : cài báo thức trong hai giờ kể từ bây giờ và thời tiết hôm nay thế nào
slots: 0 0 0 0 B-time I-time I-time I-time I-time I-time 0 0 0 B-date I-date 0 0
python3 scorer.py du_doan_dev.csv --gt dev.csv           # in Score, IntentF1, SlotF1
python3 scorer.py du_doan_dev.csv --gt dev.csv --per-class # in thêm F1 của từng intent
```

## Dữ liệu

### `train.csv`, `dev.csv`

| Cột | Ý nghĩa |
| :--- | :--- |
| `id` | Mã câu duy nhất (`u00001` ...) |
| `text` | Câu lệnh, chữ thường, các token cách nhau bằng một dấu cách |
| `intent` | Một hoặc nhiều intent, nối với nhau bằng `#` |
| `slots` | Nhãn BIO của từng token, cách nhau bằng dấu cách |

`dev.csv` có cùng định dạng với `train.csv` và dùng để tự đánh giá. Thí sinh được phép gộp dev vào train.

### `test.csv`

| Cột | Ý nghĩa |
| :--- | :--- |
| `id` | Mã câu cần dự đoán |
| `text` | Câu lệnh |

### `sample_submission.csv`

Tệp mẫu này đúng định dạng bài nộp: mọi câu được gán intent `calendar_set` và mọi token được gán nhãn `O`. Tệp hợp lệ nhưng chỉ được khoảng 0 điểm.

### `intents.txt`, `slot_types.txt`

Danh sách 60 intent và 54 loại slot xuất hiện trong train.

### `scorer.py`

Trình chấm chạy trên máy của thí sinh và cho cùng kết quả với trình chấm của nền tảng:

```bash
python3 scorer.py du_doan_dev.csv --gt dev.csv           # in Score, IntentF1, SlotF1
python3 scorer.py du_doan_dev.csv --gt dev.csv --per-class # in thêm F1 của từng intent
```
## Định dạng bài nộp

Tệp CSV mã hoá UTF-8, có dòng tiêu đề, gồm đúng 2.727 dòng dữ liệu (mỗi `id` của `test.csv` một dòng) với ba cột:

```csv
id,intent,slots
u90001,alarm_set,0 0 0 0 B-time I-time I-time 0 B-date I-date
u90002,alarm_set#weather_query,0 0 0 0 B-time I-time I-time I-time I-time I-time 0 0 0 B-date I-date 0 0
```
Hai dòng trên dùng lại hai câu ví dụ ở tab Đề bài với `id` minh hoạ; bài nộp thật dùng `id` của `test.csv`. Số nhãn ở cột `slots` phải bằng số token của câu có cùng `id` (10 và 17 token ở hai dòng trên).

Để bài nộp hợp lệ:

* Bài nộp phải có đủ mọi `id` của `test.csv`. Dòng nào thiếu thì nhận 0 điểm ở cả hai phần.
* Cột `intent` có ít nhất một nhãn; nhiều nhãn thì nối bằng `#`. Nhãn lặp lại chỉ tính một lần. Nhãn không có trong `intents.txt` được tính là dự đoán sai.
* Cột `slots` có số nhãn bằng số token của `text`. Nếu số nhãn lệch, câu đó được coi là **không dự đoán thực thể nào**: câu đó mất toàn bộ điểm slot, các câu khác vẫn được chấm bình thường.
* Bài nộp sai định dạng không làm hỏng việc chấm; ô lỗi chỉ nhận 0 điểm.

## Phương pháp đánh giá

Điểm của bài nộp là trung bình cộng của hai thành phần:

```text
Score = 0.5 × IntentF1 + 0.5 × SlotF1
```
### IntentF1: macro-F1 đa nhãn

Với mỗi intent, đếm trên toàn bộ tập được chấm số câu dự đoán đúng intent đó (TP), số câu dự đoán thừa (FP) và số câu bỏ sót (FN), rồi tính F1 của intent đó[cite: 29]. IntentF1 là **trung bình cộng F1 của mọi intent xuất hiện trong đáp án hoặc trong dự đoán** (giống `sklearn.metrics.f1_score(average="macro")` )[cite: 29]. Vì vậy một intent hiếm có trọng số ngang một intent phổ biến, và một nhãn chỉ xuất hiện trong dự đoán cũng được tính vào trung bình với F1 bằng 0[cite: 29].

### SlotF1: F1 mức thực thể

SlotF1 là micro-F1 mức thực thể theo chuẩn conlleval / seqeval[cite: 29]. Một thực thể dự đoán được tính là đúng khi trùng cả **loại lẫn vị trí đầu và cuối** với một thực thể trong đáp án[cite: 29]. Nhãn `I-` không nối tiếp một nhãn cùng loại được coi là bắt đầu một thực thể mới (luật conlleval)[cite: 29].

Ví dụ với câu `gọi tôi dậy lúc chín giờ sáng ngày thứ sáu` , đáp án có hai thực thể: `time` = "chín giờ sáng" và `date` = "thứ sáu"[cite: 29].

| Dự đoán | Đúng | Thừa | Sót |
| :--- | :--- | :--- | :--- |
| `O O O O B-time I-time I-time O B-date I-date` | 2 | 0 | 0 |
| `O O O O B-time I-time O O B-date I-date` | 1 ( `date` ) | 1 ( `time` sai ranh giới) | 1 |
| `O O O O B-time I-time I-time O B-time I-time` | 1 | 1 | 1 |

### Bảng xếp hạng và điểm cuối cùng
Vì đây là cuộc thi thử để làm quen, đề không có private test. Bảng xếp hạng chấm mỗi bài nộp trên toàn bộ tập test (2.727 câu), và điểm cuối cùng của bài là điểm cao nhất của đội trong các lần nộp được tính. Điểm tối đa là 1,0.

## Cấu trúc thư mục dữ liệu

```text
public/
|-- train.csv                 (14.191 câu có nhãn)
|-- dev.csv                   (1.275 câu có nhãn)
|-- test.csv                  (2.727 câu cần dự đoán)
|-- sample_submission.csv
|-- intents.txt               (60 intent)
|-- slot_types.txt            (54 loại slot)
|-- scorer.py                 (trình chấm chạy tại chỗ)
`-- README.md
```
## Phương pháp thực hiện

Hệ thống được thiết kế theo hướng tiếp cận độc lập, giải quyết song song hai bài toán Intent Classification và Slot Filling.

### 1. Mô hình Baseline (TF-IDF & Linear Model)

**a. Phân loại Ý định (Intent Classification)**
Bài toán được tiếp cận dưới dạng **Multi-label Classification** để xử lý các câu lệnh chứa từ 2 ý định trở lên.
* **Trích xuất đặc trưng:** Kết hợp sức mạnh của hai bộ TF-IDF Vectorizer:
  * *Word n-grams (1-2):* Bắt các cụm từ khóa có ý nghĩa ngữ nghĩa trực tiếp.
  * *Character n-grams (1-4, chế độ char_wb):* Bắt các đặc trưng hình thái từ (subwords), giúp mô hình có sức chịu đựng tốt hơn với lỗi chính tả hoặc từ dính liền.
  * Sử dụng tham số `sublinear_tf=True` (áp dụng công thức $1 + \ln(TF)$) để giảm bớt sự thống trị của các từ khóa xuất hiện với tần suất quá dày đặc.
* **Mô hình học máy:** Sử dụng `LogisticRegression` kết hợp với `OneVsRestClassifier`. Solver `liblinear` được chọn để xử lý cực nhanh các ma trận thưa (sparse matrix) nhiều chiều.
* **Luật dự đoán (Thresholding & Fallback):** Lựa chọn tất cả các nhãn có xác suất $\ge 0.5$. Trong trường hợp ngoại lệ khi mô hình phân vân (không có nhãn nào đạt ngưỡng), hệ thống tự động fallback chọn nhãn có xác suất cao nhất (`argmax`) để đảm bảo tính hợp lệ của bài nộp.

**b. Nhận diện Thực thể (Slot Filling)**
Bài toán Sequence Labeling được đơn giản hóa thành bài toán phân loại đa lớp ở mức độ từng token (Token-level Classification).
* **Kỹ thuật Cửa sổ trượt (Sliding Window):** Hàm `tok_feats` được thiết kế để trượt qua từng token trong câu, thu thập ngữ cảnh cục bộ làm đặc trưng:
  * Ngữ cảnh không gian: Lấy 2 token phía trước (`w-1`, `w-2`) và 2 token phía sau (`w+1`, `w+2`).
  * Ngữ cảnh chuỗi (Bigrams): Ghép cặp token liền kề (`w-1|w`, `w|w+1`).
  * Đặc trưng hình thái học: Xác định token có chứa chữ số hay không (`digit`).
* **Mô hình học máy:** Sử dụng `DictVectorizer` để biến đổi các từ điển đặc trưng thành ma trận số. Sau đó, huấn luyện bằng `SGDClassifier` với hàm mất mát `hinge` (bản chất là một Linear SVM). Thuật toán tối ưu dốc ngẫu nhiên (SGD) giúp mô hình hội tụ cực nhanh trên tập dữ liệu hàng trăm nghìn token.
* **Khôi phục chuỗi:** Phân loại toàn bộ token của tập test trong một mảng phẳng 1 chiều, sau đó dùng con trỏ `idx` cắt tuần tự theo đúng số lượng token của từng câu ban đầu để ghép lại chuỗi nhãn BIO chuẩn xác.
