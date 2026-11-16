# Decision record

- Shape chọn: workflow
- Commit hash: 4d04f12e3b7b425861c4968d61475ee5a36a744f
- Run id dẫn chứng: R-workflow-C-001-s11, R-workflow-C-014-s55; đủ 70 trial trong src/reports/benchmark.json
- Pass rate / pass^5 / seed min-max / variance: 1.0 / 1.0 / 1.0–1.0 / 0.0 trên public
- Cost per success (mô phỏng, VND): 27.857142857142858 (xấp xỉ 27,86đ)
- Crash safety: 20/20 trong src/reports/crash.json; T12 và test bổ sung kiểm crash của Workflow
- Secret leak / approval bypass: Không ghi nhận trong các trial public và test đã chạy; chưa có số đếm chính thức từ hidden

Hash trên là bản code/test/bằng chứng; các commit tài liệu ghi lại hash này.

## Vì sao chọn Workflow

Trên cùng 14 claim × 5 seed, Workflow, Single và Orchestrator đều đúng 70/70. Workflow rẻ nhất: 27,86đ/thành công, so với Single 33,37đ và Orchestrator 50,71đ. Swarm đúng 69/70, chi phí 49,57đ. Luật hiện tại tính được bằng code nên chưa cần thêm mức tự chủ.

## Khi nào đổi lựa chọn

Đo lại nếu task cần model thật xử lý dữ liệu phi cấu trúc hoặc có nhiều việc độc lập để phân công. Chỉ đổi khi lợi ích đủ bù chi phí và vẫn an toàn. Ablation tắt draft giảm cost Orchestrator còn 49,61đ, vẫn cao hơn Workflow.

## Trace và bằng chứng

Trace ghi tra cứu, quote, yêu cầu/kết quả refund và quyết định. Không ghi ticket, secret, header, SĐT, địa chỉ, ghi chú nội bộ hoặc tên hàng. `finish()` ghi trace mới tại `src/traces/<run_id>.json` và lấy effect/audit từ ledger.

## Phân công và giới hạn

Đức tổng hợp ADR; Hưng phụ trách kiểm chứng số liệu, chưa có xác nhận riêng. Kết quả hiện có do Codex chạy trên bản tích hợp của Đức.

MockLLM không phải model thật; chi phí mô phỏng, latency `unavailable`. Steps phụ thuộc cách ghi log. Chưa có hidden; kết luận chỉ áp dụng cho public hiện tại. Ablation không chấm chất lượng câu trả lời khách. Chi tiết: [research card](research_card.md).
