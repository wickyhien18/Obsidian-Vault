---
created: 2026-09-13
Hub: "[[MOC - Image & Processing]]"
status: seed
---

> [!summary] Tóm tắt nhanh
> -  Convolution có thể tách nhỏ để tính nhanh hơn (đặc biệt với Gaussian).
> - Nhiễu ảnh có nhiều loại, mỗi loại cần cách xử lý khác nhau; **median filter** là công cụ mạnh để khử nhiễu salt-and-pepper mà vẫn giữ cạnh.
> - Để tìm cạnh trong ảnh: tính đạo hàm (gradient) theo cả 2 chiều ngang/dọc bằng các filter như Sobel, Prewitt, Roberts, rồi tính độ lớn + hướng gradient, sau đó lọc bớt để chỉ giữ lại các cạnh "sắc nét thật sự" bằng non-maxima suppression. 
### PHẦN 1: BỘ LỌC ẢNH (FILTERS)

#### 1. Nhắc lại: Convolution (Tích chập) là gì?

![[Pasted image 20260913132928.png]]

Tưởng tượng bạn có một tấm ảnh (là một lưới các con số, mỗi số là độ sáng của 1 pixel). Convolution là thao tác: đặt một "cửa sổ nhỏ" (gọi là filter/kernel, ví dụ ma trận 3x3) lên từng vị trí trên ảnh, nhân từng số trong cửa sổ với pixel tương ứng bên dưới, rồi cộng tất cả lại → ra 1 pixel mới, đặt vào ảnh kết quả.

Nói đơn giản: filter giống như "công thức pha trộn" các điểm ảnh xung quanh để tạo ra điểm ảnh mới → dùng để làm mờ, làm nét, tìm cạnh, v.v.

#### 2. Tính chất toán học của Convolution

![[Pasted image 20260913135506.png]]

- **Giao hoán**: I * H = H * I (lọc ảnh với filter hay lọc filter với ảnh đều ra kết quả giống nhau)
- **Tuyến tính**: nhân ảnh với 1 số rồi lọc = lọc rồi nhân với số đó; lọc tổng 2 ảnh = tổng của lọc từng ảnh
- **Kết hợp**: A * (B_C) = (A_B) * C → áp filter theo thứ tự nào cũng ra kết quả như nhau

#### 3. Tính "tách được" (Separability) — mẹo giúp tính nhanh hơn

![[Pasted image 20260913135817.png]]
Một số filter lớn (ví dụ 3x5) có thể **tách** thành 2 filter nhỏ hơn: 1 filter theo chiều ngang và 1 filter theo chiều dọc, áp dụng lần lượt.

**Tại sao quan trọng?** Vì tính toán nhanh hơn rất nhiều:
![[Pasted image 20260913135952.png]]

- Filter 3x5 không tách: cần 15 phép tính cho mỗi điểm ảnh
- Tách ra Hx (ngang) rồi Hy (dọc): chỉ cần 3+5 = 8 phép tính

Với filter kích thước MxM: nếu không tách được thì độ phức tạp tăng theo M² (bình phương — chậm), còn nếu tách được thì chỉ tăng theo M (tuyến tính — nhanh hơn nhiều khi M lớn).

![[Pasted image 20260913140312.png]]
**Ví dụ điển hình: bộ lọc Gaussian** (dùng làm mờ ảnh mượt mà) có thể tách thành 2 bộ lọc Gaussian 1 chiều (ngang và dọc) nhân lại với nhau — nên khi lập trình, người ta luôn tách ra để tính cho nhanh.

#### 4. Hàm Impulse (xung Dirac)

![[Pasted image 20260913150031.png]]

Đây là một "ảnh" đặc biệt: chỉ có 1 điểm trắng ở giữa, còn lại toàn đen. Tính chất thú vị:

- Lọc ảnh bất kỳ với hàm impulse → ảnh không đổi (impulse "trung tính")
  ![[Pasted image 20260913150101.png]]
