# Báo cáo cá nhân — Nguyễn Ngọc Minh Đức

## 1. Thông tin thành viên

- Họ tên: Nguyễn Ngọc Minh Đức
- MSSV: 2A202603421
- Email: Chưa cung cấp
- GitHub: [MinDuck512](https://github.com/MinDuck512)

## 2. Phần việc phụ trách

Workflow, Single, môi trường/Git, tích hợp và ADR. Hưng tiếp nhận Orchestrator/handoff, kiểm chứng an toàn, benchmark và nghiên cứu; hai người kiểm tra chéo.

## 3. Đóng góp cụ thể

Commit `4d04f12e3b7b425861c4968d61475ee5a36a744f` chứa bản code, test và bằng chứng chung do tôi commit với hỗ trợ Codex, trước phân công nhóm 2 người.

- Workflow dùng các bước cố định; Single dùng vòng ReAct tối đa 8 bước, tool chỉ đọc.
- Tiền do code tính; host kiểm approval, giới hạn tiền và chống hoàn trùng.
- Bản tích hợp có 10 test Workflow, 10 test Single và 18 test Orchestrator/ablation bổ sung trong `src/tests/`.
- Đã tổng hợp kết quả vào báo cáo nhóm, ADR và research card; Hưng sẽ tự bổ sung commit/test của mình.

## 4. Bằng chứng

| Phần kiểm tra | Kết quả |
| --- | --- |
| Test mặc định — tôi đã tự chạy | 92 passed, 248 deselected in 2.24s |
| Student — Codex chạy | 248 passed |
| Crash suite — Codex chạy | 20/20 |
| Benchmark — Codex chạy | Workflow/Single/Orchestrator 70/70 mỗi shape; Swarm 69/70 |
| Ablation — Codex chạy | Tắt draft giảm 2,18% chi phí, cả hai bản đúng 70/70 |

Bằng chứng đầy đủ: [benchmark](../../src/reports/benchmark.json), [crash](../../src/reports/crash.json), [ablation](../../src/reports/bonus_ablation.json).

Lần đầu tự chạy có 75 lỗi do quyền thư mục Temp. Đổi thư mục tạm bằng lệnh dưới, trong `src/`, đã đạt 92 passed:

```powershell
.\.venv\Scripts\python.exe -m pytest -q --basetemp=.venv/pytest-local
```

## 5. Công cụ AI đã dùng

Codex hỗ trợ đọc đề, viết/rà soát code và test, chạy benchmark/ablation, xử lý lỗi và biên soạn báo cáo. Tôi đã tự kiểm tra bộ test mặc định; smoke, student và số đo nghiên cứu hiện do Codex chạy.

## 6. Tự đánh giá

Đã tự chạy lại test và xử lý được lỗi quyền Temp theo hướng dẫn. Cần tự chạy thêm smoke/student, nắm rõ quote đầy đủ, approval, idempotency và ranh giới code/model để giải thích bài làm; phối hợp Hưng kiểm chứng phần còn lại. Chưa có kết quả hidden.
