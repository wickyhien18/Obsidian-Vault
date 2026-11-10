---
title: Chủ Nhật 8/11 - Ôn tổng hợp 90 phút (chuẩn bị đề thi thử 10-11/11)
tags: [on-tong-hop, tuan-3]
---

# 8/11 (90 phút): ôn tổng hợp trước đề thi thử

**Mục tiêu:** làm một lần liền khoảng 90 phút (gợi ý: A 20 phút, B 10, C 25, D 20, E 15), không xem đáp án giữa chừng. Đây là buổi tập dượt; đề thi thử nhỏ (có bấm giờ) là 10-11/11. Làm tròn bình thường, lệch ≤ 0,001 vẫn đúng. Đề mơ hồ thì ghi "đề mơ hồ vì ...". Mọi câu chỉ dùng nội dung đã học (công thức cần nhớ ở ngay dưới). Không có nhật ký. Nhãn **[đã gặp]/[mới]**; câu viết lập luận có **tiêu chí chấm** trong đề.

**Năm thói quen (rút từ sổ lỗi 4/11-7/11), nhìn lại trước khi nộp:**
1. Đọc **dòng cuối của đề**: đề hỏi đại lượng nào thì đáp số là đại lượng đó (một số, không dừng giữa phép tính).
2. Câu "**đúng một ...**": có mấy thứ tự? (thường nhân 2). Các trường hợp cộng lại phải bằng 1.
3. Hàm gộp SQL (`AVG`, `SUM`, `COUNT(cột)`) **bỏ qua NULL**; `COUNT(*)` đếm mọi hàng. **Cộng lại tổng** một lần nữa.
4. **Kiểm nhanh bằng độ lớn** (MSE ≤ bình phương sai số lớn nhất) và **kiểm ngược** (nhân ngược lại có ra số ban đầu không).
5. Chạy code thì **dán nguyên văn**; chưa chắc thì ghi "không biết" hoặc "đề mơ hồ vì ...", không bỏ trống.

## Công thức cần nhớ (đều đã dạy kèm ví dụ ở các buổi trước)
- Rút không hoàn lại: `P(A ∩ B) = P(A)·P(B|A)`, mẫu số giảm 1 mỗi lần rút. Bayes: dùng tần suất tự nhiên (đếm trên số lượng lớn).
- `SE = s/√n`; khoảng tin cậy 95% = trung bình ± 1,96·SE; n tăng 4 lần thì SE giảm một nửa.
- Kiểm định tỉ lệ: kỳ vọng `n·p`, độ lệch chuẩn `√(n·p·(1−p))`, `z = (quan sát − kỳ vọng)/độ lệch chuẩn`, kiểm ngược `z·độ lệch chuẩn + kỳ vọng`; `|z| > 1,96` thì `p < 0,05` (bác bỏ H0 ở mức 5%); `|z| > 2,58` thì `p < 0,01`.
- Ma trận 2×2: `det = ad − bc`; `A⁻¹ = (1/det)·[[d, −b],[−c, a]]` (chỉ khi `det ≠ 0`); `A @ x = b` thì `x = A⁻¹ @ b`. Độ dài `‖v‖ = √(v₁² + v₂²)`, khoảng cách `‖a − b‖`, chuẩn hóa `v/‖v‖`.
- Phân loại: `accuracy = (TP+TN)/tổng`, `precision = TP/(TP+FP)`, `recall = TP/(TP+FN)`, `F1 = 2·P·R/(P+R)`. MSE = trung bình `e²`, MAE = trung bình `|e|`.
- Case study: tỉ lệ chuyển đổi = số chuyển đổi / số lượt; chi phí mỗi người trả phí = chi phí / số người trả phí; `ROAS = doanh thu / chi phí` (< 1 nghĩa là doanh thu chưa bù đủ chi phí quảng cáo). Khuyến nghị gồm **Kết luận / Căn cứ (số) / Hành động**.
- pandas: điều kiện kép dùng `&` với **ngoặc quanh từng điều kiện**. Dùng `and` thì Python báo `ValueError: The truth value of a Series is ambiguous` (vì `and` không so sánh từng phần tử của cột).

---

# A. SQL (khoảng 20 phút), file chạy kèm: `sql_8_11.py`

Bảng `rides` (chuyến taxi; `rating` có thể NULL nếu khách không đánh giá) và `drivers`:

