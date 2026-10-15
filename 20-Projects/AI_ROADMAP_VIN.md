---
title: Lộ trình tự kiểm tra - AI thực chiến Vingroup/VinUni
created: 2026-10-11
updated: 2026-10-11
tags: [lo-trinh, AI, VinUni, on-tap]
status: dang-thuc-hien
---

# Lộ trình tự kiểm tra - AI thực chiến Vingroup/VinUni

> [!info] Mục tiêu
> Coi chương trình **Đào tạo 20.000 nhân tài AI thực chiến** (Vingroup/VinUni) ở đợt tuyển sinh sau là một bài tự kiểm tra. Đỗ thì tốt. Trượt thì vẫn có nền tảng để đi thực tập.

> [!tip] Cách dùng file này
> Đây là **file duy nhất** chứa lộ trình. Mọi cập nhật tiến độ ghi vào đây. Khi Claude gửi bản mới, ghi đè file cùng tên trong vault.

## 1. Sự thật cần nhớ (đã xác minh)
- Poster đợt 2 khóa V: nộp hồ sơ 23/09 - 15/10/2026, thi 25/10/2026, khai giảng 05/11/2026. **Đợt này bỏ qua.**
- Đối tượng tuyển sinh theo thông tin công bố: người **đã tốt nghiệp** đại học, cao đẳng hoặc trung cấp ngành liên quan gần với AI.
- Quy trình: xét hồ sơ kết hợp bài kiểm tra đầu vào. Hồ sơ đạt năng lực được bảo lưu cho khóa sau.
- Học viên được miễn 100% học phí và hỗ trợ 8 triệu đồng/tháng.
- Đợt đầu: khoảng 85% người đăng ký có kinh nghiệm hoặc thực tập CNTT/AI/ML.
- Chương trình cường độ cao: lab hàng ngày, thi giữa kỳ ở tuần 6, rồi 6 tuần dự án thực chiến.

## 2. Những điều chưa biết (phải làm rõ)
- [ ] Ngày đợt tuyển sinh tiếp theo, và chương trình có còn tuyển năm sau không.
- [ ] Sinh viên chưa tốt nghiệp có được nộp không (hỏi ban tuyển sinh, giữ câu trả lời bằng văn bản).
- [ ] Nội dung và dạng đề bài thi đầu vào.
- [ ] Có học bán thời gian được không.
- [ ] Ngày tốt nghiệp và thời điểm phải chốt đề tài đồ án.

> [!warning] Chương trình này không phải kênh tìm đề tài đồ án
> Dự án thực chiến dùng bài toán và dữ liệu của doanh nghiệp. Chưa có thông tin nào nói được mang ra làm đồ án. Tìm đề tài thực tế tại nơi thực tập sẽ chắc hơn.

## 3. Quy tắc làm việc
- Học **21:00 - 23:00 hằng ngày** (2 giờ/ngày), 6 ngày/tuần, 1 ngày nghỉ. Giới hạn cứng là 3 giờ/ngày. Giờ dư dùng để bù khi lỡ buổi, không dùng thêm một cách tùy hứng.
- Bắt đầu **Chủ nhật 11/10/2026**. Các mốc ngày chỉ là dự kiến, vì còn phải học các môn trên trường.
- **Chỉ sang tuần sau khi đạt cổng:** làm lại bài kiểm tra mới của tuần đó đạt từ 70%. Trễ thì ở lại tuần đó. "Sớm hơn" chỉ hợp lệ khi đạt cổng, không dựa vào cảm giác.
- Ngày bận trường: chỉ cần 30 phút ôn sổ lỗi, không bỏ trắng. Tuần thi hoặc nộp bài lớn thì báo trước để dời lộ trình.
- Ba luật làm bài: gạch chân điều kiện của đề; tự kiểm tra bằng ví dụ nhỏ tính tay; dự đoán sai thì ghi "SAI" và nói vì sao.
- Thời gian còn lại dành cho đồ án và các môn khác.
- Không tra cứu khi làm bài kiểm tra. Chưa rõ thì ghi "không biết".
- Mỗi buổi học 2 giờ: 50 phút học, 50 phút bài tập, 15 phút tự giải thích lại bằng lời (không nhìn tài liệu), 5 phút ghi nhật ký.
- Buổi học được chạy code. Bài kiểm tra thì không. Luôn dự đoán trước khi chạy.

