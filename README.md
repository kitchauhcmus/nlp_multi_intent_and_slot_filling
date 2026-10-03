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

Với mỗi intent, đếm trên toàn bộ tập được chấm số câu dự đoán đúng intent đó (TP), số câu dự đoán thừa (FP) và số câu bỏ sót (FN), rồi tính F1 của intent đó. IntentF1 là **trung bình cộng F1 của mọi intent xuất hiện trong đáp án hoặc trong dự đoán** (giống `sklearn.metrics.f1_score(average="macro")` ). Vì vậy một intent hiếm có trọng số ngang một intent phổ biến, và một nhãn chỉ xuất hiện trong dự đoán cũng được tính vào trung bình với F1 bằng 0.

### SlotF1: F1 mức thực thể

SlotF1 là micro-F1 mức thực thể theo chuẩn conlleval / seqeval. Một thực thể dự đoán được tính là đúng khi trùng cả **loại lẫn vị trí đầu và cuối** với một thực thể trong đáp án[cite: 29]. Nhãn `I-` không nối tiếp một nhãn cùng loại được coi là bắt đầu một thực thể mới (luật conlleval).

Ví dụ với câu `gọi tôi dậy lúc chín giờ sáng ngày thứ sáu` , đáp án có hai thực thể: `time` = "chín giờ sáng" và `date` = "thứ sáu".

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

**a. Phân loại Ý định (Intent Classification)**

* **Bước 1: Trích xuất đặc trưng**
  Hệ thống kết hợp hai bộ TF-IDF Vectorizer:
  * `v_word` (Word n-grams 1-2): Trích các cụm từ khóa gồm 1-2 từ.
  * `v_char` (Character n-grams 1-4, chế độ `char_wb`): Trích xuất các đặc trưng hình thái từ (subwords) gồm 1-4 kí tự. Cài đặt `min_df=2` giúp loại bỏ nhiễu bằng cách chỉ giữ lại các n-gram xuất hiện trong ít nhất 2 câu lệnh khác nhau.
  * Khi ghép lại (`hstack`), mỗi câu train sẽ được biểu diễn thành một vector $X$ thưa (sparse vector) có số chiều bằng tổng số lượng đặc trưng từ vựng thu thập được từ toàn bộ dữ liệu huấn luyện (lên tới hàng chục ngàn chiều).
  * *Lưu ý:* Tham số `sublinear_tf=True` được sử dụng để áp dụng công thức $TF = 1 + \ln(tf)$, kết hợp với công thức mặc định $IDF = \ln(\frac{1+N}{1+df}) + 1$. Việc tính toán TF-IDF theo cách này giúp cân bằng độ quan trọng của đặc trưng và giảm bớt trọng số quá lớn của các từ khóa xuất hiện với tần suất quá dày đặc.

* **Bước 2: Xây dựng nhãn (Vector Y)**
  Tập dữ liệu có tổng cộng 60 intent. Với mỗi câu, intent nào xuất hiện thì đánh số 1, không có thì đánh số 0 (sử dụng `MultiLabelBinarizer`). Lúc này, nhãn của mỗi câu sẽ được mã hóa thành một vector $Y$ gồm đúng 60 con số.

* **Bước 3: Huấn luyện 60 mô hình Logistic Regression**
  Sử dụng chiến lược `OneVsRestClassifier`, hệ thống tạo ra 60 mô hình Logistic Regression độc lập, mỗi mô hình chuyên nhận diện 1 intent cụ thể. Quá trình train diễn ra bằng cách cho các vector $X$ và vector $Y$ chạy qua từng mô hình. Hệ thống sử dụng thuật toán `liblinear` kết hợp siêu tham số nghịch đảo chuẩn hóa `C=10` để tối ưu hóa việc phân tách các ma trận thưa nhiều chiều này một cách triệt để và nhanh chóng.

