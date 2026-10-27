---
title: Buổi học Tuần 2 - Ngày 4 (Xác suất 2, dict/set/tuple)
tags: [bai-tap, xac-suat, python, tuan-2]
---

# Tuần 2 - Ngày 4 (Thứ Sáu 23/10): Sửa nợ hôm qua, Bayes, kỳ vọng, dict/set/tuple

**Chia 3 khối, làm rải trong ngày (nghỉ ít nhất 1 giờ giữa các khối). Mỗi khối xong thì đánh dấu, không dồn lại làm một lúc:**
- **KHỐI 1 (sáng, khoảng 45 phút):** bài học ngắn về rút không hoàn lại, rồi 0.1 và 0.2
- **KHỐI 2 (chiều, khoảng 60 phút):** 0.3 và 0.4 (SQL), đọc bài học xác suất 2, bài A1 đến A3
- **KHỐI 3 (tối, khoảng 55 phút):** A4 đến A6, B1, bài học dict/set/tuple, bài C và D, ghi nhật ký (**không được để trống**)

Làm một khối mà thấy mệt hoặc làm sai liên tiếp thì dừng khối đó, ghi vào nhật ký, không cố làm tiếp.

> [!warning] Ba luật mới, rút từ lỗi hôm qua
> 1. **Mô phỏng hoặc chạy code là trọng tài.** Điền bảng "tính tay | máy | hiệu". Hiệu lớn hơn 0,01 (xác suất) hoặc khác từng ô (bảng) thì **tính tay sai hoặc code sai**, không được ghi "gần đúng" hay "khớp". Phải tìm ra lỗi nằm ở đâu.
> 2. **Ô "Kết quả mong đợi" điền TRƯỚC khi chạy.** Kết quả mong đợi trùng từng ô với một kết quả sai là dấu hiệu nó được điền sau khi chạy.
> 3. **Nhật ký trống thì bài coi như chưa nộp.** Ba ô: thời gian thực tế, câu bạn nghĩ mình sai nhiều nhất, điều chưa hiểu.

---

## Bài học ngắn: rút không hoàn lại (đọc trước khi làm Phần 0)

**Ví dụ làm sẵn.** Hộp có 3 bi đỏ (Đ) và 2 bi xanh (X), tổng 5 bi. Rút 2 bi liên tiếp, **không hoàn lại** (bi đã rút **không** được bỏ trở lại). `P(cả hai đỏ)`?
- Lần 1: 3 đỏ trong 5 bi, nên `3/5`.
- Sau khi lấy ra 1 bi đỏ: trong hộp còn **4 bi** (mẫu số giảm 1), trong đó **2 đỏ** (tử số giảm 1 vì bi vừa lấy cũng là đỏ).
- Lần 2: `2/4`. Quy tắc nhân: `P(cả hai đỏ) = 3/5 × 2/4 = 6/20 = 3/10 = 0,3`.
- `P(bi 2 đỏ | bi 1 đỏ) = 2/4 = 0,5`, đọc thẳng từ hộp sau lần 1, không cần công thức dài.

**Kiểm tra bằng liệt kê (làm mỗi lần):** đặt tên 5 bi là Đ1, Đ2, Đ3, X1, X2. Các cặp không phân biệt thứ tự: Đ1Đ2, Đ1Đ3, Đ2Đ3, Đ1X1, Đ1X2, Đ2X1, Đ2X2, Đ3X1, Đ3X2, X1X2, tổng **10 cặp**, trong đó **3 cặp** toàn đỏ. `3/10 = 0,3`. Khớp.

**Lỗi hay gặp:** quên rằng bi đã rút ra khỏi hộp nên mẫu số cũng phải giảm. Viết `3/5 × 2/5` (mẫu số 5 ở cả hai lần) là coi như có hoàn lại. Hôm qua bạn viết `5/8 × 4/8`, cùng lỗi này.

**Trước khi nhân, luôn viết ra:** lần 1 còn ___ bi (___ đỏ); lần 2 còn ___ bi (___ đỏ).

---

## Phần 0: Sửa nợ hôm qua (bài mới)

