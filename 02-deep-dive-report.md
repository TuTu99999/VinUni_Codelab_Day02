# 02 — Deep-Dive Report

## 1. Project Selected

**Dự án:** Tự động phân tích khiếu nại cước xe, phát hiện route/GPS bất thường và hỗ trợ sinh draft xử lý.

**Đơn vị:** Xanh SM — Customer Support.

> Các baseline và target dưới đây kế thừa worksheet để phục vụ scoping. Trước pilot cần lấy log/ticket thực tế để xác nhận lại.

---

## 2. Phase 3 — DEEP-DIVE

### 2.1 Current-State Workflow

| Step | Actor | Hoạt động | Thời gian ước tính | Handoff / vấn đề |
|---|---|---|---:|---|
| 1 | Khách hàng | Gửi khiếu nại về cước/route | — | 🔄 Customer → CSKH |
| 2 | CSKH Tier-1/2 | Mở ticket, lấy thông tin chuyến đi và bản đồ | 5–10 phút | 🔄 Ticket → dữ liệu GPS |
| 3 | CSKH Tier-2 | So sánh GPS thực tế với route gợi ý, xem điểm bất thường | 10–15 phút | 🔴 **Bottleneck** |
| 4 | CSKH | Tính lại cước theo quy định | 3–5 phút | Có thể sai khi phải tổng hợp nhiều dữ liệu |
| 5 | CSKH | Soạn phản hồi, voucher/đền bù nếu phù hợp | 2–5 phút | 🔄 CSKH → khách hàng |

**Tổng thời gian xử lý thủ công tại các bước 2–5:** khoảng **18–25 phút/case** theo worksheet.

### Bottleneck chính

Bước 2–3 là bottleneck vì nhân viên phải chuyển đổi giữa ticket, bản đồ, dữ liệu GPS và route gợi ý, sau đó tự diễn giải nguyên nhân của chênh lệch.

---

## 3.2 Problem Statement — 6 Fields

| Field | Nội dung |
|---|---|
| **1. Actor / Operator** | Chuyên viên CSKH Tier-2 xử lý các khiếu nại liên quan đến cước và route; khách hàng là người khởi tạo ticket. |
| **2. Current Workflow** | Khách gửi khiếu nại → CSKH lấy thông tin chuyến → mở bản đồ đối soát GPS → so sánh route thực tế/route gợi ý → tính lại cước → soạn phản hồi và xử lý đền bù nếu có. |
| **3. Bottleneck** | Đối soát GPS/route và diễn giải nguyên nhân mất nhiều thời gian, đặc biệt khi dữ liệu có nhiều điểm GPS nhảy, đường vòng hoặc thay đổi do giao thông. |
| **4. Business Impact** | SLA xử lý khiếu nại bị kéo dài; nhân viên phải dành thời gian cho các case có thể tự động tổng hợp bằng dữ liệu; chi phí xử lý ticket tăng. Worksheet đặt baseline xử lý ở 24h và 22.000 VNĐ/ticket. |
| **5. Success Metric** | Giảm thời gian đóng ticket từ **24h xuống < 5 phút** và chi phí xử lý từ **22.000 xuống < 3.000 VNĐ/ticket**, đồng thời không làm tăng tỷ lệ quyết định sai. |
| **6. Operational Boundary** | AI được phép đọc dữ liệu ticket/GPS/route, phát hiện bất thường, phân loại nguyên nhân và sinh draft. AI không được sửa dữ liệu gốc, tự thay đổi biểu phí, tự kết luận lỗi của tài xế khi thiếu bằng chứng, hoặc tự phê duyệt hoàn tiền vượt policy. Case confidence thấp phải chuyển HITL. |

---

## 3.3 Future-State Flow & AI Fit

### AI-Fit Matrix

| Cách tiếp cận | Vai trò | Đánh giá |
|---|---|---|
| **Rule / State Machine** | Kiểm tra điều kiện chắc chắn: thiếu dữ liệu, ngưỡng chênh lệch, policy hoàn tiền. | **Bắt buộc làm lớp guardrail.** |
| **LLM Feature** | Tóm tắt ticket, giải thích route bất thường, phân loại nguyên nhân và sinh draft phản hồi có cấu trúc. | **Phù hợp nhất cho prototype.** |
| **Agentic Loop** | Tự gọi nhiều công cụ và thực hiện chuỗi hành động phức tạp. | Chưa cần ở MVP vì có thể tăng rủi ro và độ khó kiểm soát. |

