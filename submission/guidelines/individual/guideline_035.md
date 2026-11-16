# Guideline gán nhãn — v0 — 035


- Phiên bản: v0
- Ngày: 2026-11-16
- Người viết / duyệt: 035

## 1. Mục tiêu
Nhãn dùng để huấn luyện mô hình phát hiện **người và phương tiện** trên ảnh đường phố
(ví dụ cho camera giao thông hoặc xe tự lái).

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

Vật **không thuộc lớp nào** thì làm gì (không gán? vùng ignore? lớp mới?).

### Thuộc tính
Mỗi thuộc tính: tên, **các mức cố định**, nghĩa từng mức, **ngưỡng cụ thể** (ví dụ che bao nhiêu % thì mức nào).
| `occluded_level` | `0`, `1`, `2`, `3` | Mức bị che: 0 là thấy rõ, 1 là bị che ít, 2 là bị che nhiều, 3 là không rõ. |
| `truncated` | `no`, `yes` | Vật bị cắt ở mép ảnh. |
| `can_review` | tick / không | Tick khi bạn không chắc, để người khác xem lại. |

## 3. Quy tắc vẽ box
1. Gán box cho mọi người và phương tiện trong ảnh.
2. Box ôm sát vật.
3. Chọn đúng lớp và thuộc tính.

## 4. Ca khó
| Ảnh | Tình huống | Quyết định | Lý do |
|---|---|---|---|
| img_004 | Xe giống motorcycle | Không gán nhãn | Xe có thể thuộc một class khác hoặc không thuộc guideline |
| img_011 | Xe giống motorcycle | Không gán nhãn | Xe này có thể chỉ là mẫu để chưng |
| img_014 | Xe và người bị nhòe | Không gán nhãn | |
| img_013 | `car` ở xa và mờ| Không gán nhãn | Guideline không đề cập vật phải nhìn rõ bao nhiêu %|
| img_015 | Xe bị nhòe | Không gán nhãn | Guideline không đề cập vật phải nhìn rõ bao nhiêu % |
| img_016 | Không rõ xe thuộc class nào | Không gán nhãn | Guideline không đề cập xe này thuộc class nào|
| img_017 | Không rõ tượng hình người thuộc class `person` | Không gán nhãn | Guideline không phân biệt người thật hay người giả |
| img_018 | Hình người trên biển quảng cáo | không gán nhãn | Guideline không phân biệt người thật hay người giả |
| img_019 | Trong hình có nhiều người | chỉ những người thấy rõ nguyên người| Guideline có class person|
| img_020 | Ảnh phản chiếu trong gương | Không gán nhãn | Guideline không đề cập |
| img_021 | Trong hình có nhiều người | chỉ những người thấy rõ nguyên người | Guideline có class person |
| img_022 | Xe motorcycle có khoang ngồi lắp bên cạnh | Không gán nhãn | Guideline không đề cập |
| img_023 | Xe xích lô| Không gán nhãn| Guideline không đề cập |
| img_024 | Xe mô hình| Không gán nhãn| Guideline không đề cập |



## 5. QC và phiên bản
Ngưỡng QC áp dụng, ai quyết ca mơ hồ, lịch sử thay đổi:

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| | | |
