# Day28_Track01_Track1badao

**Sản phẩm phân tích:** Sen (FIN-05) — AI Agent hỗ trợ tổng đài CSKH ví điện tử, xử lý case chuyển nhầm / chuyển trùng / hoàn tiền.

## 1. Thành viên nhóm

| Họ tên | MSSV | Phần phụ trách | Góp ý đã đưa cho nhóm bạn |
|---|---|---|---|
| Đào Ngọc Bích | 2A202601745 | Khoá phạm vi · chẩn đoán nguyên nhân gốc v1 (Kiến trúc tin cậy + ADKAR + Gartner-Lite + Mollick) · thiết kế AS-IS/TO-BE · hệ chỉ số dashboard · memo quyết định | *(chờ Chặng 3 — chưa phản biện)* |
| Đặng Thái Nam Sơn | 2A202601431 | Rà soát lại chẩn đoán bằng Gartner-Lite: mở Readiness từ 1 lên 4 nhánh (thêm *năng lực vận hành*), hạ Direction xuống ĐẠT CÓ ĐIỀU KIỆN, chỉ ra quan hệ **nối tiếp** NN1–NN2 · siết vòng học (4 nhãn bắt buộc · nhịp họp tuần · chốt kiểm nhãn) và bổ sung kill criteria vào lộ trình · dựng dashboard v2 kèm cột *Thay đổi so với v1* | *(chờ Chặng 3 — chưa phản biện)* |
| Lương Thanh Trang | 2A202601363 | *(đang làm — điền phần phụ trách vào đây)* | *(chờ Chặng 3 — chưa phản biện)* |

> Cột "Góp ý đã đưa cho nhóm bạn" sẽ điền sau Chặng 3 (kiểm tra chéo). Ô trống = chưa tham gia chặng đó, không có căn cứ chấm cho người đó.

## 2. Phạm vi

1 sản phẩm AI: **Sen** — agent trích xuất thông tin giao dịch từ hội thoại và dựng hồ sơ ticket hoàn tiền (đã triển khai, CSKH vẫn xác minh lại thủ công).
1 nhóm người dùng chính: **nhân viên CSKH tuyến 1** xử lý ticket sự cố giao dịch (không phải khách hàng cuối — khách chỉ là người kích hoạt luồng).
3 quy trình: (1) tiếp nhận & trích xuất → tạo ticket · (2) tra cứu chính sách hoàn tiền áp dụng · (3) thẩm định & duyệt lần đầu của CSKH.

## 3. Nguyên nhân gốc

**NN1 — Thiếu tầng kiểm chứng giữa AI và người duyệt** (Kiến trúc tin cậy: có "chuyển người", khuyết *trích nguồn · QA mẫu · phản hồi*). Sen đẩy hồ sơ sang CSKH mà không trích dẫn đoạn chat nguồn, không báo mức tin cậy, nên CSKH không phân biệt được trường nào chắc / trường nào AI đoán → buộc xác minh lại 100%.
**NN2 — Không có owner và vòng học cho tập chính sách hoàn tiền** (Gartner-Lite: Direction ĐẠT CÓ ĐIỀU KIỆN — baseline còn là số giả định; Readiness THIẾU 3/4 nhánh *dữ liệu · governance · năng lực vận hành*, chỉ nhánh *quyền* ĐẠT; Absorption THIẾU). Chính sách mỗi ví một khác và đổi liên tục, không ai chịu trách nhiệm cập nhật → RAG tra cứu lỗi thời, lỗi lặp lại.
*ADKAR: nghẽn ở **Ability**, không phải Awareness/Desire — CSKH buộc phải dùng và muốn giảm tải, nhưng không có công cụ để thẩm định nhanh output AI ⇒ mở lớp đào tạo không giải quyết được.*
*Thứ tự sửa: NN1 và NN2 **không song song**. Không thể trích dẫn phiên bản của một tập dữ liệu chưa có trường phiên bản ⇒ nửa NN1 thuộc tra cứu chính sách bị chặn bởi readiness–dữ liệu của NN2; nửa thuộc hội thoại (trích dẫn đoạn chat + mức tin cậy) thì độc lập, làm được ngay.*

