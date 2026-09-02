---
created: 2026-09-02
---
> [!summary] Tóm tắt nhanh
> - **Histogram**: đếm số pixel ở mỗi mức cường độ (0-255) → cho biết độ sáng, tương phản, dynamic range của ảnh, nhưng KHÔNG biết vị trí pixel.
> - **Cumulative Histogram**: cộng dồn histogram → dùng để tính "vị trí phần trăm" của một mức xám trong toàn ảnh.
> - **Point Operations**: biến đổi từng pixel theo hàm f(cường độ cũ) → cường độ mới, không quan tâm vị trí hay pixel lân cận. Gồm: cộng/nhân (đổi sáng/tương phản), đảo ảnh, thresholding, log/gamma transform, windowing.
> - **Nguyên lý quan trọng**: Point Operations chỉ dịch/gộp histogram, không đảo ngược được khi đã gộp.
> - **Histogram Equalization**: một Point Operation đặc biệt biến histogram thành phân phối đều để tăng tương phản ảnh tối/mờ. 

# PHẦN 0: Kiến thức nền — Ảnh là gì trong máy tính?

Với máy tính, một tấm ảnh xám (**grayscale**) chỉ là một bảng số.

Tưởng tượng ảnh là một bảng lưới ô vuông (như bảng Excel), mỗi ô vuông nhỏ gọi là **pixel** (điểm ảnh). Mỗi pixel chứa **một con số** thể hiện độ sáng/tối:

- 0 = đen tuyệt đối
- 255 = trắng tuyệt đối
- Các số ở giữa (1-254) = các mức xám

Ví dụ một ảnh nhỏ 4x4 pixel có thể là:

```
 10  10  200 200
 10  10  200 200
 50  50  180 180
 50  50  180 180
```

Đây là một ảnh có 2 vùng: góc trên trái tối (10), góc trên phải sáng (200), v.v.

Với ảnh 8-bit (loại phổ biến nhất), mỗi pixel dùng 8 bit để lưu → có 2^8 = **256 mức xám** (0 đến 255). Đây là ý nghĩa của con số K=256 xuất hiện xuyên suốt slide.

# Phần 1: Histogram
## 1.1 Histogram là gì?

Histogram trả lời câu hỏi: **"Trong ảnh này, có bao nhiêu pixel mang mỗi giá trị cường độ?"**
Quay lại ví dụ ảnh 4x4 ở trên, ta đếm:

- Giá trị 10: xuất hiện 4 lần
- Giá trị 50: xuất hiện 4 lần
- Giá trị 180: xuất hiện 4 lần
- Giá trị 200: xuất hiện 4 lần
- 
Histogram chính là biểu đồ cột thể hiện các con số đếm này, với trục hoành là giá trị cường độ (0→255), trục tung là số lượng pixel.

**Ví dụ** (K=16 mức, để đơn giản hơn 256):
```
Cường độ i:  0  1  2   3  4  5  6  7  8  9  10 11 12 13 14 15
Số pixel:    0  2  10  0  0  0  5  7  3  9  1  6  3  6  3  2
```

**Công thức formal:**
```
h(i) = card{(u,v) | I(u,v) = i}
```
Đọc là: "h(i) = số lượng (card = cardinality, tức đếm số phần tử) các tọa độ pixel (u,v) sao cho giá trị cường độ tại đó bằng i". Nói dễ hiểu: **h(i) = đếm xem có bao nhiêu pixel bằng i**.

## 1.2 Đặc điểm quan trọng: Histogram KHÔNG cho biết vị trí

Đây là điểm mấu chốt hay bị hiểu nhầm. Histogram chỉ là **thống kê tổng quát**, không biết pixel sáng/tối nằm ở đâu trong ảnh.

![[Pasted image 20260902091430.png]]
**Ví dụ minh họa**: 3 tấm ảnh bên trên có **cùng một histogram** (đều có 50% pixel xám, 50% pixel trắng):

1. Ảnh chia đôi: nửa trái xám, nửa phải trắng
2. Ảnh có hình tròn xám ở giữa, nền trắng
3. Ảnh caro (bàn cờ) xám-trắng xen kẽ

→ Cả 3 ảnh nhìn hoàn toàn khác nhau, nhưng histogram giống hệt nhau vì chỉ đếm **số lượng**, không quan tâm **vị trí**.

**Hệ quả quan trọng**: Không thể tái tạo lại ảnh gốc chỉ từ histogram — vì đã mất thông tin vị trí.

