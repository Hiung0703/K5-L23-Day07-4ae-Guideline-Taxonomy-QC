# Guideline gán nhãn — v0 — 002

> MSSV: 3062
> Bản guideline cá nhân, viết độc lập ở Pha 1.

- Phiên bản: v0-002
- Loại nhãn: bounding box (hộp chữ nhật) trên ảnh đường phố
- Công cụ: CVAT, xuất COCO 1.0
- Người viết: Thành viên mã 002

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện người và phương tiện trên ảnh đường phố. Mục tiêu là tạo box nhất quán cho từng đối tượng có thể nhận diện được, kể cả khi đối tượng bị che một phần, ngồi trên phương tiện hoặc nằm trong một phương tiện chở nhiều phương tiện khác.

## 2. Lớp và thuộc tính

### Lớp

Mỗi đối tượng được gán đúng một lớp trong danh sách dưới đây.

| Lớp | Định nghĩa | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con, sedan hoặc SUV. | Ô tô con nhìn thấy trên đường hoặc trên xe chở. | Không dùng cho xe tải, xe buýt hoặc xe thùng kín. |
| `truck` | Xe tải hoặc xe có thùng/chở hàng riêng. | Xe đầu kéo, xe tải chở ô tô. | Không dùng cho ô tô con chỉ vì ô tô đó đang được chở. |
| `van` | Xe thùng kín cỡ nhỏ. | Xe van giao hàng. | Không dùng cho xe tải có thùng hàng lớn. |
| `motorcycle` | Xe máy, mô tô và phương tiện ba bánh có động cơ như tuk-tuk. | Xe máy, xe tay ga, tuk-tuk. | Không dùng cho xe đạp hoặc người đi bộ. |
| `person` | Người đi bộ hoặc người đang ngồi/lái trên phương tiện. | Người lái xe máy, hành khách trên tuk-tuk. | Không gán cho tượng, hình nộm, hình in hoặc hình phản chiếu. |
| `ignore_region` | Vùng không thể gán đáng tin cậy từng vật thể theo quy tắc. | Vùng chen lấn quá dày hoặc chi tiết quá nhỏ để tách vật thể. | Không dùng để che một vật thể rõ ràng chỉ vì khó vẽ. |

### Thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | `0`: thấy rõ; `1`: bị che ít; `2`: bị che nhiều nhưng vẫn nhận diện được; `3`: chỉ còn dấu hiệu rất hạn chế, cần reviewer kiểm tra. |
| `truncated` | `no`, `yes` | `yes` khi vật bị cắt bởi mép ảnh; chỉ bị che bởi vật khác thì không phải truncated. |
| `can_review` | tick / không | Tick khi ảnh mờ, vật thể chồng lấn, góc nhìn qua kính hoặc lớp/ranh giới box chưa chắc chắn. |

## 3. Quy tắc vẽ box

