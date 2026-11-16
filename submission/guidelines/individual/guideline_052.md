# Guideline gán nhãn — v1.0 (NHÁP, chờ họp nhóm)

- Phiên bản: v1.0 (nháp)
- Ngày: …
- Nhóm 5 người, mã thành viên: … (điền theo TEAM.md)
- Người tổng hợp: 052
- Nguồn nháp: bản cá nhân của Loan. **Chưa có bản của 4 thành viên còn lại.** Mọi dòng `[CHỜ HỌP]` phải được cả nhóm chốt trước khi khóa v1.0.

> Khi khóa: đổi tiêu đề thành `v1.0`, bỏ mọi dấu `[CHỜ HỌP]`, sửa `submission/taxonomy.json` khớp (đặt `version` là `v1.0`, giữ nguyên mục `rules`), chạy `python scripts/make_cvat_labels.py`.
> Nhóm 5 người: mỗi ảnh 3 người gán độc lập (`split_assign.py --members <5 mã> --copy-images`).

## 0. Việc phải chốt khi họp

| # | Vấn đề | Vì sao cần chốt |
|---|---|---|
| 1 | Có thêm lớp `bus` và `tuk_tuk` không? | `taxonomy.json` hiện chỉ có `car`, `truck`, `van`, `motorcycle`, `person`, `ignore_region`. Thêm lớp thì phải sửa taxonomy và bộ nhãn CVAT. |
| 2 | Xe 9–15 chỗ thuộc lớp nào? | Bản nháp: `van` dưới 9 chỗ, `bus` từ 16 chỗ, nên 9–15 chỗ chưa có lớp. |
| 3 | Cách gán người đi xe máy (R-04) | Cảnh phổ biến nhất của bộ ảnh; mọi người phải cùng số box và cùng phần bao trong box. |
| 4 | Xe chở xe (R-10) và bán tải (R-20) | Có gán từng ô tô trên xe chở không? Bán tải gán `truck` hay `car`? |
| 5 | Ngưỡng `occluded_level` và `truncated` | Đối chiếu số % của từng người. |
| 6 | Mã R-xx cuối cùng | Mã dưới đây đánh theo bản nháp, nhóm đánh lại sau khi hợp nhất. |

## 1. Mục tiêu

- Nhãn dùng để huấn luyện mô hình phát hiện người và phương tiện 2D trên ảnh đường phố châu Á (camera giao thông, xe tự lái).
- Bỏ sót người hoặc xe thật là lỗi nguy hiểm nhất.
- Gán vật không thật (tượng, ma-nơ-canh, đồ chơi, hình in, phản chiếu) dạy mô hình báo có người/xe khi không có.
- Gán nhầm lớp (xe tải thành xe con) làm sai ước lượng kích thước và hành vi.
- Hai người đọc cùng một quy tắc phải ra cùng một kết quả: chỗ nào phụ thuộc cảm giác thì thay bằng ngưỡng đo được hoặc mã quy tắc R-xx.

## 2. Lớp và thuộc tính

### Lớp (mỗi vật thuộc đúng một lớp)

