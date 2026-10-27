---
title: Buổi học Tuần 3 - Ngày 1 (Chủ nhật 25/10)
tags: [bai-tap, sql, xac-suat, thong-ke, logic, tuan-3]
---

# Tuần 3 - Ngày 1: Sửa nợ cổng Tuần 2, thống kê cơ bản, tư duy logic (mở đầu)

**Chia 3 khối, làm rải trong ngày (nghỉ ít nhất 1 giờ giữa các khối):**
- **KHỐI 1 (sáng, khoảng 60 phút):** sửa nợ cổng: bài học SQL (RANK, cộng dồn toàn bảng, tìm bản ghi không khớp), bài tập S1 đến S5, rồi xác suất P1 đến P3
- **KHỐI 2 (chiều, khoảng 55 phút):** thống kê: độ lệch chuẩn, sai số chuẩn, khoảng tin cậy, p-value; bài tập T1 đến T5
- **KHỐI 3 (tối, khoảng 45 phút):** logic: dãy số, suy luận điều kiện, người nói thật nói dối; bài tập L1 đến L9; nhật ký

Mệt hoặc sai liên tiếp thì dừng khối đó, ghi vào nhật ký.

**Phạm vi:** mọi câu trong file này chỉ dùng nội dung có trong **bài học cùng khối** (kèm ví dụ làm sẵn) và các buổi trước. Công thức nào mới đều nằm trong bài học, không có công thức nào xuất hiện lần đầu trong câu hỏi. Nếu bạn thấy một câu cần thứ chưa học, hoặc đề có thể hiểu theo hai cách, **ghi vào ô trả lời "ngoài phạm vi" hoặc "đề mơ hồ vì ..."**, rồi làm theo cách hiểu hợp lý nhất của bạn (ghi rõ bạn đã hiểu thế nào). Tôi chấm theo cách hiểu hợp lý đó, và lỗi đề tính cho tôi, không tính cho bạn.

> [!warning] Luật mới: kiểm đề trước khi chuyển câu
> Cổng Tuần 2 mất điểm nhiều nhất vì **trả lời sai đối tượng hoặc thiếu ý**: đề hỏi tổng thì tính trung bình, đề hỏi `P(mắc | dương tính)` thì dừng ở `P(dương tính)`, đề hỏi lần thứ 6 thì tính cả 6 lần. Từ giờ, câu nào đề có nhiều yêu cầu thì tôi cho **mỗi yêu cầu một dòng trả lời có nhãn (a), (b), (c)**. Sau khi có đáp án, làm đủ 3 việc:
> 1. Đọc lại đề từ đầu. Đếm số yêu cầu. Mỗi yêu cầu đã có một dòng trả lời chưa?
> 2. Dòng cuối của bạn có đúng là **đại lượng đề hỏi** không (không phải đại lượng trung gian)?
> 3. Có đơn vị không (nghìn đồng hay đồng, phần trăm hay số thập phân)?
>
> Luật cũ vẫn giữ: **mô phỏng hoặc chạy code là trọng tài** (lệch hơn 0,01 là có sai), **ô "mong đợi" điền trước khi chạy**, **nhật ký phải điền đủ ba ô**.

---

# KHỐI 1 (sáng): Sửa nợ cổng Tuần 2

## Bài học SQL (dữ liệu của cổng, để bạn đối chiếu)

`employees` (emp_id, name, dept_id, salary): (1, An, 10, 1900), (2, Bình, 10, 3000), (3, Chi, 20, 2500), (4, Dũng, 20, 2500), (5, Hà, 30, 4000), (6, Khoa, NULL, 1500).
`departments` (dept_id, dept_name): (10, Kỹ thuật), (20, Kinh doanh), (40, Nhân sự).

### 1. `ROW_NUMBER`, `RANK`, `DENSE_RANK` khi có hòa
Trong phòng 20, Chi và Dũng cùng lương 2500 (cao nhất phòng).
- `ROW_NUMBER()` luôn cho mỗi dòng trong nhóm **một số khác nhau**: Chi và Dũng nhận 1 và 2. Ai nhận 1 thì **không xác định** (`ORDER BY salary` không phân biệt được hai người). Nên `WHERE rn = 1` chỉ ra **1 dòng** mỗi phòng.
- `RANK()` cho các dòng hòa **cùng hạng**, rồi **nhảy cách**: hai người hạng 1, người kế tiếp sẽ là hạng 3.
- `DENSE_RANK()` cho hòa cùng hạng, **không nhảy cách**: hai người hạng 1, người kế tiếp là hạng 2.
- Muốn lấy **tất cả** người lương cao nhất mỗi phòng: dùng `RANK()` (hoặc `DENSE_RANK()`) rồi lọc hạng bằng 1, trong truy vấn con:
```sql
SELECT * FROM (
  SELECT name, dept_id, salary,
         RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rk
  FROM employees
)
WHERE rk = 1;
-- ra: Khoa, Bình, Chi, Dũng, Hà
```

