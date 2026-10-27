---
title: Buổi học Tuần 2 - Ngày 3 (Xác suất 1)
tags: [bai-tap, xac-suat, tuan-2]
---

# Tuần 2 - Ngày 3: Hoàn tất window function, xác suất 1, sửa CV

**Khung giờ (khoảng 2 giờ 20, 21:00 - 23:20):**
- 21:00 - 21:10: Phần 0, trả nợ hai lỗi hôm qua
- 21:10 - 21:50: **hoàn tất Phần D trong file `buoi_hoc_tuan2_ngay2.md`**: làm DA3, DB1, DB2, DC1, DC2 (bỏ DB3)
- 21:50 - 22:10: đọc bài học xác suất
- 22:10 - 22:55: bài tập xác suất
- 22:55 - 23:15: Phần E, sửa CV
- 23:15 - 23:20: ghi nhật ký

> [!warning] Luật giữ nguyên
> Câu tính toán: chép **công thức** bạn dùng, rồi mới thay số. Tính tay trước. Câu SQL: **điều kiện đề** và **kết quả mong đợi** như cũ, so từng ô sau khi chạy.

---

## Phần 0: Trả nợ hôm qua

**0.1.** (Từ 0.4 hôm qua.) Vì sao theo chuẩn SQL, `HAVING total > 500` (dùng bí danh `total` tạo ở `SELECT`) có thể lỗi ở nhiều hệ cơ sở dữ liệu, còn `HAVING SUM(amount) > 500` thì chạy được ở mọi nơi? Dùng thứ tự `FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY` để giải thích.
Trả lời: vì  HAVING chạy trước SELECT tức HAVING sẽ không biết total là gì thì xảy ra lỗi

**0.2.** (Từ C2 hôm qua.) So sánh `o.order_id = NULL` cho kết quả **đúng, sai hay không xác định**? Vì sao `WHERE` không trả dòng nào? Cho kết quả của `NULL = NULL`.
Trả lời: so sánh o.order_id = NULL cho kết quả sai hoặc không xác định mà WHERE thì không trả được dòng nào nếu kết quả của so sánh là sai hoặc không xác định và kết quả của NULL = NULL là không xác định

---

## Phần D còn lại (hoàn tất trong file `buoi_hoc_tuan2_ngay2.md`)
Làm DA3, DB1, DB2, DC1, DC2. Bỏ DB3. Nhớ điền ô "Đúng hay sai, vì sao" cho DA2 (bạn để trống hôm qua).

---

## Bài học: Xác suất 1

**Xác suất (cách tính đơn giản):** nếu mọi kết quả có khả năng như nhau, `P(A) = số kết quả thuận lợi / tổng số kết quả`.
**Phần bù:** `P(không A) = 1 − P(A)`.

**Giao và hợp:**
- `A ∩ B` (giao): A **và** B cùng xảy ra.
- `A ∪ B` (hợp): A **hoặc** B **hoặc cả hai** xảy ra.

**Công thức hợp (luôn đúng):**
```
P(A ∪ B) = P(A) + P(B) − P(A ∩ B)
```
Phải trừ phần giao vì phần chung bị đếm **hai lần**.

**Xung khắc (không thể cùng xảy ra):** `P(A ∩ B) = 0`, nên `P(A ∪ B) = P(A) + P(B)`.
Ví dụ: tung một xúc xắc, "ra số chẵn" và "ra số lẻ".

**Độc lập (biết A xảy ra không làm đổi xác suất của B):** `P(A ∩ B) = P(A) × P(B)`.
Ví dụ: tung hai đồng xu, kết quả của đồng này không ảnh hưởng đồng kia.

**Hai khái niệm này KHÁC nhau.** Nếu A và B xung khắc và cả hai có xác suất dương thì chúng **không độc lập**: biết A đã xảy ra thì chắc chắn B **không** xảy ra, tức biết A đã làm đổi xác suất của B (về 0).

