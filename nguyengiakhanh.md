# Lab 02 — Bài làm cá nhân

**Họ và tên:** Nguyễn Gia Khánh  
**Vai trò:** AI Product Engineer — Vin Smart Future  
**Phạm vi khảo sát:** VinFast, Xanh SM, Vinhomes và Vinpearl/VinWonders  
**Lưu ý dữ liệu:** Các con số trong phần SCAN và Quick Cards là ước tính dùng để scoping ban đầu, cần được kiểm chứng bằng log vận hành trước khi triển khai.

---

## Phase 1 — SCAN: Tìm kiếm cơ hội

Tôi sử dụng bốn lens trong worksheet: Lặp lại, Tốn thời gian, AI có thể tốt hơn và Pain từ người khác. Tôi ưu tiên các bài toán có quy trình lặp lại, đầu vào dạng văn bản hoặc dữ liệu có cấu trúc, có người kiểm tra kết quả và có thể đo được thời gian xử lý.

| # | Công ty thành viên | Lens | Bài toán/bottleneck quan sát được | Ước tính ban đầu |
|---:|---|---|---|---|
| 1 | **VinFast** | Lặp lại | Nhân viên hậu mãi phải đọc và phân loại thủ công các phiếu báo lỗi xe điện theo hệ thống, bộ phận và mức độ ưu tiên. | 150–250 phiếu/ngày; 3–5 phút/phiếu |
| 2 | **Xanh SM** | Tốn thời gian | Điều phối viên tiếp nhận cuộc gọi tài xế báo hết pin/sự cố sạc, sau đó tra GPS, kiểm tra trạm và soạn hướng dẫn. | 15 phút/sự cố; khoảng 80 sự cố/ngày tại một khu vực lớn |
| 3 | **Vinhomes** | AI có thể tốt hơn | Phản ánh của cư dân về thang máy, vệ sinh, bãi xe và tiện ích bị route thủ công hoặc trả lời theo mẫu chung. | 10–15 phút để phân loại/route; SLA phản hồi ban đầu có thể tới 12 giờ |
| 4 | **Vinpearl/VinWonders** | Pain từ người khác | Nhân viên chăm sóc khách phải tìm nhiều nguồn để trả lời câu hỏi về vé, giờ diễn, quy định đổi vé và tiện ích. | 4–6 phút/câu hỏi; khách phải chờ khi cao điểm |
| 5 | **VinFast** | Tốn thời gian | Bộ phận bảo hành phải tổng hợp lịch sử sửa chữa và triệu chứng từ nhiều ghi chú trước khi chuyển ca kỹ thuật. | 10–20 phút/ca; nguy cơ bỏ sót thông tin trong ghi chú tự do |
| 6 | **Xanh SM** | AI có thể tốt hơn | Tóm tắt lý do khách hủy chuyến từ cuộc gọi và ghi chú của tài xế để tìm nhóm nguyên nhân lặp lại. | 20–30 phút tổng hợp theo ca; dữ liệu khó so sánh nếu không chuẩn hóa |

### Nhận xét sau khi SCAN

- Các bài toán #1 và #3 có thể bắt đầu bằng **rule-based routing**, sau đó dùng LLM ở phần đọc hiểu ngôn ngữ tự nhiên.
- Các bài toán #2 và #5 có tác động vận hành trực tiếp nhưng cần ranh giới an toàn và bắt buộc human-in-the-loop.
- Bài toán #4 phù hợp với LLM nhưng cần kết nối kho tri thức được phê duyệt; chatbot không được tự bịa giá vé, lịch diễn hoặc chính sách.
- Bài toán #6 phù hợp cho phân tích offline, ít rủi ro hơn các tác vụ điều phối real-time.

---

## Phase 2 — QUICK-ASSESS: Ba Quick Problem Cards

Tôi chọn ba bài toán **#2, #3 và #4**. Cả ba đều có đầu vào là văn bản tự nhiên, điểm nghẽn dễ quan sát và có thể giữ người vận hành ở bước phê duyệt cuối. Tôi không chọn bài toán liên quan đến quyết định lâm sàng của Vinmec vì rủi ro cao và cần phạm vi dữ liệu/pháp lý riêng.

### Quick Problem Card #2 — Xanh SM xử lý sự cố sạc pin

**Bài toán một câu:** Khi tài xế báo hết pin hoặc lỗi sạc, điều phối viên mất nhiều thời gian thu thập thông tin, tìm phương án gần nhất và soạn tin hướng dẫn.

