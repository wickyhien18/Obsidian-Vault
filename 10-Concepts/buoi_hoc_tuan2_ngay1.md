---
title: Buổi học Tuần 2 - Ngày 1 (SQL cơ bản)
tags: [bai-tap, sql, tuan-2]
---

# Tuần 2 - Ngày 1: Nợ từ cổng Tuần 1 và SQL cơ bản

**Khung giờ (1 giờ):**
- 10 phút: Phần 0, sửa nợ từ bài cổng
- 20 phút: đọc bài học SQL, chạy từng ví dụ
- 25 phút: bài tập A và B
- 5 phút: ghi nhật ký

(Bài C, tự giải thích, làm vào cuối buổi hoặc đầu buổi sau nếu hết giờ.)

> [!warning] Luật giữ nguyên
> **Điều kiện đề:** chép nguyên văn. **Kết quả mong đợi:** tính tay. Sau khi chạy, so **từng ô** rồi ghi "khớp" hoặc "không khớp". Dán kết quả chạy nguyên văn.

## Cách chạy SQL trong Colab
Dùng SQLite có sẵn của Python, không cần cài thêm:
```python
import sqlite3
import numpy as np
import pandas as pd

conn = sqlite3.connect(":memory:")
customers = pd.DataFrame({
    "customer_id": [1, 2, 3, 4],
    "name": ["An", "Bình", "Chi", "Dũng"],
    "city": ["Hà Nội", "Hà Nội", "Hải Phòng", "Đà Nẵng"]
})
orders = pd.DataFrame({
    "order_id": [1, 2, 3, 4, 5],
    "customer_id": [1, 1, 2, 3, 3],
    "order_date": ["2026-01-05", "2026-02-10", "2026-01-20", "2026-03-01", "2026-03-15"],
    "amount": [500, 700, 300, 900, 400]
})
customers.to_sql("customers", conn, index=False)
orders.to_sql("orders", conn, index=False)

def q(sql):
    return pd.read_sql_query(sql, conn)
```
Chạy truy vấn: `q("SELECT * FROM orders")`. Lưu ý khách hàng 4 (Dũng) **chưa có đơn hàng nào**.

---

## Phần 0: Sửa nợ từ bài cổng Tuần 1 (bài mới)

**0.1.** Dự đoán đầu ra và giải thích:
```python
a = np.array([10, 20])
b = a
b += 1
print(a)
```
Dự đoán: [11,21]
Giải thích (nêu rõ `+=` với mảng NumPy khác `append` của list thế nào): 
+= trong mảng NumPy tức là cộng giá trị với tất cả các phần từ trong mảng NumPy còn += trong list thì tương tự append trong list là thêm 1 phần từ vào sau 
**0.2.** Cho `m = np.array([[1, 2], [3, 4], [5, 6]])`. Tính trung bình **từng hàng**.
Điều kiện đề: tính trung bình của từng hàng
Kết quả mong đợi (tính tay): [1.5, 3.5, 5.5]
```python
# Trả lời:
m.mean(axis = 1)
```
Hoàn thành câu: "`axis=1` làm biến mất chiều 1, nên kết quả có 3 phần tử, mỗi phần tử ứng với một hàng."

**0.3.** Dữ liệu:
```python
df = pd.DataFrame({
    "dept":   ["A", "B", "A", "C", "B", "A"],
    "salary": [30, 45, 50, 25, 60, 40]
})
```
Viết một dòng: tổng `salary` theo `dept`, sắp xếp giảm dần.
Điều kiện đề: tính tổng salary theo dept sau đó sắp xếp giảm dần
```python
# Trả lời:
df.groupby("dept")["salary"].sum().sort_values(ascending=False)
```
Giải thích vì sao `df.groupby("dept")["salary"].sum().sort_values("salary", ascending=False)` báo lỗi (gợi ý: kết quả của `groupby(...)[...].sum()` là Series hay DataFrame?): báo lỗi vì kết quả là Series còn sort_values("salary", ascending=False)  chỉ áp dụng cho kết quả là DataFrame

**0.4.** Giải thích bằng 2 câu: "bất biến" và "khả biến" nghĩa là gì? Mỗi loại một ví dụ kiểu dữ liệu. Phải **định nghĩa**, không chỉ cho ví dụ.
Trả lời: bất biến là không sửa được tại chỗ sau khi tạo, mọi sự sửa đổi chỉ tạo đối tượng mới ví dụ a = 1, b = a, b += 1 thì khi in ra a và b có kết quả khác vì không trỏ cùng 1 đối tượng. khả biến là sửa được tại chỗ sau khi tạo, mọi sự sửa đổi không tạo đối tượng mới mà sửa chính đổi tượng đó ví dụ a = [1], b = a, b += [2] thì khi in ra và b cùng là [1,2] vì cùng trỏ tới cùng 1 đối tượng

