# Thông tin học viên

- Họ và tên: Nguyễn Thiên Phúc
- MSHV: 2A202603339
- Lớp (L2 / L3 / khác): L2B
- Tài khoản GitHub: phucnt31
- Repo bài nộp (URL): https://github.com/phucnt31/K5-L2B-NguyenThienPhuc-03339-Day08-AI-Assisted-Labeling.git
- Commit chốt bài (hash): de451e2
- Máy: Windows 11, CPU (hỗ trợ CUDA)
- CVAT (Docker): v2.81.0 (web app)

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 577 (91.7%) / 10 (1.6%) / 42 (6.7%) (mẫu số: 629 box máy); added: 17 (2.8%) (mẫu số: 604 đối tượng sau duyệt)
- So với gold: precision / recall của **máy**: 50.7% / 90.2% (mẫu số: 41 đối tượng gold)
- Bắt bẫy: 7/12 = 58.3%
- Thời gian: gán tay 7.0 phút/ảnh; duyệt có pre-label 2.54 phút/ảnh
- Lỗi QC: 16.7% theo ảnh (1/6), 1.0% theo đối tượng (1/101)
- **Quyết định:** REWORK — lý do một dòng: Tỉ lệ ảnh lỗi QC (16.7% > 2%) và tỉ lệ bắt bẫy (58.3% < 80%) chưa đạt chuẩn nghiệm thu.
