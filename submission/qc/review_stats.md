# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 621; đối tượng sau duyệt: 495; ngưỡng accepted: IoU >= 0.9
- Đã **loại 25 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 426 | 426/621 = 68.6% | 621 box máy |
| edited (sửa) | 40 | 40/621 = 6.4% | 621 box máy |
| deleted (xoá) | 109 | 109/621 = 17.6% | 621 box máy |
| added (vẽ thêm) | 28 | 28/495 = 5.7% | 495 đối tượng sau duyệt |

Trong số deleted + added, **3 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 25.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 218 | 9 | 30 | 9 |
| bicycle | 5 | 1 | 2 | 3 |
| motorbike | 139 | 19 | 47 | 9 |
| car | 46 | 2 | 7 | 0 |
| bus | 3 | 0 | 0 | 1 |
| truck | 3 | 0 | 3 | 0 |
| three_wheeler | 12 | 9 | 20 | 6 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 50 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 42 | 28 | 8 | 60.0% (KTC 95%: 48–71%) | 84.0% (KTC 95%: 71–92%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 8/12 = 66.7% (KTC 95%: 39–86%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 2/3
  - wrong_class: 2/3
  - extra: 2/3
  - missing: 2/3
  - lọt: img_025.jpg (loose), img_003.jpg (wrong_class), img_014.jpg (extra), img_029.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
