---
created: 2026-09-19
Hub: "[[MOC - AI & ML]]"
status: seed
---

> [!summary] Tóm tắt nhanh
> - logistic regression = hồi quy tuyến tính + hàm sigmoid để ép đầu ra thành xác suất hợp lệ, từ đó dùng cho bài toán phân loại.
> -
> -
## Định nghĩa

Logistic Regression (hồi quy logistic) là một mô hình học máy dùng để **phân loại** — tức là dự đoán một biến đầu ra thuộc về một trong các nhóm rời rạc (ví dụ: có bệnh/không bệnh, spam/không spam, mua hàng/không mua hàng), dựa trên một hoặc nhiều đặc trưng đầu vào.

Điều thú vị (và hơi gây nhầm lẫn) là tên gọi có chữ "regression" (hồi quy) nhưng nó lại dùng để **phân loại**, không phải để dự đoán một số liên tục như hồi quy tuyến tính.

## Tại sao không dùng hồi quy tuyến tính luôn cho bài toán phân loại?

Đây là câu hỏi cốt lõi. Giả sử bạn muốn dự đoán "khách hàng có mua hàng hay không" (y = 0 hoặc 1) dựa vào thời gian họ ở trên web. Nếu dùng hồi quy tuyến tính bình thường:
![[Pasted image 20260919142800.png]]

thì có hai vấn đề lớn:
1. **Đầu ra không bị giới hạn**: w⋅x+bw\cdot x + b w⋅x+b có thể ra bất kỳ giá trị nào — âm vô cùng đến dương vô cùng (ví dụ -5 hoặc 3,7). Nhưng xác suất "có mua hàng" thì phải nằm trong [0, 1]. Một mô hình dự đoán "xác suất mua hàng là -5" hoặc "3,7" là vô nghĩa.
2. **Đường thẳng không phù hợp với dữ liệu dạng 0/1**: nhãn thật chỉ có 2 giá trị (0 hoặc 1), trong khi đường hồi quy tuyến tính là một đường thẳng liên tục — cố "fit" một đường thẳng qua các điểm 0/1 sẽ cho kết quả rất tệ và nhạy cảm với outlier.
## Giải pháp: "bóp" đầu ra vào [0, 1] bằng hàm sigmoid

Logistic regression vẫn tính z=w⋅x+b như hồi quy tuyến tính (một tổ hợp tuyến tính của đặc trưng), nhưng sau đó đưa zz z qua hàm **sigmoid**:

![[Pasted image 20260919142907.png]]

Hàm này có dạng chữ S, luôn cho ra giá trị trong khoảng (0, 1) dù zz z là số gì:

![[Pasted image 20260919142920.png]]

Nhờ vậy, đầu ra  ![[Pasted image 20260919142953.png]]  có thể diễn giải trực tiếp là **xác suất** ![[Pasted image 20260919143037.png]] Từ xác suất này, ta chọn một ngưỡng (thường là 0,5) để ra quyết định phân loại cuối cùng: nếu p≥0,5 thì dự đoán lớp 1, ngược lại lớp 0.

## Vì sao nó hữu ích trong thực tế

- **Cho ra xác suất, không chỉ nhãn cứng** — bạn biết mô hình "tự tin" đến mức nào (95% chắc là ác tính khác với 51% chắc là ác tính), điều này rất quan trọng trong y tế, tín dụng, v.v. — như bạn thấy ở Bài 1.3 với chẩn đoán ung thư.
- **Dễ diễn giải**: mỗi hệ số wjw_j wj​ có ý nghĩa rõ ràng qua odds ratio ewje^{w_j} ewj​ — cho biết đặc trưng đó làm tăng/giảm "odds" của lớp dương bao nhiêu lần (đúng như Bài 1.1, 1.4 bạn vừa xem).
- **Nhanh, ổn định, là baseline tốt**: dù đơn giản, nó thường hoạt động rất tốt khi ranh giới giữa các lớp gần như tuyến tính (ví dụ trong notebook, Logistic Regression đạt AUC ≈ 0,995 trên Breast Cancer — ngang hoặc hơn cả Random Forest).

**Ví dụ**: Một sàn thương mại điện tử có mô hình z=−3+0,05⋅phút+1,2⋅đã\_mua. Tính xác suất mua hàng của khách A (40 phút, đã từng mua) và khách B (20 phút, chưa từng mua), cùng odds ratio của biến `đã_mua`.

