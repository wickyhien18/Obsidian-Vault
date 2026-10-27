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
Điều kiện đề (chép nguyên văn): lấy các dòng có amount > 40 và status == "paid"
Bảng kiểm từng hàng (điền đúng/sai cho từng điều kiện):

| nhãn hàng | amount > 40 ? | status == "paid" ? | cả hai ? |
| --------- | ------------- | ------------------ | -------- |
| 0         | đúng          | sai                | sai      |
| 1         | đúng          | đúng               | đúng     |
| 2         | đúng          | đúng               | đúng     |
| 3         | đúng          | sai                | sai      |
| 4         | sai           | đúng               | sai      |
| 5         | đúng          | sai                | sai      |

Kết quả mong đợi (nhãn hàng):

|     | user_id | amount | status |
| --- | ------- | ------ | ------ |
| 1   | 2       | 120    | paid   |
| 2   | 1       | 80     | paid   |

```python
# Trả lời:
df[(df["amount"] > 40) & (df["status"] == "paid")]
```
Kết quả chạy (dán nguyên văn), khớp hay không khớp: kết quả khớp
   user_id  amount status
1        2     120   paid
2        1      80   paid


**0.2.** Tính **một con số**: trung bình `amount` của các dòng có `status == "new"`.
Điều kiện đề (chép nguyên văn): tính trung bình amount của các dòng status == "new" và kết quả là một con số duy nhất
Kết quả mong đợi (tính tay, làm tròn 2 chữ số): 103.33
```python
# Trả lời:
df[df["status"] == "new"]["amount"].mean()
```
Kết quả chạy, khớp hay không khớp: kết quả khớp
103.33333333333333
Đối chiếu: số này có trùng với một kết quả bạn đã tính ở Ngày 4 không? Ở câu nào? 
Số này có trùng với kết quả ở ngày 4 và trùng với kết quả phần new của câu A2 

**0.3.** Định nghĩa bằng lời của bạn: `ascending=True` là sắp xếp tăng dần hay giảm dần, và thứ tự các giá trị trông thế nào (ví dụ với 30, 50, 120)?
Trả lời: ascending nghĩa là tăng dần và việc để True là sắp xếp tăng dần và ngược lại False là giảm dần và thứ tự khi sắp xếp giá trị trong ví dụ là: 30,50,120

Dự đoán nhãn hàng của `df.sort_values("amount", ascending=True).head(2)`:

|     | user_id | amount | status |
| --- | ------- | ------ | ------ |
| 4   | 2       | 30     | paid   |
| 0   | 1       | 50     | new    |

Kết quả chạy, khớp hay không khớp: kết quả chạy khớp
   user_id  amount status
4        2      30   paid
0        1      50    new

**0.4.** Giải thích `groupby` theo **đúng 3 bước**, mỗi bước một câu, dùng ví dụ `df.groupby("user_id")["amount"].sum()`:
Bước 1 (tách): tách theo nhóm user_id
Bước 2 (tính): tính tổng amount trên từng nhóm
Bước 3 (gộp): gộp kết quả lại 

**0.5.** Chạy `df.nlargest(2, "amount")`. Kiểu dữ liệu trả về là gì (Series hay DataFrame)?
Trả lời: là DataFrame
Sửa lại câu trả lời Ngày 4 ("nlargest cho Series, sort_values cho DataFrame") nếu nó sai, và nêu tiêu chí đúng để chọn giữa `nlargest` và `sort_values`: chọn nlargest nếu muốn biết một vài hàng có cột thuộc tính ví dụ là amount là lớn nhất còn chọn sort_values nếu muốn sắp xếp dataframe 

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
Dự đoán: có 6 hàng và city của hàng có user_id = 3 là NaN
Đúng hay sai, vì sao: dự đoán đúng
   user_id  amount status       city
0        1      50    new     Hà Nội
1        2     120   paid  Hải Phòng
2        1      80   paid     Hà Nội
3        3     200    new        NaN
4        2      30   paid  Hải Phòng
5        1      60    new     Hà Nội

