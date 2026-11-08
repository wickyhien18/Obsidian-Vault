---
title: Thứ Bảy 7/11 - ML 3 (phân loại, confusion matrix, precision, recall) và ôn Python/pandas
tags: [machine-learning, python, tuan-3]
---

# 7/11 (1 giờ): ML 3 (45 phút) + ôn Python/pandas (15 phút)

**Cách làm:** tính tay hoặc dự đoán trước, rồi chạy code kiểm và dán nguyên văn kết quả. Làm tròn bình thường, lệch ≤ 0,001 vẫn đúng. Đề mơ hồ thì ghi "đề mơ hồ vì ...". Mọi câu chỉ dùng nội dung trong bài học bên dưới. Không có nhật ký. Nhãn **[đã gặp]/[mới]** như thường lệ. Câu viết lập luận có **tiêu chí chấm** ghi ngay trong câu.

**Ba thói quen áp dụng ngay (rút từ bài 5/11 và 6/11):**
1. **Đọc dòng cuối của đề** trước khi ghi đáp số.
2. Câu hỏi "**đúng một** ..." thì tự hỏi: *có mấy thứ tự xảy ra?* (thường là 2, nhân 2).
3. **Kiểm nhanh bằng độ lớn:** MSE không thể lớn hơn bình phương sai số lớn nhất. Ví dụ: sai số lớn nhất là 3 thì MSE ≤ 9. Nếu sai số lớn nhất là 0,33 thì MSE ≤ 0,11, và 1,667 là không thể.

---

# PHẦN 1 (45 phút): ML 3 - bài toán phân loại

## Bài học (có ví dụ làm sẵn)

**1. Bài toán phân loại nhị phân:** mô hình dự đoán **dương** (có sự kiện cần tìm) hoặc **âm** (không có). Đem so với thực tế, mỗi mẫu rơi vào một trong bốn ô (ma trận nhầm lẫn):
- **TP** (dương thật): dự đoán dương, thực tế dương.
- **FP** (dương giả): dự đoán dương, thực tế **âm** (báo nhầm).
- **FN** (âm giả): dự đoán âm, thực tế **dương** (bỏ sót).
- **TN** (âm thật): dự đoán âm, thực tế âm.
Cả bốn ô cộng lại bằng tổng số mẫu.

**2. Bốn chỉ số:**
- `accuracy = (TP + TN) / tổng`
- `precision = TP / (TP + FP)`: trong số mẫu **mô hình báo dương**, bao nhiêu phần đúng.
- `recall = TP / (TP + FN)`: trong số mẫu **thực sự dương**, mô hình tìm ra bao nhiêu phần.
- `F1 = 2 · precision · recall / (precision + recall)`: một con số tóm tắt cả hai.

Ví dụ làm sẵn (hệ thống báo cháy, 100 lần theo dõi): `TP = 8`, `FP = 4`, `FN = 2`, `TN = 86` (cộng lại 100).
`accuracy = (8 + 86)/100 = 0,94`. `precision = 8/(8 + 4) = 0,667`. `recall = 8/(8 + 2) = 0,8`. `F1 = 2 · 0,667 · 0,8 / (0,667 + 0,8) = 0,727`.

**3. Accuracy có thể đánh lừa khi dữ liệu mất cân bằng.** Ví dụ: 1000 email có 10 thư lừa đảo. Mô hình đoán **tất cả là an toàn**: đúng 990/1000 nên `accuracy = 0,99`, nhưng `recall = 0/10 = 0` (không bắt được thư lừa đảo nào). Vì vậy phải xem cả precision và recall.

**4. Ưu tiên chỉ số nào tùy cái nào tốn kém hơn:**
- Bỏ sót (FN) nguy hiểm hơn → ưu tiên **recall**. Ví dụ: báo cháy, bỏ sót một đám cháy rất tốn kém, báo nhầm chỉ mất công kiểm tra.
- Báo nhầm (FP) tốn kém hơn → ưu tiên **precision**. Ví dụ: tự động khóa tài khoản khách hàng, khóa nhầm làm mất khách.

## Bài tập

**K1. [mới]** Nhà máy dùng camera AI kiểm 200 sản phẩm. Thực tế có 20 sản phẩm lỗi. Camera báo lỗi 25 sản phẩm, trong đó 15 sản phẩm đúng là lỗi.
(a) TP, FP, FN, TN (bốn số): TP = 15, FP = 10, FN = 5, TN = 170 
(b) accuracy = (15 + 170) / 200 = 0.925
(c) precision = 15 / (15 + 10) = 0.6
(d) recall = 15 / (15 + 5) = 0.75
(e) F1 = (3 chữ số thập phân) = 2 * (0.6 * 0.75) / (0.6 + 0.75) = 0.667
Kiểm lại (cộng bốn ô của (a), ghi phép cộng):15 + 10 + 5 + 170 = 200

