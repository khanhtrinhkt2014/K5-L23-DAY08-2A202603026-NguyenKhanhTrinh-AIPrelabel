# Thông tin học viên

- Họ và tên: Nguyễn Khánh Trình
- MSHV: 2A202603026
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: khanhtrinhkt2014
- Repo bài nộp (URL): https://github.com/khanhtrinhkt2014/K5-L23-DAY08-2A202603026-NguyenKhanhTrinh-AIPrelabel
- Commit chốt bài (hash): (điền sau commit cuối)
- Máy: Win11, HP Laptop 17t-cn300, không GPU (chạy CPU)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 397 / 84 / 96 (mẫu số: 618 box máy) = 64.2% / 13.6% / 15.5%; added: 52 (mẫu số: 542 đối tượng sau duyệt) = 9.6%
- So với gold: precision / recall của **máy** 38/65 = 58.5% / 38/43 = 88.4%
- Bắt bẫy: 12/12 = 100%
- Thời gian: gán tay 3.75 phút/ảnh (15 phút / 4 ảnh); duyệt có pre-label 1.25 phút/ảnh (30 phút / 24 ảnh)
- Lỗi QC: 80% theo ảnh (4/5), 5% theo đối tượng (5/100)
- **Quyết định:** Hold — lý do một dòng: guideline v1.1 còn 1 câu hỏi mở (xe đạp điện); chốt xong vẫn phải Rework vì 4/5 ảnh QC có lỗi (> 2%).
