# Guideline gán nhãn — v1.0 — Nhóm 156 & 160

> Bản đặc tả thống nhất của nhóm (`156`, `160`) ở Pha 2 sau khi đối chiếu hai bản cá nhân `guideline_156.md` và `guideline_160.md`.
> Bản này được **khóa cứng** để thực hiện gán nhãn độc lập trên CVAT ở Pha 3.

- Phiên bản: `v1.0`
- Ngày: 2026-11-16
- Người viết / duyệt: `156`, `160`

---

## 1. Mục tiêu
- Xây dựng bộ nhãn bounding box 2D chuẩn xác cho bài toán phát hiện **người và phương tiện cơ giới đường bộ thực tế** trên ảnh đường phố (phục vụ camera giám sát giao thông / ADAS).
- **Sai sót nguy hiểm cần tránh:**
  - *Bỏ sót (False Negative):* Không gán người đi bộ, người đi xe máy, xe đang lưu thông hoặc xe con nằm trên sàn xe tải (`img_002.jpg`).
  - *Báo động giả (False Positive):* Gán nhầm tượng đồng/đá (`img_011.jpg`, `img_017.jpg`, `img_027.jpg`), ma-nơ-canh (`img_033.jpg`), đồ chơi (`img_038.jpg`), hình người trên biển quảng cáo (`img_018.jpg`), hoặc ảnh phản chiếu qua gương (`img_020.jpg`).

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
| `person` | Con người thật ở mọi tư thế (đi bộ, đứng, ngồi ghế, lái hoặc ngồi sau xe máy/xe đạp/xích lô/tuk-tuk). | `img_001.jpg`, `img_003.jpg`, `img_006.jpg` | Tượng người (`img_011.jpg`, `img_017.jpg`), ma-nơ-canh (`img_033.jpg`), ảnh in trên biển quảng cáo (`img_018.jpg`), bóng phản chiếu (`img_020.jpg`). |
| `ignore_region` | Vùng đám đông người hoặc cụm phương tiện ở xa chồng lấp dày đặc không thể tách từng cá thể, hoặc vùng chứa cụm vật cao `< 20 px`. | Cụm người/xe dày đặc ở hậu cảnh xa | Không dùng để bỏ qua vật đơn lẻ nhìn rõ có chiều cao `>= 20 px`. |

- **Xử lý vật ngoài taxonomy (`R-02`):** Các phương tiện thô sơ không động cơ như **xe đạp, xích lô đạp chân, xe kéo tay** (`img_006.jpg`, `img_029.jpg`, `img_039.jpg`), tàu hỏa/tàu điện, con vật → **Không vẽ box phương tiện**, nhưng **vẫn gán `person`** cho người thật đang điều khiển hoặc ngồi trên phương tiện đó.

### 2.2. Bảng thuộc tính (`attributes`) và ngưỡng đo được
| Thuộc tính | Kiểu | Giá trị cố định | Mặc định | Ngưỡng cụ thể và quy tắc chọn |
|---|---|---|---|---|
| `occluded_level` | `select` | `0`, `1`, `2`, `3` | `0` | Tỉ lệ diện tích của đối tượng bị **vật thể khác** che khuất:<br>- `0`: Không bị che (`0%`).<br>- `1`: Bị che ít, từ `1% – 30%` diện tích.<br>- `2`: Bị che vừa/nhiều, từ `31% – 70%` diện tích.<br>- `3`: Bị che rất nặng `> 70%` diện tích nhưng phần nhìn thấy vẫn cao `>= 20 px` và nhận diện chắc chắn lớp.<br>*(Lưu ý: Luôn chỉnh thuộc tính `occluded_level` trong panel Objects của CVAT, không dùng phím `Q` mặc định của CVAT).* |
| `truncated` | `select` | `no`, `yes` | `no` | - `no`: Toàn bộ phần nhìn thấy của đối tượng nằm trọn trong khung ảnh.<br>- `yes`: Đối tượng bị cắt một phần bởi đường biên/mép bức ảnh. |
| `can_review` | `checkbox` | `false`, `true` | `false` | - `false`: Nhãn rõ ràng theo guideline.<br>- `true`: Tick khi người gán lưỡng lự về lớp, ranh giới box hoặc mức che khuất để xem lại ở Pha 4–5. |

---

## 3. Quy tắc vẽ box (Mã hóa `R-01` → `R-08`)

