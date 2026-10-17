---
title: Buổi học Ngày 1 - Tuần 1
date: 2026-10-11
tags: [bai-tap, python, tuan-1]
---

# Buổi học Ngày 1 (Chủ nhật 11/10): Tham chiếu và list comprehension

**Khung giờ 21:00 - 23:00**
- 21:00 - 21:50: đọc bài học, chạy từng ví dụ trong Python/Colab
- 21:50 - 22:40: làm bài tập (dự đoán trước, rồi chạy kiểm tra sau)
- 22:40 - 22:55: tự giải thích bằng lời, không nhìn tài liệu
- 22:55 - 23:00: ghi nhật ký vào file lộ trình

> [!note] Luật buổi học
> Đây là buổi học, **được chạy code để kiểm tra**. Quy tắc bắt buộc: **dự đoán trước, viết ra, rồi mới chạy**. Dự đoán sai thì ghi lại vì sao sai.

---

## Bài học 1: Tham chiếu và bản sao

Trong Python, gán `b = a` **không tạo bản sao** của list. Nó chỉ cho cùng một list thêm một cái tên.

```python
a = [1, 2, 3]
b = a          # a và b cùng trỏ vào MỘT list
b.append(4)
print(a)       # [1, 2, 3, 4]
```

Muốn bản sao độc lập:
```python
b = a.copy()   # hoặc a[:] hoặc list(a)
```

Với số, chuỗi thì khác, vì chúng **bất biến** (không sửa tại chỗ được). Phép `y += 1` tạo ra một số mới, không sửa số cũ.

```python
x = 5
y = x
y += 1
print(x, y)    # 5 6
```

## Bài học 2: List comprehension

Dạng tổng quát:
```python
[biểu_thức  for  phần_tử  in  tập_hợp  if  điều_kiện]
```
Phần `if` có thể bỏ. Ví dụ: vòng lặp
```python
kq = []
for i in range(1, 11):
    if i % 2 == 1:
        kq.append(i * 10)
```
viết thành comprehension là
```python
kq = [i * 10 for i in range(1, 11) if i % 2 == 1]
```
Cách đọc: "lấy `i * 10` cho mỗi `i` trong `range(1, 11)`, nếu `i` lẻ".

---

## Bài tập A: Dự đoán đầu ra (viết dự đoán trước khi chạy)

**A1.**
```python
a = [1, 2, 3]
b = a.copy()
b.append(4)
print(a, b)
```
Dự đoán: [1,2,3] [1,2,3,4]

Sau khi chạy, đúng/sai:  [1,2,3] [1,2,3,4]

**A2.**
```python
def add_item(lst):
    lst.append(99)

nums = [1, 2]
add_item(nums)
print(nums)
```
Dự đoán: [1,2,99]

Giải thích vì sao `nums` thay đổi dù hàm không `return` gì: nếu tôi nhớ không lầm thì do add_item là thay đổi cùng list chỉ là khác tên tức nums và lst đều trỏ đến cùng 1 list lên không cần return 

**A3.**
```python
x = 5
y = x
y += 1
print(x, y)
```
Dự đoán: 5,6

Giải thích vì sao kết quả khác với trường hợp list ở Bài học 1: số khác với chuỗi, còn lí do khác thì tôi chưa rõ

---

## Bài tập B: Viết list comprehension (mỗi bài đúng **một dòng**)

**B1.** Tạo danh sách lập phương của các số từ 1 đến 10.
```python
# Trả lời:
a = [a**3 for i in range(1,11)]
```

**B2.** Cho `words = ["python", "sql", "data", "ai"]`. Tạo danh sách các từ viết hoa.
```python
# Trả lời:
upper = [w.upper() for w in words]
```

**B3.** Tạo danh sách các số từ 1 đến 30 chia hết cho 3 **hoặc** cho 5.
```python
# Trả lời:
a = [i for i in range(1,31) if i % 3 == 0 or i % 5 == 0]
```

**B4.** Cho `words` như B2. Tạo danh sách độ dài của những từ có hơn 3 ký tự.
```python
# Trả lời:
a = [len(w) for w in words if len(w) > 3]
```

**B5.** Viết lại vòng lặp sau thành list comprehension.
```python
kq = []
for n in range(1, 21):
    if n % 2 == 0:
        kq.append(n * n)
```
```python
# Trả lời:
kq = [n * n for n in range(1,21) if n % 2 == 0]
```

---

## Bài tập C: Tự giải thích (15 phút, không nhìn tài liệu, mỗi ý 2 câu)

**C1.** Vì sao sửa `b` làm đổi `a` khi viết `b = a` với list?
Trả lời: tại vì đối với list thì b = a tức b và a đều trỏ cùng đến 1 list và sửa b đồng nghĩa sửa list cũng sửa cả a

**C2.** Cú pháp list comprehension gồm những phần nào, phần nào có thể bỏ?
Trả lời: gồm biểu thức, vòng lặp và điều kiện và phần điều kiện có thể bỏ để giữ lại biểu thức và vòng lặp

**C3.** Khi nào dùng for loop thông thường thay vì comprehension?
Trả lời: đối với các trường hợp loop lồng nhau hoặc trong 1 loop làm nhiều việc hơn là chỉ có mỗi điều kiện thì loop thông thường tốt hơn

---

## Ghi nhật ký (điền cuối buổi)
- Thời gian học thực tế: 2 tiếng
- Câu làm chậm nhất, vì sao: các câu phần C vì cần lục lại trí nhớ
- Điều chưa hiểu cần hỏi Claude: ngoài những cái trên ra thì cần biết cái gì để chuẩn bị cho machine learning chưa bàn đến các framework mới chỉ là những cái có sẵn trong python chưa import framework hay gì

> Làm xong gửi nguyên file này cho Claude chấm.
