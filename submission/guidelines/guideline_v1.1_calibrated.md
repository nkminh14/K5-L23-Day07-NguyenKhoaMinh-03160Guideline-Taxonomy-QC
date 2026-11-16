# Guideline gán nhãn — v1.1 (Calibrated) — Nhóm 156 & 160

> Bản đặc tả `v1.1` sau buổi họp Calibration ở Pha 5 giữa hai thành viên `156` và `160`.
> Cấp độ thay đổi: `editorial` (bổ sung ví dụ minh họa và làm rõ câu chữ cho `R-01`, `R-03`, `R-07` từ 3 ca `C001`, `C002`, `C003`; giữ nguyên bộ 8 lớp và thuộc tính của `taxonomy.json` v1.0).

- Phiên bản: `v1.1`
- Ngày: 2026-11-16
- Người viết / duyệt: `156`, `160`

---

## 1. Mục tiêu
- Xây dựng bộ nhãn bounding box 2D chuẩn xác cho bài toán phát hiện **người và phương tiện cơ giới đường bộ thực tế** trên ảnh đường phố (phục vụ camera giám sát giao thông / ADAS).
- **Sai sót nguy hiểm cần tránh:**
  - *Bỏ sót (False Negative):* Không gán người đi bộ, người đi xe máy, xe đang lưu thông hoặc xe con nằm trên sàn xe tải (`img_002.jpg`).
  - *Báo động giả (False Positive):* Gán nhầm tượng đồng/đá (`img_011.jpg`, `img_017.jpg`, `img_027.jpg`), ma-nơ-canh (`img_033.jpg`), đồ chơi (`img_038.jpg`), hình người trên biển quảng cáo (`img_018.jpg`), ảnh phản chiếu qua gương (`img_020.jpg`), hoặc vật trang trí/cờ hiệu ở đuôi xe (`img_002.jpg`).

---

## 2. Lớp và thuộc tính

### 2.1. Bảng định nghĩa lớp (`classes`)
| Lớp | Định nghĩa (một câu) | Ví dụ trong bộ ảnh | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con chở người (sedan, hatchback, SUV, crossover, coupe) 4 bánh. | `img_001.jpg`, `img_002.jpg` (8 xe con trên sàn xe tải), `img_008.jpg` | Xe bán tải có thùng hàng (`truck`), xe thùng kín (`van`), tượng xe (`img_027.jpg`), ô tô đồ chơi (`img_038.jpg`). |
| `truck` | Xe tải chở hàng, xe bán tải (pickup kể cả có lắp nắp thùng sau), xe đầu kéo/rơ-moóc, xe chuyên dụng có thùng hàng tách biệt cabin. | `img_001.jpg`, `img_002.jpg` (xe tải chở ô tô), `img_037.jpg` (bán tải `Master 75`) | Xe van thân liền khối (`van`), tượng bán tải (`img_027.jpg`). |
| `van` | Xe thùng kín cỡ nhỏ/trung (minivan, panel van, MPV/camper van) có khoang chở hàng/khách liền khối với cabin. | `img_005.jpg`, `img_010.jpg` | SUV (`car`), xe buýt cỡ lớn (`bus`), xe bán tải có thùng rời (`truck`). |
| `bus` | Xe buýt công cộng (1 tầng hoặc 2 tầng) hoặc xe khách cỡ lớn thiết kế chở nhiều hành khách. | `img_009.jpg`, `img_031.jpg`, `img_037.jpg`, `img_040.jpg` | Xe `van` cỡ nhỏ, xe `truck`. |
| `tuk_tuk` | Xe ba bánh có động cơ và khung/mái che (auto-rickshaw / tuk-tuk / xe lam) chở khách hoặc hàng hóa. | `img_004.jpg`, `img_016.jpg`, `img_035.jpg` | Xe máy 2 bánh (`motorcycle`), xích lô đạp chân (`img_006.jpg`, `img_029.jpg`). |
| `motorcycle` | Xe gắn máy, xe tay ga, mô tô 2 bánh có động cơ. | `img_003.jpg`, `img_007.jpg` | Xe đạp / xích lô không động cơ (`img_006.jpg`, `img_029.jpg`, `img_039.jpg`), người ngồi trên xe (`person`). |
| `person` | Con người thật ở mọi tư thế (đi bộ, đứng, ngồi ghế, lái hoặc ngồi sau xe máy/xe đạp/xích lô/tuk-tuk). | `img_001.jpg`, `img_003.jpg`, `img_006.jpg` | Tượng người (`img_011.jpg`, `img_017.jpg`), ma-nơ-canh (`img_033.jpg`), ảnh in trên biển quảng cáo (`img_018.jpg`), bóng phản chiếu (`img_020.jpg`), cờ/bạt đỏ đuôi xe (`img_002.jpg`). |
| `ignore_region` | Vùng đám đông người hoặc cụm phương tiện ở xa chồng lấp dày đặc không thể tách từng cá thể, hoặc vùng chứa cụm vật cao `< 20 px`. | Cụm người/xe dày đặc ở hậu cảnh xa | Không dùng để bỏ qua vật đơn lẻ nhìn rõ có chiều cao `>= 20 px`. |

