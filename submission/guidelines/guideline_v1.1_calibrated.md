# Label contract — Phố châu Á — BỔ SUNG v1.1 — SAMPLE - Bài mẫu tham khảo

> Chỉ ghi phần thay đổi so với v1.0. Mỗi ca đều có ảnh ví dụ và khớp một dòng trong `qc/calibration_decisions.csv`.
> Nguồn của các thay đổi: 37 box bị xoá và 8 box bị sửa trong lô 24 ảnh, 4 bẫy lọt, và 13 ảnh có box `can_review`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày đặc **gần** camera: từng xe hay `ignore_region`? Ngưỡng cụ thể? | Tiêu chí là **khe hở**, không phải khoảng cách. Thấy được khe hở giữa hai xe liền nhau thì gán từng xe. Cả một hàng đỗ sát nhau không thấy khe hở thì vẽ **một** `ignore_region` cho cả hàng. Không giữ một box trùm nhiều xe. | img_026 (bãi đỗ ban đêm, khoảng 40 xe), img_006 (hàng xe trong ngõ) | CAL-02 |
| 3 | Hàng nhô ra: box xe có gồm hàng không? | **Không.** Box bám thân xe. Ngoại lệ duy nhất: giỏ hoặc thùng **gắn cố định** vào khung xe thì thuộc xe. Hệ quả quan trọng: **chọn lớp bằng cách đếm bánh nhìn thấy**, không bằng hình khối của đống hàng. | img_009 (xe hai bánh chở chuối, máy gọi `three_wheeler`) | CAL-01 |
| 1, 4, 5, 6, 7 | Xe đạp điện; ngưỡng "còn thấy 20%"; kính chắn gió trong mask; box chồng nhau ảnh đêm; mức `conf` tối thiểu | **Chưa chốt.** Lô 24 ảnh này không có ca nào buộc phải quyết: không gặp xe đạp điện rõ ràng, và bài chính chỉ làm mask cho 3 vật nên chưa đủ cơ sở cho câu 5. Để lại cho v1.2 thay vì chốt bừa. | — | — |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì | Quyết định | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_003 | Xe ba gác đỗ dưới mái bạt của quầy hàng | Gộp xe và mái bạt thành một `three_wheeler`; tổng cộng 8 box `three_wheeler` cho 1 xe thật | Mái bạt là công trình của quầy, không phải khung mái của xe. Box chỉ ôm thùng xe và ba bánh | Mục 2, dòng `three_wheeler`: phân biệt "khung mái **gắn liền xe**" với mái che, ô dù, bạt của quầy hàng |
| img_003, img_014 | Quang gánh, sọt hàng, ô dù, bạt quầy | `three_wheeler` rơi vào đồ vật tĩnh: 23 trên 41 box của lớp này là rác ở ngưỡng 0.15 | Xoá, và nâng ngưỡng lớp này lên khoảng 0.45 ở lô sau | Mục 7: trước khi ghi một lỗi vào calibration như lỗi của máy, phải loại trừ khả năng đó là bẫy `extra` (box nằm hoàn toàn trên nền, không chồng vật thật nào) |
| img_024 | Ảnh phơi sáng lâu ban đêm | 5 box `car` và `truck` nằm trên **vệt đèn xe**, không có thân xe nào | Xoá cả 5 | Mục 4.1.7: bổ sung "vệt sáng do phơi sáng lâu" vào danh sách không gán, cùng nhóm với hình in, tượng, phản chiếu |
| img_010, img_013, img_035 | Xe ba bánh có mái | Trả về **hai nửa box** chồng nhau: một nửa mái, một nửa gầm và bánh | Gộp thành một box phủ cả xe | Mục 4.1.1: thêm lưu ý rằng với xe có mái, máy hay tách mái khỏi thân, phải kiểm tra xem có hai box cho một xe không |
| img_012 | Bốn tuk-tuk trong một ảnh | Cả 4 đều bị gọi `car`; box duy nhất YOLOE gán `three_wheeler` lại rơi vào **lan can** | Sửa cả 4 thành `three_wheeler`, xoá box lan can | Mục 2: giữ nguyên định nghĩa nhưng ghi rõ đây là lỗi hệ thống của model COCO, phải soát **từng** xe chứ không tin điểm số |
| img_036 | Xe đạp và xe máy ở xa | 3 box không phân biệt được lớp | Có bình xăng, ống xả hoặc biển số thì `motorbike`; chỉ thấy khung mảnh và hai bánh thì `bicycle`; không thấy gì thì giữ lớp tin nhất **và** tick `can_review` | Mục 4.3: thay hướng dẫn chung bằng ba dấu hiệu cụ thể |

