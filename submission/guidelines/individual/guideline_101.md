# Guideline gán nhãn — v1.0 — 101

- **Phiên bản:** v1.0 (bản cá nhân đề xuất)
- **Ngày:** 2026-11-16
- **Người viết:** 101
- **Phạm vi dữ liệu:** 40 ảnh đường phố `img_001.jpg`–`img_040.jpg`
- **Công cụ / định dạng:** CVAT, Bounding Box 2D, COCO 1.0

> Đây là guideline cá nhân để khảo sát và thống nhất. Các quyết định ở ca khó cần được nhóm/reviewer xác nhận trước khi khóa taxonomy và gán độc lập.

## 1. Mục tiêu

Gán nhãn người và phương tiện giao thông xuất hiện thực tế trong ảnh đường phố nhằm huấn luyện mô hình phát hiện đối tượng 2D. Nhãn phải phản ánh vật thể thật, lớp chính xác và box ôm sát phần nhìn thấy. Bỏ sót người/phương tiện hoặc gán nhãn cho tượng, hình in, đồ chơi, phản chiếu có thể làm nhiễu dữ liệu huấn luyện.

## 2. Lớp và thuộc tính
### Lớp
| Lớp | Định nghĩa | Ví dụ | Phản ví dụ / dễ nhầm |
|---|---|---|---|
| `car` | Ô tô con có cấu tạo và kích thước thông thường để chở người; gồm sedan, hatchback, SUV và ô tô con được vận chuyển trên xe chở. | Ô tô con trên đường hoặc trên sàn xe chở ô tô (`img_002.jpg`). | Xe buýt, xe tải, xe van; tượng ô tô, ô tô đồ chơi (`img_027.jpg`, `img_038.jpg`). |
| `truck` | Xe tải dùng để chở hàng, có cabin và thân/sàn chở hàng; gồm xe tải chở các phương tiện khác. | Xe tải chở ô tô (`img_002.jpg`). | Ô tô con; xe buýt; van nhỏ. |
| `van` | Xe chở người/hàng cỡ nhỏ dạng thân kín, có khoang thân liền với cabin, không phải xe tải hàng cỡ lớn. | Van thân kín. | Bán tải có nắp thùng cao hoặc xe tải nhỏ: nếu không phân biệt chắc chắn, đánh `can_review`. |
| `bus` | Phương tiện cơ giới cỡ lớn được thiết kế chủ yếu để chở nhiều hành khách. | Xe buýt (`img_009.jpg`, `img_031.jpg`, `img_037.jpg`, `img_040.jpg`). | Truck chở hàng. |
| `tuk_tuk` | Xe cơ giới ba bánh kiểu tuk-tuk/auto-rickshaw, dùng chở người hoặc hàng. | Tuk-tuk (`img_004.jpg`, `img_016.jpg`, `img_035.jpg`). | Motorcycle hai bánh; xích lô/xe đạp không động cơ. |
| `motorcycle` | Xe máy/mô tô cơ giới hai bánh, bao gồm người lái và xe được gán thành hai đối tượng riêng. | Xe máy trên đường. | Tuk-tuk ba bánh; xe đạp; xích lô. |
| `person` | Người thật đang hiện diện trong cảnh, dù đứng, đi bộ, ngồi hoặc lái xe. | Người đi bộ/người lái xe máy. | Tượng người (`img_011.jpg`, `img_017.jpg`), ma-nơ-canh (`img_033.jpg`), người trên hình in/quảng cáo (`img_018.jpg`). |
| `ignore_region` | Vùng chứa cụm đối tượng quá nhỏ/dày đặc hoặc vùng không thể gán riêng theo quy tắc; không phải lớp đối tượng thông thường. | Cụm vật nhỏ không thể tách box phù hợp. | Không dùng để thay thế việc gán từng đối tượng có thể nhận diện rõ. |

> **Lưu ý đồng bộ:** `cvat/labels_generated.json` trong gói hiện tại thể hiện `car`, `truck`, `van`, `motorcycle`, `person`, `ignore_region`; tài liệu lab mô tả taxonomy mở rộng thêm `bus`, `tuk_tuk`. Trước khi gán, reviewer/nhóm cần xác nhận taxonomy và bộ nhãn CVAT thống nhất. Không gán lớp chưa được cấu hình trong CVAT.

### 2.2 Thuộc tính

#### `occluded_level` (select: `0`, `1`, `2`, `3`)

Tỷ lệ phần diện tích vật thể bị che bởi vật khác, ước lượng trên toàn bộ hình dạng vật thể dự kiến.

| Giá trị | Ngưỡng | Ý nghĩa |
|---|---:|---|
| `0` | 0% bị che | Vật nhìn thấy đầy đủ, không bị vật khác che. |
| `1` | >0% đến 30% | Bị che nhẹ; phần lớn hình dạng còn nhìn thấy. |
| `2` | >30% đến 70% | Bị che đáng kể; vẫn nhận dạng chắc chắn được lớp. |
| `3` | >70% | Bị che rất nhiều nhưng vẫn đủ bằng chứng để xác định lớp. |

