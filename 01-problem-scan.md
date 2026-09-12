# 🔍 Phase 1 — SCAN (Cá nhân, 20 min)

Hãy sử dụng **4 Lenses** dưới đây để quét qua hoạt động vận hành của các công ty thành viên Vingroup. Ghi lại **ít nhất 5 bài toán/bottleneck** thực tế.

### 4 Lenses tìm bài toán AI cho Vingroup:
1. **Lặp lại (Repetitive):** Tác vụ lặp đi lặp lại nhiều lần hằng ngày. (Ví dụ: So khớp hóa đơn sạc điện tại VinFast, route lại chuyến taxi tại Xanh SM).
2. **Tốn thời gian (Time-consuming):** Tác vụ ngốn thời gian xử lý thủ công của nhân viên. (Ví dụ: Soạn thảo phản hồi đánh giá 1-star của cư dân Vinhomes).
3. **AI có thể tốt hơn (AI-upgrade):** Dịch vụ khách hàng hiện tại còn chậm hoặc phản hồi rập khuôn. (Ví dụ: Chatbot CSKH Vinpearl hỗ trợ đặt vé vui chơi).
4. **Pain từ người khác (Stakeholder Pain):** Bottleneck khiến khách hàng hoặc nhân viên thực địa phàn nàn. (Ví dụ: Tài xế Xanh SM phàn nàn về việc hệ thống gợi ý điểm đón khách không chính xác).

> [!TIP]
> **🤖 AI Prompts — Partner brainstorm:**
> Hãy sử dụng prompt sau để brainstorm các bài toán thực tế nếu bạn chưa có ý tưởng:
> *"Tôi là AI Engineer tại Vin Smart Future (Vingroup). Tôi đang tìm kiếm các pain point vận hành cụ thể có thể tối ưu bằng AI cho mảng [Chọn một: VinFast / Xanh SM / Vinhomes / Vinmec]. Hãy gợi ý cho tôi 5 quy trình nghiệp vụ thủ công, tốn nhiều thời gian và gây rò rỉ hiệu suất kèm con số thống kê ước tính về tổn thất."*

### 📝 List bài toán của tôi:
| # | Subsidiary | Lens | Mô tả ngắn bài toán |
|---|----------------------------------|------|---------------------|
| 1 | Xanh SM / V-Green | Operational Efficiency | Tối ưu hóa điều phối trạm sạc thông minh và cân bằng tải lưới sạc |
| 2 | Xanh SM (Fleet Ops) | Quality & Asset Protection | Tự động hóa kiểm tra ngoại quan và phát hiện hư hại xe khi giao nhận ca |
| 3 | Xanh SM (Customer Support) | Customer Experience & Cost | Tự động thẩm định lộ trình GPS bất thường và xử lý khiếu nại cước xe |
| 4 | Xanh SM / VinFast Service | Predictive Maintenance | Dự báo suy hao pin (SOH) và cảnh báo hỏng hóc linh kiện chủ động |
| 5 | Xanh SM (Platform / Anti-Fraud) | Revenue Assurance | Phát hiện gian lận vị trí GPS ảo, cuốc xe ma và trục lợi thưởng tài xế |

---

# 🃏 Phase 2 — QUICK-ASSESS (Cá nhân, 30 min)

Chọn **top 3 bài toán** từ danh sách trên và hoàn thiện **3 Quick Problem Cards** dưới đây (10 phút/card).