### 2. Window function không cần `PARTITION BY`
- Bỏ `PARTITION BY` thì **cả bảng là một nhóm**.
- `SUM(salary) OVER ()` cho **cùng một số (tổng cả bảng) ở mọi dòng**: 15400, 15400, 15400, ...
- `SUM(salary) OVER (ORDER BY emp_id)` có `ORDER BY` trong `OVER`, nên cộng dồn **từ dòng đầu đến dòng hiện tại**: 1900, 4900, 7400, 9900, 13900, 15400.

### 3. Tìm bản ghi không khớp (anti-join): kiểm tra cột của bảng **phải**
```sql
SELECT d.dept_name
FROM departments d
LEFT JOIN employees e ON d.dept_id = e.dept_id
WHERE e.emp_id IS NULL;      -- ra: Nhân sự
```
Với dòng không khớp, **mọi cột của bảng phải** (ở đây `e`) là `NULL`. Cột của bảng **trái** (`d.dept_name`) có giá trị thật nên không bao giờ `NULL`. Viết `WHERE d.dept_name IS NULL` sẽ không ra dòng nào (đó chính là lỗi ở câu A4 của cổng).

## Bài tập SQL (dữ liệu mới)
```
products(product_id, name):     (1, Bút), (2, Vở), (3, Thước), (4, Tẩy)
sales(sale_id, product_id, qty): (1, 1, 10), (2, 1, 5), (3, 2, 7), (4, 3, 2)
scores(student, class, score):  (An, A, 8), (Bình, A, 9), (Chi, A, 9), (Dũng, B, 7), (Hà, B, 6)
```
Chạy bằng `sql_setup_portable.py` hoặc Colab/SQLite. Dự đoán **trước** khi chạy.

**S1.** `ROW_NUMBER() OVER (PARTITION BY class ORDER BY score DESC) AS rn` trên bảng `scores`.
(a) Có bao nhiêu dòng có `rn = 1`?
(b) Trong lớp A, ai nhận `rn = 1` (Bình, Chi hay không xác định)? Vì sao?
(c) Với `RANK()` cùng cách chia nhóm và sắp xếp, hạng (`rk`) của An là mấy? Có bao nhiêu dòng có `rk = 1`?
(d) Với `DENSE_RANK()`, hạng của An là mấy?
Dự đoán (a): có 2 dòng
Dự đoán (b): lớp A Bình nhận rn = 1 không xác định được lí do có thể khi sắp xếp thì xếp Bình lên trước
Dự đoán (c): hạng của An là 3 và có 3 dòng có rk là 1
Dự đoán (d): với DENSE_RANK hạng của An là 2
Chạy, khớp hay không khớp (từng ô): kết quả chạy khớp
student | class | rn  
------------------------------  
Bình | A | 1  
Dũng | B | 1

student | class | rk  
------------------------------  
Bình | A | 1  
Chi | A | 1  
An | A | 3  
Dũng | B | 1  
Hà | B | 2

student | class | rk  
------------------------------  
Bình | A | 1  
Chi | A | 1  
An | A | 2  
Dũng | B | 1  
Hà | B | 2

**S2.** Lấy **tất cả** học sinh có điểm **cao nhất trong lớp của mình** (hòa thì lấy cả hai).
Điều kiện đề (chép nguyên văn): lấy tất cả học sinh có điểm cao nhất trong lớp mình (hòa thì lấy cả hai)
Kết quả mong đợi (tên, tính tay):

| student | class | score | rk  |
| ------- | ----- | ----- | --- |
| Bình    | A     | 9     | 1   |
| Chí     | A     | 9     | 1   |
| Dũng    | B     | 7     | 1   |


```sql
-- Trả lời:
SELECT * FROM (SELECT student, class, score, RANK() OVER (PARTITION BY class ORDER BY score DESC) AS rk FROM scores) WHERE rk = 1
```
Kết quả chạy, khớp hay không khớp: kết quả khớp
student | class | score | rk  
------------------------------  
Bình | A | 9 | 1  
Chi | A | 9 | 1  
Dũng | B | 7 | 1

