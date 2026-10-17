---
title: Buổi học Ngày 4 - Tuần 1
tags: [bai-tap, pandas, tuan-1]
---

# Buổi học Ngày 4: Sửa nợ Ngày 3 và groupby

**Khung giờ 21:00 - 23:00**
- 21:00 - 21:30: Phần 0, sửa nợ Ngày 3
- 21:30 - 21:55: đọc bài học `groupby`, chạy từng ví dụ trong Colab
- 21:55 - 22:40: bài tập A và B
- 22:40 - 22:55: bài C, không nhìn tài liệu
- 22:55 - 23:00: ghi nhật ký (**bắt buộc điền thời gian học thực tế**)

> [!warning] Luật mới: ô "Điều kiện đề" và ô "Kết quả mong đợi"
> Lỗi lặp lại nhiều nhất của bạn là không đọc kỹ điều kiện của đề (bài lập phương, nhãn hàng, đếm từ 1, "bằng pandas"). Với mỗi bài trong Phần 0 và Bài tập B, bạn **phải điền** hai ô trước khi viết code:
> - **Điều kiện đề:** chép lại các điều kiện (hình thức, khoảng giá trị, đếm từ 0 hay từ 1, hỏi nhãn hay giá trị).
> - **Kết quả mong đợi (tính tay):** tính tay kết quả trước khi chạy.
>
> Sau khi chạy, so sánh. Dán kết quả chạy **nguyên văn**, không chỉnh sửa.

Dữ liệu dùng chung cả buổi:
```python
import numpy as np
import pandas as pd
df = pd.DataFrame({
    "user_id": [1, 2, 1, 3, 2, 1],
    "amount":  [50, 120, 80, 200, 30, 60],
    "status":  ["new", "paid", "paid", "new", "paid", "new"]
})
```

---

## Phần 0: Sửa nợ Ngày 3 (bài mới)

**0.1.** NumPy, không vòng lặp, không comprehension: lũy thừa **bậc 4** của các số từ 1 đến 6.
Điều kiện đề: không vòng lặp, không comprehension, sử dụng numpy tính lũy thừa bậc 4 của các số từ 1 đến 6
Kết quả mong đợi (tính tay): [1,16,81,256,625,1296]
```python
# Trả lời:
m = np.arange(1,7) ** 4
```

**0.2.** Lấy giá trị cột `status` của dòng ở **vị trí thứ 5 (đếm từ 1)** bằng `iloc`.
Điều kiện đề: dùng iloc lấy giá trị hàng thứ 5 trong cột status
Kết quả mong đợi (tính tay): paid
```python
# Trả lời:
df.iloc[4,2]
```

**0.3.** Lấy các dòng có `amount < 100` **và** `status == "new"`.
Điều kiện đề: amout < 100 và status == "new"
Kết quả mong đợi (tính tay, liệt kê nhãn hàng): 

|     | amount | status |
| --- | ------ | ------ |
| 0   | 200    | new    |

```python
# Trả lời:
df[(df["amount"] < 100) & (df["status"] == "new")]
```

**0.4.** Tính trung bình cột `amount` bằng **một phương thức pandas, không vòng lặp**.
Điều kiện đề: sử dụng pandas không dùng vòng lặp
Kết quả mong đợi (tính tay): 90
```python
# Trả lời:
df["amount"].mean()
```

**0.5.** Liệt kê **nhãn hàng** (không phải giá trị) của kết quả `df[df["amount"] < 60]`.
Điều kiện đề: liệt kê nhãn không phải giá trị của kết quả df[df["amount"] < 60]
Dự đoán: 

|     | user_id | amount | status |
| --- | ------- | ------ | ------ |
| 0   | 1       | 50     | new    |
| 4   | 2       | 30     | paid   |

Kết quả chạy (dán nguyên văn):
   user_id  amount status
0        1      50    new
4        2      30   paid


**0.6.** Đọc đoạn này: *Trình thông dịch (interpreter) đọc và thực thi mã từng dòng khi chương trình chạy. Trình biên dịch (compiler) dịch toàn bộ mã sang mã máy trước khi chạy.* Python thường chạy bằng thông dịch. NumPy thực hiện vòng lặp bên trong mã C đã biên dịch sẵn.
Sửa lại câu bạn viết ở Ngày 3 (đã bị ngược) và giải thích trong 2 câu vì sao NumPy nhanh hơn for loop thuần Python: vì loop thuần Python sử dụng thông dịch cần vừa đọc vừa thực thi mã từng dòng, còn numpy thực hiện vòng lặp trong mã C được biên dịch toàn bộ sang mã máy trước khi chạy

