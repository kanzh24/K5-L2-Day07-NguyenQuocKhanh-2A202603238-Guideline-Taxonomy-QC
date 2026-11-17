# Guideline gán nhãn người và phương tiện trên ảnh đường phố

**Day 07 — Guideline, Taxonomy & QC · Bản dự thảo đầy đủ v1.0-draft**

- **Ngày soạn:** 16/11/2026.
- **Phạm vi:** 40 ảnh gốc `data/images/img_001.jpg`–`img_040.jpg`.
- **Công cụ và định dạng:** CVAT; bounding box chữ nhật song song trục; xuất COCO 1.0.
- **Người viết / người duyệt:** chưa điền mã thành viên; chưa có xác nhận của nhóm.
- **Nguồn soạn:** xem toàn bộ 40 ảnh, xem lại ảnh gốc cho các ca 011, 019, 024, 028; đối chiếu template, taxonomy, hướng dẫn QC và script của repo.
- **Trạng thái:** dự thảo có AI hỗ trợ theo yêu cầu người dùng. Các quyết định dưới đây là đề xuất để người học kiểm lại và nhóm chốt; không phải bằng chứng đã gán nhãn độc lập, họp calibration hoặc duyệt QC. Các ca cần xác minh được nêu riêng tại mục 4.3.

## Mục lục

