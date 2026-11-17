# Guideline gán nhãn — v1.0 — nhóm (101, 102, 103, 104)

- Phiên bản: v1.0
- Ngày: 2026-11-16
- Người viết: 101, 102, 103, 104 · Người chốt: cả nhóm (họp Pha 2)
- Loại nhãn: bounding box 2D trên 40 ảnh `img_001.jpg`–`img_040.jpg`; CVAT, xuất COCO 1.0

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện người và phương tiện tham gia giao thông trên ảnh 2D (camera giao thông, xe tự lái).

Hai lỗi nguy hiểm nhất:

- Bỏ sót người hoặc phương tiện thật: mô hình không thấy vật có thể va chạm.
- Gán nhãn vật không thật (tượng, đồ chơi, hình in, phản chiếu): mô hình báo động giả.

Nguyên tắc chung: chỉ gán vật thật, có mặt trong cảnh, nhận diện được lớp. Mỗi vật thật đúng một box, một lớp.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa | Ví dụ | Dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô chở người 4 bánh cỡ thường: sedan, hatchback, SUV, xe cổ; kể cả ô tô đang được xe khác chở. | img_002 (xe trên giá chở), img_015, img_019, img_023 | Tượng ô tô (img_027), ô tô đồ chơi (img_038): không gán |
| `truck` | Xe chở hàng có thùng/sàn hàng tách khỏi cabin, kể cả xe chở ô tô (box bao cả xe được chở, R-05) và xe bán tải (pickup). | img_002 (đầu kéo chở xe), img_007, img_001, img_012, img_028, img_037 (pickup) | Xe buýt → `bus`; xe thùng liền cabin → `van` |
| `van` | Xe thùng kín cỡ nhỏ, khoang hàng/khách liền cabin, dưới 16 chỗ. | img_005, img_016 | SUV → `car` |
| `bus` | Xe chở khách từ 16 chỗ trở lên: xe buýt, xe khách, minibus cỡ coaster. | img_009, img_031, img_037 (nền sau), img_040 | Xe chở hàng → `truck` |
| `motorcycle` | Xe 2 bánh có động cơ: xe máy, mô tô, xe tay ga, xe máy điện; mô tô có thùng bên gán chung một box. | img_005, img_021, img_029, img_022 | Xe đạp → `bicycle`; tuk-tuk → `three_wheeler` |
| `bicycle` | Xe 2 bánh đạp chân, không động cơ. | img_003, img_006, img_029 | Xe máy điện → `motorcycle` |
| `three_wheeler` | Xe 3 bánh chở người/hàng: tuk-tuk, auto rickshaw, xích lô. | img_004, img_016, img_035, img_040 (tuk-tuk); img_023 (xích lô) | Mô tô có thùng bên (img_022) → `motorcycle` |
| `oversized_load` | Vật cồng kềnh người/xe mang theo, vươn ra ngoài thân xe quá 1/2 chiều dài xe (người đi bộ: quá 1/2 chiều cao người). | img_029 (ống dài), img_035 (hàng chất cao trên auto rickshaw) | Hàng nằm gọn trong thùng xe (img_012): không gán |
| `person` | Người thật: đi bộ, đứng, ngồi, lái hoặc ngồi trên/trong xe. | img_019 (tài xế), img_029, img_032 | Tượng (img_017), ma-nơ-canh (img_033), người trên biển quảng cáo (img_018): không gán |
| `ignore_region` | Vùng có người/xe nhưng không gán từng vật được: quá dày, quá nhỏ, nhòe không xác định được biên. | img_015 (vệt nhòe), img_021, img_030, img_039 (xe xa, dày) | Không dùng cho vật không thật; không dùng để che vật rõ mà ngại vẽ |

Không tách lớp `rider`: "đang lái" suy ra từ box `person` chồng lên box xe; tách lớp sinh thêm ca khó (người dắt xe, người ngồi trên xe đỗ).

Vật không thuộc lớp nào (ngựa, nón giao thông, xe đẩy…): không gán. Phương tiện lạ chưa có quy tắc: tick `can_review` hoặc ghi vấn đề cấp ảnh; không tự dùng lớp thay thế.

### Thuộc tính

`occluded_level`: tỉ lệ diện tích vật bị vật khác che (ước trên hình dạng đầy đủ của vật).

| Mức | Ngưỡng |
|---|---|
| `0` | Không bị che (0%) |
| `1` | Bị che > 0% đến 30% |
| `2` | Bị che > 30% đến 70%, vẫn nhận chắc lớp |
| `3` | Bị che > 70%, vẫn đủ dấu hiệu nhận lớp |

