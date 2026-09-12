# 03 — AI Analysis Log: Phát hiện tài xế bị "kẹt" trong vùng nhu cầu thấp

> **Ngày tạo:** 2026-09-12  
> **Nguồn:** [Problem.md](file:///c:/Users/ADMIN/VinUni_Codelab_Day02/Problem.md)  
> **Mục đích:** Ghi lại quá trình tư duy, phân tích và các quyết định khi tiếp cận bài toán

---

## Log Entry #1 — Đọc hiểu bài toán

**Thời gian:** 2026-09-12 10:11  
**Hành động:** Đọc và phân tích Problem.md

### Quan sát

- Bài toán được mô tả rất chi tiết: 17 mục, từ bối cảnh → MVP → luồng xử lý.
- Bài toán **không chỉ là phát hiện tài xế rảnh** mà là **quyết định hành động** (STAY vs REPOSITION).
- Có 3 nhóm stakeholder rõ ràng: tài xế, điều phối viên, khách hàng.
- Bài toán nhấn mạnh: **di chuyển không phải lúc nào cũng tốt** — đây là insight quan trọng.

### Nhận định

- Đây là bài toán **prescriptive analytics** (không chỉ descriptive hay predictive).
- Cần 3 AI components riêng biệt, không nên gộp thành 1 model.
- Bài toán có tính chất **multi-agent coordination** — recommendation cho 1 tài xế ảnh hưởng đến tài xế khác.

---

## Log Entry #2 — Phân tích chuỗi xử lý

**Thời gian:** 2026-09-12 10:12  
**Hành động:** Phân tích pipeline Detect → Predict → Decide → Recommend

### Phân tích từng bước

```
Step 1: DETECT
├── Input: idle_time, demand, supply, ratio
├── Output: is_at_risk (boolean)
├── Complexity: LOW
└── Approach: Rule-based hoặc simple classifier

Step 2: PREDICT
├── Input: driver features + zone features + time features
├── Output: stuck_probability (0–1)
├── Complexity: MEDIUM
└── Approach: Supervised ML (classification)

Step 3: DECIDE
├── Input: stuck_probability + expected values
├── Output: STAY or MOVE
├── Complexity: MEDIUM-HIGH
└── Approach: Decision framework (expected value comparison)

Step 4: RECOMMEND
├── Input: candidate zones + scores
├── Output: target zone + reasoning
├── Complexity: HIGH
└── Approach: Scoring function + global coordination
```

### Nhận định

- Step 1–2 có thể gộp trong MVP.
- Step 3–4 là phần khó nhất — cần cân bằng lợi ích vs chi phí.
- Step 4 cần **global view** để tránh tất cả tài xế đổ về cùng 1 zone.

---

## Log Entry #3 — Phân tích edge cases

**Thời gian:** 2026-09-12 10:13  
**Hành động:** Đánh giá 4 edge cases mà hệ thống KHÔNG nên đề xuất di chuyển

### Edge Cases Analysis

| Case | Scenario | Rủi ro nếu bỏ qua | Giải pháp |
|------|----------|---------------------|-----------|
| **#1** | Nhu cầu sắp tăng tại chỗ | Tài xế rời đi ngay trước khi khách đến | Demand forecast với horizon ngắn (5–10 phút) |
| **#2** | Zone thay thế quá xa | Chi phí di chuyển > lợi ích | Hard cap trên khoảng cách + travel time |
| **#3** | Giao thông ùn tắc | Tài xế mất quá nhiều thời gian trên đường | Integrate real-time traffic data |
| **#4** | Quá nhiều tài xế cùng đến | Oversupply tại zone đích | Capacity-aware recommendation + quota |

### Insight quan trọng

> Case #4 là khó nhất vì nó đòi hỏi **coordination giữa các recommendation**. Nếu mỗi recommendation được tính independently, hệ thống có thể đồng thời gửi 50 tài xế đến cùng 1 zone.

**Giải pháp đề xuất cho MVP:**
- Đặt **capacity cap** cho mỗi zone: max N tài xế được recommend đến zone X trong window T phút.
- Khi cap đạt → fallback sang zone tiếp theo hoặc STAY.

---

## Log Entry #4 — Phân tích dữ liệu

**Thời gian:** 2026-09-12 10:14  
**Hành động:** Đánh giá tính khả dụng của dữ liệu

### Data Readiness Assessment

| Dữ liệu | Khả năng có sẵn | Ưu tiên |
|----------|-----------------|---------|
| Driver location (GPS) | ✅ Rất cao — tracking real-time | **P0** |
| Driver status | ✅ Rất cao — từ app driver | **P0** |
| Booking data | ✅ Rất cao — từ booking system | **P0** |
| Trip history | ✅ Cao — stored in DB | **P0** |
| Zone definitions | ⚠️ Cần define — H3 hexagon hoặc admin zones | **P0** |
| Travel time matrix | ⚠️ Trung bình — cần tính hoặc call Google Maps API | **P1** |
| Weather | ⚠️ Cần integrate — API bên ngoài | **P2** |
| Traffic real-time | ⚠️ Trung bình — Google/HERE API | **P2** |
| Events | ❌ Thấp — manual hoặc crawl | **P3** |

### Nhận định

- **MVP khả thi** với P0 data — driver, booking, trip, time.
- Zone definition là quyết định thiết kế quan trọng:
  - **H3 hexagonal grid** → đồng nhất, dễ aggregate, không bias.
  - **Administrative zones** → có ý nghĩa kinh doanh nhưng kích thước không đều.
  - **Gợi ý:** H3 resolution 8 (~460m edge) hoặc 7 (~1.2km edge) cho MVP.

---

## Log Entry #5 — Đánh giá approach cho từng AI component

**Thời gian:** 2026-09-12 10:15  
**Hành động:** So sánh các approach cho 3 AI components

### Component 1: Demand Forecasting

| Approach | Ưu điểm | Nhược điểm | Phù hợp |
|----------|---------|------------|---------|
| **Moving average** | Đơn giản, nhanh | Không capture pattern phức tạp | MVP quick-start |
| **Prophet** | Tự động handle seasonality | Cần nhiều data lịch sử | MVP |
| **LightGBM** | Flexible features, mạnh | Cần feature engineering | MVP–V1 |
| **LSTM/Seq2Seq** | Capture temporal patterns tốt | Phức tạp, chậm train | V2+ |
| **Transformer** | State-of-the-art | Overkill cho MVP | V3+ |

**Gợi ý:** LightGBM cho MVP (dễ deploy, nhanh inference, interpretable).

### Component 2: Stuck Risk Prediction

| Approach | Ưu điểm | Nhược điểm | Phù hợp |
|----------|---------|------------|---------|
| **Rule-based** | Đơn giản, explainable | Không adaptive | Quick-start |
| **Logistic Regression** | Fast, interpretable | Limited capacity | MVP |
| **XGBoost/LightGBM** | Accurate, handles mixed features | Less interpretable | MVP–V1 |
| **Neural Network** | High capacity | Overkill, need more data | V2+ |

**Gợi ý:** Rule-based cho quick-start → LightGBM cho MVP chính thức.

### Component 3: Reposition Recommendation

| Approach | Ưu điểm | Nhược điểm | Phù hợp |
|----------|---------|------------|---------|
| **Scoring function** | Simple, explainable, tunable | Static weights | MVP |
| **Multi-armed bandit** | Learns from feedback | Need online infrastructure | V1 |
| **Contextual bandit** | Context-aware exploration | Complex infrastructure | V2 |
| **Reinforcement Learning** | Optimal long-term | Very complex, unstable | V3+ |

**Gợi ý:** Scoring function cho MVP → Contextual bandit cho V2.

---

## Log Entry #6 — Thiết kế evaluation strategy

**Thời gian:** 2026-09-12 10:16  
**Hành động:** Xác định cách đánh giá hệ thống

### Offline Evaluation

```
Historical data → Simulate recommendations → Compare outcomes
```

- **Backtesting:** Với data quá khứ, nếu hệ thống recommend REPOSITION, tài xế có nhận chuyến nhanh hơn không?
- **Counterfactual:** So sánh idle time thực tế vs. idle time dự kiến nếu follow recommendation.

### Online Evaluation (A/B Test)

```
Control group:  Không nhận recommendation
Treatment group: Nhận recommendation từ AI
```

- **Primary metric:** Average idle time (phút)
- **Secondary metrics:** Trip count / giờ online, empty driving km, customer waiting time
- **Guard rails:** Không tăng cancellation rate, không tăng empty km

### Đánh giá mô hình riêng

| Model | Metric | Target |
|-------|--------|--------|
| Demand Forecast | MAE / RMSE | < 20% error |
| Stuck Risk | Precision@HIGH | > 70% |
| Stuck Risk | Recall@HIGH | > 80% |
| Recommendation | Acceptance rate | > 40% (MVP) |
| Recommendation | Success rate | > 60% |

---

## Log Entry #7 — Tổng hợp & Kết luận

**Thời gian:** 2026-09-12 10:17  
**Hành động:** Tổng hợp các findings

### Key Findings

1. **Bài toán có 4 layers** (Detect → Predict → Decide → Recommend), không nên gộp.
2. **Edge case quan trọng nhất** là coordination — tránh gửi nhiều tài xế đến cùng zone.
3. **MVP hoàn toàn khả thi** với dữ liệu driver + booking + time.
4. **Scoring function + LightGBM** là approach cân bằng giữa hiệu quả và độ phức tạp cho MVP.
5. **Evaluation cần A/B test** — offline metrics không đủ để đánh giá real-world impact.

### Rủi ro chính cần theo dõi

- 🔴 **Multi-agent coordination**: Recommendation cho tài xế A ảnh hưởng đến tài xế B.
- 🟡 **Feedback loop**: Tài xế follow recommendation → thay đổi pattern → model cần re-train.
- 🟡 **Trust**: Nếu recommendation sai nhiều lần → tài xế mất niềm tin → acceptance giảm.

### Next Steps

| # | Action | Responsible |
|---|--------|-------------|
| 1 | Xác nhận dữ liệu khả dụng với team Data Engineering | Data Team |
| 2 | Define zone schema (H3 resolution) | ML Team |
| 3 | Build baseline demand forecast model | ML Team |
| 4 | Implement rule-based stuck detection | Backend Team |
| 5 | Design recommendation UI cho driver app | Product + Design |
| 6 | Setup A/B test infrastructure | Platform Team |

---

> **Xem thêm:**  
> → [01-problem-scan.md](file:///c:/Users/ADMIN/VinUni_Codelab_Day02/01-problem-scan.md) — Tổng quan nhanh  
> → [02-deep-dive-report.md](file:///c:/Users/ADMIN/VinUni_Codelab_Day02/02-deep-dive-report.md) — Phân tích chi tiết  
> → [04-workflow-diagram.png](file:///c:/Users/ADMIN/VinUni_Codelab_Day02/04-workflow-diagram.png) — Sơ đồ luồng xử lý
