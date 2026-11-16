# Guideline gán nhãn — v0.1 (bản cá nhân) — 055 | Nhóm Zero_Risk

> Bản cá nhân Pha 1, viết độc lập. Không sửa lại sau họp nhóm (bản thống nhất viết ở `guideline_v1.0_agreed.md`).
> *chưa chốt, cần họp* = câu tôi chưa tự quyết, đem ra Pha 2. Mã quy tắc `R-xx` dùng trong bảng Ca khó (mục 4).

- Phiên bản: v0.1 (cá nhân, sửa từ v0)
- Ngày: 16/11/2026
- Người viết: thành viên 055

## 1. Mục tiêu

Nhãn dùng huấn luyện mô hình phát hiện **người và phương tiện tham gia giao thông thật** trên ảnh đường phố châu Á
(camera giao thông / xe tự lái).

- **Sai nguy hiểm nhất:** bỏ sót người hoặc xe thật (mô hình không thấy → không tránh).
- **Sai thứ hai:** gán vật không thật (tượng, ma-nơ-canh, hình in, phản chiếu) thành người/xe → mô hình báo động giả, phanh vô cớ.
- Nguyên tắc chung khi phân vân: *"Vật này có tự di chuyển trên đường và cần né không?"* Có → gán lớp. Không → không gán / ignore.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô chở người ≤ 9 chỗ: sedan, hatchback, SUV, taxi, xe bán tải chở người. | img_018 | Van chở hàng → `van`; mô hình/tượng xe → không gán |
| `truck` | Xe có cabin + thùng hàng tách riêng: xe tải, xe ben, xe cẩu, đầu kéo. Bán tải thùng hở: *chưa chốt, cần họp* (img_012, img_028). | img_036 | Xe chở xe khác: xe tải là `truck`, xe trên thùng xử lý theo mục 3.6 |
| `van` | Xe thùng kín liền khối cabin, cỡ nhỏ (minivan chở hàng, xe cứu thương nhỏ). | img_037 (*cần họp*) | Xe cabin kín chở khách: `car` hay `van` → *chưa chốt, cần họp* (img_037) |
| `motorcycle` | Xe 2 bánh có động cơ: xe máy, mô tô, xe tay ga. | img_010 | Xe đạp (không động cơ) → mục "ngoài taxonomy" |
| `person` | Con người thật: đi bộ, đứng, ngồi, kể cả người **ngồi trên xe** hở (xe máy, xích lô). | img_025 | Tượng, ma-nơ-canh, người trên biển quảng cáo → không gán |
| `ignore_region` | Vùng không gán từng vật: đám đông/xe chen dày, vật cao < 20 px, vùng quá nhoè/tối. | img_021 | Không dùng để "bỏ qua cho nhanh" một vật đủ to, rõ |

**Vật không thuộc lớp nào** (v0 không nói): tuk-tuk, xe ba gác/xích lô, xe buýt, xe đạp, xe ba bánh…

- Phương án A: gán vào lớp gần nhất (tuk-tuk → `car`? xe buýt → `truck`?) — dễ, nhưng dạy sai mô hình.
- Phương án B: **không gán box**, phủ `ignore_region` nếu vật to — giữ lớp sạch, mô hình không học sai, bỏ sót ở đó không tính lỗi.
- Phương án C: đề xuất thêm lớp mới (ví dụ `bus`, `three_wheeler`) → *chưa chốt, cần họp* (đổi taxonomy).
- **R-11 — Tôi chọn: Phương án B** (img_003 xe đạp, img_004 xe lam, img_009 xe buýt, img_023 xích lô). Lý do: gán vào lớp gần nhất sẽ dạy mô hình sai (xe buýt thành `truck`, xe đạp thành `motorcycle`); không gán giữ các lớp sạch. Người **ngồi trên** xe hở ngoài taxonomy (xe đạp, xích lô) vẫn gán `person`.
- Xe buýt xuất hiện nhiều (img_009, 011, 031, 037, 040) nên tôi đề xuất nhóm cân nhắc Phương án C (thêm lớp `bus`) — *chưa chốt, cần họp*.

### Thuộc tính (mức cố định + ngưỡng đo được)

