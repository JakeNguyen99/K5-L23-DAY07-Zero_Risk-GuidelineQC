# Guideline gán nhãn — v0.1 (cá nhân) — Loan

- Mã thành viên: <điền 3 chữ số>
- Phiên bản: v0.1 (cá nhân)
- Ngày: <điền>

## 1. Mục tiêu

- Nhãn dùng để huấn luyện mô hình phát hiện người và phương tiện 2D trên ảnh đường phố châu Á (camera giao thông, xe tự lái).
- Bỏ sót người hoặc xe thật là lỗi nguy hiểm nhất.
- Gán vật không thật (tượng, ma-nơ-canh, đồ chơi, hình in, phản chiếu) dạy mô hình báo có người/xe khi không có.
- Gán nhầm lớp (xe tải thành xe con) làm sai ước lượng kích thước và hành vi.
- Hai người đọc cùng một quy tắc phải ra cùng một kết quả: chỗ nào phụ thuộc cảm giác thì thay bằng ngưỡng đo được hoặc mã quy tắc R-xx.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con chở người: sedan, hatchback, SUV, taxi con. | img_023, img_012 (SUV xám), img_040 (taxi), img_024 (xe cổ trưng bày), img_019 (xe vàng giả tưởng có biển số) | Bán tải: car hay truck chưa chốt (R-20). Xe đồ chơi, mô hình không gán (R-06). Người ngồi bên trong xe không gán (R-05). |
| `truck` | Xe tải, xe có thùng hàng riêng, xe chở xe. | img_002 (xe chở ô tô), img_036 (xe tải nhỏ bạt xanh), img_007 (Isuzu), img_001 | Xe thùng liền cabin là van. Bán tải: xem R-20 (chưa chốt). |
| `van` | Xe thùng kín cỡ nhỏ, cabin và khoang liền một khối. | img_005 (van xám), img_016 (van trắng bên phải) | Xe buýt, minibus cỡ lớn không thuộc lớp này (R-19). Bán tải gắn nắp thùng cao: car hay van chưa chốt (img_037). |
| `motorcycle` | Xe hai bánh có động cơ: xe máy, xe tay ga, mô tô, kể cả xe đỗ không người. | img_006, img_008, img_029, img_039 (xe đỗ) | Xe đạp, xích lô đạp, xe ba bánh (xe lam, tuk-tuk) không gán (R-12, R-18). Xe máy kim loại trong tác phẩm điêu khắc (img_011) không gán (R-06). |
| `person` | Người thật đi bộ, đứng, hoặc ngồi trên xe máy, xe đạp, xích lô. | img_003, img_010, img_014, img_034, img_023 (người ngồi xích lô) | Tượng, ma-nơ-canh, linh vật, hình in không gán (R-06, R-07). Người ngồi trong ô tô hoặc xe lam không gán (R-05). |
| `ignore_region` | Vùng không gán từng vật: cụm dày không tách được hoặc cụm vật cao dưới 20 px. | img_021, img_019 (đám đông), img_030, img_032 | Không dùng để bỏ qua vật to, rõ ràng, trừ xe buýt nếu nhóm chọn phương án B (R-19). |

Vật không thuộc lớp nào (xe đạp, xích lô đạp, xe ba bánh như xe lam/tuk-tuk, xe buýt, xe đẩy, nón giao thông, biển báo, ghế): không gán box (phương án B); người đi hoặc ngồi trên xe đạp, xích lô vẫn gán person (R-12, R-18, R-19). Cụm vật dày hoặc dưới 20 px: ignore_region (R-13, R-14).

### Thuộc tính

