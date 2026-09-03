# Memo quyết định — Sen (FIN-05)

**Nhóm:** Track1badao — Đào Ngọc Bích (2A202601745) · Đặng Thái Nam Sơn (2A202601431) · Lương Thanh Trang (2A202601363)
**Phạm vi:** Sen — AI Agent CSKH ví điện tử · người dùng chính: CSKH tuyến 1 · 3 quy trình (viết theo góc người dùng chính): tiếp nhận hồ sơ Sen bàn giao và đối chiếu trường trích xuất · đối chiếu điều khoản chính sách Sen đề xuất · thẩm định và quyết định duyệt/trả lại.

## 1. Vấn đề và nguyên nhân gốc

Triệu chứng quan sát được: Sen đã chạy trong luồng CSKH, nhưng nhân viên vẫn dò lại lịch sử chat và xác minh thủ công từng trường — phần lớn thời gian mỗi ca không dành cho việc ra quyết định mà cho việc kiểm lại thứ AI đã làm. Đây là triệu chứng, không phải nguyên nhân.

**Nguyên nhân gốc 1 — thiếu tầng kiểm chứng giữa AI và người duyệt.** Sen bàn giao hồ sơ không kèm trích dẫn nguồn và không kèm mức tin cậy, nên CSKH không có cách nào phân biệt trường AI chắc chắn với trường AI suy đoán. Khi không phân biệt được, cách an toàn duy nhất là kiểm lại toàn bộ — và đúng lúc đó lợi ích tiết kiệm công của AI biến mất.

**Nguyên nhân gốc 2 — không có owner và không có vòng học cho tập chính sách hoàn tiền.** Chính sách khác nhau giữa các ví và thay đổi theo thời gian, nhưng không ai chịu trách nhiệm cập nhật tập dữ liệu Sen tra cứu, cũng không có đường quay ngược từ ca lỗi về dữ liệu. Hệ quả là cùng một loại sai lặp lại và chỉ được phát hiện sau khi khách đã khiếu nại.

Cả hai đều **không phải vấn đề đào tạo**: người dùng buộc phải dùng Sen (ticket được đẩy tới) và có động lực giảm tải, nên nghẽn không nằm ở nhận biết hay mong muốn.

## 2. Framework đã dùng và bằng chứng

**Kiến trúc tin cậy** (nguồn → trích nguồn → QA mẫu → chuyển người → phản hồi): hiện chỉ có mắt xích *chuyển người*; khuyết trích nguồn, QA mẫu và phản hồi — đây là căn cứ trực tiếp cho NN1.

**Gartner-Lite:**

*Direction* — **ĐẠT CÓ ĐIỀU KIỆN.** Hướng rõ về định tính: mục tiêu giảm thời gian CSKH dành cho xác minh thủ công, NSM là % ticket Approved first-pass, có định nghĩa từ Day23. Nhưng cả 7 chỉ số đang chạy trên **baseline giả định** — Day23/Day24 là bài phân tích thiết kế, chưa có log vận hành thật. Một Direction có đích mà chưa có điểm xuất phát đo được thì chưa kiểm chứng được: target 70% đang được suy ra từ con số 45% không ai đo. Thiếu thêm vế giá trị — Day24 đã liệt kê chi phí ẩn định kỳ (gán nhãn lại, retrain theo drift chính sách) nhưng chưa có phía đối ứng quy đổi lợi ích của việc rút thời gian thẩm định.

*Readiness* — **THIẾU ở 3/4 nhánh**, không chỉ ở governance:

