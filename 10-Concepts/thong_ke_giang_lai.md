---
title: Thống kê giảng lại từ đầu (3 buổi nhỏ)
tags: [thong-ke, giang-lai, tuan-3]
---

# Thống kê giảng lại từ đầu

**Cách học khác lần trước:** không bắt đầu từ công thức. Mỗi buổi bắt đầu từ một câu hỏi cụ thể, bạn **tự chạy mô phỏng để thấy con số**, rồi công thức chỉ là "đường tắt" cho kết quả mô phỏng. Công thức nào cũng có ví dụ làm sẵn trước khi có câu hỏi.

**Ba buổi, mỗi buổi khoảng 30 phút, mỗi ngày một buổi:**
- **Buổi 1 (27/10):** độ lệch chuẩn (đo độ "tản" của dữ liệu). Kèm 5 phút khởi động.
- **Buổi 2 (28/10):** sai số chuẩn và khoảng tin cậy (trung bình mẫu dao động bao nhiêu).
- **Buổi 3 (29/10):** kiểm định và p-value (kết quả này có đáng ngờ không).

**Quy ước làm tròn (để hết lệch nhau):** làm tròn bình thường đến **số chữ số đề yêu cầu** (chữ số kế tiếp từ 5 trở lên thì làm tròn lên). Khi chấm, **lệch không quá 0,001 so với đáp án tôi tính thì vẫn tính đúng**. Đề nào cần số chính xác hơn sẽ ghi rõ.

**Phạm vi:** chỉ dùng những gì có trong bài học cùng buổi. Câu nào cần thứ chưa học hoặc đề mơ hồ: ghi vào **ô trả lời** "ngoài phạm vi" hoặc "đề mơ hồ vì ...", làm theo cách hiểu hợp lý, tôi chấm theo cách đó.

Quy tắc cũ vẫn giữ: ô "dự đoán" điền **trước khi chạy**; code là trọng tài; nhật ký điền đủ ba ô; mỗi ngày không quá 3 giờ.

---

# BUỔI 1: Độ lệch chuẩn

## Khởi động 5 phút (không tính thời gian chính)
**W1.** Bảng `sales(sale_id, product_id, qty)`. Viết câu `SELECT` cho cột `c` là tổng `qty` **cộng dồn** theo thứ tự `sale_id` (dòng 1: `qty` của dòng 1; dòng 2: tổng hai dòng đầu; ...). Dự đoán, rồi chạy.
Dự đoán

| c   |
| --- |
| 10  |
| 15  |
| 22  |
| 24  |

```sql
-- Trả lời:
SELECT SUM(qty) OVER ( ORDER BY sale_id) AS c FROM sales
```
Kết quả: khớp
c  
------------------------------  
10  
15  
22  
24

**W2.** Hộp có 5 bi đỏ và 5 bi xanh. Rút hai bi liên tiếp, không hoàn lại.
(a) Nếu lần 1 rút **xanh**, hộp còn 9 bi, trong đó 4 bi xanh.
(b) `P(lần 2 xanh | lần 1 xanh)` = 4/9 = 0.444 (phân số, rồi thập phân 3 chữ số) 
(c) `P(cả hai xanh)`: công thức, rồi số: P(cả hai xanh) = P(lần 1 xanh) * P(lần 2 xanh | lần 1 xanh) = (5/10) * (4/9) = 2/9 = 0.222

## 1. Vì sao cần độ lệch chuẩn
Hai nhóm, **cùng trung bình 50**:
- Nhóm A: 48, 50, 52 (các số sát nhau)
- Nhóm B: 10, 50, 90 (các số rất xa nhau)

Trung bình không phân biệt được A và B. Ta cần một con số trả lời: **"các giá trị thường cách trung bình bao xa?"**. Con số đó là **độ lệch chuẩn**.

