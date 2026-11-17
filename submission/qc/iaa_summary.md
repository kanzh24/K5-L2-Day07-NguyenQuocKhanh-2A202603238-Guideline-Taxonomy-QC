# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 5
- Lượt gán đủ **120/120**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 1032 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 120 | 1032 | 0.836 | 95.5% | 56.0% | 92.2% | 264/573 | 13 | CHƯA ĐẠT |
| 039|223 | 13 | 83 | 0.852 | 91.6% | 54.2% | 89.2% | 14/72 | 1 | CHƯA ĐẠT |
| 039|224 | 12 | 86 | 0.810 | 93.0% | 61.6% | 91.9% | 12/160 | 9 | CHƯA ĐẠT |
| 039|228 | 12 | 85 | 0.863 | 98.8% | 63.5% | 84.7% | 4/126 | 0 | ĐẠT |
| 039|238 | 11 | 75 | 0.836 | 94.7% | 66.7% | 78.7% | 5/54 | 0 | CHƯA ĐẠT |
| 223|224 | 13 | 136 | 0.836 | 97.8% | 52.9% | 95.6% | 29/48 | 2 | CHƯA ĐẠT |
| 223|228 | 11 | 109 | 0.896 | 98.2% | 42.2% | 96.3% | 7/25 | 0 | ĐẠT |
| 223|238 | 11 | 84 | 0.835 | 95.2% | 63.1% | 98.8% | 19/8 | 0 | CHƯA ĐẠT |
| 224|228 | 11 | 142 | 0.822 | 97.9% | 38.0% | 94.4% | 53/47 | 1 | CHƯA ĐẠT |
| 224|238 | 12 | 96 | 0.803 | 92.7% | 68.8% | 88.5% | 61/19 | 0 | CHƯA ĐẠT |
| 228|238 | 14 | 136 | 0.819 | 93.4% | 62.5% | 96.3% | 60/14 | 0 | CHƯA ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 2162 ({'co_can_review': 232, 'iou_thap': 378, 'sai_thuoc_tinh': 489, 'bo_sot_thua': 489, 'bat_dong_ignore': 528, 'sai_lop': 46}), trên 38 ảnh cần calibration.
