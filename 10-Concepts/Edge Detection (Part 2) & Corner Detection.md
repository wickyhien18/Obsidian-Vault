---
created: 2026-09-27
Hub: "[[MOC - Image & Processing]]"
status: seed
---

> [!summary] Tóm tắt nhanh
> -
> -
> -

# PHẦN 1: EDGE DETECTION (Phát hiện cạnh)

## 1.1. Recall: Edge Detection (Nhắc lại: Phát hiện cạnh)

**Nội dung slide:** Edge detection là bài toán xử lý ảnh nhằm tìm ra các cạnh (edges) và đường viền (contours) trong ảnh. Slide nhấn mạnh rằng cạnh quan trọng đến mức thị giác con người có thể "tái tạo lại" toàn bộ vật thể chỉ từ các đường viền của nó (ảnh máy bay chuyển thành bản vẽ nét).

**Ví dụ đời thường:** Hãy nghĩ đến việc bạn nhận ra một người bạn thân chỉ qua bóng dáng (silhouette) của họ trong bóng tối — không cần thấy màu sắc, kết cấu da, chỉ cần đường viền cơ thể là đủ để nhận diện. Đó chính là lý do cạnh (edge) mang lượng thông tin cực kỳ cô đọng: ta có thể vứt bỏ 90% dữ liệu ảnh (màu sắc, độ sáng chi tiết) mà vẫn giữ được "cái hồn" của bức ảnh.

## 1.2. Recall: Characteristics of an Edge (Đặc điểm của một cạnh)

**Nội dung:** Cạnh thực tế (không lý tưởng) là một hàm bậc thang bị làm mờ nhẹ (slightly blurred step function). Cạnh được đặc trưng bởi giá trị lớn của đạo hàm bậc nhất: f'(x) = df/dx.

- Dốc lên (rising slope) → đạo hàm bậc nhất dương và lớn
- Dốc xuống (falling slope) → đạo hàm bậc nhất âm và lớn

**Ví dụ đời thường:** Hãy tưởng tượng bạn đang lái xe và đo độ dốc đường bằng cảm giác ở chân ga. Đường bằng phẳng → độ dốc = 0. Đường lên dốc → độ dốc dương lớn. Đường xuống dốc → độ dốc âm lớn. Cạnh trong ảnh cũng vậy: nó không phải một "bước nhảy" tức thời như bậc thang lý tưởng, mà giống như một con dốc thoải — và đạo hàm (slope) chính là công cụ đo "độ dốc" của sự thay đổi độ sáng.

## 1.3. Recall: Image Gradient (Gradient ảnh)

**Nội dung:** Ảnh là hàm rời rạc 2 chiều. Ta tính đạo hàm theo hai hướng ngang (u) và dọc (v):  

$$\nabla I(u,v) = \begin{bmatrix} \partial I/\partial u \\ \partial I/\partial v \end{bmatrix}$$

Độ lớn gradient (gradient magnitude):  

$$|\nabla I|(u,v) = \sqrt{(\partial I/\partial u)^2 + (\partial I/\partial v)^2}$$

Độ lớn này **bất biến khi ảnh xoay** (rotation invariant), nên dùng được cho edge detection dù vật thể nghiêng theo hướng nào.

**Ví dụ đời thường:** Gradient giống như "mũi tên la bàn chỉ hướng dốc nhất" trên một ngọn đồi. Nếu bạn đứng trên sườn đồi và thả một quả bóng, nó sẽ lăn theo đúng hướng gradient (hướng dốc nhất). Độ lớn của gradient là "độ dốc" tại điểm đó — dốc đứng thì độ lớn cao, đất bằng thì độ lớn gần 0. Việc "bất biến khi xoay" nghĩa là dù bạn xoay tấm bản đồ ngọn đồi theo hướng nào, độ dốc tại một điểm cụ thể vẫn không đổi — chỉ có hướng chỉ của mũi tên thay đổi thôi.

## 1.4. Recall: Gradient-Based Edge Detection (Phát hiện cạnh dựa trên gradient)

**Quy trình 3 bước:**

1. Tính đạo hàm ảnh bằng convolution (phép chập): Dx = Hx * I, Dy = Hy * I
2. Tính biên độ cạnh (edge magnitude): E(u,v) = √(Dx² + Dy²)
3. Tính hướng cạnh (edge direction): Φ(u,v) = arctan(Dy/Dx)