| Thuộc tính | Mức | Nghĩa và ngưỡng | Ghi chú |
|---|---|---|---|
| `occluded_level` | `0` | Không bị che hoặc che dưới 10% diện tích vật (ước tính cả vật). | Chỉ tính che bởi vật khác (cột, xe khác, người khác, hàng rào). Người ngồi trên xe KHÔNG tính là che xe đó. |
| `occluded_level` | `1` | Che từ 10% đến 40%. | Che nhẹ, vẫn thấy rõ hình dạng. |
| `occluded_level` | `2` | Che từ trên 40% đến 80%. | Thấy phần lớn đặc trưng nhận dạng. |
| `occluded_level` | `3` | Che trên 80%, vẫn nhận ra lớp nhờ phần còn lại. | Không đoán chắc lớp thì KHÔNG gán (R-02). |
| `truncated` | `no` | Vật nằm trọn trong ảnh. | Cắt mép ảnh KHÔNG tính vào occluded_level. |
| `truncated` | `yes` | Phần vật nằm ngoài mép ảnh từ 10% trở lên (ước tính). | Box vẽ tới sát mép ảnh (R-03). |
| `can_review` | `tick / không` | Tick khi chưa chắc lớp, mức che, hoặc có nên gán không. | Tick khi vật nhòe, người ở xa mờ, hoặc ca chưa chốt (R-15, R-23). |

## 3. Quy tắc vẽ box