* **Bước 4: Thresholding & Fallback (Ngưỡng dự đoán & Kế hoạch dự phòng)**
  Khi có một câu test mới, câu đó sẽ được vector hóa thành vector $X$ và đi qua toàn bộ 60 mô hình trên. Mô hình nào dự đoán xác suất $\ge 0.5$ thì hệ thống sẽ lấy nhãn đó. Trong trường hợp hiếm hoi mô hình "phân vân" không có nhãn nào đạt ngưỡng, hệ thống tự động fallback lấy duy nhất nhãn có xác suất cao nhất (`argmax`) để đảm bảo bài nộp luôn hợp lệ.

**b. Nhận diện Thực thể (Slot Filling)**

Bài toán gán nhãn chuỗi (Sequence Labeling) được giải quyết bằng cách chẻ nhỏ toàn bộ câu thành các âm tiết độc lập, đưa về dạng phân loại đa lớp (Token-level Classification).

* **Bước 1: Trích xuất đặc trưng (Kỹ thuật Cửa sổ trượt - Sliding Window)**
  Hàm `tok_feats` trượt một "cửa sổ" qua từng token trong câu nhằm thu thập ngữ cảnh cục bộ. Giả sử với câu: *"gọi tôi dậy lúc chín giờ sáng"*, khi vòng lặp chạy đến từ **"chín"** (vị trí `i = 4`), hàm sẽ soi xung quanh và tạo ra một Dictionary (từ điển) đặc trưng:
  * `w(0)` (Từ hiện tại): `"chín"`
  * `w-1`, `w-2` (Bên trái 1-2 bước): `"lúc"`, `"dậy"`
  * `w+1`, `w+2` (Bên phải 1-2 bước): `"giờ"`, `"sáng"`
  * `w-1|w` (Cụm 2 từ trái): `"lúc|chín"`
  * `w|w+1` (Cụm 2 từ phải): `"chín|giờ"`
  * `digit`: `False` (Kiểm tra xem "chín" có phải là con số toán học không).
  *(Lưu ý: Nếu từ nằm ở đầu hoặc cuối câu, các vị trí bị hụt sẽ được tự động điền chuỗi rỗng `""` để lấp chỗ trống).*

* **Bước 2: Chuẩn bị Dữ liệu (`feats` và `tags`)**
  * `feats`: Hai vòng lặp `for` lồng nhau sẽ duyệt qua toàn bộ 14.191 câu Train ra thành từng âm tiết độc lập. Mỗi âm tiết biến thành một Dictionary đặc trưng như ví dụ trên. Kết quả là một mảng `feats_train` chứa hàng trăm ngàn phần tử.
  * `tags`: Tương tự, mảng nhãn BIO cũng được cắt rời. Âm tiết "chín" có nhãn là `B-time`, thì nó được thêm vào mảng `tags_train`. (Độ dài của `tags` bắt buộc phải khớp tuyệt đối với `feats`).

* **Bước 3: Vector hóa (`DictVectorizer`)**
  Máy học không đọc được các Dictionary chữ. Hàm `dv = DictVectorizer()` sẽ quét qua hàng trăm ngàn Dictionary kia, gom tất cả các giá trị độc nhất lại để tạo ra một ma trận siêu thưa (sparse matrix). Nó hoạt động tương tự như việc tạo ra tờ phiếu checklist đặc trưng cho từng âm tiết độc lập.

* **Bước 4: Huấn luyện SVM (`SGDClassifier`)**
  * **SGD (Stochastic Gradient Descent):** Là thuật toán tối ưu hóa siêu tốc, cực kỳ phù hợp cho dữ liệu lớn.
  * **loss="hinge":** Chính tham số này đã biến thuật toán SGD thành một mô hình **Linear Support Vector Machine (SVM)** đa lớp. SVM sẽ cố gắng vẽ ra các siêu phẳng để phân chia hàng trăm ngàn vector đặc trưng kia vào đúng 54 nhóm slot khác nhau.
  * **Khôi phục chuỗi:** Kết quả dự đoán `preds_slot` trả về là một mảng 1 chiều. Hệ thống sẽ dùng một con trỏ `idx` cắt tuần tự theo đúng số lượng token của từng câu test ban đầu để ghép lại thành chuỗi nhãn hoàn chỉnh.
