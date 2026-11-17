# Guideline gán nhãn — v1.1 — Nhóm Lab 402

> Tài liệu đặc tả gán nhãn chính thức (Hiệu chuẩn sau đo lường IAA và họp Calibration) của Nhóm Lab 402 cho bài toán phát hiện đối tượng 2D trên ảnh giao thông đường phố (Day 07). Bản v1.1 kế thừa taxonomy v1.0 (11 lớp), đồng thời đính chính câu chữ, định lượng hóa chi tiết các quy tắc vẽ box, phân định ranh giới vùng bỏ qua và giải quyết triệt để 2162 ca bất thường phát sinh từ 120 lượt gán độc lập trên 40 ảnh.

- **Phiên bản:** v1.1 (Bản hiệu chuẩn sau Calibration)
- **Ngày hiệu chuẩn:** 17/11/2026
- **Nhóm thực hiện:** Nhóm Lab 402
  - **Người điều phối:** 238 (Nguyễn Quốc Khánh)
  - **Người review bản cuối:** 03228 (Đoàn Quang Minh)
  - **Thành viên tham gia:** 03039 (Đặng Anh Tuyền), 03223 (Nguyễn Chí Tài), 03224 (Nguyễn Quốc Việt), 03228 (Đoàn Quang Minh), 238 (Nguyễn Quốc Khánh)

---

## 1. Mục tiêu

- **Mô hình sử dụng nhãn:**
  - Bộ nhãn phục vụ huấn luyện và đánh giá các mô hình học sâu thị giác máy tính phát hiện vật thể 2D (2D Object Detection như YOLOv8/v10, Faster R-CNN, RT-DETR) chạy thời gian thực trên ảnh đường phố đô thị.
  - Ứng dụng trọng tâm trong hệ thống hỗ trợ lái xe nâng cao (ADAS), xe tự hành (Autonomous Vehicles) và hệ thống camera giám sát giao thông thông minh (ITS / ITMS) trong điều kiện giao thông hỗn hợp (mixed traffic).

- **Mục đích gán nhãn:**
  - Định vị tọa độ 2D (bounding box chữ nhật song song trục) và phân loại chính xác các tác nhân tham gia giao thông cốt lõi (người, ô tô con, xe tải, xe van, xe buýt, xe tuk-tuk, xe máy, xe đạp, đèn tín hiệu giao thông, biển báo giao thông).
  - Cung cấp dữ liệu chuẩn mực để các module nhận thức phía sau thực hiện ước lượng khoảng cách (Monocular 3D / Depth estimation), theo dõi đối tượng (Multi-Object Tracking - MOT) và dự đoán quỹ đạo tránh va chạm (Collision Avoidance).
  - Phân tách rõ ràng giữa các phương tiện/con người thật đang tham gia giao thông với các yếu tố gây nhiễu thị giác (tượng đài, ma-nơ-canh, đồ chơi, hình in quảng cáo 2D phẳng, bóng đổ, ảnh phản chiếu qua kính/gương).

- **Phân tích rủi ro & các sai sót nguy hiểm (Failure Analysis):**
  - **Bỏ sót vật thể (False Negative - FN) — Cực kỳ nguy hiểm:**
    - Bỏ sót người đi bộ (`person`), người đi xe máy (`motorcycle`), người đi xe đạp (`bicycle`) hoặc ô tô ở cự ly gần: Hệ thống điều khiển tự động sẽ không kích hoạt phanh tự động khẩn cấp (AEB), dẫn đến va chạm trực tiếp gây chết người hoặc thương vong nặng nề.
    - Bỏ sót phương tiện lớn (`truck`, `bus`) ở các góc giao cắt: Dẫn đến tai nạn liên hoàn nghiêm trọng.
    - Bỏ sót đèn tín hiệu (`traffic_light`) hoặc biển báo (`traffic_sign`): Dẫn đến xe tự hành không nhận biết tín hiệu dừng/chạy hoặc đi vào đường cấm, gây xung đột giao thông nghiêm trọng.
  - **Nhận diện nhầm vật thể (False Positive - FP) — Nguy hiểm cao:**
    - Nhận diện nhầm ma-nơ-canh trong tủ kính, tượng đá/đồng, hình in chân dung trên biển quảng cáo, tranh vẽ tường hoặc ảnh phản chiếu thành người/xe thật: Dẫn đến hiện tượng "phanh ma" (phantom braking) — phương tiện tự hành phanh gấp bất ngờ giữa đường ở tốc độ cao, gây nguy cơ bị các phương tiện phía sau đâm dồn toa.
  - **Sai lệch Bounding Box (Bounding Box Drift / Low IoU) — Nguy hiểm:**
    - Box vẽ lẹm cắt cụt bánh xe/đầu người hoặc vẽ quá rộng bao gồm nhiều nền: Làm sai lệch thuật toán ước lượng khoảng cách 3D từ camera đơn (monocular depth), dẫn đến tính sai thời gian tới điểm va chạm (Time-to-Collision - TTC) và kích hoạt phản xạ đánh lái/phanh sai thời điểm.
  - **Nhận diện sai lớp (Misclassification) — Rủi ro cao:**
    - Nhầm xe tải nặng (`truck`) hoặc xe buýt (`bus`) thành ô tô con (`car`): Khiến hệ thống ước tính sai kích thước khối bao, điểm mù và gia tốc phanh, gây nguy hiểm khi vượt xe hoặc chuyển làn.

