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

**L1. [đã gặp] (2đ)** `4, 11, 25, 53, ?` Số tiếp theo (1đ) và quy luật viết thành công thức hoặc câu, đã thử trên mọi số hạng (1đ): số tiếp theo là 109 khoảng cách của 2 số bằng khoảng cách 2 số trước nhân 2
11 - 4 = 7, 25 - 11  = 14 (7 x 2), 53 - 25 = 28 (14 x 2), 109 - 53 = 56 (28 x 2)

**L2. [đã gặp] (2đ)** `2, 6, 12, 20, 30, ?` Số tiếp theo (1đ) và quy luật theo vị trí `n` (1đ): số tiếp theo là 42, quy luật là n x (n + 1), 2 = 1 x (1 + 1), 6 = 2 x (2 + 1), 12 = 3 x (3 + 1), 20 = 4 x (4 + 1), 30 = 5 x (5 + 1), 42 = 6 x (6 + 1)

**L3. [mới] (2đ)** `3, 20, 6, 17, 9, 14, ?` Số hạng thứ 7 (1đ) và quy luật (1đ): số hạng thứ 7 là 12 quy luật là các số hạng 1,3,5 là các số chia hết cho 3, các số hạng 2,4,6 hiệu các số đều là 3 và giảm dần

**L4. [mới] (5đ)** Năm bạn Hà, Khoa, Lan, Minh, Nam ngồi trên một băng ghế có 5 chỗ, đánh số 1 đến 5 từ trái sang phải. Biết: Hà ngồi ở chỗ số 2; Khoa ngồi **ngay bên phải** Hà (chỗ liền kề, số lớn hơn); Lan **không** ngồi ở hai đầu băng ghế (chỗ 1 và 5); Minh ngồi **bên trái** Nam (số chỗ nhỏ hơn, không cần liền kề).
Chỗ ngồi của từng người (1đ mỗi người): Hà = 2 , Khoa = 3 , Lan = 4, Minh = 1, Nam = 5
Thử lại với cả bốn điều kiện: không nhớ thử lại thế nào

**L5. [mới] (5đ)** Ba bạn X, Y, Z, mỗi người hoặc luôn nói thật hoặc luôn nói dối. X nói: "Y nói dối." Y nói: "Z nói thật." Z nói: "X và Y, một người nói thật và một người nói dối."
Ai nói thật, ai nói dối (3đ):
Thử lại từng câu của ba bạn có khớp không (2đ): bỏ và đánh dấu bài này hoàn thành vì càng giải thích càng rối 

**L6. [mới] (3đ)** Cho câu: "Nếu mạng lỗi thì trang web không mở được." Với mỗi suy luận dưới đây, ghi **hợp lệ** hay **không hợp lệ** (1đ mỗi ý); nếu không hợp lệ thì gọi tên lỗi.
(i) "Trang web không mở được. Vậy mạng lỗi." không nhớ tên lỗi nhưng suy luận ở đây là sai vì trang web không mở được không chắc là mạng lỗi mà có thể trang web đã chặn người dùng
(ii) "Trang web mở được. Vậy mạng không lỗi." hợp lệ
(iii) "Mạng lỗi. Vậy trang web không mở được." hợp lệ

**L7. [đã gặp] (2đ)** Hôm nay là Chủ nhật. 100 ngày sau là thứ mấy? Đáp số (1đ) và phép tính với phần dư (1đ): thứ 3, 100  % 7 = 2 

**L8. [mới, tìm lỗi] (3đ)** Hùng nói: "Dãy `2, 4, 8, 14, ?` có quy luật nhân đôi, nên số tiếp theo là 28."
(a) Hùng sai ở đâu (nêu số hạng cụ thể mà quy luật của Hùng không khớp)? (1đ) Hùng sai nếu quy luật của Hùng đúng thì số hạng 4 phải là 16 chứ không phải là 14
(b) Quy luật đúng và số tiếp theo (2đ): quy luật đúng hiệu của hiệu là 2 và số tiếp theo là 22

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
[1,2,3,4] [1,2,3,4,5] [1,2,3,4,6]

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
`f(3)` = 9; `f(2, 5)` = 32 ; `g(4)` in ra 8; `print(y)` in ra: None

**P3. [mới] (3đ)** Với `w = ["kiwi", "fig", "banana"]`, dự đoán:
(a) `sorted(w, key=len)` (2đ): ["fig", "kiwi", "banana"]
(b) `[x.upper() for x in w if len(x) > 3]` (1đ): ["KIWI", "fig", "BANANA"]

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
dự đoán là [1 9 3 4 5] và (5,) vì slicing của mảng numby không tạo mảng numby mới mà chỉ tạo 1 cửa số tức vẫn trỏ tới mảng numby cũ thay vì tạo mới dẫn đến khi v thay đổi giá trị phần từ thì arr cũng thay đổi theo

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

| product |     |
| ------- | --- |
| bút     | 90  |
| tẩy     | 16  |
| vở      | 50  |
bút = 10 x 3 + 20 x 3 = 90
tẩy = 8 x 2 = 16
vở = 5 x 10 = 50

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

| member_id | SUM(days) |
| --------- | --------- |
| 1         | 15        |
| 2         | 21        |
| 3         | 28        |

**Q2. [đã gặp] (2đ)** `SELECT COUNT(*), COUNT(days), AVG(days) FROM borrow;` Ba số (AVG làm tròn 3 chữ số thập phân):

| COUNT(*) | COUNT(days) | AVG(days) |
| -------- | ----------- | --------- |
| 7        | 6           | 10.667    |


**Q3. [đã gặp] (3đ)** `SELECT borrow_id, RANK() OVER (PARTITION BY member_id ORDER BY days DESC) AS r FROM borrow WHERE days IS NOT NULL ORDER BY borrow_id;` Giá trị `r` cho `borrow_id` 1, 3, 4, 5, 6, 7:

| borrow_id | r   |
| --------- | --- |
| 1         | 1   |
| 3         | 1   |
| 4         | 2   |
| 5         | 1   |
| 6         | 2   |
| 7         | 1   |


**Q4. [mới] (3đ)** Viết truy vấn liệt kê **tên** các thành viên chưa mượn cuốn sách nào, và ghi kết quả: SELECT m.name FROM members m LEFT JOIN borrow b ON m.member_id = b.member_id WHERE b.days IS NULL

---

**Giờ bắt đầu - giờ nộp (một dòng):** 8h40
