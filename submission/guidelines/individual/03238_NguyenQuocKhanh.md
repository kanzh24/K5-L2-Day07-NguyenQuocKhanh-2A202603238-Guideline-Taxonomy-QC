# Guideline gán nhãn — <phiên bản> — <mã hoặc nhóm>

> Copy file này (hoặc `guideline_v0_starter.md`) rồi viết. Một guideline đủ gồm **5 phần**. Câu nào của v0
> làm hai người đọc ra hai cách khác nhau thì phải viết lại cho một cách hiểu duy nhất.

- Phiên bản: 1.0
- Ngày: 16/11/2026
- Người viết / duyệt: Nguyễn Quốc Khánh

## 1. Mục tiêu

- **Mô hình sử dụng**:
  - Dữ liệu nhãn phục vụ huấn luyện và đánh giá các mô hình học sâu thị giác máy tính phát hiện vật thể 2D (2D Object Detection như YOLOv8/v10, Faster R-CNN, RT-DETR) hoạt động theo thời gian thực.
  - Ứng dụng chính trong hệ thống hỗ trợ lái xe nâng cao (ADAS), xe tự hành (Autonomous Vehicles) và hệ thống camera giám sát giao thông thông minh đô thị (Smart City ITS).

- **Mục đích gán nhãn**:
  - Định vị chính xác tọa độ không gian 2D (bounding box) và phân loại các tác nhân tham gia giao thông (người, xe con, xe máy, xe đạp, xe buýt, xe tải, tuk-tuk), hạ tầng tín hiệu (`traffic_light`, `traffic_sign`) và chướng ngại vật/bối cảnh đường phố.
  - Cung cấp cơ sở cho các module nhận thức phía sau: ước lượng khoảng cách (Depth/Monocular 3D estimation), theo dõi đối tượng đa mục tiêu (MOT - Multi-Object Tracking), dự đoán quỹ đạo (Trajectory Prediction) và lập kế hoạch di chuyển / cảnh báo va chạm (Path Planning & Collision Avoidance).
  - Phân tách rõ ràng giữa các thực thể giao thông thực tế với các thực thể nhân tạo/trang trí/mô hình tĩnh (`person_model`, `car_model`, `other_model`, tượng, tranh vẽ) để hệ thống nhận thức không đưa ra phán đoán sai lệch.

- **Phân tích rủi ro & các sai sót nguy hiểm (Failure Analysis)**:
  - **Bỏ sót vật thể (False Negative - FN) — Cực kỳ nguy hiểm**:
    - Bỏ sót người đi bộ (`person`), người đi xe máy (`motorbike`) hoặc chướng ngại vật ở cự ly gần hoặc trong góc khuất: Hệ thống điều khiển xe tự hành không kích hoạt phanh tự động khẩn cấp (AEB - Autonomous Emergency Braking), dẫn đến va chạm trực tiếp gây chết người hoặc tai nạn thảm khốc.
    - Bỏ sót biển báo dừng/cấm (`traffic_sign`) hoặc đèn đỏ (`traffic_light`): Phương tiện vượt giao lộ trái phép, gây tai nạn giao cắt liên hoàn.
  - **Nhận diện nhầm (False Positive - FP) — Nguy hiểm cao**:
    - Nhận diện nhầm tranh vẽ quảng cáo 2D, hình in trên áo/xe buýt, hình phản chiếu qua gương/kính, hoặc các tượng/mô hình (`person_model`, `car_model`) thành người hoặc xe thật: Dẫn đến hiện tượng "phanh ma" (phantom braking) — xe phanh gấp bất ngờ giữa đường ở tốc độ cao mà không có chướng ngại vật thật, gây nguy cơ bị các phương tiện phía sau đâm dồn toa.
  - **Sai lệch Bounding Box (Bounding Box Drift / Sai kích thước & vị trí) — Nguy hiểm**:
    - Box vẽ quá rộng (ôm thừa nhiều nền) hoặc quá chật (cắt cụt bánh xe chạm đất hoặc đầu người): Module ước lượng khoảng cách 3D từ camera đơn (monocular depth) sẽ tính toán sai cự ly tiếp đất và sai thể tích vật thể, dẫn đến tính sai thời gian tới điểm va chạm (Time-to-Collision - TTC) và kích hoạt phanh quá muộn hoặc đánh lái chệch làn.
  - **Nhận diện sai lớp (Misclassification) — Rủi ro cao**:
    - Nhầm lẫn giữa phương tiện tải trọng lớn (`truck`, `bus`) với ô tô nhỏ (`car`): Khiến hệ thống ước tính sai bán kính quay xe, điểm mù và gia tốc phanh của phương tiện phía trước/bên cạnh, gây nguy hiểm khi chuyển làn hoặc vượt xe.
    - Nhầm lẫn giữa người thật (`person`) và mô hình tĩnh (`person_model`): Khiến thuật toán phân loại mức độ ưu tiên phản ứng và dự đoán hành vi bị sai lệch.

