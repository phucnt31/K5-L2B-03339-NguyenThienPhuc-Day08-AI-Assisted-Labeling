# Báo cáo Day 08 — 2A202603339 - Nguyễn Thiên Phúc

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26m | 0.15 | Quét trên gold đạt recall 95% (>= 90%), precision 47% (mẫu 19 vật gold từ `qc/threshold_sweep.md`, tương ứng khảo sát `r02`, `r03`). |
| bicycle / motorbike / car / bus / truck | YOLO26m | 0.30 | Dùng ngưỡng mặc định 0.3: bicycle, bus, truck có 0 vật trong gold (< 5); motorbike và car không có ngưỡng nào <= 0.5 đạt recall >= 90% (theo khuyến nghị `threshold_sweep.md`, dẫn `r02`, `r03`). |
| three_wheeler | YOLOE-26s (prompt: three_wheeler) | 0.15 | Đạt recall 100% (>= 90%), precision 56% trên 5 vật gold; khắc phục triệt để lỗi YOLO26 bỏ sót hoặc gọi nhầm tuk-tuk thành car (dẫn `r04`). |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.15,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Chọn **SAM 2.1 base (`sam2.1_b.pt`)**. Đây là phiên bản cân bằng tối ưu giữa chất lượng phân đoạn biên và tốc độ xử lý. So với SAM 1 (`mobile_sam`), SAM 2.1 bám sát chi tiết nan hoa, khung mui xe và tách biên người lái tốt hơn. So với SAM 3, SAM 2.1 không vướng quyền truy cập tài khoản HuggingFace (gated repo) và tiêu thụ tài nguyên RAM/VRAM vừa phải, phù hợp phần cứng hiện tại.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Lỗi nhận diện sai nhãn bản chất (misclassification / semantic confusion), tiêu biểu là YOLO26 nhận tuk-tuk/xe ba gác thành `car` hoặc `truck` (hoặc xích lô thành xe đạp). Việc tăng hay giảm ngưỡng `conf` chỉ làm ẩn/hiện box chứ không thể đổi tên lớp của đối tượng. Cách sửa: Phải kết hợp thêm model open-vocabulary chuyên biệt (YOLOE với prompt `three_wheeler`) hoặc người duyệt sửa nhãn thủ công trong CVAT (`Ctrl+số`), đồng thời chuẩn hóa quy tắc phân loại trong calibration guideline (CAL-02).
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào):
  - Điểm cao mà sai: Ở `img_013.jpg`, YOLO26 phát hiện chiếc tuk-tuk và gán nhãn `car` với điểm tin cậy rất cao (~0.85) nhưng sai hoàn toàn về ngữ nghĩa phân loại.
  - Điểm thấp mà đúng: Ở `img_004.jpg`, người đi bộ ở xa bị che khuất một phần có điểm tin cậy chỉ ~0.18 nhưng là đối tượng thật; việc hạ ngưỡng person xuống 0.15 đã giúp giữ lại đối tượng này cho người duyệt.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Việc chọn ngưỡng trên chính 4 ảnh gold (in-sample optimization) dẫn đến hiện tượng quá khớp (overfitting). Mẫu 4 ảnh chỉ có 41 vật thể (thậm chí bicycle, bus, truck có 0 vật), khoảng tin cậy rất rộng (KTC 95% precision 39–62%). Khi áp sang lô 24 ảnh batch với đa dạng góc chụp và mật độ xe cộ, precision thực tế sẽ thấp hơn.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 577 | 91.7% | 629 box máy |