**Xác suất có điều kiện:** xác suất của A khi đã biết B xảy ra.
```
P(A | B) = P(A ∩ B) / P(B)      (với P(B) > 0)
```
**Quy tắc nhân:** `P(A ∩ B) = P(B) × P(A | B)`.

**Ví dụ làm sẵn:** tung một xúc xắc. A = "ra số chẵn" = {2, 4, 6}. B = "ra số > 3" = {4, 5, 6}.
- `P(A) = 1/2`, `P(B) = 1/2`.
- `A ∩ B = {4, 6}`, nên `P(A ∩ B) = 2/6 = 1/3`.
- `P(A ∪ B) = 1/2 + 1/2 − 1/3 = 2/3`. (Kiểm tra: {2,4,5,6} có 4 phần tử, `4/6 = 2/3`. Khớp.)
- `P(A | B) = (1/3) / (1/2) = 2/3`.
- Độc lập không? `P(A) × P(B) = 1/4`, khác `1/3`, nên **không** độc lập.

---

## Bài tập A: Tính (viết công thức rồi mới thay số)

**A1.** `P(A) = 0.4`, `P(B) = 0.3`, A và B **độc lập**. Tính `P(A ∩ B)` và `P(A ∪ B)`.
Công thức: P(A ∩ B) = P(A)  *  P(B)  với A và B độc lập
P(A ∪ B) = P(A) + P(B) - P(A ∩ B)
Kết quả: P(A ∪ B) = 0.58
P(A ∩ B) = 0.12

**A2.** `P(A) = 0.4`, `P(B) = 0.3`, A và B **xung khắc**. Tính `P(A ∩ B)` và `P(A ∪ B)`.
Công thức: P(A ∩ B) = 0 với A và B xung khắc
P(A ∪ B) = P(A) + P(B) với A và B xung khắc tức P(A ∩ B) = 0
Kết quả: P(A ∩ B) = 0
P(A ∪ B) = 0.7

**A3.** `P(A) = 0.5`, `P(B) = 0.5`, A và B **xung khắc**. Chúng có độc lập không? Giải thích bằng công thức.
Trả lời: A và B xung khắc tức A xảy ra thì B không xảy ra tức A và B có liên quan không độc lập ví dụ tung xúc xắc tỉ lệ ra mặt 6 là 1/6 và tỉ lệ ra các mặt khác là 5/6 tức không thể ra mặt số 6 và các mặt khác đồng thời được chỉ có thể ra hoặc mặt 6 hoặc các mặt khác 6

**A4.** Tung hai đồng xu công bằng. Tính `P(cả hai ngửa)` và `P(ít nhất một đồng ngửa)`. (Gợi ý: dùng phần bù.)
Công thức: P(cả hai ngửa) = P(đồng xu 1 ngửa)  *  P(đồng xu 2 ngửa)
P(ít nhất 1 đồng ngửa) = 1 - P(cả hai úp)
Kết quả: P(cả hai ngửa) = 0.5 * 0.5  = 0.25
P(ít nhất 1 đồng ngửa) = 1 - 0.5 * 0.5 = 0.75

**A5.** Hộp có 5 bi đỏ và 3 bi xanh. Rút **hai bi liên tiếp, không hoàn lại**. Tính `P(cả hai đỏ)` và `P(bi thứ hai đỏ | bi thứ nhất đỏ)`.
Công thức: P(cả hai đỏ) = P(viên bi thứ 1 màu đỏ) * P(viên bi thứ 2 màu đỏ)
P(bi thứ 2 đỏ | bi thứ nhất đỏ) = P(cả 2 viên bi đều đỏ) / P(viên bi thứ nhất đỏ)
Kết quả (phân số và số thập phân 3 chữ số): P(cả hai đỏ) = 5/8 * 4/8 = 5/16 = 0.313, P(bi thứ 2 đỏ | bi thứ nhất đỏ) = (5/16) / (5/8) = 1/2 = 0.5