**Ví dụ đời thường:** Convolution (phép chập) giống như việc bạn dùng một "cái khuôn" nhỏ (ví dụ khuôn 3×3) trượt qua từng vùng nhỏ của ảnh, mỗi lần dừng lại nhân các giá trị lân cận với các trọng số trong khuôn rồi cộng lại — giống như dùng một "công thức trộn" cố định để pha một ly nước từ vài giọt nguyên liệu xung quanh mỗi điểm. Kết quả cho biết "độ thay đổi cục bộ" tại điểm đó.

---

# PHẦN 2: CÁC TOÁN TỬ CẠNH KHÁC & CANNY

## 2.1. Other Edge Operators (Các toán tử cạnh khác)

**Vấn đề với đạo hàm bậc nhất:**

- Cường độ đáp ứng của cạnh tỉ lệ thuận với độ dốc của sự chuyển tiếp cường độ ảnh
- Cạnh có thể khó định vị chính xác

**Giải pháp:** Dùng đạo hàm bậc hai — cạnh tương ứng với **điểm giao 0 (zero crossing)** của đạo hàm bậc hai. Nhưng đạo hàm bậc hai khuếch đại nhiễu ảnh, nên cần làm mượt (pre-smoothing) trước.

**Ví dụ đời thường:** Đạo hàm bậc nhất giống như hỏi "tôi đang đi lên dốc hay xuống dốc, và dốc đến mức nào?" Đạo hàm bậc hai giống như hỏi "con dốc này đang bắt đầu cong hay đang thẳng?" — giống như khi bạn lái xe và cảm nhận lúc nào bánh xe bắt đầu "nghiêng" vào khúc cua (đó là lúc độ cong thay đổi mạnh nhất), chứ không phải hỏi bạn đang cua gắt hay nhẹ.

## 2.2. Edges at Different Scales (đã giải thích ở lượt trước)

Tóm tắt ngắn: cạnh tồn tại ở nhiều tỷ lệ khác nhau (mờ/rộng hay sắc/hẹp), và hệ thị giác con người có thể "nối liền" cạnh qua vùng tương phản yếu — điều mà toán tử đơn giản không làm được. Giải pháp: kỹ thuật phân cấp (hierarchical/pyramid) — áp dụng edge detection ở nhiều tỷ lệ tại mỗi điểm ảnh, rồi chọn tỷ lệ chiếm ưu thế.

## 2.3. Canny Edge Detector

**Mục tiêu của Canny (3 tiêu chí):**

1. Giảm thiểu số điểm cạnh giả (false edge points)
2. Định vị cạnh tốt (good localization)
3. Chỉ một đường đánh dấu duy nhất trên mỗi cạnh (single mark per edge)

**Bản chất:** Vẫn dựa trên gradient, kết hợp với zero-crossing của đạo hàm bậc hai. Trong thực tế, thường triển khai ở **một tỷ lệ duy nhất** (single scale) với tham số σ điều chỉnh được (bán kính làm mượt).

**Ví dụ minh họa trong slide:** So sánh kết quả với σ = 1.0, 2.0, 4.0, 8.0, 16.0 — σ nhỏ giữ nhiều chi tiết (nhưng nhiều nhiễu), σ lớn cho đường viền mượt hơn nhưng mất chi tiết nhỏ.

**Ví dụ đời thường:** σ giống như "độ cận thị" của bạn khi nhìn một bức tranh treo tường. Đứng sát và nhìn kỹ (σ nhỏ) bạn thấy từng nét cọ, từng hạt bụi — quá nhiều chi tiết vụn vặt. Lùi ra xa (σ lớn) bạn chỉ thấy các hình khối lớn, mượt mà, bỏ qua chi tiết nhỏ. Canny cho bạn "nút vặn" để chọn khoảng cách đứng nhìn phù hợp.

## 2.4. From Edges to Contours (Từ cạnh đến đường viền)

**Contour following:** Bắt đầu từ điểm có cường độ cạnh mạnh, đi theo cạnh liên tục cho đến khi 2 đầu gặp nhau tạo thành đường khép kín.

**3 trở ngại chính:**

- Cạnh có thể kết thúc ở vùng gradient gần như biến mất
- Cạnh giao nhau gây nhập nhằng (ambiguity)
- Cạnh có thể phân nhánh thành nhiều hướng