## 2. Tính từng bước, bằng lời thường
Làm với nhóm A = 48, 50, 52:
1. **Trung bình:** (48 + 50 + 52)/3 = 50.
2. **Lệch** của mỗi số so với trung bình: −2, 0, +2.
3. **Bình phương** mỗi lệch: 4, 0, 4. Cộng lại: **8**. (Vì sao phải bình phương? Cộng thẳng các lệch được −2 + 0 + 2 = 0, luôn bằng 0 với mọi dữ liệu, nên không đo được gì. Bình phương làm mọi số dương.)
4. **Chia cho số lượng** để lấy "bình phương lệch trung bình", gọi là **phương sai**.
5. **Lấy căn bậc hai** để quay về cùng đơn vị với dữ liệu: đó là **độ lệch chuẩn**.

**Chia cho bao nhiêu ở bước 4?** Chỉ một quy tắc:
- Dữ liệu là **toàn bộ** (cả lớp, cả tập) thì chia cho **n**. Gọi là độ lệch chuẩn **tổng thể**.
- Dữ liệu là một **mẫu** lấy từ tập lớn hơn thì chia cho **n − 1**. Gọi là độ lệch chuẩn **mẫu**. (Lý do ngắn gọn: trung bình được tính từ chính mẫu đó nên các số nằm gần nó hơn thực tế, phép tính bị thấp đi một chút; chia cho n − 1 bù lại.)
- **`n` là số phần tử** (đếm), không phải trung bình, không phải tổng.

Nhóm A: tổng bình phương lệch = 8, n = 3.
- Tổng thể: `8/3 = 2,667`, căn = **1,633**.
- Mẫu: `8/2 = 4`, căn = **2**.

Nhóm B = 10, 50, 90: lệch −40, 0, +40; bình phương 1600, 0, 1600, tổng 3200.
- Tổng thể: `3200/3 = 1066,7`, căn = **32,660**.
- Mẫu: `3200/2 = 1600`, căn = **40**.

B có độ lệch chuẩn lớn hơn A rất nhiều: đúng với cảm nhận "B tản hơn". Đó là ý nghĩa của con số.

**Bẫy trong code:** `np.std(x)` chia cho **n**. `pd.Series(x).std()` chia cho **n − 1**. Cùng tên, khác kết quả. Muốn `np.std` chia n − 1 thì `np.std(x, ddof=1)`.
```python
import numpy as np, pandas as pd
x = [48, 50, 52]
print(np.std(x), pd.Series(x).std())     # 1.633 và 2.0
```

## Bài tập
**D1.** `y = [4, 8, 12, 16]`. Làm đủ từng dòng, **tính tay trước, chạy sau**.
(a) `n` (đếm): 4
(b) Trung bình: 10
(c) Lệch của từng số so với trung bình:  -6, -2, 2, 6
(d) Tổng bình phương lệch: 80
(e) Độ lệch chuẩn **tổng thể** (công thức, rồi số 3 chữ số thập phân): độ lệch tổng thể = căn 2 (tổng bình phương lệch / n) = 4.472
(f) Độ lệch chuẩn **mẫu** (công thức, rồi số 3 chữ số thập phân): độ lệch chuẩn mẫu = căn 2 (tổng bình phương lệch / (n - 1)) = 5.164
(g) Dự đoán `np.std(y)` ra số ở (e) hay (f)? `pd.Series(y).std()` ra số nào? Chạy kiểm tra, khớp hay không: np.std(y) ra số ở (e) còn pd.Series(y).std() ra số ở (f) chạy kết quả khớp:  
4.47213595499958 5.163977794943222


**D2.** `z = [7, 7, 7, 7]`. Không cần tính dài: độ lệch chuẩn là bao nhiêu? Một câu giải thích theo ý nghĩa ("các số cách trung bình bao xa").
Trả lời: độ lệch chuẩn là 0 vì các số cách trung bình tức số 7 là bằng 0

**D3.** Nhóm C = 20, 30, 40, 50, 60 và nhóm D = 40, 45, 50, 55, 60. Không tính, chỉ nhìn: nhóm nào có độ lệch chuẩn lớn hơn? Một câu lý do. Rồi chạy `np.std` cả hai nhóm để kiểm tra.
Dự đoán: nhóm C có độ lệch chuẩn lớn hơn vì các số chênh lệch nhau trong dãy là 10 lớn hơn độ chênh lệch các số trong dãy nhóm D
Kết quả chạy: 14.142135623730951 7.0710678118654755