| Nhánh | Kết quả | Nhận định |
|---|---|---|
| Dữ liệu | THIẾU | Tập chính sách hoàn tiền không có trường phiên bản / ngày hiệu lực; mỗi ví một khác và đổi liên tục. Chỉ định owner **không** tự sinh ra các trường này. |
| Governance | THIẾU | Không ai chịu trách nhiệm tính đúng-đủ-mới của tập dữ liệu Sen tra cứu. |
| Quyền | ĐẠT | Sen tuyệt đối không tự duyệt hoàn tiền — đúng, và là ràng buộc phải giữ nguyên khi mở rộng ở giai đoạn 60–90. |
| Năng lực vận hành | THIẾU | QA mẫu 50 ticket/tuần và hàng đợi ưu tiên SLA 30 phút cho ca gắn cờ là **công việc mới**, chưa giao cho ai và chưa tính vào định biên. |

*Absorption* — **THIẾU.** Ca trả lại và ca khiếu nại không quay ngược thành dữ liệu cải thiện. Đáy của vấn đề nằm sâu hơn "chưa ai làm": dữ liệu học phải do chính CSKH sinh ra bằng thao tác phân loại lý do trả lại, đúng lúc họ đang chịu tải cao nhất — nếu không ràng buộc, cột lý do sẽ dồn vào giá trị *"khác"* và vòng học nhận về nhiễu. Chốt kiểm bắt buộc: **tỷ lệ nhãn "khác" > 15% ⇒ coi như nhãn hỏng, không dùng để cải thiện.** Ngoài ra vòng phản hồi chưa có nhịp — chưa có buổi họp, hiện vật và người chốt được đặt tên; một vòng lặp không có cuộc họp thì không phải vòng lặp.

**Kết luận Gartner-Lite:** chưa đủ điều kiện rollout rộng, phải sửa readiness trước. Và hệ quả quan trọng hơn lên thứ tự làm việc — **NN1 và NN2 không song song, chúng nối tiếp một nửa.** TO-BE yêu cầu mỗi đề xuất hoàn tiền kèm *tên văn bản · phiên bản · ngày hiệu lực*; không thể trích dẫn phiên bản của một tập dữ liệu chưa có trường phiên bản. Nửa NN1 thuộc về hội thoại (trích dẫn đoạn chat + mức tin cậy từng trường) là độc lập, kỹ sư AI làm được ngay; nửa thuộc về chính sách **bị chặn bởi readiness–dữ liệu của NN2**. Vì vậy phiên bản hoá tập chính sách là hạng mục bắt buộc của giai đoạn 0–30, không phải việc làm kèm.

**ADKAR:** Awareness và Desire ĐẠT; **Ability NGHẼN** — thiếu công cụ và checklist để thẩm định nhanh output AI; Reinforcement RỦI RO — nếu chỉ chạy theo chỉ tiêu first-pass mà không có counter-metric, nhân viên có thể duyệt ẩu.

**Mollick — phân chia việc:** Sen đang bị đặt nhầm ở vùng *AI tự động* (dựng hồ sơ hoàn chỉnh cho ca rủi ro tài chính cao). Phải kéo về vùng *AI hỗ trợ – người kiểm*; chỉ giữ ở vùng tự động những tác vụ rủi ro thấp, có tiêu chí kiểm rõ (hỏi lại khách để thu trường còn thiếu, phân loại sơ bộ loại sự cố).

**Bằng chứng cụ thể:** (a) event `ticket_required_fields_snapshot` trong thiết kế tracking Day23 ghi nhận trường bắt buộc còn thiếu nhưng không có cơ chế nào dùng tín hiệu đó để chặn hay cảnh báo; (b) Day24 liệt kê chi phí ẩn định kỳ *"gán nhãn lại các ca agent tra cứu sai để cải thiện policy retrieval"* — tần suất sai đủ lớn để thành khoản chi thường xuyên; (c) hai đường phát hiện lỗi duy nhất hiện có (`ticket_returned_for_info`, `ticket_reopened_by_customer_complaint`) đều nằm **sau** thời điểm đã phát sinh hậu quả.

## 3. Ít nhất 2 thay đổi sau phản biện

