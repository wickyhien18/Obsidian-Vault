---
title: Đề thi thử nhỏ - Buổi 2: Xác suất/thống kê, ĐSTT/ML, một câu tình huống
tags: [thi-thu, xac-suat, thong-ke, dstt, ml]
---

# Đề thi thử nhỏ, Buổi 2: 55 phút, 50 điểm

**Phạm vi:** rút không hoàn lại; Bayes bằng tần suất tự nhiên; độ lệch chuẩn (tổng thể và mẫu); sai số chuẩn và khoảng tin cậy 95%; kiểm định tỉ lệ bằng `z` và p-value; giải hệ bằng nghịch đảo ma trận 2×2; độ dài/khoảng cách/chuẩn hóa vectơ; hồi quy tuyến tính bằng ma trận; ma trận nhầm lẫn (accuracy, precision, recall); MSE; một câu tình huống đọc bảng rồi khuyến nghị. Không có nội dung ngoài các buổi đã học.

**Quy tắc thi (giống Buổi 1):**
- **Bấm giờ 55 phút, làm một lần liền.** Gợi ý: Phần III 25 phút, Phần IV 20 phút, Phần V 10 phút. Hết giờ thì dừng, câu chưa làm ghi "chưa làm".
- **Không chạy code, không tra cứu, không xem lộ trình.** Dùng nháp và máy tính bỏ túi nếu muốn.
- Chưa biết thì ghi **"không biết"**, đề mơ hồ thì ghi **"đề mơ hồ vì ..."**; không bỏ trống.
- **Trước khi nộp:** đọc lại dòng cuối của mỗi câu, và với mỗi câu tính toán, kiểm ngược một lần.
- Cuối file ghi **giờ bắt đầu và giờ nộp** (một dòng). Không có nhật ký.
- Nhãn **[đã gặp]** (cùng dạng đã làm) và **[mới]** (dạng chưa làm). Điểm ghi trong ngoặc.

**Dùng để làm gì:** kết hợp với Buổi 1 để mình quyết định tuần học 24/11 cần vá phần nào. Điểm thấp không sao, hãy làm thật như đang thi.

---

# PHẦN III: XÁC SUẤT VÀ THỐNG KÊ (25 phút, 22 điểm)

**X1. [đã gặp] (4đ)** Thùng có 12 quả trứng, trong đó 5 quả nứt. Lấy lần lượt 2 quả, không hoàn lại.
(a) P(cả hai nứt) (1đ):
(b) P(đúng một quả nứt) (2đ):
(c) P(không quả nào nứt) (1đ):
Kiểm lại (cộng ba xác suất):

**X2. [mới] (4đ)** Phần mềm phát hiện đạo văn: trong 5000 bài nộp có 4% là đạo văn. Phần mềm báo đúng 90% bài đạo văn và báo nhầm 5% bài không đạo văn.
(a) Số bài đạo văn bị báo (1đ):
(b) Số bài không đạo văn bị báo nhầm (1đ):
(c) P(đạo văn | bị báo), 3 chữ số thập phân (2đ):

**X3. [đã gặp] (3đ)** Thời gian phản hồi (ms) của một máy chủ: `[10, 14, 18, 22]`.
(a) Độ lệch chuẩn tổng thể, 3 chữ số thập phân (1đ):
(b) Độ lệch chuẩn mẫu, 3 chữ số thập phân (1đ):
(c) `np.std` mặc định cho kết quả của (a) hay (b)? Vì sao (1đ):

**X4. [đã gặp] (4đ)** Điểm bài kiểm tra của 64 học sinh có trung bình 80 và độ lệch chuẩn mẫu `s = 12`.
(a) SE (1đ):
(b) Khoảng tin cậy 95%, hai đầu 2 chữ số thập phân (2đ):
(c) Nếu cỡ mẫu tăng lên 256 thì SE (1đ):

