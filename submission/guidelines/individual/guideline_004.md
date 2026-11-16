# Guideline gán nhãn — bản cá nhân — 004

- Phiên bản: v0-individual-004
- Ngày: 2026-11-16
- Người viết: 004 · Duyệt: chờ họp nhóm Pha 2

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện người và phương tiện tham gia giao thông trên ảnh 2D (camera giao thông, xe tự lái).

Hai lỗi nguy hiểm nhất:
- Bỏ sót người hoặc xe thật: mô hình không thấy vật có thể va chạm.
- Gán nhãn vật không thật (tượng, đồ chơi, ảnh in, phản chiếu): mô hình báo động giả.

Nguyên tắc chung: chỉ gán vật thật, có mặt trong cảnh và có thể tham gia giao thông.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa | Ví dụ | Dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô chở người 4 bánh: sedan, hatchback, SUV, xe cổ, xe diễu hành vẫn chạy được. | img_015, img_019 | Ô tô đồ chơi (img_038), tượng xe (img_027): không gán |
| `truck` | Xe chở hàng có thùng tách khỏi cabin, kể cả xe chở ô tô. | img_002, img_007 | Pickup: chưa chốt (mục 4) |
| `van` | Xe thùng kín cỡ nhỏ, khoang hàng/khách liền cabin, dưới 16 chỗ. | img_005 | SUV → `car` |
| `bus` | Xe chở khách từ 16 chỗ trở lên: xe buýt, xe khách. | img_040 | Minibus dưới 16 chỗ → `van` |
| `motorcycle` | Xe 2 bánh có động cơ: xe máy, mô tô, xe điện, xe tay ga. | img_005, img_029 | Xe đạp → `bicycle` |
| `bicycle` | Xe 2 bánh đạp chân, không động cơ. | img_029 | Xe đạp điện → `motorcycle` |
| `three_wheeler` | Xe 3 bánh chở người/hàng: tuk-tuk, auto rickshaw, xích lô. | img_004, img_035, img_009, img_023, img_039 | Mô tô có thùng bên (img_022): chưa chốt |
| `oversized_load` | Vật cồng kềnh người/xe mang theo, vươn ra ngoài thân xe quá 1/2 chiều dài xe (người đi bộ: quá 1/2 chiều cao người). | img_029 (ống dài) | Hàng nằm gọn trong thùng xe (img_012): không gán |
| `person` | Người thật: đi bộ, đứng, ngồi, lái hoặc ngồi trên xe. | img_011 (người hóa trang), img_029 | Tượng, ma-nơ-canh, người trên biển quảng cáo: không gán |
| `ignore_region` | Vùng có người/xe nhưng không gán từng vật được (quá dày, quá nhỏ, quá nhòe). | img_021, img_030 | Không dùng cho vật không thật |

Không tách lớp `rider`. Người lái và người đi bộ cùng hình dạng; "đang lái" suy ra được từ box người chồng lên box xe. Tách lớp sẽ sinh thêm ca khó (người dắt xe, người ngồi trên xe đang đỗ).

Vật không thuộc lớp nào (ngựa, xe đẩy, xe lăn…): không gán. Phương tiện lạ chưa có quy tắc: ghi vấn đề cấp ảnh.

#### Đề xuất thêm lớp (nhóm chốt ở Pha 2)

So với v0, bản này thêm 4 lớp: `bus`, `bicycle`, `three_wheeler`, `oversized_load`, và định nghĩa rõ hơn các lớp cũ. Nếu nhóm không thêm lớp:
- `bus` gán vào `truck`.
- `bicycle`, `three_wheeler`, `oversized_load` không gán box; người trên xe vẫn gán `person`; ghi ảnh vào `image_issues.csv`.

### Thuộc tính

| Thuộc tính | Mức | Ngưỡng |
|---|---|---|
| `occluded_level` | `0` | Bị che < 10% |
| | `1` | Bị che 10–40% |
| | `2` | Bị che 40–80% |
| | `3` | Bị che > 80%, chỉ còn nhận ra loại vật |
| `truncated` | `yes` | Mép ảnh cắt mất một phần vật |
| | `no` | Thấy trọn vật trong ảnh |
| `can_review` | tick | Không chắc về lớp, biên box, hoặc có nên gán không |

Phần bị che tính cả do vật khác lẫn do vật mang theo.

## 3. Quy tắc vẽ box

