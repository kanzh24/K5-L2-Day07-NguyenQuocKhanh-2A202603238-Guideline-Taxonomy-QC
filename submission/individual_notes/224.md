# Ghi chú cá nhân — 03224

## 1. Pha 1: ba ca khó tôi tìm ra trong 40 ảnh

| Ảnh | v0 không trả lời được gì | Quyết định ban đầu của tôi |
|---|---|---|
| img_006 | Không có lớp xe đạp; không nói người đi xe đạp gán thế nào | Đề xuất thêm lớp `bicycle`; mỗi người đi xe đạp gán 1 `person` + 1 `bicycle` |
| img_022 | Không nói xe mô tô gắn thùng bên (sidecar) | Vẽ 1 box `motorcycle` bao trọn cả xe và thùng bên |
| img_033 | Không nói tượng/ma-nơ-canh xử lý ra sao | Ban đầu dùng `ignore_region` cho tượng lớn; sau đối chiếu R-04, nhóm chốt không vẽ box |

## 2. Pha 2: chỗ tôi bất đồng với nhóm

- Xe ba bánh: tôi đề xuất `three_wheeler`, nhóm chốt `tuk_tuk`; tôi đồng ý.
- Xe đạp: ban đầu nhóm loại khỏi taxonomy, tôi đề nghị thêm `bicycle` vì có nhiều xe đạp ở `img_003`, `img_006`, `img_029`; nhóm đã bổ sung trong v1.0 cập nhật.
- Vật không thật: tôi từng dùng `ignore_region` cho tượng/đồ chơi; nhóm chốt không vẽ box (R-04). Tôi đồng ý, nhưng nhãn `224.json` hiện còn 3 `ignore_region` ở `img_020`, `img_033`, `img_038` cần sửa khi calibration.
- Ngưỡng vật nhỏ: tôi dùng tỉ lệ nhìn thấy 20–30%, nhóm dùng ngưỡng 20 px (R-06); tôi theo nhóm, và ghi nhận 17 box < 20 px cần rà lại.

## 3. Pha 3: ảnh tôi đã gán

- Số ảnh được giao: 24. Số box đã vẽ: 412 (397 đối tượng + 15 `ignore_region`). Số lần tôi tick `can_review`: 0.
- Ca tôi lưỡng lự nhất: `img_031` — cảnh đông, 32 box gồm người/xe/biển báo/đèn, có 7 box < 20 px; tiếp theo là `img_019` (44 person) và `img_021` (69 annotation, đám đông).
- Ghi nhận thêm: 17 box nhỏ hơn ngưỡng 20 px và 3 `ignore_region` trên vật không thật cần được xem lại ở Pha 4–5.

## 4. Pha 4–5: tôi lệch ở đâu

- Chưa có `calibration_candidates.csv` tại thời điểm viết; sẽ cập nhật sau khi nhóm chạy `qc_compare.py` và calibration.

## 5. Dùng AI

- Không dùng.

## 6. Bài học

- Cần chốt taxonomy và quy tắc `ignore_region` trước khi gán; với vật không thật phải thống nhất “không vẽ box” thay vì dùng `ignore_region`; luôn kiểm ngưỡng 20 px trước khi export COCO.
