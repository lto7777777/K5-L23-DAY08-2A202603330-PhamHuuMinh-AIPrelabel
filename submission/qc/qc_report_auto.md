# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 8 ảnh (3 ca khó + 5 ngẫu nhiên), 142 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 4/142 = 2.8% | 142 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 4/8 = 50.0% | 8 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 402/446 = 90.1% / 6/446 = 1.3% / 38/446 = 8.5% | 446 box máy |
| Máy: added | 12/420 = 2.9% | 420 đối tượng sau duyệt |
| Bắt lỗi bẫy | 4/12 = 33.3% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| sai_vi_tri_kich_thuoc | 2 | img_006.jpg, img_026.jpg |
| bo_sot | 2 | img_017.jpg, img_035.jpg |

## Giới hạn thống kê
- 4 ảnh lỗi trong 8 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **78%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 50.0% > 2%
- bắt bẫy 33% < 80% -> lô duyệt chưa đáng tin

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
