# Bài chẩn đoán Tuần 0 (20 câu, 100 điểm)

## Luật làm bài (nghiêm túc, không có ngoại lệ)
- Không tra Google, không hỏi AI, không chạy thử code trong lúc làm. Được dùng giấy nháp và máy tính bỏ túi.
- Chia 3 buổi, đặt hẹn giờ: **Buổi 1** (phần A + B, 50 phút), **Buổi 2** (phần C + D, 60 phút), **Buổi 3** (phần E, 30 phút).
- Mỗi câu 5 điểm. Câu nào bí thì ghi "không biết", không đoán mò rồi tự chấm đúng.
- Câu có chữ "giải thích": đáp án đúng nhưng không giải thích được thì chỉ tính nửa điểm.
- Làm xong gửi toàn bộ đáp án cho Claude chấm. Không tự chấm.

---

## A. Python và pandas (4 câu)

**A1.** Đoạn code sau in ra gì? Giải thích vì sao.
```python
a = [1, 2, 3]
b = a
b.append(4)
print(a, len(a))
```

Trả lời: Đoạn code này in ra [1,2,3] và 3 vì cho dù mảng b có thêm phần từ nữa thì chỉ in mảng a thì chỉ in ra thành phần trong mảng a và kích thước mảng a
**A2.** Viết một list comprehension tạo danh sách bình phương của các số chẵn từ 1 đến 20.
Trả lời:
``` Python
a = []
for i in range(1,21):
	if i % 2 == 0:
		a.append(i * i)

print(a)
```

**A3.** Cho DataFrame `df` có cột `user_id`, `date`, `amount`. Viết code pandas lấy ra 3 user có tổng `amount` lớn nhất.

**A4.** Phân biệt `df.loc` và `df.iloc`. Giải thích vì sao dòng sau thường gây cảnh báo `SettingWithCopyWarning` và nên viết lại thế nào:
```python
df[df["amount"] > 100]["status"] = "big"
```

Trả lời A3, A4: tôi chưa có nền tảng DataFrame này

---

## B. SQL (3 câu)

Hai bảng: `customers(customer_id, name, city)` và `orders(order_id, customer_id, order_date, amount)`.

**B1.** Viết truy vấn tính tổng `amount` theo từng `city`, chỉ giữ các thành phố có tổng lớn hơn 1000, sắp xếp giảm dần.

**B2.** Viết truy vấn liệt kê các khách hàng **chưa từng** có đơn hàng nào.

**B3.** Viết truy vấn lấy đơn hàng **mới nhất** của mỗi khách hàng (dùng window function hoặc cách khác, nói rõ).

Trả lời B1, B2, B3: tôi sẽ cần học lại phần này vì nếu không dùng gg thì tôi không kiểm chứng được

---

## C. Xác suất và thống kê (5 câu)

**C1.** Hai biến cố độc lập: P(A) = 0.3, P(B) = 0.5. Tính P(A ∪ B).
Trả lời: P(A ∪ B) = P(A) + P(B) = 0.3 + 0.5 = 0.8

**C2.** Một bệnh có tỷ lệ mắc 1% trong dân số. Xét nghiệm có độ nhạy (sensitivity) 95% và độ đặc hiệu (specificity) 90%. Một người xét nghiệm dương tính. Xác suất người đó thực sự mắc bệnh là bao nhiêu? Giải thích kết quả có làm bạn bất ngờ không.
Trả lời:2 con số này không rõ nên tôi bỏ trống câu này

**C3.** Tung một con xúc xắc công bằng một lần. Tính kỳ vọng và phương sai của số chấm.
Trả lời: haiz tôi không nhờ không thức

**C4.** Giải thích p-value bằng một câu, rồi nêu một cách hiểu sai phổ biến về p-value.
Trả lời: tôi không biết p-value

**C5.** Độ lệch chuẩn của tổng thể là 10, cỡ mẫu n = 100. Tính sai số chuẩn (standard error) của trung bình mẫu, và nói điều gì xảy ra với nó khi n tăng gấp 4.
Trả lời: tôi không biết

---

## D. Đại số tuyến tính (4 câu)

**D1.** Cho A = [[1, 2], [3, 4]] và B = [[0, 1], [1, 0]]. Tính AB.
Trả lời: AB = [[2,1][4,3]]

**D2.** Tính định thức của A ở câu D1. Giải thích định thức bằng 0 nói lên điều gì về ma trận.
Trả lời: tôi không biết định thức

**D3.** Tìm các trị riêng (eigenvalue) của M = [[4, 1], [2, 3]]. Giải thích bằng lời trị riêng và vector riêng nghĩa là gì.
Trả lời tôi không biết giá trị riêng

**D4.** Cho u = (1, 2, 2) và v = (2, 0, 1). Tính độ dài của u và cosine similarity giữa u và v.
Trả lời: Độ dài u = 3
Cosin Similarity = 0.597

---

## E. Machine Learning (4 câu)

**E1.** Mô hình có độ chính xác 99% trên tập train nhưng 70% trên tập validation. Chuyện gì đang xảy ra? Nêu 3 cách khắc phục khác nhau.
Trả lời: đây là hiện tượng overfitting tôi chưa biết cách khắc phục

**E2.** Dữ liệu chỉ có 1% mẫu dương. Giải thích vì sao accuracy gây hiểu lầm. Với TP = 40, FP = 10, FN = 60, tính precision và recall.
Trả lời: tôi không biết

**E3.** Giải thích gradient descent bằng lời của bạn. Điều gì xảy ra khi learning rate quá lớn, và khi quá nhỏ?
Trả lời: gradient descent là việc chỉnh các trọng số sao cho hàm loss có giá trị nhỏ nhất với learning rate quá lớn dễ khiến hàm loss không hội tụ đến điểm nhỏ nhất mà bắt đầu phần kỳ, còn quá nhỏ thì sẽ rất mất thời gian để hàm loss hội tụ điểm nhỏ nhất

**E4.** Giải thích data leakage. Cho ví dụ cụ thể về việc chuẩn hóa (scaling) dữ liệu gây rò rỉ, và cách tránh trong thực tế.
Trả lời: nếu tôi hiểu đúng thì rò rỉ dữ liệu là việc kết quả tập test bị rò rỉ vào tập train. tôi cũng chưa hiểu rõ lắm chỉ hiểu đến đây thôi.

---

## Thang điểm sau khi chấm
- Dưới 50: nền tảng yếu, Tuần 1-2 dành toàn bộ cho ôn lại phần bị điểm thấp, chưa nên làm gì phức tạp hơn.
- 50 đến 70: đúng hướng nhưng còn hổng, ôn vá các phần dưới 60% rồi làm lại bài tương tự.
- Trên 70: ổn nền tảng, chuyển sang làm dự án sớm hơn.