- **`R-01` (Chỉ gán vật thật trực tiếp — Loại trừ tượng, ma-nơ-canh, đồ chơi, hình in, phản chiếu, xe chủ):**
  - Tuyệt đối **không gán** tượng điêu khắc người/xe (`img_011.jpg`, `img_017.jpg`, `img_027.jpg`), ma-nơ-canh (`img_033.jpg`), mô hình/đồ chơi (`img_038.jpg`), hình người/xe in trên biển quảng cáo/poster/thân xe (`img_018.jpg`), ảnh phản chiếu trong gương/kính hoặc bộ phận của xe gắn camera chụp ảnh — ego-vehicle (`img_020.jpg`).
- **`R-02` (Phân lớp phương tiện đặc thù & Out-of-taxonomy):**
  - Xe 3 bánh có động cơ và mái che gán vào `tuk_tuk` (`img_004.jpg`, `img_016.jpg`, `img_035.jpg`).
  - Xe buýt 1–2 tầng, xe khách lớn gán vào `bus` (`img_009.jpg`, `img_031.jpg`, `img_037.jpg`, `img_040.jpg`).
  - Xe bán tải (pickup) có phần thùng hàng phía sau tách biệt với cabin — kể cả khi lắp thêm nắp thùng cao như chiếc `Master 75` ở `img_037.jpg` — thống nhất gán là `truck`.
  - Xe đạp, xích lô không động cơ (`img_006.jpg`, `img_029.jpg`, `img_039.jpg`): không gán phương tiện, chỉ gán `person`.
- **`R-03` (Box ôm sát phần nhìn thấy & xử lý che khuất/cắt mép):**
  - Box phải ôm sát cực trị phần diện tích **nhìn thấy thực tế** trên ảnh 2D; không phán đoán vẽ mở rộng ra vùng bị vật khác che khuất.
  - Nếu một vật bị cột điện/cây/người chắn ngang chia thành 2 phần rời nhau nhưng biết chắc cùng 1 vật, vẽ **1 box duy nhất** bao phủ các phần nhìn thấy của vật đó và chọn `occluded_level` tương ứng (`1`, `2` hoặc `3`).
  - Nếu vật chạm và bị cắt bởi rìa bức ảnh, vẽ sát mép ảnh và đặt `truncated = yes`.
- **`R-04` (Tách rời người và phương tiện — Người đi xe máy / xe đạp / tuk-tuk):**
  - Với người đang lái hoặc ngồi trên `motorcycle` (hoặc xe đạp/xích lô/tuk-tuk): luôn gán **box `person` riêng** cho từng người (nếu thấy rõ và cao `>= 20 px`) và **box `motorcycle`/`tuk_tuk` riêng** cho chiếc xe. Box `person` chỉ ôm cơ thể người, box xe chỉ ôm phần xe (hai box giao nhau là bình thường).
- **`R-05` (Xe chở xe — Car carrier):**
  - Trên ảnh xe tải chở ô tô (`img_002.jpg`), gán **1 box `truck`** cho xe tải/rơ-moóc chuyên chở VÀ gán **riêng từng box `car`** cho mỗi chiếc ô tô con nằm trên sàn chở (với mọi xe nhìn rõ và cao `>= 20 px`).
- **`R-06` (Ngưỡng kích thước tối thiểu `20 px` & vùng `ignore_region`):**
  - Không gán box cá thể cho đối tượng có chiều cao phần nhìn thấy `< 20 px` hoặc bị che khuất tới mức không thể phân biệt lớp.
  - Chỉ dùng `ignore_region` cho cụm đám đông người/xe ở xa dính liền nhau không thể tách từng box hoặc cụm vật nhỏ `< 20 px`.
- **`R-07` (Không bao hàng hóa quá khổ ngoài khung xe):**
  - Khi phương tiện chở hàng hóa chất cao/thò dài vượt ra ngoài kết cấu khung/thùng xe (như khối bao tải chất cao trên nóc xe `tuk_tuk` ở `img_035.jpg`), box của phương tiện **chỉ ôm phần thân/khung xe thật**, không kéo box bao trùm phần hàng hóa nhô ra ngoài.
- **`R-08` (Loại trừ xác/vỏ xe phế liệu không hoàn chỉnh):**
  - Khung vỏ xe phế liệu rỗng đặt trên nóc nhà/nóc xe hoặc tháo rời không phải phương tiện hoàn chỉnh (`img_024.jpg`) thì **không gán** là phương tiện.

---

