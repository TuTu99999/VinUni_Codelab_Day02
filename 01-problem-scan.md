# 01 — Problem Scan: Phát hiện tài xế bị "kẹt" trong vùng nhu cầu thấp

> **Ngày tạo:** 2026-09-12  
> **Nguồn:** [Problem.md](file:///c:/Users/ADMIN/VinUni_Codelab_Day02/Problem.md)  
> **Trạng thái:** Initial Scan

---

## 1. Tóm tắt vấn đề

| Hạng mục | Mô tả |
|---|---|
| **Tên bài toán** | Phát hiện tài xế đang bị "kẹt" trong vùng nhu cầu thấp |
| **Lĩnh vực** | Vận hành taxi công nghệ (ride-hailing operations) |
| **Bản chất** | Mất cân bằng cung–cầu theo không gian và thời gian |
| **Câu hỏi cốt lõi** | Tài xế nên **tiếp tục chờ** hay **di chuyển** sang khu vực khác? |

---

## 2. Stakeholders & Pain Points

### Tài xế
- Không biết nên chờ hay di chuyển.
- Quyết định dựa vào cảm tính, kinh nghiệm cá nhân.
- Di chuyển rỗng (không khách) tạo ra chi phí thực tế: thời gian, nhiên liệu, pin.

### Nhân viên điều phối
- Khó theo dõi hàng trăm/ngàn tài xế thủ công.
- Cần biết không chỉ *"ai đang idle?"* mà *"ai có nguy cơ tiếp tục idle?"*.

### Khách hàng
- Khu vực thiếu xe → thời gian chờ tăng, ETA xấu.
- Nguy cơ hủy chuyến tăng khi không được ghép tài xế kịp thời.

---

## 3. Tình huống minh họa

```
┌─────────────────────────────┐     ┌─────────────────────────────┐
│         KHU VỰC A           │     │         KHU VỰC B           │
│                             │     │                             │
│  • Nhu cầu: THẤP            │     │  • Nhu cầu: CAO             │
│  • Tài xế rảnh: NHIỀU (38)  │     │  • Tài xế rảnh: ÍT          │
│  • Idle time: 20–25 phút    │     │  • Khách đang chờ           │
│  • Dự báo 15': vẫn thấp     │     │  • Dự báo 15': vẫn cao      │
│                             │     │                             │
│          Chờ mãi...         │     │           Cần xe!            │
└─────────────────────────────┘     └─────────────────────────────┘
              │                                    │
              │         Khoảng cách: 2–3 km        │
              └──────────── ← → ──────────────────┘
```

**Nếu tài xế ở A di chuyển sang B** → tăng khả năng nhận chuyến.  
**Nhưng không phải lúc nào cũng nên di chuyển** → cần tính chi phí, giao thông, rủi ro.

---

## 4. Định nghĩa "Bị kẹt"

> **Không phải** kẹt vật lý.  
> **Mà là:** Tài xế đang ở khu vực có xác suất nhận chuyến thấp, có nguy cơ tiếp tục idle, trong khi tồn tại lựa chọn tốt hơn.

**Các yếu tố đánh giá:**

| # | Yếu tố | Mô tả |
|---|--------|-------|
| 1 | Idle duration | Tài xế đã chờ bao lâu? |
| 2 | Nhu cầu hiện tại | Số yêu cầu đặt xe tại khu vực |
| 3 | Supply hiện tại | Số tài xế rảnh trong khu vực |
| 4 | Next-trip probability | Xác suất nhận chuyến trong 10–15 phút |
| 5 | Demand forecast | Nhu cầu dự kiến 15–30 phút tới |
| 6 | Khu vực thay thế | Có nơi nào triển vọng hơn không? |
| 7 | Chi phí di chuyển | Khoảng cách, thời gian, giao thông |

---

## 5. Nguyên nhân gốc rễ

```mermaid
fishbone-diagram
    title Nguyên nhân tài xế bị "kẹt"
```

| Nguyên nhân | Chi tiết |
|---|---|
| **Thiếu thông tin cung–cầu** | Hệ thống chưa cung cấp tỷ lệ xe/nhu cầu theo khu vực real-time |
| **Chỉ nhìn hiện tại** | Không dự báo nhu cầu tương lai gần (15–30 phút) |
| **Chưa đánh giá xác suất cá nhân** | Hai tài xế cùng vị trí, cùng idle ≠ cùng khả năng nhận chuyến |
| **Không tính chi phí di chuyển** | Đề xuất khu vực nhu cầu cao nhất ≠ khu vực tối ưu nhất |

---

## 6. Chuỗi câu hỏi hệ thống cần trả lời

```
  ┌──────────┐    ┌───────────┐    ┌──────────┐    ┌─────────────┐
  │  DETECT  │ →  │  PREDICT  │ →  │  DECIDE  │ →  │  RECOMMEND  │
  └──────────┘    └───────────┘    └──────────┘    └─────────────┘
       │                │               │                 │
  Ai đang có       Xác suất       Chờ hay          Di chuyển
  nguy cơ idle     tiếp tục       di chuyển?       đến đâu?
  kéo dài?         idle?
```

---

## 7. Phạm vi tác động

| Đối tượng | Tác động kỳ vọng |
|---|---|
| **Tài xế** | ↓ idle time, ↑ chuyến, ↓ chạy rỗng |
| **Khách hàng** | ↓ thời gian chờ, ↓ hủy chuyến, ↑ ETA chính xác |
| **Doanh nghiệp** | ↑ vehicle utilization, ↓ supply-demand imbalance |

---

## 8. Đánh giá sơ bộ

| Tiêu chí | Đánh giá |
|---|---|
| **Mức độ nghiêm trọng** | 🔴 Cao — ảnh hưởng trực tiếp đến doanh thu và trải nghiệm |
| **Tần suất xảy ra** | 🔴 Cao — xảy ra liên tục, đặc biệt giờ thấp điểm |
| **Khả năng giải quyết bằng AI** | 🟢 Cao — có dữ liệu, có pattern rõ ràng |
| **Mức độ phức tạp** | 🟡 Trung bình — cần kết hợp nhiều nguồn dữ liệu |
| **ROI tiềm năng** | 🟢 Cao — giảm lãng phí tài nguyên trên toàn hệ thống |

---

> **Kết luận scan:** Bài toán có giá trị cao, khả thi, và nên được đưa vào deep-dive phân tích chi tiết.  
> → Xem tiếp: [02-deep-dive-report.md](file:///c:/Users/ADMIN/VinUni_Codelab_Day02/02-deep-dive-report.md)
