# Research card

## Phiếu của nhóm

| Field | Hướng dẫn | Nhóm điền |
| --- | --- | --- |
| CÂU | R1 / R2 / R3 / R4 | R3 — Architecture choices: tắt một thành phần trong Orchestrator. |
| SOURCE | Paper/blog, năm, URL, loại bằng chứng (benchmark, production report, blog kinh nghiệm) | Bogavelli, Subramani và Sharma (2026), [AgentArch](https://aclanthology.org/2026.aacl-long.44/), AACL-IJCNLP; benchmark nghiên cứu. |
| CLAIM | Một kết luận gốc của nguồn. Không viết rộng hơn phạm vi paper | Kiến trúc phù hợp phụ thuộc model và tác vụ; cần đánh giá từng lựa chọn bằng số liệu. |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | Nguồn đánh giá 18 cấu hình trên tác vụ doanh nghiệp. Lab này chỉ có MockLLM và 14 claim tổng hợp, không tái lập benchmark của paper. |
| HYPOTHESIS | Nếu đổi shape A→B (hoặc bật/tắt một thành phần), metric X sẽ đổi theo hướng nào, khoảng bao nhiêu | Trước khi đo: tắt riêng draft cuối vẫn giữ pass rate/pass^5 = 1, variance = 0; giảm 1 model call/trial và khoảng 1–2 VND/thành công. |
| METHOD | Biến giữ cố định: model, tools, seed, corpus, giá, evaluator. Biến duy nhất được thay đổi: … | So Orchestrator với orchestrator_no_draft, chỉ đổi cờ draft_reply. Giữ model, tool/handoff, policy/ledger, 14 claim, 5 seed 11,22,33,44,55, giá và evaluator; SQLite mới mỗi trial. Tách run_id để giữ trace riêng. |
| FAILURE / METRIC | Một lỗi dự kiến và cách đo nó: pass rate, pass^5, variance, cost/success, leak, bypass | Đo độ đúng, pass^5, variance, cost/success, model/tool calls. Test cặp kiểm tiền, audit và trace không đổi ngoài draft; kiểm thêm cả 3 điểm crash. |

## Kết quả ablation

Chạy trong `src/`:

```powershell
& '.\.venv\Scripts\python.exe' -m lab8.benchmark --shapes orchestrator,orchestrator_no_draft --extra-shape orchestrator_no_draft=lab8.shapes.orchestrator:run_claim_no_draft --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/bonus_ablation.json
```

- File: [bonus_ablation.json](../../src/reports/bonus_ablation.json), 140 trial do công cụ sinh.
- Run ID ví dụ: `R-orchestrator-C-014-s55` và `R-orchestrator_no_draft-C-014-s55`.

| Metric | Có draft | Tắt draft |
| --- | --- | --- |
| Đúng | 70/70 | 70/70 |
| Pass rate / pass^5 / variance | 1 / 1 / 0 | 1 / 1 / 0 |
| Cost/success (VND mô phỏng) | 50,712857 | 49,607143 |
| Mean model calls | 2,857143 | 1,857143 |
| Mean tool calls | 4,642857 | 4,642857 |
| Mean/p95 trace events | 8,857143 / 10 | 7,857143 / 9 |

Giả thuyết được ủng hộ trên public: giảm **2,18% chi phí**, bớt 1 model call/trial, giữ độ đúng. Đủ 70 cặp có status/tool count như nhau; baseline khớp benchmark chính. Sáu test cặp kiểm thêm tiền/audit/trace và ba test kiểm crash đều đạt.

## Phân công và giới hạn

Hưng phụ trách tự tái lập/kiểm chứng; Đức rà soát code và tích hợp ADR. Số liệu hiện có do Codex chạy, chưa có kết quả tự chạy của Hưng.

MockLLM, chi phí mô phỏng, latency `unavailable`; chưa có hidden. Grader không chấm câu chữ nên chưa kết luận về chất lượng phục vụ khách. Đây là ablation thành phần, không chứng minh riêng tác động topology hay hoàn thành bonus B4. Workflow vẫn rẻ nhất.