| edited | 10 | 1.6% | 629 box máy |
| deleted | 42 | 6.7% | 629 box máy |
| added | 17 | 2.8% | 604 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? Lớp máy tệ nhất là **`three_wheeler`** (bị deleted 21/43 box máy, edited 4) và **`truck`** (deleted 3, added 3). Nguyên nhân: Các dòng xe đường phố châu Á (xe ba gác tự chế, tuk-tuk gắn mui tôn/bạt, xe lam) có hình thái lai tạp giữa ô tô tải nhỏ và xe máy, khiến cả YOLO26 và YOLOE dễ nhận nhầm thành `car`, `truck` hoặc bắt nhầm các bóng râm, mái hiên ven đường (tiêu biểu ở `img_010.jpg`, `img_013.jpg`).
- So với gold: precision / recall của **máy**: Precision đạt **50.7%** (37/73, KTC 95%: 39–62%), Recall đạt **90.2%** (37/41, KTC 95%: 77–96%) (mẫu số: **41 đối tượng gold**). Máy đạt mục tiêu recall cao (>= 90%) để giảm thiểu tối đa việc bỏ sót vật thể cho người duyệt.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **7/12 = 58.3%** (KTC 95%: 32–81%); loại nào lọt: Lọt 5 bẫy gồm `missing` (2/3 bẫy lọt ở `img_013.jpg` và `img_028.jpg`), `extra` (1 bẫy lọt ở `img_005.jpg`), `loose` (1 bẫy lọt ở `img_031.jpg`), `wrong_class` (1 bẫy lọt ở `img_029.jpg`). Vì sao: Người duyệt đã **bỏ qua Bước 0** của checklist (không ẩn nhãn máy để tự quét toàn cảnh và đếm nhẩm số lượng vật trong 15 giây đầu). Mắt bị hiệu ứng "neo nhãn" vào các box máy gợi ý sẵn nên chỉ chăm chú nhìn vào các box có sẵn mà bỏ qua các khoảng trống nơi máy bỏ sót vật thật.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Tỉ lệ accepted rất cao (91.7%) nhưng tỉ lệ bắt bẫy thấp (58.3% < ngưỡng đạt 80%) chứng tỏ người duyệt bị ảnh hưởng nặng nề bởi **thiên kiến tự động hóa (automation bias)** và tâm lý mệt mỏi nhận thức. Người duyệt tin tưởng thụ động vào gợi ý của AI, xem lướt qua các box điểm cao mà không đối chiếu độc lập với ảnh gốc.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay **7.0 phút/ảnh** (28 phút cho 4 ảnh gold) vs duyệt **2.54 phút/ảnh** (61 phút cho 24 ảnh batch). Thiên lệch: 4 ảnh gold được gán tay trước nên người làm đã làm quen với bối cảnh phố xá châu Á và thuộc các quy tắc phân loại, giúp tốc độ duyệt ở lô batch được hưởng một phần hiệu ứng quen tay.
- Kết luận: **Có rất đáng dùng**, giúp tiết kiệm khoảng **63.7% thời gian** gán nhãn trên mỗi ảnh (từ 7.0 phút xuống 2.54 phút) cho các lớp phổ thông như `person`, `motorbike`, `car`. Tuy nhiên, với lớp `three_wheeler` và các đám đông bị che khuất thì không nên tin tưởng hoàn toàn vào máy, cần người duyệt kiểm soát chặt chẽ.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. *Quy tắc Bước 0*: Luôn ẩn nhãn máy (phím Space hoặc nút ẩn trong CVAT) và dành tối thiểu 15 giây đếm nhẩm các vật thể trước khi bật nhãn máy (kiểm soát 100% số ảnh).
  2. *Quy tắc chia góc quét*: Với ảnh có trên 20 đối tượng, chia ảnh thành 4 góc phần tư để rà soát độc lập từng góc theo đúng 4 bước checklist (Sót -> Thừa -> Sai lớp -> Lỏng).
  3. *Quy tắc nghỉ giải lao*: Cứ sau 8–10 ảnh duyệt liên tục, bắt buộc nghỉ mắt 3–5 phút để tránh mỏi mắt và suy giảm tập trung nhận thức.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_010.jpg` | SAM 2.1 base (`sam2.1_b.pt`) | 3 clicks (2 pos, 1 neg) | Tách riêng xe xích lô, không gồm người đạp xe | Cần đặt thêm 1 điểm negative ở lưng người lái để tránh mask xe tràn sang cơ thể người | `cyclo_img010.jpg` |
| `img_013.jpg` | SAM 2.1 base (`sam2.1_b.pt`) | 3 clicks (2 pos, 1 neg) | Gồm thân và mui xe tuk-tuk, tách biệt người lái | Điểm negative tại cabin giúp ngăn mask thân xe ăn lan vào trang phục người lái | `tuktuk_img013.jpg` |
| `img_017.jpg` | SAM 2.1 base (`sam2.1_b.pt`) | 3 clicks (2 pos, 1 neg) | Bao trùm thân vỏ và bánh xe tuk-tuk, tách hành khách | Mask bám viền rất tốt quanh khung sắt; tuy nhiên phần kính chắn gió trong suốt bị rách biên nhẹ | `tuktuk_img017.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu?
  - Quy tắc giúp ích: Quy tắc "xe là vật riêng, người là vật riêng" và "giỏ xe/thùng xe gắn cố định thuộc xe" giúp xác định rõ phạm vi mask của phương tiện.
  - Quy tắc còn thiếu: Guideline chưa hướng dẫn cụ thể cách xử lý mask đối với kính chắn gió trong suốt (nhìn xuyên qua thấy vô lăng/người lái) và hàng hóa chằng buộc nhô ra ngoài thùng xe.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: Thực hiện được trên CVAT thông qua AI Tools tương tác. Khác biệt lớn nhất là tính trực quan: CVAT cho phép tương tác trực tiếp bằng chuột và xem kết quả mask phản hồi tức thì để bổ sung điểm âm/dương, trong khi script offline (`sam2_points.py`) phải áng chừng tọa độ pixel để truyền qua dòng lệnh CLI. Tuy nhiên, khi chạy CPU local thì tương tác trong CVAT có độ trễ 2–4 giây mỗi lần click.