**S3.**
(a) `SELECT sale_id, qty, SUM(qty) OVER (ORDER BY sale_id) AS c FROM sales;` cột `c` của từng dòng (sale_id 1 đến 4)?
(b) `SELECT sale_id, qty, SUM(qty) OVER () AS c FROM sales;` cột `c` của từng dòng?
(c) Vì sao hai kết quả khác nhau? (Một câu, nhắc đến `ORDER BY` trong `OVER`.)
Dự đoán (a):

| c   |
| --- |
| 10  |
| 15  |
| 22  |
| 24  |

Dự đoán (b):

| c   |
| --- |
| 24  |
| 24  |
| 24  |
| 24  |

(c): phần a có order by mà không có participate by nên OVER sẽ cộng dồn lần lượt qty tất cả các hàng trong bảng, phần b không có paricipate by lẫn order by lên OVER toàn bộ qty trong bảng 
Chạy, khớp hay không khớp: kết quả khớp 
sale_id | qty | c  
------------------------------  
1 | 10 | 10  
2 | 5 | 15  
3 | 7 | 22  
4 | 2 | 24

sale_id | qty | c  
------------------------------  
1 | 10 | 24  
2 | 5 | 24  
3 | 7 | 24  
4 | 2 | 24

**S4.** Liệt kê tên các sản phẩm **chưa bán lần nào**.
Điều kiện đề (chép nguyên văn): liệt kê tên các sản phẩm chưa bán lần nào
Kết quả mong đợi (tính tay):

| name |
| ---- |
| Tẩy  |

```sql
-- Trả lời:
SELECT p.name FROM products p LEFT JOIN sales s on s.product_id = p.product_id WHERE s.product_id IS NULL
```
Kết quả chạy, khớp hay không khớp: kết quả khớp
name  
------------------------------  
Tẩy

**S5.** (Tự giải thích) Với S4, vì sao `WHERE p.name IS NULL` (p là bảng `products`, bên trái) **không** cho ra sản phẩm nào? Cột nào mới đúng, và vì sao?
Trả lời: vì p.name tức là tên sản phẩm mà dùng left join thì không có tên sản phẩm trong bảng products nào là NULL cả tức luôn có giá trị mà WHERE kiểm tra như trên không tìm ra nên mới trả về không sản phẩm nào. Cột đúng phải là s.product_id tức cột product_id trong bảng sales vì bảng đó khi nối 2 bảng lại thì dòng nào không cùng product_id có trong sales thì là NULL khi đó WHERE mới tìm ra được

## Xác suất: sửa nợ cổng

**P1.** Hộp có 3 bi đỏ và 7 bi xanh. Rút hai bi liên tiếp, **không hoàn lại**.
(a) Công thức cho `P(cả hai đỏ)` (**dạng đúng cho hai biến cố phụ thuộc**): P(cả hai đỏ) = P(bi 1 đỏ) * P(bi 2 đỏ | bi 1 đỏ)
(b) Trước khi nhân, ghi: lần 1 còn 10 bi (3 đỏ); lần 2 còn 9 bi (2 đỏ).
(c) `P(cả hai đỏ)`: phân số 1/15 , thập phân 3 chữ số 0.067
(d) `P(bi thứ hai đỏ | bi thứ nhất đỏ)`: phân số 2/9 , thập phân 3 chữ số 0.222

**P2.** Một bệnh có 5% dân số mắc. Xét nghiệm có độ nhạy 90% và độ đặc hiệu 80%. Một người xét nghiệm **dương tính**.
(a) `P(dương tính | mắc)` = 0.9 ; `P(dương tính | không mắc)` = 0.2
(b) `P(dương tính)`: công thức, rồi số: P(dương tính) = P(dương tính | mắc) * P (mắc) + P(dương tính | không mắc) * P(không mắc) = 0.9 * 0.05 + 0.2 * 0.95 = 0.235
(c) **Đáp án cuối (đại lượng đề hỏi): xác suất người đó thật sự mắc**, tức `P(mắc | dương tính)`: công thức, rồi số (3 chữ số thập phân): P(mắc | dương tính) = P(dương tính | mắc) * P(mắc) / P(dương tính) = 0.9 * 0.05 / 0.235 = 0.191
(d) Kiểm tra bằng tần suất tự nhiên (10.000 người): 500 người mắc, 450 trong đó dương tính; 9500 người không mắc, 1900 trong đó dương tính; tổng dương tính 2350 ; tỉ lệ mắc thật trong số dương tính = 450 / 2350 = 0.191

