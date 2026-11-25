# Báo cáo Day 13 — Lê Tiến Bình, 2A202603346

## 1. Kết luận trước
Mã số `2A202603346`, task `D13-2A202603346`, 10 frame: s01f14, s01f29, s02f39, s02f54, s03f00, s03f21, s04f00, s04f21, s05f10, s05f34. Máy: macOS, trình duyệt Chrome, CVAT 2.82.1 (chạy local). Ngồi cạnh: 2A202603234.

Tôi vẽ tổng cộng **136 cuboid** (109 Car, 15 Pedestrian, 7 Cyclist, 3 Truck, 2 Bus). Frame ít nhất là s02f54 và s05f34, mỗi frame 6 hộp, vì cảnh chỉ có xe con và ít vật rõ trong 30 m. Frame nhiều nhất là s04f21 (28 hộp) và s03f21 (26 hộp, nhiều người đi bộ và xe hai bánh phía sau). `selfcheck` cờ **8/136** cuboid. Tôi tin nhất phần **hướng của xe** (yaw của mọi xe đều quanh 90° hoặc 270°, khớp chiều làn) và **đáy** của các xe gần (z đáy khoảng −1,7 đến −1,8 m). Yếu nhất là **hướng người đi bộ** (7 người còn yaw = 0°, tức chưa xoay), **vật xa 30–50 m** (ít điểm, khó căn tâm) và **occluded** (chưa bật cho cuboid nào).

Cách dùng CVAT:
- Góc nhìn: kéo chuột trái để xoay, chuột phải để dời, con lăn để phóng to; xoay khung 3D tới khi nhìn thẳng xuống để xem toàn cảnh 360°.
- Vẽ: nút cuboid → chọn label → **Shape** → nháy đúp lên cụm điểm; chỉnh tâm, dài, rộng, yaw ở **Top**; chỉnh đáy và chiều cao ở **Side/Front** (không xoay ở hai khung này).
- Hướng: bật **Appearance → Cuboid orientation**, căn vạch đỏ ngắn (+X của hộp) về đầu xe; xem cuboid trên ảnh camera bằng `python scripts/lab.py preview` (ảnh trong `work/xem/`).

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ. Ảnh: `t1-s01f14-3d`, `t1-s01f14-top`, `t1-s01f29-3d`, `t1-s01f29-top`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f14 | Phía trước (−y) ở cạnh **phải** ảnh; +x (vạch đỏ) chỉ lên trên = bên trái xe; +y (vạch xanh lá) chỉ sang trái ảnh = phía sau xe | Phẳng; đáy các xe gần ở z ≈ −1,73 đến −1,79 m, tức mặt đường thấp hơn LiDAR khoảng 1,7–1,8 m | Xe con phía sau khoảng 51 m (còn 53 điểm) | Xe con, xe buýt phía trước khoảng 29 m, một người đi bộ | Phía sau: 2 xe con cùng chiều ở 19,5 m và 25,9 m; một người đi bộ sau-phải khoảng 15,8 m; bên trái: xe con sát cạnh khoảng 3,5 m | Các cụm điểm dày, cao ở hai bên lề đường, không có hình hộp rõ: có thể là cây, cột hoặc tường nhà; tôi chưa xác định được nên không gán nhãn |
| s01f29 | Phía trước (−y) ở cạnh **phải** ảnh (cùng góc nhìn mặc định như s01f14); +x (vạch đỏ) chỉ lên trên = bên trái xe; +y (vạch xanh lá) chỉ sang trái ảnh = phía sau xe | Phẳng; đáy các xe gần ở z ≈ −1,70 đến −1,76 m | Xe con phía sau khoảng 41 m (74 điểm) | Xe con, một xe tải (bán tải hoặc tải nhẹ) | Bên trái khoảng 11–12 m: một xe tải và 3 xe con đi ngược chiều (yaw ≈ 90°), ở 13–35 m; phía sau: xe con cùng chiều ở 7,2 m, 17,2 m, 41 m | Dải điểm cao chạy dọc bên phải đường: có thể là hàng rào, cây hoặc tường; chưa xác định được nên không gán nhãn |