> **Nguồn của các thay đổi dưới đây là rà soát chéo NỘI BỘ trong nhóm, không phải kiểm tra chéo Chặng 3 với nhóm khác.** Chặng 3 tại thời điểm viết chưa diễn ra. Khi có, các thay đổi phát sinh từ đó sẽ được đánh số từ **CH13** ở sheet *6. Thay doi v1 - v2*, ghi rõ tên nhóm phản biện vào cột "Người nêu".

Bản v1 do Đào Ngọc Bích dựng được hai thành viên còn lại rà soát độc lập, mỗi người theo một trục khác nhau — Đặng Thái Nam Sơn soi cấu trúc lập luận framework (CH5–CH8), Lương Thanh Trang soi tính đo được của chỉ số (CH1–CH4). Sau khi gộp, bản hợp nhất được rà soát đối kháng một lượt nữa **có hỗ trợ AI** (CH9–CH12) — cũng không phải Chặng 3. Mười hai thay đổi đã thực hiện, đánh dấu ở cột "Thay đổi so với v1" trên từng sheet và ghi đầy đủ ở sheet *6. Thay doi v1 - v2*. Bốn thay đổi thuộc trục chỉ số nêu dưới đây:

1. **CH1 — sửa quy tắc đo M3 (trục Chỉ số).** Góp ý: khoảng cách giữa `ticket_opened_by_agent` và event quyết định đo thời gian *trôi qua*, không đo công sức — CSKH mở ticket rồi bỏ đó làm việc khác. Sửa: loại khỏi mẫu ca >30 phút và báo cáo riêng tỷ lệ bị loại; ca mở lại nhiều lần chỉ tính lần mở cuối.
2. **CH2 — thêm chỉ số M6 (trục Chỉ số).** Góp ý: tầng 5 Giá trị chỉ có M2, một counter-metric âm; không có gì chứng minh Sen tạo tiết kiệm ròng — first-pass vẫn có thể tăng trong khi chi phí chỉ chuyển từ CSKH sang nhóm gán nhãn. Sửa: thêm M6 *chi phí gán nhãn lại ca tra cứu sai mỗi tháng*, product-level, owner Policy Owner. *(Target ban đầu đặt là "giảm ≥40%" nhưng đã bị gỡ ở CH10 — không đặt được mức giảm cụ thể trên baseline chưa đo.)* Đây đúng là lỗ hổng đã tự nêu khi hạ *Direction* xuống ĐẠT CÓ ĐIỀU KIỆN — thiếu vế giá trị đặt cạnh chi phí ẩn Day24 — nên M6 khép lại chẩn đoán đó chứ không phải một chỉ số gắn thêm cho đủ.
3. **CH3 — tách quyền owner M4 (trục Hành động).** Góp ý: hành động khi M4 xấu là siết ngưỡng cổng chặn, việc này làm khách bị hỏi lại nhiều hơn — người có quyền quyết (Kỹ sư AI) không phải người chịu hậu quả. Sửa: mọi thay đổi ngưỡng cổng chặn phải có đồng duyệt của Trưởng nhóm CSKH.
4. **CH4 — thêm kill criteria cho M5 (trục Hành động).** Góp ý: M5 có target nhưng không nằm trong điều kiện dừng nào; SLA hỏng thì abstention thành cái bẫy làm chậm khách, mà lộ trình vẫn cho chuyển giai đoạn. Sửa: hết 30–60 ngày mà SLA chưa đạt 95% ⇒ không mở rộng, hoặc bố trí người trực hàng đợi cờ đỏ, hoặc tắt abstention và quay lại chuyển người 100% cho nhóm ca này.

Bốn thay đổi còn lại thuộc trục chẩn đoán, do Đặng Thái Nam Sơn nêu:

