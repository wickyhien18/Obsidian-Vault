---
title: Chủ Nhật 1/11 - Bài tổng hợp 90 phút (tập dượt thi)
tags: [tong-hop, tap-dot-thi, tuan-3]
---

# Bài tổng hợp 1/11 (90 phút, 100 điểm)

Đây là buổi **tập dượt thi**: trộn mọi mảng đã học (SQL, Python và pandas, xác suất, thống kê, ML và đại số tuyến tính). Kết quả giúp mình biết bạn đang ở đâu trước đề thi thử nhỏ 10-11/11.

## Luật làm bài
- **Làm một lần liền 90 phút, bấm giờ, không tra cứu, không hỏi AI.** Giấy nháp được dùng. Hết giờ thì dừng. Ghi **giờ bắt đầu và giờ kết thúc thật**.
- **Code:** các câu code chỉ chạy để **kiểm sau khi đã viết đáp án và ô "kết quả mong đợi" (tính tay)**. Ô nào để trống thì tính nửa điểm.
- Câu xác suất, thống kê: viết **công thức** rồi mới thay số. Làm tròn bình thường, lệch ≤ 0,001 vẫn đúng.
- **Kiểm đề sau mỗi câu (10 giây):** đếm số ý đề hỏi; dòng cuối có đúng là đại lượng đề hỏi chưa; có đơn vị chưa; với khoảng tin cậy: đầu trái nhỏ hơn đầu phải và trung bình nằm giữa.
- Đề mơ hồ thì ghi "đề mơ hồ vì ..." vào ô trả lời, làm theo cách hiểu hợp lý, mình chấm theo cách đó.
- Cuối bài **nhật ký phải điền đủ ba ô bằng câu hoàn chỉnh**. Ô 2: nêu câu bạn nghĩ sai nhiều khả năng nhất và vì sao (không viết "không có"). Ô 3: nêu những câu bạn đã kiểm bằng cách chạy code.

---

## A. SQL (28 điểm, 4 câu, mỗi câu 7 điểm)
Dữ liệu:
```
reps(rep_id, name, region): (1, Hà, Bắc), (2, Khoa, Nam), (3, Lan, Bắc), (4, Minh, Trung), (5, Nga, Nam)
deals(deal_id, rep_id, amount, month):
  (1,1,100,1), (2,1,150,2), (3,2,80,1), (4,3,120,1), (5,3,60,2), (6,3,90,3), (7,2,200,3), (8,NULL,50,2)
```
(Giao dịch số 8 chưa gán cho nhân viên nào. Minh và Nga chưa có giao dịch.)

**A1.** (a) `SELECT COUNT(*) FROM deals;` (b) `SELECT COUNT(rep_id) FROM deals;` (c) `SELECT AVG(amount) FROM deals;` (công thức, rồi số)
(a): 8
(b): 7
(c): 106.25

**A2.** Liệt kê tên các nhân viên **chưa có giao dịch nào**.
Điều kiện đề (chép nguyên văn): liệt kê tên các nhân viên chưa có giao dịch nào
Kết quả mong đợi (tính tay): 

| name |
| ---- |
| Minh |
| Nga  |

```sql
-- Trả lời:
SELECT r.name FROM reps r LEFT JOIN deals d ON r.rep_id = d.rep_id WHERE d.rep_id IS NULL
```

**A3.** Với mỗi `region`, tính **tổng `amount`** (chỉ tính các giao dịch đã gán cho nhân viên). Chỉ giữ vùng có tổng **lớn hơn 300**, sắp xếp theo tổng **giảm dần**. Hiển thị `region` và tổng.
Điều kiện đề (chép nguyên văn): vối mỗi region tính tổng amount (chỉ tính các giao dịch đã gán cho nhân viên). Chỉ giữ vùng có tổng lớn hơn 300, sắp xếp theo tổng giảm dần. Hiển thị region và tổng
Kết quả mong đợi (tính tay):

| region | SUM(d.amount) |
| ------ | ------------- |
| Bắc    | 520           |

```sql
-- Trả lời:
SELECT r.region, SUM(d.amount) FROM reps r JOIN deals d ON r.rep_id = d.rep_id GROUP BY r.region HAVING SUM(d.amount) > 300 ORDER BY SUM(d.amount) DESC;
```

