---
title: Buổi học Ngày 3 - Tuần 1
tags: [bai-tap, pandas, tuan-1]
---

# Buổi học Ngày 3: Sửa nợ Ngày 2 và pandas cơ bản

**Khung giờ 21:00 - 23:00**
- 21:00 - 21:20: Phần 0, sửa nợ Ngày 2
- 21:20 - 21:55: đọc bài học pandas, chạy từng ví dụ trong Colab
- 21:55 - 22:40: bài tập A và B
- 22:40 - 22:55: bài C, không nhìn tài liệu
- 22:55 - 23:00: ghi nhật ký

> [!note] Ba luật bắt buộc (rút ra từ Ngày 1-2)
> 1. **Gạch chân điều kiện của đề** trước khi viết code: hình thức (một dòng, không vòng lặp, dùng thư viện nào), khoảng giá trị, điều kiện lọc.
> 2. **Tự kiểm tra bằng ví dụ nhỏ**: tính tay kết quả mong đợi trước, rồi so với kết quả chạy.
> 3. **Dự đoán sai thì ghi chữ "SAI"** và nói vì sao sai. Không chỉ dán kết quả thật.

---

## Phần 0: Sửa nợ Ngày 2 (bài mới, không phải bài cũ)

**0.1.** Một dòng NumPy, **không vòng lặp, không comprehension**: lập phương các số từ 1 đến 10.
```python
# Trả lời:
x = np.arange(1,11) ** 2
```

**0.2.** Cho `m = np.array([[2, 4, 6], [8, 10, 12]])`.
Tính tay trung bình từng cột: [5,7,9]
Viết code NumPy tính trung bình từng cột:
```python
# Trả lời:
m = np.array([[2, 4, 6], [8, 10, 12]])

mean = m.sum(axis=0) / m.shape[0]
```
Kết quả chạy khớp kết quả tính tay không? kết quả có khớp
Giải thích vì sao `m.sum(axis=0) / m.shape[1]` cho kết quả sai: m.shape[1] ra kết quả là số cột nhưng khi tính trung bình từng cột thì lấy giá trị của các cột ứng với các hàng tức nên chưa cho m.shape[0] là số lượng hàng thay vì m.shape[1]

**0.3.** Dùng NumPy, **không vòng lặp**: lấy bình phương các số **chia hết cho 3** từ **1 đến 30**.
```python
# Trả lời:
m = np.arange(3,31,3)
result = m ** 2
```

**0.4.** (Đọc đoạn này rồi giải thích lại bằng lời của bạn) Vòng lặp Python xử lý từng phần tử qua trình thông dịch, mỗi lần phải kiểm tra kiểu và gọi hàm. NumPy chạy vòng lặp bên trong mã đã biên dịch (C) trên vùng nhớ liền nhau, nên bỏ được phần chi phí đó.
Giải thích vì sao tính theo vectơ nhanh hơn for loop: nếu tôi hiểu đúng thứ nhất chạy trên C thì nhanh hơn là 1 thứ 2 thông dịch thì cần dịch toàn bộ nhưng biên dịch thì dịch đến đên chạy tới đó

**0.5.** Với `m` 2 hàng 3 cột: `m.sum(axis=0)` cộng theo hướng nào, kết quả có bao nhiêu phần tử, và vì sao? Điền hoàn chỉnh:
"axis=0 cộng các cột lại với nhau, kết quả có 3 phần tử."
"axis=1 cộng các hàng lại với nhau, kết quả có 2 phần tử."

---

## Bài học: pandas cơ bản

```python
import pandas as pd
df = pd.DataFrame({
    "user_id": [1, 2, 1, 3, 2, 1],
    "amount":  [50, 120, 80, 200, 30, 60],
    "status":  ["new", "paid", "paid", "new", "paid", "new"]
})
```
`df` là một **DataFrame** (bảng). Mỗi cột là một **Series** (một cột dữ liệu có chỉ số).

**Xem nhanh:** `df.head()` (vài dòng đầu), `df.shape` (số hàng, số cột), `df.info()` (kiểu dữ liệu), `df.describe()` (thống kê số).

**Lấy cột:**
```python
df["amount"]             # một Series
df[["user_id", "amount"]]  # một DataFrame con (2 cột)
```

