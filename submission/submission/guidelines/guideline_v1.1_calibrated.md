# Guideline gán nhãn — v1.1 — Nhóm Zero_Risk

- Phiên bản: v1.1 (sau calibration Pha 5)
- Công cụ: CVAT Rectangle, xuất COCO 1.0.
- Taxonomy đi kèm: `submission/taxonomy.json` (`version` = `v1.0`).

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

| Lớp | Định nghĩa | Ví dụ (ảnh) | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con, SUV, taxi dạng ô tô con và bán tải không có thùng chở hàng. | `img_023` (Honda Civic), `img_012` (SUV xám), `img_040` (taxi), `img_024` (xe cổ đen trưng bày) | Bán tải có thùng hở là `truck` (R-20). Xe đồ chơi hoặc mô hình bị loại (R-06). |
| `truck` | Xe tải có cabin và thùng, sàn hoặc khung chở hàng riêng, gồm bán tải có thùng chở hàng. | `img_002` (xe chở ô tô), `img_036` (xe tải nhẹ), `img_037` (bán tải có nắp thùng) | Ô tô được chở trên sàn có box `car` riêng nếu đủ điều kiện (R-10). Bán tải không có thùng là `car` (R-20). |
| `van` | Xe cỡ nhỏ có khoang kín liền thân với cabin. | `img_005` (xe bạc phía sau nhóm người) | Xe tải có thùng rời ở `img_036` và bán tải có nắp thùng ở `img_037` đều là `truck` (R-20). |
| `motorcycle` | Xe máy, scooter và mô tô, kể cả mô tô gắn sidecar. | `img_026` (xe máy chở hai người), `img_022` (mô tô gắn sidecar) | Tuk-tuk là `three_wheeler` (R-18). Người đi xe có box `person` riêng (R-04). |
| `person` | Người thật nhìn trực tiếp, gồm người đi bộ, lái xe và hành khách. | `img_026` (người lái và người ngồi sau), `img_034` (người đi bộ qua đường) | Tượng ở `img_017`, chân dung in ở `img_018` và ma-nơ-canh ở `img_033` đều bị loại (R-06, R-07). |
| `ignore_region` | Vùng chứa vật thuộc phạm vi nhưng quá nhỏ hoặc chen dày đến mức không tách được từng vật. | `img_021` (cụm người và xe phía xa không tách được theo R-13/R-14) | Người hoặc xe đủ kích thước và còn tách được thì gán riêng (R-13/R-14). Vật giả bị loại theo R-06. |
| `three_wheeler` | Xe có ba bánh thuộc kết cấu chính, gồm tuk-tuk, xe lam, ba gác máy và xích lô. | `img_004`, `img_016`, `img_035`, `img_040` (tuk-tuk), `img_023` (xích lô) | Mô tô gắn sidecar ở `img_022` là `motorcycle` (R-21). |
| `bus` | Xe buýt hoặc xe khách được thiết kế để chở nhiều hành khách, gồm cả xe buýt cỡ nhỏ (minibus). | `img_037` (xe buýt phía sau bán tải), `img_009` (minibus) | Xe khoang kín cỡ nhỏ liền thân ở `img_005` là `van`. Dựa vào bố trí khoang khách và cửa lên xuống khi xét `bus` (R-19). |
| `bicycle` | Xe đạp hai bánh chạy bằng bàn đạp, gồm cả xe đạp chở hàng nhận diện được. | `img_006` (xe đạp bên phải), `img_029` (xe đạp cạnh xe máy chở thanh dài), `img_039` (xe đạp ở giữa đường). | Xích lô ba bánh là `three_wheeler`. Xe máy là `motorcycle`. Với xe có bàn đạp nhưng chưa rõ có động cơ hay không, chuyển reviewer (R-24). |
| `other_vehicle` | Phương tiện đường bộ thật, nhận diện được loại xe và nằm ngoài các lớp phương tiện cụ thể trong bảng. | Chưa có ví dụ xác nhận sau khi tách `bicycle`. | Xe đạp hai bánh ở `img_039` thuộc `bicycle`. Xe chưa rõ loại vì bị che hoặc nhòe cần review (R-25, R-24). |

