# 01 — Problem Scan & Quick Problem Cards

**Tên nhóm:** Bản cá nhân — chờ nhóm trưởng cập nhật  
**Thành viên:** Phạm Khắc Tú — canhquat213@gmail.com  
**Branch cá nhân:** phamkhactu  
**Lưu ý dữ liệu:** Các con số bên dưới là mục tiêu pilot hoặc lab benchmark, không phải số liệu vận hành nội bộ đã được Vinhomes/Vingroup xác nhận.

---

## 1. Nguyên tắc lựa chọn bài toán

Tôi ưu tiên một micro-workflow có actor rõ, công việc lặp lại, có thể tạo dữ liệu kiểm thử mà không sử dụng dữ liệu cá nhân thật, và có ranh giới giữa Rule-based, LLM Feature và Agentic Loop đủ rõ để kiểm chứng bằng code.

## 2. SCAN — Sáu cơ hội trong hệ sinh thái Vingroup

| # | Đơn vị | Lens | Bài toán vận hành |
|---:|---|---|---|
| 1 | Vinhomes | Lặp lại | Nhân viên mở nhiều form/email/tài liệu đăng ký thi công căn hộ, nhập lại dữ liệu và đối chiếu checklist thủ công. |
| 2 | Vinhomes | Pain từ stakeholder | Phản ánh cháy, rò gas, kẹt thang máy hoặc rò nước viết bằng ngôn ngữ tự do có thể bị chuyển sai đội xử lý. |
| 3 | Vinpearl | Tốn thời gian | Nhân viên đọc email booking đoàn, bóc tách số phòng/ngày/loại khách và soạn báo giá thủ công. |
| 4 | VinFast | AI-upgrade | Cố vấn dịch vụ phải chuẩn hóa mô tả tiếng Việt tự do của khách thành phiếu tiếp nhận lỗi xe. |
| 5 | VinUni | Lặp lại | Trợ giảng đọc log autograder và viết lại phản hồi dễ hiểu cho từng sinh viên. |
| 6 | Xanh SM | Pain từ stakeholder | Nhân viên phân tích thủ công ghi chú và nội dung cuộc gọi để tìm nguyên nhân hủy chuyến. |

## 3. Ma trận sàng lọc

Thang điểm 1–5; tổng trọng số 100%. Điểm là đánh giá phục vụ quyết định trong lab, không phải kết quả nghiên cứu chính thức.

| Bài toán | Business value 25% | Workflow clarity 20% | Data feasibility 20% | Boundary testability 20% | Prototype feasibility 15% | Điểm /100 |
|---|---:|---:|---:|---:|---:|---:|
| Vinhomes PermitGuard | 5 | 5 | 4 | 5 | 5 | 96 |
| Vinpearl QuoteShield | 5 | 4 | 5 | 5 | 5 | 96 |
| Vinhomes IncidentGuard | 5 | 5 | 4 | 5 | 4 | 93 |
| VinFast SafeWorkOrder | 4 | 4 | 3 | 5 | 4 | 79 |
| VinUni SafeGrade | 4 | 4 | 5 | 5 | 5 | 91 |
| Xanh SM Cancellation Analyzer | 4 | 3 | 3 | 3 | 4 | 68 |

PermitGuard và QuoteShield bằng điểm định lượng. PermitGuard được chọn vì có rework loop và nhiều handoff hơn, đồng thời rủi ro thấp hơn IncidentGuard nên dễ đưa ra quyết định GO cho shadow-mode prototype.

---

## 4. Quick Problem Card #1 — Vinhomes PermitGuard

