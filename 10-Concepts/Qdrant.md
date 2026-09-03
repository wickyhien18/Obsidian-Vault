---
created: 2026-09-03
In: "[[MOC - AI & ML]]"
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



## Dự án liên kết
- [[AI-Powered-Search-Engine]]