- Lọc hàm impulse với 1 filter H → kết quả chính là filter H đó (dùng để "nhìn thấy" hình dạng thật của 1 filter)
  ![[Pasted image 20260913150123.png]]

#### 5. Nhiễu (Noise) trong ảnh

Khi chụp ảnh, ảnh có thể bị lỗi/nhiễu do mờ nét, rung tay, v.v. Các loại nhiễu chính:

- **Salt-and-pepper (nhiễu muối tiêu)**: các đốm đen/trắng ngẫu nhiên rải rác trên ảnh (giống hạt muối, hạt tiêu)
  ![[Pasted image 20260913152322.png]]
- **Gaussian noise**: nhiễu ngẫu nhiên cộng vào toàn ảnh, phân bố theo hình chuông
  ![[Pasted image 20260913152402.png]]
- **Speckle noise**: nhiễu nhân vào giá trị điểm ảnh (thay vì cộng)
  ![[Pasted image 20260913152508.png]]
- **Periodic noise**: nhiễu có tính chu kỳ (như các sọc lặp lại) — loại này phải xử lý ở "miền tần số" (nội dung sau này), còn 3 loại trên xử lý được bằng filter không gian bình thường
  ![[Pasted image 20260913152610.png]]

#### 6. Bộ lọc phi tuyến (Non-Linear Filters)

**Vấn đề**: Các filter tuyến tính (như làm mờ trung bình) làm mờ luôn cả cạnh và chi tiết ảnh → giảm chất lượng, không phải cách tốt để khử nhiễu salt-and-pepper (chỉ làm nhòe nhiễu ra chứ không xóa sạch).

**Giải pháp — bộ lọc phi tuyến**: không cộng/nhân theo công thức tuyến tính mà dùng phép so sánh, sắp xếp:

- **Min filter**: lấy giá trị nhỏ nhất trong vùng cửa sổ → làm mất các đốm sáng, mở rộng vùng tối
- **Max filter**: lấy giá trị lớn nhất → làm mất đốm tối, mở rộng vùng sáng
  ![[Pasted image 20260913153308.png]]
- **Median filter (bộ lọc trung vị)** — quan trọng nhất: sắp xếp tất cả giá trị điểm ảnh trong cửa sổ theo thứ tự, lấy giá trị **ở giữa** làm kết quả.
    - Hiệu quả rất tốt để khử nhiễu salt-and-pepper mà vẫn giữ được cạnh/đường nét ảnh
    - Nhược điểm: đôi khi tạo ra các vùng phẳng nhỏ làm giảm độ sắc nét    
![[Pasted image 20260913153518.png]]
![[Pasted image 20260913153740.png]]
- **Weighted median filter**: giống median nhưng cho một số vị trí trong cửa sổ "nhiều phiếu bầu" hơn (được lặp lại nhiều lần trong danh sách trước khi sắp xếp) → linh hoạt hơn
![[Pasted image 20260913154527.png]]
- **Outlier method**: cách khác để khử nhiễu — so sánh giá trị điểm ảnh với giá trị trung bình của 8 điểm xung quanh, nếu khác biệt quá lớn (> ngưỡng D) thì coi là nhiễu và thay bằng giá trị trung bình đó. Cách này không tốt bằng median filter.
  ![[Pasted image 20260913154827.png]]

#### 7. Xử lý viền ảnh (Extending Image Along Borders)

Khi filter chạy đến rìa ảnh, không đủ điểm ảnh xung quanh để tính → cần "mở rộng" ảnh ra ngoài biên bằng 1 trong 4 cách: đệm hằng số (pad), lặp lại điểm biên gần nhất (extend), lấy đối xứng gương (mirror), hoặc lặp tuần hoàn (wrap).
![[Pasted image 20260913155010.png]]

---

### PHẦN 2: PHÁT HIỆN CẠNH (EDGES) VÀ CONTOUR

#### 1. Cạnh (Edge) là gì?

