# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603107 - Nguyễn Thị Minh Trang

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện: bicycle hay motorbike? | Phân loại dựa trên cơ cấu bàn đạp: xe có bàn đạp hỗ trợ chân đạp xếp vào `bicycle`; xe chạy điện thuần túy không có bàn đạp (kiểu dáng scooter) xếp vào `motorbike`. | `img_036.jpg` | `CAL-01` |
| 4 | Người ngồi trong xe ba bánh có mui che: thấy thấp thoáng có gán không? | Bắt buộc vẽ 1 box `person` riêng cho tài xế/hành khách nếu phần nhìn thấy (đầu, thân hoặc tay chân) $\ge 20\%$ diện tích cơ thể người. Box `person` vẽ đè lên box `three_wheeler`. | `img_012.jpg` | `CAL-03` |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_020.jpg` | Xe buýt đô thị ở xa nhìn từ góc sau | YOLO26m gán nhầm thành `truck` do đuôi xe vuông và khuất một phần | Đổi nhãn thành `bus` | Mục 2: Xe buýt/xe khách có khối điều hòa đặc trưng trên nóc và dãy cửa kính hông luôn gán `bus`, không nhầm với `truck`. |
| `img_036.jpg` | Xe máy chở 2 người ngồi sát nhau | YOLO26m gộp chung cả 2 người vào 1 box `person` duy nhất | Tách thành 2 box `person` độc lập cho người lái và người ngồi sau | Mục 4.1: Mỗi cá thể người trên xe máy/xe đạp là 1 box `person` riêng biệt, không gộp chung. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt quy tắc xe đạp điện vs xe máy điện; quy tắc gán tài xế tuk-tuk trong khoang lái; khắc phục nhầm lẫn bus vs truck và gộp người đi xe máy. | Khắc phục các câu hỏi mở mục 6 và lỗi máy thực tế (`CAL-01`, `CAL-02`, `CAL-03`). |
