---
title: Buổi học Tuần 3 - Ngày 2 (Thứ Hai 26/10)
tags: [bai-tap, kiem-tra, thong-ke, logic, python, tuan-3]
---

# Tuần 3 - Ngày 2: Kiểm tra nhanh, chữa nợ hôm qua, hàm/lambda/sorted/slicing

**Tổng khoảng 2 giờ 30, chia 3 khối (nghỉ ít nhất 1 giờ giữa các khối). Không vượt 3 giờ.**
- **KHỐI 1 (sáng, 40 phút):** đọc Sổ lỗi 10 phút, rồi **bài kiểm tra nhanh 30 phút liền**. File riêng: `kiem_tra_nhanh_26_10.md`. Làm **trước khi** học phần chữa nợ bên dưới, lúc đầu óc tỉnh nhất, để bài đo đúng cái bạn nhớ được.
- **KHỐI 2 (chiều, 60 phút):** chữa nợ hôm qua: kiểm định 4 bước (T4, T5, T6), độ lệch chuẩn (T1), logic (L6, L8, L9), xác suất (P1, P3), 2 lỗi SQL nhỏ.
- **KHỐI 3 (tối, 50 phút):** Python: hàm, `lambda`, `sorted` với `key`, slicing (F1 đến F9); nhật ký.

**Phạm vi:** mọi câu chỉ dùng nội dung có trong bài học cùng khối (kèm ví dụ làm sẵn) và các buổi trước. Câu nào cần thứ chưa học, hoặc đề có hai cách hiểu: ghi "ngoài phạm vi" hoặc "đề mơ hồ vì ...", làm theo cách hiểu hợp lý nhất, tôi chấm theo cách đó.

> [!warning] Quy tắc kiểm đề (giữ nguyên từ hôm qua)
> Sau mỗi câu: (1) đọc lại đề, đếm số yêu cầu, mỗi yêu cầu có một dòng trả lời chưa; (2) dòng cuối có đúng là đại lượng đề hỏi (không phải đại lượng trung gian); (3) có đơn vị chưa. Ô "mong đợi" điền **trước** khi chạy code; code và mô phỏng là trọng tài (lệch hơn 0,01 là có sai). Nhật ký điền đủ ba ô, thiếu là coi như chưa nộp.

---

# KHỐI 1 (sáng): Kiểm tra nhanh 30 phút

1. Mở file lộ trình, đọc phần **Sổ lỗi** của 23, 24, 25/10 (10 phút). Không ghi gì.
2. Mở `kiem_tra_nhanh_26_10.md`, bấm giờ 30 phút, làm một lần liền theo luật trong file đó.
3. **Chưa** làm Khối 2 trước khi nộp bài kiểm tra cho tôi chấm hoặc ít nhất đã làm xong và đóng file. Đạt từ 70 điểm thì qua hẳn Tuần 3. Dưới 70 thì Tuần 3 dừng, ôn đúng phần sai.

---

# KHỐI 2 (chiều): Chữa nợ

## 2.1. Sửa lỗi kiểm định: quy trình 4 bước

Hôm qua lỗi nằm ở chỗ **lấy sai số** (dùng 62 thay vì 64) và **đọc bảng**. Quy trình dưới đây cố định, mỗi bước một dòng, không bước nào bỏ qua:

1. **Dưới H0:** kỳ vọng `n × p` và độ lệch chuẩn `√(n × p × (1 − p))`.
2. **z** = (giá trị quan sát − kỳ vọng) / độ lệch chuẩn .
3. **So `z` (bỏ dấu âm) với 1,96 và 2,58.**
4. **Kết luận** bằng một câu, về **dữ liệu**, không nói "xác suất H0 đúng".

**Bảng mốc** (đọc ngang: cột trên là `z` bỏ dấu âm, cột dưới là p hai phía tương ứng):

| z (bỏ dấu âm) | p hai phía |
|---|---|
| 1,645 | 0,10 |
| 1,96 | 0,05 |
| 2,58 | 0,01 |
| 3 | 0,0027 |

Cách dùng: `z` từ 1,96 trở lên thì `p ≤ 0,05`, **bác bỏ H0 ở mức 5%**. `z` nhỏ hơn 1,96 thì `p > 0,05`, **chưa đủ bằng chứng bác bỏ H0**. `z` từ 2,58 trở lên thì `p ≤ 0,01`.

