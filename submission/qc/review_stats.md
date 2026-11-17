# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 629; đối tượng sau duyệt: 604; ngưỡng accepted: IoU >= 0.9
- Đã **loại 18 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 577 | 577/629 = 91.7% | 629 box máy |
| edited (sửa) | 10 | 10/629 = 1.6% | 629 box máy |
| deleted (xoá) | 42 | 42/629 = 6.7% | 629 box máy |
| added (vẽ thêm) | 17 | 17/604 = 2.8% | 604 đối tượng sau duyệt |

Trong số deleted + added, **8 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 9.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 362 | 4 | 10 | 2 |
| bicycle | 7 | 0 | 1 | 2 |
| motorbike | 137 | 2 | 1 | 4 |
| car | 47 | 0 | 6 | 3 |
| bus | 3 | 0 | 0 | 2 |
| truck | 3 | 0 | 3 | 3 |
| three_wheeler | 18 | 4 | 21 | 1 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 41 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 37 | 36 | 4 | 50.7% (KTC 95%: 39–62%) | 90.2% (KTC 95%: 77–96%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 7/12 = 58.3% (KTC 95%: 32–81%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 2/3
  - loose: 2/3
  - wrong_class: 2/3
  - missing: 1/3
  - lọt: img_005.jpg (extra), img_031.jpg (loose), img_029.jpg (wrong_class), img_028.jpg (missing), img_013.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