Chỉ camera cho biết: (1) màu và kiểu xe, ví dụ xe buýt phía trước ở s01f14 có màu đỏ-trắng; (2) phân biệt đầu và đuôi xe đứng yên, đèn phanh; (3) vạch kẻ đường, đèn tín hiệu và biển báo.
Chỉ LiDAR cho biết: (1) khoảng cách theo mét, ví dụ xe buýt ở s01f14 cách 29,3 m; (2) kích thước thật, ví dụ xe buýt dài khoảng 12,4 m; (3) vật nằm ngoài tầm cam01, ví dụ các xe phía sau ở s01f14 (19,5 m, 25,9 m) và xe đi ngược chiều bên trái ở s01f29.

Tám frame lướt (phía trước luôn là −y, ngược chiều vạch xanh lá):

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f39 | −y, ngược vạch xanh lá → cạnh phải ảnh (góc nhìn mặc định) | Một xe máy có người phía sau-phải khoảng 11,8 m |
| s02f54 | −y → cạnh phải ảnh (góc nhìn mặc định) | Ít vật, chỉ có xe con |
| s03f00 | −y → cạnh phải ảnh (góc nhìn mặc định) | Tường quây công trình bên trái tạo vùng bóng che |
| s03f21 | −y → cạnh phải ảnh (góc nhìn mặc định) | Nhiều người đi bộ và xe hai bánh phía sau-phải ở 19–41 m; mặt đường phía sau cao hơn (đường dốc) |
| s04f00 | −y → cạnh phải ảnh (góc nhìn mặc định) | 4 người đi bộ bên phải ở 15–32 m |
| s04f21 | −y → cạnh phải ảnh (góc nhìn mặc định) | Cảnh đông, 28 vật; có xe cao khoảng 2,35 m (van?) |
| s05f10 | −y → cạnh phải ảnh (góc nhìn mặc định) | Xe hai bánh xa bên trái khoảng 41 m, rất ít điểm |
| s05f34 | −y → cạnh phải ảnh (góc nhìn mặc định) | Chỉ có xe con |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f14, 3 | Car | Có, yaw = 272,0° (cùng chiều xe thu dữ liệu) | Có, z đáy = −1,794 m, khoảng hở với mặt đường −0,03 m | Có | Có: 4,372 × 1,900 × 1,500 m, trong khoảng sedan (rộng 1,90 m ở cận trên) |
| (b) xe buýt hoặc xe tải | s01f14, 6 | Bus | Có, yaw = 283,7° (cùng chiều, phía trước 29,3 m) | Có, z đáy = −1,829 m, khoảng hở −0,06 m | Có | Có: 12,403 × 2,600 × 3,329 m, trong khoảng xe buýt thường |
| (c) người đi bộ hoặc xe hai bánh | s02f39, 5 | Cyclist | Có, yaw = 284,8° (cùng chiều) | Có, z đáy = −1,788 m, khoảng hở 0,01 m | Có | Có: 1,800 × 0,878 × 1,653 m, trong khoảng xe máy có người |