| ride_id | driver | city | fare | rating |
|---|---|---|---|---|
| 1 | Hùng | HN | 50 | 5 |
| 2 | Hùng | HN | 70 | NULL |
| 3 | Lan | HN | 40 | 4 |
| 4 | Lan | HCM | 90 | 5 |
| 5 | Minh | HCM | 60 | 3 |
| 6 | Minh | HCM | 50 | NULL |
| 7 | Hùng | HCM | 30 | 4 |

`drivers`: Hùng, Lan, Minh, Nam.

**A1. [đã gặp]** `SELECT city, SUM(fare) FROM rides GROUP BY city HAVING SUM(fare) > 200;` Dự đoán các hàng kết quả (kèm cách tính từng thành phố):

| city | SUM(fare) |
| ---- | --------- |
| HCM  | 230       |
HCM = 90 +60 + 50 + 30 = 230
**A2. [mới]** `SELECT COUNT(*), COUNT(rating), AVG(rating) FROM rides;` Dự đoán ba số (kèm cách tính `AVG`): 7, 5, 4.2
AVG = (5 + 4 + 5 + 3 + 4) / 5 = 4.2

**A3. [đã gặp]** `SELECT ride_id, SUM(fare) OVER (PARTITION BY driver ORDER BY ride_id) AS run FROM rides ORDER BY ride_id;` Dự đoán cột `run` cho `ride_id` 1 đến 7:

| ride_id | run |
| ------- | --- |
| 1       | 50  |
| 2       | 120 |
| 3       | 40  |
| 4       | 130 |
| 5       | 60  |
| 6       | 110 |
| 7       | 150 |

**A4. [mới]** `SELECT d.driver FROM drivers d LEFT JOIN rides r ON d.driver = r.driver WHERE r.ride_id IS NULL;`
(a) Kết quả: 

| driver |
| ------ |
| Nam    |

(b) Vì sao điều kiện lọc là `r.ride_id IS NULL` (cột của bảng **bên phải**)? Một câu: vì lệnh trên chỉ giữ lại các giá trị của bảng bên trái còn bảng bên phải đối với các dòng không trùng khóa driver thì rider_id là NULL

# B. Python/pandas (khoảng 10 phút)

```python
import pandas as pd
df = pd.DataFrame({"item": ["trà", "cà phê", "bánh", "nước"], "price": [30, 25, 20, 10], "qty": [4, 10, 5, 20]})
df["revenue"] = df["price"] * df["qty"]
```

**B1. [đã gặp]** (a) Cột `revenue` (4 số): 120, 250, 100, 200
(b) `print(df[(df["revenue"] >= 120) & (df["qty"] <= 10)])` in ra những mặt hàng nào? trà và cà phê

**B2. [mới, tìm lỗi]** `df[(df["qty"] > 5) and (df["price"] > 15)]` bị lỗi. (a) Python báo lỗi gì (tên lỗi)? bị báo lỗi ValueError
(b) Viết lại đúng: df[(df["qty"] > 5) & (df["price"] > 15)]
(c) Kết quả của bản đúng là hàng nào? kết quả ra mặt hàng cà phê

**B3. [đã gặp]** Dự đoán `print([x ** 2 for x in range(1, 6) if x % 2 == 1])`: [1,9,25]

# C. Xác suất và thống kê (khoảng 25 phút)

**C1. [đã gặp]** Lô 9 chiếc điện thoại có 4 chiếc lỗi. Kiểm lần lượt 2 chiếc, không hoàn lại.
(a) P(cả hai lỗi) = (4/9) * (3/8) = 0.167
(b) P(đúng một chiếc lỗi) = 2 * (4/9) * (5/8) = 0.556
(c) P(không chiếc nào lỗi) = (5/9) * (4/8) = 0.278
Kiểm lại (cộng ba xác suất, ghi phép cộng): 0.167 + 0.556 + 0.278 = 1.001

**C2. [mới]** Bộ lọc bình luận độc hại: trong 2000 bình luận có 5% độc hại. Bộ lọc gắn cờ đúng 80% bình luận độc hại và gắn cờ nhầm 10% bình luận bình thường.
(a) Số bình luận độc hại bị gắn cờ: 80
(b) Số bình luận bình thường bị gắn cờ nhầm: 190
(c) P(độc hại | bị gắn cờ) (3 chữ số thập phân): 0.296
(d) Trong ML 3, con số 80% đề cho là chỉ số nào (precision hay recall), và con số ở (c) là chỉ số nào? 80% là chỉ số precision còn recall là con số ở c