**Vật ngoài phạm vi.** Dùng R-12 cho vật ngoài phạm vi và R-06 đến R-09 cho vật giả, hình in, phản chiếu và xe chủ. Dùng lớp cụ thể trước khi xét `other_vehicle`. Dùng `ignore_region` cho vật nhỏ hoặc cụm không tách được theo R-13 và R-14.

Điều kiện gán người trong hoặc trên phương tiện áp dụng theo R-05. Phân biệt `van` và `bus` theo R-19.

### Thuộc tính

| Thuộc tính | Giá trị | Cách chọn | Ghi chú |
|---|---|---|---|
| `occluded_level` | `0` | 0%, không bị vật khác che. | Ước lượng tỷ lệ trên hình chiếu của vật trong khung ảnh. |
| `occluded_level` | `1` | Trên 0% đến 30%. | Chỉ tính phần bị vật khác che. |
| `occluded_level` | `2` | Trên 30% đến 70%. | Chỉ tính phần bị vật khác che. |
| `occluded_level` | `3` | Trên 70%, vẫn đủ dấu hiệu nhận diện vật. | Nếu chưa xác định được lớp, làm theo R-02 và R-23. |
| `truncated` | `no` | Vật nằm trọn trong ảnh, kể cả khi chạm mép. | |
| `truncated` | `yes` | Mép ảnh cắt mất bất kỳ phần nào của vật. | Phần ngoài ảnh không tính vào mức che. |
| `can_review` | tick / không | Tick khi đã vẽ được box nhưng còn nghi về lớp, biên hoặc thuộc tính. | Chưa vẽ được box đáng tin thì ghi ID ảnh, vùng và lý do theo R-23. Trong bản nhãn cuối sau calibration, cờ này được rà soát và hạ về false. |

Người đang ngồi trên xe không làm tăng mức che của chính chiếc xe đó. Độ nhòe xử lý theo R-15.

## 3. Quy tắc vẽ box

