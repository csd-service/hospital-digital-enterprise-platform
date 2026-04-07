# QUY CHUẨN THIẾT KẾ REST API

**Phiên bản:** 1.0
**Ngày ban hành:** 07/04/2026
**Phạm vi áp dụng:** Toàn bộ đội ngũ phát triển (Backend, Frontend, Mobile, QA)
**Trạng thái:** Bắt buộc tuân thủ

---

## MỤC LỤC

1. [Mục đích và Phạm vi](#1-mục-đích-và-phạm-vi)
2. [Quy tắc chung](#2-quy-tắc-chung)
3. [Thiết kế URL (Endpoint)](#3-thiết-kế-url-endpoint)
4. [HTTP Methods](#4-http-methods)
5. [HTTP Status Codes](#5-http-status-codes)
6. [Cấu trúc Request](#6-cấu-trúc-request)
7. [Cấu trúc Response](#7-cấu-trúc-response)
8. [Phân trang (Pagination)](#8-phân-trang-pagination)
9. [Lọc, Sắp xếp và Tìm kiếm](#9-lọc-sắp-xếp-và-tìm-kiếm)
10. [Versioning](#10-versioning)
11. [Xác thực và Phân quyền](#11-xác-thực-và-phân-quyền)
12. [Rate Limiting](#12-rate-limiting)
13. [Xử lý lỗi (Error Handling)](#13-xử-lý-lỗi-error-handling)
14. [Mô tả API bắt buộc (API Documentation)](#14-mô-tả-api-bắt-buộc-api-documentation)
15. [Bảo mật](#15-bảo-mật)
16. [Hiệu năng và Caching](#16-hiệu-năng-và-caching)
17. [Idempotency](#17-idempotency)
18. [Checklist Review API](#18-checklist-review-api)

---

## 1. MỤC ĐÍCH VÀ PHẠM VI

### 1.1. Mục đích

Tài liệu này thiết lập bộ quy chuẩn thống nhất cho việc thiết kế, phát triển và bảo trì REST API trong toàn tổ chức. Mọi API mới hoặc API được nâng cấp **bắt buộc** phải tuân thủ toàn bộ quy chuẩn trong tài liệu này.

### 1.2. Phạm vi áp dụng

- Tất cả REST API nội bộ (internal) và công khai (public).
- Tất cả microservices giao tiếp qua HTTP.
- Áp dụng cho mọi ngôn ngữ lập trình và framework.

### 1.3. Định nghĩa mức độ tuân thủ

| Ký hiệu | Ý nghĩa |
|---|---|
| **[BẮT BUỘC]** | Phải tuân thủ 100%. Vi phạm sẽ bị từ chối trong code review. |
| **[KHUYẾN NGHỊ]** | Nên tuân thủ. Ngoại lệ cần được Technical Lead phê duyệt bằng văn bản. |
| **[TÙY CHỌN]** | Áp dụng tùy theo ngữ cảnh dự án. |

---

## 2. QUY TẮC CHUNG

**[BẮT BUỘC]** Mọi API phải tuân thủ các nguyên tắc kiến trúc REST:

- **Stateless:** Mỗi request phải chứa đầy đủ thông tin cần thiết để xử lý. Server không lưu trạng thái phiên (session) giữa các request.
- **Client-Server:** Client và Server hoạt động độc lập, giao tiếp thông qua giao diện thống nhất.
- **Uniform Interface:** Sử dụng giao diện nhất quán — resource được định danh qua URI, thao tác qua HTTP methods.
- **Cacheable:** Response phải khai báo rõ ràng khả năng cache.

**[BẮT BUỘC]** Tất cả API phải sử dụng giao thức **HTTPS**. Không chấp nhận HTTP không mã hóa.

**[BẮT BUỘC]** Định dạng dữ liệu trao đổi mặc định là **JSON** (`Content-Type: application/json`).

**[BẮT BUỘC]** Sử dụng **UTF-8** cho tất cả dữ liệu text.

---

## 3. THIẾT KẾ URL (ENDPOINT)

### 3.1. Quy tắc đặt tên

**[BẮT BUỘC]** URL phải tuân thủ các quy tắc sau:

| Quy tắc | Đúng ✅ | Sai ❌ |
|---|---|---|
| Sử dụng **danh từ số nhiều** | `/users` | `/user`, `/getUser` |
| Sử dụng **chữ thường** | `/order-items` | `/OrderItems`, `/order_items` |
| Dùng **dấu gạch ngang** (kebab-case) | `/user-profiles` | `/user_profiles`, `/userProfiles` |
| Không dùng **động từ** | `/users` | `/getUsers`, `/createUser` |
| Không có **trailing slash** | `/users` | `/users/` |
| Không có **phần mở rộng file** | `/users/123` | `/users/123.json` |

### 3.2. Cấu trúc phân cấp (Hierarchical)

**[BẮT BUỘC]** Thể hiện quan hệ cha-con thông qua đường dẫn phân cấp, giới hạn tối đa **3 cấp**.

```
# Cấu trúc chuẩn
GET  /users/{userId}/orders/{orderId}/items

# Ví dụ cụ thể
GET  /users/123/orders                    # Lấy danh sách đơn hàng của user 123
GET  /users/123/orders/456                # Lấy đơn hàng 456 của user 123
GET  /users/123/orders/456/items          # Lấy items trong đơn hàng 456 (tối đa 3 cấp)
```

**[BẮT BUỘC]** Nếu vượt quá 3 cấp, tách thành endpoint riêng:

```
# SAI ❌ — quá 3 cấp
GET  /users/123/orders/456/items/789/reviews

# ĐÚNG ✅ — tách resource riêng
GET  /order-items/789/reviews
```

### 3.3. Base URL

**[BẮT BUỘC]** Cấu trúc base URL:

```
https://api.{domain}.com/{version}/{resource}
```

Ví dụ:
```
https://api.mycompany.com/v1/users
https://api.mycompany.com/v2/products
```

---

## 4. HTTP METHODS

**[BẮT BUỘC]** Sử dụng đúng HTTP method theo ngữ nghĩa:

| Method | Mục đích | Idempotent | Request Body | Ví dụ |
|---|---|---|---|---|
| `GET` | Đọc resource | ✅ Có | ❌ Không | `GET /users/123` |
| `POST` | Tạo resource mới | ❌ Không | ✅ Có | `POST /users` |
| `PUT` | Cập nhật toàn bộ resource | ✅ Có | ✅ Có | `PUT /users/123` |
| `PATCH` | Cập nhật một phần resource | ❌ Không | ✅ Có | `PATCH /users/123` |
| `DELETE` | Xóa resource | ✅ Có | ❌ Không | `DELETE /users/123` |

### 4.1. Phân biệt PUT vs PATCH

```json
// PUT /users/123 — Cập nhật TOÀN BỘ (phải gửi đầy đủ fields)
{
  "name": "Nguyễn Văn A",
  "email": "a.nguyen@example.com",
  "phone": "0901234567",
  "role": "admin"
}

// PATCH /users/123 — Cập nhật MỘT PHẦN (chỉ gửi fields cần thay đổi)
{
  "role": "admin"
}
```

### 4.2. Hành động đặc biệt (Actions)

**[KHUYẾN NGHỊ]** Với các thao tác không thuộc CRUD chuẩn, sử dụng sub-resource hoặc hậu tố hành động:

```
POST /users/123/activate          # Kích hoạt tài khoản
POST /users/123/deactivate        # Vô hiệu hóa tài khoản
POST /orders/456/cancel           # Hủy đơn hàng
POST /payments/789/refund         # Hoàn tiền
```

---

## 5. HTTP STATUS CODES

**[BẮT BUỘC]** Sử dụng đúng status code theo ngữ cảnh:

### 5.1. Nhóm 2xx — Thành công

| Code | Ý nghĩa | Sử dụng khi |
|---|---|---|
| `200 OK` | Thành công | GET, PUT, PATCH, DELETE thành công |
| `201 Created` | Tạo mới thành công | POST tạo resource thành công |
| `204 No Content` | Thành công, không trả body | DELETE thành công, không có dữ liệu trả về |

### 5.2. Nhóm 4xx — Lỗi phía Client

| Code | Ý nghĩa | Sử dụng khi |
|---|---|---|
| `400 Bad Request` | Request không hợp lệ | Dữ liệu đầu vào sai format, thiếu trường bắt buộc |
| `401 Unauthorized` | Chưa xác thực | Thiếu hoặc sai token xác thực |
| `403 Forbidden` | Không có quyền | Đã xác thực nhưng không đủ quyền truy cập |
| `404 Not Found` | Không tìm thấy | Resource không tồn tại |
| `409 Conflict` | Xung đột | Tạo resource đã tồn tại, vi phạm ràng buộc duy nhất |
| `422 Unprocessable Entity` | Không thể xử lý | Dữ liệu đúng format nhưng vi phạm logic nghiệp vụ |
| `429 Too Many Requests` | Vượt giới hạn | Client gửi quá nhiều request (rate limit) |

### 5.3. Nhóm 5xx — Lỗi phía Server

| Code | Ý nghĩa | Sử dụng khi |
|---|---|---|
| `500 Internal Server Error` | Lỗi server | Lỗi không xác định phía server |
| `502 Bad Gateway` | Lỗi gateway | Proxy/gateway nhận response không hợp lệ |
| `503 Service Unavailable` | Dịch vụ tạm ngưng | Server quá tải hoặc đang bảo trì |

---

## 6. CẤU TRÚC REQUEST

### 6.1. Headers bắt buộc

**[BẮT BUỘC]** Mọi request phải bao gồm các headers sau:

```http
Content-Type: application/json
Accept: application/json
Authorization: Bearer {access_token}
X-Request-Id: {uuid}               # Unique ID cho mỗi request, dùng để trace
```

**[KHUYẾN NGHỊ]** Headers bổ sung:

```http
Accept-Language: vi-VN              # Ngôn ngữ response
X-Idempotency-Key: {uuid}          # Cho các POST request cần idempotent
```

### 6.2. Request Body — Quy tắc đặt tên field

**[BẮT BUỘC]** Sử dụng **camelCase** cho tất cả field names:

```json
// ĐÚNG ✅
{
  "firstName": "Nguyễn",
  "lastName": "Văn A",
  "emailAddress": "a.nguyen@example.com",
  "phoneNumber": "0901234567",
  "dateOfBirth": "1990-05-15"
}

// SAI ❌
{
  "first_name": "Nguyễn",
  "LastName": "Văn A",
  "email-address": "a.nguyen@example.com"
}
```

### 6.3. Quy tắc định dạng dữ liệu

**[BẮT BUỘC]** Áp dụng các định dạng chuẩn sau:

| Loại dữ liệu | Định dạng | Ví dụ |
|---|---|---|
| Ngày giờ | ISO 8601 (UTC) | `"2026-04-03T10:30:00Z"` |
| Ngày | ISO 8601 | `"2026-04-03"` |
| Tiền tệ | Số nguyên (đơn vị nhỏ nhất) | `150000` (= 150.000 VNĐ) |
| UUID | v4 | `"f47ac10b-58cc-4372-a567-0e02b2c3d479"` |
| Enum | UPPER_SNAKE_CASE | `"ORDER_PENDING"`, `"PAYMENT_SUCCESS"` |
| Boolean | `true` / `false` | Không dùng `0`, `1`, `"yes"`, `"no"` |
| Null | `null` | Không dùng `""`, `0`, `"null"` |

---

## 7. CẤU TRÚC RESPONSE

### 7.1. Response thành công — Đối tượng đơn lẻ

**[BẮT BUỘC]** Mọi response thành công phải bọc trong field `data`:

```json
// GET /users/123
// HTTP 200 OK
{
  "data": {
    "id": "usr_123",
    "firstName": "Nguyễn",
    "lastName": "Văn A",
    "email": "a.nguyen@example.com",
    "role": "ADMIN",
    "createdAt": "2026-01-15T10:30:00Z",
    "updatedAt": "2026-03-20T14:00:00Z"
  },
  "meta": {
    "requestId": "req_abc123def456",
    "timestamp": "2026-04-03T10:30:00Z"
  }
}
```

### 7.2. Response thành công — Danh sách (có phân trang)

```json
// GET /users?page=1&limit=20
// HTTP 200 OK
{
  "data": [
    {
      "id": "usr_123",
      "firstName": "Nguyễn",
      "lastName": "Văn A",
      "email": "a.nguyen@example.com"
    },
    {
      "id": "usr_124",
      "firstName": "Trần",
      "lastName": "Thị B",
      "email": "b.tran@example.com"
    }
  ],
  "meta": {
    "requestId": "req_abc123def456",
    "timestamp": "2026-04-03T10:30:00Z",
    "pagination": {
      "currentPage": 1,
      "perPage": 20,
      "totalItems": 1543,
      "totalPages": 78
    }
  },
  "links": {
    "self": "/v1/users?page=1&limit=20",
    "next": "/v1/users?page=2&limit=20",
    "prev": null,
    "first": "/v1/users?page=1&limit=20",
    "last": "/v1/users?page=78&limit=20"
  }
}
```

### 7.3. Response tạo mới thành công

```json
// POST /users
// HTTP 201 Created
// Header: Location: /v1/users/usr_125
{
  "data": {
    "id": "usr_125",
    "firstName": "Lê",
    "lastName": "Văn C",
    "email": "c.le@example.com",
    "role": "MEMBER",
    "createdAt": "2026-04-03T10:35:00Z"
  },
  "meta": {
    "requestId": "req_xyz789",
    "timestamp": "2026-04-03T10:35:00Z"
  }
}
```

### 7.4. Response xóa thành công

```
// DELETE /users/123
// HTTP 204 No Content
// (Không có response body)
```

---

## 8. PHÂN TRANG (PAGINATION)

### 8.1. Offset-Based Pagination (Mặc định)

**[BẮT BUỘC]** Mọi endpoint trả về danh sách phải hỗ trợ phân trang.

**Tham số bắt buộc:**

| Param | Kiểu | Mặc định | Giới hạn | Mô tả |
|---|---|---|---|---|
| `page` | integer | `1` | ≥ 1 | Trang hiện tại |
| `limit` | integer | `20` | 1–100 | Số bản ghi mỗi trang |

```
GET /users?page=2&limit=25
```

**[BẮT BUỘC]** Response phải bao gồm `meta.pagination` và `links` (xem mục 7.2).

### 8.2. Cursor-Based Pagination (cho dữ liệu lớn)

**[KHUYẾN NGHỊ]** Sử dụng cursor-based pagination cho dataset lớn (>100.000 bản ghi) hoặc dữ liệu real-time.

```
GET /events?cursor=eyJpZCI6MTIzNH0&limit=50
```

Response:
```json
{
  "data": [...],
  "meta": {
    "pagination": {
      "perPage": 50,
      "hasMore": true,
      "nextCursor": "eyJpZCI6MTI4NH0",
      "prevCursor": "eyJpZCI6MTE4NH0"
    }
  }
}
```

---

## 9. LỌC, SẮP XẾP VÀ TÌM KIẾM

### 9.1. Lọc (Filtering)

**[BẮT BUỘC]** Sử dụng query parameters cho bộ lọc:

```
GET /orders?status=PENDING&createdFrom=2026-01-01&createdTo=2026-03-31
GET /products?categoryId=cat_123&minPrice=100000&maxPrice=500000
GET /users?role=ADMIN&isActive=true
```

**[BẮT BUỘC]** Tên filter phải trùng khớp với tên field trong resource hoặc sử dụng prefix rõ ràng (`min`, `max`, `From`, `To`).

### 9.2. Sắp xếp (Sorting)

**[BẮT BUỘC]** Sử dụng param `sort` với quy ước:

- Prefix `-` cho sắp xếp **giảm dần** (descending).
- Không prefix cho sắp xếp **tăng dần** (ascending).
- Hỗ trợ nhiều trường, cách nhau bằng dấu phẩy.

```
GET /products?sort=-createdAt              # Mới nhất trước
GET /products?sort=price,-rating           # Giá tăng dần, rating giảm dần
GET /users?sort=lastName,firstName         # Tên theo alphabet
```

### 9.3. Tìm kiếm (Search)

**[KHUYẾN NGHỊ]** Sử dụng param `q` cho full-text search:

```
GET /products?q=iphone+16+pro
GET /users?q=nguyễn+văn
```

### 9.4. Chọn trường (Field Selection)

**[TÙY CHỌN]** Sử dụng param `fields` để giới hạn trường trả về:

```
GET /users?fields=id,firstName,lastName,email
```

---

## 10. VERSIONING

**[BẮT BUỘC]** Sử dụng **URL Path Versioning**:

```
https://api.mycompany.com/v1/users
https://api.mycompany.com/v2/users
```

**[BẮT BUỘC]** Quy tắc versioning:

- Chỉ tăng version khi có **breaking change** (xóa field, đổi kiểu dữ liệu, thay đổi hành vi).
- Thêm field mới vào response **không** được coi là breaking change.
- Hỗ trợ đồng thời tối thiểu **2 version** (current + previous).
- Thông báo deprecation ít nhất **6 tháng** trước khi ngừng hỗ trợ version cũ.

**[BẮT BUỘC]** API deprecated phải trả header cảnh báo:

```http
Deprecation: true
Sunset: Sat, 01 Nov 2026 00:00:00 GMT
Link: </v2/users>; rel="successor-version"
```

---

## 11. XÁC THỰC VÀ PHÂN QUYỀN

### 11.1. Phương thức xác thực

**[BẮT BUỘC]** Sử dụng một trong các phương thức sau tùy ngữ cảnh:

| Phương thức | Sử dụng cho | Ví dụ |
|---|---|---|
| **JWT Bearer Token** | Client-to-server (web, mobile) | `Authorization: Bearer eyJhbGciOi...` |
| **OAuth 2.0** | Tích hợp bên thứ ba | Authorization Code Flow, Client Credentials |
| **API Key** | Server-to-server nội bộ | `X-API-Key: sk_live_abc123...` |

### 11.2. JWT Token — Quy định

**[BẮT BUỘC]** Access token:
- Thời gian sống (TTL): tối đa **15 phút**.
- Chứa tối thiểu: `sub` (user ID), `exp`, `iat`, `roles`.

**[BẮT BUỘC]** Refresh token:
- TTL: tối đa **7 ngày**.
- Lưu trữ an toàn (httpOnly cookie hoặc secure storage).
- Rotate sau mỗi lần sử dụng.

### 11.3. Endpoint xác thực

```
POST   /auth/login           # Đăng nhập, trả về access + refresh token
POST   /auth/refresh          # Làm mới access token
POST   /auth/logout           # Thu hồi refresh token
POST   /auth/forgot-password  # Yêu cầu reset password
POST   /auth/reset-password   # Thực hiện reset password
```

---

## 12. RATE LIMITING

**[BẮT BUỘC]** Mọi API phải áp dụng rate limiting.

### 12.1. Giới hạn mặc định

| Loại client | Giới hạn | Cửa sổ thời gian |
|---|---|---|
| Public API (không xác thực) | 60 requests | 1 phút |
| Authenticated User | 300 requests | 1 phút |
| Service-to-Service (API Key) | 1000 requests | 1 phút |
| Admin / Internal | 3000 requests | 1 phút |

### 12.2. Response Headers

**[BẮT BUỘC]** Mọi response phải bao gồm rate limit headers:

```http
X-RateLimit-Limit: 300
X-RateLimit-Remaining: 247
X-RateLimit-Reset: 1712140800       # Unix timestamp khi limit reset
Retry-After: 30                     # Số giây cần chờ (chỉ khi bị 429)
```

### 12.3. Response khi vượt giới hạn

```json
// HTTP 429 Too Many Requests
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Bạn đã vượt quá giới hạn 300 requests/phút. Vui lòng thử lại sau 30 giây.",
    "retryAfter": 30
  },
  "meta": {
    "requestId": "req_abc123",
    "timestamp": "2026-04-03T10:30:00Z"
  }
}
```

---

## 13. XỬ LÝ LỖI (ERROR HANDLING)

### 13.1. Cấu trúc Error Response

**[BẮT BUỘC]** Tất cả error response phải tuân thủ cấu trúc sau:

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Mô tả lỗi thân thiện với người dùng cuối",
    "details": []
  },
  "meta": {
    "requestId": "req_abc123",
    "timestamp": "2026-04-03T10:30:00Z"
  }
}
```

### 13.2. Lỗi Validation (400 / 422)

```json
// POST /users — thiếu field bắt buộc, email sai format
// HTTP 422 Unprocessable Entity
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Dữ liệu đầu vào không hợp lệ.",
    "details": [
      {
        "field": "email",
        "code": "INVALID_FORMAT",
        "message": "Email không đúng định dạng. Ví dụ: user@example.com"
      },
      {
        "field": "firstName",
        "code": "REQUIRED",
        "message": "Họ là trường bắt buộc."
      },
      {
        "field": "phone",
        "code": "INVALID_LENGTH",
        "message": "Số điện thoại phải có 10-11 ký tự.",
        "constraints": {
          "minLength": 10,
          "maxLength": 11
        }
      }
    ]
  },
  "meta": {
    "requestId": "req_abc123",
    "timestamp": "2026-04-03T10:30:00Z"
  }
}
```

### 13.3. Mã lỗi chuẩn (Error Codes)

**[BẮT BUỘC]** Sử dụng error codes nhất quán trong toàn hệ thống:

| Error Code | HTTP Status | Mô tả |
|---|---|---|
| `VALIDATION_ERROR` | 400 / 422 | Dữ liệu đầu vào không hợp lệ |
| `AUTHENTICATION_REQUIRED` | 401 | Chưa xác thực |
| `TOKEN_EXPIRED` | 401 | Token đã hết hạn |
| `INSUFFICIENT_PERMISSIONS` | 403 | Không đủ quyền |
| `RESOURCE_NOT_FOUND` | 404 | Resource không tồn tại |
| `RESOURCE_ALREADY_EXISTS` | 409 | Resource đã tồn tại (trùng lặp) |
| `BUSINESS_RULE_VIOLATION` | 422 | Vi phạm logic nghiệp vụ |
| `RATE_LIMIT_EXCEEDED` | 429 | Vượt giới hạn request |
| `INTERNAL_ERROR` | 500 | Lỗi hệ thống |
| `SERVICE_UNAVAILABLE` | 503 | Dịch vụ tạm ngưng |

### 13.4. Quy tắc bảo mật trong Error Response

**[BẮT BUỘC]** Không bao giờ trả về trong error response:
- Stack trace hoặc thông tin debug nội bộ.
- Tên bảng, tên cột database.
- Đường dẫn file server.
- Thông tin cấu hình hệ thống.

---

## 14. MÔ TẢ API BẮT BUỘC (API DOCUMENTATION)

> **Đây là phần QUAN TRỌNG NHẤT. Không có API nào được phép deploy nếu chưa có tài liệu mô tả đầy đủ.**

### 14.1. Công cụ và tiêu chuẩn

**[BẮT BUỘC]** Sử dụng **OpenAPI Specification 3.1** (hoặc mới hơn) để mô tả API.

**[BẮT BUỘC]** File spec phải được lưu trữ cùng source code (ví dụ: `docs/openapi.yaml`).

**[BẮT BUỘC]** Mọi thay đổi API phải cập nhật spec trước khi merge.

### 14.2. Thông tin bắt buộc cho MỖI endpoint

**[BẮT BUỘC]** Mỗi endpoint phải có đầy đủ các mục sau:

```yaml
# Ví dụ: POST /v1/users
paths:
  /v1/users:
    post:
      # 1. MÔ TẢ TỔNG QUÁT (BẮT BUỘC)
      summary: "Tạo tài khoản người dùng mới"
      description: |
        Tạo một tài khoản người dùng mới trong hệ thống.

        **Logic nghiệp vụ:**
        - Email phải duy nhất trong hệ thống.
        - Password phải có ít nhất 8 ký tự, bao gồm chữ hoa,
          chữ thường, số và ký tự đặc biệt.
        - Sau khi tạo thành công, hệ thống gửi email xác thực.
        - Tài khoản ở trạng thái PENDING cho đến khi xác thực email.

        **Quyền truy cập:** ADMIN, MANAGER

        **Rate limit:** 10 requests/phút/IP

      # 2. TAGS VÀ PHÂN NHÓM (BẮT BUỘC)
      tags:
        - Users
      operationId: createUser

      # 3. SECURITY (BẮT BUỘC)
      security:
        - bearerAuth: []

      # 4. PARAMETERS — MÔ TẢ CHI TIẾT TỪNG PARAM (BẮT BUỘC)
      # (Xem mục 14.3 bên dưới)

      # 5. REQUEST BODY — MÔ TẢ CHI TIẾT (BẮT BUỘC)
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserRequest'
            example:
              firstName: "Nguyễn"
              lastName: "Văn A"
              email: "a.nguyen@example.com"
              password: "SecureP@ss123"
              phone: "0901234567"
              role: "MEMBER"

      # 6. RESPONSES — MỌI STATUS CODE CÓ THỂ XẢY RA (BẮT BUỘC)
      responses:
        '201':
          description: "Tạo tài khoản thành công"
          headers:
            Location:
              description: "URL của resource vừa tạo"
              schema:
                type: string
                example: "/v1/users/usr_125"
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserResponse'
              example:
                data:
                  id: "usr_125"
                  firstName: "Nguyễn"
                  lastName: "Văn A"
                  email: "a.nguyen@example.com"
                  role: "MEMBER"
                  status: "PENDING"
                  createdAt: "2026-04-03T10:35:00Z"
                meta:
                  requestId: "req_xyz789"
                  timestamp: "2026-04-03T10:35:00Z"
        '400':
          description: "Request body không đúng format JSON"
        '401':
          description: "Chưa xác thực — thiếu hoặc sai Bearer token"
        '403':
          description: "Không đủ quyền — yêu cầu role ADMIN hoặc MANAGER"
        '409':
          description: "Email đã tồn tại trong hệ thống"
          content:
            application/json:
              example:
                error:
                  code: "RESOURCE_ALREADY_EXISTS"
                  message: "Email a.nguyen@example.com đã được sử dụng."
        '422':
          description: "Dữ liệu không hợp lệ"
          content:
            application/json:
              example:
                error:
                  code: "VALIDATION_ERROR"
                  message: "Dữ liệu đầu vào không hợp lệ."
                  details:
                    - field: "password"
                      code: "WEAK_PASSWORD"
                      message: "Mật khẩu phải có ít nhất 8 ký tự, bao gồm chữ hoa, chữ thường, số và ký tự đặc biệt."
        '429':
          description: "Vượt giới hạn rate limit"
```

### 14.3. Mô tả Parameters bắt buộc

**[BẮT BUỘC]** Mỗi parameter (path, query, header) phải có đầy đủ:

```yaml
parameters:
  # Path parameter
  - name: userId
    in: path
    required: true
    description: |
      ID duy nhất của người dùng.
      Format: chuỗi có prefix "usr_" theo sau bởi số.
    schema:
      type: string
      pattern: "^usr_[0-9]+$"
      example: "usr_123"

  # Query parameters
  - name: page
    in: query
    required: false
    description: "Số trang hiện tại. Bắt đầu từ 1."
    schema:
      type: integer
      minimum: 1
      default: 1
      example: 1

  - name: limit
    in: query
    required: false
    description: "Số bản ghi trên mỗi trang."
    schema:
      type: integer
      minimum: 1
      maximum: 100
      default: 20
      example: 20

  - name: status
    in: query
    required: false
    description: |
      Lọc theo trạng thái người dùng.
      Hỗ trợ nhiều giá trị, cách nhau bằng dấu phẩy.
    schema:
      type: string
      enum: ["ACTIVE", "PENDING", "SUSPENDED", "DELETED"]
      example: "ACTIVE"

  - name: sort
    in: query
    required: false
    description: |
      Sắp xếp kết quả. Prefix "-" cho giảm dần.
      Hỗ trợ: createdAt, lastName, email.
    schema:
      type: string
      default: "-createdAt"
      example: "-createdAt"

  - name: q
    in: query
    required: false
    description: |
      Tìm kiếm full-text theo tên hoặc email.
      Tối thiểu 2 ký tự, tối đa 100 ký tự.
    schema:
      type: string
      minLength: 2
      maxLength: 100
      example: "nguyễn"
```

### 14.4. Mô tả Schema bắt buộc

**[BẮT BUỘC]** Mỗi field trong schema phải khai báo đầy đủ:

```yaml
components:
  schemas:
    CreateUserRequest:
      type: object
      required:
        - firstName
        - lastName
        - email
        - password
      properties:
        firstName:
          type: string
          description: "Họ của người dùng"
          minLength: 1
          maxLength: 50
          example: "Nguyễn"
        lastName:
          type: string
          description: "Tên của người dùng"
          minLength: 1
          maxLength: 50
          example: "Văn A"
        email:
          type: string
          format: email
          description: "Địa chỉ email. Phải duy nhất trong hệ thống."
          maxLength: 255
          example: "a.nguyen@example.com"
        password:
          type: string
          format: password
          description: |
            Mật khẩu tài khoản.
            Yêu cầu: ≥8 ký tự, bao gồm chữ hoa, chữ thường,
            số và ít nhất 1 ký tự đặc biệt.
          minLength: 8
          maxLength: 128
          example: "SecureP@ss123"
        phone:
          type: string
          description: "Số điện thoại Việt Nam"
          pattern: "^(0[3|5|7|8|9])[0-9]{8}$"
          example: "0901234567"
          nullable: true
        role:
          type: string
          description: "Vai trò trong hệ thống"
          enum: ["ADMIN", "MANAGER", "MEMBER"]
          default: "MEMBER"
          example: "MEMBER"
        avatarUrl:
          type: string
          format: uri
          description: "URL ảnh đại diện. Chấp nhận HTTPS URL, file ≤5MB, định dạng JPG/PNG/WebP."
          maxLength: 2048
          nullable: true
          example: "https://cdn.example.com/avatars/usr_123.jpg"
```

### 14.5. Checklist mô tả API (trước khi merge)

**[BẮT BUỘC]** Mỗi endpoint mới hoặc thay đổi phải đáp ứng toàn bộ checklist sau. Pull request sẽ bị **reject** nếu thiếu bất kỳ mục nào:

- [ ] Có `summary` và `description` mô tả rõ mục đích, logic nghiệp vụ.
- [ ] Ghi rõ **quyền truy cập** (roles) cần thiết.
- [ ] Ghi rõ **rate limit** áp dụng.
- [ ] Mọi **path params** có description, type, format, example.
- [ ] Mọi **query params** có description, type, default, min/max, example.
- [ ] Mọi **request body fields** có description, type, constraints (min, max, pattern), example.
- [ ] Ghi rõ **required fields** vs optional fields.
- [ ] Liệt kê **tất cả status codes** có thể trả về (bao gồm cả error cases).
- [ ] Mỗi status code có **response example** đầy đủ.
- [ ] Enum values được liệt kê và mô tả.
- [ ] Nullable fields được đánh dấu `nullable: true`.

---

## 15. BẢO MẬT

### 15.1. Transport Security

**[BẮT BUỘC]** Chỉ chấp nhận HTTPS (TLS 1.2+). Từ chối mọi kết nối HTTP.

### 15.2. Input Validation

**[BẮT BUỘC]** Validate tất cả input phía server:

- Kiểm tra kiểu dữ liệu, độ dài, format.
- Sanitize dữ liệu để chống SQL Injection, XSS.
- Giới hạn kích thước request body: tối đa **1MB** (có thể tùy chỉnh theo endpoint).

### 15.3. CORS

**[BẮT BUỘC]** Cấu hình CORS chặt chẽ:

```http
Access-Control-Allow-Origin: https://app.mycompany.com
Access-Control-Allow-Methods: GET, POST, PUT, PATCH, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization, X-Request-Id
Access-Control-Max-Age: 86400
```

**[BẮT BUỘC]** Không bao giờ sử dụng `Access-Control-Allow-Origin: *` trên production.

### 15.4. Sensitive Data

**[BẮT BUỘC]** Không bao giờ trả về trong response:
- Password (kể cả dạng hash).
- Token bí mật, API keys.
- Thông tin nhạy cảm không thuộc phạm vi request (PII của user khác).

---

## 16. HIỆU NĂNG VÀ CACHING

### 16.1. Caching Headers

**[BẮT BUỘC]** Mọi GET response phải khai báo caching policy:

```http
# Resource ít thay đổi (ảnh, config tĩnh)
Cache-Control: public, max-age=86400
ETag: "v1-abc123"

# Resource thay đổi thường xuyên (danh sách, profile)
Cache-Control: private, no-cache
ETag: "v3-xyz789"

# Dữ liệu nhạy cảm (thông tin tài chính, sức khỏe)
Cache-Control: no-store
```

### 16.2. Compression

**[BẮT BUỘC]** Hỗ trợ nén response:

```http
# Request
Accept-Encoding: gzip, br

# Response
Content-Encoding: br
```

### 16.3. Timeout

**[BẮT BUỘC]** Quy định timeout chuẩn:

| Loại request | Timeout |
|---|---|
| API đọc (GET) | 10 giây |
| API ghi (POST, PUT, PATCH) | 30 giây |
| API xử lý nặng (report, export) | 60 giây |
| Long-running task | Trả về `202 Accepted` + polling endpoint |

---

## 17. IDEMPOTENCY

### 17.1. Nguyên tắc

**[BẮT BUỘC]** GET, PUT, DELETE phải luôn là idempotent (gọi nhiều lần cho kết quả giống nhau).

**[BẮT BUỘC]** POST endpoint quan trọng (thanh toán, tạo đơn hàng) phải hỗ trợ idempotency key:

```http
POST /orders
X-Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
Content-Type: application/json

{
  "productId": "prod_456",
  "quantity": 2
}
```

### 17.2. Hành vi khi nhận trùng Idempotency Key

- Nếu request trước đó **đã xử lý thành công**: trả về response cũ nguyên bản.
- Nếu request trước đó **đang xử lý**: trả về `409 Conflict`.
- Idempotency key hết hạn sau **24 giờ**.

---

## 18. CHECKLIST REVIEW API

**[BẮT BUỘC]** Mỗi API endpoint mới phải pass toàn bộ checklist dưới đây trước khi được phê duyệt:

### Thiết kế

- [ ] URL sử dụng danh từ số nhiều, chữ thường, kebab-case.
- [ ] Đúng HTTP method (GET/POST/PUT/PATCH/DELETE).
- [ ] Phân cấp URL không vượt quá 3 cấp.
- [ ] Versioning trong URL path (`/v1/`).

### Request

- [ ] Tất cả fields sử dụng camelCase.
- [ ] Ngày giờ theo ISO 8601 UTC.
- [ ] Có header `X-Request-Id`.
- [ ] Input validation phía server.

### Response

- [ ] Response bọc trong `data` wrapper.
- [ ] Có `meta.requestId` và `meta.timestamp`.
- [ ] Status code đúng ngữ cảnh.
- [ ] Error response tuân thủ cấu trúc chuẩn (mục 13).

### Phân trang & Lọc

- [ ] Endpoint danh sách có phân trang.
- [ ] `limit` có giá trị tối đa (≤100).
- [ ] Response có `meta.pagination` và `links`.

### Bảo mật

- [ ] HTTPS bắt buộc.
- [ ] Xác thực (Bearer token / API Key).
- [ ] Phân quyền (role-based).
- [ ] Rate limiting được áp dụng.
- [ ] Không trả về dữ liệu nhạy cảm.

### Documentation

- [ ] OpenAPI spec đầy đủ (mục 14).
- [ ] Mọi params có description, type, constraints, example.
- [ ] Mọi response codes có example.
- [ ] Logic nghiệp vụ được mô tả trong description.

---

**Phụ trách ban hành:** Engineering Team Lead
**Ngày hiệu lực:** 07/04/2026
**Lần cập nhật tiếp theo:** 07/10/2026
