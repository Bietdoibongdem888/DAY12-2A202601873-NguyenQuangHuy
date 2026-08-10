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
| Public URL | `REQUIRES_REAL_RUN` — chưa có URL công khai |
| Platform | Railway (kế hoạch ưu tiên; chưa triển khai) |
| Ngày deploy | `REQUIRES_REAL_RUN` |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ⏳ | platform sẽ tự gán |
| `AGENT_API_KEY` | ⏳ | phải đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ⏳ | phải lấy từ Redis add-on của platform |
| `RATE_LIMIT_PER_MINUTE` | ⏳ | cấu hình dự kiến: 10 |
| `MONTHLY_BUDGET_USD` | ⏳ | cấu hình dự kiến: 10.0 |
| `LOG_LEVEL` | ⏳ | cấu hình dự kiến: INFO |

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
REQUIRES_REAL_RUN — chưa có output từ service cloud.
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl

---

## Nếu Dùng Phương Án Dự Phòng

Không đăng ký được tài khoản cloud? Vẫn nộp được bài, nhưng CP5 tối đa 60% điểm:

1. Đặt `LOCAL_FALLBACK=true` trong `.env`
2. Chạy `docker compose up -d` rồi kiểm tra `docker compose ps`
3. Chụp màn hình vào `screenshots/`
4. Chạy `pytest tests/test_cp5.py -v` — bộ test sẽ tự chuyển sang kiểm tra
   `http://localhost:8000`
5. Ghi rõ lý do không deploy được vào phần dưới đây:

Không bật phương án dự phòng vì Docker daemon không chạy và ổ C chỉ còn
khoảng 270 MiB trống, không đủ an toàn để build image hoặc chạy stack.
Trạng thái này được ghi rõ để không tạo bằng chứng deploy giả.
