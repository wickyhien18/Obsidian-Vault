---
created: 2026-09-02
---
> [!summary] Tóm tắt nhanh
> -Histogram Specification/Matching — ép histogram ảnh theo phân bố tham chiếu tùy ý
> -Gamma Correction — hiệu chỉnh quan hệ phi tuyến giữa ánh sáng và tín hiệu camera/màn hình
> -Giới hạn của Point Operations: không làm mờ, không làm nét được
> -Spatial Filter / Convolution — nhân ma trận lọc với vùng lân cận từng pixel rồi cộng lại
> -Smoothing filters (Box, Gaussian): hệ số dương, dùng làm mờ/khử nhiễu
> -Difference filters (Laplacian): hệ số có âm có dương, dùng làm nét/phát hiện biên
> -Tính chất convolution: giao hoán, tuyến tính, kết hợp
> 

### 1. Nhắc lại: Histogram Equalization (Cân bằng biểu đồ mức xám)

**Ảnh số là gì trước đã:** Một ảnh xám (grayscale) là một lưới các pixel, mỗi pixel có một giá trị độ sáng từ 0 (đen) đến 255 (trắng) — tức 256 mức (K=256).

**Histogram (biểu đồ tần suất)** là biểu đồ đếm xem trong ảnh có bao nhiêu pixel ở mỗi mức sáng. Ví dụ ảnh tối thì histogram dồn hết về phía bên trái (giá trị nhỏ), ảnh sáng thì dồn về bên phải.

**Vấn đề:** Nhiều ảnh chụp bị tối, thiếu tương phản — histogram co cụm lại một chỗ thay vì trải đều.

**Giải pháp — Histogram Equalization:** Biến đổi ảnh sao cho histogram "trải đều" ra khắp dải 0-255 (uniform distribution), làm ảnh rõ nét, tương phản tốt hơn. Cách làm là dùng **cumulative histogram** H(i) — nghĩa là đếm "có bao nhiêu pixel có giá trị ≤ i" (cộng dồn).

Công thức trong slide:  

![[Pasted image 20260902122902.png]]

Giải thích từng ký hiệu:

- **a**: giá trị pixel gốc (ví dụ 50)
- **H(a)**: cumulative histogram tại giá trị a — tức tổng số pixel có giá trị ≤ 50
- **M×N**: tổng số pixel của ảnh (chiều rộng × chiều cao)
- **K-1**: 255 (giá trị lớn nhất)

**Ví dụ cụ thể:** Giả sử ảnh có 100 pixel (M×N=100), K=256. Giả sử có 30 pixel có giá trị ≤ 50, tức H(50)=30. Thì:  

![[Pasted image 20260902122959.png]]

Nghĩa là: pixel nào đang có giá trị 50 sẽ được đổi thành 76. Ý tưởng: nếu có 30% pixel ảnh tối hơn hoặc bằng mức này, thì trong ảnh mới nó nên nằm ở khoảng 30% của thang 0-255.

**Lưu ý quan trọng** slide nói: histogram không bao giờ làm phẳng tuyệt đối được vì phép toán điểm chỉ có thể _gộp_ các đỉnh lại chứ không tăng/giảm được số lượng pixel — nó chỉ làm "phẳng nhất có thể".

### 2. Ảnh và Xác suất

Đây là cách nhìn khác: nếu chọn ngẫu nhiên 1 pixel trong ảnh, xác suất nó có giá trị i là:  

![[Pasted image 20260902123029.png]]​

(số pixel có giá trị i / tổng số pixel). Đây chính là histogram được chuẩn hóa (normalize) thành xác suất — nền tảng toán học cho các phép biến đổi phía sau.

### 3. Histogram Specification (Chỉnh về một histogram tùy ý)

**Ý tưởng:** Thay vì làm phẳng (uniform), ta có thể ép ảnh có histogram giống một hình dạng bất kỳ do mình chọn (ví dụ: hình Gauss — vì ảnh thật thường có phân bố giống Gauss chứ không phẳng).

**Ứng dụng thực tế:** Nếu bạn có 2 ảnh chụp bằng 2 máy ảnh khác nhau (màu sắc/độ sáng hơi khác), bạn có thể lấy histogram của ảnh A làm "chuẩn tham chiếu" rồi ép ảnh B giống vậy — làm cho chúng trông như chụp từ cùng một máy.