| Lớp | Định nghĩa (một câu) | Ví dụ (ảnh) | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con chở người: sedan, hatchback, SUV, taxi con. | img_023 (Honda Civic), img_012 (SUV xám), img_040 (taxi), img_024 (xe cổ trưng bày) | Bán tải có thùng hở là truck (R-20). Xe đồ chơi/mô hình không gán (R-06). |
| `truck` | Xe có thùng hàng riêng sau cabin: xe tải, xe bán tải, xe chở xe. | img_002 (xe chở ô tô), img_036 (tải nhỏ bạt xanh), img_007 (Isuzu, Hilux), img_001, img_028 | Xe thùng liền cabin là van. Bán tải gắn nắp thùng cao (img_037) vẫn là truck vì còn thấy cabin riêng. |
| `van` | Xe thùng kín cỡ nhỏ, cabin và khoang liền một khối, dưới 9 chỗ. | img_005 (van xám), img_016 (van trắng bên phải) | Xe từ 16 chỗ trở lên là bus (R-19). |
| `bus` `[CHỜ HỌP]` | Xe buýt, xe khách, minibus từ 16 chỗ. | img_009 (minibus), img_031, img_037 (buýt xanh), img_040 (buýt ở nền) | Van dưới 9 chỗ là van. |
| `tuk_tuk` `[CHỜ HỌP]` | Xe ba bánh có động cơ chở khách/hàng: tuk-tuk, xe lam, auto-rickshaw. | img_004 (hai tuk-tuk xanh), img_016, img_035 (auto-rickshaw vàng), img_040 | Xích lô đạp (img_023) không có động cơ nên không gán xe (R-12). Xe máy hai bánh là motorcycle. |
| `motorcycle` | Xe hai bánh có động cơ: xe máy, xe tay ga, mô tô; kể cả xe đỗ không người và xe có sidecar. | img_006, img_008, img_022 (sidecar), img_029, img_039 (xe đỗ) | Xe đạp, xích lô đạp không gán (R-12). Xe máy bằng kim loại trong tác phẩm điêu khắc (img_011) không gán (R-06). |
| `person` | Người thật đi bộ, đứng, hoặc ngồi trên phương tiện hở (xe máy, xe đạp, xích lô, tuk-tuk). | img_003, img_010, img_014, img_034, img_023 (người đạp xích lô) | Tượng, ma-nơ-canh, linh vật nhồi bông, mặt người in trên biển quảng cáo không gán (R-06, R-07). Người ngồi trong xe kín không gán (R-05). |
| `ignore_region` | Vùng không gán từng vật: cụm dày không tách được hoặc cụm vật cao dưới 20 px. | img_021 (biển xe máy phía sau), img_019 (đám đông), img_030 (bãi xe), img_032 (hàng người) | Không dùng để bỏ qua vật to, rõ ràng (R-14). |

Vật không thuộc lớp nào (xe đạp, xích lô đạp, xe đẩy, nón giao thông, biển báo, ghế): không gán; người trên đó vẫn gán person (R-12). Cụm vật dày hoặc dưới 20 px: ignore_region (R-13, R-14).

### Thuộc tính

| Thuộc tính | Mức | Nghĩa và ngưỡng | Ghi chú |
|---|---|---|---|
| occluded_level | 0 | Không bị che hoặc che dưới 5% diện tích vật (ước tính cả vật). | Chỉ tính che bởi vật khác (cột, xe khác, người khác, hàng rào). Người ngồi trên xe KHÔNG tính là che xe đó. |
| occluded_level | 1 | Che từ 5% đến 30%. | Che nhẹ, vẫn thấy rõ hình dạng. |
| occluded_level | 2 | Che từ trên 30% đến 70%. | Thấy phần lớn đặc trưng nhận dạng. |
| occluded_level | 3 | Che từ trên 70% đến 90%, nhận ra lớp nhờ phần nhỏ còn lại. | Che trên 90% hoặc không đoán chắc lớp thì KHÔNG gán (R-02). |
| truncated | no | Vật nằm trọn trong ảnh. | Cắt mép ảnh KHÔNG tính vào occluded_level. |
| truncated | yes | Phần vật nằm ngoài mép ảnh từ 10% trở lên (ước tính). | Box vẽ tới sát mép ảnh (R-03). |

`[CHỜ HỌP]` Ngưỡng % ở trên là của một người; đối chiếu với 4 bản còn lại.

## 3. Quy tắc vẽ box

