# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn: workflow
- Commit hash:
- Run id dẫn chứng: R-workflow-C-001-s11, R-workflow-C-014-s55; toàn bộ 70 trial workflow trong src/reports/benchmark.json (14 claim, seed 11,22,33,44,55)
- Pass rate / pass^5 / seed min-max / variance: 1.0 / 1.0 / 1.0–1.0 / 0.0 trên bộ public
- Cost per success (mô phỏng, VND): 27.857142857142858 (xấp xỉ 27,86 VND)
- Crash safety: 20/20 (src/reports/crash.json); crash suite kiểm host và swarm, T12 và test bổ sung kiểm crash/resume của workflow
- Secret leak / approval bypass: Không ghi nhận vi phạm trong 70/70 trial public của workflow và các test ranh giới đã chạy; chưa có số đếm chính thức từ grader hidden

Commit hash sẽ được điền sau khi commit code và bằng chứng; hiện chưa chốt bài nộp.

## Vì sao shape này

Trên cùng 14 claim public × 5 seed, workflow, single và orchestrator đều đạt 70/70 trial, pass^5 = 1 và variance = 0. Workflow có cost/success thấp nhất: 27,86 VND so với single 33,37 và orchestrator 50,71. Swarm đạt 69/70, pass^5 = 13/14, cost/success 49,57 VND. Bài toán hiện có dữ kiện có cấu trúc và luật tính tiền xác định; workflow giải quyết được với 0 model call. Vì vậy chọn kiến trúc ít tự chủ nhất vẫn hoàn thành task. Đây là lựa chọn dựa trên public, chưa xác nhận gate hidden ≥ 0.75.

## Có đáng đổi lấy multi-agent không

Trong triển khai hiện tại, single và orchestrator chưa tăng độ đúng trên public, trong khi chi phí mô phỏng cao hơn. Chưa có bằng chứng để đổi sang multi-agent. Quyết định có thể đổi nếu có dữ liệu phi cấu trúc cần model thật, các công việc độc lập có lợi khi giao song song, hoặc bộ case mới cho thấy độ đúng tăng đủ để bù chi phí và độ phức tạp. Khi đó phải đo lại trên cùng điều kiện và vẫn giữ các gate an toàn.

Ablation R3 trong `src/reports/bonus_ablation.json` giữ nguyên độ đúng của Orchestrator khi tắt draft và giảm cost/success từ 50,712857 xuống 49,607143 VND mô phỏng (2,18%). Bản đã giảm vẫn cao hơn workflow 27,857143 VND, nên không đổi lựa chọn. Thí nghiệm chỉ đo ảnh hưởng một lượt draft, không chứng minh mọi overhead đều đến từ topology.

## Trace và bằng chứng

Workflow ghi `claim_lookup`, `order_lookup` đã lọc, `quote`, `refund_requested`, `refund_result` và `decision`. Không ghi ticket, principal context thô, `parent_secret`, header, SĐT, địa chỉ, internal note hoặc tên hàng. Trace nằm tại `src/traces/<run_id>.json`; không có debug dump riêng ngoài thư mục scanner. Mọi nhánh trả về dùng `finish()` để ghi trace mới và lấy effect/audit từ ledger. Bộ student và test bổ sung kiểm format trace, injection và crash; không suy rộng kết quả public thành bảo đảm cho mọi dữ liệu.

## Giới hạn

MockLLM không phải model sản xuất; single dùng adapter chọn action bằng code, orchestrator dùng kế hoạch tool đọc cố định. Giá là mô phỏng, latency là `unavailable`. `steps` đếm event trace, phụ thuộc mức chi tiết ghi log; swarm ghi ít event hơn nên không thể kết luận nó nhanh hơn. Các shape có prompt và số lượt gọi khác nhau, do đó chênh lệch cost chưa chứng minh tác động riêng của topology. Public chỉ có 14 claim; `student_drop/` chưa được cung cấp. Số liệu lấy nguyên từ công cụ, không sửa JSON. Research card R3 và ablation đã hoàn thành; phần draft bị tắt không được chấm chất lượng câu chữ.