**A4.** (a) Viết câu `SELECT` cho cột `c` là tổng `amount` **cộng dồn toàn bảng** theo thứ tự `deal_id` (hiển thị `deal_id`, `amount`, `c`). Kết quả mong đợi: cột `c` của 8 dòng.
(b) Chỉ xét các giao dịch của nhân viên 3 (deal 4, 5, 6): với `RANK() OVER (PARTITION BY rep_id ORDER BY amount DESC)`, hạng của deal 4, 5, 6 lần lượt là bao nhiêu?
(a) Kết quả mong đợi: 

| c   |
| --- |
| 100 |
| 250 |
| 330 |
| 450 |
| 510 |
| 600 |
| 800 |
| 850 |

```sql
-- Trả lời (a):
SELECT deal_id, amount, SUM(amount) OVER (ORDER BY deal_id) AS c FROM deals 
```
(b): hạng của deal 4, 5, 6 lần lượt là 1,3,2

---

## B. Python và pandas (24 điểm, 4 câu, mỗi câu 6 điểm)
Dữ liệu dùng chung:
```python
import pandas as pd
df = pd.DataFrame({
    "city":  ["HN", "HCM", "HN", "DN", "HCM", "HN"],
    "qty":   [3, 5, 2, 4, 1, 6],
    "price": [10, 20, 10, 35, 20, 10],
})
```
**B1.** Thêm cột `revenue = qty × price` **không dùng vòng lặp**.
Điều kiện đề (chép nguyên văn): thêm cột revenue = qty * price mà không dùng vòng lặp
Kết quả mong đợi (cột `revenue` của 6 dòng, tính tay):

| revenue |
| ------- |
| 30      |
| 100     |
| 20      |
| 140     |
| 20      |
| 60      |

```python
# Trả lời:
df["revenue"] = df["qty"] * df["price"]

print(df["revenue"])
```

**B2.** Tính **tổng `revenue` theo `city`**, sắp xếp **giảm dần**.
Điều kiện đề (chép nguyên văn): tính tổng revenue theo city, sắp xếp giảm dần
Kết quả mong đợi (tính tay, mỗi thành phố một dòng):

| city |     |
| ---- | --- |
| DN   | 140 |
| HCM  | 120 |
| HN   | 110 |


```python
# Trả lời:
df.groupby("city")["revenue"].sum().sort_values(ascending=False)
```

**B3.** Lọc các dòng có `revenue` **lớn hơn 50** **và** `city` bằng `"HN"`.
Điều kiện đề (chép nguyên văn): lọc các dòng có revenue lớn hơn 50 và city bằng HN
Kết quả mong đợi (tính tay: có bao nhiêu dòng, dòng nào):
Có 1 dòng và dòng đó là dòng HN với revenue là 60

|     | city | revenue |
| --- | ---- | ------- |
| 5   | HN   | 60      |

```python
# Trả lời:
df[(df["revenue"] > 50) & (df["city"] == "HN")]
```

**B4.** Cho `hs = [("An", 8), ("Bình", 9), ("Chi", 7), ("Dũng", 6)]`. Viết hàm `top2(hs)` **trả về** hai học sinh (cả tuple) có điểm cao nhất, điểm cao đứng trước, bằng `sorted` với `key=lambda` rồi slicing.
Điều kiện đề (chép nguyên văn): viết hàm top2(hs) trả về hai học sinh (cả tuple) có điểm cao nhất, điểm cao đứng tước, bằng sorted với key=lamda rồi slicing
Kết quả mong đợi của `top2(hs)` (tính tay):

| Bình | 9   |
| ---- | --- |
| An   | 8   |

```python
# Trả lời:
def top2(hs):
    return sorted(hs, key=lambda hs: hs[1:], reverse=True)[:2]
```

---

## C. Xác suất và thống kê (32 điểm)

**C1. (10 điểm)** Một nhà máy có hai dây chuyền. Dây chuyền A làm 30% sản phẩm, tỉ lệ lỗi 2%. Dây chuyền B làm 70% sản phẩm, tỉ lệ lỗi 4%. Lấy ngẫu nhiên một sản phẩm và thấy nó **lỗi**.
(a) `P(lỗi)`: công thức, rồi số: P(lỗi) = P(lỗi | A) * P(A) + P(lỗi | B) * P(B) = 0.034
(b) **Đại lượng đề hỏi: xác suất sản phẩm đó do dây chuyền A làm**: công thức, rồi số (3 chữ số thập phân): P(A | lỗi) = P(lỗi | A) * P(A) / P(lỗi) = 0.176
(c) Kiểm bằng tần suất tự nhiên (10.000 sản phẩm): 3000 do A, 60 lỗi; 7000 do B, 280 lỗi; tỉ lệ = 60 / 340 = 0.176