| Trường | Nội dung |
|---|---|
| Bài toán | Kiểm tra vòng đầu tính đầy đủ và nhất quán của hồ sơ đăng ký thi công căn hộ. |
| Actor | Nhân viên CSKH tiếp nhận hồ sơ; nhân viên kỹ thuật/an ninh; người có thẩm quyền duyệt. |
| Current workflow | 1. Nhận form/email/tài liệu → 2. Mở từng file và nhập lại dữ liệu → 3. Tra checklist/policy → 4. Đối chiếu, phát hiện thiếu hoặc mâu thuẫn → 5. Soạn yêu cầu bổ sung và chuyển người duyệt. |
| Bottleneck | Bước 2–4: dữ liệu nằm rải rác, cách diễn đạt không thống nhất và dễ tạo vòng lặp bổ sung. |
| AI insertion point | LLM trích xuất dữ liệu và evidence span; rule engine kiểm tra checklist, policy version và tính hợp lệ; nhân viên review. |
| Metric pilot | Giảm median first-pass review ít nhất 70%; missing-field recall ít nhất 98%; false-ready bằng 0 trên lab set; JSON hợp lệ 100%. |
| Quick architecture | Hybrid Rule + LLM Feature; không sử dụng Agentic Loop. |
| Boundary | AI không được APPROVE/REJECT, miễn phí, gửi thông báo hoặc cập nhật hệ thống. Mọi kết quả phải cần human review. |

## 5. Quick Problem Card #2 — Vinpearl QuoteShield

| Trường | Nội dung |
|---|---|
| Bài toán | Bóc tách yêu cầu booking đoàn từ email và soạn bản nháp báo giá. |
| Actor | Nhân viên reservations và revenue manager. |
| Current workflow | 1. Nhận email → 2. Đọc và nhập ngày/số phòng/loại phòng → 3. Tra tồn phòng, rate card và chính sách → 4. Tính giá → 5. Soạn email phản hồi. |
| Bottleneck | Bước 2 và 5 tốn thời gian; thông tin thường thiếu hoặc nằm trong file đính kèm. |
| AI insertion point | LLM trích xuất yêu cầu và draft email; rule/API quyết định giá, thuế, tồn phòng và discount. |
| Metric pilot | Extraction F1 ít nhất 95%; sai giá bằng 0; thời gian chuẩn bị draft giảm ít nhất 70%; 100% báo giá có human approval. |
| Quick architecture | Hybrid Rule/API + LLM Feature. |
| Boundary | AI không được hold/book/cancel phòng, tự đổi giá, miễn phí hoặc gửi email. |

## 6. Quick Problem Card #3 — Vinhomes IncidentGuard

| Trường | Nội dung |
|---|---|
| Bài toán | Trích xuất dấu hiệu và route nháp các phản ánh khẩn cấp của cư dân. |
| Actor | Nhân viên CSKH và điều phối Ban Quản lý. |
| Current workflow | 1. Nhận ticket → 2. Đọc và hỏi lại vị trí → 3. Xác định mức độ → 4. Tra SOP và đội phụ trách → 5. Route và theo dõi SLA. |
| Bottleneck | Bước 2–4 khi nội dung tiếng Việt mơ hồ, viết tắt hoặc thiếu vị trí. |
| AI insertion point | LLM trích xuất dấu hiệu; deterministic rules ép mức khẩn cấp; nhân viên xác nhận trước khi route. |
| Metric pilot | P95 tạo draft dưới 60 giây; recall P0/P1 ít nhất 99%; wrong-route không quá 1%; tự đóng ticket bằng 0. |
| Quick architecture | Hybrid Rule + LLM Feature, shadow mode. |
| Boundary | AI không được hạ cấp sự cố, dispatch, đóng ticket hoặc tiết lộ dữ liệu/mã truy cập. |

---

## 7. Quyết định lựa chọn

**Chọn Vinhomes PermitGuard** để Deep Dive.

Lý do:

1. Scope hẹp và đo được: chỉ kiểm tra first-pass completeness/consistency, không giải quyết toàn bộ quy trình phê duyệt.
2. AI-Fit rõ: LLM xử lý ngôn ngữ/OCR; rules xử lý policy và quyền hạn; con người quyết định.
3. Có thể tạo hồ sơ tổng hợp và ground truth trong lab mà không dùng dữ liệu cư dân thật.
4. Có threat model Prompt Injection tự nhiên từ email, OCR và ghi chú nhà thầu.
5. Có fallback an toàn về quy trình manual review hiện tại.

QuoteShield không được chọn vì cần rate card và logic tồn phòng đáng tin cậy để demo end-to-end. IncidentGuard không được chọn vì việc chứng minh độ an toàn cho tình huống khẩn cấp cần dữ liệu và domain expert mạnh hơn phạm vi codelab.