5. **CH5 — mở Readiness từ 1 lên 4 nhánh.** v1 chỉ soi governance. Bổ sung nhánh *năng lực vận hành*: QA mẫu 50 ticket/tuần và hàng đợi SLA 30 phút là công việc mới, chưa giao ai, chưa tính định biên — readiness không chỉ là dữ liệu, còn là người để vận hành thứ mình sắp bật.
6. **CH6 — hạ Direction từ ĐẠT xuống ĐẠT CÓ ĐIỀU KIỆN.** Hướng rõ nhưng chưa có điểm xuất phát đo được: target 70% đang suy ra từ baseline 45% không ai đo. Thiếu thêm vế giá trị đặt cạnh chi phí ẩn Day24 — chính lỗ hổng mà CH2 sau đó lấp bằng M6.
7. **CH7 — NN1 và NN2 nối tiếp, không song song.** Không thể trích dẫn *phiên bản* của một tập dữ liệu chưa có trường phiên bản ⇒ nửa NN1 thuộc tra cứu chính sách bị chặn bởi readiness–dữ liệu của NN2. Hệ quả: phiên bản hoá tập chính sách chuyển thành hạng mục bắt buộc của 0–30 ngày, không phải việc làm kèm.
8. **CH8 — vòng học phải có nhịp và có cổng kiểm chất lượng.** Bốn nhãn lý do trả lại là lựa chọn bắt buộc; họp rà 30 phút mỗi tuần, hiện vật là bảng phân bố loại lỗi, Policy Owner chốt. Thêm kill criteria: nhãn "khác" >15% ⇒ nhãn hỏng, dừng dùng để cải thiện.

Bốn thay đổi cuối từ rà soát đối kháng trên bản hợp nhất:

9. **CH9 — thêm M7, counter-metric khách bỏ cuộc (trục Hành động).** Cổng chặn bắt Sen hỏi lại khách trước khi tạo ticket. Với khách đang mất tiền, bị hỏi vòng vo thì họ bỏ sang hotline hoặc bỏ hẳn — ticket không bao giờ được tạo. Số ticket giảm, ca còn lại toàn ca đủ thông tin nên M1 tăng đẹp, M2 cũng không tăng vì khách bỏ đi im lặng: dashboard xanh trong khi khách bị bỏ rơi. Sửa: thêm M7 *% hội thoại bị cổng chặn mà khách không quay lại hoàn tất trong 24 giờ*, kèm kill criteria M7 >15% ⇒ tắt cổng chặn.
10. **CH10 — gỡ target treo trên baseline chưa đo (trục Chỉ số).** M6 đặt "giảm ≥40%" trong khi baseline ghi "chưa đo" — đúng lỗi vừa bắt ở CH6. Sửa: M5, M6, M7 chỉ chốt target sau giai đoạn 0–30 ngày.
11. **CH11 — viết lại ba quy trình theo góc người dùng chính (trục Phạm vi).** Persona khoá là CSKH nhưng quy trình 1 và 2 đang mô tả hành động của Sen, tức việc bên trong máy; chấm nghiêm thì phạm vi chỉ có một quy trình của người dùng chính. Sửa: cả ba quy trình đổi chủ ngữ sang CSKH, phần việc của Sen chuyển xuống làm đầu vào.
12. **CH12 — tách quyền owner M5 (trục Hành động).** SLA 30 phút hỏng vì định biên người trực hàng đợi cờ đỏ, thứ Trưởng ca không tự quyết — chính nhánh readiness "năng lực vận hành" đã chấm THIẾU ở CH5. Sửa: Trưởng ca vận hành và báo cáo thiếu hụt, Trưởng nhóm CSKH quyết định định biên.

**Hạn chế đã biết, không sửa được bằng viết lại:** toàn bộ bằng chứng trong bài trích từ tài liệu thiết kế Day23/Day24 của chính nhóm — chưa phỏng vấn CSKH thật, chưa xem log thật, chưa có quan sát sơ cấp nào. Chẩn đoán vì vậy vẫn là suy luận trên tài liệu. Đây là lý do hạng mục số một của giai đoạn 0–30 ngày là đi đo và đi quan sát, không phải đi xây tính năng.

