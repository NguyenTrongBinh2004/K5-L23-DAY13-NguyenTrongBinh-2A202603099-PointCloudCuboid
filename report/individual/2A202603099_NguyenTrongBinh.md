# Báo cáo Day 13 — Nguyễn Trọng Bình, 2A202603099

## 1. Kết luận trước

Mã số sinh viên: `2A202603099`, task CVAT: `D13-2A202603099`, gồm 10 frame: `s01f00`, `s01f58`, `s02f09`, `s02f42`, `s03f22`, `s03f48`, `s04f04`, `s04f49`, `s05f17`, `s05f36`. Hệ điều hành: Windows 11, trình duyệt: Google Chrome, môi trường gán nhãn: CVAT local (`localhost:8080`), MSSV bạn đổi kiểm: `2A202603089`. Tổng cộng đã gán `103` cuboid; frame có ít hộp nhất là `s02f42` (`5` cuboid) do khung cảnh thông thoáng, ít phương tiện nằm trong vùng điểm LiDAR đủ rõ để xác định hình khối; công cụ `selfcheck` gắn cờ `24` cuboid trên tổng số `103` cuboid. Phần tôi tin cậy nhất là phân loại (`class`) và vị trí tâm ngang (`x, y`) của các phương tiện trong bán kính `30 m` nhờ đối chiếu đồng thời 7 góc camera; phần còn yếu nhất là căn độ cao đáy (`z_bottom`) ở các vùng mặt đường có độ dốc nhẹ và ước lượng kích thước với các xe ở xa trên `35 m` bị thưa điểm.

Cách dùng CVAT:
- Đổi góc nhìn: giữ chuột trái và kéo để xoay góc nhìn 3D, giữ chuột phải (hoặc `Shift` + chuột trái) để dời khung hình, lăn con lăn chuột để thu phóng và xoay thẳng góc nhìn từ trên xuống (Top-down) để quan sát mặt bằng.
- Vẽ và chỉnh một cuboid: chọn công cụ vẽ cuboid ở thanh bên trái, chọn nhãn rồi bấm **Shape**; nháy đúp vào hộp hoặc chọn trong danh sách **Objects** để kéo các điểm điều khiển tại ba khung phụ **Top**, **Side**, **Front**.
- Bật mũi tên hướng và xem ảnh camera: tích chọn **Cuboid orientation** trong bảng **Appearance** để hiển thị mũi tên đỏ chỉ hướng đầu xe, đồng thời kiểm tra khung chiếu 2D của cuboid trên cột ảnh camera (`cam01`–`cam09`) bên phải.

## 2. Đọc frame (Bước 2)