**Ví dụ đời thường:** Giống như đi theo dấu chân trên bãi biển để tìm đường một người đã đi qua. Nếu dấu chân bị sóng xóa mất một đoạn (mất gradient), bạn không biết đi tiếp hướng nào. Nếu có hai bộ dấu chân giao nhau (cạnh giao nhau), bạn không chắc nên đi theo bộ nào. Nếu dấu chân rẽ thành 3 hướng ở một điểm (phân nhánh), bạn phải chọn.

## 2.5. Edge Maps (Bản đồ cạnh)

**Nội dung:** Sau khi tính cường độ cạnh, ta cần quyết định nhị phân "điểm này có phải là cạnh hợp lệ không." Cách phổ biến nhất: áp threshold (ngưỡng cố định hoặc thích nghi) lên cường độ cạnh. Trong thực tế, edge map hiếm khi liền mạch hoàn hảo — thường có các đoạn cạnh đứt rời tại những vị trí cường độ cạnh không đủ mạnh.

**Ví dụ đời thường:** Giống như vẽ lại đường viền bản đồ quốc gia chỉ dựa trên ảnh vệ tinh — có những đoạn biên giới rõ ràng (sông, núi), nhưng có đoạn mờ nhạt (đồng bằng phẳng) khiến đường vẽ bị đứt quãng, phải "đoán" nối lại bằng tay hoặc bằng luật bổ sung.

---

# PHẦN 3: LÀM SẮC NÉT ẢNH (IMAGE SHARPENING)

## 3.1. Image Sharpening — Tổng quan

**Nội dung:** Việc scan hoặc chụp ảnh có thể gây ra hiện tượng mờ (blurring). Sharpening giảm hiệu ứng mờ này bằng cách khuếch đại các thành phần tần số cao — mà tần số cao xảy ra tại các cạnh. Hai phương pháp chính: dùng bộ lọc Laplace, và Unsharp Masking.

**Ví dụ đời thường:** Giống như khi bạn chỉnh độ "Clarity" hoặc "Sharpness" trên điện thoại — ảnh chụp hơi mờ (do tay run hoặc lấy nét chưa hoàn hảo) được "cứu" phần nào bằng cách làm nổi bật ranh giới các vật thể, dù không thể khôi phục lại chi tiết đã mất thật sự.

## 3.2. Edge Sharpening using Laplace Filter (Làm sắc nét dùng bộ lọc Laplace)

**Công thức:** Ǐ(x) = f(x) − w·f''(x), trong đó f''(x) là đạo hàm bậc hai của cường độ ảnh, w là trọng số.

**Cách hoạt động:** Đầu tiên áp bộ lọc Laplace lên ảnh I, sau đó trừ đi một phần kết quả đó khỏi ảnh gốc: Ǐ ← I − w·(H^L * I)

## 3.3. Laplace Operator / Laplacian Operator (đã giải thích chi tiết ở các lượt trước — xem lại notebook đã tạo)

Tóm tắt công thức chính:  

$$(\nabla^2 f)(x,y) = \frac{\partial^2 f}{\partial^2 x} + \frac{\partial^2 f}{\partial^2 y}$$

Xấp xỉ số:  

$$\nabla^2 f(x,y) = [f(x+1,y)+f(x-1,y)+f(x,y+1)+f(x,y-1)] - 4f(x,y)$$

Kernel: `[[0,1,0],[1,-4,1],[0,1,0]]`, khả tách thành `Hx = [1,-2,1]` và `Hy = [1,-2,1]ᵀ`.

**Kết quả trên ảnh synthetic (hình tròn mờ):** Đạo hàm bậc hai theo x cho ra 2 vòng cung sáng-tối ở trái-phải hình tròn; theo y cho ra 2 vòng cung ở trên-dưới; kết hợp lại (Laplace filter) cho ra một vòng tròn viền sáng-tối bao quanh toàn bộ hình.

## 3.4. Unsharp Masking (USM)

**Ý tưởng:** Kết hợp ảnh gốc với phiên bản làm mờ (blurred) của chính nó để tạo hiệu ứng sắc nét — thay vì dùng Laplacian trực tiếp.

**Sơ đồ khối:** Original → (nhánh 1: giữ nguyên) và (nhánh 2: Blur với low-pass filter → Scale với k<1) → Subtract (nhánh 1 trừ nhánh 2) → Scale for display

**Các bước toán học:**