| Thuộc tính | Mức | Ngưỡng (ước lượng phần thân vật bị che, so với hình dạng đầy đủ) |
|---|---|---|
| `occluded_level` | `0` | che **< 10%** (thấy gần trọn) |
| | `1` | che **10% – < 40%** |
| | `2` | che **40% – < 80%** |
| | `3` | che **≥ 80%** — vẫn đủ bằng chứng nhận ra vật; nếu không nhận ra được thì **không gán** |
| `truncated` | `yes` | box chạm mép ảnh **và** phần vật nằm ngoài khung ước lượng **≥ 10%** |
| | `no` | còn lại (chạm mép nhưng thấy gần trọn vật vẫn `no`) |
| `can_review` | tick | không chắc lớp / không chắc là vật thật / không chắc ranh giới → tick và ghi ảnh vào mục 4 |

Cách ước lượng che: tưởng tượng hình dạng đầy đủ của vật, so phần bị vật khác che. Bị **mép ảnh** cắt thì dùng `truncated`, **không** tính vào `occluded_level`.
Ngưỡng 10/40/80 là đề xuất ban đầu của tôi; chưa đối chiếu đủ trên các ảnh có vật bị che (img_006, 024, 034, 036) — *chưa chốt, cần họp*.

## 3. Quy tắc vẽ box

1. **R-01 Ôm sát phần nhìn thấy** (không vẽ phần đoán sau vật che), lệch tối đa ~2 px mỗi cạnh; gồm gương, tay lái, chân chống nếu thấy.
2. **R-02 Vật bị che:** box chỉ phần thấy; hai xe chồng nhau → 2 box riêng, xe phía sau có `occluded_level` (img_024). Vật bị cột/cây cắt làm 2 mảnh: 1 box hay chỉ mảnh lớn → *chưa chốt, cần họp*.
3. **R-03 Cắt mép ảnh:** box dừng ở mép ảnh, `truncated` theo ngưỡng mục 2.
4. **R-04 Vật không thật — KHÔNG gán:** tượng, ma-nơ-canh, người/xe trên biển quảng cáo, áp phích, màn hình, **phản chiếu** trên kính/nước, xe đồ chơi, mô hình trưng bày, xe giả tưởng/trang trí không lưu thông.
   → Tôi chọn: **chỉ không gán**, không phủ `ignore_region` (img_017 ma-nơ-canh cưỡi ngựa, img_020 ô tô trong gương, img_027 và img_038 ô tô mô hình, img_033 ma-nơ-canh).
5. **R-05 Người và xe:** người ngồi/lái xe = **1 box `person`** riêng + **1 box xe** riêng (box người được chồng lên box xe).
6. **R-06 Xe chở xe / xe chở hàng:** xe chở là 1 box; xe nằm trên thùng (img_002 — các ô tô trên thùng vẫn nhận rõ) gán hay không → *chưa chốt, cần họp*. Vật được chở không thuộc lớp nào (ngựa trên bán tải, img_028) → không gán.
7. **R-07 Xe chủ (ego):** capo, taplo, gương xe mình → không gán.
8. **R-08 Vật nhỏ:** cao **< 20 px** → không gán từng vật; nhiều vật nhỏ gần nhau → 1 `ignore_region` bao cụm (theo `rules.min_box_height_px` = 20).
9. **R-09 Đám đông / xe chen dày:** tách được từng vật cao ≥ 20 px và thấy ≥ ~50% → box riêng; phần còn lại dày không tách được → 1 `ignore_region`.
10. **R-10 Nhoè / quá tối:** vẫn nhận ra lớp → gán + `can_review` nếu ranh giới mờ (img_014); không nhận ra → không gán hoặc `ignore_region`.
11. **R-11 Vật ngoài taxonomy:** xem mục 2 (Phương án B).
12. **R-12 Không chắc lớp:** gán theo lớp nhìn rõ nhất + tick `can_review`, ghi ảnh để hỏi reviewer (img_022).
13. **R-13 Người ngồi trong xe kín** (ô tô, xe lam có mui): không gán `person`; chỉ gán người ngồi trên xe hở (img_004).

