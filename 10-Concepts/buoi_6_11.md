---
title: Thứ Sáu 6/11 - ĐSTT 3 (độ dài, khoảng cách, hồi quy bằng ma trận) và ôn thống kê
tags: [dai-so-tuyen-tinh, thong-ke, tuan-3]
---

# 6/11 (1 giờ): ĐSTT 3 (45 phút) + ôn thống kê và xác suất (15 phút)

**Cách làm:** tính tay hoặc dự đoán trước, rồi chạy code kiểm và dán nguyên văn kết quả. Làm tròn bình thường, lệch ≤ 0,001 vẫn đúng. Đề mơ hồ thì ghi "đề mơ hồ vì ...". Mọi câu chỉ dùng nội dung trong bài học bên dưới. Không có nhật ký. Nhãn **[đã gặp]/[mới]** như thường lệ.

**Hai thói quen cần áp dụng ngay (rút từ bài 5/11):**
1. **Đọc dòng cuối của đề trước khi ghi đáp số.** Đề hỏi "giảm được bao nhiêu" thì đáp số là **một số**, không phải một phép tính dừng giữa chừng.
2. **Đếm vật trước khi ghi phân số.** Khi rút không hoàn lại, mẫu số bắt đầu bằng **tổng số vật ban đầu** rồi giảm 1 sau mỗi lần rút. Không cộng thêm số vật hỏng vào mẫu số.

---

# PHẦN 1 (45 phút): Đại số tuyến tính 3

## Bài học (có ví dụ làm sẵn)

**1. Độ dài (chuẩn) của vectơ:** `‖v‖ = √(v₁² + v₂²)`. Ví dụ: `v = [3, 4]`: `‖v‖ = √(9 + 16) = 5`. NumPy: `np.linalg.norm(v)` ra `5.0`. Nối với tích vô hướng: `v · v = ‖v‖²` (`[3,4]·[3,4] = 25 = 5²`).

**2. Khoảng cách giữa hai điểm** `a`, `b` là độ dài của hiệu: `‖a − b‖`. Ví dụ (hai trạm sạc): `a = [1, 2]`, `b = [4, 6]`: `a − b = [−3, −4]`, `‖a − b‖ = √(9 + 16) = 5`. Không phải `3 + 4 = 7`.

**3. Chuẩn hóa vectơ:** chia vectơ cho độ dài của nó, `v / ‖v‖`, ra vectơ cùng hướng có độ dài 1. Ví dụ: `[3, 4] / 5 = [0,6; 0,8]`. Kiểm lại: `0,6² + 0,8² = 0,36 + 0,64 = 1`.

**4. Hồi quy tuyến tính bằng ma trận.** Thêm một cột toàn số 1 vào `X` để có hệ số `b`: mỗi hàng của `X` là `[1, x]`, và `w = [b, độ dốc]`. Công thức: `w = (XᵀX)⁻¹ · Xᵀ y` (`Xᵀ` là `X` đổi hàng thành cột). Hai phép đã học: nghịch đảo 2×2 và nhân ma trận.
Ví dụ làm sẵn (tiệm bánh: `x` là số giờ quảng cáo, `y` là số đơn trong ngày): `x = [1, 2, 3]`, `y = [2, 3, 5]`.
- `X = [[1,1],[1,2],[1,3]]` (shape `(3, 2)`).
- `XᵀX = [[3, 6],[6, 14]]` (shape `(2, 2)`: ví dụ ô đầu `1·1 + 1·1 + 1·1 = 3`, ô cuối `1² + 2² + 3² = 14`).
- `Xᵀy = [2+3+5, 1·2 + 2·3 + 3·5] = [10, 23]` (shape `(2,)`).
- `det = 3·14 − 6·6 = 6`, `(XᵀX)⁻¹ = (1/6)·[[14, −6],[−6, 3]]`.
- `w = (1/6)·[14·10 − 6·23, −6·10 + 3·23] = (1/6)·[2, 9] = [0,333; 1,5]`.
Vậy `ŷ = 0,333 + 1,5x`: thêm 1 giờ quảng cáo thì dự đoán thêm 1,5 đơn.
**Lưu ý thứ tự:** cách này ra `[b, độ dốc]`, còn `np.polyfit(x, y, 1)` ra `[độ dốc, b]` (ví dụ trên: `[1.5, 0.333]`). Hai cách cùng một đường.