## 3. Cảnh báo: box ma của script trông y hệt dương tính giả của model

Bản v1.1 đầu tiên của tài liệu này có một quy tắc viết cho lỗi **không tồn tại**. Tôi thấy ba box điểm cao
nằm trên tán lá cây và vỉa hè trống, kết luận "model hay nhận nhầm tán lá là người", rồi đưa vào `CAL-04`.
Mở `traps_key.json` sau khi export mới biết cả ba là bẫy `extra` do `inject_traps.py` chèn.

Nguyên nhân: bẫy `extra` được đặt ở vị trí ngẫu nhiên sao cho IoU < 0.05 với **mọi** box thật, nên nó luôn
rơi vào nền — và nền thì hay là tán lá, tường, mặt đường. Nhìn ảnh thì không tài nào phân biệt được nó với
một dương tính giả thật của model.

**Quy tắc:** chỉ ghi một lỗi vào calibration như "lỗi của máy" khi nó **lặp trên nhiều ảnh** và **vẫn đúng
sau khi mở đáp án bẫy**. Một box đơn lẻ nằm trọn trên nền thì khoan kết luận.

## 3. Thay đổi quy trình duyệt (rút ra từ bẫy lọt)

Đây là thay đổi quan trọng nhất của v1.1, và nó đến từ số đo chứ không từ cảm nhận.

Kết quả bẫy: **4/12 = 33%**. Theo loại: `extra` 3/3, `wrong_class` 1/3, **`missing` 0/3**, **`loose` 0/3**.

Hình dạng của kết quả nói nhiều hơn con số. Bắt hết loại bẫy **thêm** một thứ không có vào ảnh, và không bắt được loại bẫy **không thêm gì**: box lệch, hoặc vật bị xoá mất. Mắt người duyệt bị kéo về chỗ có vật lạ, và gần như mù với chỗ đáng lẽ phải có mà không có. Checklist 4 bước của v1.0 không có bước nào buộc soát cạnh, cũng không có bước nào buộc đếm vật trước.

**Bước 5 mới, bắt buộc — soát cạnh:** với mỗi box được giữ lại, sau khi xác nhận lớp, nhìn lần lượt **cạnh trên, cạnh dưới, cạnh trái, cạnh phải** so với mép vật. Không Accept một box chỉ vì lớp của nó đúng.

**Bước 0 mới, bắt buộc — đếm trước khi nhìn nhãn:** ẩn nhãn máy, đếm số vật trong ảnh, rồi mới bật nhãn lên và so. Đây là cách duy nhất thấy được vật đã bị xoá mất: `missing` 0/3 nghĩa là chưa lần nào tôi thật sự nhìn ảnh trước.

Ảnh ví dụ bẫy `loose` lọt: img_005, img_027, img_033. Bẫy `missing` lọt: img_017 (nguyên một chiếc tuk-tuk), img_024, img_035 (`case_id` CAL-05).

## 4. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (`case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu hỏi mở 2 và 3; nói rõ 5 câu còn lại chưa đủ cơ sở để chốt | CAL-01, CAL-02 |
| v1.1 | Mái quầy hàng không thuộc box xe; thêm vệt đèn phơi sáng vào danh sách không gán; cảnh báo xe có mái hay bị tách làm hai box | CAL-03 và các ca img_024, img_010 ở mục 2 |
| v1.1 | Thêm lượt quét dương tính giả (quang gánh, sọt hàng, tán lá) vào mục 7 | CAL-04 |
| v1.1 | **Thêm bước 5 "soát cạnh" vào checklist duyệt** | CAL-05 |
| v1.1 | Ba dấu hiệu phân biệt `bicycle` và `motorbike` ở xa | CAL-06 |