1. Lấy ảnh gốc trừ phiên bản làm mờ Gaussian để được "mặt nạ cạnh tăng cường" (enhanced edge mask): M ← I − (I*H̃) = I − Ĩ
2. Cộng mặt nạ vào ảnh gốc với trọng số a: Ǐ ← I + a·M
3. Kết hợp lại: Ǐ ← I + a·(I − Ĩ) = (1+a)·I − a·Ĩ

**Ưu điểm của USM so với Laplace filter:**

- Ít nhạy nhiễu hơn nhờ bước làm mượt (smoothing)
- Kiểm soát tốt hơn qua 2 tham số σ (độ mượt) và a (cường độ tăng cường)

**Ví dụ đời thường:** Đây chính xác là kỹ thuật đứng sau nút "Unsharp Mask" trong Photoshop từ thập niên 1990 (tên gọi bắt nguồn từ kỹ thuật phòng tối truyền thống, nơi nhiếp ảnh gia dùng một bản âm bản mờ chồng lên bản gốc để tăng độ tương phản viền). Hãy tưởng tượng bạn có một tấm ảnh và một "bản sao mờ nhòe" của nó. Nếu bạn lấy ảnh gốc trừ đi bản mờ đó, phần còn lại chính là "những gì bị mất khi làm mờ" — tức là chính các cạnh và chi tiết. Cộng phần "chi tiết bị mất" đó trở lại vào ảnh gốc (với trọng số a) sẽ khiến các cạnh nổi bật hơn.

**Ví dụ code minh họa (ImageJ, class `ij.plugin.filter.UnsharpMask`):** slide cho code Java thực hiện đúng công thức trên — chuyển ảnh sang FloatProcessor, tạo bản sao làm mờ bằng Gaussian kernel, rồi thực hiện phép trừ có trọng số `I.multiply(1+a)`, `J.multiply(a)`, `I.copyBits(J, ..., SUBTRACT)`.

---

# PHẦN 4: CORNER DETECTION (Phát hiện góc)

## 4.1. Why Corner Detection? (Tại sao cần phát hiện góc?)

**Nội dung:** Góc (corner) là đặc trưng bền vững (robust), dùng trong thị giác máy tính để khớp cùng một điểm giữa nhiều ảnh (ví dụ: ảnh stereo trái-phải để tính độ sâu 3D).

**Ví dụ đời thường:** Giống như khi bạn chơi trò "tìm điểm khác nhau" giữa hai bức ảnh gần giống nhau — bạn sẽ tự động nhìn vào các góc rõ ràng (góc bàn, góc cửa sổ) để làm mốc so sánh, chứ không nhìn vào một mảng tường trắng trơn (vì không biết mảng tường đó ở ảnh này có khớp với điểm nào ở ảnh kia).

## 4.2. Motivation: Patch Matching (Động lực: Khớp mảng ảnh)

**Nội dung:** Xét các ô vuông nhỏ (patches) cùng kích thước trong cả hai ảnh. Bài toán: tìm patch giống nhất trong ảnh thứ hai ứng với một patch cho trước ở ảnh thứ nhất.

## 4.3. Some Patches Better Than Others (Một số patch tốt hơn các patch khác)

**Nội dung:** Nên dùng các "patch đặc trưng" (distinctive patches). Ví dụ: không nên dùng patch giống với rất nhiều patch khác trong ảnh 2 (gây nhập nhằng — ambiguous).

**Ví dụ đời thường:** Nếu bạn đang tìm một người bạn trong một đám đông mặc đồng phục giống hệt nhau, bạn không thể chỉ dựa vào "màu áo" để nhận ra họ (vì ai cũng giống ai — đây là patch "mơ hồ"). Bạn cần một đặc điểm riêng biệt như "đeo kính, tóc xoăn" — đó chính là "distinctive patch."

## 4.4. Corners are Robust "Features" (Góc là "đặc trưng" bền vững)

**Nội dung:** Góc là điểm duy nhất (unique), dùng để khớp các patch chứa góc. Góc = giao điểm của các đường viền (junctions of contours). Góc xuất hiện dưới dạng thay đổi cường độ lớn khi nhìn từ các góc nhìn khác nhau (ổn định/duy nhất).

## 4.5. Corner Points: Basic Idea (Ý tưởng cơ bản về điểm góc)

**Ý tưởng:** Nếu patch nằm tại một góc, việc dịch chuyển cửa sổ theo _bất kỳ hướng nào_ đều gây ra thay đổi cường độ lớn.

**Kiểm tra:** Dịch cửa sổ theo nhiều hướng — nếu có thay đổi cường độ lớn ở tất cả các hướng, patch đó là một góc.

