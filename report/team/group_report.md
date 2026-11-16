# Báo cáo nhóm — Lab 8 LogiX-VN

Tối đa 5 trang ngoài phụ lục. Số liệu lấy từ file trong `src/reports/`, không chép tay từ terminal.

## 1. Thông tin nhóm

- Tên nhóm: NNMD — L3A
- Thành viên: xem `TEAM.md`
- Commit chốt (trùng `report/team/decision_adr.md`): `4d04f12e3b7b425861c4968d61475ee5a36a744f` (code, test và bằng chứng; cập nhật hash bằng commit tài liệu kế tiếp)

## 2. Mục tiêu và cách tiếp cận

Xử lý yêu cầu hoàn phí/bồi thường LogiX-VN theo luật nghiệp vụ, rồi chọn mức tự chủ thấp nhất vẫn đúng và an toàn. Triển khai lần lượt workflow, single và orchestrator; giữ swarm tham chiếu để đối chứng. Thành viên hiện khai báo: Nguyễn Ngọc Minh Đức, MSSV 2A202603421, với sự hỗ trợ của Codex. Thành viên đã tự chạy bộ test mặc định và xác nhận 92 passed sau khi đổi thư mục tạm pytest; benchmark và student bên dưới do Codex chạy, chi tiết tự kiểm tra ghi trong báo cáo cá nhân.

Benchmark giữ nguyên 14 claim public, seed `11,22,33,44,55`, MockLLM, bảng giá, tool và evaluator chính thức; mỗi trial có SQLite mới. Tổng cộng 4 × 14 × 5 = 280 trial, `as_of` cố định 2026-11-15 theo config của lab.

## 3. Giải pháp

| Shape | Cách xử lý |
| --- | --- |
| Workflow | Code đọc claim, tra đơn theo principal, gọi `compute_compensation`, sau đó gửi quote đầy đủ cho host. Không gọi model. |
| Single | `run_react(max_steps=8, worker=True)` chạy vòng đọc/quan sát; adapter chọn action từ kết quả tool và đếm MockLLM mỗi vòng. Code kiểm dữ kiện và gọi host sau khi vòng lặp kết thúc. |
| Orchestrator | Tạo handoff qua `make_handoff`, chỉ cấp `lookup_order` và `lookup_policy`. Investigator trả order view đã lọc; orchestrator tính tiền bằng code và gọi host. Đếm các lượt MockLLM plan/investigate/draft. |

Tiền do code tính từ order và policy hiện hành; ticket không quyết số ngày trễ, mã đơn hay quyền hoàn tiền. Host kiểm approval đúng claim, trần theo đơn và idempotency `refund:{claim_id}`. Shape không tự trừ phần đã hoàn; ví dụ T14 quote 150.000 VND, host chỉ ghi thêm 120.000 VND sau khoản lịch sử 30.000 VND.

Handoff không chuyển ticket, secret, header hay parent context; investigator chỉ có tool đọc. Trace chỉ ghi dữ kiện đã lọc và quyết định, không ghi SĐT, địa chỉ, internal note, tên hàng. `finish()` tạo kết quả từ ledger và ghi trace ở mọi nhánh trả về. Các shape truyền `crash_at` vào host, để `InjectedCrash` truyền ra ngoài và luôn đóng connection trong `finally`.

## 4. Kết quả

Số liệu từ [benchmark.json](../../src/reports/benchmark.json), làm tròn khi trình bày:

| Shape | Trial đúng | Pass rate | Pass^5 | Seed min–max | Variance | Cost/success (VND mô phỏng) | Mean/p95 steps |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Workflow | 70/70 | 1,000000 | 1,000000 | 1–1 | 0 | 27,86 | 5,07 / 6 |
| Single | 70/70 | 1,000000 | 1,000000 | 1–1 | 0 | 33,37 | 8,86 / 10 |
| Orchestrator | 70/70 | 1,000000 | 1,000000 | 1–1 | 0 | 50,71 | 8,86 / 10 |
| Swarm | 69/70 | 0,985714 | 0,928571 | 0,928571–1 | 0,000816327 | 49,57 | 2,80 / 3 |

Swarm chỉ fail `R-swarm-C-002-s55`: ticket ghi trễ 5 ngày, order ghi 2 ngày nhưng swarm tin ticket và trả `resolved` thay vì `denied`. Đây là điểm yếu có chủ đích của bản tham chiếu. Bộ student xanh vẫn kiểm tra ca này là một lỗi dự kiến, không coi swarm đúng 100%.

[Crash suite](../../src/reports/crash.json) đạt **20/20**, kiểm host và swarm. Crash của ba shape tự viết được kiểm thêm qua T12 và test bổ sung ở ba thời điểm before effect, after effect/before checkpoint và after checkpoint. Lần kiểm tích hợp trước benchmark: student `248 passed`; mặc định `83 passed`, gồm 29 ca mới.

Lệnh tái lập (chạy trong `src/`, dùng Python trong môi trường đã kích hoạt):

