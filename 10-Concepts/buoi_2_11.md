---
title: Thứ Hai 2/11 - Logic dãy số (đề đổi dạng) và ôn thống kê bằng cách đọc kết quả
tags: [logic, thong-ke, tuan-3]
---

# 2/11 (1 giờ): Logic dãy số (45 phút) + ôn thống kê bằng cách đọc kết quả (15 phút)

**Cách làm:** tính tay hoặc dự đoán trước, kiểm lại sau. Làm tròn bình thường, lệch ≤ 0,001 vẫn đúng. Đề mơ hồ thì ghi "đề mơ hồ vì ..." vào ô trả lời. Mọi câu chỉ dùng nội dung trong bài học bên dưới. **Nhật ký điền đủ ba ô bằng câu hoàn chỉnh**, ghi giờ thật.

**Về việc tránh quá khớp (bạn đã nói đúng):** từ file này, mỗi câu gắn nhãn **[đã gặp]** (cùng dạng bài trước, dùng để giữ trí nhớ) hoặc **[mới]** (dạng chưa làm, đo khả năng áp dụng sang tình huống lạ). Mình sẽ chấm và báo điểm **riêng hai loại**, giống train và test.

---

# PHẦN 1 (45 phút): Logic dãy số

## Bài học
**Cách làm (đã học, nhắc lại):** (1) tính **hiệu** hai số liền nhau; (2) hiệu không đổi thì là cấp số cộng; (3) không thì tính **tỉ số** (không đổi là cấp số nhân); (4) nếu hiệu thay đổi đều thì tính **hiệu của hiệu**; (5) thử dãy quen: số chính phương, lập phương, mỗi số bằng tổng hai số trước. **Luôn kiểm quy luật bằng tất cả số hạng đã cho**, không chỉ hai số đầu.

**Ba kiểu mới (có ví dụ làm sẵn):**
1. **Hai phép tính liên tiếp:** mỗi số bằng số trước **nhân 2 rồi cộng 1**. Ví dụ: `1, 3, 7, 15, ?`: 1×2+1 = 3, 3×2+1 = 7, 7×2+1 = 15, nên số tiếp theo là 15×2+1 = **31**.
2. **Hai dãy đan xen:** các số ở vị trí lẻ theo một quy luật, các số ở vị trí chẵn theo quy luật khác. Ví dụ: `2, 10, 4, 20, 6, 30, ?`: vị trí lẻ (1, 3, 5, ...): 2, 4, 6 (cộng 2); vị trí chẵn: 10, 20, 30 (cộng 10). Số tiếp theo ở vị trí 7 (lẻ): **8**.
3. **Công thức theo vị trí `n`:** thử xem số thứ `n` có phải `n² + 1`, `2n + 3`, ... không. Ví dụ: `2, 5, 10, 17, 26, ?`: vị trí 1 → 2 = 1²+1; vị trí 2 → 5 = 2²+1; vị trí 3 → 10 = 3²+1. Số thứ 6 là 6²+1 = **37**.

## Bài tập (mỗi câu ghi **số cần tìm** và **quy luật một câu**)
**L1. [đã gặp]** `6, 11, 16, 21, ?`
**L2. [đã gặp]** `2, 6, 18, 54, ?`
**L3. [mới]** `3, 7, 15, 31, ?`
**L4. [mới]** `5, 1, 10, 2, 15, 3, 20, ?`
**L5. [mới]** `2, 5, 10, 17, 26, ?` Rồi: số hạng thứ 10 của dãy là bao nhiêu (công thức rồi số)?
**L6. [mới]** `2, 3, 5, 9, 17, ?`
**L7. [mới]** `4, 9, ?, 25, 36`. Tìm số bị thiếu ở giữa.
**L8. [mới, tìm lỗi]** An nói: "Dãy `1, 2, 4, 7, ?` có quy luật nhân 2, nên số tiếp là 14." (a) An sai ở đâu (kiểm bằng số hạng nào)? (b) Quy luật đúng và số tiếp theo là gì?

Trả lời L1: 26 hiệu mỗi số là 5
L2: 162 tỷ số mỗi số là 3
L3: 63 số trước đó nhân với 2 cộng 1
L4: 4 các số ở vị trí lẻ có giá trị bằng thứ tự vị trí lẻ như vị trí lẻ 1,3,5 thì có thứ tự là 1,2,3 nhân với 5, còn các số ở vị trí chẵn thì điền thứ tụ vị trí chẵn ví dụ vị trí lẻ là 2,4,6 thì thứ tự là 1,2,3
L5: 37 vị trí của số * vị trí của số + 1, số hạng thứ 10 = vị trí của số * vị trí của số + 1 = 10 * 10 + 1 = 101
L6: 33 hiệu 2 số bằng 2 ^(vị trí số trước đó) ví dụ 5 - 3 = 2^1 
L7: 16 vị trí số * vị trí số
L8 (a): An sai vì số 7 không chia hết cho 2 tức số 7 được tạo ra không phải từ phép nhân với 2 của 1 trong số các số trong dãy
L8 (b): quy luật đúng là hiệu của hiệu, hiệu của hiệu 7 với 4 là 3 với hiệu của 4 và 2 là 2 ra kết quả là 1 vậy số tiếp theo là số 7 + 4 = 11 với hiệu của 4 với hiệu của 7 với 4 là 3 bằng 1

---

# PHẦN 2 (15 phút): ôn thống kê bằng cách **đọc kết quả và kiểm ngược**

**Mẹo kiểm ngược (có ví dụ):** sau khi tính xong một phép chia, **nhân ngược lại** xem có ra số ban đầu không. Ví dụ làm sẵn: một bạn tính `z = (41 − 50)/5 = −2`. Kiểm ngược: `−2 × 5 + 50 = 40`, không phải 41, nên sai. Đúng: `−9/5 = −1,8` (kiểm: `−1,8 × 5 + 50 = 41`).

