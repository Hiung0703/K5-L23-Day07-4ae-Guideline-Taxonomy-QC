# Guideline gán nhãn — v1.1 — nhóm (101, 102, 103, 104)

- Phiên bản: v1.1
- Ngày cập nhật: 2026-11-17
- Thành viên phụ trách: 101, 102, 103, 104 · Lead: **102**. Phân công biên soạn: 101; kiểm hồ sơ/QC: 103; kiểm bàn giao: 104.
- Loại nhãn: bounding box 2D trên 40 ảnh `img_001.jpg`–`img_040.jpg`; CVAT, xuất COCO 1.0


> Trạng thái: bản nội dung phục vụ xử lý calibration. Bảng ca đã có hướng xử lý theo ảnh; xác nhận người chốt, sửa nhãn và kết quả kiểm lại phải được ghi sau khi thực hiện. Phiên bản này không xác nhận nhãn đã được sửa hoặc đã đạt QC. Phân công chi tiết tại [TEAM.md](../../TEAM.md).

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
- Xe hai bánh có người lái/ngồi che thân xe: tính cả phần người che, tối thiểu `1`. Với ô tô, chỉ thấy tài xế qua kính chưa đủ để tăng mức che khuất của xe; đánh giá phần thân xe thực sự bị che theo `CAL-01`.

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

## 5. Quy tắc calibration bổ sung

Các mã `CAL-01`–`CAL-05` dùng cùng `R-01`–`R-13` trong [bảng quyết định](../qc/calibration_decisions.csv). Mỗi quyết định phải gắn với đúng ảnh, lớp và vùng box của ca; không mặc định chọn người có nhiều box hơn hoặc lấy trung bình hai box.

### CAL-01 — Thống nhất occluded_level

- Ước phần diện tích vật bị một vật khác che so với hình dạng đầy đủ của chính vật đó: `0` = không che; `1` = trên 0% đến 30%; `2` = trên 30% đến 70%; `3` = trên 70% nhưng vẫn nhận được lớp.
- Phần nằm ngoài khung ảnh thuộc `truncated`; không cộng phần này vào tỷ lệ che khuất. Đánh giá hai thuộc tính độc lập.
- Kính trong suốt, bóng tối, màu sơn hoặc ảnh mờ tự chúng không phải vật che. Khung kính, hàng rào, người hoặc xe phía trước thực sự che vật thì tính phần bị che.
- Xe hai bánh có người lái/ngồi che thân xe áp dụng quy tắc tối thiểu `1` của v1.0. Người và xe vẫn đánh giá riêng; không sao chép mức che của người sang xe.
- Việc thấy tài xế qua kính ô tô không tự động làm toàn bộ ô tô bị che. Chỉ tăng mức khi có phần thân xe thực sự bị vật khác che; đây là điểm làm rõ phạm vi áp dụng so với câu chung về người lái trong v1.0.
- Khi không ước chắc mức ở ranh 30%/70%, ghi phần bị che và lý do cần đối chiếu, giữ `can_review` cho đến khi người kiểm chốt; không chọn mức chỉ theo số đông.

### CAL-02 — Phân biệt truncated với che khuất

- Chỉ ghi `truncated=yes` khi mép ảnh thực sự cắt mất một phần vật. Vật bị người/xe/hàng rào che trong ảnh không vì thế thành truncated.
- Box sát mép hoặc tọa độ đã làm tròn chưa đủ kết luận; xem vật gốc tại mép. Nếu vật bị cắt, box chạm đúng mép theo `R-02`; không kéo box ra ngoài ảnh.
- Box của phần nhìn thấy vẫn theo `R-01`; không ước lượng thêm chân, bánh hoặc đuôi nằm ngoài khung để làm IoU cao hơn.

### CAL-03 — Xác định phạm vi ignore

