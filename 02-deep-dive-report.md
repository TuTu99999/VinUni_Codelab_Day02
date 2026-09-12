# 02 — Deep-Dive Report: VinFast EV Charging Rescue Assistant

> Deliverable nhóm cho Gate G1–G4 — tổng hợp Phase 3 (DEEP-DIVE) và Phase 5 (EVALUATE) từ `01-worksheet.md`.
> Bài toán được chọn từ `01-problem-scan.md` — Card #1: **VinFast EV Charging Rescue Dispatch**.

---

## 3.1. Current-State Workflow Mapping

**Sơ đồ trực quan:** xem file `04-workflow-diagram.svg`.

| Bước | Mô tả                                                                                            | Người/Hệ thống thực hiện | Thời gian | Ký hiệu                                                                                      |
| ---- | ------------------------------------------------------------------------------------------------ | ------------------------ | --------- | -------------------------------------------------------------------------------------------- |
| 1    | Khách gọi hotline SOS báo pin yếu / lỗi sạc                                                      | Khách hàng → Tổng đài    | ~1 phút   | 🔄 Handoff (Khách → Điều phối viên)                                                          |
| 2    | Điều phối viên hỏi lại thông tin còn thiếu: model xe, % pin, mã lỗi, vị trí GPS                  | Điều phối viên           | ~2 phút   | 🔄 Handoff (thoại → hệ thống nội bộ)                                                         |
| 3    | Điều phối viên tra cứu thủ công trạm sạc gần nhất và xe sạc lưu động khả dụng trên bản đồ nội bộ | Điều phối viên           | ~4 phút   | 🔴 **Bottleneck** — tra cứu chéo nhiều hệ thống (bản đồ trạm sạc, lịch trực đội xe lưu động) |
| 4    | Điều phối viên ra quyết định: hướng dẫn khách tự lái đến trạm hay điều xe sạc lưu động đến       | Điều phối viên           | ~2 phút   | 🔴 **Bottleneck** — quyết định cảm tính, dễ sai khi pin <5%                                  |
| 5    | Điều phối viên soạn tin nhắn/thoại thông báo hành động cho khách                                 | Điều phối viên → Khách   | ~1 phút   | 🔄 Handoff (hệ thống → khách)                                                                |

**Tổng cộng = 10 phút/lượt** (trung bình, chưa tính thời gian chờ nếu điều phối viên đang xử lý ca khác).

---

## 3.2. Problem Statement (6-field)

| Field                       | Nội dung chi tiết                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Actor / Operator**     | Điều phối viên cứu hộ (Rescue Dispatcher) tại Trung tâm điều hành VinFast, tiếp nhận trung bình ~120 cuộc gọi SOS/ngày trên toàn quốc.                                                                                                                                                                                                                                                                                                                                 |
| **2. Current Workflow**     | Nhận cuộc gọi → hỏi lại thông tin xe/pin/vị trí bằng thoại → tra cứu thủ công bản đồ trạm sạc + lịch xe sạc lưu động → tự quyết định hướng xử lý → gửi thông báo cho khách. Công cụ hiện dùng: hệ thống CRM nội bộ + bản đồ GIS tách rời, không liên thông tự động.                                                                                                                                                                                                    |
| **3. Bottleneck**           | Bước tra cứu chéo (bước 3) và ra quyết định (bước 4): điều phối viên phải tự nhớ quy tắc an toàn (ví dụ pin <5% không được hướng dẫn đi xa >5km), dễ sai sót do áp lực thời gian và khối lượng cuộc gọi cao vào giờ cao điểm.                                                                                                                                                                                                                                          |
| **4. Business Impact**      | Ước tính mỗi lượt xử lý sai (hướng dẫn khách tự lái khi pin quá thấp) có nguy cơ khách hết pin giữa đường → phát sinh chi phí cứu hộ khẩn cấp cao hơn, ảnh hưởng trải nghiệm khách hàng và uy tín thương hiệu. Với ~120 cuộc gọi/ngày và thời gian xử lý 10 phút/lượt, tổng thời gian nhân sự tiêu tốn ~20 giờ/ngày trên toàn hệ thống.                                                                                                                                |
| **5. Success Metric**       | Giảm thời gian ra quyết định điều phối từ 10 phút ──> dưới 3 phút/lượt; 100% trường hợp pin <5% được tự động gắn cờ ưu tiên "điều xe sạc lưu động" thay vì đề xuất trạm sạc xa; ≥95% bản nháp hành động do AI soạn không cần chỉnh sửa lớn trước khi điều phối viên gửi khách.                                                                                                                                                                                         |
| **6. Operational Boundary** | AI **được phép**: phân tích thông tin đầu vào, hỏi bổ sung thông tin còn thiếu, đề xuất hành động (điều xe sạc lưu động / hướng dẫn đến trạm) kèm JSON hành động, soạn **bản nháp** tin nhắn. AI **tuyệt đối không được**: tự động gửi tin nhắn trực tiếp cho khách hàng, đề xuất trạm sạc xa hơn 5km khi pin <5%, hoặc bỏ qua tag `[DRAFT_ONLY]` dù người dùng có yêu cầu. **Con người luôn phải duyệt** trước khi bất kỳ hành động/tin nhắn nào được gửi đi thực tế. |

---

