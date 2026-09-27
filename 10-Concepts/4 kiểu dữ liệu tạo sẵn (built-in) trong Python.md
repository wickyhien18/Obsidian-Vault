---
created: 2026-09-27
Hub: "[[]]"
status: seed
---

> [!summary] Mẹo áp dụng "chuẩn" khi code
> -  Dữ liệu **cố định, không đổi** (cấu hình, tọa độ, hằng số) → **tuple**
> - Dữ liệu **thay đổi thường xuyên, có thứ tự** → **list**
> - Cần **loại bỏ trùng** hoặc **kiểm tra tồn tại nhanh** → **set**
> - Cần **tra cứu theo tên/khóa** (giống object trong JS, giống JSON) → **dict**
> - Khi làm việc với API (parse JSON) → gần như luôn là kết hợp **dict + list** (dict lồng trong list, hoặc ngược lại)

### Bảng so sánh nhanh

|Kiểu|Thứ tự|Trùng lặp|Thay đổi được|Truy cập|
|---|---|---|---|---|
|List|Có|Có|Có (mutable)|index|
|Tuple|Có|Có|Không (immutable)|index|
|Set|Không|Không|Có (mutable)|không index|
|Dict|Có (từ 3.7+)|Key: không / Value: có|Có (mutable)|key|
### 1. List (Danh sách)

**Đặc điểm:**

- Có thứ tự (ordered), truy cập bằng index
- **Mutable** (có thể thay đổi sau khi tạo)
- Cho phép phần tử trùng lặp
- Cú pháp: `[1, 2, 3]`

**Dùng khi:** cần một tập dữ liệu có thứ tự, thường xuyên thêm/xóa/sửa phần tử.

**Ví dụ thực tế:**

python

```python
todo_list = ["học Python", "làm đồ án", "ôn thi"]
todo_list.append("nộp báo cáo")   # thêm phần tử
todo_list[0] = "học Django"       # sửa phần tử
```

→ Danh sách sản phẩm trong giỏ hàng, danh sách sinh viên, hàng đợi xử lý...

### 2. Tuple (Bộ giá trị)

**Đặc điểm:**

- Có thứ tự, truy cập bằng index
- **Immutable** (không thể thay đổi sau khi tạo)
- Cho phép trùng lặp
- Cú pháp: `(1, 2, 3)`

**Dùng khi:** dữ liệu cố định, không muốn ai (kể cả chính mình) vô tình sửa nhầm. Tuple cũng nhanh hơn list một chút và có thể dùng làm key cho dictionary (list thì không).

**Ví dụ thực tế:**

python

```python
toa_do = (10.762622, 106.660172)   # (latitude, longitude) - cố định
RGB = (255, 0, 0)                  # màu đỏ
def chia(a, b):
    return a // b, a % b           # trả về nhiều giá trị dạng tuple
```

→ Tọa độ GPS, mã màu RGB, trả về nhiều giá trị từ hàm, key của dict phức hợp.

### 3. Set (Tập hợp)

**Đặc điểm:**

- **Không có thứ tự**, không truy cập bằng index
- **Mutable** nhưng các phần tử bên trong phải bất biến (immutable)
- **Không cho phép trùng lặp** — tự động loại bỏ giá trị trùng
- Hỗ trợ các phép toán tập hợp: hợp, giao, hiệu
- Cú pháp: `{1, 2, 3}`

**Dùng khi:** cần loại bỏ trùng lặp hoặc so sánh giữa các nhóm dữ liệu.

**Ví dụ thực tế:**

python

```python
sv_lop_a = {"An", "Bình", "Chi"}
sv_lop_b = {"Bình", "Dũng", "An"}

print(sv_lop_a & sv_lop_b)   # giao: học chung 2 lớp -> {'An', 'Bình'}
print(sv_lop_a | sv_lop_b)   # hợp: tổng tất cả sinh viên
print(sv_lop_a - sv_lop_b)   # hiệu: chỉ học lớp A -> {'Chi'}

emails = list(set(danh_sach_email))  # loại bỏ email trùng
```

→ Loại bỏ dữ liệu trùng, kiểm tra phần tử có tồn tại nhanh (O(1)), so sánh tập dữ liệu.

### 4. Dictionary (Từ điển)

**Đặc điểm:**

- Lưu theo cặp **key–value**
- **Mutable**
- Từ Python 3.7+ giữ thứ tự chèn (insertion order)
- Key phải là kiểu bất biến (string, số, tuple...), value thì tùy ý
- Cú pháp: `{"key": "value"}`

**Dùng khi:** cần tra cứu giá trị nhanh thông qua một "khóa" có ý nghĩa, thay vì index số.

**Ví dụ thực tế:**

python

```python
sinh_vien = {
    "mssv": "20211234",
    "ten": "Wicky",
    "diem": [8.5, 9.0, 7.5]
}
print(sinh_vien["ten"])   # tra cứu nhanh theo key
sinh_vien["gpa"] = 8.3    # thêm key mới
```

→ Lưu thông tin JSON/API response, cấu hình, đếm tần suất từ (word count), cache.