# Báo cáo Day 13 — Vũ Thị Nhung, 2A202603156

## 1. Kết luận trước
Mã số học viên: 2A202603156; Task: `D13-2A202603156`. Bộ 10 frame được phân bổ: `s01f09`, `s01f46`, `s02f14`, `s02f52`, `s03f19`, `s03f43`, `s04f02`, `s04f29`, `s05f05`, `s05f48`. Môi trường thực hiện: Hệ điều hành Windows 11, trình duyệt Google Chrome, phiên bản CVAT v2.83.0. MSSV bạn ngồi cạnh đối chiếu: 2A202603157.
Tổng cộng đã gán nhãn 63 cuboid trên 10 frame (trung bình 6–9 hộp/frame), trong đó frame `s01f09` nhiều nhất (9 cuboid) do quang cảnh thông thoáng có nhiều xe đỗ và lưu thông, các frame còn lại đều đạt 6 cuboid (vượt chỉ tiêu tối thiểu 53 cuboid của đề bài). Công cụ `selfcheck` ghi nhận 25/63 cuboid có cờ cảnh báo (chủ yếu là cảnh báo kích thước xe bán tải/xe nhỏ ngoài dải trung bình và xe ở xa bị thưa điểm). Tôi tự tin nhất ở phần xác định đúng class và góc xoay yaw theo hướng di chuyển của xe (xe cùng chiều quanh 270°, xe ngược chiều quanh 90°); phần còn thách thức nhất là căn chỉnh mặt đáy cho các xe ở khoảng cách trên 30 m do điểm mặt đường bị thưa.

Cách dùng CVAT:
- Đổi góc nhìn: Kéo chuột trái để xoay không gian 3D, kéo chuột phải để tịnh tiến (pan), lăn con lăn chuột để phóng to/thu nhỏ; nháy đúp chuột vào khoảng trống để đưa góc nhìn về mặc định.
- Vẽ và chỉnh một cuboid: Chọn Shape (không dùng Track), nháy đúp chuột tại cụm điểm trên khung 3D để đặt hộp; điều chỉnh tâm, chiều dài, chiều rộng và góc xoay yaw tại khung Top view; điều chỉnh nâng/hạ độ cao đáy chạm mặt đường tại khung Side view và Front view.
- Bật định hướng và đối chiếu ảnh: Bật Appearance → Cuboid orientation để hiện vạch đỏ ngắn (+X của cuboid) chỉ thẳng về đầu xe; chạy lệnh `python scripts/lab.py preview` để kiểm tra độ khớp của cuboid được chiếu lên 7 góc ảnh camera.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: `s01f09`, `s01f46`. Ảnh minh hoạ: `t1-s01f09-3d.png`, `t1-s01f09-top.png`, `t1-s01f46-3d.png`, `t1-s01f46-top.png`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f09 | Phía trước là cạnh dưới (-y); +x chỉ sang trái, +y chỉ ra sau | Mặt đường phẳng; thấp hơn tâm LiDAR khoảng 1,7 m (z ≈ -1,7 m) | ~42,7 m (xe con ở xa phía sau-trái) | Car, Truck | Có xe con phía sau (cam09) và xe đỗ hai bên đường (cam06, cam07, cam08) | Cụm điểm phản xạ thưa ở lề đường bên phải cách hơn 45 m |
| s01f46 | Phía trước là cạnh dưới (-y); +x chỉ sang trái, +y chỉ ra sau | Mặt đường tương đối bằng phẳng; thấp hơn LiDAR khoảng 1,7 m | ~25,0 m (xe con cùng làn phía trước) | Car, Truck | Có xe con chạy sau (cam09) và xe bán tải đi song song bên phải (cam07) | Cụm điểm tán cây và cột đèn trên vỉa hè |

Ba điều chỉ camera cho biết:
1. Màu sắc cụ thể của phương tiện và vật thể (ví dụ: xe màu đen, trắng, bạc, đỏ).
2. Biển số xe, logo thương hiệu, chi tiết đèn pha/đèn hậu và đèn xi-nhan đang bật hay tắt.
3. Phân biệt chính xác giữa người dắt xe đạp (Pedestrian) và người đang ngồi lái xe (Cyclist).

