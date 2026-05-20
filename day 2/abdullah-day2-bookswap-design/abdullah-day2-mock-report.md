# BookSwap — Mock Smoke Test Report

## Setup

OpenAPI file:

```bash
openapi/bookswap-openapi.yaml
```

Validation command:

```bash
npx @apidevtools/swagger-cli validate openapi/bookswap-openapi.yaml
```

Prism command used:

```bash
npx @stoplight/prism mock openapi/bookswap-openapi.yaml --port 4010
```

Mock base URL:

```text
http://localhost:4010
```

Authenticated request header:

```http
Authorization: Bearer mock-valid-jwt
Content-Type: application/json
```

## Results Summary

| Metric | Target | Achieved |
|--------|--------|----------|
| Tests run | 5 | 5 |
| Tests passing against the mock | 5 | 5 |
| Endpoints with explicit error responses | 4+ | 9+ |

## Smoke test results

| # | Endpoint | Method | Body / Params | Expected status | Actual status |
|---:|----------|--------|---------------|----------------:|--------------:|
| 1 | `/books` | GET | `page=1&pageSize=20` with Authorization header | 200 | 200 |
| 2 | `/books` | POST | valid book payload | 201 | 201 |
| 3 | `/books` | POST | missing required `title` field | 400 or 422 | 422 |
| 4 | `/books/book_999/borrow-requests` | POST | borrower JWT with message body | 201 | 201 |
| 5 | `/books` | GET | no Authorization header | 401 | 401 |

## Commands

### 1. List books

```bash
curl -i "http://localhost:4010/books?page=1&pageSize=20"   -H "Authorization: Bearer mock-valid-jwt"
```

### 2. Create book with valid payload

```bash
curl -i -X POST "http://localhost:4010/books"   -H "Authorization: Bearer mock-valid-jwt"   -H "Content-Type: application/json"   -d '{
    "title": "The Pragmatic Programmer",
    "author": "David Thomas and Andrew Hunt",
    "isbn": "9780201616224",
    "condition": "like_new",
    "availability": "available",
    "photo": {
      "fileName": "pragmatic-programmer.png",
      "contentType": "image/png",
      "sizeBytes": 3400000
    }
  }'
```

### 3. Create book with missing title

```bash
curl -i -X POST "http://localhost:4010/books"   -H "Authorization: Bearer mock-valid-jwt"   -H "Content-Type: application/json"   -d '{
    "author": "David Thomas and Andrew Hunt",
    "isbn": "9780201616224",
    "condition": "like_new",
    "availability": "available"
  }'
```

### 4. Request to borrow a book

```bash
curl -i -X POST "http://localhost:4010/books/book_999/borrow-requests"   -H "Authorization: Bearer mock-valid-jwt"   -H "Content-Type: application/json"   -d '{ "message": "Can I borrow this for two weeks?" }'
```

### 5. List books without Authorization

```bash
curl -i "http://localhost:4010/books?page=1&pageSize=20"
```

## Findings

1. The mock showed that the `bookId` path pattern should be consistent. I used `book_1001` style IDs instead of plain numbers, so Book IDs, Borrow Request IDs, and Loan IDs are easy to identify.
2. The missing-title negative test showed why a `422 ValidationError` schema is useful. It gives field-level validation errors rather than only a generic `400 Bad Request`.
3. The `/books` endpoint is easy to call because it uses noun resources and filters instead of action URLs like `/searchBooks`.
4. The decision action is best represented as `PATCH /borrow-requests/{borrowRequestId}` with a `decision` field, not `POST /acceptBorrowRequest`.

## Spec changes I would make

1. Add an explicit reusable `Idempotency-Key` header parameter for write endpoints such as `POST /books` and `POST /books/{bookId}/borrow-requests`.
2. Add `PATCH /notifications/{notificationId}` later so members can mark notifications as read.
3. Add more response examples for `409 Conflict`, especially for duplicate borrow requests and unavailable books.

## Reproducibility

Another intern can reproduce this report by installing Node.js 20 LTS, running the validation command, starting the Prism mock server, and running the five curl commands listed above.