## 4. Tiêu chí tự chấm (đặt trước, không đổi)
- Giải thích được kết quả mô hình mà không nhìn tài liệu.
- Demo chạy được trên máy người khác.
- Đề thi thử có bấm giờ đạt trên **70%** (con số do Claude đề xuất, có thể chỉnh).
- Nếu trượt thì ghi rõ trượt ở phần nào.

## 5. Kết quả bài chẩn đoán Tuần 0 (11/10/2026): 18/100

| Phần | Điểm | Nhận xét |
|---|---|---|
| A. Python/pandas | 2/20 | A1 sai (hiểu nhầm gán biến list), A2 đúng kết quả nhưng không dùng list comprehension, A3-A4 chưa biết |
| B. SQL | 0/15 | Chưa làm |
| C. Xác suất thống kê | 0/25 | C1 nhầm biến cố độc lập với xung khắc, các câu còn lại chưa biết |
| D. Đại số tuyến tính | 9/20 | Nhân ma trận đúng, độ dài vector đúng, cosine lệch số lẻ, chưa biết định thức và trị riêng |
| E. ML | 7/20 | Gradient descent hiểu khá tốt, nhận ra overfitting nhưng chưa nêu cách khắc phục, chưa hiểu data leakage đầy đủ |

**Kết luận:** dưới 50, nền tảng còn yếu. Theo thang đã đặt trước, Tuần 1-2 dành toàn bộ cho ôn lại phần điểm thấp, chưa làm gì phức tạp hơn.

