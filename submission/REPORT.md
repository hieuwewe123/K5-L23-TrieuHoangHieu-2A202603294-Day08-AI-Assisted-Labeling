# Báo cáo Day 08 — 2A202603294 - Triệu Hoàng Hiếu

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26n | mặc định 0.30 | `r02`–`r03`; threshold sweep cho thấy các lớp này không đạt recall 90% ở ngưỡng ≤0.5, hoặc dữ liệu gold quá ít. |
| three_wheeler | YOLOE-26s, prompt `three_wheeler` | 0.15 | `r04`; threshold sweep đạt recall 100% trên 7 vật gold, precision 78%. |

- Lệnh gộp được tái dựng từ source metadata đã lưu (lệnh terminal gốc không được lưu): `python scripts/merge_prelabels.py work\prelabel\yoloe_all\pre.coco.json work\prelabel\y26_all\pre.coco.json --conf 0.3 --thr "three_wheeler=0.15" --split "batch,gold" --out work\prelabel\merged_review\pre.coco.json`.
- **Phiên bản SAM đã chọn:** SAM 2.1 tiny (`sam2.1_t.pt`), chạy CPU; học viên đã xác nhận các mask minh họa nhìn đúng.
- Lỗi không sửa được chỉ bằng ngưỡng: nhầm lớp giữa `three_wheeler` và `truck`, cùng các vật bị bỏ sót; kiểm tra cấu trúc xe và quét toàn ảnh trước khi đối chiếu box.
- Box điểm cao mà sai / điểm thấp mà đúng: quan sát cụ thể từ lượt chạy pilot r02–r04:
  - *Điểm cao mà sai:* YOLOE trên `img_002` nhận diện mảng tường/đuôi xe mép phải thành `three_wheeler` với score 0.37–0.40; trên `img_011` một mảng góc tường nhà đạt score 0.28.
  - *Điểm thấp mà đúng:* YOLOE trên `img_032` xe ba gác ở xa chỉ có score 0.27 nhưng đúng là xe thật (nhờ ngưỡng 0.15 nên giữ được); YOLO26 trên `img_002` người đi bộ ở xa đạt score 0.35 là người thật (nếu đặt ngưỡng 0.50 như `r03` thì bị loại bỏ hoàn toàn).
- Giới hạn: threshold sweep dùng 4 ảnh gold / 49 đối tượng và chọn ngưỡng trên chính mẫu đó; precision/recall trên gold là lạc quan, đặc biệt các lớp có ít mẫu.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 220 | 220/253 = 87.0% | 253 box máy |
| edited | 8 | 8/253 = 3.2% | 253 box máy |
| deleted | 25 | 25/253 = 9.9% | 253 box máy |
| added | 12 | 12/240 = 5.0% | 240 đối tượng sau duyệt |

- `review_stats.md` loại 19 đối tượng nhân tạo quanh bẫy khỏi error profile để không tính lỗi do bẫy là lỗi tự nhiên của máy; kết quả bẫy được phân tích riêng ở mục 3.
- Lớp máy yếu nhất: `three_wheeler` — 19 box bị xóa và 5 bị sửa trong bảng theo lớp của `review_stats.md`; YOLOE cần kiểm tra cấu trúc thật, ví dụ ứng viên ở `img_034`.
- So với gold: precision / recall của **máy** là 80.6% / 59.2% (TP=29, FP=7, FN=20; 49 đối tượng gold trên 4 ảnh).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 5/12 = 41.7%. Bẫy lọt: loose ở `img_029`, `img_024`; missing ở `img_035`, `img_009`, `img_010`; extra ở `img_006`; wrong_class ở `img_013`. Các bẫy missing lọt cho thấy lượt đầu chưa quét toàn ảnh khi ẩn nhãn máy.
- Accepted 220/253 = 87.0% nhưng bắt bẫy chỉ 5/12 = 41.7% cho thấy duyệt từng box không thay thế được lượt tìm vật bị bỏ sót độc lập; dễ bị thiên kiến tự động hóa.

## 4. Máy có đáng dùng không?

