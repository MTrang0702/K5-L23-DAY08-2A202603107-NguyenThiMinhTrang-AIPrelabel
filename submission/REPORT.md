# Báo cáo Day 08 — 2A202603107 - Nguyễn Thị Minh Trang

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | motorbike=0.1, các lớp khác mặc định 0.3 | `r02` conf=0.25 bắt tốt người (131) và xe máy (55); `r03` conf=0.5 làm mất 35% box. Quét ngưỡng trên gold cho thấy motorbike cần ngưỡng 0.1 để đạt 100% recall. |
| three_wheeler | YOLOE-26s, prompt: tuk-tuk, cyclo, cargo tricycle | 0.15 | `r04` bắt được 8 xe ba bánh mà YOLO26 bỏ sót; quét ngưỡng trên gold cho thấy 0.15 đạt recall 100% (precision 78%). |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "motorbike=0.1,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1 (`sam2.1_b.pt`) vì chất lượng viền mask sắc nét, phân tách tốt các chi tiết mui xe, bánh xe, dung lượng ~160 MB và chạy ổn định trên CPU máy cá nhân.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Lỗi YOLO26 gán xe ba bánh (tuk-tuk, ba gác) thành `car` hoặc `truck` do tập nhãn COCO 80 lớp không có khái niệm xe 3 bánh. Đổi ngưỡng không giải quyết được vì model luôn phân loại vào lớp sai với độ tin cậy cao. Sửa bằng cách dùng detector mở YOLOE-26s với prompt chuyên biệt và gộp bằng NMS ưu tiên nhãn `three_wheeler`.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào):
  - Điểm cao mà sai: `img_014.jpg` có biển quảng cáo/mái hiên bị máy gán `truck` với conf=0.82.
  - Điểm thấp mà đúng: `img_001.jpg` có xe máy ở xa conf=0.12 nhưng vẫn là xe thật nhìn rõ.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh, 50 đối tượng) nên số đo máy trên gold là lạc quan (hiện tượng in-sample bias).

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 426 | 68.6% | 621 box máy |
| edited | 40 | 6.4% | 621 box máy |
| deleted | 109 | 17.6% | 621 box máy |
| added | 28 | 5.7% | 495 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? Lớp `three_wheeler` tệ nhất (bị deleted 20 box, edited 9 box trên tổng 12 box được giữ), vì xe ba bánh có hình dáng đa dạng, máy thường bắt lỏng hoặc nhầm với ô tô nhỏ. Lớp `motorbike` ở các ảnh đông người (`img_024.jpg`) cũng bị sót nhiều do che khuất.
- So với gold: precision / recall của **máy**: Precision = 60.0% (mẫu số: 42/70 box máy), Recall = 84.0% (mẫu số: 42/50 đối tượng gold).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 8/12 = 66.7%; loại nào lọt: 1 loose (`img_025.jpg`), 1 wrong_class (`img_003.jpg`), 1 extra (`img_014.jpg`), 1 missing (`img_029.jpg`); vì sao (bạn đã bỏ bước nào của checklist)? Lọt do người duyệt bị cuốn theo các box máy có sẵn (Automation Bias), không ẩn nhãn máy để rà soát toàn cảnh trước, đặc biệt ở các ảnh có mật độ phương tiện dày đặc (`img_029.jpg`).
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Accepted đạt 68.6% trong khi bắt bẫy chỉ đạt 66.7% cho thấy người duyệt có xu hướng tin tưởng máy đề xuất và chấp nhận nhanh các box máy vẽ, lơ là kiểm tra các lỗi tinh vi như box lỏng và phân loại nhầm.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 6.25 phút/ảnh (25 phút / 4 ảnh) vs duyệt 2.71 phút/ảnh (65 phút / 24 ảnh). Thiên lệch (bạn gán gold trước nên đã quen cảnh): Có thiên lệch nhẹ do đã quen thuộc bối cảnh, nhưng tốc độ duyệt thực tế nhanh hơn gấp 2.3 lần so với gán tay hoàn toàn.
- Kết luận: CÓ ĐÁNG DÙNG, dựa trên số liệu tiết kiệm hơn 56% thời gian lao động và giữ lại được 68.6% nhãn chất lượng cao. Tuy nhiên bắt buộc phải có quy trình kiểm soát bẫy để tránh lọt lỗi.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Quy tắc 15 giây mù: Luôn tắt lớp hiển thị nhãn máy trong 15 giây đầu tiên khi mở ảnh để quét mắt tìm các đối tượng bỏ sót trước.
  2. Rà soát ngược conf: Lọc và duyệt các box có thuộc tính `conf < 0.4` trước vì đây là nhóm có xác suất lỗi sai lớp và box thừa cao nhất.
  3. Nghỉ thị giác định kỳ: Nghỉ giải lao 5 phút sau mỗi 8-10 ảnh duyệt liên tục để duy trì sự tập trung thị giác, tránh bỏ sót bẫy lỏng.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 pos, 1 neg) | Không gồm người lái | Viền mui xe bị lem nhẹ ở góc tối | `tuktuk_img013.jpg` |
