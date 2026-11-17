# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 365; đối tượng sau duyệt: 450; ngưỡng accepted: IoU >= 0.9
- Đã **loại 24 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 314 | 314/365 = 86.0% | 365 box máy |
| edited (sửa) | 18 | 18/365 = 4.9% | 365 box máy |
| deleted (xoá) | 33 | 33/365 = 9.0% | 365 box máy |
| added (vẽ thêm) | 118 | 118/450 = 26.2% | 450 đối tượng sau duyệt |

Trong số deleted + added, **8 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 110.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 179 | 5 | 5 | 39 |
| bicycle | 6 | 0 | 1 | 3 |
| motorbike | 81 | 6 | 3 | 43 |
| car | 38 | 0 | 0 | 22 |
| bus | 1 | 0 | 1 | 2 |
| truck | 2 | 0 | 4 | 3 |
| three_wheeler | 7 | 7 | 19 | 6 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 46 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 42 | 16 | 4 | 72.4% (KTC 95%: 60–82%) | 91.3% (KTC 95%: 80–97%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 8/12 = 66.7% (KTC 95%: 39–86%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 2/3
  - loose: 2/3
  - wrong_class: 3/3
  - extra: 1/3
  - lọt: img_005.jpg (loose), img_033.jpg (extra), img_024.jpg (missing), img_022.jpg (extra)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
