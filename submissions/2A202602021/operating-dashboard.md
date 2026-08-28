# Operating Dashboard - AI Travel Planner

- Học viên: Nguyễn Đăng Thành Vinh
- Mã học viên: 2A202602021
- Mô hình: B2C
- Cập nhật: 2026-08-28
- North Star: M1 paid retention đạt ít nhất 93,0% cho cohort người dùng trả phí

## Chẩn đoán mô hình

Khách du lịch cá nhân Việt Nam là người trực tiếp trả phí và cũng là người dùng AI Travel Planner để tạo, lưu và chia sẻ lịch trình. Bạn đồng hành nhận lịch trình là người bị tác động nhưng không có hợp đồng, tài khoản trả phí hay quan hệ sản phẩm độc lập. Sản phẩm phân phối trực tiếp qua web/app và thu hybrid subscription cộng usage vượt hạn mức, vì vậy đây là B2C, không phải B2B2C. Theo profile B2C, đèn bật trước là đường cong retention có phẳng hay không; M1 paid retention được chọn làm North Star vì phản ánh liệu người dùng có quay lại lập kế hoạch cho nhu cầu tiếp theo trước khi doanh thu, gross margin và CAC payback xấu đi.

| Dữ liệu đầu vào | Trạng thái | Nằm ở đâu hoặc cần gì để đo | Ngày có số |
|---|---|---|---|
| Unit economics Day 24 | ĐO ĐƯỢC THEO MÔ HÌNH | `2A202602021_NguyenDangThanhVinh_Day24.xlsx`: Base ARPU 149.000đ/user/tháng; API 18.000đ; hidden 9.000đ; infra 6.000đ; churn 7%; CAC 320.000đ; GM 77,9%; payback 2,76 tháng | 2026-08-28 |
| Value Metric và Cost/Job Day 25 | CHƯA ĐO | Day 25 chưa hoàn thành; cần event taxonomy cho successful itinerary job và billing ledger join theo `job_id` trong 2 tuần | 2026-09-11 |

## Kiểm kê đèn ứng viên

| Đèn ứng viên từ handbook | Tầng | Trạng thái | Bằng chứng hiện có hoặc kế hoạch đo |
|---|---|---|---|
| M1 paid retention | L | 🔧 | Day 24 giả định churn 7% nên retention mô hình là 93%; cần subscription cohort thực tế |
| Activation trong 24 giờ | L | 🔧 | Đã định nghĩa first value là itinerary đầu tiên được lưu; cần event `itinerary_saved` nối với thời điểm thanh toán |
| W1 retained usage | L | ❌ | Chưa có event cohort tuần; cần user active tạo hoặc sửa itinerary ở ngày 7 |
| Brief completion | O | 🔧 | Có luồng nhập điểm đến, ngày và ngân sách; cần funnel event cho từng trường bắt buộc |
| Grounded itinerary pass | O | 🔧 | Đã có tiêu chí nguồn, giờ mở cửa và tuyến; cần QA sample tối thiểu 100 job/tuần |
| Cost/Job | O | ❌ | Day 25 chưa có job volume; cần token, inference, travel API, retry và successful job cùng `job_id` |
| AI COGS trên paid user | O | ✅ | Day 24 Base tính API 18.000đ + hidden 9.000đ + infra 6.000đ = 33.000đ/user/tháng |
| Gross Margin | G | ✅ | Day 24 Unit Economics Base tính 77,9%; đây là model value, chưa phải actual ledger |
| CAC payback | G | ✅ | Day 24 Unit Economics Base tính 2,76 tháng từ CAC 320.000đ và gross profit 116.000đ/tháng |
| Referral/share rate | G | ❌ | Chưa có tracking cho link share và người nhận mở lịch trình; bổ sung share event trước pilot |

## Đèn báo sớm

