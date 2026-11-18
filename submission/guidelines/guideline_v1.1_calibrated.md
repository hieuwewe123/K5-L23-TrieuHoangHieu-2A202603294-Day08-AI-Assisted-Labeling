# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603294 - Trieu Hoang Hieu

> Chỉ ghi phần bổ sung so với v1.0. Quy tắc xe điện được chốt theo định nghĩa lớp nhưng không có ảnh xe điện trong bộ 36 ảnh để làm ví dụ.

## 1. Câu hỏi mở đã chốt

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ thuộc lớp nào? | Xe có động cơ là `motorbike`; xe đạp không động cơ là `bicycle`. Nếu không thể xác định có động cơ hay không, tick `can_review`. | Không gặp xe điện trong bộ dữ liệu | CAL-01 |
| 2 | Khối xe máy đỗ dày gần camera: gán từng xe hay `ignore_region`? | Gán riêng từng xe nhìn thấy rõ và cao từ 20 px. Dùng một `ignore_region` cho cụm không tách được hoặc vật dưới 20 px; không phụ thuộc gần hay xa camera. | `img_018` | CAL-02 |

## 2. Ca khó mới

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định | Quy tắc bổ sung |
|---|---|---|---|---|
| `img_034` | Một box `three_wheeler` không khớp nhãn cuối sau review | YOLOE sinh ứng viên `three_wheeler`; một box không được giữ sau đối chiếu | Không chấp nhận chỉ theo prompt hoặc confidence; xác nhận cấu trúc xe ba bánh nhìn thấy, nếu sai thì sửa lớp hoặc xóa box. | Bổ sung mục 2: xác nhận cấu trúc xe trước khi giữ `three_wheeler` (CAL-03). |
| `img_009`, `img_010`, `img_035` | Ba bẫy `missing` đều lọt | Không phát hiện vật bị thiếu trong lượt review ban đầu | Ẩn nhãn máy và quét toàn ảnh tìm vật bị sót trước; sau đó mới kiểm tra từng box máy. | Bổ sung quy trình review: thêm lượt quét toàn ảnh trước khi xem nhãn máy (CAL-04). |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (case_id) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Làm rõ phân lớp xe điện và ngưỡng `ignore_region`; thêm bước quét toàn ảnh và kiểm tra thủ công ứng viên `three_wheeler`. | CAL-01–CAL-04 |