**Ví dụ đời thường:** Hãy tưởng tượng bạn đặt một khung ảnh nhỏ (cửa sổ) lên các vị trí khác nhau trên một tấm ảnh có hình chữ L:

- Đặt lên khoảng trống (flat/vùng phẳng): dù bạn dịch khung theo hướng nào, nội dung bên trong khung vẫn y hệt (toàn màu nền) → không đổi.
- Đặt dọc theo một cạnh thẳng của chữ L (edge): dịch dọc theo cạnh thì không đổi gì, nhưng dịch vuông góc với cạnh thì đổi mạnh → chỉ đổi theo MỘT hướng.
- Đặt đúng vào góc của chữ L (corner): dịch theo BẤT KỲ hướng nào cũng đều làm nội dung khung thay đổi mạnh → đây chính là góc thật sự.

## 4.6. Harris Corner Detector: Basic Idea

**3 trường hợp:**

- **Flat region (vùng phẳng):** không có thay đổi cường độ theo mọi hướng
- **Edge (cạnh):** không thay đổi dọc theo hướng cạnh (nhưng thay đổi vuông góc với nó)
- **Corner (góc):** thay đổi cường độ đáng kể theo nhiều hướng

Harris corner detector cung cấp một **phương pháp toán học** để xác định trường hợp nào đang xảy ra tại mỗi điểm ảnh.

## 4.7. Harris Detector: The Mathematics (Toán học của bộ dò Harris)

**Công thức thay đổi cường độ khi dịch [u,v]:**  

$$E(u,v) = \sum_{x,y} w(x,y)[I(x+u,y+v) - I(x,y)]^2$$

Trong đó:

- **w(x,y)** = hàm cửa sổ (window function): có thể là cửa sổ nhị phân (1 bên trong, 0 bên ngoài) hoặc Gaussian (trọng số giảm dần từ tâm ra ngoài)
- **[I(x+u,y+v) − I(x,y)]** = cường độ dịch chuyển trừ cường độ gốc

**Ví dụ đời thường:** Công thức E(u,v) giống như việc bạn "lắc" nhẹ một khung ảnh nhỏ đặt trên một bức tranh, rồi đo xem nội dung bên trong khung thay đổi bao nhiêu (bằng cách so sánh từng pixel trước và sau khi lắc, bình phương sai khác rồi cộng lại). Lắc mà không đổi gì → vùng phẳng. Lắc mà đổi nhiều theo mọi hướng → góc.

## 4.8. Harris Detector: The Intuition (Trực giác)

- Với **vùng gần phẳng**: đại lượng E(u,v) gần bằng 0
- Với **vùng đặc trưng (góc)**: đại lượng này gần như lớn
- **Kết luận: Góc = các patch mà E(u,v) LỚN**

## 4.9. Taylor Series Expansion (Khai triển Taylor)

**Nội dung:** Dùng khai triển Taylor để xấp xỉ f(x+u, y+v):  

$$f(x+u,y+v) = f(x,y) + uf_x(x,y) + vf_y(x,y) + \text{(bậc cao hơn)}$$

Lấy xấp xỉ bậc nhất (bỏ qua các số hạng bậc cao):  

$$E(u,v) \approx \sum [I(x,y) + uI_x + vI_y - I(x,y)]^2$$

**Ví dụ đời thường:** Khai triển Taylor giống như việc bạn dự đoán độ cao của một ngọn đồi tại một điểm gần đó, chỉ dựa trên độ cao hiện tại và độ dốc hiện tại (bỏ qua việc đồi có thể cong lên cong xuống phức tạp ở xa hơn) — một phép "xấp xỉ tuyến tính cục bộ," hữu ích khi bước dịch chuyển [u,v] nhỏ.

## 4.10. Harris Corner Derivation (Suy diễn công thức Harris)

Khai triển tiếp:  

$$E(u,v) \approx \sum u^2I_x^2 + 2uvI_xI_y + v^2I_y^2$$

Viết lại dưới dạng ma trận:  

$$E(u,v) = [u\ v]\left(\sum \begin{bmatrix} I_x^2 & I_xI_y \\ I_xI_y & I_y^2 \end{bmatrix}\right)\begin{bmatrix} u \\ v \end{bmatrix}$$

## 4.11. Harris Corner Detector — Ma trận M

Với dịch chuyển nhỏ [u,v]:  