**Công ty:** [x] Xanh SM (GSM)  
**Actor đang gặp khó khăn:** Tài xế cần được hỗ trợ nhanh; điều phối viên phải xử lý nhiều cuộc gọi cùng lúc.

**Workflow thủ công hiện tại:**

1. Tài xế gọi tổng đài và mô tả sự cố.
2. Điều phối viên xác minh biển số, dòng xe, mức pin và vị trí GPS.
3. Điều phối viên tra dashboard trạm sạc còn khả dụng, khoảng cách và loại cổng.
4. Điều phối viên soạn tin nhắn hướng dẫn hoặc gọi đội hỗ trợ.
5. Điều phối viên gửi phương án sau khi tự kiểm tra lại.

**Bước tốn thời gian/lỗi nhất:** Bước 2–4, khoảng **10–15 phút/lượt**; dễ nhầm vị trí, loại xe hoặc chọn trạm quá xa khi pin thấp.

**AI hỗ trợ ở đâu:** Trích xuất thông tin từ cuộc gọi/ghi chú, lấy dữ liệu GPS và trạm qua API, sau đó tạo bản nháp JSON/tin nhắn cho điều phối viên review.

**Metric thành công:**

- Giảm thời gian xử lý trung bình từ **15 phút xuống dưới 3 phút/lượt**.
- Đạt **≥98%** bản nháp có đúng biển số, vị trí và loại xe trong mẫu kiểm thử.
- **0** trường hợp gửi tự động khi chưa có phê duyệt.

**Quick Architecture:** [ ] No AI  [ ] Rule  [x] LLM Feature + Rule safety gate  [ ] Agent  
**Lý do:** Quy trình có cấu trúc cố định; LLM chỉ đọc hiểu và soạn nháp, còn ngưỡng pin/khoảng cách phải do code kiểm soát.

### Quick Problem Card #3 — Vinhomes phân loại phản ánh cư dân

**Bài toán một câu:** Phản ánh tự do của cư dân trên ứng dụng cần được phân loại, xác định mức ưu tiên và chuyển đúng bộ phận nhanh hơn.

**Công ty:** [x] Vinhomes  
**Actor đang gặp khó khăn:** Nhân viên CSKH, ban quản lý tòa nhà và cư dân chờ phản hồi.

**Workflow thủ công hiện tại:**

1. Cư dân gửi phản ánh bằng văn bản, ảnh hoặc mô tả ngắn.
2. CSKH đọc nội dung và tự gắn nhóm vấn đề.
3. CSKH chuyển ticket qua bộ phận kỹ thuật, an ninh, vệ sinh hoặc phí dịch vụ.
4. Bộ phận nhận ticket kiểm tra và cập nhật trạng thái.
5. CSKH soạn phản hồi cho cư dân.

**Bước tốn thời gian/lỗi nhất:** Bước 2–3, khoảng **8–12 phút/ticket**; một phản ánh có thể bị chuyển nhầm hoặc chuyển nhiều lần.

**AI hỗ trợ ở đâu:** Phân loại ý định, trích xuất tòa/căn hộ/khu vực và đề xuất bộ phận nhận ticket; sinh bản nháp phản hồi dựa trên dữ liệu đã được duyệt.

**Metric thành công:**

- Giảm thời gian route từ **10 phút xuống dưới 1 phút/ticket**.
- **≥90%** ticket được route đúng ngay lần đầu trong pilot.
- Giảm tỷ lệ ticket chuyển lại giữa các bộ phận từ giả định **15% xuống dưới 5%**.

**Quick Architecture:** [ ] No AI  [ ] Rule  [x] LLM Feature + Rule-based routing  [ ] Agent  
**Ranh giới sơ bộ:** AI không tự kết luận tranh chấp phí, bồi thường hoặc vi phạm hợp đồng; các nhóm này phải chuyển người có thẩm quyền.

### Quick Problem Card #4 — Vinpearl/VinWonders trợ lý tra cứu chính sách vé

**Bài toán một câu:** Nhân viên CSKH cần tra cứu nhanh câu trả lời nhất quán cho câu hỏi về vé, giờ hoạt động, đổi vé và tiện ích tại khu vui chơi.

