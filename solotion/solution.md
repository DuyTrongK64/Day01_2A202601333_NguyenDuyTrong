# K3 — Ngày 1: Bài Tập & Phản Ánh
## Khám Phá LLM API | Phiếu Thực Hành

**Thời lượng:** 9h00–13h00
**Cách làm:** Trả lời từng câu ngay sau khi hoàn thành block tương ứng —
đừng để dồn hết về cuối buổi. Thay phần giữ chỗ dưới mỗi câu bằng câu trả
lời thật (chấm tự động sẽ đếm số câu đã trả lời).

---

## Block 1 — API Cơ Bản (trả lời sau Checkpoint 1)

### Câu 1.1 — Độ nhạy của temperature
Gọi `call_gemini` với temperature 0.0, 0.5, 1.0 và 1.5 dùng prompt
**"Hãy kể cho tôi một sự thật thú vị về Việt Nam."**

**Bạn nhận thấy quy luật gì qua bốn phản hồi?** (2–3 câu)
> Khi temperature tăng từ 0.0 lên 1.5, phản hồi có xu hướng đa dạng hơn về
> cách diễn đạt, cách chọn sự thật và mức độ sáng tạo. Ở temperature 0.0,
> câu trả lời ổn định và trực tiếp hơn; từ 1.0–1.5, nội dung thú vị hơn nhưng
> cũng dễ lan man hoặc xuất hiện chi tiết cần được kiểm chứng.

### Câu 1.2 — Chọn temperature cho sản phẩm
**Bạn sẽ đặt temperature bao nhiêu cho chatbot hỗ trợ khách hàng, và tại sao?**
> Tôi chọn temperature khoảng 0.2. Chatbot hỗ trợ khách hàng cần câu trả lời
> nhất quán, chính xác và bám sát chính sách; một mức nhỏ hơn 0.5 vẫn cho phép
> diễn đạt tự nhiên nhưng hạn chế việc model tự sáng tạo thông tin.

### Câu 1.3 — Đánh đổi chi phí
Kịch bản: 10.000 người dùng hoạt động mỗi ngày, mỗi người gọi API 3 lần,
mỗi lần trung bình ~350 token đầu ra.

**Ước tính Gemini 2.5 Pro đắt hơn Gemini 2.5 Flash bao nhiêu lần cho workload
này? Nêu một trường hợp Pro xứng đáng với chi phí và một trường hợp nên dùng Flash:**
> Tổng đầu ra là `10.000 × 3 × 350 = 10.500.000` token/ngày. Theo bảng giá
> trong bài, Pro tốn khoảng `10.500 × 0,010 = 105 USD/ngày`, còn Flash khoảng
> `10.500 × 0,0025 = 26,25 USD/ngày`; như vậy Pro đắt gấp 4 lần nếu chỉ xét
> token đầu ra. Pro xứng đáng cho phân tích tài chính/pháp lý phức tạp cần
> suy luận tốt, còn Flash phù hợp cho FAQ, phân loại yêu cầu và trả lời hỗ
> trợ số lượng lớn.

---

## Block 2 — System Prompt & Token (trả lời sau Checkpoint 2)

### Câu 2.1 — Sức mạnh của persona
Gọi `chat_with_system_prompt` hai lần với cùng câu hỏi
**"Giải thích blockchain là gì?"** nhưng hai system prompt khác nhau:
- "Bạn là giáo viên tiểu học, giải thích thật đơn giản cho trẻ 8 tuổi."
- "Bạn là chuyên gia tài chính, trả lời chuyên sâu bằng thuật ngữ kỹ thuật."

**Hai phản hồi khác nhau như thế nào (độ dài, từ vựng, ví dụ)? System prompt
ảnh hưởng đến hành vi model ra sao?** (3–4 câu)
> Persona giáo viên tiểu học tạo phản hồi ngắn, dùng từ quen thuộc và thường
> ví blockchain như một cuốn sổ được nhiều người cùng giữ. Persona chuyên gia
> tài chính tạo câu trả lời dài hơn, dùng các thuật ngữ như sổ cái phân tán,
> cơ chế đồng thuận, tính bất biến và rủi ro thị trường. System instruction
> vì vậy không chỉ đổi giọng văn mà còn điều chỉnh độ sâu, cấu trúc, ví dụ và
> lượng kiến thức giả định ở người đọc.

### Câu 2.2 — tiktoken vs đếm từ
Chọn một đoạn văn tiếng Việt ~100 từ. So sánh số token theo `count_tokens`
(tiktoken) với ước lượng `số từ / 0.75` mà Part 1 đã dùng.