| Rule ID | Nhóm | Quy tắc | Ảnh ví dụ | Lý do |
|---|---|---|---|---|
| **R-01** | Vẽ box | Hình chữ nhật nhỏ nhất ôm sát phần **nhìn thấy** của vật, lệch tối đa khoảng 2 px mỗi cạnh; gồm gương, tay lái, chân chống nếu thấy. Không ngoại suy phần bị che hoặc nằm ngoài ảnh. Vật bị cột hay cây cắt thành nhiều mảnh: **một** box bao chung các mảnh. Đo chiều cao theo pixel ảnh gốc. | img_005, img_018 | Mọi người cùng một cách vẽ thì IoU cao, không ai phải đoán phần khuất. |
| **R-02** | Che khuất | Tính phần bị vật khác che trong khung theo ngưỡng 0/30/70%. Hai vật chồng nhau vẫn xét riêng. Nếu chưa xác định được lớp, ghi vấn đề cấp ảnh. Khi đã chọn được lớp có căn cứ và nghi về biên hoặc mức che, tick `can_review`. | img_005, img_024, img_036 | Phần ngoài ảnh và độ nhòe không tính vào mức che. |
| **R-03** | Cắt mép | Box dừng ở mép ảnh. Đặt `truncated=yes` nếu mép cắt mất bất kỳ phần nào của vật (kể cả chỉ bị cắt mất một phần tay/chân/thân như tại img_034). Chạm mép nhưng vẫn thấy trọn vật thì đặt `no`. | img_005, img_016, img_026, img_034, img_040 | Ghi cắt mép riêng với che khuất. |
| **R-04** | Người và xe máy | Một `person` cho mỗi người và một `motorcycle` cho mỗi xe máy đủ điều kiện. Box người ôm cơ thể và đồ đang mặc hoặc đeo. Box xe ôm bộ phận xe nhìn thấy. Hai box có thể chồng nhau. Xe chỉ lộ một chi tiết không đủ nhận diện thì xử lý theo R-23. | img_005, img_010, img_026 | Hai người trên một xe máy đủ điều kiện tạo ba box. |
| **R-05** | Người trong xe | Người ngồi trong xe **kín hoặc có mui** (ô tô, van, bus, xe lam, tuk-tuk) nhìn qua kính hay khung: không gán. Người trên phương tiện **hở** (xe máy, xe đạp, xích lô, sidecar, ô tô mui trần): gán `person`. | img_004, img_016, img_019, img_023, img_040 | Người trong xe kín không phải mục tiêu riêng; người trên xe hở là người thật thấy rõ. |
| **R-06** | Vật không thật | Tượng, ma-nơ-canh, mô hình, đồ chơi, linh vật nhồi bông, xe điêu khắc, xe bằng đá: không gán, không phủ `ignore_region`. | img_011, img_017, img_019, img_027, img_028, img_033, img_038 | Gán vật không thật dạy mô hình báo có người hay xe khi không có. |
| **R-07** | Hình in | Người hay xe in trên biển quảng cáo, tranh tường, poster, màn hình: không gán. | img_018, img_026 | Không phải vật thật trong cảnh. |
| **R-08** | Phản chiếu | Hình phản chiếu trong gương, kính, mặt đường ướt: không gán. Người hay xe thật nhìn **xuyên qua** kính trong suốt vẫn xét bình thường. | img_015, img_020, img_031, img_033 | Không phải vật thật tại vị trí đó. |
| **R-09** | Xe chủ | Bộ phận của xe mang camera (gương chiếu hậu, nắp capo, taplo, kính): không gán. | img_020, img_033 | Không phải đối tượng cần phát hiện. |
| **R-10** | Xe chở xe | Một `truck` cho xe chở (đầu kéo + sàn hoặc khung). Thêm một `car` cho mỗi ô tô thật thấy được trên sàn và cao từ 20 px; box được chồng nhau, mức che theo bảng. Ô tô nằm trên nóc ô tô khác: gán cả hai. Rơ-moóc hay thùng kéo phía sau xe: không gán box riêng. | img_002, img_024, img_007 | Cả xe chở và xe được chở đều là xe thật trên đường. |
| **R-11** | Hàng hóa | Hàng chở trên xe (bó hàng, thùng carton, bao tải, thanh dài, tượng trên thùng) không đưa vào box xe. Box chỉ ôm thân xe và các bộ phận gắn cố định. | img_012, img_028, img_029, img_035 | Hàng nhô ra làm box dao động; loại ra để box ổn định. |
| **R-12** | Ngoài phạm vi | Xe đẩy bán hàng, nón giao thông, biển báo, ghế, động vật và hàng hóa không gán box riêng. Vật ngoài phạm vi không tạo `ignore_region`. Xét người thật cạnh vật theo định nghĩa `person`. | img_003, img_004, img_029, img_032 | Loại vật ngoài phạm vi theo tiêu chí rõ ràng. |
| **R-13** | Vật nhỏ | Vật có box nhìn thấy cao **dưới 20 px**: không gán từng vật, gộp vào `ignore_region` nếu thành cụm. Vật từ 20 px trở lên, nhận ra lớp và tách được ranh giới: gán bình thường, kể cả khi bị che một phần. | img_001, img_013, img_025, img_039 | Khớp `min_box_height_px` = 20 của taxonomy. |
| **R-14** | Cụm dày | Từ **5 vật** sát nhau trở lên không tách được ranh giới, hoặc đều dưới 20 px: một `ignore_region` ôm sát cả cụm. Vật to, rõ ở phía trước hoặc cạnh cụm vẫn gán riêng. Không dùng `ignore_region` để né vật rõ. | img_008, img_019, img_021, img_030, img_032, img_036 | Bỏ sót trong vùng này không tính lỗi nên phải dùng đúng chỗ. |
| **R-15** | Mờ, nhòe | Nhận ra lớp và xác định được biên: gán bình thường. Nhận ra lớp nhưng biên mờ: gán và tick `can_review`. Chỉ còn vệt mờ không nhận ra lớp: không gán, hoặc ghi vấn đề cấp ảnh nếu nghi là vật thuộc phạm vi. | img_014, img_015 | Tránh đoán lớp từ vệt mờ nhưng không bỏ sót vật thật. |
| **R-16** | Tối, ngược sáng | Gán vật thấy rõ lớp. Vùng tối không đoán được thì không gán; tick `can_review` nếu lưỡng lự. | img_007, img_013, img_026, img_031 | Giữ nhãn đáng tin thay vì đoán. |
| **R-17** | Xe thật hình lạ | Phương tiện thật đang chạy hoặc trưng bày (xe cổ, xe giả tưởng có biển số và người lái) gán theo lớp gần nhất. Nghi là vỏ hay mô hình: tick `can_review`. | img_019, img_024 | Là phương tiện thật trong cảnh. |
| **R-18** | Xe ba bánh | Gán `three_wheeler` cho tuk-tuk, xe lam, ba gác máy, auto-rickshaw và xích lô đạp đủ điều kiện. Bánh thứ ba phải thuộc kết cấu chính. Mô tô gắn sidecar theo R-21. Loại hàng chở theo R-11 và xét người theo R-05. | img_004, img_016, img_023, img_035, img_040 | Hình dạng rõ thì xét từng xe trước khi áp dụng R-13 và R-14. |
| **R-19** | Xe buýt và van | Gán `bus` khi thấy thân xe chở khách: hàng cửa sổ khách, cửa lên xuống, khoang khách, kể cả minibus. Xe khoang kín liền cabin, cỡ nhỏ, không có các đặc điểm trên: `van`. Không xét theo số chỗ vì không đếm được trên ảnh. | img_005, img_009, img_031, img_037, img_040 | Xe buýt khác van về kích cỡ và hành vi giao thông; dấu hiệu nhìn thấy được thì hai người chọn giống nhau. |
| **R-20** | Bán tải | Bán tải có thùng chở hàng là `truck` dù thùng trống, hở hay có nắp. Bán tải xác định được là không có thùng chở hàng là `car`. Nếu không thấy đủ kết cấu để phân biệt, áp dụng R-23. | img_001, img_012, img_028, img_037 | Dựa vào cấu trúc của xe và tách hàng hóa theo R-11. |
| **R-21** | Xe có sidecar | Một `motorcycle` ôm cả xe và sidecar. Người lái và người ngồi trong sidecar mỗi người một `person`. | img_022 | Sidecar là bộ phận gắn bên hông mô tô. |
| **R-22** | Ảnh trống | Ảnh không có vật nào thuộc phạm vi: để trống, vẫn tính là đã gán. | img_017, img_020, img_027, img_033, img_038 | Ảnh không có vật cần gán là kết quả hợp lệ. |
| **R-23** | Chưa chắc | Tick `can_review` trên box khi gán độc lập. Sau calibration, reviewer và nhóm thống nhất phân giải toàn bộ cờ này. | mọi ảnh | Giữ gán độc lập để IAA có ý nghĩa, hiệu chỉnh đồng thuận ở Pha 5. |
| **R-24** | Xe đạp hai bánh | Gán `bicycle` cho xe đạp hai bánh chạy bàn đạp đã nhận diện được. Box gồm phần xe nhìn thấy và phụ kiện gắn cố định, loại người và hàng chở. Tách box `person` cho người đạp đủ điều kiện. Xe có bàn đạp nhưng chưa rõ có động cơ thì áp dụng R-23. | img_006, img_029, img_039 | Phân biệt xe đạp, xe máy và xích lô ba bánh. |
| **R-25** | Phương tiện khác | Chỉ gán `other_vehicle` khi đã nhận diện được loại phương tiện đường bộ thật, loại đó không có lớp cụ thể và không thuộc nhóm ngoài phạm vi ở R-12. Xe chưa rõ loại thì áp dụng R-23. | Đối chứng img_039 | Xe đạp đã có lớp riêng. Chưa xác nhận ví dụ dương tính cho lớp này trong bộ ảnh. |

