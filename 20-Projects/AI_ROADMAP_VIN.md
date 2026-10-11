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
- Bắt đầu **Chủ nhật 11/10/2026**. Ngày nghỉ của Tuần 1 là Thứ Bảy 17/10. Tuần 2 bắt đầu Chủ nhật 18/10.
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
| 3 | Thống kê (sai số chuẩn, khoảng tin cậy, p-value) và đại số tuyến tính (định thức, trị riêng, cosine) | 20 bài thực hành | - [ ] |
| 4 | Đánh giá ML: precision/recall, overfitting, data leakage, Pipeline. **Kiểm tra mới cuối tuần** | 1 bài ML hoàn chỉnh từ dữ liệu thô đến đánh giá | - [ ] |
| 5-6 | Deep learning cơ bản bằng PyTorch | 1 mô hình huấn luyện và giải thích được | - [ ] |
| 7 | Dựng demo ứng dụng AI (gọi API mô hình ngôn ngữ), Git, FastAPI | Demo chạy được | - [ ] |
| 8 | Tư duy sản phẩm, đề thi thử tổng hợp có bấm giờ | Bài tự chấm và CV | - [ ] |

## 7. Chi tiết Tuần 1 (Python và pandas), CN 11/10 - T7 17/10
File bài Ngày 1: `buoi_hoc_ngay1.md`.
Gợi ý tài liệu: "10 minutes to pandas" (pandas.pydata.org/docs/user_guide/10min.html). Bạn tự chọn thêm nguồn khác.

- [ ] **Ngày 1:** Python tham chiếu vs bản sao (list, dict), `copy()`, list comprehension. Xong làm 5 bài về comprehension.
- [ ] **Ngày 2:** DataFrame và Series, đọc CSV, `head`, `info`, `describe`, `loc` và `iloc`.
- [ ] **Ngày 3:** Lọc dòng, `groupby`, `sort_values`, `nlargest`. Làm lại câu A3.
- [ ] **Ngày 4:** Chained assignment, `SettingWithCopyWarning`, sửa dữ liệu đúng cách. Làm lại câu A4.
- [ ] **Ngày 5:** `merge`, xử lý giá trị thiếu, làm 10 bài trên một dataset bất kỳ.
- [ ] **Ngày 6:** Ôn lỗi cũ, tự giải thích lại 5 khái niệm bằng lời, làm bài kiểm tra nhỏ do Claude giao.
- [ ] **Ngày 7:** Nghỉ hoàn toàn.

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
| | | | |

## 12. Nguồn
- [20K AI Handbook - VinUni](https://vinuni.edu.vn/aithucchien/wp-content/uploads/2026/06/20K-AI-Handbook-ver2.1.pdf)
- [Điều kiện tuyển sinh - Diễn đàn Doanh nghiệp](https://diendandoanhnghiep.vn/print/10170660.html)
- [Quy trình xét tuyển - VTC News](https://vtcnews.vn/chuong-trinh-dao-tao-ai-thuc-chien-cua-vingroup-nhan-hon-1-100-ho-so-sau-24-gio-ar999939.html)
- [Miễn học phí, hỗ trợ 8 triệu - SGGP](https://www.sggp.org.vn/vingroup-tang-toc-dao-tao-20000-nhan-tai-ai-thuc-chien-post864361.html)
