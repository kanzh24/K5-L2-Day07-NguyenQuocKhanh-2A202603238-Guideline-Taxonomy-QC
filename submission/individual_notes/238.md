# Ghi chú cá nhân — 03238 (Nguyễn Quốc Khánh)

> Copy file này thành `submission/individual_notes/<mã>.md`. Mỗi thành viên một file, ngắn gọn (khoảng một trang), tự viết.

## 1. Pha 1: ba ca khó tôi tìm ra trong 40 ảnh
| Ảnh | v0 không trả lời được gì | Quyết định ban đầu của tôi |
|---|---|---|
| `img_014` | Người đi xe máy đi nhanh bị mờ chuyển động (motion blur) trong điều kiện thiếu sáng. v0 hoàn toàn không có quy định về ngưỡng mờ nhòe, không nêu rõ khi vật thể bị mờ vệt chuyển động thì có được vẽ box hay phải bỏ qua. | Vẫn gán tách 2 box `motorcycle` và `person` nếu mắt người vẫn nhận diện được cấu trúc; tick `can_review = true`. Mờ chuyển động không tự động loại bỏ nếu mắt người vẫn nhận biết được đối tượng (R-06, R-08). |
| `img_017` | Tượng đá danh nhân cưỡi ngựa, đầu có đội nón giao thông. v0 chỉ ghi chung "gán box cho mọi người và phương tiện", không phân biệt giữa người thật tham gia giao thông với tượng đài điêu khắc nghệ thuật (dễ gây lỗi False Positive / phanh ma). | **Tuyệt đối không gán box** (cả người cưỡi lẫn con ngựa). Tượng điêu khắc nghệ thuật, không phải người thật tham gia giao thông; đóng vai trò mẫu âm nền / hard negative (R-04). |
| `img_019` | Đám đông cổ vũ có nhiều người chen chúc, không xác định được từng object riêng lẻ. v0 chỉ ghi chung chung `ignore_region` là "Vùng không cần gán từng vật", không đưa ra tiêu chí cụ thể khi nào thì gom đám đông vào vùng bỏ qua. | Đánh nhãn là `ignore_region` bao trùm toàn bộ đám đông cổ vũ vì quá nhiều người chen chúc, không thể xác định và tách ranh giới của từng đối tượng riêng biệt (R-01, R-07). |

> *(Ghi chú khảo sát thêm ở Pha 1: Ngoài 3 ca tiêu biểu trên, tôi cũng đã ghi nhận và đề xuất hướng xử lý cho các ca biên khác như xe chở xe `img_002`, xe che khuất khó phân biệt `img_016`, bãi đỗ xe dày đặc `img_030`, ma-nơ-canh `img_033`, tuk-tuk chở hàng cao trên nóc `img_035`... làm cơ sở thống nhất tại Pha 2).*

## 2. Pha 2: chỗ tôi bất đồng với nhóm
- **Đưa các lớp tượng/mô hình vào taxonomy (`_model`):** Ban đầu tôi từng cân nhắc đề xuất tạo thêm lớp phụ để đánh dấu các vật thể giả lập (như tượng, ma-nơ-canh, xe mô hình). Tuy nhiên nhóm thảo luận và thấy việc này làm loãng mục tiêu chính là nhận diện đối tượng tham gia giao thông thật. Nhóm chốt áp dụng quy tắc loại trừ R-04 (tuyệt đối không gán box cho tượng, mô hình, tranh vẽ, coi là mẫu âm nền). **Tôi hoàn toàn đồng ý** vì giúp taxonomy gọn gàng và không làm phức tạp hóa bài toán cho mô hình phát hiện.
- **Xe chở xe (`img_002`):** Tôi từng phân vân giữa phương án chỉ gán 1 box `truck` cho xe chuyên chở (coi các xe con là hàng hóa) hay gán riêng từng xe. Nhóm chốt quy tắc R-09: gán 1 box `truck` cho cả xe đầu kéo + rơ-moóc và gán thêm từng box `car` cho các xe con trên giá đỡ nếu nhận diện rõ cấu trúc. **Tôi đồng ý** vì các xe con trên giá vẫn mang đầy đủ đặc trưng thị giác của `car`, giúp mô hình học tốt hơn.
- **Mở rộng taxonomy lên 11 lớp:** Nhóm thống nhất bổ sung thêm 3 lớp thực tế: `bicycle`, `traffic_light`, `traffic_sign`, đồng thời chuẩn hóa định nghĩa cho `tuk_tuk` và `bus`. **Tôi ủng hộ** để bao quát toàn diện các yếu tố hạ tầng và phương tiện trong ảnh giao thông đô thị.
- **Phân loại xe bán tải (pickup truck):** Trong nhóm có sự tranh cãi giữa việc xếp pickup vào `car`, `van` hay `truck`. Nhóm chốt quy tắc R-10: xe bán tải có cabin và thùng hở phía sau được thống nhất xếp vào `truck`. **Tôi đồng ý** vì quy tắc rõ ràng, dứt khoát, triệt tiêu sự phân vân cảm tính của annotator.

