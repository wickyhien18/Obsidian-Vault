---
title: Buổi học Tuần 2 - Ngày 2 (SQL JOIN)
tags: [bai-tap, sql, tuan-2]
---

# Tuần 2 - Ngày 2: Kiểm tra nhớ lỗi cũ và SQL JOIN

**Khung giờ tối nay (bản tăng tốc, khoảng 2,5 giờ, 21:00 - 23:30):**
- 21:00 - 21:10: Phần 0, kiểm tra xem lỗi cũ đã nhớ chưa
- 21:10 - 21:25: đọc bài học JOIN, chạy từng ví dụ
- 21:25 - 21:55: bài tập A và B
- 21:55 - 22:05: bài C
- 22:05 - 23:05: **Phần D (mới): window function**
- 23:05 - 23:25: **Phần E (mới): CV, mục 1 đến 3**
- 23:25 - 23:30: ghi nhật ký

> [!warning] Luật giữ nguyên
> **Điều kiện đề:** chép nguyên văn, **kể cả các từ như "gồm cả hai đầu", "chưa từng", "giảm dần"**. **Kết quả mong đợi:** tính tay. Sau khi chạy, so **từng ô** rồi ghi "khớp" hoặc "không khớp". Dán kết quả chạy nguyên văn.

## Cách chạy
Dùng lại ô cài đặt Ngày 1 (Colab), hoặc dùng file `sql_setup_portable.py` (chạy ở **bất kỳ đâu** có Python, không cần pandas). Với file `.py`, mỗi truy vấn viết dạng `show("SELECT ...")`.

Nếu dùng Colab/Jupyter với hàm `q(...)`: **chỉ ô cuối cùng** trong một ô lệnh tự hiển thị. Nhiều truy vấn trong một ô thì bọc từng cái trong `display(...)` hoặc `print(...)`.

Nhắc dữ liệu: khách hàng 4 (Dũng, Đà Nẵng) **chưa có đơn hàng nào**.

---

## Phần 0: Kiểm tra xem lỗi cũ đã nhớ chưa (bài mới)

**0.1.** Dự đoán đầu ra và giải thích:
```python
a = np.array([1, 2, 3])
b = a
b += 2
c = b * 10
print(a, c)
```
Dự đoán: [3,4,5][30,40,50]
Giải thích: vì a và b cùng trỏ tới mảng numpy đồng nghĩa b cộng 2 cho tất cả các phần tử thì a cũng cộng 2 cho tất cả các phần từ bẳng phép tính +=, nhưng c = b * 10 khi này b và c không trỏ tới cùng 1 mảng numpy đồng nghĩa c nhân 10 với tất cả các phần từ trong mảng thì b không thay đổi

**0.2.** Cho `m = np.array([[1, 2, 3], [4, 5, 6]])`. Tính trung bình **từng cột**.
Điều kiện đề: tính trung bình từng cột
Kết quả mong đợi (tính tay): [2.5,3.5,4.5]
```python
# Trả lời:
m.mean(axis = 0)
```
Kết quả chạy, khớp hay không khớp: kết quả khớp
[2.5 3.5 4.5]