---

## 2. Lớp và thuộc tính

### 2.1. Thống nhất Taxonomy v1.1 (Giữ nguyên 11 lớp v1.0)

Bộ taxonomy chuẩn hóa gồm 11 lớp (10 lớp đối tượng và 1 lớp vùng bỏ qua `ignore_region`):

| Lớp | Định nghĩa chuẩn (một câu) | Ví dụ thực tế trong bộ ảnh | Phản ví dụ / Dễ nhầm với |
|---|---|---|---|
| `car` | Phương tiện giao thông cơ giới 4 bánh có thân liền mạch, chủ yếu dùng chở người và hành lý gia đình (sedan, hatchback, SUV, crossover, MPV, xe wagon, taxi). | Xe đen `img_023`, xe wagon `img_024`, các ô tô con trên sàn xe chở xe `img_002`, xe taxi `img_040`. | Xe bán tải (pickup) thuộc `truck`; xe thân hộp liền chở khách 16 chỗ thuộc `van`; mô hình xe đồ chơi (`img_038`) bị loại. |
| `truck` | Phương tiện cơ giới đường bộ có cabin điều khiển và khu vực/thùng chở hàng tách biệt về kết cấu (xe tải, xe đầu kéo, xe chở ô tô, xe bán tải/pickup kể cả loại có nắp thùng cao). | Xe chở ô tô `img_002`, xe tải nhỏ mui bạt `img_007`, `img_036`, xe bán tải `img_012`, `img_028`, `img_037`. | Xe van thùng kín liền khối thuộc `van`; tượng xe mộ đá (`img_027`) bị loại; thùng hàng rời không gán riêng. |
| `van` | Phương tiện cơ giới dạng thân hộp kín một khối (monovolume), khoang lái và khoang hàng/hành khách thông liền nhau (minivan, xe van chở hàng, xe chở khách cỡ nhỏ 9–16 chỗ). | Xe van xám phía sau dòng người `img_005`, xe van trắng Hiace `img_016`. | Xe buýt lớn chở khách trên 20 chỗ (`img_009`) thuộc `bus`; xe tải thùng kín rời cabin (`img_036`) thuộc `truck`. |
| `bus` | Phương tiện giao thông cơ giới cỡ lớn chuyên dụng vận chuyển số lượng lớn hành khách (xe buýt nội đô, xe khách liên tỉnh, minibus trên 16–20 chỗ). | Xe buýt chạy giữa phố `img_009`, `img_031`, xe buýt lớn phía sau `img_037`, xe buýt cắt mép `img_040`. | Xe van 9–16 chỗ thân nhỏ thuộc `van`; xe tải thùng dài thuộc `truck`. |
| `tuk_tuk` | Phương tiện giao thông cơ giới 3 bánh chuyên chở khách hoặc hàng nhẹ đặc trưng đô thị (tuk-tuk, xe lam, auto-rickshaw có động cơ). | Xe tuk-tuk chở khách `img_004`, `img_016`, xe tuk-tuk chất hàng cao trên nóc `img_035`. | Xe máy gắn sidecar (`img_022`) thuộc `motorcycle`; xích lô đạp chân (`img_023`) ngoài taxonomy. |
| `motorcycle` | Phương tiện cơ giới hai bánh có động cơ (xe máy, xe tay ga, mô tô phân khối lớn), bao gồm cả mô tô có gắn thùng bên hông (sidecar). | Xe máy di chuyển `img_005`, `img_008`, `img_010`, `img_026`, mô tô gắn sidecar `img_022`. | Xe đạp thô sơ hoặc xe đạp điện thuộc lớp `bicycle`; xe ba bánh tuk-tuk thuộc `tuk_tuk`. |
| `bicycle` | Phương tiện giao thông thô sơ hoặc trợ lực hai bánh chạy bằng bàn đạp của người (xe đạp thường, xe đạp địa hình/thể thao, xe đạp điện). | Xe đạp sau người gánh hàng `img_003`, người đi xe đạp `img_006`, `img_029`. | Xe gắn máy/xe mô tô có động cơ (`motorcycle`); xích lô chở khách đạp chân (ngoài taxonomy); xe đẩy hàng. |
| `person` | Con người thật ở mọi tư thế (đi bộ, đứng, ngồi, lái xe, hành khách) nhìn thấy trực tiếp trong ảnh và đạt kích thước tối thiểu. | Người bán hàng `img_003`, `img_010`, người lái xe máy `img_005`, `img_026`, người đi bộ `img_034`, `img_040`. | Ma-nơ-canh `img_033`, tượng người `img_017`, mặt người in trên biển quảng cáo `img_018`, hình người in trên thân xe taxi `img_040`. |
| `traffic_light` | Thiết bị đèn tín hiệu chiếu sáng dùng để điều khiển luồng giao thông đường bộ (cụm đèn 3 màu xanh/vàng/đỏ, đèn tín hiệu cho người đi bộ). Box bao trọn cụm hộp đèn (không tính cột trụ). | Cụm đèn tín hiệu treo trên giá hoặc gắn cột bên đường (`img_006`, `img_010`, `img_031`, `img_040`). | Đèn chiếu sáng đường phố (background); đèn pha/hậu của phương tiện; đèn trang trí đô thị. |
| `traffic_sign` | Biển báo hiệu đường bộ dạng bảng tấm kim loại/nhựa mang ký hiệu hoặc chữ viết quy ước giao thông (biển cấm, biển cảnh báo, biển hiệu lệnh, biển chỉ dẫn). Box bao trọn mặt bảng biển báo (không tính cột trụ). | Biển cấm dừng đỗ, biển báo nguy hiểm, biển chỉ hướng (`img_006`, `img_012`, `img_016`, `img_037`). | Biển hiệu quảng cáo thương mại (`img_018` bị loại); biển số xe phương tiện; tranh tường 2D (`img_026`). |
| `ignore_region` | Vùng đánh dấu khu vực chứa cụm đối tượng quá nhỏ ($h < 20\text{ px}$) hoặc đám đông xếp chồng chéo quá dày đặc không thể phân ranh giới chính xác từng vật thể. | Cụm xe máy đỗ ken đặc hậu cảnh `img_030`, `img_036`, đám đông người/xe ở xa tít `img_019`, `img_021`, `img_032`. | Không dùng để thay thế cho vật đơn lẻ to rõ; không dùng cho vùng trống không có đối tượng. |

