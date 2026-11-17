# Báo cáo Day 08 — 2A202603067 - Đặng Gia Tuấn

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26m | 0.30 | Tại conf 0.30 recall đạt cao nhất (88.9%, 16/18 vật) với F1=0.604; không đạt 90% do có 2 người ở xa bị che khuất. Dẫn run `r01` & `r05`. |
| motorbike | YOLO26m | 0.30 | Tại conf 0.30 recall đạt tối đa 77.8% (7/9 vật) với F1=0.636; giữ 0.30 để tránh sinh quá nhiều box rác ở nền xa như ở conf 0.1-0.2. Dẫn run `r01` & `r05`. |
| car | YOLO26m | 0.30 | Tại conf 0.30 recall đạt tối đa 87.5% (7/8 vật), precision đạt 63.6%, F1=0.700. Dẫn run `r01` & `r05`. |
| bicycle / bus / truck | YOLO26m | 0.30 | Số lượng vật thể trong 4 ảnh gold quá ít (< 5 vật: bicycle=3, bus=1, truck=0); tuân thủ đúng quy tắc thống kê mẫu nhỏ: không chọn ngưỡng riêng mà giữ mặc định 0.30. |
| three_wheeler | YOLOE-26s, prompt: tuk-tuk, cyclo, cargo tricycle | 0.25 | Tại conf 0.25-0.30 recall đạt tối đa 85.7% (6/7 vật trên gold). YOLOE giải quyết điểm mù của YOLO26 cho xe ba bánh địa phương. Dẫn run `r03` & `r06`. |

- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Chọn **SAM 2.1** (`sam2.1_b.pt`). Lý do: Cho đường viền polygon sắc nét, bám sát mép nan hoa và vòm mui xe trên ảnh phố châu Á, cân bằng tối ưu giữa độ chính xác hình học và tài nguyên máy; không yêu cầu duyệt quyền truy cập phức tạp như SAM 3.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Lỗi gán sai ngữ nghĩa lớp (semantic misclassification) do detector danh mục đóng COCO (YOLO26) không có nhãn `three_wheeler` nên gán nhầm thành `car` hoặc `truck`. Lỗi này không thể sửa bằng cách nâng hay hạ ngưỡng conf (nâng ngưỡng thì mất hẳn xe, hạ ngưỡng thì thêm box rác). Cách sửa: Kết hợp mô hình open-vocabulary (YOLOE-26s) với prompt chuyên biệt (`tuk-tuk`, `cyclo`, `cargo tricycle`) và dùng `merge_prelabels.py` ưu tiên lớp `three_wheeler` đè lên `car`/`truck` khi IoU >= 0.6.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào):
  - *Điểm cao mà sai:* Trên `img_002.jpg`, YOLO26 phát hiện chiếc xe tuk-tuk màu xanh tiền cảnh gán nhãn `car` với độ tin cậy rất cao (~0.85); trên `img_013.jpg`, tuk-tuk cũng bị gán nhãn `car` với điểm cao (~0.78).
  - *Điểm thấp mà đúng:* Trên `img_011.jpg`, YOLOE phát hiện phương tiện ba bánh với conf chỉ 0.28 nhưng nhận định chính xác là `three_wheeler`.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Chọn và chấm trên cùng dữ liệu (in-sample) nên precision và recall đo được trên tập gold lạc quan hơn thực tế trên 24 ảnh batch. Mẫu 4 ảnh chỉ mang tính ước lượng thô; các lớp dưới 5 mẫu (bicycle=3, bus=1, truck=0) không đủ độ tin cậy để đưa ra kết luận thống kê mạnh.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 322 | 74.4% | 433 box máy |
| edited | 25 | 5.8% | 433 box máy |
| deleted | 63 | 14.5% | 433 box máy |
| added | 27 | 7.2% | 374 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)?
  - Máy tệ nhất ở lớp `three_wheeler`: có 11 box bị deleted (chiếm 44% tổng số 25 box máy của lớp này) và 7 box bị edited; chỉ có 7 box được accepted. Nguyên nhân do YOLOE phát hiện box bị lỏng hoặc nhận diện nhầm xe bán tải/ô tô con thành xe ba bánh (ví dụ trên `img_006.jpg`, `img_009.jpg`).
  - Lớp `truck` cũng bị xóa 5/7 box máy do YOLO26 nhận diện nhầm xe bán tải hoặc xe ba gác thành truck.
  - Lớp `person` bị xóa 32 box máy do máy bắt nhầm người ở xa quá nhỏ (< 20px) hoặc các cấu trúc pano/bóng đổ.
  - *Lưu ý về đổi lớp:* Trong số deleted + added có 6 trường hợp là đổi lớp (máy gán sai lớp, box đúng vị trí). Số đối tượng máy sót thật sự là 21 vật (27 added - 6 đổi lớp).