**Ví dụ làm sẵn 1 (bác bỏ):** tung đồng xu 100 lần, được 62 ngửa. H0: công bằng.
1. Kỳ vọng `100 × 0,5 = 50`; độ lệch chuẩn `√(100 × 0,5 × 0,5) = 5`.
2. `z = (62 − 50)/5 = 2,4`.
3. 2,4 ≥ 1,96 nên `p ≤ 0,05`; 2,4 < 2,58 nên `p > 0,01`.
4. Ở mức 5%, bác bỏ H0: nếu đồng xu công bằng thì kết quả lệch như vậy hiếm (dưới 5% số lần làm thí nghiệm).

**Ví dụ làm sẵn 2 (chưa bác bỏ):** 100 lần, được 54 ngửa.
1. Kỳ vọng 50; độ lệch chuẩn 5.
2. `z = (54 − 50)/5 = 0,8`.
3. 0,8 < 1,96 nên `p > 0,05`.
4. Chưa đủ bằng chứng bác bỏ H0. (Không có nghĩa đồng xu chắc chắn công bằng.)

**Ví dụ làm sẵn 3 (phát biểu đúng/sai về p):**
- "p = 0,01 nghĩa là xác suất đồng xu công bằng chỉ 1%." **Sai.** p là xác suất của **dữ liệu** (cực đoan bằng hoặc hơn) **khi giả sử H0 đúng**, không phải xác suất của H0.
- "p = 0,50 nghĩa là chưa đủ bằng chứng bác bỏ H0." **Đúng.** `p ≥ 0,05` thì không bác bỏ, nhưng cũng không chứng minh H0 đúng.

### Bài tập
**T4.** Tung một đồng xu **100 lần**, được **64 ngửa**. H0: đồng xu công bằng.
Bước 1 (kỳ vọng, độ lệch chuẩn): kỳ vọng = 100 * 0.5 = 50, độ lệch chuẩn = √(100 * 0.5 * 0.5) = 5
Bước 2 (`z`, ghi số 64 vào công thức): z = (giá trị quan sát - kỳ vọng) / độ lệch chuẩn = (64 - 50) / 5 = 2.8
Bước 3 (so với 1,96 và 2,58; p so với 0,05 và 0,01): z = 2.8 >= 1.96 tức p =< 0.05 , z = 2.8 >  2.58 tức p < 0.01
Bước 4 (kết luận một câu ở mức 5%): bác bỏ H0 ở mức 5%

**T5.** Tung một đồng xu **100 lần**, được **57 ngửa**. H0: đồng xu công bằng.
Bước 1: kỳ vọng = 100 * 0.5 = 50, độ lệch chuẩn = √(100 * 0.5 * 0.5) = 5
Bước 2: z = (gúa trị quan sát - kỳ vọng) / độ lệch chuẩn = (57 - 50) / 5 = 1.4
Bước 3: z = 1.4 < 1.96 tức p > 0.05
Bước 4: chưa đủ bằng chứng bác bỏ H0 

**T6.** Mỗi mệnh đề **đúng hay sai**? Mỗi mệnh đề một câu giải thích.
(a) "p = 0,04 nghĩa là xác suất H0 đúng là 4%."
(b) "p = 0,04 nghĩa là: nếu H0 đúng thì xác suất thấy dữ liệu cực đoan bằng hoặc hơn dữ liệu quan sát là 4%, nên ở mức 5% ta bác bỏ H0."
(c) "p = 0,30 nghĩa là đồng xu chắc chắn công bằng."
(a): chưa rõ đúng hay sai cũng chưa rõ giải thích thế nào
(b):đúng nhưng chưa biết giải thích sao
(c): sai vì p là xác suất dữ liệu khi giả sử H0 đúng

## 2.2. Sửa lỗi độ lệch chuẩn (T1): `n` là số phần tử, không phải trung bình

Hôm qua bạn lấy `n` bằng giá trị trung bình. **`n` = đếm số phần tử trong dãy** (`len`). Trung bình là một đại lượng khác.

**Ví dụ làm sẵn:** `x = [1, 2, 3, 4, 10]`. `n = 5` (đếm), trung bình = 20/5 = 4. Bình phương lệch: 9, 4, 1, 0, 36, tổng 50. Tổng thể (chia n): `50/5 = 10`, độ lệch chuẩn `√10 ≈ 3,162`. Mẫu (chia n − 1): `50/4 = 12,5`, độ lệch chuẩn `√12,5 ≈ 3,536`.