**`loc` và `iloc`:**
- `loc` dùng **nhãn** (tên hàng, tên cột). Cắt lát bằng `loc` **gồm cả đầu mút cuối**.
- `iloc` dùng **vị trí** (số thứ tự từ 0). Cắt lát **không gồm** đầu mút cuối, giống Python thường.
```python
df.loc[1, "amount"]      # nhãn hàng 1, cột amount -> 120
df.iloc[1, 1]            # vị trí hàng 1, vị trí cột 1 -> 120
df.iloc[0:2]             # hàng vị trí 0 và 1
df.loc[0:2, "amount"]    # hàng nhãn 0, 1, 2 (gồm 2!)
```

**Lọc dòng:**
```python
df[df["amount"] > 100]                         # dòng có amount > 100
df[(df["amount"] > 50) & (df["status"] == "paid")]  # nhiều điều kiện: dùng & và dấu ngoặc đơn
```

---

## Bài tập A: Dự đoán đầu ra (viết dự đoán trước khi chạy)

Dùng `df` ở trên.

**A1.** `print(df.shape)`
Dự đoán: (6,3)
Đúng hay sai, vì sao: dự đoán đúng vì có tất cả 6 hàng và 3 cột

**A2.** `print(df.loc[1, "amount"])`
Dự đoán: 120
Đúng hay sai, vì sao: dự đoán đúng vì lấy giá trị tại hàng 1 cột amount 

**A3.** `print(df.iloc[0:2])` (liệt kê các hàng sẽ xuất hiện)
Dự đoán: {"user_id": [1,2], "amount": [50,120] , "status": ["paid", "paid"]}
Đúng hay sai, vì sao:    
   user_id  amount status
0        1      50           paid
1        2     120          paid

Đúng kết quả nhưng trình bày sai, kết quả là dataframe chứ không giống như dict

**A4.** `print(df.loc[0:2, "amount"])` (có bao nhiêu phần tử?)
Dự đoán: 3
Đúng hay sai, vì sao: Dự đoán đúng vì 3 hàng trong amount tương ứng kết quả hiện ra là 3

**A5.** `print(df[df["amount"] > 100])` (liệt kê nhãn hàng xuất hiện)
Dự đoán: 120 200
Đúng hay sai, vì sao: dự đoán đúng vì trong amount có đúng 120 và 200 là hơn 100

---

## Bài tập B: Viết code pandas

**B1.** Lấy cột `amount` (một Series).
```python
# Trả lời:
df["amount"]
```

**B2.** Lấy các dòng có `status == "paid"`.
```python
# Trả lời:
df[df["status"] == "paid"]
```

**B3.** Lấy giá trị `amount` của dòng ở **vị trí thứ 3** (đếm từ 1) bằng `iloc`.
```python
# Trả lời:
df.iloc[3,1]
```

**B4.** Lấy các dòng có `amount > 50` **và** `status == "paid"`.
```python
# Trả lời:
df[df["amount"] > 50 & df["status"] == "paid"]
```

**B5.** Tính trung bình cột `amount` bằng pandas. Tính tay trước để kiểm tra.
Tính tay: 90
```python
# Trả lời:
t = 0

for i in df["amount"]:
    t += i

print(t / df.shape[0])
```

---

## Bài tập C: Tự giải thích (không nhìn tài liệu)

**C1.** `loc` và `iloc` khác nhau ở đâu? Nêu cả khác biệt khi cắt lát.
Trả lời: iloc lấy dữ liệu 1 hàng trong 1 cột thì xác định cột theo số thứ tự thay vì tên như loc. Ngoài ra xác định giá trị trong 1 cột trong 1 hàng dễ hơn vì loc xác định cột theo tên cột thay vì dùng số thứ tự như iloc

**C2.** Vì sao A4 trả về 3 phần tử chứ không phải 2?
Trả lời: tôi chưa rõ vì theo lẽ thường thì chỉ phần từ 0 và phần từ 1

**C3.** Series và DataFrame khác nhau thế nào?
Trả lời: Series là một phần của dataframe có thể ví như 1 vector với 1 ma trận

---

## Ghi nhật ký
- Thời gian học thực tế:
- Câu làm chậm nhất, vì sao: Câu B5 vì tôi cần kiểm chứng suy nghĩ bản thân
- Điều chưa hiểu cần hỏi Claude: hiện tại thì chưa

> Làm xong gửi nguyên file này cho Claude chấm.
