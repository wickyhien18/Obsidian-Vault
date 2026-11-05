---
title: Thứ Ba 3/11 - ĐSTT 2 (định thức, nghịch đảo, giải hệ) và ôn Python/pandas
tags: [dai-so-tuyen-tinh, python, tuan-3]
---

# 3/11 (1 giờ): ĐSTT 2 (45 phút) + ôn Python/pandas (15 phút)

**Cách làm:** tính tay hoặc dự đoán trước, kiểm bằng code sau, dán nguyên văn kết quả chạy. Làm tròn bình thường, lệch ≤ 0,001 vẫn đúng. Đề mơ hồ thì ghi "đề mơ hồ vì ..." vào ô trả lời. Mọi câu chỉ dùng nội dung trong bài học bên dưới. **Nhật ký điền đủ ba ô bằng câu hoàn chỉnh**, ghi giờ thật.

**Nhãn:** mỗi câu gắn **[đã gặp]** (cùng dạng bài trước) hoặc **[mới]** (dạng chưa làm). Mình chấm và báo điểm riêng hai loại.

**Chủ đề đề bài (theo góp ý 2/11):** từ file này đề bài đổi chủ thể liên tục (quán trà, rạp phim, trạm xe đạp, kho hàng, ...), không lặp mãi đồng xu và bi màu.

**Thói quen mới (rút từ bài 2/11):** khi ghi "quy luật" hoặc "lý do", viết thành **công thức hoặc câu có thể kiểm**, rồi **thử trên số hạng/ví dụ đầu tiên**. Đáp số đúng mà lý do không đúng thì chỉ được nửa điểm.

---

# PHẦN 1 (45 phút): Đại số tuyến tính 2

## Bài học (mỗi ý có ví dụ làm sẵn)

**1. Ma trận đơn vị** `I = [[1,0],[0,1]]`. Nhân với `I` không đổi gì: `A @ I = A`.

**2. Định thức 2×2:** `det [[a,b],[c,d]] = a·d − b·c`.
Ví dụ làm sẵn: `[[5,2],[3,1]]` có `det = 5·1 − 2·3 = −1`. (NumPy: `np.linalg.det(M)` ra `−1.0000000000000009`, là sai số làm tròn của máy, đọc là −1.)

**3. Ma trận nghịch đảo 2×2:** chỉ có khi `det ≠ 0`. Công thức: `A⁻¹ = (1/det) · [[d, −b], [−c, a]]` (đổi chỗ `a` và `d`, đổi dấu `b` và `c`).
Ví dụ làm sẵn: `A = [[2,1],[5,3]]`: `det = 2·3 − 1·5 = 1`, nên `A⁻¹ = [[3,−1],[−5,2]]`.
**Kiểm bằng nhân ngược** `A @ A⁻¹` phải ra `I`:
`[[2·3 + 1·(−5), 2·(−1) + 1·2], [5·3 + 3·(−5), 5·(−1) + 3·2]] = [[1,0],[0,1]]`. Đúng.

**4. Giải hệ phương trình bằng nghịch đảo:** viết hệ thành `A @ x = b`, rồi `x = A⁻¹ @ b`.
Ví dụ làm sẵn (quán trà chiều): gọi `t` là giá một ly trà, `k` là giá một cái bánh (đơn vị chục nghìn). 2 ly trà + 1 bánh = 7; 5 ly trà + 3 bánh = 18.
`A = [[2,1],[5,3]]`, `x = [t, k]`, `b = [7, 18]`. Dùng `A⁻¹` ở trên:
`x = [[3,−1],[−5,2]] @ [7,18] = [3·7 − 18, −5·7 + 2·18] = [3, 1]`. Vậy trà 3, bánh 1.
**Kiểm lại vào đề:** 2·3 + 1·1 = 7 và 5·3 + 3·1 = 18. Đúng.
NumPy: `np.linalg.solve(A, b)` ra `[3. 1.]`.

**5. Ma trận suy biến:** `det = 0` thì **không có nghịch đảo**. Dấu hiệu: một hàng là bội của hàng kia.
Ví dụ làm sẵn: `[[2,6],[1,3]]`: hàng 1 = 2 × hàng 2, `det = 2·3 − 6·1 = 0`. `np.linalg.inv` báo `LinAlgError: Singular matrix`.

