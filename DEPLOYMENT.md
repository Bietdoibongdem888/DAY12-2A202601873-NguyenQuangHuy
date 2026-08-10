# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyễn Quang Huy |
| Mã học viên | 2A202601873 |
| Repo | https://github.com/Bietdoibongdem888/DAY12-2A202601873-NguyenQuangHuy |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-z2cx.onrender.com |
| Platform | Render — Docker Web Service + Key Value (Valkey) |
| Ngày deploy | 2026-08-10 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | Render tự gán lúc chạy web service |
| `AGENT_API_KEY` | ✅ | generated secret trên Render; không nằm trong repo |
| `REDIS_URL` | ✅ | private connection URL của `day12-redis` |
| `RATE_LIMIT_PER_MINUTE` | ✅ | cấu hình trên Render: 10 |
| `MONTHLY_BUDGET_USD` | ✅ | cấu hình trên Render: 10.0 |
| `LOG_LEVEL` | ✅ | cấu hình trên Render: INFO |

## Lệnh Kiểm Tra

Gán Public URL thật vào biến `PUBLIC_URL` trước khi chạy các lệnh dưới đây:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i "$PUBLIC_URL/health"

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i "$PUBLIC_URL/ready"

# 3. Không có API key — mong đợi 401
curl -i -X POST "$PUBLIC_URL/ask" \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST "$PUBLIC_URL/ask" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST "$PUBLIC_URL/ask" \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:

```text
GET /health
HTTP 200
{"status":"ok","service":"day12-agent","version":"1.0.0"}

GET /ready
HTTP 200
{"status":"ready","redis":true}

POST /ask (không gửi X-API-Key)
HTTP 401
{"detail":"invalid or missing API key"}

POST /ask (gửi đúng X-API-Key; giá trị khóa không được ghi lại)
HTTP 200
answer_nonempty=true

Rate limit với user riêng, 11 request liên tiếp
[200, 200, 200, 200, 200, 200, 200, 200, 200, 200, 429]
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl

Không dùng phương án local fallback; các kết quả trên được gọi trực tiếp qua
HTTPS tới service Render công khai.