| Rule ID | Nhóm | Quy tắc | Ảnh ví dụ | Lý do |
|---|---|---|---|---|
| **R-01** | Vẽ box | Box ôm sát phần NHÌN THẤY của vật, không ngoại suy phần bị che hoặc nằm ngoài ảnh. Cạnh box chạm điểm ngoài cùng thấy được. | img_018, img_005 | Mọi người cùng một cách vẽ thì IoU cao; không ai phải đoán phần khuất. |
| **R-02** | Che khuất | Mức che theo bảng thuộc tính (ngưỡng 10/40/80%). Che nhiều mà không đoán chắc lớp thì không gán. Lưỡng lự giữa hai mức: chọn mức cao hơn, tick can_review. Vật che chồng nhau (ô tô chồng ô tô): vẽ box riêng, vật phía sau ghi mức che theo %. | img_005, img_006, img_024, img_028, img_034, img_036 | Ngưỡng % đo được giúp hai người chọn cùng một mức. |
| **R-03** | Cắt mép | Vẽ box tới sát mép ảnh, truncated = yes nếu phần ngoài ảnh từ 10% trở lên. Phần thấy dưới 20% vật và không đoán chắc lớp thì không gán. Tách rõ cắt mép (truncated) với bị vật khác che (occluded_level). | img_005, img_006, img_015, img_016, img_040 | Một vật có thể vừa cắt mép vừa bị che; hai thuộc tính độc lập. |
| **R-04** | Người và xe máy | Hai box chồng nhau: person (đầu/mũ tới phần thân thấy được, gồm ba lô; không gồm xe) và motorcycle (hình chữ nhật bao các phần xe thấy được từ bánh trước tới bánh sau; không gồm thân người phía trên tay lái). Xe chở hai người: 1 motorcycle + 2 person. Người ngồi trên xe không làm tăng occluded_level của xe. Nếu chỉ còn lộ gương/tay lái (dưới 20% xe): chỉ gán person. | img_005, img_006, img_010, img_026, img_034 | Một cách đếm duy nhất cho cảnh phổ biến nhất của bộ ảnh. |
| **R-05** | Người trong xe | Người ngồi trong ô tô (kín hoặc hở mui) và người ngồi trong xe lam/tuk-tuk: không gán. Người ngồi trên xe máy, xe đạp, xích lô: gán person. | img_004, img_016, img_019, img_040, img_023 | Nhất quán với việc xe lam không có lớp; người trong ô tô không phải mục tiêu. |
| **R-06** | Vật không thật | Tượng, ma-nơ-canh, mô hình, đồ chơi, linh vật nhồi bông, xe điêu khắc: không gán dù giống người/xe. | img_011, img_017, img_019, img_027, img_028, img_033, img_038 | Gán vật không thật dạy mô hình báo có người/xe khi không có. |
| **R-07** | Hình in | Mặt người, xe in trên biển quảng cáo, tranh tường, poster: không gán. | img_018, img_026 | Không phải vật thật trong cảnh. |
| **R-08** | Phản chiếu | Hình phản chiếu trong gương, kính, mặt đường ướt: không gán. | img_015, img_020, img_031, img_033 | Không phải vật thật tại vị trí đó. |
| **R-09** | Xe chủ | Bộ phận của xe mang camera (gương chiếu hậu, nắp capo, kính) không gán. | img_020, img_033 | Không phải đối tượng cần phát hiện. |
| **R-10** | Xe chở xe | Xe chở xe: 1 box truck; ô tô trên thùng xác định được thì gán car (chưa chốt, cần họp). Ô tô nằm trên nóc ô tô khác: 2 box riêng. Thùng hàng/rơ-moóc phía sau xe: không gán riêng. | img_002, img_024, img_007 | Cả xe chở và xe được chở đều là vật thật trên đường. |
| **R-11** | Hàng hóa | Hàng chở trên xe (bó hàng, thùng, thanh dài, bao tải) không đưa vào box xe. Box chỉ ôm thân xe và người. | img_012, img_029, img_035 | Hàng nhô ra làm box dao động; loại ra để các box ổn định. |
| **R-12** | Vật không có lớp | Xe đạp, xích lô đạp, xe đẩy, nón giao thông, biển báo, ghế: không có lớp nên không gán box (phương án B). Người ngồi/đi trên xe đạp, xích lô vẫn gán person. | img_003, img_006, img_023, img_029, img_039 | v0 không có lớp xe đạp/xích lô; tránh gán nhầm thành motorcycle. |
| **R-13** | Vật nhỏ | Vật cao dưới 20 px: không gán từng vật, gộp vào ignore_region nếu thành cụm. Người ở xa từ 20 px trở lên và nhận ra được: gán, tick can_review; không nhận ra thì không gán. | img_001, img_013, img_025, img_039 | Khớp ngưỡng min_box_height_px = 20 của taxonomy. |
| **R-14** | Cụm dày | Từ 5 vật sát nhau trở lên và không tách được ranh giới (hoặc đều dưới 20 px): một ignore_region ôm cả cụm. Vật to, rõ ở phía trước vẫn gán riêng. | img_008, img_019, img_021, img_030, img_032, img_036 | Bỏ sót trong vùng này không tính là lỗi nên phải dùng đúng chỗ. |
| **R-15** | Mờ nhòe | Vẫn nhận ra lớp: gán, tick can_review vì ranh giới mờ. Chỉ còn vệt mờ không nhận ra lớp: không gán. | img_014, img_015 | Tránh đoán lớp từ vệt mờ. |
| **R-16** | Tối, ngược sáng | Gán vật thấy rõ lớp. Vùng tối không đoán được thì không gán, tick can_review nếu lưỡng lự. | img_007, img_013, img_026, img_031 | Giữ nhãn đáng tin thay vì đoán. |
| **R-17** | Xe thật hình lạ | Phương tiện thật đang chạy hoặc được trưng bày (xe cổ, xe giả tưởng có biển số) vẫn gán theo lớp gần nhất. | img_019, img_024 | Là phương tiện thật trong cảnh. |
| **R-18** | Xe ba bánh | Xe ba bánh có động cơ (xe lam, tuk-tuk, auto-rickshaw): v0 không có lớp nên không gán box (phương án B); người ngồi trong xe không gán. Xe máy hai bánh vẫn là motorcycle. Chưa chốt, cần họp (có thể đề xuất thêm lớp). | img_004, img_016, img_035 | Dễ gán nhầm thành motorcycle hoặc car. |
| **R-19** | Xe buýt | Xe buýt, minibus: v0 không có lớp nên không gán box từng xe; xe to có thể phủ ignore_region (phương án B) hoặc đề xuất thêm lớp bus. Chưa chốt, cần họp. | img_009, img_011, img_031, img_037, img_040 | Người gán dễ chọn truck. |
| **R-20** | Bán tải | Bán tải (cabin như ô tô con, thùng hở phía sau): car hay truck chưa chốt, cần họp. Tạm theo nhận định ban đầu: img_001 truck; img_007, img_012, img_028, img_037 car. Thùng và hàng phía sau không gán riêng. | img_001, img_007, img_012, img_028, img_037 | Bán tải nằm giữa hai lớp: car là ô tô con, truck là xe tải. |
| **R-21** | Xe có sidecar | Xe có sidecar (img_022) không chắc car hay motorcycle: gán theo lớp nhìn rõ nhất (motorcycle, ôm cả sidecar), tick can_review, đưa ra họp (chưa chốt). | img_022 | Không xác định được lớp chắc chắn. |
| **R-22** | Ảnh trống | Ảnh không có vật nào thuộc phạm vi: để trống, vẫn tính là đã gán. | img_017, img_020, img_027, img_033, img_038 | Ảnh không có vật cần gán là kết quả hợp lệ. |
| **R-23** | Chưa chắc | Tick can_review trên box. Vật chưa vẽ được hoặc cả ảnh mơ hồ: ghi vào submission/qc/image_issues.csv (image, annotator, region, note). Không hỏi đồng đội khi đang gán. | mọi ảnh | Giữ gán độc lập để IAA có ý nghĩa. |

