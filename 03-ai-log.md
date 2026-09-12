# 03 — AI Log & Reflection

## 1. Tôi đã dùng AI như thế nào?

Trong quá trình làm Lab, tôi sử dụng AI như một **thought-partner** thay vì coi AI là nguồn sự thật tuyệt đối. Tôi đưa bối cảnh Vin Smart Future và 4 lenses của worksheet vào prompt để brainstorm các pain point có thể áp dụng AI.

AI giúp tôi:
- Mở rộng danh sách bài toán từ các hoạt động vận hành của Xanh SM/V-Green.
- Chuyển một ý tưởng chung thành Quick Problem Card có Actor, Current Workflow, Bottleneck, AI Step và Metric.
- Phản biện xem bài toán có thực sự cần AI hay có thể giải quyết bằng Rule-based.
- Gợi ý cách đặt Operational Boundary, HITL và Fallback.
- Chuyển kết quả scoping thành cấu trúc Markdown để dễ đưa vào deliverable.

## 2. AI đã sai / có nguy cơ hallucination ở đâu?

Điểm cần kiểm soát lớn nhất là **các con số vận hành**. AI có thể đưa ra thời gian xử lý, tỷ lệ lỗi, chi phí hoặc mức tiết kiệm nghe hợp lý nhưng không có nghĩa đó là số liệu thật của Xanh SM/Vingroup.

Vì vậy, tôi không coi các con số brainstorm là fact. Trong deliverable, tôi dùng chúng như **baseline/target giả định từ worksheet để scoping**, và ghi rõ cần xác minh bằng dữ liệu thực tế trước pilot.

Một rủi ro khác là AI có thể mặc định rằng một bài toán “có AI” thì chắc chắn nên dùng LLM. Tôi đã kiểm tra lại bằng cách hỏi ngược: phần nào có thể giải quyết bằng Rule? Kết quả là các policy, ngưỡng và điều kiện chắc chắn nên nằm ở Rule/guardrail, còn LLM chỉ xử lý phần cần hiểu ngôn ngữ và tổng hợp bằng chứng.

## 3. Tôi đã sửa prompt và ranh giới như thế nào?

Tôi chuyển từ prompt kiểu “hãy nghĩ ra một giải pháp AI” sang prompt có các ràng buộc rõ hơn:

1. Yêu cầu AI tách **Fact / Assumption / Recommendation**.
2. Không được tự tạo số liệu production nếu nguồn không cung cấp.
3. Với mỗi đề xuất phải chỉ ra **Actor → Workflow → Bottleneck → AI step → Metric**.
4. Phải đề xuất phương án **Rule-based** trước khi kết luận cần LLM/Agent.
5. Với các tác vụ có tác động tài chính, AI chỉ được **recommend/draft**, không tự phê duyệt.
6. Case thiếu dữ liệu hoặc confidence thấp phải chuyển **Human-in-the-loop**.
7. Khi API/LLM lỗi phải có **Fallback** rõ ràng.

## 4. Bài học cá nhân

Điều tôi rút ra là giá trị của AI Product Engineer không nằm ở việc “nhét AI vào quy trình”, mà nằm ở việc tìm đúng bottleneck, định lượng tác động, xác định ranh giới và chứng minh khi nào AI thực sự tốt hơn Rule-based.

Một prototype tốt cũng không đồng nghĩa với một sản phẩm sẵn sàng triển khai. Cần có dữ liệu thật, baseline, metric và cơ chế kiểm soát lỗi trước khi chuyển từ prototype sang production.
