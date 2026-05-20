# BookSwap — Storage and Cache Decisions

## Executive summary

BookSwap is a community book lending marketplace for members in Colombo apartment buildings. The backend needs book listings, catalogue search, borrow requests, loan tracking, notifications, weekly digest emails, and photo handling.

The design uses **Azure SQL** as the source of truth for structured business data, **Azure Blob Storage** for book photo binaries, **Azure Cache for Redis** for hot read paths, **Azure Service Bus** for asynchronous notification and digest work, **Azure Communication Services Email** for outbound email, and **Microsoft Entra External ID** for authentication.

---

## 1. Data inventory

| Data type | Example record | Volume estimate (1y) | Read/write ratio |
|-----------|----------------|----------------------|------------------|
| Member profile | member id, display name, building id, notification preference | ~2,000 members | Read-heavy |
| Private member contact data | email, address, phone number, Entra subject id | ~2,000 members | Restricted read |
| Book listing | title, author, ISBN, condition, availability, owner id | ~50,000 books across buildings | Read-heavy |
| Book photo metadata | book id, blob URL, content type, size | ~50,000 records | Read-heavy |
| Book photo binary | JPEG/PNG image | Up to 250 GB if all photos are 5 MB | Read-heavy |
| Borrow request | book id, borrower id, owner id, status, message | ~100,000 requests | Balanced |
| Loan record | book id, borrower id, owner id, status, due date | ~70,000 loans | Read-heavy after creation |
| Borrower history | previous loans for one book | Derived from loan records | Read-heavy |
| In-app notification | member id, type, title, body, read flag | ~200,000 notifications | Write-heavy then read |
| Weekly digest job | building id, week, status, generated at | 52 jobs per building | Low volume |
| Audit log | actor, action, entity id, timestamp, request id | ~300,000 events | Write-heavy append-only |
| Catalogue search cache | search/filter/page result | Short lived | Very read-heavy |
| Recently added books cache | top 10 newest books per building | Short lived | Very read-heavy |

---

## 2. Storage selection

| Data type | Chosen store | Why this store | Why not the alternatives |
|-----------|--------------|----------------|--------------------------|
| Member profile | Azure SQL | Structured data with building relationship and links to books/loans. | Cosmos DB flexibility is not needed; Redis is not durable. |
| Private contact data | Azure SQL restricted table/columns | Supports access control and helps prevent address/phone data leaking into API responses. | Document DB could accidentally return nested private data if projection is wrong. |
| Book listing | Azure SQL | Relational data: owners, books, borrow requests, loans. SQL indexes support title/author/ISBN/condition filters. | Blob cannot query fields; Redis should only cache; Cosmos DB is unnecessary at this scale. |
| Book photo metadata | Azure SQL | Small metadata belongs with the book and can be queried quickly. | Reading only blob metadata would slow listing pages. |
| Book photo binary | Azure Blob Storage | Photos are up to 5 MB; Blob Storage is cheaper and avoids database backup bloat. | SQL BLOBs make backups large; Redis memory is too expensive for images. |
| Borrow request | Azure SQL | Status transition must be durable and linked to book availability. | Queue is a transport, not the source of truth. |
| Loan record | Azure SQL | Loans require due-date queries, overdue status, and borrower history. | Redis is not durable; Blob is not suitable for structured records. |
| Borrower history | Azure SQL query/view over loans | History is derived from loan records and can be paginated. | Duplicating history risks inconsistency. |
| In-app notification | Azure SQL | Notifications must be persisted and marked read/unread. | Queue only carries the creation command. |
| Weekly digest job | Azure SQL + Azure Service Bus | SQL stores job status; Service Bus runs async processing. | Direct email in request path could block book listing creation. |
| Audit log | Azure SQL append-only table | Moderate volume and queryable by actor/entity/request ID. | Event Hubs/Kafka is overkill for beginner scale. |
| Catalogue search cache | Azure Cache for Redis | Search must be under 300 ms p95, so repeated searches can be returned quickly. | SQL remains the source of truth; cache is not durable. |
| Recently added books cache | Azure Cache for Redis | Home screen and weekly digest repeatedly need newest 10 books. | A duplicate SQL table is unnecessary. |

---

## 3. Source of truth

| Data | Source of truth |
|------|-----------------|
| Authentication identity | Microsoft Entra External ID |
| App member profile | Azure SQL |
| Book listing | Azure SQL |
| Book photo binary | Azure Blob Storage |
| Book photo metadata | Azure SQL |
| Borrow request | Azure SQL |
| Loan status and due date | Azure SQL |
| Notification read/unread state | Azure SQL |
| Digest job status | Azure SQL |
| Cached search result | Not source of truth; derived from Azure SQL |
| Queue message | Not source of truth; delivery mechanism only |

