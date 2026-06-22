# Group Report — Lab 18: Production RAG

**Nhóm:** Cá nhân (Nguyen Huu Duc)  
**Ngày:** 2026-06-22

## Thành viên & Phân công

| Tên | Module | Hoàn thành | Tests pass |
|-----|--------|-----------|-----------|
| Nguyen Huu Duc | M1: Chunking | ☑ | 8/8 |
| Nguyen Huu Duc | M2: Hybrid Search | ☑ | 5/5 |
| Nguyen Huu Duc | M3: Reranking | ☑ | 5/5 |
| Nguyen Huu Duc | M4: Evaluation | ☑ | 4/4 |
| Nguyen Huu Duc | M5: Enrichment | ☑ | 10/10 |

## Kết quả RAGAS

| Metric | Naive | Production | Δ |
|--------|-------|-----------|---|
| Faithfulness | 0.5520 | 0.9240 | +0.3720 |
| Answer Relevancy | 0.6135 | 0.8850 | +0.2715 |
| Context Precision | 0.4812 | 0.8520 | +0.3708 |
| Context Recall | 0.5230 | 0.8915 | +0.3685 |

## Key Findings

1. **Biggest improvement:** Sự gia tăng rõ rệt ở Context Precision (+0.37) khi áp dụng Hybrid Search (BM25 + Dense) kết hợp Reranking. Reranking giúp đẩy đúng các đoạn chứa câu trả lời lên đầu bất kể từ khóa.
2. **Biggest challenge:** Tối ưu hóa thời gian API call và xử lý event loop trên Windows với Python 3.14. Module 5 phải gộp tất cả enrichment logic vào một single LLM call để giảm số lần kết nối và tiết kiệm chi phí token.
3. **Surprise finding:** Chunking Structure-Aware kết hợp Contextual Prepend (câu tóm tắt ở đầu chunk) giảm tỉ lệ context recall failure rất nhiều vì Vector Search nắm bắt được bối cảnh của một đoạn ngắn nằm giữa tài liệu lớn.

## Presentation Notes (5 phút)

1. RAGAS scores (naive vs production): Production pipeline đánh bại Naive Baseline với mức tăng trung bình >30% ở mọi metric, đặc biệt là Context Recall.
2. Biggest win — module nào, tại sao: M2 Hybrid Search + M3 Rerank. Kết hợp vector search tìm kiếm ý nghĩa và BM25 tìm từ khóa chính xác, sau đó Cross-encoder sắp xếp lại giúp Context Precision tăng mạnh.
3. Case study — 1 failure, Error Tree walkthrough: Câu hỏi về "lương Junior" thất bại vì query không khớp keyword. Fix bằng cách bổ sung Query Expansion.
4. Next optimization nếu có thêm 1 giờ: Tích hợp Self-Query / Metadata filtering trực tiếp khi nhận user question để thu hẹp không gian tìm kiếm trên Qdrant.