**0.7.** Dự đoán số phần tử rồi giải thích vì sao khác nhau:
```python
print(df.loc[1:3, "amount"])
print(df.iloc[1:3, 1])
```
Dự đoán số phần tử của từng dòng:  print lần đầu là 3 còn print lần 2 là 2
Giải thích: loc là cắt lát cả đầu lẫn cuối tức 1, 2,3 còn iloc là chỉ phần từ đầu đến phần tử trước phần từ cuối thay vì phần tử cuối tức 1,2

**0.8.** Chạy thật câu này và **dán nguyên văn** kết quả, không chỉnh sửa:
```python
print(df.iloc[0:2])
```
Kết quả:    user_id  amount status
0        1      50    new
1        2     120   paid

Giá trị cột `status` của hàng 0 và hàng 1 là gì? là new và paid

**0.9.** Điền lại cho chính xác, dùng cụm "gộp lại":
"`m.sum(axis=0)` gộp các hàng lại, mỗi cột cho ra một tổng."
"`m.sum(axis=1)` gộp các cột lại, mỗi hàng cho ra một tổng."

---

## Bài học: groupby, sort_values, nlargest

**`groupby` làm 3 việc: tách dữ liệu thành nhóm, tính trên từng nhóm, gộp kết quả lại.**
```python
df.groupby("user_id")["amount"].sum()
# user_id
# 1    190
# 2    150
# 3    200
```
Đọc: "nhóm theo `user_id`, lấy cột `amount`, cộng từng nhóm". Kết quả là một Series có chỉ số là `user_id`.

**Hàm tổng hợp khác:** `.mean()`, `.count()`, `.max()`, `.agg(["sum", "mean", "count"])`.

**Sắp xếp và lấy lớn nhất:**
```python
s = df.groupby("user_id")["amount"].sum()
s.sort_values(ascending=False)   # sắp giảm dần
s.nlargest(2)                    # 2 giá trị lớn nhất
```

**Cẩn thận với `&`:** khi kết hợp điều kiện phải bọc **từng điều kiện trong ngoặc đơn**, vì `&` được tính trước `>` và ' == '.
```python
df[(df["amount"] > 50) & (df["status"] == "paid")]   # đúng
df[df["amount"] > 50 & df["status"] == "paid"]       # lỗi
```

---

## Bài tập A: Dự đoán đầu ra

**A1.** `df.groupby("user_id")["amount"].sum()`
Dự đoán (3 giá trị):

| user_id |     |
| ------- | --- |
| 1       | 190 |
| 2       | 150 |
| 3       | 200 |

Đúng hay sai, vì sao: dự đoán đúng vì nhóm theo user_id lấy cột amount cộng lại tương ứng với giá trị user_id nhóm lại

**A2.** `df.groupby("status")["amount"].mean()`
Dự đoán (làm tròn 2 chữ số): 

| status |        |
| ------ | ------ |
| new    | 103.33 |
| paid   | 76.67  |

Đúng hay sai, vì sao: dự đoán đúng vì nhóm theo status lấy cột amount tính trung bình

**A3.** `df.sort_values("amount", ascending=False).head(2)` (liệt kê **nhãn hàng**)
Dự đoán:

|     | user_id | amount | status |
| --- | ------- | ------ | ------ |
| 3   | 3       | 200    | new    |
| 1   | 2       | 120    | paid   |


Đúng hay sai, vì sao: dự đoán đúng vì đây là kết quả sau khi sắp xếp tăng dần và chỉ lấy 2 phần tử đầu

**A4.** `df["amount"].nlargest(2)` (liệt kê nhãn hàng và giá trị)
Dự đoán: 

| 3   | 200 |
| --- | --- |
| 1   | 120 |


Đúng hay sai, vì sao: dự đóan đúng vì lấy 2 giá trị lớn nhất trong cột amount
Khác nhau gì về kiểu dữ liệu trả về giữa A3 và A4? về kiểu dữ liệu thì tại A3 kiểu dữ liệu là DataFrame còn A4 thì là Series

