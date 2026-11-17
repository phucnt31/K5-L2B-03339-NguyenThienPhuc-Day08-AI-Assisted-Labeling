# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603339 - Nguyễn Thiên Phúc

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: bicycle hay motorbike? | Nếu có bàn đạp cơ học và kết cấu khung xe đạp gán `bicycle`; nếu không có bàn đạp, có sàn để chân rộng kiểu scooter gán `motorbike`. | img_004.jpg | CAL-01 |
| 2 | Khối xe máy đỗ dày đặc che khuất nhau ở hậu cảnh: từng xe hay ignore_region? | Xe ở tiền cảnh hoặc nhìn rõ chi tiết gán riêng lẻ từng `motorbike`; khối xe đỗ sát nhau ở hậu cảnh cao < 20 px gán vào `ignore_region`. | img_020.jpg | CAL-03 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013.jpg | Tuk-tuk có mui và khung kim loại giống ô tô | YOLO26 nhận nhầm thành `car` | Gán đúng `three_wheeler` cho toàn bộ thân xe và mui | Mục 2 & 4.1.4: Xe 3 bánh có mui vẫn là `three_wheeler`, không gán `car` hay `truck` (CAL-02) |
| img_028.jpg | Người đi bộ và xe máy ở khu vực khuất bị bỏ sót | Máy bỏ sót một box đúng (bẫy missing), người duyệt không phát hiện | Thực hiện nghiêm ngặt Bước 0: ẩn nhãn máy 15s quét toàn ảnh trước khi bật nhãn | Mục 5: Quy trình review Bước 0 quét độc lập trước khi mở nhãn máy (CAL-04) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên viết sẵn | — |
| v1.1 | Bổ sung phân định xe đạp điện / xe máy điện (CAL-01); làm rõ tuk-tuk có mui là three_wheeler (CAL-02); lượng hóa ngưỡng < 20 px gom ignore_region (CAL-03); siết chặt checklist Bước 0 chống lọt bẫy missing (CAL-04) | Trích xuất từ calibration sau gán nhãn gold và review batch (CAL-01, CAL-02, CAL-03, CAL-04) |
