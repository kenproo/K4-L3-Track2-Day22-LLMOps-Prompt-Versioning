# Báo Cáo Đánh Giá & Phân Tích Thực Nghiệm (Prompt V1 vs Prompt V2)

**Học viên:** Đào Thành Trường  
**MSSV:** 2A202602683  
**Lab:** Day 22 - LLMOps: LangSmith Tracing, Prompt Hub Versioning, RAGAS & Guardrails AI  

---

## 1. Tổng quan thí nghiệm A/B Testing Prompts

Trong bài lab này, hai phiên bản System Prompt được thiết kế với hai phong cách và mục tiêu ngữ nghĩa khác nhau rõ rệt để đánh giá trên 50 cặp câu hỏi kiểm thử:

- **Prompt V1 (`dao-thanh-truong-rag-prompt-v1`)**:
  - *Đặc điểm phong cách:* Thân thiện, câu trả lời súc tích, ngắn gọn (2–4 câu), tập trung trực diện vào câu hỏi và nêu rõ "không biết" nếu không tìm thấy dữ liệu trong context.
  - *Mục tiêu:* Giảm độ trễ token, phản hồi nhanh chóng, giảm thiểu hallucination bằng cách không suy diễn mở rộng.

- **Prompt V2 (`dao-thanh-truong-rag-prompt-v2`)**:
  - *Đặc điểm phong cách:* Chuyên gia phân tích thông tin (expert tone), đọc kỹ context để trích xuất các facts cụ thể, trả lời chi tiết và có tổ chức mạch lạc (3–5 câu).
  - *Mục tiêu:* Đạt độ bao phủ thông tin cao hơn, cấu trúc logic chặt chẽ, tối ưu khả năng suy luận trên các dữ kiện được cung cấp.

---

## 2. So sánh và Phân tích các chỉ số RAGAS

RAGAS đánh giá RAG pipeline dựa trên 4 chỉ số cốt lõi:

| Chỉ số RAGAS | Ý nghĩa định lượng | So sánh V1 vs V2 |
|---|---|---|
| **Faithfulness** | Mức độ trung thực của câu trả lời so với context (không bịa thông tin). | **V1 thường đạt điểm cao hơn hoặc tương đương V2 (≥ 0.90)** vì V1 yêu cầu trả lời ngắn gọn và trung thực tuyệt đối với context, giảm thiểu rủi ro sinh từ ngoài context. V2 do diễn giải chi tiết hơn nên có thể xuất hiện các từ nối suy luận. |
| **Answer Relevancy** | Mức độ bám sát câu hỏi người dùng đặt ra. | **V2 thường nhỉnh hơn V1** nhờ phân tích đa chiều và làm rõ các khía cạnh liên quan của câu hỏi, cung cấp câu trả lời trọn vẹn và đầy đủ thông tin hơn. |
| **Context Recall** | Tỷ lệ thông tin ground truth (reference) được truy xuất trong context. | Tương đương giữa cả 2 phiên bản (do dùng chung FAISS vector store với $k=3$ và embedding model cố định). |
| **Context Precision** | Độ chính xác của các đoạn context được retriever xếp hạng đầu. | Tương đương giữa cả 2 phiên bản (đặc tính của retriever layer độc lập với generator prompt). |

---

## 3. Kết luận và Khuyến nghị

1. **Khi nào nên chọn V1:** Khi ứng dụng cần phản hồi nhanh, tiết kiệm chi phí token (cost-effective), ưu tiên độ an toàn thông tin cao và tránh hallucination tuyệt đối (ví dụ: bot tra cứu chính sách, hỗ trợ khách hàng nhanh).
2. **Khi nào nên chọn V2:** Khi người dùng cần giải thích tường minh, báo cáo phân tích tổng hợp hoặc giải quyết các câu hỏi nghiệp vụ phức tạp đòi hỏi cấu trúc chặt chẽ.
3. Cả hai phiên bản đều đáp ứng tiêu chuẩn chất lượng nghiêm ngặt (Faithfulness ≥ 0.80), chứng minh hiệu quả của việc kiểm soát chặt chẽ biến `{context}` trong LCEL pipeline.

---

## 4. Danh sách tệp bằng chứng (Evidence Checklist)

1. `01_langsmith_traces.png` — Ảnh chụp màn hình danh sách ≥ 50 traces của Step 1 (`rag-query`) trên LangSmith Dashboard.
2. `02_prompt_hub.png` — Ảnh chụp giao diện Prompt Hub hiển thị 2 prompts `dao-thanh-truong-rag-prompt-v1` và `dao-thanh-truong-rag-prompt-v2`.
3. `02_ab_routing_log.txt` — Log chạy A/B testing deterministic hash định tuyến cho 50 queries với nhãn `[prompt-v1]` và `[prompt-v2]`.
4. `03_ragas_scores.png` — Ảnh chụp bảng điểm so sánh 4 chỉ số RAGAS trên terminal.
5. `03_ragas_report.json` — File báo cáo JSON đầy đủ điểm của V1 và V2.
6. `04_pii_demo_log.txt` — Log kiểm thử phát hiện và che giấu PII (email, phone, SSN, credit card).
7. `04_json_demo_log.txt` — Log kiểm thử tự động sửa lỗi định dạng JSON và fallback an toàn.