# PHẦN 2: Ứng dụng thực tế của Histogram

## 2.1 Phát hiện lỗi phơi sáng

Khi chụp ảnh, nếu histogram dồn hết về một phía → ảnh có vấn đề:

|Loại|Hình dạng histogram|Ý nghĩa|
|---|---|---|
|Thiếu sáng (underexposed)|Dồn về bên trái (gần 0)|Ảnh quá tối|
|Đủ sáng (properly exposed)|Trải đều ở giữa|Ảnh tốt|
|Dư sáng (overexposed)|Dồn về bên phải (gần 255)|Ảnh quá sáng, "cháy"|

**Ví dụ dễ hình dung**: Chụp ảnh trong phòng tối mà không bật đèn → hầu hết pixel sẽ có giá trị thấp (gần đen) → histogram sẽ có một "núi" cao ở phía trái. Ngược lại chụp ngược sáng mặt trời → nhiều pixel bị "cháy trắng" → núi dồn về bên phải.
## 2.2 Độ sáng (Brightness)

Định nghĩa cực đơn giản: **lấy trung bình cộng của TẤT CẢ giá trị pixel trong ảnh.**

```
B(I) = (1/w×h) × Σ tất cả cường độ pixel
```

Ví dụ: ảnh 4x4 ở trên có tổng = (10×4 + 50×4 + 180×4 + 200×4) = 1760, chia cho 16 pixel = **110**. Vậy độ sáng trung bình của ảnh này là 110 (khá tối, vì dưới mức giữa 127).

## 2.3 Độ tương phản (Contrast)

Tương phản = mức độ **dễ phân biệt** các vật thể trong ảnh với nhau.

- **Tương phản cao**: có cả vùng rất tối lẫn vùng rất sáng, nhiều mức xám khác biệt rõ rệt → nhìn "sắc nét", nổi bật
- **Tương phản thấp**: mọi pixel gần giống nhau về độ sáng (ví dụ toàn xám xịt) → nhìn "bẹt", mờ mịt, khó phân biệt

Trên histogram: tương phản cao = histogram **trải rộng** từ gần 0 đến gần 255; tương phản thấp = histogram **co cụm** trong một dải hẹp (ví dụ chỉ nằm trong khoảng 100-150).

**Ví dụ đời thường**: Ảnh chụp trong sương mù thường có tương phản thấp — mọi thứ đều xám xịt gần giống nhau, khó thấy chi tiết.

## 2.4 Dynamic Range (dải động)

Là **số lượng mức cường độ khác nhau THỰC SỰ xuất hiện** trong ảnh (khác với contrast — dynamic range đếm số mức riêng biệt, không quan tâm chúng có "rộng" hay không).

![[Pasted image 20260902114322.png]]
Ví dụ trong slide, cùng 1 ảnh nhưng:

- High dynamic range: dùng đủ 256 mức xám → ảnh mượt mà
- Low dynamic range (64 mức): giảm số mức xám → bắt đầu thấy "vằn" nhẹ
- Extremely low (6 mức): ảnh bị vằn rất rõ, mất chi tiết (giống hiệu ứng "posterize" trong Photoshop)

**HDR Imaging**: đôi khi cảnh thực tế có độ chênh sáng-tối vượt quá khả năng cảm biến máy ảnh ghi lại trong 1 lần chụp → giải pháp là chụp nhiều tấm với độ phơi sáng khác nhau rồi ghép lại bằng phần mềm để giữ chi tiết ở cả vùng tối và vùng sáng.

## 2.5 Phát hiện lỗi ảnh qua Histogram

- **Saturation (bão hòa)**: ánh sáng thực tế vượt quá khả năng cảm biến → giá trị bị "cắt cụt" về 0 hoặc 255 → tạo ra **gai nhọn (spike)** ở hai đầu histogram
- **Gai/khoảng trống do chỉnh sửa**: nếu một histogram có nhiều gai nhọn xen kẽ khoảng trống bất thường (như "răng lược") → dấu hiệu ảnh đã qua chỉnh sửa/xử lý, không phải ảnh gốc chụp trực tiếp
- **Nén ảnh (GIF/JPEG)**: các thuật toán nén giảm số mức màu thực tế (gọi là **lượng tử hóa - quantization**) → ví dụ ảnh gốc chỉ có 2 màu (xám, trắng) nhưng sau khi nén JPEG lại xuất hiện thêm rất nhiều mức xám không hề có trong ảnh gốc → ảnh nhìn "dơ", mờ, nhòe (đây là JPEG artifact quen thuộc khi bạn nén ảnh quá mạnh)