## 4. Ca khó

| Ảnh | Tình huống | Loại ca khó | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|---|
| img_001 | Quán bar vỉa hè: xe bán tải trắng cũ bị cắt mép, người đứng cạnh xe, người ngồi quán ở xa mờ. | cat_mep | R-03, R-13, R-20 | Truck: 1 box, truncated=yes. Người đứng cạnh xe: person. Người xa: gán nếu cao từ 20 px và nhận ra được, kèm can_review; không nhận ra thì không gán. | v0 có lớp truck nhưng không có ngưỡng truncated và không nói xử lý người xa mờ. |
| img_002 | Xe chở ô tô, các ô tô trên thùng vẫn xác định được. | xe_cho_xe | R-10 | Xe chở: 1 box truck. Ô tô trên thùng xác định được: gán car. (chưa chốt, cần họp) | v0 không nói vật đang được chở có gán hay không; hai người dễ gán khác nhau. |
| img_003 | Người gánh quang gánh, người đứng cạnh xe đạp, xe máy phía sau. | ngoai_taxonomy | R-12, R-13 | Người: person; xe máy: motorcycle. Xe đạp: không có lớp nên không gán box, người đạp xe vẫn gán person. (chưa chốt, cần họp) | v0 không có lớp bicycle; dễ bị gán nhầm thành motorcycle. |
| img_004 | Hai xe lam (tuk-tuk) xanh, có người ngồi trong xe. | ngoai_taxonomy | R-18, R-05 | Xe lam: không có lớp nên không gán box. Người ngồi trong xe lam: không gán. (chưa chốt, cần họp) | v0 không có lớp xe ba bánh; người trong xe lam không lấy. |
| img_005 | Van xám phía sau; nhiều người ngồi trên xe máy sát ống kính, che nhau; xe máy cắt mép. | nguoi_tren_xe | R-04, R-02, R-03 | Người trên xe: 1 box person + 1 box motorcycle. Xe máy cắt mép: truncated=yes. Người bị che: occluded_level theo ngưỡng 10/40/80%. Van xám: van. | v0 không nói người trên xe gán mấy box, không có ngưỡng % che. |
| img_006 | Ngã tư đông xe máy, người đạp xe bên phải, ô tô taxi cắt mép và bị che. | che_khuat | R-04, R-02, R-03, R-12 | Người trên xe: person + box xe riêng. Car: truncated=yes, occluded_level theo ngưỡng %. Người đạp xe: person, không gán xe đạp. | Ảnh có cùng lúc cắt mép và bị che: phải tách truncated (mép ảnh) với occluded (vật khác che). |
| img_007 | Bán tải trắng (thùng hàng phía sau) và xe tải Isuzu cắt mép trong hẻm kho tối. | lop_mo_ho | R-20, R-10, R-03, R-16 | Truck (Isuzu) cắt mép: truncated=yes. Bán tải: tạm gán car, thùng hàng phía sau không gán riêng. (chưa chốt, cần họp) | v0 không nói xe kéo rơ-moóc/thùng hàng là car hay truck. |
| img_008 | Bãi xe máy ban đêm, người ngồi trên xe. | nguoi_tren_xe | R-04, R-14 | Người: 1 box person; xe: 1 box riêng theo lớp tương ứng; cụm xe dày phía sau: ignore_region. | Vật thật, quy tắc người–xe áp dụng thẳng. |
| img_009 | Xe buýt (minibus) giữa đường, xe máy cắt mép, người ở xa. | ngoai_taxonomy | R-19, R-03, R-13 | Xe buýt: không có lớp nên không gán box (phương án B) / ignore_region nếu to, hoặc đề xuất lớp bus. Xe máy: truncated=yes. Người xa: theo ngưỡng 20 px. (chưa chốt, cần họp) | v0 không có lớp bus; người gán dễ chọn truck. |
| img_010 | Chợ vỉa hè, người ngồi trên xe máy, người bán rau, xe đạp chở rau. | nguoi_tren_xe | R-04, R-12 | Mỗi xe: 1 box person + 1 box motorcycle; person cho người bán rau; không gán xe đạp và hàng rau. | v0 không nói người trên xe gán mấy box. |
| img_011 | Tượng lớn và xe máy kim loại ở hội chợ; xe lớn bị che ở nền; người ở xa nhỏ. | ngoai_taxonomy | R-06, R-19, R-13 | Tượng và xe máy kim loại: không gán. Xe lớn bị che (ghi chú gốc: xe buýt): xử lý như img_009. Người cao dưới 20 px: không gán từng người, gộp ignore_region nếu thành cụm. (chưa chốt, cần họp) | Hai vấn đề: lớp bus không có, và người quá nhỏ. |
| img_012 | Bán tải trắng thùng hở chở hàng (cắt mép), SUV xám bên trái. | lop_mo_ho | R-20, R-11, R-03 | SUV: car. Bán tải: tạm gán car, truncated=yes; hàng trên thùng không vào box. (chưa chốt, cần họp) | v0: car là ô tô con, truck là xe tải — bán tải nằm giữa hai lớp. |
| img_013 | Phố đêm: người, ô tô, nhiều xe ở xa. | nho_dong | R-13, R-14, R-16 | Xe xa cao từ 20 px và tách được: box riêng; nhỏ hơn hoặc chen dày: 1 ignore_region. Người từ 20 px: person. | v0 không có ngưỡng vật nhỏ, không nói khi nào dùng ignore_region. |
| img_014 | Người đi bộ rõ nét; xe máy phía sau nhòe do chuyển động. | nhoe_toi | R-15, R-04 | Vẫn nhận ra người và xe: gán, tick can_review vì ranh giới mờ. Không nhận ra: không gán. | v0 không nói xử lý vật bị nhòe do chuyển động. |
| img_015 | Phố mưa: ô tô cắt mép hai bên, xe nhòe ở xa, phản chiếu trên đường ướt. | nhoe_toi | R-03, R-08, R-15 | Car cắt mép: truncated=yes. Car nhòe nhưng nhận ra: gán + can_review. Không gán phản chiếu. | Như img_014. |
| img_016 | Xe lam cắt mép, ô tô cắt mép, người trên xe máy, người cắt mép. | ngoai_taxonomy | R-18, R-03, R-04, R-05 | Xe lam: như img_004 (không gán box, người trong xe không gán). Car, người cắt mép: truncated=yes. Người trên xe máy: person riêng + motorcycle. Van trắng: van. (chưa chốt, cần họp) | Lặp lại ca xe lam của img_004, thêm cắt mép. |
| img_017 | Tượng người cưỡi ngựa có nón giao thông (ghi chú gốc: ma-nơ-canh). | khong_that | R-06, R-22 | Không gán (không phải người thật). Ảnh để trống. | v0 ghi 'gán mọi người' nên dễ gán nhầm tượng/ma-nơ-canh thành person. |
| img_018 | Biển quảng cáo mặt người; ô tô đỗ sau hàng rào, có xe cắt mép. | cat_mep | R-07, R-03, R-02 | Car: box ôm sát; xe cắt mép truncated=yes; ghi mức che do hàng rào. Không gán mặt trên biển. | Lớp và thuộc tính đều có trong v0, ca đơn giản. |
| img_019 | Xe vàng giả tưởng có người bên trong, chuột nhồi bông; đám đông chen lấn sau hàng rào. | nho_dong | R-17, R-05, R-06, R-14 | Car cho xe vàng. Người trong xe: không gán (nhất quán với img_004). Không gán chuột nhồi bông. Đám đông không tách được: 1 ignore_region. (chưa chốt, cần họp) | Cần chốt chung quy tắc người ngồi trong xe. |
| img_020 | Hình ô tô phản chiếu trong gương chiếu hậu xe chủ. | khong_that | R-08, R-09, R-22 | Phản chiếu: không gán. Ảnh để trống. | Không phải vật thật; v0 không nói về phản chiếu. |
| img_021 | Kẹt xe máy, nhiều người ngồi trên xe, đám đông nhỏ ở xa, ô tô cắt mép. | nho_dong | R-04, R-14, R-13, R-03 | Người trên xe: person + motorcycle. Cụm người nhỏ dưới 20 px: 1 ignore_region. Car: truncated=yes. | v0 không có quy tắc vật nhỏ/đám đông. |
| img_022 | Ảnh đen trắng cổ: xe có sidecar, người ngồi trong; không rõ car hay motorcycle. | lop_mo_ho | R-21, R-13 | Không chắc lớp: gán theo lớp nhìn rõ nhất (motorcycle, ôm cả sidecar) + tick can_review, đưa ra họp. (chưa chốt, cần họp) | Không xác định được car hay motorcycle. |
| img_023 | Xích lô, ô tô, người trên xe máy. | ngoai_taxonomy | R-12, R-04 | Xích lô: không có lớp nên không gán box; người ngồi trên xích lô vẫn gán person. Người trên xe máy: person + motorcycle. Ô tô: car. (chưa chốt, cần họp) | v0 không có lớp xích lô/xe ba bánh không động cơ. |
| img_024 | Ô tô vàng nhỏ nằm chồng trên ô tô đen trong bảo tàng; nhiều xe cổ cắt mép. | che_khuat | R-10, R-17, R-02, R-03 | Hai ô tô chồng nhau: vẽ 2 box riêng (box được chồng lên nhau), xe phía sau có occluded_level theo %. Xe cắt mép: truncated=yes. | Chồng lên nhau vẽ 2 box. |
| img_025 | Bờ biển: người đi bộ, người ở xa nhỏ, ô tô cắt mép, người trên xe máy. | nho_dong | R-13, R-04, R-03 | Người xa dưới 20 px: ignore_region. Người trên xe máy: person + motorcycle. Car cắt mép: truncated=yes. | Tổ hợp nhiều quy tắc chung, không có tình huống mới. |
| img_026 | Hai người trên một xe máy, xe cắt mép, trước tranh tường. | nguoi_tren_xe | R-04, R-07, R-03 | 2 box person + 1 box motorcycle (truncated=yes). Không gán tranh tường. | v0 không nói số box khi nhiều người trên một xe. |
| img_027 | Xe bán tải bằng đá granite trên mộ (mô hình). | khong_that | R-06, R-22 | Mô hình xe: không gán. Ảnh để trống. | Không tham gia giao thông; v0 không nói về mô hình/tượng. |
| img_028 | Bán tải cổ chở tượng ngựa trắng; ô tô đỏ phía sau bị che. | lop_mo_ho | R-20, R-06, R-02 | Bán tải: theo quy tắc chốt ở img_012 (tạm car). Ngựa: không gán. Car bị che: occluded_level theo %. (chưa chốt, cần họp) | Hai điểm mơ hồ: lớp của bán tải và vật được chở. |
| img_029 | Người đi xe máy chở thanh dài, người đi xe đạp, người đi bộ ở xa. | ngoai_taxonomy | R-12, R-11, R-04, R-13 | Xe đạp: như img_003 (không gán box, người vẫn gán person). Thanh dài không vào box xe. Người xa nhỏ: theo ngưỡng 20 px. (chưa chốt, cần họp) | Lặp lại ca xe đạp. |
| img_030 | Bãi xe máy đỏ chất đống, một người đứng giữa. | nho_dong | R-14, R-02 | Xe tách được và cao từ 20 px: box riêng; cụm xe dày: ignore_region. | v0 không nói xử lý dòng xe máy dày đặc. |
| img_031 | Phố đêm mưa: ô tô, xe buýt bị che, người trên xe, phản chiếu trên đường. | ngoai_taxonomy | R-19, R-08, R-16, R-13 | Bus: như img_009. Car xa bị che: occluded_level theo %, nếu dưới 20 px thì ignore_region. Không gán phản chiếu. (chưa chốt, cần họp) | Lặp lại ca bus. |
| img_032 | Hàng người đông ở trạm xe, ô tô cắt mép và bị che. | nho_dong | R-14, R-13, R-03 | Đám người nhỏ khó tách: 1 ignore_region (người tách rõ và cao từ 20 px thì gán riêng). Car: truncated / occluded theo ngưỡng. | Như img_021. |
| img_033 | Ma-nơ-canh sau cửa kính, phản chiếu. | khong_that | R-06, R-08, R-22 | Ma-nơ-canh: không gán. Ảnh để trống. | Như img_017; dễ bị gán nhầm person. |
| img_034 | Người băng đường, ô tô bị che, người trên xe máy cắt mép. | che_khuat | R-04, R-02, R-03 | Car bị che: occluded_level theo %. Người trên xe máy: person + motorcycle (cắt mép: truncated=yes). | Chỉ cần ngưỡng % che rõ ràng. |
| img_035 | Xe ba bánh chở bó hàng cao, xe máy, ô tô trắng (ghi chú gốc: ô tô chở hàng). | lop_mo_ho | R-18, R-11, R-04 | Xe ba bánh chở hàng: như img_004 (không gán box); hàng không vào box. Xe máy: person + motorcycle. Ô tô trắng: car. (chưa chốt, cần họp) | Ranh giới car / van / truck chưa rõ; xe ba bánh không có lớp. |
| img_036 | Xe tải nhỏ chở hàng có bạt, xe máy đỗ dày bị che và cắt mép. | che_khuat | R-14, R-02, R-03 | Truck: 1 box. Xe máy: occluded_level theo % + truncated=yes; cụm xe dày phía sau ignore_region. | Xe vừa bị che vừa cắt mép. |
| img_037 | Bán tải gắn nắp thùng cao, xe buýt cắt mép, người đi bộ. | lop_mo_ho | R-20, R-19, R-03 | Xe cabin kín nắp thùng cao: tạm gán car (car hay van chưa chốt). Bus: như img_009 (truncated=yes). Người đi bộ: person. | v0: van là 'xe thùng kín cỡ nhỏ' nhưng không phân biệt với car. |
| img_038 | Tủ kính đồ chơi, xe mô hình. | khong_that | R-06, R-22 | Mô hình: không gán. Ảnh để trống. | Như img_027. |
| img_039 | Phố xa: xe máy đỗ lớn, người đi bộ và xe nhỏ ở xa. | nho_dong | R-13, R-14, R-12 | Người xa dưới 20 px hoặc không nhận ra: ignore_region / không gán. Xe máy đỗ: motorcycle. Không gán xe đạp. | Như img_025. |
| img_040 | Taxi cắt mép và bị che, người trên xe, xe buýt bị che, ô tô ở xa. | ngoai_taxonomy | R-19, R-03, R-05, R-04 | Bus: như img_009. Car: truncated + occluded theo ngưỡng. Tài xế trong taxi không gán. Người đội mũ: person. (chưa chốt, cần họp) | Lặp lại ca bus. |