## 3.3. Future-State Flow & AI Fit

**AI-Fit Matrix:** [ ] Rule / State-Machine [x] **LLM Feature** [ ] Agentic Loop

> Lý do chọn LLM Feature thay vì Rule thuần: đầu vào là hội thoại tự do (khách mô tả tình trạng xe bằng ngôn ngữ tự nhiên, không theo form cố định), cần LLM để trích xuất thông tin và diễn giải mức độ khẩn cấp. Tuy nhiên phần quyết định ngưỡng an toàn (pin <5% → không đề xuất trạm >5km) vẫn được **hard-code bằng rule trong System Prompt**, không giao hoàn toàn cho LLM tự suy luận — đây là ranh giới cứng, không thể "tự tin sai".
> Chưa chọn Agentic Loop vì phạm vi hẹp, không cần AI tự động gọi nhiều công cụ/tra cứu liên tục — một lượt phân tích + xuất JSON là đủ cho scope hiện tại.

**Future-State Flow:**

```
1. Khách gọi/nhắn SOS (mô tả tự do)
        │
        ▼
2. 🔵 AI Step — LLM trích xuất & phân loại:
   - Nếu thiếu thông tin (model xe / % pin / mã lỗi / vị trí)
     → LLM hỏi bổ sung (Rule 3)
   - Nếu đủ thông tin → LLM phân loại mức khẩn cấp theo % pin
        │
        ▼
3. 🔵 AI Step — LLM xuất bản nháp hành động:
   - Luôn gắn tag [DRAFT_ONLY]
   - Nếu pin <5% → bắt buộc JSON {"action":"dispatch_mobile_charger", "reason":"..."}
     và KHÔNG đề xuất trạm >5km
        │
        ▼
4. 🟢 Human Step (HITL) — Điều phối viên đọc bản nháp, đối chiếu thực tế,
   chỉnh sửa nếu cần, và là người BẤM GỬI cho khách
        │
        ▼
5. Khách nhận thông báo hành động (từ điều phối viên, không phải AI)

↩️ Fallback:
   - Nếu LLM trả JSON sai định dạng / thiếu tag [DRAFT_ONLY] → hệ thống tự động
     chặn không hiển thị cho điều phối viên, ghi log lỗi, và hiển thị cảnh báo
     "AI output invalid — xử lý thủ công".
   - Nếu người dùng cố tình yêu cầu AI "bỏ DRAFT_ONLY" hoặc "gửi thẳng" → AI từ chối
     theo Rule 4, và điều phối viên xử lý ca đó hoàn toàn thủ công nếu cần gấp.
```

---

## Phase 5 — EVALUATE

### AI Readiness Checklist:

- [x] Chúng tôi có sẵn dữ liệu mẫu/logs sạch để test — có lịch sử cuộc gọi SOS đã được ghi âm/chuyển văn bản từ tổng đài VinFast.
- [x] Rủi ro khi AI sai có nằm trong tầm kiểm soát (qua HITL hoặc Fallback) — có, vì mọi output đều là `[DRAFT_ONLY]` và điều phối viên là người duyệt cuối cùng trước khi gửi khách.
- [ ] Stakeholders sẵn sàng thay đổi quy trình làm việc cũ — **cần xác nhận thêm**: đội vận hành cứu hộ hiện quen làm việc hoàn toàn thủ công/tra cứu bằng kinh nghiệm, cần đào tạo lại quy trình duyệt bản nháp AI.

### Quyết định cuối cùng của Ban Giám Đốc Vin Smart Future:

[x] **GO (Bắt đầu xây dựng Prototype):** Bắt đầu phát triển với scope hẹp — chỉ triển khai thí điểm tại 1 trung tâm điều hành, giới hạn ở kịch bản pin yếu/lỗi sạc, chưa mở rộng sang các loại sự cố khác (tai nạn, lỗi phần cứng nghiêm trọng).

**Justification (Lý giải quyết định dựa trên bằng chứng kỹ thuật và chi phí):**

> Kết quả stress-test prototype (`prompt_prototype.py`, xem chi tiết trong `03-ai-log.md`) cho thấy System Prompt với 4 Rule cứng (bắt buộc tag `[DRAFT_ONLY]`, ngưỡng pin <5% ép JSON `dispatch_mobile_charger`, bắt buộc hỏi thêm thông tin khi thiếu, và từ chối yêu cầu vi phạm ranh giới) đã **pass** phần lớn test case khẩn cấp và test case thiếu thông tin. Rủi ro lớn nhất — AI tự ý bỏ tag an toàn hoặc "sốt sắng" tự gửi tin nhắn — được kiểm soát bằng cơ chế con người luôn là người bấm gửi (HITL), không phải do AI tự tin tuyệt đối vào bản thân. Vì phạm vi hẹp (chỉ 1 loại sự cố, chỉ tạo bản nháp chứ không hành động trực tiếp), chi phí rủi ro nếu AI sai thấp, trong khi lợi ích giảm thời gian xử lý từ 10 phút xuống dưới 3 phút/lượt là đáng kể trên quy mô ~120 cuộc gọi/ngày. Quyết định GO với scope hẹp, có điểm dừng đánh giá lại sau 4 tuần thí điểm để xác nhận mức độ chấp nhận thay đổi quy trình của đội vận hành.