$$E(u,v) \cong [u,v]\ M \begin{bmatrix} u \\ v \end{bmatrix}$$

trong đó M là **ma trận cấu trúc (structure matrix)** 2×2:  

$$M = \sum_{x,y} w(x,y)\begin{bmatrix} I_x^2 & I_xI_y \\ I_xI_y & I_y^2 \end{bmatrix}$$

Có thể viết M = [[A, C], [C, B]] với A = Ix², B = Iy², C = Ix·Iy.

**Ví dụ đời thường:** Ma trận M giống như một "bảng tóm tắt" chứa toàn bộ thông tin về việc độ sáng thay đổi theo hướng nào tại một vùng nhỏ — giống như một "la bàn thống kê" tổng hợp mọi hướng gradient trong vùng đó thành 3 con số duy nhất (A, B, C).

## 4.12. Harris Corner Detector — Eigenvalues (Trị riêng)

Làm mượt A, B, C riêng biệt bằng bộ lọc Gaussian tuyến tính → được M̄. Vì M̄ đối xứng, có thể chéo hóa (diagonalize):  

$$\bar{M}' = \begin{bmatrix} \lambda_1 & 0 \\ 0 & \lambda_2 \end{bmatrix}$$

với λ1, λ2 là **trị riêng (eigenvalues)** của M̄:  

$$\lambda_{1,2} = \frac{trace(\bar{M})}{2} \pm \sqrt{\left(\frac{trace(\bar{M})}{2}\right)^2 - det(\bar{M})}$$

**Ví dụ đời thường:** Eigenvalue giống như việc bạn "vẽ một hình ellipse (hình bầu dục)" bao quanh đám mây điểm gradient (Ix, Iy) tại một vùng ảnh. Nếu ellipse tròn và to (λ1 ≈ λ2 lớn) → góc. Nếu ellipse dẹt dài theo một trục (λ1 lớn, λ2 nhỏ) → cạnh. Nếu ellipse nhỏ xíu (λ1 ≈ λ2 ≈ 0) → vùng phẳng. Đây chính xác là nội dung slide "Plotting Derivatives as 2D Points" và "Fitting Ellipse to Each Set of Points" minh họa bằng hình vẽ scatter plot.

## 4.13. Classification of Eigenvalues (Phân loại theo trị riêng)

Bảng phân loại theo (λ1, λ2):

- **λ1, λ2 nhỏ** → "Flat" region (E gần như không đổi theo mọi hướng)
- **λ2 ≫ λ1** hoặc **λ1 ≫ λ2** → "Edge" (chỉ đổi mạnh theo 1 hướng)
- **λ1 và λ2 đều lớn, λ1 ≈ λ2** → "Corner" (E tăng theo mọi hướng)

**Đặc điểm quan trọng:**

- Vùng phẳng hoàn toàn: M̄ = 0 → λ1 = λ2 = 0
- Với ramp lý tưởng: λ1 > 0, λ2 > 0
- Eigenvector tương ứng biểu diễn **hướng của cạnh** (edge orientation)
- Một góc thật sự cần: cạnh mạnh theo hướng chính (ứng với λ lớn hơn) VÀ một cạnh khác vuông góc với nó (ứng với λ nhỏ hơn)

## 4.14. Harris Corner Detector — Hàm phản hồi Q(u,v)

Vì tính eigenvalue trực tiếp tốn kém, Harris đề xuất hàm phản hồi thực dụng hơn (không cần tính căn bậc hai):  

$$Q(u,v) = det(\bar{M}) - \alpha \cdot (trace(\bar{M}))^2 = (\bar{A}\bar{B} - \bar{C}^2) - \alpha \cdot (\bar{A}+\bar{B})^2$$

Ghi nhớ: det(M) = λ1·λ2, trace(M) = λ1+λ2

**Tham số α:** thường cố định trong khoảng 0.04–0.06 (tối đa 0.25). α càng lớn → detector càng kém nhạy (phát hiện ít góc hơn).

## 4.15. Plot of Harris Corner Response Function

Đồ thị R theo (λ1, λ2) — cùng công thức Q(u,v) nhưng viết dưới dạng R = det(M) − k(trace(M))²:

- **R phụ thuộc duy nhất vào các eigenvalue của M**
- **R lớn** → góc (corner)
- **R âm, có độ lớn cao** → cạnh (edge)
- **|R| nhỏ** → vùng phẳng (flat)