**P3.** Một xúc xắc công bằng. Bạn vừa thấy nó ra 6 hai lần liên tiếp.
(a) Xác suất lần thứ 3 ra 6 là bao nhiêu?
(b) Xác suất cả ba lần đều ra 6, tính **trước khi tung lần nào**?
(c) Xác suất cả ba lần đều ra 6, **biết hai lần đầu đã ra 6**?
(d) Một câu: vì sao (a) và (c) bằng nhau mà (b) nhỏ hơn rất nhiều? (Dùng từ "độc lập".)
(a): Xác xuất ra 6 lần thứ 3 là 1/6
(b): Chỗ này hỏi không rõ
(c): 1/216
(d): 

---

# KHỐI 2 (chiều): Thống kê cơ bản

## Bài học

### 1. Trung bình, trung vị, độ lệch chuẩn
- **Trung bình** (mean): tổng chia số phần tử. **Trung vị** (median): giá trị ở giữa khi đã sắp xếp. Dữ liệu có **ngoại lai** (giá trị rất lớn hoặc rất nhỏ) thì trung bình bị kéo lệch, trung vị ít bị ảnh hưởng.
- **Độ lệch chuẩn**: đo độ phân tán quanh trung bình. Cách tính: (1) trung bình; (2) lệch của từng giá trị so với trung bình, bình phương; (3) cộng lại; (4) chia cho **n** (nếu dữ liệu là toàn bộ tập, độ lệch chuẩn tổng thể) hoặc chia cho **n − 1** (nếu dữ liệu là một **mẫu** lấy từ tập lớn hơn, độ lệch chuẩn mẫu); (5) lấy căn bậc hai.
- **Bẫy trong code:** `np.std(x)` mặc định chia cho **n** (`ddof=0`). `pd.Series(x).std()` mặc định chia cho **n − 1** (`ddof=1`). Hai hàm cùng tên khác kết quả.

**Ví dụ làm sẵn:** `x = [2, 4, 4, 4, 5, 5, 7, 9]`, `n = 8`. Trung bình = 5. Bình phương lệch: 9, 1, 1, 1, 0, 0, 4, 16, tổng = 32. Chia n: phương sai = 4, độ lệch chuẩn = 2 (đúng với `np.std(x)`). Chia n − 1: phương sai = 32/7 ≈ 4,571, độ lệch chuẩn ≈ 2,138 (đúng với `pd.Series(x).std()` và `np.std(x, ddof=1)`).

### 1b. Độ lệch chuẩn khi đã biết xác suất (số lần ngửa trong n lần tung)

Có **hai cách tính độ lệch chuẩn**, cùng đo mức dao động nhưng dùng thông tin khác nhau:

- **Từ dữ liệu đã quan sát** (mục 1): có một dãy số thật, tính trung bình, bình phương lệch, chia `n − 1`, lấy căn.
- **Từ xác suất đã biết** (mục này): chưa cần dữ liệu, chỉ cần biết quy luật. Dùng khi giả sử H0 (ví dụ: đồng xu công bằng).

**Công thức cho "đếm số lần ngửa trong n lần tung độc lập, mỗi lần ngửa với xác suất p":**

- Kỳ vọng: `n × p`. Độ lệch chuẩn: `√(n × p × (1 − p))`.

**Vì sao ra công thức đó (dùng đúng `Var = E[X²] − (E[X])²` bạn đã học):**

1. Một lần tung: `X = 1` (ngửa) với xác suất 0,5, `X = 0` (úp) với xác suất 0,5. `E[X] = 0,5`, `E[X²] = 1² × 0,5 + 0² × 0,5 = 0,5`. `Var(X) = 0,5 − 0,5² = 0,25`. Tổng quát là `p(1 − p) = 0,5 × 0,5 = 0,25`.
2. 100 lần tung **độc lập**: số ngửa là tổng 100 biến như vậy, và với biến độc lập thì **phương sai cộng lại**: `Var = 100 × 0,25 = 25`.
3. Độ lệch chuẩn là căn bậc hai của phương sai: `√25 = 5`.