---

### 2.2. Quy tắc đối với vật thể không thuộc lớp nào

- **Phương tiện thô sơ khác và vật dụng đường phố (Xích lô, xe cút kít, xe đẩy hàng):** Không vẽ bounding box cho phương tiện. Tuy nhiên, người lái hoặc người đẩy xe vẫn được vẽ box `person` ôm sát phần cơ thể nhìn thấy.
- **Hàng hóa, phụ kiện rời nhô ra:** Không gán box riêng. Nếu hàng hóa nhô ra quá lớn ($> 20\%$ kích thước xe) thì không kéo dài box phương tiện theo hàng (xem quy tắc `R-11`).
- **Động vật (chim bồ câu, chó, mèo, ngựa thật):** Bỏ qua hoàn toàn, không vẽ box.
- **Bối cảnh hạ tầng tự nhiên và đô thị (Mặt đường, vỉa hè, bầu trời, tòa nhà, cây cối, cột đèn chiếu sáng):** Không gán box (coi là background).
- **Vật thể lạ, hiếm gặp ngoài danh mục (Xe cứu thương chuyên dụng, xe công trình):** Gán lớp gần nhất có thể hoặc ghi vào nhật ký giải quyết ca biên.

---

### 2.3. Thuộc tính chuẩn hóa (Attributes)

Mọi bounding box thuộc 10 lớp đối tượng (ngoại trừ `ignore_region`) đều phải được gán đủ 3 thuộc tính đo lường định lượng được dưới đây:

