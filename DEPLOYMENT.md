# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Họ và tên | Phạm Thị Ngọc Anh |
| Mã học viên | 2A202602831 |
| Repo | https://github.com/ngocanhpham-hust/K4-L3B-DAY12-PhamThiNgocAnh-2A202602831-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-7ncz.onrender.com |
| Platform | Render |
| Ngày deploy | 29/06/2026 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | Render tự gán lúc chạy service |
| `AGENT_API_KEY` | ✅ | Render Environment, không nằm trong repository |
| `REDIS_URL` | ✅ | Internal connection string từ Render Key Value `day12-redis` |
| `RATE_LIMIT_PER_MINUTE` | ✅ | Khai báo trong `render.yaml` |
| `MONTHLY_BUDGET_USD` | ✅ | Khai báo trong `render.yaml` |
| `LOG_LEVEL` | ✅ | Khai báo trong `render.yaml` |

## Lệnh Kiểm Tra

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i https://day12-agent-7ncz.onrender.com/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i https://day12-agent-7ncz.onrender.com/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST https://day12-agent-7ncz.onrender.com/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST https://day12-agent-7ncz.onrender.com/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $DEPLOY_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — 10 request đầu được nhận, các request sau trả 429
TEST_USER="rate-test-$(date +%s)"
for i in $(seq 1 12); do
  curl -s -o /dev/null -w "%{http_code} " \
    -X POST https://day12-agent-7ncz.onrender.com/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $DEPLOY_API_KEY" \
    -H "X-User-Id: $TEST_USER" \
    -d '{"question":"test"}'
done
echo
```

## Kết Quả Chạy Thật

```
GET /health
HTTP/2 200
{"status":"ok","service":"day12-agent","version":"1.0.0"}

GET /ready
HTTP/2 200
{"status":"ready","redis":true}

POST /ask không có X-API-Key
HTTP/2 401
{"detail":"invalid or missing API key"}

POST /ask có X-API-Key hợp lệ
HTTP/2 200
{"answer":"Câu hỏi hay. Deploy là gì thường được giải quyết bằng cách chuẩn hóa môi trường chạy: cùng một image chạy giống nhau ở laptop và trên cloud.","user_id":"sv-deploy-check","history_length":0,"cost_usd":2.145e-05,"tokens":{"in":3,"out":35}}

Rate limit (12 request liên tiếp với cùng một user mới)
200 200 200 200 200 200 200 200 200 200 429 429
```

Các kết quả trên được kiểm tra trực tiếp với public URL. `DEPLOY_API_KEY` chỉ
nằm trong `.env` cục bộ; tài liệu không chứa giá trị key.

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl

---

## Phương Án Dự Phòng

Không sử dụng. Bài được deploy trực tiếp trên Render bằng public URL ở trên.
