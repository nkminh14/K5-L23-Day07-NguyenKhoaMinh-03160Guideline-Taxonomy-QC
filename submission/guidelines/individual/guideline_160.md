# Guideline gán nhãn — v0.1-draft — 160

> Bản dự thảo guideline cá nhân (Pha 1) của thành viên `160`.
> Gồm đủ **5 phần** theo mẫu chuẩn, giải quyết các điểm mơ hồ của `guideline_v0_starter.md` sau khi khảo sát 40 ảnh (`img_001.jpg` – `img_040.jpg`).

- Phiên bản: v0.1-draft (cá nhân `160`)
- Ngày: 2026-11-16
- Người viết / duyệt: `160`

## 1. Mục tiêu
Mô hình nào dùng nhãn này, để làm gì, sai ở đâu thì nguy hiểm:
- Nhãn bounding box 2D dùng để huấn luyện và đánh giá mô hình phát hiện **người và phương tiện giao thông đường bộ đang tham gia hoặc đỗ trên đường phố** (camera giao thông / hỗ trợ lái xe ADAS).
- **Nguy hiểm nếu sai:**
  - *Bỏ sót (False Negative)* người đi bộ, người đi xe máy, phương tiện thật trên đường → nguy cơ va chạm trực diện.
  - *Gán nhầm (False Positive)* tượng đồng/đá, ma-nơ-canh, đồ chơi, ảnh phản chiếu trong gương hoặc hình in trên biển quảng cáo thành người/xe thật → phanh gấp giả.

## 2. Lớp và thuộc tính

### Lớp
| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con, sedan, hatchback, SUV, crossover chở người (4 bánh). | `img_001.jpg`, `img_002.jpg` (xe con trên sàn xe tải) | Xe bán tải có thùng hàng (`truck`), xe thùng kín (`van`), tượng xe (`img_027.jpg`), đồ chơi (`img_038.jpg`). |
| `truck` | Xe tải chở hàng, xe bán tải (pickup), xe đầu kéo hoặc xe chuyên dụng có thùng/sàn chở hàng riêng. | `img_001.jpg`, `img_002.jpg` (xe tải chở ô tô), `img_037.jpg` | Xe van kín thân liền (`van`), xe buýt chở khách (`bus`). |
| `van` | Xe thùng kín cỡ nhỏ/trung (minivan, panel van, camper van) thân liền khối dùng chở người hoặc hàng nhẹ. | `img_005.jpg`, `img_010.jpg` | SUV (`car`), xe buýt cỡ lớn (`bus`), xe tải thùng rời (`truck`). |
| `bus` | Xe buýt công cộng hoặc xe khách cỡ lớn (1 tầng hoặc 2 tầng) thiết kế chở nhiều hành khách. | `img_009.jpg`, `img_031.jpg`, `img_037.jpg`, `img_040.jpg` | Xe van cỡ nhỏ (`van`), xe tải (`truck`). |
| `tuk_tuk` | Xe lam / xe ba bánh có động cơ và mái che chở khách hoặc hàng hóa. | `img_004.jpg`, `img_016.jpg`, `img_035.jpg` | Xe máy 2 bánh (`motorcycle`), xích lô/xe đạp không động cơ (loại trừ). |
| `motorcycle` | Xe gắn máy, mô tô 2 bánh có động cơ. | `img_003.jpg`, `img_007.jpg` | Xe đạp, xích lô đạp chân (`img_006.jpg`, `img_029.jpg`, `img_039.jpg`), người ngồi trên xe máy (`person`). |
| `person` | Người thật đang đi bộ, đứng, ngồi hoặc điều khiển/ngồi trên phương tiện. | `img_001.jpg`, `img_003.jpg` | Tượng (`img_011.jpg`, `img_017.jpg`), ma-nơ-canh (`img_033.jpg`), ảnh in trên biển quảng cáo (`img_018.jpg`), phản chiếu (`img_020.jpg`). |
| `ignore_region` | Vùng đám đông người/xe quá dày đặc chồng lấn không thể tách từng cá thể hoặc cụm vật ở xa cao `< 20 px`. | Đám đông xa ở hậu cảnh | Không dùng để lười gán vật rõ ràng có chiều cao `>= 20 px`. |

- **Vật không thuộc lớp nào (Out-of-taxonomy):** Xe đạp, xích lô đạp chân (`img_006.jpg`, `img_029.jpg`, `img_039.jpg`), xe kéo tay, tàu hỏa/tàu điện, con vật → **Không gán box phương tiện** (nhưng người thật đang lái/ngồi trên xe đạp/xích lô vẫn gán `person` bình thường).

### Thuộc tính
| Thuộc tính | Loại | Các mức cố định | Ngưỡng cụ thể và ý nghĩa |
|---|---|---|---|
| `occluded_level` | `select` | `0`, `1`, `2`, `3` | - `0` (Không che): Thấy `100%` vật (che `0%`).<br>- `1` (Che ít): Bị vật khác che khuất `1% – 30%` diện tích thực tế.<br>- `2` (Che vừa/nhiều): Bị vật khác che khuất `31% – 70%` diện tích.<br>- `3` (Che rất nặng): Bị che `> 70%` diện tích nhưng vẫn đủ dấu hiệu nhận diện chắc chắn lớp và có chiều cao phần nhìn thấy `>= 20 px`. (Chỉ tính bị vật khác che; bị cắt bởi mép ảnh dùng `truncated`). |
| `truncated` | `select` | `no`, `yes` | - `no`: Toàn bộ phần nhìn thấy của vật nằm trọn trong khung ảnh.<br>- `yes`: Một phần cơ thể/thân xe bị cắt bởi rìa/biên bức ảnh (kể cả chạm sát mép ảnh làm mất chi tiết). |
| `can_review` | `checkbox` | `false`, `true` | - `false` (mặc định): Tự tin theo guideline.<br>- `true` (tick): Ca mơ hồ về lớp, ranh giới box hoặc mức che khuất cần đưa ra họp nhóm xem lại. |