| Tên thuộc tính | Kiểu / Giá trị | Mặc định | Ý nghĩa | Ngưỡng đo lường định lượng cụ thể (Hiệu chuẩn v1.1) | Quy tắc liên kết |
|---|---|---|---|---|---|
| `occluded_level` | `select`<br>(`0`, `1`, `2`, `3`) | `0` | Đánh giá tỉ lệ diện tích hình chiếu của vật bị vật khác che khuất trong khung ảnh. | • `0` (Không che): Nhìn thấy trọn vẹn hoặc bị che $< 5\%$ diện tích bề mặt.<br>• `1` (Che nhẹ): Bị che khuất từ $5\%$ đến $30\%$ diện tích.<br>• `2` (Che vừa): Bị che khuất từ $31\%$ đến $70\%$ diện tích.<br>• `3` (Che nặng): Bị che khuất $> 70\%$ diện tích (nhưng phần còn lại $\ge 10-15\%$ vẫn đủ nhận diện chắc chắn lớp).<br>*(Lưu ý: Nếu bị che $> 90\%$ không còn nhận dạng chắc chắn thì bỏ qua không gán).* | `R-02` |
| `truncated` | `select`<br>(`no`, `yes`) | `no` | Đánh giá vật thể có bị đường viền khung ảnh cắt cụt hay không. | • `no`: Vật thể nằm trọn vẹn trong khung ảnh hoặc mép ngoài chỉ chạm sát viền làm mất $< 10\%$ diện tích.<br>• `yes`: Một cạnh của box tiếp xúc đúng mép ảnh ($x=0, y=0, x=W, y=H$) và phần thân vật thể bị cắt mất từ $10\%$ diện tích trở lên.<br>*(Lưu ý: Nếu phần lọt vào ảnh quá nhỏ $< 20\text{ px}$ chiều cao/rộng thì bỏ qua không gán).* | `R-03` |
| `can_review` | `checkbox`<br>(`true`, `false`) | `false` | Đánh dấu ca biên trong lúc gán độc lập. **Tại bản v1.1 sau calibration, tất cả các cờ `can_review` đều được giải quyết và chuyển về `false`.** | • `false`: Nhãn đã được xác nhận và chốt đồng thuận sau calibration.<br>• `true`: Chỉ dùng tạm thời trong Pha 3 khi người gán phân vân. | `R-12` |

---

## 3. Quy tắc vẽ box (Làm rõ và hiệu chuẩn sau Calibration)

Mọi thành viên bắt buộc tuân thủ 13 quy tắc chuẩn hóa dưới đây:

### R-01: Bounding Box ôm sát phần nhìn thấy (Tight Box — Visible Pixels Only)
- Bounding box là hình chữ nhật đứng (axis-aligned rectangle) tiếp xúc sát 4 điểm cực trị (trên, dưới, trái, phải) của **phần điểm ảnh thực tế nhìn thấy được** của vật thể.
- **Tuyệt đối KHÔNG ước lượng hay vẽ bù** phần thân thể hoặc bộ phận xe bị khuất sau vật khác hoặc nằm ngoài khung hình.
- Độ dung sai viền nền (background margin) cho phép tối đa 3–5 pixel. Không được cắt lẹm chi tiết nhìn thấy của vật và không vẽ thừa khoảng trống nền.

### R-02: Xử lý che khuất và vật thể bị chia cắt (Occlusion Handling)
- Box chỉ bao bọc các phần thực tế nhìn thấy của vật thể; phần bị che bởi vật cản phía trước nằm trong bounding box nhưng thuộc tính `occluded_level` phải được tăng tương ứng.
- **Trường hợp vật thể bị che cắt rời thành 2 mảnh nhìn thấy:**
  - Nếu hai mảnh nhìn thấy rõ ràng thuộc về cùng một đối tượng duy nhất (ví dụ: thân xe ô tô bị cột đèn chắn ngang chính giữa): Vẽ **01 box duy nhất** bao trọn cả hai mảnh phần nhìn thấy, gán `occluded_level` phù hợp (mức 1 hoặc 2).
  - Nếu phần bị che quá lớn ($> 80\%$) và hai mảnh nằm quá xa nhau khiến không thể khẳng định chắc chắn chúng thuộc cùng một vật: Không gán box gộp to mà chỉ gán phần nhận diện được chắc chắn.
- **Gán thuộc tính `occluded_level`**:
  - `0`: Không bị che khuất (nhìn thấy trọn vẹn hoặc bị che $< 5\%$ diện tích).
  - `1`: Bị che khuất nhẹ (từ $5\%$ đến $30\%$ diện tích).
  - `2`: Bị che khuất vừa (từ $31\%$ đến $70\%$ diện tích).
  - `3`: Bị che khuất nặng ($> 70\%$ diện tích, nhưng phần còn lại vẫn đủ nhận diện chắc chắn). Nếu vật bị che $> 90\%$ không còn nhận dạng được: **Bỏ qua không gán**.

### R-03: Xử lý vật thể bị cắt ở mép ảnh (Truncation & Boundary Coordinates)
- Bounding box phải nằm hoàn toàn trong tọa độ ảnh: $0 \le x_{\min} < x_{\max} \le W$, $0 \le y_{\min} < y_{\max} \le H$. Box dừng đúng tại đường biên ảnh, không mở rộng ra ngoài canvas.
- Gán `truncated = yes` khi một mép box trùng đường biên ảnh và vật thể bị cắt mất từ $10\%$ diện tích trở lên.
- Nếu phần lọt vào ảnh có chiều cao/rộng $< 20\text{ px}$ hoặc không đủ đặc trưng để xác định lớp: Bỏ qua không gán.