**S1. [mới]** Ba mẫu cùng có độ lệch chuẩn mẫu `s = 10`, mỗi mẫu cho một khoảng tin cậy 95%: A = `[48,0; 52,0]`, B = `[49,0; 51,0]`, C = `[47,5; 52,5]`.
(a) Mẫu nào có cỡ mẫu `n` **lớn nhất**? Một câu lý do (nhắc "độ rộng khoảng" và SE).
(b) B so với A: độ rộng khoảng giảm một nửa thì cỡ mẫu gấp khoảng mấy lần? (nhớ: n tăng 4 lần thì SE giảm một nửa)
(a): ta biết khoảng tin cậy 95% = trung bình +- 1.96 * SE => trung bình = (vế trái khoảng + vế phải khoảng) / 2 = (trung bình - 1.96 * SE + trung bình + 1.96 * SE) / 2 = (trung bình + trung bình) / 2
Vậy trung bình của A là 50, trung bình của B là 50, trung bình của C là  50. 
Vế phải khoảng = trung bình + 1.96 * SE => SE = (vế phải khoảng - trung bình) / 1.96 Vậy SE của A là 1.02, SE của B là 0.51, SE của C là 1.276 
SE = s / căn bậc 2 (n) => n = (s / SE) * (s / SE) 
Vậy n của A là 96.117, n của B là 384.468, n của C là 61.41 => n của B là lớn nhất
(b): để độ rộng giảm 1 nửa thì cỡ mẫu n phải gấp 4 lần

**S2. [mới]** Ba bạn mô phỏng "tung đồng xu công bằng" cho ba kết quả khác nhau, mỗi bạn tính p bằng cách đếm tỉ lệ thế giới H0 ra kết quả lệch ít nhất như đã thấy: bạn X ra `p = 0,012`, bạn Y ra `p = 0,34`, bạn Z ra `p = 0,08`. Với mỗi bạn: kết luận ở mức 5% (bác bỏ hay chưa đủ bằng chứng bác bỏ H0) và **một câu theo mẫu**: "Nếu đồng xu công bằng thì ... (p ... 0,05), nên ...".
X: nếu đồng xu công bằng thì p = 0.012 <= 0.05, nên bác bỏ H0
Y: nếu đồng xu công bằng thì p = 0.34 < 0.05 nên chưa đủ bằng chứng bác bỏ H0
Z: nếu đồng xu công bằng thì p = 0.08 > 0.05, nên chưa đủ bằng chứng bác bỏ H0

**S3. [mới, tìm lỗi]** Mỗi phát biểu dưới đây **sai**. Với mỗi cái: lỗi ở đâu và viết lại thành câu đúng.
(a) Nam nói: "p = 0,34 nên đồng xu chắc chắn công bằng."
(b) Lan nói: "p = 0,01 nghĩa là xác suất H0 đúng là 1%."
(c) Hòa nói: "Khoảng tin cậy 95% của điểm trung bình là [48; 52], nghĩa là 95% học sinh có điểm từ 48 đến 52."
(a): Sai vì với p = 0.34 > 0.05 thì chưa đủ bằng chứng nói rằng đồng xu công bằng câu đúng phải là p = 0.34 thì chưa đủ bằng chứng đồng xu công bằng
(b): Sai vì p là xác suất của dữ liệu nếu H0 đúng không phải xác suất của H0, câu đúng phải là p = 0.01 là xác suất dữ liệu nếu H0 đúng 
(c): Sai vì khoảng tin cậy 95% là khoảng tin cậy của điểm trung bình không phải khoảng điểm của học sinh, học sinh có thể có điểm 100 và 0 chứ không phải loanh quanh từ 48 đến 52, câu đúng phải là khoảng tin cậy 95% của điểm trung bình là [48,52] nghĩa là điểm trung bình của các học sinh giao động trong khoảng [48,52]

**S4. [đã gặp, có kiểm ngược]** Tung đồng xu 100 lần, được **58 ngửa**. H0: đồng xu công bằng.
Bước 1 (kỳ vọng và độ lệch chuẩn dưới H0): kỳ vọng = 50, độ lệch chuẩn 5
Bước 2 (`z`, ghi số 58 vào công thức): z = (giá trị quan sát - kỳ vọng) / độ lệch chuẩn = (58-50) / 5 = 16
Kiểm ngược (`z × độ lệch chuẩn + kỳ vọng` có ra 58 không?): có 1.6 * 5 = 8, 8 + 50 = 58
Bước 3 (so `|z|` với 1,96 và 2,58): `|z|` = 1.6 < 1.96 tức p > 0.05, `|z|` = 1.6 < 2.58 tức p > 0.01
Bước 4 (một câu theo mẫu, mức 5%): nếu đồng xu công bằng thì `|z|` = 1.6 < 1.96 (p > 0.05), nên chưa đủ bằng chứng bác bỏ H0

---

## Ghi nhật ký (điền đủ ba ô bằng câu hoàn chỉnh)
- Giờ bắt đầu và kết thúc thực tế (ghi giờ thật):8h30 - 9h14
- Câu bạn nghĩ mình sai nhiều khả năng nhất, và vì sao (nêu câu cụ thể): S3 phần c vì khoảng tin cậy tôi không rõ đã nói chuẩn chưa
- Những câu bạn đã kiểm lại và bằng cách nào (và câu nào chưa kiểm): các câu L tôi đã tính máy và làm ra nháp còn các câu S tôi đã tính bằng cách bấm máy tính bỏ túi
