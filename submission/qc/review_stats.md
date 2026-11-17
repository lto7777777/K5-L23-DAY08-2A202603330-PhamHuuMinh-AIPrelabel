# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 446; đối tượng sau duyệt: 420; ngưỡng accepted: IoU >= 0.9
- Đã **loại 29 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 402 | 402/446 = 90.1% | 446 box máy |
| edited (sửa) | 6 | 6/446 = 1.3% | 446 box máy |
| deleted (xoá) | 38 | 38/446 = 8.5% | 446 box máy |
| added (vẽ thêm) | 12 | 12/420 = 2.9% | 420 đối tượng sau duyệt |

Trong số deleted + added, **5 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 7.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 245 | 1 | 2 | 1 |
| bicycle | 5 | 0 | 3 | 2 |
| motorbike | 88 | 0 | 0 | 5 |
| car | 47 | 0 | 7 | 0 |
| bus | 3 | 0 | 0 | 2 |
| truck | 3 | 0 | 3 | 0 |
| three_wheeler | 11 | 5 | 23 | 2 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 59 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 56 | 9 | 3 | 86.2% (KTC 95%: 76–93%) | 94.9% (KTC 95%: 86–98%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 4/12 = 33.3% (KTC 95%: 14–61%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 1/3
  - extra: 3/3
  - missing: 0/3
  - loose: 0/3
  - lọt: img_034.jpg (wrong_class), img_024.jpg (missing), img_027.jpg (loose), img_010.jpg (wrong_class), img_017.jpg (missing), img_033.jpg (loose), img_035.jpg (missing), img_005.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