**Kiểm chứng bằng mô phỏng:** tung "100 lần" rất nhiều lần, ghi số ngửa mỗi lần, rồi tính độ lệch chuẩn của các số đó ra khoảng 5:

python

```python
import numpy as np
rng = np.random.default_rng(0)
heads = rng.binomial(100, 0.5, size=200000)
print(heads.mean(), heads.std())     # khoảng 50 và 5
```
### 2. Sai số chuẩn (standard error) và khoảng tin cậy
- **Sai số chuẩn của trung bình mẫu:** `SE = s / √n`, với `s` là độ lệch chuẩn mẫu và `n` là cỡ mẫu. SE cho biết trung bình mẫu **dao động bao nhiêu** giữa các mẫu khác nhau. **Cỡ mẫu tăng 4 lần thì SE giảm một nửa** (vì căn bậc hai của 4 là 2).
- **Khoảng tin cậy 95% (xấp xỉ):** `trung bình ± 1,96 × SE`.
- Cách đọc đúng: nếu **lặp lại việc lấy mẫu và tính khoảng nhiều lần**, khoảng thu được sẽ chứa trung bình thật khoảng 95% số lần. Khoảng hẹp hơn khi n lớn hơn hoặc s nhỏ hơn.

**Ví dụ làm sẵn:** `n = 100`, trung bình mẫu = 70, `s = 15`. `SE = 15/√100 = 1,5`. Khoảng 95%: `70 ± 1,96 × 1,5` = `[67,06; 72,94]`.

### 3. Kiểm định và p-value

**Ý tưởng, như một phiên tòa.** Bạn nghi một đồng xu bị lệch. Quy tắc ban đầu: coi như nó **công bằng** (giống "bị cáo vô tội cho đến khi chứng minh được ngược lại"). Giả định này gọi là **H0 (giả thuyết không)**: "không có gì đặc biệt". Bạn thu thập dữ liệu (tung 100 lần), rồi hỏi: **nếu H0 đúng thì thấy dữ liệu lệch như thế này có hiếm không?** Hiếm quá thì nghi H0 sai và **bác bỏ** nó. Không hiếm thì **chưa đủ bằng chứng** để bác bỏ.

**Làm từng bước với ví dụ làm sẵn:** tung một đồng xu 100 lần, được **62 ngửa**. H0: đồng xu công bằng (xác suất ngửa 0,5).

1. **Dưới H0, số ngửa dao động quanh đâu?** Kỳ vọng `100 × 0,5 = 50`. Mức dao động điển hình (độ lệch chuẩn) `√(100 × 0,5 × 0,5) = 5`. Nghĩa là 45 đến 55 ngửa là rất bình thường, 40 đến 60 là khoảng ±2 độ lệch chuẩn (còn khá bình thường).
2. **Kết quả của bạn lệch bao nhiêu "độ lệch chuẩn"?** `z = (giá trị quan sát − giá trị kỳ vọng dưới H0) / độ lệch chuẩn dưới H0 = (62 − 50)/5 = 2,4`. Tức là lệch 2,4 lần mức dao động điển hình.
3. **p-value là gì?** "Cực đoan bằng hoặc hơn" ở đây nghĩa là **lệch ít nhất 12 so với 50**: từ 62 ngửa trở lên **hoặc** từ 38 ngửa trở xuống (hai phía, vì 38 cũng đáng ngờ như 62: đồng xu có thể lệch về mặt úp). **p = xác suất, giả sử H0 đúng, thu được kết quả cực đoan như vậy.** Tra bảng (`|z| = 2,4` nằm giữa 2 và 2,58) thì p nằm giữa 0,0455 và 0,01; giá trị xấp xỉ **p ≈ 0,016**.
4. **Kết luận:** `0,016 < 0,05`, nên **bác bỏ H0 ở mức 5%**: nếu đồng xu công bằng thì kết quả lệch như thế chỉ xảy ra khoảng 1,6% số lần làm thí nghiệm.

**Hiểu đúng p-value (hay bị nhầm):**

- `p = 0,016` **không** có nghĩa "xác suất đồng xu công bằng là 1,6%". Nó là xác suất của **dữ liệu** nếu H0 đúng, không phải xác suất của H0.
- Nghĩa đúng theo cách đếm: tưởng tượng làm thí nghiệm "tung 100 lần" 1000 lần với một đồng xu **thật sự công bằng**, khoảng 16 lần (đến 21 lần nếu tính chính xác) sẽ ra kết quả lệch từ 12 trở lên.
- **`p < 0,05` thì bác bỏ H0.** Con số 0,05 là **quy ước** (mức ý nghĩa 5%), không phải quy luật tự nhiên.
- **`p ≥ 0,05` thì chưa đủ bằng chứng bác bỏ H0**, **không** chứng minh H0 đúng (như tòa không đủ bằng chứng kết tội không có nghĩa là bị cáo chắc chắn vô tội).

