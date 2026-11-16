# Guideline gán nhãn — v0.1 cá nhân

- Ngày: 16/11/2026
- Người viết: Hà Thanh Tùng (2A202603036)
- Phạm vi: Pha 1, bộ 40 ảnh Day 07. Bản cá nhân dựa trên taxonomy v0.
- Công cụ: CVAT Rectangle, xuất COCO 1.0.

## 1. Mục tiêu

Tạo nhãn cho mô hình phát hiện người và phương tiện thật trong ảnh đường phố 2D. Ưu tiên phát hiện người bị che hoặc đang đi xe, đồng thời tránh nhầm tượng, hình in và phản chiếu với đối tượng thật. Giữ cả ảnh không có đối tượng trong tập dữ liệu.

## 2. Lớp và thuộc tính

### Lớp (mỗi vật thuộc đúng một lớp)

| Lớp | Định nghĩa | Ví dụ (ảnh) | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con, SUV, taxi dạng ô tô con và bán tải không có thùng chở hàng. | `img_023` (Honda Civic), `img_012` (SUV xám), `img_040` (taxi), `img_024` (xe cổ đen trưng bày) | Bán tải có thùng hở là `truck` (R-01). Xe đồ chơi hoặc mô hình bị loại (R-06). |
| `truck` | Xe tải có cabin và thùng, sàn hoặc khung chở hàng riêng, gồm bán tải có thùng chở hàng. | `img_002` (xe chở ô tô), `img_036` (xe tải nhẹ), `img_037` (bán tải có nắp thùng) | Ô tô được chở trên sàn có box `car` riêng nếu đủ điều kiện (R-04). Bán tải không có thùng là `car` (R-01). |
| `van` | Xe cỡ nhỏ có khoang kín liền thân với cabin. | `img_005` (xe bạc phía sau nhóm người) | Xe tải có thùng rời ở `img_036` và bán tải có nắp thùng ở `img_037` đều là `truck` (R-01). |
| `motorcycle` | Xe máy, scooter và mô tô, kể cả mô tô gắn sidecar. | `img_026` (xe máy chở hai người), `img_022` (mô tô gắn sidecar) | Tuk-tuk là `three_wheeler` (R-02). Người đi xe có box `person` riêng (R-03). |
| `person` | Người thật nhìn trực tiếp, gồm người đi bộ, lái xe và hành khách. | `img_026` (người lái và người ngồi sau), `img_034` (người đi bộ qua đường) | Tượng ở `img_017`, chân dung in ở `img_018` và ma-nơ-canh ở `img_033` đều bị loại (R-06). |
| `ignore_region` | Vùng chứa vật thuộc phạm vi nhưng quá nhỏ hoặc chen dày đến mức không tách được từng vật. | `img_021` (cụm người và xe phía xa không tách được theo R-05) | Người hoặc xe đủ kích thước và còn tách được thì gán riêng (R-05). Vật giả bị loại theo R-06. |
| `three_wheeler` | Xe có ba bánh thuộc kết cấu chính, gồm tuk-tuk, xe lam, ba gác máy và xích lô. | `img_004`, `img_016`, `img_035`, `img_040` (tuk-tuk), `img_023` (xích lô) | Mô tô gắn sidecar ở `img_022` là `motorcycle` (R-02). |
| `bus` | Xe buýt hoặc xe khách được thiết kế để chở nhiều hành khách, gồm cả xe buýt cỡ nhỏ. | `img_037` (xe buýt phía sau bán tải) | Xe khoang kín cỡ nhỏ liền thân ở `img_005` là `van`. Dựa vào bố trí khoang khách và cửa lên xuống khi xét `bus` (R-07). |
| `other_vehicle` | Phương tiện đường bộ thật, nhận diện được loại xe và nằm ngoài các lớp phương tiện cụ thể ở trên, chẳng hạn xe đạp hai bánh. | `img_039` (xe đạp hai bánh) | Xích lô có lớp `three_wheeler` (R-02). Xe chưa rõ loại vì bị che hoặc nhòe cần review (R-07). |

Chọn lớp cụ thể trước khi xét `other_vehicle`. Nếu chưa xác định được loại xe do che khuất, cắt mép hoặc nhòe, chuyển review. Dùng `ignore_region` theo R-05.