**A5.** `df[df["amount"] > 50 & df["status"] == "paid"]` chạy được hay báo lỗi? Vì sao?
Dự đoán: báo lỗi
Đúng hay sai, vì sao: dự đoán đùng vì thiếu dấu ngoặc cho điều kiên

---

## Bài tập B: Viết code (điền ô điều kiện trước)

**B1.** Tổng `amount` theo từng `user_id`.
Điều kiện đề: tính tổng amount theo từng user_id
Kết quả mong đợi (tính tay):

| user_id |     |
| ------- | --- |
| 1       | 190 |
| 2       | 150 |
| 3       | 200 |

```python
# Trả lời:
df.groupby("user_id")["amount"].sum()
```

**B2.** Tổng `amount` theo từng `user_id`, sắp xếp giảm dần.
Điều kiện đề: tính tổng amount theo từng user_id và cần sắp xếp tăng dần
Kết quả mong đợi (tính tay):

| user_id |     |
| ------- | --- |
| 3       | 200 |
| 1       | 190 |
| 2       | 150 |

```python
# Trả lời:
df.groupby("user_id")["amount"].sum().sort_values(ascending=False)
```

**B3.** Lấy 2 user có tổng `amount` lớn nhất. (Đây là câu A3 của bài chẩn đoán Tuần 0.)
Điều kiện đề: lấy 2 user có tổng amount lớn nhất
Kết quả mong đợi (tính tay): 

| user_id |     |
| ------- | --- |
| 3       | 200 |
| 1       | 190 |

```python
# Trả lời:
df.groupby("user_id")["amount"].sum().sort_values(ascending=False).head(2)
```

**B4.** Đếm số đơn theo từng `status`.
Điều kiện đề: đếm số đơn theo từng status
Kết quả mong đợi (tính tay):

| status |     |
| ------ | --- |
| new    | 3   |
| paid   | 3   |

```python
# Trả lời:
df.groupby("status")["user_id"].count()
```

**B5.** Trung bình `amount` của các dòng có `status == "paid"`.
Điều kiện đề: trung bình amount của các dòng có status là paid
Kết quả mong đợi (tính tay, làm tròn 2 chữ số):

| user_id |     |
| ------- | --- |
| 1       | 80  |
| 2       | 75  |

```python
# Trả lời:
a = df[df["status"] == "paid"]

a.groupby("user_id")["amount"].mean()
```

---

## Bài tập C: Tự giải thích (không nhìn tài liệu)

**C1.** `loc` và `iloc` khác nhau ở đâu? Phải nêu **cả** cách xác định vị trí **và** khác biệt khi cắt lát.
Trả lời:loc xác định vị trí bằng tên cột và khi cắt là cắt từ điểm đầu đến điểm cuối ví dụ [0:3] thì cắt từ phần từ 0 đến phần tử 3 còn iloc thì xác định vị trí bằng số thứ tự của cột  và khi cắt thì cắt từ điểm đầu đến điêm trước điểm cuối ví dụ [0:3] thì cắt từ phần từ 0 đến phần từ 2

**C2.** `groupby` thực hiện 3 bước nào? Giải thích bằng ví dụ `df.groupby("user_id")["amount"].sum()`.
Trả lời: grouptby thực hiện bắt đầu nhóm sau đó lấy một cột để thực hiện các thao tác lấy ví dụ trên thì nhóm theo user_id sau đó lấy amount để cộng lại 

**C3.** Khi nào dùng `nlargest`, khi nào dùng `sort_values`?
Trả lời: nếu muốn ra kết quả Series thì dùng nlargest còn khi muốn kết quả Dataframe thì dùng sort_values

---

## Ghi nhật ký
- Thời gian học thực tế (bắt buộc điền): 30 phút
- Câu làm chậm nhất, vì sao: câu B3 và B5 vì cần phải thử nghiệm trước kèm tìm hiểu trên mạng
- Điều chưa hiểu cần hỏi Claude (nếu thực sự không có thì viết "không có" và nói đã kiểm tra những câu nào): không có

> Làm xong gửi nguyên file này cho Claude chấm.
