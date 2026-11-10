---
title: Thứ Hai 9/11 - Logic 2 (suy luận) và ôn SQL
tags: [logic, sql, tuan-3]
---

# 9/11 (1 giờ): Logic 2 - suy luận (45 phút) + ôn SQL (15 phút)

**Cách làm:** làm tay trên nháp trước, chỉ dùng nội dung trong bài học bên dưới. Làm tròn bình thường, lệch ≤ 0,001 vẫn đúng. Đề mơ hồ thì ghi "đề mơ hồ vì ...". Không có nhật ký. Nhãn **[đã gặp]/[mới]** như thường lệ. Mọi câu logic **có đúng một đáp án** (đã kiểm bằng cách thử hết mọi khả năng), nên nếu bạn ra nhiều đáp án thì là đề của mình sai, hãy ghi lại.

**Quy tắc cho cả buổi:** với mỗi đáp số, **thử lại với mọi điều kiện của đề** (ghi "thử lại: ..."), không chỉ với điều kiện đầu tiên.

---

# PHẦN 1 (45 phút): Logic 2 - suy luận

## Bài học (có ví dụ làm sẵn)

**1. Xếp thứ tự từ các điều kiện.** Đổi mỗi điều kiện thành một dấu `<` rồi nối thành chuỗi.
Ví dụ làm sẵn: ba bạn chạy thi. "Lan về trước Mai. Mai về trước Nga." → Lan < Mai < Nga (viết theo thứ tự về đích). Lan nhất, Nga cuối. Thử lại: Lan trước Mai ✓, Mai trước Nga ✓.

**2. Xếp vị trí có điều kiện loại trừ.** Đặt các điều kiện chắc chắn trước, rồi thử các khả năng còn lại và loại cái vi phạm.
Ví dụ làm sẵn: ba chiếc ghế vị trí 1, 2, 3 từ trái sang phải cho ba người X, Y, Z. "X ngồi ở vị trí 3. Y không ngồi ở vị trí 1." → X = 3; còn Y, Z ở vị trí 1, 2; Y không ở 1 nên Y = 2, Z = 1. Thử lại: X ở 3 ✓, Y không ở 1 ✓.

**3. Ai nói thật, ai nói dối.** Người nói thật thì mọi câu họ nói đều đúng; người nói dối thì mọi câu họ nói đều sai. Thử từng giả thiết, kiểm các câu có mâu thuẫn không.
Ví dụ làm sẵn: hai bạn P, Q. P nói: "Q nói dối." Q nói: "Cả hai chúng ta đều nói dối."
Giả sử Q nói thật → câu của Q phải đúng, tức cả hai nói dối, trong đó có Q: mâu thuẫn. Vậy Q nói dối → câu "cả hai đều nói dối" là sai, tức ít nhất một người nói thật; Q đã nói dối nên P nói thật. Thử lại: P nói thật và "Q nói dối" đúng ✓; Q nói dối và câu của Q sai ✓ (vì P nói thật). Kết luận: P thật, Q dối.

**4. Câu điều kiện "Nếu P thì Q".**
- **Hợp lệ (1):** "Nếu P thì Q. P đúng. Vậy Q đúng."
- **Hợp lệ (2):** "Nếu P thì Q. Q **sai**. Vậy P sai."
- **Không hợp lệ (lỗi khẳng định hậu quả):** "Nếu P thì Q. Q đúng. Vậy P đúng." Vì Q có thể đúng do nguyên nhân khác.
Ví dụ làm sẵn: "Nếu trời mưa thì sân ướt." Sân ướt → không suy ra được trời mưa (có thể do tưới nước). Sân khô → suy ra trời không mưa.

**5. Ngày trong tuần:** cộng số ngày rồi lấy **phần dư khi chia cho 7**. Ví dụ làm sẵn: hôm nay thứ Ba, 10 ngày sau: 10 chia 7 dư 3, thứ Ba + 3 ngày = **thứ Sáu**.

**6. Dãy số (đã học).** Viết quy luật thành công thức rồi **thử trên mọi số hạng đã cho**.

## Bài tập

**G1. [mới]** Bốn bạn An, Bình, Chi, Dũng chạy thi. Biết: An về trước Bình; Chi về sau Bình; Dũng về trước An.
(a) Thứ tự về đích (từ nhất đến bốn): Dũng < An < Bình < Chi
(b) Thử lại với cả ba điều kiện: Dũng trước An, Chi sau Bình, An trước Bình

**G2. [mới]** Một dãy phố có bốn cửa hàng ở các vị trí 1, 2, 3, 4 từ trái sang phải: hiệu sách, hiệu thuốc, quán cà phê, tiệm hoa. Biết: hiệu sách ở vị trí 1; quán cà phê nằm **ngay bên phải** hiệu thuốc (vị trí liền kề, lớn hơn 1); tiệm hoa không ở vị trí 2.
(a) Vị trí của từng cửa hàng: hiệu sách, hiệu thuốc, quán cà phê, tiệm hoa
(b) Thử lại với cả ba điều kiện: hiệu sách ở vị trí 1, quán cà phê bên phải hiệu thuốc, tiệm hoa không ở vị trí 2

