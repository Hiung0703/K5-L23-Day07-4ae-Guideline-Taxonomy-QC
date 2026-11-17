# Báo cáo QC — Nhóm 101–104

Ngày cập nhật: 2026-11-17. Lead: **102**. Hồ sơ bàn giao vòng 1 giữ trạng thái **Hold**.

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Thực hành đặc tả, gán bounding box 2D và QC trên 40 ảnh người/phương tiện đường phố. Bộ ảnh tuyển chọn để luyện ca biên, chưa đánh giá độ đại diện cho dữ liệu vận hành; chưa dùng bản nhãn này làm dữ liệu đã nghiệm thu.
- Nguồn và phép sử dụng dữ liệu: Giữ thông tin tác giả, giấy phép và URL từng ảnh trong [ATTRIBUTION.csv](../../data/ATTRIBUTION.csv); nộp riêng tư theo hướng dẫn của coach.
- Manifest và checksum nhãn cuối: [batch_manifest.json](batch_manifest.json), nhãn [labels_calibrated.json](labels_calibrated.json), SHA-256 `e26b1693f6b403b2338996840f8ef8bfff3cad1cdd95d97105b94c708d53c8d9`.
- Guideline và taxonomy khi gán / khi bàn giao: Gán độc lập theo [guideline v1.0](../guidelines/guideline_v1.0_agreed.md) và [taxonomy v1.0](../taxonomy.json). Bàn giao kèm [guideline v1.1](../guidelines/guideline_v1.1_calibrated.md), giữ schema và rules của taxonomy v1.0. [Thay đổi semantic](../guideline_change.json) yêu cầu rà lại cả 40 ảnh.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80/80 (100%); mỗi thành viên 20/20 ảnh, mỗi ảnh có 2 lượt độc lập. Lead 102; 101 phụ trách đặc tả, 102 công cụ/nhãn, 103 hồ sơ/QC, 104 kiểm bàn giao. Phân công xử lý và reviewer tại [TEAM.md](../../TEAM.md).
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 276 cặp box ghép, IoU trung bình 0,83617; cùng lớp 262/276 = 94,93%, chưa đạt ngưỡng IoU 0,85 và cùng lớp 95%. Chỉ tính các cặp ghép 1–1 tại IoU ít nhất 0,5, không phân biệt lớp khi ghép; giữ nguyên [IAA trước calibration](iaa_summary.json).
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: A/B = 35/147; bỏ qua do ignore = 9; có 74 cảnh báo bất đồng ignore. Đây là đơn vị cảnh báo/cặp so sánh, chưa phải số lỗi QC độc lập.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: Pickup trong ảnh 007 vẫn có box mang lớp car trong bản nhãn; cùng tên lớp giữa người gán không xác nhận đúng thực tế. Đối chiếu cabin/thùng hàng với định nghĩa truck và CAL-04, xem lỗi sai_lop tại ảnh 007 trong phiếu vòng 1.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: [calibration_decisions.csv](calibration_decisions.csv) có 466 hướng xử lý cùng mã quy tắc, căn cứ và người được giao chốt/kiểm lại. Các trường CHO_SUA/CHO_KIEM và trạng thái phân công chưa chứng minh ca đã được sửa/chốt. Không xác nhận đóng toàn bộ 466 ca; giữ [ngữ cảnh IAA](calibration_context.json) và [snapshot quyết định](reviews/round-01/calibration_decisions_snapshot.csv) của vòng 1.
- Tác động v1.1 lên nhãn và ảnh: CAL-01…CAL-05 làm rõ che khuất/cắt mép, phạm vi ignore, ghép lại cùng vật, tách người–xe và xử lý ca chưa đủ căn cứ. Phạm vi kiểm 40/40 ảnh, gồm 5 ảnh không có cờ; không thay số đo IAA hoặc sửa đầu vào độc lập.
- Tham chiếu bằng chứng khi Rework: Giữ [nhãn snapshot](reviews/round-01/labels_snapshot.json), [phiếu QC vòng 1](reviews/round-01/qc_review.csv) và [manifest vòng 1](reviews/round-01/review_manifest.json). Nếu sửa nhãn/quyết định, tạo lại nhãn cuối và mở vòng 2; không ghi đè bằng chứng vòng 1.

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: Các mã chịu trách nhiệm theo phân công trong phiếu là 101, 102, 103, 104; ngày 2026-11-17. Mỗi mã phụ trách review 10 ảnh.
- Đã duyệt / cả lô: 40/40 ảnh, vòng full; [báo cáo tự động](reviews/round-01/qc_report_auto.md) và [kết quả máy đọc](reviews/round-01/qc_result.json).
- Ảnh lỗi / ảnh đã duyệt: 35/40 = 87,5%; 5 ảnh chưa phát hiện lỗi trong vòng này là 017, 020, 027, 033, 038. Một ảnh có thể có nhiều dòng lỗi.
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 88/458 = 19,2%; mẫu số loại 8 vùng ignore. Một đối tượng có nhiều loại lỗi chỉ tính một lần ở tỷ lệ này.
- Vật bỏ sót, ảnh và vùng: 3 vật, gồm người trong khoang lái minibus ảnh 009 (khoảng x600–630, y395–430), người áo sáng ảnh 011 (x382,y334,w24,h63), và car màu cam góc dưới trái ảnh 024 (x0,y541,w184,h179). Xem các dòng bo_sot trong phiếu. Ngoài bộ đếm vật bỏ sót còn cần bổ sung vùng ignore cho cụm không tách được ở ảnh 021, khoảng [390,207,431,83]; không tính vùng ignore này thành một vật bỏ sót.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: ca_kho duyệt 35/35, lỗi 35/35 (100%); con_lai duyệt 5/5, lỗi 0/5. Phiếu có 100 dòng: 95 dòng lỗi và 5 dòng ảnh không phát hiện lỗi; gồm 67 sai thuộc tính, 9 sai biên/kích thước, 9 box thừa, 7 sai lớp, 3 bỏ sót. Lỗi thuộc tính lặp nhiều; ví dụ ảnh 035 object413 truncated=yes dù nằm trong khung, và xe có người lái nhưng occluded_level=0 trên nhiều ảnh.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Công cụ đã chặn thiếu COCO đúng phiên bản trước khi build nhãn. Lớp car trên pickup hoặc person trên thú bông vẫn là tên lớp hợp lệ nên cần đối chiếu ảnh; xem ảnh 007 và object212 ở ảnh 019.
- Box còn can_review: 4 box trong nhãn vòng 1: ảnh 001 object3/person; ảnh 024 object263/car; ảnh 035 object413/person; ảnh 037 object435/bus. Cần chốt theo ảnh và quy tắc, không tự bỏ cờ chỉ để đổi kết luận.