### Future-State Flow

```text
[Customer submits ticket]
          |
          v
[Rule: validate required data]
          |
      missing? -------- YES --------> [Fallback: human review]
          |
          NO
          v
[Retrieve GPS + route + fare policy]
          |
          v
🔵 [AI: detect route/GPS anomalies]
          |
          v
🔵 [AI: classify likely cause + confidence]
          |
     confidence low?
       /        \
     YES         NO
      |           |
      v           v
🟢 [HITL]     [Rule checks]
      |           |
      +-----+-----+
            |
            v
🔵 [LLM: generate structured draft]
            |
            v
🟢 [HITL: approve refund/response]
            |
       approved?
       /       \
     NO         YES
     |           |
     v           v
[Manual]   [Send response + close]
```

### Human-in-the-loop

HITL được đặt ở quyết định cuối cùng. AI đưa ra **evidence + classification + confidence + draft**, còn nhân viên chịu trách nhiệm phê duyệt phản hồi và khoản đền bù. Những case có dữ liệu thiếu hoặc confidence thấp cũng đi thẳng tới người xử lý.

### Fallback

1. Nếu GPS/route API lỗi → không suy đoán; chuyển case cho CSKH.
2. Nếu LLM lỗi/timeout → sử dụng template phản hồi + rule-based classification.
3. Nếu confidence dưới threshold → HITL.
4. Nếu phát hiện case ngoài policy → HITL.
5. Dữ liệu gốc luôn được giữ nguyên; AI chỉ tạo kết quả trung gian/draft.

---

# 4. Phase 5 — EVALUATE

## AI Readiness Checklist

| Checklist | Đánh giá | Bằng chứng / hành động |
|---|---|---|
| Có dữ liệu mẫu/logs sạch để test? | 🟡 **NOT YET** | Cần tập hợp ticket, GPS, route, fare và nhãn kết quả xử lý; worksheet không cung cấp dataset thật. |
| Rủi ro khi AI sai có kiểm soát được? | 🟢 **YES, nếu áp dụng HITL + Fallback** | AI không được tự sửa dữ liệu hoặc tự phê duyệt hoàn tiền; case rủi ro chuyển người. |
| Stakeholders sẵn sàng thay đổi workflow? | 🟡 **CẦN XÁC NHẬN** | Cần CSKH xác nhận quy trình, policy và quyền phê duyệt trước pilot. |

## Decision

### **NOT YET — Cần tích lũy thêm dữ liệu/xác lập baseline**

**Justification:**

Ý tưởng có AI fit tốt vì phần khó nhất không chỉ là kiểm tra một điều kiện cố định mà còn cần tổng hợp ticket, dữ liệu GPS/route và diễn giải nguyên nhân để hỗ trợ CSKH. LLM phù hợp với phần tóm tắt, phân loại và sinh draft; trong khi Rule đảm nhiệm các policy/guardrail rõ ràng.

Tuy nhiên, chưa nên đánh dấu GO ngay vì worksheet chưa cung cấp dữ liệu production, ground-truth cho các loại bất thường GPS, cũng như baseline accuracy và tỷ lệ false positive/false negative. Ngoài ra, quyết định hoàn tiền có tác động trực tiếp đến chi phí và trải nghiệm khách hàng nên cần HITL.

**Điều kiện để chuyển sang GO:**
- Có tập dữ liệu ticket + GPS/route đã ẩn thông tin nhạy cảm.
- Có nhãn/tiêu chí xác định đúng-sai cho một số nhóm khiếu nại.
- Xác lập baseline rule-based hiện tại.
- Thử nghiệm offline trước khi pilot.
- Đặt threshold confidence và cơ chế audit log.
- CSKH xác nhận policy và quy trình HITL.

**Phạm vi MVP đề xuất:** AI **chỉ phân tích + giải thích + sinh draft**, chưa tự động gửi phản hồi và chưa tự động quyết định hoàn tiền.
