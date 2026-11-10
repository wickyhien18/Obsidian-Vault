---
title: Đề thi thử nhỏ - Buổi 1 (10/11): Logic và Lập trình/Dữ liệu cơ bản
tags: [thi-thu, logic, lap-trinh]
---

# Đề thi thử nhỏ, Buổi 1 (10/11): 60 phút, 50 điểm

**Phạm vi:** dãy số (hiệu, tỉ số, hiệu của hiệu, hai phép tính, hai dãy đan xen, công thức theo vị trí); xếp thứ tự và vị trí; nói thật/nói dối; câu điều kiện; ngày trong tuần (phần dư chia 7); Python (list, hàm, comprehension, `sorted`, NumPy view); pandas (`merge`, `groupby`); SQL (`GROUP BY`/`HAVING`, NULL, `RANK`, `LEFT JOIN`). Không có nội dung ngoài các buổi đã học.

**Quy tắc thi (giống thi thật):**
- **Bấm giờ 60 phút, làm một lần liền.** Gợi ý: Phần I 25 phút, Phần II 35 phút. Hết giờ thì dừng, câu chưa làm ghi "chưa làm".
- **Không chạy code, không tra cứu, không xem lộ trình.** Dùng nháp và máy tính bỏ túi nếu muốn.
- Chưa biết thì ghi **"không biết"**, đề mơ hồ thì ghi **"đề mơ hồ vì ..."**; không bỏ trống.
- Ghi **giờ bắt đầu và giờ nộp** ở cuối file (một dòng). Không có nhật ký.
- Sau khi nộp xong mới được chạy `sql_thi_thu_10_11.py` để xem kết quả SQL (đừng sửa đáp án sau khi xem).
- Nhãn **[đã gặp]** (cùng dạng đã làm) và **[mới]** (dạng chưa làm) để mình báo điểm riêng. Điểm ghi trong ngoặc.

**Mục tiêu đo:** mỗi nhóm (Logic, Lập trình và dữ liệu) có đạt từ 65% (nhóm Logic từ 16/24, nhóm Lập trình và dữ liệu từ 17/26) hay không.

---

# PHẦN I: LOGIC (25 phút, 24 điểm)

**L1. [đã gặp] (2đ)** `4, 11, 25, 53, ?` Số tiếp theo (1đ) và quy luật viết thành công thức hoặc câu, đã thử trên mọi số hạng (1đ): số tiếp theo là 

**L2. [đã gặp] (2đ)** `2, 6, 12, 20, 30, ?` Số tiếp theo (1đ) và quy luật theo vị trí `n` (1đ):

**L3. [mới] (2đ)** `3, 20, 6, 17, 9, 14, ?` Số hạng thứ 7 (1đ) và quy luật (1đ):

**L4. [mới] (5đ)** Năm bạn Hà, Khoa, Lan, Minh, Nam ngồi trên một băng ghế có 5 chỗ, đánh số 1 đến 5 từ trái sang phải. Biết: Hà ngồi ở chỗ số 2; Khoa ngồi **ngay bên phải** Hà (chỗ liền kề, số lớn hơn); Lan **không** ngồi ở hai đầu băng ghế (chỗ 1 và 5); Minh ngồi **bên trái** Nam (số chỗ nhỏ hơn, không cần liền kề).
Chỗ ngồi của từng người (1đ mỗi người): Hà = , Khoa = , Lan = , Minh = , Nam =
Thử lại với cả bốn điều kiện:

**L5. [mới] (5đ)** Ba bạn X, Y, Z, mỗi người hoặc luôn nói thật hoặc luôn nói dối. X nói: "Y nói dối." Y nói: "Z nói thật." Z nói: "X và Y, một người nói thật và một người nói dối."
Ai nói thật, ai nói dối (3đ):
Thử lại từng câu của ba bạn có khớp không (2đ):

**L6. [mới] (3đ)** Cho câu: "Nếu mạng lỗi thì trang web không mở được." Với mỗi suy luận dưới đây, ghi **hợp lệ** hay **không hợp lệ** (1đ mỗi ý); nếu không hợp lệ thì gọi tên lỗi.
(i) "Trang web không mở được. Vậy mạng lỗi."
(ii) "Trang web mở được. Vậy mạng không lỗi."
(iii) "Mạng lỗi. Vậy trang web không mở được."

