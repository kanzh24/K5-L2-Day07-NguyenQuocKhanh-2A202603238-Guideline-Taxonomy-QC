# Guideline gán nhãn cá nhân — bản nháp để thảo luận

- Phiên bản cá nhân: Đặng Anh Tuyền
- Mã thành viên: 2A202603039

## 1. Mục tiêu

Bộ nhãn này dùng để huấn luyện mô hình phát hiện người và phương tiện trong ảnh đường phố. Hướng dẫn dưới đây trình bày cách chọn đối tượng cần gán nhãn, chọn lớp và vẽ khung bao quanh đối tượng. Trong bài, mình gọi khung này là box để tiện đối chiếu với thao tác trên CVAT.

Khi gán nhãn, mình chú ý những lỗi dễ gặp như bỏ sót người bị che khuất, gộp người với xe vào cùng một box, hoặc nhầm tượng và hình quảng cáo với người thật. Với ảnh đông người, thiếu sáng hoặc bị nhòe, cần phóng to và xem kỹ từng vùng trước khi quyết định.

Đây là bản đề xuất cá nhân để trao đổi trong buổi họp nhóm. Những trường hợp còn phân vân được ghi kèm ID ảnh và lý do để cả nhóm cùng xem và thống nhất.

## 2. Lớp và thuộc tính

### Chọn lớp cho từng vật

Mỗi đối tượng được gán vào một lớp phù hợp. Với các lớp có sẵn trong bài, mình đề xuất cách phân biệt như sau.

| Lớp | Cách hiểu và ví dụ | Ca dễ nhầm |
|---|---|---|
| `car` | Ô tô con, taxi, SUV và xe gia đình có khoang hành khách liền thân. Ví dụ xe con màu đỏ trên xe chở xe ở `img_002`. | Xe bán tải có thùng chở hàng phía sau sẽ vào `truck`. |
| `truck` | Xe tải và xe bán tải có cabin cùng phần chở hàng riêng. Ví dụ xe chở xe ở `img_002`, xe tải nhỏ ở `img_036`. | Xe bán tải lắp nắp thùng như `img_037` vẫn xếp vào đây nếu nhận ra kết cấu bán tải. |
| `van` | Xe có thân dạng hộp kín, khoang sau dùng chở người hoặc hàng, không có thùng hàng tách riêng kiểu bán tải. Ví dụ xe màu sáng phía sau người đi xe máy ở `img_005`. | Xe tải nhỏ có mui bạt như `img_036` vẫn là `truck`. Xe buýt và minibus ngoài phạm vi đề xuất này. |
| `motorcycle` | Xe máy, xe tay ga, mô tô. Xe có thùng bên gắn liền như `img_022` được tính là một xe. | Xe đạp, xích lô và tuk-tuk để ngoài phạm vi cho tới khi nhóm thống nhất. |
| `person` | Người thật, gồm người đi bộ, người ngồi, người lái xe và hành khách nhìn thấy được. Ví dụ hai người trên xe máy ở `img_026`. | Tượng, ma-nơ-canh, búp bê và người trên biển quảng cáo được loại. |
| `ignore_region` | Vùng cần bỏ qua vì vật quá nhỏ hoặc quá chồng chéo để gán riêng đáng tin cậy. | Chỉ khoanh đúng vùng khó. Vật còn phân biệt rõ bên cạnh vẫn được gán riêng. |

**R-01. Đối tượng ngoài phạm vi.** Mình đề xuất bỏ qua xe đạp, xích lô, tuk-tuk, xe buýt, động vật và các đồ vật ngoài danh sách lớp. Người thật ngồi trên hoặc đứng cạnh các phương tiện này vẫn được gán `person` nếu nhìn đủ rõ và đạt ngưỡng kích thước. Trường hợp chưa xác định được lớp của một chiếc xe cần ghi lại để người kiểm xem xét. Phạm vi các loại xe sẽ được cả nhóm chốt trong buổi họp.

**R-02. Xe chở hàng và xe trưng bày.** Xe thật vẫn được gán nhãn khi đang đỗ, được vận chuyển hoặc trưng bày. Với xe tải chở ô tô, vẽ một box cho xe tải và một box riêng cho mỗi ô tô được chở đủ điều kiện gán nhãn. Thùng chở hàng, mui bạt và nắp thùng gắn liền được tính là bộ phận của xe. Khi xác định ranh giới box, chỉ dựa vào phần xe nhìn thấy, không mở rộng theo bao tải, rau quả hoặc thanh dài nhô ra.

### Điền thuộc tính

**R-03. Mức che khuất `occluded_level`.** Thuộc tính này cho biết đối tượng bị vật khác che nhiều hay ít. Mình đề xuất các mức dưới đây. Ngưỡng 30% cần được thử trên một vài ảnh khó trong buổi họp để thống nhất cách ước lượng.

