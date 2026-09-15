# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Lê Thanh Tùng`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): `...`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | `Vì trong quá trình ấy vẫn là xe đó nên cần gán đúng và track đúng` |
| Xe bị che lâu hơn ngưỡng trên | `Tạo ID mới khi xe xuất hiện lại` | `Khi bị che quá lâu, khó đảm bảo đó vẫn là cùng một xe nên tạo ID mới để tránh gán nhầm danh tính` |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | `Khi xe đã rời khỏi khung hình, không thể đảm bảo danh tính khi quay lại nên tạo ID mới` |
| Hai xe cắt nhau / chồng lên nhau | `Giữ nguyên ID của từng xe, không đổi ID khi chúng chồng lấn/cắt nhau` | `Mỗi xe vẫn là một đối tượng riêng; cần theo dõi đặc trưng/vị trí của từng xe để tránh hoán đổi ID` |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `bbox ≥ 10×10 pixel và có thể xác định rõ là xe` |
| Xe đang đỗ, không di chuyển | `Vẫn giữ track và ID nếu xe còn xuất hiện trong khung hình` |
| Keyframe đặt dày ở đâu | `Đặt dày khi xe đổi hướng, tăng/giảm tốc, bị che khuất, cắt nhau hoặc thay đổi hình dạng bbox rõ rệt; đặt thưa khi chuyển động ổn định` |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `Clip_01 / frame 80 / ID 5`
- Tình huống: `vehicle 5 đột ngột xuất hiện cạnh xe bus vehicle 4`
- Quyết định: `Tạo track mới cho vehicle 5 ngay khi nó xuất hiện đầu xe`
- Lý do: `đó là 1 vật thể cần phải track`

### Ca 2
- Clip / frame / ID: `Clip_01 / frame 92 / ID 8`
- Tình huống: `vehicle 8 đột ngột xuất hiện cạnh xe bus vehicle 4`
- Quyết định: `Tạo track mới cho vehicle 8 ngay khi nó xuất hiện đầu xe`
- Lý do: `đó là 1 vật thể cần phải track`

### Ca 3
- Clip / frame / ID: `Clip_02 / frame 0 / ID 2`
- Tình huống: `Vehicle 2 xuất hiện ở rìa ảnh`
- Quyết định: `gán track cho nó cho đến khi box nhỏ hơn quy định`
- Lý do: `Vẫn là 1 vật thể cần được xác định để track`

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- `Khi xe bị che khuất hoặc xuất hiện rất nhỏ: cần thống nhất rõ thời điểm bắt đầu/kết thúc track. Chỉ tạo bbox khi có đủ bằng chứng xác định đó là xe bốn bánh; khi xe bị che nhưng vẫn còn nhận diện được thì giữ ID, không tự ý tạo ID mới.`
- `Khi hai xe chồng lấn hoặc cắt nhau: cần theo dõi đặc điểm, vị trí trước và sau vùng chồng lấn để đảm bảo ID không bị hoán đổi. Nếu không thể xác định chắc chắn xe nào là xe nào sau vùng che, ưu tiên tạo track mới thay vì gán nhầm ID cũ.`