## 4. Ca khó (từ 40 ảnh — tự soi, mỗi dòng một ảnh)

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_003 | Người, xe đạp, xe máy; v0 không có lớp xe đạp | R-11, R-05 | Xe đạp không gán box; người đạp xe gán `person`; xe máy `motorcycle` | Xe đạp không tự di chuyển bằng động cơ, ép vào `motorcycle` sẽ dạy sai mô hình |
| img_004 | Xe lam (tuk-tuk) chở người | R-11, R-13 | Xe lam không gán box; người ngồi trong xe lam không gán | v0 không có lớp xe ba bánh; người trong xe có mui thấy không rõ toàn thân |
| img_009 | Xe buýt, xe máy cắt mép, người ở xa | R-11, R-03, R-08 | Xe buýt không gán (đề xuất thêm lớp `bus` — cần họp); xe máy `truncated=yes`; người xa theo ngưỡng 20 px | Không có lớp bus, người gán dễ chọn `truck` |
| img_014 | Người trên xe di chuyển nhanh bị nhoè | R-10 | Vẫn nhận ra → gán, tick `can_review` | v0 không nói về vật nhoè do chuyển động |
| img_017 | Người cưỡi ngựa bị cắt mép — là ma-nơ-canh | R-04 | Không gán | Không phải người thật; v0 ghi "gán mọi người" dễ gây nhầm |
| img_021 | Người trên xe; đám đông ở xa quá nhỏ; ô tô cắt mép | R-05, R-08, R-09, R-03 | Người trên xe gán `person`; cụm người < 20 px gộp 1 `ignore_region`; car `truncated=yes` | v0 không có quy tắc vật nhỏ / đám đông |
| img_022 | Không xác định được xe là ô tô hay xe máy, người bị che | R-12 | Gán theo lớp nhìn rõ nhất + `can_review` — **cần hỏi reviewer** | Không đủ bằng chứng để chọn lớp chắc chắn |
| img_023 | Người trên xích lô, người trên xe máy, ô tô | R-11, R-05 | Xích lô không gán box; người ngồi trên xích lô gán `person`; người trên xe máy: `person` + `motorcycle` | Xích lô ngoài taxonomy nhưng người trên đó là người thật |
| img_028 | Bán tải chở ngựa; ô tô bị che | R-06, R-02 | Ngựa không gán; ô tô bị che theo ngưỡng %; lớp của bán tải — **cần họp** | Bán tải nằm giữa `car` và `truck` |
| img_037 | Xe cabin kín, xe buýt cắt mép, người đi bộ | R-11, R-03 | Xe buýt như img_009; xe cabin kín `car` hay `van` — **cần họp** | v0 không phân biệt `car` với `van` |

Câu hỏi *chưa chốt, cần họp* (mang ra Pha 2):

- Bán tải thùng hở: `car` hay `truck`? (img_012, img_028)
- Xe cabin kín chở khách: `car` hay `van`? (img_037)
- Thêm lớp `bus` hay giữ Phương án B? (img_009, 011, 031, 037, 040)
- Xe đang được chở trên xe khác có gán không? (img_002)
- Ngưỡng `occluded_level` 10/40/80% có phù hợp không? (img_006, 024, 034, 036)
- Vật bị cột/cây cắt làm 2 mảnh: 1 box hay chỉ mảnh lớn?

## 5. QC và phiên bản

- Ngưỡng QC (theo `taxonomy.json` → `rules`, không đổi): IoU ≥ 0.85 với box cao ≥ 50 px, ≥ 0.7 với box 20–50 px; IAA: IoU TB ≥ 0.85 và cùng lớp ≥ 95%; Release khi lỗi theo ảnh ≤ 2%.
- Ca mơ hồ khi gán: tick `can_review`, không tự đoán; quyết định ở họp calibration, ghi mỗi quyết định một dòng.
- Mọi thay đổi làm đổi nhãn → tăng phiên bản.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản giảng viên | — |
| v0.1 | Định nghĩa lớp + phản ví dụ; ngưỡng `occluded_level` 10/40/80%, `truncated` ≥ 10%; quy tắc R-01…R-13: vật không thật, người–xe, người trong xe kín, xe chủ, vật nhỏ, đám đông; xử lý vật ngoài taxonomy | img_003, 004, 009, 014, 017, 021, 022, 023, 028, 037 |
