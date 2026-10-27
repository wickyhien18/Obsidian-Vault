---
title: Thứ Ba 27/10 - Phần 2 (30 phút): ôn SQL, pandas, xác suất bằng cách chạy thật
tags: [bai-tap, sql, pandas, xac-suat, tuan-3]
---

# 27/10, Phần 2 (30 phút): ôn có chạy code

**Lịch hôm nay: 1 giờ.** Phần 1 = Buổi 1 trong `thong_ke_giang_lai.md` (30 phút, có 5 phút khởi động W1/W2). Phần 2 = file này (30 phút). Làm cách nhau ít nhất 1 tiếng nếu được.

**Luật:** dự đoán/tính tay **trước**, chạy code **sau** và dán nguyên văn kết quả chạy. Làm tròn: bình thường, lệch ≤ 0,001 vẫn đúng. Đề mơ hồ thì ghi "đề mơ hồ vì ..." vào ô trả lời.

## Dữ liệu (SQLite, tạo bằng `sql_setup_portable.py` hoặc Colab)
```
customers(customer_id, name): (1, An), (2, Bình), (3, Chi), (4, Dũng)
orders(order_id, customer_id, amount): (1,1,50), (2,1,30), (3,2,20), (4,3,40), (5,3,60), (6,3,10)
```

**R1.** Với mỗi đơn hàng, hiển thị `order_id`, `amount` và **tổng cộng dồn toàn bảng** theo thứ tự `order_id`.
Điều kiện đề (chép nguyên văn): với mỗi đơn hàng, hiển thị order_id, amount và tổng cộng dồn toàn bảng theo thứ tự order_id
Kết quả mong đợi (cột cộng dồn của 6 dòng, tính tay):

| tongcongdon |
| ----------- |
| 50          |
| 80          |
| 100         |
| 140         |
| 200         |
| 210         |

```sql
-- Trả lời:
SELECT order_id, amount, SUM(amount) OVER(ORDER BY order_id) FROM orders
```
Kết quả chạy, khớp hay không khớp: kết quả khớp
order_id | amount | tongcongdon  
------------------------------  
1 | 50 | 50  
2 | 30 | 80  
3 | 20 | 100  
4 | 40 | 140  
5 | 60 | 200  
6 | 10 | 210

**R2.** Liệt kê tên khách hàng **chưa có đơn nào**.
Điều kiện đề (chép nguyên văn): liệt kê tên khách hàng chưa có đơn nào
Kết quả mong đợi (tính tay):

| name |
| ---- |
| Dũng |

```sql
-- Trả lời:
SELECT c.name FROM customers c LEFT JOIN orders o ON c.customer_id = o.customer_id WHERE o.customer_id IS NULL
```
Kết quả chạy, khớp hay không khớp: kết quả khớp
name  
------------------------------  
Dũng

**R3.** Với mỗi khách hàng, tính **tổng `amount`**. Chỉ giữ khách có tổng **lớn hơn 60**, sắp xếp theo tổng **giảm dần**. Hiển thị `name` và tổng.
Điều kiện đề (chép nguyên văn): với mỗi khách hàng tính tổng amount, chỉ giữ khách có tổng lớn hơn 60, sắp xếp theo tổng giảm dần, hiển thị name và tổng
Kết quả mong đợi (tính tay, mỗi dòng có tên và tổng):

| name | SUM(o.amount) |
| ---- | ------------- |
| Chi  | 110           |
| An   | 80            |

```sql
-- Trả lời:
SELECT c.name, SUM(o.amount) FROM customers c JOIN orders o ON c.customer_id = o.customer_id GROUP BY c.customer_id HAVING SUM(o.amount) > 60 ORDER BY SUM(o.amount) DESC
```
Kết quả chạy, khớp hay không khớp: kết quả khớp
name | SUM(o.amount)  
------------------------------  
Chi | 110  
An | 80

**R4.** Với mỗi đơn hàng, hiển thị `order_id`, `customer_id`, `amount` và **tổng cộng dồn riêng của từng khách** (mỗi khách bắt đầu cộng dồn lại từ đầu), theo thứ tự `order_id`.
Điều kiện đề (chép nguyên văn): với mỗi đơn khách hiển thị order_id, customer_id, amount và tổng cộng dồn riêng từng khách (mỗi khách bắt đầu cộng dồn lại từ đầu) theo thứ tự order_id
Kết quả mong đợi (cột cộng dồn của 6 dòng, tính tay):