Cạnh là nơi có **sự thay đổi đột ngột về độ sáng** trong ảnh. Cạnh xuất hiện ở: ranh giới giữa các vật thể, thay đổi bề mặt vật thể, thay đổi ánh sáng (như bóng đổ).

#### 2. Đặc điểm toán học của cạnh

- Cạnh lý tưởng giống như một "hàm bậc thang" (đột ngột nhảy từ tối sang sáng)
- Cạnh thật thường hơi mờ (blur) chứ không nhảy vọt hoàn toàn
- **Đạo hàm bậc 1** của độ sáng sẽ có **đỉnh (peak)** tại vị trí cạnh
- **Đạo hàm bậc 2** sẽ có điểm **cắt qua 0 (zero crossing)** tại cạnh

→ Đây là lý do vì sao để tìm cạnh, người ta tính đạo hàm của ảnh.

#### 3. Tính đạo hàm cho ảnh (là dữ liệu rời rạc)

Vì ảnh gồm các điểm rời rạc (không phải hàm liên tục), người ta dùng "sai phân hữu hạn" để ước lượng đạo hàm:

- **Forward difference**: f(x+1) - f(x) (dùng điểm bên phải)
- **Backward difference**: f(x) - f(x-1) (dùng điểm bên trái)
- **Central difference**: trung bình 2 cái trên = 0.5×(f(x+1) - f(x-1)) — chính xác hơn

Các phép sai phân này có thể viết dưới dạng **filter convolution** đơn giản, ví dụ: H = [-0.5, 0, 0.5]

#### 4. Gradient (Građient) của ảnh

Gradient là vector chỉ **hướng** thay đổi nhanh nhất của độ sáng, và **độ lớn** của nó cho biết mức độ thay đổi mạnh hay yếu. Gradient luôn **vuông góc** với đường cạnh.

Với ảnh 2D, ta tính đạo hàm theo chiều ngang (x) và chiều dọc (y), rồi:

- **Độ lớn gradient** = căn bậc 2 của (đạo hàm x)² + (đạo hàm y)² → cho biết "cạnh mạnh hay yếu"
- **Hướng gradient** = arctan(đạo hàm y / đạo hàm x) → cho biết cạnh nằm theo hướng nào

#### 5. Các bộ lọc tìm cạnh (Edge Operators) cụ thể

Vì đạo hàm đơn giản rất nhạy với nhiễu, người ta kết hợp đạo hàm với việc **lấy trung bình** xung quanh để ổn định hơn:

- **Prewitt operator**: tính đạo hàm theo 1 hướng + lấy trung bình đều theo hướng vuông góc
- **Sobel operator**: giống Prewitt nhưng trọng số ở giữa cao hơn (nhấn mạnh điểm gần trung tâm) → chính xác hơn Prewitt. Đây là bộ lọc được dùng phổ biến nhất (kể cả trong ImageJ, qua menu Process → Find Edges)
- **Roberts operator**: tính gradient theo 2 đường chéo thay vì ngang/dọc — đơn giản, nhanh nhưng kém chính xác hơn
- **Compass operators (Kirsh)**: dùng tới 8 filter khác nhau, mỗi cái nhạy với 1 hướng cạnh riêng (cách nhau 45°) → xác định được cả cường độ lẫn hướng cạnh chính xác hơn, đánh đổi bằng việc phải tính nhiều filter hơn

#### 6. Non-Maxima Suppression (Triệt tiêu không cực đại)

Sau khi tính được độ lớn gradient khắp ảnh, nếu chỉ dùng ngưỡng đơn giản (threshold) thì cạnh sẽ bị "dày" (nhiều điểm liền kề đều được coi là cạnh).

Giải pháp: chỉ giữ lại 1 điểm là "cạnh thật" nếu độ lớn gradient của nó là **lớn nhất cục bộ** theo hướng gradient (so với 2 điểm lân cận trước/sau theo hướng đó) — nhờ vậy cạnh sẽ mỏng, rõ nét, chỉ 1 pixel thay vì cả dải rộng.