### R-04: Loại bỏ vật không thật, hình in, tranh vẽ, mô hình và phản chiếu (Real vs. Non-Real)
- **Tượng người, ma-nơ-canh (`img_011`, `img_017`, `img_033`):** Không gán nhãn `person`. Coi là hard negatives.
- **Mô hình đồ chơi, tượng xe, mộ đá (`img_024`, `img_027`, `img_038`):** Không gán nhãn phương tiện (`car`/`truck`).
- **Hình ảnh 2D (Biển quảng cáo `img_018`, tranh tường `img_026`, ảnh in trên thân xe taxi `img_040`):** Không vẽ box cho các hình in phẳng 2D.
- **Ảnh phản chiếu và bóng đổ (`img_015`, `img_020`):** Tuyệt đối không vẽ box cho hình ảnh ảo phản chiếu qua gương chiếu hậu, cửa kính xe, mặt đường ướt; không đưa bóng đổ trên mặt đất vào bounding box.
- **Nhìn xuyên qua kính:** Người hoặc xe nhìn thấy rõ qua cửa kính trong suốt vẫn được gán nhãn bình thường.

### R-05: Loại bỏ xe chủ mang camera (Ego Vehicle)
- Bộ phận thuộc chính chiếc xe chở camera chụp ảnh xuất hiện ở rìa mép ảnh (gương chiếu hậu, nắp ca-pô, mép kính lái như ở `img_020`): **Không vẽ box phương tiện**, không gán nhãn.

### R-06: Ngưỡng kích thước tối thiểu và độ mờ nhòe (Min Box Size Threshold)
- **Ngưỡng kích thước:** Bỏ qua tất cả các vật thể có chiều cao hoặc chiều rộng **nhỏ hơn 20 pixels** ($h < 20\text{ px}$). Nếu là cụm đối tượng xa gom vào `ignore_region` theo `R-07`.
- **Mờ nhòe do chuyển động (Motion blur) hoặc thiếu sáng ban đêm (`img_013`, `img_014`):** Nếu vẫn phân biệt được lớp và cấu trúc hình học của vật thể $\ge 20\text{ px}$ thì vẫn vẽ box ôm sát vệt pixel nhận diện được. Nếu chỉ còn là vệt sáng nhòe không xác định được hình dạng thì bỏ qua hoàn toàn.

### R-07: Đám đông dày đặc và vùng bỏ qua (`ignore_region`)
- Khi một nhóm người hoặc cụm phương tiện xếp chồng chéo quá dày đặc ở hậu cảnh (như hàng xe máy đỗ ken đặc ở `img_030`, `img_036` hoặc đám đông cổ vũ ở `img_019`, đám đông xa ở `img_021`, `img_032`) mà việc tách từng box đơn lẻ là bất khả thi hoặc chiều cao vật $< 20\text{ px}$: Vẽ **01 box `ignore_region`** bao trùm toàn bộ khu vực đó.
- Các vật thể ở tiền cảnh đủ lớn ($\ge 20\text{ px}$) và tách biệt rõ ràng ranh giới vẫn phải được vẽ box riêng bình thường.

### R-08: Tách rời người điều khiển và phương tiện (Person & Vehicle Separation)
- Áp dụng nguyên tắc **tách rời từng cá thể thành từng box độc lập:**
  - **Box `person`:** Bao gồm toàn bộ cơ thể người lái hoặc hành khách nhìn thấy (đầu, mũ bảo hiểm, thân, tay chân). Mỗi người trên xe được gán **01 box `person` riêng**. Không kéo dài box theo túi xách, gánh hàng rời.
  - **Box phương tiện (`motorcycle`, `bicycle`, `tuk_tuk`, `car`):** Bao trùm toàn bộ thân xe, bánh xe, tay lái, bàn đạp/gương xe.
  - Hai box này được phép chồng lấn hình học (overlap) lên nhau tại vùng ngồi/yên xe. Thuộc tính `occluded_level` được đánh giá theo độ che khuất thực tế giữa người và xe.
  - *Ví dụ:* 01 xe máy chở 2 người (`img_026`) sẽ có tổng cộng **3 box riêng biệt**: 1 `motorcycle` và 2 `person`; 01 người đi xe đạp (`img_006`, `img_029`) sẽ có tổng cộng **2 box riêng biệt**: 1 `bicycle` và 1 `person`.

### R-09: Xe chuyên chở ô tô và hàng hóa trên xe (Car Carrier & Cargo)
- **Phương tiện vận chuyển (`truck` trên `img_002`):** Vẽ 01 box `truck` bao toàn bộ đầu kéo, khung gầm và kết cấu sàn giá chở.
- **Các ô tô con trên sàn xe chở xe (`img_002`):** Vẽ **từng box `car` riêng biệt** cho mỗi ô tô con nhìn thấy rõ trên giá sàn (thỏa mãn $h \ge 20\text{ px}$), gán `occluded_level` từ 1 đến 2 do bị khung giàn sắt che một phần.