**0.1.** (Lỗi hôm qua: rút không hoàn lại, mẫu số.) Hộp có **4 bi đỏ và 6 bi xanh**. Rút hai bi liên tiếp, **không hoàn lại**.
(a) Sau khi lần 1 rút được một bi xanh, trong hộp còn 9 bi, trong đó 5 bi xanh.
(b) Tính `P(cả hai xanh)` và `P(bi thứ hai xanh | bi thứ nhất xanh)`.
Công thức: P(cả hai xanh) = P(bi 1 xanh) * P(bi 2 xanh)
P(bi thứ 2 xanh | bi thứ 1 xanh) = P(cả hai xanh) / P(bi thứ 1 xanh)
Kết quả (phân số và thập phân 3 chữ số):
P(cả 2 xanh) = (6/10) * (5/9) = 1/3 = 0.333
P(bi thứ 2 xanh | bi thứ 1 xanh) = (1/3) / (6/10) = 5/9 = 0.556
(c) Chạy mô phỏng dưới đây, điền bảng. Dòng cuối của bảng (bi thứ hai xanh khi chưa biết bi thứ nhất) làm ở **Khối 2**, sau khi đọc bài học xác suất toàn phần, rồi quay lại điền.
```python
import numpy as np
rng = np.random.default_rng(1)
box = np.array([1]*6 + [0]*4)      # 1 = xanh (6 bi), 0 = đỏ (4 bi)
n = 100000
both = first = second = 0
for _ in range(n):
    d = rng.choice(box, size=2, replace=False)
    first += d[0]
    second += d[1]
    both += d[0] * d[1]
print("P(cả hai xanh)         ≈", both / n)
print("P(bi2 xanh | bi1 xanh) ≈", both / first)
print("P(bi thứ hai xanh)     ≈", second / n)
```

| đại lượng                                 | tính tay | mô phỏng | hiệu    | chấp nhận được (< 0,01)? |
| ----------------------------------------- | -------- | -------- | ------- | ------------------------ |
| P(cả hai xanh)                            | 0.333    | 0.33436  | 0.00136 | chấp nhận                |
| P(bi2 xanh \| bi1 xanh)                   | 0.556    | 0.55620  | 0.0002  | chấp nhận                |
| P(bi thứ hai xanh), chưa biết bi thứ nhất | 0.6      | 0.60029  | 0.00029 | chấp nhận                |


Nếu có dòng không chấp nhận được: lỗi nằm ở tính tay hay code, và bạn tìm ra bằng cách nào?
Trả lời: cả 2 đều chấp nhận được nhưng nếu lỗi xảy ra thì tôi sẽ tính tay lại vì đây là code của ông còn nếu là code tôi thì đọc lại code rồi debug 

**0.2.** (Lỗi hôm qua: nhầm hợp với giao, và nhầm "không xung khắc" với "độc lập".) Tung một xúc xắc. `A` = "ra số chẵn", `B` = "ra số ≤ 3".
Nhắc: `∩` = **và**, `∪` = **hoặc**.
Liệt kê tập: `A = {...}`, `B = {...}`, `A ∩ B = {...}`, `A ∪ B = {...}`
A = {2,4,6}, B = {1,2,3}, A ∩ B = {2}, A ∪ B = {1,2,3,4,6}
Tính `P(A ∩ B)` và `P(A ∪ B)` **bằng đếm tập**, rồi `P(A ∪ B)` **bằng công thức**. Hai cách phải trùng:
P(A ∩ B) = 1/6
P(A ∪ B) = 5/6
P(A ∪ B) = P(A) + P(B) - P(A ∩ B) = (3/6) + (3/6) - (1/6) = 5/6
Độc lập không? Kiểm bằng công thức `P(A)·P(B)` so với `P(A ∩ B)`. (Lý do "hai biến cố có thể cùng xảy ra" **không** được chấp nhận.)
Trả lời: A và B không độc lập vì thứ 1 P(A) * P(B) = 1/4 P(A ∩ B) = 1/6 2 kết quả khác nhau, thứ 2 khi biến cố A xảy ra tức ra mặt số 2 thì biến cố B cũng xảy ra vì giá trị nhỏ hớn hoặc bằng 3 đồng nghĩa việc biến cố A xảy ra có ảnh hưởng đến biến cố B xảy ra
Câu chốt: hôm qua (A6) `A` = "số lẻ" và `B` = "số ≥ 3" **độc lập**; câu này (và ví dụ chẵn, > 3 trong bài học hôm qua) **không độc lập**. Cả ba cặp đều **không xung khắc**. Điều đó cho thấy gì về mối liên hệ giữa "không xung khắc" và "độc lập"?
Trả lời: không xung khắc là việc 2 biến cố có thể cùng xảy ra hoặc 1 biến cố xảy ra biến cố còn lại không xảy ra chứ không phải biến cố A xảy ra biến cố B không xảy ra tức 2 biến cố luôn không cùng xảy ra. Độc lập là việc xảy ra hoặc không xảy ra của biến cố này không ảnh hưởng đến việc xảy ra hay không xảy ra của biến cố kia. Lấy ví dụ nếu tung 1 con xúc xắc với A là số lẻ, B là số >= 3 thì A và B không xung khắc, nhưng nếu tung 2 con xúc sắc thì A là xúc xắc 1 ra mặt số lẻ B là xúc sắc 2 ra mặt >= 3 thì lại là độc lập 

