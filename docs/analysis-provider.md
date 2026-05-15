# Phân tích yêu cầu — vai Provider

- Cặp đàm phán: Pair 02
- Product: Smart Campus Security System
- Provider service: AI Vision Service (A4)
- Consumer service: Core Business Service (A6)
- Người viết: Phùng Thế Anh
- Ngày: 15/05/2026

---

## 1. Resource chính

| Resource | Mô tả | Thuộc tính bắt buộc | Thuộc tính tùy chọn |
|---|---|---|---|
| AnalysisResult | Kết quả phân tích từ camera AI | analysisId, cameraId, detectionType, confidence, timestamp | frameUrl, frameExpiredAt |
| DetectionEvent | Sự kiện phát hiện đối tượng | detectionId, detectionType, timestamp | metadata |

---

## 2. Action/API dự kiến

| Method | Path | Mục đích | Consumer gọi khi nào? |
|---|---|---|---|
| GET | `/analyses` | Lấy danh sách kết quả phân tích | Khi cần dashboard hoặc thống kê |
| GET | `/analyses/{analysisId}` | Lấy chi tiết một phân tích | Khi cần trace sự kiện |
| GET | `/detections/latest` | Trả detection mới nhất | Polling real-time |
| GET | `/health` | Kiểm tra trạng thái service | Monitoring hoặc health check |

---

## 3. Error case

Tối thiểu 5 case.

| Status | Tình huống | Response body dự kiến |
|---:|---|---|
| 400 | Payload sai định dạng | `Problem` |
| 401 | Thiếu Bearer token | `Problem` |
| 403 | Token hợp lệ nhưng không có quyền | `Problem` |
| 404 | Resource không tồn tại | `Problem` |
| 409 | Xung đột nghiệp vụ | `Problem` |
| 422 | Dữ liệu đúng JSON nhưng vi phạm nghiệp vụ | `Problem` |
| 429 | Consumer gọi quá giới hạn | `Problem` |
| 500 | Lỗi xử lý nội bộ AI Vision | `Problem` |
| 503 | Service đang bảo trì | `Problem` |

---

## 4. Giả định bổ sung

Ghi rõ những điểm user story chưa nói nhưng Provider cần giả định.

- Giả định 1: Consumer sử dụng JWT Bearer Authentication.
- Giả định 2: Dữ liệu thời gian dùng chuẩn ISO 8601.
- Giả định 3: frameUrl chỉ tồn tại tối đa 24 giờ để tiết kiệm lưu trữ.

---

## 5. Câu hỏi cho Consumer

1. Consumer cần polling dữ liệu với tần suất bao nhiêu?
2. Consumer có cần filter theo confidence threshold không?
3. Consumer có yêu cầu lưu lịch sử detection dài hạn không?

---

## 6. Rủi ro tích hợp

| Rủi ro | Tác động | Đề xuất xử lý |
|---|---|---|
| Tên field không thống nhất | Consumer parse lỗi | Chốt naming trong `openapi.yaml` |
| Payload lớn | Timeout/mock lỗi | Thống nhất content-type và size limit |
| Consumer polling quá nhiều | Quá tải service | Áp dụng rate limit |
| Consumer không validate schema | Dữ liệu lỗi lan sang hệ thống | Validate bằng OpenAPI schema |
| Version API thay đổi | Mất tương thích | Dùng semantic versioning |