**G3. [mới]** Ba bạn An, Bình, Chi, mỗi người hoặc luôn nói thật hoặc luôn nói dối. An nói: "Bình nói dối." Bình nói: "An và Chi cùng nói dối." Chi nói: "An nói dối."
(a) Ai nói thật, ai nói dối: An nói dối, Bình nói dối, Chi nói thật
(b) Thử lại: mỗi câu của ba bạn có khớp với kết luận của bạn không?
Giả sử Bình nói thật, Bình nói An và Chi cùng nói dối, mà An nói Bình nói dối, tức Bình nói thật, Bình mâu thuẫn. Nếu Bình nói dối, tức ít nhất 1 người nói thật là An hoặc Chi, nếu Chi nói thật thì An thực sự nói dối mà An nói Bình nói dối thì tức Bình nói thật. Nếu Chi nói dối thì An nói thật mà An nói Bình nói dối, Bình lại bị mâu thuẫn

**G4. [mới, tìm lỗi]** Mai lập luận: "Nếu ca sĩ đó nổi tiếng thì buổi biểu diễn kín chỗ. Buổi biểu diễn đã kín chỗ. Vậy ca sĩ đó chắc chắn nổi tiếng."
(a) Lập luận sai ở đâu (gọi đúng tên lỗi trong bài học)? lập luận này sai vì đây là lỗi lập luận hậu quả
(b) Với câu điều kiện đó, nếu biết buổi biểu diễn **còn trống chỗ**, ta kết luận được gì? nếu còn trống chỗ thì buổi biểu diễn của ca sĩ không nổi tiếng

**G5. [mới]** Hôm nay là thứ Tư. 45 ngày sau là thứ mấy? Ghi phép tính (phần dư): 45 / 7 lấy dư là 3, 4 + 3 = 7 vậy sau 45 ngày là thứ 7

**G6. [đã gặp]** `3, 8, 15, 24, ?` (số tiếp theo, và quy luật theo vị trí `n` viết thành công thức, thử trên cả bốn số đã cho) số tiếp theo là 35 vì các số theo quy luật n x (n + 2) với n là vị trí 3 = 1 x (1 + 2), 8 = 2 x (2 + 2)

**G7. [đã gặp]** `7, 10, 16, 28, ?` (số tiếp theo, và quy luật viết thành câu hoặc công thức, thử trên cả các hiệu đã có) số tiếp theo là 40, hiệu của hiệu các số liên tiếp là 3. 40 - 28 = 12, 28 - 16  = 9, 12 - 9 = 3

---

# PHẦN 2 (15 phút): ôn SQL, file chạy kèm: `sql_9_11.py`

## Nhắc (có ví dụ làm sẵn)
- **`COUNT(cột)` bỏ qua NULL; `COUNT(*)` đếm mọi hàng.** Ví dụ: cột `m = [4, NULL, 6]`: `COUNT(*) = 3`, `COUNT(m) = 2`, `AVG(m) = 5`.
- **`RANK` và `DENSE_RANK`:** khi có điểm bằng nhau, `RANK` nhảy cách (bỏ số), `DENSE_RANK` không nhảy. Ví dụ: điểm `10, 10, 8` (sắp giảm dần): `RANK = 1, 1, 3`; `DENSE_RANK = 1, 1, 2`.
- **NULL là "không xác định":** `WHERE x = NULL` không bao giờ đúng, nên **không trả về hàng nào**. Phải viết `WHERE x IS NULL`. Ví dụ: `[4, NULL, 6]` với `WHERE m IS NULL` trả đúng hàng chứa NULL.

Bảng `scores` (điểm thi; `score` NULL là chưa có điểm):

| id | student | subject | score |
|---|---|---|---|
| 1 | An | Toán | 9 |
| 2 | Bình | Toán | 9 |
| 3 | Chi | Toán | 7 |
| 4 | Dũng | Toán | 8 |
| 5 | An | Lý | 6 |
| 6 | Bình | Lý | NULL |
| 7 | Chi | Lý | 6 |
| 8 | Dũng | Lý | 10 |

**S1. [đã gặp]** `SELECT student, AVG(score) FROM scores GROUP BY student HAVING COUNT(score) >= 2;` Dự đoán các hàng kết quả (kèm cách tính cho **từng** học sinh, kể cả em bị loại):

| student | AVG(score) |
| ------- | ---------- |
| An      | 7.5        |
| Chi     | 6.5        |
| Dũng    | 9.0        |
An: (9 + 6) / 2 = 7.5
Chi: (7 + 6) / 2 = 6.5
Dũng: (8 + 10) / 2 = 9
Bình: 9 / 1 = 9

**S2. [mới]** `SELECT id, RANK() OVER (PARTITION BY subject ORDER BY score DESC) AS r, DENSE_RANK() OVER (PARTITION BY subject ORDER BY score DESC) AS d FROM scores WHERE subject = 'Toán' ORDER BY id;` Dự đoán `r` và `d` cho `id` 1, 2, 3, 4:

| id  | r   | d   |
| --- | --- | --- |
| 1   | 1   | 1   |
| 2   | 1   | 1   |
| 3   | 4   | 3   |
| 4   | 3   | 2   |


**S3. [mới, tìm lỗi]** `SELECT * FROM scores WHERE score = NULL;` không trả về hàng nào dù bảng có điểm chưa nhập.
(a) Vì sao? vì WHERE score = NULL luôn ra kết quả không xác định mà WHERE chỉ nhận kết quả đúng vậy nên SELECT không trả về gì
(b) Viết lại đúng và ghi `id` của hàng sẽ ra: SELECT id FROM scores WHERE score IS NULL
