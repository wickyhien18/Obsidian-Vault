---
created: 2026-09-02
Hub: "[[MOC - Image & Processing]]"
---

> [!summary] Tóm tắt nhanh
> - Histogram Equalization — kéo giãn histogram ảnh tối/thiếu tương phản ra khắp dải 0–255 bằng công thức dựa trên cumulative histogram
>- Histogram Specification (Piecewise Linear) — ép histogram ảnh theo một đường tham chiếu tùy chọn (vài điểm mốc), dùng nội suy tuyến tính; chỉ dùng được khi đường tham chiếu khả nghịch
>- Histogram Matching — khi ảnh tham chiếu là ảnh thật (không khả nghịch), dò tìm mức xám tương ứng bằng cách so sánh 2 bảng CDF, giúp 2 ảnh khác nguồn trông giống nhau
>- Gamma Correction — hiệu chỉnh quan hệ phi tuyến (lũy thừa) giữa ánh sáng thực và tín hiệu ghi nhận/hiển thị của camera, màn hình
>- Giới hạn của Point Operations: chỉ xử lý từng pixel độc lập → không làm mờ, không làm nét được
>- Spatial Filter / Convolution — tính pixel mới bằng cách kết hợp pixel đó với các pixel lân cận thông qua ma trận lọc (filter mask)
>- Smoothing filters (Box, Gaussian): hệ số toàn dương, dùng làm mờ/khử nhiễu, độ mờ tăng theo kích thước filter (σ)
>- Difference filters (Laplacian): hệ số có âm có dương, dùng phát hiện biên và làm ảnh sắc nét hơn
>- Tính chất toán học của convolution: giao hoán, tuyến tính, kết hợp — cho phép gộp nhiều filter thành một để tính nhanh hơn 

### 1. Nhắc lại: Histogram Equalization (Cân bằng biểu đồ mức xám)

**Ảnh số là gì trước đã:** Một ảnh xám (grayscale) là một lưới các pixel, mỗi pixel có một giá trị độ sáng từ 0 (đen) đến 255 (trắng) — tức 256 mức (K=256).

**Histogram (biểu đồ tần suất)** là biểu đồ đếm xem trong ảnh có bao nhiêu pixel ở mỗi mức sáng. Ví dụ ảnh tối thì histogram dồn hết về phía bên trái (giá trị nhỏ), ảnh sáng thì dồn về bên phải.

**Vấn đề:** Ảnh chụp thiếu sáng — hầu hết pixel đều nằm trong một khoảng hẹp (ví dụ chỉ từ 20 đến 90), khiến ảnh trông "bệt", không có độ tương phản, không nhìn rõ chi tiết. Nhiều ảnh chụp bị tối, thiếu tương phản — histogram co cụm lại một chỗ thay vì trải đều. 

**Giải pháp — Histogram Equalization:** Biến đổi ảnh sao cho histogram "trải đều" ra khắp dải 0-255 (uniform distribution), làm ảnh rõ nét, tương phản tốt hơn. Cách làm là dùng **cumulative histogram** H(i) — nghĩa là đếm "có bao nhiêu pixel có giá trị ≤ i" (cộng dồn).

Công thức trong slide:  

![[Pasted image 20260902122902.png]]

Giải thích từng ký hiệu:

- **a**: giá trị pixel gốc (ví dụ 50)
- **H(a)**: cumulative histogram tại giá trị a — tức tổng số pixel có giá trị ≤ 50
- **M×N**: tổng số pixel của ảnh (chiều rộng × chiều cao)
- **K**: tổng số mức xám tối đa (ví dụ K=256 đối với ảnh 8-bit).

**Ví dụ cụ thể:** Giả sử ảnh có 100 pixel (M×N=100), K=256. Giả sử có 30 pixel có giá trị ≤ 50, tức H(50)=30. Thì:  

![[Pasted image 20260902122959.png]]

Ý nghĩa: "nếu 30% ảnh tối hơn hoặc bằng mức này, thì trong ảnh mới nó cũng nên nằm ở khoảng 30% của thang đo" — nhờ vậy vùng nào có nhiều pixel sẽ được kéo giãn ra nhiều hơn, làm tăng tương phản đúng chỗ cần thiết.