## 3. Pha 3: ảnh tôi đã gán
- Số ảnh được giao: **24** | Số box đã vẽ: **245** | Số lần tôi tick `can_review`: **14**
- Phân bố lớp đã gán: 102 `person`, 74 `motorcycle`, 40 `car`, 8 `van`, 5 `bicycle`, 4 `truck`, 4 `traffic_sign`, 3 `tuk_tuk`, 3 `bus`, 2 `ignore_region`.
- Ca tôi lưỡng lự nhất khi gán:
  - **`img_016.jpg` (Xe mép trái bị che khuất nặng):** Phương tiện ở rìa bên trái chỉ lọt vào khung hình một phần nhỏ góc đầu và mui che nắng (`truncated = yes`). Tôi phân vân gay gắt giữa việc định danh đây là xe `tuk_tuk` hay ô tô con (`car`). Nhìn kết cấu khung sườn và mui bạt có nét đặc trưng của tuk-tuk, nhưng góc nhìn quá hẹp gây rủi ro nhầm lẫn cao. Tôi quyết định gán `tuk_tuk`, ôm sát mép nhìn thấy và tick `can_review = True` để đưa ra nhóm cùng phân tích.
  - **`img_023.jpg` (Cụm 6 xe máy góc trái đỗ ken đặc):** Dãy xe máy đỗ chen chúc, che khuất nhau sâu (`occluded_level` mức 2–3) khiến tôi lưỡng lự giữa việc tách từng box `motorcycle` (bằng cách nội suy theo ranh giới bộ phận nhìn thấy) hay gom vào một box `ignore_region`. Tôi đã chọn tách từng box và bật `can_review = True` trên cả 6 box.
  - **`img_014.jpg` (Ảnh bị motion blur):** 4 box gồm 1 `car`, 2 `motorcycle`, 1 `person` bị vệt mờ chuyển động kéo dài trong điều kiện thiếu sáng, khó xác định chính xác viền pixel tiếp xúc mặt đất; tôi đã bật `can_review = True` cho cả 4 box này.

## 4. Pha 4–5: tôi lệch ở đâu
- **Thống kê từ `calibration_candidates.csv`:** Có tổng cộng 626 ca liên quan đến mã `238` trong tổng số 2162 ca bất thường của cả nhóm, bao gồm: 161 ca `iou_thap`, 155 ca `sai_thuoc_tinh`, 139 ca `bo_sot_thua`, 133 ca `bat_dong_ignore`, 24 ca `sai_lop`, và 14 ca `co_can_review`.
- **Phân tích nguyên nhân lệch:**
  - *Lệch do tôi chưa nắm vững / áp dụng triệt để quy tắc:* 24 ca `sai_lop` (nhầm xe bán tải thành `car` ở `img_007` do thói quen nhìn dáng xe con gầm cao, hoặc nhầm van thành car) là do tôi chưa áp dụng dứt khoát quy tắc R-10; một số ca `iou_thap` do thói quen vẽ box chỉ ôm phần nhìn thấy thay vì bao trùm toàn bộ khối tích ngoại suy của vật thể khi bị che khuất.
  - *Lệch do guideline v1.0 còn mơ hồ:* 155 ca `sai_thuoc_tinh` (chủ yếu là `occluded_level` mức 1 vs 2, 2 vs 3) do guideline v1.0 chỉ mô tả định tính bằng lời mà không có thang đo tỉ lệ phần trăm trực quan; 133 ca `bat_dong_ignore` do guideline chưa nêu ngưỡng số lượng/mật độ cụ thể khi nào nên gom cụm đám đông/bãi xe dày đặc vào `ignore_region`.