## 4. Ca khó và kết quả calibration

Áp dụng v1.1 cho cả 40 ảnh. "Loại theo quy tắc" là không gán vì quy tắc nói rõ. "Đã hiệu chỉnh" là các ca được chốt đồng thuận ở Pha 5.

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_001 | Quán vỉa hè ban đêm: bán tải trắng cắt mép, người đứng cạnh xe, người ngồi quán ở xa mờ. | R-20, R-03, R-13, R-11 | `truck` cho bán tải (`truncated` theo ngưỡng); `person` cho người đứng; người xa gán nếu từ 20 px và nhận ra được; không gán hàng trên thùng. | Bán tải có thùng là truck; người xa theo ngưỡng 20 px. |
| img_002 | Xe chở 8 ô tô trên hai sàn, ô tô chồng và che nhau. | R-10, R-02 | Một `truck` cho cả xe chở; thêm `car` cho từng ô tô thấy được từ 20 px, mức che theo bảng. | Xe chở và xe được chở đều là xe thật. |
| img_003 | Chợ vỉa hè: người gánh hàng, người đứng cạnh xe đạp, xe máy phía sau. | R-04, R-12, R-13, R-24 | Gán người, xe máy và xe đạp `bicycle` khi đủ điều kiện. Loại quang gánh và hàng. | Xe đạp hai bánh có lớp riêng. |
| img_004 | Hai tuk-tuk xanh đỗ cạnh nhau, tài xế ngồi trong xe. | R-05, R-12, R-18 | Một `three_wheeler` cho mỗi tuk-tuk đủ điều kiện. Loại tài xế trong xe có mui và nón giao thông. | Phân biệt xe, người trong xe có mui và vật ngoài phạm vi. |
| img_005 | Van xám phía sau; 5–6 người đi xe máy sát ống kính, che nhau; xe máy chỉ lộ gương hoặc tay lái. | R-02, R-03, R-04, R-23 | Gán `van` cho xe phía sau và tách từng người. Với mỗi xe máy, gán riêng nếu nhận diện được và box đủ ngưỡng. | Cùng nguyên tắc tách người và xe như img_026. |
| img_006 | Ngã tư đông xe máy, người đạp xe bên phải, taxi trắng cắt mép và bị che. | R-02, R-03, R-04, R-14, R-24 | Gán từng xe máy và từng người đủ điều kiện. Xe đạp bên phải là `bicycle`, người đạp là `person`. Taxi là `car`; chỉ khoanh ignore cho cụm không tách được. | Tách lớp, mức che và cắt mép theo từng đối tượng. |
| img_007 | Hẻm kho tối: bán tải trắng kéo thùng hàng, xe tải Isuzu cắt mép, người trong bóng tối. | R-20, R-10, R-03, R-16 | `truck` cho bán tải (thùng kéo phía sau không gán riêng) và `truck` cho Isuzu (`truncated = yes`); `person` chỉ khi nhận ra chắc. | Giữ nhãn đáng tin thay vì đoán. |
| img_008 | Bãi xe máy ban đêm, hai người áo cam ngồi trên xe, xe dày sát nhau. | R-04, R-14, R-16 | `motorcycle` + `person` cho xe và người thấy rõ; cụm dày phía sau `ignore_region`. | Không tách được ranh giới từng xe. |
| img_009 | Minibus giữa đường, xe máy cắt mép, người ở xa. | R-19, R-03, R-13 | `bus` cho minibus (thấy rõ khoang khách/cửa lên xuống); `motorcycle` `truncated = yes`; người xa theo ngưỡng 20 px. | Thấy cửa sổ khách và cửa lên xuống; chốt bus theo R-19. |
| img_010 | Chợ vỉa hè: phụ nữ ngồi xe máy, hai người trên xe phía sau, người bán rau, xe đạp chở rau. | R-04, R-11, R-24 | Tách người và xe máy. Gán `bicycle` cho xe chở rau nếu đủ nhận diện, loại rau khỏi box. Người bán là `person`. | Căn cứ vào phần khung xe đạp nhìn thấy. |
| img_011 | Tượng lớn kim loại và xe máy kim loại ở hội chợ; xe lớn bị che ở nền; người ở xa nhỏ. | R-06, R-19, R-13 | Không gán tượng và xe kim loại; xe lớn ở nền gán `bus` nếu nhận ra; người dưới 20 px gộp `ignore_region` nếu thành cụm. | Tác phẩm điêu khắc không phải xe thật. |
| img_012 | Bán tải trắng thùng hở chở thùng carton, bao tải; SUV xám bên trái. | R-20, R-11, R-03 | `truck` cho bán tải, box không gồm hàng; `car` cho SUV. | Bán tải có thùng là truck. |
| img_013 | Phố đêm sau mưa, quán ngoài trời, nhiều nón giao thông, vài người nhỏ, xe ở xa. | R-13, R-14, R-16, R-08 | `person` cho người từ 20 px; `car` và `motorcycle` nếu từ 20 px và tách được; cụm nhỏ `ignore_region`; không gán nón, phản chiếu. | Ban đêm, vật nhỏ, đường ướt. |
| img_014 | Hai người đi bộ rõ nét; xe máy phía sau nhòe do chuyển động. | R-15, R-04 | `person` cho hai người; xe và người nhòe vẫn nhận ra thì gán; vệt không nhận ra thì không gán. | Không bỏ sót vật thật, không đoán từ vệt mờ. |
| img_015 | Mưa: ô tô cắt mép hai bên, xe nền nhòe, phản chiếu trên đường ướt. | R-03, R-15, R-08 | `car` cho hai xe ở rìa (`truncated = yes`); xe nhòe nhận ra thì gán; không gán phản chiếu. | Chỉ gán vật thật nhận ra được. |
| img_016 | Hai tuk-tuk cắt mép, người đứng sát ống kính, xe tay ga, van trắng. | R-03, R-04, R-05, R-18 | Gán từng tuk-tuk `three_wheeler` nếu đủ điều kiện và tick cắt mép khi có. Xét người đứng, xe tay ga, người trên xe tay ga và van riêng. | Tuk-tuk là xe ba bánh có mui. |
| img_017 | Tượng người cưỡi ngựa đội nón giao thông. | R-06, R-22 | Không gán gì; ảnh để trống. | Tượng không phải người thật. |
| img_018 | Biển quảng cáo có mặt người; ô tô đỗ sau hàng rào lưới, có xe cắt mép. | R-07, R-02, R-03 | Không gán mặt trên biển; `car` cho từng ô tô thấy rõ lớp, mức che do hàng rào, `truncated` nếu cắt mép. | Hình in không phải vật thật. |
| img_019 | Xe vàng giả tưởng có biển số, người lái, chuột nhồi bông; đám đông sau hàng rào. | R-17, R-05, R-06, R-14 | `car` cho xe vàng; người lái gán `person` nếu xe hở mui và thấy trực tiếp; không gán chuột; đám đông chen dày phủ `ignore_region`, người tách được giữ `person`. | Xe thật đang chạy; linh vật không phải người; cụm người theo R-14. |
| img_020 | Gương chiếu hậu xe chủ phản chiếu đường và xe. | R-08, R-09, R-22 | Không gán gì; ảnh để trống. | Phản chiếu và bộ phận xe chủ. |
| img_021 | Kẹt xe máy dày đặc: vài xe phía trước rõ, hàng chục xe và người nhỏ phía sau, ô tô cắt mép. | R-04, R-13, R-14, R-03 | `motorcycle` + `person` cho xe phía trước rõ; phần sau `ignore_region`; `car` `truncated = yes`; xe buýt nền xa trên 20 px giữ `bus`. | Không tách được ranh giới từng vật thì ignore; xe nhận ra được giữ nguyên. |
| img_022 | Ảnh đen trắng cổ: mô tô gắn sidecar, người ngồi trong sidecar. | R-21, R-13 | Một `motorcycle` ôm cả sidecar; `person` cho người lái và người trong sidecar nếu từ 20 px. | Sidecar là bộ phận của mô tô. |
| img_023 | Xích lô đạp cạnh ô tô đen, người trên xe máy. | R-04, R-05, R-18 | Gán `three_wheeler` cho xích lô và `person` riêng cho người đạp nếu đủ điều kiện. Xét xe máy, người trên xe và xe con riêng. | Ba bánh thuộc kết cấu chính, kể cả xe đạp chân. |
| img_024 | Ô tô vàng nhỏ nằm trên nóc ô tô đen trong bảo tàng; nhiều xe cổ cắt mép. | R-10, R-17, R-02, R-03 | `car` cho cả hai xe (box chồng nhau, xe dưới ghi mức che); `car` cho các xe cổ thấy rõ, `truncated = yes` nếu cắt mép. | Xe trưng bày là xe thật. |
| img_025 | Bờ biển: người đi bộ, taxi vàng cắt mép, người trên xe máy, nhiều người ngồi ghế rất nhỏ. | R-13, R-04, R-03, R-12 | `person` cho người đi bộ thấy rõ; `car` (`truncated = yes`); `motorcycle` + `person` nếu từ 20 px; người dưới 20 px `ignore_region`; không gán ghế. | Vật xa nhỏ; ghế không có lớp. |
| img_026 | Đêm: hai người trên xe máy đỏ, xe cắt ở đáy ảnh, trước tranh tường. | R-04, R-03, R-07, R-16 | 1 `motorcycle` (`truncated = yes`) + 2 `person`; không gán tranh tường. | Người lái, người ngồi sau và xe là ba đối tượng. |
| img_027 | Xe bán tải bằng đá granite trên mộ. | R-06, R-22 | Không gán gì; ảnh để trống. | Là tượng đài, không phải xe thật. |
| img_028 | Bán tải cổ chở tượng ngựa trắng; SUV đỏ phía sau bị che. | R-20, R-11, R-06, R-02 | `truck` cho bán tải, box không gồm tượng ngựa; `car` cho SUV với mức che theo bảng. | Ngựa là tượng đặt trên thùng (hàng chở). |
| img_029 | Người đi xe máy chở thanh dài nhô ra, người đi xe đạp, người ở xa. | R-04, R-11, R-13, R-24 | Gán xe máy và xe đạp lần lượt là `motorcycle` và `bicycle`. Xét từng người riêng. Loại thanh dài khỏi box xe máy. | Hai loại phương tiện có box riêng. |
| img_030 | Bãi xe máy đỏ chất đống, một người đứng giữa. | R-14, R-02, R-03 | Gán xe và người phía trước tách được từ 20 px; phần dày phía sau `ignore_region`. | Không tách được từng xe. |
| img_031 | Phố đêm mưa: ô tô, xe buýt bị che, người trên lề, phản chiếu neon. | R-19, R-08, R-16, R-13 | `car`; `bus` (mức che theo bảng); `person` và `motorcycle` nếu từ 20 px; không gán phản chiếu. | Phản chiếu không phải vật thật. |
| img_032 | Hàng khoảng 30 người xếp sát nhau ở trạm xe; ô tô cắt mép và bị che; nón giao thông. | R-14, R-13, R-03, R-12 | `person` cho người tách được từ 20 px; đoạn dính nhau `ignore_region`; `car` theo ngưỡng che và cắt mép; không gán nón. | Người nhỏ, dính nhau. |
| img_033 | Ma-nơ-canh sau cửa kính, phản chiếu, gương xe trắng bị cắt gần hết. | R-06, R-08, R-09, R-03 | Không gán gì; ảnh để trống. | Không có vật thật nhận ra chắc; phần xe có thể là xe chủ. |
| img_034 | Người băng đường, ô tô đen bị che, người trên xe máy cắt mép sát phải. | R-04, R-02, R-03, R-13 | `person` cho người băng đường; `car` với mức che; `motorcycle` + `person` (`truncated = yes` cho người và xe sát mép phải bị cắt); xe xa dưới 20 px không gán. | R-03 bắt buộc truncated=yes khi mép cắt mất một phần vật. |
| img_035 | Ba auto-rickshaw, một xe chở bó hàng khổng lồ; xe máy; ô tô trắng. | R-04, R-05, R-11, R-18 | Gán một `three_wheeler` cho mỗi auto-rickshaw đủ điều kiện, bỏ bao tải khỏi box. Xét xe máy, người đi xe máy và ô tô riêng. | Box bám phần xe nhìn thấy. |
| img_036 | Xe tải nhỏ xanh có bạt đỗ vỉa hè; xe máy đỗ dày bị che và cắt mép. | R-14, R-02, R-03 | `truck`; `motorcycle` cho xe tách được (mức che và `truncated` theo ngưỡng); cụm dày phía sau `ignore_region`. | Xe máy chồng sát nhau. |
| img_037 | Bán tải gắn nắp thùng cao, xe buýt cắt mép trái, người đi bộ. | R-20, R-19, R-03 | `truck` cho bán tải; `bus` (`truncated = yes`); `person` cho người đi bộ. | Còn thấy cabin tách thùng nên không phải van hay car. |
| img_038 | Tủ kính đầy xe đồ chơi và thú nhồi bông. | R-06, R-22 | Không gán gì; ảnh để trống. | Đồ chơi không phải xe thật. |
| img_039 | Phố nhỏ nhìn xa: xe máy đỗ ở tiền cảnh, người và xe nhỏ ở xa, xe đạp. | R-13, R-14, R-24 | Xét từng xe máy, xe đạp và người tách được. Gán `bicycle` cho xe đạp đủ điều kiện. Dùng ignore sát cụm không tách được. | Phân biệt đối tượng nhỏ với cụm thật sự không tách được. |
| img_040 | Taxi cắt mép, người đội mũ sát ống kính, xe máy gần như ngoài khung; tuk-tuk và buýt ở nền. | R-03, R-04, R-05, R-18, R-19 | Xét taxi `car`, người tiền cảnh `person`, tuk-tuk `three_wheeler` và xe buýt `bus` nếu đủ điều kiện. Xe máy chỉ lộ gương cần ghi vấn đề nếu chưa xác định được box. | Áp dụng cùng điều kiện kích thước và cắt mép cho từng vật. |

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

### 5.2 Lịch sử phiên bản

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên. | — |
| v0.1 (cá nhân) | 5 bản cá nhân: 035, 036, 052, 055, 070. | Pha 1, soi 40 ảnh. |
| v1.0 | Tích hợp quy tắc chung và bảng lớp, thuộc tính của 036. Có 10 lớp, R-01 đến R-25, giữ nguyên ngưỡng QC. | img_004, img_023, img_026, img_029, img_035 và img_039. |
| v1.1 | Hiệu chỉnh câu chữ (editorial): làm rõ quy tắc cắt mép truncated=yes cho đối tượng sát mép ảnh (img_034, R-03); xác nhận minibus gán lớp bus (img_009, R-19); chuẩn hóa ranh giới ignore_region ở cụm đông người (img_019, img_021, R-14); giữ nguyên taxonomy v1.0. | Sau rà soát IAA và kết quả kiểm chứng Pha 4 (commit c086df4, phase4_review.md). |
