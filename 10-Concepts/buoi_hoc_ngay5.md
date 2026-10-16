---
title: Buổi học Ngày 5 - Tuần 1
tags: [bai-tap, pandas, tuan-1]
---

# Buổi học Ngày 5: Sửa nợ Ngày 4, chained assignment, merge, giá trị thiếu

**Khung giờ 21:00 - 23:00**
- 21:00 - 21:25: Phần 0, sửa nợ Ngày 4
- 21:25 - 21:55: đọc bài học, chạy từng ví dụ trong Colab
- 21:55 - 22:40: bài tập A và B
- 22:40 - 22:55: bài C, không nhìn tài liệu
- 22:55 - 23:00: ghi nhật ký (điền thời gian học **thực tế** và liệt kê **những câu bạn đã tự kiểm tra**)

> [!warning] Giữ nguyên luật ô "Điều kiện đề" và "Kết quả mong đợi"
> - **Điều kiện đề:** chép **nguyên văn** từ đề, đặc biệt chữ *tăng dần / giảm dần*, *đếm từ 0 hay 1*, *nhãn hay giá trị*, *một con số hay một nhóm số*.
> - **Kết quả mong đợi:** tính tay. Sau khi chạy, ghi rõ **khớp hay không khớp**. Không khớp thì tìm xem sai ở đâu: tính tay hay code.
> - Dán kết quả chạy nguyên văn.

Dữ liệu dùng chung cả buổi:
```python
import numpy as np
import pandas as pd
df = pd.DataFrame({
    "user_id": [1, 2, 1, 3, 2, 1],
    "amount":  [50, 120, 80, 200, 30, 60],
    "status":  ["new", "paid", "paid", "new", "paid", "new"]
})
users = pd.DataFrame({
    "user_id": [1, 2, 4],
    "city":    ["Hà Nội", "Hải Phòng", "Đà Nẵng"]
})
```

---

## Phần 0: Sửa nợ Ngày 4 (bài mới)

**0.1.** Lấy các dòng có `amount > 40` **và** `status == "paid"`.
Điều kiện đề (chép nguyên văn):
Bảng kiểm từng hàng (điền đúng/sai cho từng điều kiện):

| nhãn hàng | amount > 40 ? | status == "paid" ? | cả hai ? |
|---|---|---|---|
| 0 | | | |
| 1 | | | |
| 2 | | | |
| 3 | | | |
| 4 | | | |
| 5 | | | |

Kết quả mong đợi (nhãn hàng):
```python
# Trả lời:

```
Kết quả chạy (dán nguyên văn), khớp hay không khớp:

**0.2.** Tính **một con số**: trung bình `amount` của các dòng có `status == "new"`.
Điều kiện đề (chép nguyên văn):
Kết quả mong đợi (tính tay, làm tròn 2 chữ số):
```python
# Trả lời:

```
Kết quả chạy, khớp hay không khớp:
Đối chiếu: số này có trùng với một kết quả bạn đã tính ở Ngày 4 không? Ở câu nào?

**0.3.** Định nghĩa bằng lời của bạn: `ascending=True` là sắp xếp tăng dần hay giảm dần, và thứ tự các giá trị trông thế nào (ví dụ với 30, 50, 120)?
Trả lời:

Dự đoán nhãn hàng của `df.sort_values("amount", ascending=True).head(2)`:
Kết quả chạy, khớp hay không khớp:

**0.4.** Giải thích `groupby` theo **đúng 3 bước**, mỗi bước một câu, dùng ví dụ `df.groupby("user_id")["amount"].sum()`:
Bước 1 (tách):
Bước 2 (tính):
Bước 3 (gộp):

**0.5.** Chạy `df.nlargest(2, "amount")`. Kiểu dữ liệu trả về là gì (Series hay DataFrame)?
Trả lời:
Sửa lại câu trả lời Ngày 4 ("nlargest cho Series, sort_values cho DataFrame") nếu nó sai, và nêu tiêu chí đúng để chọn giữa `nlargest` và `sort_values`:

---

## Bài học

### 1. Chained assignment (đây là câu A4 của bài chẩn đoán Tuần 0)
```python
df[df["amount"] > 100]["status"] = "big"      # thường gây SettingWithCopyWarning
```
Vấn đề: `df[df["amount"] > 100]` tạo ra một **bản sao tạm**, rồi gán `["status"] = "big"` lên **bản sao đó**. `df` gốc không đổi và pandas cảnh báo.