| Thuộc tính | Giá trị và cách chọn |
|---|---|
| `occluded_level` | Ước lượng phần diện tích hình chiếu trong khung ảnh bị vật khác che. `0` = 0%. `1` = trên 0% đến 30%. `2` = trên 30% đến 70%. `3` = trên 70% khi vẫn nhận diện được vật. Phần ngoài ảnh và độ nhòe không tính vào tỷ lệ che. |
| `truncated` | `yes` khi mép ảnh cắt mất một phần vật. `no` nếu vật còn nguyên trong ảnh, kể cả khi chạm mép. Có thể đồng thời bị che và bị cắt mép. |
| `can_review` | Tick khi đã vẽ được box mà còn nghi về lớp, biên hoặc thuộc tính. Nếu chưa vẽ được box đáng tin cậy, ghi ID ảnh, vị trí và lý do để reviewer xem. |

## 3. Quy tắc vẽ box

Dùng hình chữ nhật nhỏ nhất bao các phần nhìn thấy của cùng một vật. Nếu các phần nhìn thấy rời nhau, bao chung trong một box. Giới hạn box trong ảnh và bỏ phần chỉ suy đoán sau vật che. Đo chiều cao theo pixel ảnh gốc.

- **R-01 — Phân biệt ô tô.** Bán tải không có thùng chở hàng thuộc `car`. Bán tải có thùng chở hàng thuộc `truck`, kể cả khi thùng đang trống, để hở hoặc có nắp. Nếu chưa xác định được xe có thùng vì bị che hoặc cắt mép, chuyển review.
- **R-02 — Xe ba bánh và hàng chở.** Xét số bánh thuộc kết cấu chính. Mô tô gắn sidecar dùng một box `motorcycle` bao phần xe và sidecar nhìn thấy. Box xe bao bộ phận gắn cố định, bỏ hàng chở như túi, bao tải và vật trang trí đặt trên thùng.
- **R-03 — Người và xe.** Tách từng người và từng xe nếu đủ điều kiện. Box người bao cơ thể, quần áo và mũ bảo hiểm nhìn thấy. Box xe chỉ ôm phần thuộc xe. Box người và xe có thể chồng nhau.
- **R-04 — Xe chở xe.** Một box cho xe tải, gồm đầu kéo và sàn chở gắn với nó. Gán riêng từng xe thật trên sàn nếu phân biệt được và đủ chiều cao. Phần trống của hình chữ nhật có thể chồng lên box xe được chở.
- **R-05 — Vật nhỏ, đám đông và nhòe.** Gán riêng khi phân biệt được vật và box nhìn thấy cao từ 20 px. Dùng `ignore_region` sát vật dưới 20 px hoặc cụm không thể tách rõ. Vẫn gán riêng vật đủ điều kiện ở cạnh cụm. Nếu nhòe nhưng còn nhận ra lớp và xác định được biên, gán box. Nếu nghi có vật thuộc phạm vi mà không xác định được biên, ghi vấn đề cấp ảnh.
- **R-06 — Vật không thật và xe chủ.** Loại tượng, mô hình, ma-nơ-canh, hình in, ảnh phản chiếu và phần xe đặt máy ảnh nếu nhận ra được. Người hoặc xe thật nhìn trực tiếp qua kính vẫn được xét. Với vật trưng bày hoặc xe tạo hình lạ, dùng dấu hiệu kết cấu để xác định và chuyển review nếu còn nghi.
- **R-07 — Xe buýt và phương tiện khác.** Dùng `bus` khi nhận diện được thân xe chở khách theo hàng cửa sổ, cửa lên xuống và bố trí khoang khách. Dùng `other_vehicle` khi nhận diện được một loại phương tiện nằm ngoài những lớp cụ thể. Tách người điều khiển theo R-03.

## 4. Ca khó