1. [Mục tiêu và phạm vi](#1-mục-tiêu-và-phạm-vi)
2. [Taxonomy và thuộc tính](#2-taxonomy-và-thuộc-tính)
3. [Quy tắc vẽ box và xử lý ca biên](#3-quy-tắc-vẽ-box-và-xử-lý-ca-biên)
4. [Áp dụng trên bộ 40 ảnh](#4-áp-dụng-trên-bộ-40-ảnh)
5. [QC, calibration và quản lý phiên bản](#5-qc-calibration-và-quản-lý-phiên-bản)

## Cách dùng nhanh

Với mỗi ứng viên trong ảnh, thực hiện theo thứ tự:

1. Xác định có phải **vật thật được nhìn trực tiếp** hay hình in, phản chiếu, tượng, đồ chơi hoặc một phần xe gắn camera (R-01, R-02).
2. Xác định có thuộc một trong năm lớp đối tượng hay không (R-03–R-07). Vật rõ ràng ngoài taxonomy được loại theo R-08.
3. Tách đúng từng cá thể, đặc biệt người và phương tiện (R-09).
4. Xác định box ôm phần nhìn thấy; đo chiều cao trên ảnh gốc (R-10–R-12).
5. Nếu đủ điều kiện, gán lớp và cả ba thuộc tính. Nếu quá nhỏ hoặc không tách được, áp dụng R-13–R-15.
6. Nếu chưa chắc, dùng `can_review` hoặc ghi vấn đề cấp ảnh theo R-16. Không tự sửa quy tắc giữa lượt gán độc lập.

**Quy tắc ưu tiên:** quyết định phạm vi trước → phân biệt cá thể → xác định phần nhìn thấy → kiểm kích thước → gán thuộc tính. Vật đã loại vì là tượng, hình in hoặc ngoài taxonomy không trở thành `ignore_region` chỉ vì khó phân lớp.

## 1. Mục tiêu và phạm vi

### 1.1. Mục tiêu

Tạo nhãn nhất quán để huấn luyện và đánh giá mô hình phát hiện người và một số phương tiện trong ảnh 2D. Mỗi vật hợp lệ có một lớp và một box. Nhãn cần tái lập được giữa các thành viên, có lý do cho ca biên và truy ngược được quyết định QC.

Bỏ sót người làm giảm khả năng phát hiện; gộp người với xe làm sai đơn vị đối tượng; nhầm tượng hoặc phản chiếu thành vật thật tạo dương tính giả. Lab này đánh giá quy trình gán nhãn, không xác nhận mô hình đủ điều kiện dùng trong hệ thống giao thông thực tế.

### 1.2. R-01 — Vật thật và bối cảnh

Gán người và phương tiện thuộc taxonomy xuất hiện trực tiếp trong ảnh, dù đang chạy, đỗ, được vận chuyển, trưng bày trong nhà hoặc đứng trên vỉa hè. Không giới hạn vào lòng đường và không suy đoán trạng thái chuyển động từ một ảnh tĩnh.

Loại hình in, ảnh quảng cáo, hình trên màn hình, tượng, ma-nơ-canh và đồ chơi. Không cần đặt vùng ignore quanh các vật đã xác định chắc chắn bị loại; đây là nền âm có chủ đích.

Người mặc trang phục hóa trang vẫn là `person` nếu có đủ căn cứ là người thật. Xe có hình dáng khác thường vẫn có thể là phương tiện thật. Không dùng riêng vẻ ngoài “giống hoạt hình” để loại. Nếu không phân biệt được người hóa trang với tượng hoặc xe thật với mô hình, áp dụng R-16.

### 1.3. R-02 — Phản chiếu, kính và xe chủ

- Không gán bản sao trong gương, mặt kính hoặc mặt đường ướt; không gán bóng đổ.
- Người hoặc xe **nhìn xuyên qua kính** vẫn được gán nếu nhận diện và đặt box được. Kính trong suốt không tự động tạo che khuất; phần bị khung kính, vách xe hoặc vật khác che mới tính che khuất.
- Loại phần xe gắn camera nhìn thấy ở mép ảnh, gồm gương, cửa, nắp capo; loại cả hình phản chiếu của xe đó. Chỉ áp dụng khi có căn cứ rõ là xe chủ, như `img_020`.
- Không coi mọi xe ở sát camera là xe chủ. Pickup ở `img_012` vẫn là đối tượng theo đề xuất của bản này: ảnh không đủ căn cứ để loại nó như một phần hệ camera.

## 2. Taxonomy và thuộc tính

### 2.1. Nguyên tắc chung

Giữ sáu tên lớp của starter: `car`, `truck`, `van`, `motorcycle`, `person`, `ignore_region`. Năm lớp đầu là đối tượng; `ignore_region` là vùng loại khỏi đánh giá cá thể. Không tự thêm `bus`, `bicycle`, `rider`, `tuk_tuk` hay `other` trong lượt gán.

Định nghĩa chi tiết dưới đây cần được nhóm đồng bộ vào trường `definition` của `submission/taxonomy.json` khi chốt v1.0. **Giữ nguyên toàn bộ `rules`.** File taxonomy hiện có vẫn là v0; tài liệu dự thảo này chưa tự làm thay đổi phiên bản chính thức.

### 2.2. Định nghĩa lớp và ranh giới

### Bảng tra cứu lớp nhãn

| Lớp | Dấu hiệu nhận diện chính | Ví dụ trong bộ ảnh | Dễ nhầm / cần loại |
|---|---|---|---|
| `car` | Thân ô tô con: sedan, hatchback, wagon, SUV, crossover, MPV | `img_023`, `img_024`, `img_028` | Pickup thuộc `truck`; xe mô hình bị loại |
| `truck` | Cabin và thùng/khu vực hàng riêng; gồm pickup có nắp thùng | `img_002`, `img_036`, `img_037` | Thùng kín không tự biến thành `van`; tượng `img_027` bị loại |
| `van` | Thân hộp liền khối, mui cao, khoang sau kiểu van | `img_005`, `img_016` | Xe tải thùng riêng và xe buýt |
| `motorcycle` | Xe máy, mô tô, scooter; gồm sidecar gắn liền | `img_010`, `img_022`, `img_035` | Tuk-tuk, xe đạp, xích lô nằm ngoài phạm vi |
| `person` | Người thật, gồm người lái và hành khách; mỗi người một box | `img_010`, `img_026`, `img_034` | Tượng, ma-nơ-canh, hình in, phản chiếu |
| `ignore_region` | Vùng có vật mục tiêu quá nhỏ hoặc không tách được cá thể | Xét từng vùng ở `img_021`, `img_030`, `img_032` | Không dùng thay lớp “khác” hoặc để phủ cả đám đông |

Tra bảng để chọn nhanh; áp dụng đầy đủ các ranh giới và ngoại lệ bên dưới.

**R-03 — `car`.** Ô tô con có thân xe dành chủ yếu cho khoang hành khách: sedan, hatchback, wagon, SUV, crossover và MPV kiểu ô tô con. Không dựa riêng vào kích thước hoặc số cửa. Ví dụ: ô tô đen ở `img_023`, wagon đen ở `img_024`, SUV đỏ phía sau ở `img_028`. Phản ví dụ: pickup có cabin và thùng riêng thuộc `truck`; minibus kiểu van thuộc `van`; xe buýt nằm ngoài taxonomy.

**R-04 — `truck`.** Xe có cabin và khu vực/thùng chở hàng riêng biệt về kết cấu, gồm pickup, xe tải nhẹ, xe thùng, xe chở xe. Pickup vẫn thuộc `truck` khi lắp nắp thùng hoặc khoang thiết bị kín; thùng kín không đủ để chuyển thành `van`. Ví dụ: `img_001`, `img_002`, `img_007`, `img_012`, `img_028`, `img_036`, `img_037`. Phản ví dụ: tượng pickup ở `img_027` bị loại theo R-01. Một tổ hợp đầu xe–rơ-moóc đang gắn liền được coi là một `truck`; rơ-moóc tách rời không có lớp riêng trong phạm vi này.

**R-05 — `van`.** Xe thân hộp liền khối, mui cao, khoang sau nối với phần cabin trong cùng kiểu thân van; có thể chở người hoặc hàng, có hoặc không có cửa kính bên. Minibus dùng thân van thuộc lớp này. Ví dụ: xe bạc ở `img_005` và xe van trắng phía sau ở `img_016`. Không suy ra loại xe chỉ từ cửa kính. Xe buýt thân dài chuyên chở khách ở `img_009` và `img_037` không gán thành `van`; xe tải thùng riêng ở `img_036` là `truck`.

**R-06 — `motorcycle`.** Xe máy, mô tô, scooter có động cơ và thân xe hai bánh đặc trưng. Mô tô gắn sidecar là **một** đối tượng `motorcycle`, box bao cả xe và sidecar cố định (`img_022`). Xe đạp, xích lô đạp chân, xe đẩy, tuk-tuk/auto-rickshaw có thân ba bánh độc lập nằm ngoài lớp này. Ví dụ: scooter ở `img_010`, mô tô giữa ảnh `img_035`. Không dựa riêng vào số bánh để nhầm sidecar với tuk-tuk.

**R-07 — `person`.** Một người thật, ở mọi tư thế, tuổi và vai trò: đi bộ, đứng, ngồi, lái xe hoặc hành khách. Mỗi người một box, kể cả khi chỉ thấy đầu và thân trên nhưng vẫn đủ điều kiện kích thước và định vị. Ví dụ: người bán hàng `img_010`, người băng đường `img_034`, hai người trên xe `img_026`. Không gán ma-nơ-canh `img_033`, tượng người `img_017`, chân dung trên billboard `img_018`. Nhìn thấy mũ hoặc quần áo rời không đủ để suy ra một người.

**R-08 — Ngoài taxonomy.** Xe buýt, xe đạp, xích lô, tuk-tuk, xe đẩy, động vật và hàng hóa không có nhãn đối tượng trong phiên bản này. Loại trực tiếp khi nhận diện chắc chắn; vẫn gán người thật đi cùng và các vật hợp lệ khác. Không ép tuk-tuk vào `motorcycle` hoặc xe buýt vào `van`, không dùng `ignore_region` làm lớp “khác”. Nếu nhóm muốn mở rộng taxonomy, phải chốt trước lượt gán hoặc tăng phiên bản và rà lại toàn bộ lô.

### 2.3. Thuộc tính bắt buộc cho năm lớp đối tượng

| Thuộc tính | Giá trị | Khi nào chọn / yêu cầu |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Chọn theo mức che trong bảng dưới; mức `3` bắt buộc bật review |
| `truncated` | `no`, `yes` | `yes` khi mép ảnh cắt mất một phần vật; `no` khi vật nằm trọn |
| `can_review` | `false`, `true` | `true` khi còn nghi vấn; kèm ghi chú để reviewer tìm lại ca |

**`occluded_level`: chuỗi `"0"`, `"1"`, `"2"`, `"3"`.**

Ước lượng tỉ lệ hình chiếu của vật **bị vật khác che trong khung ảnh**, so với hình chiếu dự kiến của chính vật đó ở cùng tư thế và góc nhìn. Đây là ước lượng phục vụ thuộc tính, không phải yêu cầu vẽ box phần khuất. Không tính phần ngoài ảnh; không tính mặt sau vốn không thể thấy do góc nhìn; không coi các khoảng trống giữa tay/chân hoặc khung xe là phần bị che.

| Mức | Tỉ lệ bị vật khác che trong khung ảnh | Cách áp dụng |
|---|---|---|
| `0` | 0% | Không nhận thấy che khuất bởi vật khác |
| `1` | Trên 0% đến hết 30% | Đúng 30% vẫn chọn mức `1` |
| `2` | Trên 30% | Có đủ căn cứ ước lượng mức che |
| `3` | Không đủ căn cứ ước lượng | Vẫn nhận diện và định vị được vật; bắt buộc bật `can_review`, ghi nguyên nhân; không hiểu là “che hơn mức 2” |

Đúng 30% thuộc mức 1. Nếu ở sát ngưỡng mà không quyết được, chọn mức dự kiến 1 hoặc 2, bật `can_review` và ghi tranh chấp ngưỡng. Người ngồi sau che người lái hoặc người che xe đều là che khuất; xe che chân người cũng vậy. Tối và nhòe tự chúng không phải che khuất: xử lý theo R-15.

**`truncated`: chuỗi `"no"` hoặc `"yes"`.**

`yes` nếu khung ảnh cắt mất một phần của chính đối tượng; `no` nếu vật nằm trọn trong ảnh. Vật chạm mép nhưng đường biên vẫn trọn không tự động là `yes`. Có thể đồng thời `truncated=yes` và `occluded_level=1/2/3`. Người bị cắt chân ở mép ảnh khác người bị xe che chân ở bên trong ảnh. Khi không chắc đường biên, bật `can_review`.

**`can_review`: checkbox, boolean `true`/`false` khi biểu diễn trong COCO.**

`true` nếu còn nghi vấn về lớp, tính thật/giả, số cá thể, đường biên hoặc thuộc tính. Đây là cờ yêu cầu xem lại, không phải mức chất lượng hoặc nhãn “khác”. `false` khi đã có quyết định theo quy tắc. Mỗi cờ phải đi kèm ghi chú đủ tìm lại vùng và lý do. Nhãn cuối chỉ được bỏ cờ sau khi có quyết định và kiểm lại.

**Thuộc tính của `ignore_region`.** Nếu CVAT yêu cầu, dùng `occluded_level=0`, `truncated=no`, `can_review=false` làm giá trị kỹ thuật mặc định; chúng không mang nghĩa vật lý cho vùng. Nếu vùng ignore còn tranh chấp, ghi vấn đề cấp ảnh theo R-16 để không phụ thuộc vào cách script xử lý thuộc tính vùng ignore.

## 3. Quy tắc vẽ box và xử lý ca biên

### 3.1. R-09 — Một cá thể, một box

Mỗi người và mỗi phương tiện hợp lệ được gán riêng. Một xe máy chở hai người tạo ba box nếu cả ba đều đủ điều kiện. Người đi xe đạp/xích lô vẫn là `person` dù phương tiện bị loại. Một xe tải chở nhiều ô tô có một box `truck` và một box cho từng xe được chở đủ điều kiện; việc nằm trên xe tải không biến ô tô thành nền.

Không gộp nhiều người, không gộp người với xe và không tách đầu/thân/chân thành nhiều box. Hai box có thể chồng lấn do hình học phối cảnh. Một vật chỉ có một lớp, không gán đồng thời `car` và `truck`.

### 3.2. R-10 — Box phần nhìn thấy

Vẽ hình chữ nhật nhỏ nhất bao toàn bộ các phần nhìn thấy **thuộc cùng cá thể**. Cạnh box đặt sát cực trị trái, trên, phải, dưới; không thêm đệm chủ ý và không kéo tới vị trí suy đoán của bộ phận bị che.

Các phần nhìn thấy bị ngắt bởi vật che vẫn dùng một box; khoảng trống và vật che có thể nằm bên trong box. Không vẽ nhiều box cho các mảnh của một vật, cũng không gán box cho vật hoàn toàn không nhìn thấy.

- Với `person`: bao tóc, mũ/mũ bảo hiểm đang đội, quần áo đang mặc, giày và mọi chi nhìn thấy. Không mở rộng box vì túi xách, gánh hàng, ô hoặc vật đang cầm; bộ phận cơ thể nhìn thấy bên cạnh chúng vẫn được tính.
- Với xe: bao bánh, gương, tay lái, cản, thân xe, cửa/nắp mở và bộ phận gắn cố định. Không mở rộng vì người ngồi, bóng đổ, hàng rời, túi treo hoặc hàng dài nhô ra. Box xe `img_029` không chạy theo thanh dài; box pickup `img_028` không mở rộng theo tượng ngựa.
- Với xe tải chở xe: bao kết cấu sàn/giá chở gắn với xe tải; các ô tô chở bên trên có box riêng. Với sidecar, bao cả sidecar.

Nếu một phụ kiện không phân biệt được là kết cấu cố định hay hàng rời và làm đổi đáng kể box, bật `can_review`, ghi rõ chi tiết đó.

### 3.3. R-11 — Che khuất và mất liên tục

Tìm các mảnh nhìn thấy của cùng một vật dựa trên kết cấu và vị trí. Chỉ nối chúng trong một box khi đủ căn cứ thuộc cùng cá thể. Không “hoàn thiện” bánh, chân hoặc đuôi xe đã bị che.

Qua hàng rào (`img_018`) hoặc người đứng chen nhau (`img_019`, `img_032`), vẫn gán từng cá thể nếu phân biệt được. Khi chỉ thấy đầu người đủ cao, box ôm phần đầu thực sự thấy; không dựng thân người giả định phía dưới. Mức che có thể là 2 dù box rất nhỏ so với hình chiếu toàn thân.

### 3.4. R-12 — Cắt mép và tọa độ

Box phải nằm trong ảnh gốc: `0 ≤ x_min < x_max ≤ W`, `0 ≤ y_min < y_max ≤ H`. Phần bị mép ảnh cắt dừng đúng tại biên; không mở canvas và không suy ra phần ngoài khung.

COCO dùng `bbox=[x_min,y_min,width,height]`, với `width=x_max-x_min` và `height=y_max-y_min`; không dùng `[x_min,y_min,x_max,y_max]` cho trường này. Không đổi kích thước hoặc tên ảnh. Mọi ngưỡng pixel áp dụng trên ảnh gốc, không phải ảnh thu nhỏ hoặc mức zoom của CVAT.

### 3.5. R-13 — Vật nhỏ

Dùng chiều cao **box phần nhìn thấy** đã xác định theo R-10:

- Cao **từ 20 px trở lên**: gán đối tượng nếu biết lớp và cá thể.
- Cao **dưới 20 px**: không gán box đối tượng; tạo `ignore_region` sát vùng vật nhỏ.
- Đúng 20 px vẫn gán; không phóng box để đạt ngưỡng. Không dùng chiều cao toàn thân suy đoán cho người bị che.

Với vật nhỏ tách rời, mỗi vùng ignore ôm một vật; với cụm liền nhau không tách được, dùng vùng nhỏ nhất bao cụm. Không đặt ngưỡng chiều rộng bổ sung. Một chấm sáng hoặc mảng tối không đủ căn cứ là đối tượng mục tiêu thì không tự biến thành vùng ignore; ghi vấn đề nếu cần xác minh.

### 3.6. R-14 — Đám đông và vùng ignore

Đám đông không tự động được bỏ qua. Gán từng người/xe nhận diện được, kể cả phía sau và bị che nhiều. Chỉ dùng `ignore_region` cho phần có đối tượng mục tiêu nhưng không thể tách cá thể hoặc xác định box ổn định, hoặc phần vật nhỏ theo R-13.

1. Gán các cá thể phân biệt được trước.
2. Đánh dấu phần còn lại bằng những hình chữ nhật gọn, tách theo các cụm thực sự liền nhau.
3. Không dùng một vùng bao toàn ảnh hay toàn đám đông nếu còn các cá thể gán được.
4. Kiểm tra vùng ignore không che mất các đối tượng hợp lệ. Nếu vùng cần bỏ qua xen kẽ với người/xe rõ, chia vùng; nếu vẫn không biểu diễn được, ghi vấn đề và chuyển reviewer.

Script có phép kiểm một box được phủ khi **ít nhất 50% diện tích box nằm trong một vùng ignore**. Đây là cơ chế so sánh, không phải quyền phủ rộng vùng ignore để bỏ lỗi. Vùng ignore do các thành viên đặt khác nhau cũng là ca calibration.

### 3.7. R-15 — Nhòe, thiếu sáng và trường hợp bất định

Nhòe hoặc thiếu sáng không đủ lý do loại cả ảnh. Với vật vẫn biết lớp và xác định được cực trị, gán box theo lõi hình dạng nhìn thấy, không kéo theo vệt sáng/bóng nhòe không xác định là thân vật. Nếu ranh giới còn nghi vấn, bật `can_review`.

Nếu xác định có đối tượng mục tiêu nhưng không thể đặt box hoặc tách cá thể ổn định, đặt vùng ignore gọn theo R-14 và ghi vấn đề cấp ảnh. Nếu ngay cả bản chất có phải đối tượng mục tiêu cũng chưa rõ, ghi vấn đề trước; không tạo lớp đối tượng hoặc vùng ignore chỉ từ suy đoán. Ảnh có ca chưa giải quyết chưa đủ điều kiện bàn giao Release.

### 3.8. R-16 — Cách ghi ca cần review

Nếu có thể đặt box dự kiến: chọn lớp/thuộc tính có căn cứ nhất, bật `can_review` và ghi lý do. Nếu chưa đặt được box: ghi vào nhật ký riêng để thu vào `submission/qc/image_issues.csv` sau khi khóa nhãn độc lập. Các cột là `image,annotator,region,note`.

`image` dùng tên đầy đủ như `img_011.jpg`; `annotator` dùng mã ba chữ số; `region` mô tả vị trí và có thể kèm tọa độ pixel; `note` ghi điều chưa rõ, quy tắc liên quan và lựa chọn cần chốt. Không ghi chung chung “ảnh khó”. Ví dụ nội dung ghi chú: “Nhân vật lớn bên phải: chưa phân biệt người hóa trang hay mô hình; cần chốt R-01 trước khi quyết định person.”

Loại chắc chắn theo quy tắc không cần bật review. Ca chưa chắc phải giữ dấu vết; không dùng loại bỏ hoặc ignore để né quyết định. Khi gán độc lập, không xem nhãn người khác để giải quyết nghi vấn.

## 4. Áp dụng trên bộ 40 ảnh

### 4.1. Các ví dụ nền tảng

Các mục dưới đây mô tả ca nổi bật sau khi xem ảnh, **không phải danh sách đầy đủ mọi box**, số đếm ground truth hoặc kết quả đã được nhóm duyệt. Chiều cao và mức che từng vật phải đo/ước lượng khi gán ở ảnh gốc.

#### Ảnh 001–010

| Ảnh | Tình huống, quyết định đề xuất và quy tắc |
|---|---|
| [img_001](../../../data/images/img_001.jpg) | pickup là `truck`; gán riêng người phía trước và người trong quán đủ điều kiện. Xe bị cắt mép phải đặt `truncated=yes`; bàn ghế che người thuộc che khuất. Áp dụng R-04, R-07, R-11, R-12. |
| [img_002](../../../data/images/img_002.jpg) | một tổ hợp xe vận chuyển là `truck`; mỗi xe trên giá chở có box riêng theo kiểu thân xe, không mặc định tất cả là `car`. Khung giá chở che xe là occlusion. R-03, R-04, R-09, R-10. |
| [img_003](../../../data/images/img_003.jpg) | tách người bán hàng, người đi xe và xe máy; không gộp gánh hàng vào person; xe đạp nằm ngoài taxonomy. R-07–R-10. |
| [img_004](../../../data/images/img_004.jpg) | hai tuk-tuk bị loại theo phạm vi; người ngồi trong xe vẫn được gán khi đủ điều kiện. Không gán cả tuk-tuk thành `motorcycle` hoặc dùng ignore cho toàn xe. R-06–R-08. |
| [img_005](../../../data/images/img_005.jpg) | xe thân van bạc phía sau là `van`; tách từng người và xe máy, kể cả người chỉ thấy một phần. Mũ bảo hiểm đang đội thuộc box người. R-05, R-09–R-12. |
| [img_006](../../../data/images/img_006.jpg) | người đi xe đạp vẫn là `person`; xe đạp bị loại. Người và xe máy có box riêng; không gộp hai người ngồi cùng xe. R-08, R-09. |
| [img_007](../../../data/images/img_007.jpg) | pickup có kết cấu thùng cao và xe tải bên phải đều là `truck`; độ cao thùng không làm thành `van`. Không gán phản chiếu trên nền ướt. R-02, R-04. |
| [img_008](../../../data/images/img_008.jpg) | xe máy đang đỗ vẫn được gán; tách người đứng/ngồi với từng xe. Chỉ vùng xa không tách được mới xét ignore, không phủ cả bãi xe. R-01, R-09, R-14. |
| [img_009](../../../data/images/img_009.jpg) | xe buýt giữa ảnh nằm ngoài taxonomy; người và xe máy xung quanh vẫn xét riêng. Người sau kính chỉ gán nếu thấy thật, định vị được và đạt ngưỡng. R-02, R-08, R-13. |
| [img_010](../../../data/images/img_010.jpg) | người bán hàng, người lái và người ngồi sau là các cá thể riêng; hàng rau quả và xe đẩy không thuộc lớp mục tiêu. Box xe không mở rộng theo hàng hóa. R-07–R-10. |

#### Ảnh 011–020

| Ảnh | Tình huống, quyết định đề xuất và quy tắc |
|---|---|
| [img_011](../../../data/images/img_011.jpg) | gán người thật ở nền và xe thật nhận diện được. Nhân vật Predator lớn và các xe trang trí cần rà kỹ tính người thật/xe thật so với mô hình; ghi ca chưa chắc theo mục 4.3, không loại chỉ vì hình dáng lạ. R-01, R-16. |
| [img_012](../../../data/images/img_012.jpg) | pickup tiền cảnh là `truck`, SUV bên trái là `car`; không mở rộng box pickup theo các kiện hàng. Không tự loại pickup thành ego chỉ vì gần camera. R-02–R-04, R-10. |
| [img_013](../../../data/images/img_013.jpg) | gán người và xe thật trong cảnh đêm; bỏ phản chiếu người/đèn/xe trên đường ướt. Đo vật xa trước khi áp dụng ignore. R-02, R-13, R-15. |
| [img_014](../../../data/images/img_014.jpg) | hai người đi bộ rõ được gán riêng. Dải xe chuyển động nhòe phía sau áp dụng R-15 theo từng vùng; không suy ra số xe từ số vệt nhòe. |
| [img_015](../../../data/images/img_015.jpg) | xe ở mép trái/phải vẫn là đối tượng nếu đủ điều kiện và có `truncated=yes`. Các xe nhòe ở giữa cần đánh giá đường biên riêng; phản chiếu trên đường bị loại. R-02, R-12, R-15. |
| [img_016](../../../data/images/img_016.jpg) | scooter giữa ảnh là `motorcycle`, người lái là `person`; van trắng phía sau là `van`. Tuk-tuk hai bên bị loại nhưng người trong đó vẫn xét riêng. R-05–R-09. |
| [img_017](../../../data/images/img_017.jpg) | tượng người cưỡi ngựa không có nhãn `person`; ngựa cũng ngoài taxonomy. Không đặt ignore quanh tượng. R-01, R-08. |
| [img_018](../../../data/images/img_018.jpg) | chân dung trên billboard bị loại; xe thật sau hàng rào vẫn được gán nếu đủ điều kiện, với mức che phù hợp. Không nhầm nhìn qua khe rào thành phản chiếu. R-01, R-03, R-11. |
| [img_019](../../../data/images/img_019.jpg) | đề xuất gán xe diễu hành vàng là `car` dựa trên thân xe, bánh và khoang người lái nhìn thấy; người lái và người xem gán riêng. Nhân vật chuột cần xác minh người mặc trang phục hay mô hình trước khi chốt. R-01, R-03, R-07, R-14, R-16. |
| [img_020](../../../data/images/img_020.jpg) | gương/cửa xe chủ và hình trong gương bị loại. Không tạo nhãn xe từ hình phản chiếu. Nếu không còn vật trực tiếp hợp lệ, ảnh vẫn phải có trong COCO với không annotation. R-02. |

#### Ảnh 021–030

| Ảnh | Tình huống, quyết định đề xuất và quy tắc |
|---|---|
| [img_021](../../../data/images/img_021.jpg) | đám đông xe máy phải tách người và xe có thể nhận diện; vùng xa dưới 20 px hoặc không tách được mới dùng ignore. Không phủ cả nút giao. R-09, R-13, R-14. |
| [img_022](../../../data/images/img_022.jpg) | mô tô và sidecar gắn liền là một `motorcycle`; bao toàn bộ phần nhìn thấy của tổ hợp. Không tạo lớp xe ba bánh mới. R-06, R-10. |
| [img_023](../../../data/images/img_023.jpg) | người đạp xích lô là `person`; xích lô bị loại. Ô tô đen bên trái là `car`; xe máy ở phía xa xét riêng. R-03, R-07–R-09. |
| [img_024](../../../data/images/img_024.jpg) | wagon đen và xe thật xung quanh thuộc `car` dù ở nơi trưng bày. Vật hình xe vàng trên nóc nhỏ và khác cấu trúc xe bên dưới: chưa đủ căn cứ khẳng định xe thật hay mô hình, cần quyết định riêng; không gộp vào box xe đen. R-01, R-09, R-10, R-16. |
| [img_025](../../../data/images/img_025.jpg) | người đi bộ gần và xa đều xét theo chiều cao phần thấy; xe bên phải vẫn cần gán. Không dùng một vùng ignore bao toàn đường dạo vì có nhiều người nhỏ. R-01, R-13, R-14. |
| [img_026](../../../data/images/img_026.jpg) | hai người trên một xe tạo hai `person` và một `motorcycle` nếu đủ điều kiện; đánh giá mép dưới cho xe và người riêng. Không gộp người ngồi sau với người lái. R-09, R-12. |
| [img_027](../../../data/images/img_027.jpg) | pickup là tượng/đài tưởng niệm trên bệ, bị loại khỏi `truck`; không tạo ignore để thay thế. Căn cứ bổ sung có trong tên nguồn ở `data/ATTRIBUTION.csv`. R-01. |
| [img_028](../../../data/images/img_028.jpg) | pickup thật là `truck`; tượng ngựa trên thùng bị loại, box xe không kéo lên theo tượng. SUV đỏ phía sau là `car`; không gộp hai xe. R-03, R-04, R-09, R-10. |
| [img_029](../../../data/images/img_029.jpg) | gán xe máy và người riêng; thanh dài vận chuyển không mở rộng box xe/người. Xe đạp bị loại nhưng người đi xe đạp vẫn gán. R-08–R-10. |
| [img_030](../../../data/images/img_030.jpg) | xe máy đỗ sát nhau vẫn cần tách từng chiếc có căn cứ; không đếm mỗi gương thành một xe. Chỉ cụm không phân biệt cá thể mới dùng ignore. Người thật bên phải xét riêng; chân dung quảng cáo bị loại. R-01, R-11, R-14. |

#### Ảnh 031–040

| Ảnh | Tình huống, quyết định đề xuất và quy tắc |
|---|---|
| [img_031](../../../data/images/img_031.jpg) | gán xe/người thật trong đêm; loại nhân vật in trên biển hiệu và phản chiếu ánh đèn trên mặt đường. Mỗi vật thiếu sáng áp dụng R-15 riêng. |
| [img_032](../../../data/images/img_032.jpg) | người đứng tại trạm xe buýt còn tách được thì gán từng người; ô tô bên trái bị cắt mép vẫn gán. Không đồng nhất “đám đông” với ignore. R-07, R-12, R-14. |
| [img_033](../../../data/images/img_033.jpg) | ma-nơ-canh, hình quảng cáo và phản chiếu cửa kính không phải `person`. Chỉ gán người thật nếu quan sát trực tiếp có căn cứ; không tự sinh người từ dáng giống người. R-01, R-02. |
| [img_034](../../../data/images/img_034.jpg) | người băng đường là một `person`; xe phía sau và người đi xe có box riêng. Người/xe bị cắt ở mép phải dùng `truncated` cho từng đối tượng. R-09, R-11, R-12. |
| [img_035](../../../data/images/img_035.jpg) | auto-rickshaw, kể cả xe chở hàng cao, nằm ngoài taxonomy; không gán thành `truck` vì có hàng. Mô tô và người lái ở giữa cùng ô tô phía sau vẫn gán. R-04, R-06–R-09. |
| [img_036](../../../data/images/img_036.jpg) | xe xanh là `truck` nhờ cabin và thùng riêng, dù kích thước nhỏ. Mui bạt cố định trên kết cấu thùng thuộc box xe; xe máy phía sau vẫn xét nếu đủ điều kiện. R-04, R-10, R-11. |
| [img_037](../../../data/images/img_037.jpg) | pickup trắng có khoang thiết bị/nắp thùng là `truck`, không phải `van`. Xe buýt ở nền bị loại; người thật bên trái gán riêng. R-04, R-07, R-08. |
| [img_038](../../../data/images/img_038.jpg) | xe mô hình, búp bê và đồ chơi trong tủ bị loại; không tạo `car`, `truck`, `person` hoặc ignore chỉ từ các đồ chơi. R-01. |
| [img_039](../../../data/images/img_039.jpg) | xét người, xe máy chạy và xe đỗ ở hai bên; người đi xe đạp vẫn là person trong khi xe đạp bị loại. Vật xa đo chiều cao trên ảnh gốc. R-01, R-08, R-13. |
| [img_040](../../../data/images/img_040.jpg) | người đội mũ ở tiền cảnh là `person`; chỉ gán xe máy nếu các phần xe thấy được đủ xác định một cá thể và đạt ngưỡng. Taxi xanh và SUV phía sau là `car`; tuk-tuk và xe buýt bị loại. Hình phản chiếu trên mũ/gương không tạo box mới. R-02, R-03, R-08, R-11–R-13. |

### 4.2. Các cặp ảnh dùng để tự kiểm quy tắc

- **Pickup thật / tượng:** so `img_028` với `img_027`. Kiểu dáng giống nhau không đủ quyết định có gán.
- **Xe được chở / mô hình nghi vấn:** so `img_002` với vật trên nóc ở `img_024`. Gán từng xe thật; không suy ra mọi vật hình xe đều là xe thật.
- **Van / tải nhỏ / pickup có nắp:** so `img_005`, `img_036`, `img_037`. Dựa kết cấu thân, không chỉ độ cao mui hay thùng kín.
- **Sidecar / tuk-tuk:** so `img_022` với `img_004`. Sidecar gắn vào mô tô nằm trong phạm vi; tuk-tuk có thân riêng nằm ngoài.
- **Người thật / hình người:** so `img_034` với `img_017`, `img_018`, `img_033`.
- **Nhìn trực tiếp / phản chiếu:** so xe thật ở `img_015` với hình xe trong gương `img_020`.
- **Đám đông tách được / cụm không tách được:** dùng `img_019`, `img_021`, `img_030`, `img_032`; quyết định theo từng vùng, không theo nhãn chung của cả ảnh.

### 4.3. Các điểm phải được người học xác nhận trước khi khóa

1. **`img_011`:** tên nguồn mô tả nhân vật hóa trang và xe độ, nhưng ảnh tĩnh chưa đủ xác minh mọi chi tiết. Đề xuất: nhân vật là `person` nếu xác nhận có người mặc; xe trang trí là `motorcycle` nếu xác nhận là phương tiện thật. Nếu là tượng hoặc mô hình thì loại. Lưu căn cứ reviewer; không lấy tên file làm nhãn chuẩn tự động.
2. **`img_019`:** xác nhận đề xuất `car` cho xe diễu hành; xác minh riêng nhân vật chuột. Không suy ra “không phải người” chỉ vì đầu hoạt hình, cũng không mặc định có người bên trong.
3. **`img_024`:** xác minh vật hình xe vàng trên nóc. Đề xuất giữ ca review, không gộp vào xe dưới. Nếu là mô hình thì loại; nếu là xe thật còn cấu trúc nhận diện được thì gán `car` riêng.
4. **Chính sách nhóm:** xác nhận phạm vi loại tuk-tuk/bus/bicycle, pickup thuộc truck, ngưỡng che 30%, cách loại hàng rời và cách xử lý ego. Đây là đề xuất cụ thể của bản dự thảo, chưa phải quyết định đã được nhóm biểu quyết.

Các điểm trên có đường xử lý rõ nhưng chưa có bằng chứng người học chốt. Cần giải quyết trước khi coi tài liệu là `guideline_v1.0_agreed.md`; nếu phát hiện sau khi khóa, dùng quy trình calibration và phiên bản ở mục 5.

## 5. QC, calibration và quản lý phiên bản

### 5.1. Tự kiểm trước khi export

- Quét toàn ảnh theo hàng từ trái sang phải, trên xuống dưới; kiểm riêng nền xa, mép ảnh, người sau kính, người ngồi sau và xe đỗ.
- Kiểm từng box: đúng cá thể/lớp, bao đúng phần thấy, không kéo theo hàng hóa, không bỏ chi/gương/bánh nhìn thấy, không trùng box.
- Kiểm cao ≥20 px cho đối tượng; đo ở ảnh gốc. Kiểm vùng ignore không nuốt các cá thể rõ.
- Kiểm đủ `occluded_level`, `truncated`, `can_review`; ca review có ghi chú truy được.
- Xuất đủ các ảnh được giao, kể cả ảnh không có annotation. Dùng mã ba chữ số trong tên file.
- Chạy `python scripts/validate_labels.py submission/labels/<mã>.json`. Chỉ thay `<mã>` bằng mã thật. Script hợp lệ không chứng minh box đúng bằng mắt; ngưỡng vật nhỏ và nội dung hình ảnh vẫn cần người kiểm.

### 5.2. Ngưỡng cố định và ý nghĩa số đo

Giữ đúng các giá trị trong `submission/taxonomy.json`:

| Tham số / chỉ tiêu | Giá trị cố định | Cách đọc |
|---|---|---|
| `min_box_height_px` | 20 px | Chiều cao box phần nhìn thấy tối thiểu để gán đối tượng |
| `match_iou` | 0.5 | Ghép box một-một, không phân biệt lớp, để so sánh |
| `large_px` | 50 px | Dùng **trung bình chiều cao hai box** để phân nhóm |
| `iou_thr_large` | 0.85 | Ngưỡng so sánh khi trung bình chiều cao ≥50 px |
| `iou_thr_small` | 0.7 | Ngưỡng so sánh khi trung bình chiều cao <50 px |
| `iaa_iou` | 0.85 | IoU trung bình mục tiêu trên các cặp ghép được |
| `iaa_class` | 0.95 (95%) | Đồng thuận lớp mục tiêu trên các cặp ghép được |
| `error_rate_img_max` | 0.02 (2%) | Tỉ lệ ảnh có lỗi tối đa trong phạm vi duyệt |

IoU = diện tích giao / diện tích hợp. IAA chỉ tính trên các cặp box ghép được; không có cặp thì ghi N/A, không ghi 100%. Box không ghép được, bất đồng thuộc tính và vùng ignore phải xem riêng. Với ba người gán cùng ảnh có ba cặp người; không nhầm số cặp box với số ảnh hoặc số vật duy nhất. Phải đủ lượt phân công trước khi đọc IAA như kết quả của lô.

IAA cao không chứng minh cùng đúng: hai người có thể cùng gán tượng, cùng bỏ người hoặc cùng vẽ box sai. Sau khi khóa nhãn độc lập, đối chiếu thêm ca tham chiếu của coach và các cặp tự kiểm ở mục 4.2.

### 5.3. Calibration có dấu vết

Sau khi tập hợp nhãn độc lập, chạy `validate_labels.py --all`, `qc_compare.py` và `viz_disagreement.py --images flagged`. Giữ số đo IAA trước calibration. Dùng `build_final.py` tạo mẫu quyết định; mỗi ca có một dòng gồm `case_id`, ảnh, quyết định, mã quy tắc, phiên bản, người chốt, nơi sửa, kết quả kiểm lại và phạm vi `anh`/`lo`.

Phân biệt lỗi thực hiện với thiếu quy tắc. Nếu một người lệch khỏi quy tắc rõ, hướng dẫn lại và sửa nhãn liên quan. Nếu cả nhóm hiểu khác nhau hoặc cùng sai, làm rõ guideline và rà các ảnh cùng tình huống.

Dùng `prepare_calibration.py` để chuẩn bị đúng phạm vi sửa; người thật sửa box trong CVAT, export đúng đường dẫn mà script yêu cầu. Chạy lại `build_final.py` để tạo nhãn cuối và manifest. Không chép đè nhãn độc lập để làm đẹp IAA.

### 5.4. QC cuối và cách đếm lỗi

Duyệt toàn bộ 40 ảnh theo vòng QC, ưu tiên người kiểm khác người sửa. Tạo vòng bằng `python scripts/qc_sample.py --round 1`, điền phiếu sau khi thực sự xem ảnh, rồi chạy `python scripts/qc_report.py --round 1`.

Dùng các loại lỗi của repo: `bo_sot`, `thua`, `sai_lop`, `sai_vi_tri_kich_thuoc`, `sai_thuoc_tinh`; ảnh đã kiểm sạch dùng `khong`. Mỗi lỗi một dòng. Bỏ sót chưa có annotation thì để trống `object_id`, ghi vùng và mô tả.

- **Tỉ lệ lỗi theo ảnh:** số ảnh có ít nhất một lỗi / số ảnh đã duyệt. Một ảnh nhiều lỗi chỉ tính một lần ở tử số.
- **Tỉ lệ lỗi theo đối tượng trong script:** số annotation đối tượng hiện có bị lỗi / số annotation đối tượng hiện có đã duyệt; không tính `ignore_region`. Một annotation nhiều lỗi chỉ tính một lần. Vật bỏ sót được đếm riêng vì chưa có annotation; không trộn vào mẫu số này.
- Ghi cả tử số, mẫu số và phần trăm. Không có đối tượng thì ghi N/A cho tỉ lệ đối tượng, vẫn xét lỗi theo ảnh và số bỏ sót.

Quyết định theo thứ tự:

1. **Hold:** thiếu bằng chứng, thiếu review, guideline/dữ liệu còn vấn đề, nhãn cuối còn cờ review, vòng review không đúng phiên bản hoặc mới chỉ lấy mẫu demo.
2. **Rework:** bằng chứng đủ nhưng tỉ lệ ảnh lỗi >2%, hoặc script phát hiện lỗi lặp. Trong script hiện tại, lỗi lặp là một loại có ít nhất 3 dòng lỗi và chiếm ít nhất 50% tổng số dòng lỗi. Ghi nguyên nhân, phạm vi sửa và người kiểm lại.
3. **Release:** bằng chứng đầy đủ, duyệt toàn bộ lô, đạt tiêu chí và reviewer xác nhận. Với 40 ảnh, 1 ảnh lỗi là 2,5%, đã vượt ngưỡng; để đạt ngưỡng theo ảnh cần 0/40 ảnh lỗi trong vòng cuối.

Giữ các vòng cũ khi sửa và chạy vòng mới. Hoàn thiện `QC_REPORT.md`, ghi chú cá nhân và kiểm gói bằng `python scripts/check_submission.py --round <vòng-cuối>`. Chưa thực hiện QC thì không điền số liệu hay quyết định Release.

### 5.5. Phiên bản và điều kiện đưa vào sử dụng

**v1.0-draft — 16/11/2026:** bản đề xuất hiện tại, bổ sung định nghĩa lớp, ngưỡng thuộc tính, quy tắc box, ví dụ đủ 40 ảnh, điểm cần xác minh và quy trình QC. Chưa có người duyệt, nhãn hay kết quả QC đi kèm.

Khi người học rà xong, điền mã tác giả/reviewer và giải quyết mục 4.3. Nếu dùng làm bản cá nhân, lưu bản đã tự kiểm theo `submission/guidelines/individual/guideline_<mã>.md` và mô tả hỗ trợ AI trung thực trong ghi chú cá nhân. Khi nhóm thống nhất, lưu bản chính thức `submission/guidelines/guideline_v1.0_agreed.md`, đồng bộ taxonomy version `v1.0`, chạy `make_cvat_labels.py` rồi khóa cấu hình cho lượt gán độc lập.

Sau calibration, lưu `guideline_v1.1_calibrated.md` và `submission/guideline_change.json`. Sửa câu chữ giữ nguyên quyết định nhãn là `editorial`. Đổi cách chọn vật, lớp, box hoặc thuộc tính là `semantic`: cần ít nhất một quyết định phạm vi `lo` và rà toàn lô. Nếu đổi taxonomy, lưu `taxonomy_v1.1.json` và sinh lại cấu hình cuối theo GUIDE. Không đổi các ngưỡng `rules`.

### 5.6. Tài liệu đối chiếu

- [Hướng dẫn quy trình](../../../GUIDE.md), [quy tắc làm bài và AI](../../../RULES.md), [rubric](../../../RUBRIC.md).
- [Taxonomy hiện tại](../../../submission/taxonomy.json), [schema](../../../schemas/taxonomy.schema.json).
- [QC cheatsheet](../../../docs/QC_CHEATSHEET.md), [hướng dẫn script và phiếu QC](../../../scripts/README.md).
- [Nguồn và giấy phép ảnh](../../../data/README.md), [ghi công từng ảnh](../../../data/ATTRIBUTION.csv).

Giữ tên ảnh và ghi công gốc. Tài liệu này không thay thế việc người học quan sát ảnh, quyết định nhãn và vẽ box theo quy định Day 07.
