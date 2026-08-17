# Báo Cáo Thu Hoạch Lab 17 - Multi-Memory Agent với Zep

## 1. Trả lời 3 câu hỏi thiết kế & kiến trúc
* **Layer quan trọng nhất:** **Long-term Memory** (chiếm 4/11 ca kiểm thử: E02, E03, E08, E09). Nó đảm bảo lưu trữ bền vững cross-session về sở thích người dùng, open loop task, phân lập dữ liệu người dùng và xử lý xung đột thông tin theo thời gian.
* **Trade-off Zep Cloud vs Redis + Qdrant:** Zep tự động hóa việc xây dựng knowledge graph, trích xuất thực thể, quản lý quan hệ thời gian (temporal validity) và Context Block theo relevance nhưng có độ trễ mạng cao hơn (~0.5–2s). Ngược lại, Redis/Qdrant có độ trễ cực thấp (sub-ms) và chạy local nhưng đòi hỏi phải tự cài đặt toàn bộ pipeline phân tích đồ thị, khử trùng lặp và phân tách namespace.
* **Guardrail chống Memory Poisoning:** Áp dụng phân tách nghiêm ngặt `user_id` namespace, kiểm tra consent trước khi ghi (`memory_opt_in`), lọc bỏ PII nhạy cảm (`privacy_guard`), kiểm tra provenance/nguồn tin và cấm heartbeat tự ý thêm quyền hoặc instruction mới vào durable memory.

## 2. Phân tích kết quả Benchmark
* **Layer có hit rate thấp nhất:** Student benchmark: mọi layer 100%, không có layer thấp nhất. Baseline `no_memory`: `long_term`, `episodic`, `semantic`, `mixed` đạt 0%; `short_term` đạt 100% vì evidence còn trong thread.
* **Case retrieve nhiều token nhất:** **E02** với **1379 tokens**, thuộc Long-term Memory do Context Block kết hợp đồ thị fact edges.
* **Case Mixed (E07):** Phối hợp **Long-term Memory** (sở thích Python của Minh) và **Semantic Memory** (quy tắc retry payment từ KB chung). Bắt buộc phải có cả 2 evidence: `Python` và `Idempotency-Key`.
* **Đánh giá Token Reduction:** Baseline no-memory đạt reduction 81.8% chỉ vì không lấy được dữ liệu dẫn đến fail 9/11 case. Giảm token chỉ có giá trị khi đi kèm Evidence Hit Rate cao (như agent memory đạt 100% hit rate với budget tối ưu).

## 3. Phân tích Recency (E08) & Compaction (E10)
* **E08 (Recency):** Quy tắc *Recency wins* cho phép cập nhật stack dự án `BLUEBIRD-42` thành `TypeScript`/`NestJS`, ghi đè preference cũ mà không bị nhầm lẫn.
* **E10 (Compaction):** Cơ chế sliding window trích xuất `DURABLE_NOTES` giúp bảo toàn hạn chót `REVIEW-DEADLINE-1600 (Friday 16:00)` ngay cả khi tin nhắn gốc đã bị evict khỏi recent turns.
