# Phân tích yêu cầu — vai Consumer

- Cặp đàm phán: Pair 02
- Product: Smart Campus Security System
- Consumer service: Core Business Service (A6)
- Provider service: AI Vision Service (A4)
- Người viết: Phùng Thế Anh
- Ngày: 15/05/2026

---

## 1. Resource Consumer cần nhận/gửi

| Resource | Consumer dùng để làm gì? | Field bắt buộc với Consumer | Field có thể tùy chọn |
|---|---|---|---|
| AnalysisResult | Xử lý kết quả phân tích camera và tạo cảnh báo | analysisId, cameraId, detectionType, confidence, timestamp | frameUrl, frameExpiredAt |
| DetectionEvent | Theo dõi sự kiện phát hiện mới nhất | detectionId, detectionType, timestamp | metadata |

---

## 2. API Consumer cần gọi

| Method | Path | Lúc nào gọi? | Kỳ vọng response |
|---|---|---|---|
| GET | `/analyses` | Khi cần lấy danh sách kết quả phân tích | Danh sách AnalysisResult |
| GET | `/analyses/{analysisId}` | Khi cần xem chi tiết một phân tích | Thông tin chi tiết analysis |
| GET | `/detections/latest` | Polling mỗi 5 giây để lấy detection mới | Detection mới nhất |
| GET | `/health` | Kiểm tra AI Vision còn hoạt động không | Status = healthy |

---

## 3. Error case Consumer cần xử lý

Tối thiểu 5 case.

| Status | Consumer hiểu là gì? | Consumer sẽ xử lý thế nào? |
|---:|---|---|
| 400 | Request sai schema | Sửa payload/log lỗi |
| 401 | Thiếu token | Refresh/cấu hình token |
| 403 | Không đủ quyền | Báo lỗi quyền truy cập |
| 404 | Không tìm thấy resource | Hiển thị trạng thái không tồn tại |
| 409 | Xung đột nghiệp vụ | Retry hoặc yêu cầu thao tác lại |
| 422 | Vi phạm rule nghiệp vụ | Hiển thị lý do cụ thể |
| 429 | Gọi API quá giới hạn | Retry sau một khoảng thời gian |
| 500 | Lỗi nội bộ Provider | Ghi log và retry có giới hạn |
| 503 | Service tạm thời không khả dụng | Chuyển trạng thái degraded mode |

---

## 4. Giả định bổ sung

- Giả định 1: AI Vision trả dữ liệu JSON đúng chuẩn OpenAPI 3.1.
- Giả định 2: Tất cả timestamp sử dụng định dạng ISO 8601.
- Giả định 3: Token JWT đã được cấp trước khi gọi API.

---

## 5. Câu hỏi cho Provider

1. frameUrl sẽ hết hạn sau bao lâu?
2. Có giới hạn số request/phút cho mỗi consumer không?
3. detectionType có thể mở rộng thêm giá trị mới không?

---

## 6. Rủi ro tích hợp

| Rủi ro | Tác động | Đề xuất xử lý |
|---|---|---|
| Provider đổi kiểu dữ liệu | Consumer parse lỗi | Chốt type/format/pattern |
| Provider thiếu mã lỗi | Consumer khó xử lý lỗi | Chuẩn hóa Problem Details |
| API phản hồi chậm | Core Business bị block | Timeout + circuit breaker |
| Detection trả dữ liệu thiếu field | Không tạo được alert | Validate schema trước xử lý |
| Provider thay đổi version API | Consumer bị lỗi tương thích | Áp dụng semantic versioning |