| ID | Đèn | Định nghĩa và công thức | Nhịp · Owner | Hiện tại | 🟢 | 🟡 | 🔴 | Nguồn | Ngày kiểm tra | Báo trước cho | Luật |
|---|---|---|---|---:|---|---|---|---|---|---|---|
| L-01 | M1 paid retention | Số paid user còn active ở ngày 30 chia cho paid user đủ cửa sổ 30 ngày trong cùng cohort | THÁNG · PRODUCT ANALYTICS | 93,0% theo mô hình; actual chưa đo | >=93,0% | 89,5%-92,9% | <89,5% | [MH] MH-01; 93% là Base từ churn 7%, 89,5% là stress case từ churn 10,5%; vùng đỏ dừng acquisition kênh có retention thấp | 2026-08-28 | LTV, renewal và CAC payback | R-01 |
| L-02 | Activation trong 24 giờ | New paid user lưu ít nhất một itinerary trong 24 giờ sau thanh toán chia cho toàn bộ new paid user | TUẦN · PRODUCT GROWTH | Chưa đo: chưa có event cohort | >=TB + 10 điểm % | TB - 9,9 đến TB + 9,9 điểm % | <TB - 10 điểm % | [TB] Baseline từ 2 cohort tuần, mỗi cohort ít nhất 100 new paid user; vùng đỏ chỉ ra onboarding chưa tạo first value và kích hoạt sửa funnel; có số 2026-09-11 | 2026-08-28 | M1 paid retention | R-02 |

## Đèn vận hành

| ID | Đèn | Định nghĩa và công thức | Nhịp · Owner | Hiện tại | 🟢 | 🟡 | 🔴 | Nguồn | Ngày kiểm tra | Báo trước cho | Luật |
|---|---|---|---|---:|---|---|---|---|---|---|---|
| O-01 | Grounded itinerary pass | Successful itinerary job pass kiểm tra nguồn, giờ mở cửa, tính hợp lý tuyến và không có claim không truy vết chia cho tổng job trong mẫu QA | TUẦN · AI QUALITY LEAD | Chưa đo: chưa có QA sample | >=TB + 10 điểm % | TB đến TB + 9,9 điểm % | <TB | [TB] Baseline bằng 2 mẫu tuần, mỗi mẫu ngẫu nhiên ít nhất 100 job; dưới baseline chuyển output sang draft-only trước khi mở thêm điểm đến; có số 2026-09-11 | 2026-08-28 | Retention và support cost | R-03 |
| O-02 | Cost/Job | Tổng token, inference, travel API và retry trong cửa sổ chia cho số successful itinerary job | TUẦN · FINOPS | Chưa đo: Day 25 chưa có job volume | <=TB | >TB đến 1,20 x TB | >1,20 x TB | [TB] Join billing với successful job bằng `job_id` trong 2 tuần, tối thiểu 500 job; vượt 20% baseline kích hoạt cap retry/context; có số 2026-09-11 | 2026-08-28 | AI COGS/user và Gross Margin | R-04 |
| O-03 | AI COGS trên paid user | Tổng API, hidden QA và infrastructure trong tháng chia cho số paid user active | THÁNG · FINOPS | 33.000đ/user/tháng theo Base | <=33.000đ | 33.001-55.000đ | >55.000đ | [MH] MH-02; 33.000đ là Base và 55.000đ là Pessimistic Day 24, nên vượt stress case phải dừng mode tạo tốn phí | 2026-08-28 | Gross Margin | R-04 |

## Đèn kết quả

| ID | Đèn | Định nghĩa và công thức | Nhịp · Owner | Hiện tại | 🟢 | 🟡 | 🔴 | Nguồn | Ngày kiểm tra | Báo trước cho | Luật |
|---|---|---|---|---:|---|---|---|---|---|---|---|
| G-01 | Gross Margin | ARPU trừ total COGS trên paid user, chia cho ARPU | THÁNG · FINOPS | 77,9% theo mô hình Base | >=77,9% | 53,8%-77,8% | <53,8% | [MH] MH-03; 77,9% là Base và 53,8% là Pessimistic Day 24, nên dưới stress case dừng package/acquisition hiện tại | 2026-08-28 | Khả năng sống của hybrid pricing | R-05 |
| G-02 | CAC payback | CAC chia cho monthly gross profit trên paid user | THÁNG · GROWTH FINANCE | 2,76 tháng theo mô hình Base | <=2,76 tháng | 2,77-12 tháng | >12 tháng | [MH] MH-04; 2,76 tháng là Base và 12 tháng là checkpoint tối đa trong Day 24, nên vùng đỏ dừng paid acquisition | 2026-08-28 | Runway và khả năng scale | R-05 |

