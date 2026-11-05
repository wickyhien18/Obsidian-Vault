---
title: Thứ Năm 5/11 - Case study 2 (giao đồ ăn) và ôn xác suất
tags: [case-study, xac-suat, tuan-3]
---

# 5/11 (1 giờ): Case study 2 (45 phút) + ôn xác suất (15 phút)

**Cách làm:** tính tay hoặc dự đoán trước, rồi chạy code kiểm. Làm tròn bình thường, lệch ≤ 0,001 vẫn đúng (tỉ lệ % làm tròn 1 chữ số thập phân). Đề mơ hồ thì ghi "đề mơ hồ vì ...". Mọi câu chỉ dùng nội dung trong bài học bên dưới. Không có nhật ký; một vài ô **"Kiểm lại"** nằm trong bài. Nhãn **[đã gặp]/[mới]** như thường lệ.

**Nhắc từ bài 4/11 (cùng một lỗi lặp lại, nên đặt lên đầu):** hàm gộp SQL **bỏ qua NULL** (Q1: Bình có trung bình 50 vì NULL bị bỏ, nên Bình cũng vượt 30), và nhớ **cộng lại tổng một lần nữa** (Q2: tổng là 210, không phải 220). SQL có thể chạy bằng `sql_setup_portable.py` để kiểm trước khi nộp.

---

# PHẦN 1 (45 phút): Case study 2 - dịch vụ giao đồ ăn

## Bài học (có ví dụ làm sẵn)

**1. Tỉ lệ (rate) = số sự kiện / tổng số.**
- Tỉ lệ giao trễ = đơn trễ / tổng đơn.
- Tỉ lệ hoàn tiền = đơn hoàn tiền / tổng đơn.
- Doanh thu mỗi đơn = doanh thu / tổng đơn.
Ví dụ làm sẵn (quán cơm văn phòng): 200 đơn, 10 đơn trễ → tỉ lệ trễ = 10/200 = **5,0%**.

**2. Tỉ lệ toàn hệ thống không phải trung bình các tỉ lệ.** Phải **cộng tử, cộng mẫu rồi chia** (trung bình có trọng số theo số đơn).
Ví dụ làm sẵn: khu X có 100 đơn, 10 trễ (10%); khu Y có 300 đơn, 15 trễ (5%).
Toàn hệ thống: (10 + 15)/(100 + 300) = 25/400 = **6,25%**. Còn lấy trung bình hai tỉ lệ: (10% + 5%)/2 = 7,5% là **sai**, vì khu Y có nhiều đơn hơn nhưng bị tính ngang khu X.

**3. Tỉ lệ trên ít đơn thì dao động mạnh.** (Đã học: sai số chuẩn giảm theo √n, tức n nhỏ thì kết quả không ổn định.) Một khu chỉ có vài chục đơn mà tỉ lệ cao thì **chưa đủ căn cứ để kết luận**; cách làm đúng là thu thêm dữ liệu rồi mới quyết định.

**4. Cách viết khuyến nghị: ba phần.**
- **Kết luận** (một câu), **Căn cứ** (kèm số cụ thể), **Hành động** (việc làm được, có đo được).
Ví dụ làm sẵn (quán cơm văn phòng, khu X trễ 10%, khu Y trễ 5%, cả hai trên 100 đơn trở lên): "Kết luận: ưu tiên xử lý giao trễ ở khu X. Căn cứ: tỉ lệ trễ khu X là 10%, gấp đôi khu Y (5%). Hành động: thêm một tài xế giờ cao điểm cho khu X trong 2 tuần, rồi đo lại tỉ lệ trễ."

## Dữ liệu (3 khu vực, trong một tháng)

| Khu | Tổng đơn | Đơn trễ | Đơn hoàn tiền | Doanh thu (nghìn đồng) |
|---|---|---|---|---|
| A | 1200 | 96 | 30 | 144000 |
| B | 400 | 48 | 20 | 44000 |
| C | 50 | 10 | 1 | 5500 |

## Bài tập

**C1. [đã gặp]** Tính cho từng khu (% làm tròn 1 chữ số, doanh thu mỗi đơn đơn vị nghìn đồng):

| Khu | Tỉ lệ trễ (%) | Tỉ lệ hoàn tiền (%) | Doanh thu/đơn |
| --- | ------------- | ------------------- | ------------- |
| A   | 8%            | 2.5%                | 120           |
| B   | 12%           | 5%                  | 110           |
| C   | 20%           | 2%                  | 110           |

Kiểm lại (nhân ngược một ô: tỉ lệ × tổng đơn có ra số đơn ban đầu không, ghi phép tính): 8% x 1200 = 96

**C2. [đã gặp]** Viết khuyến nghị theo ba phần (**Kết luận / Căn cứ / Hành động**): nên ưu tiên xử lý khu nào về chất lượng giao hàng? (nhớ ý 3: khu nào có đủ đơn để tin con số)
Kết luận: Ưu tiên xử lý khu C
Căn cứ (kèm số): tỉ lệ trẽ là 20% bằng tổng tỉ lệ của 2 khu A và B mà tổng số đơn là 50 ít nhất so với khu A là 1200 khu B là 400
Hành động: cần thu thập thêm số liệu tại khu C để ra quyết định đúng đắn

