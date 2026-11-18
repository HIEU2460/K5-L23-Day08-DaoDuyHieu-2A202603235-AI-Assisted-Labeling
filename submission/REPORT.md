# Báo cáo Day 08 — 2A202603235 - Đào Duy Hiếu

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | person=0.5, motorbike=0.5, car=0.5; bicycle/bus/truck dùng 0.3 | Ở `r01`, `r02`, YOLO26 nhận ra khá nhiều người và xe thông thường. Khi tăng lên 0.5 thì box thừa giảm, nhưng một số vật ở xa cũng mất. Các lớp có dưới 5 vật gold nên tôi giữ mức mặc định. |
| three_wheeler | YOLOE-26s, prompt `tuk-tuk`, `cyclo`, `cargo tricycle` | 0.2 | Ở `r03`, model tìm được chiếc tuk-tuk chính trong `img_002`, nhưng vẫn nhầm xe máy và pickup nên tôi không dùng kết quả trực tiếp. |

- Lệnh `merge_prelabels.py` đã dùng: `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.5,motorbike=0.5,car=0.5,three_wheeler=0.2" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- Tôi chọn SAM 2.1 base (`sam2.1_b.pt`). Model chạy được bằng CPU trên Colab và mask nhìn chung bám được phần chính của người, xe. Đổi lại, tốc độ chậm hơn detector (`r04`: 20.25 giây/ảnh).
- Có lỗi không thể chữa bằng cách tăng hoặc giảm ngưỡng. Trong `img_004` và `img_013`, YOLO26 gọi tuk-tuk là `car`. Vì vậy tôi lấy `three_wheeler` từ YOLOE rồi kiểm tra và sửa lại trong lúc review.
- Trong `img_002`, box tuk-tuk đúng có điểm 0.93, nhưng YOLOE cũng tạo box 0.37 trên pickup/xe máy. Trong `img_011`, khi tăng ngưỡng từ 0.25 lên 0.5 thì một số người và xe thật ở xa bị mất.
- Phần quét ngưỡng chỉ dùng 4 ảnh gold, đồng thời cũng đánh giá lại trên đúng 4 ảnh này. Do đó precision và recall có thể đẹp hơn thực tế, nhất là với những lớp có ít vật.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 314 | 314/365 = 86.0% | 365 box máy |
| edited | 18 | 18/365 = 4.9% | 365 box máy |
| deleted | 33 | 33/365 = 9.0% | 365 box máy |
| added | 118 | 118/450 = 26.2% | 450 đối tượng sau duyệt |

- Lớp khó dùng nhất là `three_wheeler`: chỉ 7 box được giữ nguyên, 7 box phải sửa và 19 box bị xóa. Có thể thấy rõ ở `img_002`: tuk-tuk chính được tìm đúng nhưng xe máy và pickup cũng bị nhận nhầm.
- Khi so với gold, precision là 42/(42+16) = 72.4% và recall là 42/46 = 91.3%. Mẫu số của recall là 46 đối tượng gold.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Tôi bắt được 8/12 bẫy, tương đương 66.7%. Các bẫy còn lọt là `img_005` (loose), `img_033` (extra), `img_024` (missing) và `img_022` (extra). Tôi bắt đủ 3/3 lỗi sai lớp nhưng chỉ bắt được 1/3 box thừa. Lúc review, có vài ảnh tôi nhìn theo box có sẵn nhiều hơn là quét lại toàn ảnh.
- Tỷ lệ accepted là 86.0% nhưng tỷ lệ bắt bẫy chỉ 66.7%. Hai con số này cho thấy giữ được nhiều box máy không có nghĩa là vòng review đã tốt. Một số box nhìn hợp lý làm tôi chủ quan và bỏ qua lỗi.

## 4. Máy có đáng dùng không?

- Theo `time_log.csv`, gán tay mất 15/4 = 3.75 phút/ảnh, còn review pre-label mất 72/24 = 3.00 phút/ảnh. Như vậy mỗi ảnh nhanh hơn khoảng 0.75 phút, tức khoảng 20%. Tuy nhiên sau phần gold tôi đã quen nhãn và quen thao tác CVAT hơn, nên mức chênh này chỉ để tham khảo.
- Theo tôi, pre-label vẫn có ích cho `person`, `motorbike` và `car` vì không phải vẽ lại từ đầu. Tuy nhiên chưa thể lấy nhãn máy để dùng luôn: máy còn sót khoảng 110 vật thật, còn `three_wheeler` thì phải sửa khá nhiều.
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

- Mục 4.2 giúp tôi biết bóng đổ không tính vào vật, kính vẫn thuộc xe và hai mask không nên chồng nhau. Khó nhất là phần người ngồi trong khoang hở và các khoảng trống của khung tuk-tuk, vì SAM dễ lấy luôn người vào mask xe.
- Tôi không thử Interactor SAM trong CVAT mà chỉ chạy script click trên Colab.

## 6. Calibration (Part 6)

- Tôi ghi 3 quyết định vào `calibration_decisions.csv`. Quyết định quan trọng nhất là `CAL-01`: nếu xe có cấu trúc ba bánh và mui/khung gắn liền thì gán `three_wheeler`, không giữ `car` chỉ vì model đã gán như vậy.
- Trong guideline v1.1, tôi thêm cách phân biệt tuk-tuk với `car`/`truck`, giữ mốc nhìn thấy 20% cho người bị che, đồng thời thêm bước kiểm đủ bốn cạnh box.
- Ba ca này đến từ ba nguyên nhân khác nhau: máy sai (`CAL-01`), guideline còn câu hỏi mở (`CAL-02`) và tôi bỏ sót bước kiểm hình học (`CAL-03`).

## 7. QC và quyết định (Part 6)

- Tôi kiểm 5 ảnh, gồm 0 ảnh `can_review` và 5 ảnh chọn ngẫu nhiên với seed 42. Tôi cũng là người đã review lô này nên kết quả kiểm không hoàn toàn độc lập.
- Lỗi theo **đối tượng**: 0/85 = 0.0%; theo **ảnh**: 0/5 = 0.0% (ngưỡng Release ≤2%).
- Trong 5 ảnh mẫu tôi không thấy lỗi rõ ràng. Tuy vậy, ở vòng blind trước đó tôi vẫn để lọt lỗi loose trong `img_005`, extra trong `img_022`, `img_033` và missing trong `img_024`.
- Mẫu 5 ảnh còn quá nhỏ. Dù kết quả là 0/5 ảnh lỗi, cận trên 95% vẫn là 43%, nên không thể dựa vào mẫu này để nói tỷ lệ lỗi thật đã dưới 2%.
- Script có thể kiểm tra file, lớp, tọa độ box và tính tỷ lệ. Nó không thể biết box có bám đúng vật hay không, có vật nào cả tôi và máy cùng bỏ sót, hoặc tôi có thật sự làm đủ checklist.

**Quyết định: Rework**, ngày 2026-11-19. Guideline hiện đủ rõ nên tôi không chọn Hold. Tuy nhiên tôi chỉ bắt được 8/12 bẫy, thấp hơn ngưỡng 80%, nên chưa thể Release.
Nếu làm vòng tiếp theo, tôi sẽ review lại bằng checklist, chú ý box thừa và vật bị sót rồi lấy mẫu QC mới. Kết quả blind ban đầu vẫn giữ là 8/12 vì lúc này tôi đã biết đáp án bẫy.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** Tôi dùng ChatGPT/Codex để hỏi lại những chỗ chưa hiểu trong hướng dẫn, hỗ trợ câu lệnh và kiểm tra xem file còn thiếu mục nào. Phần gán nhãn và review ảnh do tôi tự làm; các con số được đối chiếu lại với output của script trong repo.
- Gold không bị nhiễm: tôi gán xong và export 4 ảnh gold trước khi chạy hoặc xem pre-label.
- Tôi dùng Windows và chạy CVAT local bằng Docker. Các model chạy bằng CPU trên Google Colab do máy local không cài bộ model. Trong lúc làm, Colab từng bị treo khi clone repo và mất file sau khi đổi runtime nên tôi phải tải model, chạy lại một số phần.
- Không dùng cache `data/cache/`; các lần chạy YOLO26, YOLOE và SAM được thực hiện trên Colab.
- Tôi không gặp lỗi logic của script. Vướng mắc chính là có lúc chạy sai thư mục `/content`, Colab mất runtime và phải tải lại ảnh `viz`.
- Nếu làm lại, tôi sẽ bật GPU từ đầu, tải kết quả về sau mỗi Part và áp dụng bước ẩn nhãn/quét toàn ảnh ngay từ ảnh đầu tiên.
