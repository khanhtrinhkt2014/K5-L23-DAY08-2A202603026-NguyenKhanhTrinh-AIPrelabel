# Báo cáo Day 08 — <<MSHV - Họ và tên>>

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m (`yolo26m.pt`) | motorbike **0.1**; person, bicycle, car, bus, truck: mặc định **0.3** (`--conf 0.3`) | YOLO26 bắt tốt người / xe máy / ô tô / bus gần camera (r01). Ngưỡng chung 0.5 làm mất 76/215 box, chủ yếu xe máy và người ở xa (r02), nên chọn ngưỡng theo lớp bằng `threshold_sweep`: **motorbike 0.1** đạt recall 7/7 = 100% (precision 7/16 = 44%); ở 0.3 chỉ 6/7 = 86%. Tôi chấp nhận 9 box thừa vì box thừa dễ thấy và xoá nhanh, còn vật bị sót khó phát hiện khi duyệt. **person**: không ngưỡng nào ≤ 0.5 đạt 90% (tối đa 16/18 = 89% ở 0.1–0.15, kèm 16 box thừa) → giữ 0.3 (recall 15/18 = 83%, precision 15/25 = 60%); 3 người sót phải tự tìm khi duyệt. **car**: recall đứng ở 7/8 = 88% ở mọi ngưỡng 0.1–0.5 → hạ ngưỡng không cứu được chiếc bị sót, giữ 0.3 (precision 7/13 = 54%). **bicycle** (2 vật), **bus** (1), **truck** (0): quá ít mẫu (< 5), không chọn ngưỡng riêng, dùng 0.3. |
| three_wheeler | YOLOE-26s (`yoloe-26s-seg.pt`), prompt: `tuk-tuk`, `cyclo`, `cargo tricycle` | **0.15** | YOLO26 không có lớp xe ba bánh: tuk-tuk / xích lô bị sót hoặc thành car / truck / bicycle (r01, r02). YOLOE tìm được tuk-tuk mà YOLO26 bỏ sót (img_002, 0.93) và xích lô (img_032) (r03). Ngưỡng 0.15: recall 7/7 = 100%, precision 7/9 = 78% trên gold; lên 0.3 thì recall rơi còn 5/7 = 71% vì một số xe ba bánh đúng chỉ có điểm 0.15–0.3. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "motorbike=0.1,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json` (file YOLOE đứng trước để box `three_wheeler` thắng box car / truck của YOLO26 khi chồng nhau, IoU ≥ 0.6).
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: **SAM 2.1 base (`sam2.1_b.pt`)**. Mask bám sát người và thân xe, máy 16 GB RAM chạy được trên CPU với 5.67 giây/ảnh (r04); chậm gấp ~7 lần YOLO26 (0.81 giây/ảnh) nhưng chấp nhận được vì Part 5 chỉ làm 3 vật. Không dùng SAM 3 vì cần quyền truy cập riêng và GPU.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? **Xe ba bánh bị YOLO26 gọi sai lớp**: trên gold, img_012 có 4 tuk-tuk bị gọi `car` với điểm 0.65–0.81, img_020 có tuk-tuk thành `car` 0.39 và `truck` 0.31; trên pilot, img_011 tuk-tuk thành `truck` 0.77, img_032 xích lô thành `bicycle` 0.79 (r01). Đây cũng là nguồn chính của box `car` thừa trong `threshold_sweep` (precision car chỉ 54%). Đổi ngưỡng chỉ làm box hiện ra / biến mất, không đổi nhãn. Cách sửa: chạy YOLOE riêng cho `three_wheeler` và đặt file YOLOE đầu tiên trong `merge_prelabels` để box three_wheeler thay box car / truck chồng lên; chỗ nào còn sót thì sửa tay khi duyệt. Ngược lại, YOLOE cũng gọi nhầm pickup thành `three_wheeler` (img_002, 0.37) → khi duyệt phải kiểm kỹ xe bán tải / xe chở hàng.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): Điểm cao mà sai: img_001 ô tô (gold: car) bị gọi `truck` 0.88; img_012 tuk-tuk thành `car` 0.81; img_011 tuk-tuk thành `truck` 0.77; img_032 xích lô thành `bicycle` 0.79. Điểm thấp mà đúng: YOLOE img_011 tuk-tuk mép trái 0.28 và img_032 xích lô 0.27 (r03); trên gold, 2/7 xe ba bánh chỉ được tìm thấy khi hạ ngưỡng xuống ≤ 0.15–0.25. Vì vậy "điểm 0.8" không có nghĩa là "đúng 80%".
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Gold chỉ có 4 ảnh, 43 vật (không tính 4 `ignore_region`); tôi chọn ngưỡng và chấm trên cùng dữ liệu nên precision / recall ở trên là **lạc quan**. Ngưỡng motorbike và three_wheeler dựa trên 7 vật mỗi lớp: lệch 1 vật là lệch 14 điểm recall, nên đây chỉ là ước lượng thô. bicycle / bus / truck không đủ mẫu để kết luận. Muốn số đo khách quan cần một tập kiểm tra riêng (bài nâng cao: pilot).

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 397 | 397/618 = 64.2% | 618 box máy |
| edited | 84 | 84/618 = 13.6% | 618 box máy |
| deleted | 96 | 96/618 = 15.5% | 618 box máy |
| added | 52 | 52/542 = 9.6% | 542 đối tượng sau duyệt |