**Cách làm — piecewise linear (tuyến tính từng đoạn):** Người ta định nghĩa đường cong CDF tham chiếu bằng vài điểm mốc (control points), ví dụ: `(0, 0.05), (60, 0.3), (140, 0.75), (200, 0.95), (255, 1)`. Giữa các điểm này, nội suy tuyến tính (đường thẳng).

Ví dụ: nếu bạn có 2 mốc `(60, 0.3)` và `(140, 0.75)`, và bạn cần biết giá trị tại điểm b=0.5 (nằm giữa), ta nội suy tuyến tính:  

![[Pasted image 20260902123058.png]]

Thuật toán (trong slide, `PiecewiseLinearHistogram`) làm như sau cho mỗi giá trị pixel a từ 0 đến 255:

1. Tính b = xác suất cộng dồn của a trong ảnh gốc (P_A(a))
2. Tìm xem b rơi vào đoạn nào trên đường cong tham chiếu
3. Nội suy ngược để ra a' — giá trị mới thay cho a

**Vấn đề:** Cách này cần đường cong tham chiếu phải **khả nghịch** (invertible) — nghĩa là hàm một-một, không được có đoạn nào giá trị xác suất = 0 (tức không có pixel nào ở mức đó) vì như vậy sẽ không nghịch đảo được.

### 4. Histogram Matching (Ghép histogram)

Khi histogram tham chiếu **không khả nghịch** (ví dụ ảnh tham chiếu có vùng cường độ không xuất hiện pixel nào), ta dùng cách khác: **Histogram Matching**.

**Ý tưởng đơn giản:** Có 2 ảnh I_A (ảnh cần đổi) và I_R (ảnh tham chiếu). Ta muốn làm cho histogram của I_A giống I_R nhất có thể, bằng cách "khớp" (match) 2 cumulative histogram lại với nhau.

Công thức:  
![[Pasted image 20260902123121.png]]

Nói dễ hiểu: với mỗi pixel giá trị a trong ảnh gốc, tính xác suất cộng dồn P_A(a) của nó (ví dụ = 0.4, tức 40% pixel ảnh gốc tối hơn hoặc bằng a). Sau đó tìm trên đường cong tham chiếu P_R, giá trị j **nhỏ nhất** sao cho P_R(j) ≥ 0.4. Giá trị j đó chính là a' — giá trị mới.

**Ví dụ số:** Nếu P_A(100) = 0.4 (40% pixel ảnh gốc ≤ 100), và trên đường cong tham chiếu ta thấy P_R(90)=0.35, P_R(91)=0.41 → chọn j=91 (vì đây là j nhỏ nhất có P_R(j) ≥ 0.4). Vậy pixel giá trị 100 trong ảnh gốc sẽ đổi thành 91.

**Nhận xét từ slide:** Phương pháp này hoạt động tốt khi 2 ảnh có nội dung tương tự nhau (cùng loại cảnh vật), còn nếu nội dung khác biệt lớn thì kết quả trông sẽ không tự nhiên.

### 5. Gamma Correction (Hiệu chỉnh Gamma)

**Vấn đề gốc:** Camera và màn hình không phản ứng "tuyến tính" với ánh sáng. Ví dụ: ánh sáng vào cảm biến camera tăng gấp đôi, nhưng giá trị pixel ghi lại không tăng gấp đôi — mối quan hệ này là phi tuyến (nonlinear), thường theo dạng hàm mũ.

**Gamma là gì?** Xuất phát từ nhiếp ảnh analog (phim): người ta đo mối quan hệ giữa log(cường độ ánh sáng) và mật độ phim (film density) — độ dốc (slope) của đoạn thẳng đó gọi là **gamma (γ)**.

**Hàm Gamma:**  

![[Pasted image 20260902123141.png]]

với a, b được chuẩn hóa về khoảng [0,1].

- Nếu γ = 1: quan hệ tuyến tính, không đổi gì.
- Nếu γ > 1 (ví dụ γ=2): ảnh bị tối đi ở vùng giữa (vì a^2 < a khi 0<a<1)
- Nếu γ < 1 (ví dụ γ=0.5, tức γ=1/2): ảnh sáng lên ở vùng giữa