- **Xử lý vật ngoài taxonomy (`R-02`):** Các phương tiện thô sơ không động cơ như **xe đạp, xích lô đạp chân, xe kéo tay** (`img_006.jpg`, `img_029.jpg`, `img_039.jpg`), tàu hỏa/tàu điện, con vật → **Không vẽ box phương tiện**, nhưng **vẫn gán `person`** cho người thật đang điều khiển hoặc ngồi trên phương tiện đó.

### 2.2. Bảng thuộc tính (`attributes`) và ngưỡng đo được
| Thuộc tính | Kiểu | Giá trị cố định | Mặc định | Ngưỡng cụ thể và quy tắc chọn |
|---|---|---|---|---|
| `occluded_level` | `select` | `0`, `1`, `2`, `3` | `0` | Tỉ lệ diện tích của đối tượng bị **vật thể khác** che khuất:<br>- `0`: Không bị che (`0%`).<br>- `1`: Bị che ít, từ `1% – 30%` diện tích.<br>- `2`: Bị che vừa/nhiều, từ `31% – 70%` diện tích.<br>- `3`: Bị che rất nặng `> 70%` diện tích nhưng phần nhìn thấy vẫn cao `>= 20 px` và nhận diện chắc chắn lớp. |
| `truncated` | `select` | `no`, `yes` | `no` | - `no`: Toàn bộ phần nhìn thấy của đối tượng nằm trọn trong khung ảnh.<br>- `yes`: Đối tượng bị cắt một phần bởi đường biên/mép bức ảnh. |
| `can_review` | `checkbox` | `false`, `true` | `false` | - `false`: Nhãn đã chốt chuẩn.<br>- `true`: Tick khi người gán lưỡng lự ở Pha 3 (được giải quyết hết và tắt về `false` ở bản cuối Pha 5). |

---

## 3. Quy tắc vẽ box (Mã hóa `R-01` → `R-08`, cập nhật làm rõ tại v1.1)

- **`R-01` (Chỉ gán vật thật trực tiếp — Loại trừ tượng, ma-nơ-canh, đồ chơi, hình in, phản chiếu, xe chủ, vật treo):**
  - Tuyệt đối **không gán** tượng điêu khắc người/xe (`img_011.jpg`, `img_017.jpg`, `img_027.jpg`), ma-nơ-canh (`img_033.jpg`), mô hình/đồ chơi (`img_038.jpg`), hình người/xe in trên biển quảng cáo/poster (`img_018.jpg`), ảnh phản chiếu trong gương/kính hoặc bộ phận của ego-vehicle (`img_020.jpg`), hay dải cờ/vải bạt cảnh báo treo ở đuôi rơ-moóc (`img_002.jpg` — ca `C002`).
- **`R-02` (Phân lớp phương tiện đặc thù & Out-of-taxonomy):**
  - Xe 3 bánh có động cơ và mái che gán vào `tuk_tuk` (`img_004.jpg`, `img_016.jpg`, `img_035.jpg`).
  - Xe buýt 1–2 tầng, xe khách lớn gán vào `bus` (`img_009.jpg`, `img_031.jpg`, `img_037.jpg`, `img_040.jpg`).
  - Xe bán tải (pickup) có phần thùng hàng phía sau tách biệt với cabin — kể cả khi lắp thêm nắp thùng cao như chiếc `Master 75` ở `img_037.jpg` — thống nhất gán là `truck`.
  - Xe đạp, xích lô không động cơ (`img_006.jpg`, `img_029.jpg`, `img_039.jpg`): không gán phương tiện, chỉ gán `person`.
- **`R-03` (Box ôm sát phần nhìn thấy & xử lý che khuất/cắt mép):**
  - Box phải ôm sát cực trị phần diện tích **nhìn thấy thực tế** của đúng đối tượng đó trên ảnh 2D; không kéo mở rộng box sang vùng bị che hoặc sang biển bạt/quầy quán kế bên (ví dụ `img_001.jpg` — ca `C001`: box `truck` dừng ở đúng mép trước thân xe tải `[400, 81, 560, 559]`, không kéo sang tấm bạt quảng cáo bên trái).
  - Nếu vật chạm và bị cắt bởi rìa bức ảnh, vẽ sát mép ảnh và đặt `truncated = yes`.
- **`R-04` (Tách rời người và phương tiện — Người đi xe máy / xe đạp / tuk-tuk):**
  - Với người đang lái hoặc ngồi trên `motorcycle` (hoặc xe đạp/xích lô/tuk-tuk): luôn gán **box `person` riêng** cho từng người và **box `motorcycle`/`tuk_tuk` riêng** cho chiếc xe.
