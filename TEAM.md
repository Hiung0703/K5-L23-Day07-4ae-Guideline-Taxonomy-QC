# Thông tin nhóm

- Tên nhóm: Nhóm 101–104
- Mã thành viên (ba chữ số): **101, 102, 103, 104**
- Người điều phối / lead: **102**
- Người review bản cuối: **104**; **102** kiểm chéo gói bàn giao, mỗi ảnh có reviewer riêng khác người sửa.
- Commit chốt bài: **3cea38a** (gói nộp đã push lên `origin/main`).

## Kết quả tóm tắt hiện tại

Ngày cập nhật: **2026-11-17**. Nguồn: [IAA trước calibration](submission/qc/iaa_summary.json), [danh sách ca](submission/qc/calibration_candidates.csv), [điểm theo ảnh](submission/qc/image_scores.csv).

- Lượt gán hoàn tất / được giao: **80/80 (100%)** trên 40 ảnh; mỗi thành viên **20/20 ảnh**, mỗi ảnh có 2 người gán độc lập.
- IAA trước calibration: **276 cặp box ghép**, IoU trung bình **0,83617**; cùng lớp **262/276 = 94,93%**. **Chưa đạt** ngưỡng IoU ≥ 0,85 và đồng thuận lớp ≥ 95%; cả 6 cặp người gán chưa đạt đồng thời hai ngưỡng.
- Thuộc tính trên 276 cặp ghép: cùng `occluded_level` **156/276 = 56,52%**; cùng `truncated` **260/276 = 94,20%**.
- Box chưa ghép A/B: **35/147**; bỏ qua do ignore: **9**. A/B là hai phía của từng cặp người gán, chưa kết luận bên nào sai.
- Số ảnh phải calibration: **35/40 (87,5%)**, với **466 cảnh báo**: 136 bỏ sót/thừa, 128 sai thuộc tính, 110 IoU thấp, 74 bất đồng ignore, 14 sai lớp, 4 cờ `can_review`. Một đối tượng có thể có nhiều cảnh báo.
- Tiến độ quyết định calibration: **466/466 ca đã có đề xuất xử lý, quy tắc, căn cứ, người được giao chốt và reviewer** trong [bảng quyết định](submission/qc/calibration_decisions.csv); **0/466 ca có kết quả kiểm bản sửa được xác nhận**. Cột `nguoi_chot` ghi mã người được giao; `kiem_lai=CHO_KIEM...` ghi reviewer và việc phải kiểm, chưa phải kết quả đã kiểm. Trạng thái vẫn là `da_phan_cong_cho_chot_sua_kiem_lai` đến khi thực hiện. Giữ [ngữ cảnh phiên bản](submission/qc/calibration_context.json).
- Guideline v1.1 đã bổ sung `CAL-01`–`CAL-05`; [hồ sơ thay đổi](submission/guideline_change.json) là **semantic**, phạm vi rà lại **40/40 ảnh**, gồm cả 5 ảnh không có cờ.
- Vòng QC và số ảnh đã duyệt / cả lô: **Vòng 1 — full, đã duyệt 40/40 ảnh**; giữ [phiếu](submission/qc/reviews/round-01/qc_review.csv), [snapshot nhãn](submission/qc/reviews/round-01/labels_snapshot.json) và [kết quả tự động](submission/qc/reviews/round-01/qc_report_auto.md).
- Tỉ lệ lỗi theo ảnh: **35/40 = 87,5%**. Tỉ lệ lỗi theo đối tượng hiện có: **88/458 = 19,2%** (loại vùng ignore khỏi mẫu số); **3 vật bỏ sót** ghi riêng. Phiếu có 95 dòng lỗi và 5 dòng ảnh chưa phát hiện lỗi.
- **Quyết định bàn giao: Hold** — đã có nhãn cuối dạng hồ sơ và QC đủ lô, nhưng 35 ảnh còn lỗi, còn 4 box `can_review`, cần sửa và mở vòng mới. Bằng chứng và việc còn lại tại [QC_REPORT.md](submission/qc/QC_REPORT.md). Lead **102** điều phối; chưa xác nhận nghiệm thu bản nhãn.
- Kiểm tra gói nộp: đã chạy `python scripts/check_submission.py --round 1`, kết quả **0 lỗi cấu trúc/đối chiếu bằng chứng**; kết luận chất lượng vẫn là **Hold** theo vòng QC hiện tại.

## Phân công trách nhiệm bàn giao

Các mục dưới đây là **công việc được giao**, chưa phải xác nhận đã hoàn tất. Mỗi người đồng thời xử lý và kiểm chéo ảnh theo phân công ở phần tiếp theo.