``` Python
import numpy as np
def sigmoid(z):
	return 1 / (1 + np.exp(-z))

z_A = -3 + 0.05 * 40 + 1.2 * 1
p_A = sigmoid(z_A)

z_B = -3 + 0.05 * 20 + 1.2 * 0
p_B = sigmoid(z_B)

OR_da_mua = np.exp(1.2)
```

## Odds Ratio

**Odds** (tỉ số cược) là một cách khác để diễn đạt "khả năng xảy ra", khác với xác suất. Nếu xác suất một sự kiện xảy ra là **P**, thì:
![[Pasted image 20260919144055.png]]

Ví dụ: xác suất mưa ngày mai là p=0,75. Vậy:
![[Pasted image 20260919144136.png]]

Nghĩa là "khả năng mưa gấp 3 lần khả năng không mưa" — đây là cách nói quen thuộc trong cá cược thể thao ("tỉ lệ cược 3-1").

**Lưu ý khác biệt quan trọng:**
- **Xác suất**:  p luôn nằm trong [0, 1]
- **Odds** nằm trong [0,+∞)]  — có thể lớn hơn 1 rất nhiều

**NẾU CHÚNG TA XÉT ![[Pasted image 20260919145447.png|278]] SAU KHI BIẾN ĐỔI TA ĐƯỢC:**
![[Pasted image 20260919145034.png]]

Vế trái là **log-odds** (còn gọi là "logit"). Phương trình trên nói rằng: **log-odds bằng đúng một hàm tuyến tính  wx+b** — giống hệt dạng của hồi quy tuyến tính bình thường, chỉ khác là biến phụ thuộc bây giờ là ln(p/(1−p)) thay vì p trực tiếp.

Đây là lý do người ta nói "logistic regression thích odds hơn xác suất": **không phải vì mô hình cố ý chọn thế**, mà vì bản chất toán học của hàm sigmoid buộc mối quan hệ tuyến tính phải nằm ở log-odds, chứ không nằm ở p p. Cụ thể:

- p liên hệ với z=wx+b qua một hàm **cong** (sigmoid) → tăng x thêm 1 đơn vị thì p tăng bao nhiêu **phụ thuộc vào pp p đang ở đâu** (gần 0, gần 0,5, hay gần 1) — không đơn giản, không cố định.
- Nhưng log-odds thì liên hệ với z=wx+b qua một đường **thẳng** → tăng x thêm 1 đơn vị thì log-odds luôn tăng thêm đúng w, bất kể đang ở mức nào. Đây là mối quan hệ **cố định, đơn giản, dễ diễn giải**.

## Vì sao điều này dẫn đến odds ratio =e^w 

Nếu x tăng từ x0​ lên x0​+1 (ví dụ "đã_mua" từ 0 lên 1):

![[Pasted image 20260919150015.png]]

Vì hệ số ww w **cộng thêm một lượng cố định vào log-odds**, khi bỏ log đi (mũ hóa) thì phép cộng biến thành phép **nhân** e^w vào odds. Đó là toàn bộ nguồn gốc của odds ratio — nó không phải một định nghĩa tùy ý, mà là hệ quả trực tiếp của việc mô hình tuyến tính với log-odds.

Ngược lại, xác suất p không nằm ở vị trí "được cộng w" trong công thức nào cả — nó chỉ là kết quả sau khi đi qua hàm cong sigmoid — nên không có phép nhân cố định nào áp dụng cho pp p.

## Hàm mất mát (Loss Function)

Sau khi có mô hình p=σ(wx+b), ta cần một cách để **đo xem mô hình đang dự đoán tệ đến mức nào**, để từ đó điều chỉnh w,b cho tốt hơn. Đó chính là vai trò của loss function — nó là một con số duy nhất, càng nhỏ càng tốt, đại diện cho "độ sai" của mô hình trên toàn bộ dữ liệu.

### Vì sao không dùng sai số bình phương (MSE) như hồi quy tuyến tính?

Câu hỏi tự nhiên: sao không lấy (y−p)2 trung bình như hồi quy tuyến tính? Về mặt kỹ thuật vẫn tính được, nhưng có vấn đề: khi kết hợp với hàm sigmoid, hàm mất mát MSE trở thành **không lồi** (non-convex) — nó có nhiều đáy cục bộ (local minima), khiến gradient descent dễ bị "kẹt" ở một điểm không tốt nhất thay vì tìm ra điểm tối ưu toàn cục. Ngoài ra, MSE không "trừng phạt" đủ mạnh khi mô hình dự đoán sai một cách rất tự tin

### Log Loss (Binary Cross-Entropy) — giải pháp phù hợp hơn

