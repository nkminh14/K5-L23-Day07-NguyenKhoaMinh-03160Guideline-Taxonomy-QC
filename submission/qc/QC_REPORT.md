# Báo cáo QC — Group-156-160

Báo cáo kiểm soát chất lượng (QC) và bàn giao lô nhãn 40 ảnh đường phố của nhóm `Group-156-160` (thành viên `156`, `160`).

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Huấn luyện và đánh giá mô hình phát hiện đối tượng 2D (người và phương tiện cơ giới đường bộ) trong bài thực hành đặc tả và QC nhãn Day 07; bộ 40 ảnh được tuyển chọn để rèn luyện xử lý tình huống biên nên độ đại diện cho phân phối vận hành thực tế chưa được đánh giá.
- Nguồn và phép sử dụng dữ liệu: Toàn bộ 40 ảnh (`img_001.jpg` – `img_040.jpg`) có nguồn gốc và giấy phép ghi đầy đủ tại `data/ATTRIBUTION.csv`; nhóm giữ nguyên thông tin tác giả và giấy phép gốc, chỉ sử dụng cho mục đích học tập.
- Manifest và checksum nhãn cuối: Hồ sơ `submission/qc/batch_manifest.json`, nhãn cuối `submission/qc/labels_calibrated.json` có mã băm SHA-256 là `7bdabe2511f5b5b856a1824769cd3f1f49df9825093cb40da00106309efed168` (bản CSV `submission/qc/labels_calibrated.csv` có SHA-256 `d47e97615513672958cf89980167059a6875d7d5ccf8f7b92dab3bb5c8cf2b2d`).
- Guideline và taxonomy khi gán / khi bàn giao: Khi gán độc lập ở Pha 3 dùng `submission/guidelines/guideline_v1.0_agreed.md` và `submission/taxonomy.json` (`version: v1.0`, 8 lớp); khi bàn giao ở Pha 5–6 dùng `submission/guidelines/guideline_v1.1_calibrated.md` (cập nhật `editorial` theo `submission/guideline_change.json`, giữ nguyên schema `submission/taxonomy.json` `v1.0`).

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: `80/80` lượt gán (`40` ảnh × `2` thành viên `156` và `160` gán độc lập theo `submission/qc/assignment.csv`).
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: `210` cặp box ghép (mẫu số `210` cặp có `IoU >= 0.5`), IoU trung bình đạt `0.946` (ngưỡng `>= 0.85`), đồng thuận lớp đạt `99.5%` (`209/210` cặp, ngưỡng `>= 95%`), đồng thuận `occluded_level` và `truncated` đạt `100.0%` (`210/210`) — ĐẠT chuẩn IAA.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: Box chưa ghép (A/B) là `1/1` (`C004` trên `img_018.jpg` của `156` và `C002` trên `img_002.jpg` của `160`), số box bỏ qua do `ignore_region` là `0`, số ca bất đồng vùng `ignore_region` là `0`.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: Tượng đồng người cưỡi ngựa ở `img_017.jpg` và tượng xe bán tải ở `img_027.jpg` (nếu cả hai cùng vẽ `person` hoặc `truck` thì IAA vẫn báo đồng thuận `100%` nhưng vi phạm quy tắc `R-01`; nhóm đã kiểm tra đối chiếu ảnh gốc và xác nhận cả hai thành viên đều để `0` box đúng quy tắc `R-01`).

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: Đã xử lý toàn bộ `8/8` ca (`C001` – `C008` trên 6 ảnh `img_001.jpg`, `img_002.jpg`, `img_003.jpg`, `img_018.jpg`, `img_035.jpg`, `img_037.jpg`) trong `submission/qc/calibration_decisions.csv` theo các quy tắc `R-01`, `R-02`, `R-03`, `R-07`; người chốt là `160`, bản sửa lưu tại `submission/qc/labels_calibrated_757ecd0bf2f8ccab.json`, người kiểm lại xác nhận đạt là `156`.
- Tác động v1.1 lên nhãn và ảnh: Thay đổi thuộc loại `editorial` (làm rõ câu chữ và ví dụ minh họa ở `R-01`, `R-02`, `R-03`, `R-07`, không đổi nghĩa quy tắc hay cấu trúc lớp), phạm vi ảnh hưởng là `anh` trên đúng 6 ảnh bị gắn cờ (`img_001.jpg`, `img_002.jpg`, `img_003.jpg`, `img_018.jpg`, `img_035.jpg`, `img_037.jpg`), tổng hợp thành `40` ảnh với `215` annotation (`210` đối tượng + `5` vùng `ignore_region`).
- Tham chiếu bằng chứng khi Rework: Không phát sinh Rework (vòng 1 đạt chuẩn Release ngay sau khi hoàn tất Calibration ở Pha 5).

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: `156` (Vòng 1 thực hiện ngày `2026-11-16`, đổi vai độc lập với người điều phối chốt sửa `160`).
- Đã duyệt / cả lô: `40/40` ảnh (chế độ `full` toàn lô).
- Ảnh lỗi / ảnh đã duyệt: `0/40 = 0.0%` (ngưỡng tối đa `<= 2.0%`).
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: `0/210 = 0.0%` (ngoài ra có `5` vùng `ignore_region`, tổng `215` annotation).
- Vật bỏ sót, ảnh và vùng: `0` vật bỏ sót trên toàn bộ 40 ảnh.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Nhóm `ca_kho` đã duyệt `6/6` ảnh (`0/6 = 0.0%` lỗi), nhóm `con_lai` đã duyệt `34/34` ảnh (`0/34 = 0.0%` lỗi), không có lỗi lặp lại; chi tiết từng ảnh ghi tại `submission/qc/reviews/round-01/qc_review.csv` và `submission/qc/reviews/round-01/qc_report_auto.md`.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Máy (`validate_labels.py`, `build_final.py`) tự động chặn được lỗi tọa độ bbox vượt biên ảnh, thiếu ảnh trong phân công hoặc tên lớp lạ ngoài `taxonomy.json`; nhưng lỗi vẽ box `tuk_tuk` bao trùm luôn khối bao tải hàng hóa trên nóc (`img_035.jpg` — `C005`) hoặc gán nhầm tượng đồng (`img_017.jpg`) thì máy không tự biết, bắt buộc người review phải nhìn ảnh đối chiếu quy tắc `R-07` và `R-01`.