| Rule ID | Nhóm | Quy tắc | Ảnh ví dụ | Lý do |
|---|---|---|---|---|
| R-01 | Vẽ box | Box ôm sát phần NHÌN THẤY của vật, không ngoại suy phần bị che hoặc nằm ngoài ảnh. Cạnh box chạm điểm ngoài cùng thấy được. | img_018, img_005 | Mọi người cùng một cách vẽ thì IoU cao; không ai phải đoán phần khuất. |
| R-02 | Che khuất | Mức che theo bảng thuộc tính. Che trên 90% hoặc không đoán chắc lớp thì không gán. Lưỡng lự giữa hai mức: chọn mức cao hơn, tick can_review. | img_018, img_005, img_028, img_030 | Ngưỡng % đo được giúp hai người chọn cùng một mức. |
| R-03 | Cắt mép | Vẽ box tới sát mép ảnh, truncated = yes nếu phần ngoài ảnh từ 10% trở lên. Phần thấy dưới 20% vật và không đoán chắc lớp thì không gán. | img_005, img_015, img_016, img_026, img_040 | Tách rõ cắt mép với che khuất. |
| R-04 `[CHỜ HỌP]` | Người và xe máy | Hai box chồng nhau: person (đầu/mũ tới phần thân thấy được, gồm ba lô; không gồm xe) và motorcycle (hình chữ nhật bao các phần xe thấy được từ bánh trước tới bánh sau; không gồm thân người phía trên tay lái). Xe chở hai người: 1 motorcycle + 2 person. Người ngồi trên xe không làm tăng occluded_level của xe. Nếu chỉ còn lộ gương/tay lái (dưới 20% xe): chỉ gán person. | img_006, img_010, img_026, img_034; thử lại trên img_008, img_040 | Một cách đếm duy nhất cho cảnh phổ biến nhất của bộ ảnh. |
| R-05 | Người trong xe | Người ngồi trong khoang kín (nhìn qua kính) không gán. Người ngồi trên/trong phương tiện hở (xe máy, xe đạp, xích lô, tuk-tuk, xe mui trần/xe đồ chơi hở) thì gán person. | img_040, img_004, img_016, img_019, img_023 | Người trong xe kín không phải mục tiêu; người trên xe hở là người thật thấy rõ. |
| R-06 | Vật không thật | Tượng, ma-nơ-canh, mô hình, đồ chơi, linh vật nhồi bông, xe điêu khắc: không gán dù trông giống người/xe. | img_011, img_017, img_019, img_027, img_028, img_033, img_038 | Gán vật không thật dạy mô hình báo có người/xe khi không có. |
| R-07 | Hình in | Mặt người, xe in trên biển quảng cáo, tranh tường, poster: không gán. | img_018, img_026 | Không phải vật thật trong cảnh. |
| R-08 | Phản chiếu | Hình phản chiếu trong gương, kính, mặt đường ướt: không gán. | img_015, img_020, img_031, img_033 | Không phải vật thật tại vị trí đó. |
| R-09 | Xe chủ | Bộ phận của xe mang camera (gương chiếu hậu, nắp capo, kính) không gán. | img_020, img_033 | Không phải đối tượng cần phát hiện. |
| R-10 `[CHỜ HỌP]` | Xe chở xe | Gán một truck cho xe chở (đầu kéo cộng khung); gán thêm car cho từng ô tô thật nhìn thấy trên khung, box chồng nhau, mức che theo bảng. Ô tô nằm trên nóc ô tô khác: gán cả hai. | img_002, img_024 | Cả xe chở và xe được chở đều là xe thật trên đường. |
| R-11 | Hàng hóa | Hàng chở trên xe (bó hàng, thùng, thanh dài, bao tải) không đưa vào box xe. Box chỉ ôm thân xe và người. | img_012, img_029, img_035 | Hàng nhô ra làm box dao động; loại ra để các box ổn định. |
| R-12 | Vật không có lớp | Xe đạp, xích lô đạp, xe đẩy, nón giao thông, biển báo, ghế: không gán. Người ngồi/đi trên đó vẫn gán person. Chỉ dùng ignore_region nếu vật nằm trong cụm dày (R-14). | img_003, img_006, img_023, img_029, img_039 | Taxonomy không có lớp tương ứng. |
| R-13 | Vật nhỏ | Vật cao dưới 20 px: không gán từng vật, gộp vào ignore_region nếu thành cụm. Vật từ 20 px trở lên và tách được ranh giới: gán bình thường. | img_013, img_025, img_039, img_011 | Khớp ngưỡng min_box_height_px = 20 của taxonomy. |
| R-14 | Cụm dày | Từ 5 vật sát nhau trở lên và không tách được ranh giới (hoặc đều dưới 20 px): một ignore_region ôm cả cụm. Vật to, rõ ở phía trước vẫn gán riêng. Không dùng ignore_region để né vật rõ. | img_021, img_019, img_030, img_032, img_008, img_036 | Bỏ sót trong vùng này không tính là lỗi nên phải dùng đúng chỗ. |
| R-15 | Mờ nhòe | Nhận ra lớp: gán bình thường. Chỉ còn vệt mờ không nhận ra lớp: không gán (hoặc ignore_region nếu nằm trong cụm). | img_014, img_015 | Tránh đoán lớp từ vệt mờ. |
| R-16 | Tối, ngược sáng | Gán vật thấy rõ lớp. Vùng tối không đoán được thì không gán, tick can_review ở vật gần nhất nếu lưỡng lự. | img_007, img_013, img_026, img_031 | Giữ nhãn đáng tin thay vì đoán. |
| R-17 | Xe thật hình lạ | Phương tiện thật đang chạy hoặc được trưng bày (xe cổ, xe giả tưởng có biển số và người lái) vẫn gán theo lớp gần nhất. | img_019, img_024 | Là phương tiện thật trong cảnh. |
| R-18 `[CHỜ HỌP]` | Lớp tuk_tuk | Xe ba bánh có động cơ (tuk-tuk, xe lam, auto-rickshaw) gán tuk_tuk. | img_004, img_016, img_035, img_040 | Mô hình phải đoán ra loại xe này nên là lớp, không phải thuộc tính. |
| R-19 `[CHỜ HỌP]` | Lớp bus | Xe buýt, xe khách, minibus từ 16 chỗ gán bus; xe thùng kín dưới 9 chỗ gán van. | img_009, img_031, img_037, img_040 | Khác van về kích cỡ và hành vi giao thông. |
| R-20 `[CHỜ HỌP]` | Bán tải | Xe bán tải gán truck, kể cả khi gắn nắp thùng cao vẫn thấy cabin riêng. | img_001, img_007, img_012, img_028, img_037 | Còn thấy cabin tách khỏi khoang hàng. |
| R-21 | Xe có sidecar | Một motorcycle ôm cả sidecar; người ngồi trong sidecar là một person riêng. | img_022 | Sidecar là phần của xe. |
| R-22 | Ảnh trống | Ảnh không có vật nào thuộc phạm vi: để trống, vẫn tính là đã gán. | img_017, img_020, img_027, img_038 | Ảnh không có vật cần gán là kết quả hợp lệ. |
| R-23 | Chưa chắc | Tick can_review trên box. Vật chưa vẽ được hoặc cả ảnh mơ hồ: ghi vào submission/qc/image_issues.csv (image, annotator, region, note). Không hỏi đồng đội khi đang gán. | mọi ảnh | Giữ gán độc lập để IAA có ý nghĩa. |

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_001 | Quán bar vỉa hè ban đêm: bán tải trắng cũ, người đứng cạnh xe, người ngồi quán sau bàn/cột. | R-20, R-11, R-02 | truck cho bán tải (truncated yes nếu cắt mép phải); person cho người đứng và người ngồi quán thấy rõ, ghi mức che do bàn/cột; không gán biển quảng cáo và hàng trên thùng. | Bán tải còn cabin riêng; hàng không thuộc thân xe. |
| img_002 | Xe chở 8 ô tô trên hai sàn, ô tô chồng và che nhau. | R-10, R-02 | Một truck cho cả xe chở; thêm car cho từng ô tô nhìn thấy, mức che theo bảng. | Mọi ô tô thật đều là vật thể trong ảnh. |
| img_003 | Chợ vỉa hè: người gánh quang gánh đội nón lá, người đứng cạnh xe đạp, xe máy phía sau. | R-12, R-02, R-13 | person cho người gánh, người đứng, người ngồi xe máy; motorcycle phía xa nếu từ 20 px; không gán xe đạp, quang gánh. | Xe đạp không có lớp; vật dưới 20 px không gán. |
| img_004 | Hai tuk-tuk xanh, tài xế ngồi trong xe hở. | R-18, R-05 | tuk_tuk cho hai xe, person cho tài xế; không gán nón giao thông. | v0 không có lớp ba bánh; hình dạng khác motorcycle và car. |
| img_005 | Van xám phía sau; 5-6 người đi xe máy sát ống kính che nhau; xe máy chỉ lộ gương/tay lái. | R-04, R-02, R-03, R-14 | van cho xe sau; person từng người thấy rõ; xe máy chỉ lộ dưới 20% thì không gán motorcycle. | Người che xe máy gần hết nên chỉ gán phần nhận ra chắc. |
| img_006 | Ngã tư đông xe máy chạy về phía camera, người đi xe đạp bên phải, taxi trắng bên trái. | R-04, R-12, R-14 | motorcycle + person cho từng xe thấy rõ; person cho người đạp xe, không gán xe đạp; car cho taxi; cụm xe xa dày thì ignore_region. | Một cách đếm duy nhất cho người đi xe máy (R-04). |
| img_007 | Hẻm kho tối ẩm: bán tải trắng và xe tải Isuzu, người đứng trong bóng tối. | R-16, R-20, R-02 | truck cho cả hai xe; person chỉ khi nhận ra chắc; không gán vùng tối không đoán được và rào chắn cam. | Giữ nhãn đáng tin thay vì đoán. |
| img_008 | Bãi xe máy ban đêm, hai người áo cam ngồi trên xe, xe dày sát nhau. | R-04, R-14, R-16 | motorcycle + person cho xe và người thấy rõ; cụm dày phía sau ignore_region. | Không tách được ranh giới từng xe. |
| img_009 | Minibus cỡ lớn ở giữa đường. | R-19 | bus. | Không phải van dưới 9 chỗ. |
| img_010 | Chợ vỉa hè: phụ nữ ngồi xe máy bấm điện thoại, hai người trên xe phía sau, người bán rau, xe đạp chở rau. | R-04, R-12, R-02 | Mỗi xe máy một motorcycle và từng người một person; person cho người bán rau; không gán xe đạp và hàng rau. | Người đứng sát xe che nhau nên ghi mức che theo bảng. |
| img_011 | Tượng lớn kim loại ở tiền cảnh và xe máy kim loại ở nền. | R-06 | Không gán tượng và xe kim loại; chỉ gán người, xe thật nếu từ 20 px trở lên. | Là tác phẩm điêu khắc, không phải xe thật. |
| img_012 | Bán tải trắng chở thùng carton, bao tải; SUV xám bên trái. | R-11, R-20, R-03 | truck, box không gồm hàng; car cho SUV. | Hàng không thuộc thân xe. |
| img_013 | Phố đêm sau mưa, quán ngoài trời có đèn dây, nhiều nón giao thông, vài người nhỏ. | R-13, R-16, R-08 | person cho hai người đứng cạnh chậu cây; motorcycle cạnh nón; car trắng nếu từ 20 px; không gán nón, phản chiếu và vật dưới 20 px. | Ban đêm, vật nhỏ, đường ướt phản chiếu. |
| img_014 | Hai người đi bộ băng đường rõ nét; xe máy phía sau nhòe do chuyển động. | R-15, R-04 | person cho hai người rõ nét; vệt nhòe không nhận ra lớp thì không gán. | Tránh đoán lớp từ vệt mờ. |
| img_015 | Mưa: xe cắt mép hai bên, xe nền nhòe, phản chiếu trên đường ướt. | R-03, R-08, R-15 | Gán hai xe rõ ở rìa (truncated yes); không gán phản chiếu; xe nhòe xa không nhận ra thì bỏ. | Chỉ gán vật thật nhận ra chắc. |
| img_016 | Hai tuk-tuk bị cắt mép, người đứng sát ống kính, xe tay ga và van. | R-18, R-03, R-04, R-05 | tuk_tuk x2 (truncated yes), person (người đứng, người lái xe máy, tài xế tuk-tuk), motorcycle, van. | Theo R-04 và R-05. |
| img_017 | Tượng người cưỡi ngựa có nón giao thông trên đầu. | R-06, R-22 | Không gán gì; ảnh để trống. | Tượng không phải người thật. |
| img_018 | Biển quảng cáo có mặt người; ô tô đỗ sau hàng rào lưới. | R-07, R-02 | Không gán mặt trên biển; gán từng ô tô thấy rõ lớp, ghi mức che do hàng rào. | Hình in không phải vật thật. |
| img_019 | Xe vàng giả tưởng có biển số, người lái, chuột nhồi bông; đám đông sau hàng rào. | R-17, R-05, R-06, R-14 | car cho xe, person cho người lái, không gán chuột; đám đông ignore_region. | Là phương tiện thật đang chạy; linh vật không phải người. |
| img_020 | Gương chiếu hậu xe chủ và hình phản chiếu đường, xe. | R-08, R-09, R-22 | Không gán gì; ảnh để trống. | Phản chiếu và bộ phận xe chủ. |
| img_021 | Kẹt xe máy dày đặc: vài xe phía trước rõ, hàng chục xe nhỏ ở phía sau. | R-04, R-14, R-13 | Gán các xe và người phía trước rõ; phần sau dày thì ignore_region. | Không tách được ranh giới từng vật. |
| img_022 | Ảnh đen trắng cổ: mô tô có sidecar trên đường rừng phủ tuyết. | R-21, R-13 | motorcycle ôm cả sidecar; person cho người lái và người ngồi trong sidecar nếu từ 20 px. | Sidecar là một phần của xe. |
| img_023 | Xích lô đạp (từ phía sau) cạnh ô tô đen. | R-12, R-04 | person cho người đạp; không gán xích lô; car cho ô tô. | Xích lô không có động cơ và không có lớp. |
| img_024 | Ô tô vàng nhỏ nằm trên nóc ô tô đen trong bảo tàng; nhiều xe cổ cắt mép. | R-10, R-17, R-03 | Gán car cho cả hai xe và các xe cổ thấy rõ; truncated yes cho xe cắt mép. | Xe trưng bày là xe thật. |
| img_025 | Bờ biển: vài người đi bộ, taxi vàng, xe máy kéo xe thồ ở xa bên phải, nhiều người ngồi ghế rất nhỏ. | R-13, R-14, R-12 | person cho người đi bộ thấy rõ; car cho taxi; motorcycle nếu từ 20 px; người ngồi ghế dưới 20 px thì ignore_region; không gán ghế. | Vật xa nhỏ, ghế không có lớp. |
| img_026 | Đêm: hai người trên xe máy đỏ trước tranh tường cảng, xe bị cắt ở đáy ảnh. | R-04, R-07, R-03, R-16 | motorcycle (truncated yes) + person x2; không gán tranh tường. | Hình in không phải vật thật; thiếu sáng. |
| img_027 | Xe bán tải bằng đá granite trên mộ. | R-06, R-22 | Không gán gì; ảnh để trống. | Là tượng/đài tưởng niệm. |
| img_028 | Bán tải cổ chở tượng ngựa trắng; SUV đỏ phía sau bị che. | R-06, R-20, R-02 | truck cho bán tải; không gán ngựa; car cho SUV với mức che theo bảng. | Ngựa là tượng. |
| img_029 | Người đi xe máy chở thanh dài nhô ra; người đi xe đạp. | R-11, R-12, R-04 | motorcycle + person cho người lái xe máy, box không gồm thanh; person cho người đạp xe, không gán xe đạp. | Hàng nhô làm box không ổn định. |
| img_030 | Bãi xe máy đỏ chất đống, một người đứng giữa. | R-14, R-02, R-03 | Gán xe và người phía trước rõ; phần dày phía sau ignore_region. | Không tách được từng xe. |
| img_031 | Phố đêm mưa: ô tô, xe buýt, người trên lề, đường ướt phản chiếu đèn neon. | R-08, R-16, R-13, R-19 | car, bus; person và motorcycle nếu từ 20 px; không gán phản chiếu. | Phản chiếu không phải vật thật. |
| img_032 | Hàng khoảng 30 người xếp sát nhau; xe cắt mép trái; nón giao thông. | R-14, R-13, R-03, R-12 | Gán từng người tách được ranh giới; đoạn sát nhau ignore_region; không gán nón. | Người nhỏ, dính nhau. |
| img_033 | Ma-nơ-canh, phản chiếu kính, cánh cửa và gương xe trắng bị cắt gần hết. | R-06, R-08, R-09, R-03 | Không gán gì. | Không có vật thật nhận ra chắc; phần xe có thể là xe chủ. |
| img_034 | Người đàn ông băng đường, ô tô đen, xe máy cắt mép bên phải. | R-04, R-03, R-13 | person cho người băng đường; car; motorcycle + person (truncated yes); xe xa dưới 20 px không gán. | Xe bên phải bị cắt mép. |
| img_035 | Ba auto-rickshaw, một xe chở bó hàng khổng lồ; xe máy; ô tô trắng. | R-18, R-11, R-05 | tuk_tuk cho từng xe, box không gồm bó hàng; person cho người lái. | Hàng không thuộc thân xe. |
| img_036 | Xe tải nhỏ xanh có bạt đỗ vỉa hè, xe máy đỗ dày bên phải. | R-14, R-02 | truck; motorcycle cho xe thấy rõ; cụm dày phía sau ignore_region. | Xe máy chồng sát nhau, không tách được. |
| img_037 | Bán tải gắn nắp thùng cao, buýt xanh cắt mép trái. | R-20, R-19, R-03 | truck cho bán tải; bus cho buýt (truncated yes); person cho người đi bộ. | Còn thấy cabin riêng nên không phải van. |
| img_038 | Tủ kính đầy xe đồ chơi và thú nhồi bông. | R-06, R-22 | Không gán gì; ảnh để trống. | Đồ chơi không phải xe thật. |
| img_039 | Phố nhỏ nhìn xa: xe máy đỗ lớn ở tiền cảnh, người và xe nhỏ ở xa, dây điện chằng chịt. | R-13, R-14, R-12 | motorcycle cho xe đỗ không người; person và motorcycle xa nếu từ 20 px; cụm xa dưới 20 px thì ignore_region; không gán xe đạp. | Nhiều vật nhỏ ở xa. |
| img_040 | Taxi cắt mép; người đội mũ bảo hiểm sát ống kính, xe máy gần như ngoài khung; tuk-tuk, buýt ở nền. | R-03, R-05, R-04, R-18, R-19 | car (taxi, truncated yes), person (người đội mũ, truncated yes), không gán tài xế trong taxi, không gán xe máy chỉ lộ gương. | Theo R-03, R-04, R-05. |

Ca chưa chắc: tick `can_review` trên box, hoặc ghi vấn đề cấp ảnh vào `submission/qc/image_issues.csv` (cột `image,annotator,region,note`). Không hỏi đồng đội khi đang gán.

## 5. QC và phiên bản

Ngưỡng QC lấy từ `submission/taxonomy.json`, mục `rules`, không đổi:

| Tham số | Giá trị |
|---|---|
| `min_box_height_px` | 20 |
| `large_px` | 50 |
| `iou_thr_large` | 0.85 |
| `iou_thr_small` | 0.7 |
| `iaa_iou` | 0.85 |
| `iaa_class` | 0.95 |
| `error_rate_img_max` | 0.02 |
| `match_iou` | 0.5 |

Ca mơ hồ do cả nhóm quyết ở buổi họp Pha 5, mỗi quyết định một dòng trong `calibration_decisions.csv` kèm Rule ID.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên | — |
| v1.0 (nháp) | Tổng hợp từ bản cá nhân của Loan; thêm lớp `bus`, `tuk_tuk` (chờ họp) | img_002, img_004, img_009, img_035, img_037 |