## Bài tập

**N1. [mới]** Độ dài và khoảng cách.
(a) `‖[5, 12]‖` = 13
(b) Khoảng cách giữa hai trạm xe buýt `a = [2, 1]` và `b = [5, 5]` (ghi `a − b` rồi độ dài): a - b = [-3,-4], ‖[-3,-4]‖ = 5
(c) Chuẩn hóa `[5, 12]` (hai số, 4 chữ số thập phân): [0.3846, 0.9231]
(d) Kiểm lại (c): tổng bình phương hai số vừa tìm = 0.3846^2 + 0.9231^2 =  1

**N2. [đã gặp]** Rạp phim: số vé bán theo ba loại `u = [3, 0, 2]`, giá mỗi loại `p = [50, 80, 120]` (nghìn đồng). Tính tích vô hướng `u · p` và nói con số đó là gì: 3 x 50 + 0 x 80 + 2 x 120 = 390

**N3. [mới, tìm lỗi]** An nói: "Khoảng cách giữa `a = [1, 2]` và `b = [4, 6]` là 3 + 4 = 7."
(a) An sai ở đâu (theo định nghĩa trong bài)? khoảng cách của a và b phải là ‖a - b‖ chứ không phải tính độ dài của a và b sau đó cộng lại
(b) Đáp số đúng và phép tính: ‖a - b‖ = ‖[-3,-4]‖ = √((-3) ^ 2 + (-4) ^ 2) = √(9 + 16) = 5

**N4. [mới]** Xe cũ: `x` là tuổi xe (năm), `y` là giá (trăm triệu đồng). Dữ liệu `x = [0, 1, 2]`, `y = [5, 3, 2]`. Làm đúng các bước như ví dụ tiệm bánh:
(a) `X` (ba hàng, mỗi hàng `[1, x]`): [[1,0][1,1][1,2]]
(b) `XᵀX` = [[3,3][3,5]]
(c) `Xᵀy` = [[10][7]]
(d) `det(XᵀX)` = 6
(e) `(XᵀX)⁻¹` = (1/6) x [[5,-3][-3,3]] = 
(f) `w = [b, độ dốc]` (3 chữ số thập phân): (1/6) * [5 x 10 + (-3) x 7, (-3) x 10 + 3 x 7] = (1/6) * [29,-9] = [4.833,-1.5]
(g) `ŷ` cho `x = 0, 1, 2` (3 số) và MSE (3 chữ số thập phân): MSE = 1.667. Đề không rõ hỏi chính xác là gì nên tôi hiểu đề hỏi MSE nên tôi tính MSE còn 
Kiểm lại (độ dốc âm có hợp lý với dữ liệu không? và `ŷ` tại `x = 1` gần `y = 3` không?): 1 x (-1.5) + 4.833 = 3.333 kết quả 3.333 gần bằng 3

**N5. [mới]** Với `X` của N4 (shape `(3, 2)`) và `y` shape `(3,)`: ghi `shape` của `Xᵀ`, `XᵀX`, `Xᵀy`: shape lần lượt là (2,3), (2,2), (2,)

**N6. [mới, chạy code rồi dán nguyên văn]** Dự đoán trước, rồi chạy:
```python
import numpy as np
x = np.array([0, 1, 2]); y = np.array([5, 3, 2])
X = np.column_stack([np.ones(3), x])
print(np.round(np.linalg.solve(X.T @ X, X.T @ y), 3))
print(np.round(np.polyfit(x, y, 1), 3))
```
Dự đoán hai dòng: dòng 1 [4.833, -1.5], dòng 2 [-1.5,4.833]
Kết quả chạy (nguyên văn): [ 4.833 -1.5  ]
[-1.5    4.833]
Hai dòng khác thứ tự ở chỗ nào, và vì sao? 2 dòng khác nhau và khác ở thứ tự kết quả với dòng 1 thì thứ tự là [b, độ dốc], dòng 2 thứ tự là [độ dốc, b] lí do tôi không rõ tôi thấy lệnh dòng 1 là tính công thức hồi quy tuyến tính như những gì chúng ta học còn phần dòng 2 tôi không nhớ rõ 