**0.3.** Thứ tự thực hiện logic của một truy vấn SQL: `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `ORDER BY` → `LIMIT`. Dựa vào đó, điền: "Khi `WHERE` chạy thì GROUP BY chưa được thực hiện, nên chưa có  SUM(amount) , vì vậy `WHERE SUM(amount) > 500` không dùng được."

**0.4.** Viết lại truy vấn B2 hôm trước **không dùng bí danh `total` trong `HAVING`**, mà dùng `HAVING SUM(amount) > 500`. Giải thích bằng một câu vì sao viết kiểu này an toàn hơn khi đổi sang hệ cơ sở dữ liệu khác.
```sql
-- Trả lời:
SELECT customer_id, SUM(amount) FROM orders GROUP BY customer_id HAVING SUM(amount) > 500 ORDER BY SUM(amount) DESC
```
Giải thích: cái này tôi chưa rõ nhưng tôi đoán là để linh hoạt giữa các hệ cơ sở dữ liệu khác hơn thay vì bó cứng vào 1 cái tên

**0.5.** Viết truy vấn: các khách có **từ 2 đơn trở lên**, hiển thị `customer_id` và số đơn.
Điều kiện đề: viết truy vấn các khách hàng có 2 đơn trở lên và hiển thị customer_id và số đơn
Kết quả mong đợi (tính tay):

| customer_id | COUNT(order_id) |
| ----------- | --------------- |
| 1           | 2               |
| 3           | 2               |

```sql
-- Trả lời:
SELECT customer_id, COUNT(order_id) FROM orders GROUP BY customer_id HAVING COUNT(order_id) >= 2
```
Kết quả chạy, khớp hay không khớp: kết quả khớp

| customer_id | COUNT(order_id) |
| ----------- | --------------- |
| 1           | 2               |
| 3           | 2               |


---

## Bài học: JOIN (ghép bảng)

Dữ liệu thường nằm ở nhiều bảng. `JOIN` ghép các dòng của hai bảng theo một cột chung.

**`JOIN` (hay `INNER JOIN`): chỉ giữ những dòng có khớp ở cả hai bảng.**
```sql
SELECT c.name, o.order_id, o.amount
FROM customers AS c
JOIN orders AS o ON c.customer_id = o.customer_id;
```
- `AS c` và `AS o` là **bí danh** của bảng, gõ ngắn hơn.
- `ON` nói hai bảng khớp nhau ở cột nào.
- Kết quả có 5 dòng (mỗi đơn một dòng). **Dũng không xuất hiện**, vì không có đơn nào khớp.

**`LEFT JOIN`: giữ mọi dòng của bảng bên trái, bên phải không khớp thì điền `NULL`.**
```sql
SELECT c.name, o.order_id, o.amount
FROM customers AS c
LEFT JOIN orders AS o ON c.customer_id = o.customer_id;
```
Kết quả có 6 dòng: 5 dòng cũ và một dòng của Dũng với `order_id` và `amount` là `NULL`. (Giống `merge(how="left")` ở pandas.)

**Tìm bản ghi không khớp:**
```sql
SELECT c.name
FROM customers AS c
LEFT JOIN orders AS o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
-- ra: Dũng
```

**Về `NULL`:**
- `NULL` nghĩa là "chưa biết / không có giá trị", **không phải số 0** hay chuỗi rỗng.
- Kiểm tra bằng `IS NULL` / `IS NOT NULL`. **Không dùng `= NULL`**: so sánh với `NULL` không bao giờ cho kết quả đúng.
- `COUNT(*)` đếm **mọi dòng**, kể cả dòng có giá trị `NULL`. `COUNT(cột)` đếm **chỉ những dòng mà cột đó khác `NULL`**. Đây là chỗ hay sai khi dùng với `LEFT JOIN`.

---

## Bài tập A: Dự đoán đầu ra

**A1.** `SELECT c.name, o.amount FROM customers c JOIN orders o ON c.customer_id = o.customer_id;` có bao nhiêu dòng? Dũng có xuất hiện không?
Dự đoán: dự đoán là 5 dòng và dúng không xuất hiện
Đúng hay sai, vì sao: dự đoán đúng vì chỉ giữ lại các dòng có khớp của cả 2 bảng tức Dũng với customer_id = 4 mà trong orders không có customer_id = 4 thành ra không có tên dũng

**A2.** Cùng truy vấn nhưng dùng `LEFT JOIN`. Có bao nhiêu dòng? `amount` của Dũng là gì?
Dự đoán: dự đoán là 6 và amount của Dũng là None
Đúng hay sai, vì sao: dự đoán đúng vì với LEFT JOIN  giữ lại toàn bộ thông tin của customers bao gồm cả các dòng không khớp đồng nghĩa có tên Dũng nhưng trong orders không có thông tin Dũng nên amount của Dũng là None

**A3.** `SELECT c.name, COUNT(*) FROM customers c LEFT JOIN orders o ON c.customer_id = o.customer_id GROUP BY c.name;` (liệt kê số đếm của từng người)
Dự đoán:

| name | COUNT(*) |
| ---- | -------- |
| An   | 2        |
| Bình | 1        |
| Chí  | 2        |
| Dũng | 1        |

Đúng hay sai, vì sao: dự đoán đúng vì với COUNT(*) thì đếm tất cả các dòng dự liệu kể cả là NULL hay là không đồng nghĩa dòng thông tin của Dũng cũng được đếm 

**A4.** Giống A3 nhưng thay `COUNT(*)` bằng `COUNT(o.order_id)`. Số đếm của Dũng là gì?
Dự đoán: số đếm của Dũng là 0
Đúng hay sai, vì sao: dự đoán đúng vì với COUNT(o.order_id) tức đếm 1 cột cụ thể thì sẽ chỉ đếm các dòng chứa dữ liệu khác NULL mà dòng của Dũng chứa dữ liệu NULL nên COUNT không đếm dòng của DŨng

**A5.** `SELECT c.name FROM customers c LEFT JOIN orders o ON c.customer_id = o.customer_id WHERE o.order_id = NULL;` trả về bao nhiêu dòng?
Dự đoán: không dòng nào
Đúng hay sai, vì sao: dự đoán đúng vì biểu thức o.order_id = NULL không bao giờ cho kết quả đúng tức lệnh SQL trên không hiển thị ra gì

---

## Bài tập B: Viết truy vấn

**B1.** Liệt kê tên khách và `amount` của từng đơn (chỉ những khách **có đơn**).
Điều kiện đề: liệt kê tên khách và amount của từng đơn  (chỉ những khách có đơn)
Kết quả mong đợi (tính tay):

| name | amount |
| ---- | ------ |
| An   | 500    |
| An   | 700    |
| Bình | 300    |
| Chi  | 400    |
| Chi  | 900    |

```sql
-- Trả lời:
SELECT c.name, o.amount FROM customers c JOIN orders o ON c.customer_id = o.customer_id
```
Kết quả chạy, khớp hay không khớp: kết quả khớp
name | amount  
------------------------------  
An | 500  
An | 700  
Bình | 300  
Chi | 400  
Chi | 900

**B2.** Liệt kê tên các khách **chưa từng** có đơn hàng nào. (Đây là câu B2 của bài chẩn đoán Tuần 0.)
Điều kiện đề: liệt kê các khách chưa từng có đơn đặt hàng nào
Kết quả mong đợi (tính tay):

| name |
| ---- |
| Dũng |

```sql
-- Trả lời:
SELECT c.name FROM customers c LEFT JOIN orders o ON c.customer_id = o.customer_id WHERE o.order_id IS NULL
```
Kết quả chạy, khớp hay không khớp: kết quả khớp
name  
------------------------------  
Dũng

**B3.** Tổng `amount` theo từng tên khách, **gồm cả khách chưa có đơn**.
Điều kiện đề: tính tổng amount theo từng khách tính cả các khách chưa có đơn
Kết quả mong đợi (tính tay; ghi rõ giá trị của Dũng):

| name | SUM(o.amount) |
| ---- | ------------- |
| An   | 1200          |
| Bình | 300           |
| Chí  | 1300          |
| Dũng | None          |

```sql
-- Trả lời:
SELECT c.name, SUM(o.amount) FROM customers c LEFT JOIN orders o ON c.customer_id = o.customer_id GROUP BY c.name
```
Kết quả chạy, khớp hay không khớp: kết quả khớp
name | SUM(o.amount)  
------------------------------  
An | 1200  
Bình | 300  
Chi | 1300  
Dũng | None

**B4.** Số đơn theo từng `city`, **gồm cả thành phố không có đơn nào** (dùng `COUNT(o.order_id)`).
Điều kiện đề: tính số đơn theo từng city gồm cả các thành phố không có đơn nào (dùng o.order_id)
Kết quả mong đợi (tính tay):

| City      | COUNT(o.order_id) |
| --------- | ----------------- |
| Hà Nội    | 3                 |
| Hải Phòng | 2                 |
| Đà Nẵng   | 0                 |

```sql
-- Trả lời:
SELECT c.city, COUNT(o.order_id) FROM customers c LEFT JOIN orders o ON c.customer_id = o.customer_id GROUP BY c.city
```
Kết quả chạy, khớp hay không khớp: kết quả khớp
city | COUNT(o.order_id)  
------------------------------  
Hà Nội | 3  
Hải Phòng | 2  
Đà Nẵng | 0

---

## Bài tập C: Tự giải thích (không nhìn tài liệu)

**C1.** `JOIN` và `LEFT JOIN` khác nhau ở đâu? Cho một ví dụ bằng dữ liệu hôm nay.
Trả lời: JOIN khi ghép bảng thì giữ lại các dòng dữ liệu khớp với khóa ví dụ: SELECT c.city, o.amount  FROM customers c JOIN orders o ON c.customer_id = o.customer_id chỉ in ra những thành phố Hà Nội, Hải Phòng mà không có Đà Nẵng vì Đà nẵng chính xác ra là Dũng tại Đà Nẵng không có thông tin đặt hàng trong orders trong khi đó LEFT JOIN giữ cả các dòng khớp và không khớp khóa của bảng phía bên trái LEFT JOIN, ví dụ: SELECT c.city, o.amount  FROM customers c LEFT JOIN orders o ON c.customer_id = o.customer_id thì đúng là Đà Nằng không có đơn đặt hàng nhưng giữ lại thành phố Đà Nẵng kể cả không có và giá trị amount lúc này là None 

**C2.** Vì sao `WHERE o.order_id = NULL` không trả về dòng nào, và phải viết thế nào cho đúng?
Trả lời: Vì where o.order_id =  NULL kết quả so sánh luôn là sai mà WHERE cần kết quả so sánh đúng để lọc dữ liệu vì vậy cách viết đúng là WHERE o.order_id  is NULL

---

## Phần D (mới, 60 phút): Window function

Nếu báo lỗi `near "OVER"` thì phiên bản SQLite quá cũ, dùng Colab hoặc Python 3.8 trở lên.

### Bài học
`GROUP BY` **gộp** nhiều dòng thành một dòng cho mỗi nhóm. **Window function** (hàm cửa sổ) tính trên một nhóm dòng nhưng **giữ nguyên mọi dòng**, thêm một cột kết quả.

```sql
SELECT order_id, customer_id, amount,
       SUM(amount) OVER (PARTITION BY customer_id) AS cust_total