Thay vào đó, logistic regression dùng **log loss**:

![[Pasted image 20260919154906.png]]
Nhìn có vẻ phức tạp, nhưng thực ra công thức này chỉ đơn giản là "chọn 1 trong 2 nhánh" tùy vào nhãn thật yi​:

- Nếu yi​=1: số hạng (1−yi​)log(1−pi​) biến mất (vì nhân với 0), chỉ còn lại −logpi​
- Nếu yi​=0: số hạng yi​logpi​ biến mất, chỉ còn lại −log(1−pi​)

Tức là:
![[Pasted image 20260919155134.png]]

**Đạo hàm**:
![[Pasted image 20260919162254.png]]
### Vì sao công thức này "khôn ngoan"

Hãy xét trường hợp yi​=1 (nhãn thật là lớp dương), loss =−log(pi​):

- Nếu mô hình dự đoán pi​≈1 (đúng, rất tự tin) → log(1)=0 → loss ≈ 0 ✓ tốt
- Nếu mô hình dự đoán pi​≈0,5 (không chắc chắn) → loss vừa phải
- Nếu mô hình dự đoán pi​≈0 (sai, rất tự tin) → log⁡(pi)→−∞ → loss →+∞ ✗ bị phạt cực nặng

Đây chính là điểm mạnh của log loss: nó **phạt rất nặng** khi mô hình sai mà lại tự tin cao (ví dụ nói "chắc chắn 99% là lành tính" nhưng thực ra là ác tính) — điều này rất hợp lý cho các bài toán như chẩn đoán bệnh, nơi sai lầm tự tin là nguy hiểm nhất. Ngược lại, nếu mô hình "phân vân" (p gần 0,5) thì bị phạt vừa phải dù đúng hay sai.

**Ví dụ**: w=b=0 ban đầu, mọi z=0 nên p=σ(0)=0,5 cho cả 3 điểm dữ liệu. Vậy:
![[Pasted image 20260919155600.png]]

(vì log(0,5) giống nhau ở mọi điểm bất kể yi​ là 0 hay 1, do pi​=0,5 luôn). Đây là giá trị loss "khởi điểm ngây thơ" khi mô hình chưa học gì — và đúng bằng kết quả kiểm tra.

**Tại sao cắt (`clip`) p vào [ε,1−ε]?**

Đây là chi tiết kỹ thuật nhưng quan trọng: nếu pi​ đúng bằng 0 hoặc 1 (do lỗi số học hoặc mô hình quá tự tin), thì log(0) sẽ ra −∞, làm hỏng phép tính (NaN hoặc lỗi). Vì vậy trước khi tính log, ta giới hạn p vào một khoảng an toàn rất sát 0 và 1 (ví dụ [10^{-12}, 1-10^{-12}] ) bằng `np.clip`, để tránh log của 0.

```Python
def log_loss(y, p, eps=1e-12):
	p = np.clip(p, eps, 1 - eps)
	return -np.mean(y * np.log(p) + (1 - y) * np.log(1 - p))

x = np.array([1., 2., 3.])
y = np.array([0., 1., 1.])
w, b, eta = 0.0, 0.0, 0.5

p = sigmoid(w * x + b)

J0 = log_loss(y, p)

grad_w = np.mean((p - y) * x)
grad_b = np.mean(p - y)

w_moi = w - eta * grad_w

b_moi = b - eta * grad_b
```

## Gradient Decent

Gradient descent là một thuật toán **lặp đi lặp lại** để tìm giá trị tham số (w,b) làm cho hàm mất mát J nhỏ nhất, theo đúng 2 bước bạn vừa nói:

1. **Tính đạo hàm riêng** của J theo từng tham số — gọi là gradient. Đạo hàm này cho biết: nếu tăng tham số đó lên một chút, loss sẽ tăng hay giảm, và tăng/giảm nhanh cỡ nào.
2. **Điều chỉnh tham số** theo hướng ngược với gradient, để loss giảm xuống.

### Vì sao lại đi **ngược hướng** gradient?

Đây là điểm quan trọng cần nắm chắc. Gradient chỉ ra hướng mà hàm số **tăng nhanh nhất**. Vì mục tiêu là **giảm** loss, ta phải đi theo hướng ngược lại — đó là lý do công thức luôn có dấu trừ:

![[Pasted image 20260919162851.png]]​