**6. Nhân ma trận không giao hoán** (đã học): nói chung `A @ B ≠ B @ A`. Nhắc lại cách nhân: phần tử hàng i cột j của tích = (hàng i của ma trận trái) · (cột j của ma trận phải).

## Bài tập

**M1. [đã gặp]** Quán ăn sáng ghi số suất bán trong 2 ngày. Mỗi hàng là một ngày, cột 1 là bánh mì, cột 2 là cà phê:
`X = [[3,2],[1,4]]`. Giá một suất: bánh mì 2, cà phê 5 (chục nghìn), `w = [2,5]`.
(a) Tính tay `X @ w`. (b) Kết quả có `shape` gì, và mỗi phần tử có nghĩa gì?
Dự đoán (a): [16,12]
Dự đoán (b): (2,) mỗi phần từ là kết quả của tích vô hướng giữ từng hằng của X với các giá trị trong cột của w

**M2. [mới]** Tính định thức ba ma trận: `P = [[4,3],[2,5]]`, `Q = [[6,4],[3,2]]`, `R = [[1,7],[0,2]]`.
(a) det P = 14
(b) det Q = 0
(c) det R = 2
(d) Ma trận nào suy biến? Một câu lý do **theo hàng** (hàng nào bằng mấy lần hàng nào): Ma trận Q suy biến vì det bằng 0 tức hàng [3,2] bằng 2 lần hàng [6,4] của ma trận Q

**M3. [mới]** Trạm sạc có ma trận `B = [[4,3],[1,1]]`.
(a) det B = 1
(b) `B⁻¹` (viết bốn phần tử) = [[1,-3][-1,4]]
(c) Kiểm ngược: tính `B @ B⁻¹`, ghi bốn phần tử: [[4 * 1 + 3 * (-1),  4 * (-3) + 3 * 4] [1 * 1 + 1 * (-1), 1 * (-3) + 1 * 4]] = [[1,0][0,1]]

**M4. [mới]** Rạp phim: 3 vé người lớn + 2 vé trẻ em giá 110 (nghìn đồng); 1 vé người lớn + 1 vé trẻ em giá 40 (nghìn đồng). Gọi `x` là giá vé người lớn, `y` là giá vé trẻ em.
(a) Viết `A` và `b` (để `A @ [x, y] = b`): A = [[3,2][1,1]], B = [110,40]
(b) det A = 1
(c) `A⁻¹` = [[1,-2],[-1,3]]
(d) `x`, `y` (tính `A⁻¹ @ b`): 30, 10
(e) Kiểm lại vào cả hai dữ kiện của đề (ghi phép tính):  [1 * 110 + (-2) * 40,  (-1) * 110 + 3 * 40] = [30,10]

**M5. [mới, tìm lỗi]** Hùng nói: "Ma trận `[[2,6],[1,3]]` có nghịch đảo là `(1/det)·[[3,−6],[−1,2]]`."
(a) Hùng sai ở đâu? (nêu **con số** cụ thể làm bằng chứng) hùng sai vì det của ma trận là 0 tức ma trận không thể có nghịch đảo
(b) Nếu chạy `np.linalg.inv(np.array([[2,6],[1,3]]))` thì Python in ra gì (tên lỗi)? bị lỗi Singular Matrix

**M6. [đã gặp]** Kho hàng ghi hai phép biến đổi: `P = [[1,2],[0,1]]`, `Q = [[1,0],[3,1]]`.
(a) Tính tay `P @ Q`: [[7,2],[3,1]]
(b) Tính tay `Q @ P`: [[1,2],[3,7]]
(c) `P @ Q` có bằng `Q @ P` không? Một câu kết luận: P @ Q không bằng Q @ P vì P @ Q là tính tích vô hướng lần lượt hàng P với cột Q trong khi đó Q @ P là tính tích vô hướng lần lượt của hàng Q với cột P

**M7. [mới, dự đoán code]** Dự đoán đầu ra trước khi chạy:
```python
import numpy as np
A = np.array([[4, 3], [1, 1]])
b = np.array([10, 3])
print(np.linalg.solve(A, b))
```
Dự đoán: [1,2]
Kiểm lại bằng tay: thay kết quả vào `4x + 3y` và `x + y`, ghi phép tính:
4 * 1 + 3 * 2 = 10, 1 + 2 = 3

---

# PHẦN 2 (15 phút): ôn Python/pandas (đọc code, dự đoán, tìm lỗi)