Nếu không thể xác định lớp đáng tin cậy, không cố gán lớp; ghi ca vào issue/review. Không dùng `occluded_level=3` cho vật chỉ bị cắt bởi mép ảnh.

#### `truncated` (select: `no`, `yes`)

- `no`: toàn bộ vật thể nằm trong khung ảnh.
- `yes`: một phần vật thể bị cắt bởi một hoặc nhiều cạnh ảnh.
- Nếu vật vừa bị che bởi vật khác vừa bị cắt mép ảnh, đặt `occluded_level` theo phần bị che và `truncated=yes`.

#### `can_review` (checkbox)

- `false`: quyết định gán nhãn rõ ràng theo guideline.
- `true`: còn nghi ngờ về lớp, phạm vi box, đối tượng bị che/cắt hoặc cách áp dụng quy tắc; cần reviewer xem lại.
- Tick `can_review` không thay thế việc ghi issue cấp ảnh khi đối tượng chưa được vẽ.

## 3. Quy tắc vẽ box

Mã quy tắc dùng trong QC: `R-01` trở đi.

- **R-01 — Box tight:** vẽ rectangle nhỏ nhất bao trọn phần nhìn thấy của đối tượng; không bao gồm bóng đổ, khoảng nền hoặc vật lân cận.
- **R-02 — Vật bị che:** box bao phần nhìn thấy của chính vật. Không suy đoán/bao vùng bị che khuất. Gán `occluded_level` theo ngưỡng tại mục 2.2.
- **R-03 — Vật bị cắt mép:** box đến đúng cạnh ảnh ở nơi vật bị cắt; đặt `truncated=yes`.
- **R-04 — Người và phương tiện:** người lái xe máy là một box `person` riêng và xe là một box `motorcycle` riêng. Box người ôm thân/người nhìn thấy, không bao gồm xe.
- **R-05 — Xe chở xe (`img_002.jpg`):** gán box `truck` cho xe vận chuyển; đồng thời gán box `car` riêng cho từng ô tô con được nhìn thấy trên xe chở, nếu taxonomy đã có lớp `car`. Không gán box cho phần xe bị che hoàn toàn. Xe có box riêng khi nhận diện được.
- **R-06 — Xe ba bánh:** tuk-tuk cơ giới ba bánh gán `tuk_tuk`; không gán thành `motorcycle`. Xích lô/xe đạp không động cơ ngoài phạm vi.
- **R-07 — Xe buýt:** gán `bus`, không gán `truck`, khi mục đích/kiểu xe chở hành khách được nhận diện rõ.
- **R-08 — Tượng, hình in, ma-nơ-canh, đồ chơi:** không gán `person`/phương tiện. Áp dụng cho tượng (`img_011.jpg`, `img_017.jpg`, `img_027.jpg`), hình in (`img_018.jpg`), ma-nơ-canh (`img_033.jpg`) và đồ chơi (`img_038.jpg`).
- **R-09 — Phản chiếu:** không gán đối tượng chỉ xuất hiện trong gương/phản chiếu (`img_020.jpg`). Gán đối tượng thật ngoài phản chiếu nếu có.
- **R-10 — Xe chủ (ego):** không gán phần xe/cabin của phương tiện mà camera đang gắn lên nếu nó chỉ xuất hiện như một phần cố định của góc nhìn camera. Nếu không xác định được ego hay xe trong cảnh, đặt `can_review=true`.
- **R-11 — Vật nhỏ:** nếu chiều cao box dự kiến <20 px, không tạo box đối tượng riêng; dùng `ignore_region` khi phù hợp với vùng/cụm không thể gán riêng. Đối tượng cao ≥20 px nhưng nhận diện được vẫn gán bình thường.
- **R-12 — Đám đông/cụm:** gán từng người/phương tiện riêng khi có thể tách và nhận diện được. Dùng `ignore_region` chỉ cho cụm quá dày hoặc các vật dưới ngưỡng không thể gán riêng; không dùng vùng ignore để che lỗi bỏ sót.
- **R-13 — Một vật, một box:** mỗi đối tượng vật lý chỉ có một box cho một lớp. Không tạo box trùng cho các bộ phận của cùng một xe.
- **R-14 — Ca chưa rõ:** đặt `can_review=true` cho box đã vẽ và ghi ảnh, vùng, mô tả ngắn vào `image_issues.csv` nếu không thể vẽ box. Không tự giải quyết bằng cách thêm lớp mới.

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định đề xuất | Lý do |
|---|---|---|---|---|
| `img_002.jpg` | Xe tải chở nhiều ô tô con | `R-05` | Một box `truck` và box `car` riêng cho từng ô tô con nhìn thấy; vùng bị che không suy đoán. | Phân biệt đối tượng vận chuyển với từng phương tiện được vận chuyển. |
| `img_004.jpg`, `img_016.jpg`, `img_035.jpg` | Tuk-tuk ba bánh | `R-06` | Gán `tuk_tuk` nếu lớp này đã có trong taxonomy/CVAT; nếu chưa, ghi `can_review`/issue và không dùng lớp thay thế tùy ý. | v0 không có lớp tuk-tuk. |
| `img_006.jpg`, `img_029.jpg`, `img_039.jpg` | Xe đạp/xích lô không động cơ | — | Không gán lớp. | Ngoài phạm vi taxonomy. |
| `img_009.jpg`, `img_031.jpg`, `img_037.jpg`, `img_040.jpg` | Xe buýt | `R-07` | Gán `bus` khi taxonomy cho phép; nếu CVAT chưa có lớp, đưa reviewer chốt đồng bộ taxonomy. | Không ép xe buýt vào `truck`. |
| `img_011.jpg`, `img_017.jpg`, `img_027.jpg` | Tượng người/tượng xe | `R-08` | Không gán `person` hoặc lớp xe. | Không phải người/phương tiện thật. |
| `img_018.jpg` | Người trong hình quảng cáo/biển | `R-08` | Không gán `person`. | Hình in không phải người hiện diện trong cảnh. |
| `img_020.jpg` | Đối tượng trong gương chiếu hậu | `R-09` | Không gán phản chiếu. | Tránh nhân đôi nhãn của vật thể. |
| `img_033.jpg` | Ma-nơ-canh trong cửa kính | `R-08` | Không gán `person`. | Không phải người thật. |
| `img_038.jpg` | Đồ chơi/mô hình | `R-08` | Không gán lớp đối tượng. | Vật mô phỏng không thuộc phạm vi. |
| `img_037.jpg` | Bán tải có nắp thùng cao (`Master 75`) | `R-14` | Đưa reviewer xác nhận `van` hay `truck`; tạm tick `can_review`. | Ca ranh giới kiểu thân xe, tránh phân loại tùy ý. |
| `img_005.jpg`, `img_026.jpg` | Người/xe bị che hoặc cắt mép | `R-02`, `R-03` | Box phần nhìn thấy; thuộc tính theo ngưỡng và cạnh ảnh. | Cần nhất quán về box và trạng thái. |