## 3. Quy tắc vẽ box
- **Box ôm sát phần nhìn thấy:** Chỉ vẽ bao quanh phần thực tế nhìn thấy được trên ảnh 2D, tuyệt đối không đoán và vẽ nối phần bị khuất sau vật cản.
- **Vật có nhiều phần (Người đi xe máy / xe đạp):** Tách riêng box `person` cho người và box `motorcycle` cho xe máy (mỗi thực thể một box độc lập, cho phép 2 box giao nhau).
- **Xe chở xe (`img_002.jpg`):** Gán 1 box `truck` cho xe tải/rơ-moóc chở xe và gán riêng từng box `car` cho mỗi ô tô con nằm trên sàn chở (nếu rõ ràng và cao `>= 20 px`).
- **Hàng hóa trên xe (`img_035.jpg`):** Box phương tiện chỉ ôm phần khung/thân xe gốc; không kéo box bao trùm phần hàng hóa chất cao quá khổ bên ngoài hoặc thân vỏ xe phế liệu đặt trên nóc (`img_024.jpg`).
- **Vật không thật (Tượng, ma-nơ-canh, đồ chơi, tranh ảnh, phản chiếu):** Tuyệt đối không gán nhãn cho tượng người/xe (`img_011.jpg`, `img_017.jpg`, `img_027.jpg`), ma-nơ-canh (`img_033.jpg`), mô hình đồ chơi (`img_038.jpg`), hình người trên poster/biển quảng cáo (`img_018.jpg`), hoặc bóng phản chiếu trong gương/kính (`img_020.jpg`).
- **Xe chủ (Ego vehicle):** Không gán phần đầu xe, gương chiếu hậu hoặc taplô của chính chiếc xe gắn camera chụp ảnh (`img_020.jpg`).
- **Vật quá nhỏ & đám đông:** Không gán box riêng lẻ cho vật có chiều cao phần nhìn thấy `< 20 px`. Nếu có cụm đám đông/xe ở xa dính chặt không tách được từng vật, bao cả cụm bằng 1 box `ignore_region`.

## 4. Ca khó
| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_002.jpg` | Xe tải chở 8 ô tô con trên 2 tầng sàn | R-05 | Gán 1 box `truck` + 8 box `car` | Các ô tô con là phương tiện thật, nhìn rõ, cần phát hiện đầy đủ. |
| `img_004.jpg`, `img_035.jpg` | Xe ba bánh tuk-tuk chở khách/hàng | R-02 | Đề xuất thêm lớp `tuk_tuk`; không gán bao tải chất cao trên nóc ở `img_035.jpg` | Tuk-tuk phổ biến trong tập ảnh nhưng chưa có trong v0; hàng hóa không phải cấu trúc xe. |
| `img_009.jpg`, `img_037.jpg` | Xe buýt cỡ lớn và bán tải gắn nắp thùng cao | R-02 | Thêm lớp `bus` cho xe buýt; bán tải nắp thùng cao ở `img_037.jpg` tick `can_review` để họp nhóm chốt `truck` vs `van` | Xe buýt khác hẳn `van`/`truck`; bán tải lắp thùng kín dễ gây bất đồng giữa 2 người gán. |
| `img_017.jpg`, `img_027.jpg` | Tượng đồng hình người và tượng xe bán tải | R-01 | Không gán (0 box) | Vật vô tri/tác phẩm điêu khắc, không phải người hay phương tiện thật. |
| `img_020.jpg` | Gương chiếu hậu xe chủ và ảnh phản chiếu trong gương | R-01 | Không gán (0 box) | Phản chiếu qua gương và bộ phận ego-vehicle không phải vật thể trực tiếp trong không gian 2D. |
| `img_033.jpg`, `img_038.jpg` | Ma-nơ-canh trong tiệm và xe mô hình đồ chơi | R-01 | Không gán (0 box) | Không phải người thật / phương tiện thật. |

## 5. QC và phiên bản
- **Ngưỡng QC áp dụng:** `min_box_height_px = 20`, `iaa_iou >= 0.85`, `iaa_class >= 0.95`, `error_rate_img_max <= 0.02` (tối đa 2% ảnh lỗi).
- **Ai quyết ca mơ hồ:** Đánh dấu `can_review` hoặc ghi vào `image_issues.csv`, sau đó cả nhóm (`156`, `160`) họp chốt ở Pha 2 và Pha 5.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| `v0` | Bản khởi đầu của giảng viên | Chưa có ngưỡng `occluded_level`, thiếu `bus`/`tuk_tuk`, chưa có quy tắc tượng/phản chiếu/xe chở xe. |
| `v0.1-draft` (`160`) | Bổ sung định nghĩa lớp `bus`, `tuk_tuk`, định lượng ngưỡng `occluded_level` theo %, quy tắc ca khó | Giải quyết các tình huống biên trong `img_002`, `img_004`, `img_009`, `img_017`, `img_020`, `img_027`, `img_033`, `img_035`, `img_037`, `img_038`. |
