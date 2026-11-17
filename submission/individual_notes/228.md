# Ghi chú cá nhân — 228

## 1. Pha 1: ba ca khó tôi tìm ra trong 40 ảnh
| Ảnh | v0 không trả lời được gì | Quyết định ban đầu của tôi |
|---|---|---|
| img_002 | Xe chở xe: các xe con trên rơ-moóc có tính là vật cần gán không? | Chỉ 1 box `truck` cho cả xe chở, không box xe con (coi là hàng chuyên chở, R-07 trong bản cá nhân) |
| img_004 / 016 / 035 | Tuk-tuk không thuộc lớp nào trong v0 | Không box tuk-tuk (ngoài taxonomy), người ngồi trên vẫn gán `person` |
| img_017 / 027 / 033 / 038 | Tượng, mộ đá hình xe, ma-nơ-canh, xe đồ chơi: v0 bảo "mọi người và phương tiện" nên dễ gán nhầm | Không gán vật không thật (R-06 trong bản cá nhân) |

Bản cá nhân đầy đủ: `submission/guidelines/individual/` (file mang mã 03228).

## 2. Pha 2: chỗ tôi bất đồng với nhóm
- **Tuk-tuk và xe buýt:** bản của tôi để ngoài taxonomy; nhóm chốt thêm hai lớp `tuk_tuk` và `bus`. **Tôi đồng ý.** Hai loại xe này xuất hiện nhiều trong ảnh (img_004, 009, 016, 031, 035, 037), nếu bỏ qua thì mô hình bị bỏ sót phương tiện thật.
- **Xe chở xe (img_002):** bản của tôi chỉ gán 1 `truck`; nhóm chốt gán thêm từng `car` trên giá chở (R-09). **Tôi đồng ý.** Các xe con trên giá vẫn có đặc trưng của `car` nên cần được gán.
- **`occluded_level`:** bản của tôi chia 4 mức theo phần trăm; nhóm từng cân nhắc bản nhị phân rồi chốt lại 4 mức (0–3) ở v1.0 mới nhất. **Tôi đồng ý** với bản 4 mức, vì ngưỡng phần trăm khớp với cách tôi đã viết ở bản cá nhân.
- **Xe đạp, đèn tín hiệu, biển báo:** bản của tôi không gán xe đạp, không có lớp cho đèn và biển; nhóm chốt thêm ba lớp `bicycle`, `traffic_light`, `traffic_sign` (taxonomy 11 lớp). **Tôi đồng ý** theo lý do nhóm ghi trong guideline v1.0: đây là các vật có trong ảnh và liên quan trực tiếp đến mục tiêu nhận diện giao thông.

## 3. Pha 3: ảnh tôi đã gán
- Số ảnh được giao: 24 (theo `assignments/228.txt`). Số box trong file nộp `labels/228.json`: 370 (person 215, motorcycle 61, car 55, truck 9, bus 6, van 5, ignore_region 5, tuk_tuk 4, traffic_light 4, traffic_sign 4, bicycle 2). Số lần tick `can_review`: 0 (đếm từ file export).
- Lần validate đầu cảnh báo 22 box cao dưới 20 px (img_002, 005, 009, 021, 024, 025, 029, 034); tôi rà lại và xử lý. Sau khi nhóm thêm lớp mới và tôi gán bổ sung, lần chạy gần nhất còn 1 cảnh báo (img_025, box #233, `traffic_sign`, cao khoảng 12,7 px, dưới ngưỡng 20 px của R-06).
- Tôi chủ động giữ box #233 vì biển báo này nhìn rất rõ và là một lớp riêng biệt, dù chưa đạt ngưỡng kích thước. Tôi biết quy tắc R-06 hiện không có ngoại lệ, nên đây là ca lệch quy tắc có chủ ý, không phải sót.
- Đề xuất cho họp calibration: nếu nhóm thấy quy tắc kích thước nên có ngoại lệ cho các lớp hạ tầng như `traffic_sign` và `traffic_light`, thì đưa ra họp để chốt. Nếu chốt thì viết vào guideline v1.1, không tự quyết một mình. Nếu nhóm giữ nguyên R-06 thì tôi sẽ xóa box #233.
- Ca lưỡng lự nhất khi gán: không có.

## 4. Pha 4–5: tôi lệch ở đâu
- [Điền sau khi có `calibration_candidates.csv`: các ca có tên tôi, lệch vì guideline mơ hồ hay vì tôi chưa nắm quy tắc.]
- Quyết định calibration tôi thấy quan trọng nhất: [điền]

## 5. Bài học
- Một điều tôi sẽ làm khác khi viết guideline lần sau: làm chuẩn chỉ ngay từ bước đầu, vì guideline và nhãn ban đầu sai thì phải sửa lại cả lô ở các pha sau.
