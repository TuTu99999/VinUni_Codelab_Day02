# 01 — Problem Scan (Vin Smart Future)

> Deliverable cá nhân/nhóm cho Gate I1 (Scan & Cards) — tổng hợp Phase 1 (SCAN) và Phase 2 (QUICK-ASSESS) từ `01-worksheet.md`.

---

## 🔍 Phase 1 — SCAN: 5 bài toán vận hành thực tế

| # | Subsidiary | Lens | Mô tả ngắn bài toán |
|---|---|---|---|
| 1 | **VinFast** | Lặp lại (Repetitive) | Đội vận hành cứu hộ phải thủ công đọc yêu cầu cứu hộ pin (SOS pin yếu/lỗi sạc), tra cứu vị trí trạm sạc gần nhất, rồi mới quyết định điều xe sạc lưu động hay hướng dẫn khách tự lái đến trạm — lặp lại hàng trăm lượt/ngày trên toàn quốc. |
| 2 | **Xanh SM (GSM)** | Pain từ người khác (Stakeholder Pain) | Tài xế phàn nàn hệ thống gợi ý điểm đón khách không chính xác trong giờ cao điểm/khu vực đông đúc, khiến tài xế phải gọi tổng đài xác nhận lại, kéo dài thời gian chờ của khách. |
| 3 | **Vinhomes** | Tốn thời gian (Time-consuming) | Nhân viên CSKH cư dân phải tự soạn thủ công từng phản hồi cho các đánh giá 1–2 sao trên app quản lý cư dân, mỗi phản hồi mất ~10 phút để vừa đúng giọng thương hiệu vừa đúng chính sách. |
| 4 | **Vinmec** | AI có thể tốt hơn (AI-upgrade) | Bác sĩ phải tự đọc và tổng hợp lại toàn bộ hồ sơ bệnh án cũ (tiền sử, đơn thuốc, kết quả xét nghiệm) trước mỗi lượt tái khám, tốn thời gian khám và dễ bỏ sót thông tin quan trọng. |
| 5 | **Vinpearl / VinWonders** | AI có thể tốt hơn (AI-upgrade) | Chatbot CSKH hiện tại trả lời rập khuôn theo kịch bản cố định, không xử lý được các câu hỏi ngoài kịch bản về đổi vé/hoàn tiền, khiến khách phải chuyển sang tổng đài viên. |

---

## 🃏 Phase 2 — QUICK-ASSESS: 3 Quick Problem Cards

### QUICK PROBLEM CARD #1 (⭐ Bài toán được chọn cho Deep-Dive)

```
Bài toán (1 câu): Đội điều phối cứu hộ VinFast xử lý thủ công yêu cầu SOS pin yếu/lỗi sạc,
dẫn đến quyết định điều xe sạc lưu động chậm và không nhất quán.

Công ty thành viên: [x] VinFast  [ ] Xanh SM  [ ] Vinhomes  [ ] Vinmec  [ ] Khác

Ai đang đau (Actor)? Nhân viên điều phối cứu hộ (Rescue Dispatcher) tại trung tâm điều hành VinFast,
và khách hàng đang mắc kẹt trên đường.

Workflow thủ công hiện tại (3-5 bước):
  1. Khách gọi hotline SOS báo pin yếu/lỗi sạc
  ──> 2. Điều phối viên hỏi lại thông tin (model xe, % pin, mã lỗi, vị trí)
  ──> 3. Điều phối viên tra cứu trạm sạc gần nhất / xe sạc lưu động khả dụng
  ──> 4. Điều phối viên quyết định: hướng dẫn tự lái đến trạm hay điều xe sạc lưu động

Bước nào tốn thời gian/lỗi nhất? Bước 3-4 (tra cứu + quyết định) ⏱ ~8 phút/lượt,
và dễ sai khi pin <5% nhưng vẫn hướng dẫn khách tự lái xa >5km.

AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2-4: LLM hỏi bổ sung thông tin còn thiếu,
phân loại mức độ khẩn cấp theo % pin, và soạn sẵn bản nháp hành động cho điều phối viên duyệt.

Đo thành công bằng gì (Metric có số)?
  Giảm thời gian ra quyết định điều phối từ 8 phút ──> dưới 2 phút/lượt;
  100% trường hợp pin <5% được gắn cờ ưu tiên điều xe sạc lưu động, không đề xuất trạm >5km.

Quick Architecture: [ ] No AI  [ ] Rule  [x] LLM  [ ] Agent
```

