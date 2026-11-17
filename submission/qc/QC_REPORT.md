# Báo cáo QC — Zero Risk

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Đặc tả và kiểm soát chất lượng (QC) nhãn 2D cho bài thực hành Day 07; tập ảnh tuyển chọn để luyện ca biên giao thông châu Á, chưa đánh giá độ đại diện cho dữ liệu vận hành diện rộng.
- Nguồn và phép sử dụng dữ liệu: Xem data/ATTRIBUTION.csv (40 ảnh nguồn mở CC BY / CC BY-SA từ Wikimedia Commons và Flickr).
- Manifest và checksum nhãn cuối: batch_manifest.json (SHA-256 nhãn cuối labels_calibrated.json: bc9c5fc8cd23414415396fc50d6bc823ee5da7d4b396bdcc64bdd69fc451537a).
- Guideline và taxonomy khi gán / khi bàn giao: Khi gán độc lập dùng guideline_v1.0_agreed.md và taxonomy.json (v1.0); khi bàn giao dùng guideline_v1.1_calibrated.md (hiệu chỉnh editorial làm rõ ca mép ảnh R-03, minibus R-19, ignore R-14) và giữ nguyên taxonomy.json v1.0.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80/80 lượt gán (40 ảnh, mỗi ảnh 2 người gán độc lập, 4 thành viên gán 035, 036, 052, 055).
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 260 cặp box ghép (mẫu số), IoU TB 0.775 (chưa đạt ngưỡng 0.85), đồng thuận lớp 96.5% (251/260, đạt ngưỡng 95%).
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 153/279 box chưa ghép (A/B), 0 bỏ qua do ignore, 114 ca bất đồng vùng ignore.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: img_034.jpg - người đội mũ xanh sát mép phải bị cắt một phần thân nhưng cả 052 (box #385) và 055 (box #190) đều để truncated=no; đối chiếu ảnh gốc và R-03 xác nhận cắt mép cần truncated=yes.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: 756 ca bất thường từ C001 đến C756 ghi trong submission/qc/calibration_decisions.csv, áp dụng các rule R-01 đến R-24, người chốt 052, người kiểm lại 035.
- Tác động v1.1 lên nhãn và ảnh: Phạm vi hiệu chỉnh 33 ảnh flagged, sửa nhãn img_034 truncated=yes, hạ cờ can_review, giữ nguyên taxonomy không cần mapping lại lớp.
- Tham chiếu bằng chứng khi Rework: Vòng 1 đạt tiêu chuẩn Release, không yêu cầu Rework; lưu vết hiệu chỉnh qua calibration_decisions.csv và labels_calibrated_79ac7e483d5f59be.json.

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: 035, 17/11/2026.
- Đã duyệt / cả lô: 40/40 ảnh đã duyệt / cả lô 40 ảnh (chế độ full).
- Ảnh lỗi / ảnh đã duyệt: 0/40 = 0.0% (đạt yêu cầu <= 2.0%).
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 0/692 = 0.0%.
- Vật bỏ sót, ảnh và vùng: 0 vật bỏ sót sau khi đã hiệu chỉnh và hợp nhất đồng thuận ở Pha 5.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Đã rà soát 33 ảnh ca khó (0/33 lỗi) và 7 ảnh còn lại (0/7 lỗi) theo phiếu submission/qc/reviews/round-01/qc_review.csv.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Máy chặn được box rỗng/lệch kích thước dưới 20px hoặc sai schema taxonomy; người đọc ảnh cần phát hiện đối tượng bị cắt mép tinh vi như người sát mép tại img_034 hoặc phân định minibus/van/bus theo cấu trúc cửa và khoang khách.

## 5. Quyết định bàn giao

- Quyết định: Release
- Lý do, người nhận và phạm vi sử dụng: Đạt toàn bộ tiêu chí QC lô (tỉ lệ lỗi ảnh 0.0% <= 2.0%, không còn cờ can_review, 100% ca bất thường có quyết định truy vết); bàn giao cho nhóm phát triển mô hình nhận diện phương tiện và người đi bộ trong khuôn khổ Day 07.
- Nếu còn việc: Không còn tồn đọng; toàn bộ hồ sơ bàn giao đã được đóng gói đầy đủ và kiểm tra toàn vẹn.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Ca C184 (img_009.jpg, minibus chốt bus theo R-19, box COCO #140 [349.88, 363.2, 318.74, 139.84]); ca C621 (img_034.jpg, người sát mép phải theo R-03, box COCO #621 [870.28, 154.56, 89.72, 301.99] truncated=yes).