---

# PHẦN 3: Cumulative Histogram (Histogram tích lũy)

## 3.1 Ý tưởng

Cumulative Histogram = "cộng dồn" histogram từ trái sang phải.

```
H(i) = Σ h(j)  với j chạy từ 0 đến i
```

Nói dễ hiểu: **H(i) = tổng số pixel có cường độ NHỎ HƠN HOẶC BẰNG i.**

## 3.2 Ví dụ minh họa

Lấy lại histogram K=16 ở trên:

```
i:     0  1  2   3  4  5  6  7  8  9  10 11 12 13 14 15
h(i):  0  2  10  0  0  0  5  7  3  9  1  6  3  6  3  2
```

Cumulative histogram H(i) sẽ là:

```
H(0) = h(0) = 0
H(1) = h(0)+h(1) = 0+2 = 2
H(2) = H(1)+h(2) = 2+10 = 12
H(3) = H(2)+h(3) = 12+0 = 12
H(4) = 12 + 0 = 12
...
```

Cứ thế cộng dồn tiếp.

**Định nghĩa đệ quy** (như trong slide) chính là cách tính từng bước như trên:

```
H(i) = h(0)              nếu i=0
H(i) = H(i-1) + h(i)      nếu i>0
```

## 3.3 Tính chất

- **Luôn tăng dần (monotonically increasing)** — vì ta chỉ cộng thêm, không bao giờ trừ
- **Giá trị cuối cùng H(K-1) = tổng số pixel trong ảnh** (M×N, với M,N là chiều rộng/cao ảnh) — vì đó là tổng của TẤT CẢ pixel
- Đây chính là khái niệm tương tự **CDF (Cumulative Density Function)** trong xác suất thống kê

## 3.4 Tại sao cần Cumulative Histogram?

Nó sẽ được dùng làm nền tảng cho **Histogram Equalization** (phần 8) và thuật toán **Histogram Specification/Matching** mình giải thích ở tin nhắn trước — vì nó cho biết "vị trí phần trăm" của một mức cường độ trong toàn bộ phân phối ảnh.

---

# PHẦN 4: Binning (Gộp nhóm)

## 4.1 Vấn đề

Với ảnh có nhiều bit hơn (ví dụ ảnh 32-bit dùng trong khoa học, y tế), số mức cường độ có thể là 2^32 = hơn 4 tỷ! Không thể vẽ histogram với 4 tỷ cột — quá lớn để hiển thị.

## 4.2 Giải pháp: Binning

Thay vì đếm từng giá trị riêng lẻ, ta **gộp một khoảng giá trị lại thành 1 "bin" (thùng chứa)**.

**Ví dụ cụ thể trong slide**: ảnh 14-bit có 2^14 = 16384 mức, muốn gộp về chỉ còn 256 bin:

```
Bin size = 2^14 / 256 = 64
```

Nghĩa là mỗi bin sẽ gộp 64 giá trị liên tiếp lại:

```
h(0) ← đếm pixel có giá trị từ 0 đến 63
h(1) ← đếm pixel có giá trị từ 64 đến 127
h(2) ← đếm pixel có giá trị từ 128 đến 191
...
h(255) ← đếm pixel có giá trị từ 16320 đến 16383
```

Giống như việc thay vì hỏi "có bao nhiêu người tuổi chính xác 23, 24, 25..." ta hỏi gộp lại "có bao nhiêu người trong độ tuổi 20-29, 30-39..." — dễ nhìn tổng quan hơn.

---

# PHẦN 5: Color Image Histogram (Histogram ảnh màu)

Ảnh màu (RGB) có 3 kênh màu: Red, Green, Blue — mỗi pixel có 3 con số thay vì 1.

**2 cách làm histogram cho ảnh màu:**

1. **Intensity histogram**: chuyển ảnh màu về xám trước (dùng công thức tính độ sáng chung từ R,G,B), rồi vẽ histogram như bình thường
2. **Individual Channel Histogram**: vẽ riêng 3 histogram cho kênh R, kênh G, kênh B

