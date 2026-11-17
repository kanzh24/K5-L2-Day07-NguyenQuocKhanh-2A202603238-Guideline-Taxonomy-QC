# Ghi chú cá nhân — Đặng Anh Tuyền — 03039

## 1. Ba ca khó trong bản guideline cá nhân

| Ảnh | Điểm guideline v0 còn thiếu | Đề xuất trong bản cá nhân |
|---|---|---|
| `img_002` | Xe tải đang chở ô tô thì có cần gán từng xe được chở hay không? | Vẽ một box cho xe tải và box riêng cho từng ô tô đủ điều kiện. |
| `img_004`, `img_035` | Tuk-tuk chưa có lớp riêng nên dễ bị xếp vào xe máy hoặc bỏ qua. | Ban đầu đề xuất để ngoài phạm vi và đưa ra họp nhóm để chốt. |
| `img_037` | Xe bán tải lắp nắp thùng cao dễ bị nhầm với van. | Đề xuất `truck` vì xe vẫn có cabin và thùng hàng riêng. |

## 2. Những điểm khác nhau khi thống nhất guideline

Trong bản cá nhân, tôi đề xuất để tuk-tuk và xe buýt ngoài phạm vi vì bộ nhãn ban đầu chưa có hai lớp này. Khi thống nhất guideline v1.0, nhóm bổ sung `tuk_tuk` và `bus`. Việc có lớp riêng giúp phân biệt các phương tiện này rõ hơn khi gán nhãn. Với xe bán tải có nắp thùng, nhóm chọn `truck` dựa trên kết cấu cabin và thùng hàng. Người lái và hành khách cũng được tách khỏi phương tiện thành từng box riêng.

## 3. Kết quả gán nhãn cá nhân

File `submission/labels/03039_DangAnhTuyen.json` mới nhất chứa 24 ảnh và 202 bbox, gồm 182 box đối tượng và 20 box `ignore_region`.

| Lớp | Số box |
|---|---:|
| `person` | 79 |
| `motorcycle` | 39 |
| `car` | 37 |
| `truck` | 4 |
| `bus` | 4 |
| `van` | 3 |
| `bicycle` | 3 |
| `tuk_tuk` | 2 |
| `traffic_light` | 6 |
| `traffic_sign` | 5 |
| `ignore_region` | 20 |
| **Tổng** | **202** |

## 4. Kết quả đối chiếu và calibration

Phần này chờ kết quả so sánh nhãn độc lập và buổi calibration của nhóm. Khi có danh sách ca bất đồng, tôi sẽ ghi lại trường hợp liên quan đến mình, nguyên nhân lệch và cách sửa đã thống nhất. Các số liệu ở phần 3 là thống kê file nhãn hiện tại, chưa phải kết quả IAA hay kết luận chất lượng.

## 5. Việc sử dụng AI

Tôi sử dụng ChatGPT/Codex để hỗ trợ diễn đạt và chỉnh sửa câu từ trong báo cáo.

## 6. Bài học cá nhân

Điểm tôi cần chú ý hơn khi viết guideline lần sau là chốt rõ phạm vi lớp ngay từ đầu và gắn mỗi quy tắc với một ảnh cụ thể. Việc bổ sung xe đạp, đèn tín hiệu và biển báo cho thấy mỗi lớp mới đều cần có định nghĩa, ví dụ và hướng dẫn xử lý ca dễ nhầm. Khi danh mục lớp thay đổi, nhóm cũng cần thống nhất cách rà lại các ảnh đã gán. Với mức che khuất, nên cùng thử trên một vài đối tượng trước khi gán độc lập để giảm khác biệt trong cách ước lượng.