## 6. Calibration (Part 6)

- Số quyết định: **4 quyết định** (`calibration_decisions.csv`: CAL-01 đến CAL-04); quan trọng nhất: **`CAL-02` (img_013)** — Phân định rõ toàn bộ xe ba bánh chở khách/hàng (tuk-tuk, ba gác) có mui hoặc cabin đều gán `three_wheeler`, chấm dứt việc gán nhầm lẫn sang `car` hoặc `truck`.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: Phân loại xe 2 bánh điện dựa vào bàn đạp (CAL-01); phân định tuk-tuk/ba gác có mui là `three_wheeler` (CAL-02); lượng hóa ngưỡng chiều cao < 20 px cho đám đông để gán `ignore_region` (CAL-03); bắt buộc thực hiện Bước 0 chống lọt bẫy missing (CAL-04).
- Nguyên nhân gốc: Kết hợp cả ba yếu tố: guideline v1.0 còn để mở chưa quy định chi tiết, máy nhận diện sai hình thái xe đặc thù châu Á, và người duyệt bị ảnh hưởng bởi thiên kiến tự động hóa.

## 7. QC và quyết định (Part 6)

- Mẫu: **6 ảnh** (1 ảnh `can_review` là `img_003.jpg` + 5 ảnh ngẫu nhiên: `img_004.jpg`, `img_008.jpg`, `img_014.jpg`, `img_017.jpg`, `img_033.jpg`, seed 42); người kiểm: **Nguyễn Thiên Phúc** (**hạn chế**: người kiểm định vừa là người trực tiếp duyệt nhãn nên khó tránh khỏi điểm mù nhận thức chủ quan).
- Lỗi theo **đối tượng**: **1/101 = 1.0%** (1 lỗi trên 101 đối tượng trong mẫu) · theo **ảnh**: **1/6 = 16.7%** (ngưỡng Release ≤ 2%).
- Top lỗi: `sai_vi_tri_kich_thuoc` (1 lỗi tại `img_014.jpg`, box người ở cự ly xa cao 19.3 px < 20 px lẽ ra phải gom vào `ignore_region`).
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): Với 1 ảnh lỗi trên 6 ảnh mẫu, cận trên khoảng tin cậy 95% (Wilson) của tỉ lệ ảnh lỗi thật lên tới **56%**. Kích thước mẫu 6 ảnh quá nhỏ, hoàn toàn không thể chứng minh được tỉ lệ lỗi thật sự nằm dưới ngưỡng 2%.
- Script kiểm được / không kiểm được: Script kiểm tra được định dạng dữ liệu, taxonomy, bounding box hợp lệ, tính tỷ lệ accepted/edited/deleted/added và tính các số đo QC. Script không thể kiểm tra được độ ôm sát thực tế của box quanh viền vật thể, không kiểm tra được tính đúng đắn ngữ nghĩa ngoài đời thực, và hoàn toàn không phát hiện được các vật thể mà cả máy lẫn người cùng bỏ sót.

