# Memo quyết định — Sen (FIN-05)

**Người viết:** Đào Ngọc Bích · 2A202601745
**Phạm vi:** Sen — AI Agent CSKH ví điện tử · người dùng chính: CSKH tuyến 1 · 3 quy trình: trích xuất/tạo ticket, tra cứu chính sách hoàn tiền, thẩm định & duyệt lần đầu.

## 1. Vấn đề và nguyên nhân gốc

Triệu chứng quan sát được: Sen đã chạy trong luồng CSKH, nhưng nhân viên vẫn dò lại lịch sử chat và xác minh thủ công từng trường — phần lớn thời gian mỗi ca không dành cho việc ra quyết định mà cho việc kiểm lại thứ AI đã làm. Đây là triệu chứng, không phải nguyên nhân.

**Nguyên nhân gốc 1 — thiếu tầng kiểm chứng giữa AI và người duyệt.** Sen bàn giao hồ sơ không kèm trích dẫn nguồn và không kèm mức tin cậy, nên CSKH không có cách nào phân biệt trường AI chắc chắn với trường AI suy đoán. Khi không phân biệt được, cách an toàn duy nhất là kiểm lại toàn bộ — và đúng lúc đó lợi ích tiết kiệm công của AI biến mất.

**Nguyên nhân gốc 2 — không có owner và không có vòng học cho tập chính sách hoàn tiền.** Chính sách khác nhau giữa các ví và thay đổi theo thời gian, nhưng không ai chịu trách nhiệm cập nhật tập dữ liệu Sen tra cứu, cũng không có đường quay ngược từ ca lỗi về dữ liệu. Hệ quả là cùng một loại sai lặp lại và chỉ được phát hiện sau khi khách đã khiếu nại.

Cả hai đều **không phải vấn đề đào tạo**: người dùng buộc phải dùng Sen (ticket được đẩy tới) và có động lực giảm tải, nên nghẽn không nằm ở nhận biết hay mong muốn.

## 2. Framework đã dùng và bằng chứng

**Kiến trúc tin cậy** (nguồn → trích nguồn → QA mẫu → chuyển người → phản hồi): hiện chỉ có mắt xích *chuyển người*; khuyết trích nguồn, QA mẫu và phản hồi — đây là căn cứ trực tiếp cho NN1.

**Gartner-Lite:** *Direction* ĐẠT (mục tiêu giảm tải CSKH rõ ràng, có định nghĩa NSM từ Day23). *Readiness* THIẾU ở nhánh governance — không có data owner cho tập chính sách hoàn tiền. *Absorption* THIẾU — không có cơ chế học từ lỗi, các ca trả lại và khiếu nại không quay ngược thành dữ liệu cải thiện. Kết luận: **chưa đủ điều kiện rollout rộng, phải sửa readiness trước.**

**ADKAR:** Awareness và Desire ĐẠT; **Ability NGHẼN** — thiếu công cụ và checklist để thẩm định nhanh output AI; Reinforcement RỦI RO — nếu chỉ chạy theo chỉ tiêu first-pass mà không có counter-metric, nhân viên có thể duyệt ẩu.

**Mollick — phân chia việc:** Sen đang bị đặt nhầm ở vùng *AI tự động* (dựng hồ sơ hoàn chỉnh cho ca rủi ro tài chính cao). Phải kéo về vùng *AI hỗ trợ – người kiểm*; chỉ giữ ở vùng tự động những tác vụ rủi ro thấp, có tiêu chí kiểm rõ (hỏi lại khách để thu trường còn thiếu, phân loại sơ bộ loại sự cố).

**Bằng chứng cụ thể:** (a) event `ticket_required_fields_snapshot` trong thiết kế tracking Day23 ghi nhận trường bắt buộc còn thiếu nhưng không có cơ chế nào dùng tín hiệu đó để chặn hay cảnh báo; (b) Day24 liệt kê chi phí ẩn định kỳ *"gán nhãn lại các ca agent tra cứu sai để cải thiện policy retrieval"* — tần suất sai đủ lớn để thành khoản chi thường xuyên; (c) hai đường phát hiện lỗi duy nhất hiện có (`ticket_returned_for_info`, `ticket_reopened_by_customer_complaint`) đều nằm **sau** thời điểm đã phát sinh hậu quả.

## 3. Ít nhất 2 thay đổi sau phản biện

Nhận phản biện từ nhóm ______ theo bốn trục. Bốn thay đổi đã thực hiện, đánh dấu tương ứng ở cột "Thay đổi so với v1" trong dashboard v2 và ghi đầy đủ ở sheet *6. Thay doi v1 - v2*.