- Vật thật nhận rõ lớp, tách được biên và cao từ 20 px trở lên: gán riêng. Vật đứng lẻ dưới 20 px bỏ qua; cụm quá nhỏ hoặc quá dày không tách được dùng `ignore_region` theo `R-10`, `R-11`.
- Với đám đông/bãi xe, tách lớp đối tượng phía trước khỏi phần nền không thể gán; thu hẹp hoặc chia vùng ignore theo phần còn lại. Không phủ một vùng ignore lớn lên các vật rõ chỉ để xóa bất đồng.
- Một bên có ignore, một bên có box không chứng minh bên nào đúng. Xem vật tại vùng giao: vật rõ thì giữ/vẽ box và thu hẹp ignore; không phân tách được thì dùng ignore cho phần đó.
- Không dùng ignore cho tượng, đồ chơi, hình in, phản chiếu hoặc xe ego; các vật này loại theo `R-06`–`R-08`.
- Mục tiêu chỉnh vùng là không để ignore che vật rõ, đặc biệt không phủ từ 50% diện tích box vật rõ trở lên. Con số 50% mô tả cách script nhận box nằm trong ignore, không phải một ngưỡng QC mới.

### CAL-04 — Ghép lại cùng vật trước khi kết luận lỗi

- Cảnh báo `bo_sot_thua` là box chưa ghép, chưa phải kết luận bỏ sót hoặc thừa. Kiểm xem box ở dòng khác có cùng vật nhưng IoU dưới 0,5, gộp nhiều người hoặc chọn sai phần nhìn thấy hay không.
- Khi hai box thực sự cùng vật: sửa theo biên nhìn thấy và giữ một annotation cho vật trong nhãn cuối; không cộng cả hai thành hai vật.
- Khi script ghép box người với xe ở cùng vùng: tách người và xe theo `R-03`, `R-04`; đánh giá hai vật độc lập. Không đổi lớp chỉ để hai box ghép sẵn có cùng tên.
- Khi thật sự thiếu vật rõ: bổ sung box đúng lớp, thuộc tính. Khi vật không thuộc phạm vi hoặc cùng vật bị vẽ hai lần: loại box thừa với lý do cụ thể.
- IoU đạt không chứng minh lớp đúng. Kiểm cả các vật mà hai người cùng gán giống nhau; ví dụ pickup vẫn phải là `truck` dù cả hai cùng chọn `car`.
- Không sửa nhãn gán độc lập v1.0 hoặc chạy lại IAA để thay số đo trước calibration. Sửa trên bản phục vụ calibration, giữ dấu vết ảnh/vùng và người kiểm.

### CAL-05 — Ca chưa đủ bằng chứng và cách ghi nhận

- Khi chưa phân biệt được thật/mô hình, phản chiếu/nhìn xuyên, hoặc lớp của vật bị che nặng: mô tả rõ phương án và bằng chứng còn thiếu. Không chuyển một đề xuất thành xác nhận đã kiểm chỉ vì các cột có nội dung.
- Trường `nguoi_phu_trach`, `nguoi_chot` và `nguoi_kiem_lai_du_kien` ghi người được giao phụ trách, chốt và kiểm lại. Mã đã điền là phân công, chưa xác nhận công việc đã hoàn tất. `kiem_lai` bắt đầu bằng `CHO_KIEM` là chỉ dẫn cho reviewer, phải thay bằng kết quả thật sau khi kiểm bản sửa.
- `phieu_sua` mô tả việc cần làm nếu bắt đầu bằng `CHO_SUA`; đổi thành dấu vết bản sửa thực tế sau khi thực hiện. Đối chiếu cùng một vật xuất hiện ở nhiều case để tránh sửa/xóa lặp.
- `trang_thai=da_phan_cong_cho_chot_sua_kiem_lai` nghĩa là đã có hướng xử lý và người được giao, còn chờ chốt, sửa nhãn và kiểm lại. Đổi thành `da_chot_va_kiem_lai` chỉ sau khi đủ bằng chứng; người phụ trách và reviewer khác nhau theo phân công. Có đủ nội dung các cột chưa đồng nghĩa ca đã đóng.
- Sau khi chốt, nếu vẫn giữ `can_review`, phải ghi rõ vấn đề còn mở và phạm vi ảnh; giữ Hold nếu vấn đề ảnh hưởng nhãn cuối mà chưa xử lý.

### Ca minh họa dùng khi chốt bảng quyết định