**T1.** `w = [2, 4, 6, 8, 10, 12]`.
(a) `n` (đếm số phần tử): 6
(b) Trung bình: 7
(c) Tổng bình phương lệch: 70
(d) Độ lệch chuẩn **mẫu** (chia n − 1), công thức rồi số 3 chữ số thập phân: 3.741
(e) Độ lệch chuẩn **tổng thể** (chia n), công thức rồi số 3 chữ số thập phân: 3.415
(f) Dự đoán `np.std(w)` và `pd.Series(w).std()` ra số nào ở (d) hoặc (e). **Rồi chạy kiểm tra**, khớp hay không khớp: np.std(w) ra số 3.415 ở e và pd.Series(w).std() ra số 3.714 ở d kết quả khớp
3.415650255319866
3.7416573867739413


## 2.3. Sửa lỗi logic

**Phản đảo (L6).** "Nếu P thì Q" tương đương "Nếu không Q thì không P". Ví dụ: "Nếu có chìa khóa thì mở được cửa." Cửa **không mở được** thì kết luận **không có chìa khóa** (dùng phản đảo). Lý do: nếu có chìa khóa thì chắc chắn mở được cửa; cửa không mở được thì không thể là trường hợp có chìa khóa.

**Phản ví dụ (L8).** Để chứng tỏ "biết Q thì **không** kết luận được P", chỉ cần **một trường hợp có Q mà không có P**. Ví dụ: "Mọi người trúng tuyển đều đã nộp hồ sơ." Biết Hoa đã nộp hồ sơ thì **không** kết luận được Hoa trúng tuyển, vì có thể Hoa nộp hồ sơ nhưng bị loại. Cách trả lời tốt: nêu **kết luận** và **một trường hợp cụ thể** làm phản ví dụ.

**Phủ định của "cả hai nói dối" (L9).** Phủ định của "cả hai đều nói dối" là "**ít nhất một người nói thật**" (không phải "cả hai nói thật"). Và sau khi tìm ra đáp án, **kiểm lại từng câu nói** với kết quả: người nói thật thì câu của họ đúng, người nói dối thì câu của họ sai.

**L6.** Mệnh đề: "Nếu hôm nay là thứ Hai thì Lan đi học." Hôm nay Lan **không đi học**. Kết luận được gì về hôm nay? Cho một câu lý do dùng từ "phản đảo".
Trả lời: Hôm nay Lan không đi học thì hôm nay không phải là thứ 2 vì nếu hôm nay là thứ 2 thì Lan chắc chắn đi học nhưng nếu nay Lan không đi học thì không thể là trường hợp không phair là thứ 2

**L8.** Mệnh đề: "Mọi học viên nộp bài đúng hạn đều được xét điểm." (a) Bình **không được xét điểm**. Kết luận gì về Bình? (b) Chi **được xét điểm**. Kết luận được Chi đã nộp đúng hạn không? Nêu kết luận và **một phản ví dụ cụ thể**.
(a):Bình không được xét điểm thì có nghĩa là Bình không nộp bài đúng hạn
(b): Chi được xét điểm không đồng nghĩa Chi nộp đúng hạn vì Chi có thể đã nộp quá hạn nhưng thầy cô châm trước coi Chi đã nộp đúng hạn để xét điểm

**L9.** Ba người An, Bình, Chi. Mỗi người luôn nói thật hoặc luôn nói dối.
- An nói: "Bình và Chi đều nói thật."
- Bình nói: "An nói dối."
- Chi nói: "Bình nói dối."
(a) Giả sử An nói thật, suy ra gì, mâu thuẫn ở đâu?
(b) Vậy An nói gì (thật hay dối)? Từ đó suy ra Bình, Chi.
(c) Kiểm lại cả ba câu nói với kết quả của bạn (mỗi người một dòng):
(a): Nếu An nói thật thì Bình và Chi thực sự nói thật, mà Bình nói An nói dối tức An thực sự nói dối => mâu thuẫn An vừa nói dối vừa nói thật
(b): Từ kết luận từ a thì An không nói thật được tức An nói dối suy ra lời An nói là ít nhất 1 người nói thật Bình hoặc Chi, nếu Bình nói thật thì lời Bình nói An nói dối là đúng => không mâu thuẫn đúng sự thật nên Bình nói thật còn Chi nói dối. Nếu Bình nói dối thì lời bình nói An nói dối là sai => mâu thuẫn An vừa nói thật vừa nói dối 
(c): Nếu An nói thật thì mâu thuẫn An vừa nói dối vừa nói thật
Nếu An nói dối tức Bình hoặc Chi nói thật, Bình nói thật thì Chi nói dối
Bình nói dồi thì mâu thuẫn An vừa nói thật vừa nói dối