- So với gold: precision / recall của **máy** đạt 54.5% / 78.3% (mẫu số: 46 đối tượng gold; TP=36, FP=30, FN=10; KTC 95% Precision: 43–66%, Recall: 64–88%).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 6/12 = 50.0% (KTC 95%: 25–75%).
  - Chi tiết từng loại bẫy:
    - `extra`: bắt được 3/3 = 100% (xóa sạch toàn bộ box ma ở nền trống).
    - `loose`: bắt được 2/3 = 66.7% (kéo lại viền ôm sát vật, lọt 1 bẫy ở `img_009.jpg`).
    - `missing`: bắt được 1/3 = 33.3% (vẽ thêm vật bị máy xóa, lọt 2 bẫy ở `img_028.jpg` và `img_034.jpg`).
    - `wrong_class`: bắt được 0/3 = 0.0% (lọt cả 3 bẫy ở `img_027.jpg`, `img_006.jpg`, `img_035.jpg`).
  - Vì sao lọt:
    - *Lọt wrong_class:* Do hiệu ứng "neo nhãn" (anchor bias). Khi thấy box bao quanh đối tượng hợp lý, mắt người duyệt có xu hướng tin tưởng vị trí và bỏ qua việc kiểm tra lại tên nhãn lớp (`motorbike` vs `bicycle`, `car` vs `truck`).
    - *Lọt missing:* Do người duyệt bỏ qua Bước 0 (quét ảnh trước khi bật nhãn), nên khi AI không vẽ box thì mắt người duyệt cũng dễ dàng lướt qua theo gợi ý của máy.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Tỉ lệ accepted đạt tới 74.4% nhưng tỉ lệ bắt bẫy chỉ đạt 50.0% (dưới chuẩn 80%) chứng minh rõ ràng sự tồn tại của **thiên kiến tự động hóa (Automation Bias)**: người duyệt có xu hướng tin tưởng thụ động vào các đề xuất của AI, duyệt nhanh và "gật đầu" chấp thuận các box có sẵn mà chưa soi kỹ từng cạnh viền và lớp nhãn theo đúng checklist 4 bước.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 2.50 phút/ảnh (10 phút / 4 ảnh gold) vs duyệt có pre-label 1.25 phút/ảnh (30 phút / 24 ảnh batch).
  - Tốc độ duyệt nhanh hơn gấp 2.0 lần, giúp tiết kiệm 50.0% thời gian so với gán tay hoàn toàn.
  - *Thiên lệch:* Người duyệt đã gán 4 ảnh gold trước đó nên đã quen thuộc bối cảnh phố châu Á, cộng thêm việc duyệt có sẵn khung hình giúp giảm bớt thao tác kéo chuột vẽ mới từ đầu.