**C2. (6 điểm)** Hộp có 7 bi trắng và 3 bi đen, rút hai bi liên tiếp **không hoàn lại**.
(a) `P(bi thứ hai trắng | bi thứ nhất đen)`: phân số 7/9 , thập phân 3 chữ số 0.778
(b) `P(cả hai bi đều đen)`: công thức P(cả hai bi đều đen) = P(bi thứ 1 đen) * P(bi thứ 2 đen | bi thứ 1 đen), rồi phân số 1/15 , thập phân 3 chữ số 0.067

**C3. (6 điểm)** Một mẫu có `n = 100`, trung bình = 52, `s = 20`.
(a) `SE`: công thức, thay số, rồi số: SE = s / căn bậc 2 (n) = 20 / căn bậc 2(100) = 2
(b) Khoảng tin cậy 95%: công thức, thay số, rồi hai đầu mút (2 chữ số thập phân): khoảng tin cậy 95% = trung bình +- 1.96 * SE = [48.08, 55.92]
(c) Kiểm: đầu trái nhỏ hơn đầu phải chưa? Trung bình mẫu có nằm giữa hai đầu chưa? đầu trái nhỏ hơn đầu phải và trung bình nằm giữa cả 2 đầu

**C4. (10 điểm)** Tung một đồng xu **100 lần**, được **41 ngửa**. H0: đồng xu công bằng.
Bước 1 (kỳ vọng và độ lệch chuẩn dưới H0):kỳ vọng = 100 * 0.5 = 50 và độ lệch chuẩn dưới H0 = căn bậc 2 (100 * 0.5 * 0.5) = 5
Bước 2 (`z`, ghi số 41 vào công thức): z = (41 - 50) / 5 = -2
Bước 3 (so `|z|` với 1,96 và 2,58): |z| = 2 >= 1.96 tức p <= 0.05, |z| = 2 < 2.58 tức p > 0.01
Bước 4 (kết luận ở mức 5%, **một câu theo mẫu**: "Nếu đồng xu công bằng thì kết quả này ... (p ... 0,05), nên ..."): Nếu đồng xu công bằng thì kết quả này |z| >= 1.96 (p <= 0.05) nên bác bỏ H0
(e) Đúng hay sai, **kèm lý do đúng**: "p = 0,07 nghĩa là xác suất đồng xu công bằng là 7%."sai vì với p = 0.07 tức p > 0.05 thì chưa đủ bằng chứng chứng minh đồng xu công bằng

---

## D. Đại số tuyến tính và machine learning (16 điểm)

**D1. (8 điểm)** `X` có shape `(5, 2)` (5 mẫu, 2 đặc trưng), `w` có shape `(2,)`, `b = 1`.
(a) Shape của `X @ w`: (5,)
(b) Mỗi số trong `X @ w + b` là gì (một câu, theo ý nghĩa mẫu và dự đoán)? mỗi số trong X @ w + b ứng với dự đoán của cho mỗi mẫu của X
(c) `X = [[1, 2], [3, 4]]`, `w = [2, 1]`, `b = 1`: tính tay `X @ w + b`:
[5,11]

**D2. (8 điểm)** Hai tình huống:
(a) Mô hình G có sai số train = 0,3 và sai số test = 7. Nó là tốt, quá khớp hay thiếu khớp? Dẫn hai số.
(b) Chị Hoa tính trung bình và độ lệch chuẩn trên **toàn bộ** dữ liệu rồi mới chia train và test để chuẩn hóa. Có rò rỉ dữ liệu không? Một câu lý do.
(a): nó là quá khớp vì sai số train rất thấp là 0.3 còn sai số test là 7 cao hơn sai số train rất nhiều
(b): Có rò rỉ dữ liệu vì tính trước rồi mới chia thì model sẽ biết được quy luật của tập dữ liệu trên cả test và train

---

## Ghi nhật ký (điền đủ ba ô bằng câu hoàn chỉnh)
- Giờ bắt đầu và giờ kết thúc thực tế: 10h-
- Câu bạn nghĩ mình sai nhiều khả năng nhất, và vì sao: câu A1 tôi không thấy sai mà tôi thấy đề lạ khi đây là SQL chứ không phải xác suất thống kê, D2 vì tôi có thể 
- Những câu bạn đã kiểm bằng cách chạy code (và câu nào chưa kiểm):
