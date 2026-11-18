# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 253; đối tượng sau duyệt: 240; ngưỡng accepted: IoU >= 0.9
- Đã **loại 19 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 220 | 220/253 = 87.0% | 253 box máy |
| edited (sửa) | 8 | 8/253 = 3.2% | 253 box máy |
| deleted (xoá) | 25 | 25/253 = 9.9% | 253 box máy |
| added (vẽ thêm) | 12 | 12/240 = 5.0% | 240 đối tượng sau duyệt |

Trong số deleted + added, **5 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 7.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 107 | 3 | 2 | 1 |
| bicycle | 4 | 0 | 1 | 1 |
| motorbike | 62 | 0 | 2 | 3 |
| car | 26 | 0 | 1 | 1 |
| bus | 2 | 0 | 0 | 2 |
| truck | 1 | 0 | 0 | 1 |
| three_wheeler | 18 | 5 | 19 | 3 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 49 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 29 | 7 | 20 | 80.6% (KTC 95%: 65–90%) | 59.2% (KTC 95%: 45–72%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 5/12 = 41.7% (KTC 95%: 19–68%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 2/3
  - loose: 1/3
  - extra: 2/3
  - missing: 0/3
  - lọt: img_029.jpg (loose), img_035.jpg (missing), img_009.jpg (missing), img_024.jpg (loose), img_010.jpg (missing), img_006.jpg (extra), img_013.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
