# Guideline gán nhãn — v1.0 — Nhóm Zero_Risk

> Bản thống nhất Pha 2, hợp nhất từ 5 bản cá nhân trong `submission/guidelines/individual/`
> (`guideline_035.md`, `guideline_036.md`, `guideline_052.md`, `guideline_055.md`, `guideline_070.md`).
> Chỗ các bản khác nhau được chốt theo **số đông** (bảng ở mục 5.3). Bản này **khóa** cho lượt gán độc lập Pha 3:
> chưa chắc thì tick `can_review` hoặc ghi vấn đề cấp ảnh, không tự đổi quy tắc.

- Phiên bản: v1.0
- Ngày: 16/11/2026
- Người viết: cả nhóm (035, 036, 052, 055, 070) · Người tổng hợp: 070 · Người duyệt: cả nhóm
- Loại nhãn: bounding box trên ảnh đường phố · Công cụ: CVAT Rectangle, xuất COCO 1.0
- Taxonomy đi kèm: `submission/taxonomy.json` (`version` = `v1.0`)

## 1. Mục tiêu

- Nhãn dùng để huấn luyện mô hình phát hiện **người và phương tiện thật** (2D) trên ảnh đường phố châu Á (camera giao thông, xe tự lái).
- **Sai nguy hiểm nhất:** bỏ sót người hoặc xe thật, vì mô hình không thấy thì không tránh.
- **Sai thứ hai:** gán vật không thật (tượng, ma-nơ-canh, đồ chơi, hình in, phản chiếu), vì mô hình sẽ báo động giả và phanh vô cớ.
- **Sai thứ ba:** gán nhầm lớp (xe tải thành xe con), làm sai ước lượng kích thước và hành vi.
- Hai người đọc cùng một quy tắc phải ra cùng một kết quả. Chỗ nào phụ thuộc cảm giác thì đã thay bằng ngưỡng đo được hoặc mã quy tắc `R-xx`.
- Khi phân vân, tự hỏi: *"Đây có phải người hoặc phương tiện thật đang ở trên đường mà xe phải né không?"*
- Ảnh không có đối tượng nào vẫn giữ trong tập dữ liệu (R-22).

## 2. Lớp và thuộc tính

### Lớp (mỗi vật thuộc đúng một lớp)

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con chở người: sedan, hatchback, SUV, taxi con, kể cả xe cổ và xe giả tưởng là xe thật. | img_023 (ô tô đen), img_012 (SUV xám), img_040 (taxi), img_024 (xe cổ trưng bày), img_019 (xe vàng có biển số) | Bán tải có thùng là `truck` (R-20). Xe đồ chơi, mô hình, xe bằng đá thì không gán (R-06). |
| `truck` | Xe có cabin và thùng, sàn hoặc khung chở hàng riêng: xe tải, xe ben, đầu kéo, xe chở xe, xe bán tải. | img_002 (xe chở ô tô), img_036 (tải nhỏ có bạt), img_007 (Isuzu), img_001, img_012, img_028, img_037 (bán tải) | Xe có khoang kín liền cabin là `van`. Ô tô nằm trên sàn xe chở có box `car` riêng (R-10). |
| `van` | Xe cỡ nhỏ có khoang kín liền một khối với cabin, không có hàng cửa sổ và cửa lên xuống của xe buýt. | img_005 (van xám phía sau), img_016 (van trắng bên phải) | Thấy hàng cửa sổ khách và cửa lên xuống thì là `bus` (R-19). Bán tải gắn nắp thùng cao vẫn là `truck` vì còn thấy cabin tách thùng (R-20). |
| `bus` | Xe buýt, xe khách, minibus: thân xe chở khách có hàng cửa sổ, cửa lên xuống và khoang khách, mọi cỡ. | img_009 (minibus), img_031, img_037 (buýt cắt mép), img_040 (buýt ở nền) | Xe khoang kín cỡ nhỏ không có đặc điểm trên là `van` (R-19). Không gán thành `truck`. |
| `motorcycle` | Xe hai bánh có động cơ: xe máy, xe tay ga, mô tô, kể cả xe đỗ không người và mô tô gắn sidecar. | img_006, img_008, img_026, img_039 (xe đỗ), img_022 (sidecar) | Xe đạp, xích lô đạp thì không gán (R-12). Xe ba bánh có động cơ thì không có lớp (R-18). Xe máy kim loại điêu khắc (img_011) thì không gán (R-06). |
| `person` | Người thật nhìn trực tiếp: đi bộ, đứng, ngồi, hoặc ngồi trên phương tiện hở (xe máy, xe đạp, xích lô, sidecar, ô tô mui trần). | img_003, img_010, img_014, img_026 (2 người trên xe máy), img_023 (người đạp xích lô) | Tượng, ma-nơ-canh, linh vật, hình in thì không gán (R-06, R-07). Người ngồi trong xe kín hoặc có mui thì không gán (R-05). |
| `ignore_region` | Vùng bỏ qua: cụm dày không tách được, cụm vật cao dưới 20 px, hoặc xe ba bánh có động cơ (R-18); bỏ sót trong vùng này không tính là lỗi. | img_021 (dòng xe máy phía sau), img_019 (đám đông), img_030 (bãi xe), img_032 (hàng người), img_004 (tuk-tuk) | Không dùng để né một vật to, rõ, có lớp (R-14). Không phủ lên vật không thật (R-06). |