**Ví dụ minh họa thực tế (Harris Corner Response Example):** Slide cho 3 kết quả với ngưỡng khác nhau trên cùng một ảnh khung cửa sổ:

- Threshold R < −10000 → chỉ ra các cạnh (đường viền khung cửa)
- Threshold R > 10000 → chỉ ra các góc (điểm giao giữa các thanh khung)
- Threshold −10000 < R < 10000 → vùng "trung tính" (không phải cạnh cũng không phải góc — bề mặt kính và khung phẳng)

## 4.16. Harris Corner Detector — Quy trình chọn điểm góc

Một vị trí (u,v) là ứng viên điểm góc khi Q(u,v) > t_H, với t_H thường nằm trong khoảng 10.000–1.000.000.

Sau khi phát hiện, các góc được sắp xếp giảm dần theo cường độ góc (corner strength). Vì nhiều góc "giả" xuất hiện gần góc thật, cần **duyệt danh sách đã sắp xếp và xóa các góc giả** nằm quá gần góc mạnh hơn (trong bán kính d_min).

**Ví dụ đời thường:** Giống như khi phát hiện đỉnh núi trên bản đồ địa hình — quanh một đỉnh núi thật, có thể có vô số điểm "gần đỉnh" cũng được tính là điểm cao cục bộ (do nhiễu địa hình nhỏ). Ta chỉ giữ lại đỉnh cao nhất trong một bán kính nhất định, loại bỏ các "đỉnh giả" xung quanh nó.

## 4.17. Thuật toán đầy đủ & Cài đặt (Harris Corner Detection Algorithm)

**Bước 1 — Tính hàm phản hồi góc (corner response function):**

1. Prefilter (làm mượt) ảnh gốc: I' ← I * Hp
2. Tính đạo hàm ngang/dọc: Ix ← I'*Hdx, Iy ← I'*Hdy
3. Tính ma trận cấu trúc cục bộ: A=Ix², B=Iy², C=Ix·Iy
4. Làm mờ từng thành phần: Ā=A_Hb, B̄=B_Hb, C̄=C*Hb
5. Tính hàm phản hồi góc: Q ← (Ā·B̄−C̄²) − α·(Ā+B̄)²

**Bước 2 — Thu thập điểm góc:**

1. Với mọi tọa độ (u,v): nếu Q(u,v) > t_H VÀ Q(u,v) là cực đại cục bộ (local max) → tạo điểm góc mới
2. Sắp xếp các góc theo q giảm dần
3. Dọn dẹp: loại bỏ các góc yếu nằm gần góc mạnh hơn (trong bán kính d_min)

**Tham số cụ thể (theo Burger & Burge):**

- Prefilter: Hp = [2 5 2]/9 (tách trục x-y)
- Gradient filter: Hdx = [−0.453014, 0, 0.453014]
- Blur filter: Hb = [1,6,15,20,15,6,1]/64
- α mặc định: 0.05
- t_H mặc định: 25.000
- d_min: 10 pixel

**Ví dụ minh họa trên ảnh synthetic (tam giác + hình chữ nhật):** Slide cho thấy A = Ix² (sáng ở các cạnh đứng), B = Iy² (sáng ở các cạnh ngang), C = Ix·Iy (chỉ khác 0 ở vùng có cả hai đạo hàm), Q(u,v) (sáng nhất tại các góc thật), và cuối cùng "detected corners" đánh dấu chính xác các góc của hình chữ nhật và tam giác.

**Ví dụ trên ảnh thật (tòa nhà):** Slide cho thấy "before thresholding" có rất nhiều điểm góc dày đặc quanh mái nhà, khung cửa sổ (nhiễu), còn "after thresholding" chỉ giữ lại các góc thật sự tách biệt, rõ ràng.

---

 NỘI DUNG CÒN THIẾU CẦN BỔ SUNG

## 1. Non-Maximum Suppression đầy đủ trong Canny

Slide chỉ nhắc "single mark on each edge" nhưng không giải thích cơ chế: tại mỗi pixel, so sánh cường độ gradient với 2 pixel lân cận **theo đúng hướng gradient** (không phải theo 8 hướng cố định) — nếu pixel hiện tại không phải cực đại cục bộ theo hướng đó, nó bị loại bỏ (đặt về 0). Đây là bước làm mỏng cạnh (thinning) then chốt của Canny.