**Hạn chế**: cả 2 cách đều **không phản ánh đúng sự phân bố màu thực tế** — vì cũng giống vấn đề ở Phần 1.2, hai ảnh có màu sắc hoàn toàn khác nhau (VD: ảnh 1 toàn pixel đỏ tươi, ảnh 2 pha trộn đỏ+xanh+vàng) vẫn có thể cho ra 3 histogram R,G,B giống hệt nhau, vì mỗi kênh chỉ đếm riêng lẻ, không biết tại một pixel cụ thể thì 3 giá trị R,G,B đó **kết hợp với nhau** ra màu gì. (Slide có nhắc "Combined Color Histogram" là giải pháp, nhưng sẽ học ở bài sau.)

---

# PHẦN 6: Point Operations (Phép toán điểm ảnh)

## 6.1 Định nghĩa

Point Operation = biến đổi **giá trị của MỘT pixel** dựa theo một hàm số f(), và **không quan tâm đến các pixel xung quanh nó, cũng không quan tâm vị trí pixel đó nằm ở đâu**.

```
I'(u,v) ← f(I(u,v))
```

Đọc: "giá trị mới tại vị trí (u,v) = áp dụng hàm f lên giá trị cũ tại (u,v)".

Đây gọi là **homogeneous operation** (phép toán đồng nhất) — vì cùng một hàm f được áp dụng **giống hệt nhau** cho MỌI pixel trong ảnh, bất kể pixel đó nằm ở góc trên hay ở giữa ảnh.

**Ví dụ dễ hiểu**: Tưởng tượng bạn có công thức "cộng thêm 10 vào mọi giá trị" — bạn áp dụng công thức này cho từng pixel một cách độc lập, không cần biết pixel bên cạnh nó là gì.

**Pseudocode minh họa** (trong slide):

```
for v = 1..h
    for u = 1..w
        I(u,v) = f(I(u,v))
```

Tức là 2 vòng lặp quét qua toàn bộ ảnh, mỗi pixel áp dụng hàm f một lần.

## 6.2 Các loại Point Operation phổ biến

**a) Addition (Cộng) — thay đổi độ sáng**

```
f(p) = p + k
```

Ví dụ: `f_bright(p) = p + 10` → mọi pixel sáng thêm 10 đơn vị. Pixel gốc 100 → thành 110. Cả ảnh sẽ sáng lên đều.

**b) Multiplication (Nhân) — thay đổi độ tương phản**

```
f(p) = k × p
```

Ví dụ: `f_contrast(p) = p × 1.5` → mọi pixel nhân lên 1.5 lần. Pixel gốc 100 → thành 150. Điều này làm khoảng cách giữa các mức sáng-tối giãn ra → tăng tương phản.

**c) Các hàm thực (log, exp, lũy thừa...)**: dùng để biến đổi phi tuyến, sẽ nói kỹ ở phần 7.

**d) Quantizing (lượng tử hóa), Global thresholding, Gamma correction**: các biến thể cụ thể sẽ giải thích bên dưới.

## 6.3 Clamping (Kẹp giá trị)

**Vấn đề**: Nếu bạn cộng thêm 10 vào một pixel đã là 250, kết quả sẽ là 260 — nhưng ảnh 8-bit chỉ cho phép giá trị từ 0-255! Giá trị 260 là **không hợp lệ**.

**Giải pháp - Clamping**: ép các giá trị vượt ngưỡng về giới hạn cho phép.

```
f(p) = a     nếu p < a
f(p) = p     nếu a ≤ p ≤ b
f(p) = b     nếu p > b
```

Đơn giản: nếu tính ra > 255 thì gán = 255; nếu < 0 thì gán = 0.

**Ví dụ code trong slide**: tăng tương phản 50% rồi clamp:

python

```python
a = int(pixels[y, x] * 1.5 + 0.5)   # tăng 50%
if a > 255:
    a = 255                          # clamp
pixels[y, x] = a
```

## 6.4 Đảo ảnh (Inverting / Negative)

```
f_invert(a) = a_max - a
```

Ví dụ với a_max=255: pixel có giá trị 50 → thành 255-50 = 205 (từ tối chuyển thành sáng, và ngược lại). Đây chính là hiệu ứng "âm bản" (negative) như phim chụp ngày xưa.

**Ứng dụng thực tế nêu trong slide**: ảnh y tế (mammogram - chụp X-quang vú) khi đảo ngược màu giúp bác sĩ nhìn rõ mô/khối u nằm trong vùng tối hơn — vì mắt người đôi khi dễ nhận ra chi tiết tối trên nền sáng hơn là ngược lại.

