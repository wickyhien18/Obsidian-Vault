---
title: Buổi học Ngày 2 - Tuần 1
date: 2026-10-13
tags: [bai-tap, numpy, tuan-1]
---

# Buổi học Ngày 2: Sửa lỗi Ngày 1 và NumPy

**Khung giờ 21:00 - 23:00**
- 21:00 - 21:15: Phần 0, sửa lỗi Ngày 1
- 21:15 - 21:55: đọc bài học NumPy, chạy từng ví dụ trong Colab
- 21:55 - 22:40: bài tập A và B (dự đoán trước, chạy sau)
- 22:40 - 22:55: bài C, tự giải thích, không nhìn tài liệu
- 22:55 - 23:00: ghi nhật ký

> [!note] Luật buổi học
> **Dự đoán trước, viết ra, rồi mới chạy.** Sau khi chạy phải ghi rõ "dự đoán đúng" hoặc "dự đoán sai vì...".

---

## Phần 0: Sửa lỗi Ngày 1

**Ghi nhớ:** với list, `b += [4]` **sửa tại chỗ** (giống `b.extend([4])`), còn `c = c + [5]` **tạo list mới** rồi gán lại tên `c`.

**0.1.** Chạy lại đoạn sau. Dự đoán trước, rồi ghi kết quả thật:
```python
a = [1, 2]
b = a
b += [3]
print(a)

c = a
c = c + [4]
print(a, c)
```
Dự đoán:

Kết quả thật, và dự đoán đúng hay sai vì sao:

**0.2.** Viết list comprehension tạo các cặp `(i, j)` với `i < j`, trong đó `i` và `j` chạy từ 0 đến 3. Gợi ý: viết for loop lồng nhau trước.
```python
# for loop lồng nhau:
x = []
for i in range(0,4):
	for j in range(0,4):
		if i < j:
			x.append([i,j])

# comprehension một dòng:
x = [[i,j] ]

```

---

## Bài học: NumPy cơ bản

NumPy không có sẵn trong Python thuần. Trong Colab đã cài sẵn, bạn chỉ cần `import numpy as np`.

```python
import numpy as np
a = np.array([1, 2, 3])
print(a.shape, a.dtype)    # (3,) int64
```

**Tính theo vectơ (vectorization):** phép toán áp dụng cho cả mảng, không cần vòng lặp.
```python
a * 2          # [2 4 6]
a + a          # [2 4 6]
a ** 2         # [1 4 9]
```

**Lọc bằng mặt nạ boolean:**
```python
x = np.array([3, 8, 1, 9, 4])
x > 3          # [False  True False  True  True]
x[x > 3]       # [8 9 4]
```

**Mảng 2 chiều và `axis`:**
```python
m = np.array([[1, 2, 3],
              [4, 5, 6]])
m.shape            # (2, 3): 2 hàng, 3 cột
m.sum(axis=0)      # cộng dọc theo hàng, mỗi cột một kết quả: [5 7 9]
m.sum(axis=1)      # cộng dọc theo cột, mỗi hàng một kết quả: [ 6 15]
```

**Lưu ý quan trọng, trùng với lỗi Ngày 1:**
- `b = a` không sao chép mảng. `b = a.copy()` mới sao chép.
- `b += 10` **sửa tại chỗ**. `b = b + 10` **tạo mảng mới**.
- Cắt lát `a[1:4]` trả về một **view** (nhìn vào cùng dữ liệu), không phải bản sao. Sửa view thì mảng gốc đổi theo.

---

## Bài tập A: Dự đoán đầu ra

**A1.**
```python
a = np.array([1, 2, 3])
b = a * 2
print(a, b)
```
Dự đoán:

Dự đoán đúng hay sai, vì sao:

**A2.**
```python
a = np.array([1, 2, 3])
b = a
b += 10
print(a)
```
Dự đoán:

Dự đoán đúng hay sai, vì sao:

**A3.**
```python
a = np.array([1, 2, 3])
b = a
b = b + 10
print(a, b)
```
Dự đoán:

Dự đoán đúng hay sai, vì sao:

**A4.**
```python
a = np.arange(6)       # [0 1 2 3 4 5]
s = a[1:4]
s[0] = 99
print(a)
```
Dự đoán:

Dự đoán đúng hay sai, vì sao:

**A5.**
```python
m = np.array([[1, 2, 3], [4, 5, 6]])
print(m.shape, m.sum(axis=0), m.sum(axis=1))
```
Dự đoán:

Dự đoán đúng hay sai, vì sao:

---

## Bài tập B: Viết code (không dùng vòng lặp)

**B1.** Tạo mảng các số từ 0 đến 9 và tính bình phương từng phần tử.
```python
# Trả lời:

```

**B2.** Cho `x = np.array([3, 8, 1, 9, 4])`. Lấy ra các phần tử lớn hơn 3.
```python
# Trả lời:

```

**B3.** Cho `m` như trên (2 hàng, 3 cột). Tính trung bình từng cột.
```python
# Trả lời:

```

**B4.** Chuẩn hóa `x` ở B2 theo công thức `(x - trung bình) / độ lệch chuẩn`.
```python
# Trả lời:

```

**B5.** Viết lại bài Ngày 1 bằng NumPy, không dùng vòng lặp: bình phương các số chẵn từ 1 đến 20.
```python
# Trả lời:

```

---

## Bài tập C: Tự giải thích (không nhìn tài liệu, mỗi ý 2 câu)

**C1.** Vì sao tính theo vectơ thường nhanh hơn for loop?
Trả lời:

**C2.** Khác biệt giữa `b = a` và `b = a.copy()` với mảng NumPy là gì?
Trả lời:

**C3.** Với `m` có 2 hàng 3 cột, `axis=0` và `axis=1` khác nhau thế nào? Giải thích bằng lời của bạn.
Trả lời:

---

## Ghi nhật ký
- Thời gian học thực tế:
- Câu làm chậm nhất, vì sao:
- Điều chưa hiểu cần hỏi Claude:

> Làm xong gửi nguyên file này cho Claude chấm.