### ▶ KHỐI 2 (chiều)

**0.3.** (Lỗi hôm qua: DB3, `AVG ... OVER` kèm `ORDER BY`. Đề hôm qua viết "chênh lệch ... so với trung bình" chưa nói rõ chiều trừ, đề lần này đã nói rõ.) Bảng `orders`: khách 1 có hai đơn (500, 700), khách 2 có một đơn (300), khách 3 có hai đơn (900, 400). Với mỗi đơn, hiển thị `order_id`, `customer_id`, `amount` và `chenhlech` = **amount trừ trung bình amount của cả khách đó**. Sắp xếp theo `order_id`.
Điều kiện đề (chép nguyên văn): hiển thị order_id, customer_id, amount và chenhlech = amount trừ trung bình amount của các khách đó. Sắp xếp theo order_id
Trung bình amount của từng khách (tính tay **trước**): khách 1 = 600 ; khách 2 = 300; khách 3 = 650
Kết quả mong đợi (cột `chenhlech`, đơn 1 đến 5, tính tay **trước khi chạy**):

| order_id | chenhlech |
| -------- | --------- |
| 1        | -100      |
| 2        | 100       |
| 3        | 0         |
| 4        | 250       |
| 5        | -250      |

```sql
-- Trả lời:
SELECT order_id, customer_id, amount, amount - AVG(amount) OVER (PARTITION BY customer_id) AS chenhlech FROM orders ORDER BY order_id
```
Kết quả chạy (dán nguyên văn), khớp hay không khớp **từng ô**: kết quả khớp
order_id | customer_id | amount | chenhlech  
------------------------------  
1 | 1 | 500 | -100.0  
2 | 1 | 700 | 100.0  
3 | 2 | 300 | 0.0  
4 | 3 | 900 | 250.0  
5 | 3 | 400 | -250.0
Giải thích: nếu thêm `ORDER BY order_id` vào trong `OVER (...)` thì `AVG` đổi nghĩa thế nào? Cho giá trị của hàm `AVG` ở đơn 2 trong hai trường hợp (có và không có `ORDER BY` trong `OVER`).
Trả lời: thêm ORDER BY order_id vào trong OVER thì thay vì AVG 1 nhóm customer_id như không có ORDER BY thì AVG 1 dòng trong orders xong trượt xuống dòng dưới tính AVG của dòng trước đó và của dòng hiện tại
AVG không thêm ORDER BY:
order_id | customer_id | amount | chenhlech  
------------------------------  
1 | 1 | 500 | -100.0  
2 | 1 | 700 | 100.0  

AVG thêm ORDER BY:
order_id | customer_id | amount | chenhlech  
------------------------------  
1 | 1 | 500 | 0.0  
2 | 1 | 700 | 100.0

**0.4.** (Lỗi hôm qua: DA3, đọc ngược chiều sắp xếp.) Truy vấn: `ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY amount ASC) AS rn`, rồi lọc `rn = 1` bằng truy vấn con.
Điều kiện đề: `ASC` nghĩa là (tăng dần hay giảm dần, từ giá trị nào đến giá trị nào): ASC nghĩa là tăng dần từ giá trị thấp cho đến giá trị cao
Các amount của từng khách, **sắp xếp theo `ASC`**: khách 1: 500 ; khách 2: 300 ; khách 3: 400
Dự đoán `order_id` trả về:

| order_id |
| -------- |
| 1        |
| 3        |
| 5        |

