---
title: Bài kiểm tra cổng Tuần 2
tags: [kiem-tra, tuan-2, cong]
---

# Bài kiểm tra cổng Tuần 2 (20 câu, 100 điểm)

Đạt từ **70 điểm** thì qua Tuần 2. Dưới 70 thì ở lại, ôn đúng phần sai rồi làm bài kiểm tra mới.

## Luật làm bài
- **Không tra cứu, không hỏi AI, không chạy code** trong lúc làm. Giấy nháp được dùng.
- **Làm một lần liền 90 phút, bấm giờ, vào lúc đầu óc tỉnh nhất** (nên là buổi sáng). Hết giờ thì dừng.
- **Trước khi làm:** đọc lại phần **Sổ lỗi** trong file lộ trình 15 phút, đặc biệt các mục Tuần 2 Ngày 3 và Ngày 4. Đây là bước 0, không tính điểm.
- Mỗi câu 5 điểm. Câu không biết thì ghi "không biết" (không bị trừ nhiều hơn một câu trả lời sai).
- Câu viết code có ô **Điều kiện đề**: chép **nguyên văn** mọi điều kiện (hình thức, "không dùng ...", "lớn hơn" hay "từ ... trở lên", giảm dần hay tăng dần, một con số hay một nhóm). Chép thiếu hoặc sai điều kiện thì chỉ tính nửa điểm.
- Câu viết code có ô **Kết quả mong đợi**: tính tay **trước** khi viết code.
- Câu có chữ "giải thích": đúng kết quả nhưng không giải thích được thì chỉ tính nửa điểm.
- Câu tính xác suất: viết **công thức** trước rồi mới thay số. Bảng phân phối thì **tổng xác suất phải bằng 1**, tự kiểm tra.
- Làm xong gửi nguyên file này cho Claude chấm.

---

## Dữ liệu cho phần A (SQL)

Bảng `employees`:

| emp_id | name | dept_id | salary |
| ------ | ---- | ------- | ------ |
| 1      | An   | 10      | 1900   |
| 2      | Bình | 10      | 3000   |
| 3      | Chi  | 20      | 2500   |
| 4      | Dũng | 20      | 2500   |
| 5      | Hà   | 30      | 4000   |
| 6      | Khoa | NULL    | 1500   |

Bảng `departments`:

| dept_id | dept_name |
|---|---|
| 10 | Kỹ thuật |
| 20 | Kinh doanh |
| 40 | Nhân sự |

(`NULL` nghĩa là Khoa chưa thuộc phòng nào. Chú ý phòng 30 không có trong bảng `departments`, và phòng 40 không có nhân viên nào.)

## A. SQL (8 câu)

**A1.** Dự đoán kết quả của `SELECT COUNT(*) FROM employees;` và của `SELECT COUNT(dept_id) FROM employees;`. Giải thích vì sao hai số khác nhau.
Dự đoán: SELECT COUNT(*) FROM EMPLOYEES ra kết quả là 6
SELECT COUNT(dept_id) FROM employees thì ra kết quả là 5
Giải thích: SELECT COUNT(*) FROM EMPLOYEES, count(*) đếm tất cả các dòng trong bảng kể cả hàng có NULL hay không
SELECT COUNT(dept_id) FROM employees, COUNT(dept_id) thì chỉ đếm các dòng trong bảng không chứa NULL

**A2.** `SELECT e.name, d.dept_name FROM employees e JOIN departments d ON e.dept_id = d.dept_id;` trả về bao nhiêu dòng? Những ai **không** xuất hiện, và **lý do của từng người** là gì?
Dự đoán: Trả về 4 dòng, Hà và Khoa không xuất hiện
Giải thích: JOIN này là INNER JOIN tức chỉ gộp các hàng dữ liệu có cùng khóa là dept_id của 2 bảng employees và departments

**A3.** Cùng truy vấn A2 nhưng dùng `LEFT JOIN`. Trả về bao nhiêu dòng? `dept_name` của Hà và của Khoa là gì?
Dự đoán: Trả về 6 dòng, dept_name của Hà và khoa là NULL hoặc NONE vì LEFT JOIN giữ lại toàn bộ hàng trong bảng employees đồng nghĩa các hàng không cùng khóa dept trong department thì dept_name sẽ để là None hoặc NULL tôi chưa chạy nên chưa rõ