Hình dung trực quan: bạn đang đứng trên một ngọn đồi (đồ thị của hàm loss) trong sương mù, không nhìn thấy toàn cảnh, chỉ cảm nhận được độ dốc dưới chân. Gradient descent giống như việc bạn liên tục nhìn xuống, xác định hướng dốc xuống nhiều nhất tại vị trí hiện tại, rồi bước một bước theo hướng đó. Lặp lại nhiều lần, bạn sẽ dần đi xuống đến đáy thung lũng (điểm loss nhỏ nhất).

### Vai trò của η (learning rate)

η\eta η quyết định **bước đi dài hay ngắn** mỗi lần:

- η quá lớn → bước đi quá dài, có thể "nhảy vọt" qua đáy thung lũng, khiến loss dao động hoặc phân kỳ (tăng mãi thay vì giảm).
- η quá nhỏ → bước đi quá ngắn, hội tụ rất chậm, tốn nhiều vòng lặp mới tới đích.

### Vì sao lặp lại nhiều lần (không chỉ 1 bước)?

Trong Bài 1.2, notebook chỉ yêu cầu **một bước** cập nhật để bạn hiểu cơ chế tính toán. Nhưng trong huấn luyện thực tế, quy trình 2 bước này (tính gradient → cập nhật) được lặp lại hàng trăm hoặc hàng nghìn lần: mỗi lần b w,b mới lại được dùng để tính lại p, tính lại loss, tính lại gradient... cho đến khi gradient gần bằng 0 (nghĩa là đang ở gần đáy, không còn dốc để đi xuống nữa) hoặc đạt số vòng lặp tối đa. Đây chính là những gì `LogisticRegression(max_iter=5000)` trong Bài 1.3 làm ngầm bên trong — `max_iter` chính là giới hạn số vòng lặp gradient descent (hoặc thuật toán tối ưu tương tự) mà sklearn cho phép chạy.

## Train trên dữ liệu thật

### Chuẩn bị dữ liệu
```Python
d = load_breast_cancer()

X_bc = pd.DataFrame(d.data, columns=d.feature_names)

y_bc = pd.Series(1 - d.target, name="ac_tinh") # sklearn mã hóa 0 = ác tính

print(X_bc.shape)

print(y_bc.value_counts().rename({0: "Lành tính", 1: "Ác tính"}))
```

- `load_breast_cancer()`: đây là một bộ dữ liệu **có sẵn** trong sklearn, gồm các phép đo từ ảnh sinh thiết khối u vú (kích thước, độ nhám, độ đối xứng...) và nhãn khối u đó là lành tính hay ác tính. Không cần tự thu thập dữ liệu — sklearn đã đóng gói sẵn.
- `d.data`: mảng numpy chứa các đặc trưng đầu vào (mỗi hàng là 1 bệnh nhân, mỗi cột là 1 đặc trưng như "bán kính trung bình", "độ nhám"...). `d.feature_names`: tên của các cột đó.
- `X_bc = pd.DataFrame(...)`: chuyển từ mảng numpy sang bảng pandas có tên cột rõ ràng, dễ đọc và thao tác hơn.
- `d.target`: nhãn gốc do sklearn quy định — trong bộ dữ liệu này, sklearn mã hóa `0 = ác tính, 1 = lành tính` (ngược với trực giác thông thường).
- `y_bc = pd.Series(1 - d.target, ...)`: đây là mẹo đảo ngược nhãn — vì `1 - 0 = 1` và `1 - 1 = 0`, nên sau phép này, `1 = ác tính, 0 = lành tính` — đúng quy ước dễ hiểu hơn (nhãn 1 = trường hợp "dương tính", "đáng lo ngại" mà ta muốn phát hiện).
- `X_bc.shape`: in ra (số hàng, số cột) — kiểm tra có bao nhiêu bệnh nhân và bao nhiêu đặc trưng.
- `y_bc.value_counts()`: đếm xem có bao nhiêu ca lành tính, bao nhiêu ca ác tính — để biết dữ liệu có bị mất cân bằng lớp hay không.
### Chia train/test

```Python
X_tr, X_te, y_tr, y_te = train_test_split(
	X_bc, y_bc, test_size=0.2, stratify=y_bc, random_state=SEED
)
```