## 5. QC và phiên bản

### Ngưỡng QC áp dụng

| Chỉ số | Quy tắc |
|---|---|
| IoU box cao ≥50 px | Ngưỡng chấp nhận ≥0.85 |
| IoU box cao <50 px | Ngưỡng chấp nhận ≥0.70 |
| Ghép cặp box để tính IAA | Ghép 1–1, IoU ≥0.50; box chưa ghép đếm riêng |
| IAA IoU trung bình | ≥0.85 trên cặp box đã ghép |
| Đồng thuận lớp | ≥95% trên cặp box đã ghép |
| Độ phủ assignment | 100% lượt gán hợp lệ mới đo IAA |
| Lỗi theo ảnh | ≤2% mới xét Release; lô 40 ảnh có 1 ảnh lỗi (2.5%) là vượt ngưỡng |
| Vật nhỏ | chiều cao box <20 px theo R-11 |
| Release | phải review toàn bộ lô; vòng demo lấy mẫu chỉ ở trạng thái Hold |

Số liệu báo cáo phải ghi `tử số/mẫu số`. IAA cần được lưu trước calibration. Nếu IAA thấp, xác định nguyên nhân là guideline mơ hồ, người gán chưa hiểu quy tắc hay cùng hiểu sai; cập nhật guideline và truy vết quyết định.

### Xử lý và duyệt ca mơ hồ

- Annotator ghi nhận ca bằng `can_review` hoặc `submission/qc/image_issues.csv`.
- Reviewer/nhóm thống nhất quyết định, ghi mã rule, phiên bản guideline, người chốt, phạm vi sửa (`anh` hoặc `lo`) và kết quả kiểm lại vào hồ sơ calibration.
- Nếu thay đổi ý nghĩa quy tắc gán nhãn, kiểm lại toàn lô. Nếu chỉ sửa câu chữ mà không đổi cách gán, ghi rõ là editorial.
- Quyết định bàn giao theo Hold → Rework → Release; lỗi lặp cùng loại vẫn yêu cầu Rework.

### Lịch sử phiên bản

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| `v0` | Guideline khởi đầu, thuộc tính che khuất mô tả định tính và thiếu ca khó. | Bản starter của lab chưa đủ để hai annotator diễn giải thống nhất. |
| `v1.0` | Đề xuất ngưỡng `occluded_level`; làm rõ box phần nhìn thấy, truncation, xe chở xe, người đi xe máy, tượng/hình in/ma-nơ-canh/đồ chơi/phản chiếu, vật nhỏ và ca cần review. | Khảo sát tình huống biên trong 40 ảnh. Chờ nhóm/reviewer xác nhận và đồng bộ taxonomy/CVAT. |
