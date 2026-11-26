# Báo cáo Day 13 — Nguyễn Khánh Trình, 2A202603026

**phiếu tự kiểm** trong `submission/`, kèm đơn vị (m, °). Ảnh để trong `report/individual/images/` (nộp LMS trong file zip, không commit); trong báo cáo chỉ ghi **tên file ảnh**. Báo cáo này nằm trên repo public: không chép biển số, chữ trên biển hiệu hay biển chỉ đường, tên đường, địa danh; nhắc tới bạn khác thì ghi MSSV, không ghi họ tên. Cả báo cáo ~2 trang.

## 1. Kết luận trước
- Mã số: `2A202603026` · Task: `D13-2A202603026` · 10 frame: `s01f02`, `s01f48`, `s02f05`, `s02f25`, `s03f01`, `s03f31`, `s04f02`, `s04f32`, `s05f00`, `s05f38`.
- Hệ điều hành: Windows · Trình duyệt: Cốc Cốc · Phiên bản CVAT: 2.8.3 · MSSV bạn ngồi cạnh: không có (làm một mình).
- Tổng cộng hiện có **92 cuboid** trên 10 frame; frame ít nhất là `s02f25` (**5 cuboid**) do đoạn đường thẳng có dải phân cách phần lớn các xe khác ở xa hoặc bị che khuất. Mình tin nhất ở **hướng yaw** (các xe cùng chiều quanh 270°, ngược chiều quanh 90°) và kích thước xe con ở gần; phần còn yếu nhất là **căn đáy chạm đường (`z_bottom`)** ở các xe xa/xe buýt lớn và giữ gọn chiều rộng hộp ở xa.
- Cách dùng CVAT:
  - Đổi góc nhìn: kéo chuột trái để xoay khung 3D (xoay thẳng đứng để nhìn từ trên xuống), kéo chuột phải để dời, cuộn con lăn để phóng to/thu nhỏ, nháy đúp chỗ trống để về góc mặc định.
  - Vẽ và chỉnh cuboid: chọn công cụ vẽ cuboid → chọn label → **Shape** (không dùng Track), nháy đúp vào cụm điểm để đặt hộp; chỉnh tâm, dài, rộng, yaw ở **Top** và chỉnh độ cao/đáy ở **Side**, **Front**.
  - Hướng & kiểm tra camera: bật **Appearance → Cuboid orientation** để căn vạch đỏ `+X` về đầu xe, rồi bấm Save và chạy `python scripts/lab.py preview` để đối chiếu hộp trên 7 ảnh camera.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: `s01f02`, `s01f48`. Ảnh: `t1-s01f02-3d.png`, `t1-s01f02-top.png`, `t1-s01f48-3d.png`, `t1-s01f48-top.png`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f02` | Phía trước (−y, hướng cam01) ở cạnh trên; +x chỉ sang trái, +y chỉ xuống dưới (ra sau) | Gần như phẳng, hơi thấp dần về phía trước: ~1,77 m dưới LiDAR ở 3–8 m, ~1,88 m ở 30–50 m | Xe tải ~30 m phía trước-trái (357 điểm); xa hơn chỉ còn vài chục điểm | Car, Truck, Pedestrian, Cyclist (xe máy), cọc phân làn, cột đèn | Ngã tư: 2 người đi bộ + 2 xe máy bên phải ~13–19 m (cam07/cam08); xe con ngay sau ~9–15 m (cam09); dòng xe ngược chiều bên trái ~21–24 m | Cụm điểm cao, thẳng đứng bên phải ~25 m: không rõ cột đèn hay thân cây |
| `s01f48` | Như trên: phía trước ở cạnh trên, +x trái, +y xuống dưới | Phẳng gần xe (~1,78 m dưới LiDAR), phía trước >30 m hơi dốc lên (~1,6 m), phía sau ~1,84 m | Xe trong hàng chờ ~35–40 m phía trước; sau 40 m làn trước gần như không còn điểm (bị hàng xe che) | Car, Cyclist (xe đạp), Pedestrian, dải cây xanh, cột đèn | 2 xe đạp + 2 người đi bộ trên làn xe đạp bên phải ~10–15 m (cam07/cam08); xe con phía sau ~7 m (cam09) | Cụm thấp, dày dọc mép phải ~10 m: bụi cây hay xe đạp dựng sát nhau |

Chỉ **camera** cho biết: (1) màu và kiểu xe (sedan trắng gần, SUV xanh phía sau-phải ở `s01f02`); (2) trạng thái đèn giao thông và vạch kẻ đường (đèn đỏ ở ngã tư, vạch người đi bộ, mũi tên chuyển làn); (3) người trên vỉa hè đang đi bộ hay đang đạp xe, dựa vào tư thế. Chỉ **LiDAR** cho biết: (1) khoảng cách thật bằng mét (xe trước 6,3 m, xe tải 30 m); (2) độ cao mặt đường so với cảm biến (~−1,8 m) và độ dốc nhẹ; (3) vật nằm ở góc mù giữa các hình quạt camera (sát sườn xe) cùng kích thước 3D thật của vật.

Tám frame lướt (đều xoay để phía trước ở cạnh trên):

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f05` | Cạnh trên | Ngã tư rộng, nhiều xe buýt; xe buýt xa nhất ~59 m chỉ 161 điểm; mặt đường thấp dần phía trước (~−2,1 m ở 30–50 m) |
| `s02f25` | Cạnh trên | Đường thẳng có hàng rào giữa, chỉ 5 cuboid; vài điểm lẻ dưới mặt đường ở bên trái xa |
| `s03f01` | Cạnh trên | Hàng xe chờ dày đặc, tường quây công trình bên trái; 6 điểm ở z ≈ −6,3 m ngay phía trước ~13 m |
| `s03f31` | Cạnh trên | Xe buýt đỗ ở trạm phía sau ~22 m; xe xa nhất ~66 m chỉ 19 điểm |
| `s04f02` | Cạnh trên | Nhóm 5 người đi bộ bên phải ~17–24 m; biển báo xanh có intensity cao hơn hẳn |
| `s04f32` | Cạnh trên | Xe hai bánh sát sườn phải ~6 m (1156 điểm); hàng rào giữa đường màu cam |
| `s05f00` | Cạnh trên | Xe tải xanh ngay phía sau ~10 m che gần hết tầm nhìn phía sau |
| `s05f38` | Cạnh trên | Xe buýt phía trước ~42 m, xe tải ngược chiều ~39 m; một xe bị che một phần chỉ cao 1,17 m |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f48`, 3 (CVAT #21; 7,2 m ngay phía sau, cùng chiều) | Car | Có: yaw 269,5° | Có: z đáy −1,86 m, ground_gap −0,06 m | Có, Top/Side/Front khít cụm điểm; đối chiếu cam09 | Dài 5,22 m khớp SUV (4,4–5,3); cao 1,64 m sát cận dưới SUV; **rộng 2,19 m hơi vượt** 2,10 |
| (b) xe buýt hoặc xe tải | `s01f02`, 14 (CVAT #5; 30,4 m phía trước-trái, ngược chiều) | Truck | Có: yaw 88,8° | Có: z đáy −1,62 m (selfcheck không đo được ground_gap vì quanh xe ít điểm mặt đường) | Có, Top/Side/Front; đối chiếu cam01/cam06 | Dài 6,43 m, cao 2,89 m khớp xe tải hộp (5,5–9,0 / 2,40–3,50); **rộng 2,86 m vượt** 2,00–2,55 |
| (c) người đi bộ hoặc xe hai bánh | `s01f02`, 13 (CVAT #6; 19,4 m bên phải, trên vạch qua đường) | Cyclist (xe máy) | Có: yaw 341,8°, theo hướng xe máy đang cắt ngang | Có: z đáy −1,71 m, ground_gap −0,04 m | Có, Top/Side/Front; đối chiếu cam07/cam08 | Dài 2,21 m, cao 1,60 m khớp xe máy có người; **rộng 1,23 m vượt** 0,70–0,95 (selfcheck cũng báo) |

Vòng hiệu chuẩn: không có — buổi lab không thực hiện vòng hiệu chuẩn trên task chung, nên chưa có so sánh cuboid với các bạn khác; mình bám theo guideline và bảng 4.3.

Vật **bỏ qua vì không đủ cơ sở**: `s03f31`, phía trước-trái >60 m, vài cụm chỉ dưới 10 điểm, không rõ xe hay cột — LOW_VISIBILITY; `s02f05`, phía trước-trái ~45–55 m sau dòng xe buýt, cụm điểm thấp bị che một nửa, không chắc là xe con hay xe ba bánh — UNCERTAIN_CLASS.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**. Cả bốn đều lấy từ frame của mình.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s02f05` / `t4-1-bong-che.png` / frame của mình | Xe buýt (idx 6, hộp vàng) cao ~3,7 m ở phía sau-trái ~21 m chắn tia laser | Trong hình quạt ngay sau xe buýt (28–60 m) có **0 điểm**, trong khi hai hình quạt bên cạnh vẫn có 29 và 10 điểm | Vùng trống là "chưa quan sát được", không phải đường trống; vật chỉ thấy một phần thì bật occluded, hộp bao phần suy luận được |
| 2. Thưa ở xa | `s02f05` / `t4-2-thua-o-xa.png` / frame của mình | Mật độ tia giảm theo khoảng cách, chỉ vài tia quét trúng vật ở 40–60 m | Xe buýt idx 12 ở 59 m chỉ còn **161 điểm** dù dài gần 13 m; xe con idx 13 ở 41 m chỉ 102 điểm (xe con ở 9 m cùng frame: 1436 điểm) | Vẫn vẽ vì camera và hình dạng cho suy luận được; kích thước bám bảng 4.3 thay vì nới theo điểm lẻ; ít hơn ~15–20% thể tích thì bỏ, ghi LOW_VISIBILITY |
| 3. Phản xạ thấp / ít điểm bất thường | `s03f01` idx 6 (CVAT #49) / `t4-3-phan-xa-thap.png` / frame của mình | Xe sơn xám tối (thấy trên cam01/cam03) và kính sau hấp thụ, phản xạ yếu | Xe ở 10,7 m: 956 điểm, intensity p90 chỉ ~0,02; xe sáng màu trong cùng frame (idx 9, 16,5 m) có p90 ~2,2; vùng kính sau gần như không có điểm | Ghi là giả thuyết, không khẳng định; vẽ hộp theo thân xe và camera, không thu hộp lại theo phần kính trống |
| 4. Điểm lạ | `s03f01` / `t4-4-diem-la.png` / frame của mình | Phản xạ kép (multipath) qua thân/kính của các xe phía trước, hoặc nhiễu cảm biến | 6 điểm sáng lẻ, thẳng hàng ở (x ≈ 2,5; y ≈ −13,2; z ≈ −6,3 m), tức **~4,5 m dưới mặt đường**, không nối với cụm nào | Không gán thành vật; không để cụm này kéo đáy cuboid xe gần đó xuống |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603026-bangA.csv`; số điểm lấy từ cột `points_in` của `-tukiem.csv`. Mặt đường khoảng −1,7 m: z đáy lệch hơn 0,3 m là lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s03f01`, 2, Car | −0,927 m | 1,561 m | −1,708 m | 5,16 m | 5,24 m | 271,6° | Chạm đường (ground_gap 0,03 m) |
| `s04f32`, 4, Car | −1,363 m | 1,754 m | −2,240 m | 25,74 m | 25,78 m | 276,9° | **Chìm**: thấp hơn −1,7 m ~0,54 m; mặt đường thật ở đó ~−1,94 m nên vẫn chìm ~0,27 m (selfcheck cũng báo) |
| `s04f02`, 15, Car | −0,221 m | 1,756 m | −1,099 m | 22,58 m | 22,58 m | 13,0° | **Lơ lửng**: cao hơn mặt đường ~0,39 m (xe bên phải ~23 m, có thể đỗ trên vỉa hè cao hơn — cần xem lại) |

(a) Gần nhất: `s03f01` idx 1, dist_m = 3,78 m, r = 3,91 m. Xa nhất: `s03f31` idx 5, dist_m = 66,18 m, r = 66,18 m. r lớn hơn dist_m rõ nhất ở vật gần vì z (~−1 m) chiếm tỉ lệ lớn; ở xa hai số gần như bằng nhau.
(b) Người đi bộ `s04f02` idx 9: 0,82 × 0,81 × 1,88 m, cả ba chiều trong khoảng người lớn. Xe con `s01f02` idx 1: dài 4,61 m, cao 1,42 m khớp sedan, nhưng **rộng 2,30 m** vượt 1,70–1,90: nhiều khả năng hộp ăn cả gương chiếu hậu hoặc vẽ lỏng ở Top; đây là lỗi lặp lại ở nhiều xe con của mình (width 2,3–2,6 m).
(c) Xe cùng chiều phía trước `s01f48` idx 1 (y = −10,1 m): yaw 271,2°, gần 270°. Xe ngược chiều `s01f02` idx 7: yaw 89,5°, gần 90°.

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `s03f01`, 2, Car | 5,2 m | 2446 |
| `s02f25`, 2, Car | 17,1 m | 523 |
| `s04f02`, 6, Car | 28,2 m | 288 |
| `s03f31`, 5, Car | 66,2 m | 19 |

Quy luật: xa gấp ~3 lần (5 → 17 m) thì số điểm còn khoảng 1/5; xa gấp ~13 lần (5 → 66 m) thì còn khoảng 1/130 — giảm cỡ 1/r² hoặc nhanh hơn. So với slide (1968 / 718 / 214 / 72 / 19 điểm): cùng dạng đường cong; xe gần của mình nhiều điểm hơn (2446 > 1968 vì ở sát 5 m), ở 17 m ít hơn (523 < 718), còn ở 60–80 m trùng đúng 19 điểm.

Hai cuboid **đã sửa** sau `preview` / `selfcheck`:
- Car 51 trong CVAT: một xe con trong `s03f01` (frame này mang ID 44–52). Sửa gì: chỉnh đáy hộp cho chạm mặt đường và chỉnh lại chiều cao ở khung Side/Front cho khớp nóc xe; sau khi sửa, không xe con nào trong `s03f01` còn cảnh báo đáy (ground_gap từ −0,14 đến 0,04 m).
- Bus 57 trong CVAT = `s03f31` idx 2 (xe buýt ở trạm phía sau, 21,9 m). Sau khi sửa: 12,44 × 3,67 × 3,10 m, yaw 268,8°, z đáy −1,80 m (ground_gap −0,11 m); vẫn còn cảnh báo rộng 3,67 m (xem bên dưới). Sửa gì: chỉnh đáy hộp xuống chạm mặt đường và chỉnh lại chiều cao ở khung Side/Front cho khớp nóc xe buýt; chưa siết chiều rộng.

Cảnh báo **giữ nguyên**: (1) width 2,30–2,61 m ở nhiều xe con (`s01f02` idx 1, 3, 4, 6, 8, 15; `s02f05` idx 2, 3, 5, 11…) — mặt cạnh xe ở 15–25 m thưa điểm nên khó siết, ghi nhận là điểm yếu; (2) `s02f05` idx 7, 12 và `s03f31` idx 2 (Bus) rộng/cao vượt bảng — xe buýt xa hoặc sát vật khác, điểm lẫn vào nhau; (3) `s05f38` idx 5 cao 1,17 m — xe bị che phần trên; (4) `s01f48` idx 10 "57 điểm sát ngay ngoài mép" — hai người đi bộ đứng sát nhau, nới hộp sẽ ăn sang người kia; (5) `s01f02` idx 13 (xe máy) rộng 1,23 m — nghi hộp ăn cả điểm của người đi bộ/cọc phân làn sát bên trên vạch qua đường; cần siết lại.

Đổi kiểm: không có — buổi lab không thực hiện kiểm chéo với bạn khác, nên chưa có con số nào được người khác đối chiếu với bảng A.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Model 3D phải dự đoán vị trí và kích thước vật trong không gian thật để xe lập kế hoạch (phanh, đổi làn), nên nhãn phải tính bằng mét trong hệ LiDAR; pixel đổi theo khoảng cách và camera nên không cho biết xe cách bao xa hay rộng bao nhiêu. Vật thưa điểm cần quy tắc riêng (ngưỡng ~15–20% thể tích, kích thước theo bảng 4.3, cờ LOW_VISIBILITY) để mọi người gán nhất quán — nếu không, model học từ nhãn lúc có lúc không, hộp lúc to lúc nhỏ, và sẽ dự đoán kém ở tầm xa.

(b) Dùng AI:
- Công cụ: Claude (Claude Code).
- Câu hỏi chính: tổng hợp số từ bảng A / phiếu tự kiểm và point cloud (mặt đường, số điểm, điểm dưới mặt đất) để điền báo cáo.
- Cách dùng: lấy số liệu và bản nháp câu chữ, tự đối chiếu lại với CVAT và ảnh camera trước khi nộp.

(c) Còn chưa chắc: mình vẽ chiều rộng xe con lớn một cách có hệ thống (thường 2,2–2,4 m), cần siết lại ở Top; hai xe bên phải ~22–25 m trong `s04f02` (idx 13, 15) có đáy cao hơn mặt đường ~0,3–0,4 m — chưa rõ do vỉa hè cao hay do mình đặt hộp lơ lửng; xe `s04f32` idx 4 chìm ~0,27 m, cần nâng lên.