Lí do: merge với tham số how="left" thì sẽ giữ toàn bộ giá trị của df những chỗ giá trị chỉ df có users không có thì để là NaN ví dụ là user_id=3

**A2.** `pd.merge(df, users, on="user_id", how="inner")` có bao nhiêu hàng?
Dự đoán: 5 hàng
Đúng hay sai, vì sao: dự đoán đúng
   user_id  amount status       city
0        1      50    new     Hà Nội
1        2     120   paid  Hải Phòng
2        1      80   paid     Hà Nội
3        2      30   paid  Hải Phòng
4        1      60    new     Hà Nội

Lí do: merge với tham số how="inner" thì sẽ giữ toàn bộ giá trị của df và users đều có như user_id = 1 và 2

**A3.** `pd.merge(df, users, on="user_id", how="outer")` có bao nhiêu hàng? Hàng của `user_id` = 4 có `amount` là gì?
Dự đoán: có 7 hàng  và hàng user_id = 4 là NaN
Đúng hay sai, vì sao: dự đoán đúng
   user_id  amount status       city
0        1    50.0    new     Hà Nội
1        1    80.0   paid     Hà Nội
2        1    60.0    new     Hà Nội
3        2   120.0   paid  Hải Phòng
4        2    30.0   paid  Hải Phòng
5        3   200.0    new        NaN
6        4     NaN    NaN    Đà Nẵng

Lí do: merge với tham số how="outer" thì sẽ giữ toàn bộ giá trị của df và users kể cả bên users có df không có hay df có users không có

**A4.** Sau khi gộp kiểu `left` và gán vào `merged`, `merged["city"].isna().sum()` bằng bao nhiêu?
Dự đoán: là 1
Đúng hay sai, vì sao: dự đoán đúng bằng 1 vì chỉ có user_id = 3 là không có city bên users

**A5.** Chạy `df[df["amount"] > 100]["status"] = "big"`. Sau đó `df["status"]` có đổi không?
Dự đoán: status không đổi
Đúng hay sai, vì sao: dự đoán đúng vì df[df["amount"] > 100] chỉ tạo 1 bản sao tạm sau đó lại gán status = "big" nên đó nên df gốc không thay đổi gì

---

## Bài tập B: Viết code (điền ô điều kiện trước)

**B1.** Gán `"big"` vào cột `status` của các dòng có `amount > 100`, **đúng cách** (tránh cảnh báo).
Điều kiện đề (chép nguyên văn): gắn big vào cột status của các dòng có amount > 100 mà không có cảnh bảo
Kết quả mong đợi (tính tay, `status` của từng dòng sau khi gán):

|     | user_id | amount | status |
| --- | ------- | ------ | ------ |
| 0   | 1       | 50     | new    |
| 1   | 2       | 120    | big    |
| 2   | 1       | 80     | paid   |
| 3   | 3       | 200    | big    |
| 4   | 2       | 30     | paid   |
| 5   | 1       | 60     | new    |

```python
# Trả lời:
df.loc[df["amount"] > 100, "status"] = "big"
```
Kết quả chạy, khớp hay không khớp: kết quả khớp
   user_id  amount status
0        1      50    new
1        2     120    big
2        1      80   paid
3        3     200    big
4        2      30   paid
5        1      60    new

**B2.** Ghép `df` với `users` theo `user_id`, giữ **mọi dòng của `df`**. Gán vào biến `merged`.
Điều kiện đề (chép nguyên văn): ghép df với users theo user_id giữ mọi dòng của df sau đó gán với biến merged
Kết quả mong đợi (tính tay, số hàng và cột `city` của từng hàng):

|     | user_id | amount | status | city      |
| --- | ------- | ------ | ------ | --------- |
| 0   | 1       | 50     | new    | Hà Nội    |
| 1   | 2       | 120    | big    | Hải Phòng |
| 2   | 1       | 80     | paid   | Hà Nội    |
| 3   | 3       | 200    | big    | NaN       |
| 4   | 2       | 30     | paid   | Hải Phòng |
| 5   | 1       | 60     | new    | Hà Nội    |