### Sổ lỗi (đọc lại mỗi cuối tuần)
- **Ngày 4 (B5):** Đề hỏi "trung bình `amount` của các dòng có `status == "paid"`", tức **một con số**: `df[df["status"] == "paid"]["amount"].mean()` = 76.67. Bạn thêm `groupby("user_id")` không ai yêu cầu và ra 80 và 75. Con số 76.67 đã xuất hiện ở A2, đối chiếu chéo sẽ bắt được lỗi.
- **Ngày 4 (0.3):** Kết quả tính tay ghi hàng 0 với `amount = 200`, sai. Đúng là hàng 0 (50, new) và hàng 5 (60, new). Code đúng nhưng tính tay sai, và bạn không dán kết quả chạy để so.
- **Ngày 4 (B2, A3):** Ghi "tăng dần" khi đề nói "giảm dần" (B2, ô điều kiện) và khi code là `ascending=False` (A3). Tăng dần: nhỏ đến lớn. Giảm dần: lớn đến nhỏ. Code vẫn đúng, nhưng ô "điều kiện đề" chép sai nghĩa là chưa đọc kỹ.
- **Ngày 4 (C3):** `nlargest` không phải "dành cho Series". `df.nlargest(2, "amount")` trả DataFrame. `nlargest(n)` dùng khi chỉ cần n giá trị lớn nhất. `sort_values` dùng khi cần sắp xếp toàn bộ hoặc sắp theo nhiều cột.
- **Ngày 4 (C2):** `groupby` gồm ba bước: tách thành nhóm, tính trên từng nhóm, gộp kết quả lại.
- **Ngày 4 (nhật ký):** Ghi thời gian 30 phút. Kế hoạch là 2 giờ/ngày. Cần biết số giờ thực tế để lộ trình khả thi.
- **Ngày 3 (lỗi chính, lặp lại ở 0.1, A5, B3, B5):** Không đọc kỹ điều kiện của đề: lập phương mà viết bình phương; hỏi nhãn hàng mà liệt kê giá trị; "vị trí thứ 3 đếm từ 1" mà dùng chỉ số 3 (đúng là 2); "bằng pandas" mà viết vòng lặp. Cách chữa: điền ô "Điều kiện đề" và "Kết quả mong đợi" trước khi viết code.
- **Ngày 3 (B4):** `df[df["amount"] > 50 & df["status"] == "paid"]` lỗi vì `&` ưu tiên cao hơn `>` và `==`. Phải là `df[(df["amount"] > 50) & (df["status"] == "paid")]`.
- **Ngày 3 (C2, A4):** Cắt lát bằng `loc` **gồm cả đầu mút cuối** (theo nhãn), `iloc` **không gồm** (theo vị trí). Nên `loc[0:2]` trả 3 phần tử.
- **Ngày 3 (0.4):** Thông dịch và biên dịch bạn nói ngược. Thông dịch: thực thi từng dòng lúc chạy. Biên dịch: dịch toàn bộ trước khi chạy. NumPy nhanh vì vòng lặp nằm trong mã C đã biên dịch.
- **Ngày 3 (0.5):** `axis=0` gộp các hàng lại, mỗi cột ra một tổng. `axis=1` gộp các cột lại, mỗi hàng ra một tổng. Cách nói "cộng các cột" gây nhầm.
- **Ngày 3 (A3):** Kết quả dán của hàng 0 ghi `paid`, nhưng dữ liệu là `new`. Phải dán nguyên văn kết quả chạy, không chỉnh sửa.
- **Ngày 2 (B3):** Trung bình từng **cột** = tổng theo `axis=0` chia cho **số hàng** (`shape[0]`). Hoặc dùng thẳng `m.mean(axis=0)`. Luôn tính tay một ví dụ nhỏ để kiểm tra.
- **Ngày 2 (B1, B5):** Không đọc kỹ điều kiện của đề (cấm vòng lặp, số chẵn, khoảng 1 đến 20). Lỗi này lặp lại ở 3 bài.
- **Ngày 2 (A2, A4):** `+=` trên mảng NumPy sửa tại chỗ. Cắt lát `a[1:4]` là một view, sửa view thì mảng gốc đổi. View là đối tượng khác nhưng dùng chung dữ liệu.
- **Ngày 2 (A1):** `a * 2` tạo mảng mới, `b` không trỏ cùng `a`.
- **Ngày 2 (C1, C3):** Vectorization nhanh vì vòng lặp chạy trong mã C, không qua trình thông dịch từng phần tử. `axis=0` gộp các hàng lại (kết quả theo cột), `axis=1` gộp các cột lại (kết quả theo hàng).
- **Ngày 1 (vặn lại):** `b += [4]` với list sửa tại chỗ (giống `extend`), nên `a` đổi theo. `c = c + [5]` tạo list mới rồi gán lại tên `c`, nên `a` không đổi. Cùng một toán tử, hành vi khác nhau tùy kiểu khả biến hay bất biến. NumPy cũng vậy: `b += 10` sửa tại chỗ, `b = b + 10` tạo mảng mới.
- **Ngày 1:** Nói "bất khả thi" mà chưa thử là bỏ cuộc, không phải lý do. Comprehension cho phép nhiều mệnh đề `for`, đọc từ trái sang phải như vòng lặp lồng nhau.
- **Ngày 1:** Giải thích bất biến: số 5 không bị sửa, tên `y` được gắn sang đối tượng khác.
- **A1:** `b = a` không tạo bản sao. `a` và `b` là hai tên cùng trỏ vào một list, nên sửa qua `b` thì `a` cũng đổi. Muốn bản sao: `b = a.copy()`.
- **A2:** Đề yêu cầu list comprehension, đã viết for loop. Phải đọc kỹ yêu cầu về **hình thức**, không chỉ kết quả.
- **C1:** Độc lập khác xung khắc. Công thức hợp tổng quát: P(A ∪ B) = P(A) + P(B) − P(A ∩ B). Không được bỏ phần giao.
- **D4:** Tính số chia dở dang rồi làm tròn sớm gây lệch số lẻ. Giữ nguyên căn thức đến bước cuối.
- **E4:** Data leakage rộng hơn "kết quả test lọt vào train": mọi thông tin từ ngoài tập train đi vào quá trình huấn luyện, kể cả tính trung bình/độ lệch chuẩn trên toàn bộ dữ liệu trước khi chia.

## 6. Lộ trình 8 tuần (đã điều chỉnh theo điểm chẩn đoán)
> Đây là suy luận từ nội dung chương trình công bố, **chưa phải đề cương chính thức**. Phần ứng dụng AI và triển khai ở tuần 7-8 bị thu gọn so với bản đầu vì nền tảng còn yếu. Nếu nền tảng chưa vững thì dựng demo lên trên là vô nghĩa.

