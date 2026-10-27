---
title: Bài kiểm tra cổng Tuần 1
tags: [kiem-tra, tuan-1, cong]
---

# Bài kiểm tra cổng Tuần 1 (20 câu, 100 điểm)

Đạt từ **70 điểm** thì qua Tuần 1 sang Tuần 2 (SQL và xác suất). Dưới 70 thì ở lại Tuần 1, ôn đúng phần sai rồi làm bài kiểm tra mới.

## Luật làm bài (khác buổi học)
- **Không tra cứu, không hỏi AI, không chạy code** trong lúc làm. Giấy nháp được dùng.
- **Trước khi làm:** đọc lại phần **Sổ lỗi** trong file lộ trình, 15 phút. Đây là bước 0, không tính điểm.
- Thời gian: tối đa **90 phút**, bấm giờ. Hết giờ thì dừng.
- Mỗi câu 5 điểm. Câu không biết thì ghi "không biết".
- Câu viết code có ô **Điều kiện đề**: chép nguyên văn các điều kiện (hình thức, khoảng giá trị, đếm từ 0 hay 1, nhãn hay giá trị, một con số hay một nhóm, tăng hay giảm dần). Câu nào chép sai điều kiện thì chỉ tính nửa điểm.
- Câu viết code có ô **Kết quả mong đợi**: tính tay.
- Câu có chữ "giải thích": đúng kết quả nhưng không giải thích được thì chỉ tính nửa điểm.
- Làm xong gửi nguyên file này cho Claude chấm.

---

## A. Python (4 câu)

**A1.** Dự đoán đầu ra và giải thích:
```python
a = [1, 2]
b = a
b += [3]
c = a
c = c + [4]
print(a, c)
```
Dự đoán: [1,2,3][1,2,3,4]
Giải thích: với b = a sau đó b += [3] thì b và a trỏ cùng đến 1 list đồng nghĩa list đó thêm 3 vào sau thì a cũng có thêm 3 vào sau. Với c =a sau đó c = c + [4] thì c và a không cùng trỏ đến 1 list, c trỏ đến list khác với list a nhưng có giá trị ban đầu bằng a sau đó thêm 4 vào sau

**A2.** Viết **một dòng** list comprehension: bình phương của các số từ 1 đến 30 **chia hết cho 3**.
Điều kiện đề: viết một dòng list comprehension tính bình phương của các số từ 1 đến 30 chi hết cho 3
```python
# Trả lời:
a = [i * i for i in range(1,31) if i % 3 == 0]
```

**A3.** Dự đoán đầu ra và giải thích:
```python
def f(lst):
    lst = lst + [9]

x = [1]
f(x)
print(x)
```
Dự đoán: [1]
Giải thích: hàm f sử dung lst = lst + [9] đồng nghĩa là tạo list mới thay vì trỏ cùng 1 list và thêm 9 vào sau đồng thời hàm không ó trả về nên bến x truyền vào hàm f không đổi giá trị x

**A4.** Giải thích bằng 2 câu: "bất biến" và "khả biến" nghĩa là gì? Mỗi loại cho một ví dụ kiểu dữ liệu.
Trả lời: chỗ này tôi nhớ là int là bất biến tức ví dụ a = 1, b = a, b += 1 thì khi in kết quả a = 1 còn b = 2 nhưng list thì khả biết tức ví dụ a = [1], b = a, b += [2] thì khi in ra a = b = [1,2] tôi chưa tìm hiểu rõ nên khó diễn tả chuẩn xác 

---

## B. NumPy (5 câu)

**B1.** Dự đoán đầu ra và giải thích:
```python
a = np.array([1, 2, 3, 4])
s = a[1:3]
s[0] = 50
print(a)
```
Dự đoán: [1,50,3,4]
Giải thích: vì s chỉ là một lát cắt của a nhưng vẫn trỏ đến dùng 1 mảng numpy với a nên đồng nghĩa vị trí số 0 trong s cùng vị trí số 1 trong a hay trong mảng numpy mà a trỏ đến nên thay đổi giá trị vị trí 0 trong s thì thay đổi giá trị vị trí 1 trong mảng numpy mà a trỏ đến