Hai frame đọc kỹ. Ảnh: `t1-s01f00-3d.png`, `t1-s01f00-top.png`, `t1-s01f58-3d.png`, `t1-s01f58-top.png`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f00` | Phía trước xe (`-y`) ở cạnh trên của ảnh nhìn từ trên xuống; trục `+x` chỉ sang trái (bên trái xe thu dữ liệu), trục `+y` chỉ xuống dưới (phía sau xe). | Mặt đường phẳng, thấp hơn tâm cảm biến LiDAR khoảng `1,7 m` (`z ≈ -1,7 m`). | Khoảng `50,9 m` (`Truck` #17). | `Car`, `Bus`, `Truck`, `Cyclist`, `Pedestrian`. | Một xe con chạy sát bên trái (`+x`, cách `3,4 m`) và nhiều xe con nối đuôi phía sau (`+y`, cách từ `9,3 m` đến `25,8 m`). | Vùng phía sau xe con sát bên trái bị che khuất hoàn toàn và các cụm điểm nhỏ ở rìa xa trên `50 m`. |
| `s01f58` | Phía trước xe (`-y`) ở cạnh trên của ảnh nhìn từ trên xuống; trục `+x` chỉ sang trái, trục `+y` chỉ xuống dưới. | Mặt đường tương đối phẳng, thấp hơn tâm cảm biến LiDAR khoảng `1,7 m`. | Khoảng `37,7 m` (`Cyclist` #13). | `Car`, `Cyclist`. | Một xe con chạy sát bên hông ở khoảng cách `3,0 m` cùng các xe phía sau và làn bên cạnh ở cự ly `6,8–16,5 m`. | Một số cụm điểm thưa sát lề đường và các vật thể nhỏ ở xa ngoài `40 m` không đủ mật độ điểm để nhận dạng. |

Ba điều **chỉ camera** cho biết, lấy từ chính hai frame trên:
1. Màu sắc, đặc điểm nhận diện bề ngoài và tín hiệu đèn phanh/đèn xi-nhan của các xe xung quanh.
2. Phân biệt rõ ranh giới giữa người đi bộ (`Pedestrian`) và người điều khiển xe hai bánh (`Cyclist`) ở khoảng cách trung bình trên `20 m` khi cụm điểm LiDAR chỉ còn vài chục điểm.
3. Nhận diện vạch kẻ đường, dải phân cách và biển báo giao thông để biết làn đường di chuyển của từng phương tiện.

Ba điều **chỉ LiDAR** cho biết, lấy từ chính hai frame trên:
1. Tọa độ 3D chính xác và khoảng cách thực tế tính bằng mét (`3,0 m`, `3,4 m`, `50,9 m`) từ cảm biến tới từng vật thể xung quanh.
2. Chiều cao thực tế của vật thể và độ cao mặt đường so với gốc tọa độ cảm biến (`z ≈ -1,7 m`).
3. Vùng không gian bị che khuất hoàn toàn phía sau các vật cản gần (bóng che hình quạt không có điểm phản hồi) và góc nhìn toàn cảnh 360° không bị giới hạn bởi góc mở ống kính từng camera.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f09` | Cạnh trên của ảnh nhìn từ trên xuống (hướng `-y`). | Xuất hiện xe buýt kích thước lớn (`Bus`) ở cự ly `53,4 m` và xe hai bánh ở cự ly gần, mật độ điểm dày. |
| `s02f42` | Cạnh trên của ảnh nhìn từ trên xuống (hướng `-y`). | Khung cảnh thưa phương tiện nhất trong 10 frame (chỉ có `5` vật thể đạt ngưỡng điểm để gán nhãn). |
| `s03f22` | Cạnh trên của ảnh nhìn từ trên xuống (hướng `-y`). | Có nhiều xe con di chuyển ở cự ly `12,5–48,5 m` với cụm điểm thẳng hàng rõ nét. |
| `s03f48` | Cạnh trên của ảnh nhìn từ trên xuống (hướng `-y`). | Xuất hiện nhiều vòng quét mặt đường bị ngắt quãng do xe xung quanh di chuyển sát hai bên hông. |
| `s04f04` | Cạnh trên của ảnh nhìn từ trên xuống (hướng `-y`). | Nhiều xe con chạy song song ở khoảng cách `15–25 m` tạo các vệt bóng che kéo dài ra hai bên. |
| `s04f49` | Cạnh trên của ảnh nhìn từ trên xuống (hướng `-y`). | Một số vật thể ở rìa ngoài `40 m` chỉ còn 1–2 đường quét ngang nên không đủ cơ sở gán nhãn. |
| `s05f17` | Cạnh trên của ảnh nhìn từ trên xuống (hướng `-y`). | Có xe tải lớn (`Truck`) ở cự ly gần `12,9 m` với hơn `6.600` điểm phản hồi, che khuất một góc quét rộng phía sau. |
| `s05f36` | Cạnh trên của ảnh nhìn từ trên xuống (hướng `-y`). | Mặt đường phía trước có độ dốc nhẹ làm tọa độ đáy của một số xe lệch nhẹ so với mức chuẩn `-1,7 m`. |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f58, idx 1` | `Car` | Có, mũi tên đỏ chỉ thẳng về đầu xe (`yaw_deg = 270,1°`). | Có, đáy hộp chạm mặt đường (`z_bottom = -1,755 m`, độ lệch `0,05 m`). | Đã kiểm đủ 3 khung phụ và đối chiếu ảnh camera. | Có (`4,747 × 1,966 × 1,599 m`), nằm trọn trong khoảng chuẩn của `Car`. |
| (b) xe buýt hoặc xe tải | `s01f00, idx 9` | `Bus` | Có, mũi tên đỏ hướng dọc theo chiều tiến của thân xe. | Có, đáy hộp chạm mặt đường (độ lệch `0,07 m`). | Đã kiểm đủ 3 khung phụ và đối chiếu ảnh camera. | Chiều dài và chiều cao đạt chuẩn; chiều rộng `3,33 m` hơi nhỉnh hơn mức `3,2 m` do bao gồm gương chiếu hậu. |
| (c) người đi bộ hoặc xe hai bánh | `s01f00, idx 7` | `Cyclist` | Có, mũi tên đỏ hướng về phía đầu xe hai bánh. | Có, đáy hộp chạm mặt đường (độ lệch `0,01 m`). | Đã kiểm đủ 3 khung phụ và đối chiếu ảnh camera. | Có, kích thước nằm trong khoảng chuẩn của `Cyclist`. |

Qua vòng hiệu chuẩn, cuboid của tôi ban đầu thường có xu hướng để chiều rộng hơi dư khoảng `0,1–0,2 m` so với thân xe thực tế do bao cả vùng điểm nhiễu ở rìa và một vài hộp bị xoay ngang trục chiều dài/chiều rộng (`s01f00, idx 1`). Cả lớp thống nhất quy tắc: ưu tiên ôm khít cụm điểm thân xe chính ở khung **Top**, căn đáy hộp chạm sát đường quét mặt đất trên khung **Side/Front** và luôn xoay mũi tên đỏ chỉ đúng đầu phương tiện thay vì để ngang thân.

Các vật bỏ qua vì không đủ cơ sở:
- `s02f42`, vùng rìa xa phía trước bên phải (`dist_m > 45 m`): cụm điểm chỉ còn vài chấm rời rạc không rõ hình dạng xe hay cột vật cản (`LOW_VISIBILITY`).
- `s04f49`, vùng vỉa hè bên trái (`dist_m > 35 m`): cụm điểm nhỏ bị che khuất một phần bởi xe đỗ phía trước, không phân biệt được người đi bộ hay biển báo/cột đèn (`UNCERTAIN_CLASS`).

## 4. Bốn hiện tượng chất lượng (Bước 4)

Nguyên nhân ghi là **giả thuyết**.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s01f00` / `t4-1-bong-che.png` / task cá nhân | Giả thuyết: tia laser của LiDAR truyền thẳng nên bị thân xe con chạy sát bên trái (`dist_m = 3,4 m`) chắn toàn bộ chùm tia hướng sang vùng không gian phía sau nó. | Xuất hiện một vùng tối hình quạt kéo dài từ sau thân xe chắn ra rìa trái khung hình, hoàn toàn không có đường vòng quét hay điểm phản hồi. | Không vẽ dự đoán cuboid vào vùng tối không có điểm; chỉ gán các vật thể có điểm phản hồi thực tế ở rìa ngoài vùng che khuất. |
| 2. Thưa ở xa | `s01f00` / `t4-2-thua-o-xa.png` / task cá nhân | Giả thuyết: các tia laser phân kỳ theo góc quét nên khi khoảng cách tăng lên `38–53 m`, khoảng cách giữa các tia rộng ra khiến mật độ điểm trên bề mặt vật giảm mạnh. | Xe con `idx 13` ở khoảng cách `38,9 m` chỉ còn `51` điểm xếp thành một vệt mỏng chữ L, mất chi tiết nóc và bánh xe. | Vẫn gán cuboid vì nhận diện được hình chữ L đặc trưng của xe và đối chiếu được trên camera, nhưng phải giữ kích thước hộp theo tỷ lệ chuẩn của `Car` thay vì thu nhỏ theo vệt điểm. |
| 3. Phản xạ thấp / ít điểm bất thường | `s01f58` / `t4-3-phan-xa-thap.png` / task cá nhân | Giả thuyết: bề mặt kính cửa sổ tối màu hoặc thân xe sơn đen hấp thụ phần lớn năng lượng tia hồng ngoại, làm cường độ tia phản xạ về cảm biến yếu hơn ngưỡng thu nhận. | Phần thân dưới và cản xe vẫn có điểm rõ nhưng vùng kính chắn gió và kính hông bị khuyết rỗng, tạo cảm giác thân xe bị lõm hoặc đứt đoạn. | Căn kích thước cuboid dựa trên khung viền ngoài của các điểm phản xạ ở cản trước, cản sau và nóc xe để không bị hụt chiều cao hoặc chiều dài. |
| 4. Điểm lạ | `s05f17` / `t4-4-diem-la.png` / task cá nhân | Giả thuyết: hiện tượng phản xạ đa đường (multipath) qua bề mặt kính/kim loại bóng hoặc nhiễu bụi mịn trong không khí tạo ra các điểm trả về sai vị trí thực. | Xuất hiện các điểm đơn lẻ lơ lửng phía trên nóc xe hoặc nằm tách biệt trên mặt đường trống không gắn với vật thể nào trên camera. | Bỏ qua các điểm nhiễu lẻ khi kéo kích thước cuboid để tránh làm hộp bị phình cao bất thường, và không gán nhãn cho các cụm điểm nhiễu không có vật thật trên camera. |