Chạy, khớp hay không khớp: kết quả khớp
order_id | customer_id | amount | rn  
------------------------------  
1 | 1 | 500 | 1  
3 | 2 | 300 | 1  
5 | 3 | 400 | 1

---

## Bài học: Xác suất 2

### 1. Xác suất toàn phần
Nếu các trường hợp `A₁, A₂` **loại nhau và phủ hết mọi khả năng** (ví dụ "bệnh" và "không bệnh"), thì xác suất của một biến cố `B` bằng tổng theo từng trường hợp:
```
P(B) = P(B | A₁)·P(A₁) + P(B | A₂)·P(A₂)
```
Ý nghĩa: "tính B xảy ra trong từng trường hợp, nhân với xác suất của trường hợp, rồi cộng lại".

### 2. Định lý Bayes (đảo chiều điều kiện)
```
P(A | B) = P(B | A) · P(A) / P(B)
```
trong đó mẫu số `P(B)` thường tính bằng xác suất toàn phần. Bayes cho phép đi từ "nếu bệnh thì xét nghiệm dương tính" sang "xét nghiệm dương tính thì bệnh". **Hai điều này khác nhau**, nhầm chúng là lỗi phổ biến nhất.

### 3. Độ nhạy, độ đặc hiệu
- **Độ nhạy** (sensitivity) = `P(dương tính | có bệnh)`: trong số người có bệnh, bao nhiêu phần được phát hiện.
- **Độ đặc hiệu** (specificity) = `P(âm tính | không bệnh)`: trong số người không bệnh, bao nhiêu phần được kết luận đúng là không bệnh.
- **Dương tính giả** = `P(dương tính | không bệnh)` = `1 − độ đặc hiệu`.

### 4. Ví dụ làm sẵn (chép lại bằng tay, rồi che đi làm lại)
Một bệnh hiếm: 1% dân số mắc. Xét nghiệm có độ nhạy 90% và độ đặc hiệu 95%. Một người dương tính. Xác suất người đó thật sự mắc?
- `P(bệnh) = 0,01`; `P(+ | bệnh) = 0,9`; `P(+ | không bệnh) = 1 − 0,95 = 0,05`.
- `P(+) = 0,9·0,01 + 0,05·0,99 = 0,009 + 0,0495 = 0,0585`.
- `P(bệnh | +) = 0,009 / 0,0585 ≈ 0,154`.

**Kiểm tra bằng tần suất tự nhiên (cách kiểm tra bạn nên làm mọi lần):** trong 10.000 người: 100 người mắc, 90 người trong số đó dương tính; 9.900 người không mắc, 495 người trong số đó dương tính (dương tính giả). Tổng dương tính 585, trong đó thật sự mắc 90. `90/585 ≈ 0,154`. Khớp.
**Bài học:** xét nghiệm "đến 90%" mà người dương tính chỉ có khoảng 15% mắc, vì bệnh hiếm (tỷ lệ nền thấp), số dương tính giả nhiều hơn số dương tính thật.

### 5. Kỳ vọng và phương sai
- **Kỳ vọng:** `E[X] = Σ x·P(X = x)` (trung bình có trọng số theo xác suất).
- **Phương sai:** `Var(X) = E[X²] − (E[X])²`. **Độ lệch chuẩn** = căn bậc hai của phương sai.
- **Biến đổi tuyến tính:** `E[aX + b] = a·E[X] + b` và `Var(aX + b) = a²·Var(X)` (cộng thêm hằng số `b` không đổi độ phân tán).

**Ví dụ làm sẵn (một xúc xắc):** `E[X] = (1+2+3+4+5+6)/6 = 3,5`. `E[X²] = (1+4+9+16+25+36)/6 = 91/6`. `Var(X) = 91/6 − 3,5² = 91/6 − 49/4 = 35/12 ≈ 2,917`.

---

## Bài tập A: Xác suất (viết công thức rồi mới thay số)