## 2.4. Sửa xác suất (P1, P3 viết lại cho rõ)

**P1.** Hộp có 4 bi đỏ và 6 bi xanh. Rút lần 1 một bi, **không bỏ lại**, rồi rút lần 2 một bi.
(a) Lúc **bắt đầu** hộp có 10 bi. Sau khi lần 1 rút được **bi đỏ**, hộp còn 9 bi, trong đó 3 bi đỏ.
(b) `P(lần 1 đỏ)` = 4 / 10 = 0.4; `P(lần 2 đỏ | lần 1 đỏ)` = 3/9 = 0.333
(c) Công thức `P(cả hai đỏ)` (dạng đúng cho hai biến cố phụ thuộc), rồi số: phân số 2/15 , thập phân 3 chữ số 0.133

**P3.** Một xúc xắc công bằng, ba câu hỏi **độc lập nhau**.
(a) Tung **một lần**. `P(ra 6)` = ?
(b) Chưa tung lần nào. Sắp tung **ba lần liên tiếp**. `P(cả ba lần đều ra 6)` = ? (công thức rồi số)
(c) Hai lần tung đầu **đã xảy ra** và đều ra 6. Tung lần thứ 3. `P(lần 3 ra 6)` = ?
(d) So sánh (b) và (c): số nào lớn hơn? Giải thích một câu (dùng từ "độc lập", và nói rõ vì sao ở (c) hai lần đầu không còn là ẩn số).
(a): P(ra 6) = 1/6
(b): P(cả 3 đều ra 6) = P(lần 1 ra 6) * P(lần 2 ra 6) * P(lần 3 ra 6) = 1/216 
(c): P(lần 3 ra 6) = 1/6
(d): kết quả c lớn hơn b vì mỗi lần tung xúc sắc đều độc lập lần 1 lần 2 ra 6 không khiến lần 3 ra 6 dễ hơn khó hơn hay không thể ra

## 2.5. Hai lỗi SQL nhỏ

Hai điều cần nhớ (bạn đã biết, chỉ cần viết đúng):
- Khi có **hòa**, `ROW_NUMBER()` đánh số khác nhau cho hai dòng hòa, nhưng ai nhận số 1 là **không xác định** vì `ORDER BY` không phân biệt được hai người hòa.
- **Cộng dồn** do **`ORDER BY` bên trong `OVER (...)`**. `PARTITION BY` chỉ chia nhóm (mỗi nhóm cộng dồn riêng), không tạo ra cộng dồn.

**Q1.** Bảng `scores` hôm qua: lớp A có Bình 9 và Chi 9 cùng điểm cao nhất. Với `ROW_NUMBER() OVER (PARTITION BY class ORDER BY score DESC)`, ai nhận `rn = 1`: Bình, Chi, hay không xác định? Một câu lý do.
Trả lời: không xác định được trong lần chạy này thì Bình nhận rn = 1 vì ORDER BY không phân biệt được 2 người hòa

**Q2.** Cần cột "tổng cộng dồn" của `qty` theo `sale_id`. Từ khóa nào bên trong `OVER (...)` tạo ra cộng dồn: `PARTITION BY` hay `ORDER BY`? Viết đúng câu `SELECT` (bảng `sales(sale_id, product_id, qty)`).
Trả lời: ORDER BY
```sql
-- Trả lời:
SELECT * SUM(qty) OVER(PARTITION BY sale_id ORDER BY qty) FROM sales
```

---

# KHỐI 3 (tối): Python: hàm, lambda, sorted với key, slicing

## Bài học

### 1. Hàm
```python
def tang_luong(luong, ty_le=0.1):      # ty_le có giá trị mặc định 0.1
    return luong * (1 + ty_le)

print(tang_luong(1000))         # 1100.0   (dùng mặc định)
print(tang_luong(1000, 0.2))    # 1200.0   (truyền giá trị khác)
```
- `return` trả giá trị về chỗ gọi hàm. **Hàm không có `return` thì trả về `None`.**
- `print` chỉ **in ra màn hình**, không trả giá trị. Hàm chỉ `print` mà không `return`, thì `kq = ham(...)` làm `kq` là `None`.
- Hàm có thể trả **nhiều giá trị**: thật ra là một tuple, nhận bằng cách tách:
```python
def min_max(x):
    return min(x), max(x)

lo, hi = min_max([3, 1, 4])    # lo = 1, hi = 4
```