### QUICK PROBLEM CARD #2

```
Bài toán (1 câu): Hệ thống Smart Dispatching của Xanh SM gợi ý điểm đón chưa chính xác
trong giờ cao điểm, gây tranh cãi giữa tài xế và khách.

Công ty thành viên: [ ] VinFast  [x] Xanh SM  [ ] Vinhomes  [ ] Vinmec  [ ] Khác

Ai đang đau (Actor)? Tài xế Xanh SM và tổng đài viên hỗ trợ.

Workflow thủ công hiện tại (3-5 bước):
  1. Hệ thống gợi ý điểm đón tự động theo GPS
  ──> 2. Tài xế thấy điểm đón không hợp lý (ví dụ trong hẻm cấm xe)
  ──> 3. Tài xế gọi tổng đài xác nhận lại vị trí thực tế
  ──> 4. Tổng đài viên gọi khách xác nhận điểm đón thay thế

Bước nào tốn thời gian/lỗi nhất? Bước 3-4 ⏱ ~4 phút/lượt, xảy ra ~15% tổng số chuyến giờ cao điểm.

AI có thể nhảy vào hỗ trợ ở bước nào? Bước 1: mô hình gợi ý điểm đón học từ lịch sử phản hồi
"điểm đón sai" để tinh chỉnh gợi ý theo thời gian thực.

Đo thành công bằng gì (Metric có số)?
  Giảm tỷ lệ điểm đón phải chỉnh tay từ 15% ──> dưới 5% số chuyến giờ cao điểm.

Quick Architecture: [ ] No AI  [x] Rule  [ ] LLM  [ ] Agent
```

### QUICK PROBLEM CARD #3

```
Bài toán (1 câu): Nhân viên CSKH Vinhomes tốn ~10 phút/lượt soạn phản hồi thủ công
cho từng đánh giá 1-2 sao của cư dân.

Công ty thành viên: [x] Vinhomes  [ ] VinFast  [ ] Xanh SM  [ ] Vinmec  [ ] Khác

Ai đang đau (Actor)? Nhân viên CSKH quản lý cư dân (Community Service Officer).

Workflow thủ công hiện tại (3-5 bước):
  1. Đánh giá 1-2 sao xuất hiện trên app cư dân
  ──> 2. CSKH đọc, xác minh vấn đề (tra cứu hồ sơ căn hộ/ticket liên quan)
  ──> 3. CSKH soạn phản hồi đúng giọng thương hiệu + đúng chính sách
  ──> 4. Trưởng nhóm duyệt trước khi đăng công khai

Bước nào tốn thời gian/lỗi nhất? Bước 3 ⏱ ~10 phút/lượt, do phải cân bằng giữa xin lỗi,
giải thích chính sách, và tránh cam kết ngoài thẩm quyền.

AI có thể nhảy vào hỗ trợ ở bước nào? Bước 3: LLM soạn bản nháp phản hồi dựa trên
nội dung đánh giá + chính sách nội bộ, CSKH chỉ cần chỉnh sửa và gửi duyệt.

Đo thành công bằng gì (Metric có số)?
  Giảm thời gian soạn phản hồi từ 10 phút ──> dưới 2 phút/lượt;
  ≥90% bản nháp không cần chỉnh sửa nội dung chính sách.

Quick Architecture: [ ] No AI  [ ] Rule  [x] LLM  [ ] Agent
```

---

**Ghi chú lựa chọn:** Nhóm chọn **Card #1 — VinFast EV Charging Rescue Dispatch** để đi tiếp Phase 3 (Deep-Dive) và Phase 4 (Prototype), vì đây là bài toán có ranh giới an toàn rõ ràng nhất (rủi ro pin cạn giữa đường), phù hợp để stress-test Operational Boundary trong prompt prototype (`prompt_prototype.py`).