- `train_test_split`: chia dữ liệu thành 2 phần — **tập huấn luyện** (train, dùng để dạy mô hình) và **tập kiểm tra** (test, dùng để đánh giá mô hình trên dữ liệu nó chưa từng thấy).
- `test_size=0.2`: dành 20% dữ liệu cho tập test, 80% còn lại cho train.
- `stratify=y_bc`: đây là điểm quan trọng — đảm bảo tỉ lệ lành tính/ác tính trong cả tập train và tập test **giống hệt** tỉ lệ trong toàn bộ dữ liệu gốc. Nếu không có `stratify`, việc chia ngẫu nhiên có thể vô tình dồn nhiều ca ác tính vào một tập, làm sai lệch đánh giá.
- `random_state=SEED`: đặt "hạt giống ngẫu nhiên" cố định để mỗi lần chạy lại code đều ra kết quả chia dữ liệu **giống hệt nhau** — giúp kết quả có thể tái lập được.
- Hàm trả về 4 phần: `X_tr, y_tr` (đặc trưng và nhãn của tập train), `X_te, y_te` (đặc trưng và nhãn của tập test).
### Chuẩn hóa + Huấn luyện

```Python
model_lr = make_pipeline(StandardScaler(), LogisticRegression(max_iter=5000))
model_lr.fit(X_tr, y_tr)
```

- `StandardScaler()`: một bước **chuẩn hóa** dữ liệu — chuyển mỗi đặc trưng về trung bình 0, độ lệch chuẩn 1. Cần thiết vì các đặc trưng trong bộ dữ liệu này có thang đo rất khác nhau (ví dụ "diện tích" có giá trị hàng trăm, "độ đối xứng" chỉ vài phần trăm) — nếu không chuẩn hóa, logistic regression sẽ bị "thiên vị" các đặc trưng có giá trị lớn.
- `LogisticRegression(max_iter=5000)`: mô hình logistic regression đã học ở các phần trước; `max_iter=5000` là số vòng lặp tối đa cho phép để thuật toán tối ưu (giống gradient descent) hội tụ.
- `make_pipeline(...)`: gộp 2 bước trên thành **một pipeline duy nhất** — nghĩa là khi gọi `.fit()`, dữ liệu sẽ tự động đi qua `StandardScaler` trước, rồi mới đến `LogisticRegression`, theo đúng thứ tự, không cần viết tay từng bước riêng lẻ. Điều này cũng đảm bảo tính nhất quán: khi dự đoán trên dữ liệu mới, nó cũng tự áp dụng đúng phép chuẩn hóa đã học từ tập train.
- `model_lr.fit(X_tr, y_tr)`: đây là lệnh **huấn luyện thực sự** — pipeline học tham số chuẩn hóa (trung bình, độ lệch chuẩn) từ `X_tr`, rồi logistic regression chạy gradient descent (hoặc thuật toán tối ưu tương tự) để tìm w,bw, b w,b tốt nhất, dựa trên `X_tr` (đặc trưng) và `y_tr` (nhãn thật).
### Dự đoán xác suất trên test

```Python
p_te = model_lr.predict_proba(X_te)[:, 1]
```

- `model_lr.predict_proba(X_te)`: dùng mô hình đã huấn luyện để tính xác suất dự đoán trên tập **test** (dữ liệu mô hình chưa từng thấy). Hàm này trả về một mảng 2 cột: cột 0 là xác suất thuộc lớp 0 (lành tính), cột 1 là xác suất thuộc lớp 1 (ác tính) — vì 2 xác suất này luôn cộng lại bằng 1.
- `[:, 1]`: chỉ lấy cột thứ 2 (xác suất ác tính) — đây chính là pp p mà ta quan tâm, giống ký hiệu pp p đã dùng xuyên suốt các bài trước.
### Áp ngưỡng ra nhãn

```Python
yhat_te = (p_te >= 0.5).astype(int)
```

`(p_te >= 0.5).astype(int)`: áp **ngưỡng 0,5** — nếu xác suất ác tính ≥0,5\geq 0{,}5 ≥0,5 thì dự đoán nhãn là 1 (ác tính), ngược lại là 0 (lành tính). `.astype(int)` chuyển giá trị True/False thành số 1/0.
### Đo accuracy và AUC

```Python
acc_lr = accuracy_score(y_te, yhat_te)

auc_lr = roc_auc_score(y_te, p_te)
```

- `accuracy_score(y_te, yhat_te)`: so sánh nhãn dự đoán `yhat_te` với nhãn thật `y_te`, tính tỉ lệ phần trăm dự đoán đúng — đây là **accuracy** (độ chính xác).
- `roc_auc_score(y_te, p_te)`: tính chỉ số **ROC-AUC**, đo khả năng mô hình **phân biệt** giữa 2 lớp dựa trên xác suất dự đoán (không phụ thuộc vào việc chọn ngưỡng 0,5 hay ngưỡng nào khác) — AUC càng gần 1 thì mô hình phân biệt càng tốt.