| order_id | customer_id | amount | tongcongdon |
| -------- | ----------- | ------ | ----------- |
| 1        | 1           | 50     | 50          |
| 2        | 1           | 30     | 80          |
| 3        | 2           | 20     | 20          |
| 4        | 3           | 40     | 40          |
| 5        | 3           | 60     | 100         |
| 6        | 3           | 10     | 110         |

```sql
-- Trả lời:
SELECT *, SUM(amount) OVER (PARTITION BY customer_id ORDER BY order_id) AS tongcongdon FROM orders ORDER BY order_id
```
Kết quả chạy, khớp hay không khớp: kết quả khớp
order_id | customer_id | amount | tongcongdon  
------------------------------  
1 | 1 | 50 | 50  
2 | 1 | 30 | 80  
3 | 2 | 20 | 20  
4 | 3 | 40 | 40  
5 | 3 | 60 | 100  
6 | 3 | 10 | 110
(Gợi ý duy nhất: so với R1, đề này khác ở chỗ nào? Một câu:)

**R5.** Làm lại **R3 bằng pandas** (ghép hai bảng, nhóm, lọc, sắp xếp). Dựng `customers` và `orders` thành hai DataFrame từ dữ liệu trên.
Điều kiện đề (chép nguyên văn, kể cả "bằng pandas"): làm lại R3 bằng pandas dựng customers và orders  thành 2 dataFrame từ dữ liệu trên
Kết quả mong đợi (tính tay):

```python
# Trả lời:
import pandas as pd 

customers = pd.DataFrame({
    "customer_id" : [1,2,3,4],
    "name": ["An", "Bình", "Chi", "Dũng"]
})

orders = pd.DataFrame({
    "order_id": [1,2,3,4,5,6],
    "customer_id": [1,1,2,3,3,3],
    "amount": [50,30,20,40,60,10]
})

merges = pd.merge(customers, orders, on="customer_id", how="inner")

results = merges.groupby("name")["amount"].sum()

results = results[results > 60].sort_values(ascending=False)

print(results)
```
Kết quả chạy, khớp với R3 hay không: chạy khớp
name
Chi    110
An      80

**R6.** Hộp có 3 bi đỏ và 5 bi xanh. Rút hai bi liên tiếp, không hoàn lại.
(a) Nếu lần 1 rút **xanh**, hộp còn 7 bi, trong đó 3 bi đỏ.
(b) `P(lần 2 đỏ | lần 1 xanh)` = 3/7 = 0.428 (phân số, rồi thập phân 3 chữ số)
(c) `P(lần 1 xanh và lần 2 đỏ)`: công thức P(lần 1 xanh và lần 2 đỏ) = P(lần 1 xanh) * P(lần 2 đỏ | lần 1 xanh), rồi phân số 15/56 , thập phân 3 chữ số 0.267
(d) Kiểm tra bằng mô phỏng (tùy chọn): 
```python
import numpy as np
rng = np.random.default_rng(0)
box = np.array([0]*3 + [1]*5)       # 0 = đỏ, 1 = xanh
n = 100000; dem = 0
for _ in range(n):
    a, b = rng.permutation(box)[:2]
    dem += (a == 1 and b == 0)
print(dem / n)
```
Kết quả chạy (dán nguyên văn), khớp với (c) hay không: 0.26777
kết quả khớp với c

## Ghi nhật ký (điền đủ ba ô)
- Thời gian bắt đầu và kết thúc thực tế (Phần 1, Phần 2): phần 2: 7h30 - 8h30
- Câu bạn nghĩ mình sai nhiều khả năng nhất, và vì sao: R4 không rõ tổng cộng dồn cột nào nên tôi hiểu là tổng cộng dồn amount, nếu có nhiều cột hơn mà không nói rõ cột nào thì tôi không biết làm
- Điều chưa hiểu (nếu viết "không có" thì nêu đã chạy code kiểm những câu nào): không có vì đã kiểm chứng hết 