## 4. Bảng ca khó đã thống nhất
| Ảnh | Tình huống | Rule ID | Quyết định của nhóm | Lý do |
|---|---|---|---|---|
| `img_002.jpg` | Xe tải chở 8 ô tô con trên 2 sàn | `R-05` | Gán 1 box `truck` + 8 box `car` riêng biệt | Các xe con trên sàn đều là ô tô thật nhìn rõ (`>= 20 px`), cần phát hiện đầy đủ. |
| `img_004.jpg`, `img_016.jpg`, `img_035.jpg` | Xe 3 bánh tuk-tuk; riêng `img_035.jpg` chở bao tải chất rất cao trên nóc | `R-02`, `R-07` | Gán lớp `tuk_tuk`; ở `img_035.jpg` box chỉ ôm khung xe tuk-tuk, không bao khối bao tải trên nóc | Thêm lớp `tuk_tuk` vào taxonomy v1.0; bao tải là hàng hóa tạm thời, không phải thân xe. |
| `img_006.jpg`, `img_029.jpg`, `img_039.jpg` | Xe đạp và xích lô chở khách | `R-02`, `R-04` | Không gán xe đạp/xích lô; chỉ gán `person` cho người đạp và hành khách | Phương tiện thô sơ không động cơ nằm ngoài taxonomy. |
| `img_009.jpg`, `img_031.jpg`, `img_037.jpg`, `img_040.jpg` | Xe buýt 1 tầng và 2 tầng | `R-02` | Gán lớp `bus` | Bổ sung lớp `bus` vào taxonomy v1.0 để phân biệt với `van` và `truck`. |
| `img_011.jpg`, `img_017.jpg`, `img_027.jpg` | Tượng người bằng đồng/đá và tượng xe bán tải | `R-01` | Không gán tượng (`img_017.jpg`, `img_027.jpg` có 0 box; `img_011.jpg` chỉ gán người/xe thật nếu có) | Tượng điêu khắc không phải đối tượng giao thông thật. |
| `img_018.jpg` | Chân dung người trên biển quảng cáo lớn | `R-01` | Không gán hình trên biển quảng cáo; chỉ gán người thật phía dưới | Hình in 2D phẳng không phải người thật. |
| `img_020.jpg` | Gương chiếu hậu xe chủ và ảnh phản chiếu trong gương | `R-01` | Không gán (0 box) | Loại trừ ego-vehicle và hình ảnh phản chiếu qua gương. |
| `img_024.jpg` | Thân vỏ ô tô phế liệu đặt trên nóc | `R-08` | Không gán vỏ xe phế liệu; chỉ gán các xe thật dưới đường | Vỏ xe rỗng tháo rời không còn là phương tiện giao thông. |
| `img_033.jpg`, `img_038.jpg` | Ma-nơ-canh trong tủ kính và xe mô hình đồ chơi | `R-01` | Không gán (0 box) | Ma-nơ-canh và đồ chơi không phải người/xe thật. |
| `img_037.jpg` | Xe bán tải `Master 75` có gắn nắp thùng cao phía sau | `R-02` | Thống nhất gán `truck` | Khung gầm xe bán tải có thùng hàng riêng biệt với cabin. |

---

## 5. QC và lịch sử phiên bản
- **Ngưỡng QC cố định (theo `submission/taxonomy.json`):**
  - Chiều cao box tối thiểu: `min_box_height_px = 20`
  - Ngưỡng ghép cặp: `match_iou = 0.5`
  - Ngưỡng IoU đánh giá kích thước: `iou_thr_large = 0.85` (vật `>= 50 px`), `iou_thr_small = 0.70` (vật `< 50 px`)
  - Ngưỡng đạt IAA toàn lô: `iaa_iou >= 0.85`, `iaa_class >= 0.95`
  - Ngưỡng tỉ lệ ảnh lỗi tối đa khi duyệt QC Release: `error_rate_img_max <= 0.02` (`<= 2%`)
- **Phân công vai trò nhóm (`156`, `160`):**
  - Điều phối & tổng hợp script: `160`
  - Review bản cuối ở Pha 6: `156`

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| `v0` | Bản khởi đầu của giảng viên (6 lớp: `car`, `truck`, `van`, `motorcycle`, `person`, `ignore_region`) | Chưa có định lượng `occluded_level`, thiếu `bus`/`tuk_tuk`, chưa có quy tắc xử lý tượng, phản chiếu, xe chở xe, hàng quá khổ. |
| `v1.0` | Hợp nhất 2 bản cá nhân (`156`, `160`): bổ sung 2 lớp `bus`, `tuk_tuk` (tổng 8 lớp); chuẩn hóa ngưỡng `%` cho `occluded_level`; thiết lập 8 quy tắc `R-01` → `R-08` | Giải quyết triệt để các bất đồng tiềm ẩn trên `img_002`, `img_004`, `img_006`, `img_009`, `img_011`, `img_017`, `img_018`, `img_020`, `img_024`, `img_027`, `img_033`, `img_035`, `img_037`, `img_038`. |
