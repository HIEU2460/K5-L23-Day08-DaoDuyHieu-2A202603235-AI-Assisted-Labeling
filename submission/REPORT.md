# Báo cáo Day 08 — 2A202603235 - Đào Duy Hiếu

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | person=0.5, motorbike=0.5, car=0.5; bicycle/bus/truck dùng 0.3 | `r01`, `r02`: YOLO26 tìm được nhiều lớp giao thông phổ biến. Ngưỡng 0.5 giảm box thừa nhưng cũng làm mất vật ở xa. Các lớp có dưới 5 vật gold giữ ngưỡng mặc định. |
| three_wheeler | YOLOE-26s, prompt `tuk-tuk`, `cyclo`, `cargo tricycle` | 0.2 | `r03`: tìm được tuk-tuk chính ở `img_002`, nhưng vẫn có box nhầm xe máy và pickup nên phải review. |

- Lệnh `merge_prelabels.py` đã dùng: `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.5,motorbike=0.5,car=0.5,three_wheeler=0.2" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM đã chọn:** SAM 2.1 base (`sam2.1_b.pt`). Bản này chạy được trên Colab CPU và mask bám được biên chính của người và xe, dù chạy chậm hơn detector (`r04`: 20.25 giây/ảnh).
- Lỗi không sửa được bằng đổi ngưỡng: YOLO26 gọi tuk-tuk là `car` ở `img_004` và `img_013`. Đây là sai lớp do tập lớp của model, nên tôi dùng YOLOE cho `three_wheeler` và sửa lại khi review.
- Ở `img_002`, YOLOE có box tuk-tuk đúng với điểm 0.93 nhưng cũng có box điểm 0.37 nhầm pickup/xe máy. Ở `img_011`, một số người và xe thật có điểm thấp và biến mất khi tăng ngưỡng từ 0.25 lên 0.5.
- Giới hạn: chỉ có 4 ảnh gold và ngưỡng được chọn rồi chấm trên chính 4 ảnh đó. Vì vậy precision/recall trên gold có xu hướng lạc quan, nhất là với lớp ít vật.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 314 | 314/365 = 86.0% | 365 box máy |
| edited | 18 | 18/365 = 4.9% | 365 box máy |
| deleted | 33 | 33/365 = 9.0% | 365 box máy |
| added | 118 | 118/450 = 26.2% | 450 đối tượng sau duyệt |

- `three_wheeler` là lớp tệ nhất: chỉ 7 accepted, trong khi 7 box phải sửa và 19 box bị xóa. Ví dụ `img_002`, YOLOE tìm đúng tuk-tuk chính nhưng cũng tạo false positive trên xe máy và pickup.
- So với gold, máy có precision 42/(42+16) = 72.4% và recall 42/46 = 91.3% (mẫu số recall: 46 đối tượng gold).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 8/12 = 66.7%. Bốn bẫy lọt là `img_005` (loose), `img_033` (extra), `img_024` (missing), `img_022` (extra). Tôi làm tốt bẫy sai lớp 3/3 nhưng chỉ bắt được extra 1/3. Nguyên nhân là có lúc nhìn theo box sẵn có và chưa quét lại toàn ảnh hoặc kiểm từng box có thật sự cần giữ không.
- Accepted 86.0% nhưng bắt bẫy chỉ 66.7% cho thấy accepted cao không đủ chứng minh review tốt. Tôi vẫn bị ảnh hưởng bởi nhãn máy và bỏ qua một số lỗi trông có vẻ hợp lý.

## 4. Máy có đáng dùng không?

- Thời gian từ `time_log.csv`: gán tay 15/4 = 3.75 phút/ảnh; duyệt 72/24 = 3.00 phút/ảnh. Review nhanh hơn 0.75 phút/ảnh, khoảng 20%. So sánh này chưa hoàn toàn công bằng vì sau khi gán gold tôi đã quen taxonomy và thao tác CVAT hơn.
- Kết luận: pre-label có ích để tạo điểm bắt đầu cho `person`, `motorbike`, `car`, nhưng chưa thể dùng thẳng. Máy vẫn bỏ sót khoảng 110 vật thật và `three_wheeler` cần sửa nhiều.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Với mỗi ảnh, ẩn annotation và quét từ trái sang phải trước khi nhìn box máy; đánh dấu đã làm bước này trong ghi chú review.
  2. Hiện annotation rồi kiểm từng box theo thứ tự: vật có thật, đúng lớp, sau đó kiểm bốn cạnh. Không chuyển ảnh khi chưa hết danh sách.
  3. Sau mỗi 10 ảnh, chọn lại 1 ảnh vừa làm để kiểm lần hai; theo dõi số box thêm/xóa thay vì chỉ nhìn tỷ lệ accepted.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013` tuk-tuk | SAM 2.1 base | 3 (2 dương, 1 âm) | Có lấy một phần người lái/hành khách và vùng bên trong xe | Biên ngoài xe khá sát nhưng mask dính người và có các lỗ nhỏ trong khung | `tuktuk_img013.jpg` |