### 2. `lambda`
Hàm một dòng, không tên: `lambda x: x * 2` tương đương `def f(x): return x * 2`.
```python
f = lambda x: x * 2
print(f(5))    # 10
```
Thường dùng làm tham số `key` của `sorted`.

### 3. `sorted` với `key`
`sorted(danh_sach, key=ham)` gọi `ham` cho từng phần tử, rồi **sắp theo giá trị hàm trả về**. `reverse=True` để giảm dần. `sorted` trả về **danh sách mới**, không sửa danh sách cũ (còn `x.sort()` sửa tại chỗ và trả về `None`).
```python
words = ["banana", "kiwi", "fig"]
print(sorted(words, key=len))                          # ['fig', 'kiwi', 'banana']
hs = [("An", 8), ("Bình", 9), ("Chi", 7)]
print(sorted(hs, key=lambda s: s[1]))                  # [('Chi', 7), ('An', 8), ('Bình', 9)]
print(sorted(hs, key=lambda s: s[1], reverse=True))    # [('Bình', 9), ('An', 8), ('Chi', 7)]
```
(`s[1]` là phần tử thứ hai của tuple: điểm.)

### 4. Slicing: `x[start:stop:step]`
- **`stop` không được lấy** (lấy từ `start` đến `stop − 1`). `start` mặc định 0, `stop` mặc định hết, `step` mặc định 1.
- Chỉ số âm đếm từ cuối: `-1` là phần tử cuối.
```python
x = [10, 20, 30, 40, 50, 60]
print(x[1:4])     # [20, 30, 40]
print(x[:3])      # [10, 20, 30]
print(x[-2:])     # [50, 60]
print(x[::2])     # [10, 30, 50]
print(x[::-1])    # [60, 50, 40, 30, 20, 10]  (đảo ngược)
```
- **Quan trọng:** cắt **list** cho **bản sao** (sửa bản cắt không đổi bản gốc). Cắt **mảng NumPy** cho **một "cửa sổ" nhìn vào cùng dữ liệu** (sửa bản cắt **đổi luôn** mảng gốc). Cùng họ với lỗi `b += 10` hôm trước.
```python
lst = [1, 2, 3, 4]; l2 = lst[1:3]; l2[0] = 99
print(lst)       # [1, 2, 3, 4]   (list gốc không đổi)

arr = np.array([1, 2, 3, 4]); a2 = arr[1:3]; a2[0] = 99
print(arr)       # [ 1 99  3  4]  (mảng gốc đổi theo)
```
- Mảng 2 chiều: `m[hàng, cột]`; `m[:, 1]` là **cả cột 1**, `m[0, :]` là **cả hàng 0**.

## Bài tập
Dự đoán / điền "Kết quả mong đợi" **trước khi chạy**, rồi chạy kiểm tra và ghi khớp hay không khớp.

**F1.** Dự đoán:
```python
def gia(x, thue=0.25):
    return x * (1 + thue)
print(gia(200))
print(gia(200, 0.5))
```
Dự đoán dòng 1: 250
Dự đoán dòng 2: 300

**F2.** Dự đoán và giải thích:
```python
def binh_phuong(x):
    print(x * x)
kq = binh_phuong(4)
print(kq)
```
Dự đoán (hai dòng in ra): in ra 16 và None
Giải thích: tôi vừa chạy thử lệnh trên và tôi nghĩ mặc định gọi hàm thì hàm sẽ chạy lệnh bên trong tức chạy lệnh in ra 4 * 4 tức 16 còn kết quả trả về nếu không có return thì mặc định là None nên kq nhận giá trị là None và in ra là None

**F3.** Viết hàm `tong_tb(x)` nhận list số `x` và **trả về hai giá trị**: tổng, rồi trung bình.
Điều kiện đề (chép nguyên văn): viết hàm tong_tb(x) nhận list số x và trả về hai giá trị tổng, rồi trung bình
Kết quả mong đợi với `tong_tb([2, 4, 6])` (tính tay):
12 4
```python
# Trả lời:
def tong_tb(x):
	tong = 0
	for i in x:
		tong += i
	return tong, tong / len(x)
	
print(tong_tb([2,4,6]))
```