| Mã | Quy tắc |
|---|---|
| R-01 | Mỗi người thật một box `person`, gồm mũ bảo hiểm, quần áo, túi đeo. Người đi bộ và người trên xe đều là `person`. |
| R-02 | Người và xe là hai box riêng. Box xe ôm thân xe (cả bánh, gương). Box người ôm cả người, kể cả chân thòng cạnh xe. Hai box được chồng nhau. Xe chở 2 người: 1 box xe + 2 box `person`. |
| R-03 | Box ôm sát phần nhìn thấy, không đoán phần bị che. Vật bị mép ảnh cắt: box chạm mép, `truncated=yes`. |
| R-04 | Không gán vật không thật: tượng (người, ngựa, xe), đài tưởng niệm, ma-nơ-canh, đồ chơi, mô hình, hình in trên biển quảng cáo, poster, màn hình. Người thật hóa trang vẫn là `person`. Xe thật có hình dạng lạ nhưng chạy được thì gán theo công dụng. |
| R-05 | Không gán hình phản chiếu (gương chiếu hậu, kính, vũng nước). Vật thật nhìn xuyên qua kính trong suốt thì vẫn gán: box ôm phần thấy được, phần bị khung che tính vào `occluded_level`. Phản chiếu là hình của vật ở phía trước mặt kính; nhìn xuyên là vật ở phía sau mặt kính. |
| R-06 | Vật nhòe nhưng nhận ra loại và ước được biên: gán bình thường, tick `can_review`. Vệt nhòe không xác định được biên: vẽ `ignore_region`. |
| R-07 | Vật cao dưới 20 px: không gán box riêng. |
| R-08 | Đám đông, bãi xe dày: vật hàng đầu tách được và cao ≥ 20 px thì gán từng box; phần còn lại chồng kín hoặc quá nhỏ thì vẽ một `ignore_region` bao cả cụm. Vật nhỏ đứng lẻ thì bỏ qua. |
| R-09 | Vật cồng kềnh vượt ngưỡng ở định nghĩa lớp: gán box riêng `oversized_load`. Box xe và box người không ôm vật này. Phần vật che xe/người tính vào `occluded_level`. Lý do: vật này chiếm chỗ trên đường, dễ gây va chạm. |
| R-10 | Xe chở xe: xe chở là `truck`, mỗi xe được chở thấy rõ là một box riêng theo lớp của nó. |
| R-11 | Xe có người lái/ngồi trên: `occluded_level` tính theo tổng phần xe bị che (kể cả người lái), nhưng tối thiểu là `1`. Ví dụ người lái che 50% xe: `2`. |

## 4. Ca khó

| Ảnh | Tình huống | Rule | Quyết định | Lý do |
|---|---|---|---|---|
| img_029 | Người lái xe máy điện vác ống dài | R-02, R-09 | 3 box: `motorcycle`, `person`, `oversized_load` | Ống chiếm làn đường. Không ôm vào box xe vì ống dài gấp 3–4 lần xe |
| img_029 | 2 người đứng cạnh xe đạp | R-01 | Mỗi xe một box `bicycle`, mỗi người một box `person` | |
| img_004, img_035 | Tuk-tuk, auto rickshaw | R-01 | `three_wheeler` + `person` cho người trên xe | Không phải ô tô, cũng không phải xe máy |
| img_009, img_023, img_039 | Xích lô | R-01 | `three_wheeler`; người đạp và khách mỗi người một box `person` | Cùng dạng xe 3 bánh chở khách |
| img_017 | Tượng người cưỡi ngựa | R-04 | Không gán | Không phải vật thật |
| img_027 | Tượng xe pickup | R-04 | Không gán | |
| img_038 | Ô tô đồ chơi trên kệ | R-04 | Không gán | |
| img_033 | Ma-nơ-canh | R-04 | Không gán | |
| img_018 | Người trên biển quảng cáo | R-04 | Không gán | |
| img_011 | Người hóa trang | R-04 | `person` | Người thật |
| img_019 | Xe diễu hành có tài xế | R-04, R-05 | Xe `car`, tài xế sau kính `person`; thú bông không gán | Xe thật chạy được |
| img_020 | Hình trong gương chiếu hậu | R-05 | Không gán | Phản chiếu |
| img_032 | Ô tô sau kính nhà chờ xe buýt | R-05 | `car`, `occluded_level` theo phần khung che | Xe thật ở sau kính |
| img_005, img_006, img_021 | Xe máy có người lái | R-11 | `occluded_level` ≥ `1` | |
| img_015 | Xe nhòe thành vệt | R-06 | Vệt: `ignore_region`; xe rõ nét: `car` | |
| img_014 | Xe máy nhòe sau người đi bộ | R-06 | Ước được biên: `motorcycle` + `can_review`; không ước được: `ignore_region` | |
| img_021, img_030, img_040 | Dòng xe dày phía xa, bãi xe kín, giao thông đông | R-07, R-08 | Hàng đầu gán từng box, phần xa `ignore_region` | |

Cần hỏi reviewer:
- img_002: xe được chở trên đầu kéo có cần gán `car` không (R-10 đang gán)?
- Pickup (img_001, img_012, img_028, img_037): `car` hay `truck`?
- Mô tô có thùng bên (img_022): `motorcycle` hay `three_wheeler`?
- Xe của người chụp, nội thất xe (img_024): có gán không?

## 5. QC và phiên bản

- Giữ nguyên ngưỡng `rules` trong `taxonomy.json`: IoU ≥ 0.85 (vật lớn), ≥ 0.7 (vật dưới 50 px); đồng thuận lớp ≥ 0.95; lỗi ảnh ≤ 2%.
- Khi gán mà chưa chắc: tick `can_review`; không vẽ được box thì ghi vấn đề cấp ảnh. Người điều phối chốt ở calibration.

| Phiên bản | Thay đổi | Ca liên quan |
|---|---|---|
| v0 | Bản khởi đầu | — |
| v0-individual-004 | Thêm `bus`, `bicycle`, `three_wheeler`, `oversized_load`; định nghĩa lại các lớp; ngưỡng `occluded_level`; quy tắc R-01…R-11 | img_029, img_004, img_009, img_017, img_020, img_015, img_032, img_040 |