**Ví dụ đời thường:** Giống như việc đo đỉnh của một dãy núi bằng cách chỉ giữ lại điểm cao nhất trên mỗi mặt cắt ngang thẳng góc với sườn núi — thay vì giữ cả một dải rộng các điểm "khá cao," ta chỉ giữ đúng đường sống núi (ridge line) mỏng manh.

## 2. Hysteresis Thresholding (ngưỡng trễ) trong Canny

Canny dùng **2 ngưỡng**: ngưỡng cao (t_high) và ngưỡng thấp (t_low). Pixel có gradient > t_high được chấp nhận ngay là cạnh chắc chắn (strong edge). Pixel có gradient nằm giữa t_low và t_high chỉ được chấp nhận nếu nó **kết nối** với một strong edge. Pixel dưới t_low bị loại bỏ hoàn toàn.

**Ví dụ đời thường:** Giống như việc xét duyệt hồ sơ xin việc — ứng viên điểm cực cao (> t_high) được nhận ngay. Ứng viên điểm trung bình (giữa t_low và t_high) chỉ được nhận nếu có người bảo lãnh/giới thiệu đáng tin cậy (kết nối với strong edge). Ứng viên điểm quá thấp (< t_low) bị loại thẳng.

## 3. Scale invariance của Harris Corner Detector

Harris corner **bất biến với phép xoay** (rotation-invariant, vì dựa trên eigenvalue của ma trận đối xứng) nhưng **KHÔNG bất biến với thay đổi tỷ lệ** (scale). Nếu ảnh phóng to/thu nhỏ, một góc có thể "biến mất" hoặc bị phát hiện sai vì kích thước cửa sổ w(x,y) cố định.

**Ví dụ đời thường:** Giống như khi bạn nhìn một chiếc bàn từ rất gần — mép bàn trông giống như một đường thẳng (edge) chứ không phải một góc, vì cửa sổ nhìn của bạn quá nhỏ so với độ cong thực tế của góc bàn. Nhưng đứng xa ra, cùng góc bàn đó lại rõ ràng là một góc nhọn.

## 4. Các bộ dò góc mở rộng (không được đề cập)

- **Shi-Tomasi ("Good Features to Track"):** thay vì dùng công thức det(M)−α·trace(M)², dùng trực tiếp **min(λ1, λ2)** > ngưỡng — đơn giản và trực quan hơn, được dùng rộng rãi trong theo dõi đối tượng (object tracking).
- **SIFT, SURF, ORB:** các bộ mô tả đặc trưng (feature descriptors) bất biến cả với tỷ lệ VÀ hướng xoay, giải quyết đúng nhược điểm của Harris đã nêu ở trên.
- **FAST (Features from Accelerated Segment Test):** một thuật toán phát hiện góc cực nhanh, dùng trong các ứng dụng thời gian thực như SLAM (Simultaneous Localization and Mapping) trên robot/drone.

## 5. Sub-pixel accuracy (độ chính xác dưới pixel)

Trong thực tế công nghiệp (ví dụ hiệu chỉnh camera - camera calibration), người ta thường tinh chỉnh vị trí góc phát hiện được xuống độ chính xác dưới 1 pixel (ví dụ 12.34 thay vì chỉ 12) bằng cách khớp một hàm parabol vào các giá trị Q lân cận — điều này không được đề cập trong slide.

## 6. Ứng dụng thực tế cụ thể hơn của Corner Detection

Ngoài stereo matching, corner detection còn dùng trong:

- **Image stitching / panorama:** ghép nhiều ảnh thành ảnh toàn cảnh (các ứng dụng chụp ảnh panorama trên điện thoại)
- **SLAM (robot/drone tự định vị):** robot dùng các góc đặc trưng để nhận biết đã đi qua vị trí nào
- **Camera calibration:** dùng góc của bàn cờ vua (checkerboard) để tính toán thông số nội tại của camera
- **Augmented Reality (AR):** các ứng dụng AR (như Pokemon Go) dùng góc để "khóa" vật thể ảo vào một vị trí thực trong không gian

## 7. So sánh hiệu năng giữa các edge operator

Slide có hình so sánh trực quan Roberts, Prewitt, Sobel, Laplacian of Gaussian, Canny nhưng không phân tích về **tốc độ tính toán** hay **độ nhạy với nhiễu** của từng loại — ví dụ Roberts nhanh nhất nhưng nhạy nhiễu nhất (kernel 2×2 nhỏ), Canny chậm nhất nhưng cho kết quả sạch và mỏng nhất.