## 2. Lớp và thuộc tính
### Lớp
| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `person` | Là con người, bao gồm cả em bé, trẻ em, người lớn, người già. Người có thể ở trong xe, trên xe, đứng, đi, chạy, ngồi, nằm, hoặc có tư thế bất kỳ. | Con người đang ngồi trên xe ô tô, con người đang đi bộ trên vỉa hè, trẻ em đang chơi đùa trên sân cỏ, người già đang ngồi nghỉ trên ghế đá. | Con người được thể hiện qua hình ảnh, tượng, tranh vẽ, video trên màn hình, hình xăm trên da, hình ảnh trên áo, quần. | 
| `car` | Là phương tiện giao thông đường bộ có 4 bánh, dùng động cơ đốt trong hoặc động cơ điện, dùng để chở người và hành lý. | Xe ô tô con, xe SUV, xe bán tải, xe tải nhỏ. | Xe bus, xe tải nhỏ, mô hình đồ chơi |
|motorbike|Là phương tiện di chuyển hai bánh dùng động cơ đốt trong hoặc động cơ điện, dùng để chở người và hành lý|Xe máy, xe tay ga| Xe đạp |
| `vegetation` | Là cây, bụi cây, tán lá  |Các cây xanh|  Đám cỏ dưới mặt đất |
| `bicycle` | Là phương điện di chuyển 2 bánh cần dùng sức người để đạp | Xe đạp đang có người sử dụng | Xe máy | 
| `traffic_light` | Là thiết bị giao thông có đèn tín hiệu dùng để điều khiển giao thông | Đèn giao thông ở ngã tư đường, đèn tín hiệu giao thông trên cao | Cột đèn đường, biển báo giao thông |
| `traffic_sign` | Là thiết bị giao thông có biển báo dùng để điều khiển giao thông | Biển báo giao thông ở ngã tư đường, biển báo giao thông trên cao | Cột đèn đường, đèn giao thông |
| `building` | Building là bề mặt thuộc công trình/tòa nhà | Tòa nhà | Công trình cầu, hàng rào | 
| `truck` | Truck là phương tiện di chuyển trên đường dùng để chở người và hàng hóa  |Xe tải nhỏ| Xe bus |
| `bus` | Bus là phương tiện di chuyển trên đường dùng để chở người và hàng hóa | Xe bus | Xe tải lớn |
| `person_model` | Là các mô hình dạng người, không phải người thật | Tượng hình người | Con người |
| `car_model` | Là các mô hình dạng xe chỉ mục đích trang trí, không phải xe thật | Mô hình xe đồ chơi, mô hình xe trang trí | Xe ô tô thật |
| `terrain` | Là bề mặt địa hình tự nhiên như đất, cỏ thấp | Bãi cỏ, bãi đất trống | Cỏ cao trên cây, cây lớn trên mặt đất |
| `tuk_tuk` | Là phương tiện di chuyển ba bánh dùng động cơ đốt trong hoặc động cơ điện, dùng để chở người và hành lý | Xe tuk tuk | Xe ô tô |
| `other_model` | Là các mô hình khác không phải mô hình người hoặc mô hình xe | Mô hình con ngựa | Mô hình người |