**A4.** Liệt kê tên các phòng ban **chưa có nhân viên nào**.
Điều kiện đề (chép nguyên văn): liệt kê các phòng ban chưa có nhân viên nào
Kết quả mong đợi (tính tay):

| dept_id | dept_name |
| ------- | --------- |
| 40      | Nhân sự   |

```sql
-- Trả lời:
SELECT d.dept_id, d.dept_name FROM departments d LEFT JOIN employees e ON e.dept_id = d.dept_id WHERE d.dept_name IS NULL
```

**A5.** Với mỗi `dept_id`, tính tổng `salary`. **Chỉ giữ** các nhóm có tổng **lớn hơn** 4000, sắp xếp theo tổng **giảm dần**. Hiển thị `dept_id` và tổng.
Điều kiện đề (chép nguyên văn): với mỗi dept_id tính tổng salary. Chỉ giữ lại các nhóm có tổng lớn hơn 4000, sau đó sắp xếp theo tổng giảm dần, Hiện thị dept_id và tổng
Kết quả mong đợi (tính tay, mỗi dòng có `dept_id` và tổng):

| dept_id | SUM(salary) |
| ------- | ----------- |
| 20      | 5000        |
| 10      | 4900        |


```sql
-- Trả lời:
SELECT dept_id, SUM(salary) FROM employees GROUP BY dept_id HAVING SUM(salary) > 4000 ORDER BY SUM(salary) DESC
```

**A6.** Khi tìm nhân viên lương cao nhất của mỗi phòng bằng `ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC)` rồi lọc `rn = 1`, **phòng 20** trả về mấy dòng? Chọn kết quả nào trong hai người lương bằng nhau? Muốn lấy **cả hai** người thì dùng hàm nào thay `ROW_NUMBER`?
Trả lời: phòng 20 trả về 2 dòng
2 câu hỏi sau tôi chưa rõ 

**A7.** Với mỗi nhân viên, hiển thị `name`, `salary` và **tổng lương cộng dồn toàn công ty** tính theo thứ tự `emp_id` (người 1 trước, người 6 sau).
Điều kiện đề (chép nguyên văn): với mỗi nhân viên hiển thị name, salary và tổng cộng dồn toàn công ty theo thứ tự emp_id 
Kết quả mong đợi (tính tay, cột cộng dồn của người 1 đến 6):

| name | salary | tongcongty |
| ---- | ------ | ---------- |
| An   | 1900   | 15400      |
| Bình | 3000   | 15400      |
| Chi  | 2500   | 15400      |
| Dũng | 2500   | 15400      |
| Hà   | 4000   | 15400      |
| Khoa | 1500   | 15400      |

```sql
-- Trả lời:

```

**A8.** `SELECT name FROM employees WHERE dept_id <> 10;` trả về những ai? Khoa có trong kết quả không? Giải thích bằng cách chọn **một** trong ba giá trị logic (đúng, sai, không xác định).
Dự đoán: trả về Chí Dũng và Hà Khoa không có tỏng kết quả 
Giải thích: WHERE chỉ lấy các hàng cho kết quả so sánh là đúng không lấy các dòng cho kết quả so sánh là sai hoặc không xác định, An và Bình có dept_id = 10 nên kết quả so sánh là sai, Khoa có dept_id là NULL nên kết quả so sánh là không xác định được

---

## B. Xác suất (8 câu)

**B1.** `P(A) = 0,5`, `P(B) = 0,4`, `P(A ∩ B) = 0,15`. Tính `P(A ∪ B)` và `P(A | B)`. Hai biến cố này có độc lập không? Kiểm bằng công thức.
Công thức: P(A ∪ B) = P(A) + P(B) - P(A ∩ B)
P(A|B) = P(A ∩ B) / P(B)
Kết quả: P(A ∪ B) = 0.75
P(A|B) = 0.375
Độc lập không, vì sao: A và B không độc lập vì P(A) * P(B) = 0.2 
P(A ∩ B) = 0.15 tức P(A ∩ B) != P(A) * P(B)  mà A và B độc lập thì P(A ∩ B) = P(A) * P(B)

