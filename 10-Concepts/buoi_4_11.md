---
title: Thứ Tư 4/11 - ML 2 (hồi quy tuyến tính, hàm mất mát) và ôn SQL
tags: [machine-learning, sql, tuan-3]
---

# 4/11 (1 giờ): ML 2 (45 phút) + ôn SQL (15 phút)

**Cách làm:** tính tay hoặc dự đoán trước, rồi chạy code kiểm và dán nguyên văn kết quả. Làm tròn bình thường, lệch ≤ 0,001 vẫn đúng. Đề mơ hồ thì ghi "đề mơ hồ vì ...". Mọi câu chỉ dùng nội dung trong bài học bên dưới.

**Thay đổi theo góp ý 3/11:** bỏ phần nhật ký ba dòng cuối. Thay vào đó vài câu có ô **"Kiểm lại"** ngay trong bài (kiểm đáp số vào đề gốc), để tập trung vào kiến thức và bài tập. Mỗi câu vẫn gắn nhãn **[đã gặp]** hoặc **[mới]**, và quy luật/lý do sai vẫn chỉ được nửa điểm.

---

# PHẦN 1 (45 phút): ML 2 - hồi quy tuyến tính và hàm mất mát

## Bài học (mỗi ý có ví dụ làm sẵn)

**1. Mô hình hồi quy tuyến tính một biến:** `ŷ = w·x + b`. `w` là độ dốc (x tăng 1 đơn vị thì `ŷ` tăng `w`), `b` là giá trị `ŷ` khi `x = 0`. `ŷ` đọc là "y mũ", là giá trị **dự đoán**; `y` là giá trị **thực**.
Ví dụ làm sẵn (quán cà phê đá): `x` là nhiệt độ ngoài trời (°C), `ŷ` là số ly dự đoán bán được, `ŷ = 2x + 10`. Với `x = 15`: `ŷ = 2·15 + 10 = 40` ly. `w = 2` nghĩa là nóng thêm 1°C thì dự đoán bán thêm 2 ly.

**2. Sai số (phần dư):** `e = y − ŷ`. `e > 0` nghĩa là dự đoán **thấp hơn** thực tế.

**3. Hàm mất mát.** Đo mô hình sai bao nhiêu trên cả tập dữ liệu:
- `MSE = trung bình của e²` (sai số bình phương trung bình).
- `MAE = trung bình của |e|` (sai số tuyệt đối trung bình).
Ví dụ làm sẵn: dữ liệu `x = [1, 2, 3]`, `y = [5, 9, 14]`, mô hình `ŷ = 3x + 2` cho `ŷ = [5, 8, 11]`.
`e = [0, 1, 3]`. `MSE = (0² + 1² + 3²)/3 = 10/3 = 3,333`. `MAE = (0 + 1 + 3)/3 = 1,333`.
Nhận xét: MSE phạt sai số **lớn** nặng hơn (3 bị bình phương thành 9). MSE và MAE càng nhỏ càng khớp dữ liệu đó.

**4. So sánh hai mô hình:** tính MSE của cả hai **trên cùng dữ liệu**, mô hình có MSE nhỏ hơn khớp dữ liệu đó tốt hơn.

**5. Train và test (đã học):** MSE trên tập train rất nhỏ nhưng trên tập test lớn là dấu hiệu **quá khớp**. Phải đánh giá bằng MSE trên **test**.

**6. Nhiều đặc trưng (nối với ĐSTT):** `ŷ = X @ w + b`, `X` shape `(n, số đặc trưng)`, `w` shape `(số đặc trưng,)`, kết quả shape `(n,)`, mỗi phần tử là dự đoán cho một mẫu.

**7. Code:** `np.polyfit(x, y, 1)` trả `[w, b]` của đường thẳng khớp nhất (MSE nhỏ nhất). Ví dụ làm sẵn: `x = [1,2,3]`, `y = [3,5,7]` (nằm đúng trên `y = 2x + 1`) cho `w = 2,0`, `b = 1,0`, MSE = 0.

## Bài tập

**R1. [mới]** Máy chủ web: `x` là số người truy cập đồng thời (đơn vị trăm người), `ŷ` là thời gian phản hồi dự đoán (ms), mô hình `ŷ = 0,5x + 20`.
(a) `ŷ` khi `x = 10`: 25
(b) `ŷ` khi `x = 30`: 35
(c) Ý nghĩa của `w = 0,5` (một câu, có đơn vị): là độ dốc để biết x tăng 1 thì ŷ tăng w
(d) Ý nghĩa của `b = 20` (một câu, có đơn vị): b = 20 là giá trị để khi x = 0 thì ŷ không nhận giá trị là 0 mà là 20

**R2. [mới]** Thư viện: `x` là số giờ ôn (giờ), `y` là điểm. Dữ liệu `x = [1, 2, 3, 4]`, `y = [50, 60, 75, 80]`. Mô hình A: `ŷ = 10x + 40`.
(a) `ŷ` (4 số): 50,60,70,80
(b) `e = y − ŷ` (4 số): 0,0,5,0
(c) MSE = 6.25
(d) MAE = 1.25
Kiểm lại (nhân/cộng ngược một sai số vào `y`, ghi phép tính):
e = y - ŷ = y - 70 = 5 => y = 70 + 5 = 75

**R3. [mới]** Cùng dữ liệu R2. Mô hình B: `ŷ = 12x + 35`.
(a) `ŷ` (4 số): 47, 59, 71, 83
(b) MSE của B = 8.75
(c) Mô hình nào khớp dữ liệu tốt hơn (A hay B), và dựa vào số nào? Mô hình A khớp dữ liệu tốt hơn vì hàm loss của A cho MSE = 6.25 bé hơn hàm loss của B cho MSE = 8.75