*Nguyên tắc giữ khi rà soát: chỉ M6 được lên dashboard vì nó trả lời được "dẫn tới quyết định gì"; chốt kiểm chất lượng nhãn ("khác" ≤15%) không vượt được bài kiểm đó nên đặt ở lộ trình và kill criteria. Cùng một nguyên tắc, hai kết cục khác nhau — không phải hai quan điểm trái nhau.*

## 4. Quyết định: tiếp tục / sửa / dừng

**SỬA.**

Không *tiếp tục* nguyên trạng vì mở rộng một hệ chưa có tầng kiểm chứng sẽ nhân rộng cả lỗi lẫn chi phí gán nhãn thủ công. Không *dừng* vì nhu cầu là thật, chỗ đứng của AI trong workflow là hợp lý, và nguyên nhân nằm ở thiết kế tin cậy chứ không ở giá trị bài toán.

## 5. Lý do và bước tiếp theo và owner

Sửa kiến trúc tin cậy trước khi mở rộng phạm vi — vì mọi chỉ số adoption đều vô nghĩa nếu người dùng vẫn phải kiểm lại 100% output.

| Giai đoạn | Bước tiếp theo | Dấu hiệu hoàn thành | Owner |
|---|---|---|---|
| 0–30 ngày | Đo baseline thật cho 7 chỉ số; gán nhãn 100 ca trả lại gần nhất theo loại lỗi; chỉ định Policy Owner; **dựng bảng phiên bản + ngày hiệu lực cho tập chính sách** | Có bảng baseline bằng số thật + phân bố loại lỗi; Policy Owner có tên trên văn bản; **100% văn bản chính sách có phiên bản + ngày hiệu lực** | Đào Ngọc Bích (PO) · Trưởng nhóm CSKH · Policy Owner |
| 30–60 ngày | Bật trích dẫn nguồn + mức tin cậy từng trường; bật cổng chặn ticket khuyết trường trọng yếu; áp checklist thẩm định 3 điểm cho 1 ca pilot; **khởi động vòng phản hồi trên nhóm pilot — phân loại lý do trả lại theo 4 nhãn, họp rà hằng tuần** | ≥90% ticket mới đủ trường bắt buộc; first-pass nhóm pilot tăng ≥10 điểm; QA mẫu 50 ticket/tuần có trích dẫn khớp ≥95%; **tỷ lệ nhãn "khác" ≤15%** | PO · Kỹ sư AI (QA mẫu) · Trưởng ca pilot |
| 60–90 ngày | Mở rộng toàn bộ ca CSKH; duy trì vòng phản hồi hằng tuần; đo giá trị nghiệp vụ | First-pass đạt target 70%; khiếu nại lại không tăng quá 6%; ≥4 chu kỳ cập nhật chính sách do Policy Owner thực hiện **và phân bố loại lỗi dịch chuyển được so với đường cơ sở 100 ca — chứng minh vòng học có tác dụng, không chỉ có chạy** | PO · Policy Owner · Giám đốc Vận hành |

**Gate chuyển giai đoạn:** chỉ sang 30–60 khi đã có baseline bằng số thật, Policy Owner được chỉ định **và tập chính sách đã có phiên bản + ngày hiệu lực** (không có ba thứ này thì tính năng trích dẫn ở giai đoạn sau không có gì để trích); chỉ sang 60–90 khi pilot đạt ngưỡng chất lượng ở trên. **Nếu quá 30 ngày mà Policy Owner vẫn chưa được chỉ định: không chuyển giai đoạn, kết luận tổ chức chưa sẵn sàng ở tầng governance — NN2 không sửa được bằng kỹ thuật — và đưa vấn đề lên cấp quyết định.** Nếu tới mốc 90 ngày mà first-pass không cải thiện dù chất lượng trích xuất đã đạt 90%, kết luận nghẽn không nằm ở AI — chuyển hướng sang quy trình/nhân sự thay vì đầu tư thêm vào mô hình.
