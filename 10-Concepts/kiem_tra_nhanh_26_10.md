---
title: Bài kiểm tra nhanh 26/10 (30 phút)
tags: [kiem-tra, tuan-3, cong-co-dieu-kien]
---

# Bài kiểm tra nhanh (10 câu, 100 điểm, 30 phút)

Đạt từ **70 điểm** thì qua hẳn Tuần 3. Dưới 70 thì Tuần 3 dừng, ôn đúng phần sai rồi làm bài mới.

## Luật làm bài
- **Làm một lần liền 30 phút, bấm giờ, không tra cứu, không chạy code, không hỏi AI.** Giấy nháp được dùng. Hết giờ thì dừng.
- **Phạm vi:** chỉ dùng nội dung đã học (SQL `JOIN`, `NULL`, window function, xác suất có điều kiện, độc lập, công thức xác suất toàn phần và Bayes). Không có công thức mới. Câu nào thấy đề mơ hồ thì ghi "đề mơ hồ vì ..." và làm theo cách hiểu hợp lý, tôi chấm theo cách hiểu đó.
- Mỗi câu 10 điểm. Câu có nhiều ý thì mỗi ý một dòng có nhãn: điền đủ từng ô.
- **Kiểm đề sau mỗi câu (10 giây):** đếm số ý đề hỏi; dòng cuối là đúng đại lượng đề hỏi chưa; có đơn vị chưa.
- Câu xác suất: viết **công thức** trước rồi mới thay số.
- Làm xong gửi nguyên file này cho Claude chấm.

## Dữ liệu cho phần A (SQL)

`staff(id, name, team_id, pay)`:

| id  | name | team_id | pay |
| --- | ---- | ------- | --- |
| 1   | Lan  | 1       | 20  |
| 2   | Minh | 1       | 30  |
| 3   | Nam  | 2       | 30  |
| 4   | Oanh | 2       | 30  |
| 5   | Phúc | NULL    | 10  |
| 6   | Quân | 3       | 25  |

`teams(team_id, team_name)`:

| team_id | team_name |
|---|---|
| 1 | Alpha |
| 2 | Beta |
| 4 | Delta |

(Phúc chưa thuộc nhóm nào. Nhóm 3 của Quân không có trong bảng `teams`. Nhóm 4 không có nhân viên nào.)

## A. SQL (6 câu, 60 điểm)

**A1.** (a) `SELECT COUNT(*) FROM staff;` trả về số nào? (b) `SELECT COUNT(team_id) FROM staff;` trả về số nào? (c) Vì sao khác nhau (một câu)?
(a): 6
(b): 5
(c): COUNT(*) tức đếm tất cả các dòng có trong bảng còn COUNT(team_id) thì chỉ đếm các dòng chứa dữ liệu khác NULL

**A2.** `SELECT name FROM staff WHERE team_id <> 1;`
(a) Trả về những ai?  
(b) Phúc có trong kết quả không? Với Phúc, điều kiện `team_id <> 1` nhận giá trị logic nào (đúng, sai, hay không xác định)?
(a): Nam, Oanh, Quân
(b): Phúc không có trong kết quả vì điều kiện team_id <> 1 chỉ nhận giá trị logic team_id là 2 và 3 là đúng, giá trị team_id là 1 là sai và giá trị team_id là NULL  không xác định được mà WHERE chỉ nhận kết quả logic là đúng còn sai và không xác định thì không nhận

**A3.** `SELECT s.name, t.team_name FROM staff s JOIN teams t ON s.team_id = t.team_id;`
(a) Trả về bao nhiêu dòng?
(b) Nếu đổi thành `LEFT JOIN`, trả về bao nhiêu dòng?
(c) Với `LEFT JOIN`, `team_name` của Quân là gì? Của Phúc là gì?
(a): 4 dòng
(b): 6 dòng
(c): team_name của Quân và Phúc đều là NULL vì không có team_id nào khớp với team_id của Quân và Phúc

**A4.** Liệt kê tên các nhóm (`team_name`) **chưa có nhân viên nào**.
Điều kiện đề (chép nguyên văn): liệt kê các nhóm (team_name) chưa có nhân viên nào
Kết quả mong đợi (tính tay):

| team_name |
| --------- |
| Delta     |

```sql
-- Trả lời:
SELECT t.team_name FROM teams t LEFT JOIN staff s ON t.team_id = s.team_id WHERE s.team_id IS NULL
```

**A5.** Xét nhóm 2 (Nam và Oanh cùng pay 30). Với `PARTITION BY team_id ORDER BY pay DESC`:
(a) Dùng `ROW_NUMBER()` rồi lọc `rn = 1`: có bao nhiêu dòng của nhóm 2? Ai được chọn?
(b) Dùng `RANK()` rồi lọc `rk = 1`: có bao nhiêu dòng của nhóm 2?
(c) Muốn lấy **cả hai** người pay cao nhất mỗi nhóm thì dùng hàm nào?
(a): chỉ có 1 dòng của nhóm 2 và người được chọn là Nam
(b): dùng RANK() thì có 2 dòng trong nhóm 2
(c): dùng hàm RANK() thì những người cùng có mức pay cao nhất sẽ được lấy dùng ROW_NUMBER() thì chỉ lấy được người có mức pay cao nhất đầu tiên xuất hiện