**R4. [mới, tìm lỗi]** Bình nói: "Mô hình của mình có MSE trên tập train bằng 0, nên chắc chắn dự đoán tốt trên dữ liệu mới."
(a) Bình sai ở đâu? mới chỉ có MSE trên tập train chưa nói lên điều gì
(b) Viết lại thành câu đúng (nhắc tập nào cần đo): cần đo MSE trên tập test đó mới là kết quá đáng quan tâm

**R5. [đã gặp]** Quán trà sữa dự đoán doanh thu ngày. Hai mô hình: A có MSE train 2 và MSE test 30; B có MSE train 10 và MSE test 12.
(a) Mô hình nào nên chọn? Mô hình B vì MSE test bé là 12 bé hơn MSE test của B là 30
(b) Mô hình còn lại bị gì, và vì sao (nêu hai con số làm bằng chứng):Mô hình còn lại là Mô hình A bị quá khớp với MSE train là 2 nhỏ hơn rất nhiều MSE test là 30

**R6. [mới, dự đoán code]** Dự đoán đầu ra từng dòng trước khi chạy (gợi ý: kiểm xem các điểm có nằm đúng trên một đường thẳng không):
```python
import numpy as np
x = np.array([1, 2, 3, 4]); y = np.array([3, 5, 7, 9])
w, b = np.polyfit(x, y, 1)
print(round(w, 2), round(b, 2))
yhat = w * x + b
print(round(np.mean((y - yhat) ** 2), 2))
```
Dòng in thứ nhất: 2 1
Dòng in thứ hai: 0
Cách bạn suy ra (một câu, thử vào điểm `x = 1`): 2 * 1 + 1 = 3

**R7. [đã gặp]** Căn hộ cho thuê: đặc trưng là diện tích (m²) và số phòng ngủ. `X = [[30, 1], [50, 2]]`, `w = [0,2; 1,5]`, `b = 3` (triệu đồng), `ŷ = X @ w + b`.
(a) `ŷ` (hai số, tính tay): 10.5 16.0
(b) `shape` của `ŷ`, và mỗi phần tử là gì: (2,) và mỗi một phần tử là dự đoán giá căn hộ cho thuê khi biết diện tích và số phòng ngủ

---

# PHẦN 2 (15 phút): ôn SQL (đọc truy vấn, dự đoán, tìm lỗi)

## Nhắc (có ví dụ làm sẵn)
- **Hàm gộp bỏ qua NULL:** `AVG`, `SUM`, `COUNT(cột)` bỏ qua NULL; `COUNT(*)` đếm mọi hàng. Ví dụ: cột `m = [10, NULL, 20]`: `COUNT(*) = 3`, `COUNT(m) = 2`, `SUM(m) = 30`, `AVG(m) = 15`.
- **`GROUP BY` + `HAVING`:** `HAVING` lọc **nhóm** sau khi gộp, viết hàm gộp đầy đủ (`HAVING AVG(m) > 12`). `WHERE` lọc hàng trước khi gộp và **không** dùng được hàm gộp.
- **`RANK() OVER (PARTITION BY nhóm ORDER BY cột DESC)`:** xếp hạng trong từng nhóm, 1 là lớn nhất. Ví dụ: nhóm `A` có `10, 30, 20` cho hạng `3, 1, 2`.

Bảng dùng cho Q1 đến Q3 (phiếu hỗ trợ khách hàng, `minutes` là số phút xử lý, NULL là chưa đóng):

| id | agent | priority | minutes |
|---|---|---|---|
| 1 | An | high | 20 |
| 2 | An | low | 40 |
| 3 | Bình | high | 50 |
| 4 | Bình | low | NULL |
| 5 | Chi | high | 10 |
| 6 | Chi | low | 30 |
| 7 | An | low | 60 |

**Q1. [đã gặp]** `SELECT agent, AVG(minutes) FROM tickets GROUP BY agent HAVING AVG(minutes) > 30;`
Dự đoán các hàng kết quả (`agent`, giá trị trung bình), kèm cách tính từng agent: 

| agent | AVG(minutes) |
| ----- | ------------ |
| An    | 40           |
An: (20 + 40 + 60) / 3 = 40


**Q2. [mới]** `SELECT COUNT(*), COUNT(minutes), SUM(minutes) FROM tickets;`
Dự đoán ba số: 7, 6, 220

**Q3. [mới]** `SELECT id, RANK() OVER (PARTITION BY agent ORDER BY minutes DESC) AS r FROM tickets WHERE minutes IS NOT NULL;`
Dự đoán cột `r` cho từng `id` (1, 2, 3, 5, 6, 7): 

| id  | r   |
| --- | --- |
| 1   | 3   |
| 2   | 2   |
| 3   | 1   |
| 4   |     |
| 5   | 2   |
| 6   | 1   |
| 7   | 1   |


**Q4. [mới, tìm lỗi]** `SELECT agent FROM tickets WHERE AVG(minutes) > 30;` báo lỗi.
(a) Vì sao lỗi? chỗ này tôi thấy có AVG mà thiếu group by đồng thời AVG không đi cùng với WHERE chỉ đi với HAVING
(b) Viết lại đúng để ra các agent có trung bình hơn 30 phút: SELECT agent, AVG(minutes) FROM tickets GROUP BY agent HAVING AVG(minutes) > 30;
