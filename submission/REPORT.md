# Báo cáo Day 08 — 2A202603067 - Đặng Gia Tuấn

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | <<YOLO26m>> | <<theo lớp, từ threshold_sweep>> | <<>> |
| three_wheeler | <<YOLOE-26s, prompt: ...>> | <<>> | <<>> |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): <<>>
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Chọn **SAM 2.1** (`sam2.1_b.pt`). Lý do: Cho đường viền polygon sắc nét, bám sát mép nan hoa và vòm mui xe trên ảnh phố châu Á, cân bằng tối ưu giữa độ chính xác hình học và tài nguyên máy; không yêu cầu duyệt quyền truy cập phức tạp như SAM 3.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Lỗi gán sai ngữ nghĩa lớp (semantic misclassification) do detector danh mục đóng COCO (YOLO26) không có nhãn `three_wheeler` nên gán nhầm thành `car` hoặc `truck`. Lỗi này không thể sửa bằng cách nâng hay hạ ngưỡng conf (nâng ngưỡng thì mất hẳn xe, hạ ngưỡng thì thêm box rác). Cách sửa: Kết hợp mô hình open-vocabulary (YOLOE-26s) với prompt chuyên biệt (`tuk-tuk`, `cyclo`, `cargo tricycle`) và dùng `merge_prelabels.py` ưu tiên lớp `three_wheeler` đè lên `car`/`truck` khi IoU >= 0.6.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào):
  - *Điểm cao mà sai:* Trên `img_002.jpg`, YOLO26 phát hiện chiếc xe tuk-tuk màu xanh tiền cảnh gán nhãn `car` với độ tin cậy rất cao (~0.85); trên `img_013.jpg`, tuk-tuk cũng bị gán nhãn `car` với điểm cao (~0.78).
  - *Điểm thấp mà đúng:* Trên `img_011.jpg`, YOLOE phát hiện phương tiện ba bánh với conf chỉ 0.28 nhưng nhận định chính xác là `three_wheeler`.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: <<sẽ chốt ở Part 3>>

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | <<>> | <<>> | <<box máy>> |
| edited | <<>> | <<>> | <<>> |
| deleted | <<>> | <<>> | <<>> |
| added | <<>> | <<>> | <<đối tượng sau duyệt>> |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? <<>>
- So với gold: precision / recall của **máy** <<>> (mẫu số: <<đối tượng gold>>)

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

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Sử dụng Antigravity AI Assistant để giải thích quy trình lab, kiểm tra môi trường và viết script hỗ trợ kiểm thử. Tất cả kết luận, nhận xét và nhãn duyệt đều được tự kiểm tra và thẩm định bằng tay.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không. 4 ảnh gold được gán tay độc lập 100% trên CVAT theo đúng guideline v1.0, tuyệt đối không chạy model hay nhìn trước pre-label.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11, GPU NVIDIA GeForce RTX 4060 Laptop (8GB VRAM, CUDA 12.7), CVAT v2.74.1 Docker local.
- Dùng cache `data/cache/` thay vì tự chạy? Không, tự chạy các mô hình cục bộ.
- Lỗi / bug script đã gặp (nếu có): <<cập nhật nếu có trong các part sau>>
- Một điều bạn làm khác lần sau: <<cập nhật ở cuối bài>>