- **Quyết định calibration tôi thấy quan trọng nhất:**
  - Quy tắc **R-08** (tách dứt khoát người điều khiển và phương tiện thành 2 box riêng) và **R-10** (chuẩn hóa toàn bộ xe bán tải thành `truck`).
  - Chuẩn hóa điều kiện bàn giao: lọc triệt để các box dưới ngưỡng $h < 20\text{ px}$ (R-06) và gỡ bỏ 100% cờ `can_review` (chuyển về `false`) sau khi đã chốt quy tắc tại Phase 5 để đảm bảo lô nhãn đạt chuẩn QC tuyệt đối.

## 5. Dùng AI
- **Công cụ sử dụng:** Trợ lý AI (Google Antigravity / Gemini) kết hợp các kịch bản kiểm thử Python.
- **Mục đích sử dụng:**
  - Hỗ trợ rà soát, đối chiếu và chuẩn hóa văn bản guideline (so sánh các điểm bất đồng giữa 5 bản cá nhân từ Pha 1 để đưa vào guideline v1.0 chung của nhóm).
  - Viết và chạy các đoạn mã Python tự động kiểm tra cú pháp, cấu trúc dữ liệu COCO 1.0 của file `submission/labels/238.json` (trích xuất thống kê số lượng ảnh, tổng số box, phân bố category, đếm số box có cờ `can_review`).
  - Tra cứu nhanh đặc tả kỹ thuật COCO format và quy ước tính IoU.
  *(Cam kết: Không sử dụng bất kỳ công cụ AI nào để gán nhãn tự động thay thế mắt người — toàn bộ 24 ảnh và 245 box đều do tôi tự quan sát và vẽ thủ công trên CVAT).*
- **Cách tôi kiểm tra kết quả:**
  - Chạy các script thẩm định chính thức của bài lab: `py scripts/validate_labels.py submission/labels/238.json` và `py scripts/check_submission.py` để đảm bảo file nhãn không có lỗi cấu trúc, đủ 24 ảnh theo phân công.
  - Đối chiếu thủ công từng số liệu thống kê (24 ảnh, 245 box, 14 can_review) với giao diện trực quan trên CVAT.
  - Tự đọc lại toàn bộ nội dung văn bản để đảm bảo tính logic, khách quan và phản ánh đúng thực tế quá trình làm việc của bản thân và nhóm.

## 6. Bài học
- **Một điều tôi sẽ làm khác khi viết guideline lần sau:**
  - **Bổ sung ảnh ví dụ trực quan (crop visuals) ngay từ v1.0:** Thay vì chỉ mô tả bằng câu chữ dễ gây hiểu nhầm hoặc tranh cãi cảm tính, guideline cần có ảnh chụp crop minh họa cho từng trường hợp biên: 1 ảnh mẫu đạt chuẩn (Good) và 1 ảnh vi phạm (Bad) cho các ca khó (xe bán tải thùng hở, người ngồi trên xe máy, xe chở xe, tượng điêu khắc).
  - **Định lượng hóa các mốc thuộc tính:** Với thuộc tính ước lượng như `occluded_level`, cần quy định rõ theo mốc phần trăm (0: 0-10%, 1: 10-35%, 2: 35-70%, 3: >70%) gắn liền với các điểm neo nhận diện trực quan trên vật thể. Điều này sẽ giúp tăng mạnh độ đồng thuận IAA giữa các annotator độc lập ngay từ vòng gán đầu tiên và giảm thiểu đáng kể khối lượng ca bất thường cần họp calibration.