### Xử lý vật không thuộc lớp nào
- **Đối tượng bối cảnh / hạ tầng thông thường** (mặt đường nhựa, vỉa hè phẳng, bầu trời, dải phân cách bê tông, hàng rào thông thường): **Không gán box** (coi là background).
- **Vật thể thuộc các lớp quan tâm nhưng ở quá xa / quá nhỏ (< 20px) hoặc cụm đám đông chồng lấn dày đặc**: Gom vào vùng **`ignore_region`** theo quy tắc `R-06` và `R-07`. Vùng này bỏ qua không tính lỗi bỏ sót (False Negative) cho mô hình.
- **Vật thể lạ, hiếm gặp hoặc nằm ngoài danh mục taxonomy** (như xe cứu thương, xe chuyên dụng sân bay, xe công trình, động vật di chuyển trên đường): Không tự ý tạo lớp mới một mình trong Pha 1; vẽ box theo lớp gần nhất hoặc không gán, đồng thời tick **`can_review = true`** và ghi lại mã ảnh để đề xuất nhóm bổ sung lớp mới trong buổi họp Pha 2.

### Thuộc tính (Attributes)

Tất cả các bounding box (trừ `ignore_region`) đều phải được gán đầy đủ 3 thuộc tính chuẩn hóa dưới đây. Các thuộc tính này được liên kết chặt chẽ với quy tắc vẽ box ở Phần 3 (`R-02`, `R-03`, `R-12`) để đảm bảo tính định lượng đo lường được:

| Tên thuộc tính | Kiểu / Giá trị | Mặc định | Định nghĩa & Ý nghĩa | Ngưỡng đo lường định lượng cụ thể | Quy tắc liên kết |
|---|---|---|---|---|---|
| `occluded_level` | `select`<br>(`0`, `1`, `2`, `3`) | `0` | Đánh giá mức độ vật thể bị che khuất bởi các vật cản khác (cột, cây cối, xe cộ, người khác). | • `0` (Không che): Nhìn thấy trọn vẹn hoặc bị che $< 5\%$ diện tích bề mặt.<br>• `1` (Che nhẹ): Bị che khuất từ $5\%$ đến $30\%$ diện tích.<br>• `2` (Che vừa): Bị che khuất từ $31\%$ đến $70\%$ diện tích.<br>• `3` (Che nặng): Bị che khuất $> 70\%$ diện tích (nhưng phần còn lại $\ge 10-15\%$ vẫn đủ nhận diện rõ ràng lớp).<br>*(Lưu ý: Nếu bị che $> 90\%$ không còn nhận dạng chắc chắn thì bỏ qua không gán).* | `R-02` |
| `truncated` | `select`<br>(`no`, `yes`) | `no` | Đánh giá vật thể có bị cắt cụt do nằm ở rìa/mép khung ảnh hay không. | • `no`: Vật thể nằm trọn vẹn trong khung ảnh hoặc mép ngoài chỉ chạm sát biên làm mất $< 10\%$ diện tích.<br>• `yes`: Vật thể nằm ở mép bức ảnh và bị khung hình cắt mất từ $10\%$ diện tích trở lên.<br>*(Lưu ý: Nếu phần lọt vào ảnh quá nhỏ $< 20\text{ px}$ chiều cao/rộng thì bỏ qua không gán).* | `R-03` |
| `can_review` | `checkbox`<br>(`true`, `false`) | `false` | Đánh dấu các ca khó, ca biên mơ hồ cần cả nhóm họp xem xét lại ở Pha 2. | • `false`: Người gán tự tin xác định lớp và bounding box theo guideline.<br>• `true`: Bật khi phân vân giữa 2 lớp (vd: xe bán tải độ thùng giữa `car`/`truck`), vật thể bị biến dạng/che khuất nghiêm trọng, hoặc chưa có quy định rõ ràng trong guideline. | `R-12` |