- Kết luận: Có đáng dùng, đặc biệt rất hiệu quả cho các lớp cơ bản có độ chính xác cao (`person`, `motorbike`, `car`) vì giúp tăng gấp đôi năng suất làm việc; tuy nhiên đối với các lớp đặc thù như `three_wheeler` hay vật thể ở xa bị che khuất thì bắt buộc phải kiểm tra kỹ lưỡng vì AI thường xuyên sai lớp hoặc bỏ sót.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. *Quy tắc quét mù (Blind Scan):* Ẩn toàn bộ nhãn máy (bật/tắt nút mắt trong CVAT) và dành ít nhất 15 giây quét đếm nhẩm số lượng từng lớp trên 100% số ảnh trước khi cho hiển thị nhãn máy.
  2. *Quy tắc kiểm duyệt phân lớp (Class Filter Review):* Sử dụng tính năng Filters trên CVAT để lọc và rà soát độc lập từng lớp đối tượng (đặc biệt là `three_wheeler` và `truck`) thay vì nhìn đồng thời toàn bộ 8 lớp, nhằm chặn đứng bẫy `wrong_class`.
  3. *Quy tắc ngắt quãng tập trung (Interval Breaks):* Bắt buộc nghỉ ngơi 3–5 phút sau mỗi 8–10 ảnh duyệt để hồi phục độ tập trung thị giác, tránh hiện tượng mỏi mắt dẫn đến bấm accept hàng loạt (được kiểm chứng qua nhật ký thời gian `time_log.csv`).

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 pos, 1 neg) | Điểm âm đã khoét bớt lưng áo người lái nhưng vẫn dính một phần đầu và tay | 1 click âm chưa đủ tách triệt để người lái khỏi buồng lái tuk-tuk; biên mui xe và bánh xe bám rất khít (1 polygon, 45.4% ảnh) | `tuktuk_img013_sam2.jpg` |
| `img_013.jpg` | SAM 1 MobileSAM (`mobile_sam.pt`) | 3 (2 pos, 1 neg) | Tách được người lái tương tự nhưng mask xe bị khuyết | Mask bị phân mảnh thành 2 polygon riêng biệt (vùng mui và thân dưới bị đứt đoạn), viền nan hoa bánh xe mờ hơn | `tuktuk_img013_sam1.jpg` |
| `img_004.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 pos, 1 neg) | Loại bỏ sạch người lái và xe bên cạnh | Điểm âm (465,450) chặn tốt xe bên cạnh nhưng viền kính chắn gió bị rách nhẹ (1 polygon, 13.3% ảnh) | `tuktuk_img004_sam2.jpg` |
| `img_017.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 pos, 1 neg) | Không vướng người lái, ôm trọn thân xe tuk-tuk | Điểm âm ngăn mask lan sang xe máy phía sau; viền cản trước rất sắc nét (1 polygon, 12.1% ảnh) | `tuktuk_img017_sam2.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu?
  - *Quy tắc giúp:* Mục 4.2.1 (polygon bám đường biên nhìn thấy) và 4.2.4 (hai vật chạm nhau không chồng lên nhau) giúp định hướng rõ ràng việc tách riêng người lái (`person`) khỏi xe (`three_wheeler`). Điểm click âm (`--neg`) là công cụ hữu hiệu để thực thi việc này.
  - *Quy tắc thiếu:* Guideline v1.0 chưa quy định cụ thể việc xử lý **kính chắn gió / khoảng trống nhìn xuyên qua khung xe** (lỗ thủng rỗng ruột) và chưa làm rõ bao nhiêu điểm âm là đủ để tách người lái khi ngồi trong buồng lái hở của tuk-tuk. Đây là đầu vào quan trọng cho Calibration v1.1.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: Trên bản CVAT Docker cục bộ không kèm serverless Nuclio, công cụ AI Tools Interactor không có sẵn model; do đó phương pháp offline bằng CLI `sam2_points.py` kiểm soát toạ độ click (`--pos`, `--neg`) chính xác hơn, lặp lại được kết quả và hiển thị trực quan tỷ lệ diện tích che phủ.

## 6. Calibration (Part 6)

- Số quyết định: 3 (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-01`): Quy chuẩn toàn bộ phương tiện ba bánh có mui chở khách (tuk-tuk) và chở hàng (ba gác, xích lô) thành lớp `three_wheeler`, ưu tiên nhãn YOLOE đè lên `car`/`truck` của YOLO khi IoU >= 0.6 và kéo khít viền 4 cạnh.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`:
  1. Phân định cơ chế vận hành xe đạp điện vs xe máy điện (CAL-02): có bàn đạp chân trợ lực tính là `bicycle`, thuần động cơ điện không bàn đạp tính là `motorbike`.
  2. Xác lập tiêu chuẩn phân định giữa vẽ từng xe máy vs vùng `ignore_region` (CAL-03): nhận diện cá thể nhìn thấy >= 20% và cao >= 20px thì vẽ từng xe; chỉ gom `ignore_region` cho các cụm xe ở xa dính chùm không tách được.
  3. Xử lý triệt để lỗi lặp xe ba bánh bị máy gọi là car/truck (CAL-01).
- Nguyên nhân gốc: Kết hợp giữa **guideline v1.0 còn mở/mơ hồ** (chưa quy định xe đạp điện, tiêu chuẩn cụm xe kẹt) và **lỗi cố hữu của mô hình đóng** (YOLO26 mù xe ba bánh).

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt, vừa là người kiểm tra QC nên có thể có điểm mù chủ quan).
- Lỗi theo **đối tượng**: 2/83 = 2.4% · theo **ảnh**: 2/5 = 40.0% (ngưỡng Release ≤ 2%).
- Top lỗi: `sai_lop` chiếm 2/2 lỗi phát hiện (chiếm 100% lỗi):
  - Ví dụ 1: Trên `img_006.jpg`, xe xích lô/xe ba bánh bị gán nhầm thành `bicycle` (bẫy wrong_class bị lọt).
  - Ví dụ 2: Trên `img_035.jpg`, xe máy nhỏ ở xa bị gán nhầm thành `bicycle` (bẫy wrong_class bị lọt).
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): Với 2 ảnh lỗi trong 5 ảnh mẫu, cận trên khoảng tin cậy Wilson 95% của tỉ lệ ảnh lỗi thật lên tới **77%**. Ngưỡng 2% không thể được chứng minh tuyệt đối bằng mẫu nhỏ 5 ảnh này.
- Script kiểm được / không kiểm được:
  - *Kiểm được:* cú pháp nhãn COCO, số lượng đối tượng, tỷ lệ ảnh lỗi trong mẫu, tính toán khoảng tin cậy Wilson, phát hiện các trường hợp bẫy bị lọt.
  - *Không kiểm được:* độ ôm sát viền thực tế của box (bounding box có bị lỏng vài px không), các đối tượng ở hậu cảnh mà cả người lẫn máy cùng bỏ sót (không có trong gold), và việc người duyệt có thực hiện đúng quy trình quét mù 15 giây hay không.

**Quyết định: REWORK**, ngày 2026-11-17.
- Lý do (đi theo cây Hold → Rework → Release):
  1. Tầng Hold: Đã chốt xong toàn bộ câu hỏi mở thành guideline v1.1, dữ liệu ảnh không có ảnh nào bị hỏng/lỗi file.
  2. Tầng Rework: Tỉ lệ ảnh lỗi trong mẫu QC đạt 40.0% (2/5 ảnh) vượt xa ngưỡng tối đa cho phép là 2.0%; đồng thời tỉ lệ bắt bẫy chỉ đạt 50.0% (6/12 bẫy) dưới chuẩn 80.0% do ảnh hưởng của thiên kiến tự động hóa.
  3. Theo đúng quy trình nghiêm ngặt: Lô nhãn không đủ điều kiện Release và bắt buộc phải **REWORK**.
- Nếu Rework / Hold, việc cần làm tiếp:
  1. Nạp lại 24 ảnh batch lên CVAT và thực hiện duyệt lại đợt 2.
  2. Áp dụng triệt để Quy tắc quét mù 15s (ẩn nhãn máy trước khi duyệt) và sử dụng bộ lọc Filters trên CVAT soi riêng từng lớp (`three_wheeler`, `bicycle`, `truck`) để bắt triệt để các bẫy `wrong_class` và `missing`.
  3. Kéo lại viền các box bị lỏng trên `img_009.jpg` và xuất bản vá `qc_fixed.json`.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Sử dụng Antigravity AI Assistant để giải thích quy trình lab, kiểm tra môi trường và viết script hỗ trợ kiểm thử. Tất cả kết luận, nhận xét và nhãn duyệt đều được tự kiểm tra và thẩm định bằng tay.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không. 4 ảnh gold được gán tay độc lập 100% trên CVAT theo đúng guideline v1.0, tuyệt đối không chạy model hay nhìn trước pre-label.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11, GPU NVIDIA GeForce RTX 4060 Laptop (8GB VRAM, CUDA 12.7), CVAT v2.74.1 Docker local.
- Dùng cache `data/cache/` thay vì tự chạy? Không, tự chạy các mô hình cục bộ.
- Lỗi / bug script đã gặp (nếu có): Không gặp lỗi nghiêm trọng; cần lưu ý bọc nháy kép các tham số có dấu phẩy trên PowerShell Windows.
- Một điều bạn làm khác lần sau: Sẽ thực hiện nghiêm túc hơn bước ẩn nhãn máy quét trước để không bị thiên kiến tự động hóa đánh lừa ở các bẫy sai nhãn lớp.