**Vật không thuộc lớp nào:**
- **Xe ba bánh có động cơ** (tuk-tuk, xe lam, ba gác máy, auto-rickshaw): không có lớp. Phủ một `ignore_region` sát xe (R-18).
- **Xe đạp, xích lô đạp, xe đẩy, nón giao thông, biển báo, ghế, động vật, hàng hóa**: không gán, không phủ ignore. Người đi hoặc ngồi trên xe đạp, xích lô vẫn gán `person` (R-12).
- **Vật không thật**: không gán, không phủ ignore (R-06, R-07, R-08).

### Thuộc tính (mức cố định, ngưỡng đo được)

| Thuộc tính | Mức | Nghĩa và ngưỡng | Ghi chú |
|---|---|---|---|
| `occluded_level` | `0` | Bị che **dưới 10%** diện tích vật (ước tính theo hình dạng đầy đủ). | Chỉ tính phần bị **vật khác** che (cột, cây, xe khác, người khác, hàng rào). |
| `occluded_level` | `1` | Bị che **từ 10% đến dưới 40%**. | Vẫn thấy rõ hình dạng. |
| `occluded_level` | `2` | Bị che **từ 40% đến dưới 80%**. | Còn thấy phần lớn đặc trưng nhận dạng. |
| `occluded_level` | `3` | Bị che **từ 80% trở lên**, vẫn nhận ra lớp nhờ phần còn lại. | Không đoán chắc lớp thì **không gán** (R-02). |
| `truncated` | `no` | Vật nằm trọn trong ảnh, hoặc chạm mép nhưng phần ra ngoài dưới 10%. | |
| `truncated` | `yes` | Phần vật nằm ngoài mép ảnh **từ 10% trở lên** (ước tính). | Phần ngoài ảnh **không** tính vào `occluded_level`. Một vật có thể vừa bị che vừa bị cắt mép. |
| `can_review` | tick / không | Tick khi đã vẽ được box nhưng còn nghi về lớp, biên, thuộc tính hoặc có nên gán không. | Lưỡng lự giữa hai mức che thì chọn **mức cao hơn** và tick (R-02). |

Người ngồi trên xe **không** làm tăng `occluded_level` của xe đó. Độ nhòe không tính vào mức che.

## 3. Quy tắc vẽ box

