# Pha 4 — Kết quả và việc cần rà soát

Bản phân tích của 036, commit c086df4. Đã chạy validate_labels.py --all, qc_compare.py và viz_disagreement.py --images flagged. Bản cục bộ 052 được đổi tên danh mục three_wheel thành three_wheeler cho 8 box, giữ nguyên tọa độ và thuộc tính. Các cảnh báo box nhỏ vẫn cần rà bằng mắt.

## Kết quả

- Đủ 80/80 lượt trên 40 ảnh, 4 người. Ghép một với một từ IoU 0,50, chưa xét lớp khi ghép. IoU trung bình 0,775 và đồng thuận lớp 96,5% (251/260) tính trên 260 cặp ghép. Ngưỡng tương ứng là 0,85 và 95%. Kết quả chung chưa đạt vì IoU.
- So với lần chạy trước ở bản 90c6ba7, đồng thuận lớp tăng 93,7% → 96,5%, số ca giảm 795 → 756 và ảnh bị gắn cờ giảm 38 → 33. IoU giảm 0,777 → 0,775. Hai lần có mẫu số khác nhau.
- 756 ca gồm 329 ca bỏ sót/thừa, 148 ca IoU thấp, 146 ca lệch thuộc tính, 114 ca bất đồng ignore, 10 cờ review và 9 ca lệch lớp. Một box có thể sinh nhiều ca.
- Box chưa ghép không vào mẫu số 260. Hai người cũng có thể cùng bỏ sót hoặc cùng gán sai. IAA chỉ đo đồng thuận, không xác nhận mọi box đúng theo guideline.

## Ca đã đối chiếu với ảnh

| Ca | Người | Đối tượng và nhận xét |
|---|---|---|
| C376 · img_021 | 035 / 052 | Xe buýt ở nền phía trên bên phải. Chỉ 035 có box cho vùng này. Rà phần xe thấy trực tiếp và ngưỡng 20 px trước khi kết luận 052 bỏ sót. |
| C460 · img_021 | 035 / 052 | Ô tô trắng bên trái, bị người và xe phía trước che. Hai box cùng lớp nhưng IoU 0,59. Đối chiếu các phần thân còn thấy và mép cắt, rồi mới giao người chỉnh biên theo R-01. |
| C288 · img_019 | 052 / 055 | 055 khoanh một vùng ignore lớn qua hàng khán giả, 052 tách nhiều person. Quy tắc ca ảnh cho phép ignore ở cụm dày, còn R-14 yêu cầu giữ người tách được. Nhóm cần chỉ rõ phần nào của cụm còn tách được. |
| C184 · img_009 | 035 / 052 | Minibus giữa đường. Bản 035 là bus, bản 052 là car. Guideline ca img_009 và R-19 chọn bus. Đề xuất giao 052 sửa lớp trong calibration. |

## Kiểm chứng ca hai người cùng sai

