# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Nguyễn Hải Minh — 2A202602074`
Ngày: `2026-09-15`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT (Rectangle Track, export MOT 1.1) |
| Thời gian gán `clip_02` (warm-up) | 15 phút |
| Thời gian gán `clip_01` | 45 phút |
| Số track đã vẽ trong `clip_01` | 8 track · 587 bbox · frame 1–190 |
| Số keyframe trung bình mỗi track | `[điền]` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe bị bus nối toa (ID 4) che khi đi song song, frame ~80–116.** Sedan tối màu ID 5 bị đầu bus che gần hết ở frame 85–94; xe trắng ID 6 nằm sau thân bus ở frame 80–113. Tôi giữ nguyên ID cho cả hai vì thời gian bị che dưới ngưỡng 25 frame, và vẽ bbox theo phần nhìn thấy.
2. **Xe vào/ra ở rìa ảnh.** Bus ID 4 vào từ rìa phải (frame 59, bbox rộng 44 px chạm rìa) rồi ra ở rìa trái (frame 136–150); xe tải ID 7 vào từ rìa phải; ID 8 vào từ rìa dưới (frame 139). Tôi cho bbox chạm đúng rìa, không đoán phần nằm ngoài ảnh.
3. **Xe đỗ đứng im suốt clip.** SUV trắng ID 3 đứng yên cả 190 frame (validator cảnh báo "bbox đứng im"). Tôi vẫn gán vì đó là xe bốn bánh thật, không phải box bị treo do quên `outside`.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

> Phần này là **tự kiểm** (không có bạn cặp kiểm chéo). Ba lượt rà được làm lại trên
> bản đã khóa, đối chiếu ảnh gốc `data/clips/clip_01/img1`, `check_mot_labels.py` và
> danh sách lỗi của evaluator. Chưa sửa gì trong annotation (xem mục 3).

- **Lượt 1 — identity/timeline: PASS.** 8 track (ID 1–8), mỗi ID một xe, không dùng lại ID. ID 5 (sedan) giữ một ID khi bị đầu bus ID 4 che ở frame 85–94; ID 6 (xe trắng) giữ một ID khi đi sau thân bus ở frame 101–113. Không có track bị tách đôi (evaluator vs gold: IDSW = 0, 8 track khớp 8 track).
- **Lượt 2 — frame đầu/cuối: tìm thấy 5 lỗi biên.**
  - ID 7 bắt đầu muộn: xe tải đã lộ ở rìa phải từ frame ~106, track bắt đầu 116.
  - ID 6 bắt đầu sớm: frame 80–100 chỉ là mảnh 12–18 px sau đuôi bus.
  - ID 5 bắt đầu sớm: frame 74–78 mảnh khoảng 16 px.
  - ID 4 (bus vào rìa phải, bắt đầu 59) và ID 8 (xe vào rìa dưới, bắt đầu 139) muộn vài frame.
  - Điểm kết thúc đều đúng: ID 5 ra rìa trái ở 139, ID 4 ra rìa trái ở 150, ID 8 ra rìa phải ở 169, không có box treo.
  - ID 3 đứng im 1–190 là xe đỗ thật: validator cảnh báo nhưng not-a-defect.
- **Lượt 3 — frame giữa hai keyframe: tìm thấy 3 đoạn bbox trôi.** ID 6 frame 101–137 (IoU ≈ 0.50–0.55 ở 101, 107, 112, 137), ID 5 frame 81–82, ID 8 frame 163–165. Nguyên nhân: keyframe quá thưa ở đoạn xe đi sau bus và đoạn xe lại gần camera.

