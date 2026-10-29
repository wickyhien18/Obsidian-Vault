---
title: Thứ Tư 28/10 - Phần 2 (30 phút): Case study 1, đọc bảng số liệu
tags: [case-study, pandas, tuan-3]
---

# 28/10, Phần 2 (30 phút): Case study 1

**Lịch hôm nay: 1 giờ.** Phần 1 = Buổi 2 trong `thong_ke_giang_lai.md` (sai số chuẩn, khoảng tin cậy, 30 phút, kèm câu trả lời cho câu hỏi hôm qua đã xong). Phần 2 = file này (30 phút), cách Phần 1 ít nhất 1 tiếng nếu được.

**Vì sao có phần này:** bài đánh giá có một nhóm là **xử lý tình huống thực tiễn**, và cho đến giờ ta chưa luyện nhóm đó. Dạng đề thường: cho một bảng số liệu, hỏi bạn **tính vài tỉ lệ, tìm điểm bất thường, rồi đưa ra khuyến nghị có căn cứ**.

**Luật:** tính tay trước, chạy code sau (làm tròn bình thường, lệch ≤ 0,001 vẫn đúng). Ô trả lời nào là **ý kiến** (khuyến nghị) thì chấm theo căn cứ bằng số của bạn, không có đáp án duy nhất. Đề mơ hồ thì ghi cách hiểu vào ô trả lời.

---

## Tình huống
Một cửa hàng online chạy quảng cáo trên 4 kênh trong tháng vừa rồi. Số liệu (tiền tính bằng **triệu đồng**):

| kênh     | chi phí | lượt click | số đơn | doanh thu |
| -------- | ------- | ---------- | ------ | --------- |
| Facebook | 40      | 8000       | 160    | 120       |
| Google   | 30      | 3000       | 150    | 150       |
| Email    | 5       | 2000       | 100    | 60        |
| Zalo     | 25      | 5000       | 25     | 20        |

## Bốn chỉ số cần biết (có ví dụ làm sẵn)
- **Tỉ lệ chuyển đổi** = số đơn / lượt click. Cho biết bao nhiêu phần người bấm vào thì mua.
- **ROAS** = doanh thu / chi phí. Mỗi 1 triệu chi cho quảng cáo thu về bao nhiêu triệu doanh thu. **Dưới 1 nghĩa là doanh thu chưa bù nổi tiền quảng cáo.**
- **Giá trị trung bình mỗi đơn** = doanh thu / số đơn.
- **Chi phí mỗi đơn** = chi phí / số đơn.

**Ví dụ làm sẵn, kênh Facebook:**
- Tỉ lệ chuyển đổi = 160/8000 = 0,02 (2%).
- ROAS = 120/40 = 3,0.
- Giá trị trung bình mỗi đơn = 120/160 = 0,75 triệu.
- Chi phí mỗi đơn = 40/160 = 0,25 triệu.

**Ví dụ làm sẵn một khuyến nghị tốt** (cho Facebook, ba phần: kết luận, căn cứ bằng số, hành động):
"Facebook đang có lãi nhưng không phải kênh tốt nhất. **Căn cứ:** ROAS 3,0 (mỗi 1 triệu chi thu 3 triệu) nhưng tỉ lệ chuyển đổi chỉ 2%, thấp hơn Google và Email (cùng 5%). **Hành động:** giữ ngân sách, kiểm tra trang đích và nội dung quảng cáo để tăng chuyển đổi trước khi tăng chi."

---

## Bài tập

**C1.** Điền bảng cho ba kênh còn lại (tính tay, chưa chạy code). Facebook đã có ở trên.

| kênh     | chi phí | lượt click | số đơn | doanh thu |
| -------- | ------- | ---------- | ------ | --------- |
| Facebook | 40      | 8000       | 160    | 120       |
| Google   | 30      | 3000       | 150    | 150       |
| Email    | 5       | 2000       | 100    | 60        |
| Zalo     | 25      | 5000       | 25     | 20        |

| kênh   | tỉ lệ chuyển đổi | ROAS | giá trị TB mỗi đơn | chi phí mỗi đơn |
| ------ | ---------------- | ---- | ------------------ | --------------- |
| Google | 0.05             | 5    | 1                  | 0.2             |
| Email  | 0.05             | 12   | 0.6                | 0.05            |
| Zalo   | 0.005            | 0.8  | 0.8                | 1               |

