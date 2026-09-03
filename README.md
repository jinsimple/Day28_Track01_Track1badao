# Day28_Track01_Track1badao

**Sản phẩm phân tích:** Sen (FIN-05) — AI Agent hỗ trợ tổng đài CSKH ví điện tử, xử lý case chuyển nhầm / chuyển trùng / hoàn tiền.

## 1. Thành viên nhóm

| Họ tên | MSSV | Phần phụ trách | Góp ý đã đưa cho nhóm bạn |
|---|---|---|---|
| Đào Ngọc Bích | 2A202601745 | Khoá phạm vi · chẩn đoán nguyên nhân gốc v1 (Kiến trúc tin cậy + ADKAR + Gartner-Lite + Mollick) · thiết kế AS-IS/TO-BE · hệ chỉ số dashboard · memo quyết định | *(chờ Chặng 3 — chưa phản biện)* |
| Đặng Thái Nam Sơn | 2A202601431 | Rà soát lại chẩn đoán bằng Gartner-Lite: mở Readiness từ 1 lên 4 nhánh (thêm *năng lực vận hành*), hạ Direction xuống ĐẠT CÓ ĐIỀU KIỆN, chỉ ra quan hệ **nối tiếp** NN1–NN2 · siết vòng học (4 nhãn bắt buộc · nhịp họp tuần · chốt kiểm nhãn) và bổ sung kill criteria vào lộ trình · dựng dashboard v2 kèm cột *Thay đổi so với v1* | *(chờ Chặng 3 — chưa phản biện)* |
| Lương Thanh Trang | 2A202601363 | Rà soát logic dashboard · đối chiếu product metric và workflow metric với nguồn dữ liệu, owner và hành động khi chỉ số xấu · rà soát nội bộ bản v1 theo trục chỉ số, thực hiện bốn thay đổi CH1–CH4 · dựng sheet *6. Thay doi v1 - v2* | *(chờ Chặng 3 — chưa phản biện)* |

> **Cột 4 đang trống vì Chặng 3 (kiểm tra chéo với nhóm khác) chưa diễn ra** — mọi thay đổi từ v1 sang v2 tới lúc này đều đến từ rà soát chéo nội bộ trong nhóm. Phải điền tay trước khi nộp: tên nhóm được phản biện và nội dung góp ý cả ba thành viên đã đưa cho họ.

## 2. Phạm vi

1 sản phẩm AI: **Sen** — agent trích xuất thông tin giao dịch từ hội thoại và dựng hồ sơ ticket hoàn tiền (đã triển khai, CSKH vẫn xác minh lại thủ công).
1 nhóm người dùng chính: **nhân viên CSKH tuyến 1** xử lý ticket sự cố giao dịch (không phải khách hàng cuối — khách chỉ là người kích hoạt luồng).
3 quy trình, viết theo góc người dùng chính: (1) tiếp nhận hồ sơ Sen bàn giao và đối chiếu trường trích xuất · (2) đối chiếu điều khoản chính sách Sen đề xuất · (3) thẩm định và quyết định duyệt/trả lại.

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

**Product:** M1 % ticket Approved – first pass (45% → 70%) · M2 % ticket bị khiếu nại lại trong 7 ngày sau duyệt (6% → giữ ≤6%, counter-metric chống duyệt ẩu) · M6 chi phí gán nhãn lại ca tra cứu sai mỗi tháng · M7 % hội thoại bị cổng chặn mà khách không quay lại trong 24 giờ (*hai chỉ số thêm ở v2, target chốt sau khi đo baseline*).
**Workflow:** M3 thời gian thẩm định trung vị/ticket (8 phút → 4 phút) · M4 % trường bắt buộc Sen điền đúng ngay lần đầu (62% → 90%) · M5 % ca gắn cờ được xử lý trong SLA 30 phút (mới → 95%).
Chi tiết baseline · target · nguồn dữ liệu · owner · hành động khi chỉ số xấu: xem [dashboard/dashboard_hanh_dong_v2.xlsx](dashboard/dashboard_hanh_dong_v2.xlsx) — cột *Thay đổi so với v1* trên từng sheet, nhật ký đầy đủ ở sheet *6. Thay doi v1 - v2*. Bản gốc: [v1/dashboard_hanh_dong_v1.xlsx](v1/dashboard_hanh_dong_v1.xlsx).
M1 phải đọc kèm **ba chỉ số chặn**: M2 (mua first-pass bằng chất lượng phục vụ), M6 (bằng công sức nhóm gán nhãn), M7 (bằng việc loại ca khó khỏi luồng trước khi thành ticket).

> **Baseline là số giả định** — Day23/Day24 là bài phân tích, chưa có log vận hành thật. Việc đo baseline thật là hạng mục đầu tiên của giai đoạn 0–30 ngày.

## 6. Quyết định

**SỬA** — không tiếp tục nguyên trạng, không dừng.
Lý do: nhu cầu và chỗ đứng của AI trong workflow là có thật, cái hỏng là kiến trúc tin cậy — nên sửa tầng kiểm chứng trước, chưa mở rộng phạm vi.
**Mười hai thay đổi so với v1** — CH1–CH8 từ rà soát chéo **nội bộ**, CH9–CH12 từ rà soát đối kháng có hỗ trợ AI; **cả hai đều không phải Chặng 3**, kiểm tra chéo với nhóm khác chưa diễn ra (chi tiết ở memo §3 và sheet *6. Thay doi v1 - v2*):
**CH1** sửa quy tắc đo M3 — loại ca >30 phút, ca mở lại chỉ tính lần cuối, vì log đo thời gian *trôi qua* chứ không đo công sức.
**CH2** thêm M6 — tầng Giá trị trước chỉ có counter-metric âm, không gì chứng minh Sen tiết kiệm *ròng*.
**CH3** tách quyền owner M4 — siết ngưỡng cổng chặn phải có đồng duyệt Trưởng nhóm CSKH, người quyết không được khác người chịu hậu quả.
**CH4** thêm kill criteria cho M5 — SLA hỏng thì abstention thành cái bẫy làm chậm khách, không được chuyển giai đoạn.
**CH5–CH8** (trục chẩn đoán): mở Readiness lên 4 nhánh · hạ Direction xuống ĐẠT CÓ ĐIỀU KIỆN · chỉ ra NN1–NN2 nối tiếp nên phiên bản hoá chính sách phải nằm ở 0–30 ngày · vòng học có nhịp và có cổng kiểm chất lượng nhãn.
**CH9** thêm M7 và kill criteria cho cổng chặn — hỏi lại khách đang mất tiền thì họ bỏ đi, ticket không bao giờ được tạo, M1 tăng giả.
**CH10** gỡ target phần trăm khỏi chỉ số có baseline "chưa đo" — chính lỗi đã bắt ở CH6, lần này áp cho mình.
**CH11** viết lại ba quy trình theo góc CSKH — v1 mô tả 2/3 quy trình bằng hành động của Sen, tức việc bên trong máy.
**CH12** tách quyền owner M5 — SLA hỏng vì định biên, thứ Trưởng ca không tự quyết được.

