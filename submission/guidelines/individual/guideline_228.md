# Guideline gán nhãn — v0.1 (bản cá nhân) — <2A202603228>

- Phiên bản: v0.1 (cá nhân, trước họp nhóm)
- Ngày: 2026-11-16
- Người viết: Đoàn Quang Minh
- Cơ sở: xem cả 40 ảnh `data/images/img_001..040`; sửa từ [guideline v0](../../../guideline/guideline_v0_starter.md)


## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình **phát hiện vật 2D** (người và phương tiện có động cơ cỡ ô tô / xe máy) trên ảnh đường phố, ví dụ cho camera giao thông.

Lỗi nguy hiểm nhất:
- **Thừa box** trên vật không thật (tượng, hình in, phản chiếu, đồ chơi): mô hình học "thấy xe" ở nơi không có xe.
- **Bỏ sót** người hoặc xe thật ở vùng đường.
- **Sai lớp** giữa `car` / `truck` / `van`.

Phép thử cho mọi lớp và thuộc tính: *"mô hình có phải đoán ra điều này không?"* Chỉ gán điều **nhìn thấy trong ảnh**, không gán điều phải suy đoán.

## 2. Lớp và thuộc tính

### Lớp (mỗi vật đúng một lớp)

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Xe ô tô con có thân kín liền mạch: sedan, hatchback, SUV, taxi, xe wagon, xe thể thao. | img_023 (xe đen), img_015, img_024 (xe wagon đen) | Xe có thùng hàng hở là `truck`; xe 7–16 chỗ dạng hộp là `van` |
| `truck` | Xe có thùng hàng riêng phía sau cabin: xe tải, **xe bán tải (pickup) có thùng hở**, đầu kéo, xe chở xe. | img_002, img_007 (cả xe isuzu và pickup trắng), img_012, img_036, img_037, img_028 (pickup cổ) | SUV kín là `car` |
| `van` | Xe thân hộp một khối, cabin và khoang hành khách/hàng liền nhau, cỡ nhỏ: minivan, xe 12–16 chỗ. | img_005 (xe xám phía sau), img_016 (xe Hiace) | Xe buýt lớn không thuộc lớp nào (xem R-05) |
| `motorcycle` | Xe hai bánh có động cơ: xe máy, xe tay ga, mô tô, **kể cả xe có thùng bên hông (sidecar)**. | img_006, img_008, img_021 | Xe đạp không có động cơ: không gán |
| `person` | Người thật, nhìn thấy ít nhất đầu hoặc thân; gồm người đi bộ, người ngồi trên xe, người lái xe nhìn thấy qua kính. | img_003, img_014, img_034, img_040 | Ma-nơ-canh, tượng, hình in trên biển quảng cáo |
| `ignore_region` | Vùng không gán từng vật, bỏ sót ở đây không tính lỗi (xem R-08). | hàng xe máy đỗ dày đặc img_030, đám đông xa img_021, img_032 | Không dùng để "lười" vẽ vật rõ ràng |

**Vật không thuộc lớp nào** (xe đạp, xích lô, tuk-tuk, xe buýt…): **không vẽ box, không dùng `ignore_region` che lên** (trừ khi nằm trong vùng đám đông ở R-08). Người ngồi trên/trong vẫn gán `person` (R-05). Ghi vào `image_issues.csv` để nhóm quyết có thêm lớp mới hay không.

### Thuộc tính

**`occluded_level`** (tính trên diện tích của vật bị vật khác che, so với tổng diện tích vật nếu không bị che):

| Mức | Ngưỡng | Ví dụ |
|---|---|---|
| 0 | che < 5% | xe đỗ trống img_036 |
| 1 | che từ 5% đến 30% | |
| 2 | che từ trên 30% đến 70% | xe van sau đám người img_005 |
| 3 | che trên 70%, chỉ còn một mảnh nhỏ nhưng vẫn nhận ra lớp | xe máy giữa dòng xe img_021 |