- Thời gian (ước tính học viên, `time_log.csv`): gán tay 90/4 = 22.5 phút/ảnh; duyệt có pre-label 210/24 = 8.75 phút/ảnh. Đây là ước tính nhớ lại, không phải đo bằng đồng hồ; gold được gán trước nên người duyệt đã quen cảnh.
- Kết luận: dùng AI làm nhãn nháp, không tự động phát hành. Căn cứ: accepted 220/253 = 87.0%, gold precision/recall 80.6%/59.2%, bẫy 5/12 = 41.7%, dù vòng QC sau sửa không thấy lỗi ở 0/5 ảnh.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Ẩn nhãn máy và quét toàn ảnh tìm vật bị sót trước khi xem box.
  2. Với từng box, kiểm tra lớp, vị trí/kích thước và đối chiếu vật thực trong ảnh; kiểm chứng `three_wheeler` bằng cấu trúc nhìn thấy.
  3. Kiểm tra mẫu QC ngẫu nhiên sau sửa và ghi từng lỗi vào phiếu, không kết luận từ accepted rate riêng lẻ.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013` | SAM 2.1 tiny | 3 (2 dương, 1 âm) | Mask bao trùm thùng xe/xung quanh người lái; không tự tách riêng người lái và hàng hoá | Mask xe loang sang người lái; phải dùng 1 click âm ở vị trí người lái nhưng viền áo vẫn bị kéo dính vào thùng xe | `tuktuk_img013.jpg` |
| `img_003` | SAM 2.1 tiny | 1 dương | Mask bao cả thùng, khung xe và lượng hàng nhỏ trong đó; không tách riêng người lái/hàng hoá | 1 click dương bắt được khung xe ba gác nhưng phần bóng đổ dưới gầm xe bị kéo vào mask (vi phạm guideline 4.2.3: bóng đổ không thuộc vật) | `three_wheeler_img003.jpg` |
| `img_017` | SAM 2.1 tiny | 1 dương | Mask bao thân xe ba bánh; không tách riêng người lái | Bám viền thân xe tốt nhưng mui xe và tay lái bị đứt đoạn do tương phản màu yếu với nền đường | `three_wheeler_img017.jpg` |

- **Đối chiếu guideline mục 4.2:** Polygon/mask phải bám sát phần nhìn thấy, bỏ bóng đổ và tách riêng người lái/hàng hoá. Khi chỉ dùng 1 click dương, SAM có xu hướng gộp toàn bộ vật thể liên kết vật lý (cả người lái ngồi trên xe và hàng trên thùng) thành 1 mask khối lớn. Cần phối hợp click âm hoặc tách mask thủ công trên CVAT.
- **Quy tắc / lỗi rút ra:** "Một click, ba mask" của SAM: máy chỉ đề xuất các mức bao phủ hình học, người gán nhãn phải quyết định mask có đúng ranh giới ngữ nghĩa hay không; mask chính xác về biên độ không cứu được việc gán sai nhãn lớp.
- **So sánh Interactor SAM trong CVAT vs Script offline:** Script offline (`sam2_points.py`) chạy ổn định trên CPU với ảnh đơn nhưng khó chỉnh sửa lặp lại; Interactor SAM trong CVAT trực quan hơn vì cho phép click sửa tức thì và chuyển ngay thành polygon layer để chỉnh sửa đỉnh.

## 6. Calibration (Part 6)

- Số quyết định: 4 (`calibration_decisions.csv`); quan trọng nhất là CAL-04 về quét toàn ảnh trước khi xem nhãn máy.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: làm rõ phân lớp xe điện; quy tắc cụm xe gần camera/ignore_region; xác minh cấu trúc ba bánh; thêm lượt quét toàn ảnh khi ẩn nhãn máy.
- Nguyên nhân gốc: chủ yếu là thiếu bước quét toàn ảnh và quá tin vào pre-label; CAL-01 cũng chốt một điểm diễn giải taxonomy nhưng không có ví dụ xe điện trong bộ ảnh.

## 7. QC và quyết định (Part 6)

- Vòng 1: 5 ảnh ngẫu nhiên (seed 42), 56 đối tượng; 7/56 lỗi = 12.5%, trên 4/5 ảnh = 80%. Ghi nhận 6 box thừa và 1 sai lớp (`img_003`, `three_wheeler` đúng là `truck`).
- Đã sửa nhãn trong CVAT và export `qc_fixed.json`; release sau sửa gồm 24 ảnh, 238 đối tượng.
- Vòng 2 sau sửa: 5 ảnh ngẫu nhiên (seed 42), 50 đối tượng; học viên xác nhận đã kiểm tra đủ 5 ảnh và không còn lỗi: 0/50 đối tượng, 0/5 ảnh.
- **Giới hạn mẫu nhỏ**: vòng 2 có 0/5 ảnh lỗi, nhưng cận trên Wilson 95% là 43%; mẫu nhỏ không chứng minh tỷ lệ lỗi thật dưới 2%.
- Script chỉ đếm lỗi đã nhập vào phiếu; không tự xác định box sai, vật bị cả người và máy cùng bỏ sót, hay việc tuân thủ checklist.

**Quyết định: Rework**, ngày 2026-11-17. Vòng 2 không phát hiện lỗi, nhưng bắt bẫy vẫn chỉ 5/12 = 41.7%, dưới ngưỡng 80%; vì vậy chưa đủ căn cứ để Release. Hạn chế còn lại: cần tiếp tục áp dụng quy trình quét toàn ảnh khi ẩn nhãn máy và theo dõi lỗi bỏ sót.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** YOLO26n/YOLOE tạo pre-label, SAM 2.1 tạo mask; kiểm tra và sửa bằng CVAT, kiểm tra ngưỡng trên gold và rà mẫu QC sau sửa.
- Gold có bị nhiễm không: không xem pre-label trước khi gán gold; có chạy smoke test trên `img_002.jpg` trước khi hoàn tất gold, nhưng ảnh đó không thuộc gold.
- Máy / GPU / CVAT: Windows, CPU-only, Python 3.12.10, CVAT Docker v2.81.0. Ban đầu chọn nhầm file COCO đầy đủ thay vì file import 24 ảnh; đã dùng file import đúng.
- Dùng cache `data/cache/`: không.
- Lỗi / bug script đã gặp: nhầm file import COCO; xử lý bằng `pre_traps_import.coco.json`.
- Một điều làm khác lần sau: ghi thời gian ngay khi làm và quét toàn ảnh trong lúc ẩn nhãn máy trước khi review từng box.
