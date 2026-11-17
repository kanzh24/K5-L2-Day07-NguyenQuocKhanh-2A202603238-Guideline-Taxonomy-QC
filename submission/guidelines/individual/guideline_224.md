# Guideline gán nhãn — v1.2 (Bounding Box Only)

- **Phiên bản:** v1.2 (Tối ưu hóa cho Bounding Box dựa trên phân tích 40 ảnh thực tế)
- **Loại nhãn:** Bounding box (Hộp chữ nhật) trên ảnh giao thông hỗn hợp.
- **Công cụ:** CVAT
- **Định dạng xuất:** COCO 1.0 (Chỉ sử dụng trường `bbox` [x, y, width, height], tuyệt đối không dùng `segmentation`).

---

## 1. Mục tiêu

Nhãn được thiết kế để huấn luyện mô hình Object Detection (phát hiện đối tượng) nhằm giải quyết các bài toán:
- **Phát hiện phương tiện và người trong bối cảnh giao thông hỗn hợp (Mixed Traffic):** Đặc biệt chú trọng đến sự đan xen của các loại phương tiện tiêu chuẩn (ô tô, xe tải) và phi tiêu chuẩn (xe ba bánh, xe đạp, xe máy chở hàng cồng kềnh).
- **Hỗ trợ giám sát giao thông thông minh (ITMS) & Xe tự lái (ADAS):** Mô hình cần đủ độ "robust" (mạnh mẽ) để hoạt động trong điều kiện thực tế khắc nghiệt: ánh sáng thay đổi (đêm, hoàng hôn, ngược sáng), thời tiết xấu (mưa, đường ướt), và vật thể bị mờ do chuyển động.
- **Phân biệt vật thể thực và vật thể giả (Noise Reduction):** Mô hình phải học cách phân biệt chính xác người/phương tiện thật với các yếu tố gây nhiễu trong ảnh như: tượng đài, ma-nơ-canh, đồ chơi, hình ảnh phản chiếu qua gương/kính, hoặc bóng đổ.

---

## 2. Lớp và Thuộc tính

### 2.1. Lớp (Mỗi vật thuộc đúng một lớp)

Hệ thống lớp được chia thành 3 nhóm chính: **Người**, **Phương tiện**, và **Vùng nhiễu**. 

| Nhóm | Lớp | Định nghĩa chi tiết | Lưu ý từ thực tế ảnh (Edge Cases) |
| :--- | :--- | :--- | :--- |
| **Người** | `person` | Người (đi bộ, đứng, ngồi, đang điều khiển phương tiện). | Không phân biệt người đi bộ hay tài xế. Vẽ box bao trọn cơ thể người (kể cả tay chân đang vươn ra). |
| **Phương tiện cơ giới** | `car` | Ô tô con, SUV, xe bán tải (Pickup) **không chở hàng hóa** hoặc chở đồ nhỏ trong cabin. | Bao gồm cả xe tự chế hình thù kỳ dị (VD: xe hình con chuột) miễn là có 4 bánh và chạy trên đường. |
| | `truck` | Xe tải, xe đầu kéo, xe bán tải (Pickup) **có chở hàng hóa** ở thùng sau. | Box phải bao trùm cả phần hàng hóa cồng kềnh (VD: bao tải chất cao, thùng carton). |
| | `van` | Xe thùng kín cỡ nhỏ, xe khách loại nhỏ (Minivan, xe 16 chỗ). | Thường có hình hộp chữ nhật, kín mít từ đầu đến đuôi. |
| | `bus` | Xe buýt, xe khách cỡ lớn (Coach). | Kích thước rất lớn, hình dáng đặc trưng. Không gộp vào `truck` hay `van`. |
| | `motorcycle` | Xe máy, mô tô, xe gắn máy, xe máy có gắn thùng (sidecar). | Nếu xe máy kéo theo thùng chở hàng, vẽ 1 box bao trùm cả cụm. |
| **Phương tiện thô sơ/đặc thù** | `three_wheeler` | Xe ba bánh (Tuk-tuk, xe lam, xe ba gác máy, xích lô máy). | Rất phổ biến trong ảnh. Giúp mô hình phân biệt với xe 4 bánh. |
| | `bicycle` | Xe đạp (thường, đạp điện), xích lô đạp (Cyclo). | Bao gồm cả xe đạp chở hàng rong. |
| **Vùng nhiễu** | `ignore_region` | Vùng không cần gán từng vật, dùng để "vô hiệu hóa" các vật thể gây nhiễu mô hình. | Dùng cho: Tượng đài, bia mộ hình xe, ma-nơ-canh, hoặc đám đông xe máy ken đặc không thể tách box. |