Che do vật **trước mặt** (người, xe khác, cột). Phần bị cắt mép ảnh **không** tính vào occluded (xem `truncated`). Ước lượng bằng mắt cũng phải theo các mốc trên; nếu lưỡng lự giữa hai mức, chọn mức cao hơn và tick `can_review`.

**`truncated`**: `yes` khi **một cạnh box trùng mép ảnh** và vật còn tiếp tục ra ngoài ảnh; `no` nếu vật nằm trọn trong ảnh. Ví dụ `yes`: xe đỏ img_015 (bên trái), người cầm lái img_040.

**`can_review`**: tick khi một trong các điều kiện sau đúng: (a) lưỡng lự lớp; (b) lưỡng lự giữa hai mức occluded; (c) vật ở ca khó đã ghi trong mục 4 mà chưa có Rule ID chốt; (d) box nhỏ gần ngưỡng 20 px. Không tick vì "mệt".

## 3. Quy tắc vẽ box

| ID | Quy tắc |
|---|---|
| R-01 | **Box ôm sát phần nhìn thấy** của vật (từ điểm ngoài cùng trái/phải/trên/dưới). Không đoán phần bị che hoặc phần ngoài ảnh. Sai số cho phép tính theo IoU: ≥ 0.85 (box cao ≥ 50 px) hoặc ≥ 0.7 (box nhỏ hơn) so với box tham chiếu. |
| R-02 | **Người đi xe máy: luôn 2 box (hoặc nhiều hơn)**: 1 box `motorcycle` ôm phần xe nhìn thấy (bánh, thân, ghi-đông; **không kéo lên đầu người**), và mỗi người ngồi trên xe có 1 box `person` ôm phần cơ thể nhìn thấy (đầu có mũ bảo hiểm tới chân). Hai box được phép chồng nhau. Xe chở 2 người: 1 xe + 2 person (img_026, img_010). Xe đỗ không người: chỉ box `motorcycle`. Người đứng cạnh xe: chỉ `person`. |
| R-03 | **Phụ kiện và hàng hóa**: box `person` chỉ ôm cơ thể, **không** kéo dài theo gánh hàng, ống dài, túi lớn vượt khỏi thân người (img_003, img_029). Hàng chất trên xe máy/xe tải **không** vào box xe nếu nhô ra ngoài thân xe quá 20% chiều dài xe (img_035, img_029). |
| R-04 | **Che khuất**: vẽ box trên phần nhìn thấy, ghi `occluded_level` theo ngưỡng. Nếu phần nhìn thấy cao < 20 px hoặc không nhận ra lớp: không box (xem R-08). |
| R-05 | **Vật ngoài taxonomy** (tuk-tuk img_004/016/035, xích lô img_023, xe đạp img_006/029/003, xe buýt img_009/031/037): **không box phương tiện**; người lái và hành khách nhìn thấy rõ vẫn gán `person`. Ghi ảnh vào `image_issues.csv`. Ca cần nhóm chốt ở họp CP2: có thêm lớp cho các phương tiện này không. |
| R-06 | **Vật không thật** không gán: tượng (img_017 tượng người cưỡi ngựa, img_027 mộ hình xe tải, img_028 tượng ngựa trên thùng xe), ma-nơ-canh (img_033), ảnh/hình in trên biển quảng cáo (img_018, tranh tường img_026), đồ chơi và mô hình xe (img_038), phản chiếu gương và mặt đường (img_020, img_015 mặt nước). Mặt người in trên xe taxi (img_040) cũng không gán. |
| R-07 | **Vật nằm trên/trong vật khác là hàng chuyên chở**: xe con đang được chở trên xe chở xe (img_002), xe mô hình đặt trên nóc xe (img_024), tượng trên thùng pickup (img_028) **không gán box riêng**; chỉ gán box cho phương tiện chở (`truck` bao cả đầu kéo và rơ-moóc img_002). |
| R-08 | **Vùng ignore**: vật có chiều cao box < 20 px (khớp ngưỡng `min_box_height_px`), hoặc cụm nhiều vật chen nhau đến mức không tách được từng cái (hàng xe máy đỗ dày img_030 và img_036 nền sau, đám đông xa img_032, dòng xe máy xa img_021). Vẽ **1 hộp `ignore_region` ôm cả cụm**; những vật phía trước đủ lớn và tách được vẫn vẽ box riêng bên trong hoặc bên ngoài vùng. Không dùng ignore để thay cho vật lớn rõ ràng. |
| R-09 | **Mờ chuyển động / thiếu sáng** (img_014, img_015, img_013 ban đêm): vẫn gán nếu nhận ra được **lớp** và cao ≥ 20 px; nếu chỉ thấy vệt mờ không biết là gì thì không box và không ignore. Tick `can_review`. |
| R-10 | **Vật bị cắt mép ảnh**: vẽ box tới đúng mép, đặt `truncated=yes`. Nếu phần còn trong ảnh cao < 20 px hoặc không nhận ra lớp thì không box. |
| R-11 | **Xe pickup / SUV**: pickup thùng hở là `truck`; SUV, xe thùng kín liền khối cỡ cabin là `car`. Pickup có mui trùm (img_037) vẫn là `truck`. Xe thùng kín kiểu Hiace chở khách là `van`. |
| R-12 | **Xe "giả tưởng" nhưng chạy thật** (img_019 xe vàng DM có biển số và người lái): vẫn là `car`, gán box; con thú nhồi bông trên xe không phải `person`. Người lái ngồi trong xe gán `person` nếu thấy rõ mặt/thân. |

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định ban đầu | Lý do |
|---|---|---|---|---|
| img_003 | Người gánh hàng, xe đạp, xe máy phía sau | R-03, R-05 | person ôm thân; không box xe đạp; gánh hàng ngoài box | Hàng không phải người; xe đạp ngoài taxonomy |
| img_004 | Hai tuk-tuk, tài xế bên trong | R-05 | Không box tuk-tuk, tick `can_review` cho person nếu có; ghi vào image_issues | Tuk-tuk chưa có lớp riêng |
| img_002 | Xe chở xe, 6–7 xe con trên rơ-moóc | R-07 | 1 box `truck` cho cả xe; không box xe trên rơ-moóc | Xe con là hàng chuyên chở |
| img_005 | Cận cảnh 4–5 người đội mũ trên xe máy, xe van phía sau bị che | R-02, R-04 | person từng người, motorcycle thấy một phần, van occluded=2; tick `can_review` ở vật bị che nặng | Che dày, ngưỡng mức 2 và 3 dễ lệch |
| img_006 | Dòng xe máy + người đạp xe, taxi bên trái | R-02, R-05 | box cặp xe + người; xe đạp thì chỉ person | |
| img_007 | Isuzu tải và pickup trắng, phía trong nhà để xe | R-11 | cả hai `truck` | Pickup thùng hở có chụp bạt vẫn có thùng riêng |
| img_009 | Xe buýt mini giữa phố | R-05 | Không box (ngoài taxonomy), ghi image_issues | Không phải `van` vì cỡ lớn, chỗ ngồi 20+ |
| img_010 | 2 người trên xe máy đỗ, 1 người đứng bán rau, xe cút kít | R-02, R-05 | 1 motorcycle + 2 person ngồi + person đứng; bỏ xe đẩy rau | |
| img_012 | Pickup lấy hàng ở thùng, xe SUV xám phía trước | R-11 | pickup `truck` truncated=yes; SUV `car` | Hàng trong thùng không tách riêng |
| img_015 | Xe đỏ cắt mép trái, xe bạc cắt mép phải, xe mờ phía sau, phản chiếu mặt đường | R-06, R-09, R-10 | 2 xe lớn `car` truncated=yes; xe mờ phía sau thì box nếu nhận ra; không box phản chiếu | |
| img_016 | Hai tuk-tuk, van Hiace, xe máy xanh có người | R-02, R-05, R-11 | van, motorcycle + person; không box tuk-tuk; người đứng đầu ảnh bên phải truncated=yes | |
| img_017 | Tượng người cưỡi ngựa, có nón giao thông trên đầu | R-06 | Không gán | Tượng, không phải người thật |
| img_018 | Biển quảng cáo in mặt người | R-06 | Không gán mặt in | Hình in, không phải người thật |
| img_019 | Xe vàng "DM", con chuột nhồi bông, đám đông sau rào | R-12, R-08 | `car` + person người lái; không gán chuột; đám đông dày xa dùng ignore | Xe thật, người thật |
| img_020 | Kính chiếu hậu có hình xe, đường phản chiếu | R-06 | Không gán gì | Toàn phản chiếu |
| img_023 | Xe Honda đen, xích lô, xe máy xa | R-05 | car; person người đạp xích lô; không box xích lô | Xích lô ngoài taxonomy |
| img_024 | Xe wagon đen, xe mô hình vàng trên nóc, nhiều xe cổ trong nhà trưng bày | R-07, R-06 | Wagon đen `car`; xe vàng trên nóc không gán; các xe cổ đỗ phía sau là xe thật `car` | Xe trưng bày vẫn là xe thật |
| img_026 | Hai người trên xe máy đỏ, tranh tường phía sau | R-02, R-06, R-09 | motorcycle + 2 person; không gán tranh | Mờ chuyển động |
| img_027 | Mộ đá hình xe bán tải | R-06 | Không gán | Tượng đá |
| img_028 | Pickup cổ chở tượng ngựa trắng, xe SUV đỏ xa | R-06, R-07, R-11 | pickup `truck`; không box ngựa; SUV đỏ `car` | Tượng là hàng |
| img_029 | Hai người, một xe máy, một xe đạp, ống kim loại dài vắt ngang | R-03, R-02, R-05 | motorcycle + person; xe đạp chỉ person; ống không vào box | |
| img_033 | Ma-nơ-canh, kính phản chiếu, gương chiếu hậu | R-06 | Không gán gì | Không phải người thật |
| img_034 | Người đi bộ qua đường, xe đen, nhiều xe máy, người cắt mép phải | R-02, R-10 | person; car; motorcycle + person cắt mép `truncated=yes` | |
| img_035 | Tuk-tuk chất hàng cao, xe máy, xe con trắng | R-03, R-05, R-02 | Không box tuk-tuk; motorcycle + person; car | Hàng nhô vượt thân xe |
| img_037 | Pickup trắng có mui, xe buýt phía sau, người đi bộ | R-11, R-05 | pickup `truck`; person; xe buýt không box | |
| img_038 | Tủ trưng bày xe và đồ chơi mô hình | R-06 | Không gán | Đồ chơi |
| img_040 | Người đội mũ cắt mép, tuk-tuk, taxi, SUV phía sau | R-10, R-05, R-06 | person truncated=yes; taxi `car` truncated=yes; SUV `car`; không box tuk-tuk; không gán mặt in trên xe | |

**Hai kiểu ca khác nhau:**
- *Loại theo quy tắc* (không cần hỏi): R-06 (vật không thật), R-07 (hàng chuyên chở), R-05 (ngoài taxonomy).
- *Cần hỏi reviewer / nhóm*: ranh giới `van` và xe buýt mini (img_009); có nên thêm lớp cho tuk-tuk / xe đạp không (img_004, 016, 035); cách đếm xe trên xe chở xe (img_002).

## 5. QC và phiên bản

Ngưỡng áp dụng (từ `taxonomy.json`, không đổi): box cao < 20 px → ignore; IoU ≥ 0.85 (box cao ≥ 50 px) hoặc ≥ 0.7; IAA IoU trung bình ≥ 0.85 và cùng lớp ≥ 95%; ghép box khi IoU ≥ 0.5; lỗi theo ảnh ≤ 2%.

Ca mơ hồ: tick `can_review` và, nếu không vẽ được box, ghi vào `image_issues.csv`. Quyết định cuối do nhóm họp ở Pha 2 và Pha 5.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0.1 | Thêm R-01..R-12, ngưỡng thuộc tính, 40 dòng ca khó | v0 không nói gì về xe máy, che khuất, tượng, xe chở xe |