## 3. Quy tắc vẽ box

Mọi bounding box phải tuân thủ nghiêm ngặt các quy tắc chuẩn hóa dưới đây (được định danh bằng mã `R-01` đến `R-12` để dẫn chiếu thống nhất):

### R-01: Ôm sát phần nhìn thấy (Tight Bounding Box — Visible Pixels Only)
- Bounding box là hình chữ nhật đứng (axis-aligned 2D rectangle) tiếp xúc sát 4 mép ngoài cùng (trên, dưới, trái, phải) của **phần điểm ảnh thực tế nhìn thấy được** của vật thể.
- **Tuyệt đối KHÔNG ước lượng hay vẽ bù** phần thân thể hoặc bộ phận xe bị khuất sau vật khác hoặc nằm ngoài khung hình.
- Độ dung sai viền nền (background margin) không vượt quá 3–5 pixel. Không được vẽ lẹm cắt mất chi tiết của vật và không vẽ dư thừa khoảng trống nền.

### R-02: Xử lý vật thể bị che khuất (Occlusion Handling)
- Nếu vật thể bị che một phần bởi vật cản khác (cột điện, cây cối, xe khác): Bounding box chỉ bao bọc trọn vẹn phần nhìn thấy được của vật thể đó.
- **Vật thể bị che cắt rời thành 2 mảnh nhìn thấy**:
  - Nếu hai mảnh rõ ràng thuộc về cùng một vật thể duy nhất (ví dụ thân xe ô tô bị cột đèn chắn ngang chính giữa): Vẽ **01 box duy nhất** bao trọn cả hai mảnh phần nhìn thấy, đồng thời tăng thuộc tính `occluded_level`.
  - Nếu phần bị che khuất quá lớn (> 80% diện tích) khiến hai mảnh nằm tách rời xa nhau và không thể khẳng định chắc chắn chúng thuộc cùng một vật: Không vẽ 1 box gộp mà gán từng phần nhìn thấy nhận diện được (hoặc đánh dấu `can_review`).
- **Gán thuộc tính `occluded_level`**:
  - `0`: Không bị che khuất (nhìn thấy trọn vẹn hoặc bị che < 5% diện tích).
  - `1`: Bị che khuất nhẹ (từ 5% đến 30% diện tích).
  - `2`: Bị che khuất vừa (từ 31% đến 70% diện tích).
  - `3`: Bị che khuất nặng (> 70% diện tích, nhưng phần còn lại vẫn đủ đặc trưng nhận dạng rõ ràng). Nếu vật bị che > 90% không còn nhận dạng được: **Bỏ qua không gán**.

### R-03: Vật thể bị cắt mép ảnh (Truncation)
- Khi vật thể nằm sát đường biên và một phần bị cắt khỏi khung ảnh: Vẽ mép box chạm đúng đường biên ảnh (tọa độ pixel mép ảnh). Không bao giờ vẽ box vượt ra ngoài giới hạn kích thước ảnh.
- Gán thuộc tính `truncated`:
  - `yes`: Vật bị cắt mất từ 10% diện tích trở lên do nằm ở mép ảnh.
  - `no`: Vật nằm trọn vẹn trong khung ảnh hoặc chỉ chạm mép không đáng kể (< 10%).
- Nếu phần lọt vào ảnh quá nhỏ (< 20 pixel chiều cao/rộng) hoặc không còn đủ đặc trưng để xác định lớp: **Bỏ qua không gán**.