| Rule ID | Nhóm | Quy tắc | Ảnh ví dụ | Lý do |
|---|---|---|---|---|
| **R-01** | Vẽ box | Hình chữ nhật nhỏ nhất ôm sát phần **nhìn thấy** của vật, lệch tối đa khoảng 2 px mỗi cạnh; gồm gương, tay lái, chân chống nếu thấy. Không ngoại suy phần bị che hoặc nằm ngoài ảnh. Vật bị cột hay cây cắt thành nhiều mảnh: **một** box bao chung các mảnh. Đo chiều cao theo pixel ảnh gốc. | img_005, img_018 | Mọi người cùng một cách vẽ thì IoU cao, không ai phải đoán phần khuất. |
| **R-02** | Che khuất | Mức che theo bảng thuộc tính (10/40/80%). Không đoán chắc lớp thì không gán. Lưỡng lự giữa hai mức thì chọn mức cao hơn và tick `can_review`. Hai vật chồng nhau: mỗi vật một box, vật phía sau ghi mức che. | img_005, img_006, img_024, img_028, img_034, img_036 | Ngưỡng % giúp hai người chọn cùng một mức. |
| **R-03** | Cắt mép | Box dừng ở mép ảnh. `truncated = yes` nếu phần ngoài ảnh từ 10% trở lên. Phần thấy dưới 20% vật và không đoán chắc lớp thì không gán. | img_005, img_015, img_016, img_026, img_040 | Tách rõ cắt mép với che khuất. |
| **R-04** | Người và xe hai bánh | Mỗi người một `person`, mỗi xe một `motorcycle`; hai box được chồng nhau. Box `person` ôm từ đầu hoặc mũ tới phần thân thấy được, gồm ba lô, không gồm xe. Box `motorcycle` ôm các phần xe thấy được từ bánh trước tới bánh sau, không gồm thân người phía trên tay lái. Xe chở hai người là 1 `motorcycle` + 2 `person`. Xe chỉ lộ gương hoặc tay lái (dưới 20% xe) thì chỉ gán `person`. | img_005, img_006, img_010, img_026, img_034 | Một cách đếm duy nhất cho cảnh phổ biến nhất của bộ ảnh. |
| **R-05** | Người trong xe | Người ngồi trong xe **kín hoặc có mui** (ô tô, van, bus, xe lam, tuk-tuk) nhìn qua kính hay khung: không gán. Người trên phương tiện **hở** (xe máy, xe đạp, xích lô, sidecar, ô tô mui trần): gán `person`. | img_004, img_016, img_019, img_023, img_040 | Người trong xe kín không phải mục tiêu riêng; người trên xe hở là người thật thấy rõ. |
| **R-06** | Vật không thật | Tượng, ma-nơ-canh, mô hình, đồ chơi, linh vật nhồi bông, xe điêu khắc, xe bằng đá: không gán, không phủ `ignore_region`. | img_011, img_017, img_019, img_027, img_028, img_033, img_038 | Gán vật không thật dạy mô hình báo có người hay xe khi không có. |
| **R-07** | Hình in | Người hay xe in trên biển quảng cáo, tranh tường, poster, màn hình: không gán. | img_018, img_026 | Không phải vật thật trong cảnh. |
| **R-08** | Phản chiếu | Hình phản chiếu trong gương, kính, mặt đường ướt: không gán. Người hay xe thật nhìn **xuyên qua** kính trong suốt vẫn xét bình thường. | img_015, img_020, img_031, img_033 | Không phải vật thật tại vị trí đó. |
| **R-09** | Xe chủ | Bộ phận của xe mang camera (gương chiếu hậu, nắp capo, taplo, kính): không gán. | img_020, img_033 | Không phải đối tượng cần phát hiện. |
| **R-10** | Xe chở xe | Một `truck` cho xe chở (đầu kéo + sàn hoặc khung). Thêm một `car` cho mỗi ô tô thật thấy được trên sàn và cao từ 20 px; box được chồng nhau, mức che theo bảng. Ô tô nằm trên nóc ô tô khác: gán cả hai. Rơ-moóc hay thùng kéo phía sau xe: không gán box riêng. | img_002, img_024, img_007 | Cả xe chở và xe được chở đều là xe thật trên đường. |
| **R-11** | Hàng hóa | Hàng chở trên xe (bó hàng, thùng carton, bao tải, thanh dài, tượng trên thùng) không đưa vào box xe. Box chỉ ôm thân xe và các bộ phận gắn cố định. | img_012, img_028, img_029, img_035 | Hàng nhô ra làm box dao động; loại ra để box ổn định. |
| **R-12** | Vật không có lớp | Xe đạp, xích lô đạp, xe đẩy, nón giao thông, biển báo, ghế, động vật: không gán, không phủ ignore. Người đi hoặc ngồi trên đó vẫn gán `person`. Chỉ dùng `ignore_region` nếu vật nằm trong cụm dày (R-14). | img_003, img_006, img_023, img_029, img_039 | Taxonomy không có lớp tương ứng; tránh ép nhầm vào `motorcycle`. |
| **R-13** | Vật nhỏ | Vật có box nhìn thấy cao **dưới 20 px**: không gán từng vật, gộp vào `ignore_region` nếu thành cụm. Vật từ 20 px trở lên, nhận ra lớp và tách được ranh giới: gán bình thường, kể cả khi bị che một phần. | img_001, img_013, img_025, img_039 | Khớp `min_box_height_px` = 20 của taxonomy. |
| **R-14** | Cụm dày | Từ **5 vật** sát nhau trở lên không tách được ranh giới, hoặc đều dưới 20 px: một `ignore_region` ôm sát cả cụm. Vật to, rõ ở phía trước hoặc cạnh cụm vẫn gán riêng. Không dùng `ignore_region` để né vật rõ. | img_008, img_019, img_021, img_030, img_032, img_036 | Bỏ sót trong vùng này không tính lỗi nên phải dùng đúng chỗ. |
| **R-15** | Mờ, nhòe | Nhận ra lớp và xác định được biên: gán bình thường. Nhận ra lớp nhưng biên mờ: gán và tick `can_review`. Chỉ còn vệt mờ không nhận ra lớp: không gán, hoặc ghi vấn đề cấp ảnh nếu nghi là vật thuộc phạm vi. | img_014, img_015 | Tránh đoán lớp từ vệt mờ nhưng không bỏ sót vật thật. |
| **R-16** | Tối, ngược sáng | Gán vật thấy rõ lớp. Vùng tối không đoán được thì không gán; tick `can_review` nếu lưỡng lự. | img_007, img_013, img_026, img_031 | Giữ nhãn đáng tin thay vì đoán. |
| **R-17** | Xe thật hình lạ | Phương tiện thật đang chạy hoặc trưng bày (xe cổ, xe giả tưởng có biển số và người lái) gán theo lớp gần nhất. Nghi là vỏ hay mô hình: tick `can_review`. | img_019, img_024 | Là phương tiện thật trong cảnh. |
| **R-18** | Xe ba bánh có động cơ | Tuk-tuk, xe lam, ba gác máy, auto-rickshaw **không có lớp**. Phủ **một `ignore_region`** sát mỗi xe (hoặc cả cụm xe) nếu cao từ 20 px. Người ngồi trong xe không gán; người đứng ngoài xe gán bình thường. Không gán thành `motorcycle` hay `car`. | img_004, img_016, img_035, img_040 | Giữ lớp sạch; phủ ignore để mô hình không bị phạt khi phát hiện xe thật. |
| **R-19** | Xe buýt và van | Gán `bus` khi thấy thân xe chở khách: hàng cửa sổ khách, cửa lên xuống, khoang khách, kể cả minibus. Xe khoang kín liền cabin, cỡ nhỏ, không có các đặc điểm trên: `van`. Không xét theo số chỗ vì không đếm được trên ảnh. | img_005, img_009, img_031, img_037, img_040 | Xe buýt khác van về kích cỡ và hành vi giao thông; dấu hiệu nhìn thấy được thì hai người chọn giống nhau. |
| **R-20** | Bán tải | Bán tải có thùng chở hàng (thùng hở, trống, có bạt hoặc nắp cao) gán `truck`. Bị che hoặc cắt mép không thấy có thùng hay không thì gán theo phần thấy và tick `can_review`. | img_001, img_007, img_012, img_028, img_037 | Còn thấy cabin tách khỏi khoang hàng. |
| **R-21** | Xe có sidecar | Một `motorcycle` ôm cả xe và sidecar. Người lái và người ngồi trong sidecar mỗi người một `person`. | img_022 | Sidecar là bộ phận gắn bên hông mô tô. |
| **R-22** | Ảnh trống | Ảnh không có vật nào thuộc phạm vi: để trống, vẫn tính là đã gán. | img_017, img_020, img_027, img_033, img_038 | Ảnh không có vật cần gán là kết quả hợp lệ. |
| **R-23** | Chưa chắc | Tick `can_review` trên box. Vật chưa vẽ được box đáng tin hoặc cả ảnh mơ hồ: ghi vào `submission/qc/image_issues.csv` (cột `image,annotator,region,note`, kèm mã quy tắc). Không hỏi đồng đội khi đang gán. | mọi ảnh | Giữ gán độc lập để IAA có ý nghĩa. |

