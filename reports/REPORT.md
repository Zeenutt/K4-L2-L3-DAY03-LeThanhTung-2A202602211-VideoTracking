# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Lê Thanh Tùng`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | `10` phút |
| Thời gian gán `clip_01` | `45` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `3.07` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `Vehicle 5 (ô tô con) bị che khuất rồi xuất hiện ngay cạnh vehicle 4 (xe bus): ngay khi vehicle 5 hiện đầu xe thì sẽ gán box cho nó`
2. `Vehicle 8 (ô tô con) bị che khuất rồi xuất hiện ngay cạnh vehicle 4 (xe bus): ngay khi vehicle 5 hiện nóc xe thì sẽ gán box cho nó`
3. `Vehicle 7 đột ngột xuất hiện ở rìa ảnh ở 1 khoảnh khắc rất nhỏ: khi xuất hiện đầu xe thì gán box để track luôn`

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `...`
- Lượt 2: `Đầu: Vehicle 1+2+3; cuối: Vehicle 2+6`
- Lượt 3: `Vehicle 2+4+5+8`

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

- `Khi xe bị che khuất hoặc xuất hiện rất nhỏ: cần thống nhất rõ thời điểm bắt đầu/kết thúc track. Chỉ tạo bbox khi có đủ bằng chứng xác định đó là xe bốn bánh; khi xe bị che nhưng vẫn còn nhận diện được thì giữ ID, không tự ý tạo ID mới.`
- `Khi hai xe chồng lấn hoặc cắt nhau: cần theo dõi đặc điểm, vị trí trước và sau vùng chồng lấn để đảm bảo ID không bị hoán đổi. Nếu không thể xác định chắc chắn xe nào là xe nào sau vùng che, ưu tiên tạo track mới thay vì gán nhầm ID cũ.`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `5c8b789164dd10b48e447dc4b6254c6b572fb4a456d99a7137f878ece91cc552` |
| Thời điểm khóa | `2026-09-15T04:04:15.847237+00:00` |
| Số row / frame / track trước khi mở reference | `583 rows, 190 frames, 8 tracks` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.845 | 0.835 | 0.856 | 0.880 | 0.986 | 0.972 | 0.868 | 13 | 3 | 0 |
| Sau rework | 0.855 | 0.846 | 0.866 | 0.881 | 0.992 | 0.984 | 0.868 | 5 | 4 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| BBOX TREO / BBOX THỪA | 93-100 | 6 | Bấm outside đúng frame xe rời khung |
| BBOX TRÔI | 81 + 95-97 | 5 | Chỉnh lại BBOX (thêm keyframe) |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `...` |
| weights / hai tracker | `...` |
| conf / IoU / imgsz / classes | `...` |
| device | `...` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | | | | | | | | | | |
| ByteTrack control vs gold | | | | | | | | | | |
| BoT-SORT + ReID vs gold | | | | | | | | | | |
| ReID vs bạn | | | | | | | | | | |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`...`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`...`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`...`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`...`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`...`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`...`

## 7. Tệp đã nộp

- [ ] `annotations/clip_01/gt.txt`
- [ ] `annotations/clip_02/gt.txt`
- [ ] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [ ] `GUIDELINE_MINI.md` đã điền
- [ ] `outputs/eval_vs_gold.json`
- [ ] `outputs/model_bytetrack_clip_01.txt`
- [ ] `outputs/model_reid_clip_01.txt`
- [ ] `outputs/model_run_config.json`
- [ ] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [ ] `reports/REPORT.md` (file này)