| Thành viên | Trách nhiệm chính | Sản phẩm và điều kiện hoàn tất | Người kiểm chéo |
|---|---|---|---|
| **101** | Đặc tả và thông tin nhóm | Duy trì [TEAM.md](TEAM.md); tổng hợp quy tắc che khuất, biên box, ignore và lớp trong [guideline v1.1](submission/guidelines/guideline_v1.1_calibrated.md) cùng [nhật ký thay đổi](submission/guideline_change.json) từ các ca đã xem. | **103** kiểm tính nhất quán giữa guideline và quyết định. |
| **102** | Lead, taxonomy, công cụ và nhãn cuối | Điều phối nhóm, chủ trì họp chốt ca và kiểm tiến độ; đối chiếu [taxonomy v1.0](submission/taxonomy.json) với guideline, giữ nguyên `rules`; chỉ tạo taxonomy phiên bản mới nếu cần đổi schema. Quản lý COCO sửa trong CVAT theo đúng phiên bản; sau khi chốt ca, tổng hợp [nhãn cuối JSON](submission/qc/labels_calibrated.json), [nhãn cuối CSV](submission/qc/labels_calibrated.csv) và [manifest](submission/qc/batch_manifest.json) bằng công cụ của lab. | **104** kiểm phiên bản, số ảnh và khả năng truy xuất nhãn. |
| **103** | Hồ sơ calibration và QC | Giữ bộ IAA trước calibration; quản lý [466 ca cần chốt](submission/qc/calibration_decisions.csv) và ngữ cảnh phiên bản, thu quyết định và kết quả kiểm lại của từng ảnh. Chuẩn bị vòng QC, tổ chức duyệt đủ 40 ảnh theo reviewer được giao; lưu mọi [vòng review](submission/qc/reviews/) và tổng hợp [QC_REPORT.md](submission/qc/QC_REPORT.md) với tử số/mẫu số thực tế. | **101** kiểm số liệu, phạm vi và căn cứ đề xuất Hold / Rework / Release. |
| **104** | Review cuối và đóng gói | Đối chiếu đủ các nhóm tài liệu trong [SUBMISSION.md](SUBMISSION.md), giữ [ghi công ảnh](data/ATTRIBUTION.csv); kiểm hồ sơ cá nhân, bằng chứng sửa và vòng QC; chạy [kiểm tra gói nộp](scripts/check_submission.py) theo vòng cuối, ghi kết quả thật và tập hợp gói nộp riêng tư theo kênh của coach. | **102** kiểm chéo gói và các phần do 104 thực hiện. |

## Phân công ảnh cho calibration và QC

Đây là phân công **sau lượt gán độc lập**, không thay đổi [assignment.csv](submission/qc/assignment.csv) hay nhãn gốc. Chia theo [điểm bất đồng](submission/qc/image_scores.csv): mỗi người phụ trách **10 ảnh** và review **10 ảnh**; người phụ trách và reviewer đều khác hai người đã gán độc lập ảnh đó.

Trong danh sách, `007 → 104` nghĩa là người đứng đầu dòng phụ trách ảnh số 007, **104** kiểm lại bản sửa và duyệt QC ảnh đó. Số 007 tương ứng [img_007.jpg](data/images/img_007.jpg); các số khác theo cùng quy ước trong [thư mục ảnh](data/images/).

- **101:** 007 → 104; 010 → 103; 011 → 104; 017 → 103; 019 → 102; 022 → 103; 023 → 103; 034 → 102; 035 → 104; 040 → 102. **9 ảnh có cờ, 124 cảnh báo, tổng điểm 151.**
- **102:** 004 → 104; 006 → 103; 008 → 103; 009 → 104; 016 → 101; 025 → 104; 027 → 101; 030 → 104; 036 → 103; 038 → 101. **8 ảnh có cờ, 114 cảnh báo, tổng điểm 158.**
- **103:** 002 → 102; 003 → 101; 012 → 102; 013 → 101; 015 → 104; 020 → 104; 026 → 101; 029 → 101; 032 → 101; 037 → 104. **9 ảnh có cờ, 114 cảnh báo, tổng điểm 153.**
- **104:** 001 → 101; 005 → 101; 014 → 102; 018 → 102; 021 → 102; 024 → 102; 028 → 103; 031 → 102; 033 → 103; 039 → 103. **9 ảnh có cờ, 114 cảnh báo, tổng điểm 154.**