**B2.** Dự đoán đầu ra và giải thích:
```python
a = np.array([1, 2, 3])
b = a
b += 5
c = a * 2
print(a, c)
```
Dự đoán: [1,2,3,5] [2,4,6,10]
Giải thích: với b = a sau đó b += [5] thì b và a trỏ cùng đến 1 mảng numpy đồng nghĩa mảng numpy đó thêm 5 vào sau thì a cũng có thêm 5 vào sau. Với c = a * 2  thì c và a không cùng trỏ đến 1 mảng numpy, c trỏ đến mảng numpy khác với mảng numpy mà  a trỏ tới nhưng có giá trị ban đầu bằng a sau đó nhân tất cả phần từ với 2

**B3.** NumPy, **không vòng lặp, không comprehension**: lập phương các số **lẻ** từ 1 đến 8.
Điều kiện đề: dùng numpy không dùng vòng lặp, không dùng comprehension tính lập phương các số lẻ từ 1 đến 8
Kết quả mong đợi (tính tay): [1,27, 125, 343]
```python
# Trả lời:
a = np.arange(1,9,2) ** 3
```

**B4.** Cho `m = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])`. Tính trung bình **từng hàng**.
Điều kiện đề: tình trung bình từng hàng
Kết quả mong đợi (tính tay): [2,5,8]
```python
# Trả lời:
mean = m.mean(axis=0)
```

**B5.** Giải thích bằng 2 câu vì sao tính theo vectơ trong NumPy thường nhanh hơn for loop của Python.
Trả lời: vì for loop của python là thông dịch vừa dịch vừa tính còn verctor trong numpy sử dụng C để biên dịch tưc dịch toàn bộ rồi mới tính nên nhanh hơn

---

## C. pandas (8 câu)

Dữ liệu dùng cho phần C và D:
```python
df = pd.DataFrame({
    "id":     [1, 2, 3, 4, 5, 6],
    "dept":   ["A", "B", "A", "C", "B", "A"],
    "salary": [30, 45, 50, 25, 60, 40]
})
depts = pd.DataFrame({
    "dept": ["A", "B"],
    "boss": ["An", "Bình"]
})
```

**C1.** `print(df.loc[1:3, "salary"])` in ra bao nhiêu phần tử, là những giá trị nào? Giải thích.
Dự đoán: 

|     | salary |
| --- | ------ |
| 1   | 45     |
| 2   | 50     |
| 3   | 25     |

Giải thích: vì loc sẽ lấy các giá trị từ hàng 1 2 3 trong cột salary

**C2.** Lấy giá trị `salary` của dòng ở **vị trí thứ 4 (đếm từ 1)** bằng `iloc`.
Điều kiện đề: lấy giá trị salary của dòng ở vị trí 4 (đếm từ 1) bằng iloc
Kết quả mong đợi (tính tay): 25
```python
# Trả lời:
df.iloc[3,2]
```

**C3.** Lấy các dòng có `dept == "A"` **và** `salary > 35`.
Điều kiện đề: lấy các dòng có dept == "A" và salary > 35
Kết quả mong đợi (liệt kê **nhãn hàng**, tính tay):

|     | id  | dept | salary |
| --- | --- | ---- | ------ |
| 3   | 4   | A    | 50     |
| 5   | 6   | A    | 40     |

```python
# Trả lời:
df[(df["dept"] == "A") & (df["salary"] > 35)]
```

**C4.** Tổng `salary` theo từng `dept`, sắp xếp **giảm dần**.
Điều kiện đề: tỉnh tổng salary theo từng dept sau đó sắp xếp giảm dần
Kết quả mong đợi (tính tay):

|     | dept | salary |
| --- | ---- | ------ |
| 0   | A    | 120    |
| 1   | B    | 105    |
| 2   | C    | 25     |

```python
# Trả lời:
df.groupby("dept")["salary"].sum().sort_values("salary", ascending=False)
```