| Tuần | Nội dung | Sản phẩm bắt buộc | Xong |
|---|---|---|---|
| 0 | Bài chẩn đoán | Bảng điểm (18/100) | - [x] |
| 1 | Python (tham chiếu, comprehension) và pandas | 20 bài thực hành, giải thích được | - [ ] |
| 2 | SQL và xác suất cơ bản | 20 bài thực hành. **Kiểm tra mới cuối tuần** | - [ ] |
| 3 | Thống kê (sai số chuẩn, khoảng tin cậy, p-value), đại số tuyến tính (định thức, trị riêng, cosine) và giải tích nền tảng (đạo hàm, quy tắc chuỗi, gradient). Có thể phải kéo sang Tuần 4 nếu quá tải | 20 bài thực hành | - [ ] |
| 4 | Đánh giá ML: precision/recall, overfitting, data leakage, Pipeline. **Kiểm tra mới cuối tuần** | 1 bài ML hoàn chỉnh từ dữ liệu thô đến đánh giá | - [ ] |
| 5-6 | Deep learning cơ bản bằng PyTorch | 1 mô hình huấn luyện và giải thích được | - [ ] |
| 7 | Dựng demo ứng dụng AI (gọi API mô hình ngôn ngữ), Git, FastAPI | Demo chạy được | - [ ] |
| 8 | Tư duy sản phẩm, đề thi thử tổng hợp có bấm giờ | Bài tự chấm và CV | - [ ] |

## 7. Chi tiết Tuần 1 (Python và pandas), CN 11/10 - T7 17/10
File bài Ngày 1: `buoi_hoc_ngay1.md`.
Gợi ý tài liệu: "10 minutes to pandas" (pandas.pydata.org/docs/user_guide/10min.html). Bạn tự chọn thêm nguồn khác.

- [x] **Ngày 1:** Python tham chiếu vs bản sao, list comprehension. Kết quả: 8 đúng, 2 một phần (A3, C3), 1 sai (B1). Còn nợ: giải thích bất biến vs khả biến, tiêu chí dùng comprehension.
- [x] **Ngày 2 (13/10):** NumPy: mảng, `shape`, chỉ số, tính theo vectơ. Kết quả: 7 đúng, 4 một phần, 4 sai hoặc chưa biết (B1, B3, B5, C1). Còn nợ: viết NumPy không vòng lặp, chia đúng chiều khi tính trung bình, giải thích vectorization và `axis`.
- [x] **Ngày 3 (14/10):** 20 phút sửa nợ Ngày 2, rồi DataFrame, Series, `loc`, `iloc`. Kết quả: 7 đúng, 5 một phần, 6 sai hoặc chưa biết (0.1, A3, A5, B3, B4, C2). Lỗi chính: không đọc kỹ đề (lập phương, nhãn hàng, đếm từ 1, "bằng pandas") và `&` thiếu ngoặc.
- [x] **Ngày 4 (15/10):** Sửa nợ Ngày 3, `groupby`, `sort_values`, `nlargest`. Kết quả: 16 đúng, 4 một phần, 2 sai (B5, C3). Tăng rõ so với Ngày 3 (7 đúng). Còn nợ: tính tay kết quả mong đợi (0.3), chép đúng "tăng dần/giảm dần", đề hỏi một con số (B5), tiêu chí `nlargest` và `sort_values`, ba bước `groupby`.
- [ ] **Ngày 5:** 25 phút sửa nợ Ngày 4, rồi chained assignment, `SettingWithCopyWarning`, `merge`, giá trị thiếu. Làm lại câu A4. File bài: `buoi_hoc_ngay5.md`.
- [ ] **Ngày 6:** Ôn lỗi cũ, tự giải thích lại 5 khái niệm bằng lời, làm bài kiểm tra nhỏ do Claude giao.
- [ ] **Ngày 7:** Nghỉ hoàn toàn.

### Vá nền Python (tự chấm 0-5 mỗi mục, ghi vào nhật ký, nhờ Claude kiểm tra)
- [ ] dict và set: tạo, truy cập, duyệt, khi nào dùng
- [ ] tuple và unpacking
- [ ] hàm: tham số mặc định, `return` nhiều giá trị, `lambda`
- [ ] `sorted` với `key`, slicing
- [ ] `try/except`
- [ ] đọc và ghi file
- [ ] class cơ bản (`__init__`, phương thức)