**0.5.** Với `df` ở 0.3, lấy các dòng có `dept == "B"` **và** `salary > 50`.
Điều kiện đề: lấy các dòng có dept == "B" và salary > 50
Bảng kiểm từng hàng:

| nhãn hàng | dept == "B" ? | salary > 50 ? | cả hai ? |
| --------- | ------------- | ------------- | -------- |
| 0         | sai           | sai           | sai      |
| 1         | đúng          | sai           | sai      |
| 2         | sai           | sai           | sai      |
| 3         | sai           | sai           | sai      |
| 4         | đúng          | đúng          | đúng     |
| 5         | sai           | sai           | sai      |

Kết quả mong đợi (nhãn hàng):

|     | dept | salary |
| --- | ---- | ------ |
| 4   | B    | 60     |

```python
# Trả lời:
df[(df["dept"] == "B") & (df["salary"] > 50)]
```
Kết quả chạy, khớp hay không khớp: kết quả khớp
  dept  salary
4    B      60

---

## Bài học: SQL cơ bản

SQL là ngôn ngữ truy vấn dữ liệu trong bảng. Viết **gần như tiếng Anh**: bạn mô tả bạn muốn gì, không mô tả cách làm.

**Khung cơ bản:**
```sql
SELECT cột
FROM bảng
WHERE điều_kiện
ORDER BY cột DESC
LIMIT n;
```
- `SELECT *` là lấy mọi cột. `SELECT DISTINCT city` bỏ giá trị trùng.
- Chuỗi trong SQL dùng **dấu nháy đơn**: `WHERE city = 'Hà Nội'`.
- `ORDER BY ... DESC` là giảm dần, `ASC` (mặc định) là tăng dần.

**Ví dụ:**
```sql
SELECT * FROM orders WHERE amount > 400 ORDER BY amount DESC;
-- ra 3 đơn: amount 900, 700, 500
```

**Gộp nhóm với `GROUP BY` và hàm tổng hợp** (`SUM`, `COUNT`, `AVG`, `MAX`, `MIN`):
```sql
SELECT customer_id, SUM(amount) AS total
FROM orders
GROUP BY customer_id;
-- customer_id 1 -> 1200, 2 -> 300, 3 -> 1300
```
(Giống `groupby` ở pandas: tách thành nhóm, tính trên từng nhóm, gộp kết quả.)

**`WHERE` và `HAVING`:**
- `WHERE` lọc **từng dòng** **trước** khi gộp nhóm.
- `HAVING` lọc **các nhóm** **sau** khi đã gộp, và dùng được với hàm tổng hợp.
```sql
SELECT customer_id, SUM(amount) AS total
FROM orders
GROUP BY customer_id
HAVING SUM(amount) > 1000;
-- chỉ giữ customer_id 1 (1200) và 3 (1300)
```

---

## Bài tập A: Dự đoán đầu ra

**A1.** `SELECT COUNT(*) FROM orders;`
Dự đoán: 5
Đúng hay sai, vì sao: dự đoán đúng vì đây là đếm tất cả các thành phần trong bảng orders

**A2.** `SELECT customer_id, COUNT(*) FROM orders GROUP BY customer_id;`
Dự đoán (từng nhóm):

|     | customer_id | COUNT(*) |
| --- | ----------- | -------- |
| 0   | 1           | 2        |
| 1   | 2           | 1        |
| 2   | 3           | 2        |

Đúng hay sai, vì sao: dự đoán đúng vì sau khi gom theo từng nhóm customer_id thì đếm xem từng nhóm có bao nhiêu phần từ

**A3.** `SELECT * FROM orders ORDER BY amount DESC LIMIT 2;` (liệt kê `order_id`)
Dự đoán: 

| order_id |
| -------- |
| 4        |
| 2        |

Đúng hay sai, vì sao: dự đoán đúng vì đây là sắp xếp giảm dần bảng orders theo amount sau đó chỉ lấy 2 phần từ đầu

**A4.** `SELECT city FROM customers WHERE city = 'Hà Nội';` có bao nhiêu dòng? Còn `SELECT DISTINCT city FROM customers;` có bao nhiêu dòng?
Dự đoán: lệnh select đầu có 2 dòng, lệnh select sau có 3 dòng
Đúng hay sai, vì sao: lệnh select đầu tìm tất cả giá trị city trong customers và giữ lại city = 'Hà Nội'. Lệnh select sau thì tìm tất cả các giá trị city là duy nhất trong customers