**Công ty:** [x] Vinpearl/VinWonders  
**Actor đang gặp khó khăn:** Nhân viên tổng đài/quầy dịch vụ và khách du lịch, đặc biệt trong giờ cao điểm.

**Workflow thủ công hiện tại:**

1. Khách hỏi qua chat, điện thoại hoặc quầy dịch vụ.
2. Nhân viên tìm thông tin trong website, file hướng dẫn hoặc hỏi bộ phận vận hành.
3. Nhân viên đối chiếu ngày áp dụng và loại vé.
4. Nhân viên tự soạn câu trả lời và ghi chú trường hợp đặc biệt.

**Bước tốn thời gian/lỗi nhất:** Bước 2–3, khoảng **4–6 phút/câu hỏi**; chính sách thay đổi có thể khiến câu trả lời giữa các kênh không đồng nhất.

**AI hỗ trợ ở đâu:** RAG trên kho tài liệu chính sách đã phê duyệt, trả lời kèm nguồn tài liệu/ngày hiệu lực và tạo bản nháp cho nhân viên.

**Metric thành công:**

- Giảm thời gian tìm thông tin từ **5 phút xuống dưới 45 giây/câu hỏi**.
- **≥95%** câu trả lời trong pilot có nguồn tài liệu hợp lệ.
- Tỷ lệ nhân viên phải tra cứu lại giảm từ giả định **25% xuống dưới 10%**.

**Quick Architecture:** [ ] No AI  [ ] Rule  [x] LLM Feature (RAG)  [ ] Agent  
**Ranh giới sơ bộ:** Nếu không tìm thấy tài liệu hoặc tài liệu hết hiệu lực, AI phải nói “chưa đủ thông tin” và chuyển nhân viên; không được tự xác nhận hoàn tiền hay thay đổi đặt chỗ.

---

## So sánh và lựa chọn ưu tiên cá nhân

| Bài toán | Giá trị vận hành | Rủi ro | Mức sẵn sàng prototype |
|---|---|---|---|
| Xanh SM — sự cố sạc | Rất cao, ảnh hưởng trực tiếp thời gian xe nằm chờ | Cao nếu chọn sai trạm hoặc tự gửi tin | **Ưu tiên 1: GO có scope hẹp** |
| Vinhomes — route phản ánh | Cao, nhiều ticket và dễ đo SLA | Trung bình, cần escalation nhóm nhạy cảm | **Ưu tiên 2: Pilot có HITL** |
| Vinpearl — tra cứu chính sách | Trung bình-cao, cải thiện tốc độ CSKH | Trung bình, phụ thuộc độ mới của tài liệu | **Ưu tiên 3: Not Yet nếu chưa có kho tài liệu chuẩn** |

Bài toán tôi đề xuất đưa vào prototype chính là **Xanh SM — xử lý sự cố sạc pin**. Đây là tác vụ đủ cụ thể để kiểm thử nhưng vẫn thể hiện rõ yêu cầu operational boundary: mọi đầu ra là draft và pin dưới 5% phải chuyển sang điều xe sạc di động, không được đề xuất trạm cách trên 5 km.

---

## Liên hệ với Phase 4 — Prompt Prototype cá nhân

Tôi đã hoàn thiện `starter-code/prompt_prototype.py` để stress-test hai ranh giới của bài toán ưu tiên:

1. Output phải bắt đầu bằng `[DRAFT_ONLY]`, kể cả khi người dùng yêu cầu gửi tin ngay hoặc bỏ qua bước duyệt.
2. Khi pin dưới 5%, output phải có action `dispatch_mobile_charger` và không đưa ra trạm sạc cách trên 5 km.

Các test adversarial gồm: yêu cầu chỉ đường 8 km khi pin 2%, yêu cầu xóa tag draft và yêu cầu giả vờ rằng cứu hộ đã được điều đi. Ngoài system prompt, prototype còn có safety gate bằng code để không phụ thuộc hoàn toàn vào việc LLM tự tuân thủ.

---

## Kết luận cá nhân

Qua quá trình SCAN, tôi nhận ra không phải tác vụ nào có chữ “AI” cũng cần Agent tự trị. Những quy trình có luật rõ ràng nên giữ phần quyết định an toàn bằng rule/state-machine; LLM chỉ đảm nhiệm đọc hiểu, tóm tắt hoặc soạn bản nháp. Với tác vụ điều phối xe, human-in-the-loop và fallback thủ công là điều kiện bắt buộc trước khi mở rộng pilot.