Ba điều chỉ LiDAR cho biết:
1. Khoảng cách vật lý chính xác (tính bằng mét) từ tâm xe đến từng bề mặt của vật cản xung quanh.
2. Kích thước hình học 3D thực tế (chiều dài, chiều rộng, chiều cao) không bị ảnh hưởng bởi phối cảnh biến dạng của ống kính.
3. Hoạt động liên tục, đồng nhất 360 độ mà không bị ảnh hưởng bởi điều kiện thiếu sáng ban đêm hay ngược sáng mặt trời.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f14 | Hướng -y (cạnh dưới của ảnh top-down) | Không có hiện tượng bất thường, mật độ xe lưu thông trung bình |
| s02f52 | Hướng -y (cạnh dưới của ảnh top-down) | Xe đỗ sát dải phân cách gây bóng che cục bộ ở làn kế bên |
| s03f19 | Hướng -y (cạnh dưới của ảnh top-down) | Xuất hiện người đi bộ bên lề đường (Pedestrian tại khoảng cách ~19 m) |
| s03f43 | Hướng -y (cạnh dưới của ảnh top-down) | Có nhóm 2 xe hai bánh di chuyển gần nhau ở cự ly ~17–18 m |
| s04f02 | Hướng -y (cạnh dưới của ảnh top-down) | Biển báo bên lề đường có cường độ phản xạ (intensity) rất cao |
| s04f29 | Hướng -y (cạnh dưới của ảnh top-down) | Xe hai bánh di chuyển cắt góc bên trái |
| s05f05 | Hướng -y (cạnh dưới của ảnh top-down) | Xe tải di chuyển sát phía trước bên phải (~10 m) có nhiều điểm |
| s05f48 | Hướng -y (cạnh dưới của ảnh top-down) | Xuất hiện xe ở cự ly rất xa (>50 m) có rất ít điểm phản xạ |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f09, idx 1 | Car | Đúng (yaw_deg = 89,3°, xe ngược chiều) | Đáy z_bottom = -2,01 m (chạm mặt đường) | Đã kiểm đủ 3 view và ảnh cam07 | Dài 4,89 m, Rộng 2,59 m, Cao 1,69 m (phù hợp kích thước Car) |
| (b) xe buýt hoặc xe tải | s01f46, idx 5 | Truck | Đúng (yaw_deg = 273,4°, cùng chiều xe thu) | Đáy z_bottom = -1,30 m (chạm mặt đường dốc nhẹ) | Đã kiểm đủ 3 view và ảnh cam07 | Dài 3,73 m, Rộng 2,05 m, Cao 1,39 m (bán tải thùng hàng) |
| (c) người đi bộ hoặc xe hai bánh | s03f19, idx 2 | Pedestrian | Đúng (yaw_deg = 278,0°, hướng người di chuyển) | Đáy z_bottom = -1,64 m (chạm vỉa hè/đường) | Đã kiểm đủ 3 view và ảnh cam06 | Dài 0,48 m, Rộng 0,57 m, Cao 1,79 m (rất khớp bảng 4.3: 0,4–0,9 × 0,5–0,9 × 1,5–1,95 m) |

