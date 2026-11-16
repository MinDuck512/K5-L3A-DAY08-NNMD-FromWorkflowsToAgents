# Báo cáo nhóm — Lab 8 LogiX-VN

## 1. Thông tin nhóm

- Nhóm NNMD — L3A: Nguyễn Ngọc Minh Đức (`2A202603421`) và Dương Tuấn Hưng (`2A202603416`).
- Commit code/bằng chứng: `4d04f12e3b7b425861c4968d61475ee5a36a744f`, trùng ADR.

## 2. Mục tiêu và cách tiếp cận

Xử lý yêu cầu hoàn tiền bằng 4 kiến trúc và chọn cách ít tự chủ nhất vẫn đúng, an toàn, tiết kiệm. Nhóm viết Workflow, Single, Orchestrator và dùng Swarm có sẵn làm đối chứng.

Đo trên cùng 14 claim, 5 seed `11,22,33,44,55`, MockLLM, tool, bảng giá và evaluator. Mỗi lượt dùng SQLite mới; tổng 280 lượt. Ngày xét policy cố định là `2026-11-15`.

## 3. Giải pháp

| Kiến trúc | Cách xử lý |
| --- | --- |
| Workflow | Code chạy các bước cố định: đọc claim, tra đơn, tính tiền, gọi host. Không gọi model. |
| Single | Vòng ReAct tối đa 8 bước; chỉ dùng tool đọc. Code gọi host sau vòng lặp. |
| Orchestrator | Giao tra đơn/policy cho Investigator qua handoff đã lọc; Orchestrator tính tiền và gọi host. |

Tiền do `compute_compensation` tính từ đơn hàng và policy. Host kiểm approval, trần theo đơn và chống hoàn trùng. Shape gửi quote đầy đủ: T14 được hưởng 150.000đ, đã nhận 30.000đ, host chỉ hoàn thêm 120.000đ.

Investigator chỉ có tool đọc. Handoff/trace không mang ticket, secret, header hay thông tin cá nhân. Mỗi nhánh trả về có trace mới; crash được truyền ra ngoài để kiểm tra chạy lại an toàn.

## 4. Kết quả

Nguồn: [benchmark.json](../../src/reports/benchmark.json). Chi phí là VND mô phỏng; steps là số event trong trace.

| Shape | Đúng | Pass rate | Pass^5 | Seed min–max | Variance | Cost/success | Mean/p95 steps |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Workflow | 70/70 | 1,000000 | 1,000000 | 1–1 | 0 | 27,86 | 5,07 / 6 |
| Single | 70/70 | 1,000000 | 1,000000 | 1–1 | 0 | 33,37 | 8,86 / 10 |
| Orchestrator | 70/70 | 1,000000 | 1,000000 | 1–1 | 0 | 50,71 | 8,86 / 10 |
| Swarm | 69/70 | 0,985714 | 0,928571 | 0,928571–1 | 0,000816327 | 49,57 | 2,80 / 3 |

Swarm sai C-002 ở seed 55 vì tin ticket nói trễ 5 ngày thay vì dữ liệu đơn ghi 2 ngày. Đây là lỗi cố ý được bộ student kiểm tra là lỗi dự kiến.

- Test cuối: **92 passed** mặc định, **248 passed** student.
- [Crash suite](../../src/reports/crash.json): **20/20**, kiểm host và Swarm. T12 và test bổ sung kiểm crash của 3 shape nhóm viết.
- [Ablation R3](research_card.md): tắt riêng draft của Orchestrator vẫn đúng 70/70, cost/success giảm 50,71 → 49,61đ (**2,18%**).
- Chọn **Workflow**: đúng toàn bộ public và chi phí thấp nhất; xem [ADR](decision_adr.md).

Lệnh tái lập, chạy trong `src/` sau khi kích hoạt môi trường:

```text
python -m lab8.crash_test --count 20 --out reports/crash.json
python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json
```

## 5. Khó khăn và cách xử lý

- Máy có Python 3.14; nhóm tạo môi trường Python 3.13.17 theo yêu cầu lab.
- Đức tự chạy pytest gặp `17 passed, 75 errors` do lỗi quyền thư mục Temp. Thêm `--basetemp=.venv/pytest-local` vào lệnh pytest trong `src/` và xác nhận **92 passed, 248 deselected in 2.24s**.

## 6. Phân công và bằng chứng

| Thành viên | Phụ trách | Trạng thái |
| --- | --- | --- |
| Minh Đức | Workflow, Single, tích hợp/môi trường/Git và ADR | Commit bản chung `4d04f12` với hỗ trợ Codex; đã tự xác nhận 92 test mặc định pass |
| Tuấn Hưng | Orchestrator/handoff, kiểm chứng an toàn, benchmark/ablation và kết quả | Commit `2748ba6` bổ sung 10 test trong `test_orchestrator_and_safety_hung.py`; tự xác nhận 102 test mặc định, 248 test student, crash 20/20 |

Chi tiết đóng góp nằm trong [TEAM.md](../../TEAM.md), báo cáo của Đức và [Hưng](2A202603416_DuongTuanHung.md). Cả hai thành viên đều có commit kỹ thuật độc lập và tự nộp link repo trên LMS.

## 7. Hạn chế và hướng cải thiện

MockLLM chỉ mô phỏng; Single dùng adapter chọn action bằng code, Orchestrator dùng kế hoạch cố định. Giá là mô phỏng, latency là `unavailable`; số event trace không phản ánh tốc độ. Chênh lệch giữa các shape còn phụ thuộc prompt và số lượt gọi.

Public chỉ có 14 claim; chưa có `student_drop/` hay điểm hidden. Ablation chưa đo chất lượng câu trả lời khách. Nhóm sẽ đo lại khi có task cần đọc dữ liệu phi cấu trúc hoặc chia việc độc lập; vẫn phải giữ các gate an toàn.

## 8. Tài liệu tham khảo

- [Luật nghiệp vụ](../../docs/business_rules.md), [contract](../../docs/contracts.md), [metric](../../docs/metrics.md).
- [Quy định](../../RULES.md), [bài nộp](../../SUBMISSION.md), [lệnh tạo bằng chứng](../../src/reports/README.md).
- [AgentArch (2026)](https://aclanthology.org/2026.aacl-long.44/), nguồn cho R3; xem phạm vi trong research card.
