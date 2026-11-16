# Guideline gán nhãn — v0.1-draft — 156

> Bản dự thảo guideline cá nhân (Pha 1) của thành viên `156`.
> Gồm đủ **5 phần** theo mẫu chuẩn, độc lập khảo sát 40 ảnh (`img_001.jpg` – `img_040.jpg`) để tìm các điểm chưa rõ của `guideline_v0_starter.md`.

- Phiên bản: v0.1-draft (cá nhân `156`)
- Ngày: 2026-11-16
- Người viết / duyệt: `156`

## 1. Mục tiêu
Mô hình nào dùng nhãn này, để làm gì, sai ở đâu thì nguy hiểm:
- Dữ liệu dùng để huấn luyện mô hình Object Detection 2D nhận diện **người và phương tiện cơ giới đường bộ** phục vụ bài toán giám sát giao thông và xe tự hành.
- **Nguy hiểm nếu sai:**
  - Nếu gán cả tượng (`img_011.jpg`, `img_017.jpg`, `img_027.jpg`), biển quảng cáo (`img_018.jpg`), ma-nơ-canh (`img_033.jpg`) hoặc đồ chơi (`img_038.jpg`), mô hình sẽ học sai đặc trưng và báo động giả trên đường.
  - Nếu không thống nhất cách gán người đi xe máy hoặc xe tải chở ô tô (`img_002.jpg`), nhãn huấn luyện sẽ mâu thuẫn làm giảm độ chính xác định vị box (IoU).

## 2. Lớp và thuộc tính

### Lớp
| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Xe ô tô con từ 4–7 chỗ (sedan, hatchback, SUV, coupe). | `img_001.jpg`, `img_008.jpg` | Xe tải/bán tải (`truck`), xe van (`van`), mô hình ô tô đồ chơi (`img_038.jpg`). |
| `truck` | Xe tải chở hàng, xe bán tải (pickup), xe bồn, xe kéo rơ-moóc. | `img_001.jpg`, `img_002.jpg`, `img_027.jpg` (chỉ xe thật, không gán tượng) | Xe van thân liền (`van`), xe buýt (`bus`). |
| `van` | Xe tải thùng kín cỡ nhỏ hoặc minivan chở người/hàng có khoang liền với cabin. | `img_005.jpg`, `img_010.jpg` | SUV (`car`), xe buýt lớn (`bus`). |
| `bus` | Xe buýt nội đô, xe buýt 2 tầng, xe khách cỡ lớn. | `img_009.jpg`, `img_031.jpg`, `img_040.jpg` | Xe `van` cỡ nhỏ, xe `truck`. |
| `tuk_tuk` | Xe ba bánh có động cơ (auto-rickshaw / tuk-tuk) dùng chở khách hoặc hàng. | `img_004.jpg`, `img_016.jpg`, `img_035.jpg` | Xe máy 2 bánh (`motorcycle`), xích lô đạp chân (`img_006.jpg`, `img_029.jpg`). |
| `motorcycle` | Xe máy, xe tay ga, mô tô 2 bánh có động cơ. | `img_003.jpg`, `img_007.jpg` | Xe đạp (`img_039.jpg`), người điều khiển (`person`). |
| `person` | Con người thật ở mọi tư thế (đi bộ, đứng, ngồi, lái xe). | `img_001.jpg`, `img_006.jpg` | Tượng người (`img_011.jpg`, `img_017.jpg`), ma-nơ-canh (`img_033.jpg`), ảnh in (`img_018.jpg`). |
| `ignore_region` | Khu vực có nhiều người/phương tiện chồng lấp dày đặc hoặc quá nhỏ (`< 20 px`) không thể tách rời từng box. | Nhóm người/xe ở xa | Vật thể đơn lẻ nhìn rõ `>= 20 px`. |

- **Vật không thuộc lớp nào:** Xe đạp, xích lô không động cơ (`img_006.jpg`, `img_029.jpg`, `img_039.jpg`), xe đẩy tay → **Không gán phương tiện**, chỉ gán `person` cho người thật đang điều khiển hoặc ngồi trên đó.

### Thuộc tính
| Thuộc tính | Loại | Các mức cố định | Ngưỡng cụ thể và ý nghĩa |
|---|---|---|---|
| `occluded_level` | `select` | `0`, `1`, `2`, `3` | - `0`: Không bị che khuất (`0%`).<br>- `1`: Bị vật thể khác che nhẹ từ `1% – 30%` diện tích.<br>- `2`: Bị che trung bình từ `31% – 70%` diện tích.<br>- `3`: Bị che nặng `> 70%` diện tích (nhưng phần lộ ra vẫn cao `>= 20 px` và nhận biết rõ lớp). |
| `truncated` | `select` | `no`, `yes` | - `no`: Vật nằm hoàn toàn bên trong khung hình.<br>- `yes`: Vật bị cắt mất một phần bởi biên/mép bức ảnh. |
| `can_review` | `checkbox` | `false`, `true` | - Tick `true` khi gặp tình huống biên chưa chắc chắn về lớp hoặc cách vẽ box để nhóm thảo luận. |