**A1.** Nhà máy có hai máy. Máy M1 sản xuất 60% sản phẩm, tỉ lệ lỗi 2%. Máy M2 sản xuất 40% sản phẩm, tỉ lệ lỗi 5%. Chọn ngẫu nhiên một sản phẩm.
(a) `P(lỗi)`. (b) Biết sản phẩm lỗi, `P(do M2)`.
Công thức: P(lỗi) = P(lỗi | M1) * P(M1) + P(lỗi | M2) * P(M2)
P(M2 | lỗi) = P(lỗi | M2) * P(M2) / P(lỗi)
Kết quả: P(lỗi) = 0.032, P(M2 | lỗi) = 0.625
Kiểm tra bằng tần suất tự nhiên (10.000 sản phẩm): M1 làm 6000, lỗi 120 ; M2 làm 4000, lỗi 200 ; tổng lỗi 320 ; trong số lỗi, do M2 = 200 / 320 = 0.625

**A2.** Bộ lọc thư rác. 30% email là spam. Nếu là spam, bộ lọc gắn cờ với xác suất 0,95. Nếu **không** phải spam, vẫn bị gắn cờ nhầm với xác suất 0,10.
(a) Ghi: độ nhạy = 95% ; độ đặc hiệu = 90% ; tỉ lệ dương tính giả = 10%
(b) `P(bị gắn cờ)`. (c) `P(spam | bị gắn cờ)`.
Công thức:P(bị gán cờ) = P(bị gán cờ | thư spam) * P(thư spam) + P(bị gán cờ | thư thường) * P(thư thường)
P(spam | bị gán cờ) = P(bị gán cờ | spam) * P(spam) / P(bị gán cờ)
Kết quả: P(bị gắn cờ) = 0.355
P(spam | bị gán cờ) = 0.8029
Kiểm tra bằng tần suất tự nhiên (1.000 email): thư thường 700, gắn cờ spam thư thường 70 ; thư spam 300, gắn cờ thư spam vào thư spam 285 ; tổng số thư gán cờ thư spam 355 ; trong số thư gán cờ thư spam, thư bị gắn cờ thư spam là thư spam = 285 / 355 = 0.8029

**A3.** Vì sao "độ nhạy 95%" **không** có nghĩa "một email bị gắn cờ thì 95% là spam"? Dùng số của A2 để giải thích.
Trả lời: Độ nhạy ở đây được hiểu là trong số các thư spam tỉ lệ phát hiện và gán cờ thư spam là 95% nhưng khi xét trong số các thư bị gán cờ thì khác vì trong số nhưng thư bị gán cờ chứa cả những thư là thư rác thật và thư thường đồng nghĩa độ nhạy 95% không phải ảnh rằng những thư bị gán cờ là 95% là spam, con số thực chất là 80,29% tức trong số 10000 là thư gán cờ là spam thì có đến 8029 chính xác là thư spam và 1971 là thư thường bị gán nhầm là thư spam

### ▶ KHỐI 3 (tối)

**A4.** Tung một đồng xu công bằng **hai lần**. `X` = số lần ngửa.
Bảng phân phối (điền đủ):

| x        | 0   | 1   | 2    |
| -------- | --- | --- | ---- |
| P(X = x) | 0   | 0.5 | 0.25 |

Công thức `E[X]`: E[X] = x0 * P(X = x0) + x1 * P(X = x1) + x2 * P(X = x3)
Kết quả `E[X]`:  E[X] = 1

**A5.** Trò chơi: tung một xúc xắc. Ra 6 thì nhận 10 nghìn; ra 4 hoặc 5 thì nhận 2 nghìn; ra 1, 2, hoặc 3 thì **mất** 3 nghìn. `X` = số tiền nhận được (nghìn đồng, âm là mất).
Bảng phân phối của `X`:

| x        | 10000 | 2000 | -3000 |
| -------- | ----- | ---- | ----- |
| P(X = x) | 1/6   | 1/3  | 1/2   |

`E[X]` (công thức, rồi số): E[X] = x0 * P(X = x0) + x1 * P(X = x1) + x2 * P(X = x3) = 10000 * 1/6 + 2000 * 1/3 + (-3000) * 1/2 = 833.333
Chơi 100 ván, **kỳ vọng** tổng tiền là bao nhiêu? Có nên chơi không, và "kỳ vọng" bảo đảm điều gì, **không** bảo đảm điều gì?
Trả lời: chơi 100 ván thì kỳ vọng tổng tiền là 83333.333 vế sau thì tôi chưa biết 