**F4.** Cho `hs = [("An", 8), ("Bình", 9), ("Chi", 7)]`. Sắp xếp theo **điểm giảm dần**.
Điều kiện đề (chép nguyên văn): sắp xếp theo điểm giảm dần
Kết quả mong đợi (tính tay): ("Bình", 9) , ("An", 8), ("Chi", 7)
```python
# Trả lời:
hs = [("An", 8), ("Bình", 9), ("Chi", 7)]

print(sorted(hs, key=lambda x: x[1], reverse=True))
```

**F5.** Cho `a = [-5, 2, -1, 4]`. Sắp xếp theo **giá trị tuyệt đối tăng dần** (số gốc giữ nguyên dấu), bằng `sorted` với `key=lambda`.
Điều kiện đề (chép nguyên văn): sắp xếp theo giá trị tuyệt đối tăng dần (số gốc giữ nguyên dấu) bằng sorted với key=lamda
Kết quả mong đợi (tính tay): -1, 2, 4 -5
```python
# Trả lời:
a = [-5, 2, -1, 4]

print(sorted(a, key=lambda x: abs(x)))
```

**F6.** Với `x = [10, 20, 30, 40, 50, 60]`, dự đoán từng biểu thức (chạy độc lập):
(a) `x[1:4]`: [20, 30,40]
(b) `x[-2:]`: [50,60]
(c) `x[::2]`: [10,30,50]
(d) `x[:-1]`: [10,20,30,40,50]

**F7.** Dự đoán đầu ra và giải thích khác biệt:
```python
lst = [1, 2, 3, 4]
l2 = lst[1:3]
l2[0] = 99
print(lst)

arr = np.array([1, 2, 3, 4])
a2 = arr[1:3]
a2[0] = 99
print(arr)
```
Dự đoán: print lần 1 [1,2,3,4] print lần 2 [1,99,3,4]
Giải thích (vì sao hai kết quả khác nhau): vì slicing của list là tạo list mới list cũ giữ nguyên còn slicing của mảng numpy thì chỉ là tạo một cửa sổ mới của dữ liệu, sửa giá trị trên cửa sổ thì sửa giá trị của mảng numpy 

**F8.** Cho `x = [1, 2, 3, 4, 5, 6]`. Lấy **3 phần tử cuối**, theo thứ tự **từ cuối về đầu**, bằng slicing.
Điều kiện đề (chép nguyên văn): lấy 3 phần từ cuổi theo thứ tự từ cuối về đầu bằng slicing
Kết quả mong đợi (tính tay): [4,5,6]
```python
# Trả lời:
x = [1, 2, 3, 4, 5, 6]

print(x[-1:2:-1])
```

**F9.** Dùng `hs` ở F4. Lấy **hai** học sinh **điểm cao nhất** (cả tuple), điểm cao đứng trước, bằng `sorted` rồi slicing.
Điều kiện đề (chép nguyên văn): lấy 2 học sinh điểm cao nhất cả tuple, điểm cao đứng trước, sử dụng sorted rồi slicing
Kết quả mong đợi (tính tay):
```python
# Trả lời:
hs = [("An", 8), ("Bình", 9), ("Chi", 7)]

print(sorted(hs, key=lambda x: x[1], reverse=True)[:2])
```

---

## Ghi nhật ký (bắt buộc điền cả ba)
- Điểm bài kiểm tra nhanh tự đánh giá và thời gian làm thực tế (Khối 1), thời gian Khối 2 và Khối 3: chiều 4h45 - 5h45, tối 7h45 - 8h25
- Câu bạn nghĩ mình sai nhiều nhất, và vì sao: Q5 vì tôi chưa xác định được rõ là ORDER BY không xác định được hay là cứ ai kết quả xuất hiện đầu tiên là rn = 1
- Điều chưa hiểu cần hỏi Claude (nếu viết "không có" thì phải nêu đã kiểm tra những câu nào bằng cách chạy code hoặc đối chiếu): F2 2 dòng in ra ý là sao nghe giống như tiết lộ đáp án, F8 đề khó hiểu không rõ là lấy kiểu gì nên tôi làm theo cách hiểu nên tính tay và kết quả chạy sẽ khác vị trí

> Làm xong gửi cho Claude chấm: **`kiem_tra_nhanh_26_10.md` (gửi sớm, ngay sau Khối 1)** và `buoi_hoc_tuan3_ngay2.md`. Từ 28/10, mỗi ngày 30 đến 60 phút theo mục 5d của lộ trình (27/10 nhẹ).
