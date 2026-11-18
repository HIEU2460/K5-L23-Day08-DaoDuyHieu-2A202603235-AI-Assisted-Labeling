# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603235 - Dao Duy Hieu

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 4 | Người trong xe ba bánh có mui che: ngưỡng nhìn thấy bao nhiêu? | Giữ ngưỡng của mục 4.1.2: gán `person` riêng nếu còn nhìn thấy ít nhất khoảng 20% cơ thể; box chỉ ôm phần nhìn thấy, không đoán phần bị che. Dưới ngưỡng thì không gán. | `img_013` | `CAL-02` |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_004`, `img_013` | Tuk-tuk có hình dáng gần ô tô nhỏ hoặc xe tải nhỏ | YOLO26 gán tuk-tuk thành `car`; SAM tạo mask theo box nhưng không sửa được sai lớp | Khi thấy cấu trúc xe ba bánh cùng khung hoặc mui gắn liền, gán `three_wheeler`; người nhìn thấy là `person` riêng | Bổ sung mục 2: cấu trúc ba bánh quyết định lớp, không giữ `car`/`truck` chỉ vì nhãn máy (`CAL-01`) |
| `img_005` | Box đúng lớp nhưng lỏng | Bẫy `loose` lọt qua vòng review đầu | Kiểm lần lượt cạnh trái, phải, trên, dưới; box phải chạm phần nhìn thấy trong sai số khoảng 2 px | Bổ sung checklist mục 4.1: kiểm bốn cạnh trước khi chuyển ảnh (`CAL-03`) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt ngưỡng người bị che trong tuk-tuk; làm rõ cách phân biệt tuk-tuk với `car`/`truck`; thêm bước kiểm bốn cạnh box | `CAL-01`, `CAL-02`, `CAL-03` |