## Nhắc (có ví dụ làm sẵn)
- **`groupby`:** nhóm các hàng theo một cột rồi tính. Ví dụ: bảng `quán = [A, B, A]`, `bán = [10, 4, 20]`. `df.groupby("quán")["bán"].mean()` ra A: 15, B: 4.
- **Điều kiện kép trong pandas:** dùng `&` và **bọc từng điều kiện trong ngoặc**: `df[(df["x"] > 1) & (df["y"] == "a")]`. Thiếu ngoặc thì `&` chạy trước `>`, gây lỗi `TypeError`.
- **Cắt lát NumPy là view, cắt lát list là bản sao.** Ví dụ làm sẵn: `a = np.array([1,2,3,4]); v = a[::2]; v[0] = 9; print(a)` in `[9 2 3 4]` (gốc đổi). Cùng thao tác với list `l = [1,2,3,4]; w = l[::2]; w[0] = 9; print(l)` in `[1, 2, 3, 4]` (gốc không đổi).
- **Dict:** `d.get(khóa, mặc_định)` trả giá trị, hoặc mặc định nếu khóa không có. `khóa in d` kiểm tra **khóa**, không kiểm giá trị. Ví dụ: `d = {"a": 1}`: `d.get("b", 0)` là `0`, `"a" in d` là `True`, `1 in d` là `False`.

Bảng dùng cho P1 và P2 (trạm xe đạp công cộng, số lượt thuê trong 6 ngày):
```python
import pandas as pd
df = pd.DataFrame({
    "station": ["Cầu Giấy", "Hà Đông", "Cầu Giấy", "Long Biên", "Hà Đông", "Cầu Giấy"],
    "rides":   [12, 30, 8, 15, 10, 20]
})
```

**P1. [đã gặp]** Dự đoán đầu ra của `print(df.groupby("station")["rides"].mean().round(2))`. Ghi trung bình của từng trạm (kèm cách tính): 

| station   |       |
| --------- | ----- |
| Cầu Giấy  | 13.33 |
| Hà Đông   | 20.00 |
| Long Biên | 15.00 |
Trung bình của trạm Cầu Giấy = (12 + 8 + 20) / 3 = 13.33
Trung bình của trạm Hà Đông = (30 + 10) / 2 = 20
Trung bình của trạm Long Biên = 15/1 = 15

**P2. [mới, tìm lỗi]** Đoạn `df[df["rides"] > 10 & df["station"] == "Hà Đông"]` bị lỗi.
(a) Vì sao lỗi (một câu, nhắc thứ tự chạy của `&`)? Bị lỗi TypeError vì bị thiếu ngoặc đồng nghĩa & chạy trước >
(b) Viết lại đúng: df[(df["rides"] > 10) & (df["station"] == "Hà Đông")]
(c) Với bảng trên, bản đúng trả về những hàng nào (ghi `station` và `rides`)? 
   station  rides
1  Hà Đông     30

**P3. [đã gặp]** Dự đoán đầu ra từng đoạn:
```python
import numpy as np
arr = np.array([5, 6, 7, 8]); v = arr[::2]; v[0] = 0; print(arr)
lst = [5, 6, 7, 8];            w = lst[::2]; w[0] = 0; print(lst)
```
Dòng in thứ nhất: [0, 6, 7, 8]
Dòng in thứ hai: [5, 6, 7, 8]
Một câu giải thích vì sao hai dòng khác nhau: vì mảng numpy slicing là 1 view tức chỉ là tạo ra 1 cửa sổ trỏ tới cũng là mảng numpy đó thay vì tạo mới còn list khi slicing là tạo 1 list mới 

**P4. [mới]** Kho hàng: `stock = {"bút": 10, "vở": 5}`. Dự đoán:
(a) `stock.get("tẩy", 0)` = 0
(b) `"vở" in stock` = true
(c) `10 in stock` = false

---

## Ghi nhật ký (điền đủ ba ô bằng câu hoàn chỉnh)
- Giờ bắt đầu và kết thúc thực tế (ghi giờ thật): 8h-8h50
- Câu bạn nghĩ mình sai nhiều khả năng nhất, và vì sao (nêu câu cụ thể): tôi không nghĩ câu nào sai cả
- Những câu bạn đã kiểm lại và bằng cách nào (và câu nào chưa kiểm): tôi đã kiểm tra các câu M bằng bấm máy tính còn các câu P đều đã chạy code
