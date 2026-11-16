# Guideline gán nhãn — v0.1 (cá nhân) — 2A202603094 - Nguyễn Thị Thúy An

- Phiên bản: v0.1
- Loại nhãn: bounding box (hộp chữ nhật) trên ảnh đường phố
- Công cụ: CVAT, xuất COCO 1.0

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện **người và phương tiện thật** trên ảnh đường phố châu Á
(ví dụ cho camera giao thông hoặc xe tự lái).

## 2. Lớp và thuộc tính

### Lớp (mỗi vật thuộc đúng một lớp)

| Lớp | Định nghĩa |
|---|---|
| `car` | Ô tô con, taxi, SUV, xe điện kín 4 bánh (img_006, 012, 023, 040). Bán tải thùng hở là `truck`. |
| `truck` | Xe tải, đầu kéo, xe có thùng hàng riêng, bao gồm bán tải (img_002, 007, 036, 001, 012, 037). |
| `van` | Xe thùng kín cỡ nhỏ, minivan (img_005, 016). Xe khách nhiều ghế là `bus`. |
| `bus` | Các loại xe buýt, minibus chở khách (img_009, 031, 037, 040). |
| `motorcycle` | Xe máy, scooter, mô tô, quad, xe điện cưỡi; xe có sidecar tính một box (img_006, 008, 022). |
| `bicycle` | Xe đạp hai bánh (img_003, 006, 010). *Chưa chốt, cần họp:* bài học ví dụ "xe đạp không có trong taxonomy". |
| `three_wheeler` | Xe ba bánh: tuk-tuk, xe lam, ba gác máy, xích lô (img_004, 016, 035, 040, 023). |
| `person` | Người ở **ngoài** thân xe: đi bộ, đứng, ngồi, người cưỡi xe máy/xe đạp, người đánh xe (img_014, 034, 005, 026). Tượng, ma-nơ-canh, hình in không phải `person`. |
| `other_vehicle` | Xe thật không thuộc các lớp trên: xe trâu bò kéo, xe đẩy, xe hoạt hình (img_010, 019). Xe mô hình, đồ chơi không gán. |
| `ignore_region` | Vùng không cần gán từng vật: đám đông hoặc xe dày đặc, vật cao < 20 px, vật chỉ lộ vài điểm không xác định được class (img_021, 030, 032, 008). Vật tiền cảnh rõ vẫn gán. |

### Thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Mức bị che: 0 là thấy rõ (che ≤ 5%), 1 là bị che dưới 50%, 2 là bị che từ 50% trở lên nhưng vẫn xác định được class, 3 là không xác định được. |
| `truncated` | `no`, `yes` | Vật bị cắt ở mép ảnh: `yes` khi từ khoảng 5% diện tích vật nằm ngoài khung hình. |
| `can_review` | tick / không | Tick khi bạn không chắc class hoặc không chắc vật có thuộc phạm vi bài, để người khác xem lại. |

## 3. Quy tắc vẽ box

1. Gán box cho mọi người và phương tiện thật trong ảnh, mỗi vật một box riêng; các box được phép chồng lấn.
2. Box ôm sát phần nhìn thấy của vật, không quá rộng hoặc quá chặt; vật bị che chỉ bao phần thấy được.
3. Chọn đúng lớp và thuộc tính.
4. Xe chở hàng: hàng dính liền với xe thì một box bao cả hàng, vì xe tự lái phải tránh phần hàng. Xe máy chở hàng cùng quy tắc, lớp `motorcycle` (img_029, 035, 001, 012).
5. Xe chở xe (img_002): một box `truck` bao cả xe được chở, không gán từng xe trên đó (chưa chốt, cần họp).
6. Người trên xe: mỗi xe máy một `motorcycle`, mỗi người nhìn thấy rõ một `person`. Người ngồi trong khoang hoặc dưới mui (ô tô, tuk-tuk có mui) không gán.
7. Xe ba bánh: một box bao cả thân và mui. Xe trâu bò kéo: `other_vehicle`, một box bao cả xe và con vật kéo.
8. Dày đặc: xe nhìn thấy một phần rõ thì một box, `occluded_level` = 2; phần không xác định được thì gom vào một `ignore_region` (img_021, 030, 008, 032, 019).
9. Vật cao < 20 px không gán từng vật, gom vào `ignore_region` (img_013, 025, 039).
10. Vật bị cắt ở mép ảnh: box dừng ở mép ảnh, `truncated` = yes (img_005, 008, 012, 015, 016, 032, 034, 040).
11. Không gán: phản chiếu, hình ảnh trên biển quảng cáo, xe của chính mình (ego), và vật không thật (tượng, ma-nơ-canh, xe mô hình, đồ chơi, xe điêu khắc, bia mộ hình xe).
12. Thiếu sáng hoặc nhoè chuyển động: xác định được class thì gán (img_013, 031, 007); không thì `ignore_region` (img_014, 015).

## 4. Ca khó

Các ca đã có quy tắc ở mục 3 không nhắc lại.

| Ảnh | Tình huống | Quyết định | Lý do |
|---|---|---|---|
| img_002 | Đầu kéo chở 7 ô tô | Một box `truck` bao cả xe chở | Áp quy tắc hàng dính liền xe; *chưa chốt, cần họp* |
| img_019 | Xe hoạt hình diễu hành, tài xế ngồi trên xe mui trần | `other_vehicle`; tài xế `person`; tick `can_review` | Xe thật nhưng không thuộc lớp nào; *chưa chốt:* mui trần có tính "trong khoang" hay không |
| img_024 | Xe vàng đặt trên nóc xe đen | Xe đen `car`; xe vàng không gán | Không rõ vật thật hay mô hình; *chưa chốt, cần họp* |
| img_022 | Xe sidecar, ảnh đen trắng, nhỏ | `motorcycle`, một box gồm sidecar | *Chưa chốt, cần họp:* có tách sidecar hay không |

## 5. QC và phiên bản

- Mọi thay đổi làm đổi nhãn đều tăng phiên bản guideline.
- Ca mơ hồ: tick `can_review`, không tự đoán.
- Lỗi khi QC gồm 5 loại: bỏ sót, thừa, sai lớp, sai vị trí hoặc kích thước, sai thuộc tính.
- Ngưỡng đạt: IoU ≥ 0,85 (vật ≥ 50 px), ≥ 0,7 (vật 20–50 px); IAA: IoU trung bình ≥ 0,85, đồng thuận lớp ≥ 95%; Release khi ≤ 2% số ảnh có lỗi.
- v0 → v0.1: thêm `bus`, `bicycle`, `three_wheeler`, `other_vehicle`; ngưỡng occluded/truncated; quy tắc xe chở hàng, người trên xe, dày đặc, vật không thật.