### 2.2. Thuộc tính (Attributes)

| Thuộc tính | Giá trị | Ý nghĩa |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Mức bị che khuất của vật thể trong box: 0: Thấy rõ (>80%); 1: Che ít (50-80%); 2: Che nhiều (20-50%); 3: Gần như không thấy (<20%). |
| `truncated` | `no`, `yes` | Vật bị cắt ở mép ảnh. |
| `can_review` | tick / không | Tick khi bạn không chắc chắn về lớp hoặc box, để người khác xem lại. |

---

## 3. Quy tắc vẽ Box (Bắt buộc tuân thủ)

1.  **Hình dạng:** Box **phải là hình chữ nhật** (Axis-aligned) ôm sát vật thể nhất có thể. Tuyệt đối không vẽ box xéo, không vẽ polygon.
2.  **Một vật - Một box:** 
    - Với người ngồi trên xe (xe máy, ô tô): Vẽ **02 box riêng biệt** (1 box cho `person`, 1 box cho phương tiện). Box của người có thể nằm đè lên box của xe.
    - Với xe máy kéo theo thùng (sidecar): Vẽ **01 box duy nhất** bao trùm cả xe và thùng.
3.  **Hàng hóa cồng kềnh:** 
    - Nếu hàng hóa gắn liền với xe (VD: Bao tải chất cao trên nóc tuk-tuk, thùng hàng trên xe bán tải), vẽ **01 box bao trùm cả xe và hàng hóa**.
    - Nếu vật thể dài (VD: Ống nước dài do 2 xe cùng khiêng), chỉ vẽ box cho từng người/xe, **không vẽ box cho ống nước**.
4.  **Ngưỡng hiển thị:** Chỉ vẽ box nếu phần nhìn thấy của vật thể chiếm **tối thiểu 20-30%**. Nếu bị che khuất nhiều hơn, bỏ qua.
5.  **Vật thể mờ (Motion blur):** Nếu vật thể bị nhòe do di chuyển nhưng vẫn nhận diện được hình dáng, vẫn vẽ box bình thường.
6.  **Không vẽ vật phản chiếu:** Tuyệt đối không vẽ box cho vật thể chỉ xuất hiện trong gương chiếu hậu, cửa kính ô tô hay vũng nước.

---

## 4. Ca khó & Hướng dẫn xử lý (Edge Cases)

1.  **Tượng đài, Bia mộ, Ma-nơ-canh, Đồ chơi:** Không gán nhãn `person` hay `car`. Nếu chúng chiếm diện tích lớn và gây nhiễu, vẽ một box `ignore_region` bao quanh.
2.  **Xe độ/chế (Custom vehicles):** Xe tự chế hình con chuột -> gán `car`. Xe ba bánh tự chế -> gán `three_wheeler`. Hộp bao quanh hình dáng tổng thể của xe.
3.  **Kẹt xe / Đám đông dày đặc:** Vẽ từng box cho các xe/người rõ ràng nhất. Với cụm xe máy ken đặc không thể phân biệt từng chiếc, vẽ một box `ignore_region` bao trùm khu vực đó.
4.  **Xe bị cắt mép ảnh:** Vẫn vẽ box như bình thường và đánh dấu `truncated: yes`. Box sẽ bị cắt cụt theo mép ảnh.
5.  **Động vật (VD: Chim bồ câu):** Không thuộc phạm vi bài toán, bỏ qua.

---

## 5. QC và Phiên bản

- Mọi thay đổi làm đổi nhãn đều tăng phiên bản guideline.
- Ca mơ hồ: tick `can_review` và ghi chú ngắn gọn lý do.
- **Quy tắc "Vàng":** *Nếu bạn phải mất hơn 5 giây để phân vân không biết vật đó là xe tải hay xe van, hãy chọn lớp gần đúng nhất và tick `can_review`.*