| Giá trị | Cách điền |
|---|---|
| `0` | Không thấy phần nào bị vật khác che. |
| `1` | Có bị che, nhưng phần bị che ước lượng dưới 30% vật. |
| `2` | Phần bị che ước lượng từ 30% trở lên, vẫn nhận ra vật và chọn được lớp. |
| `3` | Nhận ra vật, nhưng không đủ dấu hiệu để ước lượng mức che. Tick thêm `can_review`. |

Khi ước lượng mức che khuất, chỉ xét phần đối tượng nằm trong khung ảnh. Phần vượt ra ngoài mép ảnh được ghi bằng thuộc tính `truncated`. Ví dụ, chân người ngồi trên xe bị thân xe chắn được tính là che khuất. Người quay lưng hoặc xe được chụp từ phía sau vẫn có thể ở mức `0` nếu không bị vật khác che.

**R-04. Cắt mép `truncated`.** Chọn `yes` khi một phần của vật nằm ngoài khung ảnh. Chọn `no` khi vật nằm trọn trong ảnh, kể cả lúc bị một vật khác che. Nếu vật vừa bị che vừa bị cắt mép, ghi cả hai thuộc tính.

**R-05. Đánh dấu cần xem lại `can_review`.** Tick ô này khi chưa chắc về lớp, ranh giới box, mức che khuất hoặc chưa phân biệt được vật thật với mô hình. Ghi ngắn gọn lý do để người kiểm biết điểm cần xem lại. Nếu chưa thể xác định box, ghi lại tên ảnh, vị trí đối tượng và điều còn phân vân vào danh sách vấn đề của ảnh.

## 3. Quy tắc vẽ box

**R-06. Vẽ sát phần nhìn thấy.** Bốn cạnh của box nằm sát điểm cao nhất, thấp nhất, ngoài cùng bên trái và ngoài cùng bên phải của phần đối tượng nhìn thấy được. Nếu đối tượng bị che, dùng các phần còn nhìn thấy của cùng đối tượng để xác định box. Vì box có dạng chữ nhật, bên trong có thể chứa khoảng trống hoặc một phần vật che phía trước. Không kéo dài box theo phần cơ thể hay thân xe chỉ có thể suy đoán.

Box của người gồm đầu, mũ đang đội, thân và tay chân nhìn thấy. Túi xách, ô hoặc hàng đang cầm được để ngoài box nếu chúng nhô ra khỏi người. Box của xe gồm bánh xe, gương và các bộ phận gắn liền nhìn thấy được. Với xe có thùng bên ở `img_022`, box bao cả xe và thùng bên.

**R-07. Tách người và xe.** Vẽ một box `person` cho mỗi người và một box theo lớp tương ứng cho mỗi xe. Ví dụ, một xe máy chở hai người sẽ có tổng cộng ba box nếu cả hai người và xe đều đủ điều kiện gán nhãn. Các box này có thể chồng lên nhau. Người đứng sát hoặc tựa vào xe cũng được gán riêng.

**R-08. Đối tượng nhỏ.** Kiểm tra chiều cao box trên ảnh gốc. Đối tượng có box cao từ 20 px trở lên được gán riêng nếu đủ rõ để xác định lớp và ranh giới. Đối tượng có box thấp hơn 20 px được khoanh bằng `ignore_region` vừa sát vùng cần bỏ qua. Nếu trong cùng một cụm có cả đối tượng rõ và đối tượng quá nhỏ, khoanh riêng phần nhỏ để giữ lại các đối tượng vẫn gán được.

**R-09. Đám đông và xe đỗ sát nhau.** Nếu còn phân biệt được từng người hoặc từng xe, vẫn vẽ box riêng cho từng đối tượng đủ điều kiện. Chỉ dùng `ignore_region` ở phần chồng chéo đến mức khó xác định từng đối tượng. Với các đối tượng nằm sát vùng bỏ qua, cố gắng gán riêng phần còn nhận diện rõ và ghi lại trường hợp cần xem thêm.

**R-10. Tượng, hình in và phản chiếu.** Loại tượng người, ma-nơ-canh, người hoặc xe in trên biển quảng cáo, xe đồ chơi và tượng xe. Phản chiếu trong gương, kính hay mặt đường cũng được loại. Nếu vật thật xuất hiện cùng ảnh phản chiếu của nó, chỉ gán vật thật nhìn trực tiếp. Với chi tiết thuộc chính xe chụp ảnh, như gương và mép cửa trong `img_020`, mình đề xuất bỏ qua.

