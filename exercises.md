# Phiếu Phản Ánh — K3 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay từng dòng mẫu bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Quang Huy  Mã học viên: 2A202601873

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Ví dụ cụ thể: khi deploy một revision mới nhưng quên cấu hình
> `AGENT_API_KEY`, tiến trình dừng ngay và platform giữ revision cũ đang khỏe.
> Nếu ứng dụng tự dùng một khóa mặc định công khai, revision mới vẫn nhận
> traffic và người lạ có thể gọi `/ask`, tiêu quota và ngân sách trước khi tôi
> phát hiện cấu hình sai.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Dòng lấy từ lần gọi `/ask` thực tế bằng `TestClient` ngày 2026-08-10:
>
> ```json
> {"user_id":"sv-observe","tokens_in":5,"tokens_out":37,"cost_usd":2.295e-05,"event":"ask_completed","level":"info","timestamp":"2026-08-10T02:42:42.986693+00:00"}
> ```
>
> Từ log này tôi có thể (1) lọc/tổng hợp chi phí theo `user_id` và (2) tạo
> metric hoặc cảnh báo theo `level`, số token và khoảng thời gian. Một câu
> `print("đã trả lời xong")` không có các trường có cấu trúc để làm hai việc đó.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | `REQUIRES_REAL_RUN` |
| Multi-stage | `REQUIRES_REAL_RUN` |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> `REQUIRES_REAL_RUN`: Docker daemon không chạy và ổ C chỉ còn khoảng 270 MiB
> trống nên chưa có số đo đáng tin cậy. Cần giải phóng dung lượng, build lại
> Dockerfile single-stage từ commit ban đầu và
> Dockerfile hiện tại rồi chép nguyên kết quả `docker images` vào bảng. Về mặt
> cấu tạo, phần chênh lệch dự kiến đến từ base image đầy đủ, cache cài đặt và
> công cụ build không được mang sang runtime ở bản multi-stage; đây không phải
> là số đo cho tới khi hai image được build thật.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> `REQUIRES_REAL_RUN`: cần chạy hai lần build liên tiếp để ghi lại dòng
> `CACHED` thực tế. Theo cấu trúc Dockerfile hiện tại, các layer base,
> `COPY requirements.txt` và `pip install` có thể được dùng lại khi chỉ source
> đổi; các layer `COPY app`, `COPY utils` và layer sau chúng phải tạo lại. Nếu
> `COPY . .` nằm trước `pip install`, thay đổi source làm mất cache dependency
> và buộc cài lại toàn bộ package.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu lỗ hổng Python cho phép thực thi lệnh trong container, tiến trình chạy
> root cho kẻ tấn công quyền root bên trong container. Kết hợp với mount nhạy
> cảm, capability thừa hoặc lỗi container runtime, quyền đó có thể được dùng
> để sửa dữ liệu host hoặc thoát container. `USER appuser` cắt chuỗi ngay tại
> bước đầu: code bị chiếm quyền chỉ có UID 10001 với quyền tối thiểu, không có
> quyền root để thao tác các tài nguyên đặc quyền.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Tối đa 20 request: gửi 10 request ở cuối phút, ví dụ 10:00:59, rồi gửi 10
> request ngay sau khi bộ đếm reset ở 10:01:00. Hai nhóm đều hợp lệ theo từng
> phút đồng hồ nhưng dồn vào khoảng hai giây. Sliding window nhìn lại đúng 60
> giây nên nhóm thứ hai bị chặn.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn tần suất trong cửa sổ 60 giây; cost guard giới hạn tổng
> tiền của từng user trong tháng. Một request rất dài vẫn có thể nằm trong hạn
> tần suất nhưng bị cost guard chặn vì chi phí dự kiến vượt ngân sách. Ngược
> lại, nhiều request rất rẻ có thể chưa đáng kể về tiền nhưng request tiếp theo
> vẫn bị rate limiter chặn vì đến quá nhanh.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Redis mất kết nối → endpoint gộp trả 503 → orchestrator hiểu nhầm process đã
> chết và lần lượt restart cả ba container → các container mới vẫn không nối
> được Redis nên tiếp tục unhealthy/restart → cụm mất toàn bộ capacity dù tiến
> trình web vốn vẫn sống. Tách `/health` khỏi Redis giữ container chạy, còn
> `/ready` trả 503 chỉ để load balancer tạm ngừng chuyển traffic.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> `REQUIRES_REAL_RUN`: test tích hợp với hai `ConversationStore` dùng chung
> fake Redis đã xác nhận lượt hai có `history_length = 2`, nhưng chưa phải quan
> sát `docker compose --scale agent=3`. Cần chạy stack ba instance qua load
> balancer và ghi chuỗi giá trị thực. Với Redis dùng chung, giá trị phải tăng
> đều 0, 2, 4...; nếu dùng dict riêng, request đổi instance sẽ thấy các chuỗi
> tăng rời rạc hoặc tụt về 0.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> `REQUIRES_REAL_RUN`: chưa có credential và chưa thực hiện deploy cloud nên
> chưa có lỗi deploy thật để báo cáo. Khi deploy, cần chép nguyên thông báo lỗi,
> nguồn bằng chứng trong build/runtime log, nguyên nhân đã xác định và thay đổi
> khắc phục; không thay bằng một tình huống giả định.