**C3. [đã gặp]** Thời gian giao hàng (phút) của 100 đơn có trung bình 70 và độ lệch chuẩn mẫu `s = 15`.
(a) SE = 1.5
(b) Khoảng tin cậy 95% (hai đầu, 2 chữ số thập phân): [67.06,72.94]
(c) Nếu cỡ mẫu tăng lên 400 thì SE = 0.75

**C4. [mới]** Trang web có tỉ lệ chuyển đổi 10%. Sau khi đổi giao diện: 500 lượt truy cập, 64 lượt chuyển đổi. H0: tỉ lệ vẫn là 10%.
Bước 1 (kỳ vọng, độ lệch chuẩn): kỳ vọng = 50, độ lệch chuẩn = 6.708
Bước 2 (`z`, 3 chữ số thập phân): z = 2.087
Kiểm ngược: 2.087 * 6.708 + 50 = 63.999
Bước 3 (so `|z|` với 1,96 rồi với 2,58): |z| = 2.087 >= 1.96 => p <= 0.05, |z| = 2.087 > 2.58 => p > 0.01
Bước 4 (một câu theo mẫu, mức 5%): nếu H0 đúng thì xác suất xảy ra của dữ liệu là p <= 0.05 nên bác bỏ H0
(e) **Tiêu chí chấm:** một câu nêu bác bỏ H0 **không** có nghĩa là chắc chắn tỉ lệ đã tăng (nhắc p hoặc "nếu H0 đúng"). Viết: nếu p > 0.05 thì chưa có đủ bằng chứng để bác bỏ H0

# D. Đại số tuyến tính và ML (khoảng 20 phút)

**D1. [đã gặp]** Tiệm photo: 3 bản màu + 1 bản đen giá 11 (nghìn đồng); 2 bản màu + 1 bản đen giá 8 (nghìn đồng).
(a) `A`, `b`: A = [[3,1][2,1]], b = [11,8]
(b) det A = 1
(c) `A⁻¹` = [[1,-1][-2,3]]
(d) Giá một bản màu, một bản đen: 3 nghìn đồng, 2 nghìn đồng
(e) Kiểm lại vào **hai dữ kiện của đề**: 3 x 3 + 2 x 1 = 11
2 x 3 + 1 x 2 = 8

**D2. [mới]** Hai cửa hàng ở tọa độ `a = [1, 1]` và `b = [7, 9]` (km). 
(a) Khoảng cách giữa hai cửa hàng (ghi `a − b` rồi độ dài): a - b = [-6, -8]
||a - b|| = căn bậc 2 (6 ^ 2 + 8 ^ 2) = 10 km
(b) Chuẩn hóa `u = [8, 15]` (4 chữ số thập phân): [0.471, 0.882]
(c) Kiểm lại (b): tổng bình phương hai số vừa tìm = 0.471 ^ 2 + 0.882 ^ 2 = 0.999765 

**D3. [đã gặp]** Hệ thống lọc đơn hàng giả: 300 đơn, thực tế có 30 đơn giả. Hệ thống báo giả 40 đơn, trong đó đúng 24 đơn là giả.
(a) TP, FP, FN, TN: TP = 24, FP = 16, FN = 6, TN = 254
(b) accuracy = 0.927
(c) precision = 0.6
(d) recall = 0.8
Kiểm lại (cộng bốn ô): 24 + 16 + 6 + 254 = 300 

**D4. [đã gặp, tìm lỗi]** Nhóm của Hà muốn dự đoán điểm thi. Họ **tính trung bình và độ lệch chuẩn trên toàn bộ 1000 mẫu để chuẩn hóa dữ liệu, rồi mới chia train/test**.
**Tiêu chí chấm:** gọi đúng tên lỗi; nêu vì sao nó làm kết quả test đáng tin kém đi; nêu cách sửa. Viết bị lỗi rò rỉ dữ liệu vì tính trung bình và độ lệch chuẩn trước khi chia khiến thông tin về bộ test xuất hiện trên bộ train gây kết quả test đẹp nhưng không đáng tin. Cần phải chia trước rồi mới tính trung bình và độ lệch chuẩn
