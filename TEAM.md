# Team

## Thông tin nhóm

- Tên nhóm: NNMD
- Lớp: L3A
- Repo nhóm: `K5-L3A-DAY08-NNMD-FromWorkflowsToAgents` (xem [SUBMISSION.md](SUBMISSION.md))
- URL repo, nhánh nộp, commit chốt: https://github.com/MinDuck512/K5-L3A-DAY08-NNMD-FromWorkflowsToAgents · nhánh `main` · commit code/bằng chứng: `4d04f12e3b7b425861c4968d61475ee5a36a744f`

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
|   1 | Nguyễn Ngọc Minh Đức | 2A202603421 | Chưa cung cấp | [MinDuck512](https://github.com/MinDuck512) | Workflow, Single, tích hợp, môi trường/Git và ADR |
|   2 | Dương Tuấn Hưng | 03416 | Chưa cung cấp | [hunglun2711](https://github.com/hunglun2711) | Orchestrator/handoff, kiểm chứng an toàn, benchmark/ablation và báo cáo kết quả |

## Phân công nhóm 2 người

| Phần việc | Phụ trách chính | Phối hợp / sản phẩm |
| --- | --- | --- |
| Workflow và Single | Minh Đức | Rà soát `workflow.py`, `single.py`, test nghiệp vụ và vòng ReAct; Hưng kiểm tra chéo kết quả |
| Orchestrator và handoff | Tuấn Hưng | Tiếp nhận/rà soát `orchestrator.py`; kiểm quyền tool đọc, principal scope và dữ liệu qua handoff; Đức tích hợp |
| Approval, idempotency và crash | Tuấn Hưng | Rà soát/thêm test có ý nghĩa trong `src/tests/`, chạy lại crash suite; Đức đối chiếu với luồng ghi host |
| Benchmark và nghiên cứu R3 | Tuấn Hưng | Tự chạy benchmark/ablation cùng corpus, seed, giá; đối chiếu JSON và `research_card.md`; Đức kiểm tra tính tái lập |
| ADR và tích hợp bài nộp | Minh Đức | Tổng hợp lựa chọn kiến trúc, giữ hash code/bằng chứng nhất quán, xử lý môi trường/Git và chạy `check_submission` |
| Báo cáo nhóm | Cả hai | Đức phụ trách cách tiếp cận/giải pháp; Hưng rà soát kết quả, thí nghiệm và giới hạn; cùng thống nhất kết luận |
| Báo cáo cá nhân, commit và LMS | Mỗi người tự thực hiện | Tự khai đóng góp, thêm/sửa ít nhất một test trong commit kỹ thuật của mình và tự nộp link repo nhóm |

Đây là phân công phụ trách và kiểm chứng từ khi nhóm cập nhật đủ 2 người. Bản tích hợp đầu tiên `4d04f12` đã có trước phân công này, do Đức commit với sự hỗ trợ của Codex. Hưng chưa có commit hoặc kết quả tự chạy được xác nhận; cập nhật bằng chứng sau khi thực hiện.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Nguyễn Ngọc Minh Đức | Workflow, Single và tích hợp bản code chung với hỗ trợ Codex | `4d04f12` | Bản tích hợp chứa `src/tests/test_workflow_edges.py` (10 ca), `src/tests/test_single_edges.py` (10 ca) và 18 ca Orchestrator/ablation trong `src/tests/test_handoff_secret.py` |
| Dương Tuấn Hưng | Tiếp nhận Orchestrator/handoff, kiểm chứng an toàn và đánh giá | Chưa có; Hưng tự commit sau | Chưa có test riêng được xác nhận; Hưng tự bổ sung/sửa test có ý nghĩa và ghi tên test sau khi thực hiện |

## Cá nhân

- Đức: [report/team/2A202603421_NguyenNgocMinhDuc.md](report/team/2A202603421_NguyenNgocMinhDuc.md).
- Hưng: tự tạo `report/team/03416_DuongTuanHung.md` từ mẫu `report/team/MSSV_HoTen.md` và tự commit sau. Hiện chưa tạo file này theo thống nhất của nhóm.
- Mỗi người tự điền công cụ AI đã dùng, cách kiểm tra và tự đánh giá theo `RULES.md`. Giữ nguyên file mẫu cho thành viên sử dụng.
