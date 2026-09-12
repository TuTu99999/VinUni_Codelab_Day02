# Deep Dive Report

# Xanh SM AI Driver Support Assistant

## 1. Problem Statement (6-field)

## User / Actor

**Primary User:**

* Tài xế Xanh SM đang thực hiện chuyến xe.

**Secondary User:**

* Nhân viên vận hành / tổng đài hỗ trợ.

---

## Problem

Khi tài xế gặp các tình huống bất thường như bị kiểm tra giao thông, sự cố xe hoặc khiếu nại khách hàng, quá trình báo cáo và nhận hỗ trợ hiện tại mất nhiều thời gian do phải thu thập thông tin thủ công.

---

## Context

Problem xảy ra trong các tình huống:

* Tài xế đang thực hiện chuyến xe.
* Có sự cố phát sinh cần hỗ trợ.
* Tài xế cần cung cấp thông tin cho bộ phận vận hành.

Ví dụ:

* Bị kiểm tra giấy tờ.
* Va chạm giao thông.
* Khách hàng phản ánh.
* Xe gặp lỗi trong chuyến.

---

## Current Workflow

Workflow hiện tại:

```
Driver gặp sự cố

        ↓

Driver gọi tổng đài hỗ trợ

        ↓

Operator hỏi thông tin:
- ID tài xế
- Vị trí
- Biển số xe
- Trạng thái chuyến
- Nội dung sự việc

        ↓

Operator kiểm tra quy trình

        ↓

Tạo báo cáo sự cố
```

---

## Pain Point

Các điểm nghẽn chính:

### 1. Thu thập thông tin thủ công

Operator phải hỏi lại nhiều câu hỏi giống nhau.

### 2. Thiếu thông tin ban đầu

Driver trong tình huống căng thẳng có thể cung cấp thiếu dữ liệu.

### 3. Báo cáo chưa chuẩn hóa

Thông tin sự cố có thể khác nhau giữa các nhân viên.

---

## Business Impact

Nếu xử lý chậm:

* Tăng thời gian gián đoạn chuyến.
* Tăng workload cho bộ phận vận hành.
* Giảm trải nghiệm tài xế.
* Khó phân tích dữ liệu sự cố.

# 2. Future-State Flow

Workflow sau khi có AI:

```
Driver gặp sự cố

        ↓

AI Driver Support Assistant thu thập thông tin

        ↓

AI tạo Draft Incident Report

        ↓

Safety Rule kiểm tra phạm vi hỗ trợ

        ↓

Human Operator Review

        ↓

Xử lý sự cố
```

---

# AI Responsibility

AI được phép:

* Thu thập thông tin sự cố.
* Hỏi thêm thông tin còn thiếu.
* Chuẩn hóa dữ liệu.
* Tạo bản nháp báo cáo.
* Hướng dẫn quy trình nội bộ.

---

# Human-in-the-loop

Con người kiểm soát:

* Xác nhận báo cáo cuối.
* Đánh giá tình huống đặc biệt.
* Quyết định hướng xử lý.

AI không thay thế nhân viên vận hành.

---

# AI Safety Rules

AI không được:

* Kết luận tài xế đúng hay sai.
* Đưa ra quyết định pháp lý.
* Hướng dẫn né tránh cơ quan chức năng.
* Quyết định tài xế có tiếp tục chuyến hay không.

# Fallback Conditions

AI phải chuyển sang Human Operator khi:

* Có tranh chấp trách nhiệm.
* Có yếu tố pháp lý.
* Có tai nạn nghiêm trọng.
* Thiếu dữ liệu quan trọng.

# 3. Evaluate

## Data Availability

Dữ liệu có thể sử dụng:

* Lịch sử incident report.
* Ticket hỗ trợ.
* Quy trình vận hành nội bộ.
* Thông tin chuyến xe.

## Prototype Feasibility

Có thể prototype bằng:

* Gemini API.
* Prompt engineering.
* Mock incident data.

Phạm vi prototype:

* Thu thập thông tin.
* Sinh draft report.
* Kiểm tra safety rule.

## Legal Risk

Mức độ rủi ro:

**Medium**

Lý do:

AI hỗ trợ quy trình nhưng không được đưa quyết định liên quan đến pháp luật.

## Human Approval

Bắt buộc.

Mọi báo cáo sự cố trước khi xử lý chính thức cần có nhân viên duyệt.

## Success Metrics

KPI đề xuất:

* Giảm thời gian tạo báo cáo từ 10 phút xuống dưới 2 phút.
* Tăng tỷ lệ báo cáo đầy đủ ngay lần đầu >95%.
* Giảm số lần hỏi lại thông tin.
* Không có trường hợp AI đưa kết luận pháp lý.

# Final Decision

## GO

Lý do:

* Problem rõ ràng.
* Workflow hiện tại có bottleneck.
* Có dữ liệu để prototype.
* Có thể kiểm soát rủi ro bằng Human-in-the-loop.

AI nên đóng vai trò trợ lý vận hành, không thay thế quyết định của con người.