**X5. [mới] (5đ)** Một nhà máy có tỉ lệ sản phẩm lỗi 5%. Sau khi đổi máy, kiểm 400 sản phẩm thấy 12 sản phẩm lỗi. H0: tỉ lệ lỗi vẫn là 5%.
Bước 1: kỳ vọng và độ lệch chuẩn dưới H0 (1đ):
Bước 2: `z`, 3 chữ số thập phân (1đ):
Kiểm ngược (1đ):
Bước 3: so `|z|` với 1,96 (1đ):
Bước 4: một câu kết luận theo mẫu, mức 5% (1đ):

**X6. [đã gặp, tìm lỗi] (2đ)** Linh nói: "p = 0,03 nghĩa là xác suất H0 đúng là 3%."
(a) Sai ở đâu (1đ)?
(b) Viết lại thành câu đúng (1đ):

---

# PHẦN IV: ĐẠI SỐ TUYẾN TÍNH VÀ ML (20 phút, 18 điểm)

**D1. [đã gặp] (5đ)** Quán trà: 2 ly trà đào + 3 ly trà sữa giá 160 (nghìn đồng); 1 ly trà đào + 2 ly trà sữa giá 95 (nghìn đồng).
(a) `A` và `b` (1đ):
(b) det A (1đ):
(c) `A⁻¹` (1đ):
(d) Giá một ly trà đào và một ly trà sữa (1đ):
(e) Kiểm lại vào **hai dữ kiện của đề** (1đ):

**D2. [mới] (3đ)** Hai máy bay không người lái ở tọa độ `a = [2, 10]` và `b = [8, 2]` (km).
(a) Khoảng cách giữa hai máy bay (ghi `a − b` rồi độ dài) (2đ):
(b) Chuẩn hóa vectơ `[6, 8]` (1đ):

**D3. [mới] (4đ)** `x` là số giờ luyện tập, `y` là điểm: `x = [0, 1, 2]`, `y = [1, 2, 4]`. Dùng `w = (XᵀX)⁻¹Xᵀy` với mỗi hàng của `X` là `[1, x]`.
(a) `XᵀX` (1đ):
(b) `Xᵀy` (1đ):
(c) `w = [b, độ dốc]`, 3 chữ số thập phân (2đ):

**D4. [đã gặp] (4đ)** Bộ lọc email lừa đảo: 500 email, thực tế có 50 email lừa đảo. Bộ lọc báo lừa đảo 40 email, trong đó đúng 35 email là lừa đảo.
(a) TP, FP, FN, TN (1đ):
(b) accuracy (1đ):
(c) precision (1đ):
(d) recall (1đ):

**D5. [mới, tìm lỗi] (2đ)** Nam dự đoán giá nhà (triệu đồng) và tính được MSE = 5, trong khi sai số lớn nhất trên mọi mẫu của Nam là 2 (triệu đồng).
(a) Vì sao con số MSE = 5 không thể đúng (một câu, nêu hai số) (1đ):
(b) MSE tối đa có thể là bao nhiêu (1đ):

---

# PHẦN V: TÌNH HUỐNG (10 phút, 10 điểm)

Một chuỗi quán cà phê có ba chi nhánh, số liệu một tháng (doanh thu đơn vị nghìn đồng):

| Chi nhánh | Số đơn | Đơn bị phàn nàn | Doanh thu |
|---|---|---|---|
| X | 800 | 40 | 96000 |
| Y | 300 | 36 | 36000 |
| Z | 40 | 8 | 5200 |

**T1. [mới] (3đ)** Tỉ lệ đơn bị phàn nàn của từng chi nhánh (%, 1 chữ số thập phân; 1đ mỗi chi nhánh):
X = , Y = , Z =

**T2. [mới] (7đ)** Viết khuyến nghị **Kết luận / Căn cứ / Hành động**: chi nhánh nào cần ưu tiên xử lý chất lượng phục vụ?
**Tiêu chí chấm:** (1) kết luận khớp với căn cứ (2đ); (2) nêu ít nhất hai con số cụ thể (2đ); (3) nhắc cỡ mẫu của chi nhánh có ít đơn (1đ); (4) hành động cụ thể, đo được (2đ).
Kết luận:
Căn cứ:
Hành động:

---

**Giờ bắt đầu - giờ nộp (một dòng):**
