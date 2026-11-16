# Team

## Thông tin nhóm

- Tên nhóm: NNMD — L3A
- Repo: [K5-L3A-DAY08-NNMD-FromWorkflowsToAgents](https://github.com/MinDuck512/K5-L3A-DAY08-NNMD-FromWorkflowsToAgents), nhánh `main`.
- Commit code và bằng chứng: `4d04f12e3b7b425861c4968d61475ee5a36a744f`.

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
| 1 | Nguyễn Ngọc Minh Đức | 2A202603421 | Chưa cung cấp | [MinDuck512](https://github.com/MinDuck512) | Workflow, Single, tích hợp và ADR |
| 2 | Dương Tuấn Hưng | 2A202603416 | duongtuanhung2711@gmail.com | [hunglun2711](https://github.com/hunglun2711) | Orchestrator/handoff, kiểm chứng an toàn, benchmark và nghiên cứu |

Hai người kiểm tra chéo và cùng rà soát báo cáo nhóm. Mỗi người tự tạo báo cáo cá nhân, commit kỹ thuật có ít nhất một test thêm/sửa và nộp link repo trên LMS.

## Bảng đóng góp

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Nguyễn Ngọc Minh Đức | Tích hợp bản code chung với hỗ trợ Codex; phụ trách Workflow/Single | `4d04f12` | `test_workflow_edges.py`: 10 ca; `test_single_edges.py`: 10 ca; `test_handoff_secret.py`: thêm 18 ca |
| Dương Tuấn Hưng | Tiếp nhận Orchestrator/handoff và kiểm chứng an toàn | `2748ba6` | `test_orchestrator_and_safety_hung.py`: 10 ca kiểm thử Orchestrator, handoff safety, T02 seed 55, order cap, crash/resume và ablation R3 |

Các test nằm trong `src/tests/`. Cả hai thành viên đã hoàn thành phần việc và tự xác nhận kết quả kiểm thử.

## Báo cáo cá nhân

- Đức: [2A202603421_NguyenNgocMinhDuc.md](report/team/2A202603421_NguyenNgocMinhDuc.md).
- Hưng: [2A202603416_DuongTuanHung.md](report/team/2A202603416_DuongTuanHung.md).