(Đã loại 30 đối tượng quanh 12 bẫy. Trong deleted + added có 9 trường hợp là **đổi lớp**, box đúng chỗ nhưng máy gán sai lớp, nên số vật máy **sót thật** ≈ 52 − 9 = 43.)

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)?
  - **three_wheeler tệ nhất**: chỉ 9/40 box máy được giữ nguyên (22.5%), **24/40 = 60% bị xoá**. Ngưỡng 0.15 chọn để đạt recall 100% trên gold, nên YOLOE bắn nhiều box: nhiều box trùng trên cùng một xe và nhiều box ôm nhầm xe khác (ô tô, pickup, xe chở hàng, xe máy kéo hàng). Box bị xoá dồn vào img_003 (6 box), img_010 (3), img_014 (3), img_013, img_018 và img_023 (2 mỗi ảnh). Đổi lại, máy chỉ sót 4 xe ba bánh (added = 4): ngưỡng thấp đổi box thừa lấy recall cao, đúng như ý định ở mục 1.
  - **motorbike tốn công nhất**: trong 205 box máy, 54 phải sửa (26%) và 43 phải xoá (21%), chỉ 108 giữ nguyên (53%). Box phải sửa dồn vào ảnh dày xe máy: img_026 (9), img_018, img_021, img_034 (6 mỗi ảnh), img_029, img_035 (5). Ở đó xe chồng lên nhau nên box ôm lẫn người hoặc xe bên cạnh. Box bị xoá phần lớn là xe xa, điểm thấp, được giữ lại vì ngưỡng motorbike 0.1.
  - **person tốt nhất**: 227/271 box máy giữ nguyên (84%). Nhưng người cũng là lớp máy sót nhiều nhất về số lượng (added = 23), chủ yếu người bị che hoặc ở xa.
  - car: 45/50 giữ nguyên (90%), sót 8. bus và truck quá ít box (3 và 4) để kết luận.
- So với gold: precision / recall của **máy**: **precision 38/65 = 58.5%** (KTC 95%: 46–70%), **recall 38/43 = 88.4%** (KTC 95%: 76–95%). Mẫu số: 43 đối tượng gold trên 4 ảnh, không tính 4 `ignore_region`. Máy bỏ sót 5/43 vật nhưng có 27 box thừa: cứ khoảng 5 box máy thì 2 box sai. Con số này **lạc quan** vì ngưỡng được chọn trên chính 4 ảnh gold; khoảng tin cậy rộng (recall có thể chỉ ~76%), nên không được bỏ bước tự tìm vật sót khi duyệt.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: <<.../12>>; loại nào lọt: <<>>; vì sao (bạn đã bỏ bước nào của checklist)? <<>>
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? <<>>

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay <<... phút/ảnh>> vs duyệt <<... phút/ảnh>>. Thiên lệch (bạn gán gold trước nên đã quen cảnh): <<>>
- Kết luận: <<có / không / chỉ cho lớp nào>>, dựa trên <<số liệu>>
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. <<>>
  2. <<>>
  3. <<>>

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| <<img_0xx>> | <<>> | <<>> | <<>> | <<>> | <<>> |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? <<>>
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: <<>>

## 6. Calibration (Part 6)

- Số quyết định: <<>> (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): <<>>
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: <<>>
- Nguyên nhân gốc: <<guideline mơ hồ / chưa nắm quy tắc / máy>>

## 7. QC và quyết định (Part 6)

- Mẫu: <<n>> ảnh (<<m>> ảnh `can_review` + <<k>> ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: <<.../... = ...%>> · theo **ảnh**: <<.../... = ...%>> (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: <<>>
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): <<>>
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): <<>>

**Quyết định: <<Release / Hold / Rework>>**, ngày <<YYYY-MM-DD>>. Lý do (đi theo Hold → Rework → Release): <<>>
Nếu Rework / Hold, việc cần làm tiếp: <<>>

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): <<>>
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? <<không / có, như sau: ...>>
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: <<>>
- Dùng cache `data/cache/` thay vì tự chạy? <<không / có, vì ...>>
- Lỗi / bug script đã gặp (nếu có): <<prelabel.py gán run_id theo số dòng hiện có trong runs.csv và xóa dòng trùng --out. Tôi chạy lại YOLOE (lần đầu phải cài CLIP) nên runs.csv mất r03 và có hai r04. Trong model_log.csv tôi đánh số lại r01–r04; số liệu giữ nguyên từ runs.csv, dòng YOLOE là của lần chạy lại.>>
- Một điều bạn làm khác lần sau: <<>>
