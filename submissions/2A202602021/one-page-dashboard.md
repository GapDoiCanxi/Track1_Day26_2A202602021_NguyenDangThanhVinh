# Operating Dashboard - AI Travel Planner

**Model:** B2C · **Cập nhật:** 2026-08-28 · **Owner phiên họp:** Product Lead

**Chẩn đoán:** Khách du lịch cá nhân tự trả phí và trực tiếp dùng web/app; bạn đồng hành chỉ nhận lịch trình, không có quan hệ sản phẩm độc lập.

**North Star:** M1 paid retention · 93,0% theo mô hình, actual chưa đo · mục tiêu >=93,0% · trạng thái 🔧

## Cây đèn 3 tầng

| Tầng · ID | Metric và định nghĩa ngắn | Hiện tại · 🟢 / 🟡 / 🔴 · Nguồn | Nhịp · Owner | Báo trước cho · Luật |
|---|---|---|---|---|
| L · L-01 | M1 paid retention: paid user active ngày 30 / paid user đủ cửa sổ | 93,0% model · >=93,0 / 89,5-92,9 / <89,5% · [MH] MH-01 | Tháng · Product Analytics | LTV, renewal, payback · R-01 |
| L · L-02 | Activation 24h: new paid user lưu itinerary trong 24h / new paid user | Chưa đo · >=TB+10 / TB +/-9,9 / <TB-10 điểm % · [TB] 2 cohort x 100 user | Tuần · Product Growth | M1 retention · R-02 |
| O · O-01 | Grounded pass: job pass nguồn, giờ mở cửa, tuyến / mẫu QA | Chưa đo · >=TB+10 / TB đến TB+9,9 / <TB điểm % · [TB] 2 tuần x 100 job | Tuần · AI Quality Lead | Retention, support cost · R-03 |
| O · O-02 | Cost/Job: token + inference + travel API + retry / successful job | Chưa đo · <=TB / >TB-1,20xTB / >1,20xTB · [TB] 2 tuần, >=500 job | Tuần · FinOps | AI COGS/user, GM · R-04 |
| O · O-03 | AI COGS/paid user: API + hidden QA + infrastructure / active paid user | 33.000đ model · <=33.000 / 33.001-55.000 / >55.000đ · [MH] MH-02 | Tháng · FinOps | Gross Margin · R-04 |
| G · G-01 | Gross Margin: (ARPU - COGS) / ARPU | 77,9% model · >=77,9 / 53,8-77,8 / <53,8% · [MH] MH-03 | Tháng · FinOps | Hybrid pricing viability · R-05 |
| G · G-02 | CAC payback: CAC / monthly gross profit | 2,76 tháng model · <=2,76 / 2,77-12 / >12 tháng · [MH] MH-04 | Tháng · Growth Finance | Runway và scale · R-05 |

## Luật quyết định

| ID | NẾU · TRONG · VÀ | THÌ | KHÔNG THÌ | Dừng? |
|---|---|---|---|---|
| R-01 | M1 retention <89,5% · 2 cohort tháng · mỗi cohort >=100 paid user | Dừng paid acquisition kênh đỏ 14 ngày; phỏng vấn 15 churned user và sửa một return use case | Không tăng ad spend hay khuyến mãi acquisition | CÓ |
| R-02 | Activation 24h <TB-10 điểm % · 2 tuần · >=100 new paid user/tuần | Đóng băng experiment mới; sửa ba friction lớn nhất trong 7 ngày | Không giảm giá subscription để che first-value thấp | KHÔNG |
| R-03 | Grounded pass <TB · 1 tuần · QA >=100 job và có 1 destination cluster lỗi | Dừng output công khai cluster lỗi trong 24h; chuyển draft-only và sửa nguồn trong 7 ngày | Không mở thêm destination hoặc tăng traffic cluster đỏ | CÓ |
| R-04 | Cost/Job >1,20xTB hoặc COGS/user >55.000đ · 2 tuần · >=500 job và 100 paid user | Cap retry/context, cache travel lookup và tạm dừng mode tạo tốn phí | Không tăng free allowance hay tăng giá đồng loạt | KHÔNG |
| R-05 | GM <53,8% hoặc payback >12 tháng · 2 tháng · >=2 cohort và 200 paid user | Dừng paid acquisition/package expansion; tái thiết kế pricing trong 14 ngày | Không trợ giá bằng free regeneration | CÓ |

## Cổng 90 ngày

| Ngày | Một metric · ngưỡng | Evidence | Đạt / Trượt |
|---:|---|---|---|
| 30 | Event log đầy đủ >=95% trên >=200 successful job | `itinerary-event-log.csv` + data dictionary | GO / FIX |
| 60 | Cost/Job <=1,20xTB trên >=500 successful job trong 2 tuần | `cost-ledger.csv` + invoice + successful-job log | GO / FIX |
| 90 | M1 paid retention >=89,5% trên >=2 cohort x 100 paid user | cohort report + event export + churn reasons đã ẩn danh | GO / KILL |

**Kill criteria:** Ngày 90, nếu M1 paid retention thực đo <89,5% trên ít nhất 2 cohort, mỗi cohort >=100 paid user đủ cửa sổ 30 ngày, dừng paid pilot và không mở thêm paid acquisition cho đến khi retention về vùng vàng.

**Chưa đo được:** M1 retention actual, Activation 24h, Grounded pass, Cost/Job và Value Metric Day 25; owner/deadline chi tiết nằm trong worksheet nguồn, mốc gần nhất 2026-09-11.

---

# Phụ lục - phép tính và tính trung thực

## MH-01 · M1 paid retention

- Base churn Day 24 = **7,0%/tháng**; Pessimistic churn = **10,5%/tháng**.
- Base retention = `1 - 7,0%` = **93,0%**.
- Stress retention = `1 - 10,5%` = **89,5%**.
- Áp dụng L-01: xanh >=93,0%; vàng 89,5%-92,9%; đỏ <89,5%.

## MH-02 · AI COGS trên paid user

- Base = 18.000đ API + 9.000đ hidden + 6.000đ infra = **33.000đ/user/tháng**.
- Pessimistic = 30.000đ + 15.000đ + 10.000đ = **55.000đ/user/tháng**.
- Áp dụng O-03: xanh <=33.000đ; vàng 33.001-55.000đ; đỏ >55.000đ.

## MH-03 · Gross Margin

- Base GM = `(149.000 - 33.000) / 149.000` = **77,85% ≈ 77,9%**.
- Stress GM = `(119.000 - 55.000) / 119.000` = **53,78% ≈ 53,8%**.
- Áp dụng G-01: xanh >=77,9%; vàng 53,8%-77,8%; đỏ <53,8%.

## MH-04 · CAC payback

- Monthly gross profit = 149.000đ - 33.000đ = **116.000đ/user/tháng**.
- Payback = 320.000đ / 116.000đ = **2,7586 ≈ 2,76 tháng**.
- Áp dụng G-02: xanh <=2,76 tháng; vàng 2,77-12 tháng; đỏ >12 tháng.

## Evidence và khoảng trống

- Nguồn mô hình: `2A202602021_NguyenDangThanhVinh_Day24.xlsx`, các tab `1. Assumptions`, `2. Unit Economics`, `3. P&L & ROI`.
- Day 25 chưa hoàn thành nên Cost/Job là `[TB]`, không phải `[MH]`.
- ARPU, retention, COGS, GM và CAC payback trong dashboard là **model values**, chưa phải actual production.
- Cần: event taxonomy và cost ledger trước 2026-09-11; M1 cohort actual trước 2026-10-28.