- Không nhận được lớp thì không gán; ghi vấn đề cấp ảnh.
- Phần che tính cả do vật khác, do vật mang theo, và do khung kính/hàng rào.
- Xe có người lái/ngồi trên: tính cả phần người che, nhưng tối thiểu `1`.

`truncated`:

- `yes`: mép ảnh cắt mất một phần vật. `no`: vật nằm trọn trong ảnh.
- Bị vật khác che không phải truncated. Vừa bị che vừa bị cắt mép: `occluded_level` theo phần bị che và `truncated=yes`.

`can_review` (checkbox): tick khi chưa chắc về lớp, biên box, có nên gán không, hay thật/phản chiếu. Tick không thay cho ghi vấn đề cấp ảnh khi chưa vẽ được box.

## 3. Quy tắc vẽ box

| Mã | Quy tắc |
|---|---|
| R-01 | Box nhỏ nhất ôm trọn phần nhìn thấy của vật; không ôm bóng đổ, nền, vật bên cạnh. Không đoán phần bị che. |
| R-02 | Vật bị mép ảnh cắt: box chạm đúng mép, `truncated=yes`. |
| R-03 | Mỗi người thật một box `person`, gồm mũ bảo hiểm, quần áo, túi đeo. Người trong ô tô/tuk-tuk/xe buýt nhìn thấy được vẫn gán, box chỉ ôm phần người thấy được. |
| R-04 | Người và xe là hai box riêng. Box xe ôm thân xe (bánh, gương); box người ôm cả người, kể cả chân thòng cạnh xe. Hai box được chồng nhau. Xe chở 2 người: 1 box xe + 2 box `person`. Người dắt xe gán giống người lái. |
| R-05 | Xe chở xe: box `truck` bao trọn xe chở **và toàn bộ các xe đang được chở trên nó**; đồng thời mỗi xe được chở nhìn thấy là một box riêng theo lớp của nó (box nằm bên trong box `truck`). Không gộp nhiều xe được chở thành một box. Xe bị che hoàn toàn không gán. |
| R-06 | Không gán vật không thật: tượng (người, ngựa, xe), đài tưởng niệm, ma-nơ-canh, đồ chơi, mô hình, hình in trên biển quảng cáo/poster/màn hình. Xe thật hình dạng lạ nhưng chạy được: gán theo công dụng. |
| R-07 | Không gán hình phản chiếu (gương chiếu hậu, kính, vũng nước). Vật thật nhìn xuyên qua kính trong suốt hoặc hàng rào vẫn gán; phần bị khung che tính vào `occluded_level`. Phản chiếu là hình của vật ở phía trước mặt kính; nhìn xuyên là vật ở phía sau mặt kính. Cách kiểm: vật nhìn xuyên khớp với cảnh phía sau kính (bãi xe, mặt đường phía sau); hình phản chiếu lặp lại cảnh phía người chụp. Không phân biệt được: tick `can_review`. Lý do gán: xe sau hàng rào/vách kính (ví dụ ở ngã rẽ) vẫn là xe thật đang tới gần; bỏ không gán sẽ dạy mô hình bỏ qua đúng loại nguy hiểm này. |
| R-08 | Không gán xe của người chụp (ego): thân xe, cửa, cabin hiện ra như phần cố định của góc nhìn. |
| R-09 | Vật nhòe: nhận ra lớp và ước được biên → gán bình thường, tick `can_review`. Vệt nhòe không xác định được biên hoặc lớp → `ignore_region`, không đoán lớp. |
| R-10 | Vật cao dưới 20 px: không gán box riêng. |
| R-11 | Đám đông, bãi xe dày: vật tách được và cao ≥ 20 px gán từng box; phần còn lại chồng kín hoặc quá nhỏ vẽ một `ignore_region` bao cả cụm. Không vẽ một box chung thay nhiều vật. Vật nhỏ đứng lẻ: bỏ qua. |
| R-12 | Vật cồng kềnh vượt ngưỡng ở định nghĩa `oversized_load`: box riêng `oversized_load`. Box xe và box người không ôm vật này. Hàng chất trong thùng xe không gán riêng. |
| R-13 | Một vật thật một box. Không tạo box riêng cho bộ phận của cùng một xe (thùng bên mô tô, rơ-moóc gắn liền). |

## 4. Ca khó