## 5. Quyết định bàn giao

- Quyết định: Hold
- Lý do, người nhận và phạm vi sử dụng: Báo cáo tự động còn ghi nhãn có can_review; tỷ lệ ảnh lỗi 87,5% vượt ngưỡng 2% và lỗi thuộc tính lặp nhiều. Nộp cho coach như hồ sơ thực hành có bằng chứng Hold, chưa nghiệm thu bản nhãn để huấn luyện/vận hành. Nếu xử lý hết phần còn mở mà lỗi QC vẫn còn, tiếp tục Rework theo quy tắc lab.
- Nếu còn việc: Lead102 điều phối sửa theo 95 dòng lỗi; 101 cập nhật diễn giải quy tắc khi cần, 102 tổng hợp COCO đúng phiên bản, 103 kiểm đủ ca và mở vòng QC mới, 104 kiểm gói bàn giao. Từng ảnh giữ người phụ trách/reviewer đã chia trong TEAM. Sửa nhãn, kiểm lại 3 vật bỏ sót, thu vùng ignore và xử lý 4 can_review; ghi kết quả thực tế rồi chạy build_final, qc_sample với vòng mới, qc_report và check_submission.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Trong [phiếu vòng 1](reviews/round-01/qc_review.csv), tìm img_035.jpg/object413/sai_thuoc_tinh; đối chiếu R-02/CAL-02 với [ảnh gốc](../../data/images/img_035.jpg) và bbox [413,227,52,49] của object413 trong [snapshot](reviews/round-01/labels_snapshot.json). Box không chạm mép ảnh nhưng truncated=yes, là căn cứ có thể tái kiểm. Hash nhãn vòng này khớp manifest ở mục 1.
