# Báo cáo cá nhân — 2A202603421_NguyenNgocMinhDuc

## 1. Thông tin thành viên

- Họ tên: Nguyễn Ngọc Minh Đức
- MSSV: 2A202603421
- Email: Chưa cung cấp
- GitHub: https://github.com/MinDuck512

## 2. Phần việc phụ trách

Theo phân công nhóm 2 người, tôi phụ trách:

- Workflow và Single: rà soát luồng xử lý, vòng ReAct, quote bằng code, ranh giới model/tool và test nghiệp vụ tương ứng.
- Tích hợp: môi trường Python, Git, xử lý lỗi Windows, giữ các shape cùng contract và kiểm tra bài nộp.
- Báo cáo: giải pháp Workflow/Single, tổng hợp ADR, đối chiếu hash và bằng chứng; phối hợp Hưng rà soát kết luận.

Dương Tuấn Hưng tiếp nhận Orchestrator/handoff, kiểm chứng an toàn, benchmark/ablation và báo cáo kết quả. Hai người kiểm tra chéo phần việc; Hưng sẽ tự bổ sung commit/test và báo cáo cá nhân.

## 3. Đóng góp cụ thể

Các mục đã triển khai dưới đây mô tả bản tích hợp đầu tiên `4d04f12` do tôi commit với hỗ trợ Codex, trước khi cập nhật phân công đủ 2 người. Trách nhiệm rà soát/tiếp nhận hiện tại theo mục 2 và `TEAM.md`.

- Đã triển khai `src/lab8/shapes/workflow.py` với sự hỗ trợ của Codex: dùng tool đọc an toàn, tính quote bằng code, gửi quote đầy đủ cho host, trả `RunResult` và trace ở mọi nhánh kết thúc.
- Đã bổ sung `src/tests/test_workflow_edges.py`: 10 ca kiểm thử, gồm claim/đơn không tồn tại, ticket chứa injection và dữ liệu nhạy cảm, approval không hợp lệ, quote thay đổi khi retry, và 3 điểm crash.
- Đã triển khai `src/lab8/shapes/single.py` với sự hỗ trợ của Codex: dùng `run_react(max_steps=8, worker=True)`, đếm MockLLM/token theo từng vòng, giữ các dữ kiện tool đã đọc và chỉ ghi tiền sau khi vòng lặp kết thúc.
- Đã bổ sung `src/tests/test_single_edges.py`: 10 ca kiểm thử, gồm ticket/model text đối kháng, giới hạn 8 vòng, claim/đơn không tồn tại, approval sai và 3 điểm crash.
- Đã triển khai `src/lab8/shapes/orchestrator.py` với sự hỗ trợ của Codex: handoff allowlist chỉ giao tool đọc, investigator tra đơn/policy, orchestrator tính quote bằng code và gọi host. Các lượt MockLLM plan/investigate/draft được đếm trong budget; không dùng văn bản model để quyết tiền hay tạo approval.
- Đã bổ sung 9 ca Orchestrator trong file có sẵn `src/tests/test_handoff_secret.py`, giữ nguyên các test cũ. Kiểm tra handoff/prompt/trace không mang dữ liệu nhạy cảm, investigator chỉ đọc, approval sai, nhánh thiếu dữ kiện, các điểm crash và conflict khi retry.
- Đã chạy crash suite và benchmark 4 shape trên toàn bộ public; đã cập nhật kết quả vào `group_report.md` và lựa chọn workflow vào `decision_adr.md`.
- Đã hoàn thành research card R3 với nguồn AgentArch đã đối chiếu từ ACL Anthology. Thêm biến thể tắt riêng draft ngay trong `orchestrator.py`, dùng chung logic để giữ nguyên các thành phần khác; thêm 9 test cặp/crash vào file test hiện có và chạy ablation bằng benchmark chính thức.
- Commit kỹ thuật: `4d04f12e3b7b425861c4968d61475ee5a36a744f`, chứa code, test và bằng chứng. Hidden chưa được cung cấp. Thành viên đã tự chạy lại bộ test mặc định và xác nhận 92 passed sau khi xử lý lỗi quyền thư mục tạm pytest.

## 4. Bằng chứng

Bằng chứng chuẩn bị môi trường (CP1), do Codex chạy trên Windows với Python 3.13.17:

| Lệnh chạy trong `src/` | Kết quả ban đầu |
| --- | --- |
| `.\.venv\Scripts\python.exe -m lab8.smoke` | `lab8.smoke pass` |
| `.\.venv\Scripts\python.exe -m pytest -q` | `54 passed, 248 deselected` |
| `.\.venv\Scripts\python.exe -m pytest -q -m student -o addopts= --tb=no` | `186 failed, 62 passed, 54 deselected` |

186 test student thất bại là trạng thái ban đầu đúng của starter: workflow, single và orchestrator chưa triển khai. 62 test của swarm đã đạt. Đây chưa phải kết quả cuối cùng để nộp bài.

Môi trường: `src/.venv`. Python 3.13 chính thức được cài riêng tại `.venv/python-official` ở gốc repo, sau khi Windows Application Control chặn bản Python standalone do uv tải. Không sửa chính sách Application Control.

