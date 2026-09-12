# AI Usage Log

# 1. AI hỗ trợ tôi như thế nào?

Trong quá trình thực hiện Lab 02, AI được sử dụng như một công cụ hỗ trợ tư duy sản phẩm.

## Brainstorm Problem Opportunity

AI hỗ trợ tìm kiếm các cơ hội ứng dụng AI trong hệ sinh thái Vingroup:

* VinFast.
* Xanh SM.
* Vinhomes.
* Vinpearl/VinWonders.

AI giúp so sánh các bài toán dựa trên:

* Tần suất xảy ra.
* Mức độ tốn thời gian.
* Khả năng áp dụng AI.
* Giá trị vận hành.

---

## Xây dựng SCAN Framework

AI hỗ trợ tạo cấu trúc đánh giá:

* Problem.
* Bottleneck.
* Impact.
* AI Opportunity.

Sau đó tôi kiểm tra lại để đảm bảo bài toán phù hợp với thực tế vận hành.

---

## Phân tích Workflow

AI giúp mô hình hóa:

Workflow hiện tại:

```
Driver
 ↓
Call Center
 ↓
Operator
 ↓
Report
```

Workflow tương lai:

```
Driver
 ↓
AI Assistant
 ↓
Draft Report
 ↓
Human Review
```

---

## Thiết kế Prompt Prototype

AI hỗ trợ xây dựng:

* System prompt.
* Safety rules.
* Test cases.

Mục tiêu là đảm bảo AI chỉ hỗ trợ, không tự đưa quyết định.

# 2. AI có thể sai ở đâu?

## Sai lệch dữ liệu

AI có thể đưa ra giả định về:

* Quy trình nội bộ.
* Dữ liệu vận hành.
* Chính sách công ty.

Vì vậy cần kiểm chứng trước khi triển khai.

---

## Tự động hóa quá mức

AI có thể đề xuất tự động hóa những quyết định cần con người.

Ví dụ:

Sai:

"AI quyết định tài xế có vi phạm hay không."

Đúng:

"AI tạo bản nháp báo cáo để nhân viên xem xét."

---

## Rủi ro pháp lý

AI không nên:

* Kết luận trách nhiệm.
* Đưa tư vấn pháp luật.
* Thay thế nhân viên xử lý.

# 3. Prompt Improvement

## Prompt ban đầu

```
AI xử lý sự cố giao thông cho tài xế.
```

## Vấn đề

Prompt này quá rộng.

AI có thể hiểu rằng nó được phép:

* Quyết định hướng xử lý.
* Đánh giá đúng sai.
* Đưa ra tư vấn pháp lý.

---

## Prompt sau cải tiến

```
Bạn là Xanh SM AI Driver Support Assistant.

Nhiệm vụ:
- Thu thập thông tin sự cố.
- Tạo draft incident report.
- Hướng dẫn tài xế theo quy trình.

Giới hạn:
- Không kết luận đúng sai.
- Không đưa quyết định pháp lý.
- Không hướng dẫn né tránh cơ quan chức năng.
- Khi vượt phạm vi phải chuyển Human Operator.
```

# 4. Bài học cá nhân

Qua quá trình sử dụng AI, tôi nhận ra:

* AI hiệu quả nhất khi giải quyết công việc lặp lại.
* AI cần được giới hạn phạm vi rõ ràng.
* Human-in-the-loop là yếu tố quan trọng với các bài toán vận hành có rủi ro.

AI không thay thế con người mà giúp con người xử lý nhanh hơn và chính xác hơn.
