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