### R-10: Xử lý phương tiện đặc thù (Sidecar & Xe bán tải)
- **Xe mô tô có thùng bên hông (Sidecar trên `img_022`):** Vẽ **01 box `motorcycle` duy nhất** bao trọn cả xe mô tô và phần thùng xe bên hông gắn cố định.
- **Xe bán tải (Pickup):** Bán tải thùng hở (`img_012`, `img_028`) hoặc bán tải lắp thêm nắp thùng cao composite (`img_037`) đều được xếp thống nhất vào lớp **`truck`** (do cấu trúc nguyên bản cabin rời và khung gầm xe tải).

### R-11: Phụ kiện gắn liền vs. Hàng hóa nhô ra ngoài (Cargo Overhang)
- **Hàng hóa/phụ kiện gắn liền:** Thùng xe, mui bạt gắn cố định, gương chiếu hậu là bộ phận của xe, được tính vào box xe.
- **Hàng hóa nhô ra quá mức:** Nếu hàng hóa chở trên xe máy hoặc xe tải (như ống kim loại dài vắt ngang ở `img_029` hay hàng chất quá dài) nhô ra vượt quá $20\%$ chiều dài thân xe: **Không kéo dài box xe theo phần nhô ra đó**, box xe chỉ ôm theo ranh giới kết cấu thân xe. Box người cũng không kéo dài theo đòn gánh hay gậy dài (`img_003`).

### R-12: Cơ chế đối chiếu và đóng ca hiệu chuẩn (`can_review` Resolution)
- Trong giai đoạn hiệu chuẩn v1.1: Tất cả các ca từng gắn cờ `can_review` ở Pha 3 đều đã được đem ra rà soát, thảo luận và chốt phương án xử lý theo các quy tắc `R-01` đến `R-11`, `R-13`. Mọi box được giữ lại trong bản nhãn cuối đều có `can_review = false`.

### R-13: Biển báo và đèn tín hiệu giao thông (Traffic Light & Traffic Sign)
- **Đèn tín hiệu (`traffic_light`):** Bounding box chỉ bao trọn phần cụm hộp đèn (vỏ đèn và các bóng đèn chiếu sáng xanh, đỏ, vàng, đèn người đi bộ). **Tuyệt đối không kéo dài box** xuống cột thép, trụ đỡ hoặc giàn xà treo kim loại.
- **Biển báo giao thông (`traffic_sign`):** Bounding box ôm sát toàn bộ mặt bảng biển báo (hình tròn, tam giác, hình vuông, chữ nhật mang ký hiệu quy ước giao thông). **Không vẽ bao cột cắm đất** hay thanh gắn biển.
- Biển quảng cáo thương mại, pa-nô hoặc biển hiệu cửa hàng không phải biển báo giao thông: Tuyệt đối không gán nhãn (tuân thủ `R-04`).

---

## 4. Bảng ca khó và quyết định hiệu chuẩn (Calibration Summary trên 40 ảnh)

Dưới đây là bảng tổng hợp các ca biên bất đồng điển hình được đưa ra thảo luận và chốt quyết định tại buổi họp Calibration (Pha 5):