## Luật quyết định

| ID | NẾU | TRONG | VÀ | THÌ | KHÔNG THÌ | Luật dừng? |
|---|---|---|---|---|---|---|
| R-01 | L-01 M1 paid retention <89,5% | 2 cohort tháng liên tiếp | Mỗi cohort có ít nhất 100 paid user đủ cửa sổ 30 ngày | Dừng paid acquisition của kênh có retention đỏ trong 14 ngày; phỏng vấn tối thiểu 15 churned user và sửa một use case quay lại trước khi mở lại | Không tăng ad spend hoặc bù retention thấp bằng khuyến mãi acquisition | CÓ |
| R-02 | L-02 Activation 24h <TB - 10 điểm % | 2 tuần liên tiếp | Có ít nhất 100 new paid user trong mỗi tuần | Trong 7 ngày đóng băng experiment mới, sửa ba friction lớn nhất trong brief/onboarding và chỉ mở lại experiment khi activation về vùng vàng | Không giảm giá subscription để che first-value thấp | KHÔNG |
| R-03 | O-01 Grounded itinerary pass <TB | 1 tuần | Mẫu QA ngẫu nhiên có ít nhất 100 job và lỗi tập trung ở ít nhất 1 destination cluster | Trong 24 giờ dừng output công khai cho cluster lỗi, chuyển draft-only và sửa nguồn/route rule trong 7 ngày | Không mở thêm destination hoặc tăng traffic cho cluster đang đỏ | CÓ |
| R-04 | O-02 Cost/Job >1,20 x TB hoặc O-03 AI COGS/user >55.000đ | 2 tuần liên tiếp | Có ít nhất 500 successful job và 100 paid user trong cửa sổ | Trong release kế tiếp đặt cap retry/context, cache travel lookup và tạm dừng mode tạo tốn phí cho đến khi chi phí về vùng vàng | Không tăng hạn mức usage miễn phí hoặc tăng giá đồng loạt trước khi tách cohort gây vượt phí | KHÔNG |
| R-05 | G-01 Gross Margin <53,8% hoặc G-02 CAC payback >12 tháng | 2 tháng liên tiếp | Có ít nhất 2 paid cohort và tổng cộng 200 paid user | Dừng paid acquisition và mở rộng package hiện tại; trong 14 ngày tái thiết kế pricing/allowance và chỉ mở lại khi metric về vùng vàng | Không trợ giá bằng free regeneration hoặc giữ kênh CAC cao chỉ để tăng user count | CÓ |

## Cổng gác 90 ngày

| Ngày | Metric gác cổng | Ngưỡng | Bằng chứng vật lý | Nếu đạt | Nếu trượt |
|---:|---|---|---|---|---|
| 30 | Successful job có event log đầy đủ | >=95% trên ít nhất 200 job, gồm `user_id`, `job_id`, source status, retry count và outcome | `itinerary-event-log.csv` và data dictionary đã được Product Analytics ký duyệt | GO | FIX |
| 60 | Cost/Job | <=1,20 x TB trên ít nhất 500 successful job trong 2 tuần | `cost-ledger.csv`, invoice API/cloud/travel API và export successful-job log | GO | FIX |
| 90 | M1 paid retention | >=89,5% trên ít nhất 2 cohort, mỗi cohort có 100 paid user đủ cửa sổ 30 ngày | subscription cohort report, event export và danh sách churn reason đã ẩn danh | GO | KILL |

## Kill criteria

Tại ngày 90, nếu M1 paid retention thực đo vẫn dưới 89,5% trên ít nhất 2 cohort và mỗi cohort có tối thiểu 100 paid user đủ cửa sổ 30 ngày, dừng paid pilot và không mở thêm paid acquisition cho đến khi một use case quay lại chứng minh retention vùng vàng.

