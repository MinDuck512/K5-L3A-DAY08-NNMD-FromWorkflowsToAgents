# Research card

File này nằm cố định ở `report/team/research_card.md`; `python -m lab8.check_submission` đọc cột "Nhóm điền" bằng máy.

Phần này nối tiếp phần mini research ở buổi lý thuyết (slide 62–63). Chọn **một** câu R1–R4, điền 6 dòng, rồi biến nó thành **một ablation** chạy được trong lab: giữ nguyên model (MockLLM), tool, seed, corpus và giá, chỉ thay đổi một biến.

| Câu | Nguồn (thư viện tham khảo, slide 68–69) | Câu hỏi gợi ý cho lab |
| --- | --- | --- |
| R1 · Scaling | Google Research 2026, *Towards a Science of Scaling Agent Systems* | Task claims chủ yếu tuần tự hay song song được? Dự đoán thêm investigator sẽ làm pass rate và cost/success thay đổi thế nào. |
| R2 · Failure taxonomy | MAST, NeurIPS 2025, *Why Do Multi-Agent LLM Systems Fail?* | Các trial fail của swarm (seed 55) thuộc nhóm nào: lỗi giao việc, lệch mục tiêu, hay lỗi kiểm chứng? |
| R3 · Architecture choices | AgentArch, AACL 2026 | Nếu đổi topology mà giữ nguyên mọi thứ khác, biến nào phải cố định để kết luận còn đúng? |
| R4 · Evaluation confounders | EMNLP 2026, *Rethinking Evaluation of MAS Efficiency* | Làm sao chứng minh chênh lệch giữa A/B/C/D là do topology, không phải do prompt, tool hay seed? |

## Phiếu của nhóm

Điền vào cột cuối. Không xóa hai cột đầu.

| Field | Hướng dẫn | Nhóm điền |
| --- | --- | --- |
| CÂU | R1 / R2 / R3 / R4 | R3 — Architecture choices; ablation thành phần draft trong cùng một topology Orchestrator. |
| SOURCE | Paper/blog, năm, URL, loại bằng chứng (benchmark, production report, blog kinh nghiệm) | Bogavelli, Subramani và Sharma (2026), [AgentArch: A Benchmark for Evaluating Agent Architectures in Enterprise Workflows](https://aclanthology.org/2026.aacl-long.44/), AACL-IJCNLP; bằng chứng benchmark nghiên cứu đã công bố. |
| CLAIM | Một kết luận gốc của nguồn. Không viết rộng hơn phạm vi paper | Trong benchmark của nguồn, lựa chọn kiến trúc phù hợp phụ thuộc model và tác vụ; không có một thiết kế tối ưu cho mọi trường hợp. Các thành phần kiến trúc cần được đánh giá bằng thí nghiệm. |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | Nguồn đánh giá 18 cấu hình với LLM và tác vụ doanh nghiệp, xét orchestration, ReAct/function calling, memory và thinking tool. Lab này chỉ dùng MockLLM và 14 claim tổng hợp; thí nghiệm dưới đây không tái lập kết quả của paper. |
| HYPOTHESIS | Nếu đổi shape A→B (hoặc bật/tắt một thành phần), metric X sẽ đổi theo hướng nào, khoảng bao nhiêu | Đặt trước khi chạy: tắt riêng lượt draft cuối của Orchestrator giữ pass rate/pass^5 ở 1.0, variance 0 trên public; giảm 1 model call/trial và cost/success khoảng 1–2 VND mô phỏng. Vì draft không quyết status, quote, approval hoặc ledger. |
| METHOD | Biến giữ cố định: model, tools, seed, corpus, giá, evaluator. Biến duy nhất được thay đổi: … | So orchestrator với orchestrator_no_draft. Dùng chung implementation trong lab8/shapes/orchestrator.py; chỉ đổi cờ draft_reply. Giữ MockLLM, tool/handoff, policy/ledger, toàn bộ public, seed 11,22,33,44,55, giá và evaluator; mỗi trial SQLite mới. Metadata shape/run_id tách riêng để không ghi đè trace. Đăng ký biến thể bằng --extra-shape, không sửa registry hay thước đo. |
| FAILURE / METRIC | Một lỗi dự kiến và cách đo nó: pass rate, pass^5, variance, cost/success, leak, bypass | Đo pass rate, pass^5, seed variance, cost/success, model/tool calls. Test cặp kiểm status, quote, effect, audit và tool count không đổi; trace chỉ bớt event draft. Kiểm crash/resume của biến thể ở cả 3 điểm. Nếu mất kiểm duyệt hoặc đổi tiền thì bác bỏ giả thuyết, không sửa JSON để hợp kết quả. |

## Kết quả ablation

- Lệnh đã chạy (copy nguyên văn, trong `src/`):

```powershell
& '.\.venv\Scripts\python.exe' -m lab8.benchmark --shapes orchestrator,orchestrator_no_draft --extra-shape orchestrator_no_draft=lab8.shapes.orchestrator:run_claim_no_draft --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/bonus_ablation.json
```

- Run id / file report: `src/reports/bonus_ablation.json`, 140 trial; ví dụ cặp `R-orchestrator-C-014-s55` và `R-orchestrator_no_draft-C-014-s55`. Tên file dùng mẫu `bonus_*.json` đã quy định cho thí nghiệm mở rộng; đây là ablation bắt buộc của research card, không phải tuyên bố hoàn thành bonus B4.
- Kết quả so với hypothesis: **ủng hộ trên public**, theo số liệu từ JSON:

| Metric | Orchestrator | Tắt riêng draft |
| --- | --- | --- |
| Trial đúng | 70/70 | 70/70 |
| Pass rate / pass^5 | 1.0 / 1.0 | 1.0 / 1.0 |
| Seed variance | 0 | 0 |
| Cost/success (VND mô phỏng) | 50.712857 | 49.607143 |
| Mean model calls | 2.857143 | 1.857143 |
| Mean tool calls | 4.642857 | 4.642857 |
| Mean / p95 trace events | 8.857143 / 10 | 7.857143 / 9 |

Giảm 1.105714 VND/thành công (2.180343%), đúng 1 model call/trial. Đối chiếu đủ 70 cặp cho thấy status, passed/reason và tool count không đổi; baseline của thí nghiệm khớp toàn bộ trial và thống kê Orchestrator trong `benchmark.json`. Sáu test cặp kiểm thêm quote, effect, audit và trace chỉ mất event draft; ba test kiểm biến thể crash/resume. Các test approval/injection vẫn xanh. Tổng sau bổ sung: `92 passed` mặc định, `248 passed` student.

- Giới hạn: MockLLM không phải model sản xuất, latency là `unavailable`. Grader không chấm câu chữ trả lời khách và draft không đi vào tính tiền; không suy ra draft vô ích trong sản phẩm thật. Đây là ablation thành phần trong cùng topology, không cô lập toàn bộ tác động của topology hoặc tái lập benchmark AgentArch. Chỉ có 14 claim tổng hợp và chưa có hidden. Workflow vẫn rẻ hơn cả hai bản, nên giữ lựa chọn trong ADR.

Muốn đi xa hơn một ablation để lấy điểm cộng thì xem [docs/bonus_topics.md](../../docs/bonus_topics.md).