Kiểm chéo với: **không có — tự kiểm**. Chi tiết finding ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: **N/A**. Số lỗi tự kiểm tìm được trong bản của mình: **8** (5 lỗi biên + 3 đoạn bbox trôi).

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Không có người thứ hai nên không có ca quyết khác nhau. Ca tự thấy mơ hồ nhất và luật còn thiếu:
- **Ngưỡng bắt đầu track khi xe mới lộ một mảnh sau xe khác** (ID 6 frame 80–100). Đã bổ sung: phần nhìn thấy ≥ 20 × 20 px và nhận ra được thân/bánh xe.
- **Lúc bắt đầu track khi xe đang vào từ rìa ảnh** (ID 7 frame 106–115). Đã bổ sung: bắt đầu ngay khi lộ đầu/đuôi/bánh, và tua lùi từng frame từ keyframe đầu.
- **Mật độ keyframe** (bbox trôi ở ID 6, 5, 8). Đã bổ sung: mỗi 3–5 frame ở đoạn khó.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `73ececb894c96875ce6c0aa939d27523e1bbdaa14b1f46d5f835a25a815c33dc` |
| Thời điểm khóa | `2026-09-15T09:08:36Z` (16:08 giờ Việt Nam) |
| Số row / frame / track trước khi mở reference | 587 / 190 / 8 |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.718 | 0.695 | 0.752 | 0.809 | 0.947 | 0.892 | 0.779 | 38 | 24 | 0 |
| Sau rework | 0.718 | 0.695 | 0.752 | 0.809 | 0.947 | 0.892 | 0.779 | 38 | 24 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Hàng "Sau rework" giữ nguyên số vì `annotations/clip_01/gt.txt` hiện vẫn giống byte-for-byte với bản pre-gold: bản này đã qua cổng nên chưa rework. Các lỗi evaluator chỉ ra, và hướng sửa trong CVAT nếu rework:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox có trước khi track tham chiếu xuất hiện (xe chỉ lộ một mảnh 12–18 px sau bus) | 80–100 | 6 | Chưa sửa. Hướng sửa: xem lại frame 80–100, đặt `outside` trước frame xe đủ nhìn thấy |
| Bbox có trước khi track tham chiếu xuất hiện | 74–78 | 5 | Chưa sửa. Hướng sửa: như trên |
| Bắt đầu track muộn ở rìa phải (thiếu khoảng 10 frame: gold 85 frame, bản của tôi 75) | ~106–115 | 7 | Chưa sửa. Hướng sửa: lùi keyframe đầu về frame xe tải lộ ra ở rìa phải |
| Bbox trôi giữa hai keyframe (IoU ≈ 0.50–0.55) | 101, 107, 112, 137 | 6 | Chưa sửa. Hướng sửa: thêm keyframe dày hơn ở đoạn 100–140 |
| Bbox trôi giữa hai keyframe | 81–82 | 5 | Chưa sửa. Hướng sửa: thêm keyframe |
| Bbox trôi giữa hai keyframe | 163, 165 | 8 | Chưa sửa. Hướng sửa: thêm keyframe khi xe ra ở rìa phải |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json` (lần chạy trên Colab, GPU T4):

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13 |
| weights / hai tracker | `yolo26n.pt` / `bytetrack.yaml` (control) và `configs/trackers/botsort-reid.yaml` (treatment) |
| conf / IoU / imgsz / classes | 0.25 / 0.70 / 960 / `[2, 5, 7]` (car, bus, truck) |
| device | `0` (CUDA), `persist=True`, 190 frame |

Output model: ByteTrack ra 607 bbox · 16 track; BoT-SORT + ReID ra 638 bbox · 16 track. Gold có 573 bbox · 8 track.

Bảng tổng hợp mục 4 của notebook:

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.718 | 0.695 | 0.752 | 0.809 | 0.947 | 0.892 | 0.779 | 38 | 24 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.656 | 0.595 | 0.732 | 0.807 | 0.860 | 0.710 | 0.779 | 110 | 59 | 1 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của tôi (0.892) **thấp hơn** IDF1 (0.947), nên đây không phải trường hợp "MOTA cao, IDF1 thấp". Nhãn của tôi không sai ở ID: IDSW = 0, mỗi xe gold ứng với đúng một ID của tôi, và 8 track khớp 8 track. Lỗi còn lại nằm ở coverage và hình học:

- FP = 38. Khoảng 26 bbox đến từ việc bắt đầu track quá sớm (ID 6 frame 80–100, ID 5 frame 74–78). Phần còn lại là các frame bbox bị trôi, IoU dưới 0.5.
- FN = 24. Chủ yếu do bắt đầu track muộn (ID 7 thiếu khoảng 10 frame ở rìa phải, ID 4 và ID 8 thiếu vài frame lúc vào khung), cộng thêm các frame bbox trôi (một bbox trôi tính cả 1 FP lẫn 1 FN).
- Kiểm tra lại: MOTA = 1 − (38 + 24 + 0) / 573 = 0.892. IDTP ≈ 0.947 × (573 + 587) / 2 ≈ 549 = số TP, tức mọi detection đúng đều mang đúng ID.

HOTA chỉ đạt 0.718 (DetA 0.695) dù IDF1 tới 0.947. Lý do là HOTA lấy trung bình trên nhiều ngưỡng IoU từ 0.05 đến 0.95, nên bbox chưa khít (LocA 0.809) kéo DetA xuống. IDF1 và MOTA chỉ dùng một ngưỡng IoU 0.5.

Vì sao MOTA không phạt nặng lỗi ID: MOTA đếm **sự kiện** đổi ID. Mỗi lần switch chỉ cộng 1 vào tử số, trong khi FP và FN được cộng theo từng bbox. Sau khi switch, CLEAR-MOT ghép theo từng frame và chấp nhận ID mới, nên mọi frame sau đó vẫn tính là TP. Một xe bị đổi ID ở giữa quãng đời 60 frame chỉ mất khoảng 1/573 MOTA. IDF1 thì tìm một ánh xạ ID một-đối-một cho **toàn clip**, nên toàn bộ đoạn mang ID sai đều thành IDFP/IDFN. Vì vậy, nếu MOTA cao mà IDF1 thấp thì lỗi nằm ở identity (tách/đổi ID), không phải ở bỏ sót.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

| | IDF1 | AssA | IDSW | Track gold bị tách |
| --- | ---: | ---: | ---: | --- |
| ByteTrack control | 0.875 | 0.776 | 2 (gold 4 @59: 14→15; gold 5 @94: 23→32) | gold 4, 5, 7 |
| BoT-SORT + ReID | 0.900 | 0.820 | 2 (gold 5 @87: 17→18; gold 6 @113: 24→31) | gold 5, 6, 7 |

Treatment cao hơn ở IDF1 (+0.025) và AssA (+0.044), nhưng **IDSW không đổi (2 = 2)** và số track bị tách vẫn là 3. Ở đoạn occlusion khó nhất, treatment không tốt hơn:

- **Frame 82 → 87 → 94 (gold/ID của tôi 5, sedan tối màu đi sau đầu bus ID 4).** Frame 82: sedan còn thấy rõ ở x≈742–770. Frame 87: đầu bus (x 717–902) che gần hết sedan (bbox nhìn thấy x 685–725). Treatment đổi ID **ngay tại frame 87** (17→18), control giữ lâu hơn nhưng cũng đổi ở **frame 94** (23→32). Cả hai đều thua ở ca này. Crop lúc bị che chủ yếu là pixel của bus, nên appearance không đủ giống để qua `appearance_thresh: 0.80`, còn motion/IoU thì mất vì xe gần như biến mất.
- **Frame 101 → 113 (gold 6, xe trắng sau thân bus).** Treatment vẫn giữ box nhưng đổi ID ở frame 113 (24→31). Control không switch nhưng làm rơi box: chỉ phủ 42/56 frame. Cùng một ca khó, control trả bằng FN, treatment trả bằng IDSW.
- **Treatment tốt hơn ở chỗ khác:** bus gold 4 vào từ rìa phải ở frame ~56–59. Control tách 14→15 tại frame 59, còn treatment giữ một ID. Xe gold 8 vào từ rìa dưới (frame ~137–169): control chỉ phủ 20/33 frame, treatment phủ đủ; gold 5 cũng từ 77% lên đủ. AssA/IDF1 tăng chủ yếu nhờ track liền mạch hơn lúc vào khung và phủ nhiều frame hơn, **không phải** nhờ cứu được ID qua occlusion.

**Không cô lập causal effect của ReID.** Hai run dùng cùng detector input (cùng weights, conf, imgsz, classes), nhưng ByteTrack và BoT-SORT là hai implementation khác nhau: khác Kalman state, khác pipeline ghép và fuse score, khác cách quản lý track mới/track mất, và BoT-SORT có thêm gate appearance + proximity. Chênh lệch trong bảng là của **cả hệ thống**. Muốn quy cho ReID thì cần run thứ ba: BoT-SORT với `with_reid: false`, giữ nguyên các tham số khác.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

| | DetA | LocA | FP | FN | MOTA |
| --- | ---: | ---: | ---: | ---: | ---: |
| ByteTrack control | 0.649 | 0.846 | 88 | 54 | 0.749 |
| BoT-SORT + ReID | 0.711 | 0.872 | 91 | 26 | 0.792 |

DetA tăng 0.062, FN giảm gần một nửa (54 → 26), FP gần như không đổi (88 → 91). Detector giống hệt nhau, nên FN giảm không phải do detector tốt lên. Nó đến từ tracker: detection nào được khởi tạo/giữ thành track được xuất ra, và bbox xuất ra là trạng thái Kalman sau update (LocA cũng tăng 0.846 → 0.872).

Lỗi còn lại của treatment **chủ yếu là detector (FP)**, association đứng thứ hai:

- **Detector / FP:** có 5 ID không khớp xe nào trong gold: ID 7 (frame 16–116, 43 frame), ID 27 (106–121, 16 frame), ID 38 (158–178, 16 frame), ID 26 và ID 28 (mỗi ID 1 frame), tổng ≈ 77/91 FP. Trong ảnh worst-frame của notebook (frame 107), box đỏ **T7 nằm trên quầy hàng/ki-ốt** ở giữa ảnh, là vật thể tĩnh bị detector nhận nhầm thành xe. Cùng frame đó còn một box đỏ ở rìa trái trên rào chắn đỏ. Tracker không sửa được detection sai; association tốt hơn chỉ giúp FP này giữ ID ổn định hơn.
- **Association:** 2 IDSW và 3 track bị tách (gold 5, 6, 7), tập trung ở đoạn bus che xe (frame 85–116). Số bbox không nhiều nhưng đây là thứ kéo AssA/IDF1 xuống.
- FN 26 còn lại phần lớn là lúc xe bị che nặng sau bus (gold 6 mới phủ 44/56 frame), tức detector không thấy xe, không phải lỗi ghép.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

**Frame 85–94, ID 5 của tôi (= gold 5), sedan tối màu đi sau đầu bus ID 4.** Tôi giữ ID 5 liên tục từ frame 74 đến 139. Treatment đổi ID tại **frame 87 (17 → 18)**. Evaluator "ReID vs bạn" báo đúng chỗ này ("track bản A 5 đang là ID 17 → nhảy sang ID 18"), và "ReID vs gold" cũng xác nhận gold 5 bị treatment tách thành [17, 18].

Vì sao tôi đúng: xe bị che dưới 25 frame, cùng làn, cùng hướng, và hiện lại đúng vị trí dự đoán, nên theo luật occlusion ngắn phải giữ ID. ReID sai vì crop ở frame 87 chủ yếu là đầu bus, embedding không giống track cũ nên appearance gate từ chối và sinh ID mới.

Ví dụ phụ: ID 7 của treatment (frame 16–116) là quầy hàng tĩnh. Tôi không gán vì nó không phải xe bốn bánh (đúng phạm vi mục 1 của guideline).

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

**Frame 108–115, ID 7 của tôi (xe tải thùng trắng vào từ rìa phải).** Treatment có box ID 29 ở frame 108–115, còn track 7 của tôi bắt đầu từ frame 116 ("ID 29: đã có bbox trước khi track tham chiếu 7 xuất hiện (frame 108–115)").

Tôi không sửa chỉ vì model nói khác mà đối chiếu evidence độc lập:
1. Mở ảnh gốc: frame 106 và 110 đã thấy rõ đầu xe tải ở rìa phải (x ≈ 928–960).
2. Gold 7 dài 85 frame, track 7 của tôi chỉ 75 frame (116–190), nên tôi thiếu khoảng 10 frame đầu, đúng với phần FN của tôi.
3. Luật bbox đã ghi: "bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh", "bbox chạm đúng rìa". Tôi đã chờ xe vào gần hết khung mới vẽ.

Kết luận: model đúng ở đây, tôi cần lùi keyframe đầu của ID 7 về khoảng frame 106 và export lại từ CVAT. Cùng kiểu lỗi nhỏ hơn: ReID ID 34 có box ở frame 136–138 trước ID 8 của tôi (bắt đầu 139, xe vào từ rìa dưới), và ReID ID 9 ở frame 55–58 trước bus ID 4 của tôi (bắt đầu 59).

Ngược lại, ở **ID 6 frame 80–100** cả gold lẫn treatment đều không có box, còn tôi vẽ một mảnh rộng 12–18 px của xe trắng lộ sau đuôi bus. Đây là ca cần thống nhất ngưỡng "đủ nhìn thấy", không phải chỗ để bắt chước model.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

Sửa trong `GUIDELINE_MINI.md`:

1. **Luật entry ở rìa ảnh (mục 3):** "Xe vào từ rìa: bắt đầu track ở frame đầu tiên nhìn thấy được một phần đặc trưng của xe bốn bánh (đầu xe, bánh, thùng), kể cả khi bbox chạm rìa. Sau khi vẽ keyframe đầu, **tua lùi từng frame** đến khi xe biến hẳn." Lỗi đã mắc: ID 7 muộn khoảng 10 frame, ID 4 và ID 8 muộn vài frame.
2. **Ngưỡng bắt đầu track khi xe mới lộ sau xe khác (mục 3):** "Chỉ bắt đầu track khi phần nhìn thấy đủ nhận ra là xe (gợi ý: rộng ≥ 20 px và thấy được thân/bánh), không vẽ mảnh vài pixel sau bus." Lỗi đã mắc: ID 6 frame 80–100, ID 5 frame 74–78, khoảng 26 FP.
3. **Keyframe dày ở đâu (mục 3):** "Đặt keyframe mỗi 3–5 frame khi xe bị che một phần, đổi kích thước nhanh (vào/ra khung, lại gần camera), hoặc đi cạnh xe lớn; mở từng frame giữa hai keyframe để kiểm drift." Lỗi đã mắc: bbox trôi ở ID 6 frame 101–137, ID 5 frame 81–82, ID 8 frame 163–165.
4. **Xe đỗ (mục 3):** "Xe đỗ đứng im vẫn gán suốt thời gian xuất hiện; cảnh báo 'bbox đứng im' của validator không có nghĩa là lỗi" (ví dụ SUV ID 3, frame 1–190).
5. **Không gán (mục 1):** thêm "quầy hàng/ki-ốt, rào chắn, thùng rác, biển báo". Đây là những vật detector nhận nhầm (T7 trên ki-ốt), để người gán sau không bị model kéo theo.
6. **Mục 4 — ca mơ hồ có thật:** ghi ba ca ở trên với frame/ID cụ thể (sedan ID 5 bị bus che frame 85–94 → giữ ID; xe trắng ID 6 lộ một mảnh frame 80–100 → chưa bắt đầu track; xe tải ID 7 vào từ rìa phải frame 106 → bắt đầu track).

Đổi trong quy trình:

- Lượt tự kiểm 2 (endpoint) làm riêng từng track: nhảy tới frame đầu/cuối rồi tua ±10 frame, thay vì chỉ nhìn timeline.
- Chạy `tools/visualize_tracks.py` ở tốc độ chậm cho đoạn có xe lớn che xe nhỏ trước khi khóa pre-gold.
- Dùng worst-frame "ReID vs bạn" như danh sách nghi vấn: với mỗi chỗ lệch, mở ảnh gốc + guideline để quyết, chỉ sửa khi có evidence độc lập.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md` (tự kiểm, không có bạn cặp)
- [x] `reports/REPORT.md` (file này)
