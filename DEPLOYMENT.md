# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Đào Quốc Đại |
| Mã học viên | 2A202601285 |
| Repo | https://github.com/daoquocdai/Day12-2A202601285-DaoQuocDai.git |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-production-a798.up.railway.app |
| Platform | Railway |
| Ngày deploy | 10/08/2026 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `AGENT_API_KEY` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | Redis add-on của Railway, cấp tự động cho service |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

Thay `<URL>` bằng Public URL ở trên:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i <URL>/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i <URL>/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST <URL>/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST <URL>/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST <URL>/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:

```
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i <URL>/health

HTTP/1.1 200 OK
Content-Type: application/json
Date: Mon, 10 Aug 2026 04:49:13 GMT
Server: railway-hikari
x-railway-request-id: Q1-2EPD7RJaZCtUG9fVATg
Content-Length: 57
x-hikari-trace: sin1.tr00
x-railway-edge: sin1
Connection: keep-alive

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i <URL>/ready

railway.app/ready
HTTP/1.1 200 OK
Content-Type: application/json
Date: Mon, 10 Aug 2026 05:02:20 GMT
Server: railway-hikari
x-railway-request-id: i2Yrb_1VTt-fmIZk9o6EoQ
Content-Length: 31
x-hikari-trace: sin1.d1nj
x-railway-edge: sin1
Connection: keep-alive

{"status":"ready","redis":true}

# 3. Không có API key — mong đợi 401
curl -i -X POST <URL>/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

HTTP/1.1 401 Unauthorized
Content-Type: application/json
Date: Mon, 10 Aug 2026 05:04:58 GMT
Server: railway-hikari
x-railway-request-id: tg748O6IQFSHKwKsljLL4A
Content-Length: 39
x-hikari-trace: sin1.nzn2
x-railway-edge: sin1
Connection: keep-alive

{"detail":"invalid or missing API key"}

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST <URL>/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

HTTP/1.1 200 OK
Content-Type: application/json
Date: Mon, 10 Aug 2026 05:06:56 GMT
Server: railway-hikari
x-railway-request-id: 9jWnGnxmRW2QPeRxljLL4A
Content-Length: 287
x-hikari-trace: sin1.nzn2
x-railway-edge: sin1
vary: accept-encoding
Connection: keep-alive

{"answer":"Ngắn gọn: Deploy la gi phụ thuộc vào ba yếu tố — cấu hình qua biến môi trường, health check để orchestrator biết trạng thái, và giới hạn tài nguyên.","user_id":"sv-test","history_length":1,"cost_usd":2.265e-05,"tokens":{"in":3,"out":37}}

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST <URL>/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'

200 200 200 200 200 200 200 200 200 200 429 429 429 429 429 

```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl
---