Mỗi buổi dành 10-15 phút cho một mục, không cần học dồn.

## 8. Chi tiết Tuần 2 (SQL và xác suất cơ bản), CN 18/10 - T7 24/10
Gợi ý tài liệu: SQLBolt (sqlbolt.com) để luyện cú pháp.

- [ ] **Ngày 1:** `SELECT`, `WHERE`, `ORDER BY`, `GROUP BY`, khác biệt `WHERE` và `HAVING`.
- [ ] **Ngày 2:** `JOIN`, `LEFT JOIN` kết hợp `IS NULL` để tìm bản ghi không khớp.
- [ ] **Ngày 3:** Window function (`ROW_NUMBER`, `PARTITION BY`). Làm lại câu B1-B3.
- [ ] **Ngày 4:** Xác suất: độc lập vs xung khắc, hợp, giao, xác suất có điều kiện. Làm lại câu C1.
- [ ] **Ngày 5:** Định lý Bayes, độ nhạy và độ đặc hiệu, kỳ vọng và phương sai. Làm lại câu C2-C3.
- [ ] **Ngày 6:** Ôn lỗi cũ, tự giải thích lại, chuẩn bị kiểm tra.
- [ ] **Ngày 7:** Kiểm tra mới (có chỗ trả lời sẵn) rồi nghỉ.

## 9. Quy ước bài kiểm tra
- Mỗi câu hỏi có sẵn dòng **Trả lời:** và khối code trống để bạn điền, ít phải chỉnh định dạng.
- Bạn điền, gửi lại nguyên file, Claude chấm có phản biện.
- Mỗi bài mới là đề mới, không dùng lại câu đã biết đáp án.

## 10. Việc Claude sẽ làm cùng bạn
- Chấm các bài kiểm tra và vặn lại câu đúng nhưng giải thích hời hợt.
- Giao bài kiểm tra mới ở cuối Tuần 2, Tuần 4 và các mốc sau, nội dung chỉnh theo phần còn yếu.
- Chỉnh lại toàn bộ lộ trình khi có thông tin chính thức về bài thi đầu vào.

## 11. Nhật ký tiến độ
| Ngày | Đã làm | Vướng ở đâu | Giờ học |
|---|---|---|---|
| 2026-10-11 | Làm bài chẩn đoán Tuần 0, kết quả 18/100 | Xác suất thống kê, SQL, pandas | |
| 2026-10-11 (tối) | Ngày 1: 8 đúng, 2 một phần, 1 sai | Giải thích bất biến vs khả biến, tiêu chí dùng comprehension, nhầm biến ở B1 | 2 giờ |
| 2026-10-13 | Ngày 2: 7 đúng, 4 một phần, 4 sai | Không đọc kỹ yêu cầu hình thức, không tự kiểm tra, `axis`, vectorization | |
| 2026-10-14 | Ngày 3: 7 đúng, 5 một phần, 6 sai | Không đọc kỹ đề (lặp lại), `&` thiếu ngoặc, `loc` cắt lát, thông dịch vs biên dịch | (chưa điền) |
| 2026-10-15 | Ngày 4: 16 đúng, 4 một phần, 2 sai | Tính tay sai (0.3), thêm yêu cầu không có (B5), tăng/giảm dần, tiêu chí `nlargest` | 30 phút |
| | | | |

## 12. Nguồn
- [20K AI Handbook - VinUni](https://vinuni.edu.vn/aithucchien/wp-content/uploads/2026/06/20K-AI-Handbook-ver2.1.pdf)
- [Điều kiện tuyển sinh - Diễn đàn Doanh nghiệp](https://diendandoanhnghiep.vn/print/10170660.html)
- [Quy trình xét tuyển - VTC News](https://vtcnews.vn/chuong-trinh-dao-tao-ai-thuc-chien-cua-vingroup-nhan-hon-1-100-ho-so-sau-24-gio-ar999939.html)
- [Miễn học phí, hỗ trợ 8 triệu - SGGP](https://www.sggp.org.vn/vingroup-tang-toc-dao-tao-20000-nhan-tai-ai-thuc-chien-post864361.html)
