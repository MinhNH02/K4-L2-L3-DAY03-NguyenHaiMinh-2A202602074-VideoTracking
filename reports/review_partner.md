# Peer review — Day 3

Reviewer chỉ ghi finding; tác giả tự sửa bài của mình và điền closure.

| Trường | Giá trị |
| --- | --- |
| Author | `Nguyễn Hải Minh — 2A202602074` |
| Reviewer | Tự kiểm — tác giả tự review (không có bạn cặp) |
| Pair ID | N/A |
| CVAT version | `[điền]` |
| Thời điểm review | 2026-09-15, sau khi khóa pre-gold |

## Danh sách finding

Mỗi finding bắt buộc có `frame + ID + lỗi + sửa thế nào`. Nếu chưa thống nhất,
dùng `needs-review`; không ép tác giả sửa theo cảm tính.

> Ghi chú: các finding dưới đây đã được đối chiếu với ảnh gốc `data/clips/clip_01/img1`
> và output evaluator. CVAT frame = MOT frame − 1.

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure: fixed / not-a-defect / needs-review |
| ---: | ---: | ---: | ---: | --- | --- | --- | --- |
| 1 | 105–114 | 106–115 | 7 | Entry muộn | Xe tải thùng trắng đã lộ đầu ở rìa phải từ frame 106 (x ≈ 928–960), nhưng track 7 bắt đầu ở 116. Rule: "bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh", "bbox chạm đúng rìa" | Lùi keyframe đầu của ID 7 về frame xe lộ ra, bbox chạm rìa phải, export lại | needs-review |
| 2 | 79–99 | 80–100 | 6 | Bắt đầu track quá sớm / box mảnh | Xe trắng chỉ lộ một mảnh rộng 12–18 px sau đuôi bus ID 4. Rule: bbox ôm phần nhìn thấy, nhưng chưa có ngưỡng "đủ nhận ra là xe" | Thống nhất ngưỡng hiển thị tối thiểu; nếu dưới ngưỡng thì đặt `outside` tới frame xe đủ nhìn thấy | needs-review |
| 3 | 73–77 | 74–78 | 5 | Bắt đầu track quá sớm | Sedan mới lộ rộng khoảng 16 px sau bus. Cùng rule với finding 2 | Như finding 2 | needs-review |
| 4 | 100–136 | 101–137 | 6 | Interpolation drift | Bbox lệch khỏi xe giữa hai keyframe (IoU với reference ≈ 0.50–0.55 ở frame 101, 107, 112, 137). Rule: frame giữa hai keyframe không bị drift | Thêm keyframe mỗi 3–5 frame ở đoạn xe đi sau bus rồi rẽ | needs-review |
| 5 | 162–164 | 163–165 | 8 | Interpolation drift | Bbox trôi khi xe lại gần camera và ra ở rìa phải | Thêm keyframe ở frame 163–165 | needs-review |
| 6 | 58–... | 59–150 | 4 | Entry muộn nhẹ | Bus lộ ở rìa phải trước frame 59 (track bắt đầu khi bbox đã rộng 44 px) | Tua lùi từng frame từ 59 và lùi keyframe đầu | needs-review |

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | 8 track (ID 1–8), đều là xe con/bus/xe tải; không gán ki-ốt hay người |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | Không có ID nào bị dùng lại; evaluator vs gold IDSW = 0 |
| Occlusion ngắn giữ ID; crossing không đổi ID | PASS | ID 5 giữ ID khi bị đầu bus che ở frame 85–94 |
| Entry/exit đúng; không box treo sau khi xe rời khung | FINDING | #1 (ID 7 vào muộn), #2–#3 (ID 6, ID 5 bắt đầu sớm), #6 (ID 4) |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | FINDING | #2: mảnh 12–18 px ở ID 6 frame 80–100 cần ngưỡng rõ ràng |
| Frame giữa hai keyframe không bị interpolation drift | FINDING | #4 (ID 6), #5 (ID 8) |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | `check_mot_labels.py`: clip_01 0 lỗi (1 cảnh báo đứng im: ID 3 xe đỗ, not-a-defect); clip_02 0 lỗi |
| Mọi finding có cách sửa và closure do tác giả điền | PASS | Cả 6 finding có cách sửa; closure `needs-review` chờ tác giả rework trong CVAT |

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | PASS | 8 track, không reuse ID; ID 5 giữ ID qua occlusion frame 85–94; ID 6 giữ ID frame 101–113; IDSW vs gold = 0 |
| 2 — endpoint/scope | NEEDS-REVIEW | Finding #1 (ID 7 vào muộn ~106→116), #2 (ID 6 sớm 80–100), #3 (ID 5 sớm 74–78), #6 (ID 4 vào muộn); ID 3 đứng im = xe đỗ, not-a-defect |
| 3 — geometry/interpolation | NEEDS-REVIEW | Finding #4 (ID 6 trôi 101–137), #5 (ID 8 trôi 163–165), thêm ID 5 frame 81–82 |

## Exit ticket

1. Finding quan trọng nhất và rule dùng để kết luận: #1 — ID 7 bắt đầu muộn khoảng 10 frame ở rìa phải (frame 106–115). Rule: bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh, bbox chạm rìa, không chờ xe vào hết khung.
2. Một finding tác giả đóng là `not-a-defect`, kèm lý do (nếu có): cảnh báo validator "track 3 đứng im frame 1–15" là SUV trắng đỗ thật suốt clip, không phải box treo do quên `outside`.
3. Một rule cần Lab Coach làm rõ (nếu có): ngưỡng hiển thị tối thiểu để bắt đầu track khi xe chỉ lộ một mảnh sau xe khác (ID 6 frame 80–100).