1. **CH1 — sửa quy tắc đo M3 (trục Chỉ số).** Góp ý: khoảng cách giữa `ticket_opened_by_agent` và event quyết định đo thời gian *trôi qua*, không đo công sức — CSKH mở ticket rồi bỏ đó làm việc khác. Sửa: loại khỏi mẫu ca >30 phút và báo cáo riêng tỷ lệ bị loại; ca mở lại nhiều lần chỉ tính lần mở cuối.
2. **CH2 — thêm chỉ số M6 (trục Chỉ số).** Góp ý: tầng 5 Giá trị chỉ có M2, một counter-metric âm; không có gì chứng minh Sen tạo tiết kiệm ròng — first-pass vẫn có thể tăng trong khi chi phí chỉ chuyển từ CSKH sang nhóm gán nhãn. Sửa: thêm M6 *chi phí gán nhãn lại ca tra cứu sai mỗi tháng*, product-level, owner Policy Owner, target giảm ≥40% so với baseline tháng đầu.
3. **CH3 — tách quyền owner M4 (trục Hành động).** Góp ý: hành động khi M4 xấu là siết ngưỡng cổng chặn, việc này làm khách bị hỏi lại nhiều hơn — người quyết (Kỹ sư AI) không phải người chịu hậu quả. Sửa: mọi thay đổi ngưỡng cổng chặn phải có đồng duyệt của Trưởng nhóm CSKH.
4. **CH4 — thêm kill criteria cho M5 (trục Hành động).** Góp ý: M5 có target nhưng không nằm trong điều kiện dừng nào; SLA hỏng thì abstention thành cái bẫy làm chậm khách mà lộ trình vẫn cho chuyển giai đoạn. Sửa: hết 30–60 ngày mà SLA chưa đạt 95% ⇒ không mở rộng, hoặc bố trí người trực hàng đợi cờ đỏ, hoặc tắt abstention và quay lại chuyển người 100% cho nhóm ca này.

## 4. Quyết định: tiếp tục / sửa / dừng

**SỬA.**

Không *tiếp tục* nguyên trạng vì mở rộng một hệ chưa có tầng kiểm chứng sẽ nhân rộng cả lỗi lẫn chi phí gán nhãn thủ công. Không *dừng* vì nhu cầu là thật, chỗ đứng của AI trong workflow là hợp lý, và nguyên nhân nằm ở thiết kế tin cậy chứ không ở giá trị bài toán.

## 5. Lý do và bước tiếp theo và owner

Sửa kiến trúc tin cậy trước khi mở rộng phạm vi — vì mọi chỉ số adoption đều vô nghĩa nếu người dùng vẫn phải kiểm lại 100% output.

| Giai đoạn | Bước tiếp theo | Dấu hiệu hoàn thành | Owner |
|---|---|---|---|
| 0–30 ngày | Đo baseline thật cho 6 chỉ số; gán nhãn 100 ca trả lại gần nhất theo loại lỗi; chỉ định Policy Owner | Có bảng baseline bằng số thật + phân bố loại lỗi; Policy Owner có tên trên văn bản | Đào Ngọc Bích (PO) · Trưởng nhóm CSKH |
| 30–60 ngày | Bật trích dẫn nguồn + mức tin cậy từng trường; bật cổng chặn ticket khuyết trường trọng yếu; áp checklist thẩm định 3 điểm cho 1 ca pilot | ≥90% ticket mới đủ trường bắt buộc; first-pass nhóm pilot tăng ≥10 điểm; QA mẫu 50 ticket/tuần có trích dẫn khớp ≥95% | PO · Kỹ sư AI · Trưởng ca pilot |
| 60–90 ngày | Mở rộng toàn bộ ca CSKH; chạy vòng phản hồi hằng tuần; đo giá trị nghiệp vụ | First-pass đạt target 70%; khiếu nại lại không tăng quá 6%; ≥4 chu kỳ cập nhật chính sách do Policy Owner thực hiện | PO · Policy Owner · Giám đốc Vận hành |

**Gate chuyển giai đoạn:** chỉ sang 30–60 khi đã có baseline bằng số thật và Policy Owner được chỉ định; chỉ sang 60–90 khi pilot đạt ngưỡng chất lượng ở trên. Nếu tới mốc 90 ngày mà first-pass không cải thiện dù chất lượng trích xuất đã đạt 90%, kết luận nghẽn không nằm ở AI — chuyển hướng sang quy trình/nhân sự thay vì đầu tư thêm vào mô hình.