| `img_020.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 pos, 1 neg) | Gồm cả mui xe, tách riêng người lái | Vùng trống giữa tay lái cần thêm click âm | `tuktuk_img020.jpg` |
| `img_002.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 pos, 1 neg) | Bao trọn thân xe | Bị dính một phần bóng đổ dưới gầm xe | `tuktuk_img002.jpg` |

- Đối chiếu guideline mục 4.2: Quy tắc không vẽ lỗ thủng < 5% rất hữu ích giúp SAM không phải tách các khe nhỏ trên khung xe; quy tắc tách riêng người lái đòi hỏi phải click âm vào vị trí người ngồi.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: Interactor trên CVAT yêu cầu serverless/Nuclio phức tạp; chạy SAM offline bằng script python giúp kiểm soát tọa độ click điểm chính xác và tái lặp kết quả hoàn toàn.

## 6. Calibration (Part 6)

- Số quyết định: 3 (`CAL-01`, `CAL-02`, `CAL-03`); quan trọng nhất (dẫn `CAL-xx`): `CAL-01` (chốt phân biệt xe đạp điện có bàn đạp và xe máy điện không bàn đạp) và `CAL-02` (sửa lỗi máy nhầm xe buýt thành xe tải).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: Đã bổ sung chi tiết vào mục 1 (câu hỏi mở) và mục 2 (ca khó mới).
- Nguyên nhân gốc: Kết hợp giữa guideline v1.0 chưa bao quát hết các phương tiện giao thông đặc thù Đông Nam Á và giới hạn tập nhãn của mô hình đóng YOLO26.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: 0/96 = 0.0% · theo **ảnh**: 0/5 = 0.0% (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: Mẫu kiểm tra 5 ảnh đạt chất lượng tốt, không ghi nhận lỗi tồn đọng lớn nào trên các đối tượng chính.
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): Cận trên 95% khoảng tin cậy Wilson của tỷ lệ ảnh lỗi thực tế là 43.0% do cỡ mẫu chỉ có 5 ảnh.
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): Script chỉ kiểm tra định dạng hình học và đối chiếu với phiếu ghi nhận của con người; không tự động đánh giá được độ ôm sát quang học nếu không có nhãn ground truth chuẩn.

**Quyết định: HOLD**, ngày 2026-11-18. Lý do (đi theo Hold → Rework → Release): Mặc dù tỷ lệ lỗi trong mẫu kiểm tra 5 ảnh là 0%, nhưng tỷ lệ bắt bẫy trong Part 4 chỉ đạt 66.7% (< 80%), cho thấy có nguy cơ thiên kiến tự động hóa lọt lỗi trong các ảnh còn lại. Cần đưa vào trạng thái HOLD để hoàn thiện rà soát.
Nếu Rework / Hold, việc cần làm tiếp: Rà soát lại 4 ảnh có bẫy lọt (`img_025.jpg`, `img_003.jpg`, `img_014.jpg`, `img_029.jpg`), áp dụng quy tắc từ guideline v1.1 và tiến hành cross-review chéo với thành viên khác.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Dùng mô hình YOLO26m, YOLOE-26s, SAM 2.1 theo đúng chỉ dẫn của giáo trình để thực hiện pre-labeling và segmentation.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không, 4 ảnh gold được gán tay hoàn toàn độc lập từ đầu trên CVAT trước khi chạy bất kỳ mô hình AI nào.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11, CPU (không GPU), CVAT v2.81.0 chạy qua Docker Compose. Quá trình chạy CPU ban đầu mất khoảng 1-2 giây/ảnh cho YOLO và ~8.8s/ảnh cho SAM.
- Dùng cache `data/cache/` thay vì tự chạy? Không, tự chạy 100% các script inference trên máy.
- Lỗi / bug script đã gặp (nếu có): Quá trình tải clip cho YOLOE yêu cầu kết nối mạng tải thêm trọng số clip.
- Một điều bạn làm khác lần sau: Sẽ thực hiện nghiêm ngặt checklist 4 bước, luôn ẩn nhãn máy ở những ảnh đầu để rèn luyện thói quen chống thiên kiến tự động hóa.
