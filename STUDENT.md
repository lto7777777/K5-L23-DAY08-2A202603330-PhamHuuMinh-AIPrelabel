# Thông tin học viên

- Họ và tên: Phạm Hữu Minh
- MSHV: 2A202603330
- Lớp (L2 / L3 / khác): L2B
- Tài khoản GitHub: lto7777777
- Repo bài nộp (URL): https://github.com/lto7777777/K5-L23-DAY08-2A202603330-PhamHuuMinh-AIPrelabel
- Commit chốt bài (hash): (bản mẫu, không nộp)
- Máy: Windows 11, Python 3.13.2, torch 2.15.0+cpu, không GPU (chạy CPU toàn bộ)
- CVAT (Docker): không dùng cho bản mẫu này; nhãn ghi thẳng ra COCO 1.0 (xem REPORT.md mục 8)

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 404 / 7 / 40 (mẫu số: 451 box máy); added: 12 (mẫu số: 423 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 86.2% / 94.9% (mẫu số: 59 đối tượng gold, 4 ảnh, in-sample nên lạc quan)
- Bắt bẫy: 8/12 = 66.7% (loose 0/3, extra 3/3, missing 2/3, wrong_class 3/3)
- Thời gian: gold 1.2 phút/ảnh; duyệt có pre-label 0.9 phút/ảnh — đo bằng mốc thời gian tệp, KHÔNG so sánh được với số của học viên (REPORT.md mục 4)
- Lỗi QC: 25.0% theo ảnh (2/8), 1.3% theo đối tượng (2/153)
- **Quyết định:** Rework — lý do một dòng: tỉ lệ ảnh lỗi 25% vượt ngưỡng 2% và bắt bẫy 67% dưới ngưỡng 80%, nên chính lượt duyệt chưa đáng tin.
