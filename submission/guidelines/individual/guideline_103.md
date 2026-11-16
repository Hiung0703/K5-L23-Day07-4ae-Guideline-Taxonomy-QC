# Guideline gán nhãn — V1 — 103

> Copy file này (hoặc `guideline_v0_starter.md`) rồi viết. Một guideline đủ gồm **5 phần**. Câu nào của v0
> làm hai người đọc ra hai cách khác nhau thì phải viết lại cho một cách hiểu duy nhất.

- Phiên bản: V1
- Ngày: 16/11/2026
- Người viết / duyệt: 103

## 1. Mục tiêu
Nhãn dùng để huấn luyện mô hình phát hiện người và phương tiện trên ảnh đường phố (ví dụ cho camera giao thông hoặc xe tự lái).

## 2. Lớp và thuộc tính
### Lớp
| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con. |
| `truck` | Xe tải. |
| `van` | Xe thùng kín cỡ nhỏ. |
| `motorcycle` | Xe máy, mô tô. |
| `person` | Người. |
| `ignore_region` | Vùng không cần gán từng vật. |

### Thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Mức bị che: 0 là thấy rõ, 1 là bị che ít, 2 là bị che nhiều, 3 là không rõ. |
| `truncated` | `no`, `yes` | Vật bị cắt ở mép ảnh. |
| `can_review` | tick / không | Tick khi bạn không chắc, để người khác xem lại. |

## 3. Quy tắc vẽ box

1. Gán box cho mọi người và phương tiện trong ảnh.
2. Box ôm sát vật.
3. Chọn đúng lớp và thuộc tính.

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_015 | có 1 vật thể chạy nhanh qua 2 ô tô đen và trắng đang đỗ và không thể nào xác định vật thể đó là gì | R1 | không gán box khi không biết vật thể đó là gì | Không nhận dạng được vật thể do chạy quá nhanh và mờ |
| img_018 | ô tô nằm sau hàng rào| R2 | Gán box cho từng vật thể ô tô và sử dụng thuộc tính occluded | do vật thể bị hàng rào che hoàn toàn nhưng vẫn nhìn thấy |
| img_019 | người ngồi trong ô tô mui trần | R3 | gán box cho người ngồi trong ô tô và sử dụng thuộc tính occluded | do ô tô là xe mui trần, người bị cắt 1 phần khi ngồi trong ô tô |
| img_020 | ô tô được phản chiếu qua gương | R4 | Không gán box cho vật thể qua gương | vật thể hiể thị qua gương không phải là ô tô thật |
| img_021 | Người đi xe máy | R5 | Tách riêng box người và box xe| tránh trường hợp gặp tình huống người dắt xe mà AI hiểu là đi xe|
| img_027 | Ô tô giả| R6 | Không gán vật thể không phải là thật | Chỉ gắn vật thể có thật |
| img_028 | Ô tô chở ngựa giả| R7 | Chỉ gán box ô tô không bao gồm bức tượng con ngựa | Tượng ngựa không có trong class |
| img_029 | người vác thanh nhôm đi xe máy | R8 | Chỉ gán box người và box xe máy , không gán cho thanh nhôm được vác | Không có trong class |
| img_032 | Ô tô nằm sau gương | R9 | Không gán ô tô | Ô tô nằm sau gương được coi là vật thể giả |
| img_033 | Người phản chiếu qua gương, manocanh, 1 phần xe tải| R10 | Không gán box | Không gán box cho vật thể giả , chiếu qua gương, không xác định được ô tô |
| img_035 | xe ô tô chở đồ chồng cao | R11 | Chỉ gán box cho ô tô | không gán box cho đống đồ do đồ đó không thuộc class nào |
| img_038 | xe ô tô đồ chơi | R12 | Không gán box | không phải xe thật |

## 5. QC và phiên bản

Ngưỡng QC áp dụng, ai quyết ca mơ hồ, lịch sử thay đổi:

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| | | |