**K2. [mới, tìm lỗi]** Một trường có 500 học sinh, thực tế có 10 em bỏ học. Mô hình dự báo bỏ học dự đoán **cả 500 em đều không bỏ học**. Trưởng phòng nói: "Accuracy cao quá, mô hình rất tốt."
(a) accuracy của mô hình = 0.98
(b) recall của mô hình = 0/10 = 0
(c) **Tiêu chí chấm:** nêu hai con số (a) và (b); chỉ ra lập luận của trưởng phòng sai ở đâu; một câu viết lại đúng. Viết lại: Lập luận của trưởng phòng sai. Ổng sai ở chỗ quá tin tưởng accurancy vì accuracy là 0.98 rất cao nhưng bỏ sót recall = 0 tức mô hình không tìm ra bất kỳ ai bỏ học trong số 10 người bỏ học. Về phần viết lại tôi không rõ viết lại thế nào

**K3. [mới]** Chọn **recall** hay **precision** làm chỉ số ưu tiên? Mỗi tình huống một câu lý do, nêu rõ lỗi nào (FP hay FN) tốn kém hơn:
(a) Máy soi hành lý ở sân bay tìm vật nguy hiểm:nếu bỏ sót (FN) gây nguy hiểm đến các hành khách khác, còn báo nhầm (FP) thì chỉ gây phiền cho khách hàng 1 chút. Nên bỏ sót tốn kém hơn cần ưu tiên recall
(b) Diễn đàn tự động xóa bình luận bị nghi vi phạm: báo nhầm (FP) gây mất khách mất người dùng, còn bỏ sót (FN) thì không quá đáng lo nếu người dùng không để tâm đến bình luận đó. Nên báo nhầm (FP) tốn kém hơn cần ưu tiên precision

**K4. [đã gặp]** Dự đoán giá phòng trọ (triệu đồng). Giá thực `y = [3, 4, 8]`, dự đoán `ŷ = [3, 5, 5]`.
(a) `e = y − ŷ` (3 số): 0,-1,3
(b) MSE = 3.333
(c) MAE = 1.333
Kiểm nhanh bằng độ lớn (MSE phải ≤ bình phương sai số lớn nhất; ghi hai con số so sánh): 3.333 <= 9

---

# PHẦN 2 (15 phút): ôn Python/pandas (đọc code, dự đoán, tìm lỗi)

## Nhắc (có ví dụ làm sẵn)
- **`pd.merge(df1, df2, on="khóa")`** ghép hai bảng theo cột chung. Ví dụ: `df1` có `id = [1, 2]`, `name = ["An", "Bình"]`; `df2` có `id = [1, 2]`, `score = [8, 9]` → bảng ghép có ba cột `id`, `name`, `score`, hàng 1 là `1, An, 8`.
- **`groupby("cột")["cột khác"].mean()`:** trung bình theo nhóm (đã học).
- **Thuộc tính và phương thức:** `.shape` là **thuộc tính** (không ngoặc). `.mean()` là **phương thức** (phải có ngoặc). Thiếu ngoặc ở phương thức thì Python không tính mà in ra một dòng bắt đầu bằng `<bound method ...>`.
- **Gán list:** `b = a` không tạo bản sao; `a.copy()` mới tạo bản sao (đã học).

Dữ liệu cho Y1 đến Y3 (thư viện, số ngày mượn sách):
```python
import pandas as pd
books = pd.DataFrame({"book_id": [1, 2, 3], "category": ["Khoa học", "Văn học", "Khoa học"]})
loans = pd.DataFrame({"loan_id": [1, 2, 3, 4, 5], "book_id": [1, 1, 2, 3, 3], "days": [10, 14, 7, 20, 10]})
```

**Y1. [đã gặp]** `m = pd.merge(loans, books, on="book_id")` rồi `print(m.groupby("category")["days"].mean())`. Dự đoán số ngày mượn trung bình của từng thể loại (kèm cách tính): 
category
Khoa học    13.5
Văn học      7.0

Khoa học: (10 + 14 + 20 + 10) / 4 = 13.5
Văn học: 7 / 1 = 7

**Y2. [mới]** `print(len(loans[loans["days"] >= 10]))`. Dự đoán số in ra và liệt kê các hàng thỏa điều kiện (theo `loan_id`): số in ra là 4

| loan_id | book_id | days |
| ------- | ------- | ---- |
| 1       | 1       | 10   |
| 2       | 1       | 14   |
| 4       | 3       | 20   |
| 5       | 3       | 10   |


**Y3. [mới, tìm lỗi]** Bình viết `print(loans["days"].mean)` và thấy in ra một dòng bắt đầu bằng `<bound method Series.mean of ...>` thay vì một con số.
(a) Vì sao? vì mean thiếu ngoặc đằng sau nên Python hiểu đó là thuộc tính nên mới in ra là bound method Series.mean of 
(b) Sửa lại và ghi con số đúng sẽ in ra: print(loans["days"].mean())
kết quả là 12.2

**Y4. [đã gặp]** Dự đoán dòng in ra:
```python
a = [1, 2, 3]; b = a; b.append(4); c = a.copy(); c.append(5)
print(a, c)
```
Dự đoán: [1,2,3,4][1,2,3,4,5]