Các quyết định dưới đây áp dụng cho vùng nêu trong bảng.

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_037.jpg | Bán tải trắng gắn nắp thùng cao | R-01 | Gán `truck`. | Xe có thùng chở hàng riêng dưới nắp cao. |
| img_036.jpg | Xe xanh có thùng hàng | R-01 | Gán `truck`. | Cabin và thùng hàng rời là dấu hiệu của xe tải nhẹ. |
| img_004.jpg, img_016.jpg, img_035.jpg, img_040.jpg | Tuk-tuk | R-02 | Gán `three_wheeler` cho từng xe đủ điều kiện. Ở img_035, bỏ bao tải trên nóc khỏi biên box. | Xe ba bánh là đối tượng. Bao tải là hàng chở. |
| img_023.jpg, img_022.jpg | Xích lô và mô tô sidecar | R-02, R-03 | img_023 dùng một box `three_wheeler` và một `person` cho người đạp nếu đủ điều kiện. img_022 dùng một `motorcycle` gồm sidecar. | Xích lô có ba bánh thuộc kết cấu chính. Sidecar là bộ phận gắn bên hông mô tô. |
| img_026.jpg | Hai người trên một xe máy | R-03 | Cụm tiền cảnh có hai `person` và một `motorcycle`. Ôm phần nhìn thấy của từng người và của xe. | Người lái, người ngồi sau và xe là ba đối tượng riêng. |
| img_005.jpg | Thử lại quy tắc người và xe ở tiền cảnh phải | R-03 | Với người áo xanh và xe người đó đang điều khiển, dùng một `person` và một `motorcycle` khi phần xe đáp ứng điều kiện. Xét người và xe khác riêng. | Che và cắt mép làm thay đổi biên và thuộc tính của từng đối tượng. |
| img_002.jpg | Xe tải chở ô tô trên hai sàn | R-04 | Một `truck` cho xe chở và một `car` cho mỗi ô tô trên sàn đủ điều kiện. | Xe chở và các xe được chở là các vật thể riêng. |
| img_024.jpg | Vật hình xe màu vàng trên nóc | R-04, R-06 | Loại nếu xác định là vỏ trưng bày. Nếu chưa phân biệt được với xe hoàn chỉnh, chuyển review. | Hình dáng giống xe chưa đủ để xác nhận một xe hoàn chỉnh. |
| img_014.jpg | Người và xe nhòe chuyển động | R-05 | Xét từng vật còn khoanh được. Ghi vấn đề cấp ảnh cho vệt nghi là vật thuộc phạm vi nhưng không xác định được biên. | Độ nhòe làm khó xác định biên và lớp. |
| img_021.jpg, img_039.jpg | Dòng xe chen dày và vật ở xa | R-05 | Tách vật còn rõ. Đo trên ảnh gốc và khoanh sát phần đủ điều kiện dùng `ignore_region`. | Một vùng rộng có thể che mất những vật cần gán riêng. |
| img_017.jpg, img_018.jpg, img_020.jpg, img_027.jpg, img_033.jpg, img_038.jpg | Tượng, hình in, phản chiếu, ma-nơ-canh hoặc mô hình | R-06 | Loại các vật không thật theo quy tắc. Ở img_018, vẫn xét riêng ô tô thật dưới biển quảng cáo. | Nhãn thể hiện vật thật nhìn trực tiếp trong ảnh. |
| img_012.jpg | Bán tải có thùng hở chở hàng | R-01, R-02 | Gán `truck`. Box bao phần thùng nhìn thấy và loại hàng chở. | Thùng hở là bộ phận của xe. |
| img_037.jpg | Xe buýt phía sau bán tải | R-07 | Gán `bus` cho xe phía sau nếu phần thấy đủ ngưỡng. | Xe có thân và khoang chở khách của xe buýt. |
| img_039.jpg | Xe đạp hai bánh ở gần giữa đường | R-03, R-07 | Gán `other_vehicle` cho xe đạp và `person` riêng cho người đi xe nếu từng đối tượng đủ điều kiện. | Xe đạp hai bánh có thể nhận diện riêng và nằm ngoài các lớp xe cụ thể. |
| img_028.jpg | Bán tải thật chở tượng ngựa | R-01, R-02, R-06 | Gán `truck` cho bán tải có thùng, loại tượng khỏi biên box. | Thùng là bộ phận của xe. Tượng đặt trên thùng là hàng chở. |

## 5. QC và phiên bản

- Kiểm bằng mắt theo thứ tự: có đúng vật cần gán, đủ số đối tượng, đúng lớp, sát biên và đúng thuộc tính.
- Ghép box một với một từ IoU 0,50. Ngưỡng IoU từng cặp là 0,85 cho box cao từ 50 px và 0,70 cho box thấp hơn.
- Trên các cặp đã ghép, IAA yêu cầu IoU trung bình từ 0,85 và đồng thuận lớp từ 95%. Ghi riêng số box không ghép được. Khi duyệt toàn lô, ngưỡng lỗi là 2% theo ảnh.
- Reviewer xét các box có `can_review` và vấn đề cấp ảnh. Ghi ID ảnh, vùng, lý do và mã quy tắc. Nếu thay đổi cách chọn vật, lớp, biên hoặc thuộc tính, tăng phiên bản và ghi các ca bị ảnh hưởng.

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v0 | Đặc tả khởi đầu. | Mốc đối chiếu. |
| v0.1 cá nhân | Phân lớp bán tải theo việc có thùng chở hàng, bổ sung `three_wheeler`, `bus` và `other_vehicle`, định lượng thuộc tính và bổ sung quy tắc vẽ box. | Giải quyết ranh giới lớp ở img_037, img_023 và các ca chồng vật ở img_026, img_002. |
