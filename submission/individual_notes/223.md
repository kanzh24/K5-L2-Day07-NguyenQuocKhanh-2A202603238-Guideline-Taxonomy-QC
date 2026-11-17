# Ghi chú cá nhân — 223

- **Ngày cập nhật:** 17/11/2026.
- **Mã phân công:** `223` (mã dùng trong tên guideline cá nhân: `03223`).
- **Bằng chứng:** [bản nhãn COCO 223.json](../labels/223.json). Ghi chú được soạn với AI hỗ trợ, dựa trên quá trình làm việc và bản xuất hiện có; các phần chưa có kết quả được nêu riêng.

## 1. Pha 1: ba ca khó trong bộ 40 ảnh

- **`img_004` — Tuk-tuk:** v0 chưa quy định xe ba bánh có động cơ thuộc lớp nào. Trong bản cá nhân ban đầu, tôi đề xuất để tuk-tuk ngoài taxonomy, không ép vào `motorcycle`; người ngồi trong xe vẫn gán `person`.
- **`img_011` — Nhân vật Predator:** hình dáng giống người không đủ để phân biệt người hóa trang với tượng/mô hình. Bản cá nhân để ca này cần review, chỉ gán `person` nếu xác minh là người thật; xe mô tô thật xét riêng.
- **`img_024` — Xe vàng trên nóc wagon:** cần phân biệt xe thật đang được vận chuyển với mô hình trang trí. Ban đầu tôi đề xuất không gộp vật trên nóc vào box xe dưới và giữ ca cần xác minh trước khi quyết định có gán `car` riêng.

## 2. Pha 2: khác biệt giữa bản cá nhân và bản nhóm

Bản cá nhân ban đầu có 6 lớp, chưa đưa `bus` và `tuk_tuk` vào phạm vi. Bản thống nhất v1.0 của nhóm bổ sung hai lớp này, đồng thời chốt Predator ở ảnh 011 và xe vàng trên nóc ở ảnh 024 là mô hình, không gán nhãn đối tượng. Tôi lấy quyết định nhóm làm chuẩn để tránh mỗi người xử lý cùng một ca theo cách khác nhau.

Một thay đổi khác là `occluded_level`: bản cá nhân chia mức 0–3, còn bản nhóm v1.0 đã đọc chuyển sang checkbox. Bài học ở đây là phải đồng bộ cả guideline, taxonomy và cấu hình Project CVAT; sửa tài liệu mà giữ cấu hình cũ sẽ tạo dữ liệu khác kiểu.

## 3. Pha 3: phạm vi và bản nhãn tôi đã xuất

Tôi đã sử dụng CVAT và lưu bản xuất COCO tại `submission/labels/223.json`. Danh sách ảnh khớp **24/24 ảnh** được giao cho mã `223`, không thiếu hoặc thừa ảnh so với phân công nhóm đã clone.

- Bản xuất có **320 annotation**, gồm **270 box đối tượng** và **50 vùng `ignore_region`**. Đây là số box trong file, không phải số box tôi tự vẽ mới bằng tay.
- Có **218/270 box đối tượng** còn `can_review=true`, 52 box mang giá trị false. Đây là số cờ còn bật khi xuất, không phải số lần thao tác tick trong CVAT.
- Cấu hình xuất có **11 lớp**; ngoài 8 lớp trước đó còn có `bicycle`, `traffic_light`, `traffic_sign`. Thực tế có 10 box `bicycle`, 1 box `traffic_sign`, chưa có annotation `traffic_light`.
- Ảnh 017, 020, 027 được giữ trong COCO dù không có box mục tiêu, tương ứng các ca tượng, phản chiếu/xe chủ và tượng pickup.

Những ca cần chú ý khi kiểm lại là ảnh 003 (người và phương tiện chen nhau, hàng gánh không thuộc box người), ảnh 032 (người đứng sát nhau, ranh giới cá thể) và ảnh 040 (xe/người bị che hoặc cắt mép).

**Điểm còn phải đồng bộ:** file hiện có 195 box đối tượng dùng `occluded_level` dạng 0–3 và 75 box dùng boolean. Tôi cần đối chiếu cấu hình 11 lớp với phiên bản nhóm dùng để bàn giao và thống nhất kiểu thuộc tính trước khi coi đây là nhãn cuối. Chưa thể kết luận các cờ review đã được xử lý chỉ từ việc export thành công.

## 4. Pha 4–5: đối chiếu và calibration

Trong các artifact hiện có tôi chưa có `calibration_candidates.csv`, số đo IAA hoặc nhật ký calibration để xác định cụ thể mình lệch với thành viên nào. Vì vậy tôi chưa ghi kết quả đồng thuận hay quyết định Release.

Khi nhóm đối chiếu, tôi cần ưu tiên các box còn cờ review, vùng ignore trong đám đông và sự khác biệt cấu hình lớp/thuộc tính. Với lỗi do quy tắc chưa rõ, cần ghi quyết định chung vào guideline và nhật ký; với lỗi thực hiện, sửa đúng box rồi kiểm lại. Quyết định tôi thấy quan trọng nhất là phân biệt vật thật với mô hình và tách người khỏi phương tiện, vì hai người vẫn có thể gán giống nhau nhưng cùng sai.

## 5. Dùng AI và cách kiểm lại

Tôi đã dùng **Codex** để hỗ trợ xem ảnh, soạn guideline, giải thích quy tắc, tạo cấu hình Raw Labels, viết script và soạn ghi chú này. **YOLO11m và YOLO11m-seg** được dùng tạo nhãn sơ bộ và hỗ trợ xác định phần nhìn thấy; sau đó có các chỉnh sửa theo quan sát của AI trước khi nhập vào CVAT. Tôi tiếp tục thao tác trong CVAT và xuất bản nhãn đang lưu.

Tôi không mô tả phần pre-label này là toàn bộ box được vẽ độc lập bằng tay. Để kiểm lại, dùng ảnh gốc và ảnh chồng box, đối chiếu quyết định nhóm cho từng ca, rồi kiểm file xuất. Đã đối chiếu danh sách 24 ảnh, đếm annotation/cờ review và kiểm cơ bản ID, tọa độ box; các kiểm tra đó không thay thế việc duyệt nội dung từng box. Những điểm chưa thống nhất được giữ lại trong ghi chú, không tự tạo số đo IAA hoặc kết quả QC.

## 6. Bài học

Lần sau tôi sẽ khóa phiên bản guideline cùng taxonomy và thử vòng nhập–xuất trên một vài ảnh khó trước khi gán cả lô. Tôi cũng sẽ ghi ngắn lý do ngay khi đánh dấu `can_review`, để lúc calibration truy lại được vùng ảnh và quy tắc đang gây phân vân.