Cách đúng:
```python
df.loc[df["amount"] > 100, "status"] = "big"   # lọc và gán trong MỘT lệnh loc
```
Khi cần tạo bản sao độc lập có chủ đích: `df2 = df[df["amount"] > 100].copy()`.

### 2. merge (ghép bảng theo khóa chung)
```python
pd.merge(df, users, on="user_id", how="left")    # giữ mọi dòng của df
pd.merge(df, users, on="user_id", how="inner")   # chỉ giữ dòng khớp ở cả hai bảng
pd.merge(df, users, on="user_id", how="outer")   # giữ tất cả dòng của cả hai bảng
```
Ô nào không có dữ liệu tương ứng thì thành `NaN` (giá trị thiếu).

### 3. Giá trị thiếu
```python
s.isna()           # True ở ô thiếu
s.isna().sum()     # đếm số ô thiếu
s.fillna("Không rõ")   # điền giá trị
s.dropna()             # bỏ ô thiếu
```
Lưu ý: `groupby` mặc định **bỏ qua** nhóm có khóa là `NaN`.

---

## Bài tập A: Dự đoán đầu ra

**A1.** `pd.merge(df, users, on="user_id", how="left")` có bao nhiêu hàng? `city` của hàng có `user_id` = 3 là gì?
Dự đoán:
Đúng hay sai, vì sao:

**A2.** `pd.merge(df, users, on="user_id", how="inner")` có bao nhiêu hàng?
Dự đoán:
Đúng hay sai, vì sao:

**A3.** `pd.merge(df, users, on="user_id", how="outer")` có bao nhiêu hàng? Hàng của `user_id` = 4 có `amount` là gì?
Dự đoán:
Đúng hay sai, vì sao:

**A4.** Sau khi gộp kiểu `left` và gán vào `merged`, `merged["city"].isna().sum()` bằng bao nhiêu?
Dự đoán:
Đúng hay sai, vì sao:

**A5.** Chạy `df[df["amount"] > 100]["status"] = "big"`. Sau đó `df["status"]` có đổi không?
Dự đoán:
Đúng hay sai, vì sao:

---

## Bài tập B: Viết code (điền ô điều kiện trước)

**B1.** Gán `"big"` vào cột `status` của các dòng có `amount > 100`, **đúng cách** (tránh cảnh báo).
Điều kiện đề (chép nguyên văn):
Kết quả mong đợi (tính tay, `status` của từng dòng sau khi gán):
```python
# Trả lời:

```
Kết quả chạy, khớp hay không khớp:

**B2.** Ghép `df` với `users` theo `user_id`, giữ **mọi dòng của `df`**. Gán vào biến `merged`.
Điều kiện đề (chép nguyên văn):
Kết quả mong đợi (tính tay, số hàng và cột `city` của từng hàng):
```python
# Trả lời:

```
Kết quả chạy, khớp hay không khớp:

**B3.** Từ `merged`, lấy các dòng bị thiếu `city`.
Điều kiện đề (chép nguyên văn):
Kết quả mong đợi (nhãn hàng):
```python
# Trả lời:

```

**B4.** Điền `"Không rõ"` vào các ô `city` bị thiếu trong `merged`.
Điều kiện đề (chép nguyên văn):
```python
# Trả lời:

```

**B5.** Sau B4, tính trung bình `amount` **theo từng `city`**.
Điều kiện đề (chép nguyên văn):
Kết quả mong đợi (tính tay, làm tròn 2 chữ số):
```python
# Trả lời:

```
Kết quả chạy, khớp hay không khớp:

---

## Bài tập C: Tự giải thích (không nhìn tài liệu)

**C1.** Vì sao `df[df["amount"] > 100]["status"] = "big"` không làm đổi `df`? Cách viết đúng khác ở chỗ nào?
Trả lời:

**C2.** Khác biệt giữa `how="left"`, `"inner"`, `"outer"` là gì? Cho một ví dụ bằng dữ liệu hôm nay.
Trả lời:

**C3.** Vì sao `groupby("city")` có thể làm "mất" một số dòng khi cột `city` có `NaN`, và cách xử lý là gì?
Trả lời:

---

## Ghi nhật ký
- Thời gian học thực tế (bắt buộc điền): 1 tiếng
- Những câu bạn đã tự kiểm tra bằng cách chạy code:
- Điều chưa hiểu cần hỏi Claude (nếu viết "không có" thì phải nêu đã kiểm tra những câu nào):

> Làm xong gửi nguyên file này cho Claude chấm.
