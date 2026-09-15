# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Nguyễn Hải Minh — 2A202602074`
Clip: `clip_01`, `clip_02`

> Quy ước frame: số frame trong file này là **MOT frame** (bắt đầu từ 1).
> CVAT frame = MOT frame − 1.

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm:

- **Xe buýt nối toa (bus hai khoang) = một bbox, một ID**, ôm cả hai khoang và khớp nối (`clip_01` ID 4).
- **Xe đang đỗ vẫn gán** suốt thời gian nó ở trong khung (`clip_01` ID 3, SUV trắng đỗ frame 1–190).
- **Không gán vật tĩnh trông giống xe:** quầy hàng/ki-ốt, rào chắn đỏ trắng, thùng rác, biển báo, cột đèn. Detector YOLO từng nhận nhầm ki-ốt giữa ảnh `clip_01` thành xe (frame 16–116). Đó là lỗi của model, không phải chỗ bạn bỏ sót.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (= 2 giây @ 12.5 fps). Trong lúc bị che vẫn giữ bbox phần nhìn thấy; nếu bị che **hoàn toàn** thì đặt `outside` rồi mở lại **cùng track** khi xe hiện lại (không vẽ track mới) | Xe vẫn là một vật thể, đi cùng làn, cùng hướng, hiện lại đúng chỗ dự đoán. Đổi ID ở đây là lỗi identity, IDF1/AssA phạt nặng dù MOTA gần như không đổi |
| Xe bị che lâu hơn ngưỡng trên | mở **track mới** | Sau hơn 2 giây không đủ bằng chứng để chắc là cùng xe; theo luật mặc định của lab để mọi người gán giống nhau |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Đã ra khỏi khung là kết thúc track; không có evidence nối qua ngoài khung |
| Hai xe cắt nhau / chồng lên nhau | **mỗi xe giữ ID của mình**. Xe phía trước giữ bbox đầy đủ; xe phía sau chỉ ôm phần nhìn thấy. Tua **từng frame** qua đoạn chồng lấn và kiểm hướng đi/làn đường trước khi đi tiếp | Lúc chồng lấn là chỗ dễ vô tình kéo bbox của xe A sang xe B nhất (cũng là chỗ tracker đổi ID: BoT-SORT + ReID đổi ID sedan `clip_01` ở frame 87 khi bị đầu bus che) |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh. **Xe đang vào khung: bắt đầu track ngay frame đầu tiên thấy được đầu/đuôi/bánh xe, kể cả khi mới lộ một phần ở rìa.** Sau khi vẽ keyframe đầu, **tua lùi từng frame** cho đến khi xe biến hẳn |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được**; không kéo bbox sang phần bị che |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: **phần nhìn thấy rộng ≥ 20 px và cao ≥ 20 px, thấy được thân xe hoặc bánh xe**. Mảnh vài pixel lộ sau xe khác (chưa nhận ra là xe) thì **chưa** bắt đầu track, trừ khi xe đang vào từ rìa ảnh (dùng luật rìa ảnh ở trên) |
| Xe đang đỗ, không di chuyển | **vẫn gán**, một ID suốt thời gian trong khung. Đặt keyframe ở frame đầu, frame cuối và mỗi khi bị vật khác che. Cảnh báo "bbox đứng im" của `check_mot_labels.py` là bình thường với xe đỗ |
| Keyframe đặt dày ở đâu | **mỗi 3–5 frame** khi: xe vào/ra khung, xe lại gần camera (bbox to nhanh), xe rẽ hoặc đổi làn, xe đi cạnh hoặc sau xe lớn (bus/xe tải). Đi thẳng đều ở xa: mỗi 10–15 frame là đủ. Sau khi xong mỗi track, **mở frame giữa hai keyframe xa nhất** để kiểm drift |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01` / frame 82–94 / ID 5 (sedan tối màu), bị bus nối toa ID 4 che
- Tình huống: frame 82 sedan còn thấy rõ; từ frame 85 đầu bus đi lướt qua che gần hết, frame 87 chỉ còn một phần đuôi xe (bbox nhìn thấy x ≈ 685–725 so với bus x ≈ 717–902); sau đó sedan hiện lại cùng làn, cùng hướng.
- Quyết định: **giữ nguyên ID 5** suốt frame 74–139; bbox ôm phần nhìn thấy trong lúc bị che.
- Lý do: bị che dưới 25 frame và xe hiện lại đúng vị trí dự đoán, nên theo luật occlusion ngắn là cùng một xe. Gold xác nhận (IDSW của tôi = 0). BoT-SORT + ReID lại đổi ID tại frame 87 (17→18) vì crop lúc đó chủ yếu là pixel của bus; ByteTrack đổi ID tại frame 94. **Không đổi nhãn theo model.**

### Ca 2
- Clip / frame / ID: `clip_01` / frame 80–100 / ID 6 (xe con trắng đi sau thân bus ID 4)
- Tình huống: xe trắng chỉ lộ một mảnh rộng 12–18 px sau đuôi bus; từ frame ~101 mới đủ thấy thân xe (bbox ≈ 68×40 px).
- Quyết định lúc gán: bắt đầu track từ frame 80. **Sau khi chấm gold: sửa luật**, chỉ bắt đầu track khi phần nhìn thấy đạt ngưỡng ≥ 20 px và nhận ra được thân xe (ở ca này khoảng frame 101). Ca tương tự: ID 5 frame 74–78 (mảnh rộng khoảng 16 px).
- Lý do: mảnh vài pixel không đủ để khẳng định là xe bốn bánh; gold và cả hai tracker đều không có box ở đoạn này. Bắt đầu sớm sinh ra khoảng 26 FP (ID 6 frame 80–100 và ID 5 frame 74–78), phần lớn trong 38 FP của bản pre-gold.

### Ca 3
- Clip / frame / ID: `clip_01` / frame 106–115 / ID 7 (xe tải thùng trắng vào từ rìa phải)
- Tình huống: frame 106 và 110 đã thấy đầu xe tải ở rìa phải (x ≈ 928–960), nhưng lúc gán tôi chờ xe vào gần hết khung và bắt đầu track từ frame 116.
- Quyết định: **track phải bắt đầu từ frame xe lộ ra ở rìa (~106)**, bbox chạm rìa phải. Cách làm: sau khi vẽ keyframe đầu, tua lùi từng frame đến khi xe biến hẳn.
- Lý do: luật "bắt đầu từ frame đầu tiên xác định được là xe bốn bánh" và "bbox chạm đúng rìa" áp dụng cả khi xe đang vào khung. Evidence độc lập: ảnh gốc frame 106/110; gold ID 7 dài 85 frame so với 75 frame của tôi. Model chỉ **gợi ý** chỗ nghi vấn (ReID ID 29 có box frame 108–115). Cùng kiểu lỗi nhỏ hơn: ID 4 (bus vào rìa phải, bắt đầu 59) và ID 8 (xe vào rìa dưới, bắt đầu 139) đều muộn vài frame.

### Ca 4
- Clip / frame / ID: `clip_01` / frame 1–190 / ID 3 (SUV trắng đỗ bên kia đường)
- Tình huống: xe đứng im cả clip; validator cảnh báo "bbox gần như đứng im" và gợi ý có thể quên `outside`.
- Quyết định: **giữ track suốt 190 frame**, không đặt `outside`.
- Lý do: đó là xe bốn bánh thật đang đỗ (GUIDE: xe đỗ vẫn là `vehicle`). Cảnh báo này chỉ nghĩa là "hãy nhìn lại", không phải lỗi; gold cũng có xe này.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Kết quả bản pre-gold `clip_01` so với gold: IDF1 0.947 · MOTA 0.892 · MOTP 0.779 · FP 38 · FN 24 · IDSW 0. Identity đúng hết; lỗi nằm ở **điểm bắt đầu track** và **bbox trôi giữa keyframe**. Luật còn thiếu hoặc còn mơ hồ, đã viết lại ở mục 2–3:

- **Thiếu ngưỡng bắt đầu track khi xe mới lộ sau xe khác.** Luật cũ chỉ ghi "frame đầu tiên xác định được là xe" nên tôi vẽ cả mảnh 12–18 px (ID 6 frame 80–100). Luật mới: phần nhìn thấy ≥ 20 × 20 px và nhận ra được thân/bánh xe.
- **Luật rìa ảnh chỉ nói về bbox, không nói về lúc bắt đầu.** Tôi vào track muộn khoảng 10 frame ở ID 7 (frame 106–115). Luật mới: xe đang vào từ rìa thì bắt đầu ngay khi lộ đầu/đuôi/bánh, và bắt buộc **tua lùi từng frame** từ keyframe đầu.
- **"Keyframe đặt dày ở đâu" để trống nên tôi đặt keyframe quá thưa.** Bbox trôi (IoU với gold ≈ 0.50–0.55) ở ID 6 frame 101–137, ID 5 frame 81–82, ID 8 frame 163–165. Luật mới: mỗi 3–5 frame khi xe vào/ra khung, lại gần camera, rẽ, hoặc đi cạnh xe lớn; kiểm frame giữa hai keyframe xa nhất.
- **Chưa nói gì về vật tĩnh giống xe và xe đỗ.** Đã bổ sung ở mục 1 và mục 3, để người gán sau không bị model (box trên ki-ốt) hay cảnh báo validator (xe đỗ ID 3) kéo sai.
- **Quy trình khi dùng model để soi lỗi:** danh sách "ReID vs bạn" chỉ là danh sách nghi vấn. Mỗi chỗ lệch phải mở ảnh gốc và đối chiếu với luật trong file này; chỉ sửa trong CVAT khi có evidence độc lập, rồi export lại (không sửa tay file MOT).
