# Báo cáo Day 08 — 2A202603330 - Phạm Hữu Minh

> Mọi con số đều kèm **mẫu số** và lấy từ `qc/review_stats.md`, `qc/threshold_sweep.md`, `qc/qc_report_auto.md`.
> Không có số nào gõ tay. Cách dựng tập gold và các hạn chế của bài nằm ở mục 8; đọc mục đó trước khi dùng số ở mục 2 và 4.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | person 0.35 · motorbike 0.5 · còn lại mặc định 0.3 | `r01`, `r02`: detector danh mục cố định, 0.5–0.68 s/ảnh trên CPU |
| three_wheeler | YOLOE-26s-seg, prompt `tuk-tuk` / `cyclo` / `cargo tricycle` | 0.15 | `r03`: YOLO26 cho **0 box** `three_wheeler` trên cả 8 ảnh pilot |

- Lệnh merge: `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.35,motorbike=0.5,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM đã chọn: SAM 2.1 base (`sam2.1_b.pt`).** Máy chỉ có CPU. `sam2.1_b` chạy 1–3 s/vật, mask đủ tốt; `mobile_sam` nhanh hơn nhưng biên xấu; SAM 3 phải xin quyền HuggingFace nên bỏ.
- **Lỗi không sửa được bằng ngưỡng.** YOLO26 gọi tuk-tuk là `car`: ở img_012 **cả 4/4 chiếc** đều bị gọi `car`. COCO không có lớp xe ba bánh, nên nâng hay hạ ngưỡng đều không tạo ra lớp đó. Chỉ còn hai đường: model open-vocabulary, hoặc sửa tay.
- **Box điểm cao mà sai:** ở img_012 box `three_wheeler` duy nhất của YOLOE lại rơi vào **lan can sắt** (điểm 0.45). Ở img_003 có 8 box `three_wheeler` cho đúng 1 chiếc xe ba gác, các box thừa nằm trên ô dù và quầy hàng.
- **Một lỗi tôi từng quy oan cho model, phải nói rõ ở đây.** Ở vòng duyệt đầu tôi ghi rằng model hay sinh box `person` điểm 0.69–0.73 trên **tán lá cây** và box `car` trên **vỉa hè trống**, rồi còn viết thành quy tắc guideline. Sai. Mở `traps_key.json` sau khi export mới thấy đó đúng là ba **bẫy `extra`** do `inject_traps.py` chèn vào, không phải output của YOLO26. Bẫy `extra` được đặt ngẫu nhiên sao cho IoU < 0.05 với mọi box thật, nên nó luôn rơi vào nền: tán lá, tường trống, mặt đường. Chi tiết và hệ quả ở mục 6.
- **Giới hạn phải nói:** ngưỡng chọn trên **chính 4 ảnh gold** rồi chấm lại trên 4 ảnh đó (in-sample), nên precision/recall ở mục 2 là con số **lạc quan**, và 4 ảnh chỉ cho ước lượng thô.
- `bicycle`, `bus`, `truck` **không** được chọn ngưỡng riêng (chỉ 4, 1 và 0 vật trong gold, dưới mức 5 vật script yêu cầu). `car` thì **không ngưỡng nào ≤ 0.5 đạt recall 90%** (cao nhất 88%, trên 8 vật). Cả hai là kết quả đúng, không ép.

## 2. AI error profile (từ `qc/review_stats.md`)

Tính trên 446 box máy; script đã **loại 29 đối tượng quanh vùng bẫy** để lỗi nhân tạo không làm bẩn số đo chất lượng máy.

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 402 | 90.1% | 446 box máy |
| edited | 6 | 1.3% | 446 box máy |
| deleted | 38 | 8.5% | 446 box máy |
| added | 12 | 2.9% | 420 đối tượng sau duyệt |

Trong `deleted + added` có **5 ca đổi lớp** (box đúng chỗ, sai tên), nên số vật máy **sót thật** chỉ khoảng **7**.

**Lớp tệ nhất là `three_wheeler`, cách biệt rất xa:**

| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 245 | 1 | 2 | 1 |
| motorbike | 88 | 0 | 0 | 5 |
| car | 47 | 0 | 7 | 0 |
| **three_wheeler** | **11** | **5** | **23** | **2** |

41 box `three_wheeler` thì **23 bị xoá**, hơn một nửa. Chúng nằm trên quang gánh và sọt hàng (img_014), ô dù và bạt quầy (img_003, img_009), đám xe hai bánh (img_005), nền tối ban đêm (img_022). Truy ngược được về đúng một quyết định ở Part 3: ngưỡng 0.15 chọn để đạt recall 100% trên **6 vật** `three_wheeler` trong gold. Sáu vật quá ít để đặt ngưỡng, và cái giá hiện ra trên lô thật chứ không hiện ra lúc chọn.

**So với gold** (59 đối tượng, không tính `ignore_region`): precision **86.2%** (KTC 95%: 76–93%, 56 TP / 9 FP), recall **94.9%** (KTC 95%: 86–98%, 3 FN). Đọc khoảng tin cậy, và nhớ đây là số in-sample.

## 3. Thiên kiến tự động hóa và ảnh bẫy

**Bắt bẫy: 4/12 = 33.3%.** Ngưỡng đạt là 80%, nên **trượt rất xa**.

| Loại bẫy | Bắt được |
|---|---|
| `extra` (box ma) | 3/3 |
| `wrong_class` (sai lớp) | 1/3 |
| `missing` (xoá mất vật đúng) | 0/3 |
| `loose` (box lệch / lỏng) | 0/3 |

Bẫy lọt: img_005, img_027, img_033 (`loose`); img_017, img_024, img_035 (`missing`); img_010, img_034 (`wrong_class`).

**Hình dạng của kết quả quan trọng hơn con số.** Bắt 3/3 box ma — loại lỗi *thêm* một thứ không có. Bắt 0/3 box lệch và 0/3 vật bị xoá — hai loại lỗi *không thêm gì vào ảnh*. Mắt người duyệt bị kéo về chỗ **có vật lạ**, và gần như mù với chỗ **đáng lẽ phải có mà không có**, hoặc chỗ có rồi nhưng sai một chút. accepted 90.1% nghe thì giỏi; ghép với 0/3 `loose` thì nó chỉ có nghĩa là tôi bấm Accept nhanh.

Bước nào bị bỏ? Checklist v1.0 có bốn bước và ba bước đầu đều có thao tác rõ ràng. Bước "lỏng" không có thao tác nào. Bước không có thao tác là bước không được làm. v1.1 thêm bước 5 "soát cạnh" (`CAL-05`), và lần sau phải đo lại xem `loose` có khác 0/3 không.

**Hai điều nữa phát hiện khi mở `traps_key.json` sau lúc export:**
- img_034: box bị đổi lớp vốn là một **`person`** bị gán thành `bicycle`. Tôi xoá nó vì tưởng là box thừa chồng lên `motorbike` #468. Hậu quả: **một người thật đang thiếu** trong `batch_reviewed.json`. Tôi **không** sửa lại sau khi biết đáp án, để con số 4/12 giữ nguyên tính trung thực; việc sửa nằm trong danh sách Rework ở mục 7.
- img_010: tôi gộp hai nửa box thành một `three_wheeler` đúng và xoá nửa mang lớp `car`. Về nghĩa là đúng, nhưng `review_stats` vẫn tính là **lọt**, vì script bám theo annotation chứ không theo kết quả. Sửa bằng cách xoá rồi vẽ lại thì script không ghi nhận.

## 4. Máy có đáng dùng không?

**Có, nhưng chỉ cho 6 lớp COCO, và chỉ khi có bước soát cạnh.**

- `person`, `motorbike`, `car`: accepted 245/248, 88/93, 47/54 — gần như không phải sửa. Ở đây máy tiết kiệm thật.
- `three_wheeler`: 23/41 box phải xoá. Ở mức đó, lọc nhãn máy **tốn công hơn** tự khoanh 18 chiếc xe ba bánh trong 24 ảnh. Làm lại, tôi sẽ đặt ngưỡng `three_wheeler` khoảng 0.45 (xem `r03`: ngưỡng đó cắt 3 dương tính giả trên img_002 mà vẫn giữ box 0.93) và chấp nhận sót vài chiếc — sót thì còn thấy khi quét ảnh, còn 23 box rác thì làm mỏi mắt và chính nó đẻ ra thói bấm Accept hàng loạt.
- **Về thời gian: bài này không trả lời được.** `time_log.csv` ghi 5 phút cho 4 ảnh gold và 21 phút cho 24 ảnh duyệt, đo bằng mốc thời gian tệp, nhưng gold ở đây **dựng có máy hỗ trợ** (mục 8) chứ không gán tay từ đầu. So sánh "gán tay bao lâu / duyệt bao lâu" vì thế vô nghĩa.

**Ba quy tắc giảm thiên kiến** (mỗi quy tắc kiểm được bằng số hoặc việc làm):

1. **Soát bốn cạnh mọi box giữ lại.** Kiểm bằng số: `loose` phải khác 0/3 ở lô sau. Hai vòng liên tiếp đều 0/3, nên đây là quy tắc cấp thiết nhất.
2. **Đếm vật trong ảnh trước khi bật nhãn máy lên, rồi so với số box.** Kiểm bằng số: `missing` vòng này 0/3 — tôi chưa từng phát hiện một vật bị xoá nào, kể cả chiếc tuk-tuk to ở img_017. Đếm trước là cách duy nhất thấy được cái không có.
3. **Lớp nào có `deleted` vượt 30% thì dừng, sửa ngưỡng, nạp lại, không duyệt tiếp.** Kiểm bằng số: `three_wheeler` 23/41 = 56%, lẽ ra phải dừng sau 3–4 ảnh.

## 5. Mask SAM cho 3 vật (Part 5)

Tất cả dùng **SAM 2.1 base** (`sam2.1_b.pt`), prompt bằng click, mô phỏng Interactor của CVAT.

| Ảnh | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát được | File |
|---|---|---|---|---|
| img_013 (tuk-tuk xanh đỏ) | 3 dương + 2 âm | **Có, dù đã click âm lên người lái** | Mask 315 320 px (45.2% ảnh) phủ đúng cả xe nhưng tràn sang chân và tay người lái; click âm chỉ khoét một mảng quanh điểm click | `masks/tuktuk_img013.jpg` |
| img_002 (tuk-tuk xanh lá) | 2 dương + 2 âm | **Không** — người lái áo cam bị loại đúng | Mask sạch, biên bám thân xe | `masks/tuktuk_img002.jpg` |
| img_013 (xe máy bị che) | 1 dương | Không áp dụng | Chỉ ra **một mảnh** quanh điểm click: bánh sau và một phần ốp, bỏ hẳn đầu xe | `masks/motorbike_occluded_img013.jpg` |

**Đối chiếu guideline mục 4.2.** Mục 4.2 nói polygon bám biên phần nhìn thấy và vùng chạm thuộc vật phía trước, nhưng **không nói người ngồi trên xe có thuộc mask của xe hay không**, trong khi 4.1.4 nói rõ điều đó cho box. Thiếu sót lộ ra ngay ở img_013: SAM gộp người lái vào xe và guideline không có câu nào để bác. Quy tắc rút ra: **mask của xe không bao giờ gồm người**, chiếu theo 4.1.4.

Lỗi mask rút ra: một click âm **không** loại được cả một vật, nó chỉ đẩy biên ra khỏi vùng quanh điểm click; muốn loại hẳn người lái phải click âm nhiều điểm dọc theo người. Với vật bị che nhiều, mỗi mảnh nhìn thấy cần một click dương riêng.

Đối chiếu với `r04`: cùng chiếc tuk-tuk img_013, khi đưa **box** của YOLO26 cho SAM thì mask chỉ phủ mái che và chắn bùn, bỏ cả thân xe. Cùng model, cùng ảnh, khác cách ra lệnh: **click tốt hơn hẳn box** khi vật có khung rỗng nhìn xuyên qua được.

## 6. Calibration (Part 6)

- **6 quyết định** trong `qc/calibration_decisions.csv` (`CAL-01` … `CAL-06`): 2 từ câu hỏi mở của v1.0, 2 từ lỗi lặp của máy, 1 từ bẫy lọt, 1 từ box `can_review`.
- Quan trọng nhất là **`CAL-05`**: thêm bước 5 "soát cạnh". Đây là quyết định duy nhất mà số đo *chứng minh* được là cần, và hai vòng đo liên tiếp đều cho 0/3.
- Đã chốt 2/7 câu hỏi mở (câu 2 `ignore_region`, câu 3 hàng nhô ra). **Năm câu còn lại ghi rõ là chưa chốt**: lô này không có ca nào buộc phải quyết.
- **Bài học lớn nhất của mục này: suýt nữa tôi viết một quy tắc cho lỗi không tồn tại.** Ở vòng đầu tôi thấy ba box điểm cao nằm trên tán lá và vỉa hè trống, kết luận "model hay nhận nhầm tán lá là người", và đưa vào `CAL-04`. Sau khi export và mở `traps_key.json` mới biết cả ba là bẫy `extra` do script chèn. `inject_traps.py` đặt box ma ở vị trí ngẫu nhiên với IoU < 0.05 so với mọi box thật, nên nó **luôn** rơi vào nền — và nền thì hay là tán lá, tường, mặt đường. Vòng hai lặp lại y hệt: box ma rơi vào tán lá (img_004), tường trống (img_009), dãy đèn lồng (img_025).
  Hệ quả cho người làm bài: **không thể phân biệt box ma do script chèn với dương tính giả thật của model khi chỉ nhìn ảnh.** Ai gặp box ma trước lúc mở đáp án sẽ quy cho model và viết quy tắc guideline cho một hành vi model không hề có. `CAL-04` đã được sửa lại theo đúng sự thật này.
- **Phân biệt "guideline mơ hồ" / "máy sai" / "tôi sai":**
  - *Guideline mơ hồ:* hàng nhô ra có nằm trong box xe không (`CAL-01`), hàng xe đỗ sát nhau có thành `ignore_region` không (`CAL-02`), mask xe có gồm người lái không (mục 5). v1.0 không trả lời được, nên không ai sai — phải viết thêm luật.
  - *Máy sai:* tuk-tuk bị gọi `car` ở cả 4/4 chiếc trong img_012; 23/41 box `three_wheeler` là rác ở ngưỡng 0.15. Guideline đã rõ, model nhìn nhầm. Xoá và sửa ngưỡng, không sửa luật.
  - *Tôi sai:* lọt 0/3 `loose`, 0/3 `missing`, và xoá nhầm một `person` ở img_034. Guideline rõ, model không lừa, quy trình duyệt của tôi thiếu bước.

## 7. QC và quyết định (Part 6)

- **Mẫu:** 8/24 ảnh (3 ảnh có `can_review` + 5 ngẫu nhiên, seed 42), **142 đối tượng**. Người kiểm: chính tôi — **hạn chế lớn nhất của mục này**, vì tôi vừa là người duyệt.
- **Lỗi theo đối tượng: 4/142 = 2.8%.** **Lỗi theo ảnh: 4/8 = 50.0%** (ngưỡng Release ≤ 2% theo ảnh).
- Bốn lỗi, hai loại: `sai_vi_tri_kich_thuoc` ở img_006 và img_026 (một box trùm nhiều xe máy đỗ sát nhau — đã thành luật `CAL-02`); `bo_sot` ở img_017 và img_035 (hai vật bị bẫy `missing` xoá mà lượt duyệt không phát hiện).
- **Giới hạn mẫu nhỏ:** 4 lỗi trên 8 ảnh cho cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **78%**. Với 8 ảnh thì **không thể chứng minh** tỉ lệ dưới 2%, kể cả khi mẫu sạch hoàn toàn. Con số 50% cũng không nên đọc như ước lượng chắc chắn — nó đủ để nói "chưa đạt", không đủ để nói "tệ đến đâu".
- Script kiểm được: số lượng, lớp lạ, ngưỡng 20px, trùng ảnh, định dạng. Script **không** kiểm được: box có ôm sát không, có cùng bỏ sót với máy không, checklist có thật sự được làm không. Ba thứ đó đúng là chỗ bài này hỏng, nên "validate OK" không có nghĩa là "nhãn tốt".

**Quyết định: Rework**, ngày 2026-11-17.

Theo cây Hold → Rework → Release: không phải **Hold**, vì dữ liệu không hỏng và guideline đã đủ rõ để làm tiếp. Là **Rework**, vì hai điều kiện độc lập cùng trượt: tỉ lệ ảnh lỗi **50% > 2%**, và bắt bẫy **33% < 80%**. Trượt bắt bẫy nặng hơn: nó nói rằng con số 2.8% lỗi theo đối tượng ở trên *cũng* không đáng tin, vì do chính người đã bỏ lọt 8/12 bẫy chấm.

**Việc cần làm tiếp, theo thứ tự:**
1. Khôi phục box `person` ở img_034 (407,391,446,471) mà tôi xoá nhầm.
2. Thêm lại hai vật bị bẫy `missing` lấy mất: tuk-tuk ở img_017 và người ở img_035.
3. Nâng ngưỡng `three_wheeler` lên khoảng 0.45, nạp lại pre-label.
4. Duyệt lại cả 24 ảnh theo checklist v1.1 **có bước 5 soát cạnh** và bước đếm vật trước khi bật nhãn.
5. Tách box trùm nhiều xe ở img_006 và img_026 theo `CAL-02`.
6. Chạy lại `review_stats` với bộ bẫy mới; chỉ Release khi bắt bẫy đạt từ 80% trở lên.

## 8. Ghi chú cá nhân

- **Tập gold dựng có máy hỗ trợ.** Quy trình: chạy YOLO26m và YOLOE trên 4 ảnh gold, lấy kết quả làm nháp, rồi sửa tay từng box theo label contract — 24 chỗ sửa, ghi đủ trong `sample-notes/gold-corrections/`. Theo `RULES.md` đây là **gold bị nhiễm**: tôi khai báo đúng như mục "Gold phải sạch" yêu cầu. Hệ quả khi đọc bài: precision/recall ở mục 2 đo "máy so với chính nó đã được sửa", nên **lạc quan hơn** số của một tập gold gán tay thật.
- **Công cụ AI đã dùng:** Claude (Anthropic) để chạy script, đọc ảnh và soạn báo cáo. Cách kiểm: mọi con số đều do script trong repo sinh ra (`review_stats.py`, `threshold_sweep.py`, `qc_report.py`) rồi đối chiếu lại với JSON/CSV tương ứng; mọi nhận xét về lỗi của máy đều kèm ảnh và id box. Không có số nào gõ tay. Một kết luận ở vòng đầu **đã bị chính dữ liệu bác bỏ** và được sửa công khai ở mục 1 và mục 6.
- **Hạt giống bẫy không phải mặc định.** Lệnh đã chạy: `python scripts/inject_traps.py --pre work/prelabel/merged_review/pre.coco.json --student 2A202603330 --seed 7`. Lý do: ở lần tạo bẫy trước (seed mặc định 0) tôi đã lỡ in ra số box từng ảnh của hai bộ bẫy khác nhau, mà chỉ cần so hai dãy số đó là biết ảnh nào chứa bẫy `extra` và `missing` — vì chỉ hai loại đó làm đổi số đếm. Biết trước như vậy thì tỉ lệ bắt bẫy không còn là phép đo. Đổi sang `--seed 7` để vô hiệu hoá phần đã lỡ biết. Đây cũng là một lỗ hổng của chính cơ chế bẫy: **so sánh file là đủ để lộ đáp án mà không cần mở `traps_key.json`.**
- **Không dùng `data/cache/`.** Tự chạy model. Đối chiếu: `--split all --conf 0.05` cho **1669 box**, trùng khớp số lượng với `data/cache/yolo26m_all_c005.coco.json` của giảng viên.
- **Không dùng CVAT cho bài này.** Nhãn ghi thẳng ra COCO 1.0 bằng chính `build_coco` và `make_ann` của repo, nên định dạng giống bản export từ CVAT và mọi script đều nhận.
- **Máy:** Windows 11, Python 3.13.2, torch 2.15.0+cpu, ultralytics 8.4.215, **không GPU**. YOLO26m 0.5 s/ảnh, YOLOE 0.26 s/ảnh, SAM 2.1 b 1–3 s/vật.
- **Vướng mắc cài đặt:** `check_env.py` báo FAIL vì thiếu `transformers`, trong khi `transformers` chỉ dùng cho bài nâng cao. Ai chỉ làm bài chính vẫn thấy một dòng FAIL, bỏ qua được.
- **`time_log.csv` đo bằng mốc thời gian tệp**, không phải bấm đồng hồ; cột `note` ghi rõ. Vì gold có máy hỗ trợ nên hai số đó không dùng để so "gán tay với duyệt" (mục 4).
- **Lần sau sẽ làm khác:** đếm vật trong ảnh trước khi bật nhãn máy. Cả hai vòng đo đều cho `missing` 0/3 và `loose` 0/3, tức là tôi chưa bao giờ thật sự nhìn ảnh trước — chỉ nhìn box rồi quyết.
