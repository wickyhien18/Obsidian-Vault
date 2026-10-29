---
title: Thứ Năm 29/10 - Phần 2 (30 phút): ôn tổng hợp SQL, xác suất, thống kê
tags: [on-tap, sql, xac-suat, thong-ke, tuan-3]
---

# 29/10, Phần 2 (30 phút): ôn tổng hợp

**Lịch hôm nay: 1 giờ.** Phần 1 = Buổi 3 trong `thong_ke_giang_lai.md` (kiểm định, p-value, 30 phút). Phần 2 = file này (30 phút), cách Phần 1 ít nhất 1 tiếng nếu được. **Làm file này không tra cứu** (giống điều kiện thi): tính tay trước, chạy code sau (dán nguyên văn kết quả chạy). Làm tròn bình thường, lệch ≤ 0,001 vẫn đúng. Đề mơ hồ thì ghi "đề mơ hồ vì ..." vào ô trả lời. Câu nào **không phải trả lời** thì đề ghi rõ.

**Nhắc một quy tắc (có ví dụ):** các hàm gộp `SUM`, `AVG`, `COUNT(cột)` **bỏ qua `NULL`**; riêng `COUNT(*)` đếm mọi dòng. Ví dụ: cột `x` = 4, 6, NULL thì `COUNT(*) = 3`, `COUNT(x) = 2`, `SUM(x) = 10`, `AVG(x) = 10/2 = 5` (chia cho số giá trị **không NULL**, không phải 3).

## Dữ liệu SQL
```
courses(course_id, title): (1, SQL), (2, Python), (3, Stats), (4, ML)
enroll(id, student, course_id, score):
  (1, An, 1, 8), (2, An, 2, 9), (3, Bình, 1, 7), (4, Bình, 3, 6), (5, Chi, 2, 9), (6, Dũng, 2, NULL)
```
(Dũng ghi danh môn Python nhưng chưa có điểm. Môn ML chưa ai ghi danh.)

## A. SQL (4 câu)
**M1.** (a) `SELECT COUNT(*) FROM enroll;` (b) `SELECT COUNT(score) FROM enroll;` (c) `SELECT AVG(score) FROM enroll;` (d) `SELECT SUM(score) OVER (ORDER BY id) FROM enroll;` cột này của 6 dòng (id 1 đến 6) là gì?
(a):
(b):
(c) (công thức, rồi số):
(d):
Chạy kiểm tra, khớp hay không (từng ý):

**M2.** Liệt kê tên các môn (`title`) **chưa có ai ghi danh**.
Điều kiện đề (chép nguyên văn):
Kết quả mong đợi (tính tay):
```sql
-- Trả lời:

```
Kết quả chạy, khớp hay không khớp:

**M3.** Với mỗi môn, đếm số **dòng ghi danh** (kể cả dòng chưa có điểm). Chỉ giữ các môn có **từ 2 dòng trở lên**, sắp xếp theo số dòng **giảm dần**. Hiển thị `title` và số dòng.
Điều kiện đề (chép nguyên văn):
Kết quả mong đợi (tính tay, mỗi dòng có tên môn và số dòng):
```sql
-- Trả lời:

```
Kết quả chạy, khớp hay không khớp:

**M4.** Trong nhóm các dòng ghi danh môn Python (course_id = 2), xếp hạng điểm từ cao xuống thấp bằng `RANK()`, và **bỏ qua các dòng chưa có điểm**. Hiển thị `student`, `score`, hạng.
Điều kiện đề (chép nguyên văn):
Kết quả mong đợi (tính tay):
```sql
-- Trả lời:

```
Kết quả chạy, khớp hay không khớp:

---

## B. Xác suất và thống kê (4 câu)
**X1.** `P(A) = 0,3`, `P(B) = 0,4`, `P(A ∩ B) = 0,12`.
(a) `P(A ∪ B)`: công thức, rồi số:
(b) `P(A | B)`: công thức, rồi số:
(c) A và B có độc lập không? Kiểm bằng công thức, ghi cả hai vế:

**X2.** Bộ lọc thư rác: 20% thư là thư rác. Bộ lọc đánh dấu "rác" đúng 90% thư rác, và đánh dấu nhầm 5% thư không phải rác. Một thư bị **đánh dấu rác**.
(a) `P(đánh dấu | rác)` = ___ ; `P(đánh dấu | không rác)` = ___
(b) `P(đánh dấu)`: công thức, rồi số:
(c) **Đại lượng đề hỏi: xác suất thư đó thật sự là rác** (biết nó bị đánh dấu): công thức, rồi số (3 chữ số thập phân):
(d) Kiểm bằng tần suất tự nhiên (10.000 thư): ___ thư rác, ___ trong đó bị đánh dấu; ___ thư không rác, ___ trong đó bị đánh dấu; tỉ lệ = ___ / ___ = ___

**X3.** Hộp có 6 bi đỏ và 4 bi xanh. Rút hai bi liên tiếp, **không hoàn lại**.
(a) `P(bi thứ hai xanh | bi thứ nhất đỏ)`: phân số ___ , thập phân 3 chữ số ___
(b) `P(bi thứ nhất đỏ và bi thứ hai xanh)`: công thức, rồi phân số ___ , thập phân 3 chữ số ___

**X4.** Một mẫu có `n = 64`, trung bình = 80, `s = 16`.
(a) `SE`: công thức, thay số, rồi số:
(b) Khoảng tin cậy 95%: công thức, thay số, rồi hai đầu mút (2 chữ số thập phân):
(c) Kiểm: đầu trái nhỏ hơn đầu phải chưa? Trung bình mẫu có nằm giữa hai đầu chưa? Rồi chạy `print(...)` để đối chiếu và dán nguyên văn:

## Ghi nhật ký (điền đủ ba ô)
- Thời gian bắt đầu và kết thúc thực tế (Phần 1, Phần 2):
- Câu bạn nghĩ mình sai nhiều khả năng nhất, và vì sao:
- Điều chưa hiểu (nếu "không có" thì nêu đã chạy code kiểm những câu nào):