**C5.** Tính **một con số**: trung bình `salary` của các dòng có `dept == "B"`.
Điều kiện đề: tính ra một con số trung bình salary của các dòng có dept == "B"
Kết quả mong đợi (tính tay): 52.5
```python
# Trả lời:
df[df["dept"] == "B"]["salary"].mean()
```

**C6.** Dự đoán **nhãn hàng** của `df.sort_values("salary", ascending=False).head(2)`.
Dự đoán:

|     | id  | dept | salary |
| --- | --- | ---- | ------ |
| 4   | 5   | B    | 60     |
| 2   | 3   | A    | 50     |

Giải thích (nêu rõ đây là tăng dần hay giảm dần): đây là sắp xếp giảm dần và chỉ cần hiển thị đúng 2 giá trị đầu của dataframe

**C7.** Gán `salary = 0` cho các dòng có `dept == "C"` **đúng cách**. Rồi giải thích vì sao `df[df["dept"] == "C"]["salary"] = 0` không làm đổi `df`.
Điều kiện đề: gắn salary = 0 cho các dòng có dept == "C" đúng cách
```python
# Trả lời:
df.loc[df["dept"] == "C", "salary"] = 0
```
Giải thích: vì df[df["dept"] == "C"] chỉ là một bản sao tạm sau đó gắn trong bản sao tạm đó những chỗ salary gắn băng 0 nên không làm thay đổi df

**C8.** Dự đoán số hàng của `pd.merge(df, depts, on="dept", how="left")` và của `how="inner"`. Giá trị `boss` của dòng `dept == "C"` ở kết quả `left` là gì?
Dự đoán: left có 6 hàng con inner có 5 hàng, giá trị boss của dòng dept == "C" có kết quả left là NaN
Giải thích: left là giữ lại tất cả các giá trị dataframe bên trái tức dataframe làm tham số đầu tiên truyền vào merge đồng nghĩa toàn bộ giá trị trong df được giữ nguyên khi merge. Do đó dept == "C" thì giá trị boss là NaN vì dataframe depts không có giá trị boss cho dept == "C" mà merge left là giữ toàn bộ giá trị df nên không có trị thì mặc định điền NaN. Còn left thì giữ lại tất cả các trị trị dataframe mà cả 2 dataframe đều có dự theo khóa là dept

---

## D. Tổng hợp (3 câu, không nhìn tài liệu)

**D1.** `groupby` gồm 3 bước nào? Mỗi bước một câu, dùng ví dụ `df.groupby("dept")["salary"].sum()`.
Bước 1: nhóm giá trị theo dept lại
Bước 2: tính tổng salary dự theo từng nhóm dept
Bước 3: gom các kết quả lại

**D2.** Khi nào dùng `nlargest`, khi nào dùng `sort_values`?
Trả lời: khi muốn lấy ra một vài hàng có giá trị đặc trưng lớn nhất thì dùng nlargest, con muốn sắp xếp các hàng dựa theo giá trị đặc trưng thì dùng sort_values

**D3.** `loc` và `iloc` khác nhau ở đâu? Nêu cả cách xác định vị trí (hàng và cột) và khác biệt khi cắt lát.
Trả lời: loc thì xác định vị trí cột và hàng theo tên của cột và hàng đó và cắt lát thì từ hàng đầu đến hàng cuối ví dụ [1,3] thì từ hàng 1 đến hàng 3, còn iloc thì xác định vị trí cột và hàng theo thứ tụ của cột và hàng đó ví dụ từ phần C id = 1 thì cột 0 (cột id) hàng 0, còn cắt lát từ hàng đầu đến hàng trước hàng cuối ví dụ [1,3] thì từ hàng 1 đến hàng 2

---

## Ghi nhật ký
- Thời gian làm thực tế: 8:30 - 9:35 (1h5)
- Câu bạn thấy mình sai nhiều khả năng nhất: câu A4 vì giải thích khá khó
- Điều chưa hiểu: câu A4 cần làm rõ thêm