---

# PHẦN 2 (15 phút): ôn thống kê và xác suất (đổi chủ thể)

## Nhắc (có ví dụ làm sẵn)
- **Độ lệch chuẩn:** tổng thể chia `n`, mẫu chia `n − 1`. Ví dụ `[1, 3, 5, 7]`: trung bình 4, bình phương lệch `9 + 1 + 1 + 9 = 20`; tổng thể `√(20/4) = 2,236`; mẫu `√(20/3) = 2,582`. `np.std` mặc định chia `n`; `pd.Series.std` mặc định chia `n − 1`.
- **Kiểm định 4 bước** (ví dụ: tung 100 lần được 58 ngửa, H0 công bằng): kỳ vọng `n·p = 50`, độ lệch chuẩn `√(n·p·(1−p)) = 5`; `z = (58 − 50)/5 = 1,6`, kiểm ngược `1,6 · 5 + 50 = 58`; `|z| = 1,6 < 1,96` nên `p > 0,05`; kết luận: "Nếu H0 đúng thì kết quả này không hiếm (p > 0,05), nên chưa đủ bằng chứng bác bỏ H0".
- **Rút không hoàn lại:** hộp 5 tai nghe có 2 cái lỗi, lấy 2 cái. `P(cả hai lỗi) = 2/5 · 1/4 = 0,1`. `P(đúng một lỗi) = 2 · (2/5 · 3/4) = 0,6` (nhân 2 vì có hai thứ tự: lỗi rồi tốt, hoặc tốt rồi lỗi).

**T1. [đã gặp]** Thời gian tải trang (giây) của một trang web: `[4, 6, 8]`.
(a) Độ lệch chuẩn tổng thể: 1.633 
(b) Độ lệch chuẩn mẫu: 2
(c) `np.std([4, 6, 8])` in ra số nào, và `pd.Series([4, 6, 8]).std()` in ra số nào? np.std([4, 6, 8]) ra số tại phần a, pd.Series([4, 6, 8]).std() ra số tại phần b

**T2. [mới]** Công ty có tỉ lệ nhấn quảng cáo đã biết là 40%. Chiến dịch mới hiển thị 200 lượt, được 92 lượt nhấn. H0: tỉ lệ vẫn là 40%. Làm đủ 4 bước:
Bước 1 (kỳ vọng, độ lệch chuẩn): kỳ vọng = 200 * 0.4 = 80, độ lệch chuẩn = √(200 * 0.4 * 0.6) = 6.929
Bước 2 (`z`, 3 chữ số thập phân): z = (92 - 80) / 6.929 = 1.732
Kiểm ngược (`z × độ lệch chuẩn + kỳ vọng` có ra 92 không?): 1.732 x 6.929 + 80 = 92.001 gần bằng 92
Bước 3 (so `|z|` với 1,96): |z| = 1.732 < 1.96 nên p > 0.05
Bước 4 (một câu theo mẫu, mức 5%): nếu H0 đúng thì kết quả này không hiếm (p > 0.05), nên chưa đủ bằng chứng bác bỏ H0

**T3. [đã gặp, làm lại]** Hộp có **tổng cộng 8 viên pin, trong đó 3 viên yếu** (5 viên còn lại tốt). Lấy lần lượt 2 viên không hoàn lại.
(a) P(cả hai yếu) = (3/8) * (2/7) = 0.107
(b) P(đúng một viên yếu) = (3/8) * (5/7) = 0.268
(c) P(không viên nào yếu) = (5/8) * (4/7) = 0.357
Kiểm lại (cộng (a), (b), (c) phải bằng bao nhiêu, ghi phép cộng):0.107 + 0.268 + 0.357 = 0.732. Câu này có thể sai hãy kiểm tra suy đoán của ông giúp toi