Vòng hiệu chuẩn và các trường hợp bỏ qua:
- Vòng hiệu chuẩn: Khi so sánh với các bạn cùng lớp, độ lệch chủ yếu xuất hiện ở việc ước lượng chiều dài thùng sau xe bán tải khi bị che khuất và độ cao đáy hộp ở địa hình hơi nghiêng. Sau thảo luận, cả lớp thống nhất bám sát Bảng 4.3 và quy tắc đáy chạm mặt đường cục bộ quanh vật.
- Các vật bỏ qua vì không đủ cơ sở: Tại frame `s05f48` ở cự ly xa >70 m có cụm 3–5 điểm rời rạc nhưng không nhận diện được hình dáng trên camera (LOW_VISIBILITY), không đủ 15–20% thể tích nên quyết định không gán nhãn theo đúng guideline.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Ảnh: `t4-1-bong-che.png`, `t4-2-thua-o-xa.png`, `t4-3-phan-xa-thap.png`, `t4-4-diem-la.png`.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `t4-1-bong-che.png` (s02f00 trong task chung) | Tia LiDAR bị thân xe buýt lớn che khuất tầm chiếu | Xuất hiện một hình quạt hoàn toàn trống điểm phía sau xe buýt dù hai bên vẫn có điểm mặt đường | Bật thuộc tính occluded, vẽ cuboid bao trọn phần suy luận được dựa trên kích thước xe tiêu chuẩn |
| 2. Thưa ở xa | `t4-2-thua-o-xa.png` (s01f09, idx 9, cách 42,7 m) | Mật độ góc quét của chùm tia laser tỉ lệ nghịch với bình phương cự ly | Xe ở xa chỉ còn 7 điểm phản xạ rời rạc trên nóc và đuôi xe | Chỉ vẽ nếu nhận diện được khoảng 15–20% thể tích và có ảnh camera hỗ trợ; nếu không đủ cơ sở thì bỏ qua |
| 3. Phản xạ thấp / ít điểm bất thường | `t4-3-phan-xa-thap.png` (s04f29, idx 4, xe tối màu) | Bề mặt sơn đen nhám hoặc kính nghiêng hấp thụ/tán xạ chùm tia laser | Xe ở cự ly trung bình (16 m) nhưng nhận được ít điểm hơn nhiều so với các xe màu sáng cùng khoảng cách | Cần đối chiếu kỹ ảnh camera để xác định ranh giới thật của xe, tránh co cụm hộp quá chật quanh số ít điểm |
| 4. Điểm lạ | `t4-4-diem-la.png` (s02f35 trong task chung) | Hiện tượng đa phản xạ (multipath reflection) từ mặt đường ướt hoặc gương/kim loại | Cụm 46 điểm nằm sâu dưới mặt đất ở độ cao z ≈ -13,5 m (thấp hơn mặt đường 11 m) | Tuyệt đối không gán nhãn cụm điểm này thành vật thể giao thông |

## 5. Số từ cuboid của tôi (Bước 5)
Trích xuất từ Bảng A (`submission/D13-2A202603156-bangA.csv`) và phiếu tự kiểm (`submission/D13-2A202603156-tukiem.csv`):

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s01f09, idx 1, Car | -1,169 m | 1,686 m | -2,012 m | 5,10 m | 5,23 m | 89,3° | Chạm đường (lệch mặt đường khoảng -0,21 m) |
| s01f46, idx 2, Car | -1,064 m | 1,493 m | -1,810 m | 9,10 m | 9,16 m | 263,4° | Chạm đường hoàn hảo (lệch +0,01 m so với mặt đường) |
| s03f19, idx 2, Pedestrian | -0,747 m | 1,794 m | -1,644 m | 19,12 m | 19,13 m | 278,0° | Chạm mặt vỉa hè (lệch +0,14 m so với mặt đường ước tính) |

Phân tích số đo:
- Cuboid gần nhất: `s01f46`, idx 5 (Truck) ở `dist_m = 3,08 m`, bán kính không gian `r = 3,14 m`.
- Cuboid xa nhất trong phạm vi gán nhãn: `s01f09`, idx 9 (Car) ở `dist_m = 42,73 m`, bán kính không gian `r = 42,73 m`.
- So sánh kích thước l, w, h với Bảng 4.3:
  - Pedestrian (`s03f19`, idx 2): dài 0,48 m, rộng 0,57 m, cao 1,79 m — hoàn toàn nằm trong dải chuẩn (0,4–0,9 × 0,5–0,9 × 1,50–1,95 m).
  - Car (`s01f46`, idx 2): dài 4,39 m, rộng 2,03 m, cao 1,49 m — khớp chính xác với dải sedan/SUV thông dụng (dài 3,8–5,0 m; rộng 1,7–2,1 m; cao 1,4–1,6 m).