**L7. [đã gặp] (2đ)** Hôm nay là Chủ nhật. 100 ngày sau là thứ mấy? Đáp số (1đ) và phép tính với phần dư (1đ):

**L8. [mới, tìm lỗi] (3đ)** Hùng nói: "Dãy `2, 4, 8, 14, ?` có quy luật nhân đôi, nên số tiếp theo là 28."
(a) Hùng sai ở đâu (nêu số hạng cụ thể mà quy luật của Hùng không khớp)? (1đ)
(b) Quy luật đúng và số tiếp theo (2đ):

---

# PHẦN II: LẬP TRÌNH VÀ DỮ LIỆU CƠ BẢN (35 phút, 26 điểm)

**P1. [đã gặp] (3đ)** Dự đoán dòng in ra (1đ mỗi list):
```python
a = [1, 2, 3]
b = a
b += [4]
c = a + [5]
d = a.copy()
d.append(6)
print(a, c, d)
```

**P2. [đã gặp] (4đ)** Dự đoán (1đ mỗi ô):
```python
def f(x, k=2):
    return x ** k

def g(x):
    print(x * 2)

print(f(3), f(2, 5))
y = g(4)
print(y)
```
`f(3)` = ; `f(2, 5)` = ; `g(4)` in ra ; `print(y)` in ra:

**P3. [mới] (3đ)** Với `w = ["kiwi", "fig", "banana"]`, dự đoán:
(a) `sorted(w, key=len)` (2đ):
(b) `[x.upper() for x in w if len(x) > 3]` (1đ):

**P4. [đã gặp] (3đ)**
```python
import numpy as np
arr = np.array([1, 2, 3, 4, 5])
v = arr[1:4]
v[0] = 9
print(arr)
print(arr.shape)
```
Dự đoán hai dòng in ra (2đ cho dòng 1, 1đ cho dòng 2) và một câu giải thích vì sao `arr` đổi (không tính điểm nhưng cần thiết cho nhận xét):

**P5. [đã gặp] (3đ)** Cho hai bảng:
```python
import pandas as pd
o = pd.DataFrame({"oid": [1, 2, 3, 4], "product": ["bút", "vở", "bút", "tẩy"], "qty": [10, 5, 20, 8]})
pr = pd.DataFrame({"product": ["bút", "vở", "tẩy"], "price": [3, 10, 2]})
m = pd.merge(o, pr, on="product")
m["revenue"] = m["qty"] * m["price"]
print(m.groupby("product")["revenue"].sum())
```
Dự đoán doanh thu theo từng sản phẩm (1đ mỗi sản phẩm), kèm cách tính:

## SQL (10 điểm)
Hai bảng thư viện (`days` là số ngày mượn, NULL là chưa trả):

`members`: (1, Hà), (2, Khoa), (3, Lan), (4, Minh) với cột `member_id`, `name`.

`borrow`:

| borrow_id | member_id | book | days |
|---|---|---|---|
| 1 | 1 | Toán | 10 |
| 2 | 1 | Lý | NULL |
| 3 | 2 | Hóa | 14 |
| 4 | 2 | Văn | 7 |
| 5 | 3 | Toán | 14 |
| 6 | 1 | Sử | 5 |
| 7 | 3 | Sử | 14 |

**Q1. [đã gặp] (2đ)** `SELECT member_id, SUM(days) FROM borrow GROUP BY member_id HAVING SUM(days) >= 15;` Các hàng kết quả (kèm tổng của từng thành viên):

**Q2. [đã gặp] (2đ)** `SELECT COUNT(*), COUNT(days), AVG(days) FROM borrow;` Ba số (AVG làm tròn 3 chữ số thập phân):

**Q3. [đã gặp] (3đ)** `SELECT borrow_id, RANK() OVER (PARTITION BY member_id ORDER BY days DESC) AS r FROM borrow WHERE days IS NOT NULL ORDER BY borrow_id;` Giá trị `r` cho `borrow_id` 1, 3, 4, 5, 6, 7:

**Q4. [mới] (3đ)** Viết truy vấn liệt kê **tên** các thành viên chưa mượn cuốn sách nào, và ghi kết quả:

---

**Giờ bắt đầu - giờ nộp (một dòng):**