| `img_004` tuk-tuk | SAM 2.1 base | 3 (2 dương, 1 âm) | Không lấy rõ người; chủ yếu là thân xe | Mask chọn đúng tuk-tuk giữa ảnh, còn một ít vùng tối/hở ở gầm và khung | `tuktuk_img004.jpg` |
| `img_013` người lái | SAM 2.1 base | 4 (2 dương, 2 âm) | Chỉ chọn người lái, không tính xe là một phần người | Mask bám khá sát thân và chân nhưng vùng tiếp xúc với ghế/khung vẫn cần nhìn lại | `driver_img013.jpg` |

- Mục 4.2 giúp xác định bóng đổ không thuộc vật, kính thuộc xe và hai vật chạm nhau không được chồng mask. Phần còn khó là người ngồi trong khoang hở và các lỗ lớn của khung xe; các vùng này làm SAM dễ lấy cả người vào mask xe.
- Không thử Interactor SAM trực tiếp trong CVAT; tôi dùng script click offline trên Colab.

## 6. Calibration (Part 6)

- Có 3 quyết định trong `calibration_decisions.csv`. Quan trọng nhất là `CAL-01`: tuk-tuk có cấu trúc ba bánh và mui/khung gắn liền phải là `three_wheeler`, không giữ `car` chỉ vì model dự đoán như vậy.
- Guideline v1.1 đã thêm cách phân biệt tuk-tuk với `car`/`truck`, giữ ngưỡng 20% cho người bị che trong tuk-tuk, và thêm bước kiểm bốn cạnh box.
- Nguyên nhân gồm cả lỗi máy (`CAL-01`), câu hỏi guideline còn mở (`CAL-02`) và tôi bỏ sót bước kiểm hình học (`CAL-03`).

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ảnh `can_review` + 5 ảnh ngẫu nhiên, seed 42); người kiểm là tôi. Hạn chế là tôi vừa là người review lô này nên không độc lập.
- Lỗi theo **đối tượng**: 0/85 = 0.0%; theo **ảnh**: 0/5 = 0.0% (ngưỡng Release ≤2%).
- Không ghi nhận lỗi rõ ràng trong 5 ảnh mẫu. Dù vậy, vòng blind trước đó đã lọt lỗi loose ở `img_005`, extra ở `img_022` và `img_033`, missing ở `img_024`.
- Giới hạn mẫu nhỏ: dù thấy 0/5 ảnh lỗi, cận trên 95% của tỷ lệ ảnh lỗi thật vẫn là 43%. Vì vậy mẫu này không chứng minh được tỷ lệ lỗi thật dưới 2%.
- Script kiểm được số file, lớp, box nằm trong ảnh và các tỷ lệ. Script không biết box có ôm sát vật thật, có vật bị cả người lẫn máy cùng bỏ sót, hoặc tôi có thực hiện đúng checklist hay không.

**Quyết định: Rework**, ngày 2026-11-19. Guideline đã đủ để tiếp tục nên không Hold, nhưng tỷ lệ bắt bẫy 8/12 = 66.7% thấp hơn ngưỡng 80%, nên chưa được Release.
Việc cần làm tiếp: review lại lô bằng checklist bắt buộc, ưu tiên kiểm box thừa và quét vật thiếu; sau đó lấy mẫu QC mới. Kết quả blind 8/12 vẫn được giữ nguyên, không sửa thành số cao hơn sau khi đã biết đáp án.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** Tôi dùng ChatGPT/Codex để giải thích hướng dẫn, hỗ trợ câu lệnh và rà cấu trúc file. Tôi tự gán nhãn, review ảnh và kiểm tra lại số liệu bằng output của các script trong repo.
- Gold không bị nhiễm: tôi gán xong và export 4 ảnh gold trước khi chạy hoặc xem pre-label.
- Máy Windows, CVAT chạy local bằng Docker. Model chạy bằng CPU trên Google Colab vì máy local không cài bộ model; lúc đầu Colab clone repo bị treo và runtime làm mất file tạm nên phải tải/chạy lại.
- Không dùng cache `data/cache/`; các lần chạy YOLO26, YOLOE và SAM được thực hiện trên Colab.
- Không gặp bug trong script. Khó khăn chủ yếu là Colab mất runtime, chạy sai thư mục `/content`, và phải tải lại model/ảnh `viz`.
- Lần sau tôi sẽ bật GPU ngay từ đầu, tải kết quả sau từng Part, và dùng checklist ẩn nhãn/quét ảnh trước ngay từ ảnh đầu tiên.