**R-11. Ảnh tối hoặc bị nhòe.** Phóng to để kiểm tra xem có xác định được lớp và ranh giới đối tượng hay không. Nếu vẫn nhận diện được, tiếp tục gán nhãn và tick `can_review` khi còn phân vân về box. Nếu đối tượng chỉ còn là một vệt mờ, ghi lại vị trí và lý do để nhóm thống nhất cách xử lý.

## 4. Các ca mang vào buổi họp

Bảng dưới đây tập hợp những tình huống cần trao đổi, kèm cách xử lý đề xuất và mã quy tắc liên quan. Các lựa chọn còn phân vân cần được đối chiếu lại trên ảnh gốc trước khi nhóm chốt.

| Ảnh | Điểm cần chú ý | Quy tắc | Đề xuất ban đầu và lý do |
|---|---|---|---|
| `img_001` | Người đứng sát xe bán tải, nhiều người ngồi phía sau | R-03, R-06, R-07 | Tách từng người nhìn thấy và xe bán tải. Người bị bàn hoặc xe chắn vẫn được gán nếu đủ rõ và cao ít nhất 20 px. |
| `img_002` | Xe tải chở nhiều ô tô | R-02, R-06 | Một box cho xe tải và box riêng cho từng xe được chở đủ điều kiện. Các box có thể chồng nhau. |
| `img_003`, `img_010` | Người bán hàng, xe đạp và xe máy ở sát nhau | R-01, R-06, R-07 | Gán người và xe máy. Loại xe đạp, giỏ hàng và gánh hàng theo phạm vi đề xuất. |
| `img_004`, `img_016`, `img_035` | Tuk-tuk hoặc xe ba bánh chở khách | R-01, R-05 | Đề xuất để xe ngoài phạm vi, gán người thật đủ rõ. Cần nhóm chốt cách xử lý vì hình dáng xe dễ bị hiểu thành `motorcycle`. |
| `img_005`, `img_026` | Người lái và người ngồi sau xe máy | R-03, R-07 | Tách từng người và xe. Với xe có hai người, dự kiến ba box nếu các phần nhìn thấy đủ điều kiện. |
| `img_006`, `img_023` | Xe đạp hoặc xích lô có người điều khiển | R-01, R-07 | Gán người riêng, để phương tiện ngoài phạm vi đề xuất. |
| `img_007`, `img_036` | Xe chở hàng có cabin nhỏ, thùng kín hoặc mui bạt | R-02 | Xếp `truck` khi thấy phần chở hàng riêng. Kích thước nhỏ của xe không đủ để chọn `van`. |
| `img_008`, `img_030` | Nhiều xe máy đỗ chồng lên nhau | R-08, R-09 | Gán riêng các xe có thể phân biệt. Chỉ khoanh vùng bỏ qua ở phần quá nhỏ hoặc không tách được. |
| `img_009`, `img_032` | Xe buýt hoặc xe chở khách lớn | R-01, R-05 | Đề xuất để xe ngoài phạm vi. Riêng ranh giới giữa van và minibus cần nhóm chốt bằng ví dụ. Người ở trạm vẫn gán bình thường. |
| `img_011` | Nhân vật Predator cạnh xe máy độ | R-05, R-10 | Cần xác minh hình người là người mặc trang phục hay mô hình. Nếu là người thật thì gán `person`, nếu là tượng thì loại. Xe máy thật đủ rõ vẫn được gán. |
| `img_012`, `img_029` | Hàng hóa hoặc thanh dài nhô khỏi xe | R-02, R-06 | Box xe ôm phần xe. Hàng rời nhô ra được để ngoài. Người chở hàng vẫn có box riêng. |
| `img_013`, `img_014`, `img_031` | Đêm tối, nhòe chuyển động và ánh đèn phản chiếu | R-10, R-11 | Gán vật thật còn nhận diện được. Loại vệt phản chiếu. Ca không xác định được ranh giới cần ghi lại. |
| `img_015` | Xe bị cắt ở mép ảnh | R-04, R-06 | Box dừng ở mép ảnh và chọn `truncated=yes`. |
| `img_017`, `img_018`, `img_033`, `img_038` | Tượng người, người trên biển quảng cáo, ma-nơ-canh và xe đồ chơi | R-10 | Loại các hình người và xe giả này. Vật thật khác xuất hiện trong ảnh vẫn xét riêng. |
| `img_019` | Xe hình nhân vật hoạt hình trong đoàn diễu hành | R-05, R-10 | Cần xác minh đây là phương tiện thật được trang trí hay mô hình diễu hành. Đám đông phía sau được xử lý riêng theo độ rõ của từng người. |
| `img_020` | Gương xe, ảnh phản chiếu và mép cửa xe chụp ảnh | R-10 | Đề xuất bỏ qua các phần này, tránh tạo box cho một chiếc xe chỉ xuất hiện qua phản chiếu. |
| `img_021`, `img_032` | Nhiều người và xe ở xa | R-08, R-09 | Phóng to, kiểm kích thước trên ảnh gốc và tách từng vật còn rõ. Không lấy một box bỏ qua phủ toàn bộ đám đông. |
| `img_022` | Mô tô gắn thùng bên | R-06 | Một box `motorcycle` bao cả xe và thùng bên gắn liền. |
| `img_024` | Xe trưng bày và xe nhỏ đặt trên nóc | R-02, R-05, R-10 | Xe thật trưng bày vẫn gán `car`. Cần kiểm tra chiếc xe nhỏ trên nóc là mô hình hay xe thật trước khi quyết định. |
| `img_025`, `img_039` | Người đi bộ và xe nhỏ ở xa | R-08 | Kiểm chiều cao trên ảnh gốc. Vật đủ lớn, đủ rõ thì gán, phần dưới 20 px chuyển sang vùng bỏ qua. |
| `img_027`, `img_028` | Tượng xe bán tải và xe bán tải có hình ngựa trên thùng | R-02, R-10 | Loại tượng xe ở `img_027`. Với `img_028`, gán xe bán tải thật, phần hình ngựa trên thùng để ngoài box xe. |
| `img_034`, `img_040` | Người đi bộ hoặc người đi xe sát mép ảnh | R-04, R-07 | Tách người và xe. Ghi cắt mép cho đúng vật bị cắt, kể cả khi box của hai vật chồng nhau. |
| `img_037` | Xe bán tải lắp nắp thùng cao | R-02, R-05 | Đề xuất `truck` dựa vào kết cấu bán tải. Cần nhóm thống nhất dấu hiệu phân biệt với `van`. |

