---
created: 2026-09-04
Hub: "[[MOC - AI & ML]]"
status: seed
---

> [!summary] Tóm tắt nhanh
> -
> -
> -
## Định nghĩa

**RAG (Retrieval-Augmented Generation)** = một kỹ thuật giúp LLM trả lời dựa trên nguồn tài liệu bên ngoài, thay vì chỉ dựa vào kiến thức đã học sẵn lúc training.

**Vấn đề mà RAG giải quyết**

Một LLM (GPT, Claude, Llama...) được train xong là "đóng băng" kiến thức tại 1 thời điểm — nó không biết:

- Sự kiện xảy ra sau ngày train
- Dữ liệu riêng tư/nội bộ của bất kỳ tổ chức nào (tài liệu công ty, hợp đồng, database sản phẩm)
- Nội dung 1 cuốn sách/website cụ thể bạn muốn hỏi

Nếu hỏi thẳng LLM về những thứ này, nó sẽ trả lời "tôi không biết", hoặc tệ hơn — **hallucination**: bịa ra câu trả lời nghe rất tự tin và hợp lý nhưng hoàn toàn sai, vì bản chất LLM là dự đoán từ tiếp theo hợp lý về mặt thống kê, không phải tra cứu sự thật.

**3 bước của RAG, theo đúng thứ tự luôn xảy ra:**

1. **Retrieval (Truy xuất)** — khi có câu hỏi, hệ thống trước tiên **tìm kiếm** trong 1 kho dữ liệu bên ngoài (thường là vector database) để lấy ra những đoạn văn bản liên quan nhất đến câu hỏi đó
2. **Augmentation (Tăng cường)** — những đoạn văn bản tìm được ở bước 1 được **ghép vào prompt**, cùng với câu hỏi gốc, tạo thành 1 prompt "giàu ngữ cảnh" hơn để gửi cho LLM
3. **Generation (Sinh câu trả lời)** — LLM đọc prompt đã được tăng cường đó và **viết câu trả lời**, dựa trên thông tin vừa được cung cấp thay vì chỉ dựa vào kiến thức nó đã học từ trước

**Sơ đồ tổng quát:**

```
Câu hỏi người dùng
   → Tìm tài liệu liên quan (Retrieval)
   → Ghép tài liệu + câu hỏi thành 1 prompt (Augmentation)
   → LLM đọc prompt, sinh câu trả lời (Generation)
   → Trả lời + (tuỳ chọn) chỉ ra nguồn tài liệu đã dùng
```

**Điểm quan trọng cần nhớ:** RAG **không phải** một loại model AI mới, không phải kỹ thuật huấn luyện đặc biệt — nó chỉ là 1 **kiến trúc/pattern**: đặt bước tìm kiếm đứng ngay trước bước sinh văn bản, rồi nối chúng lại bằng cách nhét kết quả tìm kiếm vào prompt. Bất kỳ LLM có sẵn nào (không cần train lại) đều có thể "gắn" RAG vào để trở nên hữu ích với 1 kho dữ liệu cụ thể.

## Tại sao quan trọng

- **Cập nhật kiến thức không cần train lại** — train lại 1 LLM cực kỳ tốn kém (tiền, thời gian, dữ liệu); RAG cho phép "dạy" LLM về thông tin mới chỉ bằng cách thêm tài liệu vào kho dữ liệu tìm kiếm
- **Giảm hallucination** — buộc LLM bám vào tài liệu thật thay vì tự bịa, đặc biệt quan trọng trong các lĩnh vực cần độ chính xác cao (y tế, pháp lý, tài chính)
- **Minh bạch, có thể kiểm chứng** — vì biết chính xác tài liệu nào được dùng để tạo câu trả lời, người dùng có thể click vào nguồn để xác minh, thay vì phải tin tưởng mù quáng vào model
- **Áp dụng được cho dữ liệu riêng tư** — 1 công ty có thể cho LLM "đọc" tài liệu nội bộ của họ mà không cần gửi dữ liệu đó đi train model ở đâu cả — chỉ cần đưa vào kho tìm kiếm dùng lúc trả lời

## Áp dụng vào công việc như nào?

- **Trợ lý hỏi-đáp tài liệu nội bộ công ty** — tình huống RAG phổ biến nhất trong công việc thật. Công ty có hàng nghìn tài liệu (wiki nội bộ, hợp đồng, quy trình...), nhân viên mất thời gian tìm kiếm thủ công → build 1 chatbot RAG đọc toàn bộ tài liệu đó, trả lời câu hỏi kèm trích dẫn nguồn. Đây gần như nguyên xi kiến trúc bạn đã làm, chỉ đổi nguồn dữ liệu.
- **Customer support tự động** — RAG trên toàn bộ ticket/FAQ cũ của công ty, giúp bot trả lời khách hàng chính xác hơn, ít bịa hơn so với chatbot chỉ dựa vào LLM thuần.
- **Trợ lý code/tài liệu kỹ thuật** — RAG trên codebase hoặc docs nội bộ, giúp dev mới hoặc chính AI assistant trả lời câu hỏi về hệ thống cụ thể của công ty (đúng hướng dự án "AI agent secretary" bạn từng có ý định làm).

## Dự án liên kết
- [[AI-Powered-Search-Engine]]