## 5. Số từ cuboid của tôi (Bước 5)

Lấy từ `submission/D13-2A202603099-bangA.csv`; số điểm lấy từ cột `points_in` của `-tukiem.csv`. Mặt đường khoảng −1,7 m: z đáy lệch hơn 0,3 m là lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f58, idx 1, Car` | `-0,955 m` | `1,599 m` | `-1,755 m` | `3,0 m` | `3,11 m` | `270,1°` | Chạm đường (độ lệch đáy `0,05 m` so với `-1,7 m`, nằm trong ngưỡng `±0,3 m`). |
| `s05f17, idx 11, Car` | `-0,268 m` | `1,980 m` | `-1,258 m` | `20,2 m` | `20,25 m` | `90,2°` | Lơ lửng (đáy cao hơn mặt đường chuẩn `0,70 m`). |
| `s03f22, idx 5, Car` | `-1,063 m` | `1,572 m` | `-1,849 m` | `17,1 m` | `17,15 m` | `88,8°` | Chìm (đáy thấp hơn mặt đường chuẩn `0,55 m`). |

- **(a) Cuboid gần nhất và xa nhất:** Cuboid gần nhất là `s01f58, idx 1` (`Car`) có `dist_m = 3,0 m` và `r = 3,11 m`; cuboid xa nhất là `s02f09, idx 8` (`Bus`) có `dist_m = 53,4 m` và `r = 53,4 m`.
- **(b) Kích thước so với bảng 4.3:** Người đi bộ `s01f00, idx 11` (`Pedestrian`) có `l = 0,782 m`, `w = 0,753 m`, `h = 1,801 m`, tất cả các chiều đều nằm trong khoảng chuẩn của người đi bộ. Xe con `s01f00, idx 4` (`Car`) có `l = 4,667 m`, `w = 1,989 m`, `h = 2,346 m` (`2,35 m`): chiều cao vượt ngưỡng chuẩn `1,2–2,3 m` của `Car` khoảng `0,05 m` vì đây là dòng xe gầm cao (SUV/MPV) có giá nóc và có vài điểm nhiễu phía trên mui xe.
- **(c) Góc `yaw_deg` theo hướng di chuyển:** Một xe chạy cùng chiều phía trước (`s01f58, idx 1, Car`) có `yaw_deg = 270,1°`, gần sát mốc `270°` (tương ứng hướng trục `-y` phía trước xe); một xe chạy ngược chiều (`s01f00, idx 6, Car`) có `yaw_deg = 88,0°`, gần sát mốc `90°` (tương ứng hướng trục `+y`).

Mật độ theo khoảng cách (4 cuboid `Car` ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `s01f58, idx 1, Car` | `3,0 m` | `3175` |
| `s01f00, idx 3, Car` | `12,7 m` | `701` |
| `s01f00, idx 6, Car` | `25,8 m` | `410` |
| `s01f00, idx 13, Car` | `38,9 m` | `51` |

Quy luật mật độ điểm: khi khoảng cách tăng xa gấp khoảng `4,2 lần` (từ `3,0 m` lên `12,7 m`), số điểm phản hồi giảm xuống chỉ còn khoảng `1/4,5` (`3.175` xuống `701` điểm), và khi xa tới `38,9 m` số điểm chỉ còn `51` điểm (giảm hơn `60 lần` so với cự ly `3,0 m`). Xu hướng suy giảm nhanh theo bình phương khoảng cách này khớp với đường cong tham chiếu trong bài giảng (`1.968` điểm ở `0–10 m`, `718` điểm ở `10–20 m`, `214` điểm ở `20–30 m`, `72` điểm ở `30–40 m`, `19` điểm ở `60–80 m`), trong đó xe ở `3,0 m` có số điểm cao hơn mức trung bình nhóm `0–10 m` do nằm sát cảm biến nhất.

Tự kiểm và sửa lỗi:
- Hai cuboid đã sửa sau khi xem `preview` và `selfcheck`:
  1. `s01f58, idx 9`: điều chỉnh lại phân loại phương tiện cỡ lớn phù hợp với kích thước thực tế sau khi đối chiếu lại ảnh camera.
  2. `s02f09, idx 4`: thu gọn chiều rộng hộp trên khung **Top** để loại bỏ phần điểm nhiễu lề đường dính vào hông xe.
- Các cảnh báo giữ nguyên kèm lý do: nhóm 8 hộp `s04f04, idx 1` đến `idx 8` (`Car`) bị gắn cờ `width = 2,31 m` (vượt ngưỡng `2,30 m` đúng `0,01 m`) được giữ nguyên vì khi kiểm tra ảnh chiếu `D13-2A202603099-s04f04.jpg`, khung hộp đã ôm khít thân xe và gương chiếu hậu; tương tự, `s01f00, idx 4` (`height = 2,35 m`) chỉ lệch `0,05 m` do đặc thù xe gầm cao nên không ép nhỏ hộp làm mất điểm nóc xe.

Đổi kiểm: Đã đối chiếu chéo trên màn hình với bạn có MSSV `2A202603089`; hai con số được đối chiếu trực tiếp từ file `D13-2A202603099-bangA.csv` là giá trị `z_bottom` (`-1,755 m`) và `yaw_deg` (`270,1°`) của cuboid gần nhất (`s01f58, idx 1`), cả hai số liệu đều khớp hoàn toàn với kết quả hiển thị trên khung kiểm tra của CVAT.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

- **(a) Góc nhìn từ mô hình học máy 3D:** Mô hình nhận diện 3D cần nhãn đo bằng mét trong không gian thực thay vì pixel trên ảnh phẳng vì kích thước vật thể trên ảnh 2D bị biến dạng theo phối cảnh xa gần, trong khi xe tự hành cần biết chính xác khoảng cách, kích thước vật lý và quỹ đạo thực của chướng ngại vật để lập kế hoạch tránh va chạm. Các vật thể ở xa bị thưa điểm cần quy tắc gán nhãn riêng (giữ kích thước theo chuẩn lớp vật thể thay vì co sát vài điểm lẻ) để mô hình học được hình khối đầy đủ của vật thể ngay cả khi dữ liệu đầu vào bị khuyết thông tin.
- **(b) Khai báo sử dụng AI:**
  - Công cụ: Gemini.
  - Câu hỏi chính: hỗ trợ đọc hiểu tài liệu hướng dẫn (guide) và giải đáp các vấn đề nhỏ phát sinh trong quá trình thực hiện.
  - Cách dùng kết quả: dùng làm thông tin tham khảo để tự thao tác gán nhãn trên CVAT, kiểm tra lại dữ liệu và hoàn thiện báo cáo.
- **(c) Điều còn chưa chắc và lỗi phát hiện sau hạn:** Một số cuboid ở rìa xa trên `38 m` có góc xoay đầu xe (`yaw`) chỉ được ước lượng dựa trên hướng làn đường do cụm điểm LiDAR quá mảnh để thấy rõ mặt trước hay mặt sau. Ngoài ra, một vài hộp bị cờ trong `tukiem.csv` như hộp bị xoay ngang trục chiều dài/chiều rộng ở frame đầu tiên và một số hộp có đáy bị lệch độ cao mặt đường ở hai frame cuối vẫn chưa kịp cập nhật lại lên server trước thời điểm đồng bộ cuối cùng.