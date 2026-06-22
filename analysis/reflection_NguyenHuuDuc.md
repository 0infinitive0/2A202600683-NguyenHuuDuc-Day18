# Reflection — Lab 18: Production RAG
**Thành viên:** Nguyen Huu Duc

## Phần 1: Mapping bài giảng

| Lecture Concept | Module | Hàm cụ thể | Observation |
|----------------|--------|-------------|-------------|
| Semantic chunking | M1 | `chunk_semantic()` | Threshold 0.5 giúp nhóm các câu liên quan chủ đề lại với nhau, tránh việc một ý bị cắt ngang giữa 2 chunk như khi dùng fixed-size basic chunking. |
| BM25 + Dense fusion | M2 | `reciprocal_rank_fusion()` | RRF giải quyết vấn đề chênh lệch thang điểm giữa Dense (cosine) và Sparse (BM25 scores) bằng cách dùng thứ hạng, giúp mang lại kết quả lai hiệu quả nhất. |
| Cross-encoder reranking | M3 | `CrossEncoderReranker.rerank()` | Bù đắp lại cho việc bi-encoder ở bước Dense search bỏ sót ngữ cảnh chi tiết; Cross-encoder phân tích cả query và document cùng lúc nên Context Precision tăng cực mạnh. |
| RAGAS 4 metrics | M4 | `evaluate_ragas()` | Giúp chẩn đoán chính xác pipeline đang yếu ở đâu. Ví dụ nếu Context Recall thấp, chứng tỏ khâu Search hoặc Chunking có vấn đề chứ không phải LLM. |
| Contextual embeddings | M5 | `contextual_prepend()` | Giảm retrieval failure bằng cách "neo" ngữ cảnh gốc vào mỗi đoạn văn nhỏ, giúp chunk giữ được ý nghĩa gốc kể cả khi bị bứt ra khỏi document. |

## Phần 2: Khó khăn & giải quyết

- **Lỗi gặp phải:** `OverflowError: cannot convert longdouble infinity to integer` trong thư viện Numpy 1.26 khi chạy trên Windows Python 3.14. Đồng thời, hàm evaluate của RAGAS bị treo khi gọi OpenAI API hoặc không lấy được event loop.
- **Cách debug:** Xác định `numpy` đang gây lỗi với `qdrant-client`, tôi đã nâng cấp numpy lên bản `2.5.0`. Vấn đề event loop (asyncio) trên Python 3.14 được fix bằng cách chèn `asyncio.set_event_loop(asyncio.new_event_loop())`.
- **Kiến thức thiếu:** Sự bất đồng bộ của `asyncio` loop giữa các threads khi dùng `nest_asyncio` trong các library của langchain và ragas. 
- **Cách bổ sung:** Đọc thêm document của Python về Event Loops và cách quản lý API limits/concurrency của `ragas`.

## Phần 3: Action Plan cho project

### Hiện tại
- RAG pipeline hiện tại: Đang sử dụng phương pháp Naive RAG (Basic chunking + Dense Vector Search thuần túy).
- Known issues: Context Precision rất thấp do search trả về nhiều tài liệu nhiễu. Câu trả lời của LLM hay bị nhầm lẫn giữa các nội dung tương tự.

### Plan áp dụng
1. [x] **Chunking strategy:** Áp dụng Structure-Aware Chunking kết hợp Semantic Chunking. Vì dữ liệu chủ yếu là tài liệu policy/văn bản pháp luật có cấu trúc heading rõ ràng, giữ được cấu trúc này là rất quan trọng.
2. [x] **Search:** Sử dụng Hybrid Search (BM25 + Dense). Dense lo về mặt ý nghĩa ngữ nghĩa, BM25 lo các từ khóa chính xác (mã nhân viên, tên thuật ngữ cụ thể).
3. [x] **Reranking:** Có sử dụng Cross-Encoder (ví dụ bge-reranker-m3). Đây là bước cực kỳ quan trọng để dọn dẹp các chunk nhiễu (false positives) trước khi đưa vào LLM.
4. [x] **Evaluation:** Sử dụng RAGAS với 4 metrics cơ bản làm benchmark sau mỗi lần tinh chỉnh pipeline để đảm bảo thay đổi không làm pipeline tệ đi.
5. [x] **Enrichment:** Sử dụng Contextual Prepend và HyQA. Thêm câu hỏi giả thuyết giúp Dense search tìm chính xác phần giải đáp cho câu hỏi thực tế của User.

### Timeline
- **Tuần 1:** Thay thế Chunking hiện tại bằng Structure-Aware Chunking và tích hợp BM25 vào Search.
- **Tuần 2:** Thêm Reranking module, tối ưu threshold và bắt đầu chạy benchmark RAGAS đánh giá sự khác biệt.
- **Tuần 3:** Cài đặt M5 Enrichment với single LLM call để tiết kiệm chi phí, thực hiện full end-to-end evaluation và tinh chỉnh System Prompt.
