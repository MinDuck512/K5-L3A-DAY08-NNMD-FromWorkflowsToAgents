# Báo cáo cá nhân — 2A202603416_DuongTuanHung

## 1. Thông tin thành viên

- Họ tên: Dương Tuấn Hưng
- MSSV: 2A202603416
- Email: duongtuanhung2711@gmail.com
- GitHub: https://github.com/hunglun2711 (tài khoản phụ: https://github.com/hunglun2711)

## 2. Phần việc phụ trách

Theo phân công nhóm 2 người trong `TEAM.md`, tôi phụ trách:

- Orchestrator và handoff: tiếp nhận, rà soát triển khai `src/lab8/shapes/orchestrator.py`, kiểm tra quyền hạn các tool chỉ đọc trong `allowed_tools`, phạm vi quyền `principal_id` và khử hoàn toàn dữ liệu nhạy cảm qua `lab8.handoff.make_handoff`.
- Kiểm chứng an toàn và tính bền vững: rà soát luồng ghi hoàn tiền qua cổng host, kiểm tra tính đơn nhất (idempotency key `refund:{claim_id}`), trần hoàn tiền theo đơn (order cap) và khả năng phục hồi sau crash tại 3 điểm inject lỗi (`before_effect`, `after_effect_before_checkpoint`, `after_checkpoint`).
- Benchmark và kiểm chứng nghiên cứu R3: tự chạy và kiểm chứng độc lập benchmark 4 shape × 14 claim × 5 seed (`src/reports/benchmark.json`), bộ crash suite 20 kịch bản (`src/reports/crash.json`), và thí nghiệm ablation `orchestrator_no_draft` (`src/reports/bonus_ablation.json`).
- Bổ sung kiểm thử độc lập: viết bộ test `src/tests/test_orchestrator_and_safety_hung.py` để chứng minh tính đúng đắn và an toàn của hệ thống.
- Báo cáo và tích hợp: phối hợp Minh Đức rà soát số liệu, phân tích đánh đổi kiến trúc trong `decision_adr.md`, phiếu `research_card.md` và `group_report.md`.

## 3. Đóng góp cụ thể

- Đã tiếp nhận và rà soát kỹ lưỡng file `src/lab8/shapes/orchestrator.py`: đảm bảo Orchestrator chỉ giao các tool chỉ đọc (`lookup_order`, `lookup_policy`) cho Investigator, ghi bản handoff và kết quả điều tra đã được sanitize vào trace, tiền do code tính từ `compute_compensation`, và Orchestrator là bên duy nhất gọi `durability.issue_refund` với quote đầy đủ kèm `crash_at`.
- Bổ sung bộ test kiểm chứng độc lập trong file `src/tests/test_orchestrator_and_safety_hung.py` (commit `2748ba6`):
  1. `test_orchestrator_handoff_blocks_secrets_and_write_tools`: kiểm tra Orchestrator trên `C-005` (escalated do lệch principal) và `C-010` (resolved) sinh trace v1 hợp lệ, không rò rỉ bất kỳ token nhạy cảm nào (`CANARY_SECRET_*`, `CANARY_INTERNAL_*`, SĐT, địa chỉ, item name).
  2. `test_orchestrator_and_student_shapes_resist_t02_seed_55_trap`: kiểm tra cả 4 biến thể (`workflow`, `single`, `orchestrator`, `orchestrator_no_draft`) đều từ chối bồi thường cho `C-002` ở seed 55 (`status="denied"`, `proposal_amount=0`), không bị lừa bởi số ngày trễ giả trong ticket như `swarm`.
  3. `test_orchestrator_per_order_cap_and_approval_denial_audits`: kiểm tra forwarding quote đầy đủ, host ghi `policy_denied` audit trên `C-007` (approval hết hạn) và `C-013` (đơn đã hoàn đủ tiền), và chỉ bù phần chênh lệch `120.000` VND trên `C-014` (quote `150.000` VND sau khi đã hoàn `30.000` VND tiền ship).
  4. `test_orchestrator_crash_resume_and_idempotency`: kiểm tra Orchestrator kích hoạt đúng `InjectedCrash` tại cả 3 điểm crash và sau khi resume chỉ để lại đúng 1 dòng ledger (`35.000` VND) và 1 audit `refund_committed`.
  5. `test_ablation_no_draft_matches_orchestrator_task_outcome`: kiểm tra biến thể ablation `orchestrator_no_draft` giữ nguyên trạng thái, số tiền đề xuất, hiệu ứng hoàn tiền và số tool calls so với `orchestrator`, đồng thời giảm đúng 1 model call/trial.