**Bằng chứng:** event `ticket_required_fields_snapshot` (Day23 `06-tracking.md`) có ghi nhận trường bắt buộc còn thiếu nhưng không chặn/cảnh báo gì; Day24 liệt kê chi phí ẩn định kỳ *"gán nhãn lại các ca agent tra cứu sai để cải thiện policy retrieval"* — thừa nhận RAG sai đủ thường xuyên để thành khoản chi; lỗi chỉ lộ qua `ticket_returned_for_info` (sau khi CSKH mất công) hoặc `ticket_reopened_by_customer_complaint` (sau khi khách đã khiếu nại).

## 4. Cách làm mới

**Nguồn kiểm chứng:** mỗi trường Sen điền kèm trích dẫn đoạn chat gốc + mức tin cậy; mỗi đề xuất hoàn tiền kèm tên điều khoản · phiên bản · ngày hiệu lực.
**Người chịu trách nhiệm:** CSKH tuyến 1 ký thẩm định · Trưởng ca duyệt cuối (Sen tuyệt đối không tự duyệt hoàn tiền) · **Policy Owner** (Vận hành CSKH) chịu trách nhiệm tập chính sách đúng-đủ-mới.
**Khi AI không chắc:** không đoán — thiếu trường trọng yếu thì Sen hỏi lại khách trước khi tạo ticket (cổng chặn); không khớp điều khoản thì gắn cờ *"Không xác định — cần người tra"* và đẩy vào hàng đợi ưu tiên, không trộn vào hàng chờ chung.
*Vòng học có nhịp: mỗi ca trả lại phải chọn 1 trong 4 nhãn lý do (bắt buộc, không để trống); họp rà 30 phút mỗi tuần, hiện vật là bảng phân bố loại lỗi, Policy Owner chốt. Chốt kiểm chất lượng nhãn: tỷ lệ "khác" > 15% ⇒ nhãn hỏng, không dùng để cải thiện.*

## 5. Chỉ số

**Product:** % ticket Approved – first pass (baseline 45% → target 70%) · % ticket bị khiếu nại lại trong 7 ngày sau duyệt (baseline 6% → giữ ≤6%, counter-metric chống duyệt ẩu).
**Workflow:** thời gian thẩm định trung vị/ticket (8 phút → 4 phút) · % trường bắt buộc Sen điền đúng ngay lần đầu (62% → 90%) · % ca gắn cờ được xử lý trong SLA 30 phút (mới → 95%).
Chi tiết baseline · target · nguồn dữ liệu · owner · hành động khi chỉ số xấu: xem [dashboard/dashboard_hanh_dong_v2.xlsx](dashboard/dashboard_hanh_dong_v2.xlsx) — bản v2 có thêm cột *Thay đổi so với v1* trên cả 5 sheet. Bản gốc: [v1/dashboard_hanh_dong_v1.xlsx](v1/dashboard_hanh_dong_v1.xlsx).
v2 giữ nguyên **đúng 5 chỉ số** — không thêm chỉ số mới; chốt kiểm chất lượng nhãn nằm ở lộ trình và kill criteria, không đưa lên dashboard.

> **Baseline là số giả định** — Day23/Day24 là bài phân tích, chưa có log vận hành thật. Việc đo baseline thật là hạng mục đầu tiên của giai đoạn 0–30 ngày.

## 6. Quyết định

**SỬA** — không tiếp tục nguyên trạng, không dừng.
Lý do: nhu cầu và chỗ đứng của AI trong workflow là có thật, cái hỏng là kiến trúc tin cậy — nên sửa tầng kiểm chứng trước, chưa mở rộng phạm vi.
Hai thay đổi so với v1 *từ phản biện chéo*: *(chờ Chặng 3 — điền sau khi nhận phản biện chéo)*