| Ảnh | Tình huống / Đối tượng thực tế | Rule ID | Quyết định hiệu chuẩn v1.1 | Lý do kỹ thuật |
|---|---|---|---|---|
| `img_002` | Xe chuyên chở ô tô con trên sàn 2 tầng giá | `R-01, R-09` | Vẽ 01 box `truck` cho xe chuyên chở; vẽ từng box `car` riêng cho mỗi ô tô con nhìn thấy trên sàn ($h \ge 20\text{ px}$), `occluded_level=1-2`. | Ô tô trên sàn mang đầy đủ đặc trưng thị giác của `car`, phục vụ phát hiện vật thể che khuất trong không gian 3D. |
| `img_004` | Xe tuk-tuk 3 bánh chở khách | `R-01, R-08` | Gán `tuk_tuk` ôm sát thân xe; tách riêng 01 box `person` cho tài xế trong buồng lái. | Định danh đúng phương tiện cơ giới 3 bánh; tách người điều khiển theo `R-08`. |
| `img_006` | Người đi xe đạp và xe máy di chuyển | `R-08` | Bắt buộc tách 2 box riêng: box `person` và box `bicycle`/`motorcycle`. | Tránh gộp chung người và phương tiện làm sai phân loại mô hình. |
| `img_007` | Xe tải nhỏ có mui bạt chở hàng | `R-01` | Chốt lớp `truck` (không gán `car`). | Phương tiện có cabin và thùng hàng riêng biệt dùng để vận chuyển hàng hóa. |
| `img_008` | Xe van chở hàng phía sau và cụm xe máy | `R-01, R-08` | Xe thân hộp kín liền một khối gán `van`; tách box người và xe máy rõ ràng. | Phân định chính xác giữa xe van chở hàng và xe tải. |
| `img_011` | Xe ô tô gia đình SUV/Crossover | `R-01` | Chốt lớp `car` (không gán `van`). | Thân xe liền khối chở người gia đình, không phải xe thùng chở hàng hay chở khách 16 chỗ. |
| `img_012` | Xe bán tải bốc hàng cạnh SUV xám | `R-10` | Xe bán tải gán `truck`; SUV gán `car`; người bốc dỡ gán `person`. | Bán tải khung gầm rời xếp vào `truck`; hàng trong thùng gắn liền với xe. |
| `img_014` | Người và xe máy ban đêm bị motion blur | `R-06, R-08` | Vẫn gán `motorcycle` và `person` nếu mắt người còn nhận biết được cấu trúc; bỏ cờ `can_review`. | Mờ chuyển động không tự động loại bỏ nếu còn nhận diện được vật thể $\ge 20\text{ px}$. |
| `img_016` | Xe cơ giới 3 bánh và xe van trắng ở góc ảnh | `R-01, R-10` | Xe 3 bánh gán `tuk_tuk`; xe van trắng Hiace gán `van`. | Nhìn kết cấu mui bạt và khung xe 3 bánh; xe Hiace là xe van chở khách. |
| `img_017` | Tượng đá danh nhân cưỡi ngựa | `R-04` | **Tuyệt đối không gán box** (cả người lẫn ngựa). | Tượng nghệ thuật điêu khắc, mẫu âm nền (hard negative), tránh lỗi phanh ma. |
| `img_019` | Đám đông cổ vũ không tách được cá thể | `R-07` | Khoanh 01 box `ignore_region` bao trùm toàn bộ đám đông. | Số lượng quá đông chen chúc không thể tách rời ranh giới cá thể. |
| `img_023` | Cụm nhiều xe máy đỗ chen chúc góc trái | `R-01, R-02` | Xe nào nhận diện được ranh giới rõ ràng thì vẽ từng box `motorcycle` và tăng `occluded_level`. | Ưu tiên gán cá thể nếu tách được ranh giới bộ phận nhìn thấy. |
| `img_025` | Người đi bộ ở rất xa hậu cảnh ($h < 20\text{ px}$) | `R-06` | Loại bỏ các box người quá nhỏ dưới $20\text{ px}$ hoặc gom vào `ignore_region`. | Tuân thủ nghiêm ngặt ngưỡng kích thước tối thiểu của bài toán 2D detection. |
| `img_027` | Bia mộ/khối đá tạc hình xe bán tải | `R-04` | **Tuyệt đối không gán box**. | Tượng đá điêu khắc tĩnh, mẫu âm nền. |
| `img_028` | Xe bán tải chở tượng ngựa trắng trên thùng | `R-04, R-10` | Xe bán tải gán `truck`; không vẽ box cho tượng ngựa trên thùng. | Bán tải là `truck`; tượng ngựa là hàng trang trí tĩnh. |
| `img_030` | Bãi đỗ xe máy dày đặc nhiều dãy chồng chéo | `R-07` | Khoanh box `ignore_region` bao trùm cụm xe dày đặc; xe đơn lẻ ở rìa vẫn gán `motorcycle`. | Áp dụng đúng quy tắc xử lý cụm đám đông ken đặc. |
| `img_031` | Xe buýt lớn và đèn tín hiệu treo cao | `R-01, R-13` | Xe buýt gán `bus`; hộp đèn tín hiệu gán `traffic_light` (không tính cột). | Đúng taxonomy v1.0 và quy tắc ôm sát hộp đèn tín hiệu. |
| `img_033` | Tủ kính thời trang có ma-nơ-canh và ảnh phản chiếu | `R-04` | **Tuyệt đối không gán box**. | Ma-nơ-canh và ảnh ảo phản chiếu không phải người thật tham gia giao thông. |
| `img_035` | Xe tuk-tuk chở các bao tải hàng cao trên nóc | `R-01, R-11` | Gán `tuk_tuk` bao trọn thân xe và khối hàng buộc cố định trên mui; tài xế gán `person`. | Hàng chằng buộc trên nóc gắn liền với thể tích di chuyển của xe. |
| `img_037` | Xe bán tải nắp thùng cao composite | `R-10` | Gán `truck` (không gán `van`). | Khung gầm cabin rời của xe bán tải; nắp thùng composite chỉ là phụ kiện che chắn. |
| `img_038` | Tủ kính trưng bày hàng xe mô hình đồ chơi | `R-04` | **Tuyệt đối không gán box**. | Đồ chơi thu nhỏ, không phải phương tiện giao thông. |
| `img_040` | Xe taxi và hình chân dung in trên thân xe | `R-01, R-04` | Xe taxi gán `car`; hình người in 2D phẳng trên thân xe **tuyệt đối không gán box**. | Phân biệt rõ thực thể 3D và hoa văn/hình in 2D phẳng. |