## 4. Ca khó

Áp dụng v1.0 cho cả 40 ảnh. "Loại theo quy tắc" là không gán vì quy tắc nói rõ. "Cần reviewer" là gán theo quy tắc và tick `can_review`.

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_001 | Quán vỉa hè ban đêm: bán tải trắng cắt mép, người đứng cạnh xe, người ngồi quán ở xa mờ. | R-20, R-03, R-13, R-11 | `truck` cho bán tải (`truncated` theo ngưỡng); `person` cho người đứng; người xa gán nếu từ 20 px và nhận ra được (kèm `can_review`), không thì không gán; không gán hàng trên thùng. | Bán tải có thùng là truck; người xa theo ngưỡng 20 px. |
| img_002 | Xe chở 8 ô tô trên hai sàn, ô tô chồng và che nhau. | R-10, R-02 | Một `truck` cho cả xe chở; thêm `car` cho từng ô tô thấy được từ 20 px, mức che theo bảng. | Xe chở và xe được chở đều là xe thật. |
| img_003 | Chợ vỉa hè: người gánh hàng, người đứng cạnh xe đạp, xe máy phía sau. | R-12, R-04, R-13 | `person` cho từng người; `motorcycle` nếu từ 20 px; không gán xe đạp, quang gánh. | Xe đạp không có lớp. |
| img_004 | Hai tuk-tuk xanh đỗ cạnh nhau, tài xế ngồi trong xe. | R-18, R-05 | Phủ `ignore_region` sát hai xe; không gán tài xế trong xe; không gán nón giao thông. | Xe ba bánh có động cơ không có lớp. |
| img_005 | Van xám phía sau; 5–6 người đi xe máy sát ống kính, che nhau; xe máy chỉ lộ gương hoặc tay lái. | R-04, R-02, R-03 | `van` cho xe sau; `person` cho từng người thấy rõ (mức che theo bảng, `truncated` nếu cắt mép); xe máy lộ dưới 20% thì không gán. | Chỉ gán phần nhận ra chắc. |
| img_006 | Ngã tư đông xe máy, người đạp xe bên phải, taxi trắng cắt mép và bị che. | R-04, R-12, R-02, R-03, R-14 | `motorcycle` + `person` cho từng xe thấy rõ; `person` cho người đạp xe, không gán xe đạp; `car` cho taxi (`truncated` và `occluded_level` theo ngưỡng); cụm xe xa dày thì `ignore_region`. | Tách cắt mép với che khuất. |
| img_007 | Hẻm kho tối: bán tải trắng kéo thùng hàng, xe tải Isuzu cắt mép, người trong bóng tối. | R-20, R-10, R-03, R-16 | `truck` cho bán tải (thùng kéo phía sau không gán riêng) và `truck` cho Isuzu (`truncated = yes`); `person` chỉ khi nhận ra chắc. | Giữ nhãn đáng tin thay vì đoán. |
| img_008 | Bãi xe máy ban đêm, hai người áo cam ngồi trên xe, xe dày sát nhau. | R-04, R-14, R-16 | `motorcycle` + `person` cho xe và người thấy rõ; cụm dày phía sau `ignore_region`. | Không tách được ranh giới từng xe. |
| img_009 | Minibus giữa đường, xe máy cắt mép, người ở xa. | R-19, R-03, R-13 | `bus` cho minibus; `motorcycle` `truncated = yes`; người xa theo ngưỡng 20 px. | Thấy cửa sổ khách và cửa lên xuống. |
| img_010 | Chợ vỉa hè: phụ nữ ngồi xe máy, hai người trên xe phía sau, người bán rau, xe đạp chở rau. | R-04, R-12, R-02 | Mỗi xe máy một `motorcycle`, mỗi người một `person`; `person` cho người bán rau; không gán xe đạp và hàng rau. | Một cách đếm cho người trên xe. |
| img_011 | Tượng lớn kim loại và xe máy kim loại ở hội chợ; xe lớn bị che ở nền; người ở xa nhỏ. | R-06, R-19, R-13 | Không gán tượng và xe kim loại; xe lớn ở nền gán `bus` nếu nhận ra (kèm `can_review`); người dưới 20 px gộp `ignore_region` nếu thành cụm. | Tác phẩm điêu khắc không phải xe thật. |
| img_012 | Bán tải trắng thùng hở chở thùng carton, bao tải; SUV xám bên trái. | R-20, R-11, R-03 | `truck` cho bán tải, box không gồm hàng; `car` cho SUV. | Bán tải có thùng là truck. |
| img_013 | Phố đêm sau mưa, quán ngoài trời, nhiều nón giao thông, vài người nhỏ, xe ở xa. | R-13, R-14, R-16, R-08 | `person` cho người từ 20 px; `car` và `motorcycle` nếu từ 20 px và tách được; cụm nhỏ `ignore_region`; không gán nón, phản chiếu. | Ban đêm, vật nhỏ, đường ướt. |
| img_014 | Hai người đi bộ rõ nét; xe máy phía sau nhòe do chuyển động. | R-15, R-04 | `person` cho hai người; xe và người nhòe vẫn nhận ra thì gán và tick `can_review`; vệt không nhận ra thì không gán. | Không bỏ sót vật thật, không đoán từ vệt mờ. |
| img_015 | Mưa: ô tô cắt mép hai bên, xe nền nhòe, phản chiếu trên đường ướt. | R-03, R-15, R-08 | `car` cho hai xe ở rìa (`truncated = yes`); xe nhòe nhận ra thì gán và tick `can_review`; không gán phản chiếu. | Chỉ gán vật thật nhận ra được. |
| img_016 | Hai tuk-tuk cắt mép, người đứng sát ống kính, xe tay ga, van trắng. | R-18, R-03, R-04, R-05 | `ignore_region` cho hai tuk-tuk, không gán người trong tuk-tuk; `person` cho người đứng và người lái xe tay ga; `motorcycle`; `van`. | Theo R-18 và R-04. |
| img_017 | Tượng người cưỡi ngựa đội nón giao thông. | R-06, R-22 | Không gán gì; ảnh để trống. | Tượng không phải người thật. |
| img_018 | Biển quảng cáo có mặt người; ô tô đỗ sau hàng rào lưới, có xe cắt mép. | R-07, R-02, R-03 | Không gán mặt trên biển; `car` cho từng ô tô thấy rõ lớp, mức che do hàng rào, `truncated` nếu cắt mép. | Hình in không phải vật thật. |
| img_019 | Xe vàng giả tưởng có biển số, người lái, chuột nhồi bông; đám đông sau hàng rào. | R-17, R-05, R-06, R-14 | `car` cho xe vàng; người lái gán `person` nếu xe hở mui và thấy trực tiếp, ngồi sau kính kín thì không gán (tick `can_review` trên xe nếu lưỡng lự); không gán chuột; đám đông `ignore_region`. | Xe thật đang chạy; linh vật không phải người. |
| img_020 | Gương chiếu hậu xe chủ phản chiếu đường và xe. | R-08, R-09, R-22 | Không gán gì; ảnh để trống. | Phản chiếu và bộ phận xe chủ. |
| img_021 | Kẹt xe máy dày đặc: vài xe phía trước rõ, hàng chục xe và người nhỏ phía sau, ô tô cắt mép. | R-04, R-13, R-14, R-03 | `motorcycle` + `person` cho xe phía trước rõ; phần sau `ignore_region`; `car` `truncated = yes`. | Không tách được ranh giới từng vật. |
| img_022 | Ảnh đen trắng cổ: mô tô gắn sidecar, người ngồi trong sidecar. | R-21, R-13 | Một `motorcycle` ôm cả sidecar; `person` cho người lái và người trong sidecar nếu từ 20 px. | Sidecar là bộ phận của mô tô. |
| img_023 | Xích lô đạp cạnh ô tô đen, người trên xe máy. | R-12, R-04 | `person` cho người đạp xích lô, không gán xích lô; `motorcycle` + `person`; `car` cho ô tô. | Xích lô không có động cơ và không có lớp. |
| img_024 | Ô tô vàng nhỏ nằm trên nóc ô tô đen trong bảo tàng; nhiều xe cổ cắt mép. | R-10, R-17, R-02, R-03 | `car` cho cả hai xe (box chồng nhau, xe dưới ghi mức che); `car` cho các xe cổ thấy rõ, `truncated = yes` nếu cắt mép; nghi xe vàng là vỏ trưng bày thì tick `can_review`. | Xe trưng bày là xe thật. |
| img_025 | Bờ biển: người đi bộ, taxi vàng cắt mép, người trên xe máy, nhiều người ngồi ghế rất nhỏ. | R-13, R-04, R-03, R-12 | `person` cho người đi bộ thấy rõ; `car` (`truncated = yes`); `motorcycle` + `person` nếu từ 20 px; người dưới 20 px `ignore_region`; không gán ghế. | Vật xa nhỏ; ghế không có lớp. |
| img_026 | Đêm: hai người trên xe máy đỏ, xe cắt ở đáy ảnh, trước tranh tường. | R-04, R-03, R-07, R-16 | 1 `motorcycle` (`truncated = yes`) + 2 `person`; không gán tranh tường. | Người lái, người ngồi sau và xe là ba đối tượng. |
| img_027 | Xe bán tải bằng đá granite trên mộ. | R-06, R-22 | Không gán gì; ảnh để trống. | Là tượng đài, không phải xe thật. |
| img_028 | Bán tải cổ chở tượng ngựa trắng; SUV đỏ phía sau bị che. | R-20, R-11, R-06, R-02 | `truck` cho bán tải, box không gồm tượng ngựa; `car` cho SUV với mức che theo bảng. | Ngựa là tượng đặt trên thùng (hàng chở). |
| img_029 | Người đi xe máy chở thanh dài nhô ra, người đi xe đạp, người ở xa. | R-11, R-12, R-04, R-13 | `motorcycle` + `person`, box xe không gồm thanh; `person` cho người đạp xe, không gán xe đạp; người xa theo ngưỡng 20 px. | Hàng nhô làm box không ổn định. |
| img_030 | Bãi xe máy đỏ chất đống, một người đứng giữa. | R-14, R-02, R-03 | Gán xe và người phía trước tách được từ 20 px; phần dày phía sau `ignore_region`. | Không tách được từng xe. |
| img_031 | Phố đêm mưa: ô tô, xe buýt bị che, người trên lề, phản chiếu neon. | R-19, R-08, R-16, R-13 | `car`; `bus` (mức che theo bảng); `person` và `motorcycle` nếu từ 20 px; không gán phản chiếu. | Phản chiếu không phải vật thật. |
| img_032 | Hàng khoảng 30 người xếp sát nhau ở trạm xe; ô tô cắt mép và bị che; nón giao thông. | R-14, R-13, R-03, R-12 | `person` cho người tách được từ 20 px; đoạn dính nhau `ignore_region`; `car` theo ngưỡng che và cắt mép; không gán nón. | Người nhỏ, dính nhau. |
| img_033 | Ma-nơ-canh sau cửa kính, phản chiếu, gương xe trắng bị cắt gần hết. | R-06, R-08, R-09, R-03 | Không gán gì; ảnh để trống. | Không có vật thật nhận ra chắc; phần xe có thể là xe chủ. |
| img_034 | Người băng đường, ô tô đen bị che, người trên xe máy cắt mép. | R-04, R-02, R-03, R-13 | `person` cho người băng đường; `car` với mức che; `motorcycle` + `person` (`truncated = yes`); xe xa dưới 20 px không gán. | Chỉ cần ngưỡng % rõ ràng. |
| img_035 | Ba auto-rickshaw, một xe chở bó hàng khổng lồ; xe máy; ô tô trắng. | R-18, R-11, R-04 | `ignore_region` cho từng auto-rickshaw (không gồm bó hàng); không gán người ngồi trong; `motorcycle` + `person`; `car` cho ô tô trắng. | Xe ba bánh có động cơ không có lớp. |
| img_036 | Xe tải nhỏ xanh có bạt đỗ vỉa hè; xe máy đỗ dày bị che và cắt mép. | R-14, R-02, R-03 | `truck`; `motorcycle` cho xe tách được (mức che và `truncated` theo ngưỡng); cụm dày phía sau `ignore_region`. | Xe máy chồng sát nhau. |
| img_037 | Bán tải gắn nắp thùng cao, xe buýt cắt mép trái, người đi bộ. | R-20, R-19, R-03 | `truck` cho bán tải; `bus` (`truncated = yes`); `person` cho người đi bộ. | Còn thấy cabin tách thùng nên không phải van hay car. |
| img_038 | Tủ kính đầy xe đồ chơi và thú nhồi bông. | R-06, R-22 | Không gán gì; ảnh để trống. | Đồ chơi không phải xe thật. |
| img_039 | Phố nhỏ nhìn xa: xe máy đỗ ở tiền cảnh, người và xe nhỏ ở xa, xe đạp. | R-13, R-14, R-12 | `motorcycle` cho xe đỗ; `person` và `motorcycle` xa nếu từ 20 px; cụm xa dưới 20 px `ignore_region`; không gán xe đạp, người đạp xe gán `person`. | Nhiều vật nhỏ ở xa. |
| img_040 | Taxi cắt mép, người đội mũ sát ống kính, xe máy gần như ngoài khung; tuk-tuk và buýt ở nền. | R-03, R-05, R-04, R-18, R-19 | `car` (`truncated = yes`), không gán tài xế trong taxi; `person` (`truncated = yes`), không gán xe máy chỉ lộ gương; `ignore_region` cho tuk-tuk; `bus` cho buýt. | Theo R-03, R-04, R-05, R-18. |