**Ví dụ số cụ thể:** a = 0.5 (mức xám trung bình, tương ứng pixel giá trị 127/255)

- γ=2: b = 0.5² = 0.25 → ảnh tối hơn nhiều
- γ=0.5: b = 0.5^0.5 ≈ 0.707 → ảnh sáng hơn

**Tại sao cần correction (hiệu chỉnh)?** Camera có gamma riêng γ_c (ví dụ 1.3), màn hình có gamma riêng γ_m (ví dụ 2.6). Nếu không hiệu chỉnh, ảnh chụp ra sẽ không hiển thị đúng màu/sáng trên màn hình khác so với thực tế.

Cách hiệu chỉnh: dùng gamma nghịch đảo γˉ=1/γ\bar\gamma = 1/\gamma γˉ​=1/γ. Camera "làm méo" tín hiệu bằng s=Bγcs = B^{\gamma_c} s=Bγc​ (B là ánh sáng thật). Để lấy lại B đúng, ta áp dụng:  

![[Pasted image 20260902123213.png]]

Đây gọi là **gamma correction** — áp dụng hàm mũ nghịch đảo để "sửa lại" tín hiệu về đúng giá trị gốc.

**Ví dụ pipeline thực tế** (trong slide có hình minh họa): Máy ảnh có γ_c=1.3 → áp dụng hiệu chỉnh 1/1.3 trước khi lưu trữ → xử lý → khi xuất ra màn hình (γ_m=2.6) áp dụng thêm 1/2.6 nữa, để cuối cùng ảnh hiển thị đúng với thực tế mắt người nhìn thấy.

**Code minh họa** (Java/ImageJ, đơn giản hóa):

java

```java
int K = 256;
double GAMMA = 2.8;
int[] lookupTable = new int[K];
for (int a = 0; a < K; a++) {
    double aa = (double) a / 255;        // chuẩn hóa về [0,1]
    double bb = Math.pow(aa, GAMMA);      // áp dụng hàm gamma
    lookupTable[a] = (int) Math.round(bb * 255);  // scale lại về [0,255]
}
// Áp dụng lookupTable cho từng pixel của ảnh
```

Kỹ thuật ở đây là dùng **lookup table (bảng tra cứu)**: tính trước 256 giá trị đầu ra tương ứng 256 giá trị đầu vào có thể có, để không phải tính `pow()` lại cho từng pixel — nhanh hơn rất nhiều.

### 6. Các phép toán điểm khác trong ImageJ

Bảng trong slide liệt kê các hàm đơn giản, mỗi hàm chỉ dựa vào **giá trị của chính pixel đó** (không cần biết hàng xóm):

- `add(p)`: cộng thêm p vào mọi pixel → làm ảnh sáng lên
- `invert(p)`: 255 - I(u,v) → đảo ngược màu (âm bản)
- `multiply(s)`: nhân với hệ số s → tăng/giảm độ tương phản
- `sqr()`, `sqrt()`: bình phương/căn bậc 2

Và các phép toán giữa **2 ảnh** cùng kích thước: ADD (cộng), SUBTRACT (trừ, hay dùng để tìm khác biệt giữa 2 ảnh), MULTIPLY, DIVIDE...

**Ví dụ Alpha Blending** (pha trộn ảnh) — rất trực quan trong slide:  

![[Pasted image 20260902123247.png]]

Đây là công thức pha trộn 2 ảnh: ảnh nền (BG) và ảnh tiền cảnh (FG), với α là tỉ lệ trộn. Nếu α=0.5 thì mỗi ảnh đóng góp 50%. Đây chính là nguyên lý đứng sau việc ghép 2 ảnh chồng lên nhau (ví dụ ghép ảnh chiếc xe vào ảnh thành phố trong slide).

### 7. Điểm mấu chốt: Point Operations "bất lực" ở đâu?

Đây là chuyển tiếp quan trọng của bài: **Point Operations chỉ nhìn từng pixel riêng lẻ**, không biết gì về pixel xung quanh. Vì vậy nó **không thể** làm được:

- **Làm mờ (blur/smoothing)**: cần biết trung bình vùng xung quanh
- **Làm nét (sharpening)**: cần biết sự thay đổi (gradient) giữa các pixel lân cận
- Tạo hiệu ứng nghệ thuật (như ảnh con hoẵng bị "chấm bi" trong slide)

→ Đây là lý do cần **Spatial Filter (bộ lọc không gian)**.

### 8. Spatial Filter (Bộ lọc không gian) là gì?

**Định nghĩa:** Một phép toán ảnh kết hợp giá trị của một pixel với giá trị của **các pixel lân cận** để tính ra giá trị mới.

**Ví dụ dễ hiểu nhất — Mean Filter (bộ lọc trung bình), dùng để làm mờ:**

Xét vùng lân cận 3×3 xung quanh 1 pixel (pixel trung tâm + 8 pixel hàng xóm = 9 pixel). Ta thay pixel trung tâm bằng **trung bình cộng** của 9 pixel đó.

Ví dụ số cụ thể từ slide:

```
104  100  108
99   106   98
95    90   85
```

Pixel trung tâm đang có giá trị 106. Sau khi lọc:  

![[Pasted image 20260902123317.png]]

Vậy pixel 106 sẽ được thay bằng 98. **Việc này lặp lại cho MỌI pixel trong ảnh** → kết quả là ảnh bị mờ đi (các chi tiết nhỏ, nhiễu bị san phẳng bớt).

### 9. Filter Matrix / Filter Mask (Ma trận lọc)

Người ta biểu diễn bộ lọc trung bình 3×3 dưới dạng ma trận:  

![[Pasted image 20260902123337.png]]​

Ma trận này gọi là **filter mask H(i,j)**. Vị trí trung tâm của ma trận (0,0) gọi là **"hot spot"** — nó sẽ được đặt trùng lên vị trí pixel đang xét trong ảnh.

### 10. Convolution (Tích chập) — Cách áp dụng filter

Đây là công thức toán học tổng quát cho việc áp dụng filter:  

![[Pasted image 20260902123354.png]]

**Giải thích từng bước như quy trình:**

1. Đặt ma trận H sao cho tâm (hot spot) trùng vị trí pixel (u,v) đang xét trong ảnh
2. Nhân từng hệ số H(i,j) với giá trị pixel tương ứng bên dưới nó I(u+i, v+j)
3. Cộng tất cả kết quả lại → đó là giá trị pixel mới I'(u,v)
4. Di chuyển ma trận sang vị trí tiếp theo, lặp lại cho toàn bộ ảnh

Đây chính là phép **convolution (tích chập)** nổi tiếng — nền tảng của cả xử lý ảnh cổ điển lẫn Convolutional Neural Network (CNN) trong AI sau này (điều này chắc rất liên quan tới hướng AI integration mà bạn đang theo đuổi).

**Lưu ý kỹ thuật quan trọng:** Không được ghi kết quả mới đè lên ảnh gốc ngay trong lúc đang tính, vì các pixel sau sẽ cần dùng giá trị **gốc** của pixel lân cận (chưa bị filter), không phải giá trị đã bị thay đổi. Vì vậy luôn cần: (1) tạo ảnh trung gian/bản sao để lưu kết quả, hoặc (2) copy ảnh gốc ra làm nguồn đọc, rồi ghi kết quả vào ảnh gốc.

**Code minh họa (từ slide, sát với thực tế):**

java

```java
public void run(ImageProcessor orig) {
    int w = orig.getWidth();
    int h = orig.getHeight();
    ImageProcessor copy = orig.duplicate(); // QUAN TRỌNG: copy làm nguồn đọc

    for (int v = 1; v <= h-2; v++) {
        for (int u = 1; u <= w-2; u++) {
            int sum = 0;
            for (int j = -1; j <= 1; j++) {
                for (int i = -1; i <= 1; i++) {
                    int p = copy.getPixel(u+i, v+j); // đọc từ bản copy
                    sum = sum + p;
                }
            }
            int q = (int) Math.round(sum/9.0);
            orig.putPixel(u, v, q); // ghi kết quả vào ảnh gốc
        }
    }
}
```