---

# BUỔI 2: Sai số chuẩn và khoảng tin cậy

## 1. Vấn đề
Bạn đo điểm của **100 học sinh**, trung bình **70**. Nhưng đó chỉ là **một mẫu**. Nếu chọn 100 học sinh **khác**, trung bình có thể là 69 hoặc 72. Câu hỏi: **trung bình mẫu dao động bao nhiêu giữa các lần chọn mẫu?** Con số đó là **sai số chuẩn (SE)**: chính là **độ lệch chuẩn của các trung bình mẫu**.

## 2. Tự thấy bằng mô phỏng (chạy đoạn này)
Giả sử cả trường có 100.000 học sinh, điểm có độ lệch chuẩn khoảng 15. Ta lặp lại 5000 lần: chọn n học sinh ngẫu nhiên, tính trung bình, rồi xem các trung bình đó **tản bao nhiêu**.
```python
import numpy as np
rng = np.random.default_rng(0)
pop = rng.normal(70, 15, 100000)          # cả trường
for n in (25, 100, 400):
    means = [rng.choice(pop, n).mean() for _ in range(5000)]
    print(n, np.std(means), 15 / np.sqrt(n))
```
Kết quả khoảng:
```
25   2.98   3.0
100  1.48   1.5
400  0.74   0.75
```
Cột giữa (độ lệch chuẩn của các trung bình mẫu, đo bằng mô phỏng) **khớp** cột cuối `15/√n`. Vậy công thức:

**`SE = s / √n`** (s: độ lệch chuẩn mẫu, n: cỡ mẫu).

Rút ra: mẫu **lớn gấp 4 lần** thì SE **giảm một nửa** (25 → 100: 3,0 → 1,5). Đo nhiều hơn thì trung bình ổn định hơn.

## 3. Khoảng tin cậy 95%
Các trung bình mẫu tản theo hình chuông quanh trung bình thật, với độ lệch chuẩn là SE. Với hình chuông, **khoảng 95% các giá trị nằm trong ±1,96 độ lệch chuẩn**. Từ đó:

**`Khoảng tin cậy 95% = trung bình mẫu ± 1,96 × SE`.**

**Ví dụ làm sẵn:** n = 100, trung bình mẫu = 70, s = 15. `SE = 15/√100 = 1,5`. `1,96 × 1,5 = 2,94`. Khoảng: `[67,06; 72,94]`.

**Cách đọc đúng:** nếu bạn lặp lại việc "chọn mẫu rồi dựng khoảng" nhiều lần, khoảng chứa trung bình thật khoảng **95% số lần**. Kiểm chứng:
```python
mu = pop.mean(); cover = 0
for _ in range(5000):
    s = rng.choice(pop, 100)
    se = s.std(ddof=1) / np.sqrt(100)
    cover += (s.mean() - 1.96*se <= mu <= s.mean() + 1.96*se)
print(cover / 5000)       # khoảng 0.95
```
**Đọc sai hay gặp:** "95% học sinh có điểm trong khoảng [67,06; 72,94]." **Sai.** Khoảng này nói về **trung bình**, không nói về từng học sinh (học sinh thì điểm tản rộng hơn nhiều, độ lệch chuẩn 15).

## Bài tập
**E1.** Một mẫu có `n = 36`, trung bình = 50, `s = 12`.
(a) `SE` (công thức, rồi số): SE = s / √n = 2
(b) Khoảng tin cậy 95% (công thức, rồi hai đầu mút, 2 chữ số thập phân): khoảng tin cậy 95% = trung bình +- 1.96 * SE = [32.08.29.92]  

**E2.** Một mẫu có `n = 144`, `s = 24`.
(a) `SE`: SE = s / √n = 2
(b) Nếu tăng cỡ mẫu lên `n = 576` (s giữ nguyên), `SE` mới là bao nhiêu? Khoảng tin cậy rộng hơn hay hẹp hơn, hẹp bao nhiêu lần? SE mới = 24 / √(576) = 1 < 2 => Khoảng tin cậy hẹp hơn và hẹp 1 nửa so với lúc n = 144