**Quyết định: REWORK**, ngày 2026-11-17.
- Lý do (đi theo cây quyết định Hold → Rework → Release):
  1. Guideline đã được hiệu chỉnh xong ở Part 6 (không rơi vào Hold).
  2. Tỉ lệ ảnh có lỗi ở mẫu QC là 16.7% (1/6 ảnh), vượt xa ngưỡng cho phép (<= 2%).
  3. Tỉ lệ bắt bẫy chỉ đạt 58.3% (7/12 bẫy), thấp hơn chuẩn 80% của hợp đồng kiểm định chất lượng -> lô nhãn chưa đủ độ tin cậy.
  => Theo đúng cây quyết định, kết luận trung thực bắt buộc phải là **REWORK**.
- Nếu Rework, việc cần làm tiếp:
  1. Mở lại task trong CVAT, rà soát lại các ảnh có mật độ phương tiện đông đúc (`img_005`, `img_018`, `img_024`, `img_028`, `img_029`).
  2. Thực hiện nghiêm Bước 0 để bắt triệt để các vật bị bỏ sót (bẫy missing).
  3. Gom các cụm người và xe ở xa có chiều cao < 20 px vào `ignore_region` để giải quyết cảnh báo kích thước.
  4. Xuất lại `batch_reviewed.json` và chạy lại quy trình nghiệm thu QC.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Dùng mô hình YOLO26m, YOLOE-26s và SAM 2.1 base để sinh nhãn gợi ý theo đúng quy trình; dùng trợ lý AI hỗ trợ tính toán số liệu thống kê thời gian và kiểm tra cú pháp bài nộp; toàn bộ kết quả đều được xác thực chéo qua các script kiểm thử tự động của repo.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? **Không**, 4 ảnh gold được gán tay độc lập từ đầu ở Part 1 trước khi chạy bất kỳ pipeline pre-label hay tạo bẫy nào.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11, Python 3.12, chạy môi trường ảo `.venv`, tính toán trên CPU local; CVAT chạy qua trình duyệt; cấu hình ban đầu gặp chú ý về encoding UTF-8 trên PowerShell.
- Dùng cache `data/cache/` thay vì tự chạy? **Không**, tự chạy trực tiếp các lệnh CLI trên máy (như ghi nhận đầy đủ trong `model_log.csv` và `runs.csv`).
- Lỗi / bug script đã gặp (nếu có): Trên môi trường Windows PowerShell, các chuỗi tham số có dấu phẩy hoặc dấu bằng cần được bao bọc cẩn thận bằng dấu ngoặc kép.
- Một điều bạn làm khác lần sau: Sẽ tuân thủ triệt để Bước 0 (ẩn nhãn máy để tự quét và đếm trước 15s) ngay từ những ảnh đầu tiên để không bị thiên kiến tự động hóa đánh lừa làm lọt bẫy missing.