Người phụ trách xem ảnh gốc, đối chiếu guideline, đề xuất quyết định cho toàn bộ ca của ảnh và sửa khi cần; reviewer kiểm lại độc lập. Mỗi ca phải có quy tắc, phiên bản, người chốt, dấu vết sửa và kết quả kiểm lại. Bất đồng còn lại đưa ra họp nhóm do lead **102** điều phối, **103** ghi nhận. Chỉ điền đã sửa/đã kiểm lại khi thực sự hoàn thành.

Năm ảnh không có cờ là **017, 020, 027, 033, 038** vẫn phải duyệt toàn ảnh trong QC; không mặc định là ảnh sạch. Nếu đổi nghĩa quy tắc, cả **40 ảnh** phải được rà lại theo guideline mới, kể cả năm ảnh này.

## Việc cá nhân của cả bốn thành viên

Mỗi thành viên tự đối chiếu guideline cá nhân, COCO và ghi chú của mình; bổ sung trải nghiệm thực tế ở Pha 2–5, cách kiểm lại việc dùng AI và bài học. Bản cá nhân ghi lại quyết định ban đầu, giữ rõ khác biệt với quyết định nhóm sau calibration.

- **101:** kiểm [guideline cá nhân](submission/guidelines/individual/guideline_101.md), [COCO](submission/labels/101.json), hoàn thiện [ghi chú 101](submission/individual_notes/101.md); sửa mã nội dung và đường dẫn còn trỏ tới `001` cho khớp mã `101`.
- **102:** kiểm [guideline cá nhân](submission/guidelines/individual/guideline_102.md), [COCO](submission/labels/102.json), hoàn thiện [ghi chú 102](submission/individual_notes/102.md); thống nhất mã nội dung `002` thành `102` và loại thông tin định danh cá nhân khỏi bản nộp theo quy định của lab.
- **103:** kiểm [guideline cá nhân](submission/guidelines/individual/guideline_103.md), [COCO](submission/labels/103.json), hoàn thiện các mục Pha 2–5 và bài học trong [ghi chú 103](submission/individual_notes/103.md).
- **104:** kiểm [guideline cá nhân](submission/guidelines/individual/guideline_104.md), [COCO](submission/labels/104.json), hoàn thiện các mục còn trống và bằng chứng tự kiểm trong [ghi chú 104](submission/individual_notes/104.md).

## Thứ tự hoàn thành phần còn lại

1. **103 chuẩn bị, cả nhóm xem ca:** dùng [công cụ chồng nhãn](scripts/viz_disagreement.py) với `--images flagged`, xử lý các ảnh có điểm cao trước; dùng bảng quyết định đã có, giữ bộ IAA trước calibration.
2. **Cả bốn chốt ca, 102 điều phối, 101 tổng hợp đặc tả:** bảng đã có đề xuất và phân công cho 466 ca. Mỗi người đối chiếu các ca theo ảnh được giao, xác nhận hoặc chỉnh hướng xử lý; 103 tập hợp kết quả. Guideline v1.1 hiện khai báo thay đổi `semantic`, cần rà lại toàn bộ 40 ảnh. Reviewer thay nội dung `CHO_KIEM` bằng kết quả thực tế và cập nhật trạng thái khi đủ bằng chứng.
3. **102 tổ chức sửa và tổng hợp nhãn:** khi quyết định và guideline đủ, dùng [chuẩn bị calibration](scripts/prepare_calibration.py); người phụ trách sửa trong CVAT, reviewer kiểm lại. Lưu COCO đúng đường dẫn phiên bản công cụ in ra, rồi dùng [tạo nhãn cuối](scripts/build_final.py).
4. **103 mở vòng QC, cả bốn review:** dùng [lấy mẫu QC](scripts/qc_sample.py) với `--round 1` và mặc định duyệt toàn lô; mỗi reviewer duyệt 10 ảnh theo phân công, điền phiếu thật. Nhãn thay đổi sau QC thì mở vòng mới và giữ vòng cũ.
5. **103 lập báo cáo, 101 và 104 đối chiếu:** dùng [tính kết quả QC](scripts/qc_report.py), ghi tỷ lệ lỗi theo ảnh và đối tượng kèm mẫu số. Release cần đủ bằng chứng duyệt toàn lô; với 40 ảnh, 1 ảnh lỗi = 2,5% đã vượt ngưỡng 2%. Giữ Hold nếu thiếu bằng chứng; Rework nếu lỗi vượt ngưỡng hoặc lặp theo quy tắc lab.
6. **104 đóng gói, 102 kiểm chéo:** kiểm toàn bộ [checklist nộp bài](SUBMISSION.md), chạy kiểm tra theo vòng QC cuối; 101 cập nhật kết quả thực tế ở đầu trang, lead 102 chốt commit bàn giao. Nộp repo riêng tư hoặc gói nén qua kênh coach công bố.