Vòng hiệu chuẩn: tôi không tham gia vòng hiệu chuẩn nên không có số liệu so với các bạn; tôi tự đối chiếu với guideline (hướng theo đầu xe, đáy chạm đường, bảng 4.3).
Vật bỏ qua vì không đủ cơ sở: không bỏ qua vật rõ nào trong 30 m. Ca sát ngưỡng: s03f21 idx 14 (người đi bộ, bên trái khoảng 23 m, chỉ 16 điểm) là LOW_VISIBILITY; tôi vẫn giữ hộp nhưng chưa chỉnh kích thước, ghi ở §6(c).

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s03f00 / `t4-1-bong-che` / task của tôi | Giả thuyết: tường quây công trình bên trái, cách khoảng 6 m, chắn tia LiDAR nên phía sau tường không có tia nào tới | Một vùng trống hình quạt phía sau bức tường, trong khi hai bên vùng đó vẫn dày điểm; vùng trống là **chưa quan sát được**, không phải đường trống | Vật nằm một phần trong vùng che thì bật occluded và cuboid bao trọn phần suy luận được; vật bị che hẳn thì không vẽ |
| 2. Thưa ở xa | s04f21 idx 28 / `t4-2-thua-o-xa` / task của tôi | Giả thuyết: LiDAR 40 tia có khoảng cách giữa các tia tăng theo khoảng cách, nên vật xa chỉ bị 1–2 tia quét qua | Xe con ở 49,99 m chỉ còn 47 điểm (xe con ở 0–10 m có trung vị khoảng 2209 điểm), chỉ thấy một mặt, khó xác định đầu đuôi và chiều dài | Vẽ khi thấy khoảng 15–20% thể tích hoặc đối chiếu được với ảnh camera; không đủ thì bỏ qua và ghi LOW_VISIBILITY |
| 3. Phản xạ thấp / ít điểm bất thường | s03f21 idx 12 / `t4-3-phan-xa-thap` / task của tôi | Giả thuyết: xe sơn tối màu hoặc có nhiều mặt kính hấp thụ hoặc phản xạ lệch tia laser | Xe con ở 27,35 m chỉ có 57 điểm, trong khi xe con cùng tầm 20–30 m có trung vị khoảng 265 điểm; intensity thấp | Không khẳng định nguyên nhân; vẫn vẽ theo hình dạng suy ra được và kích thước bảng 4.3, kiểm bằng ảnh camera |
| 4. Điểm lạ | s02f35 / `t4-4-diem-la` / **frame mượn từ task chung D13-HC485087** | Giả thuyết: phản xạ nhiều lần (multipath) qua mặt kính hoặc vật bóng, hoặc nhiễu cảm biến | Cụm khoảng 46 điểm phía sau-trái, cách khoảng 29 m, ở z ≈ −13,5 m, thấp hơn mặt đường quanh đó khoảng 11 m; chỉ thấy rõ khi xoay nghiêng khung 3D | Không gán thành vật, không vẽ cuboid |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603346-bangA.csv`; số điểm lấy từ cột `points_in` của `-tukiem.csv`.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s01f14, 3, Car | −1,044 m | 1,500 m | −1,794 m | 3,53 m | 3,68 m | 272,0° | Chạm đường (khoảng hở −0,03 m) |
| s01f14, 7, Car | −0,406 m | 1,500 m | −1,156 m | 34,45 m | 34,45 m | 275,4° | **Lơ lửng**: cao hơn mặt đường 0,59 m, cần hạ xuống |
| s04f21, 19, Car | −0,953 m | 1,563 m | −1,734 m | 14,22 m | 14,25 m | 89,5° | z đáy gần −1,7 m nhưng `selfcheck` báo thấp hơn mặt đường tại chỗ 0,74 m: mặt đất chỗ đó cao hơn (có thể là dải phân cách hoặc vỉa hè), cần xem lại ở Side |

(a) Gần nhất: s01f14 idx 3 (Car), `dist_m` = 3,53 m, r = 3,68 m. Xa nhất: s03f21 idx 18 (Car), `dist_m` = 52,08 m, r = 52,08 m, nằm ngoài 50 m nên không được chấm. Xa nhất trong 50 m: s04f21 idx 28 (Car), `dist_m` = 49,99 m, r = 50,01 m.

(b) Người đi bộ s01f14 idx 4: 0,554 × 0,629 × 1,739 m, cả ba chiều trong khoảng người lớn (0,4–0,9 × 0,5–0,9 × 1,50–1,95 m). Xe con s04f21 idx 2: 5,272 × 1,789 × 2,353 m. Chiều cao 2,353 m vượt cả sedan (1,40–1,60 m) lẫn SUV/van (1,65–2,10 m), chiều dài cũng vượt sedan. Nhiều khả năng đây là xe van hoặc minibus, nên cần xem lại ảnh camera để đổi class (Truck nếu là van chở hàng).

(c) Xe cùng chiều phía trước: s01f14 idx 5, cách 14,81 m, `yaw_deg` = 269,7°, gần 270°. Xe ngược chiều: s01f29 idx 4, bên trái 13,12 m, `yaw_deg` = 89,5°, gần 90°.

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s01f14, 3, Car | 3,53 m | 3053 |
| s01f14, 5, Car | 14,81 m | 717 |
| s01f14, 2, Car | 25,87 m | 195 |
| s01f29, 6, Car | 34,80 m | 96 |

Quy luật: cứ xa thêm khoảng 10 m thì số điểm còn khoảng 1/3 đến 1/4; xa gấp khoảng 10 lần (3,5 m → 35 m) thì số điểm còn khoảng 1/30. Trung vị xe con của tôi theo khoảng (0–10 m: 2209, 10–20 m: 753, 20–30 m: 265, 30–50 m: 105 điểm) khớp với đường cong trên slide (1968, 718, 214, 72 điểm); số ở 30–50 m cao hơn một chút vì khoảng này rộng hơn và gồm cả xe ở 30–40 m.

Tự kiểm, hai cuboid đã sửa sau `selfcheck` (Save rồi chạy lại `sync`): (1) s01f14 idx 4, Pedestrian: hạ hộp 0,08 m (z từ −0,515 m xuống −0,595 m), khoảng hở với mặt đường giảm từ 0,31 m còn 0,23 m; vẫn còn cờ "88 điểm sát mép" và yaw chưa xoay. (2) s01f14 idx 8, Car: hạ hộp 0,055 m, z đáy từ −1,644 m thành −1,698 m, sát mặt đường −1,7 m. Chưa sửa s01f14 idx 7 (vẫn lơ lửng 0,59 m).
Cảnh báo giữ nguyên và lý do: s05f34 idx 5 (cao 2,32 m): xe cao, nhiều khả năng SUV hoặc van, hộp vẫn sát cụm điểm. s03f21 idx 19 (chìm 0,31 m, 45 m, 41 điểm): ít điểm nên mặt đất tại chỗ ước lượng kém tin cậy. s03f21 idx 23 (27 điểm sát mép): có người đứng cạnh trong nhóm người đi bộ. s05f10 idx 17 (Cyclist rộng 1,58 m, 41 m, 53 điểm): chưa chắc hướng do quá ít điểm, ghi vào §6(c).

Đổi kiểm: kiểm chéo với bạn 2A202603234 (ngồi cạnh), đối chiếu số trong §5 với bảng A mở trên máy.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Model phát hiện vật 3D phải dự đoán vị trí và kích thước thật của vật để xe tính khoảng cách an toàn và lập đường đi. Nhãn đo bằng mét trong hệ tọa độ LiDAR không đổi theo góc nhìn, còn pixel thì co giãn theo khoảng cách và camera, nên không dùng làm mục tiêu 3D được. Vật thưa điểm cần quy tắc riêng (ngưỡng 15–20% thể tích, LOW_VISIBILITY): nếu người gán nhãn đoán bừa, model sẽ học cách "tưởng tượng" ra vật không có bằng chứng; nếu bỏ sót tuỳ tiện, model học cách bỏ qua vật xa thật. Quy tắc thống nhất giúp nhãn nhất quán giữa các người gán.

(b) Dùng AI:
- Công cụ: Claude Code (trợ lý AI trong VS Code).
- Câu hỏi chính: cài môi trường, hệ trục VF và hướng xe ego, cách thao tác CVAT 3D; soát bảng A và phiếu tự kiểm để chỉ ra cuboid nghi lỗi; đặt tên và đóng gói ảnh; soạn nháp báo cáo từ bảng A và phiếu tự kiểm.
- Cách dùng kết quả: tôi tự vẽ và sửa cuboid trên CVAT, tự chụp ảnh; số trong báo cáo lấy từ bảng A, phiếu tự kiểm do `sync` sinh; tôi đã đối chiếu và chỉnh lại các mục quan sát theo ảnh của mình. AI không xem ảnh camera hay ảnh chụp màn hình.

(c) Điều còn chưa chắc: hướng của 7 người đi bộ còn yaw = 0° (s01f14 idx 4; s03f21 idx 3, 14, 21; s04f00 idx 14, 15; s05f10 idx 21), có thể bị tính là ngược đầu. s03f21 idx 14 vẫn là hộp mặc định 1 × 1 × 1 m. Class của các "Car" cao trên 2,3 m (s04f21 idx 2, s05f34 idx 5) có thể là van hoặc Truck. Chưa bật occluded cho cuboid nào dù có vật bị che. Ba hộp ngoài 50 m (s01f14 idx 8, s03f21 idx 18, s05f10 idx 16) không được chấm.
