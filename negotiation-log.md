# Bien ban dam phan hop dong API

- Cap dam phan: Cap 2 (Core <-> AI Vision) va Cap 3 (Core <-> Access Gate)
- Product: A
- Provider: AI Vision (A4) — Cap 2 | Access Gate (A3) — Cap 3
- Consumer: Core Business (A6)
- Phien: v1.0
- Ngay: 2026-05-15

---

## Issue #1

- Raised by: Consumer
- Endpoint: GET /detections/latest
- Concern: Core can nhan ket qua phan tich kip thoi de trigger alert.
  Consumer muon webhook push de giam do tre. Provider chua co
  infrastructure webhook.
- Proposal: Consumer de xuat webhook POST /core/detections.
  Provider de xuat polling GET /detections/latest interval 5 giay.
- Resolution: Modified
- Rationale: Polling don gian hon, du dap ung yeu cau demo trong Lab 02.
  Webhook tang do phuc tap infrastructure khong can thiet o giai doan nay.
  Ghi nhan webhook vao backlog Lab 03.
- Impact: Core tu quan ly polling interval toi thieu 5 giay.
  AI Vision dam bao /detections/latest tra ve duoi 200ms.

---

## Issue #2

- Raised by: Consumer
- Endpoint: GET /analyses, GET /detections/latest
- Concern: Ket qua detect co truong confidence (0.0-1.0). Provider muon
  loc san confidence > 0.7 truoc khi tra. Consumer muon nhan toan bo
  va tu loc theo nghiep vu.
- Proposal: Them query param minConfidence optional, default null
  (khong loc).
- Resolution: Accepted
- Rationale: Consumer co nghiep vu khac nhau (alert an ninh vs thong ke)
  can threshold khac nhau. Provider khong nen hard-code logic nghiep vu.
- Impact: Them param minConfidence: number vao GET /analyses va
  GET /detections/latest.

---

## Issue #3

- Raised by: Consumer
- Endpoint: GET /analyses, GET /analyses/{analysisId}
- Concern: AnalysisResult co truong frameUrl de Core hien thi anh bang
  chung. Provider chi luu frame 24h vi dung luong. Consumer khong biet
  khi nao frameUrl bi null.
- Proposal: Them truong frameExpiredAt de Consumer biet han luu tru
  cua frame.
- Resolution: Accepted
- Rationale: Consumer can biet truoc de khong hien thi nut Xem anh khi
  frame da het han, tranh UX loi.
- Impact: Them truong frameExpiredAt: string (date-time) | null vao
  schema AnalysisResult.

---

## Issue #4

- Raised by: Consumer
- Endpoint: GET /access-logs
- Concern: Provider muon Unix timestamp cho param from/to vi nhe hon.
  Consumer muon ISO 8601 string de de debug va dong bo voi cac endpoint
  khac.
- Proposal: Dung ISO 8601 (format: date-time) cho ca from va to.
- Resolution: Accepted
- Rationale: ISO 8601 de doc, de debug, la convention chuan trong OpenAPI.
  Overhead khong dang ke. Dong bo voi cac truong timestamp khac trong
  toan bo hop dong.
- Impact: Params from/to khai bao format: date-time trong schema.

---

## Issue #5

- Raised by: Provider
- Endpoint: GET /access-logs
- Concern: Provider muon cap limit = 100 de tranh qua tai.
  Consumer muon lay toi da 500 de batch xu ly.
- Proposal: Thong nhat limit toi da = 200. Neu vuot qua tra 400
  Bad Request voi ProblemDetails.
- Resolution: Modified
- Rationale: 200 du cho batch xu ly ma khong gay qua tai Provider.
  Can bao loi ro rang thay vi silently truncate.
- Impact: Schema them maximum: 200 cho param limit. Them case 400
  vao response GET /access-logs.

---

## Issue #6

- Raised by: Consumer
- Endpoint: Tat ca endpoint
- Concern: Can thong nhat format loi 4xx/5xx giua cac service.
  Provider muon tra JSON tu do. Consumer muon format chuan de xu ly
  loi tu dong.
- Proposal: Dung RFC 7807 Problem Details voi content-type
  application/problem+json.
- Resolution: Accepted
- Rationale: Problem Details la chuan cong nghiep, giup Core parse loi
  nhat quan ma khong can hard-code tung endpoint. Dong bo voi cac cap
  khac trong Smart Campus platform.
- Impact: Tat ca 4xx/5xx response trong openapi.yaml dung
  application/problem+json voi schema Problem.

---

# Chot hop dong v1.0

Provider sign-off: AI Vision (A4) + Access Gate (A3) — Nhom 4
Consumer sign-off: Core Business (A6) — Nhom 4
Witness (GV/TA):
Date: 2026-05-15

---

## Ghi chu warning neu Spectral con canh bao

| Warning | Ly do chap nhan tam thoi |
|---------|--------------------------|
| operation-description missing | Da bo sung description cho tat ca operation |
| oas3-unused-component Forbidden | Da su dung Forbidden o endpoint /analyses |