## Chưa đo được

| Đèn hoặc giả định | Cần gì để đo | Ai chịu trách nhiệm | Ngày có số |
|---|---|---|---|
| M1 paid retention thực tế | Subscription cohort đủ cửa sổ 30 ngày, event active và churn reason đã ẩn danh | Product Analytics | 2026-10-28 |
| Activation trong 24 giờ | Payment timestamp, event `itinerary_saved` và cohort tối thiểu 100 new paid user/tuần | Product Growth | 2026-09-11 |
| Grounded itinerary pass | Hai mẫu QA ngẫu nhiên, mỗi mẫu tối thiểu 100 job có reviewer outcome | AI Quality Lead | 2026-09-11 |
| Cost/Job | Billing token/inference/travel API/retry join successful job bằng `job_id` trong 2 tuần | FinOps | 2026-09-11 |
| Value Metric Day 25 | Chốt định nghĩa successful itinerary job, event taxonomy và allocation cost theo job | Product Ops | 2026-09-11 |

## Phụ lục ngưỡng suy từ mô hình

| ID | Metric | Input Day 24-25 | Phép tính | Kết quả và ngưỡng áp dụng |
|---|---|---|---|---|
| MH-01 | M1 paid retention | Base churn 7,0%/tháng; Pessimistic churn 10,5%/tháng | Base retention = 1 - 7,0% = 93,0%; stress retention = 1 - 10,5% = 89,5% | L-01: xanh >=93,0%; vàng 89,5%-92,9%; đỏ <89,5% |
| MH-02 | AI COGS trên paid user | Base API 18.000đ + hidden 9.000đ + infra 6.000đ; Pessimistic API 30.000đ + hidden 15.000đ + infra 10.000đ | Base COGS = 18.000 + 9.000 + 6.000 = 33.000đ/user/tháng; stress COGS = 30.000 + 15.000 + 10.000 = 55.000đ/user/tháng | O-03: xanh <=33.000đ; vàng 33.001-55.000đ; đỏ >55.000đ |
| MH-03 | Gross Margin | Base ARPU 149.000đ và COGS 33.000đ; Pessimistic ARPU 119.000đ và COGS 55.000đ | Base GM = (149.000 - 33.000) / 149.000 = 77,85% = 77,9%; stress GM = (119.000 - 55.000) / 119.000 = 53,78% = 53,8% | G-01: xanh >=77,9%; vàng 53,8%-77,8%; đỏ <53,8% |
| MH-04 | CAC payback | Base CAC 320.000đ; ARPU 149.000đ/user/tháng; COGS 33.000đ/user/tháng; checkpoint tối đa 12 tháng | Monthly gross profit = 149.000 - 33.000 = 116.000đ; payback = 320.000 / 116.000 = 2,7586 = 2,76 tháng | G-02: xanh <=2,76 tháng; vàng 2,77-12 tháng; đỏ >12 tháng |

## Ghi nhận AI critique

| Phản biện | Chấp nhận hay bác bỏ | Thay đổi đã thực hiện | Lý do |
|---|---|---|---|
| B2C phải dùng retention làm đèn bật trước, không dùng Cost/Job làm North Star | CHẤP NHẬN | Đổi North Star thành M1 paid retention >=93,0%; Cost/Job chuyển về tầng vận hành | Retention phản ánh người dùng quay lại trước khi unit economics đủ dữ liệu để kết luận scale |
| Không được gọi Cost/Job là `[MH]` khi Day 25 chưa có job volume | CHẤP NHẬN | Gắn Cost/Job `[TB]`, nêu cách join billing theo `job_id`, sample và ngày có số | Giữ tính trung thực và đúng quy tắc nguồn của lab |
| Mỗi cổng 30/60/90 chỉ nên có một metric chính | CHẤP NHẬN | Day 30 dùng event completeness, Day 60 dùng Cost/Job, Day 90 dùng M1 paid retention | Tránh ghép nhiều tiêu chí khiến quyết định GO/FIX/KILL không rõ |