Ca chưa chắc: tick `can_review` trên box hoặc ghi vấn đề cấp ảnh vào `submission/qc/image_issues.csv` (R-23).

## 5. QC và phiên bản

Ngưỡng áp dụng (từ `submission/taxonomy.json`, không đổi): `min_box_height_px` = 20, `large_px` = 50, `iou_thr_large` = 0.85, `iou_thr_small` = 0.7, `iaa_iou` = 0.85, `iaa_class` = 0.95, `error_rate_img_max` = 0.02, `match_iou` = 0.5.

- 1. Hold: guideline chưa đủ rõ hoặc dữ liệu hỏng, chưa thể kết luận.
- 2. Rework: hơn 2% ảnh lỗi, hoặc lỗi lặp cùng một kiểu, hoặc IAA dưới ngưỡng.
- 3. Release: đạt mọi ngưỡng ở trên.
- Ca mơ hồ do ai quyết: cả nhóm họp Pha 5, ghi mỗi quyết định một dòng vào calibration_decisions.csv kèm rule_id R-xx.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên. | — |
| v0.1 (cá nhân) | Viết lại theo phiếu soi 40 ảnh: giữ 5 lớp của v0 (xe ba bánh và xe buýt chưa có lớp, phương án B); mức che 10/40/80%; quy tắc người–xe, vật không thật, vật nhỏ, nhòe; mã quy tắc R-01 đến R-23. | img_003, img_004, img_009, img_012, img_017, img_020, img_024, img_027, img_033, img_038 |
