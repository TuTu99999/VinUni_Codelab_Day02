Lab 02 — Bài làm cá nhân

Họ và tên: Nguyễn Bảo Sơn
Vai trò: AI Product Engineer — Vin Smart Future
Phạm vi khảo sát: VinFast, Xanh SM, Vinhomes và Vinpearl/VinWonders
Lưu ý dữ liệu: Các con số trong phần SCAN và Quick Cards là ước tính dùng để scoping ban đầu, cần được kiểm chứng bằng log vận hành trước khi triển khai.

Phase 1 — SCAN: Tìm kiếm cơ hội

Tôi sử dụng bốn lens trong worksheet: Lặp lại, Tốn thời gian, AI có thể tốt hơn và Pain từ người khác. Tôi ưu tiên các bài toán có quy trình lặp lại, xảy ra thường xuyên trong vận hành, có dữ liệu từ ứng dụng/tổng đài và có thể giữ human-in-the-loop trong các bước quyết định quan trọng.

|  # | Công ty thành viên | Lens               | Bài toán/bottleneck quan sát được                                                                                                                        | Ước tính ban đầu                                                  |
| -: | ------------------ | ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
|  1 | **Xanh SM**        | Pain từ người khác | Tài xế đang thực hiện chuyến xe bị công an dừng kiểm tra nhưng không biết cần xử lý theo quy trình nào, cần cung cấp giấy tờ gì và báo cáo sự cố ra sao. | 5–10 phút/tình huống; tăng khi tài xế phải gọi tổng đài nhiều lần |
|  2 | **Xanh SM**        | Tốn thời gian      | Tổng đài phải thu thập thủ công thông tin từ tài xế khi xảy ra sự cố: vị trí, biển số, trạng thái chuyến, loại tình huống.                               | 5–15 phút/cuộc gọi hỗ trợ                                         |
|  3 | **Xanh SM**        | Lặp lại            | Bộ phận vận hành phải tiếp nhận nhiều incident liên quan đến lỗi giao thông, mất giấy tờ, gián đoạn chuyến và tạo báo cáo thủ công.                      | Hàng chục ticket/ngày                                             |
|  4 | **Xanh SM**        | AI có thể tốt hơn  | Hệ thống hỗ trợ tài xế hiện tại chủ yếu dựa vào kịch bản chung, chưa hiểu được bối cảnh thực tế của từng tình huống.                                     | Có thể giảm thời gian hỏi đáp nếu tự động lấy context từ app      |
|  5 | **VinFast**        | Lặp lại            | Nhân viên hậu mãi phải đọc và phân loại phiếu báo lỗi xe điện theo nhóm lỗi, dòng xe và mức độ ưu tiên.                                                  | 150–250 phiếu/ngày; 3–5 phút/phiếu                                |
|  6 | **Vinhomes**       | AI có thể tốt hơn  | Phản ánh cư dân về tiện ích, kỹ thuật, vệ sinh cần được phân loại và chuyển bộ phận thủ công.                                                            | 8–15 phút/ticket                                                  |


Tôi chọn ba bài toán:

Xanh SM AI Driver Incident Assistant
Xanh SM Incident Classification
Vinhomes Resident Complaint Routing
Quick Problem Card #1 — Xanh SM AI Driver Incident Assistant
Bài toán một câu

Khi tài xế Xanh SM đang chạy xe bị công an dừng kiểm tra, tài xế mất thời gian tìm hiểu cách xử lý, liên hệ hỗ trợ và cung cấp thông tin cho bộ phận vận hành.

Công ty: Xanh SM (GSM)

Actor đang gặp khó khăn:

Tài xế cần hỗ trợ nhanh.
Nhân viên vận hành cần đủ thông tin để xử lý.
Workflow thủ công hiện tại
Tài xế bị dừng xe.
Tài xế tự xử lý hoặc gọi tổng đài.
Nhân viên hỏi:
ID tài xế
biển số
vị trí
trạng thái chuyến
nguyên nhân sự cố
Nhân viên tra cứu quy trình.
Hướng dẫn tài xế và tạo báo cáo.
Bottleneck

Bước 3–5 mất khoảng 5–15 phút/tình huống.

Vấn đề:

Tài xế dễ thiếu thông tin.
Nhân viên phải hỏi lại nhiều lần.
Không có incident report chuẩn ngay từ đầu.
AI hỗ trợ ở đâu

AI Driver Incident Assistant:

Thu thập context từ app:
GPS
biển số
trạng thái chuyến
thông tin tài xế
Hiểu mô tả bằng LLM.

Ví dụ:

Input:

"Tôi đang chạy chuyến thì bị dừng xe kiểm tra giấy tờ."

Output:

{
 "incident_type":"traffic_stop",
 "trip_status":"active",
 "need":"driver_support"
}
Sinh bản nháp hướng dẫn và báo cáo cho nhân viên review.
Metric thành công
Metric	Target
| Metric                        | Target                            |
| ----------------------------- | --------------------------------- |
| Thời gian tạo incident report | Giảm từ 10 phút xuống dưới 2 phút |
| Ticket đủ thông tin lần đầu   | ≥95%                              |
| AI tự đưa kết luận pháp lý    | 0%                                |


Quick Architecture

☐ No AI
☐ Rule
☑ LLM Feature + Rule safety gate
☐ Agent

Lý do: LLM chỉ hiểu ngôn ngữ và tạo draft. Các quyết định pháp lý/vận hành cần rule và con người kiểm soát.

Phase 3 — Deep Dive
Problem Statement
User

Tài xế Xanh SM đang vận hành trên đường.

Need

Cần nhận hỗ trợ nhanh khi gặp tình huống bất thường.

Current Problem

Tài xế phải gọi tổng đài và tự mô tả toàn bộ tình huống.

Impact
Tăng thời gian xe không hoạt động.
Tăng tải tổng đài.
Giảm trải nghiệm tài xế.
AI Opportunity

AI hỗ trợ:

NLP hiểu tình huống.
Trích xuất thông tin.
Tra cứu quy trình.
Tạo draft response.
Success Criteria

Pilot đạt:

Giảm 50% thời gian xử lý.
95% ticket đủ thông tin.
100% tình huống nhạy cảm có human review.
Evaluate
| Tiêu chí           | Kết quả          |
| ------------------ | ---------------- |
| Pain rõ ràng       | PASS             |
| Có dữ liệu đầu vào | PASS             |
| Đo được KPI        | PASS             |
| Rủi ro pháp lý     | Cần human review |
| Có thể prototype   | PASS             |

Quyết định
GO — Prototype có scope hẹp

MVP:

 Hỗ trợ khi bị dừng kiểm tra
 Thu thập thông tin sự cố
 Tạo báo cáo vận hành

Không bao gồm:

 Tư vấn pháp luật
 Tranh luận trách nhiệm
 Quyết định xử lý vi phạm

Kết luận cá nhân

Qua quá trình SCAN, tôi nhận ra AI trong vận hành doanh nghiệp không nên được triển khai chỉ vì một quy trình có nhiều thao tác thủ công.

Với bài toán Xanh SM, giá trị lớn nhất của AI không nằm ở việc thay thế nhân viên vận hành mà là giảm thời gian thu thập thông tin, chuẩn hóa báo cáo và hỗ trợ tài xế trong thời điểm áp lực.

LLM nên đóng vai trò trợ lý hiểu ngôn ngữ và tạo bản nháp, trong khi rule engine và human-in-the-loop đảm nhiệm các giới hạn an toàn.