Ca chưa chắc: tick `can_review` trên box hoặc ghi vấn đề cấp ảnh vào `submission/qc/image_issues.csv` (R-23), kèm mã quy tắc trong nhật ký QC.

## 5. QC và phiên bản

### 5.1 Ngưỡng QC (lấy từ `submission/taxonomy.json`, mục `rules`, không đổi)

| Tham số | Giá trị | Ý nghĩa |
|---|---|---|
| `min_box_height_px` | 20 | Vật cao dưới ngưỡng không gán từng vật (R-13). |
| `match_iou` | 0.5 | IoU tối thiểu để ghép hai box thành một cặp. |
| `large_px` | 50 | Vật cao từ 50 px dùng ngưỡng IoU lớn. |
| `iou_thr_large` | 0.85 | Ngưỡng IoU cho vật cao từ 50 px. |
| `iou_thr_small` | 0.7 | Ngưỡng IoU cho vật cao 20–50 px. |
| `iaa_iou` | 0.85 | IAA: IoU trung bình tối thiểu trên các cặp đã ghép. |
| `iaa_class` | 0.95 | IAA: tỉ lệ cùng lớp tối thiểu. |
| `error_rate_img_max` | 0.02 | Tỉ lệ ảnh lỗi tối đa để Release. |

Kiểm bằng mắt theo thứ tự: đúng vật cần gán, đủ số đối tượng, đúng lớp, sát biên, đúng thuộc tính.