**A6.** (a) Tính `Var(X)` cho đồng xu ở A4 bằng `E[X²] − (E[X])²`. Công thức, rồi số: Var(X) = E[X²] − (E[X])² = 21805556,11
(b) Biết `E[X] = 3` và `Var(X) = 4`. Tính `E[2X + 1]` và `Var(2X + 1)`.
Công thức:  E[2X + 1] = 2 * E[X] + 1, Var(2X + 1) = 4 * Var(X)
Kết quả: E[2X + 1] = 7, Var(2X + 1) = 16

---

## Bài tập B: Tự giải thích (không nhìn tài liệu)

**B1.** Ở A2, `P(spam | gắn cờ)` ≈ 0,80. Nếu spam hiếm hơn, chỉ 3% email là spam (độ nhạy và dương tính giả giữ nguyên), `P(spam | gắn cờ)` sẽ **lớn hơn hay nhỏ hơn** 0,80? Dự đoán **trước**, rồi tính để kiểm tra.
Dự đoán: nhỏ hơn
Tính: P(bị gán cờ) = P(bị gán cờ | thư spam) * P(thư spam) + P(bị gán cờ | thư thường) * P(thư thường) = 0.95 * 0.03 + 0.1 * 0.97 = 0.1255
P(spam | bị gán cờ) = P(bị gán cờ | spam) * P(spam) / P(bị gán cờ) = 0.227
Rút ra bài học gì về "tỷ lệ nền"? tôi chưa rõ

---

## Bài học: dict, set, tuple

```python
# dict: tra cứu theo KHÓA
d = {"an": 20, "binh": 25}
d["chi"] = 30                # thêm khóa mới (hoặc ghi đè nếu khóa đã có)
d["an"] += 1                 # sửa giá trị
d.get("dung", 0)             # trả 0 nếu khóa thiếu (d["dung"] sẽ báo KeyError)
"an" in d                    # kiểm tra KHÓA, không phải giá trị
for k, v in d.items():       # duyệt cả khóa và giá trị
    print(k, v)

# set: tập hợp, KHÔNG trùng, KHÔNG có thứ tự đáng tin cậy
s = {1, 2, 2, 3}             # -> {1, 2, 3}
t = {3, 4}
s & t                        # giao  -> {3}
s | t                        # hợp   -> {1, 2, 3, 4}
s - t                        # hiệu  -> {1, 2}
len(set([1, 1, 2]))          # số phần tử khác nhau -> 2
type({})                     # dict (!). Tập rỗng phải viết set()

# tuple: như list nhưng BẤT BIẾN
p = (1, 2)
x, y = p                     # unpacking: x = 1, y = 2
x, y = y, x                  # đổi chỗ hai biến
(5,)                         # tuple một phần tử cần dấu phẩy
```
- **Khóa của dict phải bất biến** (số, chuỗi, tuple), nên **list không làm khóa được**, tuple làm được.
- `in` trên `set` và `dict` nhanh hơn trên `list` (với dữ liệu lớn), vì dùng bảng băm (hash).
- Liên hệ với Ngày 1: tuple bất biến nên `t[0] = 9` báo lỗi, giống số và chuỗi. list khả biến nên sửa tại chỗ được.

---

## Bài tập C: Dự đoán đầu ra

**C1.**
```python
d = {"a": 1, "b": 2}
d["c"] = 3
d["a"] += 10
print(d, len(d), "b" in d, 2 in d)
```
Dự đoán: {"a" : 11, "b": 2, "c" : 3} , 3, true, false
Đúng hay sai, vì sao: dự đoán đúng vì giá trị tại khóa a được cộng thêm 10 và thêm 1 khóa c với giá trị 3, nên mới có kết quả d như trên, kích cỡ là 3 vì có 3 cặp khóa và giá trị, khóa b có trong d nhưng khóa 2 thì không

**C2.**
```python
s = {1, 2, 2, 3}
t = {3, 4}
print(len(s), s & t, s | t, s - t)
```
(Thứ tự phần tử trong set không quan trọng.)
Dự đoán: 3, {3}, {1, 2, 3, 4}, {1, 2}
Đúng hay sai, vì sao: dự đoán đúng vì kết quả len chỉ tính là số lượng các phần tử là duy nhất trong s, s & t thì chỉ có 3 là phần tử có ở cả s và t, s | t thì gộp tất cả phần từ của s và t lại, s - t thì chỉ những phần từ thuộc s chứ không thuộc t