**A6.** (a) `SELECT id, pay, SUM(pay) OVER (ORDER BY id) AS c FROM staff;` Viết cột `c` của id 1 đến 6.
(b) `SELECT id, pay, SUM(pay) OVER () AS c FROM staff;` Viết cột `c` của id 1 đến 6.
(c) Vì sao hai cột khác nhau (một câu, nhắc `ORDER BY` trong `OVER`)?
(a):

| c   |
| --- |
| 20  |
| 50  |
| 80  |
| 110 |
| 120 |
| 145 |

(b):

| c   |
| --- |
| 145 |
| 145 |
| 145 |
| 145 |
| 145 |
| 145 |

(c): với việc không có partition by trong over thì OVER thực hiện SUM trên cả bảng không có ORDER BY thì OVER tính tổng toàn bộ pay rồi hiển thị tại cột c là tổng toàn bộ pay của bảng nhưng thêm ORDER BY thì OVER tính tổng cộng dồn lần lượt 

---

## B. Xác suất (4 câu, 40 điểm)

**B1.** Hộp có 5 bi đỏ và 3 bi xanh. Rút hai bi liên tiếp, **không hoàn lại**.
(a) Trước khi nhân, ghi: lần 1 hộp có ___ bi (___ đỏ); nếu lần 1 rút đỏ thì lần 2 hộp có ___ bi (___ đỏ).
(b) Công thức `P(cả hai đỏ)`:
(c) `P(cả hai đỏ)`: phân số ___ , thập phân 3 chữ số ___
(a): lần 1 hộp có 8 bi (5 đỏ) , nếu lần 1 rút đỏ thì lần 2 hộp có 7 bi (4 đỏ)
(b): P(cả hai đỏ) = P(bi 1 đỏ) * P(bi 2 đỏ | bi 1 đỏ)
(c): P(cả hai đỏ) = 5 / 14 = 0.357

**B2.** Hai máy sản xuất ra toàn bộ hàng của một xưởng. Máy A làm 60% sản phẩm, tỉ lệ lỗi 5%. Máy B làm 40% sản phẩm, tỉ lệ lỗi 10%. Lấy ngẫu nhiên một sản phẩm.
(a) `P(lỗi)`: công thức, rồi số: P(lỗi) = P(lỗi | A) * P(A) + P(lỗi | B) * P(B) = 0.05 * 0.6 + 0.4 * 0.1 = 0.07
(b) Biết sản phẩm đó **lỗi**, `P(do máy A làm)`: công thức, rồi số (3 chữ số thập phân): P(A | lỗi) = P(lỗi | A) * P(A) / P(lỗi) = 0.428
(c) Kiểm tra bằng tần suất tự nhiên (10.000 sản phẩm): 700 lỗi tổng cộng, trong đó 300 do máy A; tỉ lệ = 300 / 700 = 0.428

**B3.** `P(A) = 0,6`, `P(B) = 0,5`, `P(A ∩ B) = 0,3`.
(a) `P(A ∪ B)`: công thức, rồi số: P(A ∪ B) = P(A) + P(B) - P(A ∩ B) = 0.8
(b) `P(A | B)`: công thức, rồi số: P(A | B) = P(A ∩ B) / P(B) = 0.6
(c) A và B có độc lập không? Kiểm bằng công thức, ghi cả hai vế:
(a): P(A ∪ B) = P(A) + P(B) - P(A ∩ B) = 0.8
(b): P(A | B) = P(A ∩ B) / P(B) = 0.6
(c): P(A) * P(B) = 0.3, P(A ∩ B) = 0.3 tức P(A ∩ B) = P(A) * P(B) suy ra A và B có độc lập

**B4.** Hộp có 4 bi đỏ và 6 bi xanh. Rút hai bi liên tiếp, **không hoàn lại**. Đề hỏi **hai** đại lượng khác nhau:
(a) `P(bi thứ hai xanh | bi thứ nhất đỏ)`: phân số ___ , thập phân 3 chữ số ___
(b) `P(bi thứ nhất đỏ và bi thứ hai xanh)`: công thức, rồi phân số ___ , thập phân 3 chữ số ___
(c) Hai số trên khác nhau ở chỗ nào (một câu)?
(a): P(bi thứ 2 xanh | bi thứ nhất đỏ) = 
(b):
(c):

---

## Ghi nhật ký (bắt buộc điền cả ba)
- Thời gian làm thực tế: 2h30 - 3h
- Câu bạn nghĩ mình sai nhiều khả năng nhất, và vì sao: A5 do tôi chưa rõ ROW_NUMBER() có đúng là 2 người cùng mức pay thì ai xuất hiện trước thì mới đánh số 1 hay cái khác Câu B4 tôi chưa làm được 
- Điều chưa hiểu: tôi hiểu yêu cầu đề mà tôi không rõ làm thế nào
