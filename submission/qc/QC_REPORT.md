# Báo cáo QC — Nhóm Lab 402

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Đặc tả và kiểm tra chất lượng (QC) nhãn 2D bounding box phục vụ huấn luyện mô hình phát hiện người và phương tiện. Giới hạn: tập ảnh gồm 40 ảnh tuyển chọn luyện ca biên khó, độ đại diện cho dữ liệu vận hành thực tế ở quy mô lớn chưa được đánh giá toàn diện.
- Nguồn và phép sử dụng dữ liệu: Xem data/ATTRIBUTION.csv (ảnh thu thập từ các nguồn mở/cc-by, sử dụng phi thương mại cho mục đích học tập và nghiên cứu).
- Manifest và checksum nhãn cuối: batch_manifest.json (SHA-256: 4263a8be1a771fec17ba986ef6fd5bc34c680210a05318f10ea1212012d15b1d); labels_calibrated.json (SHA-256: 22d67c24751e0e8a1a8192c1edc9513f6a8f04674421cd3659541e12f2e8cdc8).
- Guideline và taxonomy khi gán / khi bàn giao: Khi gán độc lập dùng guideline v1.0 và taxonomy v1.0 (11 lớp); khi bàn giao dùng guideline v1.1 calibrated (hiệu chuẩn câu từ, làm rõ quy tắc) và giữ nguyên taxonomy v1.0 (11 lớp).

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 120/120 lượt gán hoàn tất (40 ảnh x 3 người gán độc lập).
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 1032 cặp box ghép được; IoU trung bình: 0.836; đồng thuận lớp: 95.5%; mẫu số tính trên các cặp box có IoU >= 0.5.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 264 box chưa ghép phía A, 573 box chưa ghép phía B; 13 box bỏ qua do ignore_region; 528 ca bất đồng vùng ignore.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: img_002.jpg (cùng gán 1 box truck ôm cả xe con trên sàn thay vì vẽ tách từng box car); img_028.jpg (cùng gán tượng ngựa trên thùng bán tải thành đối tượng giao thông); img_040.jpg (cùng gán hình ảnh in 2D trên thân taxi). Đối chiếu điều khoản R-04 và R-09 trong guideline v1.1 để kiểm tra lại và loại trừ.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: Đã xử lý toàn bộ 2162 ca bất thường từ IAA, dẫn xuất chi tiết tại submission/qc/calibration_decisions.csv; người chốt: 238 (Nguyễn Quốc Khánh), người kiểm lại: 228 (Đoàn Quang Minh, đạt).
- Tác động v1.1 lên nhãn và ảnh: Giữ nguyên taxonomy 11 lớp (thay đổi editorial); phạm vi tác động 38 ảnh flagged; lọc bỏ 22 box nhỏ dưới 20px theo R-06; đưa toàn bộ 232 cờ can_review về false theo R-12; chuẩn hóa 4 mức occluded_level theo R-02 và truncated theo R-03.
- Tham chiếu bằng chứng khi Rework: Không phát sinh Rework; vòng 1 đạt kết quả Release trực tiếp.

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: 228 (Đoàn Quang Minh), 17/11/2026
- Đã duyệt / cả lô: 40/40 ảnh đã duyệt (100% cả lô).
- Ảnh lỗi / ảnh đã duyệt: 0/40 = 0.0%.
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 0/680 = 0.0%.
- Vật bỏ sót, ảnh và vùng: 0 vật bỏ sót.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Dẫn phiếu submission/qc/reviews/round-01/qc_review.csv; toàn bộ 38 ca khó và 2 ảnh còn lại đều đạt trạng thái sạch (error_type = khong); không có lỗi lặp.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Lỗi máy chặn được: box vượt ra ngoài khung hình canvas (tọa độ pixel ngoài W, H) hoặc box có chiều cao dưới ngưỡng 20px (R-06); Lỗi cần người đọc ảnh: phân biệt người thật đang đi bộ với ma-nơ-canh trong tủ kính (img_033) hoặc tượng nghệ thuật điêu khắc cưỡi ngựa (img_017) theo R-04.

## 5. Quyết định bàn giao

- Quyết định: Release
- Lý do, người nhận và phạm vi sử dụng: Tỉ lệ lỗi ảnh 0.0% (đạt ngưỡng cho phép <= 2.0%); tỉ lệ lỗi annotation 0.0%; không còn cờ can_review. Bàn giao cho nhóm phát triển mô hình Object Detection phục vụ huấn luyện và đánh giá trên ảnh giao thông đô thị bài lab Day 07.
- Nếu còn việc: Không còn tồn đọng; toàn bộ hồ sơ dữ liệu đã được khóa và xác thực tính toàn vẹn qua batch_manifest.json.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: C393 (img_006.jpg, rule R-08, tách riêng box person và motorcycle, kiểm tra trực tiếp trên nhãn labels_calibrated.json); C720 (img_012.jpg, rule R-10, chốt bán tải là truck); C819 (img_016.jpg, rule R-01, chốt xe 3 bánh là tuk_tuk). Tất cả đều khớp hoàn toàn giữa quyết định, nhãn cuối và báo cáo.