- **`R-05` (Xe chở xe — Car carrier):**
  - Trên ảnh xe tải chở ô tô (`img_002.jpg`), gán **1 box `truck`** cho xe tải/rơ-moóc chuyên chở VÀ gán **riêng từng box `car`** cho mỗi chiếc ô tô con nằm trên sàn chở.
- **`R-06` (Ngưỡng kích thước tối thiểu `20 px` & vùng `ignore_region`):**
  - Không gán box cá thể cho đối tượng có chiều cao phần nhìn thấy `< 20 px`.
- **`R-07` (Không bao hàng hóa quá khổ ngoài khung xe hoặc quang gánh ngoài cơ thể người):**
  - Box phương tiện chỉ ôm phần thân/khung xe thật, không bao khối bao tải chất cao trên nóc (`img_035.jpg`).
  - Tương tự với `person`: box `person` chỉ ôm cơ thể người thật (đầu, thân, tay, chân), **không kéo box bao trùm quang gánh, thúng hàng hóa** mang theo bên dưới/bên cạnh (ví dụ `img_003.jpg` — ca `C003`: chốt box `[506, 0, 202, 396]` chỉ ôm thân người bán hàng rong).
- **`R-08` (Loại trừ xác/vỏ xe phế liệu không hoàn chỉnh):**
  - Khung vỏ xe phế liệu rỗng đặt trên nóc nhà/nóc xe (`img_024.jpg`) thì **không gán** là phương tiện.

---

## 4. Bảng ca khó & ca Calibration đã chốt
| Ảnh | Tình huống | Rule ID | Quyết định của nhóm | Lý do |
|---|---|---|---|---|
| `img_001.jpg` (`C001`) | Xe tải Suzuki đỗ trước quán có bạt quảng cáo bên trái | `R-03` | Chốt box `truck` `[400, 81, 560, 559]` theo `156` | Ôm sát đúng phần thân xe tải nhìn thấy, không kéo sang bạt quảng cáo bên trái. |
| `img_002.jpg` (`C002`) | Xe tải chở 8 ô tô con trên 2 sàn và dải cờ/bạt đỏ treo ở đuôi rơ-moóc | `R-01`, `R-05` | Giữ 1 `truck` + 8 `car` + 1 `person` + 1 `van`; xóa box `person` thừa `[912, 315, 15, 59]` ở đuôi xe | Dải vải/cờ đỏ ở đuôi rơ-moóc không phải người thật. |
| `img_003.jpg` (`C003`) | Người bán hàng rong gánh hai thúng mít | `R-07` | Chốt box `person` `[506, 0, 202, 396]` theo `160` | Chỉ ôm cơ thể người, không kéo box bao trùm thúng mít phía dưới. |
| `img_018.jpg` (`C004`) | Chân dung người trên biển quảng cáo `STAROSTA` | `R-01` | Xóa box `person` `[211, 207, 107, 108]` của `156`; chỉ giữ 5 box `car` dưới đường | Hình người in phẳng 2D trên biển quảng cáo không phải người thật. |
| `img_035.jpg` (`C005`) | Xe 3 bánh tuk-tuk chở bao tải chất cao gấp đôi thân xe | `R-02`, `R-07` | Chốt box `tuk_tuk` `[394, 204, 152, 211]` theo `160` | Chỉ ôm khung xe tuk-tuk, không bao khối bao tải hàng quá khổ trên nóc. |
| `img_037.jpg` (`C006`–`C008`) | Xe bán tải `Master 75` lắp nắp thùng cao đi cạnh xe buýt | `R-02` | Chốt lớp `truck` cho box `[221, 138, 506, 358]` và tắt `can_review` | Khung gầm xe bán tải có thùng hàng tách biệt với cabin. |
| `img_011.jpg`, `img_017.jpg`, `img_027.jpg` | Tượng người bằng đồng/đá và tượng xe bán tải | `R-01` | Không gán tượng (`img_017`, `img_027` có 0 box) | Vật điêu khắc vô tri. |
| `img_020.jpg`, `img_033.jpg`, `img_038.jpg` | Phản chiếu gương/xe chủ, ma-nơ-canh, đồ chơi | `R-01` | Không gán (0 box) | Không phải đối tượng giao thông thật trực tiếp. |

---

## 5. QC và lịch sử phiên bản
| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| `v0` | Bản khởi đầu của giảng viên | Chưa định lượng `occluded_level`, thiếu `bus`/`tuk_tuk`, thiếu quy tắc ca biên. |
| `v1.0` | Hợp nhất 2 bản cá nhân (`156`, `160`): 8 lớp, ngưỡng `%` cho `occluded_level`, mã hóa `R-01` → `R-08` | Khóa đặc tả cho lượt gán độc lập Pha 3. |
| `v1.1` | Đính chính câu chữ (`editorial`) ở `R-01`, `R-02`, `R-03`, `R-07` và bổ sung 8 ca Calibration (`C001` → `C008`) | Giải quyết 8 ca lệch giữa `156` và `160` trên 6 ảnh (`img_001`, `img_002`, `img_003`, `img_018`, `img_035`, `img_037`). |