**C3.**
```python
t = (1, 2, 3)
a, b, c = t
b = 20
print(t, a, b, c)
```
Dự đoán: (1,2,3) 1, 20, 3
Đúng hay sai, vì sao: dự đoán đúng vì a, b, c ban đầu nhận giá trị của các phần tử trong t, sau đó b đổi giá trị thành 20 nhưng t là tuple tức nó bất biến nên việc sửa giá trị của b không thay đổi t tức t được giữ nguyên a, c được giữ nguyên mỗi b thay đổi

**C4.**
```python
t = (1, 2, 3)
t[0] = 9
```
Dự đoán (kết quả hoặc loại lỗi): ra lỗi
Giải thích bằng **định nghĩa** của bất biến:bất biến là việc không sửa giá trị tại chỗ được, với list  chúng ta sửa giá trị tại chỗ như t[0] = 9 được vì list là khả biến nhưng tuple thì không cho phép làm điều đó

---

## Bài tập D: Viết code (điền ô điều kiện trước)

**D1.** Cho `words = ["a", "b", "a", "c", "b", "a"]`. Dùng **dict** đếm số lần xuất hiện của từng từ (không dùng thư viện).
Điều kiện đề (chép nguyên văn): dùng dict đếm số lần xuất hiện của từng từ 
Kết quả mong đợi (tính tay):
{"a" : 3, "b" : 2, "c" : 1}
```python
# Trả lời:
dict = {}
for i in words:
    count = dict.get(i, 0)
    if i in dict:
        dict[i] = count + 1
    else:
        dict[i] = 1

print(dict)
```
Kết quả chạy, khớp hay không khớp (từng ô): kết quả chạy khớp
{'a': 3, 'b': 2, 'c': 1}


**D2.** Cho `emails = ["x@a.com", "y@b.com", "x@a.com", "z@c.com"]`. Tính **một con số**: số email **khác nhau**.
Điều kiện đề (chép nguyên văn): tìm số lượng email khác nhau
Kết quả mong đợi (tính tay): 3
```python
# Trả lời:
s = set()

for i in emails:
    s.add(i)

print(len(s))
```
Kết quả chạy, khớp hay không khớp: kết quả khớp

**D3.** Cho `a = {1, 2, 3, 4}` và `b = {3, 4, 5}`. Lấy các phần tử **chỉ có trong `a`, không có trong `b`**.
Điều kiện đề (chép nguyên văn): lấy các phần tử chỉ có trong a không có trong b
Kết quả mong đợi (tính tay): {1,2}
```python
# Trả lời:
c = a - b
```
Kết quả chạy, khớp hay không khớp: kết quả khớp

**D4.** Vì sao `{[1, 2]: "x"}` báo lỗi còn `{(1, 2): "x"}` thì chạy được? Dùng khái niệm bất biến/khả biến, **định nghĩa** rồi mới kết luận.
Trả lời: khóa trong dict là bất biến tức không được phép sửa tại chỗ mà list là khả biến tức được phép sửa giá trị tại chỗ nên số, tuple, chuỗi làm khóa ví dụ {(1, 2): "x"} thì được còn list làm khóa cho dict thì báo lỗi ngay

---

## Ghi nhật ký (bắt buộc điền cả ba)
- Thời gian học thực tế (giờ bắt đầu và kết thúc): sáng 8h - 8h45, chiều 13h - 14h, tối 20h - 20h45
- Câu bạn nghĩ mình sai nhiều nhất, và vì sao: A4 vì đề nói công bằng nên tôi chưa rõ là độc lập tức tung lần 2 thì xác suất có bị ảnh hưởng từ lần 1 không, A5 vì tôi cảm giác vậy chứ chư chắc chắn, B1 tôi chưa rõ được
- Điều chưa hiểu cần hỏi Claude (nếu viết "không có" thì phải nêu đã kiểm tra những câu nào bằng cách chạy code): tôi đã ghi không biết không rõ trong các phần tôi làm, hoặc trong chỗ này do đến đây tôi mệt nên không viết rõ ra

> Làm xong gửi nguyên file này cho Claude chấm. Ngày mai (24/10) là **bài cổng Tuần 2** (SQL, xác suất cơ bản, dict/set/tuple, 90 phút, không tra cứu). Cổng cần từ 70%. Hôm nay đúng bằng nội dung cổng, nên hôm nay làm kỹ.