**Lưu ý:** Không bao giờ làm histogram phẳng tuyệt đối được, chỉ làm "phẳng gần nhất có thể" — vì phép này chỉ được phép gộp/di chuyển các cột, không được tạo thêm hay xóa bớt pixel.

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

**Hạn chế:** Cách này chỉ dùng được khi đường tham chiếu **không có đoạn nào bị "phẳng lì" tại 0** (tức không có mức xám nào hoàn toàn không xuất hiện) — nếu có, không lần ngược lại được.

#### Adjusting Linear Distribution Piecewise (thuật toán `PiecewiseLinearHistogram`)

##### Bước chuẩn bị: đường tham chiếu piecewise-linear là gì cho dễ hình dung?

Tưởng tượng bạn vẽ một đường gấp khúc trên biểu đồ, trục ngang là mức xám (0→255), trục dọc là xác suất cộng dồn (0→1). Đường này được định nghĩa bằng vài **điểm mốc**, ví dụ:

![[Pasted image 20260904085926.png]]

Nghĩa là: bạn muốn 30% pixel ảnh mới nằm ở mức ≤ 60, 75% pixel nằm ở mức ≤ 140. Nối 3 đoạn thẳng lại giữa các điểm này là xong đường tham chiếu.

##### Thuật toán chạy như thế nào — từng bước với số liệu cụ thể

Input: `h_A` (histogram ảnh gốc), `L_R` (đường tham chiếu như trên).

**Bước 1:** Tính K = 256, tính **CDF chuẩn hóa** P_A của ảnh gốc — nghĩa là với mỗi mức xám a, tính `P_A(a) = H(a) / (M×N)` (M×N là tổng số pixel).

**Bước 2:** Với mỗi mức xám a từ 0 đến 255, làm như sau:

1. Lấy `b = P_A(a)` (ví dụ pixel a=90 có P_A(90) = 0.5, tức 50% ảnh tối hơn hoặc bằng 90)
2. Nếu `b ≤ q_0` (nhỏ hơn cả điểm mốc đầu) → gán `a' = 0`
3. Nếu `b ≥ 1` → gán `a' = K-1 = 255`
4. Ngược lại: **tìm xem b rơi vào đoạn thẳng nào** trong đường tham chiếu

**Ví dụ cụ thể:** b = 0.5. Nhìn vào các điểm mốc `⟨0,0⟩, ⟨60,0.3⟩, ⟨140,0.75⟩, ⟨255,1⟩`, ta thấy 0.5 nằm giữa 0.3 (tại a=60) và 0.75 (tại a=140) → đoạn thẳng cần dùng là đoạn nối (60, 0.3) và (140, 0.75).