```text
python -m lab8.crash_test --count 20 --out reports/crash.json
python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json
```

Chưa có `student_drop/`, nên chưa chạy hidden self-check và chưa có pass rate hidden chính thức. Chọn workflow dựa trên độ đúng ngang single/orchestrator và cost/success thấp nhất trên public.

Quyết định kiến trúc và lý do: xem `report/team/decision_adr.md`. Phiếu nghiên cứu và ablation: xem `report/team/research_card.md`.

Ablation R3 tắt riêng lượt draft cuối của Orchestrator, giữ nguyên các thành phần khác. [bonus_ablation.json](../../src/reports/bonus_ablation.json) ghi 70 trial mỗi bản: cả hai có pass rate/pass^5 = 1, variance = 0; cost/success giảm từ 50,712857 xuống 49,607143 VND mô phỏng (2,18%). Mean model calls giảm đúng 1, tool count không đổi; baseline khớp benchmark chính. Chín test mới kiểm sự tương đương tiền/trace và crash; kiểm tra cuối: mặc định `92 passed`, student `248 passed`. Draft không được grader chấm về câu chữ, nên chưa đánh giá chất lượng phục vụ khách. Workflow vẫn rẻ nhất.

## 5. Khó khăn và cách xử lý

- Máy ban đầu có Python 3.14; bản Python standalone do uv tải bị Application Control chặn (`os error 4551`). Cài Python 3.13 chính thức riêng cho dự án, tạo `src/.venv` và cài requirements. Kiểm tra bằng `.\.venv\Scripts\python.exe --version` trong `src/` ra Python 3.13.17; không đổi chính sách Application Control. Đã dọn bản tải thử/cache không dùng.
- Lọc pytest bằng `-k single` chọn cả hàm tên `test_public_single_run_cases` của shape khác, gây 60 lỗi orchestrator ở thời điểm chưa viết shape này. Đổi thành `-k "single and not workflow and not orchestrator and not swarm"` đạt 62/62; sau khi hoàn tất cả ba shape, chạy toàn bộ student đạt 248/248.
- Thành viên tự chạy pytest trên Windows gặp `17 passed, 248 deselected, 75 errors`: tất cả 75 lỗi ở bước tạo `tmp_path`, do `PermissionError: [WinError 5]` khi đọc `C:\Users\ADMIN\AppData\Local\Temp\pytest-of-ADMIN`. Đổi sang thư mục tạm riêng bằng `.\.venv\Scripts\python.exe -m pytest -q --basetemp=.venv/pytest-local` trong `src/`; thành viên xác nhận `92 passed, 248 deselected in 2.24s`. Không sửa code nghiệp vụ, bộ chấm hay quyền thư mục Temp hệ thống. Thư mục tạm nằm trong `.venv/` đã gitignore.

## 6. Tổng hợp đóng góp

Chi tiết đóng góp của từng thành viên nằm trong `TEAM.md` (bảng đóng góp) và báo cáo cá nhân `report/team/MSSV_HoTen.md`.

## 7. Hạn chế và hướng cải thiện

- MockLLM mô phỏng văn bản/token; single chọn action qua adapter code, orchestrator dùng kế hoạch tool cố định. Kết quả chưa đo được năng lực tự chủ của model thật.
- Cost là giá mô phỏng, latency là `unavailable`. `steps` là số event trace, không phải thời gian chạy; swarm ghi log ít hơn các shape khác nên không dùng số steps thấp để kết luận nhanh hơn.
- Mỗi shape có prompt và số lượt model/tool khác nhau. Chênh lệch hiện tại phản ánh toàn bộ triển khai, chưa cô lập tác động topology. Public có ít case và hidden chưa được chấm.
- Có thể đổi lựa chọn nếu task mới cần xử lý dữ liệu phi cấu trúc hoặc có việc độc lập để phân công, và phép đo mới cho thấy lợi ích bù được chi phí mà vẫn qua gate an toàn.
- Ablation chỉ tắt một thành phần trong Orchestrator, không chứng minh tác động riêng của topology hoặc thay thế đánh giá chất lượng câu trả lời khách. Research card và commit code/bằng chứng đã chốt trong ADR. Thành viên đã xác nhận test mặc định; cần tiếp tục tự chạy student và củng cố phần giải thích code trước khi nộp.

## 8. Tài liệu tham khảo

- [Luật nghiệp vụ](../../docs/business_rules.md), [contract kết quả/trace](../../docs/contracts.md), [kiến trúc](../../docs/architecture.md), [định nghĩa metric](../../docs/metrics.md).
- [Quy định bài lab](../../RULES.md), [yêu cầu bài nộp](../../SUBMISSION.md), [cách tạo bằng chứng](../../src/reports/README.md).
- Bogavelli, Subramani và Sharma (2026), [AgentArch](https://aclanthology.org/2026.aacl-long.44/), nguồn nghiên cứu cho câu R3; xem phạm vi và giả thuyết riêng của lab trong research card.