## 6.5 Thresholding (Ngưỡng hóa)

Biến ảnh xám thành ảnh **nhị phân** (chỉ có 2 giá trị: đen hoặc trắng) dựa trên một ngưỡng a_th:

```
f(a) = a0   nếu a < a_th   (thường a0 = 0, tức đen)
f(a) = a1   nếu a ≥ a_th   (thường a1 = 1 hoặc 255, tức trắng)
```

**Ví dụ trực quan trong slide**: ảnh một con robot đồ chơi trên nền tối, với ngưỡng a_th=128 → mọi pixel tối hơn 128 thành đen tuyệt đối, mọi pixel sáng hơn thành trắng tuyệt đối → tách được hình con robot ra khỏi nền (kỹ thuật này rất hay dùng để "tách vật thể khỏi nền" trong xử lý ảnh cơ bản).

## 6.6 Non-Homogeneous Point Operation (Phép toán KHÔNG đồng nhất)

Khác với Point Operation thông thường (chỉ phụ thuộc giá trị cũ), loại này còn phụ thuộc **vị trí (u,v)** của pixel:

```
I'(u,v) ← g(I(u,v), u, v)
```

Ví dụ: bạn muốn làm sáng dần từ trái sang phải của ảnh — thì công thức phải biết pixel đang ở cột nào (u) để tính độ sáng thêm phù hợp, chứ không thể áp dụng y hệt một công thức cho mọi vị trí như Point Operation thường.

---

# PHẦN 7: Basic Grey Level Transformations (Các phép biến đổi mức xám cơ bản)

Slide liệt kê 3 loại biến đổi thường gặp nhất, với đồ thị trục hoành = giá trị đầu vào (r), trục tung = giá trị đầu ra (s):

## 7.1 Linear (Tuyến tính)

- **Identity**: không đổi gì (s=r) — đường chéo thẳng
- **Negative**: đảo ngược (đã giải thích ở 6.4)

## 7.2 Logarithmic (Logarit)

```
s = c × log(1 + r)
```

**Tác dụng**: ánh xạ **một dải hẹp giá trị đầu vào tối** thành **một dải rộng giá trị đầu ra** — tức là "kéo giãn" các chi tiết tối để dễ nhìn thấy hơn, đồng thời "nén lại" các chi tiết sáng.

**Ví dụ trực quan trong slide**: ảnh biến đổi Fourier (một loại ảnh có 1 điểm cực sáng ở giữa, còn lại rất tối) — nếu nhìn trực tiếp sẽ chỉ thấy 1 điểm sáng, mọi chi tiết tối xung quanh gần như không thấy được. Sau khi áp dụng log transform, các chi tiết tối được "kéo giãn sáng lên" → nhìn thấy rõ nhiều vòng tròn chi tiết mà trước đó bị ẩn đi.

## 7.3 Power Law (Lũy thừa) — còn gọi là Gamma Correction

```
s = c × r^γ  (γ đọc là "gamma")
```

Đây là họ hàm rất linh hoạt — tùy chỉnh giá trị γ sẽ cho ra các đường cong khác nhau:

- γ < 1 (ví dụ γ=0.4): giống hàm log — kéo giãn vùng tối, nén vùng sáng
- γ > 1 (ví dụ γ=2.5): ngược lại — kéo giãn vùng sáng, nén vùng tối
- γ = 1: không đổi gì (identity)

**Ví dụ trực quan trong slide**: ảnh MRI cột sống gãy — với γ=0.6 thấy rõ chi tiết khác so với γ=0.4 hay γ=0.3. Mỗi giá trị γ khác nhau sẽ "làm nổi bật" các chi tiết khác nhau trong cùng 1 ảnh — đây là lý do gamma correction rất hay dùng trong xử lý ảnh y tế, camera, màn hình (bạn có nghe từ "chỉnh gamma" trên TV/màn hình — chính là ứng dụng công thức này).

---

# PHẦN 8: Intensity Windowing (Cửa sổ cường độ)

## 8.1 Ý tưởng

Là sự kết hợp giữa **Clamping** (phần 6.3) và **kéo giãn tuyến tính (linear stretching)**.

Công thức: với khoảng muốn "windowing" là [a,b], và giá trị sáng tối đa cho phép là M:

```
f(p) = 0                nếu p < a
f(p) = M × (p-a)/(b-a)   nếu a ≤ p ≤ b
f(p) = M                nếu p > b
```

**Ý tưởng dễ hiểu**: bạn chỉ quan tâm đến một **khoảng giá trị cụ thể** [a,b] trong ảnh (ví dụ chỉ quan tâm vùng cường độ từ 80 đến 180), mọi giá trị ngoài khoảng đó bị cắt về 0 hoặc M. Còn bên trong khoảng đó, giá trị sẽ được **kéo giãn ra** để lấp đầy toàn bộ dải 0→M.

**Ví dụ trực quan trong slide**: ảnh con robot tối, sau khi windowing thì nền đen tuyệt đối hoàn toàn (bị cắt về 0), còn vùng thân robot (vốn chỉ dao động trong khoảng hẹp) được kéo giãn ra để thấy rõ chi tiết tương phản trên thân robot mà trước đó khó nhìn thấy vì quá tối.

**Ứng dụng thực tế**: rất phổ biến trong ảnh y tế (CT scan) — bác sĩ chỉ muốn nhìn rõ 1 loại mô cụ thể (ví dụ mô xương) nằm trong 1 khoảng cường độ nhất định, nên "windowing" vào đúng khoảng đó để làm nổi bật, bỏ qua các mô khác.

---

# PHẦN 9: Mối quan hệ giữa Point Operations và Histogram

## 9.1 Nguyên lý quan trọng

Slide nhấn mạnh: **Point Operations chỉ DỊCH CHUYỂN (shift) hoặc GỘP (merge) các cột của histogram, không tạo ra thông tin mới.**

**Ví dụ minh họa trong slide**: có 4 cột histogram màu đen, xanh lá, hồng, xám ở các vị trí khác nhau. Sau khi áp dụng một phép toán (ví dụ cộng dồn 2 giá trị a1, a2 lại thành 1), cột xanh lá và cột hồng bị **gộp lại thành 1 cột** duy nhất tại vị trí a2, với chiều cao = tổng 2 cột cũ.

## 9.2 Hệ quả: KHÔNG THỂ ĐẢO NGƯỢC (irreversible)

Đây là điểm cực kỳ quan trọng cần nhớ: **một khi 2 cột histogram đã bị gộp lại thành 1, ta KHÔNG THỂ tách chúng ra lại được** — vì đã mất thông tin về việc ban đầu chúng là 2 giá trị riêng biệt.

**Ví dụ dễ hiểu**: giống như việc bạn trộn 2 ly nước có 2 màu khác nhau vào 1 ly — sau khi trộn xong, không thể tách lại thành 2 ly màu riêng như ban đầu. Tương tự, khi bạn áp dụng phép nhân/cộng làm 2 giá trị cường độ khác nhau (ví dụ 100 và 101) trở thành cùng 1 giá trị sau biến đổi (ví dụ cả 2 đều thành 150 do làm tròn số) — thông tin "ban đầu chúng khác nhau" đã mất vĩnh viễn.

---

# PHẦN 10: Automatic Contrast Adjustment (Tự động điều chỉnh tương phản)

## 10.1 Ý tưởng

Đây là một Point Operation cụ thể: tìm giá trị **sáng nhất** và **tối nhất** thực sự có trong ảnh, rồi **kéo giãn tuyến tính** để lấp đầy toàn bộ dải 0-255.

**Bước 1**: Tìm a_low (giá trị tối nhất) và a_high (giá trị sáng nhất) trong ảnh gốc.

**Bước 2**: Kéo giãn tuyến tính:

```
f_ac(a) = a_min + (a - a_low) × (a_max - a_min)/(a_high - a_low)
```

Nếu a_min=0, a_max=255, công thức đơn giản lại thành:

```
f_ac(a) = (a - a_low) × 255/(a_high - a_low)
```

**Ví dụ dễ hiểu**: giả sử ảnh gốc chỉ dùng các giá trị từ 50 đến 200 (không dùng hết dải 0-255, tức tương phản kém). Sau khi Automatic Contrast Adjustment, giá trị 50 sẽ được kéo về 0, giá trị 200 sẽ được kéo lên 255, và mọi giá trị ở giữa cũng được kéo giãn tương ứng theo tỷ lệ → toàn bộ dải 0-255 được sử dụng đầy đủ → ảnh nhìn tương phản hơn hẳn.

## 10.2 Nhược điểm