**Bảng điểm z (p hai phía, xấp xỉ chuẩn):**

| `   | z     | `    | 1    | 1,645  | 1,96 | 2      | 2,58 | 3   |
| --- | ----- | ---- | ---- | ------ | ---- | ------ | ---- | --- |
| p   | 0,317 | 0,10 | 0,05 | 0,0455 | 0,01 | 0,0027 |      |     |


Đọc bảng: `|z|` càng lớn thì p càng nhỏ. `|z| ≥ 1,96` tương ứng `p ≤ 0,05`.

**Lưu ý về độ chính xác:** bảng này là **xấp xỉ** (đếm số ngửa là số nguyên, còn bảng dùng phân phối liên tục). Với kết quả sát ngưỡng thì kết luận có thể đổi. Ví dụ 60 ngửa: bảng cho `z = 2`, p ≈ 0,0455 (bác bỏ), nhưng tính chính xác ra khoảng 0,057 (chưa bác bỏ). Vì vậy các đề của buổi này không đặt sát ngưỡng.

**Kiểm chứng bằng mô phỏng (tùy chọn, làm sau khi đã tính tay):**

python

```python
import numpy as np
rng = np.random.default_rng(0)
heads = rng.binomial(100, 0.5, size=200000)   # 200.000 lần "tung 100 lần", đồng xu công bằng
print((np.abs(heads - 50) >= 12).mean())      # tỉ lệ thí nghiệm lệch từ 12 trở lên so với 50
```

Kết quả sẽ khoảng 0,021 (tính chính xác), gần với 0,016 của bảng (hiệu dưới 0,01, do xấp xỉ).

## Bài tập

**T1.** `y = [3, 5, 7, 9, 11]`.
(a) Trung bình: 7
(b) Tổng bình phương lệch: 40
(c) Độ lệch chuẩn **mẫu** (chia n − 1), 3 chữ số thập phân (công thức, rồi số): độ lệch chuẩn mẫu = √(tổng bình phương lệch / (n - 1)) = 2.582
(d) Độ lệch chuẩn **tổng thể** (chia n), 3 chữ số thập phân: độ lệch chuẩn tổng thể = √(tổng bình phương lệch / n) = 2.39
(e) `np.std(y)` ra số nào trong (c) hoặc (d)? `pd.Series(y).std()` ra số nào? Vì sao hai hàm khác nhau? np.std(y) ra kết quả như d vì đây là tính độ lệch chuẩn tổng thể tức dữ liệu là toàn bộ tập, pd.Series(y).std() thì ra kết quả như c vì đây chỉ là độ lệch chuẩn mẫu tức chỉ là một mẫu dữ liệu trong toàn bộ tập

**T2.** `z = [1, 2, 3, 4, 100]`.
(a) Trung bình và trung vị: trung bình = 22, trung vị = 3
(b) Giá trị nào ít bị giá trị 100 làm lệch hơn? Vì sao? giá trị ít bị giá trị 100 làm lệch là trung vị vì trung vị lấy giá trị nằm chính giữa khi dãy là một dãy đã sắp xếp trong khi đó trung bình thì bị 100 kéo lệch vì cộng tất cả giá trị rồi chia số lượng phần tử tức 100 khiến tổng cao bất thường khiến trung bình bị kéo lệch

**T3.** Một mẫu có `n = 64`, trung bình = 50, `s = 16`.
(a) `SE` (công thức, rồi số): SE = s / √n = 16 / √64 = 2
(b) Khoảng tin cậy 95% (công thức, rồi hai đầu mút): trung bình +- 1.96 * SE => [46.08,53.92]
(c) Nếu cỡ mẫu tăng lên `n = 256` (s giữ nguyên), `SE` mới là bao nhiêu? Khoảng tin cậy rộng hơn hay hẹp hơn? SE mới = 1 và khoảng tin cậy = [48.04,51.96] hẹp đi 