**C2.** Tính cho **cả bốn kênh gộp**: (a) tổng chi phí, (b) tổng doanh thu, (c) ROAS chung. (Chú ý: ROAS chung là tổng doanh thu chia tổng chi phí, **không** phải trung bình bốn ROAS.)
(a): 100 triệu
(b): 350 triệu
(c): 3.5 triệu

**C3.** Kênh nào **bất thường** nhất? Nêu kênh, **hai con số** làm căn cứ (và so với các kênh khác), và nói rõ "bất thường" theo nghĩa nào (lỗ, chuyển đổi thấp, ...).
Trả lời: kênh Zalo là bất thường nhất giá trị trung bình mỗi đơn là 0.8 cao hơn hẳn giá trị trung bình mỗi đơn trên Facebook và Email song tỉ lệ chuyển đổi lại thấp nhất là 0.005

**C4.** Viết **khuyến nghị cho kênh Zalo** theo ba phần như ví dụ làm sẵn (kết luận, căn cứ bằng số, hành động).
Kết luận: kênh Zalo là nơi lỗ nặng nhất
Căn cứ: chi phí trên 1 đơn là 1 cao nhất trong cả 3 kênh song tỉ lệ chuyển đổi thấp nhất 3 kênh và ROAS thì cũng thấp nhất 3 kênh
Hành động: giảm mức độ chi và thay đổi cách thức marketing

**C5.** Giám đốc nói: "Email có ROAS cao nhất (12), hãy dồn hết ngân sách quảng cáo vào Email." Nêu **hai điều cần kiểm tra hoặc lo ngại** trước khi đồng ý (mỗi điều một dòng, có gắn với số liệu trong bảng nếu được).
(1): Tuy rằng ROAS là cao nhất nhưng giá trị trung bình mỗi đơn ít nhất 3 kênh là 0.6
(2): điều thứ 2 tôi chưa nghĩ ra

**C6.** Bằng pandas, dựng bảng trên thành DataFrame, thêm bốn cột chỉ số và in ra các kênh **sắp xếp theo ROAS giảm dần** (chỉ cần hiện cột `kenh` và `roas`).
Điều kiện đề (chép nguyên văn): dùng pandas dựng bẳng trên thành DataFrame, thêm 4 cột chỉ số và in ra các kênh sắp xếp theo ROAS giảm dần (chỉ cần hiện cột kenh và roas)
Kết quả mong đợi (thứ tự bốn kênh, tính tay từ C1):

| kenh     | roas |
| -------- | ---- |
| Email    | 12   |
| Google   | 5    |
| Facebook | 3    |
| Zalo     | 0.8  |

```python
# Trả lời:
import numpy as np, pandas as pd

x = pd.DataFrame({
    "kênh": ["Facebook", "Google", "Email","Zalo"],
    "chi phí": [40,30,5,25],
    "lượt click": [8000,3000,2000,5000],
    "số đơn": [160,150,100,25],
    "doanh thu": [120,150,60,20],
    "tỉ lệ chuyển đổi": [0.02,0.05, 0.05, 0.005],
    "roas": [3.0, 5.0, 12.0, 0.8],
    "giá trị TB mỗi đơn": [0.75, 1, 0.6, 0.8],
    "chi phí mỗi đơn": [0.25, 0.2, 0.05, 1]
})

print(x.loc[:,['kênh', 'roas']].sort_values("roas",ascending=False))
```
Kết quả chạy, khớp C1 hay không: khớp với C1
       kênh  roas
2     Email  12.0
1    Google   5.0
0  Facebook   3.0
3      Zalo   0.8

---

## Ghi nhật ký (điền đủ ba ô)
- Thời gian bắt đầu và kết thúc thực tế (Phần 1, Phần 2): Phần 2: 8h30 - 9h
- Câu bạn nghĩ mình sai nhiều khả năng nhất, và vì sao: C6 vì tôi không rõ thêm ở đây là thêm bằng tay hay dùng hàm tính, nên tôi tự tính bằng tay
- Điều chưa hiểu (nếu "không có" thì nêu đã chạy code kiểm những câu nào): C6 vì tôi không rõ thêm ở đây là thêm bằng tay hay dùng hàm tính, nên tôi tự tính bằng tay