**img_034.jpg — người đội mũ xanh đi xe máy sát mép phải.** Cả 052 (box COCO #385) và 055 (box COCO #190) đều chọn person và truncated=no. Ảnh gốc cho thấy mép phải cắt mất một phần người. R-03 và dòng img_034 trong guideline yêu cầu truncated=yes cho người này. Cả hai đồng thuận về lớp và cùng ghi sai thuộc tính. Chuyển quyết định sửa thuộc tính vào bản calibration.

Bằng chứng: work/viz/img_034.png trong gói ZIP. Hai box ở ngoài cùng bên phải, tâm theo ảnh gốc lần lượt khoảng (921, 302) px và (909, 309) px.

Đã đối chiếu trực tiếp với ảnh và guideline nhóm. COACH_GUIDE.md có các ảnh gợi ý khác. Chưa có bộ tham chiếu riêng do coach cung cấp, vì vậy ca này là bằng chứng tự kiểm của nhóm.

## Phân công rà soát 40 ảnh

Bảng sau dùng mã ca của riêng lần chạy c086df4. Xem chi tiết vị trí box trong bang_ca_co_vi_tri.csv. Ảnh có 0 ca vẫn cần kiểm nếu dùng để tìm lỗi cùng bỏ sót.

| Ảnh | Người đối chiếu | Số ca | Việc ưu tiên | Quy tắc |
|---|---|---:|---|---|
| img_001.jpg | 036 / 052 | 14 | biên box 6, thuộc tính 6, box chưa ghép 2 | R-20, R-03, R-13, R-11 |
| img_002.jpg | 035 / 055 | 5 | biên box 4, box chưa ghép 1 | R-10, R-02 |
| img_003.jpg | 036 / 055 | 47 | thuộc tính 18, biên box 15, box chưa ghép 13, cờ review 1 | R-04, R-12, R-13, R-24 |
| img_004.jpg | 035 / 052 | 7 | biên box 3, box chưa ghép 2, thuộc tính 2 | R-05, R-12, R-18 |
| img_005.jpg | 036 / 052 | 24 | box chưa ghép 14, thuộc tính 6, biên box 4 | R-02, R-03, R-04, R-23 |
| img_006.jpg | 035 / 055 | 28 | thuộc tính 11, box chưa ghép 8, biên box 6, cờ review 3 | R-02, R-03, R-04, R-14, R-24 |
| img_007.jpg | 036 / 052 | 3 | thuộc tính 2, biên box 1 | R-20, R-10, R-03, R-16 |
| img_008.jpg | 035 / 055 | 31 | box chưa ghép 18, thuộc tính 9, cờ review 3, biên box 1 | R-04, R-14, R-16 |
| img_009.jpg | 035 / 052 | 28 | box chưa ghép 18, biên box 6, thuộc tính 3, lớp 1 | R-19, R-03, R-13 |
| img_010.jpg | 036 / 055 | 8 | box chưa ghép 4, biên box 2, thuộc tính 2 | R-04, R-11, R-24 |
| img_011.jpg | 036 / 052 | 40 | box chưa ghép 26, biên box 9, thuộc tính 5 | R-06, R-19, R-13 |
| img_012.jpg | 035 / 055 | 3 | thuộc tính 3 | R-20, R-11, R-03 |
| img_013.jpg | 036 / 055 | 10 | thuộc tính 4, lớp 3, box chưa ghép 2, biên box 1 | R-13, R-14, R-16, R-08 |
| img_014.jpg | 035 / 052 | 11 | vùng ignore 9, biên box 2 | R-15, R-04 |
| img_015.jpg | 035 / 036 | 0 | Kiểm quy tắc và lỗi cùng bỏ sót | R-03, R-15, R-08 |
| img_016.jpg | 052 / 055 | 17 | biên box 9, thuộc tính 6, lớp 2 | R-03, R-04, R-05, R-18 |
| img_017.jpg | 036 / 055 | 0 | Kiểm quy tắc và lỗi cùng bỏ sót | R-06, R-22 |
| img_018.jpg | 035 / 052 | 11 | biên box 5, thuộc tính 5, box chưa ghép 1 | R-07, R-02, R-03 |
| img_019.jpg | 052 / 055 | 75 | vùng ignore 75 | R-17, R-05, R-06, R-14 |
| img_020.jpg | 035 / 036 | 0 | Kiểm quy tắc và lỗi cùng bỏ sót | R-08, R-09, R-22 |
| img_021.jpg | 035 / 052 | 106 | box chưa ghép 84, vùng ignore 13, thuộc tính 5, biên box 4 | R-04, R-13, R-14, R-03 |
| img_022.jpg | 036 / 055 | 2 | biên box 1, thuộc tính 1 | R-21, R-13 |
| img_023.jpg | 036 / 055 | 17 | box chưa ghép 7, biên box 5, thuộc tính 3, cờ review 2 | R-04, R-05, R-18 |
| img_024.jpg | 035 / 052 | 18 | box chưa ghép 11, thuộc tính 4, biên box 3 | R-10, R-17, R-02, R-03 |
| img_025.jpg | 035 / 052 | 39 | box chưa ghép 15, biên box 9, vùng ignore 8, thuộc tính 6, lớp 1 | R-13, R-04, R-03, R-12 |
| img_026.jpg | 036 / 055 | 0 | Kiểm quy tắc và lỗi cùng bỏ sót | R-04, R-03, R-07, R-16 |
| img_027.jpg | 052 / 055 | 1 | box chưa ghép 1 | R-06, R-22 |
| img_028.jpg | 035 / 036 | 0 | Kiểm quy tắc và lỗi cùng bỏ sót | R-20, R-11, R-06, R-02 |
| img_029.jpg | 036 / 055 | 12 | thuộc tính 7, biên box 4, box chưa ghép 1 | R-04, R-11, R-13, R-24 |
| img_030.jpg | 035 / 052 | 34 | box chưa ghép 22, vùng ignore 4, biên box 4, thuộc tính 4 | R-14, R-02, R-03 |
| img_031.jpg | 035 / 052 | 22 | box chưa ghép 15, biên box 5, thuộc tính 2 | R-19, R-08, R-16, R-13 |
| img_032.jpg | 036 / 055 | 39 | box chưa ghép 18, thuộc tính 11, biên box 10 | R-14, R-13, R-03, R-12 |
| img_033.jpg | 035 / 036 | 0 | Kiểm quy tắc và lỗi cùng bỏ sót | R-06, R-08, R-09, R-03 |
| img_034.jpg | 052 / 055 | 36 | box chưa ghép 25, biên box 9, thuộc tính 2 | R-04, R-02, R-03, R-13 |
| img_035.jpg | 036 / 052 | 21 | biên box 9, thuộc tính 7, box chưa ghép 5 | R-04, R-05, R-11, R-18 |
| img_036.jpg | 035 / 055 | 8 | box chưa ghép 4, thuộc tính 2, biên box 1, lớp 1 | R-14, R-02, R-03 |
| img_037.jpg | 035 / 036 | 2 | box chưa ghép 1, thuộc tính 1 | R-20, R-19, R-03 |
| img_038.jpg | 052 / 055 | 0 | Kiểm quy tắc và lỗi cùng bỏ sót | R-06, R-22 |
| img_039.jpg | 035 / 036 | 24 | box chưa ghép 9, vùng ignore 5, biên box 5, thuộc tính 4, cờ review 1 | R-13, R-14, R-24 |
| img_040.jpg | 052 / 055 | 13 | biên box 5, thuộc tính 5, box chưa ghép 2, lớp 1 | R-03, R-04, R-05, R-18, R-19 |

## Bàn giao

Giữ đầu ra Pha 4 làm mốc. Trong Pha 5, nhóm xem ảnh, chốt cách xử lý và người sửa trong calibration_decisions.csv, sau đó kiểm lại. Không dùng kết quả IAA để tự chọn ai đúng trong các ca còn mơ hồ.