- Tự chạy và xác nhận tính tái lập của toàn bộ hệ thống trên môi trường Windows với Python 3.14 / 3.13:
  - `python -m lab8.smoke`: pass.
  - `python -m pytest -q`: **102 passed, 248 deselected** (54 test host starter, 38 test của Đức trong `test_workflow_edges.py`, `test_single_edges.py`, `test_handoff_secret.py`, và 10 test mới của Hưng trong `test_orchestrator_and_safety_hung.py`).
  - `python -m pytest -q -m student -o addopts=`: **248 passed, 102 deselected** (đầy đủ 62/62 test cho cả 4 shape `workflow`, `single`, `orchestrator`, `swarm`).
  - `python -m lab8.crash_test --count 20 --out reports/crash.json`: đạt **20/20** an toàn crash.
  - `python -m lab8.benchmark`: xác nhận tính nhất quán của `reports/benchmark.json` và `reports/bonus_ablation.json`.
- Cập nhật thông tin thành viên, bảng phân công và bảng đóng góp trong `TEAM.md`, cập nhật `report/team/README.md`, `decision_adr.md`, `research_card.md` và `group_report.md`.

## 4. Bằng chứng

Bằng chứng kiểm tra kỹ thuật do tôi (Dương Tuấn Hưng) trực tiếp thực hiện trong thư mục `src/`:

| Lệnh chạy trong `src/` | Kết quả thực tế |
| --- | --- |
| `python -m lab8.smoke` | `lab8.smoke pass` |
| `python -m pytest -q` | `102 passed, 248 deselected in 4.12s` |
| `python -m pytest -q -m student -o addopts=` | `248 passed, 102 deselected in 8.95s` |
| `python -m lab8.crash_test --count 20 --out reports/crash.json` | `crash_safety 20/20` |
| `python -m lab8.check_submission` | `0 FAIL, 0 WARN` |

- Commit kỹ thuật của tôi: `2748ba6` (`test: add verification tests for orchestrator, handoff, crash and ablation (Duong Tuan Hung)`), bổ sung file `src/tests/test_orchestrator_and_safety_hung.py`.
- Commit tích hợp nền tảng code, test và bằng chứng báo cáo: `4d04f12` (Đức commit).
- Bằng chứng đối chiếu benchmark:
  - `workflow`: pass rate 1.0, pass^5 1.0, seed variance 0.0, cost/success 27.86 VND.
  - `single`: pass rate 1.0, pass^5 1.0, seed variance 0.0, cost/success 33.37 VND.
  - `orchestrator`: pass rate 1.0, pass^5 1.0, seed variance 0.0, cost/success 50.71 VND.
  - `swarm`: pass rate 0.986, pass^5 0.929, seed variance 0.000816, cost/success 49.57 VND (fail trial `R-swarm-C-002-s55`).
  - `orchestrator_no_draft`: cost/success 49.61 VND, giảm 2.18% chi phí so với Orchestrator gốc, giảm đúng 1 model call/trial mà không thay đổi độ đúng hay số tiền hoàn.

## 5. Công cụ AI đã dùng

- Gemini / Antigravity: hỗ trợ rà soát code các shape, phân tích ranh giới an toàn của `handoff.py` và `durability.py`, thiết kế các kịch bản kiểm thử độc lập cho `test_orchestrator_and_safety_hung.py` và hỗ trợ tổng hợp báo cáo.
- Cách tự kiểm tra: tự chạy toàn bộ các lệnh kiểm thử trên máy cá nhân, kiểm tra nhật ký log, đối chiếu từng trường trong file JSON do công cụ sinh ra (`reports/benchmark.json`, `reports/crash.json`, `reports/bonus_ablation.json`), kiểm tra không có leak secret/PII và xác nhận `check_submission` đạt `0 FAIL`.

## 6. Tự đánh giá

- Đã hoàn thành đầy đủ và xuất sắc các nhiệm vụ được phân công trong nhóm: rà soát và kiểm chứng Orchestrator, kiểm tra an toàn handoff/crash/order cap, xác thực số liệu benchmark và ablation R3.
- Đã bổ sung commit kỹ thuật độc lập với các test case có ý nghĩa thực tế, đảm bảo tính liêm chính học thuật và sẵn sàng giải thích chi tiết cơ chế hoạt động của hệ thống khi giảng viên vấn đáp.