**A6.** Tung một xúc xắc. A = "ra số lẻ", B = "ra số ≥ 3". Tính `P(A)`, `P(B)`, `P(A ∩ B)`, `P(A | B)`. Hai biến cố này có độc lập không?
Công thức: P(A) = P(ra số 1) + P(ra số 3) + P(ra số 5)
P(B) = P(ra số 3) + P(ra số 4) + P(ra số 5) + P(ra số 6)
P(A ∩ B) = P(ra số 3) + P(ra số 5)
P(A | B) = P(A ∩ B) / P(B)
Kết quả: P(A) = 1/2
P(B) = 2/3
P(A ∩ B) = 1/3
P(A | B) = 1/2
Độc lập không, vì sao: độ lập vì khi A xảy ra thì B có thể xảy ra chứ không phải A xảy ra B không xảy ra

**A7.** `P(A) = 0.2`, `P(B) = 0.6`, A và B **độc lập**. Tính `P(A ∪ B)`. (Đây cùng dạng với câu C1 bài chẩn đoán Tuần 0 bạn từng sai.)
Công thức: P(A ∪ B) = P(A) + P(B) - P(A ∩ B) = P(A) + P(B) - P(A) * P(B)
Kết quả: P(A ∪ B) = 0.68

---

## Bài tập B: Tự giải thích (không nhìn tài liệu)

**B1.** Phân biệt **độc lập** và **xung khắc** bằng lời của bạn. Cho một ví dụ cho mỗi loại, và nói vì sao hai khái niệm này không đồng nghĩa.
Trả lời: độc lập là khi A xảy ra thì B có thể xảy ra hoặc không hay A xảy ra thì không ảnh hưởng việc B xảy ra ví dụ tung 2 đồng xu thì đồng xu thứ 1 ra ngửa hay úp không ảnh hưởng kết quả đồng xu thứ 2 ra ngửa hoặc úp. Xung khắc thì A xảy ra thì B không xảy ra và ngược lại B xảy ra thì A không xảy ra ví dụ tung con xúc xắc nếu A là các mặt chẵn, B là các mặt lẻ thì kết quả chỉ có thể là lẻ hoặc chẵn tức A xảy ra B không thể xảy ra

**B2.** Vì sao công thức hợp phải trừ `P(A ∩ B)`? Giải thích bằng một ví dụ cụ thể.
Trả lời: vì hợp là tổng P(A) và P(B) đã chứ P(A ∩ B) rồi nên cần trừ đi ví dụ tung 1 con xúc sắc P(A) là tỉ lệ ra mặt chẵn, P(B) tỉ lệ ra mặt lớn hơn 3 thì P(A ∪ B) là tỉ lệ ra mặt chẵn lớn hơn 3
A là các mặt 2,4,6 P(A) = 1/2
B là các mặt 4,5,6, P(B) = 1/2
A ∪ B là các mặt 4,6 P(A ∪ B) = 1/3
Nếu cộng P(A) + P(B) thì tính mặt 4 và 6 tận 2 lần thay vì 1

---

## Tùy chọn: kiểm chứng A5 bằng mô phỏng (chỉ làm sau khi đã tính tay)
```python
import numpy as np
rng = np.random.default_rng(0)
box = np.array([1]*5 + [0]*3)      # 1 = đỏ, 0 = xanh
n = 100000
count = 0
for _ in range(n):
    draw = rng.choice(box, size=2, replace=False)
    if draw.sum() == 2:
        count += 1
print(count / n)
```
Kết quả mô phỏng: 0.35767

Có gần với đáp án tính tay không: có


---

## Ghi nhật ký
- Thời gian học thực tế:
- Câu bạn thấy mình sai nhiều khả năng nhất:
- Điều chưa hiểu:

> Làm xong gửi nguyên file này cho Claude chấm (kèm file `buoi_hoc_tuan2_ngay2.md` đã hoàn tất Phần D).