```
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #01                                      │
│                                                             │
│ Bài toán: Tự động điều phối xe điện sắp hết pin đến đúng    │
│ trạm sạc khả dụng để giảm tắc nghẽn và thời gian chờ sạc.   │
│ Công ty thành viên: [ ] VinFast  [x] Xanh SM  [ ] Vinhomes  │
│                     [ ] Vinmec   [x] Khác: V-Green          │
│                                                             │
│ Ai đang đau (Actor)? Tài xế Xanh SM & Điều phối viên trạm sạc│
│                                                             │
│ Workflow thủ công hiện tại:                                 │
│   1. Xe báo pin < 20% ──> 2. Tài xế nhìn app tìm trạm ──>   │
│   3. Tự lái tới trạm (thường theo thói quen) ──>            │
│   4. Xếp hàng chờ nếu full trụ / quay đầu tìm trạm khác     │
│                                                             │
│ Bước nào tốn thời gian nhất? Bước 4 (⏱ 35 - 50 phút/xe/ngày)│
│ AI hỗ trợ ở bước nào? Bước 2 & 3: Dự báo tải trạm theo thời │
│ gian thực và tự động phân bổ slot sạc tối ưu lộ trình.      │
│                                                             │
│ Đo thành công bằng gì?                                      │
│   - Giảm thời gian chờ sạc bình quân: 40 phút ──> < 12 phút │
│   - Giảm tỉ lệ cạn pin cứu hộ khẩn cấp: 1.8% ──> < 0.2%    │
│                                                             │
│ Quick Architecture: [ ] No AI  [ ] Rule  [ ] LLM  [x] Agent │
│ (Agent kết hợp Operations Research / Reinforcement Learning)│
└─────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #02                                      │
│                                                             │
│ Bài toán: Tự động phát hiện vết xước móp thân vỏ qua ảnh để │
│ rút ngắn thời gian giao nhận ca và quy đúng trách nhiệm xe. │
│ Công ty thành viên: [ ] VinFast  [x] Xanh SM  [ ] Vinhomes  │
│                     [ ] Vinmec   [ ] Khác                   │
│                                                             │
│ Ai đang đau (Actor)? Đội ngũ Fleet Ops & Tài xế giao ca     │
│                                                             │
│ Workflow thủ công hiện tại:                                 │
│   1. Đưa xe về bãi ──> 2. Đi vòng quanh soi đèn ghi biên bản│
│   3. Chụp 6-8 góc ảnh lưu kho ──> 4. Ký đối soát bàn giao   │
│                                                             │
│ Bước nào tốn thời gian nhất? Bước 2 & 3 (⏱ 12 - 15 phút/xe) │
│ AI hỗ trợ ở bước nào? Bước 3: Computer Vision quét ảnh/video│
│ 360, so khớp với ảnh đầu ca trước để xuất heatmap sai khác. │
│                                                             │
│ Đo thành công bằng gì?                                      │
│   - Thời gian kiểm tra bàn giao: 15 phút ──> < 3 phút/lượt   │
│   - Thất thoát chi phí sơn sửa không rõ thủ phạm: giảm 70%  │
│                                                             │
│ Quick Architecture: [ ] No AI  [ ] Rule  [x] LLM/Vision  [ ] Agent │
│ (Vision Model / Object Detection: YOLOv10 + ViT)            │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #03                                      │
│                                                             │
│ Bài toán: Tự động phân tích khiếu nại cước xe, phát hiện xe │
│ chạy vòng do GPS nhảy/tắc đường và sinh draft giải quyết.   │
│ Công ty thành viên: [ ] VinFast  [x] Xanh SM  [ ] Vinhomes  │
│                     [ ] Vinmec   [ ] Khác                   │
│                                                             │
│ Ai đang đau (Actor)? Chuyên viên CSKH Tier-2 & Khách hàng   │
│                                                             │
│ Workflow thủ công hiện tại:                                 │
│   1. Khách gửi khiếu nại ──> 2. CSKH mở bản đồ đối soát GPS │
│   3. So sánh route thực tế với route gợi ý ──>              │
│   4. Tính lại cước thủ công & gửi email/voucher đền bù      │
│                                                             │
│ Bước nào tốn thời gian nhất? Bước 2 & 3 (⏱ 18 - 25 phút/case)│
│ AI hỗ trợ ở bước nào? Bước 2, 3, 4: Thẩm định bất thường GPS│
│ và soạn sẵn phản hồi kèm quyết định hoàn tiền tự động.      │
│                                                             │
│ Đo thành công bằng gì?                                      │
│   - Thời gian đóng ticket khiếu nại: 24h ──> < 5 phút       │
│   - Chi phí xử lý/ticket: 22.000 VNĐ ──> < 3.000 VNĐ        │
│                                                             │
│ Quick Architecture: [ ] No AI  [ ] Rule  [x] LLM  [ ] Agent │
│ (LLM kết hợp Spatial Map API)                               │
└─────────────────────────────────────────────────────────────┘
```

> [!TIP]
> **🤖 AI Prompts — Stress-Test thẻ bài toán:**
> Hãy dán nội dung thẻ bài toán của bạn vào LLM để nhận phản biện:
> *"Đây là một thẻ bài toán vận hành tôi đề xuất cho Vin Smart Future: [Dán nội dung]. Hãy đóng vai trò là một CFO và Trưởng phòng Vận hành cực kỳ khắt khe, chỉ ra cho tôi 3 điểm yếu về logic, metric, và giải thích vì sao rule-based code thông thường có thể giải quyết bài toán này tốt hơn là dùng AI."*

---