Quyết định lô:
1. **Hold**: guideline chưa đủ rõ hoặc dữ liệu hỏng.
2. **Rework**: hơn 2% ảnh lỗi, lỗi lặp cùng kiểu, hoặc IAA dưới ngưỡng.
3. **Release**: đạt mọi ngưỡng.

### 5.2 Ai quyết ca mơ hồ

- Khi gán: tick `can_review` hoặc ghi `image_issues.csv`; không tự đoán, không hỏi đồng đội.
- Reviewer xét mọi box có `can_review` và vấn đề cấp ảnh.
- Ca IAA thấp do cả nhóm quyết ở buổi calibration: mỗi quyết định một dòng trong `calibration_decisions.csv`, kèm mã `R-xx`.
- Quyết định đổi cách chọn vật, lớp, box hay thuộc tính thì tăng phiên bản (v1.1) và ghi các ca bị ảnh hưởng.

### 5.3 Giải quyết bất đồng giữa các bản cá nhân (Pha 2)

Cách chốt: mỗi bản cá nhân là một phiếu, lấy theo **số đông**. Bản không nói tới điểm đó thì không tính phiếu. Khi hòa, chọn phương án khớp với mục tiêu ở mục 1 và nhất quán với các quy tắc đã chốt; lý do ghi ở cột cuối.

