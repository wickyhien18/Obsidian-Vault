---
created: 2026-09-03
Hub: "[[MOC - AI & ML]]"
---

> [!summary] Tóm tắt nhanh
> -
> -
> -
## Định nghĩa

Qdrant là một **vector database** — hệ quản trị cơ sở dữ liệu chuyên lưu trữ vector (mảng số, ví dụ 768 số từ embedding) và tìm kiếm cực nhanh các vector "gần" một vector truy vấn, ngay cả khi có hàng triệu/tỷ bản ghi. 
## Tại sao quan trọng

Không có vector DB, bạn sẽ phải tự so sánh vector câu hỏi với **từng vector một** trong toàn bộ tập dữ liệu (brute-force) — với 7583 vector thì còn ổn, nhưng với hàng triệu bản ghi (dữ liệu thật ở công ty) sẽ chậm không dùng được. Qdrant dùng thuật toán ANN (Approximate Nearest Neighbor, cụ thể là HNSW) để tìm gần đúng cực nhanh, đánh đổi một chút độ chính xác lấy tốc độ — đây là hạ tầng bắt buộc cho bất kỳ hệ thống RAG/semantic search nào ở quy mô thực tế, và đúng là công nghệ nằm thẳng trong JD AI Engineer bạn đang nhắm.

## Lựa chọn tốt khi nào

- Xây dựng **RAG/semantic search làm sản phẩm độc lập** — Qdrant là vector DB "thuần", tối ưu riêng cho việc này, hiệu năng cao, hỗ trợ cả dense + sparse vector (hybrid search bạn vừa làm), filtering, self-host dễ (chỉ 1 lệnh Docker)
- Cần **mã nguồn mở, tự host miễn phí**, không phụ thuộc dịch vụ trả phí
- Dataset **lớn, tăng trưởng liên tục**, cần tối ưu tốc độ/bộ nhớ chuyên sâu (quantization, sharding)
- Team đã quen làm việc với **microservice riêng biệt** cho từng chức năng (Qdrant là 1 service độc lập, tách khỏi database chính)
## Lựa chọn không tốt khi nào?

|Tình huống|Vì sao Qdrant không tối ưu|Thay thế|
|---|---|---|
|Dữ liệu **đã có sẵn trong PostgreSQL**, không muốn thêm 1 service mới để vận hành/bảo trì|Thêm Qdrant nghĩa là thêm 1 hệ thống riêng phải deploy, giám sát, backup song song với DB chính|**pgvector** — extension cho PostgreSQL, lưu vector ngay trong DB quan hệ đang dùng, không cần hạ tầng mới (đây cũng là lựa chọn có trong JD của bạn)|
|Cần **tích hợp cực nhanh, prototype nhỏ**, không quan tâm hiệu năng dài hạn|Setup Docker + client riêng hơi thừa cho việc test nhanh|**ChromaDB** — chạy embedded ngay trong code Python, không cần server riêng, nhẹ nhất để bắt đầu học|
|Cần **scale cực lớn ở doanh nghiệp**, có ngân sách, ưu tiên hỗ trợ enterprise chính thức|Qdrant vẫn scale được nhưng hệ sinh thái enterprise (support, SLA) chưa lớn bằng|**Milvus** — mạnh hơn ở quy mô rất lớn, nhưng setup phức tạp hơn (cần nhiều service đi kèm: etcd, MinIO...)|
|Muốn **dùng chung hạ tầng cloud có sẵn** (AWS/GCP/Azure), không muốn tự quản lý server|Tự host Qdrant vẫn cần lo vận hành, backup, uptime|**Pinecone** — vector DB dạng managed service (trả phí), không cần tự vận hành gì cả, đánh đổi lấy chi phí và ít quyền kiểm soát hơn|
|Ứng dụng đã dùng **Elasticsearch** cho full-text search sẵn, chỉ cần thêm chút semantic search|Thêm Qdrant là thêm hệ thống thứ 3 (bên cạnh DB chính + Elasticsearch)|**Elasticsearch/OpenSearch với dense_vector field** — tận dụng lại hạ tầng search đã có, thêm khả năng vector search vào cùng 1 chỗ|
## Áp dụng vào công việc như nào?

**1. Công ty đã có hạ tầng sẵn — việc đầu tiên là kiểm tra, không phải chọn công nghệ mới**  
Thực tế công việc: hiếm khi bạn được tự do chọn "em thích Qdrant nên dùng Qdrant". Câu hỏi đầu tiên luôn là: hệ thống hiện tại đang dùng DB gì? Nếu công ty đã chạy PostgreSQL cho toàn bộ sản phẩm, việc đề xuất thêm Qdrant nghĩa là thuyết phục team DevOps triển khai, giám sát, backup thêm 1 service mới — chi phí vận hành thực sự, không chỉ là code. Đây là lúc kỹ năng **đánh giá trade-off** (như bảng lúc nãy) quan trọng hơn kỹ năng code — biết đề xuất pgvector trước, chỉ chuyển sang Qdrant/Milvus khi có lý do rõ ràng (quy mô, hiệu năng) là tư duy engineer thật sự, không phải chỉ biết dùng API.

**2. Prototype/PoC nội bộ trước khi đưa ra quyết định hạ tầng**  
Giống với Kaggle — khi cần chứng minh nhanh "semantic search có work với dữ liệu công ty không" trước khi xin ngân sách/hạ tầng chính thức, ChromaDB (chạy embedded, không cần server) hoặc Qdrant tự host bằng Docker (như bạn đang làm) là lựa chọn nhanh nhất để demo cho sếp/khách hàng, sau đó mới bàn chuyện chọn hạ tầng production lâu dài.

**3. Maintain/optimize hệ thống RAG đã có sẵn trong công ty**  
Nếu vào công ty đã có sẵn RAG pipeline chạy Qdrant hoặc Milvus, công việc thực tế thường là: tối ưu tốc độ search (quantization, tuning HNSW params), xử lý khi dataset tăng trưởng (sharding), hoặc debug khi kết quả search không chính xác (đúng những khái niệm bạn vừa học: chunk_size, chunk_overlap, hybrid search). Đây là lý do project của bạn quan trọng — không phải để "dùng đúng Qdrant" mãi mãi, mà để hiểu **cơ chế chung** (vector similarity, ANN, ingest pipeline) áp dụng được cho bất kỳ vector DB nào công ty đang dùng, kể cả pgvector hay Milvus.

## Dự án liên kết
- [[AI-Powered-Search-Engine]]