**E3.** Điểm có độ lệch chuẩn 10. **Dự đoán trước** độ lệch chuẩn của các trung bình mẫu khi n = 25 và khi n = 100 (dùng công thức). Rồi sửa đoạn mô phỏng mục 2 (dùng `rng.normal(70, 10, 100000)` và `n` trong `(25, 100)`), chạy và đối chiếu.
Dự đoán n = 25: SE = s / √n = 10 / √(25) = 2
Dự đoán n = 100: SE = s / √n = 10 / √(100) = 1
Kết quả chạy (dán nguyên văn): 
25 2.0
100 1.0
Khớp hay không khớp: kết quả khớp

**E4.** Đúng hay sai, một câu giải thích mỗi mệnh đề:
(a) "Khoảng tin cậy 95% cho trung bình là [46; 54], nghĩa là 95% học sinh có điểm từ 46 đến 54."
(b) "Tăng cỡ mẫu từ 100 lên 400 thì SE giảm còn một nửa."
(a): sai vì khoảng này dùng cho trung bình không phản ảnh việc 95% học sinh có điểm từ 46 đến 54
(b): đúng vì khi cỡ mấu tăng 4 lần tức s / √(4 * n) = s / (2 * √(n)) đồng nghĩa SE giảm đi 1 nửa

---

# BUỔI 3: Kiểm định và p-value

## 1. Câu hỏi
Bạn tung một đồng xu **100 lần**, được **62 ngửa**. Câu hỏi: **đồng xu có vấn đề không, hay chỉ là may rủi?**

Cách nghĩ: **giả sử đồng xu công bằng** (giả định này gọi là **H0**). Rồi hỏi: **trong "thế giới đồng xu công bằng", kết quả lệch như 62 ngửa có hiếm không?** Hiếm thì nghi H0 sai. Không hiếm thì chưa có lý do nghi.

## 2. Bước A: tự dựng "thế giới H0" bằng mô phỏng
Tung "100 lần" rất nhiều lần với đồng xu **thật sự công bằng**, đếm số ngửa mỗi lần, rồi xem **bao nhiêu phần** các lần đó lệch **ít nhất 12** so với 50 (tức 62 trở lên **hoặc** 38 trở xuống; hai phía vì 38 cũng đáng ngờ như 62).
```python
import numpy as np
rng = np.random.default_rng(0)
heads = rng.binomial(100, 0.5, size=200000)    # 200.000 lần "tung 100 lần", đồng xu công bằng
print((np.abs(heads - 50) >= 12).mean())       # khoảng 0.021
print(heads.std())                             # khoảng 5
```
Kết quả khoảng **0,021**: trong thế giới đồng xu công bằng, chỉ khoảng 2 lần trong 100 lần làm thí nghiệm ra kết quả lệch như vậy.

**Đó chính là p-value:** *"nếu H0 đúng, xác suất thấy dữ liệu lệch ít nhất như đã thấy"*. Ở đây `p ≈ 0,021`.

Quy ước: **`p < 0,05` thì bác bỏ H0** ("đủ hiếm để nghi"); **`p ≥ 0,05` thì chưa đủ bằng chứng bác bỏ**.

## 3. Bước B: đường tắt không cần mô phỏng
Mô phỏng thì chậm. Đường tắt là hỏi: **kết quả lệch bao nhiêu "độ lệch chuẩn"?**
1. Dưới H0, số ngửa kỳ vọng `100 × 0,5 = 50`, độ lệch chuẩn `√(100 × 0,5 × 0,5) = 5` (mô phỏng ở trên cũng ra `heads.std()` ≈ 5, khớp).
2. `z = (62 − 50)/5 = 2,4`: kết quả lệch 2,4 độ lệch chuẩn.
3. Từ buổi 2: các giá trị nằm trong ±1,96 độ lệch chuẩn khoảng 95% thời gian. Lệch **hơn 1,96** thì là "hiếm hơn 5%". Nên: `|z|` (bỏ dấu âm) **≥ 1,96 thì p ≤ 0,05, bác bỏ H0**; `|z|` **< 1,96 thì chưa bác bỏ**. Mốc 1%: **2,58**.
4. 2,4 ≥ 1,96 nên bác bỏ H0 ở mức 5%.