| Ảnh | Tình huống | Rule | Quyết định |
|---|---|---|---|
| img_002 | Đầu kéo chở nhiều ô tô | R-05 | 1 `truck` bao cả đầu kéo, giá chở và các xe trên giá + từng `car` nhìn thấy |
| img_004, img_016, img_040 | Tuk-tuk | R-03 | `three_wheeler`; người trong xe `person` |
| img_035 | Auto rickshaw chở hàng chất cao | R-12 | `three_wheeler` + `oversized_load` cho đống hàng |
| img_023 | Xích lô | R-03, R-04 | `three_wheeler`; người đạp `person` |
| img_029 | Xe máy điện vác ống dài; 2 xe đạp | R-04, R-12 | `motorcycle` + `person` + `oversized_load`; mỗi xe đạp `bicycle` |
| img_022 | Mô tô có thùng bên | R-13 | Một box `motorcycle` bao cả thùng bên |
| img_001, img_012, img_028, img_037 | Xe bán tải (img_037 có nắp thùng cao) | — | `truck` |
| img_028 | Bán tải chở tượng ngựa | R-06 | Xe `truck`; tượng ngựa không gán, không ôm vào box xe |
| img_009, img_031, img_037, img_040 | Xe buýt (img_009 là minibus) | — | `bus` |
| img_017 | Tượng người cưỡi ngựa | R-06 | Không gán |
| img_027 | Tượng/bia mộ hình pickup | R-06 | Không gán |
| img_038 | Ô tô đồ chơi trên kệ | R-06 | Không gán |
| img_018 | Người trên biển quảng cáo; ô tô sau hàng rào | R-06, R-07 | Người in: không gán. Ô tô: gán `car`, `occluded_level` theo phần hàng rào che |
| img_033 | Ma-nơ-canh trong tủ kính; gương xe tải | R-06, R-07 | Ma-nơ-canh và hình trong gương: không gán |
| img_020 | Hình trong gương chiếu hậu; thân xe mình | R-07, R-08 | Không gán |
| img_032 | Ô tô sau kính nhà chờ; người đứng chờ xe buýt | R-07, R-11 | Ô tô sau kính: `car`. Người tách được: từng `person` |
| img_019 | Xe diễu hành có tài xế; thú bông trên xe | R-03, R-06 | Xe `car`, tài xế `person`; thú bông không gán |
| img_015 | Xe nhòe thành vệt giữa hai xe đỗ | R-09 | Vệt: `ignore_region`; xe rõ nét: `car` |
| img_014 | Xe máy nhòe sau người đi bộ | R-09 | Ước được biên: `motorcycle` + `can_review`; không ước được: `ignore_region` |
| img_021, img_030, img_039 | Dòng xe máy dày phía xa, bãi xe kín | R-10, R-11 | Hàng đầu gán từng box, phần xa `ignore_region` |
| img_005, img_006, img_021 | Xe máy có người lái | R-04 | `occluded_level` ≥ `1` |
| img_011 | Hình Predator bằng kim loại; dãy mô tô trưng bày | R-06 | Hình Predator là tượng: không gán. Mô tô thật đang đỗ: `motorcycle` |
| img_024 | Ô tô trưng bày trong nhà; xe vàng đặt trên nóc xe đen | R-06 | Ô tô thật: `car`. Xe vàng trên nóc chưa rõ thật hay mô hình: gán `car` + tick `can_review` |

Ca chưa chốt khi gán: tick `can_review` trên box hoặc ghi vấn đề cấp ảnh vào `image_issues.csv` (cột `image,annotator,region,note`) sau khi khóa nhãn.

## 5. QC và phiên bản

- Giữ nguyên `rules` trong `taxonomy.json`: IoU ≥ 0.85 (box cao ≥ 50 px), ≥ 0.70 (box nhỏ hơn); ghép cặp 1–1 tại IoU ≥ 0.5; đồng thuận lớp ≥ 95%; lỗi theo ảnh ≤ 2% (40 ảnh: 1 ảnh lỗi = 2.5% là vượt).
- Số liệu luôn ghi `tử số/mẫu số`. Release cần duyệt toàn lô; vòng demo lấy mẫu chỉ là Hold.
- Ca mơ hồ: người gán tick `can_review`/ghi vấn đề; nhóm chốt ở calibration, ghi mã quy tắc, phiên bản, người chốt, phạm vi (`anh` / `lo`) và kết quả kiểm lại.
- Đổi cách gán (lớp, box, thuộc tính) là thay đổi `semantic`, phải kiểm lại cả lô; chỉ sửa câu chữ là `editorial`.

| Phiên bản | Thay đổi | Ca liên quan |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên | — |
| v1.0 | Gộp 4 bản cá nhân. Thêm `bus`, `bicycle`, `three_wheeler`, `oversized_load`; pickup vào `truck`; ngưỡng `occluded_level`; quy tắc R-01…R-13 | img_002, img_004, img_015, img_020, img_029, img_032, img_035 |