1. Gán box cho mọi người và phương tiện có thể nhận diện được trong ảnh.
2. Box ôm sát toàn bộ phần nhìn thấy của đúng đối tượng; không kéo box sang đối tượng khác.
3. Với người đi xe máy, vẽ **hai box riêng**: một box `person` ôm người và một box `motorcycle` ôm toàn bộ xe máy. Không gộp người và xe máy vào một box.
4. Với xe chở xe, xác định và vẽ box cho **từng xe được chở** nếu từng xe còn nhận diện được. Đồng thời vẽ một box `truck` lớn bao trọn xe chở và toàn bộ phần xe được chở trên đó. Không gộp nhiều xe được chở thành một box duy nhất.
5. Với tuk-tuk, vẽ box `motorcycle` cho toàn bộ tuk-tuk và vẽ box `person` riêng cho từng người nhìn thấy trong xe. Người bị che một phần vẫn được gán nếu còn nhận diện được.
6. Người trong ô tô, xe tải, xe van hoặc tuk-tuk vẫn thuộc lớp `person` nếu quan sát được là người thật. Box chỉ ôm phần người nhìn thấy; không mở rộng box theo thân xe.
7. Nếu người hoặc phương tiện bị cắt ở mép ảnh, vẫn vẽ box phần nhìn thấy và đặt `truncated=yes`.
8. Ảnh mờ do chuyển động: nếu vẫn nhận diện được người/phương tiện, vẽ box theo phần hình ảnh quan sát được và tick `can_review`. Nếu không thể xác định đáng tin cậy đó là đối tượng nào, không tự suy đoán; đánh dấu vùng/vấn đề để reviewer quyết định.
9. Nhiều vật thể chen lấn: cố gắng vẽ từng người/phương tiện còn tách được. Nếu các vật thể chồng lấn đến mức không thể xác định ranh giới hoặc số lượng, đặt `ignore_region` cho vùng đó và tick `can_review`; không vẽ một box chung thay cho nhiều vật thể.
10. Góc nhìn qua kính chắn gió hoặc cửa kính: người thật nhìn thấy qua kính vẫn được vẽ box `person`; kính chỉ làm tăng mức che/khó quan sát. Không vẽ box cho người chỉ xuất hiện trong hình phản chiếu trên kính hoặc gương.
11. Không gán box cho tượng, hình nộm, đồ chơi, hình in, ảnh trên biển quảng cáo hoặc vật phản chiếu. Đây là vật không thuộc lớp mục tiêu.
12. Vật thể có chiều cao dưới 20 px hoặc không đủ bằng chứng nhận diện được xử lý theo `ignore_region`/`can_review` và phải ghi chú để reviewer xem lại.

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_006.jpg`, `img_004.jpg` | Người đi xe máy, có thể bị che bởi xe hoặc người khác. | R-01 | Vẽ `person` và `motorcycle` thành hai box độc lập; tick `can_review` nếu ranh giới bị che khó xác định. | Mô hình cần nhận diện riêng người và phương tiện. |
| `img_002.jpg` | Xe tải chở nhiều ô tô. | R-02 | Vẽ từng `car` được chở và một box `truck` lớn bao gồm xe chở cùng toàn bộ xe được chở. | Bảo toàn cả số lượng xe con và đối tượng xe chở. |
| `img_003.jpg`, `img_015.jpg`, `img_032.jpg` | Tuk-tuk/xe ba bánh có người ngồi bên trong. | R-03 | Vẽ toàn bộ xe là `motorcycle`, vẽ từng người nhìn thấy là `person`; tick `can_review` nếu phân loại tuk-tuk không chắc. | Taxonomy v0 chưa có lớp tuk-tuk riêng; tạm quy về phương tiện ba bánh có động cơ gần nhất là `motorcycle`. |
| `img_014.jpg` | Phương tiện đang di chuyển bị nhòe. | R-04 | Nếu còn nhận diện được thì vẽ box và tick `can_review`; nếu không đủ bằng chứng thì không suy đoán lớp, ghi vấn đề để reviewer chốt. | Tránh tạo nhãn dựa trên hình dạng không đáng tin cậy. |
| `img_019.jpg`, `img_027.jpg` | Nhiều phương tiện chen lấn, ranh giới chồng lấn. | R-05 | Vẽ từng vật thể tách được; vùng không thể tách thì dùng `ignore_region` và tick `can_review`, không vẽ box chung. | Box chung làm sai số lượng và ranh giới đối tượng. |
| `img_020.jpg` | Người hoặc phương tiện ở trên/đằng sau kính, có phản chiếu. | R-06 | Người thật nhìn thấy qua kính vẫn vẽ `person`; phản chiếu không vẽ box; tick `can_review` nếu không phân biệt được thật và phản chiếu. | Chỉ gán đối tượng thực sự xuất hiện trong cảnh. |
| `img_017.jpg`, `img_030.jpg`, `img_035.jpg` | Tượng, ma-nơ-canh hoặc đồ chơi có hình dạng giống người/phương tiện. | R-07 | Không gán `person`/phương tiện cho tượng, hình nộm, đồ chơi hoặc hình in. | Đây không phải đối tượng thật cần phát hiện. |

Nếu chưa chốt được ca khó, phải tick `can_review` trên box liên quan hoặc ghi vấn đề ở cấp ảnh. Không tự thay đổi taxonomy; mọi đề xuất thêm lớp phải được reviewer/nhóm xác nhận.

## 5. QC và phiên bản

- Mọi thay đổi làm thay đổi lớp, số lượng box, phạm vi box hoặc cách xử lý ca khó phải tăng phiên bản guideline.
- Ca mơ hồ phải có `can_review` và được đưa ra reviewer trong buổi họp ca khó.
- Box phải được kiểm tra về lớp, vị trí, kích thước, `occluded_level` và `truncated`.
- Ngưỡng tham chiếu QC của bộ dữ liệu: box lớn đạt IoU từ 0,85; box nhỏ đạt IoU từ 0,70; vật thể dưới 20 px cần xem xét `ignore_region`.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| `v0-002` | Bổ sung quy tắc tách người/xe máy, xe chở xe, tuk-tuk, ảnh mờ, chen lấn, kính/phản chiếu và tượng. | Các ca khó trong `img_002.jpg`, `img_003.jpg`, `img_004.jpg`, `img_006.jpg`, `img_014.jpg`, `img_017.jpg`, `img_019.jpg`, `img_020.jpg`, `img_027.jpg`, `img_030.jpg`. |