**B2.** Tung một xúc xắc. `A` = "ra số nguyên tố" (nhớ: 1 **không** phải số nguyên tố), `B` = "ra số lẻ".
Liệt kê tập: `A = {2,3,5}`, `B = {1,3,5}`, `A ∩ B = {3,5}`, `A ∪ B = {1,2,3,5}`
Tính `P(A ∩ B)`, `P(A ∪ B)` (bằng đếm tập, rồi kiểm lại bằng công thức), và `P(A | B)`. Hai biến cố có độc lập không, bằng công thức?
Trả lời: P(A ∩ B) = 2/6 = 1/3
P(A ∪ B) = 4/6 = 2/3
P(A ∩ B) = P(A | B) * P(B) = (2/3) * (1/2) = 1/3
P(A ∪ B) = P(A) + P(B) - P(A ∩ B) = (1/2) + (1/2) - (1/3) = 2/3

**B3.** Hộp có 4 bi đỏ và 5 bi xanh. Rút **hai** bi liên tiếp, **không hoàn lại**. Tính `P(cả hai xanh)` và `P(bi thứ hai xanh | bi thứ nhất xanh)`.
Trước khi nhân, ghi: lần 1 còn 9 bi (5 xanh); lần 2 còn 8 bi (4 xanh).
Công thức: P(cả hai xanh) = (P bi thứ 1 xanh) * P(bi thứ 2 xanh) 
P(bi thứ 2 xanh | bi thứ 1 xanh) = P(cả hai xanh) / P(bi 1 xanh)
Kết quả (phân số và thập phân 3 chữ số):
P(cả hai xanh) = 0.278
P(bi thứ 2 xanh | bi thứ 1 xanh) = 0.5

**B4.** Cửa hàng có hai nhà cung cấp. X cung cấp 70% hàng, tỉ lệ hàng lỗi 4%. Y cung cấp 30% hàng, tỉ lệ lỗi 10%. Lấy ngẫu nhiên một món hàng.
(a) `P(lỗi)`. (b) Biết món hàng lỗi, `P(do Y)`.
Công thức: P(lỗi) = P(lỗi | X) * P(X) + P(lỗi | Y) * P(Y)
P(Y | lỗi)  = P(lỗi | Y) * P(Y) / P(lỗi)
Kết quả (3 chữ số thập phân):
P(lỗi) = 0.058
P(Y | lỗi) = 0.517
Kiểm tra bằng tần suất tự nhiên (10.000 món): X 7000 món, lỗi 280, Y 3000 món, lỗi 300, tổng số hàng lỗi của X và Y là 580, trong đó hàng lỗi thuộc về Y là = 300/580 = 0.517

**B5.** Một bệnh có 2% dân số mắc. Xét nghiệm có độ nhạy 80% và độ đặc hiệu 90%. Một người dương tính. Tính xác suất người đó thật sự mắc.
Ghi: `P(dương tính | mắc)` = 0.8 ; `P(dương tính | không mắc)` = 0.1
Công thức: P(dương tính) = P(dương tính | mắc) * P(mắc) + P(dương tính | không mắc) * P(không mắc)
Kết quả (3 chữ số thập phân): P(dương tính) = 0.114
Kiểm tra bằng tần suất tự nhiên (10.000 người): có 200 người mắc, 160 người xét nghiệm dương tính, 9800 người không mắc, 980 người xét nghiệm là dương tính, tổng số những người dương tính là 1140

**B6.** Rút thăm: nhận 5 nghìn với xác suất 0,2; nhận 1 nghìn với xác suất 0,5; **mất** 2 nghìn với xác suất còn lại. `X` = số tiền nhận được (nghìn đồng, âm là mất).
Bảng phân phối (tổng xác suất phải bằng 1):

| x        | 5000 | 1000 | -2000 |
| -------- | ---- | ---- | ----- |
| P(X = x) | 0.2  | 0.5  | 0.3   |
|          |      |      |       |

