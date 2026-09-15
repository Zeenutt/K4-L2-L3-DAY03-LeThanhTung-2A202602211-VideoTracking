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
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13` |
| weights / hai tracker | `yolo26n.pt / ByteTrack control và BoT-SORT + ReID treatment` |
| conf / IoU / imgsz / classes | `0.25 / 0.7 /  960 / 2, 5, 7` |
| device | `0` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.855 | 0.846 | 0.866 | 0.881 | 0.992 | 0.984 | 0.869 | 5 | 4 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.820 | 0.762 | 0.883 | 0.915 | 0.908 | 0.808 | 0.907 | 86 | 22 | 2 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`MOTA của tôi thấp hơn IDF1: MOTA = 0.984 trong khi IDF1 = 0.992. Cả hai đều rất cao, cho thấy annotation có ít lỗi detection và association. Nếu MOTA cao nhưng IDF1 thấp, điều đó có thể cho thấy hệ thống phát hiện đối tượng tốt nhưng vẫn gặp vấn đề trong việc duy trì đúng danh tính của từng track. MOTA không phạt nặng lỗi ID vì MOTA chủ yếu phản ánh FP, FN và IDSW, trong khi IDF1 tập trung vào mức độ chính xác của việc duy trì identity qua các frame. Vì vậy, một số lỗi identity có thể không làm MOTA giảm nhiều nhưng vẫn làm IDF1 giảm đáng kể.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`ByteTrack control có IDF1 = 0.875, AssA = 0.776 và IDSW = 2. BoT-SORT + ReID có IDF1 = 0.900, AssA = 0.820 và IDSW = 2. Như vậy ReID treatment có IDF1 cao hơn 0.025 và AssA cao hơn 0.044, nhưng số IDSW không thay đổi, đều bằng 2. Điều này cho thấy BoT-SORT + ReID có association tốt hơn tổng thể nhưng không loại bỏ được các lần đổi ID. Một sequence đáng chú ý là frame 80–100, liên quan đến vehicle/GT track 5. Diagnostics cho thấy ReID có một ID switch tại frame 87: GT track 5 chuyển từ prediction track 17 sang track 18. GT track 5 được phân mảnh thành hai prediction track, trong đó track 17 xuất hiện 1 frame và track 18 xuất hiện 52 frame trên tổng chiều dài GT là 59 frame. Đây là vùng cần chú ý khi xe bị che khuất và xuất hiện lại. Tuy nhiên, không thể kết luận rằng sự khác biệt hoàn toàn do ReID, vì control dùng ByteTrack còn treatment dùng BoT-SORT + ReID. Hai tracker implementation khác nhau nên đây không phải phép đo causal effect riêng của ReID.`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`So với ByteTrack control, BoT-SORT + ReID có DetA tăng từ 0.649 lên 0.711. FN giảm đáng kể từ 54 xuống 26, trong khi FP tăng nhẹ từ 88 lên 91. Điều này cho thấy treatment có khả năng giảm bỏ sót đối tượng tốt hơn, nhưng vẫn tạo ra một số false positive. So với annotation của tôi, ReID vẫn có FP = 86 và FN = 22, trong khi bản của tôi chỉ có FP = 5 và FN = 4. Vì vậy model vẫn còn lỗi detection đáng kể. Đồng thời, ReID có 2 IDSW và một số GT track bị phân mảnh, nên vẫn còn lỗi association. Có thể kết luận lỗi còn lại đến từ cả detector và association, không chỉ một phía.`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`Frame 87, GT track 5 là một ví dụ rõ ràng. Diagnostics cho thấy ReID đổi từ prediction track 17 sang track 18 tại frame 87. GT track 5 bị phân mảnh thành hai prediction track: track 17 chỉ xuất hiện 1 frame, còn track 18 xuất hiện 52 frame trong khi GT track 5 dài 59 frame. Vì annotation của tôi giữ một identity nhất quán cho track 5 và bản ReID xảy ra ID switch tại frame 87, đây là evidence cho thấy annotation của tôi đúng hơn ở đoạn này.`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`Frame 112, GT track 6 là điểm tôi cần xem lại annotation. ReID xảy ra ID switch từ prediction track 28 sang track 31 tại frame 112. GT track 6 được phân mảnh thành hai prediction track, trong đó track 28 xuất hiện 1 frame và track 31 xuất hiện 44 frame trên tổng chiều dài GT là 55 frame. Ngoài ra, tại frame 112, IoU giữa GT track 6 và prediction track 31 chỉ là 0.537, khá sát ngưỡng IoU 0.5. Vì vậy tôi nên kiểm tra lại bbox của ID 6 quanh frame 107–112 để xác định liệu annotation có bị lệch bbox hay việc đổi track hoàn toàn là lỗi của model. Tuy nhiên, evidence hiện tại nghiêng về lỗi model/association vì diagnostics xác định rõ ID switch tại frame 112 và không có missed GT track.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`Nếu phải gán thêm 10 clip, tôi sẽ sửa GUIDELINE_MINI.md theo hướng cụ thể hơn về các trường hợp dễ nhầm ID và bbox. Đặc biệt, tôi sẽ bổ sung rõ quy tắc xác định thời điểm bắt đầu/kết thúc track đối với xe xuất hiện ở rìa ảnh, xe rất nhỏ hoặc bị che khuất; quy tắc giữ ID khi xe bị che và cách xử lý khi hai xe chồng lấn/cắt nhau để tránh hoán đổi ID. Tôi cũng sẽ ghi thêm các ví dụ frame thực tế từ những clip đã gán để người gán nhãn sau có thể áp dụng thống nhất.`

`Về quy trình làm việc, tôi sẽ:`

`Kiểm tra và thống nhất guideline trước khi bắt đầu mỗi clip.`
`Khi gặp ca mơ hồ, ghi ngay frame và ID vào guideline thay vì quyết định theo cảm tính.`
`Đặt keyframe dày hơn tại các đoạn xe xuất hiện/biến mất, bị che khuất, cắt nhau hoặc bbox thay đổi rõ rệt.`
`Sau khi hoàn thành mỗi clip, kiểm tra lại ID ở đầu/cuối track và một số frame giữa để phát hiện ID swap hoặc bbox bị trôi.`
`Thực hiện cross-check sớm với người khác để phát hiện lỗi trước khi hoàn tất toàn bộ dữ liệu.`
`Sau mỗi lần kiểm tra hoặc chấm gold, cập nhật lại guideline nếu phát hiện một trường hợp chưa được quy định rõ.`

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
