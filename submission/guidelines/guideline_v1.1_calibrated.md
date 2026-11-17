# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603067 - Đặng Gia Tuấn

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của người duyệt. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ xếp vào `bicycle` hay `motorbike`? | Xe đạp điện có bàn đạp trợ lực xếp vào `bicycle`; xe máy điện thuần tay ga, không bàn đạp xếp vào `motorbike`. | img_011.jpg | CAL-02 |
| 2 | Khối xe máy đỗ dày đặc hoặc kẹt xe ở cự ly gần/trung bình: từng xe hay `ignore_region`? | Xe nhìn rõ cá thể (diện tích nhìn thấy >= 20% và chiều cao >= 20px) phải gán từng xe; chỉ vùng xe ở nền xa dính chùm không tách được mới gom bằng `ignore_region`. | img_005.jpg, img_024.jpg | CAL-03 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_002.jpg, img_013.jpg | Xe tuk-tuk Thái Lan có mui và ghế sau chở khách | YOLO26 gọi là `car` hoặc `truck` (conf > 0.7); YOLOE nhận diện đúng nhưng box lỏng dư nền | Luôn quy chuẩn về `three_wheeler`; ưu tiên box của YOLOE đè lên box của YOLO26 khi IoU >= 0.6 và kéo lại 4 cạnh cho khít mép xe. Người lái và khách vẫn có box `person` riêng. | Bổ sung Mục 2 & Mục 5: Xe 3 bánh có mui chở khách (tuk-tuk) hoặc ba gác chở hàng đều là `three_wheeler`, không gán `car` dù kích thước tương đương. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên viết sẵn | Khung tiêu chuẩn khởi tạo |
| v1.1 | Chốt phân loại xe đạp điện vs xe máy điện; xác lập tiêu chuẩn phân định từng xe vs ignore_region cho đám đông kẹt xe; chuẩn hóa xử lý tuk-tuk/ba gác bị máy nhận nhầm thành car/truck | Bổ sung các câu hỏi mở còn thiếu của v1.0 và khắc phục lỗi lặp có hệ thống của mô hình (CAL-01, CAL-02, CAL-03) |