FROM orders
ORDER BY order_id;
-- vẫn 5 dòng, mỗi dòng thêm tổng của khách đó
```
- `OVER (...)` biến hàm thành window function.
- `PARTITION BY` chia dòng thành các nhóm để tính (giống `GROUP BY` nhưng không gộp dòng).
- `ORDER BY` trong `OVER` sắp xếp trong mỗi nhóm và quyết định thứ tự tính (ví dụ tổng cộng dồn).

**Cộng dồn:** `SUM(amount) OVER (PARTITION BY customer_id ORDER BY order_date)`.

**`ROW_NUMBER()`:** đánh số thứ tự 1, 2, 3, ... trong mỗi nhóm.
```sql
SELECT order_id, customer_id, order_date,
       ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS rn
FROM orders;
```
Dòng có `rn = 1` là đơn **mới nhất** của mỗi khách.

**Lọc theo kết quả window function: phải bọc trong truy vấn con.**
```sql
SELECT * FROM (
  SELECT order_id, customer_id, order_date,
         ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS rn
  FROM orders
)
WHERE rn = 1;
-- ra order_id 2, 3, 5
```
Không viết `WHERE rn = 1` ngay trong truy vấn gốc được, vì `WHERE` chạy **trước** khi window function được tính.

### Bài tập D-A: Dự đoán đầu ra

**DA1.** Chạy `SELECT order_id, customer_id, amount, SUM(amount) OVER (PARTITION BY customer_id) AS cust_total FROM orders ORDER BY order_id;`. Có bao nhiêu dòng? Cột `cust_total` của từng dòng (theo `order_id` 1 đến 5)?
Dự đoán: có 5 dòng 

| order_id | cust_total |
| -------- | ---------- |
| 1        | 1200       |
| 2        | 1200       |
| 3        | 300        |
| 4        | 1300       |
| 5        | 1300       |

Đúng hay sai, vì sao: dự đoán đúng vì là tính tổng của các nhóm customer_id mà không gộp lại thành ra các order_id đi cùng với customer_id có thêm 1 cột cust_total với dòng giá trị tương ứng order_id

**DA2.** `SELECT order_id, customer_id, amount, SUM(amount) OVER (PARTITION BY customer_id ORDER BY order_date) AS running FROM orders ORDER BY order_id;`. Cột `running` của từng dòng (theo `order_id` 1 đến 5)?
Dự đoán:

| order_id | running |
| -------- | ------- |
| 1        | 500     |
| 2        | 1200    |
| 3        | 300     |
| 4        | 900     |
| 5        | 1300    |

Đúng hay sai, vì sao: dự đoán đúng 
nhưng tôi không rõ phải giải thích thế nào tôi thấy việc thêm ORDER BY vào khiến cho việc thay vì tổng amount của 2 order_id của cùng 1 người là giống nhau thì giờ order_id lần lượt của cùng 1 người thì hiển thị tổng cộng dồn

**DA3.** Truy vấn con có `ROW_NUMBER() ... ORDER BY order_date DESC` rồi `WHERE rn = 1` ở trên trả về những `order_id` nào?
Dự đoán: 

| order_id |
| -------- |
| 1        |
| 2        |
| 3        |

Đúng hay sai, vì sao: dự đoán đúng vì  khi nhóm theo customer_id và đánh số thứ tự các order theo order_data tăng dần thì lấy rn = 1 tức sẽ lấy những order cũ nhất của từng người dùng

### Bài tập D-B: Viết truy vấn

**DB1.** Lấy đơn **mới nhất** của mỗi khách. (Đây là câu B3 của bài chẩn đoán Tuần 0.)
Điều kiện đề: lấy đơn mới nhất của mỗi khách
Kết quả mong đợi (liệt kê `order_id`, tính tay):

| order_id |
| -------- |
| 2        |
| 3        |
| 5        |

```sql
-- Trả lời:
SELECT * FROM (SELECT order_id, customer_id, order_date, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) as rn FROM orders) WHERE rn = 1;
```
Kết quả chạy, khớp hay không khớp: kết quả khớp
order_id | customer_id | order_date | rn  
------------------------------  
2 | 1 | 2026-02-10 | 1  
3 | 2 | 2026-01-20 | 1  
5 | 3 | 2026-03-15 | 1

**DB2.** Lấy đơn có `amount` **lớn nhất** của mỗi khách.
Điều kiện đề: lấy đơn có amount lớn nhất của mỗi khách
Kết quả mong đợi (liệt kê `order_id`, tính tay):

| order_id |
| -------- |
| 2        |
| 3        |
| 4        |

```sql
-- Trả lời:
SELECT * FROM (SELECT order_id, customer_id, amount, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY amount DESC) as rn FROM orders) WHERE rn = 1;
```
Kết quả chạy, khớp hay không khớp: kết quả khớp
order_id | customer_id | amount | rn  
------------------------------  
2 | 1 | 700 | 1  
3 | 2 | 300 | 1  
4 | 3 | 900 | 1

**DB3.** Với mỗi đơn, hiển thị `order_id`, `customer_id`, `amount` và **chênh lệch** `amount` so với **trung bình amount của khách đó** (dùng `AVG(...) OVER (PARTITION BY ...)`). Sắp xếp theo `order_id`.
Điều kiện đề: với mỗi đơn hiển thị order_id, customer_id, amount và chênh lệch amount so với trung bình amount của khách đó 
Kết quả mong đợi (tính tay, cột chênh lệch của đơn 1 đến 5):

| chenhlech |
| --------- |
| 0.0       |
| -100.0    |
| 0.0       |
| 0.0       |
| 250.0     |

```sql
-- Trả lời:
SELECT order_id, customer_id, amount, AVG(amount) OVER (PARTITION BY customer_id ORDER BY order_id) - amount as chenhlech FROM orders
```
Kết quả chạy, khớp hay không khớp: kết quả khớp 
order_id | customer_id | amount | chenhlech  
------------------------------  
1 | 1 | 500 | 0.0  
2 | 1 | 700 | -100.0  
3 | 2 | 300 | 0.0  
4 | 3 | 900 | 0.0  
5 | 3 | 400 | 250.0

### Bài tập D-C: Tự giải thích

**DC1.** `GROUP BY` và window function khác nhau ở đâu? Cho một ví dụ từ dữ liệu hôm nay.
Trả lời: GROUP BY thì gom các dòng theo một nhóm và hiển thị 1 kết quả như tổng, trung bình là duy nhất 1 dòng ví dụ SELECT order_id, customer_id, amount, SUM(amount) FROM orders GROUP BY customer_id thì sẽ thấy gom các dòng lại thành 1 nhóm và chỉ 1 dòng tương ứng 1 nhóm là tổng amount, nhưng window function thì không gom các dòng lại mà chỉ nhóm lại mà vẫn giữ nguyên cấu trúc bảng dữ liệu, tức tương tự thêm 1 cột ví dụ SELECT order_id, customer_id, amount, SUM(amount) OVER (PARTITION BY customer_id) FROM orders thì kết quả chính là thêm một cột như tôi nói và giá trị cột đó lại chính là tổng amount theo từng nhóm được hiển thị tương ứng với các thành phần trong nhóm đó

**DC2.** Vì sao phải bọc `ROW_NUMBER()` trong truy vấn con rồi mới lọc `rn = 1`, mà không viết `WHERE rn = 1` trực tiếp? (Gợi ý: thứ tự `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT`.)
Trả lời: vì WHERE chạy trước SELECT tức WHERE chạy trước cả window function


---

## Ghi nhật ký
- Thời gian học thực tế:
- Câu bạn thấy mình sai nhiều khả năng nhất:
- Điều chưa hiểu:

> Làm xong gửi nguyên file này cho Claude chấm.

[^1]: 