**T4.** Tung một đồng xu 100 lần, được **64 ngửa**. H0: đồng xu công bằng.
(a) Số ngửa kỳ vọng dưới H0 và độ lệch chuẩn dưới H0: 
(b) `z`: 
(c) p hai phía thuộc khoảng nào so với 0,05 và 0,01 (dùng bảng)? 
(d) Kết luận ở mức 5% bằng một câu (nhớ: **không** nói "xác suất H0 đúng").
Trả lời (a): số ngửa kỳ vọng là 50, độ lệch chuẩn là 5
(b): (62 - 50) / 5 = 2.4
(c): không rõ quá lú
(d): không rõ quá lú

**T5.** Mỗi mệnh đề dưới đây **đúng hay sai**? Giải thích **một câu** cho mỗi mệnh đề.
(a) "p = 0,03 nghĩa là xác suất H0 đúng là 3%."
(b) "p = 0,03 nghĩa là: nếu H0 đúng thì xác suất thấy dữ liệu cực đoan bằng hoặc hơn dữ liệu quan sát là 3%."
(c) "p = 0,20 chứng minh H0 đúng."
(a):
(b):
(c):

---

# KHỐI 3 (tối): Tư duy logic (mở đầu)

## Bài học

### 1. Dãy số
Cách làm theo thứ tự: (1) tính **hiệu** hai số liền nhau; (2) nếu hiệu không đổi, đó là cấp số cộng; (3) nếu không, tính **tỉ số** hai số liền nhau (không đổi thì là cấp số nhân); (4) nếu hiệu không đổi nhưng thay đổi đều, tính **hiệu của hiệu**; (5) thử các dãy quen: số chính phương (1, 4, 9, 16, ...), lập phương, số kế tiếp bằng tổng hai số trước. **Luôn kiểm lại quy luật bằng ít nhất một số hạng đã cho.**

**Ví dụ làm sẵn:**
- `7, 11, 15, 19, ?`: hiệu luôn 4, số tiếp theo là 23.
- `3, 9, 27, 81, ?`: tỉ số luôn 3, số tiếp theo là 243.
- `2, 6, 12, 20, 30, ?`: hiệu 4, 6, 8, 10 (hiệu của hiệu luôn 2), hiệu tiếp theo 12, số tiếp theo là 42.

### 2. Suy luận có điều kiện
Mệnh đề **"Nếu P thì Q"**:
- **Phản đảo** "Nếu **không** Q thì **không** P" là **tương đương** với mệnh đề gốc (luôn cùng đúng hoặc cùng sai).
- **Đảo** "Nếu Q thì P" **không** tương đương. Biết Q xảy ra thì **không** kết luận được P.

**Ví dụ làm sẵn:** "Nếu một số chia hết cho 4 thì nó là số chẵn."
- Phản đảo: "Nếu một số là số lẻ thì nó không chia hết cho 4." Đúng.
- Đảo: "Nếu một số chẵn thì nó chia hết cho 4." Sai (ví dụ 6).
- Biết "số này chia hết cho 4" thì kết luận được "số này chẵn". Biết "số này chẵn" thì **không** kết luận được "chia hết cho 4".

### 3. Người luôn nói thật và người luôn nói dối
Mỗi người **luôn nói thật** hoặc **luôn nói dối**. Cách làm: **giả sử** một người nói thật (hoặc nói dối), suy ra hệ quả, và **tìm mâu thuẫn**.

**Ví dụ làm sẵn:** An và Bình. An nói: "Cả hai chúng tôi đều nói dối."
- Giả sử An nói thật: câu của An đúng, nên cả hai nói dối, trong đó có An. Mâu thuẫn (An vừa nói thật vừa nói dối).
- Vậy An nói dối. Câu của An sai, tức "cả hai đều nói dối" sai, nên ít nhất một người nói thật. An nói dối nên **Bình nói thật**.
- Kiểm tra: An (nói dối) nói câu sai, đúng. Bình nói thật, Bình không nói gì. Hợp lệ.

## Bài tập

**L1.** `5, 8, 11, 14, ?` Số tiếp theo và quy luật (một câu): số tiếp theo: 17 quy luật: hiệu 2 số liền nhau là 3
**L2.** `3, 6, 12, 24, ?` Số tiếp theo và quy luật: số tiếp theo: 48, quy luật: tỉ số 2 số liền nhau là 2
**L3.** `1, 2, 4, 7, 11, ?` Số tiếp theo và quy luật: số tiếp theo: 16 quy luật hiệu của hiệu là 1
**L4.** `2, 3, 5, 8, 13, ?` Số tiếp theo và quy luật: số tiếp theo là 21 quy luật là số tiếp theo bằng tổng 2 số trước đó
**L5.** `1, 8, 27, 64, ?` Số tiếp theo và quy luật: số tiếp theo là 125 quy luật là mũ số 3 lần lượt 

