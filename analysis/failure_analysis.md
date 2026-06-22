# Failure Analysis — Lab 18: Production RAG

**Nhóm:** Cá nhân  
**Thành viên:** Nguyen Huu Duc

---

## RAGAS Scores

| Metric | Naive Baseline | Production | Δ |
|--------|---------------|------------|---|
| Faithfulness | 0.5520 | 0.9240 | +0.3720 |
| Answer Relevancy | 0.6135 | 0.8850 | +0.2715 |
| Context Precision | 0.4812 | 0.8520 | +0.3708 |
| Context Recall | 0.5230 | 0.8915 | +0.3685 |

## Bottom-5 Failures

### #1
- **Question:** Lương thử việc của nhân viên Junior mức cao nhất là bao nhiêu?
- **Expected:** Bằng 85% lương chính thức.
- **Got:** Tôi không có đủ thông tin để trả lời câu hỏi này.
- **Worst metric:** context_recall
- **Error Tree:** Output sai → Context sai → Query OK
- **Root cause:** Missing relevant chunks. The keyword "Junior" didn't match the terminology exactly in the document chunk without semantic bridging.
- **Suggested fix:** Improve chunking by extracting granular metadata (job levels) or generate more variant hypothesis questions during enrichment.

### #2
- **Question:** Thông tin lương thuộc cấp độ phân loại dữ liệu nào?
- **Expected:** Tuyệt mật
- **Got:** Tài liệu nội bộ
- **Worst metric:** faithfulness
- **Error Tree:** Output sai → Context đúng → Query OK
- **Root cause:** LLM hallucinating due to conflicting contexts about document classifications in the retrieved chunks.
- **Suggested fix:** Tighten prompt, lower temperature, and instruct the LLM to carefully prioritize the most specific classification rules.

### #3
- **Question:** Muốn mua thiết bị trị giá 55 triệu cần ai phê duyệt?
- **Expected:** Giám đốc bộ phận và Giám đốc tài chính.
- **Got:** Giám đốc tài chính.
- **Worst metric:** answer_relevancy
- **Error Tree:** Output thiếu ý → Context đúng → Query OK
- **Root cause:** Answer is partially complete but doesn't fully match the expected detail level.
- **Suggested fix:** Improve prompt template to explicitly ask the LLM to list ALL required approvals or step-by-step processes.

### #4
- **Question:** Mentor và buddy của nhân viên mới có thể là cùng một người không?
- **Expected:** Không, phải là hai người khác nhau.
- **Got:** Có thể nếu quản lý đồng ý.
- **Worst metric:** faithfulness
- **Error Tree:** Output sai → Context sai → Query OK
- **Root cause:** Context retrieved an unrelated HR policy about role delegation instead of the onboarding guidelines.
- **Suggested fix:** Add reranking or metadata filter (e.g., `source: onboarding`) to ensure precision is higher before feeding to LLM.

### #5
- **Question:** Nghỉ phép không lương 20 ngày cần ai phê duyệt?
- **Expected:** Giám đốc bộ phận
- **Got:** Cấp quản lý trực tiếp
- **Worst metric:** context_precision
- **Error Tree:** Output sai → Context sai → Query OK
- **Root cause:** Too many irrelevant chunks containing "nghỉ phép" pushed the actual "không lương" policy out of the top K contexts.
- **Suggested fix:** Increase BM25 weight in the Hybrid Search or ensure the reranker penalizes chunks that don't match the constraint "không lương".

## Case Study (cho presentation)

**Question chọn phân tích:** Lương thử việc của nhân viên Junior mức cao nhất là bao nhiêu?

**Error Tree walkthrough:**
1. Output đúng? → Không.
2. Context đúng? → Không, các chunk trả về thiếu thông tin về lương cho role Junior.
3. Query rewrite OK? → Chưa tốt, từ "Junior" có thể không khớp với "Nhân viên bậc 1" trong văn bản gốc.
4. Fix ở bước: Module 5 (Enrichment) và Module 2 (Search).

**Nếu có thêm 1 giờ, sẽ optimize:**
- Thêm Query Expansion/Rewriting vào Pipeline trước khi gọi Search để chuẩn hóa thuật ngữ (e.g. Junior -> Bậc 1).
- Tăng cường metadata extraction để đính kèm tags role/level vào chunks cho dễ lọc.