Trong buổi họp, mình muốn ưu tiên thống nhất cách xử lý tuk-tuk, xích lô và minibus; dấu hiệu phân biệt xe bán tải với van; cùng các trường hợp chưa rõ là vật thật hay mô hình ở `img_011`, `img_019`, `img_024`. Với ngưỡng che khuất 30%, cả nhóm nên cùng đánh giá một vài đối tượng trong cùng ảnh rồi so sánh cách hiểu.

## 5. Kiểm tra chất lượng và phiên bản

Trước khi xuất COCO, xem lại từng ảnh từ trái sang phải để kiểm tra đối tượng bị bỏ sót, lớp, độ sát của box và các thuộc tính. Mọi ảnh được phân công đều phải có trong file COCO, kể cả ảnh không có đối tượng thuộc phạm vi gán nhãn.

Các ngưỡng QC dùng đúng cấu hình lab. Vật dưới 20 px xử lý theo R-08. Khi kiểm box, ngưỡng IoU là 0.85 cho box cao từ 50 px và 0.7 cho box nhỏ hơn. IoU là tỉ lệ phần giao trên phần hợp của hai box, dùng để kiểm độ trùng khớp vị trí.

Khi so nhãn độc lập, script ghép box với IoU tối thiểu 0.5. IAA yêu cầu IoU trung bình ít nhất 0.85 và đồng thuận lớp ít nhất 95% trên các cặp ghép được. Nhóm cần xem thêm box chưa ghép và các trường hợp cùng gán sai.

Trước khi bàn giao, nhóm cần kiểm tra chất lượng (QC) trên cả 40 ảnh. Báo cáo ghi rõ số ảnh có lỗi trên tổng số ảnh đã duyệt, đồng thời ghi số đối tượng có lỗi trên tổng số đối tượng đã duyệt. Ngưỡng ảnh lỗi tối đa là 2%, nên chỉ một ảnh có lỗi trong lô 40 ảnh cũng đã vượt ngưỡng. Nếu thiếu bằng chứng, kết quả là Hold. Nếu vượt ngưỡng hoặc có lỗi lặp, kết quả là Rework. Nhóm chỉ Release khi người kiểm xác nhận đã đáp ứng yêu cầu bàn giao.

Với trường hợp còn mơ hồ, nhóm cùng xem ảnh, thống nhất cách xử lý và ghi lại quyết định cùng mã người chốt. Bản thống nhất trước lượt gán độc lập được lưu thành guideline v1.0. Sau buổi đối chiếu và xử lý bất đồng (calibration), nhóm cập nhật thành v1.1 và ghi rõ những điểm đã sửa. Bản cá nhân đã nộp được giữ lại để đối chiếu với quyết định chung.

| Phiên bản | Nội dung | Tình trạng |
|---|---|---|
| draft-01 | Đề xuất phạm vi lớp, cách vẽ box, mức che khuất và cách xử lý ca khó | Cần người viết đối chiếu ảnh, điền mã thành viên và xác nhận trước khi nộp |