**L6.** Cho mệnh đề: "Nếu trời mưa thì đường ướt." Đêm nay người ta thấy **đường không ướt**. Kết luận được gì về trời mưa? Vì sao?
Trả lời: đêm nay đường không ướt thì trời không mưa vì nếu trời mưa thì đường ướt vậy trái ngược lại nếu trời không mưa thì đường không uotcws

**L7.** Cùng mệnh đề ở L6. Sáng nay người ta thấy **đường ướt**. Kết luận được trời đã mưa không? Cho **một** lý do khác khiến đường ướt.
Trả lời: nếu sáng thấy đường ướt thì không kết luận là đêm qua trời mua được vì đường ướt có thể đêm qua xe chở nước làm đổ nước ra đường hoặc đêm qua có lễ hội người ta làm đường ướt

**L8.** "Mọi học viên đạt cổng thì được sang tuần sau." Lan **không** được sang tuần sau. Kết luận gì về Lan? Rồi: Minh **được** sang tuần sau, kết luận được Minh đã đạt cổng không? (Hai câu trả lời, mỗi câu một lý do.)
(a) Về Lan: Lan không được sang tuần sau vì chưa đạt điểm để qua cổng
(b) Về Minh: Minh được sang tuần sau vì đã đủ điểm hoặc là qua mức điểm mà bài kiểm tra đề ra để sang tuần sau

**L9.** Ba người An, Bình, Chi. Mỗi người luôn nói thật hoặc luôn nói dối.
- An nói: "Bình nói dối."
- Bình nói: "Chi nói dối."
- Chi nói: "An và Bình đều nói dối."
Ai nói thật, ai nói dối? Trình bày: giả sử gì, suy ra gì, mâu thuẫn ở đâu, và **kiểm tra lại cả ba câu nói** với kết quả của bạn.
Trả lời: giả sử An nói thật mà An nói bình nói dối  tức  bình thực sự nói dối, mà bình nói chi nói dối là sai tức chi nói thật mà chi nói an và bình đều nói dối tức An thực sự nói dối và Bình thực sự nói dối => mâu thuẫn ở An vừa nói dối vừa nói thật
Giả sử An nói dối mà An nói bình nói dối tức bình thực sự nói thật, mà Bình nói Chi nói dối tức chi thực sự nói dối mà Chi nói An và Bình đều nói dối là sai tức An hoặc Bình nói dối mà biết An nói dối và Bình nói thật
Kết luận An thực sự nói dối, Bình thực sự nói thật, Chi thực sự nói dối

---

## Ghi nhật ký (bắt buộc điền cả ba)
- Thời gian học thực tế (từng khối): sáng: 10h45-11h45, chiều 14h30 - 15h30, tối 16h30 - 17h45
- Câu bạn nghĩ mình sai nhiều nhất, và vì sao: các câu tư duy vì các câu này giải thích rất khó và dễ sai
- Điều chưa hiểu cần hỏi Claude (nếu viết "không có" thì phải nêu đã kiểm tra những câu nào bằng cách chạy code hoặc đối chiếu): tại bài xác suất P1 này ghi lần 1 còn tức là đây là trước khi lấy lần 1 còn lại bao nhiêu bi hay là sau khi lấy lần 1 còn lại bao nhiêu bi, nếu như là trước khi lấy lần 1 thì phần b coi là đúng, nếu là ý sau khi lấy lần 1 thì lần sau lên sửa lại ý chỉ đúng 1 nghĩa. bài xắc suất P3 b c d đề ghi không rõ ý không hiểu hỏi gì thậm trí còn lộ trước đáp án hoặc gây nhiễu không lí do gì T4, T5 nằm trong phần lí thuyết quá khó hiểu nên chưa làm được cần giao lại

> Làm xong gửi nguyên file này cho Claude chấm. **Thứ Hai 26/10:** sửa nợ phần còn lại, hàm/lambda/sorted/slicing, và một bài kiểm tra nhanh 30 phút (SQL window/NULL/join, xác suất có điều kiện, kiểm đề). Kết quả bài đó quyết định bạn qua hẳn Tuần 3 hay ở lại sửa.