Bằng chứng bước workflow (CP2 và phần xử lý crash của CP3), do Codex chạy:

| Lệnh chạy trong `src/` | Kết quả |
| --- | --- |
| `.\.venv\Scripts\python.exe -m pytest -q -m student -o addopts= -k workflow` | `62 passed, 250 deselected` |
| `.\.venv\Scripts\python.exe -m pytest -q` | `64 passed, 248 deselected` (gồm 10 ca bổ sung) |
| `.\.venv\Scripts\python.exe -m lab8.run --shape workflow --claim-id C-001 --seed 11 --db .venv/workflow-cp2-c001.sqlite` | `resolved`, quote 35.000 VND, ghi 35.000 VND, 1 audit committed |
| `.\.venv\Scripts\python.exe -m lab8.run --shape workflow --claim-id C-014 --seed 11 --db .venv/workflow-cp2-c014.sqlite` | `resolved`, quote 150.000 VND, ghi thêm 120.000 VND vì đơn đã nhận 30.000 VND, 1 audit committed |

Hai lần chạy minh họa đều có 0 model call và 3 read tool call; cost 30 VND là giá mô phỏng của lab. Chưa có kết quả hidden hay benchmark đủ 4 shape.

Bằng chứng bước single agent, do Codex chạy:

| Lệnh chạy trong `src/` | Kết quả |
| --- | --- |
| `.\.venv\Scripts\python.exe -m pytest -q -m student -o addopts= -k "single and not workflow and not orchestrator and not swarm"` | `62 passed, 260 deselected` |
| `.\.venv\Scripts\python.exe -m pytest -q` | `74 passed, 248 deselected` (gồm 10 ca mới cho single) |
| `.\.venv\Scripts\python.exe -m lab8.run --shape single --claim-id C-001 --seed 11 --db .venv/single-cp2-c001.sqlite` | `resolved`, quote và refund 35.000 VND, 1 audit committed; 4 model call, 3 read tool call, cost mô phỏng 36,14 VND |
| `.\.venv\Scripts\python.exe -m lab8.run --shape single --claim-id C-002 --seed 55 --db .venv/single-cp2-c002.sqlite` | `denied`, quote 0, không refund: đơn trễ 2 ngày dù ticket nói 5 ngày |

Lưu ý lệnh test: `-k single` cũng khớp tên hàm `test_public_single_run_cases` của các shape khác, vì vậy cần bộ lọc đầy đủ ở trên. Lần chạy bộ lọc ngắn đã báo 60 lỗi của orchestrator chưa triển khai; bộ lọc riêng single sau đó đạt 62/62.

Giới hạn mô phỏng: MockLLM không sinh function call có cấu trúc. `_read_action()` là adapter chọn tool bằng code từ observation; văn bản planning của MockLLM không được dùng để quyết tiền hay sinh approval. Vì vậy kết quả ở đây kiểm chứng vòng lặp, ranh giới quyền và chi phí mô phỏng, chưa chứng minh năng lực tự chọn tool của model thật.

Bằng chứng bước orchestrator và kiểm tra tích hợp ba shape, do Codex chạy:

| Lệnh chạy trong `src/` | Kết quả |
| --- | --- |
| `.\.venv\Scripts\python.exe -m pytest -q -m student -o addopts=` | `248 passed, 83 deselected`: đủ 62 ca/shape trên workflow, single, orchestrator và swarm |
| `.\.venv\Scripts\python.exe -m pytest -q` | `83 passed, 248 deselected`: gồm 9 ca Orchestrator bổ sung |

Trong test ranh giới handoff, investigator chạy dưới `PRAGMA query_only = ON`, không tạo refund; chỉ orchestrator gọi host sau khi nhận kết quả. Test đưa secret/PII vào ticket và thay văn bản model bằng lệnh giả để xác nhận chúng không thay số tiền và không đi vào handoff, prompt, trace hay stdout.

Lưu ý kết quả student của swarm: T02 seed 55 được test xác nhận là lỗi cố ý; ma trận xanh không có nghĩa benchmark của swarm đúng 100%. Chưa có kết quả hidden hay kết luận chọn kiến trúc cuối cùng.

Bằng chứng bước crash suite và benchmark, do Codex chạy:

| Lệnh chạy trong `src/` | Kết quả |
| --- | --- |
| `.\.venv\Scripts\python.exe -m lab8.crash_test --count 20 --out reports/crash.json` | `crash_safety 20/20` |
| `.\.venv\Scripts\python.exe -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json` | 280 trial; workflow/single/orchestrator mỗi shape đạt 70/70; swarm 69/70 |

Workflow có cost/success thấp nhất, xấp xỉ 27,86 VND mô phỏng so với single 33,37, orchestrator 50,71 và swarm 49,57. Đã chọn workflow dựa trên public trong ADR; commit hash còn trống đến khi chốt code. Các file JSON do lệnh lab sinh ra, không sửa tay. Chưa có `student_drop/`, nên không công bố kết quả hidden.