```python
# Trả lời:
merged = pd.merge(df, users, on="user_id", how="left")
```
Kết quả chạy, khớp hay không khớp: kết quả khớp
   user_id  amount status       city
0        1      50    new     Hà Nội
1        2     120   paid  Hải Phòng
2        1      80   paid     Hà Nội
3        3     200    new        NaN
4        2      30   paid  Hải Phòng
5        1      60    new     Hà Nội

**B3.** Từ `merged`, lấy các dòng bị thiếu `city`.
Điều kiện đề (chép nguyên văn): từ biến merged lấy các dòng chứa giá trị NaN trong cột city
Kết quả mong đợi (nhãn hàng):

|     | user_id | amount | status | city |
| --- | ------- | ------ | ------ | ---- |
| 3   | 3       | 200    | new    | NaN  |

```python
# Trả lời:
merged[merged["city"].isna()]
```

**B4.** Điền `"Không rõ"` vào các ô `city` bị thiếu trong `merged`.
Điều kiện đề (chép nguyên văn): điền không rõ vào các ô city chứa giá trị NaN trong merged
```python
# Trả lời:
merged.loc[merged["city"].isna(), "city"] = "Không rõ"
```

**B5.** Sau B4, tính trung bình `amount` **theo từng `city`**.
Điều kiện đề (chép nguyên văn): sau khi có kết quả từ B4 tính trung bình amount theo từng city 
Kết quả mong đợi (tính tay, làm tròn 2 chữ số): 

| city      |        |
| --------- | ------ |
| Hà Nội    | 63.33  |
| Hải Phòng | 75.00  |
| Không rõ  | 200.00 |

```python
# Trả lời:
merged.groupby("city")["amount"].mean()
```
Kết quả chạy, khớp hay không khớp: kết quả khớp
city
Hà Nội        63.333333
Hải Phòng     75.000000
Không rõ     200.000000

---

## Bài tập C: Tự giải thích (không nhìn tài liệu)

**C1.** Vì sao `df[df["amount"] > 100]["status"] = "big"` không làm đổi `df`? Cách viết đúng khác ở chỗ nào?
Trả lời: vì df[df["amount"] > 100] chỉ tạo 1 bản sao tạm sau đó lại gán status = "big" nên đó nên df gốc không thay đổi gì. Cách viết đúng phải là df.loc[df["amount"] > 100, "status"] = "big"

**C2.** Khác biệt giữa `how="left"`, `"inner"`, `"outer"` là gì? Cho một ví dụ bằng dữ liệu hôm nay.
Trả lời: left là giữ toàn bộ giá trị của dataframe bên trái (tức tham số đầu tiên khi truyền vào merge), inner là giữ lại các giá trị cả 2 dataframe giao nhau, outer là giữ lại toàn bộ giá trị của cả 2 dataframe khi ghép vào. Từ dữ liệu hôm nay thì left giữ lại hàng user_id = 3 trong df trong khi bên users không có hàng nào user_id = 3 và giá trị thiếu điền bằng NaN, inner thì cả df lẫn users chỉ giữ lại các hàng user_id bằng 1 và 2 vì có sự giao nhau, outer thì ngoài giữ user_id = 3 từ df ra thì còn giữ user_id = 4 từ users

**C3.** Vì sao `groupby("city")` có thể làm "mất" một số dòng khi cột `city` có `NaN`, và cách xử lý là gì?
Trả lời: vì mặc định groupby sẽ bỏ qua hàng nào có cột chứa giá trị NaN cách xử lý là đổi giá trị NaN thành một giá trị khác ví dụ là Không có hoặc không biết, không rõ

---

## Ghi nhật ký
- Thời gian học thực tế (bắt buộc điền): 1 tiếng
- Những câu bạn đã tự kiểm tra bằng cách chạy code: câu 0.2 vì ban đầu tôi nghĩ cú pháp mean() là cho groupby sau tìm hiểu mới biết là không phải
- Điều chưa hiểu cần hỏi Claude (nếu viết "không có" thì phải nêu đã kiểm tra những câu nào): không có ví đã tự kiểm tra những câu code

> Làm xong gửi nguyên file này cho Claude chấm.