---

## 4. Cache plan

### What is hot enough to cache?

1. Catalogue search page 1 results by building, search term, condition, availability, page, and pageSize.
2. Recently added 10 books per building for home screen and weekly digest.
3. Single book detail only if a book becomes popular.

### When not to cache

Do not cache addresses or phone numbers. The privacy requirement says these must never be returned in API responses. Also do not cache pending borrow request decisions because members need the latest status.

### Cache-aside pseudocode

```javascript
async function searchBooks(buildingId, filters) {
  const cacheKey = `books:search:${buildingId}:${filters.search || ''}:${filters.condition || 'any'}:${filters.availability || 'any'}:${filters.page || 1}:${filters.pageSize || 20}`;

  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached);

  const page = await sql.query(
    `SELECT id, title, author, isbn, condition, availability, photo_url, owner_member_id
     FROM books
     WHERE building_id = @buildingId
       AND (@search IS NULL OR title LIKE @search OR author LIKE @search OR isbn LIKE @search)
       AND (@condition IS NULL OR condition = @condition)
       AND (@availability IS NULL OR availability = @availability)
     ORDER BY created_at DESC
     OFFSET @offset ROWS FETCH NEXT @pageSize ROWS ONLY`,
    filters
  );

  // 60 seconds is acceptable because catalogue results may be slightly stale.
  await redis.set(cacheKey, JSON.stringify(page), 'EX', 60);
  return page;
}

async function updateBook(bookId, patch) {
  const updated = await sql.updateBook(bookId, patch);
  await redis.del(`book:${bookId}`);
  await redis.delByPrefix(`books:search:${updated.buildingId}:`);
  await redis.del(`books:recent:${updated.buildingId}`);
  return updated;
}
```

### TTL and invalidation strategy

| Cache key | TTL | Invalidation |
|-----------|-----|--------------|
| `books:search:{buildingId}:{filters}` | 60 seconds | Delete by building prefix when book is created/updated/borrowed/returned |
| `books:recent:{buildingId}` | 5 minutes | Delete when a new book is listed in that building |
| `book:{bookId}` | 60 seconds | Delete when owner updates book or availability changes |

If Redis is down, the API should read Azure SQL directly. The system becomes slower but still works.

---

## 5. Queue plan

| Queue message | Producer | Consumer | Why queue is used |
|---------------|----------|----------|------------------|
| `book-created` | API service | Notification/digest worker | Listing creation must not fail if email service is down. |
| `borrow-request-created` | API service | Notification worker | Owner notification is a side effect after SQL save. |
| `borrow-request-decided` | API service | Notification worker | Borrower notification can be async. |
| `weekly-digest-requested` | Scheduled job | Digest worker | Weekly digest is best-effort and retryable. |
| `loan-overdue-detected` | Scheduled job | Notification worker | Overdue reminders are background work. |

### If the consumer is down for 30 minutes

Azure Service Bus keeps messages. The API continues accepting book listings and borrow requests because SQL writes still succeed. When the consumer returns, it processes the backlog. Repeated failures move to a Dead Letter Queue (DLQ), where operations can inspect and replay them.

### Retry and DLQ policy

- Retry transient email failures up to 5 times with exponential backoff.
- Move poison messages to DLQ after the maximum delivery count.
- Store final notification/digest status in Azure SQL.
- Avoid automatic retry for write endpoints unless an idempotency key is supplied.

---

## 6. 5xx retry policy

For safe read endpoints such as `GET /books`, clients may retry 500/503 responses up to 2 times with exponential backoff and jitter. For write endpoints such as `POST /books` and `POST /books/{bookId}/borrow-requests`, clients should only retry automatically when an `Idempotency-Key` header is supplied. Every response should include an `X-Request-Id` so logs, database operations, and queue messages can be correlated.

---

## 7. NFR mapping

| Non-functional requirement | Design response |
|----------------------------|-----------------|
| Search under 300 ms p95 | SQL indexes plus Redis cache for repeated searches |
| Listing creation succeeds if email is down | Email/digest work goes to Service Bus, not request path |
| Photos up to 5 MB, never in DB | Blob Storage stores binary; SQL stores metadata only |
| In-app notification within 2 seconds | Queue message is created immediately after SQL transaction |
| Email digest best-effort | Digest is async, retried, and can fail without blocking user actions |
| No addresses/phones in API responses | API schemas exclude private fields; private contact table is restricted |