---

## Bài tập B: Viết truy vấn SQL

**B1.** Lấy các đơn hàng có `amount` từ 400 đến 800 (**gồm cả hai đầu**). (Gợi ý: `BETWEEN`.)
Điều kiện đề: lấy các đơn hàng có amount từ 400 đến 800 
Kết quả mong đợi (liệt kê `order_id`, tính tay):

| order_id |
| -------- |
| 1        |
| 2        |
| 5        |

```sql
-- Trả lời:
SELECT * FROM orders
WHERE amount BETWEEN 400 AND 800;
```
Kết quả chạy, khớp hay không khớp: kết quả khớp

|     | order_id | customer_id | order_date | amount |
| --- | -------- | ----------- | ---------- | ------ |
| 0   | 1        | 1           | 2026-01-05 | 500    |
| 1   | 2        | 1           | 2026-02-10 | 700    |
| 2   | 5        | 3           | 2026-03-15 | 400    |


**B2.** Tổng `amount` theo từng `customer_id`, **chỉ giữ** các khách có tổng lớn hơn 500, sắp xếp **giảm dần** theo tổng.
Điều kiện đề: tính tổng amount theo từng customer_id, chỉ giữ lại các khách có tổng amount lớn hơn 500 sau đó sắp xếp giảm dần theo tổng
Kết quả mong đợi (tính tay):

|     | customer_id | total |
| --- | ----------- | ----- |
| 0   | 3           | 1300  |
| 1   | 1           | 1200  |

```sql
-- Trả lời:
SELECT customer_id, SUM(amount) AS total FROM orders GROUP BY customer_id HAVING total > 500 ORDER BY total DESC
```
Kết quả chạy, khớp hay không khớp: kết quả chạy khớp

|     | customer_id | total |
| --- | ----------- | ----- |
| 0   | 3           | 1300  |
| 1   | 1           | 1200  |

**B3.** Với từng `customer_id`, đếm số đơn, **chỉ tính các đơn có `amount > 350`**.
Điều kiện đề: với từng customer_id chỉ đếm các đơn có amount > 350
Kết quả mong đợi (tính tay):

|     | customer_id | COUNT(order_id) |
| --- | ----------- | --------------- |
| 0   | 1           | 2               |
| 1   | 3           | 2               |


```sql
-- Trả lời:
SELECT customer_id, COUNT(order_id) FROM orders WHERE amount > 350 GROUP BY customer_id 
```
Kết quả chạy, khớp hay không khớp: kết quả chạy khớp

|     | customer_id | COUNT(order_id) |
| --- | ----------- | --------------- |
| 0   | 1           | 2               |
| 1   | 3           | 2               |


**B4.** Tính trung bình `amount` của **toàn bảng** `orders` (**một con số**).
Điều kiện đề: tính trung bình amount của toàn bảng orders (một con số)
Kết quả mong đợi (tính tay): 560
```sql
-- Trả lời:
SELECT AVG(amount) FROM orders
```
Kết quả chạy, khớp hay không khớp: kết quả khớp

|     | AVG(amount) |
| --- | ----------- |
| 0   | 560.0       |

---

## Bài tập C: Tự giải thích (không nhìn tài liệu)

**C1.** `WHERE` và `HAVING` khác nhau thế nào? Cho một ví dụ từ dữ liệu hôm nay mà **chỉ** `HAVING` làm được.
Trả lời: where là lọc trước rồi mới gộp kết quả lại, having là nhóm/gộp các nhóm xong rồi mới lọc ví dụ muốn chỉ hiện customer_id có số lượng đơn trên 2 thì sẽ cần nhóm các customer_id lại rồi đếm rồi lọc

**C2.** Vì sao B2 phải dùng `HAVING` chứ không dùng `WHERE`?
Trả lời: vì theo cú pháp thì where không dùng được các hàm như AVG(), COUNT() còn having thì dùng được

---

## Ghi nhật ký
- Thời gian học thực tế: 1 tiếng
- Câu bạn thấy mình sai nhiều khả năng nhất: 2 câu phần C vì có theer khả năng giải thích tôi không được chuẩn
- Điều chưa hiểu: code chạy trên gg colab kia không được chuẩn vì thiếu display() thì mới hiển thị được tôi không rõ ông cố tình hay lỗi thật và chỉ gg colab mới chạy được tôi muốn chạy trên cả python trên các web online

> Làm xong gửi nguyên file này cho Claude chấm.
