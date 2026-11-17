# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 4
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 276 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 276 | 0.836 | 94.9% | 56.5% | 94.2% | 35/147 | 9 | CHƯA ĐẠT |
| 101|102 | 6 | 24 | 0.862 | 91.7% | 66.7% | 87.5% | 3/6 | 0 | CHƯA ĐẠT |
| 101|103 | 9 | 56 | 0.857 | 85.7% | 53.6% | 100.0% | 3/42 | 6 | CHƯA ĐẠT |
| 101|104 | 5 | 34 | 0.829 | 100.0% | 47.1% | 91.2% | 2/35 | 0 | CHƯA ĐẠT |
| 102|103 | 5 | 36 | 0.838 | 94.4% | 61.1% | 88.9% | 11/8 | 0 | CHƯA ĐẠT |
| 102|104 | 9 | 75 | 0.826 | 100.0% | 54.7% | 92.0% | 11/34 | 0 | CHƯA ĐẠT |
| 103|104 | 6 | 51 | 0.819 | 96.1% | 60.8% | 100.0% | 5/22 | 3 | CHƯA ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 466 ({'co_can_review': 4, 'iou_thap': 110, 'sai_thuoc_tinh': 128, 'bo_sot_thua': 136, 'bat_dong_ignore': 74, 'sai_lop': 14}), trên 35 ảnh cần calibration.