| # | Điểm bất đồng | Phương án và người chọn | Chốt v1.0 | Lý do |
|---|---|---|---|---|
| 1 | Lớp cho xe buýt | Thêm `bus`: 036, 052 · Không gán / ignore (phương án B, kèm đề xuất thêm `bus`): 055, 070 · 035 không nói | **Thêm `bus`** (R-19) | Hòa 2–2. 055 và 070 đều tự đề xuất thêm `bus` nên không bản nào phản đối lớp này. Xe buýt xuất hiện ở 5 ảnh; bỏ trống một xe to là trái mục tiêu "không bỏ sót xe thật". |
| 2 | Ranh giới van / bus | Theo số chỗ (dưới 9 / từ 16): 052 · Theo hình dáng (cửa sổ khách, cửa lên xuống): 036 | **Theo hình dáng** (R-19) | Số chỗ không đếm được trên ảnh và để hở khoảng 9–15 chỗ; dấu hiệu nhìn thấy thì hai người chọn giống nhau. |
| 3 | Xe ba bánh có động cơ | Không có lớp (phương án B): 035, 055, 070 · Lớp `tuk_tuk`: 052 · Lớp `three_wheeler`: 036 | **Không có lớp, phủ `ignore_region`** (R-18) | Số đông 3/5. Phủ ignore theo 055 để mô hình không bị phạt khi phát hiện xe thật. |
| 4 | Xích lô đạp | Không gán xe, người gán `person`: 052, 055, 070 · `three_wheeler`: 036 · Không gán gì: 035 | **Không gán xe, người gán `person`** (R-12) | Số đông 3/5. |
| 5 | Xe đạp | Không gán xe, người gán `person`: 052, 055, 070 · Lớp `other_vehicle`: 036 | **Không gán xe** (R-12) | Số đông 3/5; giữ taxonomy gọn. |
| 6 | Mô tô sidecar | `motorcycle` ôm sidecar: 036, 052, 070 · Lớp nhìn rõ nhất + `can_review`: 055 · Không gán: 035 | **`motorcycle` ôm sidecar** (R-21) | Số đông 3/5. |
| 7 | Người trong ô tô kín | Không gán: 052, 055, 070 · Gán nếu thấy rõ qua kính: 036 | **Không gán** (R-05) | Số đông 3/5. |
| 8 | Người trong tuk-tuk, xe lam | Gán: 036, 052 · Không gán: 055, 070 | **Không gán** (R-05, R-18) | Hòa 2–2. Chọn không gán vì xe đã phủ `ignore_region` (điểm 3), người bên trong nằm trong vùng bỏ qua. |
| 9 | Người trong ô tô mui trần | Gán: 036, 052, 055 · Không gán: 070 | **Gán** (R-05) | Số đông 3/5. |
| 10 | Ngưỡng `occluded_level` | 10/40/80%: 055, 070 · 5/30/70/90%: 052 · 0/30/70%: 036 · Định tính: 035 | **10/40/80%** | Nhiều phiếu nhất (2/5). |
| 11 | Ngưỡng `truncated` | Từ 10% ra ngoài ảnh: 052, 055, 070 · Bất kỳ phần nào: 036 | **Từ 10%** | Số đông 3/5. |
| 12 | Vật nhòe | Gán + `can_review`: 055, 070 · Gán bình thường khi xác định được biên: 036, 052 · Không gán: 035 | **Biên rõ: gán thường; biên mờ: gán + `can_review`** (R-15) | Hòa 2–2 giữa hai cách cùng chọn gán; quy tắc chốt khớp cả bốn bản. |
| 13 | Vật nhỏ, người bị che một phần | Gán mọi vật từ 20 px nhận ra được: 036, 052, 055, 070 · Chỉ gán người thấy nguyên người: 035 | **Ngưỡng 20 px** (R-13) | Số đông 4/5. |
| 14 | Xe trưng bày, xe hình lạ | Gán theo lớp gần nhất: 052, 055, 070 · Chỉ gán khi chắc là xe hoàn chỉnh: 036 · Không gán: 035 | **Gán, nghi là vỏ thì `can_review`** (R-17) | Số đông 3/5; lấy thêm ý review của 036. |
| 15 | Ngưỡng cụm dày | Từ 5 vật: 052, 070 · Không đặt số: 036, 055 | **Từ 5 vật** (R-14) | Hòa 2–2. Chọn ngưỡng đo được theo mục tiêu ở mục 1. |
| 16 | Bán tải | `truck`: 036, 052 · Tạm `car`: 070 · Chưa chốt: 055 | **`truck`** (R-20) | Nhiều phiếu nhất (2/5); 070 chỉ chọn tạm. |
| 17 | Xe chở xe | 1 `truck` + `car` cho từng ô tô: 036, 052, 070 · Chưa chốt: 055 | **Như trên** (R-10) | Số đông 3/5. |
| 18 | Vật bị cột cắt thành 2 mảnh | 1 box bao chung: 036 · Chưa chốt: 055 | **1 box** (R-01) | Chỉ một bản đưa phương án; giữ một vật một box. |

Các điểm còn lại (mục tiêu, 6 lớp gốc, vật không thật, hình in, phản chiếu, xe chủ, cách vẽ box, người và xe máy, hàng chở, tách che với cắt mép, `can_review`, ảnh trống, ngưỡng QC) mọi bản có nói tới đều quyết định giống nhau.

### 5.4 Lịch sử phiên bản

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên. | — |
| v0.1 (cá nhân) | 5 bản cá nhân: 035, 036, 052, 055, 070. | Pha 1, soi 40 ảnh. |
| v1.0 | Hợp nhất 5 bản theo số đông. Thêm lớp `bus`. Xe ba bánh có động cơ không có lớp, phủ `ignore_region`. Ngưỡng che 10/40/80%, `truncated` từ 10%, cụm dày từ 5 vật, vật nhỏ dưới 20 px. Quy tắc R-01 đến R-23. | img_002, img_004, img_009, img_012, img_022, img_023, img_035, img_037 (chi tiết ở 5.3) |