### R-04: Phản chiếu, hình in 2D, tượng và mô hình (Real vs. Non-Real / Artifacts)
- **Tượng người, ma-nơ-canh trong tủ kính/ngoài trời**: Gán nhãn `person_model`, **tuyệt đối không** gán nhãn `person`.
- **Mô hình ô tô trưng bày, xe đồ chơi, tượng xe**: Gán nhãn `car_model`, **tuyệt đối không** gán nhãn `car`.
- **Mô hình động vật hoặc vật thể khác (tượng ngựa, tượng voi...)**: Gán nhãn `other_model`.
- **Tranh ảnh 2D, biển quảng cáo, poster, hình in trên áo/thân xe buýt**: **KHÔNG vẽ box**, vì đây là hoa văn/họa tiết 2D phẳng, không phải vật thể 3D trong không gian giao thông.
- **Hình phản chiếu qua gương chiếu hậu, cửa kính, mặt nước**: **KHÔNG vẽ box** cho ảnh ảo phản chiếu.
- **Bóng đổ (Shadow)**: **KHÔNG** đưa bóng đổ của người hoặc phương tiện trên mặt đường vào trong bounding box.

### R-05: Xe chủ mang camera (Ego Vehicle)
- Phần nắp ca-pô (hood), kính chắn gió, cần gạt nước hoặc viền gương của chính chiếc xe chở camera chụp ảnh xuất hiện ở rìa dưới khung hình: **KHÔNG vẽ box** lớp phương tiện (`car`/`truck`...) cho xe chủ.

### R-06: Ngưỡng kích thước tối thiểu và độ mờ nhòe (Min Box Size)
- **Ngưỡng kích thước**: Bỏ qua tất cả các vật thể có chiều cao hoặc chiều rộng **nhỏ hơn 20 pixels** (`min_box_height_px < 20`).
- **Độ phân giải & mờ nhòe**: Các vật thể ở quá xa, bị mờ nhòe do chuyển động (motion blur) hoặc thiếu sáng đến mức mắt người không thể nhận diện chắc chắn hình dáng và lớp: **Bỏ qua không gán**, không cố suy đoán.

### R-07: Đám đông và vùng vật thể dày đặc (Crowd & Ignore Region)
- Khi có một nhóm người hoặc cụm phương tiện xếp chồng chéo quá dày đặc ở hậu cảnh (background) không thể phân tách ranh giới từng đối tượng riêng lẻ: Vẽ **01 box `ignore_region`** bao trùm toàn bộ khu vực đó.
- Vùng `ignore_region` không tính lỗi bỏ sót (False Negative) cho mô hình trong quá trình huấn luyện và đánh giá.

### R-08: Người điều khiển và phương tiện (Person & Vehicle Interaction)
- Áp dụng nguyên tắc **tách rời 02 bounding box độc lập**:
  - **Box 1 — `person`**: Bao gồm người lái hoặc hành khách ngồi trên xe. Box ôm từ đỉnh đầu/mũ bảo hiểm xuống đến vị trí thấp nhất của cơ thể nhìn thấy được (bàn chân, đùi). Mỗi người ngồi trên xe được vẽ **01 box `person` riêng**.
  - **Box 2 — `motorbike` / `bicycle`**: Bao trùm toàn bộ phương tiện (thân xe, hai bánh xe, ghi-đông, gương chiếu hậu).
  - Hai box này sẽ có vùng đè lấn lên nhau (overlap) ở khu vực người ngồi trên yên xe. Thuộc tính `occluded_level` của người và xe được tính theo tỷ lệ bị che khuất thực tế.