**Lưu ý trung thực:** đường tắt cho `p ≈ 0,016`, mô phỏng cho `p ≈ 0,021`. Hai số hơi khác (đường tắt là xấp xỉ), nhưng **cùng nằm dưới 0,05 nên kết luận như nhau**. Đề thi chỉ cần kết luận, các đề của tôi không đặt sát ngưỡng.

**Công thức đường tắt, nhắc lại:** đếm số ngửa trong n lần tung, mỗi lần ngửa với xác suất p: kỳ vọng `n × p`, độ lệch chuẩn `√(n × p × (1 − p))`. (Lý giải đã có ở file `buoi_hoc_tuan3_ngay1.md`, mục 1b, không cần đọc lại nếu bạn đã chấp nhận công thức.)

## 4. Viết kết luận đúng (và nhầm lẫn cần tránh)
- **Đúng:** "Nếu đồng xu công bằng thì kết quả lệch như vậy hiếm (p < 0,05), nên ở mức 5% ta bác bỏ H0."
- **Sai:** "Xác suất đồng xu công bằng là 2%." p là xác suất của **dữ liệu** khi giả sử H0, không phải xác suất của H0.
- **Sai:** "p = 0,30 chứng minh đồng xu công bằng." `p ≥ 0,05` chỉ là **chưa đủ bằng chứng bác bỏ**.

**Ví dụ làm sẵn (chưa bác bỏ):** 54 ngửa trong 100 lần. `z = (54 − 50)/5 = 0,8 < 1,96`, nên chưa đủ bằng chứng bác bỏ H0.

## Bài tập
**S1.** 100 lần tung, **64 ngửa**. H0: đồng xu công bằng.
(a) Kỳ vọng và độ lệch chuẩn dưới H0:
(b) `z` (ghi số 64 vào công thức):
(c) So với 1,96 và 2,58, rồi kết luận ở mức 5% (một câu theo mẫu ở mục 4):
(d) Mô phỏng: sửa số trong đoạn mục 2 để đếm lệch **ít nhất 14** (vì 64 − 50 = 14). Dự đoán trước p khoảng nhỏ hơn hay lớn hơn 0,05. Kết quả chạy (dán nguyên văn):
(e) Kết luận từ mô phỏng có khớp với kết luận ở (c) không?

**S2.** 100 lần tung, **55 ngửa**. H0: đồng xu công bằng.
(a) `z`:
(b) Kết luận ở mức 5% (một câu):
(c) Mô phỏng lệch **ít nhất 5**: dự đoán rồi chạy, dán nguyên văn kết quả:
(d) Với kết quả này, "đồng xu chắc chắn công bằng" có đúng không? Một câu.

**S3.** Mỗi mệnh đề đúng hay sai? Một câu giải thích.
(a) "p = 0,03 nghĩa là xác suất H0 đúng là 3%."
(b) "p = 0,03 nghĩa là: nếu H0 đúng thì xác suất thấy dữ liệu lệch ít nhất như đã thấy là 3%."
(c) "p = 0,40 nghĩa là chưa đủ bằng chứng bác bỏ H0."

---

## Ghi nhật ký (mỗi buổi điền đủ ba ô)
- Thời gian bắt đầu và kết thúc thực tế buổi 1:  7h45-8h15 buổi 2: 11h-11h30 buổi 3: 9h-9h30
- Câu bạn nghĩ mình sai nhiều khả năng nhất, và vì sao: bài buổi 1: D3 tôi nghĩ tôi giải thích không đúng ý ông vì đó là những gì tôi thấy về nhóm C và D
- Điều chưa hiểu (nếu viết "không có" thì nêu đã kiểm tra những câu nào bằng cách chạy code): buổi 2 không có vì đã kiểm chứng toàn bộ câu

> Làm xong mỗi buổi gửi lại phần đã làm cho Claude chấm. Có chỗ nào đọc không hiểu thì ghi **đúng câu đó** vào ô "Điều chưa hiểu", tôi sẽ giảng lại chỗ đó, không giảng lại cả buổi.