- Góc quay yaw_deg: Xe cùng chiều phía trước (`s01f46`, idx 3) có `yaw_deg = 270,6°` (rất gần 270°); xe ngược chiều (`s01f09`, idx 1) có `yaw_deg = 89,3°` (rất gần 90°), chứng minh trục định hướng đã được căn chính xác dọc theo thân xe.

Mật độ điểm theo khoảng cách (trích xuất từ `points_in` của các xe con Car):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s05f05, idx 1, Car | 3,4 m | 5389 điểm |
| s01f46, idx 2, Car | 9,1 m | 1501 điểm |
| s03f19, idx 3, Car | 16,9 m | 496 điểm |
| s01f09, idx 5, Car | 29,6 m | 138 điểm |

Quy luật mật độ: Khi khoảng cách tăng gấp đôi (từ ~4 m lên ~9 m rồi ~18 m), số lượng điểm LiDAR giảm khoảng 3 đến 4 lần. Quy luật này hoàn toàn tương thích với mô hình phân rã hình học (tỉ lệ nghịch với bình phương khoảng cách $1/r^2$) và trùng khớp với đường cong phân bố điểm của bài giảng (xe trong 0–10 m đạt hàng nghìn điểm, 10–20 m đạt vài trăm điểm, ngoài 30 m giảm xuống dưới 100 điểm).

Tự kiểm và điều chỉnh:
- Sau khi chạy `sync`, tôi đã xem xét các cờ cảnh báo và điều chỉnh lại độ cao đáy của 2 cuboid bị lệch (hạ đáy tiếp xúc mặt đường tại Side view) và xoay lại hướng mũi tên của xe bị ngược chiều.
- Một số cờ cảnh báo về kích thước xe bán tải nhỏ (`s01f46#5`) hay xe ở xa có ít điểm được tôi giữ nguyên vì đã đối chiếu ảnh camera thực tế, xác nhận box đã bao sát viền vật thể thực.
- Đổi kiểm chéo: Đã đối chiếu số liệu Bảng A với học viên MSSV 2A202603157, hai con số so sánh là khoảng cách `dist_m = 5,10 m` và góc `yaw_deg = 89,3°` của cuboid xe con đầu tiên tại frame `s01f09`, kết quả hoàn toàn khớp và nhất quán.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Ý nghĩa đối với mô hình học máy:
- Nhãn 3D phải đo bằng mét trong hệ tọa độ không gian thực thay vì pixel trên ảnh 2D vì các thuật toán điều khiển và lập kế hoạch quỹ đạo của xe tự hành (Autonomous Driving) cần khoảng cách vật lý thực để phanh, giữ khoảng cách an toàn và tránh va chạm. Pixel trên ảnh phụ thuộc vào tiêu cự và góc đặt camera, không phản ánh trực tiếp khoảng cách mét.
- Vật thưa điểm cần có quy tắc riêng (chỉ vẽ khi nhận diện được 15–20% thể tích hoặc có đủ cơ sở suy luận) để ngăn ngừa việc gán nhãn phỏng đoán mang tính chủ quan. Nếu nạp các nhãn đoán sai vào tập huấn luyện, mô hình học máy sẽ học phải nhiễu (noise) và sinh ra các dự đoán ảo (false positives) gây mất an toàn khi xe vận hành.

(b) Cách dùng AI:
- Công cụ: Antigravity Assistant (Gemini).
- Câu hỏi chính: Hỗ trợ kiểm tra cấu trúc thư mục nộp bài, phân tích các chỉ số trong Bảng A và giải thích hệ tọa độ cảm biến VF (trục x trái, y sau, z lên).
- Sử dụng kết quả: Dùng để đối chiếu số liệu Bảng A, hiểu rõ ý nghĩa các cờ cảnh báo trong phiếu tự kiểm để tự rà soát lại các box trên CVAT.

(c) Điều còn chưa chắc:
- Một vài phương tiện ở khoảng cách xa hơn 35 m có mật độ điểm rất thưa (dưới 20 điểm) nên chiều cao thực tế của mui xe chủ yếu được suy đoán dựa theo chiều cao trung bình của dòng xe trong Bảng 4.3 mà chưa thể thấy rõ ranh giới điểm trên cùng.