| Ca | Nhận định để đối chiếu và xử lý | Quy tắc |
|---|---|---|
| C096, ảnh 007 | Xe thùng ISUZU bên phải là `truck`; kiểm thêm pickup giữa ảnh dù hai người cùng chọn `car`, vì cùng lớp không chứng minh đúng lớp. | Định nghĩa lớp, CAL-04 |
| C184–C188, ảnh 018 | Các xe nhìn xuyên qua hàng rào cần đối chiếu về lớp ô tô, không chuyển thành `three_wheeler` chỉ vì thân xe bị che; hình người trên biển quảng cáo loại khỏi nhãn. | R-06, R-07, CAL-01 |
| C208, ảnh 019 | Phân biệt người lái thật với thú bông/nhân vật trang trí trên xe; hình trang trí không phải `person`. | R-03, R-06 |
| C279/C281, ảnh 024 | Xe vàng trên nóc cần giữ vấn đề thật/mô hình ở trạng thái cần xác nhận; không tự đổi sang `oversized_load` hoặc bỏ cờ khi chưa đủ căn cứ. | R-06, R-12, CAL-05 |
| C441, ảnh 037 | Pickup có nắp thùng cao vẫn là `truck`; không gộp bus phía sau vào box pickup. | Định nghĩa lớp, R-01, CAL-04 |
| C341/C448/C465, ảnh 031/039/040 | Các vật quá xa hoặc bị mép ảnh cắt còn thiếu dấu hiệu phân lớp: giữ rõ phương án và vấn đề cần kiểm, không coi lớp đề xuất là đã chốt. | R-02, R-09, CAL-05 |

### Phạm vi áp dụng và bàn giao

- Bổ sung cách xử lý người lái nhìn qua kính, vùng ignore và ghép sai có thể làm thay đổi nhãn: ghi `kind=semantic` trong [hồ sơ thay đổi](../guideline_change.json), rà lại **toàn bộ 40 ảnh**, không chỉ 35 ảnh bị cờ.
- Giữ nguyên schema lớp/thuộc tính và toàn bộ `rules` của [taxonomy v1.0](../taxonomy.json). Guideline v1.1 không tự động yêu cầu taxonomy v1.1 khi schema không đổi.
- Lead **102** tập hợp các trường hợp chưa chốt; **101** cập nhật diễn giải, **103** kiểm hồ sơ và số đo, **104** kiểm bằng chứng bàn giao. Mỗi người vẫn phụ trách và kiểm chéo 10 ảnh theo [TEAM.md](../../TEAM.md).
- Năm ảnh không có cờ 017, 020, 027, 033, 038 vẫn phải kiểm theo phiên bản mới và tham gia QC toàn lô.

## 6. QC và phiên bản

- Giữ nguyên `rules` trong `taxonomy.json`: IoU ≥ 0.85 (box cao ≥ 50 px), ≥ 0.70 (box nhỏ hơn); ghép cặp 1–1 tại IoU ≥ 0.5; đồng thuận lớp ≥ 95%; lỗi theo ảnh ≤ 2% (40 ảnh: 1 ảnh lỗi = 2.5% là vượt).
- Số liệu luôn ghi `tử số/mẫu số`. Release cần duyệt toàn lô; vòng demo lấy mẫu chỉ là Hold.
- Ca mơ hồ: người gán tick `can_review`/ghi vấn đề; nhóm chốt ở calibration, ghi mã quy tắc, phiên bản, người chốt, phạm vi (`anh` / `lo`) và kết quả kiểm lại.
- Đổi cách gán (lớp, box, thuộc tính) là thay đổi `semantic`, phải kiểm lại cả lô; chỉ sửa câu chữ là `editorial`.

| Phiên bản | Thay đổi | Ca liên quan |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên | — |
| v1.0 | Gộp 4 bản cá nhân. Thêm `bus`, `bicycle`, `three_wheeler`, `oversized_load`; pickup vào `truck`; ngưỡng `occluded_level`; quy tắc R-01…R-13 | img_002, img_004, img_015, img_020, img_029, img_032, img_035 |
| v1.1 | Bổ sung CAL-01…CAL-05: che khuất/cắt mép, vùng ignore, ghép lại cùng vật, tách người–xe và trạng thái chốt/kiểm lại; làm rõ người lái qua kính. Thay đổi semantic, rà lại cả lô; chưa xác nhận sửa nhãn/QC. | 466 ca trên 35 ảnh có cờ; kiểm lại toàn bộ 40 ảnh |