**C3. [mới]** Nếu khu B đưa tỉ lệ trễ xuống bằng tỉ lệ khu A, thì trong 400 đơn mỗi tháng sẽ **giảm được bao nhiêu đơn trễ**? Ghi phép tính: tỉ lệ trễ mới x số đơn mỗi tháng = 8% x 400 = 32 so với số đơn trễ lúc trước là 48 thì số đơn trễ giảm 2/3 lần

**C4. [mới, tìm lỗi]** Quản lý nói: "Khu C có tỉ lệ trễ 20%, cao nhất, nên đóng ngay khu C."
(a) Lập luận này yếu ở đâu (nêu **con số** làm bằng chứng)? lập luận này yếu vì số đơn của khu C có 50 đơn chưa đủ bằng chứng khu C làm ăn lỗ vốn
(b) Viết lại thành một khuyến nghị đúng (ba phần): khu C có tỉ lệ trễ 20%, số đơn là 50 thấp nhất trong 3 khu, cần thu thập thêm số đơn từ khu C để ra quyết định đúng đắn

**C5. [mới]** Tính tỉ lệ giao trễ của **toàn hệ thống** (cả ba khu).
(a) Cách đúng (cộng tử, cộng mẫu): kết quả % = (86 + 48 + 10)/(1200 + 400 + 50) = 9.3%
(b) Cách sai (trung bình ba tỉ lệ): kết quả % = (8% + 12% + 20%) / 3 = 13.3%
(c) Vì sao hai kết quả khác nhau (một câu, nhắc số đơn của từng khu)? vì số đơn của mỗi khu khác nhau, khu A là 1200, khu B là 400, khu C là 50 

---

# PHẦN 2 (15 phút): ôn xác suất (đổi chủ thể)

## Nhắc (có ví dụ làm sẵn)
- **Rút không hoàn lại:** `P(A ∩ B) = P(A) · P(B | A)`. Ví dụ: hộp 5 tai nghe có 2 cái lỗi, lấy lần lượt 2 cái không hoàn lại: `P(cả hai lỗi) = 2/5 · 1/4 = 1/10`.
- **Độc lập:** `P(A ∪ B) = P(A) + P(B) − P(A ∩ B)`, và nếu độc lập thì `P(A ∩ B) = P(A)·P(B)`.
- **Bayes bằng tần suất tự nhiên:** tưởng tượng một số lượng lớn rồi đếm. Ví dụ làm sẵn (bộ lọc thư rác): 1000 email, 20% là thư rác; bộ lọc chặn đúng 90% thư rác và chặn nhầm 5% thư thường. Thư rác bị chặn: 200 · 0,9 = 180. Thư thường bị chặn nhầm: 800 · 0,05 = 40. `P(thư rác | bị chặn) = 180/(180 + 40) = 0,818`.

**X1. [đã gặp]** Kho có 10 bóng đèn, 3 cái hỏng. Lấy lần lượt 2 cái không hoàn lại.
(a) P(cả hai hỏng) = (3/13) * (2/12) = 0.038
(b) P(đúng một cái hỏng) = (3/13) * (10/12) = 0.192

**X2. [mới]** Hệ thống phát hiện gian lận thẻ: trong 10000 giao dịch có 2% là gian lận. Hệ thống báo đúng 95% giao dịch gian lận và báo nhầm 4% giao dịch thật.
(a) Số giao dịch gian lận bị báo đúng: 190
(b) Số giao dịch thật bị báo nhầm: 392
(c) P(gian lận | bị báo) (3 chữ số thập phân): 190 /(190 + 392) = 0.326
Kiểm lại (cộng hai số (a) và (b) rồi chia, ghi phép tính): tôi đã ghi và kiểm tra như trên rồi

**X3. [mới, tìm lỗi]** Một nhân viên nói: "Hệ thống báo đúng 95% gian lận, nên giao dịch nào bị báo thì chắc chắn 95% là gian lận."
(a) Nhầm giữa hai xác suất nào? 95% là xác suất hệ thống tìm ra bao nhiêu giao dịch gian lận trong số những giao dịch thực sự gian lận chứ không phải là xác suất 1 giao dịch bị báo cáo là gian lận thực sự gian lận
(b) Viết lại đúng bằng số của X2: Giao dịch bị báo thì 32.6% là gian lận

**X4. [đã gặp]** Hai tuyến xe buýt hoạt động độc lập. Tuyến 1 đến trễ với xác suất 0,3, tuyến 2 với xác suất 0,2. P(ít nhất một tuyến trễ) = 1 - P(cả 2 tuyến đều về sớm) = 1 - (1- 0.2) * (1- 0.3) = 0.44