### R-09: Xe chở xe và hàng hóa chuyên chở (Car Carrier & Cargo)
- **Phương tiện vận tải (`truck`)**: Bounding box của `truck` phải bao trọn cả cabin, khung gầm, sàn xe và kết cấu giá đỡ thùng hàng.
- **Ô tô con được chở trên giá sàn xe tải (ví dụ trên `img_002.jpg`)**:
  - Vẫn vẽ **box `car` riêng cho từng chiếc xe con** trên giàn xe tải nếu chúng nhìn thấy rõ ràng và thỏa mãn điều kiện kích thước >= 20px.
  - Gán thuộc tính `occluded_level` phù hợp (mức 1, 2 hoặc 3) do bị che khuất bởi khung giàn sắt hoặc xe bên cạnh.
- **Hàng hóa thông thường trên thùng xe**: Box của xe (`truck`/`motorbike`) bao trùm luôn cả phần hàng hóa chằng buộc trên xe nếu hàng hóa đó gắn liền với xe khi lưu thông.

### R-10: Đèn tín hiệu giao thông và Biển báo (`traffic_light`, `traffic_sign`)
- `traffic_light`: Bounding box ôm sát toàn bộ cụm hộp đèn tín hiệu (gồm cả khung viền che đèn nếu có). **Không** vẽ bao gồm cột đèn đứng hoặc xà ngang kéo dài treo đèn.
- `traffic_sign`: Bounding box ôm sát mặt biển báo giao thông (hình tròn, tam giác, chữ nhật). **Không** bao gồm cột cắm biển báo.
- Nếu nhiều biển báo hoặc nhiều cụm đèn gắn chung một cột nhưng cách rời nhau: Vẽ từng box riêng biệt cho từng mặt biển / cụm đèn.

### R-11: Công trình xây dựng và môi trường (`building`, `vegetation`, `terrain`)
- `building`: Vẽ box bao quanh các khối bề mặt công trình/tòa nhà rõ rệt ở tiền cảnh hoặc trung cảnh.
- `vegetation`: Vẽ cho các tán cây lớn, rặng cây, bụi cây có ranh giới rõ ràng. Không vẽ cho từng ngọn cỏ đơn lẻ.
- `terrain`: Dành cho các mảng địa hình tự nhiên bằng phẳng hoặc đồi đất, bãi cỏ thấp trên mặt đất.

### R-12: Xử lý ca mơ hồ / phân vân (`can_review`)
- Khi gặp tình huống biên không hoàn toàn chắc chắn (ví dụ: xe bán tải độ nắp thùng cao gây phân vân giữa `car` và `truck`, hoặc vật thể bị che khuất nghiêm trọng):
  - Người gán vẫn chủ động vẽ box theo phán đoán tốt nhất của bản thân dựa trên guideline.
  - **Bắt buộc tích chọn thuộc tính `can_review = true`** trên box đó.
  - Ghi lại tên file ảnh, vị trí và lý do phân vân vào bảng Ca khó ở Phần 4 để đưa ra thảo luận thống nhất cùng nhóm trong buổi họp Pha 2.

## 4. Ca khó
| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| image_017.jpg |Mô hình người cưỡi mô hình ngựa có đầu bị đội vật thể |  | Chỉ tô phần nhìn thầy của `person_model` và đánh ocluded |Vì bị che khuất nửa phần đầu nên xác định nên không vẽ |
| image_024.jpg | Mô hình xe ô tô được đặt trên xe ô tô thật |---| Tô xe ở dưới là `car` còn mô hình ở trên là `car_model` |Vì xe trên là xe giả còn xe dưới là xe thật |
| image_033.jpg | Mô hình người và cửa kính phản chiếu người |---| Chỉ đánh mô hình người là `person_model` còn mô hình người phản chiếu không đánh | Vì mô hình người và mô hình người phản chiếu không phải người thật |

Nếu chưa chốt, đánh dấu `can_review` trên box hoặc ghi vấn đề cấp ảnh. Dùng mã quy tắc trong nhật ký QC.

## 5. QC và phiên bản
Ngưỡng QC áp dụng, ai quyết ca mơ hồ, lịch sử thay đổi:

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| | | |
    