Vì đây cũng là Point Operation, nên nó tuân theo nguyên lý ở Phần 9: **kéo giãn tuyến tính sẽ tạo ra các khoảng trống (gaps) trong histogram**.

Lý do: nếu ảnh gốc chỉ có 150 mức xám khác nhau (50-200), nhưng sau khi kéo giãn ra thành 0-255 (256 mức) → không đủ dữ liệu gốc để lấp đầy hết 256 vị trí → tạo ra các "khe hở" xen kẽ trong histogram (giống như kéo giãn 1 sợi dây thun có 150 hạt cườm ra chiều dài của 256 hạt — sẽ có khoảng trống giữa các hạt).

## 10.3 Modified Contrast Adjustment (phiên bản cải tiến)

**Vấn đề của bản gốc**: nếu ảnh có nhiễu (noise) — chỉ 1-2 pixel bị lỗi có giá trị cực đoan (ví dụ 1 pixel bị nhiễu thành giá trị 5, trong khi cả ảnh dao động 100-200) — thì a_low sẽ bị kéo về 5, làm hỏng toàn bộ phép kéo giãn.

**Giải pháp**: thay vì lấy đúng giá trị min/max tuyệt đối, người ta bỏ qua một **tỷ lệ phần trăm nhỏ (percentile)** ở 2 đầu — ví dụ bỏ qua 1% pixel tối nhất và 1% pixel sáng nhất (coi đó là nhiễu), rồi mới tìm â_low, â_high từ phần còn lại để kéo giãn — kết quả ổn định hơn, ít bị ảnh hưởng bởi nhiễu.

---

# PHẦN 11: Histogram Equalization (Cân bằng Histogram)

## 11.1 Ý tưởng

Đây là kỹ thuật rất nổi tiếng trong xử lý ảnh: biến đổi histogram của ảnh sao cho nó trở thành **phân phối đều (uniform)** — nghĩa là mọi mức cường độ (0 đến 255) đều có **số lượng pixel gần bằng nhau**.

**Tại sao làm vậy?** Vì ảnh tối hoặc ảnh "bẹt" (như đã nói ở phần 2.3 - tương phản thấp) thường có histogram co cụm trong 1 khoảng hẹp. Nếu "trải đều" ra toàn bộ dải 0-255, ảnh sẽ trông tương phản hơn, chi tiết dễ thấy hơn — đặc biệt hữu ích với ảnh tối hoặc ảnh "bị mờ nhạt" (washed out).

## 11.2 Cách hoạt động

Dùng **Cumulative Histogram** (Phần 3) làm nền tảng: nếu cumulative histogram gốc H(i) có hình dạng cong bất kỳ, ta biến đổi sao cho cumulative histogram mới H_eq(i) trở thành **một đường thẳng** (vì đường thẳng chính là đặc trưng toán học của phân phối đều).

Công thức tổng quát:

```
s_k = T(r_k)
```

với r_k là cường độ đầu vào, s_k là cường độ đầu ra sau biến đổi, T() là hàm biến đổi (chính là hàm tính từ cumulative histogram chuẩn hóa).

## 11.3 Ví dụ trực quan trong slide

Slide cho thấy 2 ảnh lá cây: 1 ảnh tối (histogram co cụm bên trái), 1 ảnh sau equalization (histogram được kéo dàn ra khắp dải, nhưng có dạng "răng lược" — điều này khớp với nguyên lý ở Phần 9: point operation gây ra gaps).

Bên phải slide là đồ thị **hàm biến đổi (transformation function)**: trục X là input intensity (0-1, chuẩn hóa), trục Y là output intensity. Đường cong này chính là T() được tính tự động từ cumulative histogram của ảnh gốc.

**Lưu ý quan trọng**: Đây chính xác là **trường hợp đặc biệt** của thuật toán **Piecewise Linear Histogram Specification** mà mình đã giải thích ở tin nhắn trước — histogram equalization = specification với đường tham chiếu L_R là MỘT đường thẳng duy nhất nối từ (0,0) đến (K-1, 1).

## 11.4 Các biến thể

Slide cho thấy có nhiều "Equalization Transformation Function" khác nhau (đánh số 1-4 trong đồ thị) — mỗi hàm sẽ tạo ra kết quả equalization hơi khác nhau tùy vào cách xử lý làm tròn số và các chi tiết kỹ thuật khác, nhưng ý tưởng chung đều giống nhau: biến histogram thành phân phối đều.