**Nội suy tuyến tính** trên đoạn này (tức tìm điểm a' tương ứng với b=0.5 trên đường thẳng đó):  

a′=an+(b−qn)⋅an+1−anqn+1−qn=60+(0.5−0.3)⋅140−600.75−0.3=60+0.2×177.8≈95.5≈96a' = a_n + (b - q_n) \cdot \frac{a_{n+1}-a_n}{q_{n+1}-q_n} = 60 + (0.5-0.3)\cdot\frac{140-60}{0.75-0.3} = 60 + 0.2 \times 177.8 \approx 95.5 \approx 96a′=an​+(b−qn​)⋅qn+1​−qn​an+1​−an​​=60+(0.5−0.3)⋅0.75−0.3140−60​=60+0.2×177.8≈95.5≈96

Vậy: pixel nào đang có giá trị 90 trong ảnh gốc → sẽ được đổi thành **96** trong ảnh mới.

**Cách tìm đúng đoạn thẳng trong thuật toán (dò từ phải sang trái):** Pseudocode trong slide làm việc này bằng cách bắt đầu từ đoạn cuối cùng (n = N-1), rồi lùi dần về trước (`n = n-1`) miễn là điểm mốc `q_n` vẫn còn lớn hơn `b`. Khi dừng lại, đó chính là đoạn `[a_n, a_{n+1}]` chứa `b`.

**Bước 3:** Lưu `a'` vào một **bảng tra cứu (lookup table)** `f_hs[a] = a'` cho tất cả 256 mức xám — làm 1 lần duy nhất.

**Bước 4:** Dùng bảng tra cứu này để đổi **toàn bộ ảnh** — mỗi pixel chỉ cần tra bảng, không cần tính lại công thức, nên rất nhanh.

**Trường hợp đặc biệt đáng chú ý:** Nếu đường tham chiếu chỉ có 2 điểm `⟨0,0⟩` và `⟨255,1⟩` (tức 1 đường thẳng từ góc dưới trái lên góc trên phải = phân bố đều tuyệt đối), thuật toán này **chính là** phép Histogram Equalization tuyến tính đã học ở phần đầu bài — tức đây là bản tổng quát hóa của phép đó, cho phép chọn hình dạng phân bố mong muốn tùy ý thay vì bắt buộc phải phẳng đều.
### 4. Histogram Matching (Ghép histogram)

Khi ảnh tham chiếu là ảnh thật (không phải đường lý tưởng), nó thường có những mức xám hoàn toàn vắng bóng (0 pixel) → không dùng được cách ở Phần 3.

**Giải pháp:** Với mỗi pixel của ảnh gốc, tính phần trăm cộng dồn của nó, rồi tìm trên ảnh tham chiếu xem mức xám nào có phần trăm cộng dồn **vừa đủ bắt kịp** giá trị đó — gán mức đó làm giá trị mới.

![[Pasted image 20260903090907.png]]
**Ý tưởng đơn giản:** Có 2 ảnh I_A (ảnh cần đổi) và I_R (ảnh tham chiếu). Ta muốn làm cho histogram của I_A giống I_R nhất có thể, bằng cách "khớp" (match) 2 cumulative histogram lại với nhau.

Công thức:  
![[Pasted image 20260902123121.png]]

Nói dễ hiểu: với mỗi pixel giá trị a trong ảnh gốc, tính xác suất cộng dồn P_A(a) của nó (ví dụ = 0.4, tức 40% pixel ảnh gốc tối hơn hoặc bằng a). Sau đó tìm trên đường cong tham chiếu P_R, giá trị j **nhỏ nhất** sao cho P_R(j) ≥ 0.4. Giá trị j đó chính là a' — giá trị mới.

**Ví dụ số:** Nếu P_A(100) = 0.4 (40% pixel ảnh gốc ≤ 100), và trên đường cong tham chiếu ta thấy P_R(90)=0.35, P_R(91)=0.41 → chọn j=91 (vì đây là j nhỏ nhất có P_R(j) ≥ 0.4). Vậy pixel giá trị 100 trong ảnh gốc sẽ đổi thành 91.

**Nhận xét:** Cách này hoạt động tốt nếu 2 ảnh có nội dung/cảnh vật tương tự nhau, còn nếu khác nhau hoàn toàn thì kết quả trông sẽ hơi "giả".

#### Adjusting to a Given Histogram (thuật toán `MatchHistograms`)

##### Vấn đề cần giải quyết trước

Thuật toán A ở trên đòi hỏi đường tham chiếu phải **khả nghịch** — tức là hàm CDF tham chiếu phải luôn tăng dần đều, không có đoạn nào bằng phẳng ở xác suất = 0 (nghĩa là không được có mức xám nào hoàn toàn vắng bóng trong ảnh tham chiếu). Nếu ảnh tham chiếu là **ảnh thật** (không phải đường vẽ tay lý tưởng), gần như chắc chắn sẽ có vài mức xám không xuất hiện pixel nào (p(k)=0) → không nghịch đảo được → thuật toán A "gãy".

Vì vậy cần thuật toán mới: `MatchHistograms` — không cần nghịch đảo tường minh.

##### Thuật toán chạy như thế nào — từng bước với số liệu cụ thể

Input: `h_A` (histogram ảnh cần biến đổi — gọi là "target"), `h_R` (histogram ảnh tham chiếu — gọi là "reference"), 2 ảnh này **có cùng kích thước bảng** (256 mức) nhưng không nhất thiết cùng số lượng pixel.

**Bước 1 & 2:** Tính CDF chuẩn hóa cho cả hai:

- `P_A` = CDF của ảnh target
- `P_R` = CDF của ảnh reference

**Bước 3:** Với mỗi mức xám `a` từ 0 đến 255, thuật toán dò tìm giá trị `j` theo cách sau (đây chính là vòng lặp `repeat...while` trong pseudocode):

```
j ← 255                              (bắt đầu từ mức cao nhất)
lặp:
    f_hs[a] ← j                      (tạm gán a' = j)
    j ← j - 1                        (lùi xuống 1 mức)
cho đến khi: KHÔNG còn thỏa (j ≥ 0 VÀ P_A(a) ≤ P_R(j))
```

Nói cách khác: ta **lùi dần j từ 255 xuống**, và chỉ dừng lại (giữ giá trị `j+1` vừa rồi) ngay khi điều kiện `P_A(a) ≤ P_R(j)` **bắt đầu sai**. Kết quả cuối cùng chính là giá trị `j` **nhỏ nhất** mà vẫn còn thỏa `P_A(a) ≤ P_R(j)`.

**Ví dụ cụ thể bằng số:** Giả sử pixel a=100 trong ảnh target có `P_A(100) = 0.4` (tức 40% ảnh target tối hơn hoặc bằng 100). Ta có bảng CDF của ảnh reference như sau (chỉ trích một đoạn):

|j (mức xám reference)|P_R(j)|
|---|---|
|88|0.33|
|89|0.36|
|90|0.41|
|91|0.44|

Ta cần tìm **j nhỏ nhất** sao cho `0.4 ≤ P_R(j)`. Nhìn bảng: tại j=89, P_R=0.36 < 0.4 (chưa đủ) → tại j=90, P_R=0.41 ≥ 0.4 (đủ rồi). Vậy `j=90` là giá trị nhỏ nhất thỏa điều kiện → **a' = 90**.

Vậy: mọi pixel đang có giá trị 100 trong ảnh target sẽ được đổi thành **90**.

**Trực giác đằng sau con số này:** Ta đang tìm mức xám trên ảnh reference mà tại đó, phần trăm pixel tối hơn-hoặc-bằng nó (trên ảnh reference) **cũng xấp xỉ 40%** — giống hệt tỉ lệ của pixel a=100 trên ảnh target. Nói cách khác: "pixel này ở target đứng thứ 40% từ đáy lên — vậy trên reference, vị trí thứ 40% từ đáy lên tương ứng với mức xám nào? Lấy mức đó."

**Bước 4:** Lưu tất cả các `a'` này thành bảng tra cứu `f_hs`, rồi áp dụng lên toàn bộ ảnh target — mỗi pixel tra bảng 1 lần.
### 5. Gamma Correction (Hiệu chỉnh Gamma)

**Vấn đề gốc:** Camera và màn hình không phản ứng "tuyến tính" với ánh sáng. Ví dụ: ánh sáng vào cảm biến camera tăng gấp đôi, nhưng giá trị pixel ghi lại không tăng gấp đôi — mối quan hệ này là phi tuyến (nonlinear), thường theo dạng hàm mũ.

**Gamma là gì?** Là con số mô tả độ "cong" của quan hệ đó, khái niệm bắt nguồn từ nhiếp ảnh phim analog. Xuất phát từ nhiếp ảnh analog (phim): người ta đo mối quan hệ giữa log(cường độ ánh sáng) và mật độ phim (film density) — độ dốc (slope) của đoạn thẳng đó gọi là **gamma (γ)**.

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

Cách hiệu chỉnh: dùng gamma nghịch đảo γˉ​=1/γ. Camera "làm méo" tín hiệu bằng s=Bγc (B là ánh sáng thật). Để lấy lại B đúng, ta áp dụng:    

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