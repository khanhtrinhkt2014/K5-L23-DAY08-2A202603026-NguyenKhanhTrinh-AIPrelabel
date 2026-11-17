# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603026 - Nguyễn Khánh Trình

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`? | **CHƯA CHỐT.** Trong 36 ảnh tôi không gặp xe nào xác định chắc là xe điện, nên không có ảnh ví dụ để chốt. Tạm thời: chọn lớp tin nhất theo dáng xe (có bàn đạp và khung xe đạp → `bicycle`; có sàn để chân, yếm xe → `motorbike`) và **luôn** tick `can_review`. | — | (chưa có) |
| 2 | Khối xe máy đỗ dày đặc **gần camera**: từng xe hay `ignore_region`? | Gần camera → **gán từng xe** nếu box cao ≥ 20 px và còn thấy ≥ 20% xe. `ignore_region` chỉ dùng cho cụm ≥ 5 vật cao < 20 px, và **không được phủ lên** vật cao ≥ 20 px. | img_026, img_006, img_035 | CAL-03, CAL-04 |
| 3 | Hàng nhô ra: box / mask xe có gồm hàng không? | Box **và mask** xe **không** gồm hàng nhô ra > 20% chiều dài xe. Hàng nằm trong thùng / giỏ gắn cố định **thuộc xe**. | img_031 (khay trái cây ngoài xe đạp), img_003 (hàng trong thùng ba gác) | CAL-05 |
| 4 | Người ngồi trong xe ba bánh có mui: ngưỡng "thấy ≥ 20%" có hợp lý? | **Giữ** ngưỡng 20% để gán `person`; ước lượng bằng mắt dễ lệch giữa hai người nên ca sát ngưỡng → `can_review`. Với **mask**: khung xe nằm trước người thuộc xe; phần người nhìn thấy thuộc `person`; khoang khách mở là lỗ lớn → bỏ khỏi mask xe. | img_013, img_002 | CAL-06 |
| 5 | Kính chắn gió / cửa sổ có thuộc mask xe? | Kính **thuộc** mask xe, kể cả khi nhìn xuyên thấy người. Người sau kính vẫn gán `person` riêng nếu thấy ≥ 20%. | img_013 | CAL-07 |
| 6 | Box máy chồng nhau nhiều trong ảnh đêm: gộp hay giữ? | **Không gộp.** Mỗi xe một box. Hai box máy cùng lớp trên cùng một xe (IoU ≥ 0.7) → giữ box ôm sát hơn, xoá box còn lại. | img_026 | CAL-08 |
| 7 | Có đặt mức `conf` tối thiểu để giữ box máy không cần sửa? | **Không đặt.** Box điểm cao vẫn sai (img_001 `truck` 0.88 trên ô tô; img_012 `car` 0.81 trên tuk-tuk), box điểm thấp vẫn đúng (xích lô 0.27, img_032). `conf` chỉ dùng để xếp thứ tự duyệt. | img_001, img_012, img_032 | CAL-09 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_012, img_020, img_011 | Tuk-tuk / xe ba bánh trong phố | YOLO26m gọi `car` (0.65–0.81) hoặc `truck` (0.31–0.77); hạ / tăng ngưỡng không đổi được nhãn | Luôn `three_wheeler` khi đếm được 3 bánh | Mục 5: thêm ca "máy gọi xe ba bánh là car / truck → `three_wheeler`" (CAL-01) |
| img_002, img_003 | Xe bán tải, quầy / xe hàng có mái | YOLOE-26s (prompt tuk-tuk / cyclo / cargo tricycle) gán `three_wheeler` (pickup 0.37); 24/40 box three_wheeler bị xoá khi duyệt | Đếm bánh: bán tải → `car`; không đếm được bánh → lớp tin nhất + `can_review` | Mục 4.3: box `three_wheeler` của máy phải đếm được 3 bánh (CAL-02) |
| img_032 | Mask xích lô có bánh nan hoa | SAM 2.1 base tô đặc cả bánh; click âm vào lòng bánh không khoét được | Bánh xe tính là một khối đặc trong mask | Mục 4.2.2: ngoại lệ bánh xe; các lỗ lớn khác (khoang khách, khung mở) vẫn phải bỏ (CAL-10) |
| img_035 | `ignore_region` phủ lên xe máy đỗ rõ, cao ~90 px | (lỗi của người duyệt, QC phát hiện) | Xoá vùng ignore ở đó, giữ box từng xe | Mục 2, dòng `ignore_region`: không phủ lên vật cao ≥ 20 px (CAL-04) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt 6/7 câu hỏi mở (câu 2–7); câu 1 (xe điện) chưa chốt vì chưa có ảnh ví dụ. Thêm 3 quy tắc mới: xe ba bánh luôn `three_wheeler` dù máy gọi car / truck; box `three_wheeler` phải đếm được 3 bánh; bánh xe là khối đặc trong mask. Siết `ignore_region`. | CAL-01 … CAL-10 |