Kiểm tra định dạng sau bước này bằng `.\.venv\Scripts\python.exe -m lab8.check_submission`: shape và hai report đều OK; còn `2 FAIL` do commit hash chưa điền và research card chưa hoàn thành, cùng `1 WARN` do chưa commit thay đổi. Đây là các phần cần hoàn tất trước khi nộp, không phải kết quả `0 FAIL` cuối cùng.

Bằng chứng bước research card/ablation, do Codex chạy:

- Lệnh benchmark đầy đủ trong `research_card.md`; file do công cụ sinh là `src/reports/bonus_ablation.json` (140 trial), không sửa tay JSON.
- Cả baseline và biến thể đạt 70/70; cost/success 50,712857 → 49,607143 VND mô phỏng, giảm 2,18%; mỗi trial bớt đúng 1 model call. Baseline khớp nguyên trial và thống kê benchmark chính.
- Sáu ca cặp xác nhận status/quote/effect/audit/tool count không đổi, trace chỉ mất event draft; ba ca kiểm crash/resume. Tổng test mới cả bài: 38 ca. Kiểm tra sau thay đổi: `92 passed` mặc định và `248 passed` student.
- Đây là nghiên cứu thành phần bắt buộc, chưa phải bài bonus B4 đầy đủ. Chưa đo chất lượng câu trả lời khách hàng và chưa có kết quả hidden; giữ lựa chọn workflow.

Bằng chứng tự kiểm tra của thành viên:

- Lệnh ban đầu `.\.venv\Scripts\python.exe -m pytest -q` gặp `17 passed, 248 deselected, 75 errors in 12.68s`. Stack trace cho thấy lỗi `PermissionError: [WinError 5]` tại thư mục `C:\Users\ADMIN\AppData\Local\Temp\pytest-of-ADMIN`, khi pytest chuẩn bị fixture `tmp_path`.
- Sau hướng dẫn của Codex, thành viên tự chạy `.\.venv\Scripts\python.exe -m pytest -q --basetemp=.venv/pytest-local` trong `src/` và cung cấp kết quả `92 passed, 248 deselected in 2.24s`.
- Cách xử lý chỉ đổi nơi chứa dữ liệu tạm của pytest trong `.venv/`, không sửa bài test hoặc code nghiệp vụ để bỏ qua lỗi.

Kiểm tra định dạng sau research card bằng `.\.venv\Scripts\python.exe -m lab8.check_submission`: research card, ba shape và các report bắt buộc đều OK; tại thời điểm đó còn `1 FAIL` vì commit hash trống và `1 WARN` vì chưa commit thay đổi.

Chốt code/test/bằng chứng tại commit `4d04f12e3b7b425861c4968d61475ee5a36a744f`. ADR, TEAM và báo cáo nhóm đã được cập nhật cùng hash bằng commit tài liệu kế tiếp; không dùng hash starter `034c6c4` làm hash bài làm.

## 5. Công cụ AI đã dùng

- Codex: hỗ trợ đọc đề bài, chuẩn bị môi trường, triển khai workflow/single/orchestrator, đọc nguồn nghiên cứu, viết/chạy test và ablation, tổng hợp báo cáo và giải thích từng bước.
- Cách tự kiểm tra: thành viên đã chạy bộ test mặc định, gửi lỗi quyền thư mục tạm để phân tích, rồi tự chạy lại với `--basetemp` và xác nhận 92 passed. Benchmark, ablation và bộ student hiện do Codex chạy; chưa kê khai thành viên đã tự chạy các phần này.

Các lệnh kiểm tra trong PowerShell ở thư mục `src/` (hai lệnh pytest dùng thư mục tạm riêng để tránh lỗi quyền Windows):

```powershell
.\.venv\Scripts\python.exe -m lab8.smoke
.\.venv\Scripts\python.exe -m pytest -q --basetemp=.venv/pytest-local
.\.venv\Scripts\python.exe -m pytest -q -m student -o addopts= --basetemp=.venv/pytest-local
```

Kết quả mong đợi tại trạng thái code hiện tại: `lab8.smoke pass`, `92 passed` mặc định và `248 passed` student. Thành viên đã xác nhận 92 passed mặc định; chưa xác nhận tự chạy smoke và student. Chạy lần lượt các lệnh vì pytest dùng lại thư mục tạm riêng này.

## 6. Tự đánh giá

- Tôi đã tự chạy lại test mặc định, gặp lỗi quyền thư mục tạm Windows, rồi làm theo hướng dẫn đổi `--basetemp` và xác nhận 92 passed. Tôi chưa tự xác nhận mức độ hiểu code của từng shape.
- Phần triển khai, chạy test và tổng hợp số liệu hiện có sự hỗ trợ của Codex, được ghi rõ trong báo cáo.
- Việc cần làm trước khi nộp: tự chạy thêm smoke/student, đối chiếu output và đọc lại phần Workflow/Single, ranh giới code/model, quote đầy đủ, approval và idempotency; phối hợp Hưng rà soát handoff, số đo và cập nhật bằng chứng sau commit của Hưng.