**Vùng biên (Computation Range):** Với filter kích thước (2K+1)×(2L+1), filter chỉ áp dụng được ở những vị trí mà toàn bộ ma trận filter còn nằm gọn trong ảnh — tức không áp dụng được ở sát mép ảnh (vì sẽ "thò" ra ngoài, không có pixel để lấy).

### 11. Weighted Smoothing Filter (Lọc trung bình có trọng số)

Thay vì cho 9 pixel trọng số bằng nhau (1/9 mỗi cái), ta có thể cho pixel gần trung tâm trọng số lớn hơn:  

![[Pasted image 20260902123420.png]]​

(tổng các hệ số = 16, để giá trị pixel không bị phóng đại sáng/tối)

Đây là dạng đơn giản của **Gaussian filter** — pixel càng gần trung tâm càng quan trọng, giống hình chuông (bell-shaped).

### 12. Gaussian Filter

Công thức toán:  

![[Pasted image 20260902123443.png]]​

- **r**: khoảng cách từ tâm đến điểm đang xét
- **σ (sigma)**: độ rộng của hình chuông — σ càng lớn, ảnh càng bị mờ nhiều hơn (vì trọng số phân tán rộng hơn, lấy trung bình trên vùng lớn hơn)

Slide cho ví dụ trực quan rất tốt: ảnh người mẫu gốc → lọc 7×7 (mờ nhẹ) → 15×15 (mờ vừa) → 41×41 (mờ rất nặng, gần như mất hình dạng). **Kích thước filter càng lớn → mờ càng mạnh.**

### 13. Integer Coefficients (Hệ số nguyên) — tối ưu tính toán

Thay vì dùng số thực (floating point) như 0.075, 0.125, 0.2 (tốn tài nguyên tính toán hơn), người ta nhân tất cả lên thành số nguyên rồi chia 1 lần ở cuối:  

![[Pasted image 20260902123505.png]]​

Trong Photoshop, đây chính là ô "Scale" (hệ số chia) và "Offset" (nếu kết quả âm thì cộng thêm để đưa về khoảng hiển thị được [0,255]):  

![[Pasted image 20260902123525.png]]

### 14. Hai loại Linear Filter chính

**a) Smoothing filters (làm mờ)** — tất cả hệ số đều dương (+ve): Box filter (trung bình đều), Gaussian filter (trung bình có trọng số hình chuông). Dùng để: giảm nhiễu, làm mượt ảnh.

**b) Difference filters (làm nét / phát hiện biên)** — có cả hệ số dương và âm: ví dụ **Laplacian filter**:  

![[Pasted image 20260902123545.png]]​

**Ý tưởng của Laplacian:** hệ số trung tâm rất lớn (dương, 16), xung quanh âm. Nghĩa là: kết quả = (giá trị pixel trung tâm × 16) − (tổng các pixel xung quanh có trọng số). Nếu vùng đó **đồng nhất** (mọi pixel gần bằng nhau), kết quả ≈ 0. Nếu có **sự thay đổi đột ngột** (biên/cạnh, ví dụ ranh giới giữa vật thể sáng và nền tối), kết quả sẽ lớn (khác 0 nhiều) → đây chính là cách phát hiện biên (edge detection) và làm ảnh sắc nét hơn.

### 15. Tính chất toán học của Convolution

- **Commutativity (giao hoán):** I_H = H_I — chập ảnh với filter hay filter với ảnh đều ra cùng kết quả
- **Linearity (tuyến tính):** (s·I)_H = s·(I_H) — nhân ảnh với hằng số trước hay sau khi lọc đều như nhau; (I₁+I₂)_H = (I₁_H)+(I₂*H) — cộng 2 ảnh rồi lọc = lọc từng ảnh rồi cộng lại
- **Associativity (kết hợp):** A*(B_C) = (A_B)*C — thứ tự áp dụng nhiều filter liên tiếp không quan trọng, kết quả như nhau

Tính chất này rất hữu ích trong thực hành: ví dụ nếu bạn cần áp dụng 2 bộ lọc Gaussian liên tiếp, bạn có thể "gộp" (kết hợp) 2 ma trận filter lại thành 1 ma trận duy nhất trước, rồi chỉ chạy convolution 1 lần — nhanh hơn nhiều so với chạy 2 lần riêng biệt.