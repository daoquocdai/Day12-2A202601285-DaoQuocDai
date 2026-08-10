# Phiếu Phản Ánh — K3 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
> Họ và tên: Đào Quốc Đại  Mã học viên: 2A202601285

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Khi deploy lên Railway, nếu tôi quên cấu hình AGENT_API_KEY mà chương trình vẫn dùng mặc định "changeme", service vẫn có thể khởi động và vô tình cho phép người biết giá trị mặc định truy cập API. Việc để agent_api_key là trường bắt buộc làm ứng dụng lỗi ngay lúc khởi động, nhờ đó tôi phát hiện thiếu cấu hình từ deploy log trước khi service được đưa ra Internet. Theo tôi đây an toàn hơn nhiều so với việc để ứng dụng chạy với một secret mặc định yếu.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

{
  "message": "",
  "severity": "info",
  "attributes": {
    "event": "ask_completed",
    "cost_usd": 0.00003465,
    "timestamp": "2026-08-10T05:23:25.997781+00:00",
    "user_id": "sv-test",
    "tokens_in": 43,
    "tokens_out": 47,
    "level": "info"
  },
  "tags": {
    "project": "facadcb9-5ebf-4c2d-9c78-1a5e0236c542",
    "environment": "849cfa29-24e6-46fe-bb49-f8a8936cac06",
    "service": "1df6764f-7528-474d-8694-d6fda330e396",
    "deployment": "ba132c2f-fc01-43b9-ba1f-681fb919c247",
    "replica": "a03d2cbc-9be4-4342-904a-1367c9fc85ec"
  },
  "timestamp": "2026-08-10T05:23:34.681756396Z"
} 
>Từ dòng log JSON này, tôi có thể lọc và tìm kiếm request theo các trường cụ thể như user_id, event hoặc level, ví dụ tìm toàn bộ request của sv-test hoặc chỉ các event ask_completed. 
>Ngoài ra, tôi có thể thống kê chi phí và số token từ các trường cost_usd, tokens_in, tokens_out để theo dõi mức sử dụng và tạo dashboard. Nếu chỉ dùng print("đã trả lời xong") thì tôi không biết request thuộc user nào, tốn bao nhiêu token hay chi phí bao nhiêu.

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
| 1 stage (bản đầu) | 1.69 GB |
| Multi-stage | 270 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Bản 1-stage của tôi có dung lượng 1.69 GB, còn bản multi-stage chỉ 270 MB, tức giảm khoảng 1.42 GB. Phần chênh lệch chủ yếu đến từ việc bản 1-stage dùng python:3.11 đầy đủ và giữ lại toàn bộ môi trường build cùng các thành phần hệ thống không cần thiết khi chạy production. Bản multi-stage dùng python:3.11-slim và chỉ copy các dependency đã cài từ builder sang runtime stage, nên không mang theo các thành phần build thừa. Ngoài ra, việc dùng --no-cache-dir cũng tránh giữ pip cache trong image cuối.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Docker cache layer theo thứ tự. Với Dockerfile của tôi, requirements.txt được copy và pip install chạy trước khi copy source code. Vì vậy khi tôi chỉ sửa một ký tự trong app/main.py, các layer từ base image, WORKDIR, COPY requirements.txt và RUN pip install vẫn có thể dùng cache; Docker chủ yếu phải chạy lại từ layer copy source code trở xuống. Nếu đặt COPY . . trước RUN pip install, chỉ cần sửa một file Python cũng làm layer COPY . . thay đổi, khiến layer cài dependency phía sau bị invalid cache và phải chạy pip install lại dù requirements.txt không thay đổi.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu code Python có lỗ hổng cho phép kẻ tấn công thực thi lệnh, process bị khai thác sẽ có cùng quyền với user đang chạy ứng dụng. Nếu container chạy bằng root thì attacker có quyền root bên trong container, từ đó mức độ ảnh hưởng lớn hơn, đặc biệt nếu container còn được cấu hình nguy hiểm như mount thư mục host, cấp quyền đặc biệt hoặc tồn tại lỗ hổng container escape. USER appuser làm process Uvicorn chạy với một user ít quyền hơn, nên ngay cả khi ứng dụng bị khai thác thì attacker không có sẵn quyền root trong container. Nó không đảm bảo tuyệt đối rằng host không thể bị tấn công, nhưng cắt giảm đáng kể quyền và phạm vi thiệt hại.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Với fixed window và giới hạn 10 request/phút, user có thể gửi tối đa 20 request trong khoảng 2 giây quanh ranh giới phút. Ví dụ gửi 10 request vào lúc 10:00:59, sau đó khi đồng hồ chuyển sang phút mới thì counter reset và gửi tiếp 10 request lúc 10:01:00. Mỗi phút riêng vẫn chỉ có 10 request nhưng thực tế hệ thống phải nhận 20 request gần như liên tiếp. Sliding window tránh lỗ hổng này vì tại 10:01:00 nó vẫn nhìn lại toàn bộ 60 giây gần nhất và thấy các request vừa gửi ở giây 59.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn tốc độ/số request trong một khoảng thời gian, còn cost guard giới hạn tổng tiền mà user được phép tiêu. Ví dụ một user chỉ gửi 1 request mỗi phút nên rate limit cho qua, nhưng mỗi request xử lý tài liệu rất lớn và tốn nhiều token; khi tổng chi phí tháng vượt budget thì cost guard phải trả 402. Trường hợp ngược lại, user có thể gửi 15 request rất nhỏ trong vài giây, tổng chi phí vẫn thấp nên cost guard chưa chặn nhưng rate limiter sẽ chặn các request vượt giới hạn và trả 429.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu gộp /health và /ready thành một endpoint có kiểm tra Redis, khi Redis mất kết nối thì cả 3 container đều bắt đầu báo endpoint lỗi. Orchestrator có thể coi các process là không còn sống và restart cả 3 container. Các container mới khởi động lên nhưng Redis vẫn đang lỗi nên lại tiếp tục fail health check và có thể bị restart tiếp, tạo thành vòng restart dù bản thân FastAPI không bị chết. Khi tách hai endpoint, /health vẫn trả 200 để báo process còn sống, còn /ready trả 503 để load balancer tạm ngừng gửi traffic vào instance. Khi Redis hoạt động lại, /ready trở về 200 mà không cần restart toàn bộ ứng dụng.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Khi tôi gọi /ask nhiều lần với cùng một X-User-Id, history_length tăng dần và ở request thứ 5 tôi thấy giá trị là 5. Điều này vẫn hoạt động khi có nhiều replica vì lịch sử được lưu trong Redis và các container cùng nhìn thấy một state chung. Nếu lưu bằng dict Python thì mỗi container sẽ có bộ nhớ riêng. Với 3 replica và Nginx round-robin, năm request có thể cho kết quả kiểu 1, 1, 1, 2, 2 thay vì 1, 2, 3, 4, 5, vì request thứ hai có thể rơi vào container chưa từng xử lý user đó.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Khi deploy agent lên Railway, container liên tục restart và log báo Error: Invalid value for '--port': 'PORT' is not a valid integer. Ban đầu tôi kiểm tra phần Custom Start Command trên Railway nhưng nó đang để trống. Sau đó tôi tìm trong cấu hình project và phát hiện railway.toml có startCommand = "uvicorn app.main:app --host 0.0.0.0 --port $PORT". Railway chạy command đó theo cách khiến $PORT không được shell expand, nên Uvicorn nhận nguyên chuỗi $PORT thay vì một số port. Tôi xóa startCommand trong railway.toml và để Railway sử dụng CMD của Dockerfile: CMD ["sh", "-c", "exec uvicorn app.main:app --host 0.0.0.0 --port {PORT:-8000}"]. Sau khi push và redeploy, service khởi động thành công và /health trên URL public trả về 200.