---

## 5. QC và quản lý phiên bản

### 5.1. Ngưỡng QC áp dụng (Khóa cứng theo hệ thống)

| Tiêu chí kỹ thuật | Giá trị ngưỡng | Ý nghĩa & Ứng dụng |
|---|---|---|
| Kích thước box tối thiểu | $h \ge 20\text{ px}$ | Chiều cao box $< 20\text{ px}$ bắt buộc gom vào `ignore_region`, không gán riêng rẽ. |
| Ngưỡng IoU box lớn | $\text{IoU} \ge 0.85$ | Áp dụng cho các box có chiều cao $h \ge 50\text{ px}$. |
| Ngưỡng IoU box nhỏ | $\text{IoU} \ge 0.70$ | Áp dụng cho các box có chiều cao $20\text{ px} \le h < 50\text{ px}$. |
| Ngưỡng ghép box (Match IoU) | $\text{IoU} \ge 0.50$ | Mức trùng lặp không gian tối thiểu để script tự động ghép 2 box so sánh. |
| Chỉ số đồng thuận vị trí (IAA IoU) | $\ge 0.85$ | Điểm IoU trung bình trên toàn bộ các cặp box ghép được phải đạt tối thiểu $85\%$. |
| Chỉ số đồng thuận lớp (IAA Class) | $\ge 95\%$ | Tỉ lệ thống nhất nhãn lớp trên các cặp box ghép được phải đạt tối thiểu $95\%$. |
| Tỉ lệ lỗi tối đa theo ảnh | $\le 2.0\%$ | Đảm bảo tính khắt khe của bộ dữ liệu bàn giao (tương đương 0 ảnh lỗi trong lô 40 ảnh). |

---

### 5.2. Lịch sử thay đổi phiên bản

| Phiên bản | Ngày | Nội dung thay đổi trọng tâm | Lý do / Ca thực tế kích hoạt |
|---|---|---|---|
| `v0` | 16/11/2026 | Bản khung khởi tạo ban đầu (5 lớp cơ bản: `car`, `truck`, `van`, `motorcycle`, `person`). Định nghĩa định tính, thiếu quy tắc ca biên. | Điểm xuất phát của bài lab; chưa đủ chi tiết để gán thực tế. |
| `v1.0` | 16/11/2026 | **Bản thống nhất chính thức của nhóm trước khi gán độc lập:**<br>1. Thêm các lớp: `bus`, `tuk_tuk`, `bicycle`, `traffic_light`, `traffic_sign` (taxonomy 11 lớp).<br>2. Định lượng hóa 4 mức `occluded_level` ($0, 1, 2, 3$) và ngưỡng `truncated = yes` ($\ge 10\%$).<br>3. Khóa cứng 13 quy tắc vẽ box (`R-01` $\rightarrow$ `R-13`). | Giải quyết bất đồng giữa 5 bản cá nhân trên các ca `img_002`, `img_004`, `img_009`, `img_011`, `img_017`, `img_024`, `img_037`. |
| `v1.1` | 17/11/2026 | **Bản hiệu chuẩn chính thức sau Calibration (Pha 5):**<br>1. Làm rõ chi tiết dung sai tiếp xúc viền nền không quá 3–5px (`R-01`).<br>2. Chuẩn hóa định lượng 4 mức che khuất theo tỉ lệ diện tích bề mặt (`R-02`).<br>3. Chuẩn hóa điều kiện `truncated = yes` phải tiếp xúc mép ảnh và mất $\ge 10\%$ diện tích (`R-03`).<br>4. Thống nhất cách vẽ box cho xe mờ do motion blur ban đêm (`R-06`).<br>5. Chuẩn hóa việc khoanh vùng `ignore_region` cho đám đông và bãi xe dày đặc (`R-07`).<br>6. Tái khẳng định quy tắc bắt buộc tách người và xe hai bánh thành 2 box riêng (`R-08`).<br>7. Chốt thống nhất bán tải nắp thùng cao luôn là `truck` (`R-10`).<br>8. Xử lý triệt để và đóng toàn bộ 2162 ca bất thường từ IAA, đưa tất cả cờ `can_review` về `false`. | Đối chiếu và hiệu chuẩn dữ liệu từ 120 lượt gán trên toàn bộ 40 ảnh tại buổi họp Calibration. |