## 3. Quy tắc vẽ box
- **Ôm sát phần nhìn thấy:** Box chỉ bao quanh phần diện tích nhìn thấy trực tiếp trên ảnh, không suy đoán phần khuất sau vật cản. Nếu vật bị chia thành nhiều mảnh rời do vật chắn ở giữa, vẽ 1 box bao toàn bộ các phần nhìn thấy của vật đó và đặt `occluded_level` tương ứng.
- **Người và xe máy (`person` + `motorcycle`):** Luôn vẽ 2 box riêng biệt: 1 box `person` ôm sát người lái (và mỗi người ngồi sau nếu có) và 1 box `motorcycle` ôm sát chiếc xe máy.
- **Xe chở xe (`img_002.jpg`):** Vẽ box `truck` cho xe tải lớn; với các ô tô con trên thùng xe, cần thống nhất khi họp nhóm xem gán từng box `car` hay chỉ gán xe tải (tạm thời đề xuất gán cả từng box `car`).
- **Tượng, hình in, ma-nơ-canh, đồ chơi, phản chiếu, xe chủ:** Loại trừ hoàn toàn (không vẽ box) đối với tượng (`img_011.jpg`, `img_017.jpg`, `img_027.jpg`), poster quảng cáo (`img_018.jpg`), phản chiếu trong gương và thân xe chủ (`img_020.jpg`), ma-nơ-canh (`img_033.jpg`), xe đồ chơi (`img_038.jpg`).
- **Kích thước tối thiểu:** Bỏ qua các đối tượng có chiều cao phần nhìn thấy `< 20 px` (hoặc gom vào `ignore_region` nếu tụ thành đám đông ở xa).

## 4. Ca khó
| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_002.jpg` | Xe đầu kéo chở 8 xe con trên 2 sàn | R-05 | Gán `truck` + từng xe `car` (đưa ra họp Pha 2 để chốt) | Cần thống nhất giữa 2 thành viên để tránh lệch `bo_sot_thua`. |
| `img_006.jpg`, `img_029.jpg` | Xích lô / xe đạp chở khách trên phố | R-02 | Chỉ gán `person`, không gán phương tiện | Xe đạp/xích lô không có động cơ, nằm ngoài taxonomy phương tiện cơ giới. |
| `img_011.jpg`, `img_017.jpg`, `img_027.jpg` | Tượng nghệ thuật hình người và tượng xe pickup | R-01 | Không gán box tượng; chỉ gán người/xe thật xung quanh nếu có | Tránh mô hình học nhầm tượng điêu khắc là đối tượng giao thông. |
| `img_018.jpg` | Khuôn mặt/người in trên biển quảng cáo lớn | R-01 | Không gán ảnh in; chỉ gán người thật đi dưới đường | Hình ảnh 2D trên biển quảng cáo không phải người thật. |
| `img_024.jpg`, `img_035.jpg` | Vỏ xe phế liệu trên nóc và hàng hóa chất cao trên tuk-tuk | R-07 | Chỉ ôm thân xe thật bên dưới, không gán vỏ xe phế liệu / bao tải hàng | Vỏ xe rỗng và bao tải hàng không phải là bộ phận cố định của phương tiện. |
| `img_037.jpg` | Xe bán tải `Master 75` lắp nắp thùng cao đi cạnh xe buýt | R-02 | Gán `bus` cho xe buýt; xe `Master 75` đánh dấu `can_review` để chốt `truck` hay `van` | Thùng sau có nắp cao dễ nhầm giữa `truck` (pickup) và `van`. |

## 5. QC và phiên bản
- **Ngưỡng QC áp dụng:** Giữ nguyên các ngưỡng chuẩn trong `taxonomy.json` (`min_box_height_px = 20`, `iaa_iou = 0.85`, `iaa_class = 0.95`, `error_rate_img_max = 0.02`).
- **Quy trình xử lý ca mơ hồ:** Ghi nhận qua `can_review` và họp nhóm thống nhất thành `guideline_v1.0_agreed.md`.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| `v0` | Bản gốc ban đầu | Chưa định nghĩa rõ ranh giới lớp và thuộc tính. |
| `v0.1-draft` (`156`) | Đề xuất thêm `bus`, `tuk_tuk`; định lượng `occluded_level`; lập danh sách ca loại trừ | Xử lý các ca khó phát hiện khi duyệt 40 ảnh (`img_002`, `img_006`, `img_011`, `img_018`, `img_024`, `img_035`, `img_037`). |