**Hai con số chênh nhau bao nhiêu phần trăm? Vì sao tiếng Việt thường tốn
nhiều token hơn tiếng Anh cùng độ dài?**
> Với một đoạn thử nghiệm 100 từ tiếng Việt dài khoảng 612 ký tự,
> `count_tokens` rơi về fallback và ước lượng `612 // 4 = 153` token, còn
> công thức số từ cho `100 / 0,75 ≈ 133` token. Hai kết quả lệch khoảng
> `(153 - 133) / 133 × 100 ≈ 15%`. Tiếng Việt có dấu, nhiều ký tự Unicode và
> một từ viết cách có thể gồm nhiều âm tiết; tùy tokenizer, các chuỗi này dễ
> bị tách thành nhiều token hơn những từ tiếng Anh phổ biến có sẵn trong từ
> vựng của tokenizer.

---

## Block 3 — Streaming & Độ Bền (trả lời sau Checkpoint 3)

### Câu 3.1 — Trải nghiệm người dùng với streaming
**Streaming quan trọng nhất trong trường hợp nào, và khi nào thì
non-streaming lại phù hợp hơn?** (1 đoạn văn)
> Streaming quan trọng nhất khi model sinh câu trả lời dài hoặc có độ trễ
> cao, chẳng hạn chatbot, trợ lý viết nội dung và giải thích tài liệu, vì
> người dùng thấy phản hồi ngay và cảm nhận hệ thống nhanh hơn. Non-streaming
> phù hợp hơn khi đầu ra ngắn, cần nhận trọn kết quả để kiểm tra schema,
> kiểm duyệt, ghi vào cơ sở dữ liệu hoặc xử lý tiếp trước khi hiển thị.

### Câu 3.2 — Vì sao backoff theo cấp số nhân?
**So với delay cố định (ví dụ luôn chờ 1 giây), exponential backoff có lợi
thế gì khi API bị quá tải? Điều gì xảy ra nếu hàng nghìn client cùng retry
với delay cố định giống nhau?**
> Exponential backoff tăng dần thời gian chờ sau mỗi lỗi, nhờ đó giảm áp lực
> lên API trong lúc quá tải nhưng vẫn thử lại nhanh ở lỗi tạm thời đầu tiên.
> Nếu hàng nghìn client đều retry sau đúng một giây, chúng có thể đồng loạt
> gửi lại request và tạo “thundering herd”, khiến server tiếp tục quá tải.
> Trong sản phẩm thật nên kết hợp backoff với jitter ngẫu nhiên để phân tán
> thời điểm retry giữa các client.

---

## Block 4 — Mini-Project (trả lời sau Checkpoint 4)

### Câu 4.1 — Thiết kế persona
**Bạn chọn persona gì cho trợ lý của mình? Viết lại system prompt đó và giải
thích 1–2 lựa chọn từ ngữ quan trọng trong prompt (ví dụ: vì sao yêu cầu
"trả lời ngắn gọn", vì sao chỉ định ngôn ngữ...):**
> Persona tôi chọn là: “Bạn là trợ giảng thân thiện của khóa AI, trả lời
> ngắn gọn bằng tiếng Việt. Giải thích thuật ngữ bằng ví dụ thực tế; nếu
> không chắc chắn, hãy nói rõ giới hạn thay vì đoán.” Cụm “ngắn gọn bằng
> tiếng Việt” giúp phản hồi phù hợp với người học và tiết kiệm token; yêu cầu
> “nói rõ giới hạn” giảm nguy cơ trình bày thông tin suy đoán như một sự thật.

### Câu 4.2 — Hạn chế & cải thiện
**Trợ lý của bạn hiện có hạn chế lớn nhất là gì (ví dụ: history chỉ 3 lượt,
không có bộ nhớ dài hạn, không kiểm duyệt nội dung...)? Đề xuất một cải
thiện cụ thể và mô tả ngắn cách triển khai:**
> Hạn chế lớn nhất là trợ lý chỉ giữ ba lượt gần nhất, nên mất ngữ cảnh quan
> trọng trong cuộc trò chuyện dài. Tôi sẽ bổ sung bộ nhớ tóm tắt: trước khi
> loại các lượt cũ, dùng Gemini tạo một bản tóm tắt ngắn gồm mục tiêu, dữ kiện
> và quyết định đã thống nhất; lưu bản tóm tắt riêng và đưa nó vào
> `system_instruction` hoặc phần đầu `contents` ở các lượt sau. Cách này giữ
> được ngữ cảnh dài hạn mà không làm số token tăng liên tục.

---

## Danh Sách Kiểm Tra Nộp Bài

- [x] `python grade.py` — xem điểm tự động, mục tiêu ≥ 75/100
- [x] Cả 4 checkpoint pytest đều pass
- [x] Tất cả 9 câu trong file này đã được trả lời
- [ ] Đã copy bài làm vào folder `solution/` và zip theo hướng dẫn README
