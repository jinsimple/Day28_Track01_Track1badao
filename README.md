# Day28_Track01_Track1badao

**Sản phẩm phân tích:** Sen (FIN-05) — AI Agent hỗ trợ tổng đài CSKH ví điện tử, xử lý case chuyển nhầm / chuyển trùng / hoàn tiền.

## 1. Thành viên nhóm

| Họ tên | MSSV | Phần phụ trách | Góp ý đã đưa cho nhóm bạn |
|---|---|---|---|
| Đào Ngọc Bích | 2A202601745 | Khoá phạm vi · chẩn đoán nguyên nhân gốc (Kiến trúc tin cậy + ADKAR + Gartner-Lite) · thiết kế AS-IS/TO-BE · hệ chỉ số dashboard | *(chờ Chặng 3 — chưa phản biện)* |
| Lương Thanh Trang | 2A202601363 | Rà soát logic dashboard · đối chiếu product metric và workflow metric với nguồn dữ liệu, owner và hành động khi chỉ số xấu · dựng bản v2 sau phản biện (CH1–CH4) và nhật ký thay đổi · hoàn thiện README | Nhóm ______: *(điền góp ý đã đưa)* |
| | | | |
| | | | |

> **Còn phải điền tay trước khi nộp:** tên nhóm được phản biện và nội dung góp ý đã đưa cho họ (cột 4, cả hai thành viên), cùng ô "Nguồn góp ý" ở sheet *6. Thay doi v1 - v2*. Ô trống = chưa tham gia Chặng 3, không tính điểm cho người đó.

## 2. Phạm vi

1 sản phẩm AI: **Sen** — agent trích xuất thông tin giao dịch từ hội thoại và dựng hồ sơ ticket hoàn tiền (đã triển khai, CSKH vẫn xác minh lại thủ công).
1 nhóm người dùng chính: **nhân viên CSKH tuyến 1** xử lý ticket sự cố giao dịch (không phải khách hàng cuối — khách chỉ là người kích hoạt luồng).
3 quy trình: (1) tiếp nhận & trích xuất → tạo ticket · (2) tra cứu chính sách hoàn tiền áp dụng · (3) thẩm định & duyệt lần đầu của CSKH.

## 3. Nguyên nhân gốc

**NN1 — Thiếu tầng kiểm chứng giữa AI và người duyệt** (Kiến trúc tin cậy: có "chuyển người", khuyết *trích nguồn · QA mẫu · phản hồi*). Sen đẩy hồ sơ sang CSKH mà không trích dẫn đoạn chat nguồn, không báo mức tin cậy, nên CSKH không phân biệt được trường nào chắc / trường nào AI đoán → buộc xác minh lại 100%.
**NN2 — Không có owner và vòng học cho tập chính sách hoàn tiền** (Gartner-Lite: Direction ĐẠT, Readiness–governance THIẾU, Absorption THIẾU). Chính sách mỗi ví một khác và đổi liên tục, không ai chịu trách nhiệm cập nhật → RAG tra cứu lỗi thời, lỗi lặp lại.
*ADKAR: nghẽn ở **Ability**, không phải Awareness/Desire — CSKH buộc phải dùng và muốn giảm tải, nhưng không có công cụ để thẩm định nhanh output AI ⇒ mở lớp đào tạo không giải quyết được.*

**Bằng chứng:** event `ticket_required_fields_snapshot` (Day23 `06-tracking.md`) có ghi nhận trường bắt buộc còn thiếu nhưng không chặn/cảnh báo gì; Day24 liệt kê chi phí ẩn định kỳ *"gán nhãn lại các ca agent tra cứu sai để cải thiện policy retrieval"* — thừa nhận RAG sai đủ thường xuyên để thành khoản chi; lỗi chỉ lộ qua `ticket_returned_for_info` (sau khi CSKH mất công) hoặc `ticket_reopened_by_customer_complaint` (sau khi khách đã khiếu nại).

## 4. Cách làm mới

**Nguồn kiểm chứng:** mỗi trường Sen điền kèm trích dẫn đoạn chat gốc + mức tin cậy; mỗi đề xuất hoàn tiền kèm tên điều khoản · phiên bản · ngày hiệu lực.
**Người chịu trách nhiệm:** CSKH tuyến 1 ký thẩm định · Trưởng ca duyệt cuối (Sen tuyệt đối không tự duyệt hoàn tiền) · **Policy Owner** (Vận hành CSKH) chịu trách nhiệm tập chính sách đúng-đủ-mới.
**Khi AI không chắc:** không đoán — thiếu trường trọng yếu thì Sen hỏi lại khách trước khi tạo ticket (cổng chặn); không khớp điều khoản thì gắn cờ *"Không xác định — cần người tra"* và đẩy vào hàng đợi ưu tiên, không trộn vào hàng chờ chung.

## 5. Chỉ số

**Product:** M1 % ticket Approved – first pass (baseline 45% → target 70%) · M2 % ticket bị khiếu nại lại trong 7 ngày sau duyệt (baseline 6% → giữ ≤6%, counter-metric chống duyệt ẩu) · **M6 chi phí gán nhãn lại ca tra cứu sai mỗi tháng** (chưa đo → giảm ≥40% so với baseline tháng đầu — *thêm ở v2*).
**Workflow:** M3 thời gian thẩm định trung vị/ticket (8 phút → 4 phút) · M4 % trường bắt buộc Sen điền đúng ngay lần đầu (62% → 90%) · M5 % ca gắn cờ được xử lý trong SLA 30 phút (mới → 95%).
Chi tiết baseline · target · nguồn dữ liệu · owner · hành động khi chỉ số xấu: xem [dashboard/dashboard_hanh_dong_v2.xlsx](dashboard/dashboard_hanh_dong_v2.xlsx).

> **Baseline là số giả định** — Day23/Day24 là bài phân tích, chưa có log vận hành thật. Việc đo baseline thật là hạng mục đầu tiên của giai đoạn 0–30 ngày.

## 6. Quyết định

**SỬA** — không tiếp tục nguyên trạng, không dừng.
Lý do: nhu cầu và chỗ đứng của AI trong workflow là có thật, cái hỏng là kiến trúc tin cậy — nên sửa tầng kiểm chứng trước, chưa mở rộng phạm vi.

**Bốn thay đổi so với v1** (chi tiết ở sheet *6. Thay doi v1 - v2* trong dashboard v2):
1. **CH1 — quy tắc đo M3.** Thời gian thẩm định lấy thô từ log là thời gian *trôi qua*, không phải công sức. Bổ sung: loại ca >30 phút khỏi mẫu và báo cáo riêng tỷ lệ bị loại; ca mở lại nhiều lần chỉ tính lần mở cuối.
2. **CH2 — thêm chỉ số M6.** Tầng 5 (Giá trị) trước đây chỉ có counter-metric âm. Thêm chi phí gán nhãn lại ca tra cứu sai mỗi tháng — chỉ số duy nhất trả lời "Sen tạo tiết kiệm ròng hay chỉ dời việc sang nhóm gán nhãn".
3. **CH3 — tách quyền owner M4.** Siết ngưỡng cổng chặn làm khách bị hỏi lại nhiều hơn ⇒ không để Kỹ sư AI quyết một mình; phải có đồng duyệt của Trưởng nhóm CSKH.
4. **CH4 — thêm kill criteria cho M5.** SLA 30 phút không đạt sau giai đoạn 30–60 ⇒ không mở rộng: hoặc bố trí người trực hàng đợi cờ đỏ, hoặc tắt abstention.