Công thức `E[X]`, rồi số: E[X] = x1 * P(X = x1) + x2 * P(X = x2) + x3 * P(X = x3) = 5000 * 0.2 + 1000 * 0.5 + (-2000) * 0.3 = 900

**B7.** Biết `E[X] = 2` và `E[X²] = 7`. Tính `Var(X)`, `E[3X − 1]` và `Var(3X − 1)`.
Công thức: Var(X) = E[X²] - E[X]²
E[3X - 1] = 3 * E[X] - 1
Var(3X - 1) = 9 * Var(X)
Kết quả: Var(X) = 3
E[3X - 1] = 5
Var(3X - 1) = 27

**B8.** Một đồng xu công bằng vừa ra ngửa 5 lần liên tiếp. Xác suất lần thứ 6 ra ngửa là bao nhiêu? Giải thích bằng khái niệm độc lập.
Trả lời: Xắc suất ra ngửa thứ 6 = 0.015625 vì việc tung đồng xu là độc lập tức mỗi lần tung đồng xu kết quả trước đó không ảnh hưởng đến kết quả sau, việc ra mặt ngửa ở lần đầu không khiến cho việc ngửa lần thứ 2 xảy ra hoặc không xảy ra 

---

## C. Python (4 câu)

**C1.** Dự đoán đầu ra và giải thích:
```python
a = np.array([1, 2, 3])
b = a
b += 10
c = a + 1
print(a, c)
```
Dự đoán: [11,12,13] [12,13,14]
Giải thích: b = a, b += 10 khi này b và a cùng trỏ tới 1 mảng numpy khi b cộng tất cả các phần từ với 10 thì mảng numpy mà a trỏ tới cũng được cộng khiến a cũng thay đổi theo, c = a + 1 khi này c và a không trỏ cùng 1 mảng numpy c trỏ tới một mảng numpy mới với giá trị giống a sau đó cộng tất cả phần từ với 1 còn a thì giữ nguyên

**C2.** Cho `m = np.array([[1, 2, 3], [4, 5, 6]])`. Tính **tổng từng hàng**.
Điều kiện đề (chép nguyên văn): tính tổng trung bình từng hàng
Kết quả mong đợi (tính tay): [2,5]
```python
# Trả lời:
m.mean(axis = 1)
```
Hoàn thành câu: "`axis=1` làm biến mất chiều 1, nên kết quả có 2 phần tử."

**C3.** Cho `a = [1, 2, 2, 3, 4, 4]` và `b = [3, 4, 5]`. Lấy các số xuất hiện ở **cả hai** list, **không trùng**, dưới dạng **set**.
Điều kiện đề (chép nguyên văn):lấy các số xuất hiện ở cả 2 list không trùng và dưới dạng set
Kết quả mong đợi (tính tay): {3,4}
```python
# Trả lời:
seta = set()
setb = set()

for i in a:
	seta.add(i)

for i in b:
	setb.add(i)
	
print(seta & setb)
```

**C4.** Cho `d = {"a": 1, "b": 2}`. Dự đoán kết quả (hoặc loại lỗi) của từng dòng, **chạy độc lập từng dòng**:
```python
d["c"]
d.get("c", 0)
"a" in d
1 in d
```
Dự đoán: lỗi vì khóa "c" không có trong dict
0 vì không có khóa "c" nên trả kết quả là 0
true vì có khóa "a" trong dict
false vì không có khóa 1 trong dict
Giải thích (vì sao dòng 3 và 4 khác nhau): vì in kiểm tra xem khóa có trong dict không mà "a" thì có còn 1 thì không

---

## Ghi nhật ký (bắt buộc điền cả ba)
- Thời gian làm thực tế: 8h40 - 9h54
- Câu bạn nghĩ mình sai nhiều khả năng nhất, và vì sao: A6 lí do vì tôi biết kết quả trả về nhưng không biết xử lý 2 yêu cầu sau, A7 thì tôi biết kết quả tính tay nhưng không rõ tỉnh tổng cả công ty thế nào B8 thì tôi không rõ giải thích chuẩn chưa
- Điều chưa hiểu: yêu cầu trong A6 và A7, độc lập giải thích chính xác là gì và trung bình dài hạn chính xác là gì để không bị lẫn với trung bình thường thây