## 5. Quyết định bàn giao

- Quyết định: Release
- Lý do, người nhận và phạm vi sử dụng: Lô nhãn đã hoàn thành `80/80` lượt gán độc lập, IAA đạt `0.946` (`210` cặp ghép), giải quyết triệt để `8/8` ca Calibration, và vượt qua vòng duyệt QC toàn lô `40/40` ảnh với `0/40 = 0.0%` ảnh lỗi (`0/210 = 0.0%` đối tượng lỗi, `0` bỏ sót, `0` cờ `can_review`); bàn giao cho Coach / Giảng viên lớp Day 07 để sử dụng trong phạm vi huấn luyện và đánh giá mô hình phát hiện vật 2D của học phần.
- Nếu còn việc: Không còn hạng mục tồn đọng (toàn bộ 40 ảnh và hồ sơ truy vết đã hoàn tất 100%).
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Truy vết ca `C005` (`img_035.jpg`) → trong `submission/qc/calibration_candidates.csv` ghi nhận bất đồng `iou_thap` (`0.57`) giữa box `[380,95,175,320]` của `156` và `[394,204,152,211]` của `160` → trong `submission/qc/calibration_decisions.csv` người chốt `160` áp dụng quy tắc `R-07` tại `submission/guidelines/guideline_v1.1_calibrated.md` (không bao hàng hóa quá khổ trên nóc) để chốt box `[394,204,152,211]` → kiểm tra trong `submission/qc/labels_calibrated.json` (`image_id: 35`, annotation `id: 185`, lớp `tuk_tuk`) và ảnh overlay `work/review/round-01/img_035.jpg.png` (`#185 tuk_tuk`) khớp chính xác tọa độ `[394.0, 204.0, 152.0, 211.0]`.
