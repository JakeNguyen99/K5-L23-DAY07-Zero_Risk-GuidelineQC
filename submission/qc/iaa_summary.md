# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 4
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 260 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 260 | 0.775 | 96.5% | 48.8% | 86.2% | 153/279 | 0 | CHƯA ĐẠT |
| 035|036 | 6 | 23 | 0.893 | 100.0% | 78.3% | 95.7% | 1/12 | 0 | ĐẠT |
| 035|052 | 9 | 47 | 0.664 | 95.7% | 34.0% | 76.6% | 16/178 | 0 | CHƯA ĐẠT |
| 035|055 | 5 | 41 | 0.874 | 97.6% | 43.9% | 85.4% | 2/29 | 0 | ĐẠT |
| 036|052 | 5 | 36 | 0.688 | 100.0% | 38.9% | 83.3% | 15/32 | 0 | CHƯA ĐẠT |
| 036|055 | 9 | 88 | 0.808 | 96.6% | 48.9% | 94.3% | 28/17 | 0 | CHƯA ĐẠT |
| 052|055 | 6 | 25 | 0.721 | 88.0% | 72.0% | 72.0% | 91/11 | 0 | CHƯA ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 756 ({'bo_sot_thua': 329, 'iou_thap': 148, 'sai_thuoc_tinh': 146, 'co_can_review': 10, 'sai_lop': 9, 'bat_dong_ignore': 114}), trên 33 ảnh cần calibration.
